# OSINT Techniques

## Unit 5 - Topic 3 | Reconnaissance Phase

---

## Overview

**OSINT (Open Source Intelligence)** is intelligence gathered from publicly available sources. In penetration testing, OSINT provides valuable information about targets — **employees**, **infrastructure**, **technologies**, and **vulnerabilities** — all without touching the target's systems.

---

## 1. OSINT Categories

| Category | Sources | Information |
|----------|---------|-------------|
| **Domain** | WHOIS, DNS, crt.sh | Ownership, subdomains, IPs |
| **Infrastructure** | Shodan, Censys | Exposed services, versions |
| **People** | LinkedIn, social media | Employees, roles, contacts |
| **Code** | GitHub, GitLab, Bitbucket | Source code, credentials |
| **Documents** | Google dorks, metadata | Internal docs, file metadata |
| **Dark web** | Paste sites, breach databases | Leaked credentials, data |
| **Financial** | SEC filings, annual reports | Business relationships |

---

## 2. OSINT Framework

```
OSINT COLLECTION METHODOLOGY:

1. DOMAIN INTELLIGENCE
   ├── WHOIS records
   ├── DNS enumeration (subdomains)
   ├── SSL certificate transparency
   ├── Historical DNS (SecurityTrails)
   └── Web archive (Wayback Machine)

2. INFRASTRUCTURE INTELLIGENCE
   ├── Shodan / Censys searches
   ├── IP range enumeration
   ├── Cloud asset discovery
   ├── Email infrastructure (MX records)
   └── Content delivery networks

3. PERSONNEL INTELLIGENCE
   ├── LinkedIn employee enumeration
   ├── Email address patterns
   ├── Social media profiles
   ├── Conference presentations
   └── Published research

4. CODE INTELLIGENCE
   ├── GitHub organization repos
   ├── Employee personal repos
   ├── Code snippets on paste sites
   ├── Docker Hub images
   └── npm/PyPI packages

5. CREDENTIAL INTELLIGENCE
   ├── Breach databases (HaveIBeenPwned)
   ├── Paste sites (Pastebin)
   ├── Dark web monitoring
   └── Credential stuffing lists
```

---

## 3. Key OSINT Tools

```bash
# === DOMAIN & INFRASTRUCTURE ===
theHarvester -d target.com -b all     # Emails, subdomains, hosts
amass enum -d target.com              # Advanced subdomain discovery
subfinder -d target.com               # Fast subdomain finder
shodan search "hostname:target.com"   # Exposed services

# === EMAIL ENUMERATION ===
theHarvester -d target.com -b google,linkedin,bing
hunter.io                              # Email finder (web)
# Common email patterns:
# first.last@target.com
# flast@target.com
# firstl@target.com

# === SOCIAL MEDIA ===
# Maltego — visual link analysis
# SpiderFoot — automated OSINT
# Sherlock — username search across platforms
sherlock username                      # Find accounts across 300+ sites

# === CODE SEARCH ===
# GitHub search:
# org:targetorg password
# org:targetorg api_key
# org:targetorg secret
# Trufflehog — scan git repos for secrets
trufflehog github --org=targetorg
gitrob -org targetorg                  # Git repo analysis

# === METADATA ===
exiftool document.pdf                  # Extract file metadata
# Reveals: Author, software, GPS, dates, usernames
metagoofil -d target.com -t pdf -l 20 -o metadata_results
```

---

## 4. Credential OSINT

```bash
# === CHECK FOR BREACHED CREDENTIALS ===
# HaveIBeenPwned (HIBP)
# https://haveibeenpwned.com
# API: Check if email is in known breaches

# === DEHASHED ===
# Commercial service for searching breach databases
# Search by: email, username, IP, name, password hash

# === PASTE SITES ===
# Pastebin, Ghostbin, dpaste
# Search for: target domain, employee emails, internal data

# ⚠️ ETHICAL CONSIDERATIONS:
# ✅ Check if client's credentials appear in breaches
# ✅ Report credential exposure as a finding
# ❌ Do NOT download or store actual breach databases
# ❌ Do NOT use breached passwords to access systems
#    (unless specifically authorized)
```

---

## 5. OSINT Automation

```bash
# === SPIDERFOOT — Automated OSINT Platform ===
spiderfoot -l 127.0.0.1:5001         # Start web UI
# Modules: 200+ OSINT modules
# Automates collection across all categories

# === RECON-NG — OSINT Framework ===
recon-ng
# [recon-ng] > marketplace search
# [recon-ng] > marketplace install all
# [recon-ng] > workspaces create target
# [recon-ng] > modules load recon/domains-hosts/google_site_web
# [recon-ng] > options set SOURCE target.com
# [recon-ng] > run

# === CUSTOM OSINT SCRIPT (Bash) ===
#!/bin/bash
TARGET="$1"
echo "=== OSINT for $TARGET ==="
echo "--- WHOIS ---"
whois "$TARGET" | grep -E "Registrant|Admin|Tech|Name Server"
echo "--- DNS ---"
dig "$TARGET" ANY +short
echo "--- Subdomains (crt.sh) ---"
curl -s "https://crt.sh/?q=%.${TARGET}&output=json" | jq -r '.[].name_value' | sort -u
echo "--- theHarvester ---"
theHarvester -d "$TARGET" -b google -l 100
```

---

## Summary Table

| OSINT Category | Key Tools |
|---------------|-----------|
| **Domains** | WHOIS, crt.sh, amass, subfinder |
| **Infrastructure** | Shodan, Censys, SecurityTrails |
| **People** | LinkedIn, theHarvester, Sherlock |
| **Code** | GitHub search, Trufflehog, gitrob |
| **Credentials** | HIBP, Dehashed, paste sites |
| **Automation** | SpiderFoot, Recon-ng, Maltego |

---

## Quick Revision Questions

1. **What does OSINT stand for and why is it important?**
2. **How can GitHub reveal sensitive information?**
3. **What is certificate transparency and how does it help?**
4. **Name 3 OSINT automation tools.**
5. **What ethical considerations apply to credential OSINT?**

---

[← Previous: Active Reconnaissance](02-active-reconnaissance.md) | [Next: Information Gathering Tools →](04-information-gathering-tools.md)

---

*Unit 5 - Topic 3 of 6 | [Back to README](../README.md)*
