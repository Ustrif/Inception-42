FROM debian:bookworm-slim

# Update and install WordPress + php-fpm
RUN apt-get update -y && \
	apt-get install -y curl php php-fpm php-mysql mariadb-client

# download configuration file for wp-cli
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
	&& chmod +x wp-cli.phar \
	&& mv wp-cli.phar /usr/local/bin/wp

RUN mkdir -p /run/php

COPY conf/www.conf /etc/php/8.2/fpm/pool.d/www.conf
COPY tools/wordpress.sh /usr/local/bin/wordpress.sh

RUN chmod +x /usr/local/bin/wordpress.sh

WORKDIR /var/www/wordpress
EXPOSE 9000

ENTRYPOINT ["/usr/local/bin/wordpress.sh"]

CMD ["php-fpm8.2", "-F"]