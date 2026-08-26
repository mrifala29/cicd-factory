---
description: >-
  Clone a GitLab repository into the project/ directory and set up CI/CD files.
  Use this workflow when the user wants to set up a new CI/CD project from a GitLab repo URL.
slash_command: setup-project
---

# /setup-project — Setup New CI/CD Project

This workflow clones a GitLab repository and prepares the CI/CD configuration.

## Prerequisites
- User provides the GitLab repo URL.
- `.env` file has `GITLAB_USERNAME`, `GITLAB_PASS` populated.

## Steps

### 1. Read Credentials
Read `.env` from workspace root to get `GITLAB_USERNAME` and `GITLAB_PASS`.

### 2. Clone Repository
Extract the repo URL from user input. Clone into `project/` directory:
```bash
cd /Users/alfarizy/Desktop/Project/cicd-factory/project
git clone http://${GITLAB_USERNAME}:${GITLAB_PASS}@<host>/<path>/<repo>.git
```
- Parse the user-provided URL to extract host/path/repo.
- If URL is HTTPS, strip the protocol and inject credentials.
- If URL is SSH (git@), convert to HTTP format with credentials.

### 3. Check Port Availability
Before creating server files, run `/check-ports` workflow to check used ports on the server.
This prevents port collisions with existing applications.

### 4. Copy CI/CD Template
Copy the CI/CD template files from `foundation/` into the cloned project:
```bash
cp foundation/gitlab-ci.yml project/<repo-name>/.gitlab-ci.yml
cp -r foundation/.gitlab project/<repo-name>/
```
Note: The main file is renamed from `gitlab-ci.yml` to `.gitlab-ci.yml` (with leading dot).

### 5. Create docker-compose-server.yml
Create `project/<repo-name>/docker-compose-server.yml` based on the project's needs.
This file will be SCP'd to the server as `docker-compose.yml`.
- Ask the user what services are needed if not obvious from existing Dockerfile/code.
- Use `${IMAGE_TAG}` variable for the image tag so deploy pipeline can inject it.
- Assign ports that are NOT already in use (validated in step 3).

### 6. Create server.env
Create `project/<repo-name>/server.env` with the environment variables the app needs on the server.
This file will be SCP'd to the server as `.env`.
- Ask the user what environment variables are needed.
- If any env var defines a port (e.g. APP_PORT), ensure it doesn't collide.

### 7. Summary
Report what was done:
- Repo cloned to `project/<repo-name>/`
- CI/CD files copied
- `docker-compose-server.yml` created
- `server.env` created
- Remind user to:
  - Set CI/CD variables in GitLab UI
  - Run `/deploy-server` to push files to the server
  - Push the CI/CD files to the repo
