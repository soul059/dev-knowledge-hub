# Chapter 7: Container Cleanup and Management

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Docker resources (containers, images, volumes, networks) accumulate over time and can consume significant disk space. Regular cleanup is essential for maintaining a healthy Docker environment.

---

## 7.1 The Disk Space Problem

```
  Over time without cleanup:
  
  +------------------------------------------+
  |  Docker Disk Usage                        |
  |                                           |
  |  Stopped containers:      2.5 GB          |
  |  Dangling images:         8.0 GB          |
  |  Unused volumes:          5.0 GB          |
  |  Build cache:             12.0 GB         |
  |  ──────────────────────────────           |
  |  Total wasted:            27.5 GB!        |
  +------------------------------------------+
  
  docker system df
  TYPE            TOTAL    ACTIVE   SIZE      RECLAIMABLE
  Images          45       5        12.5GB    10.2GB (81%)
  Containers      30       3        2.5GB     2.3GB (92%)
  Local Volumes   15       3        5.0GB     4.2GB (84%)
  Build Cache     -        -        12.0GB    12.0GB
```

---

## 7.2 Checking Disk Usage

```bash
# Overview of Docker disk usage
docker system df

# Detailed view
docker system df -v

# Container sizes
docker ps -as
# Shows: SIZE column with writable layer size

# Image sizes
docker images
docker images -a    # Including intermediate layers

# Volume sizes
docker volume ls
docker system df -v | grep -A 100 "Local Volumes"
```

---

## 7.3 Container Cleanup

```bash
# Remove specific stopped container
docker rm mycontainer

# Force remove (even if running)
docker rm -f mycontainer

# Remove multiple containers
docker rm container1 container2 container3

# Remove all stopped containers
docker container prune
docker container prune -f      # No confirmation

# Remove all stopped containers (alternative)
docker rm $(docker ps -aq -f status=exited)

# Remove containers older than 24 hours
docker container prune --filter "until=24h"

# Auto-remove: run with --rm flag
docker run --rm ubuntu echo "temporary container"
```

---

## 7.4 Image Cleanup

```bash
# Remove specific image
docker rmi nginx:latest
docker image rm nginx:latest

# Remove dangling images (untagged, not used)
docker image prune
docker image prune -f

# Remove ALL unused images (not just dangling)
docker image prune -a
docker image prune -a -f

# Remove images older than 24h
docker image prune -a --filter "until=24h"

# Remove images matching pattern
docker images --format '{{.Repository}}:{{.Tag}}' | grep myapp | xargs docker rmi
```

```
  Dangling vs Unused:
  
  Dangling Images:              Unused Images:
  (no tag, no container)        (tagged but no container using them)
  
  <none>:<none>   ← dangling    nginx:1.23   ← unused (if no
  <none>:<none>                  python:3.10     container uses it)
  
  docker image prune            docker image prune -a
  (removes dangling only)       (removes ALL unused)
```

---

## 7.5 Volume Cleanup

```bash
# Remove specific volume
docker volume rm myvolume

# Remove all unused volumes
docker volume prune
docker volume prune -f

# List volumes
docker volume ls

# Find dangling volumes
docker volume ls -f dangling=true

# WARNING: Volume prune deletes DATA!
# Always verify before pruning volumes
docker volume ls -f dangling=true
# Then: docker volume prune
```

---

## 7.6 Network Cleanup

```bash
# Remove specific network
docker network rm mynetwork

# Remove all unused networks
docker network prune
docker network prune -f

# List networks
docker network ls
```

---

## 7.7 Build Cache Cleanup

```bash
# Remove build cache
docker builder prune

# Remove all build cache
docker builder prune -a

# Remove cache older than 24h
docker builder prune --filter "until=24h"

# Check build cache size
docker system df
```

---

## 7.8 System-Wide Cleanup

```bash
# Remove ALL unused data (containers, images, networks, cache)
docker system prune

# Including volumes (DANGEROUS — deletes data!)
docker system prune --volumes

# Remove everything, no confirmation
docker system prune -af --volumes

# With time filter
docker system prune --filter "until=48h"
```

```
  docker system prune removes:
  
  +---------------------------+
  | ✓ Stopped containers      |
  | ✓ Dangling images         |
  | ✓ Unused networks         |
  | ✓ Build cache             |
  +---------------------------+
  
  docker system prune --volumes ALSO removes:
  
  +---------------------------+
  | ✓ All of the above        |
  | ✓ Unused volumes (DATA!)  |
  +---------------------------+
  
  docker system prune -a ALSO removes:
  
  +---------------------------+
  | ✓ All of the above        |
  | ✓ ALL unused images       |
  |   (not just dangling)     |
  +---------------------------+
```

---

## 7.9 Automated Cleanup

### Cron Job (Linux)

```bash
# Daily cleanup at 3 AM
# /etc/cron.d/docker-cleanup
0 3 * * * root docker system prune -f --filter "until=48h" >> /var/log/docker-cleanup.log 2>&1
```

### Scheduled Task (Windows)

```powershell
# PowerShell script for scheduled cleanup
docker system prune -f --filter "until=48h"
docker image prune -a -f --filter "until=168h"  # 7 days
```

### Docker Compose Down

```bash
# Stop and remove containers, networks
docker compose down

# Also remove volumes
docker compose down -v

# Also remove images
docker compose down --rmi all

# Remove everything from the project
docker compose down -v --rmi all --remove-orphans
```

---

## 7.10 Cleanup Commands Reference

```
  +--------------------------------+--------------------------------------+
  | What to Clean                  | Command                              |
  +--------------------------------+--------------------------------------+
  | One stopped container          | docker rm <name>                     |
  | All stopped containers         | docker container prune               |
  | One image                      | docker rmi <image>                   |
  | Dangling images                | docker image prune                   |
  | All unused images              | docker image prune -a                |
  | One volume                     | docker volume rm <name>              |
  | Unused volumes                 | docker volume prune                  |
  | Unused networks                | docker network prune                 |
  | Build cache                    | docker builder prune                 |
  | EVERYTHING unused              | docker system prune                  |
  | EVERYTHING + volumes           | docker system prune --volumes        |
  | Nuclear option                 | docker system prune -af --volumes    |
  +--------------------------------+--------------------------------------+
```

---

## Summary Table

| Resource | List Command | Remove One | Remove Unused |
|----------|-------------|-----------|--------------|
| **Containers** | `docker ps -a` | `docker rm` | `docker container prune` |
| **Images** | `docker images -a` | `docker rmi` | `docker image prune -a` |
| **Volumes** | `docker volume ls` | `docker volume rm` | `docker volume prune` |
| **Networks** | `docker network ls` | `docker network rm` | `docker network prune` |
| **Build Cache** | `docker system df` | — | `docker builder prune` |
| **Everything** | `docker system df` | — | `docker system prune` |

---

## Quick Revision Questions

1. **What is a dangling image?**
   - A dangling image is an image with no tag (`<none>:<none>`) and not referenced by any container. They typically result from rebuilding images with the same tag

2. **What is the difference between `docker image prune` and `docker image prune -a`?**
   - `docker image prune` removes only dangling (untagged) images. `docker image prune -a` removes ALL images not used by any container, including tagged ones

3. **Why should you be careful with `docker system prune --volumes`?**
   - It deletes unused volumes, which contain persistent data (databases, uploads, etc.) that cannot be recovered

4. **How do you prevent accumulating stopped containers?**
   - Use the `--rm` flag when running temporary containers: `docker run --rm myapp`. This auto-removes the container on exit

5. **How do you check Docker's disk usage?**
   - `docker system df` shows a summary. `docker system df -v` shows detailed breakdown of images, containers, volumes, and cache

6. **What does `docker compose down -v --rmi all` do?**
   - Stops and removes all containers defined in the Compose file, removes their networks, removes associated named volumes, and removes all images used by the services

---

[← Previous: Resource Limits](06-resource-limits.md) | [Next Unit: Docker Networking →](../06-Docker-Networking/01-networking-overview.md)
