# Chapter 2: Volumes

> **Unit 7: Docker Volumes** | [Course Home](../README.md)

---

## Overview

Docker volumes are the preferred mechanism for persisting data generated and used by containers. They are completely managed by Docker, independent of the host filesystem structure, and work on both Linux and Windows containers.

---

## 2.1 Creating and Managing Volumes

```bash
# Create a named volume
docker volume create mydata

# Create with specific driver and options
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/shared \
  nfs_data

# List all volumes
docker volume ls

# Inspect a volume
docker volume inspect mydata
# Output:
# [{ "Name": "mydata",
#    "Driver": "local",
#    "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
#    "Labels": {},
#    "Scope": "local" }]

# Remove a volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
docker volume prune --all  # Including named volumes
```

---

## 2.2 Using Volumes with Containers

```bash
# Named volume
docker run -d \
  --name db \
  --mount type=volume,source=pgdata,target=/var/lib/postgresql/data \
  postgres:16

# Same with -v shorthand
docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16

# Anonymous volume (Docker generates name)
docker run -d -v /var/lib/postgresql/data postgres:16

# Read-only volume
docker run -d \
  --mount type=volume,source=config,target=/etc/myapp,readonly \
  myapp

# Same with -v shorthand
docker run -d -v config:/etc/myapp:ro myapp
```

```
  Volume Lifecycle:
  
  1. Create              2. Mount              3. Use
  docker volume create   docker run -v ...     Container R/W
  +--------+             +--------+            +--------+
  | mydata |             | mydata | <--------> | /data  |
  | (empty)|             | (empty)|            | in C1  |
  +--------+             +--------+            +--------+
  
  4. Stop Container      5. New Container      6. Data persists
  docker stop C1         docker run -v ...     Same data!
  +--------+             +--------+            +--------+
  | mydata |             | mydata | <--------> | /data  |
  | (data) |             | (data) |            | in C2  |
  +--------+             +--------+            +--------+
```

---

## 2.3 Volume Content Pre-population

```
  Pre-population Behavior:
  
  Image has files at /data:     Volume is empty:
  +------------------+          +------------------+
  | /data/           |          | myvolume/        |
  |   config.json    |   --->   |   config.json    |
  |   defaults.yml   |  mount   |   defaults.yml   |
  +------------------+          +------------------+
  
  Docker copies image content INTO the volume
  (only if volume is empty on first mount)
  
  IMPORTANT: This ONLY works with volumes,
  NOT with bind mounts!
```

```bash
# Example: nginx default content
docker volume create web_content
docker run -d -v web_content:/usr/share/nginx/html nginx:alpine

# The volume now contains nginx's default HTML files
# This pre-population only happens once (when volume is empty)
```

---

## 2.4 Sharing Volumes Between Containers

```bash
# Container 1: Write data
docker run -d --name writer \
  -v shared_data:/data \
  alpine sh -c "while true; do date >> /data/log.txt; sleep 5; done"

# Container 2: Read data
docker run -it --name reader \
  -v shared_data:/data:ro \
  alpine tail -f /data/log.txt
```

```
  Shared Volume Pattern:
  
  +----------+     +----------+
  | Writer   |     | Reader   |
  | Container|     | Container|
  +----+-----+     +----+-----+
       |                |
       v (rw)           v (ro)
  +----+----------------+-----+
  |      shared_data           |
  |   /var/lib/docker/volumes  |
  +----------------------------+
  
  Caution: No built-in locking!
  Two writers = potential data corruption
  Use databases or file locks for concurrent writes
```

---

## 2.5 Volume Backup and Restore

```bash
# Backup: Mount volume in temp container, tar the data
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/mydata-backup.tar.gz -C /source .

# Restore: Extract tar into volume
docker volume create mydata_restored
docker run --rm \
  -v mydata_restored:/target \
  -v $(pwd):/backup:ro \
  alpine tar xzf /backup/mydata-backup.tar.gz -C /target

# Copy between volumes
docker run --rm \
  -v source_vol:/source:ro \
  -v dest_vol:/dest \
  alpine sh -c "cp -a /source/. /dest/"
```

```
  Backup Workflow:
  
  +----------+     +-----------+     +----------+
  | mydata   |     | Temp      |     | Host     |
  | volume   |---->| Container |---->| Backup   |
  | /source  |     | tar czf   |     | .tar.gz  |
  +----------+     +-----------+     +----------+
  
  Restore Workflow:
  
  +----------+     +-----------+     +----------+
  | Host     |     | Temp      |     | new_vol  |
  | Backup   |---->| Container |---->| volume   |
  | .tar.gz  |     | tar xzf   |     | /target  |
  +----------+     +-----------+     +----------+
```

---

## 2.6 Volume Drivers

```bash
# Default: local driver (stores on host filesystem)
docker volume create --driver local mydata

# Third-party drivers for external storage:
# - REX-Ray:    AWS EBS, S3, Azure, GCE
# - NetApp:     ONTAP, SolidFire
# - Portworx:   Distributed storage
# - GlusterFS:  Distributed filesystem
# - NFS:        Network File System

# NFS volume (using local driver)
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=nfs-server.example.com,rw \
  --opt device=:/exports/data \
  nfs_volume

# CIFS/SMB volume (Windows shares)
docker volume create \
  --driver local \
  --opt type=cifs \
  --opt o=addr=fileserver,username=user,password=pass \
  --opt device=//fileserver/share \
  smb_volume
```

---

## 2.7 Volume Labels and Filtering

```bash
# Create volume with labels
docker volume create --label project=myapp --label env=prod appdata

# Filter volumes by label
docker volume ls --filter label=project=myapp
docker volume ls --filter label=env=prod

# Filter by driver
docker volume ls --filter driver=local

# Filter dangling (unused) volumes
docker volume ls --filter dangling=true
```

---

## Summary Table

| Operation | Command |
|-----------|---------|
| **Create volume** | `docker volume create mydata` |
| **List volumes** | `docker volume ls` |
| **Inspect volume** | `docker volume inspect mydata` |
| **Remove volume** | `docker volume rm mydata` |
| **Prune unused** | `docker volume prune` |
| **Mount (run)** | `-v mydata:/path` or `--mount type=volume,...` |
| **Read-only** | `-v mydata:/path:ro` |
| **Anonymous** | `-v /container/path` |
| **Backup** | Mount in temp container, tar the data |
| **Pre-populate** | Auto-copies image content to empty volume |

---

## Quick Revision Questions

1. **What is the difference between a named volume and an anonymous volume?**
   - Named volumes have a user-specified name and are easy to reference. Anonymous volumes get a random hash name and are harder to manage

2. **How does volume content pre-population work?**
   - When a volume is first mounted to a container, if the volume is empty, Docker copies the contents from the image at that mount point into the volume (only works with volumes, not bind mounts)

3. **How do you make a volume read-only?**
   - Add `:ro` flag: `-v mydata:/path:ro` or use `readonly` in `--mount`: `--mount type=volume,source=mydata,target=/path,readonly`

4. **How do you backup a Docker volume?**
   - Run a temporary container that mounts both the volume and a host directory, then use tar to archive the volume contents to the host

5. **What happens to a volume when you remove a container?**
   - The volume persists independently. Named volumes must be explicitly removed with `docker volume rm`. Use `docker run --rm -v` to auto-remove anonymous volumes

6. **What is the default volume driver and where does it store data?**
   - The default driver is `local`, storing data at `/var/lib/docker/volumes/<name>/_data/` on Linux

---

[← Previous: Storage Overview](01-storage-overview.md) | [Next: Bind Mounts →](03-bind-mounts.md)
