# Unit 2: Reconnaissance for Social Engineering — Topic 2: OSINT for Social Engineering

## Overview

**Open Source Intelligence (OSINT)** is the collection and analysis of publicly available information to support social engineering operations. OSINT provides the raw material for building convincing pretexts, identifying targets, and understanding organizational structures — all without any direct contact with the target.

---

## 1. OSINT Sources for Social Engineering

```
OSINT SOURCE CATEGORIES:

CORPORATE SOURCES:
  → Company website (about, team, blog, careers)
  → SEC filings (10-K, proxy statements)
  → Press releases and news articles
  → Annual reports
  → Patent filings
  → Government contracts
  → BBB and review sites

PEOPLE SEARCH:
  → LinkedIn (primary for professional info)
  → Social media (Facebook, Twitter, Instagram)
  → Public records (property, court, voter)
  → Professional associations
  → Conference speaker lists
  → Published papers and articles
  → GitHub / Stack Overflow profiles

TECHNICAL INTELLIGENCE:
  → DNS records (MX, TXT, SPF)
  → Job postings (technology stack revealed)
  → Wayback Machine (historical site data)
  → Shodan/Censys (exposed infrastructure)
  → SSL certificate transparency logs
  → Google dorking for exposed documents

PHYSICAL INTELLIGENCE:
  → Google Maps / Street View
  → Building directories
  → Parking lot photos (badge types visible)
  → Delivery entrance locations
  → Business hours
  → Nearby businesses (for pretexting)
```

---

## 2. OSINT Tools for Social Engineering

```bash
# === EMAIL DISCOVERY ===
# theHarvester - find emails, subdomains
theHarvester -d targetcompany.com -b all -l 500

# Hunter.io - email pattern and addresses
# https://hunter.io/domain-search
# Reveals: email format, verified addresses

# Phonebook.cz - email, URL, domain search
# Comprehensive email discovery

# === SOCIAL MEDIA ===
# Sherlock - find username across platforms
sherlock targetusername

# Social Searcher - social media monitoring
# https://www.social-searcher.com/

# Twint - Twitter intelligence (no API needed)
twint -u targetuser --timeline

# === PEOPLE SEARCH ===
# Maltego - visual link analysis
# Map relationships between people, orgs, domains
# Community Edition is free

# SpiderFoot - automated OSINT
spiderfoot -s targetcompany.com -o output.html

# === DOCUMENT SEARCH ===
# Google Dorking
site:targetcompany.com filetype:pdf
site:targetcompany.com filetype:xlsx
site:linkedin.com "targetcompany" "IT manager"
"@targetcompany.com" filetype:csv
intitle:"index of" site:targetcompany.com

# FOCA - metadata extraction from documents
# Extract: author names, software versions, paths
# Reveals: internal usernames, directory structure

# === BREACH DATA ===
# HaveIBeenPwned - check if emails are in breaches
# https://haveibeenpwned.com/
# Useful for: password reuse attacks, knowledge of breaches

# === INFRASTRUCTURE ===
# Discover email security
dig targetcompany.com MX
dig targetcompany.com TXT  # SPF, DMARC records
# Weak email security = easier to spoof
```

---

## 3. Intelligence Analysis for SE

```
TURNING OSINT INTO ATTACK VECTORS:

FROM LINKEDIN:
  → Employee X is "New IT Manager at Company"
  → Attack: Impersonate IT manager to staff
  → "Hi, I'm the new IT manager. I'm doing a 
     security audit and need to verify your access."

FROM JOB POSTINGS:
  → Company hiring "Salesforce Administrator"
  → Attack: Pretend to be Salesforce support
  → "We noticed your Salesforce license needs renewal.
     Can you verify your admin credentials?"

FROM SOCIAL MEDIA:
  → CFO posted vacation photos from Maldives
  → Attack: "Hi, the CFO asked me to handle this
     wire transfer while he's traveling."

FROM DOCUMENT METADATA:
  → Internal document reveals username format: j.smith
  → Attack: Email from spoofed internal address
  → Craft emails using correct username format

FROM BREACH DATA:
  → Employee's personal email in breach
  → Attack: "Your corporate credentials may be 
     compromised. Reset your password here."

FROM TECHNICAL RECON:
  → No DMARC policy on email domain
  → Attack: Spoof emails from the exact domain
  → Emails appear 100% legitimate
```

---

## 4. OSINT Methodology

```
STRUCTURED OSINT PROCESS:

1. DEFINE REQUIREMENTS:
   → What do we need to know?
   → Who are potential targets?
   → What will support our pretext?

2. COLLECT:
   → Systematic search across all sources
   → Document everything found
   → Screenshot and archive (sources change)

3. ANALYZE:
   → Cross-reference information
   → Identify patterns and connections
   → Assess reliability of sources
   → Fill in gaps

4. PRODUCE INTELLIGENCE:
   → Create target profiles
   → Identify attack vectors
   → Design pretexts based on findings
   → Document confidence levels

OPSEC FOR OSINT COLLECTION:
  → Use VPN / Tor for searches
  → Create fake social media accounts (sock puppets)
  → Don't interact with targets directly
  → Use search aggregators, not direct queries
  → Monitor your own digital footprint
```

---

## Summary Table

| OSINT Source | Information Gained | SE Application |
|-------------|-------------------|---------------|
| LinkedIn | Role, connections, experience | Impersonation, targeting |
| Job postings | Technology, processes | Tech impersonation |
| Social media | Personal details, schedule | Pretext building |
| DNS/Email records | Email security posture | Spoofing feasibility |
| Breach databases | Compromised accounts | Credential attacks |
| Google dorking | Exposed documents | Internal knowledge |

---

## Revision Questions

1. What are the main categories of OSINT sources for social engineering?
2. How do you use LinkedIn effectively for target profiling?
3. What can job postings reveal about an organization?
4. How do you turn raw OSINT into actionable attack vectors?
5. What OPSEC measures should be taken during OSINT collection?

---

*Previous: [01-target-profiling.md](01-target-profiling.md) | Next: [03-organizational-reconnaissance.md](03-organizational-reconnaissance.md)*

---

*[Back to README](../README.md)*
