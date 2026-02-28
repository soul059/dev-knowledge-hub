# Chapter 1: Compose Overview

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

Docker Compose is a tool for defining and running multi-container applications. Using a YAML file, you describe all the services, networks, and volumes your application needs, then bring the entire stack up with a single command.

---

## 1.1 What is Docker Compose?

```
  Without Compose:                 With Compose:
  
  $ docker network create mynet    $ docker compose up -d
  $ docker volume create dbdata    
  $ docker run -d \                That's it! 
    --name db \                    All containers, networks,
    --network mynet \              and volumes created from
    -v dbdata:/var/lib/... \       a single YAML file.
    -e POSTGRES_PASS=... \         
    postgres                       
  $ docker run -d \                
    --name api \                   
    --network mynet \              
    -p 8080:8080 \               
    -e DB_HOST=db \               
    myapi                          
  $ docker run -d ...              
  ...repeat for each service       
```

```
  Docker Compose Architecture:
  
  compose.yaml
  +---------------------------+
  | services:                 |
  |   web:    (nginx)         |
  |   api:    (node app)      |     docker compose up
  |   db:     (postgres)      | ──────────────────────>
  | networks:                 |
  |   frontend:               |
  |   backend:                |
  | volumes:                  |
  |   dbdata:                 |
  +---------------------------+
  
  Creates:
  +----------+  +----------+  +----------+
  |   web    |  |   api    |  |   db     |
  |  :80     |  |  :3000   |  |  :5432   |
  +----+-----+  +----+-----+  +----+-----+
       |              |              |
  frontend net   both nets     backend net
                      |              |
                 +----+--------------+----+
                 |        dbdata          |
                 +------------------------+
```

---

## 1.2 Compose V1 vs V2

```
  Evolution:
  
  Compose V1 (Legacy)           Compose V2 (Current)
  +---------------------+      +---------------------+
  | docker-compose      |      | docker compose      |
  | (separate binary)   |      | (Docker CLI plugin)  |
  | Python-based        |      | Go-based            |
  | Slower              |      | Faster              |
  | docker-compose.yml  |      | compose.yaml        |
  +---------------------+      +---------------------+
  
  V1 is DEPRECATED since July 2023
  Always use V2: docker compose (space, no hyphen)
```

```bash
# V2 commands (use these)
docker compose up
docker compose down
docker compose ps

# V1 commands (deprecated)
docker-compose up    # Don't use!

# Check version
docker compose version
# Docker Compose version v2.x.x
```

---

## 1.3 Compose File Basics

```yaml
# compose.yaml (also: compose.yml, docker-compose.yml)
name: myproject

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db

  db:
    image: postgres:16
    volumes:
      - dbdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret

volumes:
  dbdata:

networks:
  default:
    driver: bridge
```

---

## 1.4 Essential Commands

```bash
# Start all services (detached)
docker compose up -d

# Start with build
docker compose up -d --build

# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v

# View running services
docker compose ps

# View logs
docker compose logs
docker compose logs -f api    # Follow specific service

# Execute command in service
docker compose exec api sh

# Scale a service
docker compose up -d --scale api=3

# Restart a service
docker compose restart api

# Pull latest images
docker compose pull

# Build images
docker compose build

# View configuration (merged & validated)
docker compose config
```

---

## 1.5 Project Name and File Discovery

```bash
# Compose project name (prefixes all resources)
# Default: directory name

# Specify project name
docker compose -p myproject up -d

# Or set in compose.yaml
# name: myproject

# Or via environment variable
COMPOSE_PROJECT_NAME=myproject docker compose up -d

# File discovery order:
# 1. compose.yaml
# 2. compose.yml
# 3. docker-compose.yaml
# 4. docker-compose.yml

# Specify file explicitly
docker compose -f custom-compose.yaml up -d

# Multiple compose files (merged)
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

```
  Resource Naming Convention:
  
  Project: "myapp" (from directory name or -p flag)
  
  Containers:  myapp-web-1, myapp-api-1, myapp-db-1
  Networks:    myapp_default
  Volumes:     myapp_dbdata
  
  +----- myapp_default network -----+
  |                                  |
  | myapp-web-1  myapp-api-1        |
  | myapp-db-1                      |
  +----------------------------------+
```

---

## 1.6 How Compose Works

```
  Compose Workflow:
  
  compose.yaml
       |
       v
  Parse & Validate
       |
       v
  Create Networks  ──>  myapp_default, myapp_backend
       |
       v
  Create Volumes   ──>  myapp_dbdata
       |
       v
  Pull/Build Images ──> postgres:16, build ./api
       |
       v
  Create & Start    ──> myapp-db-1 (first: no deps)
  Containers            myapp-api-1 (after: db)
  (respecting           myapp-web-1 (after: api)
   depends_on)
  
  docker compose down reverses this order:
  Stop web → Stop api → Stop db
  Remove containers → Remove networks
  (Volumes kept unless -v flag used)
```

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `docker compose up -d` | Start all services in background |
| `docker compose down` | Stop and remove containers/networks |
| `docker compose down -v` | Also remove volumes |
| `docker compose ps` | List running services |
| `docker compose logs -f` | Follow logs |
| `docker compose exec <svc> sh` | Shell into service |
| `docker compose build` | Build/rebuild images |
| `docker compose pull` | Pull latest images |
| `docker compose config` | Validate and view merged config |
| `docker compose --scale` | Scale service instances |

---

## Quick Revision Questions

1. **What problem does Docker Compose solve?**
   - It simplifies multi-container application deployment by defining all services, networks, and volumes in a single YAML file, replacing multiple docker run commands

2. **What is the difference between Compose V1 and V2?**
   - V1 is a separate Python binary (`docker-compose`), V2 is a Go-based Docker CLI plugin (`docker compose`). V1 is deprecated; always use V2

3. **What is the default compose file name?**
   - Docker Compose looks for `compose.yaml`, `compose.yml`, `docker-compose.yaml`, or `docker-compose.yml` in that order

4. **How does Compose name its resources?**
   - Resources are prefixed with the project name (default: directory name): `projectname-service-1` for containers, `projectname_network` for networks

5. **What does `docker compose down -v` do?**
   - Stops and removes all containers, networks, AND named volumes defined in the compose file

6. **How do you view the merged configuration from multiple compose files?**
   - Run `docker compose config` to see the validated, merged YAML output

---

[← Previous Unit: Docker Volumes](../07-Docker-Volumes/05-storage-drivers.md) | [Next: Compose File →](02-compose-file.md)
