# Chapter 4.3 — Containerization

[← Previous: CI/CD Tools](02-cicd-tools.md) | [Next: Container Orchestration →](04-container-orchestration.md)

---

## Overview

**Containerization** packages an application with all its dependencies into a standardized unit (a container) that runs consistently on any environment. Docker is the dominant containerization platform. Containers solve the age-old problem: *"But it works on my machine!"*

---

## Containers vs Virtual Machines

```
┌──────────────────────────────────────────────────────────┐
│  VIRTUAL MACHINES vs CONTAINERS                          │
│                                                          │
│  ┌────────────────────┐  ┌────────────────────┐         │
│  │   VIRTUAL MACHINES │  │    CONTAINERS      │         │
│  │                    │  │                    │         │
│  │ ┌────┐ ┌────┐     │  │ ┌────┐ ┌────┐     │         │
│  │ │App1│ │App2│     │  │ │App1│ │App2│     │         │
│  │ ├────┤ ├────┤     │  │ ├────┤ ├────┤     │         │
│  │ │Bins│ │Bins│     │  │ │Bins│ │Bins│     │         │
│  │ ├────┤ ├────┤     │  │ └──┬─┘ └──┬─┘     │         │
│  │ │ OS │ │ OS │     │  │    └──┬───┘        │         │
│  │ └────┘ └────┘     │  │  ┌───▼────────┐    │         │
│  │ ┌────────────────┐│  │  │ Container  │    │         │
│  │ │  Hypervisor    ││  │  │  Runtime   │    │         │
│  │ └────────────────┘│  │  └────────────┘    │         │
│  │ ┌────────────────┐│  │ ┌────────────────┐ │         │
│  │ │   Host OS      ││  │ │   Host OS      │ │         │
│  │ └────────────────┘│  │ └────────────────┘ │         │
│  │ ┌────────────────┐│  │ ┌────────────────┐ │         │
│  │ │   Hardware     ││  │ │   Hardware     │ │         │
│  │ └────────────────┘│  │ └────────────────┘ │         │
│  └────────────────────┘  └────────────────────┘         │
│                                                          │
│  VMs: Each has FULL OS     Containers: SHARE host OS    │
│  Boot: Minutes             Boot: Seconds                │
│  Size: Gigabytes           Size: Megabytes              │
│  Isolation: Strong         Isolation: Process-level     │
└──────────────────────────────────────────────────────────┘
```

| Feature | Virtual Machine | Container |
|---------|----------------|-----------|
| **Boot Time** | Minutes | Seconds |
| **Size** | GBs | MBs |
| **OS** | Full guest OS per VM | Shares host OS kernel |
| **Isolation** | Hardware-level | Process-level |
| **Performance** | Near-native | Native |
| **Density** | 10s per host | 100s per host |
| **Use Case** | Different OSes, strong isolation | Microservices, CI/CD |

---

## Docker Architecture

```
┌──────────────────────────────────────────────────────────┐
│  DOCKER ARCHITECTURE                                     │
│                                                          │
│  ┌────────────────────────────────────────────┐          │
│  │  Docker Client (CLI)                       │          │
│  │  docker build, docker run, docker push     │          │
│  └──────────────────────┬─────────────────────┘          │
│                         │ REST API                       │
│  ┌──────────────────────▼─────────────────────┐          │
│  │  Docker Daemon (dockerd)                   │          │
│  │  ├── Manages images, containers, networks  │          │
│  │  └── Manages volumes                       │          │
│  └──────────┬───────────┬─────────────────────┘          │
│             │           │                                │
│  ┌──────────▼───┐ ┌─────▼──────────┐                    │
│  │ Local Images │ │  Docker Hub    │                    │
│  │              │ │  (Registry)    │                    │
│  │ ubuntu:22.04 │ │  ┌──────────┐ │                    │
│  │ node:20      │ │  │ Public   │ │                    │
│  │ my-app:v1    │ │  │ Private  │ │                    │
│  └──────────────┘ │  └──────────┘ │                    │
│                   └───────────────┘                    │
└──────────────────────────────────────────────────────────┘
```

---

## Dockerfile: Building Images

### Production-Ready Dockerfile

```dockerfile
# ═══ Stage 1: Build ═══
FROM node:20-alpine AS builder
WORKDIR /app

# Copy dependency files first (cache optimization)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY src/ ./src/
COPY tsconfig.json ./
RUN npm run build

# ═══ Stage 2: Production ═══
FROM node:20-alpine AS production

# Security: create non-root user
RUN addgroup -g 1001 appgroup && \
    adduser -u 1001 -G appgroup -s /bin/sh -D appuser

WORKDIR /app

# Copy only what's needed from build stage
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Security: don't run as root
USER appuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1

# Start application
CMD ["node", "dist/server.js"]
```

### Dockerfile Best Practices

```
┌──────────────────────────────────────────────────────────┐
│  DOCKERFILE BEST PRACTICES                               │
│                                                          │
│  1. Use multi-stage builds    (smaller final image)      │
│  2. Use specific base tags    (node:20-alpine, not node) │
│  3. COPY package.json first   (leverage layer cache)     │
│  4. Run as non-root user      (security)                 │
│  5. Use .dockerignore         (exclude node_modules etc) │
│  6. One process per container (single responsibility)    │
│  7. Add HEALTHCHECK           (container self-reporting) │
│  8. Use COPY, not ADD         (more explicit)            │
│  9. Minimize layers           (combine RUN commands)     │
│  10. Scan for vulnerabilities (docker scout, trivy)      │
└──────────────────────────────────────────────────────────┘
```

### .dockerignore

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
Dockerfile
docker-compose*.yml
README.md
.vscode
coverage
.nyc_output
```

---

## Docker Commands

```bash
# ═══ IMAGES ═══
docker build -t my-app:v1 .                    # Build image
docker images                                   # List images
docker pull nginx:latest                        # Download image
docker push registry.io/my-app:v1              # Upload image
docker rmi my-app:v1                            # Remove image

# ═══ CONTAINERS ═══
docker run -d -p 8080:3000 --name web my-app:v1  # Run container
docker ps                                         # List running
docker ps -a                                      # List all
docker stop web                                   # Stop container
docker start web                                  # Start container
docker rm web                                     # Remove container
docker logs web                                   # View logs
docker logs -f web                                # Follow logs
docker exec -it web /bin/sh                       # Shell into container

# ═══ DOCKER COMPOSE ═══
docker compose up -d                              # Start all services
docker compose down                               # Stop all services
docker compose logs -f                            # Follow all logs
docker compose ps                                 # List services

# ═══ CLEANUP ═══
docker system prune -a                            # Remove unused everything
docker volume prune                               # Remove unused volumes
```

---

## Docker Compose: Multi-Container Applications

```yaml
# docker-compose.yml
version: '3.9'

services:
  # Web Application
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=database
      - REDIS_HOST=cache
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped
    networks:
      - app-network

  # PostgreSQL Database
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis Cache
  cache:
    image: redis:7-alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # Nginx Reverse Proxy
  nginx:
    image: nginx:1.25-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - web
    networks:
      - app-network

volumes:
  db-data:
  redis-data:

networks:
  app-network:
    driver: bridge

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

## Container Image Layers

```
┌──────────────────────────────────────────────────────────┐
│  IMAGE LAYERS (each instruction creates a layer)         │
│                                                          │
│  FROM node:20-alpine          ┌─────────────────────┐   │
│                               │ Layer 1: Base OS    │   │
│                               │ (alpine + node)     │   │
│  COPY package*.json ./        ├─────────────────────┤   │
│                               │ Layer 2: pkg files  │   │
│  RUN npm ci                   ├─────────────────────┤   │
│                               │ Layer 3: node_mods  │   │
│  COPY src/ ./src/             ├─────────────────────┤   │
│                               │ Layer 4: source     │   │
│  CMD ["node", "server.js"]    ├─────────────────────┤   │
│                               │ Layer 5: metadata   │   │
│                               └─────────────────────┘   │
│                                                          │
│  ⚡ CACHE: If a layer hasn't changed, Docker reuses it  │
│  Put frequently-changing layers LAST for fast rebuilds   │
│                                                          │
│  Change src/ code → only layers 4-5 rebuild              │
│  Change package.json → layers 2-5 rebuild                │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Microservices with Docker Compose

```
ARCHITECTURE:
                        ┌─────────┐
                        │  Nginx  │ :80
                        │ (proxy) │
                        └────┬────┘
                    ┌────────┼────────┐
               ┌────▼───┐ ┌─▼──────┐ ┌▼───────┐
               │ User   │ │ Order  │ │Product │
               │Service │ │Service │ │Service │
               │ :3001  │ │ :3002  │ │ :3003  │
               └───┬────┘ └───┬────┘ └───┬────┘
                   │          │          │
               ┌───▼────┐ ┌──▼─────┐ ┌──▼─────┐
               │Postgres│ │Postgres│ │  Redis  │
               │(users) │ │(orders)│ │ (cache) │
               └────────┘ └────────┘ └────────┘

Each service:
├── Has its own Dockerfile
├── Has its own database
├── Communicates via HTTP/gRPC
├── Can be deployed independently
└── Scales independently
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Image too large (1GB+) | Not using multi-stage builds | Use multi-stage builds with alpine base images |
| Container exits immediately | Process runs in background | Use foreground mode; check `docker logs` |
| Permission denied | Running as root, file ownership | Use `USER` directive; match file permissions |
| Build is slow | Not leveraging cache | Order Dockerfile: dependencies first, code last |
| "Port already in use" | Host port conflict | Use different host port: `-p 8081:3000` |
| Can't connect between containers | Different networks | Put containers on same Docker network; use service names as hostnames |

---

## Summary Table

| Container Concept | Description |
|------------------|-------------|
| **Container** | Lightweight, portable package with app + dependencies |
| **Image** | Read-only template for creating containers |
| **Dockerfile** | Instructions to build a Docker image |
| **Docker Compose** | Tool for defining multi-container applications |
| **Registry** | Storage for Docker images (Docker Hub, ECR, GCR) |
| **Layer** | Each Dockerfile instruction creates a cached layer |
| **Multi-stage Build** | Multiple FROM statements to reduce final image size |
| **Volume** | Persistent storage that survives container restarts |

---

## Quick Revision Questions

1. **What is a container, and how does it differ from a virtual machine?**
   <details><summary>Answer</summary>A container is a lightweight, portable package that includes an application and its dependencies. Unlike VMs, containers share the host OS kernel (no guest OS), boot in seconds (not minutes), use MBs (not GBs), and provide process-level isolation. VMs offer stronger isolation with separate OS instances.</details>

2. **What is a multi-stage Docker build, and why use it?**
   <details><summary>Answer</summary>Multi-stage builds use multiple FROM statements in a Dockerfile. Build tools and source code exist in early stages, while only the compiled output is copied to the final stage. This dramatically reduces image size (e.g., from 1GB to 100MB) and removes build tools from production images, improving security.</details>

3. **Why should you order Dockerfile instructions carefully?**
   <details><summary>Answer</summary>Docker caches layers. If a layer changes, all subsequent layers are rebuilt. By placing rarely-changing instructions first (OS, dependencies) and frequently-changing ones last (source code), you maximize cache hits. Example: COPY package.json before COPY src/ so npm install is cached when only code changes.</details>

4. **What is Docker Compose, and when would you use it?**
   <details><summary>Answer</summary>Docker Compose defines and runs multi-container applications using a YAML file. Use it when your app needs multiple services (web app + database + cache + proxy). It manages networking, volumes, dependencies, and startup order. Ideal for local development and testing environments.</details>

5. **Why should containers not run as root?**
   <details><summary>Answer</summary>Running as root inside a container is a security risk. If an attacker escapes the container, they'd have root access on the host. Best practice: create a non-root user in the Dockerfile with `RUN adduser` and switch to it with `USER`. This follows the principle of least privilege.</details>

6. **What is a container registry, and name three examples?**
   <details><summary>Answer</summary>A container registry is a storage service for Docker images. Teams push built images to registries and pull them for deployment. Examples: 1) Docker Hub — public default registry. 2) Amazon ECR — AWS-native private registry. 3) GitHub Container Registry (ghcr.io) — GitHub-integrated. Others: Google GCR, Azure ACR, Harbor (self-hosted).</details>

---

[← Previous: CI/CD Tools](02-cicd-tools.md) | [Next: Container Orchestration →](04-container-orchestration.md)
