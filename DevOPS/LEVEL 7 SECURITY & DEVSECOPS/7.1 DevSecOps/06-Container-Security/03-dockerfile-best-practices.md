# Chapter 6.3: Dockerfile Best Practices

## Overview

A well-written Dockerfile reduces image size, minimizes vulnerabilities, prevents privilege escalation, and avoids leaking secrets. This chapter covers security-focused Dockerfile best practices with examples and linting tools.

---

## Secure Dockerfile Template

```dockerfile
# ====================================================
# SECURE DOCKERFILE TEMPLATE
# ====================================================

# 1. Pin specific version (never use :latest)
FROM python:3.11.7-slim-bookworm AS builder

# 2. Set build arguments (not secrets!)
ARG APP_VERSION=1.0.0

# 3. Install build dependencies in builder stage
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ====================================================
# PRODUCTION STAGE
# ====================================================
FROM python:3.11.7-slim-bookworm

# 4. Add labels for tracking
LABEL maintainer="team@company.com" \
      version="${APP_VERSION}" \
      description="My secure application"

# 5. Update OS packages
RUN apt-get update && \
    apt-get upgrade -y --no-install-recommends && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 6. Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser -d /app -s /sbin/nologin appuser

# 7. Copy only what's needed from builder
COPY --from=builder /install /usr/local
WORKDIR /app
COPY --chown=appuser:appuser . .

# 8. Set proper permissions
RUN chmod -R 550 /app && \
    chmod -R 770 /app/tmp || true

# 9. Remove setuid/setgid binaries
RUN find / -perm /6000 -type f -exec chmod a-s {} + 2>/dev/null || true

# 10. Switch to non-root user
USER appuser

# 11. Expose only necessary ports
EXPOSE 8080

# 12. Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health')" || exit 1

# 13. Set read-only filesystem hint
# (Enforced at runtime with --read-only flag)

# 14. Use exec form for CMD (proper signal handling)
CMD ["python", "-u", "app.py"]
```

---

## Security Anti-Patterns

```
COMMON DOCKERFILE SECURITY MISTAKES:

  BAD: Using latest tag
  ┌────────────────────────────────────────────────────┐
  │ FROM python:latest     ← Non-deterministic!       │
  │ FROM node:20           ← Also non-deterministic    │
  │                                                      │
  │ GOOD:                                                │
  │ FROM python:3.11.7-slim-bookworm                    │
  │ FROM node:20.11.0-slim                               │
  │ FROM python@sha256:abc123...  ← Best (digest pin)  │
  └────────────────────────────────────────────────────┘

  BAD: Running as root
  ┌────────────────────────────────────────────────────┐
  │ # No USER directive = runs as root                  │
  │ CMD ["python", "app.py"]                            │
  │                                                      │
  │ GOOD:                                                │
  │ RUN useradd -r appuser                               │
  │ USER appuser                                         │
  │ CMD ["python", "app.py"]                             │
  └────────────────────────────────────────────────────┘

  BAD: Secrets in Dockerfile
  ┌────────────────────────────────────────────────────┐
  │ ENV DATABASE_PASSWORD=SuperSecret123               │
  │ COPY .env /app/.env                                │
  │ ARG API_KEY=sk-live-abc123                         │
  │                                                      │
  │ GOOD:                                                │
  │ # Pass secrets at runtime via environment            │
  │ # or use Docker BuildKit secrets                     │
  │ RUN --mount=type=secret,id=db_pass \                │
  │     cat /run/secrets/db_pass > /dev/null             │
  └────────────────────────────────────────────────────┘

  BAD: Installing unnecessary packages
  ┌────────────────────────────────────────────────────┐
  │ RUN apt-get install -y vim curl wget ssh telnet    │
  │                                                      │
  │ GOOD:                                                │
  │ RUN apt-get install -y --no-install-recommends \    │
  │     ca-certificates                                  │
  └────────────────────────────────────────────────────┘
```

---

## Multi-Stage Builds

```dockerfile
# MULTI-STAGE BUILD — Node.js Example

# Stage 1: Build
FROM node:20.11.0-slim AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20.11.0-slim
WORKDIR /app

# Non-root user
RUN groupadd -r app && useradd -r -g app app

# Copy only production artifacts
COPY --from=build --chown=app:app /app/dist ./dist
COPY --from=build --chown=app:app /app/node_modules ./node_modules
COPY --from=build --chown=app:app /app/package.json ./

USER app
EXPOSE 3000
CMD ["node", "dist/server.js"]

# Result: No source code, no dev tools, no build cache in final image
```

---

## Docker BuildKit Secrets

```dockerfile
# Using BuildKit for build-time secrets (NEVER in ENV or ARG)
# syntax=docker/dockerfile:1

FROM python:3.11-slim

# Mount secret during build — NOT stored in image layers
RUN --mount=type=secret,id=pip_conf,target=/etc/pip.conf \
    pip install --no-cache-dir -r requirements.txt

# Mount SSH key for private repos
RUN --mount=type=ssh \
    pip install git+ssh://git@github.com/company/private-lib.git
```

```bash
# Build with secrets
DOCKER_BUILDKIT=1 docker build \
    --secret id=pip_conf,src=./pip.conf \
    --ssh default \
    -t myapp .
```

---

## Dockerfile Linting with Hadolint

```bash
# Install Hadolint
# macOS
brew install hadolint
# Docker
docker run --rm -i hadolint/hadolint < Dockerfile
# Windows
choco install hadolint

# Lint Dockerfile
hadolint Dockerfile

# Example output:
# Dockerfile:3 DL3006 warning: Always tag the version of an image explicitly
# Dockerfile:7 DL3008 warning: Pin versions in apt get install
# Dockerfile:12 DL3002 error: Last USER should not be root
# Dockerfile:15 DL4006 warning: Set the SHELL option -o pipefail

# Config file (.hadolint.yaml)
ignored:
  - DL3008   # Pin versions in apt-get (sometimes impractical)
trustedRegistries:
  - docker.io
  - gcr.io
  - registry.company.com
```

### Hadolint in CI/CD

```yaml
# GitHub Actions
name: Dockerfile Lint
on: [pull_request]

jobs:
  hadolint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          failure-threshold: error
```

---

## .dockerignore

```
# .dockerignore — Prevent secrets and unnecessary files from entering build context

# Version control
.git
.gitignore

# CI/CD
.github
.gitlab-ci.yml
Jenkinsfile

# Environment and secrets
.env
.env.*
*.pem
*.key
*.crt
id_rsa*

# Development
node_modules
__pycache__
*.pyc
.pytest_cache
.coverage
tests/

# Docker
Dockerfile*
docker-compose*
.dockerignore

# Documentation
README.md
docs/
LICENSE
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Image rebuilds every time | COPY before dependency install | Copy dependency files first, install, then copy source code |
| Secrets visible in image history | Used ENV/ARG for secrets | Use BuildKit `--mount=type=secret`; never ENV for secrets |
| Container runs as root | Missing USER directive | Add `USER nonroot` after creating non-root user |
| Image is 1GB+ | Using full base image, no multi-stage | Use `-slim` base, multi-stage build, clean apt cache |
| Hadolint false positive | Rule too strict for your case | Ignore specific rule in `.hadolint.yaml` with explanation |

---

## Summary Table

| Best Practice | Why |
|--------------|-----|
| **Pin image versions** | Reproducible builds; prevent unexpected changes |
| **Multi-stage builds** | Exclude build tools from production image |
| **Non-root user** | Prevent container escape / privilege escalation |
| **No secrets in Dockerfile** | Secrets visible in image layers and history |
| **Minimal packages** | Fewer packages = fewer vulnerabilities |
| **Use .dockerignore** | Prevent secrets/source from entering build context |
| **Hadolint** | Automated Dockerfile linting in CI/CD |
| **HEALTHCHECK** | Enables container orchestrator health monitoring |

---

## Quick Revision Questions

1. **Why should you never use `FROM image:latest` in production?**
   > `:latest` is non-deterministic — it changes every time the upstream image is updated. This means rebuilds can produce different images, breaking reproducibility. It can also introduce new vulnerabilities or breaking changes without notice. Always pin specific versions or use digest (`@sha256:...`).

2. **Why are multi-stage builds important for security?**
   > Multi-stage builds keep build tools (compilers, dev dependencies, source code) out of the final production image. This reduces the attack surface, prevents source code leakage, and shrinks image size. Only production artifacts are copied to the final stage.

3. **How should secrets be handled during Docker builds?**
   > Never use ENV or ARG for secrets (they persist in image layers). Use Docker BuildKit's `--mount=type=secret` which mounts secrets temporarily during build but never stores them in any layer. Pass runtime secrets via environment variables or secrets managers.

4. **What does Hadolint check for?**
   > Hadolint lints Dockerfiles for security and best practice issues: unpinned image versions, running as root, missing `--no-install-recommends`, shell form vs exec form CMD, unnecessary package installation, missing HEALTHCHECK, and more.

5. **Why is running containers as non-root critical?**
   > If an attacker exploits a vulnerability in the application, running as root gives them full OS-level access inside the container, and potential container escape through kernel vulnerabilities. Running as non-root limits the blast radius.

---

[← Previous: 6.2 Base Image Selection](02-base-image-selection.md) | [Next: 6.4 Runtime Security →](04-runtime-security.md)

[Back to Table of Contents](../README.md)
