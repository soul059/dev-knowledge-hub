# Chapter 6: Security Best Practices

> **Unit 9: Docker Security** | [Course Home](../README.md)

---

## Overview

This chapter consolidates Docker security best practices into actionable guidelines covering the entire lifecycle — from development to production deployment. Use this as a comprehensive checklist for securing your containerized applications.

---

## 6.1 Dockerfile Security Hardening

```dockerfile
# Production-hardened Dockerfile
# syntax=docker/dockerfile:1

# 1. Use specific, minimal base image
FROM node:20.10-alpine AS builder

WORKDIR /app

# 2. Install dependencies first (cache-friendly)
COPY package*.json ./
RUN npm ci --omit=dev

# 3. Copy application code
COPY . .
RUN npm run build

# ------- Production Stage -------
FROM node:20.10-alpine

# 4. Install tini for PID 1
RUN apk add --no-cache tini

# 5. Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

WORKDIR /app

# 6. Copy only needed artifacts with proper ownership
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/package.json ./

# 7. Set environment
ENV NODE_ENV=production

# 8. Expose only needed ports
EXPOSE 3000

# 9. Add health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# 10. Switch to non-root user
USER appuser

# 11. Use tini as init
ENTRYPOINT ["tini", "--"]

# 12. Run application
CMD ["node", "dist/server.js"]
```

---

## 6.2 Runtime Security Hardening

```bash
# Maximum security container run command
docker run -d \
  --name secure-app \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --tmpfs /var/run:rw,noexec,nosuid \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  --security-opt no-new-privileges:true \
  --security-opt seccomp=default \
  --memory 512m \
  --memory-swap 512m \
  --cpus 1.0 \
  --pids-limit 100 \
  --user 1001:1001 \
  --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  myapp:v1.0
```

```
  Security Layers Applied:
  
  +-------------------------------------------+
  | --read-only        → Immutable filesystem  |
  | --cap-drop ALL     → No root capabilities  |
  | --no-new-privileges → No escalation        |
  | --security-opt     → Seccomp filtering     |
  | --memory 512m      → Resource limit        |
  | --pids-limit 100   → Fork bomb protection  |
  | --user 1001        → Non-root process      |
  | -p 127.0.0.1:...   → Localhost only        |
  +-------------------------------------------+
```

---

## 6.3 Compose Security Template

```yaml
# Secure compose.yaml template
services:
  api:
    image: myregistry/api:v1.2.3      # Specific tag
    read_only: true                     # Read-only FS
    user: "1001:1001"                  # Non-root
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    tmpfs:
      - /tmp:size=64m,noexec,nosuid
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          memory: 256M
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"

  db:
    image: postgres:16-alpine
    user: "999:999"
    read_only: true
    tmpfs:
      - /tmp
      - /run/postgresql
    volumes:
      - pgdata:/var/lib/postgresql/data
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    networks:
      - backend
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_OVERRIDE
      - FOWNER
      - SETGID
      - SETUID
    deploy:
      resources:
        limits:
          memory: 1G

networks:
  frontend:
  backend:
    internal: true

volumes:
  pgdata:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

## 6.4 CI/CD Security Pipeline

```
  Secure CI/CD Pipeline:
  
  Code Push
     |
     v
  +-----------+
  | Lint      | ← hadolint (Dockerfile linting)
  | Dockerfile|
  +-----------+
     |
     v
  +-----------+
  | Build     | ← BuildKit secrets (no leaks)
  | Image     |   Multi-stage (minimal image)
  +-----------+
     |
     v
  +-----------+
  | Scan      | ← Trivy/Grype/Snyk
  | Image     |   FAIL on CRITICAL CVEs
  +-----------+
     |
     v
  +-----------+
  | Sign      | ← Cosign / DCT
  | Image     |   Verify provenance
  +-----------+
     |
     v
  +-----------+
  | Push to   | ← Private registry
  | Registry  |   Access-controlled
  +-----------+
     |
     v
  +-----------+
  | Deploy    | ← Signed images only
  | (verify)  |   Runtime hardening
  +-----------+
```

```bash
# CI/CD security commands
# 1. Lint Dockerfile
hadolint Dockerfile

# 2. Build with secrets
docker build --secret id=npmrc,src=.npmrc -t myapp:v1.0 .

# 3. Scan for vulnerabilities
trivy image --exit-code 1 --severity CRITICAL,HIGH myapp:v1.0

# 4. Sign image
cosign sign --key cosign.key myregistry/myapp:v1.0

# 5. Push
docker push myregistry/myapp:v1.0
```

---

## 6.5 Audit and Monitoring

```bash
# Docker events (real-time audit)
docker events --filter type=container
docker events --format '{{.Time}} {{.Action}} {{.Actor.Attributes.name}}'

# Check container security settings
docker inspect --format='
  User: {{.Config.User}}
  Privileged: {{.HostConfig.Privileged}}
  ReadOnly: {{.HostConfig.ReadonlyRootfs}}
  CapDrop: {{.HostConfig.CapDrop}}
  SecurityOpt: {{.HostConfig.SecurityOpt}}
' mycontainer

# Docker Bench Security (CIS benchmark audit)
docker run --rm --net host --pid host \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security
```

---

## 6.6 Complete Security Checklist

```
  +===+==========================================+=========+
  | # | Practice                                 | Priority|
  +===+==========================================+=========+
  |   | IMAGE SECURITY                           |         |
  +---+------------------------------------------+---------+
  | 1 | Use official/verified base images         | HIGH    |
  | 2 | Pin specific image versions (tags+digest) | HIGH    |
  | 3 | Scan images in CI/CD pipeline             | HIGH    |
  | 4 | Use minimal base (Alpine/distroless)      | MEDIUM  |
  | 5 | Multi-stage builds (no build tools)       | MEDIUM  |
  | 6 | No secrets in images or build args        | HIGH    |
  | 7 | Sign images (Cosign/DCT)                  | MEDIUM  |
  +---+------------------------------------------+---------+
  |   | RUNTIME SECURITY                         |         |
  +---+------------------------------------------+---------+
  | 8 | Run as non-root user                      | HIGH    |
  | 9 | Drop ALL capabilities, add specific       | HIGH    |
  |10 | Read-only root filesystem                 | MEDIUM  |
  |11 | Set no-new-privileges                     | HIGH    |
  |12 | Set resource limits (mem, CPU, PIDs)       | HIGH    |
  |13 | Never use --privileged                    | HIGH    |
  |14 | Don't mount Docker socket                | HIGH    |
  +---+------------------------------------------+---------+
  |   | NETWORK SECURITY                         |         |
  +---+------------------------------------------+---------+
  |15 | Separate networks per tier                | MEDIUM  |
  |16 | Use internal networks for databases      | HIGH    |
  |17 | Bind ports to localhost                   | MEDIUM  |
  |18 | Use encrypted overlay for multi-host     | MEDIUM  |
  +---+------------------------------------------+---------+
  |   | SECRET MANAGEMENT                        |         |
  +---+------------------------------------------+---------+
  |19 | Use Docker secrets or external vault     | HIGH    |
  |20 | BuildKit secret mounts for builds        | HIGH    |
  |21 | .gitignore for .env and secret files     | HIGH    |
  |22 | Rotate secrets regularly                 | MEDIUM  |
  +---+------------------------------------------+---------+
  |   | OPERATIONAL                              |         |
  +---+------------------------------------------+---------+
  |23 | Configure log rotation                   | MEDIUM  |
  |24 | Monitor with Docker events               | MEDIUM  |
  |25 | Run Docker Bench Security audit          | LOW     |
  |26 | Keep Docker and host OS updated          | HIGH    |
  +===+==========================================+=========+
```

---

## Summary Table

| Area | Key Practice | Implementation |
|------|-------------|----------------|
| **Images** | Minimal + scanned | Alpine/distroless + Trivy in CI |
| **User** | Non-root | `USER 1001` in Dockerfile |
| **Capabilities** | Minimal | `--cap-drop ALL --cap-add ...` |
| **Filesystem** | Read-only | `--read-only` + tmpfs |
| **Privileges** | Block escalation | `no-new-privileges` |
| **Resources** | Limited | `--memory`, `--cpus`, `--pids-limit` |
| **Secrets** | File-based | Docker secrets or vault |
| **Networks** | Isolated | Separate networks, internal |
| **Logging** | Rotated | `max-size: 10m, max-file: 5` |
| **Audit** | Continuous | Docker Bench, events monitoring |

---

## Quick Revision Questions

1. **What are the top 3 most critical Docker security practices?**
   - Run as non-root, never use `--privileged`, and scan images for vulnerabilities in CI/CD

2. **How do you create a maximally hardened container run command?**
   - Combine: `--read-only`, `--cap-drop ALL`, `--no-new-privileges`, `--user`, `--memory`, `--pids-limit`, and bind to localhost

3. **What is Docker Bench Security?**
   - An automated script that checks for CIS Docker Benchmark compliance, auditing host configuration, daemon settings, container runtime, and image security

4. **What should a security-focused CI/CD pipeline include?**
   - Dockerfile linting (hadolint), image scanning (Trivy), image signing (Cosign), private registry, and signed image verification at deploy

5. **Why is `--read-only` important?**
   - It prevents attackers from modifying container binaries, installing backdoors, or changing configuration. Combined with tmpfs, it allows writable temp dirs without persistent modification

6. **How do you audit the security posture of running containers?**
   - Use `docker inspect` to check security settings, `docker events` for real-time monitoring, and Docker Bench for comprehensive CIS benchmark auditing

---

[← Previous: Secrets Management](05-secrets-management.md) | [Next Unit: Docker in Production →](../10-Docker-in-Production/01-production-best-practices.md)
