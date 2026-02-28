# Chapter 7: Compose in Production

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

While Docker Compose is primarily a development tool, it can be used effectively for small-to-medium production deployments. This chapter covers production hardening patterns, deployment strategies, and knowing when to graduate to container orchestration platforms.

---

## 7.1 Production-Ready Compose File

```yaml
# compose.prod.yaml
name: myapp-production

services:
  web:
    image: nginx:1.25-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/usr/share/nginx/html/static:ro
    depends_on:
      api:
        condition: service_healthy
    restart: always
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"

  api:
    image: registry.example.com/myapi:${VERSION:-latest}
    expose:
      - "3000"
    environment:
      NODE_ENV: production
    env_file:
      - .env.production
    secrets:
      - db_password
      - jwt_secret
    depends_on:
      db:
        condition: service_healthy
    restart: always
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: "1"
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    restart: always
    deploy:
      resources:
        limits:
          cpus: "2"
          memory: 1G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  pgdata:
  static_files:

networks:
  default:
  backend:
    internal: true

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

---

## 7.2 Production Checklist

```
  Production Compose Checklist:
  
  +--+--------------------------------------------------+
  | ✓| Use specific image tags (not :latest)            |
  | ✓| Set restart: always or unless-stopped            |
  | ✓| Configure resource limits (CPU, memory)          |
  | ✓| Add healthchecks to all services                 |
  | ✓| Use secrets for sensitive data (not env vars)    |
  | ✓| Configure log rotation (max-size, max-file)      |
  | ✓| Use read-only mounts for config files            |
  | ✓| Isolate networks (internal for databases)        |
  | ✓| Don't expose database ports to host              |
  | ✓| Set deploy.resources.limits                      |
  | ✓| Use .env.production (not .env)                   |
  | ✓| Add depends_on with service_healthy              |
  | ✓| Run containers as non-root user                  |
  | ✓| Use named volumes for persistent data            |
  +--+--------------------------------------------------+
```

---

## 7.3 Deployment Strategies

```bash
# Strategy 1: Pull and recreate
docker compose -f compose.prod.yaml pull
docker compose -f compose.prod.yaml up -d

# Strategy 2: Zero-downtime rolling update
# Update one service at a time
docker compose -f compose.prod.yaml pull api
docker compose -f compose.prod.yaml up -d --no-deps api

# Strategy 3: Blue-Green with nginx
docker compose -f compose.prod.yaml up -d --scale api=4
# Wait for new containers to be healthy
docker compose -f compose.prod.yaml up -d --scale api=2
```

```
  Rolling Update Pattern:
  
  Time ────────────────────────────────>
  
  Step 1: Pull new image
  api-1: [old v1] [old v1]
  api-2: [old v1] [old v1]
  
  Step 2: Recreate service
  api-1: [old v1] → [stopping] → [new v2 starting]
  api-2: [old v1] → [old v1]  → [old v1]
  
  Step 3: Health check passes
  api-1: [new v2 ✓ healthy]
  api-2: [old v1] → [stopping] → [new v2 starting]
  
  Step 4: Complete
  api-1: [new v2 ✓]
  api-2: [new v2 ✓]
  
  Note: Brief downtime possible per container
  For true zero-downtime, use orchestrators
```

---

## 7.4 Monitoring and Health

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
```

```bash
# Monitor service health
docker compose ps
docker compose ps --format json

# Check specific service health
docker inspect --format='{{.State.Health.Status}}' myapp-api-1

# View resource usage
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Follow logs across services
docker compose logs -f --tail=100

# Follow specific service
docker compose logs -f api
```

---

## 7.5 Backup Strategies

```bash
# Database backup script
#!/bin/bash
BACKUP_DIR=/backups
DATE=$(date +%Y%m%d_%H%M%S)

# Backup PostgreSQL
docker compose exec -T db \
  pg_dump -U postgres myapp | \
  gzip > ${BACKUP_DIR}/db_${DATE}.sql.gz

# Backup volumes
docker run --rm \
  -v myapp_pgdata:/source:ro \
  -v ${BACKUP_DIR}:/backup \
  alpine tar czf /backup/pgdata_${DATE}.tar.gz -C /source .

# Rotate old backups (keep 7 days)
find ${BACKUP_DIR} -name "*.gz" -mtime +7 -delete
```

```
  Backup Strategy:
  
  ┌─────────────────┐
  │  Cron Job       │
  │  (daily)        │
  └────────┬────────┘
           │
      ┌────▼────┐     ┌──────────────┐
      │ pg_dump │────>│ /backups/    │
      │ via exec│     │  db_DATE.gz  │
      └─────────┘     └──────────────┘
           │
      ┌────▼────┐     ┌──────────────┐
      │ Volume  │────>│ /backups/    │
      │ tar     │     │  vol_DATE.gz │
      └─────────┘     └──────────────┘
           │
      ┌────▼────┐
      │ Rotate  │ (keep 7 days)
      │ cleanup │
      └─────────┘
```

---

## 7.6 When to Move Beyond Compose

```
  Compose vs Orchestrators:
  
  Docker Compose              Orchestrators
  (single host)               (Swarm / Kubernetes)
  +--------------------+      +--------------------+
  | ✓ Simple setup     |      | ✓ Multi-host       |
  | ✓ Fast dev cycles  |      | ✓ Auto-scaling     |
  | ✓ Easy debugging   |      | ✓ Self-healing     |
  | ✓ Low overhead     |      | ✓ Rolling updates  |
  |                    |      | ✓ Service mesh     |
  | ✗ Single host only |      | ✓ Load balancing   |
  | ✗ Manual scaling   |      | ✓ Secret rotation  |
  | ✗ No auto-healing  |      |                    |
  | ✗ Basic networking |      | ✗ Complex setup    |
  +--------------------+      | ✗ Higher overhead  |
                              +--------------------+
  
  Move to orchestrators when you need:
  - High availability (multi-host)
  - Auto-scaling based on load
  - Zero-downtime deployments
  - Managing 10+ services
```

---

## Summary Table

| Production Aspect | Recommendation |
|-------------------|---------------|
| **Image tags** | Specific versions, never `:latest` |
| **Restart policy** | `always` or `unless-stopped` |
| **Resources** | Set CPU and memory limits |
| **Health checks** | On every service |
| **Secrets** | Use `secrets:` not environment variables |
| **Logging** | json-file with rotation (10m, 5 files) |
| **Networks** | Internal for databases |
| **Ports** | Only expose what's needed |
| **Backups** | Automated with rotation |
| **Updates** | Pull + up with `--no-deps` per service |

---

## Quick Revision Questions

1. **Why shouldn't you use `:latest` tag in production?**
   - It's mutable and can change unexpectedly. Use specific version tags for reproducible deployments and easy rollbacks

2. **How do you prevent log files from filling up disk space?**
   - Configure log rotation: `logging: { driver: json-file, options: { max-size: "10m", max-file: "5" } }`

3. **What is the recommended restart policy for production?**
   - `always` or `unless-stopped`. Both restart on crashes and Docker daemon restarts. `unless-stopped` respects manual `docker compose stop`

4. **How do you perform a rolling update with Compose?**
   - Pull the new image (`docker compose pull api`), then recreate the service (`docker compose up -d --no-deps api`). This minimizes downtime

5. **When should you consider moving from Compose to Kubernetes?**
   - When you need multi-host deployment, auto-scaling, zero-downtime rolling updates, or are managing many services at scale

6. **How do you backup a database running in Compose?**
   - Use `docker compose exec -T db pg_dump` to create a logical backup, piped to a file on the host. Automate with cron and add backup rotation

---

[← Previous: Environment and Secrets](06-environment-and-secrets.md) | [Next Unit: Docker Security →](../09-Docker-Security/01-security-overview.md)
