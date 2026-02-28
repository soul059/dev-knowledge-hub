# Chapter 2: Compose File

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

The Compose file is a YAML file that defines the complete application stack — services, networks, volumes, and configurations. Understanding the file structure and available options is essential for effective multi-container orchestration.

---

## 2.1 File Structure

```yaml
# Top-level elements of a compose file

name: myproject           # Project name (optional)

services:                 # Container definitions (REQUIRED)
  web: ...
  api: ...
  db: ...

networks:                 # Custom networks (optional)
  frontend: ...
  backend: ...

volumes:                  # Named volumes (optional)
  dbdata: ...
  uploads: ...

configs:                  # Configuration files (optional)
  nginx_conf: ...

secrets:                  # Sensitive data (optional)
  db_password: ...
```

```
  Compose File Hierarchy:
  
  compose.yaml
  +------------------------------------------+
  | name: myproject                           |
  |                                           |
  | services:         ← What to run          |
  |   +-- image / build                      |
  |   +-- ports / expose                     |
  |   +-- environment / env_file             |
  |   +-- volumes / networks                 |
  |   +-- depends_on / healthcheck           |
  |   +-- deploy / restart                   |
  |                                           |
  | networks:         ← How they connect     |
  |   +-- driver / ipam                      |
  |   +-- internal / external                |
  |                                           |
  | volumes:          ← Where data lives     |
  |   +-- driver / driver_opts               |
  |   +-- external                           |
  +------------------------------------------+
```

---

## 2.2 Service Configuration

```yaml
services:
  api:
    # Image or build
    image: node:20-alpine           # Use existing image
    # OR
    build:                          # Build from Dockerfile
      context: ./api
      dockerfile: Dockerfile.prod
      args:
        NODE_ENV: production

    # Container settings
    container_name: my-api          # Fixed name (no scaling)
    hostname: api-server
    restart: unless-stopped         # Restart policy
    
    # Port mapping
    ports:
      - "3000:3000"                 # host:container
      - "127.0.0.1:9229:9229"      # localhost only
    expose:
      - "3000"                      # Internal only (no host)
    
    # Environment
    environment:
      NODE_ENV: production
      DB_HOST: db
    env_file:
      - .env
      - .env.production
    
    # Volumes
    volumes:
      - ./src:/app/src              # Bind mount
      - node_modules:/app/node_modules  # Named volume
      - /app/tmp                    # Anonymous volume
    
    # Networking
    networks:
      - frontend
      - backend
    
    # Resource limits
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 256M
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Dependencies
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # Command override
    command: ["node", "server.js"]
    entrypoint: ["/docker-entrypoint.sh"]
    working_dir: /app
    user: "1000:1000"
```

---

## 2.3 Build Configuration

```yaml
services:
  app:
    # Simple build
    build: ./app

    # Detailed build configuration
    build:
      context: ./app                    # Build context path
      dockerfile: Dockerfile.prod       # Custom Dockerfile name
      args:                             # Build arguments
        NODE_ENV: production
        APP_VERSION: "2.0"
      target: production                # Multi-stage target
      cache_from:                       # Cache sources
        - myapp:latest
      labels:                           # Image labels
        com.example.version: "2.0"
      platforms:                        # Multi-platform
        - linux/amd64
        - linux/arm64
      ssh:                              # SSH agent forwarding
        - default
```

```
  Build vs Image:
  
  image: nginx:alpine     build: ./api
  +------------------+    +------------------+
  | Pull from        |    | Build from       |
  | registry         |    | local Dockerfile |
  +------------------+    +------------------+
  
  Both together:
  build: ./api
  image: myregistry/api:latest
  +------------------+
  | Build from local |
  | Dockerfile AND   |
  | tag the result   |
  | as the image name|
  +------------------+
```

---

## 2.4 Network Configuration

```yaml
services:
  web:
    networks:
      - frontend
  api:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true          # No internet access

  # Use existing network
  existing_net:
    external: true
    name: my-existing-network

  # Custom IPAM
  custom:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
```

```
  Network Isolation in Compose:
  
  +------ frontend ------+
  |                       |
  |  +-----+   +-----+   |
  |  | web |   | api |   |
  |  +-----+   +--+--+   |
  |                |      |
  +----------------+------+
                   |
  +----------------+------+
  |                |      |
  |             +--+--+   |
  |             | db  |   |
  |             +-----+   |
  |                       |
  +------ backend --------+
         (internal)
  
  web → api ✓ (frontend)
  api → db  ✓ (backend)
  web → db  ✗ (different networks)
  db → internet ✗ (internal network)
```

---

## 2.5 Volume Configuration

```yaml
services:
  db:
    volumes:
      - dbdata:/var/lib/postgresql/data    # Named volume
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro  # Bind mount
      - type: tmpfs                         # tmpfs mount
        target: /tmp
        tmpfs:
          size: 100000000                   # 100MB

volumes:
  dbdata:
    driver: local

  # Volume with driver options
  nfs_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=nfs-server,rw
      device: ":/exports/data"

  # Use existing volume
  existing_vol:
    external: true
    name: my-existing-volume
```

---

## 2.6 Multiple Compose Files (Override Pattern)

```yaml
# compose.yaml (base)
services:
  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db

# compose.override.yaml (auto-loaded for development)
services:
  api:
    volumes:
      - ./api/src:/app/src        # Live reload
    environment:
      DEBUG: "true"
    command: npm run dev

# compose.prod.yaml (production overrides)
services:
  api:
    image: registry.example.com/api:latest
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
```

```bash
# Development (auto-loads compose.override.yaml)
docker compose up -d

# Production (explicit override file)
docker compose -f compose.yaml -f compose.prod.yaml up -d

# Verify merged configuration
docker compose -f compose.yaml -f compose.prod.yaml config
```

```
  File Merge Order:
  
  compose.yaml          compose.prod.yaml     Merged Result
  +---------------+     +---------------+     +---------------+
  | build: ./api  |     | image: reg/.. | --> | image: reg/.. |
  | ports: 3000   |  +  | restart: alw  | =   | ports: 3000   |
  | DB_HOST: db   |     | replicas: 3   |     | restart: alw  |
  +---------------+     +---------------+     | replicas: 3   |
                                              | DB_HOST: db   |
  Later files override                        +---------------+
  earlier values
```

---

## 2.7 YAML Anchors and Extensions

```yaml
# Use YAML anchors to avoid repetition

# Extension fields (x- prefix, ignored by Compose)
x-common: &common
  restart: unless-stopped
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

x-env: &default-env
  TZ: UTC
  LOG_LEVEL: info

services:
  api:
    <<: *common                    # Merge common settings
    image: myapi:latest
    environment:
      <<: *default-env
      PORT: "3000"

  worker:
    <<: *common                    # Same common settings
    image: myworker:latest
    environment:
      <<: *default-env
      QUEUE: "default"
```

---

## Summary Table

| Section | Purpose | Key Options |
|---------|---------|-------------|
| **services** | Define containers | image, build, ports, environment, volumes |
| **networks** | Define networks | driver, internal, external, ipam |
| **volumes** | Define volumes | driver, external, driver_opts |
| **configs** | Config files | file, external |
| **secrets** | Sensitive data | file, external |
| **build** | Build images | context, dockerfile, args, target |
| **deploy** | Resource constraints | replicas, resources, restart_policy |
| **healthcheck** | Health monitoring | test, interval, timeout, retries |
| **depends_on** | Start ordering | service_started, service_healthy |
| **x-** extensions | Reusable fragments | YAML anchors with `<<: *name` |

---

## Quick Revision Questions

1. **What are the top-level elements in a compose file?**
   - `services` (required), `networks`, `volumes`, `configs`, `secrets`, and `name`

2. **How do you use multiple compose files?**
   - Use `-f` flags: `docker compose -f compose.yaml -f compose.prod.yaml up`. Later files override values from earlier files. `compose.override.yaml` is auto-loaded

3. **What is the difference between `ports` and `expose`?**
   - `ports` publishes ports to the host (accessible externally). `expose` only makes ports available to other containers on the same network

4. **How do you prevent a service from accessing the internet?**
   - Place it on a network with `internal: true`, which blocks outbound internet traffic while allowing inter-container communication

5. **What are YAML anchors and how do they help?**
   - Anchors (`&name`) and aliases (`*name`) with merge keys (`<<:`) let you define reusable configuration blocks, reducing duplication across services

6. **What does `depends_on` with `condition: service_healthy` do?**
   - It delays starting the dependent service until the dependency's healthcheck reports healthy, ensuring the dependency is truly ready (not just started)

---

[← Previous: Compose Overview](01-compose-overview.md) | [Next: Services →](03-services.md)
