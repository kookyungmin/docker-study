FROM nginx:1.29.4-alpine

COPY resources/index.html /usr/share/nginx/html/index.html
COPY resources/docker_logo.png /usr/share/nginx/html/docker_logo.png

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
