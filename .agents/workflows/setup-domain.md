---
description: >-
  Configure Nginx reverse proxy on server-prod for a service running on server-dev or server-prod.
  Use this workflow when the user provides a domain name and port after deployment.
slash_command: setup-domain
---

# /setup-domain — Setup Nginx Domain & Reverse Proxy

Configure an Nginx reverse proxy site on `server-prod` to route traffic from Cloudflare to an application service running on `server-dev` or `server-prod`.

## Architecture Flow

- **Flow 1 (Dev Service)**: `Cloudflare -> Nginx (server-prod) -> Service (server-dev:10.0.0.220:<PORT>)`
- **Flow 2 (Prod Service)**: `Cloudflare -> Nginx (server-prod) -> Service (server-prod:127.0.0.1:<PORT>)`

## Prerequisites

- Domain name (e.g. `grafana.linkit360.ai` or `app.linkit360.ai`).
- Service port (e.g. `3000`).
- Target server: `server-dev` or `server-prod`.
- `.env` contains `SSH_PASS_PROD` (or falls back to `SSH_PASS` if passwords match).
- `.env` contains `UPSTREAM_HOST_DEV` (e.g. `10.0.0.220`) and optionally `UPSTREAM_HOST_PROD` (default: `127.0.0.1`).
- SSL certificate exists on `server-prod` in `/etc/ssl/` (e.g. `/etc/ssl/linkit360.ai.pem`).

## Steps

### 1. Read Configuration
Read `.env` from workspace root to get:
- `SSH_PASS_PROD` (fallback to `SSH_PASS` if not defined)
- `UPSTREAM_HOST_DEV` (default: `10.0.0.220`)
- `UPSTREAM_HOST_PROD` (default: `127.0.0.1`)

### 2. Identify Parameters
Collect from user prompt or ask if not specified:
- **`DOMAIN`**: Full domain name (e.g., `app.linkit360.ai`).
- **`TARGET_SERVER`**: Where the container/service is running (`server-dev` or `server-prod`).
- **`TARGET_PORT`**: Port exposed by the application container.
- **`BASE_DOMAIN`**: Derived from `DOMAIN` (e.g., `linkit360.ai` for `app.linkit360.ai`).

Set upstream host from `.env`:
- If `TARGET_SERVER` is `server-dev`: `UPSTREAM_HOST="${UPSTREAM_HOST_DEV:-10.0.0.220}"`
- If `TARGET_SERVER` is `server-prod`: `UPSTREAM_HOST="${UPSTREAM_HOST_PROD:-127.0.0.1}"`

### 3. Generate Nginx Configuration
Generate `/tmp/nginx-${DOMAIN}.conf` from `foundation/nginx/template.conf`:

```nginx
server {
    listen 80;
    server_name ${DOMAIN};
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name ${DOMAIN};

    ssl_certificate /etc/ssl/linkit360.ai.pem;
    ssl_certificate_key /etc/ssl/linkit360.ai.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    location / {
        proxy_pass http://${UPSTREAM_HOST}:${TARGET_PORT}/;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 86400;
        proxy_send_timeout 86400;
    }
}
```
*(Note: If domain is outside `linkit360.ai`, adjust certificate path to `/etc/ssl/${BASE_DOMAIN}.pem`).*

### 4. Transfer to server-prod
SCP the generated configuration file to `/tmp` on `server-prod`:
```bash
sshpass -p "${SSH_PASS_PROD}" scp -o StrictHostKeyChecking=no \
  /tmp/nginx-${DOMAIN}.conf \
  server-prod:/tmp/${DOMAIN}
```

### 5. Install & Symlink on server-prod
Move to `/etc/nginx/sites-available/${DOMAIN}` and create symlink to `sites-enabled`:
```bash
sshpass -p "${SSH_PASS_PROD}" ssh -o StrictHostKeyChecking=no server-prod \
  "echo '${SSH_PASS_PROD}' | sudo -S mv /tmp/${DOMAIN} /etc/nginx/sites-available/${DOMAIN} && \
   echo '${SSH_PASS_PROD}' | sudo -S ln -sf /etc/nginx/sites-available/${DOMAIN} /etc/nginx/sites-enabled/${DOMAIN}"
```

### 6. Test Nginx Configuration
Validate syntax before reloading:
```bash
sshpass -p "${SSH_PASS_PROD}" ssh -o StrictHostKeyChecking=no server-prod \
  "echo '${SSH_PASS_PROD}' | sudo -S nginx -t"
```

If test fails:
- Remove invalid symlink: `echo '${SSH_PASS_PROD}' | sudo -S rm /etc/nginx/sites-enabled/${DOMAIN}`
- Report the error to the user without reloading Nginx.

### 7. Reload Nginx
If test succeeds, reload Nginx service gracefully:
```bash
sshpass -p "${SSH_PASS_PROD}" ssh -o StrictHostKeyChecking=no server-prod \
  "echo '${SSH_PASS_PROD}' | sudo -S systemctl reload nginx"
```

### 8. Cleanup & Summary
Remove temporary local file:
```bash
rm -f /tmp/nginx-${DOMAIN}.conf
```

Report:
- Domain URL: `https://${DOMAIN}`
- Proxy Target: `http://${UPSTREAM_HOST}:${TARGET_PORT}/` (`${TARGET_SERVER}`)
- Config File: `/etc/nginx/sites-available/${DOMAIN}` -> `/etc/nginx/sites-enabled/${DOMAIN}`
- Nginx Status: Syntax OK & reloaded successfully
- Remind user to ensure Cloudflare DNS record points `${DOMAIN}` (Proxied) to `server-prod` IP.
