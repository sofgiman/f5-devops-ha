# Non-root proxy image: NGINX Inc's unprivileged nginx on Alpine.
# Runs nginx as UID 101 (master included), listens on 8080.
# No curl: the compose healthcheck uses the busybox wget already in the image.

FROM nginxinc/nginx-unprivileged:stable-alpine

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
