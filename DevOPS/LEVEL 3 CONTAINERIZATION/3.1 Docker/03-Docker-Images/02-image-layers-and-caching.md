# Chapter 2: Image Layers and Caching

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

Docker images use a **layered filesystem** where each instruction in a Dockerfile creates a new layer. Understanding layers and Docker's build cache is crucial for creating efficient images and fast builds.

---

## 2.1 How Layers Work

```
  Dockerfile:                        Image Layers:
  
  FROM ubuntu:22.04          ──>     +---------------------------+
                                     | Layer 1: Ubuntu base      | 77 MB
                                     +---------------------------+
  RUN apt-get update &&      ──>     +---------------------------+
      apt-get install -y             | Layer 2: Python + deps    | 150 MB
      python3 python3-pip            +---------------------------+
  
  COPY requirements.txt .    ──>     +---------------------------+
                                     | Layer 3: requirements.txt | 2 KB
                                     +---------------------------+
  RUN pip install -r         ──>     +---------------------------+
      requirements.txt               | Layer 4: pip packages     | 80 MB
                                     +---------------------------+
  COPY . /app                ──>     +---------------------------+
                                     | Layer 5: Application code | 5 MB
                                     +---------------------------+
  CMD ["python3", "app.py"]  ──>     (metadata only, no layer)
  
  TOTAL IMAGE SIZE: ~312 MB
  
  Each layer is a DIFF — only the changes from the previous layer.
  Layers are READ-ONLY and content-addressed (SHA256).
```

---

## 2.2 Layer Sharing

One of Docker's most powerful features — layers are **shared** across images and containers.

```
  Image A: Python App            Image B: Flask App
  
  +------------------+           +------------------+
  | App A code       | 5 MB      | App B code       | 8 MB
  +------------------+           +------------------+
  | pip packages A   | 80 MB     | pip packages B   | 120 MB
  +------------------+           +------------------+
  | requirements.txt | 2 KB      | requirements.txt | 3 KB
  +------------------+           +------------------+
  | Python + deps    | 150 MB  ← SHARED →  150 MB    |
  +------------------+           +------------------+
  | Ubuntu base      | 77 MB  ← SHARED →   77 MB    |
  +------------------+           +------------------+
  
  WITHOUT sharing: 235 + 355 = 590 MB on disk
  WITH sharing:    235 + 128 = 363 MB on disk  (38% saved!)
  
  Shared layers are stored ONCE and referenced by both images.
```

---

## 2.3 Union Filesystem (OverlayFS)

Docker uses **OverlayFS** to combine layers into a single unified view.

```
  OverlayFS Structure:
  
  +-----------------------------------------------+
  |            MERGED VIEW (what container sees)    |
  |  /bin  /etc  /lib  /app  /tmp                  |
  +-----------------------------------------------+
       ^         ^         ^         ^
       |         |         |         |
  +----+----+----+----+----+----+----+----+
  |         |         |         |         |
  | Upper   | Lower   | Lower  | Lower  |
  | Layer   | Layer   | Layer  | Layer  |
  |(writable)|(read)  | (read) | (read) |
  | Container| App    | Deps   | Base   |
  | changes  | code   |        | OS     |
  +----------+---------+--------+--------+
  
  Copy-on-Write (CoW):
  ┌──────────────────────────────────────────┐
  │ When container modifies a file from a    │
  │ lower layer, the file is COPIED to the   │
  │ upper layer first, then modified.        │
  │ Original lower layer file is unchanged.  │
  └──────────────────────────────────────────┘
```

### Copy-on-Write Example

```
  Container wants to edit /etc/nginx/nginx.conf
  
  Before:
  Upper (writable):  (empty)
  Lower (read-only): /etc/nginx/nginx.conf  (original)
  
  Step 1: Copy file from lower to upper layer
  Upper (writable):  /etc/nginx/nginx.conf  (copy)
  Lower (read-only): /etc/nginx/nginx.conf  (original, untouched)
  
  Step 2: Modify the copy in upper layer
  Upper (writable):  /etc/nginx/nginx.conf  (modified)
  Lower (read-only): /etc/nginx/nginx.conf  (original, untouched)
  
  Container sees: the modified version (upper wins)
  Other containers see: the original (lower layer)
```

---

## 2.4 Build Cache — How It Works

Docker caches each layer during builds. If nothing changed, it reuses the cached layer.

```
  FIRST BUILD (no cache):
  
  Step 1/5 : FROM node:20-alpine
   ---> Downloading... (130 MB)              4.2s
  Step 2/5 : WORKDIR /app
   ---> Running...                           0.1s
  Step 3/5 : COPY package*.json ./
   ---> Copying...                           0.1s
  Step 4/5 : RUN npm install
   ---> Running... (downloads packages)      45.0s
  Step 5/5 : COPY . .
   ---> Copying...                           0.3s
  
  Total: 49.7s
  
  ============================================
  
  SECOND BUILD (only app code changed):
  
  Step 1/5 : FROM node:20-alpine
   ---> Using cache  ✓                       0.0s
  Step 2/5 : WORKDIR /app
   ---> Using cache  ✓                       0.0s
  Step 3/5 : COPY package*.json ./
   ---> Using cache  ✓                       0.0s  (package.json unchanged)
  Step 4/5 : RUN npm install
   ---> Using cache  ✓                       0.0s  (npm install skipped!)
  Step 5/5 : COPY . .
   ---> Copying... (files changed)           0.3s
  
  Total: 0.3s  (99% faster!)
```

### Cache Invalidation Rules

```
  CACHE IS INVALIDATED when:
  
  1. Instruction changes
     RUN apt-get update          (cached)
     RUN apt-get update && ...   (MISS — instruction text changed)
  
  2. File content changes (for COPY/ADD)
     COPY package.json .         (cached if file unchanged)
     COPY package.json .         (MISS if file content changed)
  
  3. Previous layer was invalidated
     FROM node:20               ✓ cached
     RUN npm install            ✓ cached
     COPY src/ .                ✗ MISS (source changed)
     RUN npm run build          ✗ MISS (previous layer changed!)
     CMD ["node", "dist/main"]  ✗ MISS (cascade invalidation!)
  
  KEY INSIGHT: Once a layer cache misses, 
  ALL subsequent layers are rebuilt!
  
  +---------+  +---------+  +---------+  +---------+  +---------+
  | Layer 1 |  | Layer 2 |  | Layer 3 |  | Layer 4 |  | Layer 5 |
  | CACHED  |  | CACHED  |  | CHANGED |  | REBUILD |  | REBUILD |
  |   ✓     |  |   ✓     |  |   ✗     |  |   ✗     |  |   ✗     |
  +---------+  +---------+  +---------+  +---------+  +---------+
```

---

## 2.5 Optimizing Layer Order for Cache Hits

### BAD Order (cache busted on every code change):

```dockerfile
# BAD: Application code changes frequently
FROM node:20-alpine
WORKDIR /app
COPY . .                    # All files copied — changes every time!
RUN npm install             # Reinstalls ALL deps every time!
CMD ["node", "server.js"]
```

### GOOD Order (caching optimized):

```dockerfile
# GOOD: Dependencies change rarely, code changes often
FROM node:20-alpine
WORKDIR /app
COPY package.json package-lock.json ./    # Only dependency files
RUN npm ci                                # Cached unless deps change!
COPY . .                                  # Only code changes
CMD ["node", "server.js"]
```

```
  Code change frequency:

  Changes RARELY                    Changes OFTEN
  ◄──────────────────────────────────────────────►
  
  FROM        RUN apt-get    COPY pkg.json   RUN npm    COPY . .
  base image  install deps   dependency file install    app code
  
  PUT THESE FIRST ──────────────────────── PUT THESE LAST
  (maximize cache hits)                    (minimize rebuild scope)
```

---

## 2.6 Layer Size Best Practices

### Combine RUN Commands (Reduce Layers)

```dockerfile
# BAD: Creates 3 layers, intermediate files stick around
RUN apt-get update
RUN apt-get install -y python3
RUN rm -rf /var/lib/apt/lists/*

# GOOD: Single layer, cleanup in same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 && \
    rm -rf /var/lib/apt/lists/*
```

### Why Cleanup Must Be in the Same Layer

```
  Layer 1: RUN apt-get update        +100 MB (apt cache)
  Layer 2: RUN apt-get install       +200 MB (packages)
  Layer 3: RUN rm -rf /var/lib/apt   +0 MB   (marks as deleted)
  
  TOTAL: 300 MB (deleted files still in Layer 1!)
  
  VS:
  
  Layer 1: RUN update && install     +200 MB (no apt cache)
           && rm -rf /var/lib/apt
  
  TOTAL: 200 MB (cleanup happens in same layer!)
  
  Files deleted in a later layer are NOT removed from
  previous layers! They just get a "whiteout" marker.
```

---

## 2.7 Viewing Layers

```bash
# View layer history of an image
docker image history nginx:latest

# IMAGE          CREATED       CREATED BY                                 SIZE
# a1b2c3d4e5f6   2 days ago    CMD ["nginx" "-g" "daemon off;"]           0B
# <missing>      2 days ago    EXPOSE map[80/tcp:{}]                      0B
# <missing>      2 days ago    STOPSIGNAL SIGQUIT                         0B
# <missing>      2 days ago    RUN /bin/sh -c set -x ...                  61.1MB
# <missing>      2 days ago    COPY file:xxx in /docker-entrypoint.d      4.62kB
# <missing>      2 days ago    RUN /bin/sh -c apt-get update ...          23.3MB
# <missing>      3 days ago    /bin/sh -c #(nop) ADD file:xxx in /        74.8MB

# Inspect layer details
docker image inspect nginx:latest --format '{{json .RootFS.Layers}}' | jq .

# View image size breakdown
docker system df -v
```

---

## 2.8 BuildKit Cache Improvements

Modern Docker uses **BuildKit** which offers advanced caching:

```bash
# Enable BuildKit (default in Docker 23+)
export DOCKER_BUILDKIT=1

# Use cache mounts for package managers
# Dockerfile:
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# This caches pip downloads between builds!
# Even if requirements.txt changes, already-downloaded
# packages are reused from cache.
```

```dockerfile
# Cache mount examples for different ecosystems
# Python
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Node.js
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Go
RUN --mount=type=cache,target=/go/pkg/mod \
    go build -o /app .

# Apt
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y python3
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Layer | A filesystem diff created by each Dockerfile instruction |
| Union FS | OverlayFS combines layers into unified view |
| Copy-on-Write | Files copied to writable layer only when modified |
| Build Cache | Layers reused if instruction and content haven't changed |
| Cache Invalidation | One changed layer invalidates all subsequent layers |
| Layer Sharing | Common layers stored once, referenced by multiple images |
| Whiteout | Marker for deleted files (original still in earlier layer) |
| Cache Mount | BuildKit feature to persist package manager caches |

---

## Quick Revision Questions

1. **What happens when a Dockerfile instruction changes during a rebuild?**
   - That layer's cache is invalidated, and ALL subsequent layers must also be rebuilt (cascade invalidation)

2. **Why should you copy package.json before copying the entire application?**
   - package.json changes rarely, so `RUN npm install` can use the cache. If you copy all files first, any code change invalidates the npm install layer

3. **Why must cleanup commands be in the same RUN instruction as installations?**
   - Files deleted in a later layer are not removed from earlier layers. They get a "whiteout" marker but still consume space in the image

4. **How does the Copy-on-Write mechanism work?**
   - When a container modifies a file from a read-only layer, the file is copied to the writable layer first, then modified. The original remains unchanged

5. **How do shared layers save disk space?**
   - When multiple images share the same base layers, Docker stores the shared layers only once on disk and references them from each image

6. **What is a BuildKit cache mount and how does it help?**
   - A cache mount persists a directory (like pip/npm cache) between builds. Even when dependencies change, already-downloaded packages are reused, speeding up builds significantly

---

[← Previous: Image Concepts](01-image-concepts.md) | [Next: Pulling and Pushing Images →](03-pulling-and-pushing-images.md)
