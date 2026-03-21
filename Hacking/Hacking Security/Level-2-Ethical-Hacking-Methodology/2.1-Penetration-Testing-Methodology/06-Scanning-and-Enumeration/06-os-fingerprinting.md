# OS Fingerprinting

## Unit 6 - Topic 6 | Scanning and Enumeration

---

## Overview

**OS fingerprinting** determines the operating system running on a target by analyzing how it responds to network probes. Knowing the OS helps select **appropriate exploits**, **tools**, and **techniques** for the target platform.

---

## 1. Active vs Passive Fingerprinting

```
┌──────────────────────────────────────────────────────────────┐
│              OS FINGERPRINTING METHODS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ACTIVE FINGERPRINTING:                                      │
│  ├── Send specially crafted packets                          │
│  ├── Analyze responses (TTL, window size, flags)            │
│  ├── Tool: nmap -O                                           │
│  ├── More accurate                                           │
│  └── Detectable by target                                    │
│                                                              │
│  PASSIVE FINGERPRINTING:                                     │
│  ├── Capture existing network traffic                        │
│  ├── Analyze TCP/IP characteristics                          │
│  ├── Tool: p0f                                               │
│  ├── Less accurate                                           │
│  └── Completely undetectable                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Active OS Fingerprinting

```bash
# === NMAP OS DETECTION ===
sudo nmap -O 10.0.0.5                  # Basic OS detection
sudo nmap -O --osscan-guess 10.0.0.5   # Aggressive guessing
sudo nmap -A 10.0.0.5                  # OS + version + scripts + traceroute

# NMAP TECHNIQUES:
# 1. TCP ISN (Initial Sequence Number) analysis
# 2. TCP options analysis
# 3. IP ID sampling
# 4. TCP window size analysis
# 5. ICMP response analysis
# 6. TCP/UDP timestamp analysis

# EXAMPLE OUTPUT:
# OS details: Linux 5.4 - 5.15 (Ubuntu)
# OS CPE: cpe:/o:linux:linux_kernel:5.4
# Aggressive OS guesses: Ubuntu 22.04 (96%)

# === REQUIREMENTS ===
# Needs at least 1 open and 1 closed port for best results
# Root/admin privileges required
# TCP/IP stack must be reachable (not through proxy)
```

---

## 3. Passive OS Fingerprinting

```bash
# === P0F (Passive OS Fingerprinter) ===
sudo p0f -i eth0                       # Monitor interface
sudo p0f -r capture.pcap              # Analyze pcap file

# P0F analyzes:
# • TCP SYN packet characteristics
# • Window size, TTL, options
# • No packets sent to target!

# === INDICATORS ===
# TTL Values (initial):
# Linux:   64
# Windows: 128
# Cisco:   255
# Solaris: 255

# TCP Window Size (varies):
# Linux:   5840, 14600, 29200
# Windows: 8192, 65535
```

---

## 4. OS Identification Clues

| Indicator | Linux | Windows | macOS |
|-----------|:-----:|:-------:|:-----:|
| **TTL** | 64 | 128 | 64 |
| **SSH** | OpenSSH | ❌ (usually) | OpenSSH |
| **RDP (3389)** | ❌ (usually) | ✅ | ❌ |
| **SMB (445)** | Samba | Native SMB | Native SMB |
| **Web server** | nginx/Apache | IIS | Apache |
| **Port 135** | ❌ | ✅ (MSRPC) | ❌ |
| **Port 5985** | ❌ | ✅ (WinRM) | ❌ |

```bash
# QUICK OS IDENTIFICATION:

# See port 135, 445, 3389 open?
# → Almost certainly Windows

# See port 22 with OpenSSH?
# → Likely Linux/macOS

# See IIS web server?
# → Definitely Windows

# TTL ~128 in ping response?
# → Windows

# TTL ~64 in ping response?
# → Linux or macOS
```

---

## 5. Evading OS Fingerprinting

```bash
# DEFENSE: Make OS harder to fingerprint

# Linux: Change default TTL
sudo sysctl -w net.ipv4.ip_default_ttl=128  # Mimic Windows

# Firewall: Block fingerprint probes
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

# Use IDS/IPS to detect OS fingerprint scans
# Suricata rule example:
# alert tcp any any -> $HOME_NET any (msg:"OS Fingerprint attempt"; 
#   flags:S; threshold:type both,track by_src,count 6,seconds 60; sid:1;)

# PEN TESTER NOTE:
# If OS fingerprinting fails:
# 1. Try banner grabbing (nmap -sV)
# 2. Check service-specific clues
# 3. Look at web server type (IIS = Windows)
# 4. Check open ports pattern
# 5. Use p0f for passive detection
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Active** | nmap -O, sends probes, detectable |
| **Passive** | p0f, analyzes existing traffic, undetectable |
| **TTL** | 64 = Linux/macOS, 128 = Windows, 255 = Cisco |
| **Port clues** | 135/445/3389 = Windows, 22 = Linux |
| **Service clues** | IIS = Windows, nginx = usually Linux |
| **Accuracy** | Active > Passive, but active is detectable |

---

## Quick Revision Questions

1. **What is the difference between active and passive OS fingerprinting?**
2. **What nmap flag performs OS detection?**
3. **What TTL value suggests a Windows system?**
4. **Which open ports strongly suggest Windows?**
5. **How can a defender make OS fingerprinting harder?**

---

[← Previous: Banner Grabbing](05-banner-grabbing.md)

---

*Unit 6 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Exploitation Phase →](../07-Exploitation-Phase/01-exploitation-planning.md)*
