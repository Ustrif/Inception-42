# Developer Documentation

## Prerequisites

The project must run in a Virtual Machine with:

- Linux
- Docker Engine
- Docker Compose v2 (`docker compose`)
- GNU Make
- `sudo` access for the `fclean` target
- TLS domain resolution for `login.42.fr`

## Project Structure

The project is organized as follows:

```text
Makefile
README.md
USER_DOC.md
DEV_DOC.md
srcs/
  .env (local, gitignored)
  docker-compose.yml
  requirements/
    mariadb/
      Dockerfile
      conf/
      tools/
    nginx/
      Dockerfile
      conf/
      tools/
    wordpress/
      Dockerfile
      conf/
      tools/
```

## Environment Setup

Clone or copy the repository into the VM, then enter the repository root:

```bash
cd /home/login/Desktop/Inception
```

The main environment file is:

```text
srcs/.env
```

It contains:

- `DATA_PATH`: host directory used as the base for named-volume bind storage
- `DOMAIN_NAME`: WordPress domain
- MariaDB database name and credentials
- WordPress title and administrator credentials
- Regular WordPress user credentials

Before the first launch, verify that `DATA_PATH` uses the learner's host home directory. The project currently uses:

```text
/home/login/data
```

For the Docker named volumes to work, this directory must be available on the host. The Makefile also creates:

```text
$(HOME)/data/mariadb
$(HOME)/data/wordpress
```

If the VM username differs, update `DATA_PATH` consistently in `srcs/.env` and ensure the Makefile data directory matches the same location.

## Build and Launch

The Makefile is the main entry point at the repository root.

Build and start the complete stack:

```bash
make
```

This target is equivalent to:

```bash
make up
```

The `up` target creates the host directories and runs:

```bash
docker compose -f ./srcs/docker-compose.yml up -d --build
```

You can run Docker Compose directly with:

```bash
docker compose -f srcs/docker-compose.yml up -d --build
```

The Compose file builds the three project images from the Dockerfiles under `srcs/requirements/`. It then starts the following containers:

- `nginx`
- `wordpress`
- `mariadb`

The Compose dependencies wait for health checks before starting dependent containers.

## Container Management

Check container status:

```bash
docker compose -f srcs/docker-compose.yml ps
```

View service logs:

```bash
docker logs nginx
docker logs wordpress
docker logs mariadb
```

Follow WordPress logs in real time:

```bash
docker logs -f wordpress
```

Stop containers without deleting them:

```bash
make stop
```

Start existing containers again:

```bash
make start
```

Stop and remove containers and networks, while retaining volumes:

```bash
make down
```

Rebuild and restart everything:

```bash
make re
```

`make re` performs the destructive `fclean` target first, so it deletes project volumes and data.

## Volume Management

The Compose file defines two Docker named volumes:

- `mariadb_data`: MariaDB database files
- `wordpress_data`: WordPress website files

Their driver options bind them to host directories:

```text
mariadb_data -> /home/login/data/mariadb
wordpress_data -> /home/login/data/wordpress
```

Inside containers, the mount points are:

```text
MariaDB:  /var/lib/mysql
WordPress: /var/www/wordpress
Nginx:    /var/www/wordpress
```

List project volumes:

```bash
docker volume ls
docker volume inspect srcs_mariadb_data
docker volume inspect srcs_wordpress_data
```

Inspect volume mounts in Compose:

```bash
docker compose -f srcs/docker-compose.yml config
```

Stop containers and remove their volumes:

```bash
docker compose -f srcs/docker-compose.yml down -v
```

Delete the host data directories used by this project:

```bash
sudo rm -rf /home/login/data/mariadb /home/login/data/wordpress
```

The Makefile provides the equivalent through:

```bash
make fclean
```

## Data Persistence

MariaDB persists database files in `mariadb_data`, mounted at `/var/lib/mysql` inside the MariaDB container and backed by `/home/login/data/mariadb` on the host.

WordPress persists website files in `wordpress_data`, mounted at `/var/www/wordpress` inside WordPress and Nginx containers and backed by `/home/login/data/wordpress` on the host.

The data survives:

- container restarts
- `make stop` and `make start`
- `make down`

The data is deleted only when:

- volumes are removed with `docker compose down -v`
- host directories are deleted
- `make fclean` is run

Rebuilding images does not by itself delete named-volume data.

## Networks and Ports

All containers share the `inception` Docker bridge network. They communicate using container service names:

- `mariadb:3306`
- `wordpress:9000`
- Nginx connects to `wordpress:9000`
- WordPress connects to `mariadb:3306`

Only Nginx publishes a host port:

```text
443 -> 443
```

MariaDB and PHP-FPM remain internal to the Docker network.

## Verification

Verify the Compose configuration:

```bash
docker compose -f srcs/docker-compose.yml config
```

Verify HTTPS availability:

```bash
curl -kI https://login.42.fr
```

Verify WordPress content or the administration login at:

```text
https://login.42.fr
https://login.42.fr/wp-admin
```