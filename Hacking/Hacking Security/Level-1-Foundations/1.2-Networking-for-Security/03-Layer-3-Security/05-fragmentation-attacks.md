# Fragmentation Attacks

## Unit 3 - Topic 5 | Layer 3 Security

---

## Overview

**IP fragmentation** breaks large packets into smaller pieces to fit through networks with smaller **MTU (Maximum Transmission Unit)**. Attackers exploit fragmentation to **evade firewalls and IDS**, **crash systems**, and **bypass packet inspection**. Understanding fragmentation attacks is essential for both pen testers and defenders.

---

## 1. How IP Fragmentation Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  NORMAL FRAGMENTATION                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Original Packet (4000 bytes)                                   │
│  ┌──────────────────────────────────────────┐                   │
│  │ IP Header │         Payload (3980 bytes)  │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
│  MTU = 1500 bytes (Ethernet standard)                           │
│  Must fragment into smaller pieces:                             │
│                                                                  │
│  Fragment 1:                                                     │
│  ┌──────────┬────────────────────┐                              │
│  │ IP Hdr   │ Payload (1480 B)   │  Offset: 0, MF=1            │
│  └──────────┴────────────────────┘                              │
│                                                                  │
│  Fragment 2:                                                     │
│  ┌──────────┬────────────────────┐                              │
│  │ IP Hdr   │ Payload (1480 B)   │  Offset: 185, MF=1          │
│  └──────────┴────────────────────┘                              │
│                                                                  │
│  Fragment 3:                                                     │
│  ┌──────────┬────────────────────┐                              │
│  │ IP Hdr   │ Payload (1020 B)   │  Offset: 370, MF=0 (last)   │
│  └──────────┴────────────────────┘                              │
│                                                                  │
│  Destination reassembles fragments using offsets.               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Fragmentation-Based Attacks

### Tiny Fragment Attack

```
Split TCP header across two fragments so firewall can't read ports:

Fragment 1: [IP Header][Partial TCP Header (8 bytes)]
  → Only has source port, no destination port or flags
  → Firewall can't determine if it's HTTP, SSH, etc.
  → Firewall passes it through ✅ (can't inspect)

Fragment 2: [IP Header][Rest of TCP Header + Data]
  → Contains destination port and flags
  → But firewall already let Fragment 1 through

Target reassembles → complete malicious packet delivered!
```

### Overlapping Fragment Attack (Teardrop)

```
Send fragments with OVERLAPPING offsets:

Fragment 1: Offset 0, Length 100
Fragment 2: Offset 80, Length 100  ← Overlaps with Fragment 1!

Poorly implemented TCP/IP stacks CRASH when trying to reassemble.
→ Denial of Service (Blue Screen on old Windows)

Variation: Overwrite first fragment's port info with second
fragment to bypass firewall rules.
```

### Ping of Death

```
Send an ICMP packet larger than 65,535 bytes (max IP packet size)
by manipulating fragment offsets:

Fragment offset + fragment size > 65,535 bytes

When reassembled → buffer overflow → system crash

HISTORICAL: Affected Windows 95/98, old Linux kernels
MODERN: Patched in all modern OS, but concept still relevant
```

### Rose Attack / Fragment Flood

```
Send thousands of incomplete fragment sets:
→ Target allocates memory for reassembly
→ Fragments never complete → memory exhaustion → DoS

DEFENSE: Fragment reassembly timeout (60 seconds default)
```

---

## 3. Fragmentation for IDS/Firewall Evasion

```bash
# Nmap — Fragment scan to evade IDS
nmap -f target.com              # 8-byte fragments
nmap -f -f target.com           # 16-byte fragments
nmap --mtu 24 target.com        # Custom fragment size

# fragrouter — Fragment traffic in transit
fragrouter -B1                  # Basic fragmentation

# hping3 — Custom fragmented packets
hping3 -f -p 80 target.com     # Fragmented SYN to port 80
```

---

## 4. Defenses Against Fragmentation Attacks

| Defense | Description |
|---------|-------------|
| **Reassemble at firewall** | Firewall reassembles fragments before inspection |
| **Block tiny fragments** | Drop fragments smaller than minimum header size |
| **Fragment timeout** | Limit time to wait for all fragments (e.g., 15 sec) |
| **Block overlapping fragments** | Drop fragments with overlapping offsets |
| **Path MTU Discovery** | Use PMTUD to avoid fragmentation entirely |
| **DF bit** | Set Don't Fragment bit — force PMTUD |
| **IDS/IPS with reassembly** | Modern IDS reassembles before analyzing |

```cisco
! Block tiny fragments (Cisco ACL)
access-list 100 deny ip any any fragments

! Set fragment reassembly timeout
ip virtual-reassembly timeout 15
ip virtual-reassembly max-reassemblies 64
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Fragmentation** | Breaking packets to fit MTU — exploitable process |
| **Tiny Fragments** | Split TCP header to evade firewall inspection |
| **Overlapping** | Overwrite data during reassembly — DoS or evasion |
| **Ping of Death** | Oversized packet causes buffer overflow (historical) |
| **Evasion** | Nmap `-f` fragments scans to bypass IDS |
| **Defense** | Reassemble at firewall, block tiny/overlapping fragments |

---

## Quick Revision Questions

1. **Why does IP fragmentation exist?**
2. **How do tiny fragment attacks evade firewalls?**
3. **What is an overlapping fragment attack?**
4. **How does Nmap use fragmentation for evasion?**
5. **What is the best firewall defense against fragmentation attacks?**
6. **What is the Ping of Death and why was it effective?**

---

[← Previous: IP Spoofing](04-ip-spoofing.md) | [Next: IPv6 Security →](06-ipv6-security.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*
