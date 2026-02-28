# Chapter 5: Volumes in Compose

> **Unit 8: Docker Compose** | [Course Home](../README.md)

---

## Overview

Docker Compose simplifies volume management by defining volumes alongside services in a single file. This chapter covers named volumes, bind mounts, tmpfs, and volume sharing patterns within Compose applications.

---

## 5.1 Volume Types in Compose

```yaml
services:
  app:
    image: myapp
    volumes:
      # Named volume
      - appdata:/app/data

      # Bind mount (short syntax)
      - ./src:/app/src

      # Bind mount read-only
      - ./config.yml:/app/config.yml:ro

      # Anonymous volume
      - /app/tmp

      # Long syntax (more explicit)
      - type: volume
        source: appdata
        target: /app/data

      - type: bind
        source: ./src
        target: /app/src

      - type: tmpfs
        target: /app/cache
        tmpfs:
          size: 100000000    # 100MB

volumes:
  appdata:                   # Declare named volume
```

```
  Volume Types in Compose:
  
  compose.yaml
  +--------------------------------------------+
  | volumes:                                    |
  |   - appdata:/data        Named Volume      |
  |     (Docker-managed)                        |
  |                                             |
  |   - ./src:/app/src       Bind Mount        |
  |     (host directory)                        |
  |                                             |
  |   - /app/tmp             Anonymous Volume  |
  |     (auto-named)                            |
  |                                             |
  |   - type: tmpfs          tmpfs Mount       |
  |     (in-memory)                             |
  +--------------------------------------------+
```

---

## 5.2 Named Volumes

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret

  redis:
    image: redis:alpine
    volumes:
      - redisdata:/data
    command: redis-server --appendonly yes

volumes:
  pgdata:                     # Default local driver
  redisdata:
    driver: local             # Explicit driver
    labels:
      com.example.env: production
```

```
  Named Volume Lifecycle:
  
  docker compose up:
  +----------+     +-----------+
  | pgdata   |<--->| db :5432  |
  | (created)|     | /var/lib/ |
  +----------+     +-----------+
  
  docker compose down:
  +----------+     +-----------+
  | pgdata   |     | Removed   |
  | (KEPT!)  |     |           |
  +----------+     +-----------+
  
  docker compose down -v:
  +----------+     +-----------+
  | REMOVED  |     | Removed   |
  |          |     |           |
  +----------+     +-----------+
  
  Named volumes survive compose down
  unless -v flag is used!
```

---

## 5.3 Bind Mounts for Development

```yaml
# Development compose with bind mounts
services:
  frontend:
    build: ./frontend
    volumes:
      - ./frontend/src:/app/src          # Live code reload
      - ./frontend/public:/app/public    # Static assets
      - /app/node_modules                # Isolate node_modules
    ports:
      - "3000:3000"
    command: npm run dev

  backend:
    build: ./backend
    volumes:
      - ./backend/src:/app/src
      - /app/node_modules
    ports:
      - "8080:8080"
    command: npm run dev

  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data  # Persistent volume
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro

volumes:
  pgdata:
```

```
  Development Volume Pattern:
  
  Host (IDE)                     Containers
  +------------------+
  | frontend/        |
  |   src/ ----------+---------> /app/src (bind mount)
  |   public/ -------+---------> /app/public (bind mount)
  |                  |     +---> /app/node_modules (volume)
  +------------------+     |
                           |    Isolated from host!
  +------------------+     |
  | backend/         |     |
  |   src/ ----------+---------> /app/src (bind mount)
  |                  |
  +------------------+
  
  Edit in IDE → Instantly reflected in container
                → Hot reload picks up changes
```

---

## 5.4 Sharing Volumes Between Services

```yaml
services:
  # Upload handler writes files
  api:
    image: myapi
    volumes:
      - uploads:/app/uploads

  # Web server serves uploaded files
  web:
    image: nginx
    volumes:
      - uploads:/usr/share/nginx/html/uploads:ro
    ports:
      - "80:80"

  # Backup job reads files
  backup:
    image: alpine
    volumes:
      - uploads:/data:ro
    command: tar czf /backup/uploads.tar.gz -C /data .
    profiles:
      - maintenance

volumes:
  uploads:
```

```
  Shared Volume Pattern:
  
  +--------+    +--------+    +--------+
  |  api   |    |  web   |    | backup |
  |  (rw)  |    |  (ro)  |    |  (ro)  |
  +---+----+    +---+----+    +---+----+
      |             |             |
      v             v             v
  +---+-------------+-------------+---+
  |           uploads volume           |
  |   /var/lib/docker/volumes/...     |
  +------------------------------------+
  
  api uploads files → web serves them
  backup reads for archival
```

---

## 5.5 External Volumes

```yaml
services:
  app:
    image: myapp
    volumes:
      - existing_data:/app/data

volumes:
  existing_data:
    external: true               # Must already exist
    name: my-shared-data         # Actual volume name
```

```bash
# Create volume before compose up
docker volume create my-shared-data

# If volume doesn't exist, compose up will fail:
# Error: volume "my-shared-data" not found
```

---

## 5.6 Volume Configuration Options

```yaml
volumes:
  # Default local volume
  simple:

  # NFS volume
  nfs_data:
    driver: local
    driver_opts:
      type: nfs
      o: "addr=nfs-server.local,rw,nfsvers=4"
      device: ":/exports/data"

  # CIFS/SMB volume
  smb_data:
    driver: local
    driver_opts:
      type: cifs
      o: "addr=fileserver,username=user,password=pass,file_mode=0777,dir_mode=0777"
      device: "//fileserver/share"

  # Volume with labels
  labeled:
    labels:
      com.example.project: "myapp"
      com.example.environment: "production"
```

---

## 5.7 Volume Patterns Comparison

```
  Volume Pattern Decision:
  
  What do you need?
  
  Persistent app data     → Named Volume
  (databases, uploads)      volumes:
                              dbdata:
  
  Development live reload → Bind Mount
  (source code)             - ./src:/app/src
  
  Config injection        → Bind Mount (ro)
  (single files)            - ./nginx.conf:/etc/nginx.conf:ro
  
  Temp processing files   → tmpfs
  (in-memory)               type: tmpfs, target: /tmp
  
  Shared between services → Named Volume
  (uploads, assets)         Both mount same volume
  
  Pre-existing data       → External Volume
  (from another project)    external: true
```

---

## Summary Table

| Volume Type | Syntax | Persists | Use Case |
|-------------|--------|----------|----------|
| **Named volume** | `mydata:/path` | Yes (survives `down`) | Databases, app data |
| **Bind mount** | `./src:/app/src` | Yes (on host) | Development, config |
| **Anonymous** | `/app/tmp` | Until removed | Temp isolation |
| **tmpfs** | `type: tmpfs` | No (RAM only) | Secrets, cache |
| **External** | `external: true` | Pre-existing | Cross-project sharing |
| **Read-only** | `:ro` suffix | Depends on type | Config, security |

| Command | Purpose |
|---------|---------|
| `docker compose down` | Keep volumes |
| `docker compose down -v` | Remove volumes |
| `docker compose down --remove-orphans` | Remove unused services |
| `docker volume ls` | List all volumes |
| `docker volume prune` | Remove unused volumes |

---

## Quick Revision Questions

1. **What happens to named volumes when you run `docker compose down`?**
   - Named volumes are preserved. They are only removed with `docker compose down -v`

2. **How do you share data between two services in Compose?**
   - Define a named volume in the `volumes:` section and mount it in both services. Use `:ro` on consumers that shouldn't write

3. **How do you prevent host node_modules from overriding container packages?**
   - Add an anonymous volume for node_modules: `- /app/node_modules`. This creates a separate volume that takes precedence over the bind mount

4. **What is an external volume?**
   - A volume created outside of Compose (e.g., `docker volume create`). Referenced with `external: true` in the compose file. Compose won't create or delete it

5. **When should you use bind mounts vs named volumes in Compose?**
   - Bind mounts for development (live code editing) and config file injection. Named volumes for persistent data that should survive container recreation

---

[← Previous: Networking in Compose](04-networking-in-compose.md) | [Next: Environment and Secrets →](06-environment-and-secrets.md)
