# Chapter 5: Docker Objects

> **Unit 2: Docker Architecture** | [Course Home](../README.md)

---

## Overview

Docker works with several core **objects** — images, containers, networks, and volumes. These are the building blocks of any Docker deployment. Understanding how they relate and interact is key to architecting containerized applications.

---

## 5.1 The Four Core Docker Objects

```
+-------------------------------------------------------------------+
|                     Docker Objects                                  |
|                                                                    |
|  +------------------+          +------------------+                |
|  |     IMAGES       |  creates |   CONTAINERS     |                |
|  |                  | -------> |                  |                |
|  | Read-only        |          | Running instances |                |
|  | templates        |          | of images        |                |
|  +------------------+          +--------+---------+                |
|                                         |                          |
|                                connects to                        |
|                                         |                          |
|  +------------------+          +--------v---------+                |
|  |    VOLUMES       |          |    NETWORKS      |                |
|  |                  |          |                  |                |
|  | Persistent       |          | Communication   |                |
|  | data storage     |          | between          |                |
|  |                  |          | containers       |                |
|  +------------------+          +------------------+                |
+-------------------------------------------------------------------+
```

---

## 5.2 Images

An image is a **read-only template** containing the filesystem and configuration needed to create a container.

```
  IMAGE STRUCTURE:
  
  +---------------------------------------+
  |  Image: myapp:v1                      |
  |                                       |
  |  Metadata:                            |
  |  - Created: 2024-01-15               |
  |  - Author: developer                 |
  |  - Architecture: amd64               |
  |  - OS: linux                         |
  |                                       |
  |  Configuration:                       |
  |  - Env: NODE_ENV=production          |
  |  - Expose: 3000                      |
  |  - Cmd: ["node", "server.js"]        |
  |  - WorkDir: /app                     |
  |                                       |
  |  Layers (read-only):                  |
  |  +----------------------------------+ |
  |  | Layer 5: COPY . /app             | |
  |  +----------------------------------+ |
  |  | Layer 4: RUN npm install         | |
  |  +----------------------------------+ |
  |  | Layer 3: COPY package.json       | |
  |  +----------------------------------+ |
  |  | Layer 2: RUN apt-get install     | |
  |  +----------------------------------+ |
  |  | Layer 1: node:18-slim base       | |
  |  +----------------------------------+ |
  +---------------------------------------+
```

### Key Image Properties

| Property | Description |
|----------|-------------|
| **Immutable** | Once built, an image cannot be changed |
| **Layered** | Built from stacked read-only layers |
| **Shareable** | Base layers shared across images/containers |
| **Identifiable** | Identified by name:tag or SHA256 digest |
| **Portable** | Same image runs anywhere Docker is installed |

```bash
# Essential image commands
docker images                         # List images
docker pull nginx:1.25               # Pull specific version
docker image inspect nginx           # View image details
docker image history nginx           # View layer history
docker rmi nginx                     # Remove image
docker image prune                   # Remove unused images
```

---

## 5.3 Containers

A container is a **running (or stopped) instance** of an image, with its own writable layer.

```
  CONTAINER = IMAGE + WRITABLE LAYER + CONFIG
  
  +------------------------------------------+
  |  Container: "web-server"                  |
  |  State: Running                           |
  |  PID: 12345 (on host)                    |
  |                                           |
  |  +--------------------------------------+ |
  |  | Writable Layer (container layer)     | |
  |  | - Log files being written            | |
  |  | - Temp files                          | |
  |  | - Runtime data                        | |
  |  +--------------------------------------+ |
  |  | Image Layer 5 (read-only)            | |
  |  +--------------------------------------+ |
  |  | Image Layer 4 (read-only)            | |
  |  +--------------------------------------+ |
  |  | Image Layer 3 (read-only)            | |
  |  +--------------------------------------+ |
  |  | Image Layer 2 (read-only)            | |
  |  +--------------------------------------+ |
  |  | Image Layer 1 (read-only)            | |
  |  +--------------------------------------+ |
  |                                           |
  |  Network: bridge (172.17.0.2)            |
  |  Ports: 80/tcp -> 0.0.0.0:8080          |
  |  Volumes: /data -> myvolume             |
  +------------------------------------------+
```

### Container States

```
  Created ──> Running ──> Paused
     |           |           |
     |           v           v
     |        Stopped    Running (unpause)
     |           |
     v           v
   Removed    Removed
  
  docker create  -> Created
  docker start   -> Running
  docker pause   -> Paused
  docker unpause -> Running
  docker stop    -> Stopped (Exited)
  docker kill    -> Stopped (Exited)
  docker rm      -> Removed
```

### Multiple Containers from One Image

```
  One Image, Many Containers:
  
           nginx:latest (image)
           +-------------+
           |  read-only  |
           |   layers    |
           +------+------+
                  |
     +------------+------------+
     |            |            |
  +--v--+     +--v--+     +--v--+
  |web-1|     |web-2|     |web-3|
  |write|     |write|     |write|   <- Each has own writable layer
  |layer|     |layer|     |layer|
  +-----+     +-----+     +-----+
  
  All three share the same read-only base layers!
  Extremely storage-efficient.
```

---

## 5.4 Networks

Networks enable **communication** between containers and between containers and the outside world.

```
  Docker Network Types:
  
  1. BRIDGE (default)
  +--------------------------------------------------+
  |  Host                                             |
  |  +---------------------------------------------+ |
  |  | docker0 bridge (172.17.0.0/16)              | |
  |  |                                              | |
  |  |  +-------+  +-------+  +-------+            | |
  |  |  |  C1   |  |  C2   |  |  C3   |            | |
  |  |  |.0.2   |  |.0.3   |  |.0.4   |            | |
  |  |  +-------+  +-------+  +-------+            | |
  |  +---------------------------------------------+ |
  |                                                   |
  |  eth0 (host network interface)                    |
  +--------------------------------------------------+
  
  2. HOST
  +--------------------------------------------------+
  |  Host                                             |
  |  Container shares host's network stack            |
  |  No network isolation, but best performance       |
  +--------------------------------------------------+
  
  3. NONE
  +--------------------------------------------------+
  |  Container has no network connectivity            |
  |  Used for isolated batch processing               |
  +--------------------------------------------------+
  
  4. OVERLAY
  +--------------------------------------------------+
  |  Spans multiple Docker hosts (Swarm/K8s)          |
  |  Containers on different hosts can communicate    |
  +--------------------------------------------------+
```

```bash
# Essential network commands
docker network ls                     # List networks
docker network create mynetwork       # Create network
docker network inspect mynetwork      # View details
docker network connect mynetwork c1   # Connect container
docker network disconnect mynet c1    # Disconnect
docker network rm mynetwork           # Remove network
docker network prune                  # Remove unused
```

---

## 5.5 Volumes

Volumes provide **persistent data storage** that survives container restarts and removal.

```
  WITHOUT VOLUMES:                     WITH VOLUMES:
  
  +----------+                         +----------+
  | Container|                         | Container|
  | +------+ |                         | +------+ |
  | | Data | |  Container deleted      | | Data |-+---> Volume
  | +------+ |  = DATA LOST!          | +------+ |     (persistent)
  +----------+                         +----------+
                                       Container deleted
                                       = Data SAFE in volume!
```

### Volume Types

```
  1. Named Volume (managed by Docker)
  +----------+     +---------------------------+
  | Container| --> | /var/lib/docker/volumes/   |
  | /app/data|     | myvolume/_data/            |
  +----------+     +---------------------------+
  
  2. Bind Mount (host directory)
  +----------+     +---------------------------+
  | Container| --> | /home/user/project/data/   |
  | /app/data|     | (any host path)            |
  +----------+     +---------------------------+
  
  3. tmpfs Mount (memory only)
  +----------+     +---------------------------+
  | Container| --> | RAM (not written to disk)  |
  | /app/tmp |     | Lost when container stops  |
  +----------+     +---------------------------+
```

```bash
# Essential volume commands
docker volume create mydata           # Create volume
docker volume ls                      # List volumes
docker volume inspect mydata          # View details
docker volume rm mydata               # Remove volume
docker volume prune                   # Remove unused

# Using volumes
docker run -v mydata:/app/data nginx             # Named volume
docker run -v /host/path:/container/path nginx    # Bind mount
docker run --tmpfs /app/tmp nginx                 # tmpfs
```

---

## 5.6 How Objects Interact

```
  REAL-WORLD EXAMPLE: Web Application Stack
  
  +------------------------------------------------------------+
  |  Docker Host                                                |
  |                                                             |
  |  Network: "app-network" (bridge)                            |
  |  +--------------------------------------------------------+|
  |  |                                                         ||
  |  |  +-------------+    +-------------+    +-------------+  ||
  |  |  |   nginx     |    |   node-api  |    |  postgres   |  ||
  |  |  |  Container  |--->|  Container  |--->|  Container  |  ||
  |  |  |  Port 80    |    |  Port 3000  |    |  Port 5432  |  ||
  |  |  +------+------+    +------+------+    +------+------+  ||
  |  |         |                  |                  |         ||
  |  +--------------------------------------------------------+|
  |            |                  |                  |           |
  |  +---------v----+   +--------v------+   +-------v--------+  |
  |  | Volume:      |   | Volume:       |   | Volume:        |  |
  |  | nginx-config |   | app-uploads   |   | pg-data        |  |
  |  +--------------+   +---------------+   +----------------+  |
  |                                                             |
  |  Port Mapping: Host:80 -> nginx:80                          |
  +------------------------------------------------------------+
```

### Creating This Setup

```bash
# Create network
docker network create app-network

# Create volumes
docker volume create nginx-config
docker volume create app-uploads
docker volume create pg-data

# Run containers
docker run -d --name postgres \
  --network app-network \
  -v pg-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

docker run -d --name node-api \
  --network app-network \
  -v app-uploads:/app/uploads \
  -e DATABASE_URL=postgres://postgres:secret@postgres:5432/mydb \
  myapp:latest

docker run -d --name nginx \
  --network app-network \
  -v nginx-config:/etc/nginx/conf.d \
  -p 80:80 \
  nginx:latest
```

---

## 5.7 Object Identification

| Object | Identified By | Example |
|--------|--------------|---------|
| Image | name:tag | `nginx:1.25-alpine` |
| Image | digest | `nginx@sha256:abc123...` |
| Container | name | `web-server` |
| Container | ID (short) | `a1b2c3d4e5f6` |
| Container | ID (full) | `a1b2c3d4e5f6...` (64 hex chars) |
| Network | name | `app-network` |
| Volume | name | `pg-data` |

---

## Summary Table

| Object | Purpose | Persistence | Key Commands |
|--------|---------|-------------|-------------|
| **Image** | Read-only template for containers | Persists until deleted | `pull`, `build`, `push`, `rmi` |
| **Container** | Running instance of an image | Temporary (writable layer lost on delete) | `run`, `start`, `stop`, `rm` |
| **Network** | Communication channel | Persists until deleted | `network create`, `connect` |
| **Volume** | Persistent data storage | Persists independently of containers | `volume create`, `-v` flag |

---

## Quick Revision Questions

1. **What is the relationship between a Docker image and a container?**
   - An image is a read-only template; a container is a running instance of an image with an added writable layer. One image can create many containers

2. **What happens to data written inside a container when the container is deleted?**
   - Data in the container's writable layer is lost. Only data written to volumes persists after container deletion

3. **Name the four types of Docker network drivers.**
   - bridge (default, isolated network), host (shares host network), none (no networking), overlay (multi-host networking)

4. **What is the difference between a named volume and a bind mount?**
   - Named volumes are managed by Docker (stored in `/var/lib/docker/volumes/`). Bind mounts map a specific host directory into the container

5. **How do multiple containers from the same image share storage efficiently?**
   - All containers share the same read-only image layers. Each container only adds a thin writable layer on top, saving disk space

6. **How do containers on the same Docker network communicate?**
   - Containers on the same user-defined network can reach each other by container name (Docker's built-in DNS) or by IP address

---

[← Previous: containerd and runc](04-containerd-and-runc.md) | [Next Unit: Docker Images →](../03-Docker-Images/01-image-concepts.md)
