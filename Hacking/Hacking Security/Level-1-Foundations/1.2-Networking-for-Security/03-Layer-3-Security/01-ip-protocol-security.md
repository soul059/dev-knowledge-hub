# IP Protocol Security Issues

## Unit 3 - Topic 1 | Layer 3 Security

---

## Overview

The **Internet Protocol (IP)** is the foundation of all network communication, but it was designed for functionality — not security. IPv4 has inherent weaknesses: no built-in authentication, no encryption, and a header that can be easily manipulated. Understanding these security issues is essential for both attackers and defenders.

---

## 1. IPv4 Header — Attack Surface

```
┌──────────────────────────────────────────────────────────────────┐
│                  IPv4 HEADER (20-60 bytes)                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  0       4       8      16      19          31                  │
│  ├───────┼───────┼───────┼───────┼───────────┤                  │
│  │Version│  IHL  │  ToS  │    Total Length    │                  │
│  ├───────┴───────┼───────┼───────────────────┤                  │
│  │ Identification│ Flags │ Fragment Offset    │ ← Fragmentation │
│  ├───────────────┼───────┼───────────────────┤                  │
│  │     TTL       │Protocol│  Header Checksum  │ ← TTL tricks    │
│  ├───────────────┴───────┴───────────────────┤                  │
│  │          Source IP Address                 │ ← SPOOFABLE!    │
│  ├───────────────────────────────────────────┤                  │
│  │        Destination IP Address              │                  │
│  ├───────────────────────────────────────────┤                  │
│  │          Options (if IHL > 5)              │ ← Abuse possible│
│  └───────────────────────────────────────────┘                  │
│                                                                  │
│  KEY VULNERABILITIES:                                            │
│  • Source IP can be FORGED (IP spoofing)                        │
│  • No authentication of sender                                 │
│  • No encryption (data travels in plaintext)                   │
│  • Fragmentation can be abused                                 │
│  • TTL can be manipulated for evasion                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. IP-Based Attacks Summary

| Attack | Description | Layer | Defense |
|--------|-------------|:-----:|---------|
| **IP Spoofing** | Forging source IP address | 3 | Ingress/egress filtering, uRPF |
| **Fragmentation** | Abusing IP fragmentation to evade detection | 3 | Reassembly at firewall |
| **ICMP Abuse** | Ping flood, Smurf, redirect | 3 | Rate limiting, ICMP filtering |
| **Route Hijacking** | Injecting false routing info | 3 | Route authentication |
| **TTL Manipulation** | Evading IDS/traceroute | 3 | Deep packet inspection |
| **IP Options Abuse** | Source routing, record route | 3 | Disable IP options |

---

## 3. TTL (Time to Live) Security

```
┌──────────────────────────────────────────────────────────────┐
│              TTL — SECURITY IMPLICATIONS                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TTL starts at a value (64, 128, or 255) and decrements     │
│  by 1 at each router hop. Packet dies when TTL = 0.         │
│                                                              │
│  LEGITIMATE USE:                                             │
│  • Prevents infinite routing loops                           │
│  • traceroute uses incrementing TTL to map hops              │
│                                                              │
│  ATTACK USE:                                                 │
│  • IDS Evasion: Send packets with TTL that reach IDS         │
│    but expire BEFORE reaching the target                     │
│  • OS Fingerprinting: Default TTL reveals the OS             │
│    Linux = 64, Windows = 128, Cisco = 255                    │
│  • TTL Expiry Attack: Flood router with TTL=1 packets       │
│    → Router generates ICMP Time Exceeded → CPU overload      │
│                                                              │
│  DEFENSE:                                                    │
│  • Rate-limit ICMP Time Exceeded messages                    │
│  • Don't rely solely on IDS at one point                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. IP Options Abuse

| Option | Description | Abuse |
|--------|-------------|-------|
| **Loose Source Routing** | Sender specifies route hops | Bypass firewalls, access internal networks |
| **Strict Source Routing** | Sender specifies exact route | Same as above |
| **Record Route** | Records IP addresses of each hop | Network reconnaissance |
| **Timestamp** | Records time at each hop | Timing attacks |

```cisco
! DEFENSE: Disable IP source routing (Cisco)
no ip source-route

! Block IP options at firewall
! Most firewalls drop packets with IP options by default
```

---

## 5. IPsec — The Solution

```
┌──────────────────────────────────────────────────────────────┐
│              IPsec — SECURING IP                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  IPsec adds security at Layer 3:                             │
│                                                              │
│  AH (Authentication Header):                                 │
│  • Integrity (hash)                                          │
│  • Authentication (who sent it)                              │
│  • NO encryption                                             │
│                                                              │
│  ESP (Encapsulating Security Payload):                       │
│  • Integrity                                                 │
│  • Authentication                                            │
│  • ENCRYPTION ✅                                              │
│                                                              │
│  MODES:                                                      │
│  Transport Mode: Encrypts payload only                       │
│  Tunnel Mode: Encrypts entire original packet (VPN)          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IPv4 Design** | No built-in authentication or encryption |
| **Source IP** | Can be spoofed — no verification mechanism |
| **Fragmentation** | Can bypass firewalls and IDS |
| **TTL** | Reveals OS, can evade IDS |
| **IP Options** | Source routing bypasses security controls |
| **Solution** | IPsec (ESP) provides encryption + auth at Layer 3 |

---

## Quick Revision Questions

1. **Why is the IP protocol inherently insecure?**
2. **What fields in the IPv4 header can be manipulated by attackers?**
3. **How can TTL be used for IDS evasion?**
4. **What is IP source routing and why is it dangerous?**
5. **What is the difference between IPsec AH and ESP?**
6. **What are IPsec transport mode and tunnel mode?**

---

[Next: ICMP and Its Abuse →](02-icmp-abuse.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
