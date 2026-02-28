# Chapter 6: Dockerfile Best Practices

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

Writing an efficient, secure, and maintainable Dockerfile requires following established best practices. This chapter covers the essential rules and patterns that professional Docker users follow.

---

## 6.1 Use Official and Verified Base Images

```dockerfile
# GOOD: Official images (maintained, patched, optimized)
FROM node:20-alpine
FROM python:3.11-slim
FROM nginx:1.25-alpine

# BAD: Random unverified images
FROM randomuser/node-custom:latest
FROM some-org/mystery-python:v1
```

```
  Image Trust Hierarchy:
  
  +--------------------------+  Highest Trust
  | Docker Official Images   |  (library/node, library/python)
  +--------------------------+
  | Verified Publisher        |  (bitnami, chainguard)
  +--------------------------+
  | Docker Sponsored OSS     |  (community maintained)
  +--------------------------+
  | User Images              |  Lowest Trust
  +--------------------------+
```

---

## 6.2 Use Specific Tags, Never `latest`

```dockerfile
# BAD: Unpredictable, can change anytime
FROM node:latest
FROM python:latest

# GOOD: Pinned version
FROM node:20.11-alpine3.19
FROM python:3.11.7-slim-bookworm

# BEST: Pin by digest (immutable)
FROM node:20.11-alpine3.19@sha256:abcd1234...
```

---

## 6.3 Minimize Layers

```dockerfile
# BAD: Each RUN creates a layer
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get install -y vim
RUN rm -rf /var/lib/apt/lists/*

# GOOD: Chain commands in single RUN
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        git \
        vim && \
    rm -rf /var/lib/apt/lists/*
```

```
  BAD (5 layers):              GOOD (1 layer):
  
  +-------------------+       +-------------------+
  | Layer 5: rm cache |       | Layer 1:          |
  +-------------------+       |   update +        |
  | Layer 4: vim      |       |   install +       |
  +-------------------+       |   clean up        |
  | Layer 3: git      |       +-------------------+
  +-------------------+       
  | Layer 2: curl     |       Files deleted in same
  +-------------------+       layer = not in image
  | Layer 1: update   |       
  +-------------------+       
  
  Note: rm in layer 5 does NOT
  reduce image size! The files
  still exist in layer 1.
```

---

## 6.4 Order Instructions for Cache Efficiency

```dockerfile
# GOOD: Least changing → Most changing

FROM node:20-alpine

# 1. System deps (rarely change)
RUN apk add --no-cache tini

# 2. App configuration (infrequent changes)
WORKDIR /app
ENV NODE_ENV=production

# 3. Dependencies (change when deps update)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# 4. Source code (changes most frequently)
COPY . .

# 5. Runtime config (metadata)
EXPOSE 3000
USER node
CMD ["tini", "--", "node", "server.js"]
```

```
  Layer Cache Flow:
  
  +-----------------------+
  | FROM node:20-alpine   |  Cached ✓ (base rarely changes)
  +-----------------------+
  | RUN apk add tini      |  Cached ✓ (system deps stable)
  +-----------------------+
  | COPY package*.json     |  Cached ✓ (deps unchanged)
  | RUN npm ci             |  Cached ✓ (same deps = cache hit)
  +-----------------------+
  | COPY . .               |  REBUILT (source changed)
  +-----------------------+
  
  Only the last layer rebuilds when code changes!
  npm ci does NOT run again = FAST builds
```

---

## 6.5 Run as Non-Root User

```dockerfile
# BAD: Running as root (default)
FROM node:20-alpine
COPY . /app
CMD ["node", "/app/server.js"]
# Runs as root — security risk!

# GOOD: Create and switch to non-root user
FROM node:20-alpine
RUN addgroup --system app && \
    adduser --system --ingroup app app
WORKDIR /app
COPY --chown=app:app . .
USER app
CMD ["node", "server.js"]
```

```
  Root (UID 0):                Non-Root (UID 1000):
  
  +------------------+        +------------------+
  | Full system      |        | Limited          |
  | access           |        | access           |
  | Can modify OS    |        | Cannot modify OS |
  | Container escape |        | Blast radius     |
  | = host root!     |        | minimized        |
  +------------------+        +------------------+
```

---

## 6.6 Use .dockerignore

```
# Always create .dockerignore
.git
node_modules
*.log
.env
.env.*
*.md
!README.md
coverage/
.vscode/
.idea/
```

---

## 6.7 Clean Up in the Same Layer

```dockerfile
# BAD: Cleanup in separate layer (files still in image)
RUN apt-get update && apt-get install -y build-essential
RUN make && make install
RUN apt-get purge -y build-essential && rm -rf /var/lib/apt/lists/*

# GOOD: Install, build, and clean in ONE layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential && \
    make && make install && \
    apt-get purge -y build-essential && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*
```

---

## 6.8 Use COPY Instead of ADD

```dockerfile
# GOOD: Explicit, predictable
COPY requirements.txt /app/
COPY src/ /app/src/

# ADD only when you need tar extraction
ADD archive.tar.gz /app/

# AVOID: ADD for URL downloads
# BAD:
ADD https://example.com/file /app/
# GOOD:
RUN curl -fsSL https://example.com/file -o /app/file
```

---

## 6.9 Use HEALTHCHECK

```dockerfile
# Provide health check so orchestrators know container status
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# For containers without curl
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:3000/health || exit 1

# For non-HTTP services
HEALTHCHECK --interval=30s --timeout=3s \
  CMD pg_isready -U postgres || exit 1
```

---

## 6.10 Use Exec Form for CMD and ENTRYPOINT

```dockerfile
# GOOD: Exec form — process gets PID 1, receives signals properly
CMD ["node", "server.js"]
ENTRYPOINT ["python3", "app.py"]

# BAD: Shell form — runs through /bin/sh -c
CMD node server.js
# Actually runs: /bin/sh -c "node server.js"
# Node is NOT PID 1, won't receive SIGTERM properly
```

```
  Exec Form:                    Shell Form:
  
  PID 1: node server.js        PID 1: /bin/sh -c "node ..."
                                  └── PID 2: node server.js
  
  SIGTERM ──> node (received!)  SIGTERM ──> sh (NOT node!)
  Graceful shutdown ✓           Ungraceful kill after timeout ✗
```

---

## 6.11 Use Multi-Stage Builds

```dockerfile
# Build stage
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage — only runtime deps
FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./
USER node
CMD ["node", "dist/index.js"]
```

---

## 6.12 Add Metadata with LABELS

```dockerfile
LABEL org.opencontainers.image.title="My Application"
LABEL org.opencontainers.image.description="Production web server"
LABEL org.opencontainers.image.version="2.1.0"
LABEL org.opencontainers.image.vendor="My Company"
LABEL org.opencontainers.image.source="https://github.com/org/repo"
LABEL org.opencontainers.image.created="2024-01-15T10:30:00Z"
```

---

## 6.13 Best Practices Checklist

```
  Dockerfile Best Practices Checklist:
  
  BASE IMAGE
  [ ] Use official/verified base image
  [ ] Use specific tag (not :latest)
  [ ] Use minimal variant (-alpine, -slim)
  
  SECURITY
  [ ] Run as non-root USER
  [ ] No secrets in ARG or ENV
  [ ] Use BuildKit secret mounts
  [ ] .dockerignore excludes sensitive files
  [ ] Scan image for vulnerabilities
  
  EFFICIENCY
  [ ] Order: least → most changing
  [ ] Combine RUN commands
  [ ] Clean up in same layer
  [ ] Use .dockerignore
  [ ] Multi-stage builds
  [ ] --no-install-recommends
  
  RELIABILITY
  [ ] HEALTHCHECK defined
  [ ] Exec form for CMD/ENTRYPOINT
  [ ] Use tini or dumb-init for PID 1
  [ ] OCI LABELs for metadata
  
  MAINTAINABILITY
  [ ] Comments explaining non-obvious steps
  [ ] Consistent formatting
  [ ] Pin dependency versions
```

---

## 6.14 Complete Best-Practice Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

# ==================== Build Stage ====================
FROM node:20-alpine AS builder

# System dependencies
RUN apk add --no-cache python3 make g++

WORKDIR /app

# Dependencies (cached layer)
COPY package.json package-lock.json ./
RUN npm ci

# Build application
COPY . .
RUN npm run build && npm prune --production

# ==================== Production Stage ====================
FROM node:20-alpine

# Metadata
LABEL org.opencontainers.image.title="My App" \
      org.opencontainers.image.version="2.0.0" \
      org.opencontainers.image.source="https://github.com/org/app"

# Security: tini for proper PID 1 signal handling
RUN apk add --no-cache tini

# Security: non-root user
RUN addgroup --system app && adduser --system --ingroup app app

WORKDIR /app

# Copy only production artifacts
COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
COPY --from=builder --chown=app:app /app/package.json ./

# Runtime configuration
ENV NODE_ENV=production
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -q --spider http://localhost:3000/health || exit 1

# Run as non-root
USER app

# Use tini as PID 1 + exec form
ENTRYPOINT ["tini", "--"]
CMD ["node", "dist/server.js"]
```

---

## Summary Table

| Practice | Why | Example |
|----------|-----|---------|
| **Official base images** | Trusted, maintained, patched | `FROM node:20-alpine` |
| **Specific tags** | Reproducible builds | `FROM python:3.11-slim` |
| **Minimize layers** | Smaller image, faster builds | Chain RUN commands |
| **Cache-friendly order** | Faster rebuilds | Deps before source code |
| **Non-root USER** | Security hardening | `USER app` |
| **Exec form** | Proper signal handling | `CMD ["node", "app.js"]` |
| **HEALTHCHECK** | Container monitoring | `HEALTHCHECK CMD ...` |
| **Multi-stage** | Smaller final image | `COPY --from=builder` |
| **Clean in same layer** | Remove temp files effectively | `RUN install && clean` |
| **.dockerignore** | Fast builds, no leaked secrets | Exclude .git, node_modules |

---

## Quick Revision Questions

1. **Why should you never use the `latest` tag?**
   - The `latest` tag is mutable and can change without notice, leading to unpredictable and irreproducible builds. Always pin specific versions

2. **Why is instruction order important in a Dockerfile?**
   - Docker caches layers sequentially. Putting frequently changing instructions (like COPY source) last means earlier layers (like dependency installation) can be cached, dramatically speeding up builds

3. **Why should containers run as non-root?**
   - Running as root is a security risk. If a container is compromised, the attacker gains root privileges. Container escapes with root-in-container can lead to root-on-host

4. **What's wrong with `CMD node server.js` (shell form)?**
   - Shell form wraps the command in `/bin/sh -c`, so the application is NOT PID 1 and won't receive SIGTERM signals properly, preventing graceful shutdown

5. **Why must cleanup happen in the same RUN layer as installation?**
   - Docker layers are additive. Files deleted in a later layer still exist in earlier layers and contribute to image size. Only cleanup in the same layer truly removes files

6. **What is tini and why is it used?**
   - Tini is a minimal init system that runs as PID 1, properly forwards signals, and reaps zombie processes — solving issues that arise when application processes run directly as PID 1

---

[← Previous: Build Args and ENV](05-build-args-and-env.md) | [Next: Image Optimization →](07-image-optimization.md)
