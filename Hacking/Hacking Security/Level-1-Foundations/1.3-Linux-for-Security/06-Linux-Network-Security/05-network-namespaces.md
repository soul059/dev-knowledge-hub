# Network Namespaces

## Unit 6 - Topic 5 | Linux Network Security

---

## Overview

**Network namespaces** provide complete isolation of the Linux network stack — each namespace has its own **interfaces**, **routing table**, **iptables rules**, and **sockets**. They are the foundation of **container networking** and are used for **security testing**, **network simulation**, and **process isolation**.

---

## 1. How Network Namespaces Work

```
┌──────────────────────────────────────────────────────────────┐
│              NETWORK NAMESPACE ISOLATION                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DEFAULT NAMESPACE (Host):                                   │
│  ├── eth0: 10.0.0.5                                         │
│  ├── lo: 127.0.0.1                                          │
│  ├── Routing table                                           │
│  └── iptables rules                                          │
│                                                              │
│  NAMESPACE "red":                                            │
│  ├── veth-red: 192.168.1.1     ← Completely separate!       │
│  ├── lo: 127.0.0.1                                          │
│  ├── Own routing table                                       │
│  └── Own iptables rules                                      │
│                                                              │
│  NAMESPACE "blue":                                           │
│  ├── veth-blue: 192.168.2.1    ← Also separate!             │
│  ├── lo: 127.0.0.1                                          │
│  ├── Own routing table                                       │
│  └── Own iptables rules                                      │
│                                                              │
│  Processes in "red" CANNOT see "blue" network               │
│  Processes in "red" CANNOT see host network                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Network Namespace Commands

```bash
# === CREATE AND MANAGE ===
ip netns add red                     # Create namespace "red"
ip netns add blue                    # Create namespace "blue"
ip netns list                        # List all namespaces

# === RUN COMMANDS IN NAMESPACE ===
ip netns exec red ip addr            # View interfaces in "red"
ip netns exec red bash               # Shell inside namespace
ip netns exec red ping 192.168.1.2   # Ping from namespace
ip netns exec red ss -tlnp           # View listeners in namespace

# === DELETE ===
ip netns delete red
```

---

## 3. Connecting Namespaces with veth Pairs

```bash
# Virtual Ethernet (veth) pairs act like a cable between namespaces

# Create veth pair
ip link add veth-red type veth peer name veth-blue

# Assign to namespaces
ip link set veth-red netns red
ip link set veth-blue netns blue

# Configure IP addresses
ip netns exec red ip addr add 192.168.1.1/24 dev veth-red
ip netns exec blue ip addr add 192.168.1.2/24 dev veth-blue

# Bring interfaces up
ip netns exec red ip link set veth-red up
ip netns exec blue ip link set veth-blue up
ip netns exec red ip link set lo up
ip netns exec blue ip link set lo up

# Test connectivity
ip netns exec red ping 192.168.1.2   # Ping blue from red ✅
```

```
RESULT:
┌─────────────────┐ veth pair ┌─────────────────┐
│ Namespace "red"  │◄────────►│ Namespace "blue" │
│ 192.168.1.1      │          │ 192.168.1.2      │
└─────────────────┘          └─────────────────┘
```

---

## 4. Security Use Cases

| Use Case | Description |
|----------|-------------|
| **Container networking** | Each Docker container gets its own namespace |
| **Testing** | Simulate complex networks on a single machine |
| **Isolation** | Run untrusted code with no network access |
| **Firewall testing** | Test iptables rules without affecting host |
| **Honeypots** | Isolated network environments for deception |
| **Penetration testing** | Simulate target networks |

```bash
# SECURITY: Run process with NO network
unshare --net bash
# This shell has no network access at all!
# Not even localhost

# TESTING: Run iptables in namespace without affecting host
ip netns exec testns iptables -A INPUT -p tcp --dport 80 -j DROP
# Only affects traffic within testns
```

---

## 5. Docker Networking and Namespaces

```bash
# Docker creates a network namespace per container
# View container's namespace
docker inspect <container> | grep NetworkMode
docker inspect <container> | grep SandboxKey
# SandboxKey = /var/run/docker/netns/<id>

# Enter container's namespace
nsenter -t $(docker inspect -f '{{.State.Pid}}' <container>) -n bash

# Docker networking is built on:
# 1. Network namespaces (isolation)
# 2. veth pairs (container ↔ bridge)
# 3. Linux bridge (docker0)
# 4. iptables NAT (container → internet)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Network namespace** | Isolated network stack (interfaces, routes, iptables) |
| **veth pair** | Virtual cable connecting two namespaces |
| **ip netns** | Command to create and manage namespaces |
| **Container networking** | Built on network namespaces |
| **Security** | Complete network isolation for processes |
| **Testing** | Simulate networks without hardware |

---

## Quick Revision Questions

1. **What does a network namespace isolate?**
2. **How do you connect two namespaces?**
3. **How do you run a process with no network access?**
4. **How do containers use network namespaces?**
5. **What command creates a network namespace?**

---

[← Previous: Network Services Security](04-network-services-security.md)

---

*Unit 6 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Linux Security Tools →](../07-Linux-Security-Tools/01-network-scanning-nmap.md)*
