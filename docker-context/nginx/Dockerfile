FROM nginx:1.17.3-alpine

RUN adduser -D -H -s /bin/sh -u 1000 nhandd

# timezone
RUN cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && chmod 0644 /etc/localtime

# port
EXPOSE 80 443

## nginx
RUN mkdir -p /etc/nginx/sites-enabled

RUN mkdir -p /var/www/gmo \ 
    && chown -R nhandd:nhandd /var/www/gmo

CMD nginx -g 'daemon off;' -c /etc/nginx/nginx.conf
