# Chapter 5: Container Networking Basics

> **Unit 5: Docker Containers** | [Course Home](../README.md)

---

## Overview

Every container needs network connectivity to communicate with other containers, the host, and the outside world. This chapter introduces Docker's default networking model from the container perspective. (Detailed networking is covered in Unit 6.)

---

## 5.1 Default Container Networking

```
  When you run a container without --network:
  
  +--------------------------------------------------+
  |  Host Machine                                     |
  |                                                   |
  |  +-------------------------------------------+   |
  |  |  docker0 bridge (172.17.0.1)               |   |
  |  |                                            |   |
  |  |  +-------------+    +-------------+        |   |
  |  |  | Container A |    | Container B |        |   |
  |  |  | 172.17.0.2  |    | 172.17.0.3  |        |   |
  |  |  | veth1       |    | veth2       |        |   |
  |  |  +------+------+    +------+------+        |   |
  |  |         |                  |               |   |
  |  +---------+------------------+---------------+   |
  |                                                   |
  |  eth0 (Host NIC) ────── Internet                  |
  +--------------------------------------------------+
  
  Default: All containers on "bridge" network
  Containers get IP from 172.17.0.0/16 range
```

---

## 5.2 Port Publishing

```bash
# Containers are isolated by default
# Must publish ports to access from host

# Publish specific port
docker run -d -p 8080:80 nginx
# Host:8080 → Container:80

# Publish to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Random host port
docker run -d -p 80 nginx
docker port <container>   # See assigned port

# Publish all EXPOSE'd ports
docker run -d -P nginx

# Multiple ports
docker run -d -p 80:80 -p 443:443 nginx
```

```
  Port Publishing:
  
  Internet
      |
      v
  +---+---+
  | Host  |
  | :8080 | ──── docker run -p 8080:80 nginx
  +---+---+
      |
      v
  +---+---+
  | nginx |
  | :80   |
  +-------+
```

---

## 5.3 Container-to-Container Communication

```bash
# On default bridge: Use IP (not name)
docker run -d --name web nginx
docker run -d --name app myapp

# Get container IP
docker inspect -f '{{.NetworkSettings.IPAddress}}' web
# 172.17.0.2

# Connect from app to web (by IP only on default bridge)
docker exec app curl http://172.17.0.2:80

# On user-defined bridge: DNS resolution works!
docker network create mynet
docker run -d --name web --network mynet nginx
docker run -d --name app --network mynet myapp

# Connect by container name (DNS)
docker exec app curl http://web:80
# Docker provides automatic DNS resolution on user-defined networks
```

```
  Default Bridge:                 User-Defined Bridge:
  
  +------+    +------+           +------+    +------+
  | web  |    | app  |           | web  |    | app  |
  +------+    +------+           +------+    +------+
     |           |                  |           |
  IP only     IP only           DNS name     DNS name
  172.17.0.2  172.17.0.3        "web"        "app"
     |           |                  |           |
  +--+-----------+--+           +--+-----------+--+
  |  docker0        |           |  mynet          |
  +-----------------+           +-----------------+
```

---

## 5.4 Network Modes

```bash
# Bridge (default) — isolated network with NAT
docker run -d nginx

# Host — share host's network stack
docker run -d --network host nginx
# No port mapping needed, container uses host ports directly

# None — no networking
docker run -d --network none myapp
# Complete network isolation

# Container — share another container's network
docker run -d --name web nginx
docker run -d --network container:web myapp
# Both containers share same network namespace (same IP)
```

```
  Network Modes:
  
  Bridge:            Host:              None:           Container:
  +--------+        +--------+        +--------+      +--------+
  | eth0   |        | (uses  |        | (no    |      | shares |
  | 172.17.|        |  host  |        |  NIC)  |      | net    |
  | .0.2   |        |  eth0) |        |        |      | with   |
  +--------+        +--------+        +--------+      | other) |
   Isolated          Direct            No network     +--------+
   NAT               Performance       Maximum         Same IP
                                       isolation
```

---

## 5.5 DNS in Containers

```bash
# Default DNS resolution
docker run --rm alpine cat /etc/resolv.conf
# nameserver 127.0.0.11  (Docker's embedded DNS)

# Custom DNS
docker run --dns 8.8.8.8 --dns 8.8.4.4 myapp

# Custom hostname
docker run --hostname myhost myapp

# Extra hosts (like /etc/hosts entry)
docker run --add-host db:192.168.1.100 myapp
# Adds: 192.168.1.100  db  in container's /etc/hosts
```

---

## 5.6 Common Networking Patterns

### Web App + Database

```bash
# Create network
docker network create app-net

# Run database
docker run -d \
  --name db \
  --network app-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

# Run web app (connects to db by name)
docker run -d \
  --name web \
  --network app-net \
  -p 8080:3000 \
  -e DATABASE_URL=postgres://postgres:secret@db:5432/app \
  myapp
```

```
  +------------------------------------------------------+
  |  app-net (user-defined bridge)                       |
  |                                                      |
  |  +------------------+    +------------------+        |
  |  | web              |    | db               |        |
  |  | Port 3000        |--->| Port 5432        |        |
  |  | DNS: "web"       |    | DNS: "db"        |        |
  |  +------------------+    +------------------+        |
  |         |                                            |
  +---------+--------------------------------------------+
            |
    -p 8080:3000 (published to host)
            |
    Host:8080 → accessible from browser
```

---

## Summary Table

| Concept | Default Bridge | User-Defined Bridge |
|---------|---------------|-------------------|
| **DNS resolution** | By IP only | By container name |
| **Isolation** | All containers connected | Network-specific |
| **Port publishing** | `-p host:container` | `-p host:container` |
| **Auto-connect** | All new containers | Must specify `--network` |
| **Recommended** | Dev/testing only | Production use |

---

## Quick Revision Questions

1. **What network does a container join by default?**
   - The default `bridge` network (docker0). Containers get an IP in the 172.17.0.0/16 range

2. **Why should you use a user-defined bridge network instead of the default bridge?**
   - User-defined bridges provide automatic DNS resolution (containers can reach each other by name), better isolation, and can be connected/disconnected at runtime

3. **Does EXPOSE in a Dockerfile publish a port?**
   - No. EXPOSE is documentation only. You must use `-p` flag with `docker run` to actually publish and map ports to the host

4. **What is `--network host` mode?**
   - The container shares the host's network stack directly. No network isolation or port mapping — the container's ports are directly on the host

5. **How do containers on the default bridge communicate?**
   - Only by IP address. DNS name resolution between containers is NOT available on the default bridge network

6. **What happens with `--network none`?**
   - The container has no network interface (except loopback). It is completely network-isolated, useful for batch processing or security-sensitive workloads

---

[← Previous: Container Logging](04-container-logging.md) | [Next: Resource Limits →](06-resource-limits.md)
