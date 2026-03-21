# OSINT Frameworks

## Unit 2 - Topic 5 | OSINT (Open Source Intelligence)

---

## Overview

**OSINT frameworks** provide structured methodologies and tool collections for conducting organized intelligence gathering. They help pen testers follow a systematic approach rather than ad-hoc searching, ensuring comprehensive coverage and repeatable results.

---

## 1. Popular OSINT Frameworks

| Framework | Type | Best For |
|-----------|------|---------|
| **OSINT Framework** (osintframework.com) | Web-based directory | Finding the right tool for each task |
| **Maltego** | Visual link analysis | Mapping relationships between entities |
| **SpiderFoot** | Automated scanner | Comprehensive automated OSINT |
| **Recon-ng** | Modular framework | Structured recon with modules |
| **theHarvester** | Email/subdomain tool | Quick email and subdomain collection |
| **Amass** | Subdomain discovery | Deep subdomain enumeration |

---

## 2. OSINT Framework (osintframework.com)

```
OSINT FRAMEWORK CATEGORIES:

├── Username
│   ├── Sherlock, Namechk, KnowEm
├── Email Address
│   ├── Hunter.io, Phonebook.cz, Have I Been Pwned
├── Domain Name
│   ├── WHOIS, DNS, Subdomains, Analytics
├── IP Address
│   ├── Geolocation, Reverse DNS, BGP
├── Social Networks
│   ├── LinkedIn, Facebook, Twitter tools
├── Search Engines
│   ├── Google, Bing, Yandex, Shodan
├── Images / Videos
│   ├── Reverse image search, EXIF data
├── Dark Web
│   ├── Tor search engines, paste sites
├── People Search
│   ├── Pipl, Spokeo, Whitepages
├── Phone Numbers
│   ├── Caller ID, carrier lookup
└── Threat Intelligence
    ├── VirusTotal, AlienVault OTX
```

---

## 3. Maltego

```
MALTEGO WORKFLOW:

1. Create new graph
2. Add entity: Domain (target.com)
3. Run transforms:
   ├── Domain → DNS Names
   ├── Domain → Email Addresses
   ├── Domain → WHOIS Info
   ├── Email → Person
   ├── Person → Phone Number
   ├── Person → Social Media
   └── IP → Geolocation

RESULT: Visual relationship graph showing
connections between domains, people, emails,
IPs, and social media accounts

TRANSFORMS:
├── Built-in: DNS, WHOIS, search engines
├── Shodan: Internet-connected devices
├── Have I Been Pwned: Breach data
├── VirusTotal: Malware/URL analysis
└── Social media: LinkedIn, Twitter

USE CASE: Visualize the entire attack surface
and relationships in one interactive graph
```

---

## 4. SpiderFoot

```bash
# SpiderFoot — automated OSINT collection

# Install
pip install spiderfoot

# Run web interface
spiderfoot -l 127.0.0.1:5001

# Command-line scan
spiderfoot -s target.com -m all -o csv

# MODULE CATEGORIES:
# ├── DNS lookups
# ├── WHOIS queries
# ├── Email harvesting
# ├── Breach data search
# ├── Dark web search
# ├── Social media search
# ├── Technology detection
# ├── Certificate analysis
# └── 200+ modules total

# OUTPUT:
# Comprehensive report with all findings
# Categorized by type (emails, IPs, domains, etc.)
# Correlation between findings
```

---

## 5. Choosing a Framework

| Need | Best Framework |
|------|:---:|
| Quick email/subdomain collection | theHarvester |
| Visual relationship mapping | Maltego |
| Comprehensive automated scan | SpiderFoot |
| Modular scripted recon | Recon-ng |
| Deep subdomain discovery | Amass |
| Finding the right tool | osintframework.com |

---

## Summary Table

| Framework | Strength | Type |
|-----------|----------|:---:|
| **OSINT Framework** | Tool directory | Web resource |
| **Maltego** | Visual analysis | GUI application |
| **SpiderFoot** | Automation | CLI + Web UI |
| **Recon-ng** | Modular recon | CLI framework |
| **theHarvester** | Quick collection | CLI tool |
| **Amass** | Subdomain enum | CLI tool |

---

## Quick Revision Questions

1. **What is an OSINT framework?**
2. **How does Maltego help with OSINT?**
3. **What makes SpiderFoot useful for automation?**
4. **When would you use osintframework.com?**
5. **Which framework is best for subdomain discovery?**

---

[← Previous: Public Records](04-public-records.md) | [Next: OSINT Tools →](06-osint-tools.md)

---

*Unit 2 - Topic 5 of 6 | [Back to README](../README.md)*
