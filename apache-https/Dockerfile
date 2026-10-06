
FROM httpd:alpine
COPY index.html /usr/local/apache2/htdocs/
COPY selfsigned.crt /usr/local/apache2/conf/server.crt
COPY selfsigned.key /usr/local/apache2/conf/server.key
RUN sed -i \
    -e 's/^#\(LoadModule .*mod_ssl.so\)/\1/' \
    -e 's/^#\(LoadModule .*mod_socache_shmcb.so\)/\1/' \
    -e 's|^#\(Include .*httpd-ssl.conf\)|\1|' \
    /usr/local/apache2/conf/httpd.conf
EXPOSE 80 443

