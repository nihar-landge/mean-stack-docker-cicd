# MEAN Stack CRUD - Dockerized CI/CD Deployment

Production-ready DevOps implementation of a MEAN stack CRUD application using Docker, Docker Compose, GitHub Actions, EC2, and Nginx reverse proxy.

Repository: `https://github.com/nihar-landge/mean-stack-docker-cicd`

## Overview

This project provides a Tutorials CRUD application where each tutorial has:

- `id`
- `title`
- `description`
- `published`

Capabilities:

- Create, read, update, delete tutorials
- Search tutorials by title
- Serve application through Nginx on port `80`
- Deploy automatically using CI/CD

## Architecture

Browser --> Nginx (Frontend Container, Port 80) -->

- `/` -> Angular static app
- `/api/*` -> Node.js/Express backend (internal container)

Backend --> MongoDB (`mongo:5` container)

CI/CD flow:

- Push to `main`
- GitHub Actions builds multi-arch Docker images (`amd64`, `arm64`)
- Pushes images to Docker Hub
- SSH deploy to EC2
- `docker compose pull && docker compose up -d`

## Repository Structure

`backend/`

- `Dockerfile`
- `server.js`
- `app/config/db.config.js`

`frontend/`

- `Dockerfile`
- `nginx.conf`
- Angular source

Root:

- `docker-compose.yml` (local build-based compose)
- `docker-compose.prod.yml` (Docker Hub image-based compose)
- `.github/workflows/cicd.yml`

## Prerequisites

- Docker + Docker Compose
- Git
- GitHub repository
- Docker Hub account and access token
- AWS EC2 Ubuntu instance (recommended: `t3.small`)

## Local Development (without Docker)

Backend:

```bash
cd backend
npm install
node server.js
```

Frontend:

```bash
cd frontend
npm install
npx ng serve --port 8081
```

Open `http://localhost:8081`.

## Local Container Run

```bash
docker compose up -d --build
docker compose ps
```

Open `http://localhost`.

## Production Deployment on EC2

### 1) Provision VM

- Launch Ubuntu EC2 instance
- Security Group:
  - `22/tcp` (SSH)
  - `80/tcp` (HTTP)

### 2) Install Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin git
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
newgrp docker
```

### 3) Clone Repository

```bash
sudo mkdir -p /opt/crud-dd-task-mean-app
sudo chown -R ubuntu:ubuntu /opt/crud-dd-task-mean-app
cd /opt/crud-dd-task-mean-app
git clone https://github.com/nihar-landge/mean-stack-docker-cicd.git
cd mean-stack-docker-cicd
```

### 4) Deploy with Production Compose

```bash
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d --remove-orphans
docker compose -f docker-compose.prod.yml ps
```

Open `http://<EC2_PUBLIC_IP>`.

## CI/CD Configuration (GitHub Actions)

Workflow file: `.github/workflows/cicd.yml`

### Required GitHub Secrets

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `EC2_HOST`
- `EC2_USER` (`ubuntu`)
- `EC2_SSH_KEY` (full private key content)

### Pipeline Steps

1. Checkout repository
2. Setup QEMU + Buildx
3. Login to Docker Hub
4. Build and push backend image (`latest` and commit SHA tags)
5. Build and push frontend image (`latest` and commit SHA tags)
6. SSH to EC2 and run deployment commands

## Nginx Reverse Proxy Details

Config file: `frontend/nginx.conf`

- `location /` serves Angular app
- `location /api/` proxies to backend service (`http://backend:8080`)

Frontend API base URL is set to relative path in `frontend/src/app/services/tutorial.service.ts`:

- `const baseUrl = '/api/tutorials';`

This ensures same-origin requests through Nginx and avoids CORS issues in deployment.

## Verification Checklist

On EC2:

```bash
docker compose -f docker-compose.prod.yml ps
curl -I http://localhost
curl -i http://localhost/api/tutorials
```

Expected:

- Containers are `Up`
- HTTP status returns `200`
- API returns JSON (empty array or records)

## Screenshots

Add screenshots to `docs/screenshots/` using the names below:

1. `01-github-actions-workflow.png` - CI/CD workflow definition
2. `02-github-actions-success-run.png` - Successful pipeline execution
3. `03-dockerhub-images.png` - Docker image build and push results
4. `04-ec2-deployment-containers.png` - Running containers on VM
5. `05-app-ui-home.png` - Application UI running on EC2 public IP
6. `06-nginx-config-and-routing.png` - Nginx config/infrastructure proof

Example markdown usage:

```md
![CI/CD Run](docs/screenshots/02-github-actions-success-run.png)
```

## Troubleshooting

- `exec format error`: image architecture mismatch -> use multi-arch builds in CI
- `npm ci` fails in backend image: ensure `backend/package-lock.json` is committed
- SSH deploy fails from Actions:
  - check `EC2_SSH_KEY` format
  - verify EC2 `22` inbound rule allows GitHub runner access

## Author

Nihar Landge
