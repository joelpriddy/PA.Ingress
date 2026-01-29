# PA.Ingress

Shared reverse proxy (nginx) and Cloudflare tunnel infrastructure for Priddy Acres services.

## Architecture

- **nginx** - Reverse proxy routing traffic to backend services by hostname
- **cloudflared** - Cloudflare tunnel connecting the public internet to nginx

## Hostnames

| Hostname | Upstream | Description |
|----------|----------|-------------|
| `hive.priddyacres.net` | `pa-hive-inventory-service:8080` | PA.Hive inventory API |
| `auth.priddyacres.net` | `auth-gateway:8080` | Shared JWT auth service |

## Deployment

```bash
# First time: create the shared network
docker network create ingress-net

# Copy .env.example to .env and set the tunnel token
cp .env.example .env

# Start services
docker compose up -d --build
```

## Prerequisites

The following containers must be on `ingress-net`:
- `auth-gateway` (from PA.Gateway)
- `pa-hive-inventory-service` (from PA.Hive)

Nginx uses dynamic DNS resolution (`resolver 127.0.0.11`) so it starts even if upstreams aren't available yet.

## Network

All services communicate via the `ingress-net` Docker network. This network is created by this compose project and referenced as `external` by other projects.
