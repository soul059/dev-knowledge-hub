# Chapter 2: Build Context

> **Unit 4: Dockerfile** | [Course Home](../README.md)

---

## Overview

The **build context** is the set of files and directories sent to the Docker daemon when you run `docker build`. Understanding build context is essential for fast, efficient, and secure image builds.

---

## 2.1 What Is the Build Context?

```
  docker build -t myapp .
                         ^
                         |
                    Build Context Path
  
  +------------------+         +--------------------+
  |  Docker Client   |  TAR    |   Docker Daemon     |
  |                  | ------> |                     |
  |  Reads build     | context |  Receives context   |
  |  context path    |         |  + Dockerfile        |
  |  Creates tarball |         |  Builds image        |
  +------------------+         +--------------------+
  
  "Sending build context to Docker daemon  45.2MB"
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  This message shows the context size being sent
```

The build context is:
- The directory (or URL) specified in `docker build` command
- Sent as a **tar archive** to the Docker daemon
- Used by `COPY` and `ADD` instructions to access files
- **Everything** in the context path is sent (unless excluded)

---

## 2.2 Context Path Examples

```bash
# Current directory as context
docker build -t myapp .

# Specific directory as context
docker build -t myapp ./src

# URL as context (Git repo)
docker build -t myapp https://github.com/user/repo.git

# URL with branch/tag
docker build -t myapp https://github.com/user/repo.git#main

# URL with subdirectory
docker build -t myapp https://github.com/user/repo.git#main:docker

# Tar archive as context
docker build -t myapp - < context.tar.gz

# Stdin as context (no file access)
echo "FROM alpine" | docker build -t myapp -
```

---

## 2.3 Dockerfile Location

```bash
# Default: Dockerfile in context root
docker build -t myapp .
# Looks for ./Dockerfile

# Custom Dockerfile path
docker build -t myapp -f docker/Dockerfile.prod .

# Dockerfile outside context
docker build -t myapp -f /path/to/Dockerfile ./context-dir

# Multiple Dockerfiles for different purposes
docker build -t myapp-dev -f Dockerfile.dev .
docker build -t myapp-prod -f Dockerfile.prod .
```

```
  Project Structure:
  
  my-project/                            
  ├── docker/                           
  │   ├── Dockerfile.dev        <--- -f docker/Dockerfile.dev
  │   ├── Dockerfile.prod       <--- -f docker/Dockerfile.prod
  │   └── Dockerfile.test       <--- -f docker/Dockerfile.test
  ├── src/                      
  │   └── app.py                
  ├── tests/                    
  ├── Dockerfile                <--- Default (docker build .)
  └── .dockerignore             
```

---

## 2.4 Build Context Size Impact

```
  Large Context (BAD)              Small Context (GOOD)
  
  Project: 500MB                   Project: 500MB
  .dockerignore: None              .dockerignore: Configured
  
  +------------------+             +------------------+
  | Sent to daemon:  |             | Sent to daemon:  |
  | 500MB            |             | 5MB              |
  | - node_modules/  |             | - src/           |
  | - .git/          |             | - package.json   |
  | - build/         |             | - Dockerfile     |
  | - test-data/     |             +------------------+
  | - src/           |             
  | - videos/        |             Build time: 2s
  +------------------+             
                                   
  Build time: 45s                  
```

### How large context hurts:
1. **Slow builds** — entire context must be sent to daemon
2. **Wasted bandwidth** — especially with remote daemon
3. **Security risk** — secrets in context could end up in image
4. **Cache invalidation** — any change in context can bust cache

---

## 2.5 Optimizing Build Context

### Strategy 1: Use .dockerignore

```
  # .dockerignore
  .git
  node_modules
  *.log
  .env
  test/
```

### Strategy 2: Specific Context Path

```bash
# Instead of sending whole project
docker build -t myapp .

# Send only what's needed
docker build -t myapp -f Dockerfile ./src
```

### Strategy 3: Separate Build Directory

```
  my-project/
  ├── docker/
  │   ├── Dockerfile
  │   └── build-context/       <--- Only this is sent
  │       ├── app.py
  │       ├── requirements.txt
  │       └── config/
  ├── src/                     <--- Not sent
  ├── tests/                   <--- Not sent
  └── docs/                    <--- Not sent
```

### Strategy 4: Multi-Stage Builds

```dockerfile
# Only final stage artifacts end up in image
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
# Only the dist folder makes it to the final image
```

---

## 2.6 Context with BuildKit

```bash
# Enable BuildKit (default in Docker 23.0+)
export DOCKER_BUILDKIT=1

# Or prefix build command
DOCKER_BUILDKIT=1 docker build -t myapp .
```

BuildKit improvements over legacy builder:

```
  Legacy Builder                     BuildKit
  +---------------------+           +---------------------+
  | Sends ENTIRE        |           | Sends files only    |
  | context upfront     |           | when needed         |
  |                     |           |                     |
  | Sequential layer    |           | Parallel execution  |
  | execution           |           | of independent      |
  |                     |           | stages              |
  | No cache export     |           |                     |
  |                     |           | Cache export/import |
  | Basic output        |           |                     |
  |                     |           | Progress tracking   |
  +---------------------+           +---------------------+
```

### BuildKit Cache Mounts

```dockerfile
# Cache package manager downloads
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

RUN --mount=type=cache,target=/root/.npm \
    npm ci
```

### BuildKit Secret Mounts

```dockerfile
# Mount secrets without leaking to image layers
RUN --mount=type=secret,id=aws_key \
    cat /run/secrets/aws_key && do_something

# Build with secret:
# docker build --secret id=aws_key,src=./aws_key.txt .
```

---

## 2.7 Remote Build Context

```bash
# Git repository
docker build https://github.com/user/repo.git
docker build https://github.com/user/repo.git#branch
docker build https://github.com/user/repo.git#tag
docker build https://github.com/user/repo.git#:subdir

# Tar archive from URL
docker build https://example.com/context.tar.gz

# How remote context works:
#
#  +--------+    clone/    +----------+    tar     +--------+
#  | Git    | ----------> | Temp Dir  | --------> | Docker |
#  | Repo   |   download  | (client)  |  context  | Daemon |
#  +--------+             +----------+            +--------+
```

---

## 2.8 Debugging Build Context

```bash
# Check context size before building
du -sh --exclude='.git' .

# List what would be in context (respecting .dockerignore)
# Using a dry-run approach
docker build --no-cache -t test . 2>&1 | head -1
# "Sending build context to Docker daemon  4.096kB"

# View effective .dockerignore
cat .dockerignore

# Test with minimal context
echo "FROM alpine" | docker build -t test -

# List files sent in context (debug trick)
# In Dockerfile:
# RUN find / -path /proc -prune -o -print  (NOT recommended for production)
```

---

## Summary Table

| Concept | Description | Example |
|---------|-------------|---------|
| **Build Context** | Files sent to Docker daemon | `docker build .` |
| **Context Path** | Directory specified in build command | `.`, `./src`, URL |
| **-f Flag** | Custom Dockerfile location | `-f docker/Dockerfile.prod` |
| **.dockerignore** | Exclude files from context | `node_modules`, `.git` |
| **Remote Context** | Build from Git/URL | `https://github.com/...` |
| **BuildKit** | Next-gen builder with optimizations | `DOCKER_BUILDKIT=1` |
| **Secret Mount** | Build-time secrets without leaking | `--mount=type=secret` |
| **Cache Mount** | Persistent cache across builds | `--mount=type=cache` |
| **Context Size** | Smaller = faster builds | Use .dockerignore |

---

## Quick Revision Questions

1. **What is the build context in Docker?**
   - The build context is the set of files and directories sent as a tar archive to the Docker daemon. It's the path specified in the `docker build` command

2. **How does build context size affect performance?**
   - Large context means more data transferred to the daemon, slower builds, more bandwidth usage, and increased risk of accidentally including sensitive files

3. **How do you specify a custom Dockerfile location?**
   - Use the `-f` flag: `docker build -f path/to/Dockerfile .`

4. **What advantages does BuildKit provide for build context?**
   - BuildKit sends files lazily (only when needed), supports parallel stage execution, cache export/import, secret mounts, and provides better progress tracking

5. **Can the Dockerfile be outside the build context?**
   - Yes. The Dockerfile path (`-f`) and context path are independent. For example: `docker build -f /elsewhere/Dockerfile ./context-dir`

6. **What is a BuildKit secret mount used for?**
   - Secret mounts allow build-time access to sensitive data (API keys, credentials) without those secrets being stored in any image layer

---

[← Previous: Dockerfile Instructions](01-dockerfile-instructions.md) | [Next: Multi-Stage Builds →](03-multi-stage-builds.md)
