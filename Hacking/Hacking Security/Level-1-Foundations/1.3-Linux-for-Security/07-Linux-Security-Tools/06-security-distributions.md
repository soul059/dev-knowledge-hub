# Security Distributions

## Unit 7 - Topic 6 | Linux Security Tools

---

## Overview

**Security-focused Linux distributions** come pre-loaded with hundreds of security tools, making them essential for penetration testing, forensics, and security research. Understanding the strengths of each distribution helps professionals choose the right tool for the job.

---

## 1. Major Security Distributions

| Distribution | Focus | Base | Tools |
|-------------|-------|------|:---:|
| **Kali Linux** | Penetration testing | Debian | 600+ |
| **Parrot OS** | Pen testing + privacy | Debian | 600+ |
| **BlackArch** | Pen testing (advanced) | Arch Linux | 2800+ |
| **REMnux** | Malware analysis | Ubuntu | 200+ |
| **CAINE** | Digital forensics | Ubuntu | 100+ |
| **Tails** | Anonymity / privacy | Debian | Tor-based |
| **Whonix** | Anonymity (VM-based) | Debian | Tor-based |
| **Security Onion** | Network security monitoring | Ubuntu | IDS/SIEM |

---

## 2. Kali Linux (Most Popular)

```bash
# === TOOL CATEGORIES IN KALI ===
# 01 - Information Gathering
#   nmap, recon-ng, maltego, theharvester, amass
# 02 - Vulnerability Analysis
#   nikto, openvas, nessus, wpscan
# 03 - Web Application Analysis
#   burpsuite, sqlmap, dirb, gobuster, zaproxy
# 04 - Password Attacks
#   john, hashcat, hydra, medusa, cewl
# 05 - Wireless Attacks
#   aircrack-ng, wifite, kismet, fern
# 06 - Exploitation Tools
#   metasploit, searchsploit, beef-xss
# 07 - Sniffing & Spoofing
#   wireshark, tcpdump, bettercap, responder
# 08 - Post Exploitation
#   mimikatz, empire, bloodhound, linpeas
# 09 - Forensics
#   autopsy, binwalk, foremost, volatility
# 10 - Reporting
#   dradis, pipal, cutycapt

# === KALI SETUP ===
sudo apt update && sudo apt upgrade -y
sudo apt install kali-linux-default     # Default toolset
sudo apt install kali-linux-large       # Extended toolset
sudo apt install kali-tools-top10       # Top 10 tools only

# === COMMON USAGE ===
# Live USB boot (no installation)
# VM (VirtualBox, VMware)
# WSL2 (Windows Subsystem for Linux)
# Docker container
# Cloud instance (AWS, Azure)
```

---

## 3. Comparison: Kali vs Parrot vs BlackArch

| Feature | Kali | Parrot | BlackArch |
|---------|:----:|:------:|:---------:|
| **Tools** | 600+ | 600+ | 2800+ |
| **Base** | Debian | Debian | Arch |
| **Desktop** | XFCE | MATE | Multiple |
| **RAM** | 2GB+ | 1GB+ | 1GB+ |
| **Beginner friendly** | ✅ | ✅ | ❌ |
| **Privacy tools** | Some | ✅ Built-in | Some |
| **AnonSurf** | ❌ | ✅ | ❌ |
| **Rolling release** | ✅ | ✅ | ✅ |
| **Community** | Largest | Growing | Moderate |

---

## 4. Specialized Distributions

### REMnux (Malware Analysis)

```bash
# Pre-installed tools:
# • Ghidra, Radare2 — Reverse engineering
# • YARA — Malware pattern matching
# • PEframe, pefile — PE analysis
# • olevba — Office macro analysis
# • Wireshark, NetworkMiner — Traffic analysis
# • CyberChef — Data transformation
```

### Security Onion (Network Monitoring)

```bash
# Includes:
# • Suricata — IDS/IPS
# • Zeek (Bro) — Network analysis
# • Elasticsearch — Log storage
# • Kibana — Visualization
# • TheHive — Incident response
# • CyberChef — Data analysis
# Full SIEM/SOC platform in a single distro
```

### CAINE (Computer Aided INvestigative Environment)

```bash
# Forensic-focused:
# • Autopsy — File system analysis
# • Guymager — Disk imaging
# • Foremost, Scalpel — File carving
# • RegRipper — Windows registry analysis
# • Write-blockers built-in
# Designed to never modify evidence
```

---

## 5. Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│              SECURITY DISTRO BEST PRACTICES                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ✅ Run in VM (isolated from host)                           │
│  ✅ Use snapshots (revert after testing)                     │
│  ✅ Keep updated (sudo apt update && upgrade)                │
│  ✅ Use ONLY with authorization                              │
│  ✅ Document everything (screenshots, logs)                  │
│  ❌ Never run as daily driver OS                             │
│  ❌ Never use on production networks without permission      │
│  ❌ Never store sensitive data without encryption            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Distribution | Use Case |
|-------------|----------|
| **Kali Linux** | General penetration testing |
| **Parrot OS** | Pen testing + privacy |
| **BlackArch** | Advanced users, most tools |
| **REMnux** | Malware analysis |
| **Security Onion** | Network monitoring / SOC |
| **CAINE** | Digital forensics |
| **Tails** | Anonymous browsing |

---

## Quick Revision Questions

1. **What is Kali Linux based on?**
2. **How does Parrot OS differ from Kali?**
3. **Which distro is best for malware analysis?**
4. **What does Security Onion provide?**
5. **Why should security distros be run in a VM?**
6. **Which distro has the most security tools?**

---

[← Previous: Forensic Tools](05-forensic-tools.md)

---

*Unit 7 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Bash Scripting for Security →](../08-Bash-Scripting-for-Security/01-bash-basics-review.md)*
