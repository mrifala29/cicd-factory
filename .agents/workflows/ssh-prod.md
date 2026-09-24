---
description: >-
  SSH into the prod server for manual inspection or troubleshooting.
  Use this workflow when the user wants to connect to the prod server.
slash_command: ssh-prod
---

# /ssh-prod — SSH to Prod Server

Connect to the prod server for manual inspection, Nginx configuration, or troubleshooting.

## Steps

### 1. Read Config
Read `.env` from workspace root to get `SSH_PASS_PROD` (or fallback to `SSH_PASS`).

### 2. Connect
```bash
sshpass -p "${SSH_PASS_PROD}" ssh -o StrictHostKeyChecking=no server-prod
```

### 3. Common Tasks
Once connected, useful commands:
- `sudo nginx -t` — Test Nginx configuration
- `sudo systemctl status nginx` — Check Nginx service status
- `sudo systemctl reload nginx` — Reload Nginx without downtime
- `ls -la /etc/nginx/sites-enabled/` — List active Nginx virtual hosts
- `docker compose ps` — Check container status (if prod applications exist)
