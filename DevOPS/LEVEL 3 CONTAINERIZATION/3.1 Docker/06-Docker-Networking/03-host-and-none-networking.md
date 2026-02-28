# Chapter 3: Host, None, and Other Network Modes

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

Beyond bridge networking, Docker provides **host**, **none**, **macvlan**, **ipvlan**, and **overlay** network drivers for specialized use cases. Each driver offers different trade-offs between isolation, performance, and features.

---

## 3.1 Host Networking

```bash
# Use host's network stack directly
docker run -d --network host nginx

# No port mapping needed — container binds directly to host ports
# nginx listens on host's port 80 directly
```

```
  Bridge Mode:                    Host Mode:
  
  +-------------------+          +-------------------+
  | Host              |          | Host              |
  |                   |          |                   |
  | +-----------+     |          | Container shares  |
  | | Container |     |          | host network      |
  | | eth0 (veth)|    |          | stack directly    |
  | | 172.17.0.2|     |          |                   |
  | +-----+-----+     |          | No NAT            |
  |       |           |          | No port mapping   |
  |   +---+----+      |          | No isolation      |
  |   | bridge |      |          | Best performance  |
  |   +---+----+      |          |                   |
  |       | NAT       |          | Container sees    |
  | +-----+------+    |          | all host ports    |
  | | eth0 (host)|    |          | and interfaces    |
  | +------------+    |          +-------------------+
  +-------------------+
```

### When to Use Host Networking

| Use Case | Reason |
|----------|--------|
| Performance-critical apps | No NAT overhead |
| Apps binding many dynamic ports | No port mapping needed |
| Network monitoring tools | Need to see host traffic |
| Legacy apps requiring specific ports | Direct binding |

### Limitations

```
  Host networking limitations:
  
  ✗ No network isolation
  ✗ Port conflicts with host services
  ✗ Port conflicts between containers
  ✗ Only works on Linux (not Docker Desktop)
  ✗ Cannot connect to other Docker networks
  ✗ Less portable
```

---

## 3.2 None Networking

```bash
# Complete network isolation
docker run -d --network none myapp

# Container has only loopback interface
docker exec mycontainer ip addr
# 1: lo: <LOOPBACK,UP>
#     inet 127.0.0.1/8
# (no other interfaces)
```

### Use Cases for None

```
  +------------------------------+
  | Container (--network none)   |
  |                              |
  | lo: 127.0.0.1 (loopback)    |
  |                              |
  | No eth0                      |
  | No internet                  |
  | No container-to-container    |
  |                              |
  | Use cases:                   |
  | - Batch processing           |
  | - Offline computation        |
  | - Security-sensitive tasks   |
  | - Cryptographic operations   |
  | - Data processing pipelines  |
  +------------------------------+
```

---

## 3.3 Macvlan Networking

```bash
# Assign a real MAC address to container
# Container appears as physical device on network

docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-macvlan

docker run -d \
  --network my-macvlan \
  --ip 192.168.1.50 \
  nginx
```

```
  Physical Network:
  
  +--------+  +--------+  +--------+  +-----------+
  | Server |  | Laptop |  | Router |  | Container |
  | .1.10  |  | .1.20  |  | .1.1   |  | .1.50     |
  +---+----+  +---+----+  +---+----+  +---+-------+
      |           |           |            |
  ====+===========+===========+============+====
                Physical Switch / Network
  
  Container has its OWN MAC address
  Container appears as another device on the network
  No NAT, no bridge, no port mapping
```

### Macvlan Modes

```
  Bridge Mode (default):          802.1q Trunk Mode:
  
  +-------------+                 +------------------+
  | Container 1 |                 | VLAN 10:         |
  | MAC: aa:bb..|                 |   Container 1    |
  +------+------+                 | VLAN 20:         |
         |                        |   Container 2    |
  +------+------+                 +--------+---------+
  | parent: eth0|                          |
  +------+------+                 +--------+---------+
         |                        | eth0.10  eth0.20 |
    Physical NIC                  |  (trunk port)    |
                                  +------------------+
```

---

## 3.4 IPvlan Networking

```bash
# Like macvlan but shares host's MAC address
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-ipvlan

# L2 mode (default) — like macvlan but same MAC
docker network create -d ipvlan \
  -o ipvlan_mode=l2 \
  -o parent=eth0 \
  --subnet=192.168.1.0/24 \
  my-ipvlan-l2

# L3 mode — routing between subnets
docker network create -d ipvlan \
  -o ipvlan_mode=l3 \
  -o parent=eth0 \
  --subnet=10.10.0.0/24 \
  my-ipvlan-l3
```

```
  Macvlan vs IPvlan:
  
  Macvlan:                      IPvlan:
  +-------+  +-------+         +-------+  +-------+
  | C1    |  | C2    |         | C1    |  | C2    |
  | MAC-A |  | MAC-B |         | MAC-H |  | MAC-H |
  +---+---+  +---+---+         +---+---+  +---+---+
      |          |                  |          |
  Each container has            All containers share
  unique MAC address            host's MAC address
  
  Use macvlan when:             Use ipvlan when:
  - Need unique MACs            - MAC address limits
  - Legacy network               (wireless, cloud)
    requirements                - Simpler switching
```

---

## 3.5 Overlay Networking

```bash
# Overlay networks for multi-host communication
# Requires Docker Swarm or manual setup

# Initialize Swarm
docker swarm init

# Create overlay network
docker network create -d overlay my-overlay

# Deploy service on overlay
docker service create --name web --network my-overlay nginx
```

```
  Overlay Network (VXLAN):
  
  Host A                          Host B
  +------------------+           +------------------+
  | +------+ +------+|           |+------+ +------+ |
  | | C1   | | C2   ||           || C3   | | C4   | |
  | +--+---+ +--+---+|           |+--+---+ +--+---+ |
  |    |        |     |           |   |        |     |
  | +--+--------+---+ |           | +-+--------+--+ |
  | | overlay network| |           | | overlay net | |
  | | 10.0.0.0/24   | |           | | 10.0.0.0/24| |
  | +-------+-------+ |           | +------+------+ |
  |         |          |           |        |        |
  |    VXLAN encap     |           |   VXLAN decap   |
  |   +-----+-----+   |           |  +-----+-----+  |
  |   | eth0       |   |           |  | eth0      |  |
  +---+---+--------+---+           +--+-----+-----+--+
          |                                  |
          +--------  Physical Network -------+
  
  VXLAN: Virtual Extensible LAN
  Encapsulates L2 frames in UDP (port 4789)
```

---

## 3.6 Network Driver Comparison

| Feature | Bridge | Host | None | Macvlan | IPvlan | Overlay |
|---------|--------|------|------|---------|--------|---------|
| **Isolation** | Good | None | Complete | Network-level | Network-level | Good |
| **Performance** | Good | Best | N/A | Excellent | Excellent | Good |
| **DNS** | User-defined only | Host DNS | N/A | Manual | Manual | Built-in |
| **Port mapping** | Yes | No | No | No | No | Yes |
| **Multi-host** | No | No | No | No | No | Yes |
| **Complexity** | Low | Low | Low | Medium | Medium | High |
| **Use case** | Default | Performance | Security | Legacy/physical | Cloud/wireless | Swarm |

---

## Summary Table

| Driver | Key Characteristic | Best For |
|--------|-------------------|----------|
| **bridge** | Private network with NAT | General-purpose single-host |
| **host** | No isolation, native performance | High-performance, many ports |
| **none** | No networking at all | Security, offline processing |
| **macvlan** | Own MAC, part of physical network | Legacy apps, direct L2 access |
| **ipvlan** | Shared MAC, separate IPs | Cloud environments, MAC limits |
| **overlay** | Multi-host via VXLAN | Docker Swarm clusters |

---

## Quick Revision Questions

1. **When should you use host networking?**
   - For performance-critical applications that need to avoid NAT overhead, or when an application needs to bind to many dynamic ports

2. **What is the trade-off of host networking?**
   - No network isolation — the container shares the host's network stack entirely. Port conflicts between containers and host services are possible

3. **What is macvlan and when would you use it?**
   - Macvlan gives containers their own MAC addresses, making them appear as physical devices on the network. Use it for legacy applications that need to be directly on the physical network

4. **What is the difference between macvlan and ipvlan?**
   - Macvlan assigns unique MAC addresses to each container; ipvlan shares the host's MAC address. IPvlan is better when there are MAC address limits (wireless, some cloud providers)

5. **What is an overlay network?**
   - An overlay network uses VXLAN tunneling to enable communication between containers on different Docker hosts, typically used with Docker Swarm

6. **When would you use `--network none`?**
   - For containers that don't need any network access, such as batch processing, offline computation, or security-critical operations like cryptographic processing

---

[← Previous: Bridge Networking](02-bridge-networking.md) | [Next: Network DNS and Service Discovery →](04-dns-and-service-discovery.md)
