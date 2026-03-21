# Firewalls

## Unit 8 - Topic 1 | Network Security Devices

---

## Overview

A **firewall** is a network security device that monitors and controls incoming and outgoing network traffic based on predetermined security rules. Firewalls are the **cornerstone of network security**, establishing a barrier between trusted internal networks and untrusted external networks.

---

## 1. Firewall Types

```
┌──────────────────────────────────────────────────────────────┐
│              FIREWALL EVOLUTION                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  GENERATION 1: PACKET FILTER (Stateless)                     │
│  • Examines individual packets (src/dst IP, port, protocol)  │
│  • No connection state tracking                              │
│  • Fast but easily bypassed                                  │
│                                                              │
│  GENERATION 2: STATEFUL INSPECTION                           │
│  • Tracks connection state (NEW, ESTABLISHED, RELATED)       │
│  • Understands TCP handshakes                                │
│  • More secure, standard for years                           │
│                                                              │
│  GENERATION 3: APPLICATION LAYER (Proxy)                     │
│  • Deep packet inspection (DPI)                              │
│  • Understands application protocols (HTTP, FTP, DNS)        │
│  • Can filter by content, not just headers                   │
│                                                              │
│  GENERATION 4: NEXT-GENERATION FIREWALL (NGFW)               │
│  • Application awareness and control                         │
│  • Integrated IPS                                            │
│  • User identity awareness                                   │
│  • Threat intelligence feeds                                  │
│  • SSL/TLS inspection                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Firewall Comparison

| Type | Layer | State | DPI | Speed | Security |
|------|:-----:|:-----:|:---:|:-----:|:--------:|
| **Packet Filter** | 3-4 | ❌ | ❌ | ★★★★★ | ★★ |
| **Stateful** | 3-4 | ✅ | ❌ | ★★★★ | ★★★ |
| **Application Proxy** | 7 | ✅ | ✅ | ★★ | ★★★★ |
| **NGFW** | 3-7 | ✅ | ✅ | ★★★ | ★★★★★ |

---

## 3. Firewall Rule Structure

```
# RULE FORMAT: Action | Source | Destination | Port | Protocol

# Example iptables rules (Linux)
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -j DROP              # Default deny

# Rule processing order:
# Rules are evaluated TOP to BOTTOM
# First match wins
# Default rule (policy) at the bottom

RULE BEST PRACTICES:
┌────┬──────────┬───────────┬───────────┬──────┬──────────┐
│ #  │ Action   │ Source    │ Dest      │ Port │ Proto    │
├────┼──────────┼───────────┼───────────┼──────┼──────────┤
│ 1  │ ALLOW    │ Admin Net │ Firewall  │ 22   │ TCP      │
│ 2  │ ALLOW    │ Any       │ Web Srv   │ 443  │ TCP      │
│ 3  │ ALLOW    │ Internal  │ DNS Srv   │ 53   │ UDP/TCP  │
│ 4  │ DENY     │ Any       │ Any       │ Any  │ Any      │
└────┴──────────┴───────────┴───────────┴──────┴──────────┘
```

---

## 4. Firewall Deployment

```
                    INTERNET
                       │
                 ┌─────┴─────┐
                 │ EXTERNAL  │
                 │ FIREWALL  │
                 └─────┬─────┘
                       │
              ┌────────┼────────┐
              │        │        │
         ┌────┴───┐ ┌──┴───┐ ┌─┴────┐
         │ DMZ    │ │ DMZ  │ │ DMZ  │
         │ Web Srv│ │ Mail │ │ DNS  │
         └────────┘ └──────┘ └──────┘
                       │
                 ┌─────┴─────┐
                 │ INTERNAL  │
                 │ FIREWALL  │
                 └─────┬─────┘
                       │
              ┌────────┼────────┐
              │        │        │
         ┌────┴───┐ ┌──┴───┐ ┌─┴────┐
         │ Users  │ │ Srv  │ │ DB   │
         └────────┘ └──────┘ └──────┘
```

---

## 5. Firewall Evasion Techniques

| Technique | Description | Countermeasure |
|-----------|-------------|----------------|
| **Fragmentation** | Split attack across fragments | Fragment reassembly |
| **Port hopping** | Use allowed ports (80, 443) | Application-layer inspection |
| **Tunneling** | Encapsulate in allowed protocols | DPI, protocol validation |
| **Encryption** | Hide payload in TLS | SSL/TLS inspection |
| **Source routing** | Manipulate packet path | Block source-routed packets |

---

## 6. Major Firewall Vendors

| Vendor | Product | Type |
|--------|---------|------|
| **Palo Alto** | PA-Series | NGFW |
| **Fortinet** | FortiGate | NGFW |
| **Cisco** | Firepower | NGFW |
| **Check Point** | Quantum | NGFW |
| **pfSense** | pfSense | Open Source |
| **Linux** | iptables/nftables | Host-based |
| **Windows** | Windows Firewall | Host-based |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Firewall** | Controls traffic based on rules |
| **Stateless** | Examines individual packets only |
| **Stateful** | Tracks connection state |
| **NGFW** | App awareness, IPS, user identity |
| **Default Deny** | Block everything not explicitly allowed |
| **Rule Order** | First match wins, process top-to-bottom |

---

## Quick Revision Questions

1. **What is the difference between stateless and stateful firewalls?**
2. **What capabilities make a NGFW "next-generation"?**
3. **Why is "default deny" important in firewall rules?**
4. **What are 3 firewall evasion techniques?**
5. **Where should firewalls be placed in a network?**

---

[Next: IDS/IPS →](02-ids-ips.md)

---

*Unit 8 - Topic 1 of 6 | [Back to README](../README.md)*
