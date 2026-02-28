# Chapter 1: Storage Overview

> **Unit 7: Docker Volumes** | [Course Home](../README.md)

---

## Overview

Containers are ephemeral by design — when a container is removed, its writable layer is lost. Docker provides several storage mechanisms to persist data beyond container lifecycles, share data between containers, and optimize I/O performance.

---

## 1.1 The Ephemeral Container Problem

```
  Without Persistent Storage:
  
  Container Running:          Container Removed:
  +------------------+       
  | Writable Layer   |       +------------------+
  | - app data       |  -->  | DATA LOST!       |
  | - logs           |  rm   | Everything gone   |
  | - uploads        |       +------------------+
  +------------------+       
  | Image Layers (R) |       Image Layers remain
  +------------------+       (shared, read-only)
  
  Problem: Database files, uploads, configs
  all disappear when container stops/removed
```

---

## 1.2 Docker Storage Types

```
  Three Ways to Persist Data:
  
  +---------------------------------------------------+
  |                Docker Host                         |
  |                                                    |
  |  +-----------+  +-----------+  +-----------+      |
  |  |  Volume   |  |   Bind    |  |   tmpfs   |      |
  |  |           |  |   Mount   |  |   Mount   |      |
  |  | /var/lib/ |  |           |  |           |      |
  |  | docker/   |  | Any host  |  | Host      |      |
  |  | volumes/  |  | directory |  | memory    |      |
  |  +-----------+  +-----------+  +-----------+      |
  |       |              |              |              |
  |       +---------+----+----+---------+              |
  |                 |         |                        |
  |            +----v---------v----+                   |
  |            |    Container      |                   |
  |            |  /data  /app  /tmp|                   |
  |            +-------------------+                   |
  +---------------------------------------------------+
```

### Comparison

| Feature | Volumes | Bind Mounts | tmpfs |
|---------|---------|-------------|-------|
| **Location** | Docker-managed area | Anywhere on host | Host memory |
| **Managed by** | Docker | User | Docker |
| **Persists** | Yes | Yes | No (RAM only) |
| **Shareable** | Yes | Yes | No |
| **Content pre-population** | Yes | No | No |
| **Performance** | Good | Native | Fastest |
| **Portability** | High | Low (host-dependent) | N/A |
| **Docker Desktop support** | Full | Full | Linux only |
| **Best for** | Databases, app data | Dev config, source code | Secrets, temp files |

---

## 1.3 How Container Storage Works

```
  Union Filesystem (OverlayFS):
  
  WRITE       +-------------------+
  operations  | Container Layer   | ← Writable (thin)
  go here     | (copy-on-write)   |
              +-------------------+
              | Image Layer 3     | ← Read-only
              +-------------------+
              | Image Layer 2     | ← Read-only
              +-------------------+
              | Image Layer 1     | ← Read-only
              +-------------------+
  
  Copy-on-Write (CoW):
  1. Read: Search layers top-down, return first match
  2. Write: Copy file to writable layer, modify there
  3. Delete: Whiteout file in writable layer
  
  Problem: CoW adds overhead for write-heavy workloads
  Solution: Use volumes for databases and write-heavy data
```

---

## 1.4 Volume Mount Syntax

```bash
# Two syntax styles:

# 1. -v / --volume (traditional)
docker run -v volume_name:/container/path image
docker run -v /host/path:/container/path image

# 2. --mount (verbose, recommended)
docker run --mount type=volume,source=mydata,target=/data image
docker run --mount type=bind,source=/host/path,target=/app image
docker run --mount type=tmpfs,target=/tmp image
```

```
  -v vs --mount Differences:
  
  Feature              -v               --mount
  +------------------+----------------+------------------+
  | Syntax           | Compact        | Key=value pairs  |
  | Missing host dir | Creates it     | Returns error    |
  | Options          | Limited format | Explicit options |
  | Readability      | Less clear     | Self-documenting |
  | Recommended      | Legacy         | Yes              |
  +------------------+----------------+------------------+
  
  Example:
  -v mydata:/app/data:ro
  
  --mount type=volume,source=mydata,target=/app/data,readonly
```

---

## 1.5 Storage Locations on Host

```bash
# Docker's default storage location
# Linux:   /var/lib/docker/
# Windows: C:\ProgramData\docker\
# macOS:   Inside Docker Desktop VM

# Volumes are stored at:
# /var/lib/docker/volumes/<volume_name>/_data/

# Check Docker root directory
docker info | grep "Docker Root Dir"
```

```
  Docker Storage on Host:
  
  /var/lib/docker/
  ├── volumes/          ← Named volumes
  │   ├── mydata/
  │   │   └── _data/    ← Actual files here
  │   └── pgdata/
  │       └── _data/
  ├── overlay2/         ← Image & container layers
  │   ├── abc123.../
  │   └── def456.../
  ├── containers/       ← Container metadata
  ├── image/            ← Image metadata
  └── network/          ← Network config
```

---

## 1.6 When to Use What

```
  Decision Tree:
  
  Need persistent data?
    No --> Use container layer (default)
    Yes --+
          v
  Need to share between containers?
    Yes --> Use a Volume
    Maybe --+
            v
  Need specific host path?
    Yes --> Bind Mount
    No --+
         v
  Sensitive data in RAM only?
    Yes --> tmpfs Mount
    No --+
         v
  Use a Volume (default choice)
  
  Rule of thumb: When in doubt, use Volumes!
```

### Real-World Examples

```bash
# Database: Volume (data must persist)
docker run -d --mount type=volume,source=pgdata,target=/var/lib/postgresql/data postgres

# Development: Bind Mount (edit code on host)
docker run -d --mount type=bind,source=$(pwd)/src,target=/app/src myapp

# Secrets: tmpfs (never written to disk)
docker run -d --mount type=tmpfs,target=/run/secrets myapp

# Logs: Volume with size limit
docker run -d --mount type=tmpfs,target=/tmp,tmpfs-size=100m myapp
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Container layer** | Writable layer lost when container removed |
| **Copy-on-Write** | Files copied to writable layer on first write |
| **Volume** | Docker-managed persistent storage (recommended) |
| **Bind mount** | Maps host directory into container |
| **tmpfs mount** | In-memory storage (Linux), not persisted |
| **-v flag** | Short-form mount syntax (legacy) |
| **--mount flag** | Long-form mount syntax (recommended) |
| **Storage driver** | Provides union filesystem (overlay2) |

---

## Quick Revision Questions

1. **What happens to data in a container's writable layer when the container is removed?**
   - The data is permanently lost. The writable layer is deleted along with the container

2. **What are the three types of mounts Docker supports?**
   - Volumes (Docker-managed), bind mounts (host directory), and tmpfs mounts (in-memory)

3. **Why is `--mount` recommended over `-v`?**
   - `--mount` is more explicit, self-documenting, and returns errors for missing paths instead of silently creating directories

4. **When should you use a volume vs a bind mount?**
   - Use volumes for persistent application data (databases, uploads); use bind mounts for development when you need to edit files on the host and reflect changes in the container

5. **What is copy-on-write and why does it matter for performance?**
   - CoW copies a file from a read-only layer to the writable layer on first modification. This adds I/O overhead, which is why write-heavy workloads (databases) should use volumes instead

---

[← Previous Unit: Docker Networking](../06-Docker-Networking/06-network-troubleshooting.md) | [Next: Volumes →](02-volumes.md)
