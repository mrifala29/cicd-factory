---
description: >-
  Clone a repository from GitLab using credentials from .env.
  Use this workflow when the user wants to clone a specific GitLab repo.
slash_command: clone
---

# /clone — Clone GitLab Repository

Clone a GitLab repository using stored credentials.

## Prerequisites
- User provides the GitLab repo URL.
- `.env` has `GITLAB_USERNAME` and `GITLAB_PASS`.

## Steps

### 1. Read Credentials
Read `.env` from workspace root.

### 2. Parse URL
Convert the user-provided URL to authenticated HTTP format:
- Input: `http://gitlab.example.com/group/repo.git` or `git@gitlab.example.com:group/repo.git`
- Output: `http://${GITLAB_USERNAME}:${GITLAB_PASS}@gitlab.example.com/group/repo.git`

### 3. Clone
```bash
cd /Users/alfarizy/Desktop/Project/cicd-factory/project
git clone <authenticated-url>
```

### 4. Confirm
List the cloned directory contents and report success.
