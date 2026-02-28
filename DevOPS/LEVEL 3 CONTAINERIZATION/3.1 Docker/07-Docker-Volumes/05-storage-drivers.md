# Chapter 5: Storage Drivers

> **Unit 7: Docker Volumes** | [Course Home](../README.md)

---

## Overview

Storage drivers (also called graph drivers) control how Docker stores image layers and the container's writable layer. They implement the union filesystem that makes Docker's layered architecture possible. The choice of storage driver affects performance, stability, and features.

---

## 5.1 What Storage Drivers Do

```
  Storage Driver's Role:
  
  Container
  +---------------------------+
  |    Writable Layer          | ← Storage driver manages
  +---------------------------+   copy-on-write here
  |    Image Layer 3 (RO)     |
  +---------------------------+
  |    Image Layer 2 (RO)     | ← Storage driver presents
  +---------------------------+   unified view of all layers
  |    Image Layer 1 (RO)     |
  +---------------------------+
  
  Storage Driver provides:
  1. Layer stacking (union mount)
  2. Copy-on-write for container layer
  3. Efficient storage via deduplication
  4. Shared read-only layers across containers
```

---

## 5.2 Available Storage Drivers

```
  Storage Driver Options:
  
  Driver        Backing FS     Status
  +-----------+-------------+------------------+
  | overlay2  | xfs, ext4   | Recommended      |
  | fuse-     | Any         | Rootless only     |
  | overlayfs |             |                   |
  | btrfs     | btrfs       | Native btrfs     |
  | zfs       | zfs         | Native ZFS       |
  | vfs       | Any         | No CoW (testing) |
  | devicemap | direct-lvm  | Legacy           |
  | aufs      | ext4, xfs   | Deprecated       |
  +-----------+-------------+------------------+
  
  Default: overlay2 (recommended for most workloads)
```

### overlay2 — The Default

```bash
# Check current storage driver
docker info | grep "Storage Driver"
# Storage Driver: overlay2

# How overlay2 works
# Uses Linux OverlayFS kernel module
# Requires: Linux kernel 4.0+ with overlay support
# Backing filesystem: xfs (recommended) or ext4
```

```
  overlay2 Architecture:
  
  +------ merged (unified view) ------+
  |  file_a  file_b  file_c  file_d   |
  +------------------------------------+
       ^        ^        ^        ^
       |        |        |        |
  +----+---+  +-+------+-+--+  +-+------+
  | upper  |  |   lower layers  |  base  |
  | (rw)   |  |   (read-only)   |        |
  | file_d |  | file_b  file_c  | file_a |
  +--------+  +-----------------+ +------+
  
  OverlayFS merges upper (writable) and
  lower (read-only) directories into a
  single unified view at the merged dir
```

---

## 5.3 Copy-on-Write In Detail

```
  Copy-on-Write (CoW) Process:
  
  Step 1: Read file_b           Step 2: Modify file_b
  (found in lower layer)        (copied to upper, modified)
  
  Upper:  [empty]               Upper: [file_b'] ← modified
  Lower:  [file_a] [file_b]    Lower: [file_a] [file_b]
  Merged: [file_a] [file_b]    Merged:[file_a] [file_b']
  
  Step 3: Delete file_a         Step 4: Create file_c
  (whiteout in upper layer)     (written directly to upper)
  
  Upper:  [.wh.file_a][file_b'] Upper: [.wh.file_a][file_b'][file_c]
  Lower:  [file_a] [file_b]    Lower: [file_a] [file_b]
  Merged: [file_b']            Merged:[file_b'] [file_c]
  
  .wh. = "whiteout" file (marks deletion)
```

### CoW Performance Impact

```bash
# CoW is efficient for reads but adds overhead for writes
# First write to a file: COPY entire file + modify
# Subsequent writes: Direct (file already in upper layer)

# Large file example:
# 1GB database file in image layer
# Container modifies 1 byte
# → Entire 1GB copied to writable layer!
# → This is why databases should use VOLUMES
```

---

## 5.4 Configuring Storage Drivers

```json
// /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true",
    "overlay2.size=10G"
  ]
}
```

```bash
# Restart Docker after configuration change
sudo systemctl restart docker

# WARNING: Changing storage drivers makes
# existing images and containers inaccessible!
# Back up everything first

# Check storage driver details
docker info
# Storage Driver: overlay2
#  Backing Filesystem: xfs
#  Supports d_type: true
#  Using metacopy: false
#  Native Overlay Diff: true
```

---

## 5.5 Storage Driver vs Volume Performance

```
  Write Performance Comparison:
  
  Write-Heavy Workloads (e.g., Database):
  
  +----- Container Layer (CoW) -----+
  | First write: Copy + Modify      |
  | Performance: SLOWER              |
  | Use for: App binaries, configs  |
  +----------------------------------+
  
  +----- Docker Volume -----+
  | Direct I/O to filesystem |
  | Performance: NATIVE      |
  | Use for: Databases, logs |
  +---------------------------+
  
  Rule: Use volumes for ANY write-heavy workload!
  
  Benchmarks (approximate):
  +---------------+-----------+----------+
  | Operation     | CoW Layer | Volume   |
  +---------------+-----------+----------+
  | Sequential W  | 50-70%    | 100%     |
  | Random W      | 30-50%    | 100%     |
  | Sequential R  | 95-100%   | 100%     |
  | Random R      | 90-95%    | 100%     |
  +---------------+-----------+----------+
  (% of native filesystem performance)
```

---

## 5.6 Inspecting Storage Usage

```bash
# Check Docker disk usage
docker system df
# TYPE           TOTAL   ACTIVE   SIZE      RECLAIMABLE
# Images         15      5        3.2GB     2.1GB (65%)
# Containers     8       3        150MB     100MB (66%)
# Local Volumes  10      4        500MB     200MB (40%)
# Build Cache    20      0        1.5GB     1.5GB (100%)

# Detailed disk usage
docker system df -v

# Check specific container size
docker ps -s
# Shows SIZE column: writable layer / virtual size

# Inspect overlay2 layers
ls /var/lib/docker/overlay2/
# Each directory = one layer
```

---

## Summary Table

| Storage Driver | Status | Backing FS | Best For |
|----------------|--------|------------|----------|
| **overlay2** | Recommended | xfs, ext4 | Most workloads |
| **btrfs** | Supported | btrfs | Btrfs filesystems |
| **zfs** | Supported | zfs | ZFS filesystems |
| **vfs** | Testing only | Any | No CoW, simple |
| **devicemapper** | Deprecated | direct-lvm | Legacy systems |
| **aufs** | Deprecated | ext4, xfs | Legacy Ubuntu |

| Concept | Description |
|---------|-------------|
| **Union filesystem** | Merges multiple layers into single view |
| **Copy-on-Write** | Copies file to writable layer on first modification |
| **Whiteout file** | Marks a deleted file in the upper layer |
| **overlay2** | Default driver using Linux OverlayFS |
| **Performance tip** | Use volumes for write-heavy workloads |

---

## Quick Revision Questions

1. **What does a storage driver do in Docker?**
   - It implements the union filesystem that stacks image layers and provides a copy-on-write writable layer for containers

2. **What is the recommended storage driver and why?**
   - overlay2 is recommended. It uses the Linux kernel's OverlayFS, is well-tested, performant, and supported on most Linux distributions

3. **Why should databases use volumes instead of the container layer?**
   - The container layer uses copy-on-write, which copies entire files on first modification. For write-heavy workloads, this causes significant I/O overhead. Volumes provide direct filesystem I/O

4. **What is a whiteout file?**
   - A special file created in the upper (writable) layer to mark a file as deleted. The file still exists in lower layers but is hidden from the merged view

5. **What happens to existing containers when you change the storage driver?**
   - All existing images, containers, and layers become inaccessible. You must back up data and pull/rebuild images after changing the storage driver

6. **How do you check which storage driver Docker is using?**
   - Run `docker info | grep "Storage Driver"` or `docker info` for full details including backing filesystem and options

---

[← Previous: tmpfs Mounts](04-tmpfs-mounts.md) | [Next Unit: Docker Compose →](../08-Docker-Compose/01-compose-overview.md)
