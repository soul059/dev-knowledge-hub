# TCP Attacks (SYN Flood, Session Hijacking)

## Unit 4 - Topic 2 | Layer 4 Security

---

## Overview

TCP was designed for reliable communication, not security. Attackers exploit TCP's connection management to launch **SYN flood** (denial of service), **session hijacking** (connection takeover), and **RST injection** (connection disruption) attacks. Understanding these attacks is essential for network defense.

---

## 1. SYN Flood Attack

```
┌──────────────────────────────────────────────────────────────────┐
│                  SYN FLOOD (DoS Attack)                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NORMAL:           ATTACK:                                       │
│  Client→SYN        Attacker sends thousands of SYN packets      │
│  Server→SYN-ACK    with SPOOFED source IPs                      │
│  Client→ACK ✅      Server sends SYN-ACK to fake IPs            │
│                     No ACK ever comes back                      │
│                     Server's connection table FILLS UP           │
│                                                                  │
│  ┌──────────┐  SYN SYN SYN SYN SYN  ┌──────────────┐          │
│  │ Attacker │═══════════════════════►│   Server     │          │
│  │(spoofed) │                        │ ┌──────────┐ │          │
│  └──────────┘                        │ │Half-open │ │          │
│                                      │ │conn table│ │          │
│  Server waits for ACKs that          │ │FULL! ❌  │ │          │
│  will NEVER arrive.                  │ └──────────┘ │          │
│  Legitimate users can't connect! ❌   └──────────────┘          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### SYN Flood Tools

```bash
# hping3 — SYN flood
hping3 -S --flood -V -p 80 target.com
hping3 -S -a 10.0.0.100 --flood -p 80 target.com   # With spoofing

# Scapy
from scapy.all import *
while True:
    pkt = IP(src=RandIP(), dst="target") / TCP(dport=80, flags="S")
    send(pkt, verbose=0)
```

### SYN Flood Defenses

| Defense | Description |
|---------|-------------|
| **SYN Cookies** | Server doesn't store state until handshake completes |
| **SYN Proxy** | Firewall completes handshake on server's behalf |
| **Rate Limiting** | Limit SYN packets per source IP |
| **Increase Backlog** | Larger half-open connection queue |
| **Reduce SYN Timeout** | Shorter wait for ACK (e.g., 10 sec vs 75 sec) |
| **DDoS Mitigation** | Cloud services (Cloudflare, AWS Shield) |

```bash
# Linux — Enable SYN cookies
echo 1 > /proc/sys/net/ipv4/tcp_syncookies

# Linux — Increase backlog queue
echo 4096 > /proc/sys/net/ipv4/tcp_max_syn_backlog

# Linux — Reduce SYN timeout
echo 2 > /proc/sys/net/ipv4/tcp_synack_retries
```

---

## 2. TCP Session Hijacking

```
┌──────────────────────────────────────────────────────────────────┐
│                  TCP SESSION HIJACKING                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Attacker takes over an ESTABLISHED TCP session                  │
│  by predicting or sniffing sequence numbers.                    │
│                                                                  │
│  BEFORE:                                                         │
│  Client ◄══════════════════════════► Server                     │
│         ESTABLISHED (seq=12345)                                 │
│                                                                  │
│  ATTACK:                                                         │
│  1. Attacker sniffs traffic to learn sequence numbers           │
│  2. Attacker injects packet with correct seq/ack numbers        │
│  3. Server accepts it as legitimate                             │
│  4. Attacker can inject commands or data                        │
│                                                                  │
│  Client ◄──────────────────────────► Server                     │
│              ▲                          │                        │
│              │   ATTACKER injects       │                        │
│              │   packet with            │                        │
│              │   seq=12346              │                        │
│              └──────────────────────────┘                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Session Hijacking Defenses

| Defense | Description |
|---------|-------------|
| **Encryption (TLS)** | Encrypted sessions can't be hijacked (can't read/inject) |
| **Randomized ISN** | Random Initial Sequence Numbers (modern OS default) |
| **IPsec** | Network-layer encryption prevents injection |
| **Network segmentation** | Limits attacker's ability to sniff traffic |

---

## 3. TCP RST Injection

```
Attacker sends RST (Reset) packet with correct seq number
→ Connection immediately terminated

Used for:
• Censorship (Great Firewall of China)
• Disrupting specific connections
• DoS against individual sessions

Tool: tcpkill (dsniff suite)
tcpkill -i eth0 host target.com
```

---

## 4. TCP Sequence Prediction

```
Old systems used predictable sequence numbers:
ISN = previous ISN + constant increment

Modern systems use RANDOM ISN → much harder to hijack

Check ISN randomness:
nmap -O target.com    # OS detection includes ISN analysis
```

---

## Summary Table

| Attack | Description | Defense |
|--------|-------------|---------|
| **SYN Flood** | Exhaust connection table with half-open connections | SYN cookies, rate limiting |
| **Session Hijacking** | Inject into active TCP session using seq numbers | TLS encryption |
| **RST Injection** | Force-close connections with fake RST | Encryption, IPsec |
| **Seq Prediction** | Predict ISN to hijack without sniffing | Random ISN (modern OS) |

---

## Quick Revision Questions

1. **How does a SYN flood cause denial of service?**
2. **What are SYN cookies and how do they help?**
3. **What information does an attacker need for TCP session hijacking?**
4. **Why does TLS prevent TCP session hijacking?**
5. **What is RST injection and how is it used for censorship?**

---

[← Previous: TCP Handshake](01-tcp-three-way-handshake.md) | [Next: UDP Security →](03-udp-security.md)

---

*Unit 4 - Topic 2 of 5 | [Back to README](../README.md)*
