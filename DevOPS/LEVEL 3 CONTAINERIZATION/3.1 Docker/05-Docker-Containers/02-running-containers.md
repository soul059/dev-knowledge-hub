# Chapter 2: Running Containers

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

The `docker run` command is the most-used Docker command. It combines image pulling, container creation, and container starting into a single operation. Mastering its flags and options is essential for effective container management.

---

## 2.1 docker run Anatomy

```
  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  
  docker run -d -p 8080:80 --name web nginx:alpine nginx -g "daemon off;"
             |   |          |           |           |
             |   |          |           |           +-- Override CMD
             |   |          |           +-- Image
             |   |          +-- Container name
             |   +-- Port mapping
             +-- Detached mode
```

---

## 2.2 Foreground vs Detached Mode

```bash
# Foreground (default) — attached to terminal
docker run nginx
# Ctrl+C to stop

# Interactive mode — with terminal input
docker run -it ubuntu bash
# -i: Keep STDIN open
# -t: Allocate pseudo-TTY

# Detached mode — runs in background
docker run -d nginx
# Returns container ID, runs in background

# Attach to running container
docker attach <container_id>
# Ctrl+P, Ctrl+Q to detach without stopping
```

```
  Foreground:                  Detached:
  
  Terminal                     Terminal
  +----------+                 +-----------+
  | $ docker |                 | $ docker  |
  | run nginx|                 | run -d    |
  |          |                 | nginx     |
  | (logs    |                 |           |
  |  stream  |                 | abc123... |  (ID returned)
  |  here)   |                 | $         |  (prompt back)
  | ^C stop  |                 +-----------+
  +----------+
```

---

## 2.3 Port Mapping

```bash
# Map host port to container port
docker run -d -p 8080:80 nginx
#                ^^^^:^^
#                host:container

# Map to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Random host port
docker run -d -p 80 nginx          # Random port → 80
docker run -d -P nginx              # Map ALL exposed ports

# Multiple port mappings
docker run -d -p 80:80 -p 443:443 nginx

# UDP port
docker run -d -p 53:53/udp dns-server

# Check mapped ports
docker port mycontainer
```

```
  Port Mapping:
  
  Host Machine                    Container
  +-------------------+          +-----------------+
  |                   |          |                 |
  | Port 8080 --------+-----------> Port 80       |
  |                   |          |    (nginx)      |
  | Port 8443 --------+-----------> Port 443      |
  |                   |          |                 |
  +-------------------+          +-----------------+
  
  Browser: http://localhost:8080  -->  nginx:80
```

---

## 2.4 Environment Variables

```bash
# Set environment variable
docker run -e NODE_ENV=production myapp
docker run -e DB_HOST=db.example.com myapp

# Pass host variable (no value = use host's value)
docker run -e HOME myapp

# Multiple variables
docker run \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  -e DB_USER=admin \
  myapp

# Environment file
docker run --env-file .env myapp

# .env file format:
# DB_HOST=localhost
# DB_PORT=5432
# # Comments are supported
# DB_USER=admin
```

---

## 2.5 Naming and Labeling

```bash
# Name container
docker run -d --name webserver nginx

# Labels
docker run -d \
  --label env=production \
  --label team=backend \
  nginx

# Filter by label
docker ps --filter label=env=production
```

---

## 2.6 Resource Constraints

```bash
# Memory limits
docker run -d --memory=512m nginx             # Hard limit
docker run -d --memory=512m --memory-swap=1g nginx  # Memory + swap
docker run -d --memory-reservation=256m nginx  # Soft limit

# CPU limits
docker run -d --cpus=1.5 myapp               # 1.5 CPU cores
docker run -d --cpu-shares=512 myapp          # Relative weight
docker run -d --cpuset-cpus="0,1" myapp       # Pin to CPUs 0,1

# Combined
docker run -d \
  --memory=512m \
  --cpus=1.0 \
  --name limited-app \
  myapp
```

```
  Resource Limits:
  
  +-------------------------------------------+
  |  Host System (8 CPU, 16GB RAM)            |
  |                                           |
  |  +------------------+  +----------------+ |
  |  | Container A      |  | Container B    | |
  |  | --memory=512m    |  | --memory=1g    | |
  |  | --cpus=1.0       |  | --cpus=2.0     | |
  |  +------------------+  +----------------+ |
  |                                           |
  |  Remaining: 5 CPU, 14.5GB available      |
  +-------------------------------------------+
```

---

## 2.7 Networking Options

```bash
# Connect to specific network
docker run -d --network mynet nginx

# Host networking (container uses host's network stack)
docker run -d --network host nginx

# No networking
docker run -d --network none myapp

# DNS settings
docker run -d --dns 8.8.8.8 myapp
docker run -d --hostname myhost myapp

# Add host entry
docker run -d --add-host db:192.168.1.100 myapp
```

---

## 2.8 Volume and Bind Mounts

```bash
# Named volume
docker run -d -v mydata:/data myapp

# Bind mount (host directory)
docker run -d -v /host/path:/container/path myapp
docker run -d -v $(pwd)/src:/app/src myapp

# Read-only mount
docker run -d -v /config:/app/config:ro myapp

# tmpfs mount (in-memory)
docker run -d --tmpfs /tmp:rw,size=100m myapp

# Modern --mount syntax
docker run -d \
  --mount type=bind,source=/host/data,target=/data \
  --mount type=volume,source=logs,target=/app/logs \
  myapp
```

---

## 2.9 Cleanup Options

```bash
# Auto-remove container when it exits
docker run --rm ubuntu echo "one-off task"

# Auto-remove is great for:
# - Testing
# - One-off commands
# - CI/CD build steps
# - Debugging
docker run --rm -it node:20 node -e "console.log(process.version)"
```

---

## 2.10 Advanced Run Options

```bash
# Override entrypoint
docker run --entrypoint /bin/sh myapp

# Working directory
docker run -w /app myapp npm test

# User
docker run -u 1000:1000 myapp
docker run -u nobody myapp

# Hostname
docker run --hostname myhost myapp

# Privileged mode (full host access — use with caution)
docker run --privileged myapp

# Read-only filesystem
docker run --read-only myapp

# Capabilities
docker run --cap-add NET_ADMIN myapp
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp

# PID namespace
docker run --pid=host myapp          # Share host PID namespace
docker run --pid=container:other myapp  # Share with another container

# Init process (proper signal handling)
docker run --init myapp
```

---

## 2.11 docker run Quick Reference

| Flag | Purpose | Example |
|------|---------|---------|
| `-d` | Detached mode | `docker run -d nginx` |
| `-it` | Interactive TTY | `docker run -it ubuntu bash` |
| `-p` | Port mapping | `-p 8080:80` |
| `-e` | Environment variable | `-e KEY=val` |
| `-v` | Volume/bind mount | `-v vol:/data` |
| `--name` | Container name | `--name web` |
| `--rm` | Auto-remove on exit | `docker run --rm` |
| `--network` | Network to join | `--network mynet` |
| `--memory` | Memory limit | `--memory=512m` |
| `--cpus` | CPU limit | `--cpus=1.5` |
| `--restart` | Restart policy | `--restart=always` |
| `-w` | Working directory | `-w /app` |
| `-u` | User | `-u 1000:1000` |
| `--init` | Use tini | `--init` |
| `--read-only` | Read-only rootfs | `--read-only` |

---

## Summary Table

| Category | Flags | Purpose |
|----------|-------|---------|
| **Mode** | `-d`, `-it`, `--rm` | How container runs |
| **Network** | `-p`, `--network`, `--dns` | Connectivity |
| **Storage** | `-v`, `--mount`, `--tmpfs` | Data persistence |
| **Resources** | `--memory`, `--cpus` | Resource limits |
| **Security** | `-u`, `--cap-drop`, `--read-only` | Hardening |
| **Config** | `-e`, `--env-file`, `--name` | Configuration |

---

## Quick Revision Questions

1. **What is the difference between `-d` and `-it` flags?**
   - `-d` runs the container in detached (background) mode. `-it` runs in interactive mode with a terminal attached, useful for shells and interactive programs

2. **How does port mapping work with `-p`?**
   - `-p HOST:CONTAINER` maps a port on the host to a port in the container. Traffic to the host port is forwarded to the container port

3. **What does `--rm` do?**
   - It automatically removes the container (and its anonymous volumes) when the container exits, preventing accumulation of stopped containers

4. **How do you limit container memory?**
   - Use `--memory=512m` to set a hard memory limit. The container's process will be OOM-killed if it exceeds this limit

5. **What is the difference between `-v` and `--mount`?**
   - `-v` is a shorthand syntax (`-v source:target:options`). `--mount` is more explicit with named parameters (`--mount type=bind,source=...,target=...`). `--mount` is recommended for clarity

6. **When should you use `--network host`?**
   - When the container needs direct access to the host's network stack, typically for performance-sensitive applications or when the container needs to bind to many dynamic ports

---

[← Previous: Container Lifecycle](01-container-lifecycle.md) | [Next: Container Inspection →](03-container-inspection.md)
