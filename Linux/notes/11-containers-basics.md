# Linux Containers Basics

A comprehensive guide to containerization on Linux with Docker and Podman.

## Table of Contents

1. [Container Fundamentals](#1-container-fundamentals)
2. [Docker Installation & Setup](#2-docker-installation--setup)
3. [Docker Images](#3-docker-images)
4. [Docker Containers](#4-docker-containers)
5. [Dockerfile](#5-dockerfile)
6. [Docker Networking](#6-docker-networking)
7. [Docker Volumes](#7-docker-volumes)
8. [Docker Compose](#8-docker-compose)
9. [Podman (Rootless Alternative)](#9-podman-rootless-alternative)
10. [Container Security](#10-container-security)

---

## 1. Container Fundamentals

### What Are Containers?

Containers are lightweight, isolated environments that package applications with their dependencies.

```
┌─────────────────────────────────────────────┐
│              Traditional VMs                 │
├─────────┬─────────┬─────────┬───────────────┤
│  App A  │  App B  │  App C  │               │
├─────────┼─────────┼─────────┤               │
│  Bins/  │  Bins/  │  Bins/  │               │
│  Libs   │  Libs   │  Libs   │               │
├─────────┼─────────┼─────────┤               │
│Guest OS │Guest OS │Guest OS │               │
├─────────┴─────────┴─────────┤               │
│         Hypervisor          │               │
├─────────────────────────────┤               │
│         Host OS             │               │
├─────────────────────────────┤               │
│         Hardware            │               │
└─────────────────────────────┘               │
                                              │
┌─────────────────────────────────────────────┤
│              Containers                      │
├─────────┬─────────┬─────────┬───────────────┤
│  App A  │  App B  │  App C  │               │
├─────────┼─────────┼─────────┤               │
│  Bins/  │  Bins/  │  Bins/  │               │
│  Libs   │  Libs   │  Libs   │               │
├─────────┴─────────┴─────────┤               │
│     Container Runtime       │               │
├─────────────────────────────┤               │
│         Host OS             │               │
├─────────────────────────────┤               │
│         Hardware            │               │
└─────────────────────────────┘
```

### Key Technologies

| Technology | Purpose |
|------------|---------|
| **namespaces** | Process isolation (PID, network, mount, etc.) |
| **cgroups** | Resource limiting (CPU, memory, I/O) |
| **Union filesystems** | Layered image storage |
| **Container runtime** | OCI-compliant execution (runc, crun) |

### Containers vs VMs

| Aspect | Containers | Virtual Machines |
|--------|------------|------------------|
| Size | Megabytes | Gigabytes |
| Startup | Seconds | Minutes |
| Isolation | Process-level | Hardware-level |
| OS | Shares host kernel | Full guest OS |
| Overhead | Minimal | Significant |
| Density | High | Lower |

---

## 2. Docker Installation & Setup

### Install Docker (Ubuntu/Debian)

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release

# Add Docker GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Verify installation
sudo docker run hello-world
```

### Install Docker (Fedora/RHEL)

```bash
# Install
sudo dnf install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Start Docker
sudo systemctl enable --now docker
```

### Install Docker (Arch Linux)

```bash
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
```

### Post-Installation

```bash
# Run Docker without sudo
sudo usermod -aG docker $USER
newgrp docker  # Or logout/login

# Verify
docker run hello-world

# Configure Docker daemon
sudo vim /etc/docker/daemon.json
```

```json
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-address-pools": [
    {"base": "172.17.0.0/16", "size": 24}
  ]
}
```

---

## 3. Docker Images

### Image Basics

```bash
# Search images
docker search nginx

# Pull image
docker pull nginx
docker pull nginx:1.25
docker pull nginx:alpine

# List images
docker images
docker image ls

# Image details
docker inspect nginx
docker history nginx

# Remove image
docker rmi nginx
docker image rm nginx

# Remove unused images
docker image prune
docker image prune -a  # Remove all unused
```

### Image Tags

```bash
# Pull specific version
docker pull ubuntu:22.04
docker pull python:3.11-slim
docker pull node:20-alpine

# Tag image
docker tag myapp:latest myregistry.com/myapp:v1.0

# Push to registry
docker login
docker push myregistry.com/myapp:v1.0
```

### Save and Load Images

```bash
# Save image to file
docker save -o nginx.tar nginx:latest

# Load image from file
docker load -i nginx.tar

# Export container filesystem
docker export container_id > container.tar

# Import as image
docker import container.tar myimage:latest
```

---

## 4. Docker Containers

### Running Containers

```bash
# Basic run
docker run nginx

# Run in background (detached)
docker run -d nginx

# Run with name
docker run -d --name webserver nginx

# Run interactively
docker run -it ubuntu bash

# Run with auto-remove
docker run --rm ubuntu echo "Hello"

# Run with port mapping
docker run -d -p 8080:80 nginx
docker run -d -p 127.0.0.1:8080:80 nginx

# Run with environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Run with volume
docker run -d -v /host/path:/container/path nginx
docker run -d -v myvolume:/data nginx

# Run with resource limits
docker run -d --memory=512m --cpus=1 nginx

# Run with restart policy
docker run -d --restart=always nginx
docker run -d --restart=unless-stopped nginx
```

### Container Management

```bash
# List containers
docker ps              # Running only
docker ps -a           # All containers
docker ps -q           # IDs only

# Container info
docker inspect container_name
docker logs container_name
docker logs -f container_name  # Follow
docker logs --tail 100 container_name

# Container stats
docker stats
docker top container_name

# Start/stop/restart
docker start container_name
docker stop container_name
docker restart container_name

# Pause/unpause
docker pause container_name
docker unpause container_name

# Remove container
docker rm container_name
docker rm -f container_name  # Force

# Remove all stopped containers
docker container prune
```

### Executing Commands

```bash
# Execute command in running container
docker exec container_name ls /app
docker exec -it container_name bash
docker exec -it container_name sh

# Execute as different user
docker exec -u root container_name command
```

### Copy Files

```bash
# Copy to container
docker cp file.txt container_name:/path/

# Copy from container
docker cp container_name:/path/file.txt ./

# Copy directory
docker cp ./mydir container_name:/app/
```

---

## 5. Dockerfile

### Basic Dockerfile

```dockerfile
# Base image
FROM ubuntu:22.04

# Metadata
LABEL maintainer="user@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY . .
# Or selectively
COPY package*.json ./

# Run commands (creates layers)
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Environment variables
ENV APP_ENV=production
ENV PORT=8080

# Expose port (documentation)
EXPOSE 8080

# Default command
CMD ["python3", "app.py"]
```

### Multi-Stage Build

```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Python Application

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first (caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

### Go Application

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Final stage
FROM scratch
COPY --from=builder /app/main /main
EXPOSE 8080
ENTRYPOINT ["/main"]
```

### Dockerfile Best Practices

```dockerfile
# 1. Use specific base image tags
FROM node:20.10-alpine  # Not :latest

# 2. Minimize layers
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# 3. Use .dockerignore
# .dockerignore file:
# node_modules
# .git
# *.log

# 4. Don't run as root
USER nonrootuser

# 5. Use COPY instead of ADD (unless extracting archives)
COPY file.txt /app/

# 6. Set proper WORKDIR
WORKDIR /app

# 7. Use multi-stage builds for smaller images

# 8. Order commands by change frequency (least to most)
```

### Build Image

```bash
# Build image
docker build -t myapp:latest .

# Build with different Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:1.0 .

# Build with no cache
docker build --no-cache -t myapp:latest .

# Build for different platform
docker build --platform linux/amd64 -t myapp:latest .
```

---

## 6. Docker Networking

### Network Types

| Driver | Description |
|--------|-------------|
| bridge | Default, isolated network |
| host | Use host's network |
| none | No networking |
| overlay | Multi-host networking |
| macvlan | Assign MAC address |

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create mynetwork
docker network create --driver bridge --subnet 172.20.0.0/16 mynetwork

# Inspect network
docker network inspect bridge

# Connect container to network
docker network connect mynetwork container_name

# Disconnect
docker network disconnect mynetwork container_name

# Remove network
docker network rm mynetwork
docker network prune
```

### Container Communication

```bash
# Create network
docker network create app-network

# Run containers on same network
docker run -d --name db --network app-network postgres
docker run -d --name app --network app-network -e DB_HOST=db myapp

# Containers can reach each other by name
# Inside 'app' container: ping db
```

### Port Mapping

```bash
# Map port
docker run -p 8080:80 nginx           # Host 8080 → Container 80
docker run -p 127.0.0.1:8080:80 nginx # Localhost only
docker run -p 8080-8090:80-90 nginx   # Port range
docker run -P nginx                    # Random ports for EXPOSE

# Check port mappings
docker port container_name
```

---

## 7. Docker Volumes

### Volume Types

```bash
# Named volume (managed by Docker)
docker volume create mydata
docker run -v mydata:/app/data nginx

# Bind mount (host path)
docker run -v /host/path:/container/path nginx
docker run -v $(pwd):/app nginx

# tmpfs mount (memory)
docker run --tmpfs /tmp nginx
```

### Volume Commands

```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

### Volume Mount Options

```bash
# Read-only mount
docker run -v mydata:/app/data:ro nginx

# Mount with specific options
docker run --mount type=volume,source=mydata,target=/app/data nginx
docker run --mount type=bind,source=/host/path,target=/container/path nginx
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=100m nginx
```

### Backup and Restore

```bash
# Backup volume
docker run --rm -v mydata:/data -v $(pwd):/backup ubuntu \
    tar cvf /backup/backup.tar /data

# Restore volume
docker run --rm -v mydata:/data -v $(pwd):/backup ubuntu \
    tar xvf /backup/backup.tar -C /
```

---

## 8. Docker Compose

### Basic Compose File

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - api

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgres://db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

networks:
  default:
    name: myapp-network
```

### Full-Featured Compose

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    env_file:
      - .env
    volumes:
      - ./data:/app/data
      - logs:/app/logs
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  db:
    image: postgres:15-alpine
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  postgres-data:
  logs:

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Compose Commands

```bash
# Start services
docker compose up
docker compose up -d              # Detached
docker compose up --build         # Rebuild images
docker compose up -d --scale web=3  # Scale service

# Stop services
docker compose down
docker compose down -v            # Remove volumes too
docker compose down --rmi all     # Remove images too

# Service management
docker compose start
docker compose stop
docker compose restart
docker compose pause
docker compose unpause

# View status
docker compose ps
docker compose logs
docker compose logs -f web
docker compose top

# Execute command
docker compose exec web sh

# Build images
docker compose build
docker compose build --no-cache

# Pull images
docker compose pull
```

---

## 9. Podman (Rootless Alternative)

### What is Podman?

Podman is a daemonless container engine compatible with Docker CLI.

### Install Podman

```bash
# Ubuntu
sudo apt install podman

# Fedora
sudo dnf install podman

# Arch
sudo pacman -S podman
```

### Podman Commands

```bash
# Same as Docker
podman pull nginx
podman run -d -p 8080:80 nginx
podman ps
podman images
podman stop container_id
podman rm container_id

# Rootless containers (no sudo needed)
podman run --rm -it alpine sh
```

### Docker Compatibility

```bash
# Create Docker alias
alias docker=podman

# Or use podman-docker package
sudo apt install podman-docker
```

### Podman Compose

```bash
# Install podman-compose
pip install podman-compose

# Use with existing docker-compose.yml
podman-compose up -d
podman-compose down
```

### Podman Pods

```bash
# Create pod (like Kubernetes pod)
podman pod create --name mypod -p 8080:80

# Add containers to pod
podman run -d --pod mypod nginx
podman run -d --pod mypod redis

# Manage pod
podman pod ps
podman pod stop mypod
podman pod rm mypod
```

### Generate Kubernetes YAML

```bash
# Generate from container
podman generate kube mycontainer > container.yaml

# Generate from pod
podman generate kube mypod > pod.yaml

# Play Kubernetes YAML
podman play kube pod.yaml
```

---

## 10. Container Security

### Security Best Practices

```bash
# 1. Don't run as root
docker run --user 1000:1000 myapp

# 2. Use read-only filesystem
docker run --read-only myapp

# 3. Drop capabilities
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# 4. Limit resources
docker run --memory=512m --cpus=1 myapp

# 5. Use security options
docker run --security-opt no-new-privileges myapp
docker run --security-opt seccomp=profile.json myapp

# 6. Network restrictions
docker run --network none myapp
```

### Dockerfile Security

```dockerfile
# Use non-root user
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Use specific versions
FROM python:3.11.6-slim

# Don't store secrets in image
# Use build secrets or runtime secrets

# Scan for vulnerabilities
# docker scan myimage
```

### Scan Images

```bash
# Docker Scout (built-in)
docker scout cves myimage

# Trivy
trivy image myimage

# Snyk
snyk container test myimage
```

### Runtime Security

```bash
# Read-only containers
docker run --read-only --tmpfs /tmp myapp

# No privilege escalation
docker run --security-opt no-new-privileges myapp

# AppArmor profile
docker run --security-opt apparmor=docker-default myapp

# Seccomp profile
docker run --security-opt seccomp=/path/to/profile.json myapp
```

---

## Quick Reference

### Essential Commands

```bash
# Images
docker pull IMAGE
docker images
docker rmi IMAGE
docker build -t NAME .

# Containers
docker run -d -p HOST:CONTAINER IMAGE
docker ps [-a]
docker logs CONTAINER
docker exec -it CONTAINER sh
docker stop CONTAINER
docker rm CONTAINER

# Volumes
docker volume create NAME
docker volume ls
docker volume rm NAME

# Networks
docker network create NAME
docker network ls

# Compose
docker compose up -d
docker compose down
docker compose logs -f

# Cleanup
docker system prune -a
```

### Dockerfile Cheatsheet

```dockerfile
FROM image:tag
LABEL key=value
WORKDIR /app
COPY src dest
RUN command
ENV KEY=value
EXPOSE port
USER username
CMD ["executable", "param"]
ENTRYPOINT ["executable"]
```

---

*Last Updated: April 2026*
