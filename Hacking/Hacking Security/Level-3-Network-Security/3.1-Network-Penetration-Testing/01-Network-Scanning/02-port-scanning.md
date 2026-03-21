# Port Scanning (TCP, UDP, SYN, Stealth)

## Unit 1 - Topic 2 | Network Scanning

---

## Overview

**Port scanning** probes target hosts to identify open ports and determine which services are accessible. Different scan types offer varying trade-offs between speed, accuracy, stealth, and the type of information returned.

---

## 1. TCP Scan Types Comparison

```
TCP THREE-WAY HANDSHAKE REFERENCE:
Client ──── SYN ────→ Server
Client ←── SYN/ACK ── Server
Client ──── ACK ────→ Server  (Connection established)

┌────────────────────────────────────────────────────────────┐
│ SCAN TYPE    │ SENDS    │ OPEN        │ CLOSED    │ NOTES  │
├──────────────┼──────────┼─────────────┼───────────┼────────┤
│ TCP Connect  │ SYN      │ SYN/ACK→ACK │ RST       │ Logged │
│ SYN (Half)   │ SYN      │ SYN/ACK→RST │ RST       │ Stealth│
│ FIN          │ FIN      │ No response │ RST       │ Stealth│
│ XMAS         │ FIN+PSH+ │ No response │ RST       │ Stealth│
│              │ URG      │             │           │        │
│ NULL         │ No flags │ No response │ RST       │ Stealth│
│ ACK          │ ACK      │ RST         │ RST       │ FW map │
│ Window       │ ACK      │ RST (win>0) │ RST (w=0) │ FW map │
└────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Scan Types

```bash
# === TCP CONNECT SCAN (-sT) ===
nmap -sT -p 1-1000 target.com
# • Completes full 3-way handshake
# • No root/sudo required
# • Most reliable detection
# • ⚠️ Easily logged by target (full connection)
# • Use when: no root access, need reliability

# === SYN SCAN (-sS) "Half-Open" ===
sudo nmap -sS -p 1-1000 target.com
# • Sends SYN, receives SYN/ACK, sends RST (never completes)
# • Faster than TCP Connect
# • Less likely to be logged (no full connection)
# • DEFAULT scan with root/sudo
# • Use when: standard scan, stealth desired

# === UDP SCAN (-sU) ===
sudo nmap -sU --top-ports 100 target.com
# • Sends empty UDP datagram (or protocol-specific)
# • Open: UDP response or no response
# • Closed: ICMP Port Unreachable
# • ⚠️ VERY SLOW (no handshake = must wait for timeout)
# • Critical: finds DNS(53), SNMP(161), TFTP(69), NTP(123)
# • Use when: need full picture (TCP + UDP)

# === COMBINED TCP + UDP ===
sudo nmap -sS -sU --top-ports 100 target.com
# Scans both TCP and UDP simultaneously

# === FIN SCAN (-sF) ===
sudo nmap -sF -p 1-1000 target.com
# • Sends FIN flag only
# • Open ports: no response (per RFC 793)
# • Closed ports: RST
# • Bypasses simple packet filters checking for SYN
# • ⚠️ Doesn't work on Windows (always sends RST)

# === XMAS SCAN (-sX) ===
sudo nmap -sX -p 1-1000 target.com
# • Sends FIN + PSH + URG flags ("lit up like a Christmas tree")
# • Same behavior as FIN scan
# • ⚠️ Doesn't work on Windows

# === NULL SCAN (-sN) ===
sudo nmap -sN -p 1-1000 target.com
# • Sends packet with NO flags set
# • Same behavior as FIN/XMAS
# • ⚠️ Doesn't work on Windows

# === ACK SCAN (-sA) ===
sudo nmap -sA -p 1-1000 target.com
# • Sends ACK flag
# • Does NOT determine open/closed
# • Determines: filtered vs unfiltered
# • Maps firewall rules
```

---

## 3. Port Selection Strategies

```bash
# === COMMON PORT SELECTIONS ===
nmap target.com                       # Default: top 1000 ports
nmap -p 80 target.com                 # Single port
nmap -p 80,443,8080 target.com        # Multiple specific ports
nmap -p 1-1024 target.com             # Port range
nmap -p- target.com                   # ALL 65535 ports
nmap --top-ports 100 target.com       # Top 100 most common
nmap --top-ports 10 target.com        # Top 10 most common
nmap -p U:53,161,T:80,443 target.com  # TCP and UDP specific

# === CRITICAL PORTS TO ALWAYS CHECK ===
# TCP:
# 21(FTP), 22(SSH), 23(Telnet), 25(SMTP),
# 53(DNS), 80(HTTP), 110(POP3), 111(RPC),
# 135(MSRPC), 139(NetBIOS), 143(IMAP), 443(HTTPS),
# 445(SMB), 993(IMAPS), 995(POP3S), 1433(MSSQL),
# 1521(Oracle), 3306(MySQL), 3389(RDP), 5432(PostgreSQL),
# 5900(VNC), 6379(Redis), 8080(HTTP-Alt), 8443(HTTPS-Alt),
# 27017(MongoDB)

# UDP:
# 53(DNS), 67-68(DHCP), 69(TFTP), 123(NTP),
# 161(SNMP), 500(IKE/VPN), 514(Syslog), 1900(SSDP)
```

---

## 4. Port States

| State | Meaning | Implication |
|:-----:|---------|-------------|
| **open** | Service listening and accepting connections | Attack surface — investigate |
| **closed** | No service listening, but host is alive | Host responds but no service |
| **filtered** | Firewall/filter dropping packets | Can't determine state |
| **unfiltered** | Accessible but can't determine open/closed | ACK scan result |
| **open\|filtered** | Can't determine which (UDP/FIN/XMAS) | Ambiguous — try other scan |
| **closed\|filtered** | Can't determine which (rare) | Ambiguous |

---

## 5. Scan Type Selection Guide

```
WHICH SCAN TYPE TO USE?

START WITH:
└── sudo nmap -sS target.com  (SYN scan — fast, reliable, stealth)

NEED UDP SERVICES?
└── sudo nmap -sU --top-ports 50 target.com

FIREWALL BLOCKING SYN?
├── Try: -sF (FIN scan)
├── Try: -sX (XMAS scan)
├── Try: -sN (NULL scan)
└── Try: -sA (ACK scan to map firewall rules)

NO ROOT ACCESS?
└── nmap -sT target.com  (TCP Connect — only option without root)

NEED FULL COVERAGE?
└── sudo nmap -sS -sU -p- target.com  (ALL TCP + UDP ports)
    ⚠️ This will take hours!

MAP FIREWALL RULES?
└── sudo nmap -sA target.com  (ACK scan — filtered vs unfiltered)
```

---

## Defensive Countermeasures

| Scan Type | Detection Method |
|-----------|-----------------|
| TCP Connect | Connection logs, IDS signature |
| SYN scan | IDS (half-open detection), SYN flood alerts |
| FIN/XMAS/NULL | IDS signature for unusual flag combos |
| UDP scan | ICMP unreachable rate monitoring |
| Any scan | Port scan detection (many ports in short time) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SYN scan** | Default, fast, semi-stealth |
| **TCP Connect** | Full connection, no root needed |
| **UDP scan** | Slow but essential for DNS/SNMP |
| **FIN/XMAS/NULL** | Bypass simple firewalls (not Windows) |
| **ACK scan** | Map firewall rules, not port states |
| **-p-** | Scan all 65535 ports |

---

## Quick Revision Questions

1. **Why is SYN scanning called "half-open"?**
2. **When is TCP Connect scan the only option?**
3. **Why is UDP scanning so slow?**
4. **What do FIN, XMAS, and NULL scans exploit?**
5. **What does an ACK scan tell you that a SYN scan doesn't?**
6. **Name 5 critical ports every pen tester should check.**

---

[← Previous: Host Discovery Techniques](01-host-discovery-techniques.md) | [Next: Nmap Fundamentals →](03-nmap-fundamentals.md)

---

*Unit 1 - Topic 2 of 6 | [Back to README](../README.md)*
