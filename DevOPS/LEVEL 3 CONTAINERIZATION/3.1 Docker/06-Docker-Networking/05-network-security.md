# Chapter 5: Network Security

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

Docker network security involves isolating containers, controlling traffic flow, encrypting communication, and minimizing the attack surface. Proper network segmentation is one of the most important security measures in containerized environments.

---

## 5.1 Network Isolation

```
  Isolation via Separate Networks:
  
  +----- frontend -----+     +----- backend ------+
  |                     |     |                     |
  | +------+  +------+ |     | +------+  +------+  |
  | | web  |  | api  |-+-----+-| api  |  | db   |  |
  | +------+  +------+ |     | +------+  +------+  |
  |                     |     |                     |
  +---------------------+     +---------------------+
  
  web → api: ✓ (same frontend network)
  api → db:  ✓ (same backend network)
  web → db:  ✗ (different networks - BLOCKED)
  
  The api container bridges both networks,
  acting as a controlled gateway.
```

```bash
# Create isolated networks
docker network create frontend
docker network create backend --internal  # No internet access

# Deploy with isolation
docker run -d --name web --network frontend -p 80:80 nginx
docker run -d --name api --network frontend myapi
docker network connect backend api
docker run -d --name db --network backend postgres
```

---

## 5.2 Inter-Container Communication (ICC)

```bash
# Disable ICC on a bridge (containers can't talk to each other)
docker network create \
  --opt com.docker.network.bridge.enable_icc=false \
  isolated-net

# With ICC disabled, containers on the same bridge
# can ONLY communicate through published ports
```

```
  ICC Enabled (default):         ICC Disabled:
  
  +------+    +------+          +------+    +------+
  | C1   |--->| C2   |          | C1   | X  | C2   |
  | .0.2 |<---| .0.3 |          | .0.2 |    | .0.3 |
  +------+    +------+          +------+    +------+
  
  Direct communication           Only through published
  between containers             ports (via host)
```

---

## 5.3 Port Binding Security

```bash
# BAD: Bind to all interfaces (0.0.0.0)
docker run -d -p 8080:80 nginx
# Accessible from anywhere on the network!

# GOOD: Bind to localhost only
docker run -d -p 127.0.0.1:8080:80 nginx
# Only accessible from the host machine

# GOOD: Bind to specific interface
docker run -d -p 10.0.0.5:8080:80 nginx
# Only accessible via this IP
```

```
  Port Binding:
  
  -p 8080:80                    -p 127.0.0.1:8080:80
  
  Internet                      Internet
     |                              |
     v ✓ Accessible                 X BLOCKED
  +------+                      +------+
  | Host |                      | Host |
  | :8080|                      | :8080| (localhost only)
  +------+                      +------+
     |                              |
     v                              v
  Container:80                  Container:80
```

### Default Binding Address

```json
// /etc/docker/daemon.json
{
  "ip": "127.0.0.1"
}
// All published ports default to 127.0.0.1
// instead of 0.0.0.0
```

---

## 5.4 Encrypted Overlay Networks

```bash
# Create encrypted overlay (Swarm mode)
docker network create -d overlay --opt encrypted secure-overlay

# Traffic between nodes is encrypted using IPsec
# ESP (Encapsulating Security Payload) in tunnel mode
```

```
  Unencrypted Overlay:           Encrypted Overlay:
  
  Host A        Host B          Host A        Host B
  +-----+      +-----+         +-----+      +-----+
  | C1  |      | C2  |         | C1  |      | C2  |
  +--+--+      +--+--+         +--+--+      +--+--+
     |            |                |            |
  VXLAN         VXLAN           VXLAN+IPsec  VXLAN+IPsec
     |            |                |            |
  ===+============+===          ===+============+===
     Plaintext traffic            Encrypted traffic
     on the wire                  on the wire
```

---

## 5.5 iptables and Docker

```bash
# Docker manipulates iptables for:
# 1. Port publishing (DNAT rules)
# 2. Container-to-internet (MASQUERADE/SNAT)
# 3. Inter-container communication (FORWARD)

# View Docker's iptables rules (Linux)
sudo iptables -L -n -v
sudo iptables -t nat -L -n -v

# Docker uses these chains:
# DOCKER         — Container port forwarding
# DOCKER-USER    — User rules (applied BEFORE Docker rules)
# DOCKER-ISOLATION — Network isolation
```

### Custom Firewall Rules

```bash
# Add rules to DOCKER-USER chain (persists across restarts)
# These are evaluated BEFORE Docker's own rules

# Block external access to port 8080
sudo iptables -I DOCKER-USER -p tcp --dport 8080 \
  -j DROP -i eth0

# Allow only specific IP
sudo iptables -I DOCKER-USER -p tcp --dport 8080 \
  -s 10.0.0.0/24 -j ACCEPT
sudo iptables -I DOCKER-USER -p tcp --dport 8080 \
  -j DROP

# IMPORTANT: Don't modify DOCKER chain directly!
# Docker recreates it. Use DOCKER-USER instead.
```

---

## 5.6 Network Security Best Practices

```
  Network Security Checklist:
  
  +----+------------------------------------------------+
  | #  | Practice                                       |
  +----+------------------------------------------------+
  | 1  | Use user-defined networks (not default bridge) |
  | 2  | Create separate networks per tier              |
  | 3  | Bind ports to 127.0.0.1 when possible         |
  | 4  | Use --internal for networks that don't need    |
  |    | internet access                                |
  | 5  | Disable ICC if containers shouldn't talk       |
  | 6  | Use encrypted overlay for multi-host           |
  | 7  | Never run with --network host unless required  |
  | 8  | Use DOCKER-USER iptables chain for custom rules|
  | 9  | Don't expose ports that aren't needed          |
  | 10 | Use TLS for inter-service communication        |
  +----+------------------------------------------------+
```

---

## 5.7 Securing Docker Daemon

```bash
# Protect Docker socket
# Docker socket gives ROOT access to the host!

# Default: Unix socket (local only)
# /var/run/docker.sock

# Remote access with TLS
{
  "tls": true,
  "tlscacert": "/etc/docker/ca.pem",
  "tlscert": "/etc/docker/server-cert.pem",
  "tlskey": "/etc/docker/server-key.pem",
  "tlsverify": true,
  "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
}

# NEVER expose Docker daemon without TLS!
# Port 2375 = unencrypted (DANGEROUS)
# Port 2376 = TLS encrypted
```

---

## Summary Table

| Security Measure | Implementation | Protection |
|-----------------|----------------|------------|
| **Network isolation** | Separate user-defined networks | Container segmentation |
| **Internal networks** | `--internal` flag | No internet access |
| **ICC disable** | `enable_icc=false` | No container-to-container |
| **Localhost binding** | `-p 127.0.0.1:port:port` | No external access |
| **Encrypted overlay** | `--opt encrypted` | Wire-level encryption |
| **DOCKER-USER chain** | iptables rules | Custom firewall rules |
| **TLS daemon** | TLS certificates | Secure remote access |

---

## Quick Revision Questions

1. **How do you prevent containers from communicating with each other on the same network?**
   - Disable Inter-Container Communication with `--opt com.docker.network.bridge.enable_icc=false` on the network

2. **Why is `-p 8080:80` potentially dangerous?**
   - It binds to 0.0.0.0 (all interfaces), making the port accessible from anywhere on the network. Use `-p 127.0.0.1:8080:80` to restrict to localhost

3. **What is the DOCKER-USER iptables chain?**
   - A chain where administrators can add custom firewall rules that are evaluated before Docker's own rules and persist across container restarts

4. **How do you create a network with no internet access?**
   - Use the `--internal` flag: `docker network create --internal mynet`

5. **How is overlay network traffic encrypted?**
   - Using IPsec (ESP in tunnel mode) when the overlay is created with `--opt encrypted`. This encrypts all VXLAN traffic between Docker hosts

6. **Why should the Docker socket never be exposed without TLS?**
   - The Docker socket provides root-level access to the host. Anyone who can reach it can create privileged containers, mount the host filesystem, and gain complete host control

---

[← Previous: DNS and Service Discovery](04-dns-and-service-discovery.md) | [Next: Network Troubleshooting →](06-network-troubleshooting.md)
