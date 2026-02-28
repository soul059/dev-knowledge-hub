# Chapter 3: Multi-Stage Builds

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

**Multi-stage builds** allow you to use multiple `FROM` statements in a single Dockerfile. Each `FROM` starts a new build stage, and you can selectively copy artifacts from one stage to another. This dramatically reduces final image size by leaving build tools, dependencies, and source code behind.

---

## 3.1 The Problem Multi-Stage Builds Solve

```
  WITHOUT Multi-Stage:
  
  +------------------------------------------+
  |  Final Image (800MB)                      |
  |                                           |
  |  +-----------+  +-----------+  +-------+  |
  |  | Go        |  | Source    |  | Built |  |
  |  | Compiler  |  | Code     |  | Binary|  |
  |  | (500MB)   |  | (50MB)   |  | (15MB)|  |
  |  +-----------+  +-----------+  +-------+  |
  |                                           |
  +------------------------------------------+
  
  WITH Multi-Stage:
  
  Stage 1 (builder):            Stage 2 (final):
  +---------------------+      +-------------+
  | Go Compiler         |      | Alpine      |
  | Source Code          | COPY | Built       |
  | Dependencies         | ---> | Binary      |
  | Built Binary         |      | (20MB)      |
  | (800MB total)        |      +-------------+
  +---------------------+
```

---

## 3.2 Basic Multi-Stage Build

```dockerfile
# ======= Stage 1: Build =======
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# ======= Stage 2: Runtime =======
FROM alpine:3.19
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=builder /app/myapp .
EXPOSE 8080
CMD ["./myapp"]
```

```bash
# Build produces only the final stage image
docker build -t myapp .

# Image size comparison:
# Without multi-stage: ~800MB (golang base + source + binary)
# With multi-stage:    ~20MB  (alpine + binary only)
```

---

## 3.3 Referencing Stages

```dockerfile
# By name (RECOMMENDED)
FROM node:20 AS builder
# ...
COPY --from=builder /app/dist ./dist

# By index (0-based)
FROM node:20
# ...
FROM nginx:alpine
COPY --from=0 /app/dist /usr/share/nginx/html

# From external image (not a stage)
COPY --from=nginx:alpine /etc/nginx/nginx.conf /etc/nginx/
```

---

## 3.4 Common Multi-Stage Patterns

### Pattern 1: Builder Pattern (Compiled Languages)

```dockerfile
# Go application
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o server .

FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

### Pattern 2: Frontend + Backend

```dockerfile
# Stage 1: Build React frontend
FROM node:20-alpine AS frontend
WORKDIR /app
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

# Stage 2: Build Go backend
FROM golang:1.21 AS backend
WORKDIR /app
COPY backend/ .
RUN go build -o server .

# Stage 3: Production
FROM alpine:3.19
WORKDIR /app
COPY --from=backend /app/server .
COPY --from=frontend /app/build ./static
EXPOSE 8080
CMD ["./server"]
```

```
  +----------+     +----------+     +----------+
  | Stage 1  |     | Stage 2  |     | Stage 3  |
  | Frontend |     | Backend  |     | Final    |
  | Build    |     | Build    |     |          |
  | (Node)   |     | (Go)     |     | (Alpine) |
  +----+-----+     +----+-----+     +----+-----+
       |                |                |
       | /app/build     | /app/server    |
       +--------------->+--------------->|
         COPY --from      COPY --from
```

### Pattern 3: Test Stage

```dockerfile
FROM python:3.11 AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

# Test stage
FROM base AS test
RUN pip install pytest
RUN pytest tests/

# Production stage
FROM base AS production
EXPOSE 8000
CMD ["gunicorn", "app:app"]
```

```bash
# Run tests
docker build --target test -t myapp-test .

# Build production
docker build --target production -t myapp .
```

### Pattern 4: Development vs Production

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./

# Development stage
FROM base AS development
RUN npm install           # All dependencies (including devDependencies)
COPY . .
CMD ["npm", "run", "dev"]

# Production stage
FROM base AS production
RUN npm ci --only=production
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

```bash
# Build dev image
docker build --target development -t myapp:dev .

# Build production image
docker build --target production -t myapp:prod .
```

### Pattern 5: Distroless Final Image

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o server .

# Use Google's distroless image (no shell, no package manager)
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /
CMD ["/server"]
```

---

## 3.5 Target-Specific Builds

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM base AS lint
RUN npm run lint

FROM base AS test
COPY . .
RUN npm test

FROM base AS build
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=build /app/dist /usr/share/nginx/html
```

```bash
# Build specific target
docker build --target lint -t myapp:lint .
docker build --target test -t myapp:test .
docker build --target production -t myapp:prod .
```

```
  Build Graph:
  
           +------+
           | base |
           +--+---+
              |
     +--------+--------+--------+
     |        |        |        |
  +--v--+  +-v---+  +-v----+   |
  | lint |  | test|  | build|   |
  +------+  +-----+  +--+---+   |
                        |        |
                    +---v--------v---+
                    |  production    |
                    +----------------+
```

---

## 3.6 BuildKit Parallel Execution

```
  Legacy Builder:                BuildKit:
  
  Stage 1 ──> Stage 2           Stage 1 ──┐
      (sequential)               Stage 2 ──┤ (parallel!)
                                           │
                                 Stage 3 <─┘
  
  BuildKit detects independent stages
  and builds them in parallel
```

```dockerfile
# These two stages run in PARALLEL with BuildKit
FROM node:20 AS frontend
WORKDIR /frontend
COPY frontend/ .
RUN npm ci && npm run build

FROM python:3.11 AS backend
WORKDIR /backend
COPY backend/ .
RUN pip install -r requirements.txt

# This stage waits for both
FROM python:3.11-slim
COPY --from=backend /backend /app
COPY --from=frontend /frontend/dist /app/static
CMD ["python", "/app/main.py"]
```

---

## 3.7 Copying from External Images

```dockerfile
FROM alpine:3.19

# Copy binary from another image (not a build stage)
COPY --from=docker:24-cli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=docker/compose:v2.24.0 /usr/local/bin/docker-compose /usr/local/bin/

# Copy configuration from official images
COPY --from=nginx:alpine /etc/nginx/nginx.conf /etc/nginx/
```

---

## 3.8 Size Comparison

```
  Application: Node.js Web API
  
  +-------------------------------+----------+
  | Build Approach                | Size     |
  +-------------------------------+----------+
  | Single stage (node:20)        | 1.1 GB   |
  | Single stage (node:20-slim)   | 250 MB   |
  | Multi-stage (node:20-alpine)  | 120 MB   |
  | Multi-stage (distroless)      | 80 MB    |
  +-------------------------------+----------+
  
  Application: Go REST API
  
  +-------------------------------+----------+
  | Build Approach                | Size     |
  +-------------------------------+----------+
  | Single stage (golang:1.21)    | 850 MB   |
  | Multi-stage (alpine)          | 15 MB    |
  | Multi-stage (scratch)         | 8 MB     |
  | Multi + UPX compression       | 3 MB     |
  +-------------------------------+----------+
```

---

## Summary Table

| Feature | Description | Example |
|---------|-------------|---------|
| **Multi-Stage** | Multiple FROM in one Dockerfile | `FROM ... AS builder` |
| **Named Stages** | Label stages for reference | `FROM node:20 AS build` |
| **COPY --from** | Copy between stages | `COPY --from=builder /app .` |
| **--target** | Build specific stage only | `docker build --target test .` |
| **Parallel Builds** | BuildKit runs independent stages together | Automatic with BuildKit |
| **External COPY** | Copy from any image | `COPY --from=nginx:alpine ...` |
| **scratch** | Empty base for static binaries | `FROM scratch` |
| **distroless** | Minimal runtime, no shell | `FROM gcr.io/distroless/...` |

---

## Quick Revision Questions

1. **What problem do multi-stage builds solve?**
   - They eliminate the need to include build tools, compilers, source code, and development dependencies in the final image, dramatically reducing image size and attack surface

2. **How do you copy files between build stages?**
   - Using `COPY --from=<stage>` instruction, where stage can be a name (`COPY --from=builder`) or index (`COPY --from=0`)

3. **What is the `--target` flag used for?**
   - It tells Docker to build up to a specific stage and stop, allowing different builds from the same Dockerfile (e.g., `--target test` vs `--target production`)

4. **Can stages run in parallel?**
   - Yes. BuildKit automatically detects independent stages and executes them in parallel, significantly reducing build time

5. **What is `FROM scratch` used for?**
   - `FROM scratch` starts from an empty image, ideal for statically compiled binaries (like Go apps) that don't need an OS or any dependencies

6. **Can you copy from an external image that isn't a build stage?**
   - Yes. `COPY --from=nginx:alpine /path /dest` copies from any published image directly

---

[← Previous: Build Context](02-build-context.md) | [Next: Dockerignore →](04-dockerignore.md)
