# Chapter 7: Image Optimization

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

Optimizing Docker images reduces storage costs, speeds up deployments, minimizes attack surface, and improves container startup time. This chapter covers techniques to make images as small, fast, and secure as possible.

---

## 7.1 Why Image Size Matters

```
  1 GB Image                          50 MB Image
  
  Pull time:    45 seconds            Pull time:    3 seconds
  Push time:    60 seconds            Push time:    5 seconds
  Storage:      1 GB × N replicas     Storage:      50 MB × N replicas
  Startup:      Slow (layer extract)  Startup:      Fast
  Attack surface: Large               Attack surface: Minimal
  CVE scan:     100+ vulnerabilities  CVE scan:     5 vulnerabilities
  
  In a cluster with 100 nodes pulling the image:
  1 GB × 100  = 100 GB network traffic
  50 MB × 100 = 5 GB network traffic
```

---

## 7.2 Choose the Right Base Image

```
  Base Image Sizes (approximate):
  
  +-----------------------------------+---------+
  | Base Image                        | Size    |
  +-----------------------------------+---------+
  | ubuntu:22.04                      | 77 MB   |
  | debian:bookworm                   | 139 MB  |
  | debian:bookworm-slim              | 74 MB   |
  | alpine:3.19                       | 7 MB    |
  | node:20                           | 1.1 GB  |
  | node:20-slim                      | 200 MB  |
  | node:20-alpine                    | 130 MB  |
  | python:3.11                       | 1 GB    |
  | python:3.11-slim                  | 130 MB  |
  | python:3.11-alpine                | 50 MB   |
  | golang:1.21                       | 800 MB  |
  | gcr.io/distroless/static          | 2 MB    |
  | scratch                           | 0 MB    |
  +-----------------------------------+---------+
  
  Selection Guide:
  
  Need shell/debug? ──> alpine (7MB) or slim
  Static binary?    ──> scratch (0MB) or distroless (2MB)
  Need glibc?       ──> slim variant
  Need apt/dpkg?    ──> debian-slim
  Need everything?  ──> full image (dev only!)
```

### Alpine Considerations

```dockerfile
# Alpine uses musl libc instead of glibc
# Most apps work fine, but some need glibc

# Alpine
FROM python:3.11-alpine
# Pros: Tiny (50MB), minimal packages
# Cons: Some Python packages need compilation,
#        musl compatibility issues with some C libraries,
#        longer build times for compiled packages

# Slim (Debian-based)
FROM python:3.11-slim
# Pros: glibc (better compatibility), pre-built wheels work
# Cons: Slightly larger (130MB)
```

---

## 7.3 Multi-Stage Build for Size Reduction

```dockerfile
# ---- Build (large, has all tools) ----
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o server .

# ---- Runtime (tiny, binary only) ----
FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]

# Result: ~5-10 MB instead of ~800 MB
```

```
  Size Reduction Pipeline:
  
  golang:1.21   ──build──>  Binary     ──copy──>   scratch
  (800 MB)                  (15 MB)                 (15 MB total)
  
  Reduction: 98% smaller!
```

---

## 7.4 Minimize Layer Count and Size

### Combine RUN Instructions

```dockerfile
# BAD: 5 layers, no cleanup
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y pip
RUN pip install flask
RUN apt-get clean

# GOOD: 1 layer, with cleanup
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3 \
        python3-pip && \
    pip install --no-cache-dir flask && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Remove Unnecessary Files

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential && \
    # Build your app
    make && make install && \
    # Remove build tools and cache
    apt-get purge -y --auto-remove build-essential && \
    rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/doc/* \
        /usr/share/man/*
```

---

## 7.5 Optimize Package Installation

### APT (Debian/Ubuntu)

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        package1 \
        package2 && \
    rm -rf /var/lib/apt/lists/*
#          ^^^^^^^^^^^^^^^^^^^^^^^^
#          Removes apt cache (~30-50MB)
```

### APK (Alpine)

```dockerfile
RUN apk add --no-cache \
    package1 \
    package2
#   ^^^^^^^^^^
#   --no-cache avoids storing index locally
```

### PIP (Python)

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
#               ^^^^^^^^^^^^^^
#               Don't cache downloaded packages

# Or with BuildKit cache mount (faster rebuilds)
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

### NPM (Node.js)

```dockerfile
# Use npm ci instead of npm install (clean, deterministic)
RUN npm ci --only=production
#          ^^^^^^^^^^^^^^^^
#          Skip devDependencies

# Clean npm cache
RUN npm ci --only=production && npm cache clean --force
```

---

## 7.6 Optimize COPY Instructions

```dockerfile
# BAD: Copy everything, then install
COPY . /app
RUN npm ci

# GOOD: Copy deps file first, then source
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
# Only source code changes trigger re-copy,
# npm ci is cached if deps unchanged
```

---

## 7.7 Use Static Linking and Compression

### Go: Static Build with Link Flags

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-s -w" \
    -o server .
# -s: Strip symbol table
# -w: Strip debug info
# Reduces binary by ~30%
```

### UPX Compression

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o server .

# Compress with UPX
RUN apt-get update && apt-get install -y upx && \
    upx --best --lzma server
# Further reduces binary by 50-70%

FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

```
  Binary Size Reduction:
  
  Normal build:     25 MB
  + strip (-s -w):  17 MB  (-32%)
  + UPX compress:    5 MB  (-80%)
```

---

## 7.8 Reduce Image with Distroless

```
  Image Type Comparison:
  
  Ubuntu (77MB):
  +-----------------+
  | Shell (bash)    |
  | Package manager |
  | System utils    |
  | Your app        |
  +-----------------+
  
  Alpine (7MB):
  +----------+
  | Shell (sh)|
  | apk       |
  | Your app  |
  +----------+
  
  Distroless (2MB):
  +----------+
  | Runtime  |
  | Your app |
  +----------+
  No shell, no package manager, no utils!
  
  Scratch (0MB):
  +----------+
  | Your app |
  +----------+
  Nothing at all — just your binary!
```

```dockerfile
# Distroless for Java
FROM eclipse-temurin:21-jdk AS builder
WORKDIR /app
COPY . .
RUN ./gradlew build

FROM gcr.io/distroless/java21-debian12
COPY --from=builder /app/build/libs/app.jar /app.jar
CMD ["app.jar"]

# Distroless for Node.js
FROM node:20 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM gcr.io/distroless/nodejs20-debian12
COPY --from=builder /app/dist /app
CMD ["/app/server.js"]
```

---

## 7.9 Docker Slim (Automated Optimization)

```bash
# DockerSlim automatically optimizes images
# Install: https://github.com/slimtoolkit/slim

# Analyze and optimize
slim build --target myapp:latest

# Result: Dramatically smaller image
# myapp:latest       → 350 MB
# myapp.slim:latest  → 35 MB
```

---

## 7.10 Image Analysis Tools

```bash
# Docker built-in
docker images                    # List image sizes
docker history myimage           # Layer breakdown
docker inspect myimage           # Full metadata

# Dive — Interactive layer explorer
dive myimage:latest
# Shows: Layer contents, wasted space, efficiency score

# Trivy — Security + size analysis
trivy image myimage:latest

# Hadolint — Dockerfile linter
hadolint Dockerfile
# Catches antipatterns and suggests improvements
```

```
  Dive Output Example:
  
  ┌──────────────────────────────────────────┐
  │ Layer Details                            │
  ├──────────────────────────────────────────┤
  │  47 MB  FROM node:20-alpine             │
  │  15 MB  RUN npm ci                      │
  │  22 MB  COPY . .          ← investigate │
  │   0 B   CMD ["node"...]                 │
  ├──────────────────────────────────────────┤
  │ Image efficiency: 92%                    │
  │ Wasted space: 4.2 MB                    │
  └──────────────────────────────────────────┘
```

---

## 7.11 Optimization Checklist

```
  Image Optimization Checklist:
  
  +----+---------------------------------------+--------+
  | #  | Technique                             | Savings|
  +----+---------------------------------------+--------+
  | 1  | Use alpine/slim base image            | 50-90% |
  | 2  | Multi-stage build                     | 50-98% |
  | 3  | Remove package manager cache          | 30-50MB|
  | 4  | --no-install-recommends               | 20-40% |
  | 5  | Combine && clean in same RUN          | Varies |
  | 6  | Use .dockerignore                     | Varies |
  | 7  | Strip debug symbols (-ldflags -s -w)  | ~30%   |
  | 8  | UPX compression                       | 50-70% |
  | 9  | Only production dependencies          | 30-60% |
  | 10 | distroless / scratch base             | 80-99% |
  +----+---------------------------------------+--------+
```

---

## 7.12 Real-World Optimization Example

```dockerfile
# BEFORE: 1.1 GB
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]

# AFTER: 85 MB (92% reduction)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build && npm prune --production

FROM node:20-alpine
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=builder --chown=app:app /app/dist ./dist
COPY --from=builder --chown=app:app /app/node_modules ./node_modules
COPY --from=builder --chown=app:app /app/package.json ./
USER app
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

```
  Optimization Steps Applied:
  
  1.1 GB ─── node:20         → node:20-alpine ──── 130 MB
  130 MB ─── all deps        → production only ─── 100 MB
  100 MB ─── single stage    → multi-stage ──────── 85 MB
   85 MB ─── + non-root user, healthcheck ────────  85 MB
  
  Total reduction: 92%
```

---

## Summary Table

| Technique | Impact | Complexity |
|-----------|--------|------------|
| **Alpine/slim base** | High (50-90% reduction) | Low |
| **Multi-stage build** | Very High (50-98%) | Medium |
| **Remove caches** | Medium (30-50 MB) | Low |
| **--no-install-recommends** | Medium (20-40%) | Low |
| **Combine RUN layers** | Medium | Low |
| **.dockerignore** | Variable | Low |
| **Strip symbols** | Medium (30%) | Low |
| **UPX compression** | High (50-70%) | Low |
| **Distroless/scratch** | Very High (80-99%) | Medium |
| **DockerSlim** | High (automated) | Low |

---

## Quick Revision Questions

1. **Why is image size important?**
   - Smaller images mean faster pulls/pushes, less storage, quicker deployments, less network bandwidth, and reduced attack surface with fewer vulnerabilities

2. **What is the difference between alpine and slim base images?**
   - Alpine uses musl libc and is ~7MB; slim is Debian-based with glibc and is ~75MB. Alpine is smaller but may have compatibility issues with some C libraries

3. **How do multi-stage builds reduce image size?**
   - Build tools, compilers, source code, and development dependencies are left in the build stage. Only the final compiled artifacts are copied to the minimal production stage

4. **What is `FROM scratch` used for?**
   - It creates an image with absolutely nothing — no OS, no shell, no utilities. Only suitable for statically compiled binaries that have no external dependencies

5. **Why should package manager caches be removed in the same RUN layer?**
   - Docker layers are additive. Removing files in a separate RUN creates a new layer that marks files as deleted, but the files still exist in the previous layer, consuming space

6. **What tools can help analyze and optimize Docker images?**
   - `docker history` shows layer sizes, `dive` provides interactive layer analysis, `hadolint` lints Dockerfiles for anti-patterns, and DockerSlim can automatically optimize images

---

[← Previous: Dockerfile Best Practices](06-dockerfile-best-practices.md) | [Next Unit: Container Lifecycle →](../05-Docker-Containers/01-container-lifecycle.md)
