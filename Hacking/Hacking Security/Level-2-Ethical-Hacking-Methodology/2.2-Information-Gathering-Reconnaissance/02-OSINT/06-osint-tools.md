# OSINT Tools

## Unit 2 - Topic 6 | OSINT (Open Source Intelligence)

---

## Overview

A comprehensive OSINT toolkit includes tools for every collection category — from email harvesting to subdomain enumeration to breach data lookup. This topic provides a practical reference of the most effective OSINT tools organized by purpose.

---

## 1. Email & People OSINT Tools

```bash
# === theHarvester (multi-source harvesting) ===
theHarvester -d target.com -b all -f report
# Sources: google, bing, linkedin, twitter, shodan, etc.

# === Hunter.io (email finder) ===
# Web: hunter.io — find email format and addresses
# API: curl "https://api.hunter.io/v2/domain-search?domain=target.com&api_key=KEY"

# === Phonebook.cz ===
# Web: phonebook.cz — email and subdomain search

# === CrossLinked (LinkedIn email gen) ===
crosslinked -f '{first}.{last}@target.com' -t 'Target Company'

# === Holehe (email account checker) ===
holehe user@target.com
# Checks if email is registered on 120+ websites

# === Sherlock (username search) ===
sherlock targetusername
# Checks 300+ social media sites for username
```

---

## 2. Domain & Subdomain Tools

```bash
# === Subfinder ===
subfinder -d target.com -o subdomains.txt
# Fast passive subdomain enumeration

# === Amass ===
amass enum -d target.com -o amass_output.txt
# Comprehensive (passive + active)

# === Assetfinder ===
assetfinder target.com
# Quick and simple subdomain finder

# === crt.sh (Certificate Transparency) ===
curl "https://crt.sh/?q=target.com&output=json" | jq '.[].name_value' | sort -u

# === DNSdumpster ===
# Web: dnsdumpster.com — visual DNS recon

# === Sublist3r ===
sublist3r -d target.com -o subs.txt

# === COMBINE TOOLS for best results ===
subfinder -d target.com -silent | sort -u > all_subs.txt
amass enum -d target.com -passive | sort -u >> all_subs.txt
sort -u all_subs.txt -o all_subs.txt
```

---

## 3. Infrastructure Tools

```bash
# === Shodan (internet device search) ===
shodan host 203.0.113.10              # Lookup specific IP
shodan search "hostname:target.com"   # Search by hostname
shodan search "org:Target Corp"       # Search by organization

# === Censys ===
# Web: search.censys.io — similar to Shodan

# === SecurityTrails ===
# Web: securitytrails.com — historical DNS, WHOIS, subdomains

# === BuiltWith ===
# Web: builtwith.com — technology profiling
# Shows: CMS, frameworks, analytics, CDN, hosting

# === Wappalyzer ===
# Browser extension — identifies web technologies

# === WhatWeb ===
whatweb https://target.com
# Command-line technology fingerprinting
```

---

## 4. Breach & Credential Tools

```bash
# === Have I Been Pwned ===
# Web: haveibeenpwned.com
# API: check if email appeared in data breaches

# === DeHashed ===
# Web: dehashed.com — search breached databases

# === LeakCheck ===
# Web: leakcheck.io — credential leak search

# === Snusbase ===
# Web: snusbase.com — breach data search engine

# === h8mail ===
h8mail -t user@target.com
# Checks multiple breach databases

# ⚠️ ETHICAL NOTE:
# Only use breach data to demonstrate risk
# Do NOT use breached credentials for unauthorized access
# Report found credentials to client immediately
```

---

## 5. Complete OSINT Toolkit Reference

| Category | Top Tools |
|----------|----------|
| **Email harvesting** | theHarvester, Hunter.io, CrossLinked |
| **Subdomains** | Subfinder, Amass, crt.sh |
| **Technology** | Wappalyzer, BuiltWith, WhatWeb |
| **Infrastructure** | Shodan, Censys, SecurityTrails |
| **People** | Sherlock, Holehe, Maltego |
| **Breaches** | HIBP, DeHashed, h8mail |
| **DNS** | dig, DNSdumpster, DNSrecon |
| **Metadata** | exiftool, FOCA |
| **Automation** | SpiderFoot, Recon-ng |
| **Visualization** | Maltego |

---

## Summary Table

| Tool | Purpose | Type |
|------|---------|:---:|
| **theHarvester** | Email and subdomain collection | CLI |
| **Subfinder** | Fast subdomain enumeration | CLI |
| **Shodan** | Internet device search | Web + CLI |
| **Sherlock** | Username search across platforms | CLI |
| **HIBP** | Breach data lookup | Web + API |
| **Maltego** | Visual OSINT analysis | GUI |

---

## Quick Revision Questions

1. **Name 3 tools for email harvesting.**
2. **What is the best tool combination for subdomain discovery?**
3. **How does Shodan differ from traditional scanning?**
4. **What ethical considerations apply to breach data?**
5. **What tool checks if an email is registered on websites?**

---

[← Previous: OSINT Frameworks](05-osint-frameworks.md)

---

*Unit 2 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Domain and DNS Reconnaissance →](../03-Domain-and-DNS-Reconnaissance/01-whois-lookups.md)*
