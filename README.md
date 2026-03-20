# DevOps CI/CD Demo (GitHub Actions + Docker + AWS EC2)

A sample application demonstrating a complete **CI/CD pipeline** using **GitHub Actions**, **Docker**, **Docker Compose**, and an **AWS EC2** instance.

This repo is intended to showcase practical DevOps skills: automated deployments on PRs/pushes to `main`, container builds, and remote orchestration.

---

## What this project showcases

- CI/CD triggered on updates to the **`main`** branch
  - `push` to `main`
  - `pull_request` targeting `main`
- Containerization using **Dockerfiles**
- Multi-service orchestration using **docker-compose**
- Automated remote deployment to an **AWS EC2** instance over SSH
- Automated rebuild + restart using `docker-compose up --build -d`
- Image cleanup on the server with `docker image prune -af`

---

## Repository Structure

- `.github/workflows/deploy.yml` — GitHub Actions workflow that deploys to EC2
- `docker-compose.yml` — defines the multi-container setup
- `backend/` — Node.js + Express backend (containerized)
- `frontend/` — React (Vite) frontend built and served via Nginx (containerized)

---

## Services & Ports

Configured in `docker-compose.yml`:

- **Frontend**: `80:80` (served by Nginx)
- **Backend**: `5000:5000` (Node/Express)

> HTTP only (demo project).

---

## Run Locally (Docker Compose)

### Prerequisites
- Docker
- Docker Compose

### Start
From the repository root:

```bash
docker-compose up --build
```

Open:
- Frontend: `http://localhost/`
- Backend: `http://localhost:5000/`

### Stop

```bash
docker-compose down
```

---

## CI/CD Pipeline (GitHub Actions → EC2)

Workflow file: `.github/workflows/deploy.yml`

### Triggers
The workflow runs on:
- `push` to `main`
- `pull_request` targeting `main`

### What the workflow does
1. Checks out the repository code.
2. Connects to the EC2 instance using SSH (`appleboy/ssh-action`).
3. Runs the following commands on the EC2 host:

```bash
cd <APP_DIR>
git pull origin main
docker-compose down
docker-compose up --build -d
docker image prune -af
```

### Configurable app directory on EC2
The workflow currently uses a fixed path:

- `/home/ec2-user/DevOps`

If you want it configurable, update your workflow to use a secret or env variable (e.g., `APP_DIR`).

---

## Required GitHub Secrets

The workflow expects these repository secrets:

- `EC2_HOST` – EC2 public IP or DNS
- `EC2_USER` – SSH user (e.g., `ec2-user`)
- `EC2_SSH_KEY` – private key for SSH authentication

*(Optional improvement)*
- `APP_DIR` – directory where the repo is cloned on the EC2 instance

---

## EC2 Setup (high level)

On your EC2 instance you should have:

- Docker installed
- Docker Compose installed
- This repository cloned into your chosen directory (for example `/home/ec2-user/DevOps`)

Security Group inbound rules typically needed:
- `22/tcp` (SSH)
- `80/tcp` (HTTP)
- (optional) `5000/tcp` if you want to access the backend directly from the internet

---

## Dockerfiles

- `frontend/dockerfile`
  - Multi-stage build: builds the Vite app using Node, then serves `/dist` with Nginx on port **80**.
- `backend/dockerfile`
  - Runs the Node/Express backend and exposes port **5000**.

---

## License

See [`LICENSE`](./LICENSE).