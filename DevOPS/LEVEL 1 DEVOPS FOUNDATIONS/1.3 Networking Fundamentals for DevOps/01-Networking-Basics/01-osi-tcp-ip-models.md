# Chapter 1: OSI and TCP/IP Models

## Overview

The OSI (Open Systems Interconnection) and TCP/IP models are foundational frameworks that describe how data travels across a network. Every DevOps engineer must understand these layers — they are the language of networking and are critical for debugging connectivity issues, designing architectures, and configuring cloud infrastructure.

---

## 1.1 The OSI Model (7 Layers)

The OSI model is a **conceptual framework** developed by the ISO. It divides network communication into 7 distinct layers, each with a specific responsibility.

```
┌─────────────────────────────────────────────────────────────┐
│                    OSI MODEL (7 Layers)                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 7 ─ Application    │ HTTP, FTP, DNS, SSH, SMTP       │
│  ──────────────────────────────────────────────────────────  │
│  Layer 6 ─ Presentation   │ SSL/TLS, JPEG, ASCII, Encryption│
│  ──────────────────────────────────────────────────────────  │
│  Layer 5 ─ Session        │ NetBIOS, RPC, session mgmt      │
│  ──────────────────────────────────────────────────────────  │
│  Layer 4 ─ Transport      │ TCP, UDP (ports, segments)      │
│  ──────────────────────────────────────────────────────────  │
│  Layer 3 ─ Network        │ IP, ICMP, routing (packets)     │
│  ──────────────────────────────────────────────────────────  │
│  Layer 2 ─ Data Link      │ Ethernet, MAC addresses (frames)│
│  ──────────────────────────────────────────────────────────  │
│  Layer 1 ─ Physical       │ Cables, hubs, signals (bits)    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
    ▲ Data goes UP on receiving end
    │
    ▼ Data goes DOWN on sending end
```

### Mnemonic to Remember

**"All People Seem To Need Data Processing"** (Top to Bottom: Application → Physical)

**"Please Do Not Throw Sausage Pizza Away"** (Bottom to Top: Physical → Application)

---

### Layer-by-Layer Breakdown

#### Layer 1 — Physical Layer
- **What it does**: Transmits raw bits (0s and 1s) over a physical medium
- **Examples**: Ethernet cables, fiber optics, Wi-Fi radio signals, hubs, repeaters
- **DevOps relevance**: Rarely interact directly, but data center cabling, NIC configuration
- **Data Unit**: Bits

#### Layer 2 — Data Link Layer
- **What it does**: Handles node-to-node data transfer using MAC addresses; error detection
- **Examples**: Ethernet (802.3), Wi-Fi (802.11), switches, bridges, ARP
- **DevOps relevance**: VLANs, MAC filtering, switch configuration
- **Data Unit**: Frames

#### Layer 3 — Network Layer
- **What it does**: Routes packets between different networks using IP addresses
- **Examples**: IP (IPv4/IPv6), ICMP, routers, routing protocols (OSPF, BGP)
- **DevOps relevance**: IP addressing, subnetting, route tables, VPC routing, `traceroute`
- **Data Unit**: Packets

#### Layer 4 — Transport Layer
- **What it does**: Ensures reliable (TCP) or fast (UDP) end-to-end data delivery
- **Examples**: TCP, UDP, port numbers
- **DevOps relevance**: Port mapping, security group rules, load balancer health checks
- **Data Unit**: Segments (TCP) / Datagrams (UDP)

#### Layer 5 — Session Layer
- **What it does**: Manages sessions/connections between applications
- **Examples**: NetBIOS, RPC, session tokens
- **DevOps relevance**: Session persistence in load balancers, API session management

#### Layer 6 — Presentation Layer
- **What it does**: Data translation, encryption/decryption, compression
- **Examples**: SSL/TLS, JPEG, JSON/XML serialization
- **DevOps relevance**: TLS termination at load balancers, certificate management

#### Layer 7 — Application Layer
- **What it does**: Provides network services directly to end-user applications
- **Examples**: HTTP, HTTPS, FTP, DNS, SSH, SMTP
- **DevOps relevance**: Application load balancers, reverse proxies, API gateways, WAFs

---

## 1.2 The TCP/IP Model (4 Layers)

The TCP/IP model is the **practical model** that the internet actually uses. It simplifies the OSI model into 4 layers.

```
┌────────────────────────────────────────────────────────────┐
│              TCP/IP MODEL vs OSI MODEL                      │
├────────────────────────────────────────────────────────────┤
│                                                            │
│   TCP/IP Model          │    OSI Model                     │
│   ─────────────────────────────────────────────            │
│                         │                                  │
│   Application           │    Application (7)               │
│   (Layer 4)             │    Presentation (6)              │
│                         │    Session (5)                   │
│   ─────────────────────────────────────────────            │
│   Transport             │    Transport (4)                 │
│   (Layer 3)             │                                  │
│   ─────────────────────────────────────────────            │
│   Internet              │    Network (3)                   │
│   (Layer 2)             │                                  │
│   ─────────────────────────────────────────────            │
│   Network Access        │    Data Link (2)                 │
│   (Layer 1)             │    Physical (1)                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### TCP/IP Layers Explained

| TCP/IP Layer | OSI Equivalent | Key Protocols | Function |
|---|---|---|---|
| Application | Layers 5-7 | HTTP, DNS, SSH, FTP, SMTP | User-facing services |
| Transport | Layer 4 | TCP, UDP | End-to-end communication |
| Internet | Layer 3 | IP, ICMP, ARP | Addressing & routing |
| Network Access | Layers 1-2 | Ethernet, Wi-Fi | Physical transmission |

---

## 1.3 Data Encapsulation

As data moves DOWN the layers (sending), each layer adds its own header — this process is called **encapsulation**. The reverse (receiving) is called **de-encapsulation**.

```
┌─────────────────────────────────────────────────────────────┐
│                  DATA ENCAPSULATION                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Application Layer                                          │
│  ┌──────────────────────────┐                               │
│  │         DATA             │                               │
│  └──────────────────────────┘                               │
│              │                                              │
│              ▼                                              │
│  Transport Layer                                            │
│  ┌──────┬───────────────────┐                               │
│  │TCP/  │       DATA        │    = SEGMENT                  │
│  │UDP   │                   │                               │
│  │HDR   │                   │                               │
│  └──────┴───────────────────┘                               │
│              │                                              │
│              ▼                                              │
│  Network Layer                                              │
│  ┌────┬──────┬──────────────┐                               │
│  │ IP │TCP/  │     DATA     │    = PACKET                   │
│  │HDR │UDP   │              │                               │
│  └────┴──────┴──────────────┘                               │
│              │                                              │
│              ▼                                              │
│  Data Link Layer                                            │
│  ┌─────┬────┬──────┬────────┬───────┐                       │
│  │FRAME│ IP │TCP/  │  DATA  │  FCS  │  = FRAME              │
│  │ HDR │HDR │UDP   │        │(check)│                       │
│  └─────┴────┴──────┴────────┴───────┘                       │
│              │                                              │
│              ▼                                              │
│  Physical Layer                                             │
│  01101001 01110100 01100001 ...      = BITS                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.4 Why DevOps Engineers Need to Know This

```
┌──────────────────────────────────────────────────────────┐
│          REAL-WORLD DEVOPS MAPPING                        │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Layer 7 (Application)                                   │
│  ├── Configure Nginx/Apache reverse proxies              │
│  ├── Set up Application Load Balancers (ALB)             │
│  ├── Deploy API Gateways                                 │
│  └── Configure WAF rules                                 │
│                                                          │
│  Layer 4 (Transport)                                     │
│  ├── Define Security Group rules (ports)                 │
│  ├── Configure Network Load Balancers (NLB)              │
│  ├── Set up port forwarding in Docker                    │
│  └── Kubernetes Service port mapping                     │
│                                                          │
│  Layer 3 (Network)                                       │
│  ├── Design VPC CIDR blocks                              │
│  ├── Configure route tables                              │
│  ├── Set up VPN connections                              │
│  └── Manage subnet allocation                            │
│                                                          │
│  Layer 2 (Data Link)                                     │
│  ├── VLAN configuration                                  │
│  ├── Docker bridge networks                              │
│  └── Container network interfaces                        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 1.5 Hands-On: Seeing Layers in Action

### View your network configuration (Layer 2-3)
```bash
# Linux
ip addr show
ip route show

# Windows
ipconfig /all
route print

# macOS
ifconfig
netstat -rn
```

### Test Layer 3 connectivity
```bash
# Ping tests ICMP (Layer 3)
ping 8.8.8.8

# Traceroute shows Layer 3 hops
traceroute google.com        # Linux/macOS
tracert google.com           # Windows
```

### Test Layer 4 connectivity
```bash
# Check if a TCP port is open
telnet example.com 443
nc -zv example.com 443

# View active connections with port info
ss -tuln                     # Linux
netstat -an                  # Windows/macOS
```

### Test Layer 7
```bash
# HTTP request (Application layer)
curl -v https://example.com

# DNS query (Application layer)
dig example.com
nslookup example.com
```

---

## 1.6 Real-World Scenario

**[~] Scenario: Debugging a failing health check in AWS**

Your Application Load Balancer (ALB) reports targets as "unhealthy." Here's how OSI layers help you troubleshoot:

```
Step 1: Layer 3 — Can the ALB reach the instance?
        → Check route tables, subnet associations, NACL rules

Step 2: Layer 4 — Is the correct port open?
        → Check Security Group: inbound on port 8080
        → Verify the app is listening: ss -tuln | grep 8080

Step 3: Layer 7 — Is the application responding correctly?
        → curl http://instance-ip:8080/health
        → Check if response is HTTP 200
        → Review application logs
```

---

## 1.7 Troubleshooting Tips

| Problem | OSI Layer | What to Check |
|---------|-----------|---------------|
| Cannot ping a server | Layer 3 | IP config, route tables, ICMP allowed? |
| Connection refused on port | Layer 4 | Security groups, firewall, app listening? |
| SSL/TLS handshake failure | Layer 6 | Certificate validity, cipher suites |
| HTTP 502 Bad Gateway | Layer 7 | Backend app running? Proxy config correct? |
| Slow network performance | Layer 1-2 | Cable quality, NIC speed, duplex settings |
| DNS not resolving | Layer 7 | DNS server config, record exists? |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| OSI Model | 7 layers: Physical → Application; conceptual framework |
| TCP/IP Model | 4 layers: Network Access → Application; practical model used by the internet |
| Encapsulation | Each layer wraps data with its header (Data → Segment → Packet → Frame → Bits) |
| Layer 3 | IP addressing, routing — core of cloud networking |
| Layer 4 | TCP/UDP, port numbers — security group rules operate here |
| Layer 7 | Application protocols — ALBs, reverse proxies, API gateways |
| DevOps Focus | Layers 3, 4, and 7 are the most relevant for daily DevOps work |

---

## Quick Revision Questions

1. **How many layers does the OSI model have, and what are the names from bottom to top?**
2. **Which OSI layer do Security Groups primarily operate at? Why?**
3. **What is the difference between the OSI model and the TCP/IP model?**
4. **Explain the encapsulation process — what happens to data as it moves from Layer 7 to Layer 1?**
5. **A web application returns HTTP 502. Which OSI layer should you start troubleshooting at?**
6. **Name the data units at each layer: Application, Transport, Network, and Data Link.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| — | [README](../README.md) | [IP Addressing & Subnetting →](02-ip-addressing-subnetting.md) |
