# Chapter 2: Bridge Networking

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

The **bridge** driver is Docker's default networking mode. It creates a private internal network on the host, allowing containers to communicate with each other and with the outside world through NAT.

---

## 2.1 Default Bridge vs User-Defined Bridge

```
  Default Bridge (docker0):            User-Defined Bridge:
  
  +---------------------------+       +---------------------------+
  | docker0 (172.17.0.1)      |       | my-net (172.20.0.1)      |
  |                           |       |                           |
  | +-----+  +-----+         |       | +-----+  +-----+         |
  | | C1  |  | C2  |         |       | | C1  |  | C2  |         |
  | | .0.2|  | .0.3|         |       | | .0.2|  | .0.3|         |
  | +-----+  +-----+         |       | +-----+  +-----+         |
  |                           |       |                           |
  | ✗ No DNS resolution      |       | ✓ DNS resolution          |
  | ✗ All containers exposed  |       | ✓ Network isolation       |
  | ✗ Can't connect/disconnect|       | ✓ Live connect/disconnect |
  | ✗ Must use --link (legacy)|       | ✓ Env config isolation    |
  +---------------------------+       +---------------------------+

  ALWAYS use user-defined bridge networks!
```

---

## 2.2 Creating and Managing Bridge Networks

```bash
# Create a bridge network
docker network create mynet

# Create with options
docker network create \
  --driver bridge \
  --subnet 10.0.0.0/24 \
  --gateway 10.0.0.1 \
  --ip-range 10.0.0.128/25 \
  --opt com.docker.network.bridge.name=br-mynet \
  mynet

# List networks
docker network ls

# Inspect network
docker network inspect mynet

# Remove network
docker network rm mynet

# Remove unused networks
docker network prune
```

---

## 2.3 Connecting Containers to Networks

```bash
# Run container on specific network
docker run -d --name web --network mynet nginx

# Connect running container to additional network
docker network connect mynet existing-container

# Disconnect from network
docker network disconnect mynet existing-container

# Connect with specific IP
docker network connect --ip 10.0.0.50 mynet mycontainer

# Container on multiple networks
docker run -d --name app --network frontend nginx
docker network connect backend app
# Now 'app' is on both 'frontend' and 'backend' networks
```

```
  Multi-Network Container:
  
  +----------+                   +----------+
  | frontend |                   | backend  |
  | network  |                   | network  |
  |          |                   |          |
  | +------+ |  +------+        | +------+ |
  | | web  | |  | app  |--------+-| db   | |
  | +------+ |  | (on   |       | +------+ |
  |          |  | both!) |       |          |
  +----------+  +--------+      +----------+
  
  web can reach app (frontend)
  app can reach db (backend)
  web CANNOT reach db (different networks!)
```

---

## 2.4 DNS Resolution

```bash
# On user-defined networks, containers resolve each other by name
docker network create mynet

docker run -d --name api --network mynet myapi
docker run -d --name db --network mynet postgres

# From api container:
docker exec api ping db          # Resolves to db's IP
docker exec api nslookup db      # Shows DNS resolution

# Network aliases
docker run -d --name postgres-primary \
  --network mynet \
  --network-alias db \
  --network-alias database \
  postgres

# Both 'db' and 'database' resolve to this container
docker exec api ping db           # Works
docker exec api ping database     # Also works
```

```
  Docker Embedded DNS (127.0.0.11):
  
  Container "api"
  +------------------+
  | /etc/resolv.conf |
  | nameserver       |       +----------------+
  | 127.0.0.11  -----+-----> | Docker DNS     |
  +------------------+       | Server         |
                              | 127.0.0.11     |
  "ping db"                   |                |
       |                      | db → 172.20.0.3|
       v                      | api → 172.20.0.2
  Resolved!                   +----------------+
```

---

## 2.5 Internal Networks

```bash
# Create internal network (no external/internet access)
docker network create --internal secure-net

# Containers can talk to each other but NOT the internet
docker run -d --name internal-app --network secure-net myapp
docker exec internal-app ping google.com  # FAILS
docker exec internal-app ping other-container  # WORKS
```

```
  Regular Bridge:              Internal Bridge:
  
  Internet                     Internet
     |                            X (blocked)
     v                            |
  +--+---+                     +--+---+
  | NAT  |                     |      |
  +--+---+                     +------+
     |                            |
  +--+----------+             +--+----------+
  | bridge      |             | internal    |
  | +---+ +---+ |             | +---+ +---+ |
  | | C1| | C2| |             | | C1| | C2| |
  | +---+ +---+ |             | +---+ +---+ |
  +--------------+             +--------------+
  C1, C2 can reach             C1, C2 can talk to
  each other AND internet      each other ONLY
```

---

## 2.6 Port Publishing with Bridge Networks

```bash
# Publish port
docker run -d -p 8080:80 --network mynet nginx

# How it works internally:
#
# 1. iptables DNAT rule maps host:8080 → container:80
# 2. docker-proxy listens on host:8080 (userland proxy)
# 3. Traffic forwarded to container's IP on bridge

# Disable userland proxy (use iptables only)
# In /etc/docker/daemon.json:
# { "userland-proxy": false }
```

---

## 2.7 Bridge Network Configuration

```bash
# Default bridge options in daemon.json
{
  "bip": "172.26.0.1/16",           # Bridge IP
  "fixed-cidr": "172.26.0.0/24",    # Container IP range
  "mtu": 1500,                       # MTU
  "default-address-pools": [         # Pools for user-defined networks
    { "base": "10.10.0.0/16", "size": 24 }
  ]
}

# Custom bridge options
docker network create \
  --opt com.docker.network.bridge.enable_icc=true \
  --opt com.docker.network.bridge.enable_ip_masquerade=true \
  --opt com.docker.network.bridge.host_binding_ipv4=0.0.0.0 \
  --opt com.docker.network.driver.mtu=9000 \
  mynet
```

---

## 2.8 Practical Example: Three-Tier Application

```bash
# Create separate networks
docker network create frontend
docker network create backend

# Database (backend only)
docker run -d \
  --name db \
  --network backend \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

# API server (both networks)
docker run -d \
  --name api \
  --network backend \
  -e DATABASE_URL=postgres://postgres:secret@db:5432/app \
  myapi:latest
docker network connect frontend api

# Web server (frontend only, exposed)
docker run -d \
  --name web \
  --network frontend \
  -p 80:80 \
  -e API_URL=http://api:3000 \
  myweb:latest
```

```
  Three-Tier Network Isolation:
  
  Internet
     |
     | :80
     v
  +--+-------------------------------+
  | frontend network                  |
  |  +------+        +------+        |
  |  | web  |------->| api  |        |
  |  | :80  |        | :3000|        |
  |  +------+        +--+---+        |
  +----------------------+------------+
                         |
  +----------------------+------------+
  | backend network      |            |
  |                 +----v----+       |
  |                 | db      |       |
  |                 | :5432   |       |
  |                 +---------+       |
  +-----------d-----------------------+
  
  web → api: ✓ (both on frontend)
  api → db:  ✓ (both on backend)
  web → db:  ✗ (different networks!)
```

---

## Summary Table

| Feature | Default Bridge | User-Defined Bridge |
|---------|---------------|-------------------|
| **DNS** | IP only | Container name resolution |
| **Isolation** | All containers share | Per-network isolation |
| **Connect/Disconnect** | Cannot hot-swap | Live connect/disconnect |
| **Links** | Legacy `--link` | Built-in DNS |
| **Configuration** | Limited | Custom subnet, gateway, MTU |
| **Recommendation** | Dev only | Use in production |

---

## Quick Revision Questions

1. **Why should you always use user-defined bridge networks?**
   - They provide automatic DNS resolution, better isolation, live connect/disconnect capability, and configurable subnet options — none of which the default bridge offers

2. **How does DNS resolution work on user-defined bridges?**
   - Docker runs an embedded DNS server at 127.0.0.11 inside each container's network namespace. It resolves container names and network aliases to their IP addresses

3. **Can a container be on multiple networks?**
   - Yes. Use `docker network connect` to add a running container to additional networks. The container gets an IP on each network

4. **What is an internal network?**
   - A network created with `--internal` that blocks external/internet access. Containers can communicate with each other but cannot reach the internet

5. **How does port publishing work with bridge networks?**
   - Docker uses iptables DNAT rules and a userland proxy (docker-proxy) to forward traffic from the host port to the container's IP and port on the bridge network

6. **How do you implement network isolation for a three-tier app?**
   - Create separate networks (frontend, backend). Web server joins frontend only, API joins both, database joins backend only. This prevents direct web-to-database access

---

[← Previous: Networking Overview](01-networking-overview.md) | [Next: Host and None Networking →](03-host-and-none-networking.md)
