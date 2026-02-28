# Chapter 4: DNS and Service Discovery

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

Docker provides an **embedded DNS server** for containers on user-defined networks, enabling automatic service discovery by container name. This is a fundamental mechanism for microservice communication.

---

## 4.1 Docker's Embedded DNS

```
  Container
  +------------------------------+
  | /etc/resolv.conf             |
  | nameserver 127.0.0.11       |----+
  | options ndots:0              |    |
  +------------------------------+    |
                                      v
                              +---------------+
                              | Docker DNS    |
                              | 127.0.0.11    |
                              |               |
                              | Container     |
                              | name → IP     |
                              | alias → IP    |
                              | service → IP  |
                              |               |
                              | Falls back to |
                              | host DNS for  |
                              | external names|
                              +---------------+
  
  Only works on USER-DEFINED networks
  NOT on the default bridge network!
```

---

## 4.2 DNS Resolution Types

```bash
# Container name resolution
docker network create mynet
docker run -d --name api --network mynet myapi
docker run -d --name db --network mynet postgres

docker exec api nslookup db
# Server:    127.0.0.11
# Name:      db
# Address:   172.20.0.3

# Network alias resolution
docker run -d --name postgres-primary \
  --network mynet \
  --network-alias db \
  --network-alias database \
  postgres

docker exec api nslookup db        # Resolves
docker exec api nslookup database  # Also resolves
```

---

## 4.3 Network Aliases for Load Balancing

```bash
# Multiple containers with same alias
docker network create mynet

docker run -d --name api-1 --network mynet --network-alias api myapi
docker run -d --name api-2 --network mynet --network-alias api myapi
docker run -d --name api-3 --network mynet --network-alias api myapi

# DNS returns all IPs (round-robin)
docker exec client nslookup api
# Name: api
# Address: 172.20.0.2
# Address: 172.20.0.3
# Address: 172.20.0.4
```

```
  DNS Round-Robin Load Balancing:
  
  Client                   Docker DNS
  +--------+              +-----------+
  | nslookup|   "api" --> | api-1: .2 |
  | api     |             | api-2: .3 |
  +----+----+             | api-3: .4 |
       |                  +-----------+
       v
  Returns all 3 IPs
  (DNS round-robin)
  
  Request 1 → api-1 (172.20.0.2)
  Request 2 → api-2 (172.20.0.3)
  Request 3 → api-3 (172.20.0.4)
  
  NOTE: DNS round-robin is basic load balancing.
  Use a proper load balancer for production.
```

---

## 4.4 Custom DNS Configuration

```bash
# Custom DNS servers
docker run --dns 8.8.8.8 --dns 8.8.4.4 myapp

# Custom search domains
docker run --dns-search example.com myapp
# Container can resolve "api" as "api.example.com"

# Custom hostname
docker run --hostname myhost myapp

# Custom /etc/hosts entries
docker run --add-host db:192.168.1.100 myapp
docker run --add-host "host.docker.internal:host-gateway" myapp
# host-gateway resolves to the host's IP

# Custom domain name
docker run --domainname example.com myapp
```

### Daemon-Level DNS Configuration

```json
// /etc/docker/daemon.json
{
  "dns": ["8.8.8.8", "8.8.4.4"],
  "dns-search": ["example.com"],
  "dns-opts": ["ndots:1"]
}
```

---

## 4.5 Service Discovery Patterns

### Pattern 1: Direct Container Name

```bash
# Simplest — use container name
docker run -d --name db --network mynet postgres
docker run -d --name api --network mynet \
  -e DATABASE_URL=postgres://user:pass@db:5432/mydb myapi
#                                       ^^
#                            Container name as hostname
```

### Pattern 2: Network Aliases

```bash
# Decouple container name from service name
docker run -d --name postgres-primary-v16 \
  --network mynet \
  --network-alias db \
  postgres:16

# Applications connect to "db", not the container name
# Easy to swap implementations
```

### Pattern 3: Multi-Network Discovery

```bash
# API on both frontend and backend
docker network create frontend
docker network create backend

docker run -d --name api --network frontend myapi
docker network connect --alias api-service backend api

# api is reachable as:
# "api" on frontend network
# "api-service" on backend network
```

---

## 4.6 Debugging DNS Issues

```bash
# Check container DNS config
docker exec mycontainer cat /etc/resolv.conf

# Test DNS resolution
docker exec mycontainer nslookup target-container
docker exec mycontainer dig target-container

# Use debug container
docker run --rm --network mynet nicolaka/netshoot \
  nslookup api

# Check if containers are on same network
docker network inspect mynet -f '{{json .Containers}}' | jq

# Common DNS issues:
# 1. Containers on different networks
# 2. Using default bridge (no DNS)
# 3. Container not running
# 4. Typo in container/alias name
```

---

## 4.7 Connecting to Host Services

```bash
# Access host machine from container

# Docker Desktop (Mac/Windows)
docker run --rm myapp curl http://host.docker.internal:3000

# Linux
docker run --add-host host.docker.internal:host-gateway \
  myapp curl http://host.docker.internal:3000

# Or use host's IP directly
docker run --rm myapp curl http://172.17.0.1:3000
```

```
  Container → Host Communication:
  
  +---------------------------+
  | Host Machine              |
  |                           |
  | App running on :3000      |
  |          ^                |
  |          |                |
  |  +-------+------------+  |
  |  | Container          |  |
  |  |                    |  |
  |  | curl host.docker.  |  |
  |  |   internal:3000    |  |
  |  +--------------------+  |
  +---------------------------+
```

---

## Summary Table

| Feature | Description | Scope |
|---------|-------------|-------|
| **Embedded DNS** | 127.0.0.11 in containers | User-defined networks |
| **Name resolution** | Container name → IP | Same network |
| **Network aliases** | Multiple names for one container | Per-network |
| **DNS round-robin** | Same alias on multiple containers | Basic load balancing |
| **Custom DNS** | `--dns`, `--dns-search` | Per-container |
| **Host access** | `host.docker.internal` | Container → host |
| **Extra hosts** | `--add-host` | Custom /etc/hosts |

---

## Quick Revision Questions

1. **Why doesn't DNS work on the default bridge network?**
   - The default bridge network uses a legacy networking model that doesn't include Docker's embedded DNS server. Only user-defined networks get automatic DNS resolution

2. **What is a network alias and how is it different from a container name?**
   - A network alias is an additional DNS name assigned to a container on a specific network. Unlike container names which are unique, multiple containers can share the same alias, enabling DNS round-robin

3. **How does Docker implement basic load balancing with DNS?**
   - When multiple containers share the same network alias, Docker's DNS returns all their IPs. Clients receive different IPs in round-robin order

4. **How does a container connect to a service on the host?**
   - Use `host.docker.internal` as the hostname. On Linux, add `--add-host host.docker.internal:host-gateway` since it's not automatic

5. **What DNS server address is used inside Docker containers?**
   - 127.0.0.11 — Docker's embedded DNS server that intercepts DNS queries and resolves container names, then falls back to host DNS for external names

---

[← Previous: Host and None Networking](03-host-and-none-networking.md) | [Next: Network Security →](05-network-security.md)
