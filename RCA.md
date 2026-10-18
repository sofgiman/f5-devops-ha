# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output
```
15.24 Configuration file '/etc/nginx/nginx.conf'
15.24  ==> File on system created by you or by a script.
15.24  ==> File also in package provided by package maintainer.
15.24    What would you like to do about it ?  Your options are:
15.24     Y or I  : install the package maintainer's version
15.24     N or O  : keep your currently-installed version
15.24       D     : show the differences between the versions
15.24       Z     : start a shell to examine the situation
15.24  The default action is to keep your current version.
15.24 *** nginx.conf (Y/I/N/O/D/Z) [default=N] ? dpkg: error processing package nginx-common (--configure):
15.24  end of file on stdin at conffile prompt
```

### How I diagnosed it
Ran the build (`docker build ./proxy`) and it died with non-zero exit code.
The last line just said "dpkg returned an error code (1)", which wasn't helpful - scrolled up and found the real error.  

### Root cause
When you install nginx, it installs 'nginx-common' package which create a default `nginx.conf` in `/etc/nginx/nginx.conf`.  
But because we copied this file in the first Dockerfile instruction (to `/etc/nginx/nginx.conf`),
the package manager asked the user whether to keep our file or the package's file - the docker build doesn't have an answer as there is no terminal, so the build fails. 

### Fix
First, install the packages, then copy the `nginx.conf`.

---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output
```
curl -i -s http://localhost:8081/api/info 
HTTP/1.1 502 Bad Gateway
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 18 Oct 2026 07:40:18 GMT
Content-Type: text/html
Content-Length: 166
Connection: keep-alive
```

### How I diagnosed it
Asked Claude how to demonstrate the error, as a following:
1. run `docker compose up -d --build backend proxy`
2. run `curl -s http://localhost:8081/nginx-health` and  `curl -i -s http://localhost:8081/api/info` to see nginx default route succeed and the other one (reverse proxy for backend) fails.

### Root cause
The following blocks were in the `nginx.conf`:
```
    upstream app_backend {
        server 127.0.0.1:8080;
    }
```
and
```
    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://app_backend;
```

This two blocks means, that when a http request comes to `nginx`, then forward to the `app_backend` upstream happens.  
Which means to forward to 127.0.0.1:8080, but the proxy container lives inside its own network namespace, which means 127.0.0.1 points back to itself.   
So the `nginx` forwards to itself and not to the backend container (and the proxy doesn't have a service listening in 8080).

### Fix
The fix is to utilize `docker compose` abilities, and change the previous upstream block to the following:
``` 
    upstream app_backend {
        server backend:8080;
    }
```

`backend:8080` reaches the backend container because of docker compose, that puts all services inside a shared network.  
And runs built-in DNS resolver to its services names. 

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output
run `docker compose up` and saw:

```
test-runner-1  | Starting integration tests against http://proxy:80...
test-runner-1  | Testing /nginx-health...
test-runner-1  | PASSED: /nginx-health
proxy-1        | 192.168.156.4 - - [18/Oct/2026:08:39:13 +0000] "GET /nginx-health HTTP/1.1" 200 15 "-" "Python-urllib/3.11"
test-runner-1  | Testing proxied backend /api/info...
proxy-1        | 2026/10/18 08:39:13 [error] 7#7: *2 connect() failed (111: Connection refused) while connecting to upstream, client: 192.168.156.4, server: localhost, request: "GET /api/info HTTP/1.1", upstream: "http://192.168.156.2:8080/api/info", host: "proxy:80"
proxy-1        | 192.168.156.4 - - [18/Oct/2026:08:39:13 +0000] "GET /api/info HTTP/1.1" 502 166 "-" "Python-urllib/3.11"
test-runner-1  | TEST RUNNER CRASHED: HTTP Error 502: Bad Gateway
```

### How I diagnosed it
Asked Claude to find where should I look into, and what commands to run to see the error.

### Root cause
The test-runner sends http requests to the backend service (through the proxy). It depends on `proxy` service, which also depends on `backend`.  
But, the default `depends_on` just check the container is up and not if it's ready. The backend has a delay because of the following block:
```
    for key, value in _CONFIG_STORE.items():
        time.sleep(CONFIG_FETCH_SECONDS)
        print(f"[Backend] loaded config key: {key}", flush=True)
        config[key] = value
    return config
```

This blocks waits a few seconds in between a config load, to demonstrate latency. So when the test runner runs, it goes to:
```
    req = urllib.request.Request(f"{PROXY_URL}/api/info")
    with urllib.request.urlopen(req, timeout=3) as res:
```
Which sends the http request to the backend before its able to receive http requests, causing the failure.

### Fix, and why I chose it over the alternatives

The fix I chose is to add a `healtcheck` to the backend service in docker compose:
```
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')"]
      interval: 2s
      timeout: 3s
      retries: 10
      start_period: 12s
```
This healthcheck will make sure the `/health` api on the app is available and sends a `200` status code.
It will also start the healthcheck after 12 seconds to allow the delay.  
Moreover, to make sure the healthcheck is meaningful, I added the following block under the proxy container:
```
    depends_on:
      backend:
        condition: service_healthy
```
This means, the proxy will be up only after the backend is fully health, which will also affect the `test runner` which depends on the `proxy`.

I also applied the same healthcheck pattern to the proxy (probing `/nginx-health`) and made the test-runner wait on `condition: service_healthy` for the proxy, so the whole chain (test-runner -> proxy -> backend) waits for readiness and not just container start.

I chose this fix over 2 alternatives:
1. A wait-for-ready loop in the test runner's entrypoint:  
I chose my fix over this one, because it will try to re-implement a readiness logic, that `docker compose` already has builtin
2. Add environment variable `CONFIG_FETCH_SECONDS` with the value `0`:  
This would make the delay much faster (around 0 seconds), but its a workaround and not a real fix.   
Cause real life could have real delays. Moreover, even in `app.py`, it's said that in real life, the conf will come from a remote service, making this option not valid. 

---

## Defect 4: CI pipeline fails

### Symptom / error output
running `./scripts/run-tests.sh` both in terminal and in CI actions, results in error output:  
`/home/runner/work/_temp/adeef573-07a1-4239-8bd1-d96cb809ef67.sh: line 1: ./scripts/run-tests.sh: Permission denied`  
`zsh: permission denied: ./scripts/run-tests.sh`

### How I diagnosed it
Knew the error is on the CI, so looked at error messages. Ran also in my terminal and verified with Claude.

### Root cause
the `./scripts/run-tests.sh` doesn't have the `x` (execution bit) on the file, meaning no one can execute the file.
You can see it with the following command 
```
 git ls-files -s scripts/run-tests.sh
100644 ab1f9a3696f5fff1a004f335bbe2cfaea626dc46 0	scripts/run-tests.sh
```
`100` is for a regular files, but `644` means the permission are:
- u(user/owner)  - write + read
- g(group)       - read
- o(other)       - read

No `x` bit in any slot, so no one can execute it - not CI, not even the owner locally.
### Fix
Add the `x` bit for the file, using the following commands:  
```
chmod +x scripts/run-tests.sh  
git add scripts/run-tests.sh
```
This will make the `git ls-files -s` permissions to be `100755` instead of `100644`

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

> 1. Minimize Proxy Image

### What I changed
- Swapped the base image from `ubuntu:24.04` to the official `nginx:alpine`.
- Dropped `htop`, `vim`, and `git` - interactive/debug tools with no job inside
  a running proxy (git alone pulled in ~57MB of perl).
- Kept only `curl`, installed with `apk add --no-cache curl`, because the compose
  healthcheck probes `/nginx-health` with `curl -fsS`. `--no-cache` leaves no
  package index behind in the layer.
- Nginx now comes pre-installed and pre-configured from the upstream image
  instead of via `apt-get install nginx`.

### Image size: before and after
before, the image size was:
```
idan74@MacBook-Air-sl-ydn f5-devops-ha % docker images | grep f5-devops-ha-proxy
f5-devops-ha-proxy ... latest ... 9fb892b7034d   2 hours ago .. 325MB
```
and now:
```
idan74@MacBook-Air-sl-ydn f5-devops-ha % docker images | grep f5-devops-ha-proxy
f5-devops-ha-proxy .. latest ... 97313c10b3fb   5 minutes ago ... 61.7MB
```

325MB shrunk into 61.7MB.

### Update (after the non-root bonus below)
Swapped the base to `nginxinc/nginx-unprivileged:stable-alpine` to run as non-root.
Size barely moved (dropping `curl`/`apk` offsets the unprivileged layer):
```
idan74@MacBook-Air-sl-ydn f5-devops-ha % docker images | grep f5-devops-ha-proxy
f5-devops-ha-proxy .. latest ... 886c7002a8e4 ... 60MB
```

---

> 2. Run the proxy as a non-root user

### What I changed
- Unprivileged image: base is now `nginxinc/nginx-unprivileged:stable-alpine`, so nginx runs as UID 101 and the master (PID 1) is no longer root.
- `proxy/nginx.conf`: set `listen 8080` and added `pid /tmp/nginx.pid;` - a non-root process can't bind privileged :80 or write the default `/run/nginx.pid`.
- `docker-compose.yml`: publish `80:8080` (edge still on :80), healthcheck probes `127.0.0.1:8080` via wget, and the test-runner targets `proxy:8080`.

---

> 3. Additional security / performance improvements

### `server_tokens off;` (security)
- Nginx leaks its exact version in the `Server` header and error pages by default - a free CVE fingerprint.
- Off leaves just `Server: nginx`, no behavior change.

### Pin the base image by digest (security / reproducibility)
- `stable-alpine` is a moving tag, so the same Dockerfile can build a different image tomorrow.
- Pinning `@sha256:<digest>` locks the build to the exact bits we reviewed (patches now bumped deliberately).

### Proxy timeouts (resilience)
- Default upstream timeouts are 60s; this backend answers in ms (no DB), so that's just slow failure detection.
- Set `connect 3s` / `send+read 10s` to fail fast on a stuck backend with plenty of margin.

---

## Anything you would do differently with more time

Two more things I'd add to the proxy if I had the time:

**Make the container read-only and drop its privileges.**
Set `read_only: true` and give nginx a `tmpfs` for `/tmp` (that's where the pid lives now).
Plus `no-new-privileges` and `cap_drop: ALL` - non-root nginx on 8080 doesn't need any capabilities.
So even if someone pops the proxy, they can't write to disk or escalate.
Didn't do it yet because read-only can break startup if I miss a writable path, and I'd want a proper test pass first.

**Reuse the connection to the backend (keepalive).**
Right now the proxy opens a fresh TCP connection to the backend on every request.
Adding `keepalive 32` in the upstream + `proxy_http_version 1.1` + clearing the `Connection` header makes it reuse a pool instead.
That's a handshake saved per request once there's real load.
Skipped it for now because traffic here is tiny, so the payoff is basically nothing for the tests.
