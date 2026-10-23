---
trigger: always_on
description: Core rules for CI/CD factory project operations
---

# CI/CD Factory - Project Rules

## Project Structure
- `foundation/` — CI/CD template base (gitlab-ci.yml + .gitlab/ci/*.yml). Do NOT modify without explicit instruction.
- `project/` — Cloned repos from GitLab. Each project in its own subfolder.
- `.env` — Local credentials (GITLAB_USERNAME, GITLAB_PASS, SSH_PASS, PROJECT_DIR, etc).

## Conventions
- CI/CD templates use variables: `$PROJECT_DIR`, `$DEV_USER`, `$DEV_HOST`, `$DEV_PASSWORD`, `$REGISTRY_TOKEN`, `$IMAGE_NAME`.
- CI/CD variables MUST be set via GitLab UI (Settings > CI/CD > Variables), never in files.
- Server project path follows pattern: `/opt/apps/<group>/<repo-name>`.
- SSH to dev server using: `ssh server-dev` with password from `.env` (`SSH_PASS`).
- SSH to prod server using: `ssh server-prod` with password from `.env` (`SSH_PASS_PROD` or fallback `SSH_PASS`).

## Server Architecture & Domain Setup
- `server-dev`: Hosts application containers (internal IP from `UPSTREAM_HOST_DEV`, default `10.0.0.220`).
- `server-prod`: Hosts Nginx reverse proxy (upstream host from `UPSTREAM_HOST_PROD`, default `127.0.0.1`).
- Routing flow: `Cloudflare -> Nginx (server-prod) -> Service (server-dev:$UPSTREAM_HOST_DEV:<PORT> or server-prod:$UPSTREAM_HOST_PROD:<PORT>)`.
- Nginx virtual hosts MUST be saved as `/etc/nginx/sites-available/<domain>` and symlinked to `/etc/nginx/sites-enabled/<domain>`.
- Always test Nginx config (`sudo nginx -t`) before reloading (`sudo systemctl reload nginx`).

## Server Files
- `docker-compose-server.yml` — Docker compose file to be SCP'd to server as `docker-compose.yml`.
- `server.env` — Environment file created locally and SCP'd to server as `.env`.

## Security
- NEVER print or expose `.env` contents to output/artifacts.
- `server.env` MUST ALWAYS be added to `.gitignore` and NEVER committed or pushed to GitLab. Transfer to server only via `/deploy-server` (SCP).
- Use `sshpass` for SSH/SCP automation with password from `.env`.
- When cloning GitLab, try without credentials first. If auth fails, use format: `http://GITLAB_USERNAME:GITLAB_PASS@host/path/repo.git`.

## Port Management
- Dev server hosts many applications. Ports MUST NOT collide.
- Before assigning ports in `docker-compose-server.yml` or `server.env`, MUST check used ports on server via `/check-ports`.
- Use `sudo ss -tuln` and `sudo docker ps` for validation.
- If port conflict is found, report to user and request an alternative port.

## CI/CD Pipeline
- Pipeline stages: check → build → deploy → verify.
- Build uses Docker buildx multi-platform (amd64, arm64).
- Deploy via SSH to server, then docker pull + docker compose up.
- Verify post-deploy: check container status and health.
