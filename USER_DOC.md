# User Documentation

## Provided Services

This project provides a WordPress website with the following services:

- **Nginx**: Receives HTTPS requests on port `443`, uses TLSv1.2 or TLSv1.3, and forwards PHP requests to WordPress.
- **WordPress**: Runs WordPress with PHP-FPM.
- **MariaDB**: Stores the WordPress database.
- **Docker volumes**: Persist the WordPress database and website files outside the containers.

The Nginx container is the only service that accepts connections from the host.

## Start the Project

Run this command from the repository root:

```bash
make
```

The first start may take several minutes while packages and Docker images are downloaded and configured.

## Stop the Project

Stop all running containers while keeping the containers and stored data:

```bash
make stop
```

Start stopped containers again:

```bash
make start
```

To remove containers, images, networks, and project data:

```bash
make fclean
```

This command deletes the project data directories and is destructive.

## Access the Website

Open the website at:

```text
https://login.42.fr
```

The browser may show a security warning because the project uses a local self-signed TLS certificate.

Open the WordPress administration panel at:

```text
https://login.42.fr/wp-admin
```

The domain must resolve to the host machine's IP address. If it does not resolve in the local environment, add the appropriate local mapping to `/etc/hosts`.

## Credentials

Environment variables are stored in:

```text
srcs/.env
```

This file contains the database name, database credentials, WordPress domain, administrator account, and regular WordPress user account.

Credentials must not be shared or committed to Git. Keep the `.env` file and secret files local. Changing database or WordPress credentials after initial setup requires rebuilding with clean data (make fclean && make) or updating them inside the service.

## Check Service Status

List containers and their current state:

```bash
docker compose -f srcs/docker-compose.yml ps
```

MariaDB, WordPress, and Nginx should show an `Up` status. Health checks should complete successfully after startup.

Read container logs:

```bash
docker logs mariadb
docker logs wordpress
docker logs nginx
```

Check the HTTPS endpoint:

```bash
curl -kI https://login.42.fr
```

A successful check returns an HTTP response from Nginx, usually `200 OK` or another valid WordPress status.

Confirm that Nginx publishes port `443`:

```bash
docker compose -f srcs/docker-compose.yml port nginx 443
```

If the website does not respond, verify container status, read the logs, and confirm that the domain resolves to the host.