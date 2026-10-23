---
description: >-
  Commit and push the newly generated CI/CD configuration files to the dev branch.
  Use this workflow after running /setup-project and /deploy-server.
slash_command: push-cicd
---

# /push-cicd — Push CI/CD Config to GitLab

Commit and push the CI/CD template and server configuration files to the project repository.

## Prerequisites
- User specifies the project name.
- Project exists in the `project/` directory and is a valid Git repository.

## Steps

### 1. Identify Project
Ask the user which project in `project/` to push if not provided.
Set `REPO_NAME` to the project folder name.

### 2. Switch Branch
Navigate to the repository and ensure we are on the `dev` branch.
```bash
cd /Users/alfarizy/Desktop/Project/cicd-factory/project/<repo-name>
git checkout -b dev || git checkout dev
```

### 3. Stage Files
Add only the CI/CD related files (never add `server.env` or `.env` as they contain server secrets):
```bash
git add .gitlab/ .gitlab-ci.yml docker-compose-server.yml .gitignore
```

### 4. Commit
```bash
git commit -m "chore: setup CI/CD pipeline configuration"
```

### 5. Push
Pull any remote changes to avoid conflicts, then push.
```bash
git pull origin dev --rebase
git push -u origin dev
```
*(Note: If the push hangs or requires interactive auth, run it with `BypassSandbox: true`)*

### 6. Summary
Report success and remind the user that the GitLab pipeline should now trigger automatically.
