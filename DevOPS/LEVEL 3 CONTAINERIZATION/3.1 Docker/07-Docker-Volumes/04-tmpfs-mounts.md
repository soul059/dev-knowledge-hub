# Chapter 4: tmpfs Mounts

> **Unit 7: Docker Volumes** | [Course Home](../README.md)

---

## Overview

tmpfs mounts store data in the host's memory (RAM) only. The data is never written to the host's filesystem and is removed when the container stops. This makes tmpfs ideal for sensitive data like secrets, session tokens, and temporary processing files.

---

## 4.1 Creating tmpfs Mounts

```bash
# Using --mount (recommended)
docker run -d \
  --mount type=tmpfs,target=/run/secrets \
  myapp

# Using --tmpfs flag (simpler)
docker run -d --tmpfs /tmp myapp

# With size and mode options
docker run -d \
  --mount type=tmpfs,target=/tmp,tmpfs-size=100m,tmpfs-mode=1777 \
  myapp

# --tmpfs with options
docker run -d --tmpfs /tmp:rw,noexec,nosuid,size=64m myapp
```

---

## 4.2 How tmpfs Works

```
  Storage Comparison:
  
  Volume / Bind Mount:              tmpfs Mount:
  
  Container                         Container
  +-------------+                   +-------------+
  | /data       |                   | /secrets    |
  +------+------+                   +------+------+
         |                                 |
         v                                 v
  +------+------+                   +------+------+
  | Host Disk   |                   | Host RAM    |
  | (persists)  |                   | (ephemeral) |
  +-------------+                   +-------------+
  
  tmpfs data:
  ✓ Never written to disk
  ✓ Removed when container stops
  ✓ Very fast (memory speed)
  ✗ Not shared between containers
  ✗ Linux containers only
  ✗ Limited by available RAM
```

---

## 4.3 tmpfs Options

```bash
# tmpfs-size: Maximum size in bytes (default: unlimited = 50% of RAM)
--mount type=tmpfs,target=/tmp,tmpfs-size=67108864    # 64MB
--mount type=tmpfs,target=/tmp,tmpfs-size=100m        # 100MB

# tmpfs-mode: File permissions (octal)
--mount type=tmpfs,target=/tmp,tmpfs-mode=1777        # Sticky bit (like /tmp)
--mount type=tmpfs,target=/run,tmpfs-mode=0755        # Standard directory

# Multiple options
docker run -d \
  --mount type=tmpfs,target=/tmp,tmpfs-size=256m,tmpfs-mode=1777 \
  --mount type=tmpfs,target=/run/secrets,tmpfs-size=1m,tmpfs-mode=0500 \
  myapp
```

---

## 4.4 Use Cases

### Secrets and Sensitive Data

```bash
# Store secrets in RAM only
docker run -d \
  --mount type=tmpfs,target=/run/secrets,tmpfs-size=1m,tmpfs-mode=0500 \
  --env DB_PASSWORD=secret123 \
  myapp

# The application writes secrets to /run/secrets/
# They exist only in memory, never on disk
```

### Temporary Processing Files

```bash
# Image/video processing temp files
docker run -d \
  --mount type=tmpfs,target=/tmp,tmpfs-size=512m \
  image-processor

# Build artifacts that don't need persistence
docker run --rm \
  --mount type=tmpfs,target=/tmp,tmpfs-size=1g \
  -v $(pwd):/src:ro \
  -v $(pwd)/output:/output \
  builder make -C /src OUTPUT=/output
```

### Performance-Critical Scratch Space

```bash
# Database temp tables / sort space
docker run -d \
  --mount type=tmpfs,target=/var/lib/mysql/tmp,tmpfs-size=256m \
  -v dbdata:/var/lib/mysql \
  mysql:8

# Application cache in RAM
docker run -d \
  --mount type=tmpfs,target=/app/cache,tmpfs-size=128m \
  webapp
```

```
  tmpfs Use Case Decision:
  
  Is data sensitive?
    Yes → tmpfs (never touches disk)
  
  Is data temporary?
    Yes → tmpfs (auto-cleaned on stop)
  
  Need maximum I/O speed?
    Yes → tmpfs (memory speed)
  
  Need to share between containers?
    Yes → Use volumes instead
  
  Need data after restart?
    Yes → Use volumes instead
```

---

## 4.5 tmpfs vs Other Mount Types

```
  Comparison Matrix:
  
  Feature          Volume    Bind     tmpfs
  +--------------+--------+--------+--------+
  | Persists     | Yes    | Yes    | No     |
  | Location     | Docker | Host   | RAM    |
  | Shareable    | Yes    | Yes    | No     |
  | Performance  | Good   | Good   | Best   |
  | Security     | Medium | Low    | High   |
  | OS Support   | All    | All    | Linux  |
  | Size limit   | Disk   | Disk   | RAM    |
  | Written to   | Disk   | Disk   | Never  |
  | Use case     | Data   | Dev    | Secrets|
  +--------------+--------+--------+--------+
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **tmpfs mount** | In-memory filesystem, never persisted to disk |
| **--tmpfs** | Simple flag for tmpfs mount |
| **--mount type=tmpfs** | Verbose tmpfs mount syntax with options |
| **tmpfs-size** | Max size in bytes (default: 50% of host RAM) |
| **tmpfs-mode** | File permission mode (octal, e.g., 1777) |
| **Persistence** | None — data lost when container stops |
| **Sharing** | Cannot be shared between containers |
| **Platform** | Linux containers only (not Windows) |
| **Best for** | Secrets, temp files, caches, scratch space |

---

## Quick Revision Questions

1. **What happens to data in a tmpfs mount when the container stops?**
   - The data is permanently lost. tmpfs exists only in RAM and is cleaned up when the container stops

2. **Why is tmpfs good for storing secrets?**
   - Secrets in tmpfs are never written to the host's disk, reducing the risk of sensitive data being recovered from disk storage

3. **Can tmpfs mounts be shared between containers?**
   - No, tmpfs mounts are exclusive to a single container

4. **What is the default size limit for tmpfs?**
   - If no size is specified, tmpfs can use up to 50% of the host's RAM. Use `tmpfs-size` to set explicit limits

5. **On which platforms does tmpfs work?**
   - tmpfs is only available on Linux. It does not work with Windows containers. Docker Desktop on Mac/Windows runs Linux containers in a VM, so tmpfs works there

---

[← Previous: Bind Mounts](03-bind-mounts.md) | [Next: Storage Drivers →](05-storage-drivers.md)
