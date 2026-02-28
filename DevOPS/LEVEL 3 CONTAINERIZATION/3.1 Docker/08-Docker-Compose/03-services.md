# Chapter 3: Services

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

Services are the core building blocks of a Compose application. Each service defines a container configuration including its image, dependencies, scaling behavior, and lifecycle management. Understanding service configuration patterns is key to building robust multi-container applications.

---

## 3.1 Service Lifecycle

```
  Service Lifecycle with Compose:
  
  compose.yaml
       |
       v
  docker compose up
       |
       +---> Create network(s)
       +---> Create volume(s)
       +---> Pull/Build image(s)
       +---> Create container(s)
       +---> Start container(s)
             (respecting depends_on order)
  
  docker compose down
       |
       +---> Stop container(s)
       +---> Remove container(s)
       +---> Remove network(s)
       +---> (Keep volumes unless -v)
```

```bash
# Service management commands
docker compose up -d              # Start all services
docker compose up -d api          # Start specific service + deps
docker compose stop api           # Stop a service
docker compose start api          # Start a stopped service
docker compose restart api        # Restart a service
docker compose rm api             # Remove stopped service
docker compose up -d --no-deps api  # Start without dependencies
```

---

## 3.2 depends_on and Startup Order

```yaml
services:
  web:
    image: nginx
    depends_on:
      api:
        condition: service_healthy
        restart: true             # Restart if dep restarts

  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

```
  Startup Order with depends_on:
  
  Time ──────────────────────────────────>
  
  db:    [Start]---[healthy ✓]
                        |
  redis: [Start]--[started ✓]
                   |    |
  api:             [Start]------[healthy ✓]
                                     |
  web:                          [Start]---[running]
  
  condition: service_started  → Wait for start only
  condition: service_healthy  → Wait for healthcheck ✓
  
  Shutdown is REVERSE order:
  web → api → redis, db
```

---

## 3.3 Restart Policies

```yaml
services:
  api:
    image: myapi
    restart: unless-stopped    # Recommended for production

# Restart options:
# "no"            - Never restart (default)
# always          - Always restart
# on-failure      - Restart only on non-zero exit
# unless-stopped  - Restart unless explicitly stopped
```

```
  Restart Policy Behavior:
  
  Policy           Exit 0   Exit 1   Docker    After
                   (clean)  (error)  Restart   docker stop
  +---------------+--------+--------+---------+----------+
  | no            | Stay   | Stay   | No      | No       |
  | always        | Start  | Start  | Start   | Start    |
  | on-failure    | Stay   | Start  | No      | No       |
  | unless-stopped| Start  | Start  | Start   | No       |
  +---------------+--------+--------+---------+----------+
  
  "Docker Restart" = After Docker daemon restarts
```

---

## 3.4 Scaling Services

```bash
# Scale during up
docker compose up -d --scale api=3

# Scale running service
docker compose scale api=3

# Scale multiple services
docker compose up -d --scale api=3 --scale worker=5
```

```yaml
# In compose file (deploy section)
services:
  api:
    image: myapi
    deploy:
      replicas: 3          # Start 3 instances
    # NOTE: Don't use container_name with replicas!
    # (each instance needs unique name)
```

```
  Scaling Services:
  
  --scale api=3
  
  +----------+  +----------+  +----------+
  | api-1    |  | api-2    |  | api-3    |
  | :3000    |  | :3000    |  | :3000    |
  +----+-----+  +----+-----+  +----+-----+
       |              |              |
       +--------------+--------------+
                      |
             myapp_default network
             DNS: "api" resolves to
             all 3 container IPs
             (round-robin)
  
  IMPORTANT: Don't map fixed host ports
  when scaling! Use a load balancer instead.
```

### Load Balancing Scaled Services

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - api

  api:
    build: ./api
    expose:
      - "3000"
    deploy:
      replicas: 3
```

```nginx
# nginx.conf
upstream api_backend {
    server api:3000;  # Docker DNS resolves to all replicas
}

server {
    listen 80;
    location / {
        proxy_pass http://api_backend;
    }
}
```

---

## 3.5 Profiles

```yaml
# Profiles let you selectively start services
services:
  web:
    image: nginx
    # No profile = always started
  
  api:
    image: myapi
    # No profile = always started
  
  db:
    image: postgres
    # No profile = always started

  adminer:
    image: adminer
    profiles:
      - debug               # Only with debug profile
    ports:
      - "8080:8080"

  mailhog:
    image: mailhog/mailhog
    profiles:
      - debug               # Only with debug profile
    ports:
      - "8025:8025"

  test-runner:
    image: myapp-test
    profiles:
      - test                 # Only with test profile
```

```bash
# Start without profiles (web, api, db only)
docker compose up -d

# Start with debug tools
docker compose --profile debug up -d

# Start multiple profiles
docker compose --profile debug --profile test up -d

# Set via environment variable
COMPOSE_PROFILES=debug docker compose up -d
```

---

## 3.6 Service Communication Patterns

```
  Microservices in Compose:
  
  External Traffic
       |
       v :80
  +---------+
  |  nginx  | (reverse proxy)
  +---------+
    |      |
    v      v
  +----+ +------+
  | web| | api  | ← Services talk by name
  +----+ +--+---+
             |
        +----+----+
        |         |
     +--+--+  +---+---+
     | db  |  | redis  |
     +-----+  +-------+
  
  Service names = DNS hostnames
  api connects to "db:5432"
  api connects to "redis:6379"
  
  No IP addresses needed!
```

```yaml
# Service communication example
services:
  api:
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/mydb
      REDIS_URL: redis://redis:6379
      # "db" and "redis" are resolved by Docker DNS
```

---

## 3.7 Init Containers and One-Off Services

```yaml
services:
  # Run database migrations before app starts
  migrate:
    build: ./api
    command: npm run migrate
    depends_on:
      db:
        condition: service_healthy
    restart: "no"                  # Run once and exit
    profiles:
      - setup

  # Seed data
  seed:
    build: ./api
    command: npm run seed
    depends_on:
      migrate:
        condition: service_completed_successfully
    restart: "no"
    profiles:
      - setup

  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
```

```bash
# Run one-off commands
docker compose run --rm api npm test
docker compose run --rm api npm run migrate

# Run setup tasks
docker compose --profile setup up migrate seed
```

---

## Summary Table

| Feature | Configuration | Purpose |
|---------|--------------|---------|
| **depends_on** | `condition: service_healthy` | Startup ordering |
| **healthcheck** | `test`, `interval`, `retries` | Service readiness |
| **restart** | `unless-stopped` | Auto-recovery |
| **replicas** | `deploy.replicas: 3` | Horizontal scaling |
| **profiles** | `profiles: [debug]` | Conditional services |
| **expose** | `expose: ["3000"]` | Internal-only ports |
| **container_name** | `container_name: myapi` | Fixed name (no scaling) |
| **init** | `init: true` | PID 1 signal handling |
| **run** | `docker compose run` | One-off commands |
| **scale** | `--scale api=3` | Runtime scaling |

---

## Quick Revision Questions

1. **What is the difference between `service_started` and `service_healthy` in depends_on?**
   - `service_started` only waits for the container to start. `service_healthy` waits for the healthcheck to pass, ensuring the service is actually ready

2. **Why can't you use `container_name` with replicas?**
   - Each container needs a unique name. With replicas, Compose generates names like `api-1`, `api-2`. A fixed `container_name` would conflict

3. **How do services communicate in Compose?**
   - By service name as DNS hostname. Docker's embedded DNS resolves service names to container IPs on the same network

4. **What are profiles used for?**
   - To define optional services that only start when the profile is activated with `--profile`. Useful for debug tools, test runners, or admin interfaces

5. **What restart policy is recommended for production?**
   - `unless-stopped` — restarts on crashes and after Docker daemon restarts, but respects manual `docker compose stop`

---

[← Previous: Compose File](02-compose-file.md) | [Next: Networking in Compose →](04-networking-in-compose.md)
