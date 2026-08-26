---
description: >-
  SSH into the dev server for manual inspection or troubleshooting.
  Use this workflow when the user wants to connect to the dev server.
slash_command: ssh-dev
---

# /ssh-dev — SSH to Dev Server

Connect to the dev server for manual inspection.

## Steps

### 1. Read Config
Read `.env` from workspace root to get `SSH_PASS`.

### 2. Connect
```bash
sshpass -p "${SSH_PASS}" ssh -o StrictHostKeyChecking=no server-dev
```

### 3. Common Tasks
Once connected, useful commands:
- `cd ${PROJECT_DIR}` — Go to project directory
- `docker compose ps` — Check container status
- `docker compose logs -f` — Tail logs
- `docker compose down && docker compose up -d` — Restart
