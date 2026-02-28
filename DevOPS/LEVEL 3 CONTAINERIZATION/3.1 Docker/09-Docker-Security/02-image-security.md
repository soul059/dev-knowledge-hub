# Chapter 2: Image Security

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

Image security is the foundation of container security. A compromised or vulnerable image undermines all other security measures. This chapter covers trusted image sources, vulnerability scanning, minimal images, and supply chain integrity.

---

## 2.1 Trusted Image Sources

```
  Image Trust Hierarchy:
  
  Most Trusted ─────────────────> Least Trusted
  
  Official        Verified        Community      Random
  Images          Publishers      Images         Images
  +----------+   +-----------+   +-----------+  +--------+
  | nginx    |   | bitnami/  |   | user123/  |  | ???    |
  | postgres |   | datadog/  |   | myapp     |  | No info|
  | node     |   | hashicorp/|   | untested  |  |        |
  +----------+   +-----------+   +-----------+  +--------+
  Docker-curated  Company-backed  Variable       DANGER
  Regular updates Verified org    quality        
  
  Rule: ALWAYS prefer official or verified images
```

```bash
# Check image source on Docker Hub
# Official: "Docker Official Image" badge
# Verified: "Verified Publisher" badge

# Use specific versions, not latest
FROM node:20.10-alpine    # Good: specific version
FROM node:latest          # Bad: unpredictable
FROM node                 # Bad: same as latest
```

---

## 2.2 Vulnerability Scanning

```bash
# Docker Scout (built into Docker Desktop)
docker scout cves myimage:latest
docker scout cves --format sarif myimage:latest
docker scout recommendations myimage:latest

# Trivy — comprehensive open-source scanner
# Scan local image
trivy image myimage:latest

# Scan and fail on critical vulnerabilities
trivy image --severity CRITICAL --exit-code 1 myimage:latest

# Scan Dockerfile
trivy config Dockerfile

# Grype — fast vulnerability scanner
grype myimage:latest
grype myimage:latest --only-fixed  # Show only fixable CVEs
```

```
  CI/CD Scanning Pipeline:
  
  Code Commit
       |
       v
  Build Image
       |
       v
  +---------+     +----------+
  | Scan    |---->| CRITICAL |---> FAIL BUILD ✗
  | Image   |     | found?   |
  |         |     +----------+
  | trivy / |         |
  | scout / |         No
  | grype   |         |
  +---------+         v
                 Push to Registry ✓
                      |
                      v
                 Deploy
```

---

## 2.3 Minimal Base Images

```
  Base Image Size & Attack Surface:
  
  Image              Size      Packages    CVEs
  +-----------------+--------+-----------+--------+
  | ubuntu:22.04    | ~77MB  | ~100+     | Many   |
  | debian:bookworm | ~120MB | ~100+     | Many   |
  | alpine:3.19     | ~7MB   | ~15       | Few    |
  | distroless      | ~2MB   | Minimal   | Fewer  |
  | scratch         | 0MB    | None      | None   |
  +-----------------+--------+-----------+--------+
  
  Smaller image = Fewer packages = Fewer vulnerabilities
  
  Less is more!
```

```dockerfile
# Option 1: Alpine-based (small + has package manager)
FROM node:20-alpine
# Good for: Most applications

# Option 2: Distroless (no shell, no package manager)
FROM gcr.io/distroless/nodejs20-debian12
# Good for: Production, maximum security

# Option 3: Scratch (empty image)
FROM scratch
COPY myapp /myapp
# Good for: Statically compiled binaries (Go, Rust)
```

```bash
# Compare image sizes
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h
```

---

## 2.4 Secrets in Images

```
  Common Secret Leaks:
  
  ✗ BAD: Secrets baked into image
  
  Dockerfile:
  ENV API_KEY=sk-1234567890     ← Visible in any layer!
  COPY .env /app/.env           ← Included in image!
  RUN git clone https://TOKEN@github.com/repo  ← In layer!
  
  Even if you delete the file later:
  RUN rm /app/.env              ← Still in previous layer!
  
  Anyone with the image can:
  docker history myimage        ← See ENV commands
  docker save myimage | tar x  ← Extract all layers
```

```dockerfile
# ✓ GOOD: Use build secrets (BuildKit)
# syntax=docker/dockerfile:1
FROM node:20-alpine

# Mount secret at build time (not stored in layer)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm install

# Use ARG for non-sensitive build values only
ARG APP_VERSION
```

```bash
# Pass secrets at build time
docker build --secret id=npmrc,src=.npmrc .

# NEVER do this
docker build --build-arg API_KEY=secret .  # Visible in history!
```

---

## 2.5 Image Hardening

```dockerfile
# Production-hardened Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
RUN npm run build

FROM node:20-alpine
# 1. Create non-root user
RUN addgroup -g 1001 appgroup && \
    adduser -u 1001 -G appgroup -D appuser

# 2. Remove unnecessary packages
RUN apk --no-cache add dumb-init && \
    rm -rf /var/cache/apk/*

WORKDIR /app

# 3. Copy only what's needed
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules

# 4. Use non-root user
USER appuser

# 5. Read-only file system compatible
ENV NODE_ENV=production

# 6. Use init system for proper signal handling
ENTRYPOINT ["dumb-init", "--"]

# 7. Run with exec form
CMD ["node", "dist/server.js"]
```

---

## 2.6 .dockerignore for Security

```bash
# .dockerignore — prevent sensitive files from entering build context
.git
.env
.env.*
*.key
*.pem
*.p12
secrets/
credentials/
*.secret
docker-compose*.yml
.ssh/
id_rsa*
```

---

## Summary Table

| Practice | Why | How |
|----------|-----|-----|
| **Official images** | Trusted, maintained | Use Docker Official Images |
| **Specific tags** | Reproducible builds | `node:20.10-alpine` not `:latest` |
| **Minimal base** | Fewer CVEs | Alpine, distroless, scratch |
| **Scan images** | Find vulnerabilities | Trivy, Scout, Grype in CI/CD |
| **No secrets in images** | Prevent leaks | BuildKit secrets, not ENV/ARG |
| **Non-root user** | Limit damage | USER instruction in Dockerfile |
| **.dockerignore** | Keep secrets out of build | Exclude .env, keys, .git |
| **Multi-stage builds** | Remove build tools | Only copy runtime artifacts |
| **Image signing** | Verify authenticity | Docker Content Trust (DCT) |

---

## Quick Revision Questions

1. **Why are smaller base images more secure?**
   - Fewer installed packages means fewer potential vulnerabilities. Alpine has ~15 packages vs ubuntu's 100+, dramatically reducing the attack surface

2. **How can secrets end up in Docker images?**
   - Through ENV instructions, COPY of .env files, build-arg values, and RUN commands with tokens. Even deleted files persist in earlier layers

3. **How do you safely pass secrets during docker build?**
   - Use BuildKit secret mounts: `RUN --mount=type=secret,id=key,target=/run/secrets/key`. The secret is never stored in any image layer

4. **When should you scan images for vulnerabilities?**
   - In CI/CD before pushing to registry, periodically for deployed images, and whenever base images are updated

5. **What is the difference between Alpine, distroless, and scratch?**
   - Alpine: minimal Linux with package manager (~7MB). Distroless: no shell or package manager (~2MB). Scratch: empty image (0MB, for static binaries only)

---

[← Previous: Security Overview](01-security-overview.md) | [Next: Runtime Security →](03-runtime-security.md)
