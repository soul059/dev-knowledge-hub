# Chapter 5: Secrets Management

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

Secrets — passwords, API keys, certificates, and tokens — require special handling in containerized environments. Improperly managed secrets are one of the most common security vulnerabilities. This chapter covers the spectrum of secret management approaches from basic to enterprise-grade.

---

## 5.1 The Secrets Problem

```
  Where Secrets Can Leak:
  
  ✗ Dockerfile (ENV, ARG)     → Baked into image layers
  ✗ docker-compose.yaml       → Committed to git
  ✗ Environment variables     → Visible in inspect/proc
  ✗ Command line              → Visible in process list
  ✗ Build arguments           → Visible in image history
  ✗ Log output                → Persisted in log files
  
  docker inspect container_name
  → Shows ALL environment variables!
  
  /proc/1/environ inside container
  → Shows ALL environment variables!
  
  docker history image_name
  → Shows ARG and ENV values from build!
```

```
  Secret Exposure Points:
  
  Source Code   →   Build   →   Registry   →   Runtime
  +--------+      +------+     +--------+     +--------+
  | .env   |      | ARG  |     | Image  |     | docker |
  | config |      | ENV  |     | layers |     | inspect|
  | hard-  |      | COPY |     | contain|     | /proc/ |
  | coded  |      | .env |     | secrets|     | environ|
  +--------+      +------+     +--------+     +--------+
  
  Every stage is a potential leak point!
```

---

## 5.2 Docker Swarm Secrets

```bash
# Create a secret
echo "my-db-password" | docker secret create db_password -
docker secret create ssl_cert ./server.crt

# List secrets
docker secret ls

# Inspect (metadata only, not the value)
docker secret inspect db_password

# Use in a service
docker service create \
  --name api \
  --secret db_password \
  --secret source=ssl_cert,target=/etc/ssl/cert.pem,mode=0400 \
  myapi

# Secret is available inside container at:
# /run/secrets/db_password
# /etc/ssl/cert.pem (custom target)
```

```
  Docker Swarm Secrets:
  
  +-- Manager Node -----------+
  |                            |
  | Encrypted Raft Log         |
  | +------------------------+ |
  | | db_password (encrypted)| |
  | | ssl_cert (encrypted)   | |
  | +------------------------+ |
  |          |                 |
  +----------+-----------------+
             |
  TLS encrypted transfer
             |
  +----------+-----------------+
  |          v                 |
  | Worker Node               |
  | +--------+  +----------+  |
  | | tmpfs  |  | Container|  |
  | | /run/  |<>| reads    |  |
  | | secrets|  | secret   |  |
  | +--------+  +----------+  |
  |                            |
  +----------------------------+
  
  ✓ Encrypted at rest (Raft log)
  ✓ Encrypted in transit (TLS)
  ✓ In-memory only (tmpfs)
  ✓ Only sent to nodes running the service
```

---

## 5.3 Compose Secrets

```yaml
# compose.yaml
services:
  db:
    image: postgres:16
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

  api:
    build: ./api
    secrets:
      - db_password
      - api_key
      - source: ssl_cert
        target: /etc/ssl/cert.pem
        uid: "1000"
        mode: 0400

secrets:
  db_password:
    file: ./secrets/db_password.txt   # From file
  api_key:
    environment: API_KEY              # From host env var
  ssl_cert:
    file: ./certs/server.crt
```

```bash
# Reading secrets in application code
# Python
with open('/run/secrets/db_password', 'r') as f:
    db_password = f.read().strip()

# Node.js
const fs = require('fs');
const dbPassword = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();

# Go
password, err := os.ReadFile("/run/secrets/db_password")
```

---

## 5.4 BuildKit Secrets

```dockerfile
# syntax=docker/dockerfile:1

FROM node:20-alpine
WORKDIR /app

# Mount secret at build time (NOT stored in image layers)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm install --production

# SSH for private repos
RUN --mount=type=ssh \
    git clone git@github.com:private/repo.git
```

```bash
# Build with secrets
docker build --secret id=npmrc,src=.npmrc .

# Build with SSH agent
docker build --ssh default .

# Verify secret is not in any layer
docker history myimage
# No trace of .npmrc content!
```

```
  BuildKit Secret Mount:
  
  Build Time:
  +------- Layer N --------+
  | RUN npm install         |
  |   /root/.npmrc mounted  |  ← Secret available HERE ONLY
  |   (tmpfs, not in layer) |
  +-------------------------+
  
  Resulting Image:
  +------- Layer N --------+
  | RUN npm install         |
  |   (packages installed)  |  ← Secret NOT in image
  |   (no .npmrc)           |
  +-------------------------+
```

---

## 5.5 External Secret Managers

```
  Enterprise Secret Management:
  
  +----- HashiCorp Vault ------+
  | • Dynamic secrets          |
  | • Auto-rotation            |
  | • Audit logging            |
  | • Transit encryption       |
  | • PKI certificates         |
  +----------------------------+
  
  +----- AWS Secrets Manager --+
  | • AWS-native integration   |
  | • Auto-rotation            |
  | • CloudFormation support   |
  | • Cross-account sharing    |
  +----------------------------+
  
  +----- Azure Key Vault ------+
  | • Azure-native             |
  | • HSM-backed keys          |
  | • Managed identity access  |
  +----------------------------+
  
  Pattern: Init container or sidecar
  fetches secrets at startup
```

```yaml
# Pattern: Init container fetches secrets
services:
  secret-init:
    image: vault-agent
    volumes:
      - secrets:/secrets
    command: >
      vault agent -config=/etc/vault/agent.hcl
    restart: "no"

  api:
    image: myapi
    volumes:
      - secrets:/run/secrets:ro
    depends_on:
      secret-init:
        condition: service_completed_successfully

volumes:
  secrets:
    driver_opts:
      type: tmpfs
      device: tmpfs
```

---

## 5.6 Secret Management Best Practices

```
  Secret Rotation Strategy:
  
  +-- Secrets Lifecycle --------------------+
  |                                         |
  | Create → Distribute → Use → Rotate     |
  |    |         |          |       |       |
  |    v         v          v       v       |
  | Vault    tmpfs/file   App    New secret |
  | or file  mounting     reads  replaces   |
  |                               old       |
  +----------------------------------------+
  
  Best Practices:
  ✓ Rotate regularly (30-90 days)
  ✓ Use short-lived credentials
  ✓ Audit all secret access
  ✓ Don't log secrets
  ✓ Use _FILE convention for DB images
  ✓ Store in memory only (tmpfs)
  ✓ Encrypt at rest and in transit
```

```bash
# .gitignore for secrets
secrets/
*.key
*.pem
*.p12
*.pfx
.env
.env.*
!.env.example
```

---

## Summary Table

| Method | Security Level | Complexity | Best For |
|--------|---------------|------------|----------|
| **Environment vars** | Low | Simple | Non-sensitive config |
| **env_file (.env)** | Low-Medium | Simple | Development |
| **Compose secrets** | Medium | Low | Small deployments |
| **Swarm secrets** | High | Medium | Swarm clusters |
| **BuildKit secrets** | High | Low | Build-time secrets |
| **HashiCorp Vault** | Very High | High | Enterprise |
| **Cloud KMS** | Very High | Medium | Cloud-native apps |

---

## Quick Revision Questions

1. **Why are environment variables bad for storing secrets?**
   - They're visible via `docker inspect`, in `/proc/1/environ`, in process listings, and often logged. Anyone with container access can read them

2. **How do Docker Swarm secrets work?**
   - Secrets are encrypted in the Raft log, transferred over TLS, and mounted as tmpfs files at `/run/secrets/` in containers. They're only sent to nodes running the service

3. **How do BuildKit secret mounts differ from COPY?**
   - BuildKit `--mount=type=secret` makes secrets available during build but never stores them in image layers. COPY permanently embeds files in the layer

4. **What is the `_FILE` convention?**
   - Many official images (postgres, mysql) support reading secrets from files instead of environment variables: `POSTGRES_PASSWORD_FILE=/run/secrets/db_password`

5. **What should be in .gitignore for secrets security?**
   - `.env` files, `secrets/` directory, private keys (*.key, *.pem), certificates, and any file containing credentials. Keep `.env.example` with placeholder values

---

[← Previous: Docker Content Trust](04-docker-content-trust.md) | [Next: Security Best Practices →](06-security-best-practices.md)
