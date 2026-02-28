# Chapter 1: Networking Overview

> **Unit 6: Docker Networking** | [Course Home](../README.md)

---

## Overview

Docker networking enables containers to communicate with each other, the host, and the outside world. Docker implements its networking stack using Linux network namespaces, virtual ethernet pairs, bridges, and iptables rules.

---

## 1.1 Docker Networking Architecture

```
  +----------------------------------------------------------------+
  |  Host Machine                                                    |
  |                                                                  |
  |  +------------------------------------------+                   |
  |  |  Docker Network Subsystem                 |                   |
  |  |                                           |                   |
  |  |  +-----------+  +----------+              |                   |
  |  |  | libnetwork|  | Drivers  |              |                   |
  |  |  | (CNM)     |  | bridge   |              |                   |
  |  |  |           |  | host     |              |                   |
  |  |  |           |  | overlay  |              |                   |
  |  |  |           |  | macvlan  |              |                   |
  |  |  |           |  | none     |              |                   |
  |  |  +-----------+  +----------+              |                   |
  |  +------------------------------------------+                   |
  |                                                                  |
  |  +----------+  +----------+  +----------+      eth0             |
  |  | Container|  | Container|  | Container|       |               |
  |  | veth0    |  | veth0    |  | veth0    |       |               |
  |  +----+-----+  +----+-----+  +----+-----+      |               |
  |       |             |             |              |               |
  |  +----+-------------+-------------+----+         |               |
  |  |     docker0 (bridge)                |         |               |
  |  |     172.17.0.1                      |---------+               |
  |  +-------------------------------------+   NAT / iptables       |
  +----------------------------------------------------------------+
```

---

## 1.2 Container Network Model (CNM)

```
  CNM Components:
  
  +--------------------+
  |  Network Sandbox   |    Container's network namespace
  |  (Network NS)      |    Contains interfaces, routes, DNS
  +--------+-----------+
           |
  +--------v-----------+
  |  Endpoint          |    Virtual ethernet pair (veth)
  |  (veth pair)       |    Connects sandbox to network
  +--------+-----------+
           |
  +--------v-----------+
  |  Network           |    Group of endpoints that
  |  (bridge/overlay)  |    can communicate
  +--------------------+
  
  One container can have multiple endpoints,
  connecting to multiple networks simultaneously.
```

---

## 1.3 Network Drivers

| Driver | Description | Use Case |
|--------|-------------|----------|
| **bridge** | Default. Private internal network | Single-host container communication |
| **host** | Shares host network stack | Performance-critical applications |
| **overlay** | Multi-host networking | Docker Swarm / cross-host |
| **macvlan** | Assigns MAC address to container | Legacy apps needing L2 access |
| **ipvlan** | Similar to macvlan, shares MAC | When MAC address limit is an issue |
| **none** | No networking | Maximum isolation |

```
  Bridge:            Host:              Overlay:
  +------+          +------+           +------+   +------+
  | C1   |          | C1   |           | C1   |   | C2   |
  +--+---+          +------+           +--+---+   +--+---+
     |              | uses |              |           |
  +--+---+          | host |           +--+-----------+--+
  | bridge|         | stack|           |   VXLAN tunnel  |
  +--+---+          +------+           +-----------------+
     |                                  Host A     Host B
  +--+---+
  | NAT  |
  +------+
```

---

## 1.4 Listing and Inspecting Networks

```bash
# List all networks
docker network ls

# Output:
# NETWORK ID     NAME      DRIVER    SCOPE
# abc123         bridge    bridge    local
# def456         host      host      local
# ghi789         none      null      local

# Inspect network
docker network inspect bridge

# Key information in inspect:
# - Subnet and gateway
# - Connected containers with IPs
# - Driver options
# - IPAM configuration

# Filter output
docker network inspect -f '{{.IPAM.Config}}' bridge
docker network inspect -f '{{json .Containers}}' bridge | jq
```

---

## 1.5 Network Scope

```
  +------------------+-------------------------------+
  | Scope            | Description                   |
  +------------------+-------------------------------+
  | local            | Single Docker host            |
  |                  | (bridge, host, macvlan, none)  |
  |                  |                               |
  | swarm            | Multi-host Swarm cluster       |
  |                  | (overlay)                      |
  +------------------+-------------------------------+
  
  Local Scope:              Swarm Scope:
  
  +--------+               +--------+   +--------+
  | Host 1 |               | Host 1 |   | Host 2 |
  | +----+ |               | +----+ |   | +----+ |
  | | C1 | |               | | C1 |-+--->-| C2 | |
  | +----+ |               | +----+ |   | +----+ |
  +--------+               +--------+   +--------+
  C1 isolated               C1 and C2 can
  to this host              communicate across hosts
```

---

## Summary Table

| Component | Description | Example |
|-----------|-------------|---------|
| **CNM** | Docker's networking model | Sandbox + Endpoint + Network |
| **Sandbox** | Container's network namespace | Isolates interfaces, routes |
| **Endpoint** | veth pair connecting container to network | Virtual NIC |
| **Network** | Communication channel for containers | bridge, overlay |
| **Driver** | Implements the network type | bridge, host, overlay, macvlan |
| **Scope** | Reach of the network | local, swarm |

---

## Quick Revision Questions

1. **What is the Container Network Model (CNM)?**
   - CNM defines three components: Sandbox (network namespace for a container), Endpoint (virtual interface connecting sandbox to network), and Network (group of endpoints that can communicate)

2. **What is the default network driver in Docker?**
   - The `bridge` driver. All containers join the default bridge network unless otherwise specified

3. **What are the main network drivers available in Docker?**
   - bridge (single-host isolation), host (shared host stack), overlay (multi-host), macvlan (direct L2), ipvlan (shared MAC), and none (no networking)

4. **What is the difference between local and swarm scope?**
   - Local scope networks exist only on a single Docker host. Swarm scope networks (overlay) span across multiple hosts in a Docker Swarm cluster

5. **How does Docker implement container networking on Linux?**
   - Using network namespaces for isolation, veth pairs to connect containers to bridges, Linux bridge (docker0) for switching, and iptables for NAT and port mapping

---

[← Previous Unit: Container Cleanup](../05-Docker-Containers/07-container-cleanup.md) | [Next: Bridge Networking →](02-bridge-networking.md)
