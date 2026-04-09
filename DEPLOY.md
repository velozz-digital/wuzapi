# WuzAPI Docker Deployment Guide

This project supports two Docker deployment profiles:

## 1) Standalone (fastest to deploy)

Use SQLite with a persistent Docker volume. No PostgreSQL is required.

```bash
cp .env.sample .env
# Edit at least WUZAPI_ADMIN_TOKEN and WUZAPI_GLOBAL_ENCRYPTION_KEY
docker compose -f docker-compose.standalone.yml up -d --build
```

Checks:

```bash
curl -s http://localhost:8080/health
docker compose -f docker-compose.standalone.yml logs -f wuzapi
```

Notes:
- Persistent data is stored in `wuzapi_data` volume (`/data` inside container).
- This is the recommended mode for quick fallback/proxy testing.

## 2) Full stack (PostgreSQL + RabbitMQ)

Use the default compose file when you need PostgreSQL and built-in RabbitMQ service:

```bash
cp .env.sample .env
docker compose up -d --build
```

Checks:

```bash
curl -s http://localhost:8080/health
docker compose logs -f wuzapi-server
```

## Production recommendations

- Set strong secrets:
  - `WUZAPI_ADMIN_TOKEN`
  - `WUZAPI_GLOBAL_ENCRYPTION_KEY` (32 bytes)
  - `WUZAPI_GLOBAL_HMAC_KEY` (32+ chars if webhooks are used)
- Put the API behind HTTPS reverse proxy (Nginx, Traefik, Caddy, Cloudflare Tunnel).
- Restrict source IPs and rate limit admin endpoints.
- Keep backups of `.env` and persistent volumes.

## Suggested fallback strategy (WuzAPI <-> Z-API)

1. Start using WuzAPI as shadow/fallback endpoint.
2. Route a small percentage of traffic to WuzAPI.
3. Compare delivery success, latency, and reconnection behavior.
4. Promote WuzAPI to primary only after stability thresholds are met.

