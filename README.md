# PA.Ingress

Shared reverse proxy (nginx) and Cloudflare tunnel infrastructure for Priddy Acres services.

## Architecture

- **nginx** - Reverse proxy routing traffic to backend services by hostname
- **cloudflared** - Cloudflare tunnel connecting the public internet to nginx

## Deployment

```bash
# First time: create the shared network
docker network create ingress-net

# Copy .env.example to .env and set the tunnel token
cp .env.example .env

# Start services
docker compose up -d --build
```

## Adding Services

To route traffic to a new service:

1. Create a new `.conf` file in `nginx/conf.d/`
2. Use dynamic DNS resolution for resilience:
   ```nginx
   server {
       listen 80;
       server_name myapp.priddyacres.net;
       resolver 127.0.0.11 valid=10s ipv6=off;

       location / {
           set $upstream myapp-container:8080;
           proxy_pass http://$upstream;
       }
   }
   ```
3. Ensure the backend container is on `ingress-net`
4. Add the hostname to Cloudflare tunnel config

## Network

All services communicate via the `ingress-net` Docker network. This network is created by this compose project and referenced as `external` by other projects.

Nginx uses dynamic DNS resolution (`resolver 127.0.0.11`) so it starts even if upstreams aren't available yet.
