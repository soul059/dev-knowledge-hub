# Nmap Fundamentals

## Unit 1 - Topic 3 | Network Scanning

---

## Overview

**Nmap (Network Mapper)** is the most essential tool in a penetration tester's arsenal. It performs host discovery, port scanning, service detection, OS fingerprinting, and vulnerability detection — all in one versatile tool.

---

## 1. Nmap Command Structure

```bash
nmap [SCAN TYPE] [OPTIONS] [TARGET]

# EXAMPLES:
nmap target.com                    # Basic scan (top 1000 ports)
nmap -sS -sV -O target.com        # SYN + Version + OS detection
nmap -A target.com                 # Aggressive (OS + Version + Scripts + Traceroute)
nmap -p- -T4 target.com           # All ports, aggressive timing
```

---

## 2. Essential Nmap Options

```bash
# ══════════════════════════════════════
# TARGET SPECIFICATION
# ══════════════════════════════════════
nmap 192.168.1.1                  # Single host
nmap 192.168.1.1-100              # IP range
nmap 192.168.1.0/24               # CIDR notation
nmap -iL targets.txt              # From file
nmap --exclude 192.168.1.1        # Exclude host
nmap --excludefile exclude.txt    # Exclude from file

# ══════════════════════════════════════
# DISCOVERY OPTIONS
# ══════════════════════════════════════
nmap -sn target.com               # Ping scan only (no port scan)
nmap -Pn target.com               # Skip discovery (assume alive)
nmap -PS80,443 target.com         # TCP SYN discovery on ports
nmap -PA80 target.com             # TCP ACK discovery
nmap -PE target.com               # ICMP echo discovery

# ══════════════════════════════════════
# SCAN TYPES
# ══════════════════════════════════════
nmap -sS target.com               # SYN scan (default with root)
nmap -sT target.com               # TCP Connect scan
nmap -sU target.com               # UDP scan
nmap -sA target.com               # ACK scan (firewall mapping)
nmap -sV target.com               # Version detection
nmap -sC target.com               # Default scripts
nmap -O target.com                # OS detection

# ══════════════════════════════════════
# PORT SPECIFICATION
# ══════════════════════════════════════
nmap -p 80 target.com             # Single port
nmap -p 80,443,8080 target.com    # Multiple ports
nmap -p 1-1024 target.com         # Range
nmap -p- target.com               # All 65535 ports
nmap --top-ports 100 target.com   # Top 100 ports
nmap -F target.com                # Fast (top 100 ports)
```

---

## 3. Advanced Nmap Features

```bash
# ══════════════════════════════════════
# SERVICE & OS DETECTION
# ══════════════════════════════════════
nmap -sV target.com                       # Service version detection
nmap -sV --version-intensity 9 target.com # Maximum version detection
nmap -O target.com                        # OS detection
nmap -O --osscan-guess target.com         # Aggressive OS guessing
nmap -A target.com                        # Aggressive: OS + Version + Scripts + Traceroute

# ══════════════════════════════════════
# NSE (Nmap Scripting Engine)
# ══════════════════════════════════════
nmap -sC target.com                       # Default safe scripts
nmap --script=vuln target.com             # Vulnerability scripts
nmap --script=http-enum target.com        # Web enumeration
nmap --script=smb-vuln* target.com        # SMB vulnerability scripts
nmap --script=banner target.com           # Banner grabbing
nmap --script "default and safe" target.com  # Combined categories

# ══════════════════════════════════════
# EVASION & SPOOFING
# ══════════════════════════════════════
nmap -f target.com                        # Fragment packets
nmap -D 192.168.1.10,ME target.com        # Decoy scan
nmap --source-port 53 target.com          # Source port spoofing
nmap --data-length 50 target.com          # Append random data
nmap -S 192.168.1.100 target.com          # Spoof source IP (no results back)
nmap --proxies socks4://proxy:1080 target # Through proxy
```

---

## 4. Common Nmap Recipes

```bash
# === QUICK INITIAL SCAN ===
nmap -sS -sV --top-ports 100 -T4 target.com

# === COMPREHENSIVE SCAN ===
sudo nmap -sS -sV -sC -O -p- -T4 -oA full_scan target.com

# === STEALTH SCAN ===
sudo nmap -sS -T2 -f -D RND:5 target.com

# === WEB SERVER SCAN ===
nmap -sV -p 80,443,8080,8443 --script http-enum,http-title,http-headers target.com

# === SMB SCAN ===
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-vuln* target.com

# === FULL NETWORK DISCOVERY ===
sudo nmap -sn 10.0.0.0/24 -oG - | grep "Up" | cut -d" " -f2 > alive.txt
sudo nmap -sS -sV -sC -p- -T4 -iL alive.txt -oA network_scan

# === VULNERABILITY SCAN ===
nmap --script vuln -p- target.com

# === FIREWALL DETECTION ===
sudo nmap -sA -p 80,443 target.com
```

---

## 5. Nmap Cheat Sheet

| Flag | Purpose | Example |
|------|---------|---------|
| `-sS` | SYN scan | `nmap -sS target` |
| `-sT` | TCP Connect | `nmap -sT target` |
| `-sU` | UDP scan | `nmap -sU target` |
| `-sV` | Version detection | `nmap -sV target` |
| `-sC` | Default scripts | `nmap -sC target` |
| `-O` | OS detection | `nmap -O target` |
| `-A` | Aggressive | `nmap -A target` |
| `-p-` | All ports | `nmap -p- target` |
| `-Pn` | Skip ping | `nmap -Pn target` |
| `-T4` | Aggressive timing | `nmap -T4 target` |
| `-oA` | All output formats | `nmap -oA scan target` |
| `-f` | Fragment packets | `nmap -f target` |
| `-D` | Decoy scan | `nmap -D RND:5 target` |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Basic syntax** | `nmap [type] [options] [target]` |
| **-sS** | Default SYN scan (needs root) |
| **-sV** | Detect service versions |
| **-sC** | Run default NSE scripts |
| **-A** | Aggressive (OS + version + scripts) |
| **-oA** | Save in all output formats |

---

## Quick Revision Questions

1. **What is the default scan type when running Nmap as root?**
2. **What does the `-A` flag combine?**
3. **How do you scan all 65535 ports?**
4. **What Nmap flag runs vulnerability detection scripts?**
5. **How do you save scan results in all formats?**

---

[← Previous: Port Scanning](02-port-scanning.md) | [Next: Scan Timing and Evasion →](04-scan-timing-and-evasion.md)

---

*Unit 1 - Topic 3 of 6 | [Back to README](../README.md)*
