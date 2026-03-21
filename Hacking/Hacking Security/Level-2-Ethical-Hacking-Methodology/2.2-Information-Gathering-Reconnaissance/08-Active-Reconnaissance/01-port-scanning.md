# Port Scanning

## Unit 8 - Topic 1 | Active Reconnaissance

---

## Overview

**Port scanning** is the foundation of active reconnaissance — sending probes to target systems to identify open ports, determine available services, and map the network attack surface. Unlike passive reconnaissance, port scanning directly interacts with target systems and can be detected.

---

## 1. Port Scanning Fundamentals

```
TCP THREE-WAY HANDSHAKE:

NORMAL CONNECTION:
Client ──── SYN ────→ Server
Client ←── SYN/ACK ── Server
Client ──── ACK ────→ Server
                      Connection established!

PORT STATES:
├── OPEN      → Service is listening (accepting connections)
├── CLOSED    → No service listening (RST response)
├── FILTERED  → Firewall dropping/rejecting probes
└── UNFILTERED→ Accessible but can't determine open/closed

PORT RANGES:
├── 0-1023     → Well-known ports (HTTP:80, SSH:22, etc.)
├── 1024-49151 → Registered ports (MySQL:3306, etc.)
└── 49152-65535→ Dynamic/ephemeral ports
```

---

## 2. Nmap Scan Types

```bash
# === TCP CONNECT SCAN (-sT) ===
nmap -sT target.com
# Full TCP handshake — most reliable but logged
# Use when: you don't have raw packet privileges

# === SYN SCAN (-sS) — "Half-Open" / "Stealth" ===
sudo nmap -sS target.com
# Sends SYN, gets SYN/ACK, sends RST (no full connection)
# Faster and less likely to be logged
# DEFAULT scan type with root/sudo

# === UDP SCAN (-sU) ===
sudo nmap -sU target.com
# Scans UDP ports (DNS:53, SNMP:161, TFTP:69)
# Very slow — no handshake, relies on ICMP unreachable

# === FIN SCAN (-sF) ===
sudo nmap -sF target.com
# Sends FIN flag — open ports don't respond, closed send RST
# Evades some simple firewalls

# === XMAS SCAN (-sX) ===
sudo nmap -sX target.com
# Sends FIN+PSH+URG flags — like a Christmas tree
# Similar evasion as FIN scan

# === NULL SCAN (-sN) ===
sudo nmap -sN target.com
# Sends no flags — open ports don't respond
# Stealthy but unreliable on Windows (always sends RST)

# === ACK SCAN (-sA) ===
sudo nmap -sA target.com
# Determines firewall rules — filtered vs unfiltered
# Does NOT determine open/closed
```

---

## 3. Nmap Practical Commands

```bash
# === BASIC SCANNING ===
nmap target.com                    # Top 1000 ports
nmap -p 80,443,22 target.com      # Specific ports
nmap -p 1-65535 target.com        # ALL ports
nmap -p- target.com               # ALL ports (shorthand)
nmap --top-ports 100 target.com   # Top 100 ports

# === SPEED / TIMING ===
nmap -T0 target.com               # Paranoid (IDS evasion)
nmap -T1 target.com               # Sneaky
nmap -T2 target.com               # Polite
nmap -T3 target.com               # Normal (default)
nmap -T4 target.com               # Aggressive
nmap -T5 target.com               # Insane (fastest)

# === MULTIPLE TARGETS ===
nmap 192.168.1.0/24               # Entire subnet
nmap 192.168.1.1-100              # IP range
nmap -iL targets.txt              # From file

# === OUTPUT FORMATS ===
nmap -oN scan.txt target.com      # Normal text
nmap -oX scan.xml target.com      # XML
nmap -oG scan.gnmap target.com    # Grepable
nmap -oA scan target.com          # ALL formats

# === COMPREHENSIVE SCAN ===
sudo nmap -sS -sV -sC -O -p- -T4 -oA full_scan target.com
# SYN scan + version detection + scripts + OS detection + all ports
```

---

## 4. Alternative Scanning Tools

```bash
# === MASSCAN (Fastest Scanner) ===
masscan 203.0.113.0/24 -p 1-65535 --rate 10000
# Scans entire internet in ~6 minutes
# Much faster than Nmap for large ranges
# Less accurate — use Nmap for follow-up

# === RUSTSCAN (Fast + Nmap Integration) ===
rustscan -a target.com -- -sV -sC
# Quickly finds open ports, then feeds to Nmap
# Best of both worlds: speed + accuracy

# === UNICORNSCAN ===
unicornscan -mT target.com:1-65535
# Alternative fast scanner

# === ZMAP ===
zmap -p 80 203.0.113.0/24
# Designed for internet-wide scanning
```

---

## 5. Scan Comparison Table

| Scan Type | Speed | Stealth | Reliability | Root Required |
|-----------|:-----:|:-------:|:-----------:|:------------:|
| TCP Connect (-sT) | Slow | ❌ Low | ★★★★★ | No |
| SYN (-sS) | Fast | ★★★ | ★★★★★ | Yes |
| UDP (-sU) | Very slow | ★★★ | ★★★ | Yes |
| FIN (-sF) | Fast | ★★★★ | ★★★ | Yes |
| XMAS (-sX) | Fast | ★★★★ | ★★★ | Yes |
| NULL (-sN) | Fast | ★★★★ | ★★ | Yes |
| ACK (-sA) | Fast | ★★★ | ★★★ | Yes |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SYN scan** | Default, fast, stealthy (half-open) |
| **TCP connect** | Full handshake, no root needed |
| **UDP scan** | Slow but finds DNS, SNMP, TFTP |
| **Nmap** | Most versatile port scanner |
| **Masscan** | Fastest scanner for large ranges |
| **RustScan** | Fast discovery + Nmap accuracy |

---

## Quick Revision Questions

1. **What is the difference between SYN and TCP connect scans?**
2. **Why is UDP scanning slower than TCP scanning?**
3. **What does a "filtered" port state indicate?**
4. **When would you use Masscan over Nmap?**
5. **What timing template is best for IDS evasion?**

---

[Next: Service Detection →](02-service-detection.md)

---

*Unit 8 - Topic 1 of 5 | [Back to README](../README.md)*
