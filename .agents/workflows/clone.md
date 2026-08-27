---
description: >-
  Clone a repository from GitLab using credentials from .env.
  Use this workflow when the user wants to clone a specific GitLab repo.
slash_command: clone
---

# /clone — Clone GitLab Repository

Clone a GitLab repository into the `project/` directory.

## Prerequisites
- User provides the GitLab repo URL.
- If repo is private, `.env` must have `GITLAB_USERNAME` and `GITLAB_PASS`.

## Steps

### 1. Parse URL
Extract host, group, and repo name from the user-provided URL.
- Strip trailing slashes and `.git` suffix if present.
- Normalize to HTTP format: `http://<host>/<group>/<repo>.git`

### 2. Clone
```bash
git clone http://<host>/<group>/<repo>.git
```
Run from `/Users/alfarizy/Desktop/Project/cicd-factory/project`.

If clone fails with authentication error:
- Read `.env` to get `GITLAB_USERNAME` and `GITLAB_PASS`.
- Retry with credentials: `http://${GITLAB_USERNAME}:${GITLAB_PASS}@<host>/<group>/<repo>.git`

### 3. Confirm
List the cloned directory contents and report success.
