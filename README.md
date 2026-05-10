# Traefik Docker Reverse Proxy

This repository contains my Docker Compose setup for running [Traefik](https://traefik.io/traefik/) as a reverse proxy for Docker-based services.

Traefik routes incoming HTTP/HTTPS traffic to containers using Docker labels, allowing multiple services to run behind a single server IP using separate domains or subdomains.

## Features

- Traefik reverse proxy managed with Docker Compose
- Docker provider with containers hidden by default
- HTTP and HTTPS entrypoints
- Let's Encrypt certificate generation using HTTP challenge or Cloudflare API
- External Docker network for proxied services
- Optional Traefik dashboard protected with Basic Auth
- Environment variables used for configurable values and secrets

## Repository Structure

```text
.
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

- Docker and Docker Compose installed
- A domain or subdomain pointed to the server
- Ports `80` and `443` open on the server/firewall
- An external Docker network named `proxy`

Create the proxy network if it does not already exist:

```bash
docker network create proxy
```

## Environment Variables

Create a `.env` file from the example file:

```bash
cp .env.example .env
```

Example `.env.example`:

```env
TRAEFIK_VERSION=v3.3
ACME_EMAIL=your-email@example.com
TRAEFIK_URL=traefik.example.com
TRAEFIK_DASHBOARD_AUTH=admin:REPLACE_WITH_HASHED_PASSWORD
```

Do not commit your real `.env` file.

## Basic Auth

The Traefik dashboard is protected with Basic Auth. Generate a hashed username and password with:

```bash
htpasswd -nbB admin your-password
```

If the generated hash contains `$`, escape each `$` as `$$` when placing it in the `.env` file so Docker Compose reads it correctly.

## Start Traefik

Start the reverse proxy:

```bash
docker compose up -d
```

View logs:

```bash
docker compose logs -f traefik
```

Stop the stack:

```bash
docker compose down
```

## Exposing a Service

Attach the service to the same external `proxy` network and add Traefik labels.

Example:

```yaml
services:
  app:
    image: example/app:latest
    networks:
      - proxy
    labels:
      - traefik.enable=true
      - traefik.http.routers.app.rule=Host(`app.example.com`)
      - traefik.http.routers.app.entrypoints=websecure
      - traefik.http.routers.app.tls=true
      - traefik.http.routers.app.tls.certresolver=le
      - traefik.http.services.app.loadbalancer.server.port=8080

networks:
  proxy:
    external: true
```

The `loadbalancer.server.port` value should match the internal container port used by the application.

## Security Notes

- Keep `.env` out of source control
- Use a pinned Traefik version instead of `latest`
- Mount the Docker socket as read-only when possible
- Use a strong password for the dashboard
- Consider limiting dashboard access by IP address or VPN for production use

Suggested `.gitignore`:

```gitignore
.env
```

## Troubleshooting

### 404 from Traefik

Traefik is running, but no router matched the request. Check the hostname, router rule, labels, and network.

### Bad Gateway

Traefik matched the route but cannot reach the backend container. Check that the service is running, attached to the `proxy` network, and using the correct internal port.

### Certificate issues

Check that DNS points to the server, ports `80` and `443` are open, and the ACME email is set correctly.
