---
description: >-
  Check used ports on the dev server to avoid port collisions.
  Use this workflow when the user wants to check available ports, or before assigning ports to a new project.
slash_command: check-ports
---

# /check-ports — Check Server Port Usage

Check which ports are already in use on the dev server to prevent collisions.

## Prerequisites
- `.env` has `SSH_PASS` populated.

## Steps

### 1. Read Config
Read `.env` from workspace root to get `SSH_PASS`.

### 2. Check Listening Ports
```bash
sshpass -p "${SSH_PASS}" ssh -o StrictHostKeyChecking=no server-dev "sudo ss -tuln"
```

### 3. Check Docker Container Ports
```bash
sshpass -p "${SSH_PASS}" ssh -o StrictHostKeyChecking=no server-dev "sudo docker ps --format 'table {{.Names}}\t{{.Ports}}\t{{.Status}}'"
```

### 4. Report
Compile a clear summary:
- List all ports currently in use (from both ss and docker ps).
- Highlight common port ranges already occupied.
- Suggest available port ranges for the new project.
- If user provides a specific port to check, confirm whether it's free or taken.
