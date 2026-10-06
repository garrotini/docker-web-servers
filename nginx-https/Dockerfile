FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
COPY selfsigned.crt selfsigned.key /etc/nginx/ssl/
COPY default.conf /etc/nginx/conf.d/
EXPOSE 80 443

