# Minimal proxy image: official nginx on Alpine.
# Only curl is added, for the compose healthcheck (curl -fsS /nginx-health).

FROM nginx:alpine

RUN apk add --no-cache curl

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
