# Chapter 2: IP Addressing and Subnetting

## Overview

IP addressing is the foundation of all network communication. Every device on a network needs a unique IP address to send and receive data. Subnetting allows you to divide large networks into smaller, manageable sub-networks. As a DevOps engineer, you'll use these skills daily — from designing VPC architectures to configuring Kubernetes pod networks.

---

## 2.1 IPv4 Address Structure

An IPv4 address is a **32-bit** number, written as four octets separated by dots (dotted decimal notation).

```
┌──────────────────────────────────────────────────────────────┐
│                  IPv4 ADDRESS STRUCTURE                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Decimal:    192  .  168  .    1  .   10                     │
│              ───     ───     ───    ───                       │
│  Binary:  11000000.10101000.00000001.00001010                │
│           ────────  ────────  ────────  ────────              │
│           Octet 1   Octet 2   Octet 3   Octet 4             │
│           (8 bits)  (8 bits)  (8 bits)  (8 bits)             │
│                                                              │
│  Total = 32 bits = 4 bytes                                   │
│  Range per octet: 0 - 255                                    │
│  Total addresses: 2^32 = ~4.3 billion                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Binary-to-Decimal Conversion

```
Bit Position:  128   64   32   16    8    4    2    1
               ───   ──   ──   ──   ──   ──   ──   ──
Example:         1    1    0    0    0    0    0    0  = 192
                 1    0    1    0    1    0    0    0  = 168
                 0    0    0    0    0    0    0    1  =   1
                 0    0    0    0    1    0    1    0  =  10
```

---

## 2.2 Network and Host Portions

Every IP address has two parts:

```
┌──────────────────────────────────────────────────────────────┐
│              NETWORK vs HOST PORTION                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  IP Address:     192.168.1.10                                │
│  Subnet Mask:    255.255.255.0                               │
│                                                              │
│  ┌──────────────────────────┬───────────────┐                │
│  │    NETWORK PORTION       │  HOST PORTION │                │
│  │    192.168.1             │  .10          │                │
│  │    (identifies network)  │  (identifies  │                │
│  │                          │   device)     │                │
│  └──────────────────────────┴───────────────┘                │
│                                                              │
│  Binary view:                                                │
│  IP:   11000000.10101000.00000001 . 00001010                 │
│  Mask: 11111111.11111111.11111111 . 00000000                 │
│        ─────────────────────────── ─────────                 │
│              Network bits            Host bits               │
│              (24 bits)               (8 bits)                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.3 Subnet Mask

A subnet mask tells you which bits belong to the network and which belong to hosts.

```
Common Subnet Masks:
┌──────────────────┬────────────────────────────────────┬──────────┐
│   Subnet Mask    │              Binary                │  CIDR    │
├──────────────────┼────────────────────────────────────┼──────────┤
│  255.0.0.0       │  11111111.00000000.00000000.00000000│  /8      │
│  255.255.0.0     │  11111111.11111111.00000000.00000000│  /16     │
│  255.255.255.0   │  11111111.11111111.11111111.00000000│  /24     │
│  255.255.255.128 │  11111111.11111111.11111111.10000000│  /25     │
│  255.255.255.192 │  11111111.11111111.11111111.11000000│  /26     │
│  255.255.255.224 │  11111111.11111111.11111111.11100000│  /27     │
│  255.255.255.240 │  11111111.11111111.11111111.11110000│  /28     │
│  255.255.255.248 │  11111111.11111111.11111111.11111000│  /29     │
│  255.255.255.252 │  11111111.11111111.11111111.11111100│  /30     │
└──────────────────┴────────────────────────────────────┴──────────┘
```

---

## 2.4 IP Address Classes (Legacy but Important)

```
┌─────────────────────────────────────────────────────────────────┐
│                    IPv4 ADDRESS CLASSES                          │
├───────┬───────────────────┬───────────────┬─────────┬───────────┤
│ Class │  Range            │ Default Mask  │  CIDR   │  Use      │
├───────┼───────────────────┼───────────────┼─────────┼───────────┤
│   A   │ 1.0.0.0 -         │ 255.0.0.0     │  /8     │ Large     │
│       │ 126.255.255.255   │               │         │ networks  │
├───────┼───────────────────┼───────────────┼─────────┼───────────┤
│   B   │ 128.0.0.0 -       │ 255.255.0.0   │  /16    │ Medium    │
│       │ 191.255.255.255   │               │         │ networks  │
├───────┼───────────────────┼───────────────┼─────────┼───────────┤
│   C   │ 192.0.0.0 -       │ 255.255.255.0 │  /24    │ Small     │
│       │ 223.255.255.255   │               │         │ networks  │
├───────┼───────────────────┼───────────────┼─────────┼───────────┤
│   D   │ 224.0.0.0 -       │ N/A           │  N/A    │ Multicast │
│       │ 239.255.255.255   │               │         │           │
├───────┼───────────────────┼───────────────┼─────────┼───────────┤
│   E   │ 240.0.0.0 -       │ N/A           │  N/A    │ Reserved  │
│       │ 255.255.255.255   │               │         │           │
└───────┴───────────────────┴───────────────┴─────────┴───────────┘

Note: 127.0.0.0/8 is reserved for loopback (localhost)
```

---

## 2.5 Subnetting — Step by Step

Subnetting divides a larger network into smaller sub-networks.

### Why Subnet?
- **Efficient IP usage** — don't waste addresses
- **Security** — isolate network segments (e.g., web tier vs DB tier)
- **Performance** — reduce broadcast domain size
- **Cloud design** — VPC subnets for public/private tiers

### The Subnetting Formula

```
Number of Subnets  = 2^(subnet bits borrowed)
Number of Hosts    = 2^(remaining host bits) - 2
                     (-2 because: 1 for network address, 1 for broadcast)
```

### Example: Subnet 192.168.1.0/24 into 4 subnets

```
Step 1: How many bits to borrow?
        We need 4 subnets → 2^n ≥ 4 → n = 2 bits

Step 2: New subnet mask
        Original: /24 (255.255.255.0)
        Borrowed: 2 bits
        New mask: /26 (255.255.255.192)

Step 3: Calculate subnet size
        Host bits remaining = 32 - 26 = 6
        Hosts per subnet = 2^6 - 2 = 62

Step 4: Determine subnets
┌──────────┬──────────────────┬──────────────────┬──────────────────┐
│ Subnet # │ Network Address  │  Usable Range    │ Broadcast        │
├──────────┼──────────────────┼──────────────────┼──────────────────┤
│    1     │ 192.168.1.0/26   │ .1   - .62       │ 192.168.1.63     │
│    2     │ 192.168.1.64/26  │ .65  - .126      │ 192.168.1.127    │
│    3     │ 192.168.1.128/26 │ .129 - .190      │ 192.168.1.191    │
│    4     │ 192.168.1.192/26 │ .193 - .254      │ 192.168.1.255    │
└──────────┴──────────────────┴──────────────────┴──────────────────┘
```

```
         192.168.1.0/24 (Original Network)
         ┌────────────────────────────────┐
         │                                │
    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
    │ Subnet 1│    │ Subnet 2│    │ Subnet 3│    │ Subnet 4│
    │  .0/26  │    │ .64/26  │    │ .128/26 │    │ .192/26 │
    │ 62 hosts│    │ 62 hosts│    │ 62 hosts│    │ 62 hosts│
    └─────────┘    └─────────┘    └─────────┘    └─────────┘
```

---

## 2.6 Quick Subnetting Reference

```
┌──────┬──────────────────┬──────────────┬────────────────┐
│ CIDR │ Subnet Mask      │ Total IPs    │ Usable Hosts   │
├──────┼──────────────────┼──────────────┼────────────────┤
│ /32  │ 255.255.255.255  │     1        │     1          │
│ /31  │ 255.255.255.254  │     2        │     2*         │
│ /30  │ 255.255.255.252  │     4        │     2          │
│ /29  │ 255.255.255.248  │     8        │     6          │
│ /28  │ 255.255.255.240  │    16        │    14          │
│ /27  │ 255.255.255.224  │    32        │    30          │
│ /26  │ 255.255.255.192  │    64        │    62          │
│ /25  │ 255.255.255.128  │   128        │   126          │
│ /24  │ 255.255.255.0    │   256        │   254          │
│ /23  │ 255.255.254.0    │   512        │   510          │
│ /22  │ 255.255.252.0    │  1024        │  1022          │
│ /21  │ 255.255.248.0    │  2048        │  2046          │
│ /20  │ 255.255.240.0    │  4096        │  4094          │
│ /16  │ 255.255.0.0      │ 65536        │ 65534          │
│ /8   │ 255.0.0.0        │ 16777216     │ 16777214       │
└──────┴──────────────────┴──────────────┴────────────────┘

* /31 is a special case used for point-to-point links (RFC 3021)
```

---

## 2.7 IPv6 Overview

IPv6 uses **128-bit** addresses to solve IPv4 exhaustion.

```
┌──────────────────────────────────────────────────────────────┐
│                    IPv4  vs  IPv6                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  IPv4:  192.168.1.10                                         │
│         32 bits, ~4.3 billion addresses                      │
│                                                              │
│  IPv6:  2001:0db8:85a3:0000:0000:8a2e:0370:7334              │
│         128 bits, ~340 undecillion addresses                  │
│                                                              │
│  IPv6 shortcuts:                                             │
│  ├── Remove leading zeros: 2001:db8:85a3:0:0:8a2e:370:7334  │
│  └── Replace consecutive 0 groups with ::                    │
│       2001:db8:85a3::8a2e:370:7334                           │
│                                                              │
│  Loopback: ::1  (equivalent to 127.0.0.1)                    │
│  Link-local: fe80::/10                                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.8 Real-World DevOps Scenario

**[~] Scenario: Designing a VPC subnet plan**

You're tasked with designing a VPC for a 3-tier application:
- Public subnet (web servers)
- Private subnet (app servers) 
- Private subnet (databases)
- Management subnet

```
VPC CIDR: 10.0.0.0/16 (65,536 IPs)

┌─────────────────────────────────────────────────────────────┐
│                     VPC: 10.0.0.0/16                        │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Public Sub   │  │  Private Sub │  │  Private Sub │      │
│  │  Web Tier     │  │  App Tier    │  │  DB Tier     │      │
│  │ 10.0.1.0/24  │  │ 10.0.2.0/24 │  │ 10.0.3.0/24 │      │
│  │ 254 hosts    │  │ 254 hosts    │  │ 254 hosts    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
│  ┌──────────────┐                                           │
│  │  Mgmt Sub    │     Remaining: 10.0.4.0/24 - 10.0.255.0  │
│  │ 10.0.4.0/24  │     (252 more /24 subnets available)      │
│  │ 254 hosts    │                                           │
│  └──────────────┘                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Best Practice**: Always leave room for growth. Use /16 for VPC, /24 for subnets.

---

## 2.9 Hands-On Commands

```bash
# View IP address configuration
ip addr show                      # Linux
ipconfig                          # Windows

# Calculate subnet info using ipcalc
ipcalc 192.168.1.0/26            # Linux tool

# Check which subnet an IP belongs to
# Is 192.168.1.100 in 192.168.1.0/26?
# 192.168.1.0/26 range: .0 - .63 → No!
# 192.168.1.64/26 range: .64 - .127 → Yes!

# Python quick calculator
python3 -c "
import ipaddress
net = ipaddress.ip_network('192.168.1.0/24')
subnets = list(net.subnets(new_prefix=26))
for s in subnets:
    print(f'Network: {s}, Hosts: {list(s.hosts())[0]} - {list(s.hosts())[-1]}, Broadcast: {s.broadcast_address}')
"
```

---

## 2.10 Troubleshooting Tips

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Two devices can't communicate | Different subnets, no route | Check subnet masks, add a route |
| IP conflict | Duplicate IPs | Use DHCP or maintain IP inventory |
| "No route to host" | Missing route entry | Check `ip route` or route table in cloud |
| Can't reach internet | No default gateway | Verify gateway config, NAT setup |
| AWS: Instance unreachable | Wrong subnet CIDR or SG | Verify CIDR, Security Group, NACL |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| IPv4 | 32-bit address, 4 octets, ~4.3 billion total |
| Subnet Mask | Separates network and host portions |
| Subnetting | Divides networks; Hosts = 2^(host bits) - 2 |
| CIDR | Shorthand for subnet mask (e.g., /24 = 255.255.255.0) |
| Network Address | First IP in a subnet (all host bits = 0) |
| Broadcast Address | Last IP in a subnet (all host bits = 1) |
| IPv6 | 128-bit addresses, virtually unlimited |
| DevOps Use | VPC design, subnet planning, Kubernetes networking |

---

## Quick Revision Questions

1. **An IP address is 192.168.10.50/28. What is the network address, broadcast address, and usable host range?**
2. **You need to create 8 subnets from 10.0.0.0/24. What will the new CIDR prefix be and how many hosts per subnet?**
3. **What's the difference between a network address and a broadcast address?**
4. **Why do we subtract 2 from the total number of IPs to get usable hosts? (What's the exception?)**
5. **How is a /16 VPC typically subdivided for a multi-tier cloud application?**
6. **Convert the decimal IP 172.16.254.1 to binary.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← OSI & TCP/IP Models](01-osi-tcp-ip-models.md) | [README](../README.md) | [CIDR Notation →](03-cidr-notation.md) |
