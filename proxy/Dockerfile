# Non-root proxy image: NGINX Inc's unprivileged nginx on Alpine.
# Runs nginx as UID 101 (master included), listens on 8080.
# No curl: the compose healthcheck uses the busybox wget already in the image.

# Pinned by digest for reproducible builds (tag kept for readability).
FROM nginxinc/nginx-unprivileged:stable-alpine@sha256:782bb89768cc87b7f95c18b95baa1af250e5d451f26794f59363540161147c01

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
