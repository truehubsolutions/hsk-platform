# Deployment & Infrastructure

This document covers the hosting setup, deployment process, and environment configuration for the HSK Platform backend.

---

## Infrastructure Overview

| Component | Provider | Notes |
|---|---|---|
| Backend API | Hetzner VPS (Ubuntu 22.04) | Docker-based deployment |
| Container orchestration | Docker Compose | Single-server setup |
| Process manager | Systemd + Docker | Auto-restart on failure |
| Reverse proxy | Nginx | SSL termination, port routing |
| SSL | Let's Encrypt (Certbot) | Auto-renew |
| Operational data | Google Sheets | No separate DB required |
| Bookkeeping | Xero Cloud | SaaS, no self-hosting |

---

## Backend Deployment (FastAPI)

### Docker Compose setup

```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "8000:8000"
    env_file:
      - .env
    restart: unless-stopped
    volumes:
      - ./logs:/app/logs
```

### Environment variables required

```env
# Xero OAuth
XERO_CLIENT_ID=
XERO_CLIENT_SECRET=
XERO_REFRESH_TOKEN=
XERO_TENANT_ID=

# Google Sheets
GOOGLE_SERVICE_ACCOUNT_JSON=
SPREADSHEET_ID=

# App config
SECRET_KEY=
ALLOWED_ORIGINS=https://your-appsheet-domain.com
ENVIRONMENT=production
```

### Deploy steps

```bash
# Pull latest
git pull origin main

# Rebuild and restart
docker compose down
docker compose build --no-cache
docker compose up -d

# Verify
docker compose ps
docker compose logs api --tail=50
```

---

## Nginx Configuration

```nginx
server {
    listen 443 ssl;
    server_name api.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/api.your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.your-domain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Monitoring

- **Health check endpoint:** `GET /health` — returns `{"status": "ok", "version": "x.x.x"}`
- **Logs:** `docker compose logs api -f`
- **Uptime:** Monitored via UptimeRobot (free tier, 5-minute checks)

---

## Backup

- Google Sheets data: inherently versioned by Google
- Xero data: managed by Xero's own backup infrastructure
- VPS: weekly Hetzner snapshot (automated)
- `.env` files: stored in Bitwarden, never in git
