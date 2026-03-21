# Port Scanning Techniques

## Unit 4 - Topic 4 | Layer 4 Security

---

## Overview

**Port scanning** is the process of probing a target system to discover which network ports are open, closed, or filtered. It's a fundamental technique in both **penetration testing** (finding attack surfaces) and **defense** (auditing your own exposure). Nmap is the industry-standard tool.

---

## 1. Port States

| State | Meaning | Response |
|-------|---------|----------|
| **Open** | Service is listening and accepting connections | SYN-ACK (TCP), response (UDP) |
| **Closed** | No service listening, but host is reachable | RST (TCP), ICMP unreachable (UDP) |
| **Filtered** | Firewall is blocking — can't determine state | No response or ICMP admin prohibited |
| **Unfiltered** | Port is accessible but can't determine open/closed | RST (ACK scan) |
| **Open/Filtered** | Can't determine if open or filtered | No response (UDP, FIN scan) |

---

## 2. TCP Scan Types

```
┌──────────────────────────────────────────────────────────────────┐
│                  TCP SCAN TYPES                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SYN SCAN (-sS) "Half-Open" — Most common, stealthy            │
│  ┌──────┐ SYN  ┌──────┐        ┌──────┐ SYN  ┌──────┐         │
│  │Client│─────►│Server│        │Client│─────►│Server│         │
│  │      │◄─────│ OPEN │        │      │◄─────│CLOSED│         │
│  │      │SYN-ACK      │        │      │ RST  │      │         │
│  │      │─────►│      │        └──────┘      └──────┘         │
│  │      │ RST  │      │  OPEN ✅              CLOSED ❌         │
│  └──────┘      └──────┘                                        │
│  Never completes handshake → less likely to be logged            │
│                                                                  │
│  CONNECT SCAN (-sT) "Full Connect" — Reliable but noisy         │
│  Completes full 3-way handshake → logged by application          │
│                                                                  │
│  FIN SCAN (-sF) — Stealth scan                                  │
│  Sends FIN → Open port: no response, Closed: RST               │
│                                                                  │
│  XMAS SCAN (-sX) — FIN+PSH+URG flags                           │
│  Same logic as FIN scan, "lights up like a Christmas tree"      │
│                                                                  │
│  NULL SCAN (-sN) — No flags set                                 │
│  Same logic as FIN scan                                         │
│                                                                  │
│  ACK SCAN (-sA) — Firewall mapping                              │
│  Sends ACK → RST = unfiltered, No response = filtered          │
│  Reveals firewall rules, not open/closed ports                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Nmap Command Reference

```bash
# DISCOVERY
nmap -sn 192.168.1.0/24              # Ping sweep (host discovery)
nmap -Pn target.com                   # Skip ping, assume host is up

# TCP SCANS
nmap -sS target.com                   # SYN scan (default with root)
nmap -sT target.com                   # Connect scan (no root needed)
nmap -sA target.com                   # ACK scan (firewall mapping)

# UDP SCAN
nmap -sU target.com                   # UDP scan (slow)

# VERSION & OS DETECTION
nmap -sV target.com                   # Service version detection
nmap -O target.com                    # OS fingerprinting
nmap -A target.com                    # Aggressive (OS + version + scripts)

# PORT SPECIFICATION
nmap -p 80,443,8080 target.com        # Specific ports
nmap -p 1-1000 target.com             # Port range
nmap -p- target.com                   # ALL 65535 ports
nmap --top-ports 100 target.com       # Top 100 common ports

# SPEED & STEALTH
nmap -T0 target.com                   # Paranoid (very slow, IDS evasion)
nmap -T1 target.com                   # Sneaky
nmap -T3 target.com                   # Normal (default)
nmap -T4 target.com                   # Aggressive (fast)
nmap -T5 target.com                   # Insane (fastest, loud)

# EVASION
nmap -f target.com                    # Fragment packets
nmap -D RND:5 target.com             # Decoy scan (5 random decoys)
nmap --source-port 53 target.com      # Spoof source port as DNS
nmap -S 10.0.0.100 target.com        # Spoof source IP

# OUTPUT
nmap -oN scan.txt target.com          # Normal output
nmap -oX scan.xml target.com          # XML output
nmap -oG scan.gnmap target.com        # Grepable output
nmap -oA scan target.com              # All formats
```

---

## 4. Other Port Scanning Tools

| Tool | Use Case |
|------|----------|
| **Masscan** | Fastest port scanner — scans entire internet in minutes |
| **Unicornscan** | Fast, accurate, good for UDP |
| **RustScan** | Fast Rust-based scanner, feeds results to Nmap |
| **Netcat (nc)** | Simple port scanning and banner grabbing |
| **Zmap** | Internet-wide single-port scanning |

```bash
# Masscan — Fast scanning
masscan 192.168.1.0/24 -p 1-65535 --rate 1000

# Netcat — Banner grab
nc -v target.com 80
nc -zv target.com 20-100           # Port range scan

# RustScan — Feed to Nmap
rustscan -a target.com -- -sV -sC
```

---

## 5. Defending Against Port Scanning

| Defense | Description |
|---------|-------------|
| **Firewall** | Block unnecessary ports (default deny) |
| **IDS/IPS** | Detect and alert on scanning patterns |
| **Port knocking** | Open ports only after secret knock sequence |
| **Tarpit/honeypot** | Slow down or trap scanners |
| **Minimize services** | Reduce open ports to only what's needed |
| **Rate limiting** | Limit connection attempts per source IP |

---

## Summary Table

| Scan Type | Flag | Stealth | Root Required | Best For |
|-----------|------|:-------:|:-------------:|---------|
| **SYN** | -sS | ✅ High | Yes | Default scan, most common |
| **Connect** | -sT | ❌ Low | No | When no root access |
| **FIN** | -sF | ✅ High | Yes | Firewall evasion |
| **Xmas** | -sX | ✅ High | Yes | Firewall evasion |
| **ACK** | -sA | — | Yes | Firewall rule mapping |
| **UDP** | -sU | — | Yes | UDP service discovery |

---

## Quick Revision Questions

1. **What is the difference between SYN scan and Connect scan?**
2. **What does a "filtered" port state indicate?**
3. **How does an ACK scan help map firewall rules?**
4. **What Nmap flags scan ALL 65535 ports?**
5. **Name 3 Nmap evasion techniques.**
6. **What defenses can detect or prevent port scanning?**

---

[← Previous: UDP Security](03-udp-security.md) | [Next: Firewall Evasion Basics →](05-firewall-evasion-basics.md)

---

*Unit 4 - Topic 4 of 5 | [Back to README](../README.md)*
