# Network Scanning with Nmap

## Unit 7 - Topic 1 | Linux Security Tools

---

## Overview

**Nmap (Network Mapper)** is the most widely used **network scanner** and **port scanner** in cybersecurity. It discovers hosts, services, operating systems, and vulnerabilities on a network. Every penetration tester and security professional must master Nmap.

---

## 1. Nmap Scan Types

| Scan Type | Flag | Description | Root Required |
|-----------|------|-------------|:---:|
| **TCP Connect** | `-sT` | Full TCP handshake | ❌ |
| **SYN Scan** | `-sS` | Half-open (stealth) scan | ✅ |
| **UDP Scan** | `-sU` | UDP port scan | ✅ |
| **ACK Scan** | `-sA` | Detect firewall rules | ✅ |
| **FIN Scan** | `-sF` | Stealth (FIN flag only) | ✅ |
| **XMAS Scan** | `-sX` | FIN+PSH+URG flags | ✅ |
| **NULL Scan** | `-sN` | No flags set | ✅ |
| **Ping Scan** | `-sn` | Host discovery only | ❌ |
| **Version Scan** | `-sV` | Detect service versions | ❌ |
| **OS Detection** | `-O` | Operating system fingerprint | ✅ |

---

## 2. Scan Techniques Explained

```
TCP Connect Scan (-sT):
  Client ──── SYN ────►  Target
  Client ◄── SYN/ACK ──  Target    Port OPEN
  Client ──── ACK ────►  Target
  Client ──── RST ────►  Target    Connection logged!

SYN Scan (-sS) — "Stealth":
  Client ──── SYN ────►  Target
  Client ◄── SYN/ACK ──  Target    Port OPEN
  Client ──── RST ────►  Target    Never completes = no log!

  Client ──── SYN ────►  Target
  Client ◄── RST ──────  Target    Port CLOSED

UDP Scan (-sU):
  Client ──── UDP ────►  Target
  Client ◄── Response ──  Target    Port OPEN
  Client ◄── ICMP Unreachable ──    Port CLOSED
  (No response)                      Port OPEN|FILTERED
```

---

## 3. Essential Nmap Commands

```bash
# === HOST DISCOVERY ===
nmap -sn 10.0.0.0/24                # Ping sweep (find live hosts)
nmap -sn -PE 10.0.0.0/24            # ICMP echo ping
nmap -sn -PA 10.0.0.0/24            # TCP ACK ping
nmap -Pn 10.0.0.1                   # Skip ping, assume host is up

# === PORT SCANNING ===
nmap 10.0.0.1                       # Top 1000 ports (default)
nmap -p 80,443 10.0.0.1             # Specific ports
nmap -p 1-1000 10.0.0.1             # Port range
nmap -p- 10.0.0.1                   # ALL 65535 ports
nmap -F 10.0.0.1                    # Fast (top 100 ports)
nmap --top-ports 20 10.0.0.1        # Top 20 ports

# === SERVICE/VERSION DETECTION ===
nmap -sV 10.0.0.1                   # Service version detection
nmap -sV --version-intensity 5 10.0.0.1  # More aggressive version detection
nmap -O 10.0.0.1                    # OS detection
nmap -A 10.0.0.1                    # Aggressive: OS + version + scripts + traceroute

# === TIMING & EVASION ===
nmap -T0 10.0.0.1                   # Paranoid (IDS evasion)
nmap -T1 10.0.0.1                   # Sneaky
nmap -T2 10.0.0.1                   # Polite
nmap -T3 10.0.0.1                   # Normal (default)
nmap -T4 10.0.0.1                   # Aggressive
nmap -T5 10.0.0.1                   # Insane (fastest)

# === OUTPUT ===
nmap -oN scan.txt 10.0.0.1          # Normal output
nmap -oX scan.xml 10.0.0.1          # XML output
nmap -oG scan.gnmap 10.0.0.1        # Grepable output
nmap -oA scan 10.0.0.1              # All formats
```

---

## 4. Nmap Scripting Engine (NSE)

```bash
# NSE scripts are in /usr/share/nmap/scripts/
ls /usr/share/nmap/scripts/ | wc -l  # ~600+ scripts

# === VULNERABILITY SCANNING ===
nmap --script vuln 10.0.0.1
nmap --script vulners -sV 10.0.0.1

# === SPECIFIC SCRIPTS ===
nmap --script http-title 10.0.0.1            # Web page titles
nmap --script http-enum 10.0.0.1             # Web directory enumeration
nmap --script ssh-brute 10.0.0.1             # SSH brute force
nmap --script smb-vuln-ms17-010 10.0.0.1     # EternalBlue check
nmap --script ssl-heartbleed 10.0.0.1        # Heartbleed check
nmap --script dns-brute example.com          # DNS subdomain brute force

# === SCRIPT CATEGORIES ===
nmap --script "default" 10.0.0.1             # Default scripts (same as -sC)
nmap --script "safe" 10.0.0.1                # Non-intrusive scripts
nmap --script "discovery" 10.0.0.1           # Discovery scripts
nmap --script "exploit" 10.0.0.1             # Exploitation scripts ⚠️
```

---

## 5. Firewall Evasion Techniques

```bash
# Fragment packets
nmap -f 10.0.0.1
nmap --mtu 16 10.0.0.1               # Custom fragment size

# Decoy scan
nmap -D 10.0.0.2,10.0.0.3,ME 10.0.0.1
nmap -D RND:10 10.0.0.1              # 10 random decoys

# Source port spoofing
nmap --source-port 53 10.0.0.1       # Use DNS source port

# Idle/Zombie scan
nmap -sI zombie_host 10.0.0.1        # Completely anonymous!

# MAC spoofing
nmap --spoof-mac 00:11:22:33:44:55 10.0.0.1
nmap --spoof-mac Apple 10.0.0.1      # Random Apple MAC
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SYN scan** | Default, stealthy, requires root |
| **-sV** | Service version detection |
| **-O** | OS fingerprinting |
| **-A** | Aggressive scan (everything) |
| **NSE** | 600+ scripts for vulnerability detection |
| **-oA** | Save results in all formats |
| **-T4** | Good balance of speed and reliability |

---

## Quick Revision Questions

1. **What is the difference between -sT and -sS scans?**
2. **How do you scan all 65535 ports?**
3. **What does `nmap -A` do?**
4. **How do you use Nmap to check for a specific vulnerability?**
5. **What is a decoy scan and why is it used?**
6. **Name 3 NSE script categories.**

---

[Next: Vulnerability Scanning →](02-vulnerability-scanning.md)

---

*Unit 7 - Topic 1 of 6 | [Back to README](../README.md)*
