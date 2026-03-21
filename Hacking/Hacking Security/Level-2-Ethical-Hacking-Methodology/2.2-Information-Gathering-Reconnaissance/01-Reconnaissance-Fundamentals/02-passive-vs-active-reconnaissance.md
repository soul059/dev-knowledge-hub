# Passive vs Active Reconnaissance

## Unit 1 - Topic 2 | Reconnaissance Fundamentals

---

## Overview

Reconnaissance techniques are broadly classified as **passive** (no direct interaction with the target) or **active** (direct interaction with target systems). Understanding the differences, trade-offs, and appropriate use of each is essential for effective and stealthy information gathering.

---

## 1. Comparison

```
┌──────────────────────────────────────────────────────────────┐
│              PASSIVE vs ACTIVE RECONNAISSANCE                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PASSIVE RECONNAISSANCE:                                     │
│  ├── NO direct contact with target                           │
│  ├── Uses public/third-party sources                         │
│  ├── Undetectable by target                                  │
│  ├── Legal in most cases                                     │
│  └── Examples: OSINT, WHOIS, Google dorking                  │
│                                                              │
│  ACTIVE RECONNAISSANCE:                                      │
│  ├── DIRECT contact with target systems                      │
│  ├── Sends packets/requests to target                        │
│  ├── Detectable by IDS/IPS/firewalls                        │
│  ├── Requires authorization                                  │
│  └── Examples: Port scanning, banner grabbing                │
│                                                              │
│  SEMI-PASSIVE:                                               │
│  ├── Minimal direct contact                                  │
│  ├── Mimics normal user behavior                             │
│  ├── Hard to distinguish from legitimate traffic            │
│  └── Examples: Browsing website, DNS queries                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Detailed Comparison

| Aspect | Passive | Active |
|--------|:-------:|:------:|
| **Target interaction** | None | Direct |
| **Detection risk** | None | High |
| **Authorization needed** | Usually no | Yes |
| **Information depth** | Broad, surface-level | Deep, specific |
| **Speed** | Can be slow | Fast |
| **Tools** | OSINT tools, search engines | Nmap, Nikto, Burp |
| **Legal risk** | Minimal | Requires permission |
| **Noise** | Zero | Can trigger alerts |

---

## 3. Passive Techniques

```bash
# === SEARCH ENGINE QUERIES ===
# Google dorking
site:target.com filetype:pdf
site:target.com inurl:admin
"target.com" password OR credential

# === WHOIS LOOKUP ===
whois target.com                       # Domain registration info

# === DNS RECORDS ===
dig target.com ANY                     # DNS records (passive if using public DNS)

# === CERTIFICATE TRANSPARENCY ===
# https://crt.sh/?q=target.com
# Shows all SSL certificates → reveals subdomains

# === SHODAN / CENSYS ===
shodan search "hostname:target.com"    # Cached scan data
# No packets sent to target!

# === SOCIAL MEDIA ===
# LinkedIn — employee profiles
# GitHub — source code, secrets
# Twitter — company announcements

# === BREACH DATABASES ===
# haveibeenpwned.com — check email breaches
# dehashed.com — search breached credentials

# === WEB ARCHIVES ===
# web.archive.org — historical website snapshots
```

---

## 4. Active Techniques

```bash
# === PORT SCANNING ===
nmap -sS -sV -p- 10.0.0.5             # SYN scan + version detection
masscan 10.0.0.0/24 -p 1-65535 --rate=1000

# === BANNER GRABBING ===
nc -v 10.0.0.5 80                      # Connect and read banner
nmap -sV --version-intensity 5 10.0.0.5

# === DIRECTORY ENUMERATION ===
gobuster dir -u http://target.com -w wordlist.txt
ffuf -u http://target.com/FUZZ -w wordlist.txt

# === DNS ZONE TRANSFER ===
dig @ns1.target.com target.com axfr    # Attempt zone transfer

# === VULNERABILITY SCANNING ===
nmap --script vuln 10.0.0.5
nikto -h http://target.com

# === WEB SPIDER/CRAWL ===
burp suite spider                      # Crawl web application
```

---

## 5. Recommended Approach

```
RECON WORKFLOW:

PHASE 1: PASSIVE (Always do first)
├── OSINT gathering
├── Domain/DNS research
├── Search engine queries
├── Social media analysis
├── Breach data checks
└── Duration: Days to weeks

PHASE 2: SEMI-PASSIVE (Low risk)
├── Browse target website normally
├── DNS lookups via public resolvers
├── View robots.txt and sitemap
├── Check SSL certificate details
└── Duration: Hours to days

PHASE 3: ACTIVE (Only with authorization)
├── Port scanning
├── Service enumeration
├── Vulnerability scanning
├── Directory brute-forcing
└── Duration: Hours to days
```

---

## Summary Table

| Type | Detection | Authorization | Depth |
|------|:---------:|:------------:|:-----:|
| **Passive** | Undetectable | Usually not needed | Surface |
| **Semi-passive** | Very low risk | Borderline | Moderate |
| **Active** | Detectable | Required | Deep |

---

## Quick Revision Questions

1. **What defines passive reconnaissance?**
2. **Why perform passive recon before active recon?**
3. **Name 3 passive reconnaissance techniques.**
4. **What makes active recon detectable?**
5. **What is semi-passive reconnaissance?**

---

[← Previous: Importance of Reconnaissance](01-importance-of-reconnaissance.md) | [Next: Attack Surface Concept →](03-attack-surface-concept.md)

---

*Unit 1 - Topic 2 of 5 | [Back to README](../README.md)*
