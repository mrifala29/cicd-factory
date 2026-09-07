# CI/CD Factory

Centralized repository for managing and standardizing CI/CD pipelines, templates, and deployments across all applications.

## 📂 Project Structure

- **`foundation/`** — CI/CD template base (`gitlab-ci.yml` and `.gitlab/ci/*.yml`). This folder contains standard definitions and should not be modified without explicit instruction.
- **`project/`** — Workspace for cloned repositories from GitLab. Each project resides in its own subfolder.
- **`.env`** — Local configuration containing credentials and environment variables (e.g., `GITLAB_USERNAME`, `GITLAB_PASS`, `SSH_PASS`, `PROJECT_DIR`). **Do not commit this file.**

## 🚀 CI/CD Pipeline Flow

The standard pipeline consists of four main stages:

1. **`check`**: Pre-deployment validation, such as checking for port collisions on the dev server.
2. **`build`**: Builds the application using Docker buildx with multi-platform support (e.g., `amd64`, `arm64`).
3. **`deploy`**: Deploys via SSH to the target server. This involves pulling the latest Docker image and executing `docker compose up`.
4. **`verify`**: Post-deployment verification to check container status and health.

### 📝 Deployment Conventions

- **Server Project Path**: `/opt/apps/<group>/<repo-name>`
- **Environment Variables**:
  - CI/CD templates utilize variables like `$PROJECT_DIR`, `$DEV_USER`, `$DEV_HOST`, `$DEV_PASSWORD`, `$REGISTRY_TOKEN`, `$IMAGE_NAME`.
  - These variables **MUST** be set directly within the GitLab UI (Settings > CI/CD > Variables), never hardcoded in files.
- **Server Configuration Files**:
  - `docker-compose-server.yml` is SCP'd to the server and renamed to `docker-compose.yml`.
  - `server.env` is SCP'd to the server and renamed to `.env`.

---

## 🛠 Operations Workflows

This project uses specific automated workflows to manage deployments securely and efficiently.

### 1. `/clone`
Clones a specific repository from GitLab into the `project/` directory using credentials from your `.env` file. (Attempts anonymous clone first, falls back to authenticated clone).

### 2. `/check-ports`
Before assigning ports for a new project, this workflow SSHs into the dev server and checks used ports (using `ss -tuln` and `docker ps`) to avoid port collisions.

### 3. `/setup-project`
Initializes a new project by cloning the GitLab repository and setting up the baseline CI/CD files (linking them to templates in `foundation/`).

### 4. `/deploy-server`
Deploys server configuration files. It uses `sshpass` to SCP `docker-compose-server.yml` and `server.env` to the correct project path on the dev server.

### 5. `/push-cicd`
Commits and pushes the newly generated or updated CI/CD configuration files back to the project's development branch on GitLab.

### 6. `/ssh-dev`
Provides direct SSH access into the dev server for manual inspection, debugging, or troubleshooting.

## 🎯 End-to-End Usage Pipeline (How to Use)

Berikut adalah panduan langkah demi langkah menggunakan CI/CD Factory ini dari awal (kloning) hingga pipeline berjalan di GitLab:


### 1. Kloning dan Setup Project Lokal
Gunakan perintah *slash command* berikut melalui chat assistant:
> "Tolong `/setup-project` untuk repository `https://gitlab.com/...`"

*Assistant akan otomatis mengkloning repo ke folder `project/` dan menyiapkan base file CI/CD yang terhubung ke template `foundation/`.*

### 2. Pengecekan Port Server (Mencegah Tabrakan)
Sebelum menentukan port aplikasi di file konfigurasi, pastikan port yang ingin dipakai masih kosong di server dev:
> "Tolong jalankan `/check-ports`"

*Assistant akan login ke server dan memastikan port tersebut belum terpakai oleh aplikasi lain.*

### 3. Siapkan Konfigurasi Server (Local)
Sesuaikan file konfigurasi aplikasi untuk deployment:
- Edit `docker-compose-server.yml` dan pastikan port yang digunakan sesuai dengan hasil pengecekan di langkah 2.
- Edit `server.env` jika ada environment variable tambahan yang dibutuhkan server.

### 4. Setup CI/CD Variables di GitLab (Manual Activity)
Langkah ini **wajib** dilakukan secara manual oleh Anda di GitLab UI. Buka repository Anda di GitLab, masuk ke **Settings** > **CI/CD** > **Variables**, dan tambahkan:
- `PROJECT_DIR`: Path tujuan deploy di server (contoh: `/opt/apps/mygroup/myrepo`)
- `DEV_USER`: Username SSH untuk server dev
- `DEV_HOST`: IP atau hostname server dev
- `DEV_PASSWORD`: Password SSH untuk server dev
- `REGISTRY_TOKEN`: Token akses untuk Docker Registry
- `IMAGE_NAME`: Nama image Docker (contoh: `myregistry/mygroup/myrepo`)

### 5. Deploy File Konfigurasi ke Server
Setelah konfigurasi siap, kirimkan file tersebut ke server dev:
> "Tolong jalankan `/deploy-server` untuk project ini"

*Assistant akan mengirim file `docker-compose-server.yml` (sebagai `docker-compose.yml`) dan `server.env` (sebagai `.env`) ke folder `PROJECT_DIR` di server menggunakan SCP.*

### 6. Push CI/CD & Trigger Pipeline
Terakhir, push perubahan CI/CD lokal ke GitLab untuk menjalankan prosesnya:
> "Tolong jalankan `/push-cicd`"

*Assistant akan melakukan commit & push file CI/CD ke branch `dev`. Begitu masuk GitLab, pipeline CI/CD akan otomatis berjalan untuk melakukan tahap `check`, `build`, `deploy`, dan `verify`!*

---

## 🔒 Security

- Local `.env` contents must **never** be printed in output or exposed in artifacts.
- Dev server hosts many applications. Always run `/check-ports` before defining services in `docker-compose-server.yml` or `server.env` to prevent conflicting services.
