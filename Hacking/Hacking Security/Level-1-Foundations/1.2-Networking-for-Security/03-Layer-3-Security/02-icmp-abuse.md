# ICMP and Its Abuse

## Unit 3 - Topic 2 | Layer 3 Security

---

## Overview

**ICMP (Internet Control Message Protocol)** is used for network diagnostics and error reporting (ping, traceroute). While essential for network operations, ICMP can be abused for **reconnaissance**, **denial of service**, **tunneling**, and **man-in-the-middle attacks**.

---

## 1. ICMP Message Types

| Type | Code | Name | Purpose | Abuse Potential |
|:----:|:----:|------|---------|:---------------:|
| 0 | 0 | Echo Reply | Response to ping | Recon |
| 3 | 0-15 | Destination Unreachable | Host/port unreachable | Recon/scanning |
| 5 | 0-3 | Redirect | Route optimization | MITM |
| 8 | 0 | Echo Request | Ping | DoS, tunneling |
| 11 | 0-1 | Time Exceeded | TTL expired | Traceroute recon |

---

## 2. ICMP-Based Attacks

### Ping Flood (ICMP Flood)

```
┌──────────────────────────────────────────────────────────────┐
│              PING FLOOD (DoS)                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker sends massive number of ICMP Echo Requests         │
│  Target must process each one → resource exhaustion          │
│                                                              │
│  ┌──────────┐  Echo Request x 1000000  ┌──────────┐         │
│  │ Attacker │════════════════════════►│  Target  │         │
│  └──────────┘                          │ (overloaded)│       │
│                                        └──────────┘         │
│                                                              │
│  Tool: ping -f target_ip    (Linux flood ping)               │
│  Tool: hping3 -1 --flood target_ip                           │
│                                                              │
│  DEFENSE: Rate-limit ICMP, block at perimeter                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Smurf Attack (Amplification)

```
┌──────────────────────────────────────────────────────────────┐
│              SMURF ATTACK                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker spoofs SOURCE IP to VICTIM's IP                 │
│  2. Sends ICMP Echo Request to BROADCAST address             │
│  3. ALL hosts on network reply to VICTIM                     │
│                                                              │
│  ┌──────────┐  Echo Req (src=VICTIM)  ┌──────────────┐      │
│  │ Attacker │─────────────────────────│  Broadcast   │      │
│  └──────────┘  to broadcast address   │  Network     │      │
│                                        │  (100 hosts) │      │
│                                        └──────┬───────┘      │
│                                               │              │
│          100 Echo Replies to VICTIM ◄─────────┘              │
│                                                              │
│  ┌──────────┐                                                │
│  │  VICTIM  │ ◄── Overwhelmed by 100x amplified traffic      │
│  └──────────┘                                                │
│                                                              │
│  DEFENSE:                                                    │
│  • Disable directed broadcasts (no ip directed-broadcast)    │
│  • Ingress filtering (block spoofed source IPs)              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### ICMP Redirect Attack (MITM)

```
Attacker sends ICMP Redirect (Type 5) to victim:
"A better route to destination X is through ME"

Victim updates routing table → sends traffic through attacker
→ Man-in-the-Middle

DEFENSE:
# Linux — Disable ICMP redirects
echo 0 > /proc/sys/net/ipv4/conf/all/accept_redirects
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects

# Cisco — Disable ICMP redirects
no ip redirects
```

### ICMP Tunneling

```
ICMP tunnel hides data inside ICMP Echo Request/Reply payloads.
Used for: Data exfiltration, bypassing firewalls, C2 channels

Normal ping payload: Random padding data
Tunnel payload: Actual data (commands, stolen files)

Tools: icmpsh, ptunnel, hans

DEFENSE: Deep packet inspection, limit ICMP payload size,
         monitor ICMP traffic patterns
```

---

## 3. ICMP for Reconnaissance

```bash
# Ping sweep — discover live hosts
nmap -sn 192.168.1.0/24          # Network sweep
fping -a -g 192.168.1.0/24      # Fast ping sweep

# Traceroute — map network path
traceroute target.com            # Linux (ICMP/UDP)
tracert target.com               # Windows (ICMP)

# OS fingerprinting via TTL
# TTL 64  → Linux/macOS
# TTL 128 → Windows
# TTL 255 → Cisco/Network device

# ICMP unreachable analysis
# Type 3 Code 1 → Host unreachable (host is down)
# Type 3 Code 3 → Port unreachable (port closed)
# Type 3 Code 13 → Administratively prohibited (firewall)
```

---

## 4. ICMP Security Best Practices

| Practice | Description |
|----------|-------------|
| **Rate-limit ICMP** | Allow limited ICMP, prevent floods |
| **Block at perimeter** | Block unnecessary ICMP types at edge firewall |
| **Disable redirects** | Prevent ICMP redirect MITM attacks |
| **Disable directed broadcast** | Prevent Smurf amplification |
| **Monitor ICMP** | Alert on unusual ICMP traffic (tunneling) |
| **Allow essential types** | Permit Type 3 (unreachable) and Type 11 (TTL exceeded) for network function |

### Recommended Firewall ICMP Policy

| ICMP Type | Direction | Action |
|-----------|-----------|--------|
| Echo Request (8) | Outbound | Allow (rate-limited) |
| Echo Reply (0) | Inbound | Allow (rate-limited) |
| Destination Unreachable (3) | Both | Allow (needed for PMTUD) |
| Time Exceeded (11) | Inbound | Allow (traceroute) |
| Redirect (5) | Both | **DENY** |
| All others | Both | **DENY** |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ICMP** | Diagnostic protocol — no authentication |
| **Ping Flood** | DoS via massive Echo Requests |
| **Smurf** | Amplification via broadcast + spoofed source |
| **Redirect** | MITM by redirecting routing |
| **Tunneling** | Hiding data inside ICMP payloads |
| **Defense** | Rate-limit, filter types, disable redirects |

---

## Quick Revision Questions

1. **What is ICMP used for legitimately?**
2. **How does a Smurf attack achieve amplification?**
3. **How can ICMP redirects be used for MITM attacks?**
4. **What is ICMP tunneling and how is it detected?**
5. **Which ICMP types should be blocked at the firewall?**
6. **How can ICMP be used for OS fingerprinting?**

---

[← Previous: IP Protocol Security](01-ip-protocol-security.md) | [Next: Routing Protocol Security →](03-routing-protocol-security.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*
