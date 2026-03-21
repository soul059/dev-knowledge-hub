# IP Spoofing

## Unit 3 - Topic 4 | Layer 3 Security

---

## Overview

**IP spoofing** is the act of forging the **source IP address** in a packet to disguise the sender's identity or impersonate another system. Because IPv4 has no built-in source address verification, any device can craft packets with any source IP. IP spoofing enables DoS attacks, MITM attacks, and authentication bypass.

---

## 1. How IP Spoofing Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  IP SPOOFING                                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NORMAL PACKET:                                                  │
│  ┌────────────────────────────────────────┐                     │
│  │ Source IP: 10.0.0.5 (Attacker's real)  │                     │
│  │ Dest IP:   10.0.0.1 (Target)           │                     │
│  │ Data: ...                              │                     │
│  └────────────────────────────────────────┘                     │
│                                                                  │
│  SPOOFED PACKET:                                                 │
│  ┌────────────────────────────────────────┐                     │
│  │ Source IP: 10.0.0.100 (FAKE - Victim)  │ ← Forged!          │
│  │ Dest IP:   10.0.0.1 (Target)           │                     │
│  │ Data: ...                              │                     │
│  └────────────────────────────────────────┘                     │
│                                                                  │
│  Target thinks packet came from 10.0.0.100                      │
│  Any replies go to 10.0.0.100 (not the attacker)               │
│  This is a "BLIND" attack — attacker doesn't see replies        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. IP Spoofing Attack Scenarios

| Attack | How Spoofing Is Used | Type |
|--------|---------------------|------|
| **SYN Flood** | Spoof random source IPs → target sends SYN-ACKs to nowhere | DoS |
| **Smurf Attack** | Spoof source as victim → broadcast ping → amplified replies to victim | DoS |
| **DNS Amplification** | Spoof source as victim → large DNS responses flood victim | DDoS |
| **IP-Based Auth Bypass** | Spoof trusted IP to bypass `hosts.allow` or IP whitelists | Access |
| **BGP Hijacking** | Announce routes with spoofed origin | MITM |
| **TCP Session Hijacking** | Inject packets with spoofed source into active session | MITM |

### DNS Amplification with Spoofing

```
┌──────────────────────────────────────────────────────────────┐
│              DNS AMPLIFICATION DDoS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker sends small DNS query (60 bytes)                │
│     Source IP = VICTIM (spoofed!)                            │
│                                                              │
│  2. DNS server sends large reply (3000+ bytes)               │
│     TO THE VICTIM (because source was spoofed)               │
│                                                              │
│  Amplification factor: 50x–100x                              │
│  Multiply by thousands of open DNS resolvers                 │
│                                                              │
│  ┌──────────┐  60B query   ┌──────┐  3000B reply  ┌──────┐ │
│  │ Attacker │─(src=Victim)─│ DNS  │──────────────►│Victim│ │
│  └──────────┘              └──────┘               └──────┘ │
│                                                              │
│  DEFENSE: BCP38 (ingress filtering), disable open resolvers  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Spoofing Tools

```bash
# hping3 — Craft custom packets with spoofed IPs
hping3 -S -a 10.0.0.100 -p 80 target.com
# -S = SYN flag
# -a = spoofed source IP
# -p = destination port

# Scapy — Python packet crafting
from scapy.all import *
pkt = IP(src="10.0.0.100", dst="target.com") / TCP(dport=80, flags="S")
send(pkt)

# Nmap — Decoy scan (partial spoofing)
nmap -D 10.0.0.100,10.0.0.101,ME target.com
# Mixes real scan with decoy spoofed IPs
```

---

## 4. Defenses Against IP Spoofing

| Defense | Description | Where |
|---------|-------------|-------|
| **Ingress Filtering (BCP38)** | Drop packets with source IP not from your network | Edge router |
| **Egress Filtering** | Drop outbound packets with source IP not from your network | Edge router |
| **uRPF** | Unicast Reverse Path Forwarding — validates source IP is reachable via the interface it arrived on | Router |
| **ACLs** | Block private IPs on internet-facing interfaces | Firewall |
| **TCP SYN Cookies** | Validate TCP handshake without state table | Server |
| **Encryption** | TLS/IPsec prevent packet injection even if spoofed | Application/Network |

### uRPF Configuration (Cisco)

```cisco
! Strict mode — source must be reachable via receiving interface
interface GigabitEthernet0/0
  ip verify unicast source reachable-via rx

! Loose mode — source must exist in routing table
interface GigabitEthernet0/0
  ip verify unicast source reachable-via any
```

### Ingress Filtering ACL

```cisco
! Drop packets with private/bogon source IPs on internet-facing interface
access-list 100 deny ip 10.0.0.0 0.255.255.255 any
access-list 100 deny ip 172.16.0.0 0.15.255.255 any
access-list 100 deny ip 192.168.0.0 0.0.255.255 any
access-list 100 deny ip 127.0.0.0 0.255.255.255 any
access-list 100 permit ip any any

interface GigabitEthernet0/0    ! Internet-facing
  ip access-group 100 in
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IP Spoofing** | Forging source IP — enabled by IPv4's lack of verification |
| **Blind Spoofing** | Attacker doesn't see replies (they go to spoofed IP) |
| **Main Uses** | DoS (SYN flood, amplification), auth bypass, MITM |
| **BCP38** | Internet standard for ingress filtering at ISP edge |
| **uRPF** | Router validates source IP reachability |
| **Ultimate Defense** | Encryption (TLS/IPsec) — spoofed packets can't join encrypted sessions |

---

## Quick Revision Questions

1. **Why is IP spoofing possible in IPv4?**
2. **What is "blind" spoofing and why is it called that?**
3. **How does IP spoofing enable DNS amplification DDoS?**
4. **What is BCP38 and how does it prevent spoofing?**
5. **What is uRPF and what are its two modes?**
6. **Why does encryption effectively prevent spoofing-based MITM?**

---

[← Previous: Routing Protocol Security](03-routing-protocol-security.md) | [Next: Fragmentation Attacks →](05-fragmentation-attacks.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*
