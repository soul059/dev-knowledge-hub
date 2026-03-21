# IP Addressing and Subnetting

## Unit 1 - Topic 3 | Network Fundamentals Review

---

## Overview

**IP addressing** and **subnetting** are core networking skills essential for security professionals. Understanding how networks are divided allows you to identify network boundaries, configure firewall rules, perform reconnaissance, and design segmented architectures.

---

## 1. IPv4 Address Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                  IPv4 ADDRESS STRUCTURE                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  IP Address: 192.168.1.100                                      │
│                                                                  │
│  Decimal:     192    .  168    .    1    .  100                  │
│  Binary:   11000000.10101000.00000001.01100100                  │
│            ├────────────────────┤├────────────┤                  │
│                Network Part       Host Part                      │
│                                                                  │
│  32 bits total = 4 octets (8 bits each)                         │
│  Range: 0.0.0.0 to 255.255.255.255                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. IP Address Classes

| Class | Range | Default Mask | Networks | Hosts/Network | Use |
|:-----:|-------|:------------:|:--------:|:-------------:|-----|
| **A** | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | 126 | ~16.7M | Large organizations |
| **B** | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | 16,384 | ~65K | Medium organizations |
| **C** | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | ~2.1M | 254 | Small networks |
| **D** | 224.0.0.0 – 239.255.255.255 | — | — | — | Multicast |
| **E** | 240.0.0.0 – 255.255.255.255 | — | — | — | Reserved |

---

## 3. Private (RFC 1918) vs Public IPs

```
┌──────────────────────────────────────────────────────────────┐
│              PRIVATE vs PUBLIC IP ADDRESSES                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PRIVATE (Non-routable on internet):                         │
│  ┌────────────────────────────────────────────┐              │
│  │ 10.0.0.0    – 10.255.255.255    (/8)       │  Class A    │
│  │ 172.16.0.0  – 172.31.255.255   (/12)      │  Class B    │
│  │ 192.168.0.0 – 192.168.255.255  (/16)      │  Class C    │
│  └────────────────────────────────────────────┘              │
│                                                              │
│  SPECIAL ADDRESSES:                                          │
│  127.0.0.0/8     → Loopback (localhost)                      │
│  169.254.0.0/16  → Link-local (APIPA — no DHCP)             │
│  0.0.0.0         → Default route / unspecified               │
│  255.255.255.255 → Broadcast                                 │
│                                                              │
│  SECURITY NOTE:                                              │
│  Private IPs should NEVER appear on the public internet.     │
│  If they do → potential routing misconfiguration / leak.     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Subnetting

### Subnet Mask Basics

```
IP Address:    192.168.1.100    = 11000000.10101000.00000001.01100100
Subnet Mask:   255.255.255.0    = 11111111.11111111.11111111.00000000
                                  ├───── Network Bits ─────┤├─ Host ─┤

Network:       192.168.1.0      (all host bits = 0)
Broadcast:     192.168.1.255    (all host bits = 1)
Usable Hosts:  192.168.1.1 – 192.168.1.254  (254 hosts)
```

### CIDR Notation Quick Reference

| CIDR | Subnet Mask | Usable Hosts | Common Use |
|:----:|:-----------:|:------------:|-----------|
| /32 | 255.255.255.255 | 1 | Single host (loopback, host route) |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /29 | 255.255.255.248 | 6 | Small server segment |
| /28 | 255.255.255.240 | 14 | Small office |
| /27 | 255.255.255.224 | 30 | Department |
| /26 | 255.255.255.192 | 62 | Floor/building |
| /25 | 255.255.255.128 | 126 | Large department |
| /24 | 255.255.255.0 | 254 | Standard network |
| /16 | 255.255.0.0 | 65,534 | Large campus |
| /8 | 255.0.0.0 | 16,777,214 | Enterprise (private) |

### Subnetting Example

```
TASK: Divide 192.168.1.0/24 into 4 subnets

Need 4 subnets → borrow 2 bits (2² = 4)
New mask: /26 (255.255.255.192)

┌──────────┬───────────────────────┬───────────────────┬──────────┐
│ Subnet # │   Network Address     │   Usable Range    │Broadcast │
├──────────┼───────────────────────┼───────────────────┼──────────┤
│    1     │  192.168.1.0/26       │ .1 – .62          │ .63      │
│    2     │  192.168.1.64/26      │ .65 – .126        │ .127     │
│    3     │  192.168.1.128/26     │ .129 – .190       │ .191     │
│    4     │  192.168.1.192/26     │ .193 – .254       │ .255     │
└──────────┴───────────────────────┴───────────────────┴──────────┘

Each subnet: 64 addresses (62 usable hosts)
```

---

## 5. Subnetting for Security

| Security Use | How Subnetting Helps |
|-------------|---------------------|
| **Segmentation** | Isolate users, servers, IoT into separate subnets |
| **Firewall Rules** | Rules based on subnet addresses (source/destination) |
| **ACLs** | Permit/deny traffic per subnet |
| **Recon Defense** | Smaller subnets = smaller attack surface |
| **Pen Testing** | Identify live hosts per subnet during scanning |
| **Compliance** | PCI DSS requires cardholder data in isolated subnet |

---

## 6. Useful Commands

```bash
# Windows — View IP configuration
ipconfig /all

# Linux — View IP configuration
ip addr show
ifconfig

# Calculate subnet info
ipcalc 192.168.1.0/26     # Linux tool

# Scan a specific subnet (Nmap)
nmap -sn 192.168.1.0/24   # Ping sweep
nmap -sn 10.0.1.0/26      # Scan specific subnet
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IPv4** | 32-bit address, 4 octets, ~4.3 billion addresses |
| **Private IPs** | 10.x, 172.16-31.x, 192.168.x — not internet-routable |
| **Subnetting** | Dividing a network into smaller networks |
| **CIDR** | /24 = 254 hosts, /26 = 62 hosts, /30 = 2 hosts |
| **Security Value** | Segmentation, access control, compliance |

---

## Quick Revision Questions

1. **What are the three RFC 1918 private IP ranges?**
2. **How many usable hosts are in a /26 subnet?**
3. **What is the difference between network address and broadcast address?**
4. **Why is subnetting important for network security?**
5. **Subnet 192.168.10.0/24 into 8 subnets — what is the new mask and host count?**

---

[← Previous: Network Devices](02-network-devices.md) | [Next: VLANs & Segmentation →](04-vlans-and-segmentation.md)

---

*Unit 1 - Topic 3 of 5 | [Back to README](../README.md)*
