# Chapter 5: Build Args and Environment Variables

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

Docker provides two mechanisms for parameterizing builds and runtime: **ARG** (build-time variables) and **ENV** (environment variables). Understanding their differences, scope, and interaction is key to creating flexible, configurable images.

---

## 5.1 ARG vs ENV at a Glance

```
  Build Time                              Runtime
  +--------------------------+            +------------------+
  |  docker build            |            |  docker run      |
  |                          |            |                  |
  |  ARG  ✓  Available       |            |  ARG  ✗  Gone    |
  |  ENV  ✓  Available       |            |  ENV  ✓  Available|
  |                          |            |                  |
  +--------------------------+            +------------------+
  
  ARG = Build-time ONLY
  ENV = Build-time AND Runtime
```

---

## 5.2 ARG — Build Arguments

### Basic Usage

```dockerfile
# Declare with default value
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}-alpine

# Declare without default
ARG APP_VERSION
LABEL version=${APP_VERSION}

# Multiple ARGs
ARG BUILD_DATE
ARG GIT_COMMIT
ARG BUILD_NUMBER
```

```bash
# Override at build time
docker build --build-arg NODE_VERSION=18 .
docker build --build-arg APP_VERSION=2.1.0 .

# Multiple build args
docker build \
  --build-arg APP_VERSION=2.1.0 \
  --build-arg BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --build-arg GIT_COMMIT=$(git rev-parse HEAD) \
  .
```

### ARG Scope Rules

```dockerfile
# ARG BEFORE FROM: Only available in FROM
ARG BASE_VERSION=3.19
FROM alpine:${BASE_VERSION}

# ARG not available here! (new stage resets scope)
# RUN echo ${BASE_VERSION}  <-- Empty!

# Must re-declare after FROM to use
ARG BASE_VERSION
RUN echo "Version: ${BASE_VERSION}"
```

```
  ARG Scope Diagram:
  
  ARG VERSION=20           <-- Global scope (before FROM)
  |
  FROM node:${VERSION}     <-- ✓ Can use VERSION
  |
  |  ARG VERSION           <-- Must re-declare
  |  RUN echo ${VERSION}   <-- ✓ Now available
  |
  FROM alpine              <-- New stage
  |
  |  ARG VERSION           <-- Must re-declare again
  |  RUN echo ${VERSION}   <-- ✓ Available again
```

### Predefined ARGs

```dockerfile
# Docker provides these automatically (no ARG declaration needed)
# HTTP_PROXY, HTTPS_PROXY, FTP_PROXY, NO_PROXY
# http_proxy, https_proxy, ftp_proxy, no_proxy
# ALL_PROXY, all_proxy

# They don't create layers and aren't shown in image history
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl
# Proxy args work automatically if set during build

# Platform ARGs (available with BuildKit)
# TARGETPLATFORM   - linux/amd64, linux/arm64
# TARGETOS         - linux, windows
# TARGETARCH       - amd64, arm64, arm
# TARGETVARIANT    - v7, v8
# BUILDPLATFORM    - platform of the build node
# BUILDOS, BUILDARCH, BUILDVARIANT
```

---

## 5.3 ENV — Environment Variables

### Basic Usage

```dockerfile
# Set environment variables
ENV NODE_ENV=production
ENV APP_PORT=3000

# Old syntax (still works, but use = form)
ENV APP_PORT 3000

# Multiple on one line
ENV DB_HOST=localhost DB_PORT=5432

# Reference in subsequent instructions
ENV APP_HOME=/app
WORKDIR ${APP_HOME}
COPY . ${APP_HOME}
```

### Runtime Override

```bash
# Override ENV at runtime
docker run -e NODE_ENV=development myapp
docker run -e APP_PORT=8080 myapp

# Pass host environment variable
docker run -e DB_HOST myapp  # Uses host's DB_HOST value

# Environment file
docker run --env-file .env myapp
```

```
  Priority (highest to lowest):
  
  1. docker run -e KEY=val     (CLI override)
  2. docker run --env-file     (env file)
  3. ENV KEY=val               (Dockerfile)
  4. (empty if not set)
```

---

## 5.4 ARG and ENV Interaction

```dockerfile
# Common pattern: Convert ARG to ENV
ARG APP_VERSION=1.0.0
ENV APP_VERSION=${APP_VERSION}

# Now APP_VERSION is:
# - Configurable at build time (--build-arg)
# - Available at runtime (as environment variable)
```

```
  Build Command:
  docker build --build-arg APP_VERSION=2.0 .
  
  +-------------------------------------------+
  |  Dockerfile                               |
  |                                           |
  |  ARG APP_VERSION=1.0.0                    |
  |      ^                                    |
  |      | Overridden to 2.0 by --build-arg   |
  |                                           |
  |  ENV APP_VERSION=${APP_VERSION}            |
  |      |                                    |
  |      v                                    |
  |  Runtime: APP_VERSION=2.0                 |
  +-------------------------------------------+
```

### Complete Example

```dockerfile
FROM python:3.11-slim

# Build arguments
ARG APP_VERSION=1.0.0
ARG BUILD_DATE
ARG GIT_COMMIT

# Convert to ENV (available at runtime)
ENV APP_VERSION=${APP_VERSION}

# Use ARG for metadata only (not in runtime)
LABEL build-date=${BUILD_DATE}
LABEL git-commit=${GIT_COMMIT}
LABEL version=${APP_VERSION}

WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

ENV FLASK_ENV=production
ENV PORT=5000

EXPOSE ${PORT}
CMD ["python", "app.py"]
```

```bash
# Build with all args
docker build \
  --build-arg APP_VERSION=2.1.0 \
  --build-arg BUILD_DATE=$(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --build-arg GIT_COMMIT=$(git rev-parse --short HEAD) \
  -t myapp:2.1.0 .

# Run with ENV override
docker run -e PORT=8080 -e FLASK_ENV=development myapp:2.1.0
```

---

## 5.5 Variable Substitution

```dockerfile
# Dollar sign with braces
ENV APP_HOME=/app
WORKDIR ${APP_HOME}

# Dollar sign without braces
WORKDIR $APP_HOME

# Default value if not set
ENV PORT=${PORT:-3000}

# Escape dollar sign
RUN echo \$HOME      # Prints: $HOME
```

### Supported Instructions

```
  +------------------+-------------------+
  | Supports $VAR    | Does NOT Support  |
  +------------------+-------------------+
  | ADD              | (all support      |
  | COPY             |  variable         |
  | ENV              |  substitution     |
  | EXPOSE           |  from ENV and     |
  | FROM             |  ARG values)      |
  | LABEL            |                   |
  | STOPSIGNAL       |                   |
  | USER             |                   |
  | VOLUME           |                   |
  | WORKDIR          |                   |
  | RUN              |                   |
  +------------------+-------------------+
```

---

## 5.6 Security Considerations

### ARG Secrets Problem

```dockerfile
# DANGER: ARG values are stored in image history!
ARG DB_PASSWORD=secret123
RUN echo "Connecting to DB with $DB_PASSWORD"

# Anyone can see it:
# docker history myimage
# Shows: ARG DB_PASSWORD=secret123
```

### Secure Alternatives

```dockerfile
# Option 1: BuildKit Secret Mounts (RECOMMENDED)
# syntax=docker/dockerfile:1
RUN --mount=type=secret,id=db_pass \
    export DB_PASS=$(cat /run/secrets/db_pass) && \
    ./setup-db.sh

# Build with:
# docker build --secret id=db_pass,src=./db_password.txt .

# Option 2: Runtime environment variables
# Don't bake secrets into the image at all
# Pass them at runtime:
# docker run -e DB_PASSWORD=secret myapp

# Option 3: Docker secrets (Swarm mode)
# docker secret create db_pass ./db_password.txt
```

```
  Bad Practice:                     Good Practice:
  
  +--------------------+           +--------------------+
  | ARG SECRET=abc123  |           | RUN --mount=secret |
  |                    |           | (BuildKit)         |
  | Stored in layer    |           | NOT stored in      |
  | history!           |           | image layers       |
  | Visible to anyone  |           | Visible only       |
  | with image access  |           | during RUN         |
  +--------------------+           +--------------------+
```

---

## 5.7 Practical Patterns

### Pattern 1: Configurable Base Image

```dockerfile
ARG PYTHON_VERSION=3.11
ARG VARIANT=slim
FROM python:${PYTHON_VERSION}-${VARIANT}
```

### Pattern 2: Feature Flags

```dockerfile
ARG INSTALL_DEV_TOOLS=false
RUN if [ "$INSTALL_DEV_TOOLS" = "true" ]; then \
        apt-get update && apt-get install -y vim curl htop; \
    fi
```

### Pattern 3: OCI Labels

```dockerfile
ARG BUILD_DATE
ARG VCS_REF
ARG VERSION

LABEL org.opencontainers.image.created=${BUILD_DATE} \
      org.opencontainers.image.revision=${VCS_REF} \
      org.opencontainers.image.version=${VERSION} \
      org.opencontainers.image.source="https://github.com/org/repo"
```

### Pattern 4: Application Configuration

```dockerfile
# Sensible defaults that can be overridden
ENV LOG_LEVEL=info
ENV MAX_WORKERS=4
ENV CACHE_TTL=3600
ENV DB_HOST=localhost
ENV DB_PORT=5432

# Application reads these at runtime
CMD ["python", "app.py"]
```

### Pattern 5: Multi-Platform Build

```dockerfile
ARG TARGETARCH
RUN case ${TARGETARCH} in \
        amd64) ARCH="x86_64" ;; \
        arm64) ARCH="aarch64" ;; \
    esac && \
    curl -L "https://example.com/tool-${ARCH}" -o /usr/local/bin/tool
```

---

## 5.8 Layer Caching Impact

```dockerfile
# ENV/ARG changes invalidate cache from that point

# GOOD: Put infrequently changing ENV early
ENV NODE_ENV=production
COPY package.json .
RUN npm ci
COPY . .

# BAD: Changing ARG busts cache for all subsequent layers
ARG BUILD_TIMESTAMP          # Changes every build!
RUN echo ${BUILD_TIMESTAMP}  # Cache busted every time
COPY . .                     # Also re-runs unnecessarily
RUN npm ci                   # Also re-runs!

# FIX: Put volatile ARGs as late as possible
COPY package.json .
RUN npm ci
COPY . .
ARG BUILD_TIMESTAMP          # Only affects layers after this
LABEL build-time=${BUILD_TIMESTAMP}
```

---

## Summary Table

| Feature | ARG | ENV |
|---------|-----|-----|
| **Available during build** | Yes | Yes |
| **Available at runtime** | No | Yes |
| **Set via CLI** | `--build-arg` | `-e` / `--env` |
| **Default value** | `ARG key=val` | `ENV key=val` |
| **Scope** | Stage-specific | Persists across stages |
| **Security** | Visible in history | Override at runtime |
| **Use for** | Build config | App config |

---

## Quick Revision Questions

1. **What is the key difference between ARG and ENV?**
   - ARG is available only during build time; ENV is available during both build time and runtime in the running container

2. **How do you pass a build argument at build time?**
   - Using `docker build --build-arg KEY=VALUE .`

3. **Why should you NOT use ARG for secrets?**
   - ARG values are stored in the image history/metadata and can be viewed by anyone with access to the image using `docker history`

4. **What happens to ARG scope after a FROM instruction?**
   - ARG scope resets after each FROM. You must re-declare the ARG in the new stage to use it

5. **How do you make an ARG value available at runtime?**
   - Convert it to ENV: `ARG MY_VAR` followed by `ENV MY_VAR=${MY_VAR}`

6. **What is the recommended way to handle build-time secrets?**
   - Use BuildKit secret mounts: `RUN --mount=type=secret,id=mysecret ...` which are not stored in any image layer

---

[← Previous: Dockerignore](04-dockerignore.md) | [Next: Dockerfile Best Practices →](06-dockerfile-best-practices.md)
