---
description: >-
  SCP docker-compose-server.yml and server.env to the dev server.
  Use this workflow when the user wants to deploy server config files to the remote server.
slash_command: deploy-server
---

# /deploy-server — Deploy Config to Server

SCP the `docker-compose-server.yml` and `server.env` files to the dev server.

## Prerequisites
- `.env` has `SSH_PASS` and `PROJECT_DIR` populated.
- `docker-compose-server.yml` and `server.env` exist in the project directory.
- User specifies which project (repo name) in `project/`.

## Steps

### 1. Read Config
Read `.env` from workspace root to get `SSH_PASS` and `PROJECT_DIR`.

### 2. Identify Project
Ask the user which project in `project/` to deploy, or infer from context.
Set `REPO_NAME` to the project folder name.

### 3. Create Remote Directory
```bash
sshpass -p "${SSH_PASS}" ssh -o StrictHostKeyChecking=no server-dev "mkdir -p ${PROJECT_DIR}"
```

### 4. SCP docker-compose
```bash
sshpass -p "${SSH_PASS}" scp -o StrictHostKeyChecking=no \
  project/${REPO_NAME}/docker-compose-server.yml \
  server-dev:${PROJECT_DIR}/docker-compose.yml
```

### 5. SCP .env
```bash
sshpass -p "${SSH_PASS}" scp -o StrictHostKeyChecking=no \
  project/${REPO_NAME}/server.env \
  server-dev:${PROJECT_DIR}/.env
```

### 6. Verify
SSH into server and confirm files exist:
```bash
sshpass -p "${SSH_PASS}" ssh -o StrictHostKeyChecking=no server-dev "ls -la ${PROJECT_DIR}"
```

### 7. Summary
Report:
- Files deployed to `${PROJECT_DIR}` on server-dev
- `docker-compose.yml` and `.env` are in place
- Server is ready for the CI/CD pipeline to run
