# Chapter 4: Networking in Compose

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

Docker Compose automatically creates a default network for your application and provides powerful networking features including multi-network isolation, DNS-based service discovery, and cross-project communication. This chapter covers network configuration patterns in Compose.

---

## 4.1 Default Network

```bash
# Compose creates a default bridge network automatically
# Named: <project>_default

docker compose up -d
docker network ls
# NETWORK ID    NAME              DRIVER
# abc123        myapp_default     bridge

# All services join the default network
# All services can reach each other by service name
```

```
  Default Network:
  
  compose.yaml (in myapp/ directory)
  services:
    web: ...
    api: ...
    db: ...
  
  Creates: myapp_default network
  
  +-------- myapp_default --------+
  |                                |
  |  web ←→ api ←→ db            |
  |  (all can communicate         |
  |   using service names)        |
  +--------------------------------+
  
  web → http://api:3000  ✓
  api → postgres://db:5432  ✓
```

---

## 4.2 Custom Networks

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"

  api:
    build: ./api
    networks:
      - frontend
      - backend

  db:
    image: postgres:16
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true       # No internet access
```

```
  Multi-Network Isolation:
  
  Internet
     |
     v :80
  +-------- frontend --------+
  |                           |
  | +-----+      +-----+    |
  | | web  |<--->| api  |    |
  | +-----+      +--+--+    |
  |                  |       |
  +---------+--------+------+
            |        |
  +---------+--------+------+
  |         |        |       |
  |      +--+--+  +--+---+  |
  |      | db  |  | redis | |
  |      +-----+  +------+  |
  |                          |
  +-------- backend ---------+
       (internal: true)
  
  web → api    ✓ (same frontend)
  api → db     ✓ (same backend)
  api → redis  ✓ (same backend)
  web → db     ✗ (different networks)
  db → internet ✗ (internal network)
```

---

## 4.3 Network Aliases and Links

```yaml
services:
  api:
    image: myapi
    networks:
      backend:
        aliases:
          - app-server
          - api-service
    # Reachable as: api, app-server, api-service

  db:
    image: postgres:16
    networks:
      backend:
        aliases:
          - database
          - postgres
    # Reachable as: db, database, postgres

networks:
  backend:
```

```
  DNS Resolution with Aliases:
  
  Container: api
  Networks: backend
  DNS names on backend:
    ├── api          (service name)
    ├── app-server   (alias)
    └── api-service  (alias)
  
  Any container on 'backend' can use
  ANY of these names to reach the api
```

---

## 4.4 Static IP Addresses

```yaml
services:
  api:
    image: myapi
    networks:
      mynet:
        ipv4_address: 172.20.0.10

  db:
    image: postgres
    networks:
      mynet:
        ipv4_address: 172.20.0.20

networks:
  mynet:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24
          gateway: 172.20.0.1
```

> **Note**: Static IPs are rarely needed. Prefer DNS-based service discovery.

---

## 4.5 External Networks (Cross-Project)

```yaml
# Project A: compose.yaml
services:
  api:
    image: myapi
    networks:
      - shared

networks:
  shared:
    name: shared-network        # Explicit name (no prefix)
```

```yaml
# Project B: compose.yaml  
services:
  frontend:
    image: frontend
    networks:
      - shared

networks:
  shared:
    external: true
    name: shared-network        # Use existing network
```

```bash
# Create shared network first
docker network create shared-network

# Then start both projects
cd project-a && docker compose up -d
cd project-b && docker compose up -d
```

```
  Cross-Project Communication:
  
  Project A                    Project B
  +------------------+        +------------------+
  | api service      |        | frontend service |
  +--------+---------+        +--------+---------+
           |                           |
  +--------+---------------------------+---------+
  |              shared-network                   |
  |         (external, pre-created)               |
  +-----------------------------------------------+
  
  frontend → http://api:3000  ✓
  (services from different projects
   communicate on the shared network)
```

---

## 4.6 Host and None Networking

```yaml
services:
  # Host networking (no isolation)
  monitoring:
    image: prometheus
    network_mode: host        # Use host's network directly
  
  # No networking
  batch-job:
    image: processor
    network_mode: none        # Completely isolated
  
  # Share another container's network
  debug:
    image: nicolaka/netshoot
    network_mode: "service:api"  # Share api's network
    profiles:
      - debug
```

---

## 4.7 DNS and Port Configuration

```yaml
services:
  api:
    image: myapi
    ports:
      # HOST:CONTAINER
      - "3000:3000"               # All interfaces
      - "127.0.0.1:3000:3000"     # Localhost only
      - "8080-8090:8080-8090"     # Port range
      - target: 3000              # Long syntax
        published: "3000"
        protocol: tcp
        mode: host
    
    expose:
      - "3000"                    # Internal only
    
    dns:
      - 8.8.8.8
      - 8.8.4.4
    
    dns_search:
      - mycompany.local
    
    extra_hosts:
      - "myhost:192.168.1.100"
      - "host.docker.internal:host-gateway"
```

---

## Summary Table

| Feature | Configuration | Purpose |
|---------|--------------|---------|
| **Default network** | Auto-created `<project>_default` | All services connected |
| **Custom networks** | `networks:` section | Service isolation |
| **Internal network** | `internal: true` | Block internet access |
| **Network aliases** | `aliases: [name]` | Additional DNS names |
| **Static IP** | `ipv4_address` + IPAM | Fixed addressing |
| **External network** | `external: true` | Cross-project sharing |
| **Host mode** | `network_mode: host` | No network isolation |
| **Service mode** | `network_mode: "service:X"` | Share network namespace |
| **DNS config** | `dns:`, `dns_search:` | Custom DNS servers |
| **Extra hosts** | `extra_hosts:` | Custom /etc/hosts entries |

---

## Quick Revision Questions

1. **What network does Compose create by default?**
   - A bridge network named `<projectname>_default` — all services are automatically connected to it

2. **How do you isolate frontend and backend services on different networks?**
   - Define two networks (e.g., `frontend`, `backend`), assign web to `frontend`, db to `backend`, and the API to both

3. **How can two separate Compose projects communicate?**
   - Create a shared external network (`docker network create shared`), then reference it with `external: true` in both compose files

4. **What is `network_mode: "service:api"` used for?**
   - It makes a container share the network namespace of the `api` service — same IP, same interfaces. Useful for debugging with tools like netshoot

5. **Why should you use `internal: true` on a backend network?**
   - It prevents containers on that network from accessing the internet, adding security for services like databases that should only be reachable internally

---

[← Previous: Services](03-services.md) | [Next: Volumes in Compose →](05-volumes-in-compose.md)
