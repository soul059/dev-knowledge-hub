# Chapter 1: Production Best Practices

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Running Docker in production requires careful consideration of reliability, performance, observability, and operational processes. This chapter consolidates the essential practices for production-grade container deployments.

---

## 1.1 Production Architecture

```
  Production Container Architecture:
  
  Internet
     |
     v
  +----------- Load Balancer -----------+
  |  (HAProxy / Nginx / Cloud LB)       |
  +------+-------------------+----------+
         |                   |
  +------v------+     +------v------+
  | Docker Host |     | Docker Host |
  | (Node 1)    |     | (Node 2)    |
  +-------------+     +-------------+
  | web-1       |     | web-2       |
  | api-1 api-2 |     | api-3 api-4 |
  +------+------+     +------+------+
         |                   |
  +------v-------------------v------+
  |     Shared Storage / Database    |
  |  (Managed RDS, NFS, etc.)       |
  +----------------------------------+
```

---

## 1.2 Image Management

```bash
# Use specific image tags + digest for immutability
docker pull myregistry/api:v1.2.3@sha256:abc123...

# Tag strategy
myapp:latest         # ✗ Never in production
myapp:v1.2.3         # ✓ Semantic version
myapp:v1.2.3-alpine  # ✓ With variant
myapp:abc1234        # ✓ Git commit SHA
myapp:2024-01-15     # ✓ Date-based

# Use private registry
docker tag myapp:v1.2.3 registry.example.com/myapp:v1.2.3
docker push registry.example.com/myapp:v1.2.3
```

```
  Image Promotion Pipeline:
  
  Dev Registry        Staging Registry     Prod Registry
  +------------+      +------------+       +------------+
  | myapp:dev  | ---> | myapp:stg  | --->  | myapp:v1.2 |
  | (any push) |  QA  | (tested)   | Gate  | (approved) |
  +------------+      +------------+       +------------+
  
  Same image binary promoted through stages
  Never rebuild for different environments!
```

---

## 1.3 Container Configuration

```yaml
# Production-ready compose configuration
services:
  api:
    image: registry.example.com/api:v1.2.3
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 1G
        reservations:
          cpus: "0.5"
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "5"
    stop_grace_period: 30s
    init: true
```

---

## 1.4 Graceful Shutdown

```
  Container Shutdown Sequence:
  
  docker stop (or SIGTERM)
       |
       v
  +-----------+
  | SIGTERM   | ← App receives signal
  | sent to   |
  | PID 1     |
  +-----------+
       |
       v (grace period: 10s default)
  +-----------+
  | App       | ← Complete in-flight requests
  | cleanup   |   Close DB connections
  |           |   Flush buffers
  +-----------+
       |
  Still running after grace period?
       |
       v
  +-----------+
  | SIGKILL   | ← Forced termination
  | (kill -9) |   Data loss possible!
  +-----------+
```

```javascript
// Node.js graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down...');
  
  // Stop accepting new requests
  server.close(async () => {
    // Close database connections
    await db.close();
    // Close Redis connections
    await redis.quit();
    process.exit(0);
  });
  
  // Force exit after timeout
  setTimeout(() => process.exit(1), 25000);
});
```

```bash
# Increase grace period for slow apps
docker stop --time 30 mycontainer

# In compose
services:
  api:
    stop_grace_period: 30s
```

---

## 1.5 Health Checks in Production

```yaml
# Different health check patterns

# HTTP health check
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 60s

# TCP health check (no curl needed)
healthcheck:
  test: ["CMD-SHELL", "nc -z localhost 3000 || exit 1"]

# Custom script
healthcheck:
  test: ["CMD", "/app/healthcheck.sh"]
```

```
  Health Check States:
  
  Container Start
       |
       v
  [starting] ──── start_period (60s) ────+
       |                                  |
       v                                  v
  First check                        Ignore failures
       |                             during startup
       v
  [healthy] ←── interval (30s) ──── pass
       |
       v (retries failures)
  [unhealthy] ──── restart policy kicks in
  
  Healthy:   Load balancer sends traffic
  Unhealthy: Load balancer removes, restart triggered
```

---

## 1.6 Production Checklist

```
  +===+==============================================+
  | # | Production Readiness Checklist               |
  +===+==============================================+
  |   | IMAGES                                       |
  | 1 | Specific version tags (no :latest)           |
  | 2 | Images scanned for CVEs                      |
  | 3 | Multi-stage build (minimal size)             |
  | 4 | Non-root user in Dockerfile                  |
  | 5 | Private registry for proprietary images      |
  +---+----------------------------------------------+
  |   | RUNTIME                                      |
  | 6 | Resource limits set (CPU, memory)            |
  | 7 | Health checks configured                     |
  | 8 | Graceful shutdown handling                    |
  | 9 | Restart policy (unless-stopped / always)     |
  |10 | Log rotation configured                      |
  +---+----------------------------------------------+
  |   | DATA                                         |
  |11 | Named volumes for persistent data            |
  |12 | Automated backup strategy                    |
  |13 | Tested restore procedure                     |
  +---+----------------------------------------------+
  |   | SECURITY                                     |
  |14 | Secrets via Docker secrets or vault          |
  |15 | Network isolation (internal networks)        |
  |16 | Capabilities dropped                         |
  |17 | Read-only filesystem where possible          |
  +---+----------------------------------------------+
  |   | OPERATIONS                                   |
  |18 | Centralized logging                          |
  |19 | Monitoring and alerting                      |
  |20 | Documented deployment process                |
  |21 | Rollback procedure tested                    |
  +===+==============================================+
```

---

## Summary Table

| Aspect | Recommendation |
|--------|---------------|
| **Image tags** | Semantic version + digest |
| **Restart policy** | `unless-stopped` |
| **Resource limits** | Always set CPU + memory |
| **Health checks** | HTTP/TCP with start_period |
| **Graceful shutdown** | Handle SIGTERM, 30s grace |
| **Init process** | Use `tini` or `--init` |
| **Logging** | json-file with rotation |
| **Registry** | Private, access-controlled |
| **Stop grace** | 30s for most apps |
| **Deployments** | Same image across environments |

---

## Quick Revision Questions

1. **Why should you never use `:latest` tag in production?**
   - It's mutable — a new push changes what `:latest` means. Use specific versions for reproducible deployments and reliable rollbacks

2. **What is graceful shutdown and why does it matter?**
   - Handling SIGTERM to complete in-flight requests, close connections, and flush data before exiting. Without it, SIGKILL after the grace period can cause data loss

3. **Why use `--init` or tini in production containers?**
   - Node apps running as PID 1 don't handle signals properly by default. Tini acts as a proper init system, forwarding signals and reaping zombie processes

4. **What is the purpose of `start_period` in health checks?**
   - It gives the application time to start up before health check failures count toward the retry limit. Prevents premature restart of slow-starting apps

5. **Why should the same image be promoted across environments?**
   - Rebuilding for each environment can introduce differences. Promoting the same binary ensures what was tested in staging is exactly what runs in production

---

[← Previous Unit: Docker Security](../09-Docker-Security/06-security-best-practices.md) | [Next: Monitoring and Observability →](02-monitoring-and-observability.md)
