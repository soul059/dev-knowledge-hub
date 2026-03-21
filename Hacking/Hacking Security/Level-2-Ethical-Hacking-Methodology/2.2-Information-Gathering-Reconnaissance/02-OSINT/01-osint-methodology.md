# OSINT Methodology

## Unit 2 - Topic 1 | OSINT (Open Source Intelligence)

---

## Overview

**OSINT (Open Source Intelligence)** is the practice of collecting and analyzing information from publicly available sources. In penetration testing, OSINT provides critical intelligence about targets without sending a single packet to their systems — making it completely undetectable.

---

## 1. OSINT Process

```
┌──────────────────────────────────────────────────────────────┐
│              OSINT INTELLIGENCE CYCLE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. PLANNING & DIRECTION                                     │
│     └── Define what you need to find                         │
│                                                              │
│  2. COLLECTION                                               │
│     └── Gather data from sources                             │
│                                                              │
│  3. PROCESSING                                               │
│     └── Clean, organize, and format data                     │
│                                                              │
│  4. ANALYSIS                                                 │
│     └── Interpret data, find patterns                        │
│                                                              │
│  5. DISSEMINATION                                            │
│     └── Report findings to stakeholders                      │
│                                                              │
│  6. FEEDBACK                                                 │
│     └── Refine requirements, iterate                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. OSINT Source Categories

| Source Type | Examples | Value |
|------------|---------|:---:|
| **Search engines** | Google, Bing, DuckDuckGo | ★★★★★ |
| **Social media** | LinkedIn, Twitter, Facebook | ★★★★★ |
| **Code repositories** | GitHub, GitLab, Bitbucket | ★★★★ |
| **Domain/DNS** | WHOIS, crt.sh, DNSdumpster | ★★★★ |
| **Internet scanners** | Shodan, Censys, ZoomEye | ★★★★ |
| **Public records** | SEC filings, patents, court records | ★★★ |
| **Job boards** | LinkedIn Jobs, Indeed, Glassdoor | ★★★ |
| **Dark web** | Paste sites, forums | ★★★ |
| **Web archives** | Wayback Machine, cached pages | ★★★ |
| **Geospatial** | Google Maps, satellite imagery | ★★ |

---

## 3. OSINT Collection Workflow

```bash
# STEP 1: Define collection requirements
# Target: target.com
# Goals: emails, subdomains, technologies, employees, credentials

# STEP 2: Automated broad collection
theHarvester -d target.com -b all -f target_report
# Collects: emails, subdomains, IPs from multiple sources

# STEP 3: Manual deep dives per category
# Emails → Hunter.io, LinkedIn
# Subdomains → crt.sh, subfinder, amass
# Tech stack → BuiltWith, Wappalyzer
# People → LinkedIn, company website
# Credentials → HIBP, dehashed

# STEP 4: Analyze and correlate
# Connect email format with employee names
# Map subdomains to services
# Identify technology versions for CVE research

# STEP 5: Document everything
# Organized by category
# Source attribution for each finding
# Confidence level for each data point
```

---

## 4. OSINT Principles

| Principle | Description |
|-----------|-------------|
| **Passive first** | Collect without target interaction |
| **Source diversity** | Use multiple sources to cross-validate |
| **Verification** | Confirm findings from independent sources |
| **Documentation** | Record source and timestamp for each finding |
| **Legal compliance** | Stay within legal boundaries |
| **Operational security** | Don't reveal your identity or intent |
| **Iteration** | Each finding may lead to new collection targets |

---

## 5. OSINT Operational Security

```
PROTECT YOUR IDENTITY DURING OSINT:

DO:
├── Use a VPN or Tor for all research
├── Use dedicated research accounts (not personal)
├── Use a separate browser profile
├── Clear cookies between searches
└── Use disposable email for sign-ups

DON'T:
├── Use personal social media accounts
├── Connect from your home/office IP
├── Log in to target systems
├── Download files that may beacon home
└── Leave traces on target-owned platforms
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **OSINT** | Intelligence from public sources |
| **Process** | Plan → Collect → Process → Analyze → Report |
| **Sources** | Search engines, social media, code repos, DNS |
| **Principles** | Passive, diverse sources, verify, document |
| **OpSec** | VPN, dedicated accounts, clean browser |
| **Value** | Undetectable intelligence gathering |

---

## Quick Revision Questions

1. **What is OSINT and why is it valuable for pen testing?**
2. **What are the 6 steps of the OSINT intelligence cycle?**
3. **Name 5 categories of OSINT sources.**
4. **Why is operational security important during OSINT?**
5. **How do you verify OSINT findings?**

---

[Next: Search Engine Techniques →](02-search-engine-techniques.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
