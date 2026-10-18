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

### How I diagnosed it

### Root cause

### Fix

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix, and why I chose it over the alternatives

---

## Defect 4: CI pipeline fails

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>

---

## Anything you would do differently with more time
