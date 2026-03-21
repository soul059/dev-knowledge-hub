# Port Scanning

## Unit 6 - Topic 2 | Scanning and Enumeration

---

## Overview

**Port scanning** determines which TCP/UDP ports are open on a target. Each open port represents a potential **attack vector**. Understanding different scan techniques and their trade-offs is essential for thorough yet efficient testing.

---

## 1. Important Port Ranges

| Range | Name | Example Ports |
|-------|------|:---:|
| 0-1023 | Well-known | SSH(22), HTTP(80), HTTPS(443) |
| 1024-49151 | Registered | MySQL(3306), RDP(3389), Tomcat(8080) |
| 49152-65535 | Dynamic/Ephemeral | Client-side connections |

### Critical Ports for Pen Testers

| Port | Service | Why It Matters |
|:----:|---------|---------------|
| 21 | FTP | Anonymous access, credential theft |
| 22 | SSH | Brute force, key-based bypass |
| 23 | Telnet | Cleartext credentials |
| 25 | SMTP | Email relay, user enumeration |
| 53 | DNS | Zone transfer, cache poisoning |
| 80/443 | HTTP/S | Web application attacks |
| 110/143 | POP3/IMAP | Email access |
| 135 | MSRPC | Windows RPC attacks |
| 139/445 | SMB | EternalBlue, file share access |
| 389/636 | LDAP | Active Directory enumeration |
| 1433 | MSSQL | Database attacks |
| 3306 | MySQL | Database attacks |
| 3389 | RDP | Brute force, BlueKeep |
| 5432 | PostgreSQL | Database attacks |
| 5985/5986 | WinRM | Remote management |
| 8080/8443 | Alt HTTP/S | Web app, admin panels |

---

## 2. TCP Scan Techniques

```bash
# === SYN SCAN (Half-open / Stealth) ===
sudo nmap -sS 10.0.0.1                # Default (requires root)
# SYN → SYN/ACK = Open
# SYN → RST = Closed
# No response = Filtered

# === TCP CONNECT SCAN ===
nmap -sT 10.0.0.1                     # Full TCP handshake (no root needed)
# Complete 3-way handshake — gets logged

# === FIN SCAN ===
sudo nmap -sF 10.0.0.1                # Only FIN flag
# Used to bypass simple firewalls

# === XMAS SCAN ===
sudo nmap -sX 10.0.0.1                # FIN + PSH + URG flags
# Named "Xmas" because all flags lit up like a tree

# === NULL SCAN ===
sudo nmap -sN 10.0.0.1                # No flags set
# Unusual packets may bypass some firewalls

# === ACK SCAN ===
sudo nmap -sA 10.0.0.1                # ACK flag
# Not for finding open ports — maps firewall rules
# Unfiltered = no firewall; Filtered = firewall present
```

---

## 3. UDP Scanning

```bash
# === UDP SCAN ===
sudo nmap -sU 10.0.0.1                # UDP scan
sudo nmap -sU --top-ports 20 10.0.0.1 # Top 20 UDP ports
sudo nmap -sU -p 53,161,500 10.0.0.1  # Specific UDP ports

# UDP Scanning is SLOW because:
# • Open ports may not respond at all
# • Must wait for ICMP Unreachable (closed)
# • Timeout for each non-responding port
# • Rate limiting by targets

# IMPORTANT UDP PORTS:
# 53  — DNS
# 67  — DHCP
# 69  — TFTP
# 123 — NTP
# 161 — SNMP (community strings!)
# 500 — IKE/IPSec VPN
# 514 — Syslog
```

---

## 4. Scanning Strategies

```bash
# STRATEGY 1: Quick & Broad
nmap -sS --top-ports 100 -T4 10.0.0.0/24

# STRATEGY 2: Targeted & Deep
nmap -sS -sV -p- -T3 10.0.0.5

# STRATEGY 3: Combined TCP + UDP
nmap -sS -sU --top-ports 20 -T4 10.0.0.5

# STRATEGY 4: Stealthy
nmap -sS -T1 -f --source-port 53 10.0.0.5

# PORT SPECIFICATION:
nmap -p 80               # Single port
nmap -p 80,443,8080      # Multiple ports
nmap -p 1-1000           # Range
nmap -p-                 # All 65535 ports
nmap --top-ports 1000    # Top 1000 common ports
nmap -F                  # Fast (top 100)
```

---

## 5. Output and Documentation

```bash
# === OUTPUT FORMATS ===
nmap -oN scan.txt 10.0.0.1            # Normal text
nmap -oX scan.xml 10.0.0.1            # XML (for tools)
nmap -oG scan.gnmap 10.0.0.1          # Grepable
nmap -oA scan 10.0.0.1                # All three formats

# === PARSE RESULTS ===
# Extract open ports from grepable output
grep "open" scan.gnmap | cut -d' ' -f2
cat scan.gnmap | grep "/open" | awk '{print $2}'

# Convert XML to HTML report
xsltproc scan.xml -o scan.html
```

---

## Summary Table

| Scan Type | Flag | Root | Stealth | Speed |
|-----------|:----:|:----:|:-------:|:-----:|
| SYN | -sS | ✅ | ★★★★ | ★★★★ |
| Connect | -sT | ❌ | ★ | ★★★ |
| UDP | -sU | ✅ | ★★ | ★ |
| FIN | -sF | ✅ | ★★★★ | ★★★ |
| XMAS | -sX | ✅ | ★★★★ | ★★★ |
| ACK | -sA | ✅ | ★★★ | ★★★ |

---

## Quick Revision Questions

1. **Why is SYN scanning considered "stealth"?**
2. **Why is UDP scanning slow?**
3. **What ports should pen testers always check?**
4. **How does an ACK scan differ from other scan types?**
5. **What output format is best for automated processing?**

---

[← Previous: Network Scanning](01-network-scanning.md) | [Next: Service Enumeration →](03-service-enumeration.md)

---

*Unit 6 - Topic 2 of 6 | [Back to README](../README.md)*
