# Information Categories

## Unit 1 - Topic 4 | Reconnaissance Fundamentals

---

## Overview

During reconnaissance, vast amounts of data are collected. Organizing this data into **categories** helps pen testers systematically identify gaps, prioritize collection efforts, and build a comprehensive target profile.

---

## 1. Five Information Categories

| Category | What to Collect | Sources |
|----------|----------------|---------|
| **Organizational** | Company structure, people, locations, partners | Website, LinkedIn, SEC filings |
| **Infrastructure** | IPs, domains, DNS, email, cloud, hosting | WHOIS, DNS, Shodan, crt.sh |
| **Technical** | OS, frameworks, databases, languages, versions | Wappalyzer, headers, job posts |
| **Human** | Names, emails, social profiles, breach data | LinkedIn, Hunter.io, HIBP |
| **Security Posture** | Policies, certifications, bug bounty, breaches | Website, press, security blogs |

---

## 2. Information Value Assessment

| Information | Attack Value | Example Use |
|-------------|:---:|------------|
| **Email format** | ★★★★★ | Brute force, phishing, credential stuffing |
| **Software versions** | ★★★★★ | Targeted CVE exploitation |
| **Employee names** | ★★★★ | Social engineering, spear phishing |
| **IP ranges** | ★★★★ | Scope scanning, target identification |
| **Technology stack** | ★★★★ | Attack planning, exploit selection |
| **Org hierarchy** | ★★★ | Social engineering pretexting |
| **Job postings** | ★★★ | Technology identification |
| **Physical locations** | ★★ | Physical pen testing |

---

## 3. Collection Framework

```
ORGANIZATIONAL:
├── Company website (About, Team pages)
├── SEC filings and annual reports
├── Crunchbase, Bloomberg, LinkedIn
├── Press releases and news articles
└── Patent and trademark databases

INFRASTRUCTURE:
├── WHOIS databases
├── DNS lookups (dig, nslookup)
├── Certificate Transparency (crt.sh)
├── Shodan, Censys, ZoomEye
├── BGP looking glass tools
└── IP geolocation services

TECHNICAL:
├── Wappalyzer, WhatWeb, BuiltWith
├── HTTP headers analysis
├── Error pages and debug info
├── robots.txt, sitemap.xml
├── Source code (GitHub, GitLab)
└── Job postings (required skills)
```

---

## 4. Target Profile Template

```
TARGET PROFILE: [Company Name]

ORGANIZATIONAL:
  Industry: [e.g., Financial Services]
  Employees: [e.g., 500-1000]
  Locations: [e.g., New York, London]
  Key People: CEO, CTO, CISO names

INFRASTRUCTURE:
  Primary Domain: target.com
  Subdomains: [count] discovered
  IP Ranges: 203.0.113.0/24 (ASN: AS12345)
  Email: MX points to Google Workspace
  CDN: Cloudflare
  Cloud: AWS (us-east-1)

TECHNICAL:
  Web: React frontend, Node.js backend
  Database: PostgreSQL
  OS: Ubuntu 22.04 (web servers)
  WAF: Cloudflare
  Languages: JavaScript, Python
```

---

## 5. Collection Completeness Checklist

| Category | Collected? | Gaps |
|----------|:---:|------|
| Company structure | ✅ | Need subsidiary info |
| IP ranges | ✅ | Complete |
| Subdomains | ✅ | 42 found |
| Email format | ✅ | first.last@target.com |
| Tech stack | ⚠️ | Missing database info |
| Employees | ✅ | 85 names gathered |
| Security posture | ❌ | Need more research |

---

## Summary Table

| Category | Priority | Key Sources |
|----------|:--------:|------------|
| **Organizational** | Medium | LinkedIn, website |
| **Infrastructure** | High | WHOIS, DNS, Shodan |
| **Technical** | High | Wappalyzer, headers |
| **Human** | High | LinkedIn, HIBP |
| **Security** | Medium | Bug bounty, press |

---

## Quick Revision Questions

1. **What are the 5 categories of recon information?**
2. **Which information type has the highest attack value?**
3. **How do job postings reveal technical information?**
4. **Why create a target profile during recon?**
5. **How do you identify gaps in collected information?**

---

[← Previous: Attack Surface Concept](03-attack-surface-concept.md) | [Next: Reconnaissance Lifecycle →](05-reconnaissance-lifecycle.md)

---

*Unit 1 - Topic 4 of 5 | [Back to README](../README.md)*