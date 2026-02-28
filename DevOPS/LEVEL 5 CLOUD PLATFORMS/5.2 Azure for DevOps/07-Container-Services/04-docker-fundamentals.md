# Chapter 4: Docker and Container Fundamentals

## Overview
Containers package applications with their dependencies into isolated, portable units. **Docker** is the most popular container runtime. Understanding containers is essential for working with AKS, ACI, Container Apps, and modern DevOps pipelines.

---

## 4.1 Container vs Virtual Machine

```
┌────── CONTAINERS vs VMs ──────────────────────┐
│                                                │
│  VIRTUAL MACHINES:          CONTAINERS:        │
│  ┌──────────────────┐      ┌──────────────┐   │
│  │ App A │ App B    │      │App A│App B   │   │
│  ├───────┼──────────┤      ├─────┼────────┤   │
│  │ Bins/Libs│Bins/Libs│     │Bins │ Bins   │   │
│  ├───────┼──────────┤      │Libs │ Libs   │   │
│  │Guest OS│ Guest OS │      ├─────┴────────┤   │
│  ├────────┴─────────┤      │Container     │   │
│  │   Hypervisor     │      │Runtime       │   │
│  ├──────────────────┤      ├──────────────┤   │
│  │   Host OS        │      │   Host OS    │   │
│  ├──────────────────┤      ├──────────────┤   │
│  │   Hardware       │      │   Hardware   │   │
│  └──────────────────┘      └──────────────┘   │
│                                                │
│  VMs: Minutes to start, GBs in size           │
│  Containers: Seconds to start, MBs in size    │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 4.2 Dockerfile Basics

```dockerfile
# Multi-stage build example
# Stage 1: Build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production image
FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

### Dockerfile Instructions

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files from host to image |
| `RUN` | Execute command during build |
| `EXPOSE` | Document the port (doesn't publish) |
| `ENV` | Set environment variable |
| `CMD` | Default command when container starts |
| `ENTRYPOINT` | Fixed command (CMD becomes arguments) |
| `USER` | Set non-root user (security best practice) |
| `HEALTHCHECK` | Container health check command |

---

## 4.3 Essential Docker Commands

```bash
# Build image
docker build -t myapp:v1.0 .

# Run container
docker run -d -p 8080:3000 --name myapp myapp:v1.0

# List running containers
docker ps

# View logs
docker logs myapp
docker logs -f myapp    # follow

# Execute command in container
docker exec -it myapp /bin/sh

# Stop and remove container
docker stop myapp
docker rm myapp

# List images
docker images

# Remove image
docker rmi myapp:v1.0

# Tag for ACR
docker tag myapp:v1.0 myregistry.azurecr.io/myapp:v1.0

# Push to ACR
docker push myregistry.azurecr.io/myapp:v1.0
```

---

## 4.4 Docker Compose for Local Dev

```yaml
# docker-compose.yml
version: '3.8'
services:
  web:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://api:8080
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_PASSWORD=secret
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop and remove
docker-compose down

# Rebuild after changes
docker-compose up -d --build
```

---

## 4.5 Best Practices for Containers

```
┌────── CONTAINER BEST PRACTICES ──────────┐
│                                           │
│  1. USE MULTI-STAGE BUILDS               │
│     → Smaller images, no build tools      │
│                                           │
│  2. USE SPECIFIC BASE IMAGE TAGS          │
│     ✅ node:18-alpine                     │
│     ❌ node:latest                        │
│                                           │
│  3. RUN AS NON-ROOT USER                  │
│     USER node (not root)                  │
│                                           │
│  4. USE .dockerignore                     │
│     Exclude node_modules, .git, etc.      │
│                                           │
│  5. MINIMIZE LAYERS                       │
│     Combine RUN commands with &&          │
│                                           │
│  6. SCAN FOR VULNERABILITIES              │
│     docker scan / trivy / ACR scanning    │
│                                           │
│  7. ONE PROCESS PER CONTAINER             │
│     Don't run multiple services           │
│                                           │
│  8. USE HEALTH CHECKS                     │
│     HEALTHCHECK CMD curl -f /health       │
│                                           │
└───────────────────────────────────────────┘
```

---

## 4.6 .dockerignore Example

```
node_modules
.git
.gitignore
Dockerfile
docker-compose.yml
*.md
.env
.vscode
coverage/
dist/
```

---

## 4.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Build fails | Missing file or bad instruction | Check COPY paths and Dockerfile syntax |
| Image too large | No multi-stage, unnecessary files | Use multi-stage build + .dockerignore |
| Container exits immediately | CMD process crashes | Check logs with `docker logs` |
| Port not accessible | Port not mapped | Use `-p hostPort:containerPort` |
| Permission denied | Running as root, file permissions | Add `USER` instruction, fix file ownership |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Containers** | Lightweight, portable, share host OS kernel |
| **Dockerfile** | Instructions to build a container image |
| **Multi-stage Build** | Separate build and runtime for smaller images |
| **Docker Compose** | Multi-container orchestration for local dev |
| **Best Practices** | Non-root user, specific tags, multi-stage, scan vulnerabilities |
| **ACR Integration** | Tag and push images to Azure Container Registry |

---

## Quick Revision Questions

1. **How do containers differ from VMs?**
   > Containers share the host OS kernel — they're lighter (MBs vs GBs), start in seconds, and are more portable. VMs include a full guest OS.

2. **What is a multi-stage build?**
   > A Dockerfile technique using multiple `FROM` stages — build tools in one stage, copy only artifacts to a minimal runtime stage.

3. **Why should containers run as non-root?**
   > Security: if the container is compromised, the attacker has limited privileges instead of root access.

4. **What is Docker Compose?**
   > A tool for defining and running multi-container applications locally using a YAML file.

5. **What is .dockerignore?**
   > A file that excludes files/folders from the Docker build context, reducing image size and build time.

---

[⬅ Previous: Azure Container Apps](03-azure-container-apps.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Unit 8 - Azure Repos ➡](../08-Azure-DevOps-Services/01-azure-repos.md)
