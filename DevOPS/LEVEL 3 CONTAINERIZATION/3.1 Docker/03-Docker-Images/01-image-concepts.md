# Chapter 1: Image Concepts

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

Docker images are the **foundation** of containerization. An image is a lightweight, standalone, read-only package that includes everything needed to run an application — code, runtime, libraries, environment variables, and configuration files.

---

## 1.1 What Is a Docker Image?

```
  A Docker image is like a CLASS in programming:
  
  Image (class)                    Container (instance)
  +------------------+             +------------------+
  |  nginx:latest    |  creates    | Container "web1" |
  |                  | --------+-> | Running nginx    |
  |  Blueprint/      |         |   +------------------+
  |  Template        |         |   +------------------+
  |                  |         +-> | Container "web2" |
  |  Read-only       |         |   | Running nginx    |
  |  Immutable       |         |   +------------------+
  |                  |         |   +------------------+
  |                  |         +-> | Container "web3" |
  +------------------+             | Running nginx    |
                                   +------------------+
```

### Image Characteristics

| Characteristic | Description |
|---------------|-------------|
| **Read-only** | Cannot be modified after creation |
| **Layered** | Built from stacked filesystem layers |
| **Portable** | Same image runs on any Docker host |
| **Versioned** | Tagged with versions (e.g., `nginx:1.25`) |
| **Cacheable** | Layers cached locally for faster builds |
| **Shareable** | Pushed to/pulled from registries |
| **Composable** | Built on top of other images (FROM) |

---

## 1.2 Image Internals

### Image Components

```
  +---------------------------------------------------+
  |  Docker Image                                      |
  |                                                    |
  |  1. MANIFEST                                       |
  |     - Lists all layers                             |
  |     - Points to config                             |
  |     - Platform info (OS, arch)                     |
  |                                                    |
  |  2. CONFIG (JSON)                                  |
  |     - Environment variables                        |
  |     - Default command (CMD)                        |
  |     - Entrypoint                                   |
  |     - Exposed ports                                |
  |     - Working directory                            |
  |     - User to run as                               |
  |     - Labels                                       |
  |                                                    |
  |  3. LAYERS (filesystem diffs)                      |
  |     +--------------------------------------+       |
  |     | Layer N: Application code            |       |
  |     +--------------------------------------+       |
  |     | Layer 3: npm install dependencies    |       |
  |     +--------------------------------------+       |
  |     | Layer 2: apt-get install packages    |       |
  |     +--------------------------------------+       |
  |     | Layer 1: Base OS (debian slim)       |       |
  |     +--------------------------------------+       |
  +---------------------------------------------------+
```

### Image Identification

Images are identified in three ways:

```
  1. REPOSITORY:TAG (human-friendly)
     nginx:1.25-alpine
     myapp:latest
     python:3.11-slim
  
  2. IMAGE ID (truncated SHA256 of config)
     sha256:a1b2c3d4e5f6
     (shown as a1b2c3d4e5f6 in docker images)
  
  3. DIGEST (content-addressable, immutable)
     nginx@sha256:32fdf92db...
     (points to exact manifest — cannot change)
     
  +------------------+-------------------+---------------------------+
  |  Name:Tag        |  Image ID         |  Digest                   |
  |  nginx:latest    |  a1b2c3d4e5f6     |  sha256:32fdf92db...     |
  |  (can change!)   |  (based on config)|  (immutable, exact)       |
  +------------------+-------------------+---------------------------+
  
  WARNING: "latest" is just a tag — it doesn't guarantee
  the most recent version! It's simply the default tag.
```

---

## 1.3 Base Images

Every Docker image starts from a **base image** specified in the `FROM` instruction.

```
  Common Base Image Hierarchy:
  
  scratch (empty image)
  ├── alpine:3.19 (~5 MB)
  │   ├── nginx:alpine
  │   ├── python:3.11-alpine
  │   └── node:20-alpine
  │
  ├── debian:bookworm-slim (~80 MB)
  │   ├── python:3.11-slim
  │   ├── node:20-slim
  │   └── openjdk:21-slim
  │
  ├── ubuntu:22.04 (~77 MB)
  │   ├── Custom app images
  │   └── Development images
  │
  └── distroless (~20 MB)
      ├── gcr.io/distroless/base
      ├── gcr.io/distroless/java
      └── gcr.io/distroless/nodejs
```

### Base Image Size Comparison

```
  Image                    Size
  ─────────────────────    ──────
  scratch                  0 MB
  alpine:3.19              ~5 MB     ████
  debian:bookworm-slim     ~80 MB    ████████████████████
  ubuntu:22.04             ~77 MB    ███████████████████
  node:20-alpine           ~130 MB   ██████████████████████████
  node:20-slim             ~250 MB   ██████████████████████████████████████████
  node:20                  ~1 GB     ███████████████████████████████████████████████████████
  python:3.11-alpine       ~55 MB    ██████████████
  python:3.11-slim         ~155 MB   ████████████████████████████████
  python:3.11              ~1 GB     ███████████████████████████████████████████████████████
```

### Choosing a Base Image

| Base Image | Size | Best For | Trade-off |
|-----------|------|----------|-----------|
| `scratch` | 0 MB | Static Go/Rust binaries | No shell, no tools |
| `alpine` | ~5 MB | Small, production images | musl libc (compatibility) |
| `*-slim` | ~80-250 MB | Good balance | Missing some dev tools |
| `debian/ubuntu` | ~80-1000 MB | Development, full tooling | Larger image size |
| `distroless` | ~20 MB | Security-focused production | No shell, minimal |

---

## 1.4 Image vs Container — Key Differences

```
  IMAGE:                              CONTAINER:
  
  +------------------+                +------------------+
  | Read-only        |                | Read-write       |
  | Static           |                | Dynamic          |
  | Template         |                | Instance         |
  | Stored on disk   |                | Running process  |
  | Shareable        |                | Isolated         |
  | Immutable        |                | Has state        |
  +------------------+                +------------------+
  
  docker images  <-- shows these     docker ps  <-- shows these
  docker build   <-- creates these   docker run <-- creates these
  docker pull    <-- downloads these docker exec<-- interacts with these
```

---

## 1.5 Image Creation Methods

```
  Method 1: Dockerfile (most common)
  +-----------+     +-------------+     +--------+
  | Dockerfile| --> | docker build| --> | Image  |
  +-----------+     +-------------+     +--------+
  
  Method 2: Commit a container (not recommended for production)
  +----------+     +--------+     +---------------+     +--------+
  | Run      | --> | Modify | --> | docker commit | --> | Image  |
  | container|     | files  |     |               |     |        |
  +----------+     +--------+     +---------------+     +--------+
  
  Method 3: Import a tarball
  +----------+     +---------------+     +--------+
  | tar file | --> | docker import | --> | Image  |
  +----------+     +---------------+     +--------+
  
  Method 4: BuildKit / Buildx (advanced)
  +-----------+     +---------------+     +--------+
  | Dockerfile| --> | docker buildx | --> | Multi- |
  |           |     | build         |     | platform|
  +-----------+     +---------------+     | Image  |
                                          +--------+
```

---

## 1.6 Hands-On: Exploring Images

```bash
# List all local images
docker images
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# nginx        latest    a1b2c3d4e5f6   2 weeks ago    187MB
# python       3.11      b2c3d4e5f6a1   1 week ago     1.01GB

# Pull an image
docker pull alpine:3.19

# View image details
docker image inspect alpine:3.19

# View image layers
docker image history alpine:3.19
# IMAGE          CREATED       CREATED BY                          SIZE
# c1aabb73d233   2 weeks ago   CMD ["/bin/sh"]                     0B
# <missing>      2 weeks ago   ADD alpine-minirootfs-3.19 / ...    7.38MB

# Check disk usage
docker system df
# TYPE       TOTAL    ACTIVE   SIZE      RECLAIMABLE
# Images     5        2        2.1GB     1.2GB (57%)
# Containers 3        1        120MB     80MB (66%)
# Volumes    4        2        500MB     200MB (40%)

# Remove unused images
docker image prune        # Dangling only
docker image prune -a     # All unused
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Image | Read-only, layered template for creating containers |
| Layer | A filesystem diff representing one build step |
| Manifest | JSON listing all layers and config for an image |
| Config | JSON with runtime settings (CMD, ENV, EXPOSE, etc.) |
| Tag | Human-readable label (e.g., `v1.0`, `latest`) |
| Digest | Immutable content-addressable identifier (SHA256) |
| Base Image | Starting point specified by `FROM` in Dockerfile |
| scratch | Empty base image for minimal containers |

---

## Quick Revision Questions

1. **What are the three main components of a Docker image?**
   - Manifest (lists layers and config), Config (runtime settings like CMD, ENV), and Layers (filesystem diffs)

2. **What is the difference between an image tag and an image digest?**
   - A tag is a mutable label (can be moved to a different image). A digest is an immutable SHA256 hash of the manifest, guaranteeing exact content

3. **Why is the `latest` tag misleading?**
   - It's just the default tag name — it doesn't guarantee the most recent version. It only points to whatever was last pushed without a specific tag

4. **When would you choose `alpine` over `debian-slim` as a base image?**
   - When image size is critical and your application doesn't depend on glibc. Alpine uses musl libc (~5MB) while debian-slim uses glibc (~80MB)

5. **What is the `scratch` base image?**
   - An empty image with no OS, shell, or libraries. Used for statically compiled binaries (Go, Rust) that include all dependencies in the binary itself

6. **What are the four methods to create a Docker image?**
   - Dockerfile + `docker build`, `docker commit` from a container, `docker import` from tarball, and `docker buildx` for multi-platform builds

---

[← Previous Unit: Docker Objects](../02-Docker-Architecture/05-docker-objects.md) | [Next: Image Layers and Caching →](02-image-layers-and-caching.md)
