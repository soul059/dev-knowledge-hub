# Unit 3: CEH — Topic 2: Reconnaissance Domains

## Overview

**Reconnaissance** is the heaviest domain on the CEH exam (21%). It encompasses passive and active information gathering techniques, footprinting, scanning, and enumeration — the foundational phases of any penetration test.

---

## 1. Passive Reconnaissance (Footprinting)

```
PASSIVE RECONNAISSANCE TECHNIQUES:

OSINT (Open Source Intelligence):
  → Search engines: Google, Bing, DuckDuckGo
  → Social media: LinkedIn, Twitter, Facebook
  → Job postings: Technologies used
  → Financial records: SEC filings
  → DNS records: Whois, DNS lookup
  → Archive.org: Historical data
  → Pastebin/GitHub: Leaked data

GOOGLE DORKING (CEH FAVORITE):
  → site:target.com          Limit to domain
  → filetype:pdf             Find file types
  → intitle:"index of"       Directory listings
  → inurl:admin              Admin pages
  → "password" filetype:xls  Password files
  → cache:target.com         Cached versions
  → link:target.com          Backlinks

DNS FOOTPRINTING:
  → Whois lookup: Registration info
  → DNS zone transfer: All records
  → Subdomain enumeration
  → MX records: Email servers
  → NS records: Name servers
  → A/AAAA records: IP addresses
  → TXT records: SPF, DKIM

TOOLS:
  → theHarvester: Emails, subdomains
    theHarvester -d target.com -b google
  → Maltego: Visual relationship mapping
  → Recon-ng: Modular recon framework
  → FOCA: Metadata extraction
  → Shodan: Internet-facing devices
    shodan search "apache" country:"US"
  → Censys: Certificate/host search
```

---

## 2. Active Reconnaissance (Scanning)

```
NETWORK SCANNING TECHNIQUES:

PORT SCANNING (Nmap):
  Scan Type        | Flag    | Description
  TCP Connect      | -sT     | Full TCP handshake
  SYN Scan         | -sS     | Half-open (stealth)
  UDP Scan         | -sU     | UDP ports
  FIN Scan         | -sF     | FIN flag only
  XMAS Scan        | -sX     | FIN+PSH+URG flags
  NULL Scan        | -sN     | No flags
  ACK Scan         | -sA     | Firewall detection
  Idle/Zombie      | -sI     | IP ID spoofing

NMAP COMMANDS FOR CEH:
  # Host discovery
  nmap -sn 192.168.1.0/24
  
  # Service version detection
  nmap -sV -sC target.com
  
  # OS detection
  nmap -O target.com
  
  # Aggressive scan
  nmap -A target.com
  
  # Script scanning
  nmap --script vuln target.com
  
  # Stealth scan
  nmap -sS -T2 -f target.com

TCP FLAGS (must know):
  SYN: Initiate connection
  ACK: Acknowledge
  FIN: Terminate connection
  RST: Reset connection
  PSH: Push data immediately
  URG: Urgent data

THREE-WAY HANDSHAKE:
  Client → SYN → Server
  Server → SYN/ACK → Client
  Client → ACK → Server
  → Connection established
```

---

## 3. Enumeration Techniques

```
ENUMERATION:

  Protocol/Service | Port | Information
  NetBIOS         | 137-139| Shares, users
  SNMP            | 161    | Device config
  LDAP            | 389    | Directory info
  SMB             | 445    | Shares, users
  NTP             | 123    | Time, hosts
  DNS             | 53     | Zone transfers
  SMTP            | 25     | User enumeration
  RPC             | 111    | Services

NETBIOS ENUMERATION:
  → nbtstat -A <IP>: NetBIOS table
  → net view: Shared resources
  → nbtscan: Scan for NetBIOS names
  → enum4linux: Comprehensive SMB enum

SNMP ENUMERATION:
  → Community strings: public, private
  → snmpwalk: Walk MIB tree
  → snmp-check: Enumerate SNMP
  → OID values contain device info
  
  snmpwalk -v2c -c public <target>

LDAP ENUMERATION:
  → ldapsearch: Query directory
  → JXplorer: GUI LDAP browser
  → Users, groups, OUs
  → Password policies

SMB ENUMERATION:
  → smbclient -L //target
  → enum4linux -a target
  → CrackMapExec: SMB enumeration
  → Shares, users, policies

SMTP ENUMERATION:
  → VRFY: Verify user exists
  → EXPN: Expand mailing list
  → RCPT TO: Recipient check
  
  telnet target 25
  VRFY admin
  → 250 = exists, 550 = not found
```

---

## 4. Vulnerability Assessment

```
VULNERABILITY SCANNING:

TOOLS:
  → Nessus: Industry standard
  → OpenVAS: Open-source alternative
  → Qualys: Cloud-based
  → Nexpose: Rapid7
  → Nikto: Web server scanner

NESSUS SCAN TYPES:
  → Basic Network Scan
  → Advanced Scan
  → Web Application Scan
  → Credentialed Scan (more thorough)
  → Compliance Scan (PCI, HIPAA)
  → Malware Scan

VULNERABILITY SCORING:
  CVSS (Common Vulnerability Scoring System):
  Score  | Severity
  0.0    | None
  0.1-3.9| Low
  4.0-6.9| Medium
  7.0-8.9| High
  9.0-10 | Critical

  CVE: Common Vulnerabilities and Exposures
  → Unique identifier (CVE-YYYY-NNNNN)
  → NVD (National Vulnerability Database)

SCAN ANALYSIS:
  → True Positive: Real vulnerability found
  → False Positive: Not actually vulnerable
  → True Negative: Correctly shows clean
  → False Negative: Missed real vulnerability
  → Always validate with manual testing

EXAM TIP: Know the difference between
  vulnerability scanning (finding vulns) and
  penetration testing (exploiting vulns).
```

---

## 5. Evasion Techniques

```
IDS/IPS EVASION:

SCANNING EVASION:
  → Fragmentation: -f flag in Nmap
  → Decoys: -D RND:10 in Nmap
  → Source port manipulation: --source-port
  → Timing: -T0 (paranoid) to -T5 (insane)
  → IP spoofing: -S <spoofed_IP>
  → Idle scan: -sI <zombie_host>

FIREWALL EVASION:
  → Tunneling: DNS, ICMP, HTTP
  → Proxy chains: Multiple proxies
  → VPN: Encrypted tunnel
  → Tor: Anonymization network
  → IPv6 tunneling over IPv4
  → SSH tunneling

PACKET CRAFTING:
  → Hping3: Custom TCP/UDP packets
    hping3 -S -p 80 -c 1 target
  → Scapy: Python packet manipulation
  → Nping: Nmap packet generator
  → Colasoft Packet Builder: GUI tool

LOG EVASION:
  → Clear event logs
  → Modify timestamps
  → Use encrypted channels
  → Disable auditing (if access allows)
  → Use trusted binaries (living off the land)

EXAM TOPICS:
  → Know IDS evasion techniques
  → Understand firewall bypass methods
  → Recognize packet crafting scenarios
  → Know anti-forensics concepts
```

---

## Summary Table

| Phase | Key Tools | CEH Exam Weight |
|-------|----------|----------------|
| Passive Recon | theHarvester, Maltego, Shodan | High |
| Active Scanning | Nmap, Hping3 | High |
| Enumeration | enum4linux, snmpwalk, ldapsearch | Medium |
| Vuln Assessment | Nessus, OpenVAS, Nikto | Medium |
| Evasion | Fragmentation, decoys, tunneling | Medium |

---

## Revision Questions

1. What is the difference between passive and active reconnaissance?
2. What Nmap scan type is considered "stealth" and why?
3. What SNMP information can be enumerated with default community strings?
4. What is a CVSS score and how are severity levels categorized?
5. What techniques can evade IDS detection during scanning?

---

*Previous: [01-exam-objectives-overview.md](01-exam-objectives-overview.md) | Next: [03-scanning-domains.md](03-scanning-domains.md)*

---

*[Back to README](../README.md)*
