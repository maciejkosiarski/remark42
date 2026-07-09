# remark42 self-hosted kit

Docker-based [remark42](https://remark42.com) setup that supports two deployment modes:

1. **Behind an existing `caddy-docker-proxy`** — no ports published, routing driven by container labels. Useful when a single Caddy already fronts multiple services on the host.
2. **Standalone** — bundled Caddy container publishes a port on the host.

## Prerequisites

- Docker and Docker Compose.
- A domain pointing at your VPS (e.g. `remark42.example.com`).
- The site that will embed the widget (e.g. `https://example.com`).

## 1. Configure environment

```bash
cp .env.dist .env
```

Fill in at minimum:

- `REMARK_URL` — public URL of your remark42 instance (`https://remark42.example.com`).
- `SECRET` — 32+ random characters. Generate with `openssl rand -hex 32`.
- `SITE` — site identifier, used later in the embed snippet.
- `ALLOWED_HOSTS` — origins allowed to embed the widget (comma-separated).

Default auth mode is `AUTH_ANON=true`. Swap for OAuth by uncommenting the relevant block.

## 2a. Deployment behind `caddy-docker-proxy`

Assumes a running [`lucaslorentz/caddy-docker-proxy`](https://github.com/lucaslorentz/caddy-docker-proxy) container attached to an external Docker network named `edge`, listening on port `40147` (adjust in `compose.yaml` if you use a different port).

```bash
docker compose up -d
```

The container joins the `edge` network and Caddy picks up the routing from its labels.

## 2b. Standalone deployment

```bash
cp Caddyfile.dist Caddyfile
docker compose --profile standalone up -d
```

The bundled Caddy publishes `${PORT}` (default `40147`) on the host. Put TLS termination in front (Cloudflare, another reverse proxy, or extend the Caddyfile).

## Verify

```bash
curl https://remark42.example.com/ping
# expected: pong
```

## Data persistence

BoltDB lives in the `remark42_data` volume mounted at `/srv/var`. Back it up with `docker run --rm -v remark42_data:/data -v $PWD:/backup alpine tar czf /backup/remark42-$(date +%F).tar.gz -C /data .`.
