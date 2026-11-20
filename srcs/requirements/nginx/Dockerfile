FROM debian:bookworm-slim

RUN apt update && apt upgrade -y \
	&& apt install -y nginx openssl

COPY conf/nginx.conf /etc/nginx/conf.d/default.conf
COPY tools/nginx.sh /var/www/nginx.sh

RUN mkdir -p /etc/nginx/ssl
RUN chmod +x /var/www/nginx.sh

EXPOSE 443

ENTRYPOINT [ "/var/www/nginx.sh" ]

CMD ["nginx", "-g", "daemon off;"]