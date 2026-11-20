FROM debian:bookworm-slim

# Update and install MariaDB server and client
RUN apt-get update -y && \
	apt-get install -y mariadb-server mariadb-client procps

# copy configuration file and entrypoint script to container
COPY conf/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf
COPY tools/mariadb.sh /usr/local/bin/

# we need permissions to execute the entrypoint script
RUN chmod +x /usr/local/bin/mariadb.sh

EXPOSE 3306

ENTRYPOINT ["/usr/local/bin/mariadb.sh"]

CMD ["mariadbd"]