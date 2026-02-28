# Chapter 1: Dockerfile Instructions

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

A **Dockerfile** is a text file containing instructions to build a Docker image. Each instruction creates a layer in the image. Mastering Dockerfile instructions is fundamental to creating efficient, secure, and maintainable container images.

---

## 1.1 Dockerfile Structure

```
  # Comment
  INSTRUCTION argument
  
  +---------------------------------------------------+
  |  Dockerfile                                        |
  |                                                    |
  |  FROM node:20-alpine          # Base image         |
  |  LABEL maintainer="dev@co"    # Metadata           |
  |  WORKDIR /app                  # Set directory      |
  |  COPY package*.json ./         # Copy deps file     |
  |  RUN npm ci                    # Install deps       |
  |  COPY . .                      # Copy source        |
  |  EXPOSE 3000                   # Document port      |
  |  USER node                     # Run as non-root    |
  |  CMD ["node", "server.js"]     # Default command    |
  +---------------------------------------------------+
```

---

## 1.2 All Dockerfile Instructions

### FROM — Base Image

```dockerfile
# Every Dockerfile MUST start with FROM
FROM ubuntu:22.04
FROM node:20-alpine
FROM python:3.11-slim
FROM scratch                  # Empty base (for static binaries)

# Multi-platform
FROM --platform=linux/amd64 node:20

# Named build stage (for multi-stage builds)
FROM node:20 AS builder
```

### RUN — Execute Commands

```dockerfile
# Shell form (runs in /bin/sh -c)
RUN apt-get update && apt-get install -y python3

# Exec form (no shell processing)
RUN ["apt-get", "install", "-y", "python3"]

# Multi-line for readability
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        git \
        vim && \
    rm -rf /var/lib/apt/lists/*

# With BuildKit cache mount
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y python3
```

### COPY — Copy Files from Build Context

```dockerfile
# Copy single file
COPY package.json /app/

# Copy multiple files
COPY package.json package-lock.json /app/

# Copy directory contents
COPY src/ /app/src/

# Copy with wildcard
COPY *.py /app/

# Change ownership during copy
COPY --chown=node:node . /app/

# Copy from a build stage
COPY --from=builder /app/dist /app/dist
```

### ADD — Copy with Extra Features

```dockerfile
# Similar to COPY but with extras:
ADD app.tar.gz /app/          # Auto-extracts tar archives
ADD https://example.com/file /app/  # Downloads from URL (not recommended)

# BEST PRACTICE: Prefer COPY over ADD
# Use ADD only when you need tar extraction
```

### CMD — Default Command

```dockerfile
# Exec form (PREFERRED — no shell processing)
CMD ["node", "server.js"]
CMD ["python3", "app.py"]

# Shell form (runs through /bin/sh -c)
CMD node server.js

# Can be overridden at runtime:
# docker run myapp node other-script.js
```

### ENTRYPOINT — Container Executable

```dockerfile
# Makes container behave like an executable
ENTRYPOINT ["python3", "app.py"]

# CMD provides default arguments to ENTRYPOINT
ENTRYPOINT ["python3"]
CMD ["app.py"]
# Runs: python3 app.py
# Override args: docker run myapp script.py
# Runs: python3 script.py
```

### ENTRYPOINT vs CMD Interaction

```
  +------------------+------------------+-----------------------+
  |                  | No ENTRYPOINT    | ENTRYPOINT ["ep"]     |
  +------------------+------------------+-----------------------+
  | No CMD           | Error            | ep                    |
  | CMD ["cmd"]      | cmd              | ep cmd                |
  | CMD cmd          | /bin/sh -c cmd   | ep /bin/sh -c cmd     |
  +------------------+------------------+-----------------------+
  
  docker run <image> arg1 arg2
  -> Replaces CMD, ENTRYPOINT stays
  
  docker run --entrypoint /bin/sh <image>
  -> Replaces ENTRYPOINT
```

### WORKDIR — Working Directory

```dockerfile
# Set working directory for subsequent instructions
WORKDIR /app

# Creates directory if it doesn't exist
WORKDIR /app/src/components

# Can use environment variables
WORKDIR $APP_HOME
```

### ENV — Environment Variables

```dockerfile
# Set environment variables (persist in running container)
ENV NODE_ENV=production
ENV APP_PORT=3000
ENV DB_HOST=localhost DB_PORT=5432

# Available during build AND runtime
```

### ARG — Build Arguments

```dockerfile
# Build-time only variables (NOT in running container)
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}-alpine

ARG APP_VERSION=1.0.0
LABEL version=${APP_VERSION}

# Pass at build time:
# docker build --build-arg NODE_VERSION=18 .
```

### EXPOSE — Document Ports

```dockerfile
# Documents which ports the container listens on
EXPOSE 80
EXPOSE 443
EXPOSE 3000/tcp
EXPOSE 5000/udp

# NOTE: EXPOSE does NOT publish ports!
# You still need -p flag: docker run -p 80:80
```

### VOLUME — Declare Mount Points

```dockerfile
# Creates a mount point for persistent data
VOLUME /data
VOLUME ["/data", "/logs"]

# Docker creates anonymous volume at runtime
# Better to use named volumes with -v flag
```

### USER — Set Runtime User

```dockerfile
# Switch to non-root user
RUN addgroup --system app && adduser --system --ingroup app app
USER app

# Or use numeric UID:GID
USER 1000:1000
```

### LABEL — Metadata

```dockerfile
LABEL maintainer="team@example.com"
LABEL version="1.0"
LABEL description="My application"
LABEL org.opencontainers.image.source="https://github.com/org/repo"
```

### HEALTHCHECK — Container Health

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

HEALTHCHECK NONE   # Disable health check
```

### STOPSIGNAL — Shutdown Signal

```dockerfile
STOPSIGNAL SIGTERM    # Default
STOPSIGNAL SIGQUIT    # Graceful shutdown for nginx
```

### SHELL — Change Default Shell

```dockerfile
SHELL ["/bin/bash", "-c"]   # Use bash instead of sh
RUN echo "Now using bash"
```

---

## 1.3 Complete Dockerfile Example

```dockerfile
# ---- Build Stage ----
FROM node:20-alpine AS builder

# Metadata
LABEL maintainer="dev@company.com"
LABEL org.opencontainers.image.source="https://github.com/company/app"

# Set working directory
WORKDIR /app

# Install dependencies (cached layer)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Copy source and build
COPY . .
RUN npm run build

# ---- Production Stage ----
FROM node:20-alpine

# Create non-root user
RUN addgroup --system app && adduser --system --ingroup app app

# Set working directory
WORKDIR /app

# Copy built artifacts from builder
COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
COPY --from=builder --chown=app:app /app/package.json ./

# Environment
ENV NODE_ENV=production
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:3000/health || exit 1

# Run as non-root
USER app

# Start application
CMD ["node", "dist/server.js"]
```

---

## 1.4 Instruction Quick Reference

| Instruction | Purpose | Creates Layer? |
|-------------|---------|---------------|
| `FROM` | Set base image | Yes |
| `RUN` | Execute command | Yes |
| `COPY` | Copy files from context | Yes |
| `ADD` | Copy + extract/download | Yes |
| `CMD` | Default run command | No (metadata) |
| `ENTRYPOINT` | Container executable | No (metadata) |
| `WORKDIR` | Set working directory | Yes (if creates dir) |
| `ENV` | Set environment variable | No (metadata) |
| `ARG` | Build-time variable | No |
| `EXPOSE` | Document port | No (metadata) |
| `VOLUME` | Declare mount point | No (metadata) |
| `USER` | Set runtime user | No (metadata) |
| `LABEL` | Add metadata | No (metadata) |
| `HEALTHCHECK` | Container health check | No (metadata) |
| `STOPSIGNAL` | Shutdown signal | No (metadata) |
| `SHELL` | Default shell | No (metadata) |

---

## Summary Table

| Category | Instructions | Purpose |
|----------|-------------|---------|
| **Base** | FROM | Starting image |
| **Execute** | RUN | Install packages, build |
| **Copy** | COPY, ADD | Add files to image |
| **Runtime** | CMD, ENTRYPOINT | What the container runs |
| **Config** | ENV, ARG, WORKDIR, USER | Configure environment |
| **Documentation** | EXPOSE, LABEL, VOLUME | Metadata and documentation |
| **Health** | HEALTHCHECK, STOPSIGNAL | Monitoring and shutdown |

---

## Quick Revision Questions

1. **What is the difference between CMD and ENTRYPOINT?**
   - CMD sets the default command that can be overridden at runtime. ENTRYPOINT sets the container's main executable — runtime arguments are appended to it

2. **Why should you prefer COPY over ADD?**
   - COPY is more transparent and predictable. ADD has implicit behaviors (URL download, tar extraction) that can be surprising. Use ADD only when tar extraction is needed

3. **What is the difference between ENV and ARG?**
   - ENV sets variables available during build AND in the running container. ARG sets variables available only during build time

4. **Does EXPOSE actually publish a port?**
   - No. EXPOSE is documentation only. You must use `-p` flag with `docker run` to actually publish and map ports

5. **Which instructions create filesystem layers?**
   - FROM, RUN, COPY, and ADD create layers. Other instructions (CMD, ENV, EXPOSE, etc.) only add metadata

6. **How do CMD and ENTRYPOINT work together?**
   - When both are specified, CMD provides default arguments to ENTRYPOINT. Example: `ENTRYPOINT ["python3"]` + `CMD ["app.py"]` runs `python3 app.py`

---

[← Previous Unit: Image Inspection](../03-Docker-Images/06-image-inspection.md) | [Next: Build Context →](02-build-context.md)
