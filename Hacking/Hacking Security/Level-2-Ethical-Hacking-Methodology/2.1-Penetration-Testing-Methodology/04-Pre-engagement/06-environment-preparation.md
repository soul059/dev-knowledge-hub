# Environment Preparation

## Unit 4 - Topic 6 | Pre-engagement

---

## Overview

**Environment preparation** ensures your testing platform, tools, and infrastructure are ready before the engagement begins. Proper preparation prevents wasted billable hours troubleshooting tool issues and ensures you can start testing efficiently on day one.

---

## 1. Testing Environment Checklist

```
┌──────────────────────────────────────────────────────────────┐
│              PRE-ENGAGEMENT PREPARATION                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  HARDWARE:                                                   │
│  ☐ Primary testing laptop (Kali/Parrot)                     │
│  ☐ Backup laptop                                            │
│  ☐ External wireless adapter (monitor mode)                 │
│  ☐ USB drives (encrypted)                                    │
│  ☐ Ethernet adapter/cables                                   │
│  ☐ Physical testing tools (if applicable)                   │
│                                                              │
│  SOFTWARE:                                                   │
│  ☐ OS updated (Kali, Parrot, or custom)                     │
│  ☐ Tools updated (apt update && apt upgrade)                │
│  ☐ VPN client configured                                     │
│  ☐ Burp Suite licensed and configured                        │
│  ☐ Reporting templates ready                                 │
│  ☐ Note-taking tool configured                               │
│                                                              │
│  NETWORK:                                                    │
│  ☐ VPN access tested and working                             │
│  ☐ Stable internet connection                                │
│  ☐ Backup internet connection                                │
│  ☐ Test connectivity to target network                      │
│                                                              │
│  DOCUMENTATION:                                              │
│  ☐ Signed authorization                                      │
│  ☐ Rules of engagement                                       │
│  ☐ Emergency contacts                                        │
│  ☐ Scope document                                            │
│  ☐ Previous test reports (if available)                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Tool Preparation

```bash
# === UPDATE EVERYTHING ===
sudo apt update && sudo apt upgrade -y

# === VERIFY KEY TOOLS ===
nmap --version                        # Port scanner
burpsuite                             # Web proxy
msfconsole --version                  # Metasploit
python3 --version                     # Python
gobuster version                      # Web fuzzer
john --list=formats | head -5         # Password cracker
hashcat --version                     # GPU cracker
sqlmap --version                      # SQL injection

# === INSTALL MISSING TOOLS ===
sudo apt install -y \
    nmap nikto gobuster dirb \
    sqlmap john hashcat \
    enum4linux smbclient \
    crackmapexec bloodhound \
    responder impacket-scripts

# === WORDLISTS ===
ls /usr/share/wordlists/
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
# Download SecLists if not present
# git clone https://github.com/danielmiessler/SecLists.git

# === VERIFY VPN CONNECTION ===
sudo openvpn --config client.ovpn &
ip addr show tun0                     # Confirm VPN IP
ping target_ip                        # Confirm connectivity
```

---

## 3. Note-Taking Setup

| Tool | Best For |
|------|----------|
| **CherryTree** | Hierarchical notes with screenshots |
| **Obsidian** | Markdown-based, linked notes |
| **Notion** | Collaborative note-taking |
| **KeepNote** | Legacy, simple structure |
| **Joplin** | Open-source, encrypted |
| **OneNote** | Microsoft ecosystem |

```
NOTE STRUCTURE:

📁 Project Name
├── 📁 Pre-engagement
│   ├── Scope document
│   ├── Authorization
│   └── Contacts
├── 📁 Reconnaissance
│   ├── OSINT findings
│   ├── DNS records
│   └── Technology stack
├── 📁 Scanning
│   ├── Nmap results
│   ├── Vulnerability scans
│   └── Web enumeration
├── 📁 Exploitation
│   ├── Successful exploits (with screenshots)
│   ├── Failed attempts
│   └── Credentials found
├── 📁 Post-Exploitation
│   ├── Privilege escalation
│   ├── Lateral movement
│   └── Data access
├── 📁 Evidence
│   ├── Screenshots
│   ├── Tool output
│   └── Network captures
└── 📁 Report
    ├── Draft
    └── Final
```

---

## 4. VM Snapshots

```bash
# BEST PRACTICE: Use VM snapshots

BEFORE ENGAGEMENT:
├── Clean Kali VM (fully updated)
├── Take snapshot: "Clean_Pre-Engagement"
├── Configure tools for this specific engagement
└── Take snapshot: "Configured_[ProjectName]"

DURING ENGAGEMENT:
├── Take daily snapshots
├── Name: "Day1_Recon", "Day2_Scanning", etc.
└── Allows rollback if VM is compromised

AFTER ENGAGEMENT:
├── Export evidence from VM
├── Securely wipe engagement data
├── Revert to clean snapshot
└── Ready for next engagement
```

---

## 5. Client-Side Preparation

```
WHAT TO REQUEST FROM CLIENT:

FOR EXTERNAL TEST:
├── List of external IP ranges/domains
├── Any known WAF/IPS information
└── Testing window confirmation

FOR INTERNAL TEST:
├── VPN credentials and configuration
├── Physical access (if on-site)
├── Network drop location
├── DHCP or static IP for testing machine
└── User credentials (for gray/white box)

FOR WEB APP TEST:
├── Application URL(s)
├── User accounts (different roles)
├── API documentation
├── Technology stack information
└── Known issues/previous findings

FOR ALL TESTS:
├── Emergency contact confirmation
├── Signed authorization (final copy)
├── Verified scope document
└── Communication channel setup
```

---

## Summary Table

| Area | Key Preparation |
|------|----------------|
| **Hardware** | Laptop, adapters, encrypted storage |
| **Software** | Updated OS, tools, VPN client |
| **Network** | VPN tested, connectivity verified |
| **Notes** | Structured note-taking system ready |
| **VMs** | Clean snapshots before engagement |
| **Client** | Access, credentials, documents received |

---

## Quick Revision Questions

1. **Why is environment preparation important?**
2. **What tools should be verified before an engagement?**
3. **How should notes be organized during a pen test?**
4. **Why should you take VM snapshots?**
5. **What should you request from the client before testing?**

---

[← Previous: Communication Protocols](05-communication-protocols.md)

---

*Unit 4 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Reconnaissance Phase →](../05-Reconnaissance-Phase/01-passive-reconnaissance.md)*
