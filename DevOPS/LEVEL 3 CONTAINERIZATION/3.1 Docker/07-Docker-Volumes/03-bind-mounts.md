# Chapter 3: Bind Mounts

> **Unit 7: Docker Volumes** | [Course Home](../README.md)

---

## Overview

Bind mounts map a specific file or directory on the host machine into a container. Unlike volumes, the host path controls the content — making bind mounts ideal for development workflows where you need real-time code synchronization between host and container.

---

## 3.1 Creating Bind Mounts

```bash
# Using --mount (recommended)
docker run -d \
  --mount type=bind,source=/home/user/app,target=/app \
  myimage

# Using -v shorthand
docker run -d -v /home/user/app:/app myimage

# Windows paths
docker run -d -v C:\Users\user\app:/app myimage
docker run -d -v "C:\My Projects\app":/app myimage

# Using $(pwd) for current directory
docker run -d -v $(pwd):/app myimage           # Linux/macOS
docker run -d -v ${PWD}:/app myimage           # PowerShell
docker run -d -v "%cd%":/app myimage           # CMD
```

```
  Bind Mount vs Volume:
  
  Volume:                        Bind Mount:
  Docker manages location        You specify the path
  
  /var/lib/docker/volumes/       /home/user/project/
  └── mydata/                    ├── src/
      └── _data/                 ├── config/
          └── files...           └── data/
          |                          |
          v                          v
     Container: /data           Container: /app
  
  Key difference: Bind mount content 
  is controlled by the HOST
```

---

## 3.2 Bind Mount Behavior

```
  Critical Behavior Differences:
  
  Scenario: Image has files at /app, you bind mount to /app
  
  Volume mount:               Bind Mount:
  (volume is empty)           (host dir has files)
  +------------------+       +------------------+
  | Volume gets      |       | Host content     |
  | IMAGE content    |       | REPLACES image   |
  | (pre-populated)  |       | content entirely |
  +------------------+       +------------------+
  
  Bind Mount ALWAYS wins!
  Image content at that path is hidden/obscured
```

```bash
# Example: nginx with custom content
# This REPLACES default nginx HTML with your files
docker run -d \
  -v $(pwd)/my-site:/usr/share/nginx/html:ro \
  -p 8080:80 \
  nginx

# WARNING: Empty host directory hides image content
docker run -d -v $(pwd)/empty-dir:/app myimage
# Container sees empty /app even if image had files there!
```

---

## 3.3 Development Workflow with Bind Mounts

```bash
# Node.js development
docker run -d \
  --name dev-app \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/package.json:/app/package.json \
  -v /app/node_modules \
  -p 3000:3000 \
  node:20 npm run dev

# Python development  
docker run -d \
  --name dev-api \
  -v $(pwd):/app \
  -p 8000:8000 \
  -w /app \
  python:3.12 python -m flask run --host=0.0.0.0

# Go development with hot reload
docker run -d \
  --name dev-go \
  -v $(pwd):/app \
  -p 8080:8080 \
  -w /app \
  cosmtrek/air
```

```
  Development Pattern:
  
  Host (your machine)          Container
  +------------------+        +------------------+
  | VS Code / IDE    |        | Running App      |
  |                  |        |                  |
  | src/             | <----> | /app/src/        |
  |   index.js  (*)  |  bind  |   index.js       |
  |   app.js         | mount  |   app.js         |
  |                  |        |                  |
  | Edit & Save ---> | -----> | Hot Reload!      |
  +------------------+        +------------------+
  
  (*) Edit on host → instantly visible in container
      No rebuild needed!
```

### The node_modules Problem

```bash
# Problem: node_modules from host may not work in container
# (different OS, binary dependencies)

# Solution: Use anonymous volume to isolate node_modules
docker run -d \
  -v $(pwd):/app \
  -v /app/node_modules \
  node:20 npm start

# The anonymous volume at /app/node_modules
# prevents host node_modules from obscuring
# the container's installed packages
```

```
  node_modules Isolation:
  
  Host bind mount:          Anonymous volume:
  +------------------+     +------------------+
  | /app (from host) |     | /app/node_modules|
  | - src/           |     | (container's own)|
  | - package.json   |     | - express/       |
  | - node_modules/  |<-X  | - react/         |
  +------------------+ |   +------------------+
         Host's          |
         node_modules    +-- Volume takes precedence
         is hidden           at this specific path
```

---

## 3.4 Read-Only Bind Mounts

```bash
# Read-only mount (container can't modify host files)
docker run -d \
  -v $(pwd)/config:/etc/myapp:ro \
  myapp

# With --mount
docker run -d \
  --mount type=bind,source=$(pwd)/config,target=/etc/myapp,readonly \
  myapp

# Practical: Config files read-only, data writable
docker run -d \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -v nginx_logs:/var/log/nginx \
  nginx
```

---

## 3.5 Single File Bind Mounts

```bash
# Mount a single file (not a directory)
docker run -d \
  -v $(pwd)/custom.conf:/etc/app/config.conf:ro \
  myapp

# Override specific config files
docker run -d \
  -v $(pwd)/my-nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx

# Mount .env file
docker run -d \
  -v $(pwd)/.env:/app/.env:ro \
  myapp
```

```
  Single File Mount:
  
  Host                     Container
  +----------------+       +------------------+
  | nginx.conf (*) | ----> | /etc/nginx/      |
  +----------------+  bind |   nginx.conf (*) |
                     mount |   mime.types     |
                           |   conf.d/       |
                           +------------------+
  
  Only nginx.conf is replaced,
  other files in /etc/nginx/ remain unchanged
```

---

## 3.6 Bind Mount Propagation

```bash
# Mount propagation controls whether mounts created
# inside a container are visible on the host and vice versa

# rprivate (default) — No propagation
docker run -v /data:/data:rprivate myimage

# rshared — Bidirectional propagation
docker run -v /data:/data:rshared myimage

# rslave — Host-to-container propagation only
docker run -v /data:/data:rslave myimage
```

```
  Propagation Modes:
  
  rprivate (default):
  Host mount change → NOT seen in container
  Container mount   → NOT seen on host
  
  rslave:
  Host mount change → Seen in container ✓
  Container mount   → NOT seen on host
  
  rshared:
  Host mount change → Seen in container ✓
  Container mount   → Seen on host ✓
  
  Most use cases: rprivate (default) is fine
```

---

## 3.7 Security Considerations

```bash
# DANGER: Bind mounting sensitive host paths
docker run -v /etc:/host-etc myimage        # BAD!
docker run -v /:/host-root myimage          # VERY BAD!
docker run -v /var/run/docker.sock:/var/run/docker.sock  # Careful!

# SAFE: Mount only what's needed, read-only when possible
docker run -v $(pwd)/config.yml:/app/config.yml:ro myimage

# SELinux systems: Add :z or :Z label
docker run -v $(pwd)/data:/data:z myimage   # Shared label
docker run -v $(pwd)/data:/data:Z myimage   # Private label
```

---

## Summary Table

| Feature | Description |
|---------|-------------|
| **Bind mount** | Maps host file/directory to container path |
| **Syntax** | `-v /host/path:/container/path` or `--mount type=bind,...` |
| **Behavior** | Host content replaces/hides image content |
| **Pre-population** | No — host content always wins |
| **Read-only** | Append `:ro` or use `readonly` option |
| **Single file** | Can mount individual files, not just directories |
| **Propagation** | rprivate (default), rslave, rshared |
| **Dev workflow** | Edit on host, changes instantly in container |
| **SELinux** | Use `:z` (shared) or `:Z` (private) labels |
| **Best for** | Development, config files, source code |

---

## Quick Revision Questions

1. **What happens when you bind mount a host directory to a path that has files in the image?**
   - The host directory content replaces (obscures) the image content at that path. The image files still exist but are hidden behind the mount

2. **How do you solve the node_modules conflict in development?**
   - Use an anonymous volume for node_modules: `-v $(pwd):/app -v /app/node_modules`. This prevents the host's node_modules from overriding the container's

3. **Why should bind mounts be read-only when possible?**
   - To prevent the container from modifying host files, reducing the attack surface. A compromised container with write access to host paths can alter system files

4. **What is the difference between `-v` and `--mount` for bind mounts?**
   - With `-v`, if the host path doesn't exist, Docker creates it as a directory. With `--mount`, Docker returns an error — which is safer and prevents accidental directory creation

5. **When would you use bind mounts vs volumes in production?**
   - Volumes are preferred in production (Docker-managed, portable, better performance). Bind mounts are best for development (real-time code editing) and injecting specific config files

---

[← Previous: Volumes](02-volumes.md) | [Next: tmpfs Mounts →](04-tmpfs-mounts.md)
