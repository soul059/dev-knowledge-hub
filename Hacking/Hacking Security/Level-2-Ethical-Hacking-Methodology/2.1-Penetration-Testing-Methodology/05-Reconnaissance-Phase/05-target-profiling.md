# Target Profiling

## Unit 5 - Topic 5 | Reconnaissance Phase

---

## Overview

**Target profiling** is the process of consolidating all reconnaissance data into a structured profile of the target organization. This profile informs attack planning, helps prioritize targets, and identifies the most likely attack paths.

---

## 1. Target Profile Components

```
┌──────────────────────────────────────────────────────────────┐
│              TARGET PROFILE STRUCTURE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ORGANIZATION PROFILE:                                       │
│  ├── Company name, size, industry                            │
│  ├── Physical locations                                      │
│  ├── Business relationships (partners, vendors)              │
│  └── Revenue, public filings                                 │
│                                                              │
│  TECHNICAL PROFILE:                                          │
│  ├── IP ranges and domains                                   │
│  ├── Technology stack (OS, web server, frameworks)           │
│  ├── Cloud services (AWS, Azure, GCP)                       │
│  ├── Email infrastructure                                    │
│  └── Security controls (WAF, IPS, SIEM)                     │
│                                                              │
│  PERSONNEL PROFILE:                                          │
│  ├── Key employees (IT, security, executives)               │
│  ├── Email address patterns                                  │
│  ├── Social media presence                                   │
│  └── Published credentials in breaches                       │
│                                                              │
│  VULNERABILITY PROFILE:                                      │
│  ├── Exposed services and versions                           │
│  ├── Known CVEs for discovered software                      │
│  ├── Misconfigurations                                       │
│  └── Historical vulnerabilities                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Technology Stack Profiling

```
EXAMPLE TARGET TECHNOLOGY PROFILE:

DOMAIN: target.com
├── DNS Provider: Cloudflare
├── CDN: Cloudflare
├── Hosting: AWS (us-east-1)
│
├── Web Stack:
│   ├── Web Server: nginx 1.22
│   ├── Language: Python 3.9
│   ├── Framework: Django 4.1
│   ├── Database: PostgreSQL 14
│   ├── Cache: Redis 7.0
│   └── CMS: None (custom)
│
├── Email:
│   ├── MX: Google Workspace
│   ├── SPF: Configured ✅
│   ├── DKIM: Configured ✅
│   └── DMARC: p=reject ✅
│
├── Security Controls:
│   ├── WAF: Cloudflare
│   ├── SSL: TLS 1.3, Let's Encrypt
│   └── Headers: HSTS, CSP, X-Frame-Options
│
└── Exposed Services:
    ├── 443/tcp — nginx (HTTPS)
    ├── 22/tcp — OpenSSH 8.9 (filtered)
    └── No other external services
```

---

## 3. Personnel Profiling

```
EMPLOYEE INFORMATION:

KEY PERSONNEL:
├── CEO: Jane Smith (LinkedIn, Twitter)
├── CTO: John Doe (GitHub: johndoe, LinkedIn)
├── CISO: Alice Johnson (conference speaker)
├── IT Admin: Bob Williams (GitHub repos with company code)
└── Developer: Charlie Brown (Stack Overflow active)

EMAIL PATTERN: first.last@target.com

IDENTIFIED EMAILS (from theHarvester):
├── jane.smith@target.com
├── john.doe@target.com
├── alice.johnson@target.com
├── bob.williams@target.com
└── charlie.brown@target.com

BREACH EXPOSURE (from HIBP):
├── john.doe@target.com — LinkedIn breach (2012)
├── bob.williams@target.com — Dropbox breach (2012)
└── charlie.brown@target.com — Adobe breach (2013)

⚠️ These employees may reuse passwords
```

---

## 4. Prioritizing Targets

| Priority | Criteria | Examples |
|:--------:|----------|---------|
| **Critical** | Internet-facing, known vulns | Unpatched web server |
| **High** | Contains sensitive data | Customer database |
| **Medium** | Internal access point | VPN gateway |
| **Low** | Limited exposure | Internal wiki |

```
ATTACK PATH PRIORITIZATION:

PATH 1 (Highest priority):
Phish IT admin → Steal VPN creds → Access internal network
→ Exploit AD → Domain admin

PATH 2:
Exploit web app vuln → Shell on web server
→ Pivot to internal → Database access

PATH 3:
Use breached creds → Password spray O365
→ Access email → Find internal docs → Social engineer
```

---

## 5. Profile Documentation

```
TARGET PROFILE REPORT:
━━━━━━━━━━━━━━━━━━━━━━

1. EXECUTIVE OVERVIEW
   Company: Target Corp
   Industry: Financial Services
   Employees: ~500
   Revenue: $50M

2. ATTACK SURFACE SUMMARY
   External IPs: 15
   Subdomains: 23
   Web Applications: 4
   Email Accounts: ~500

3. KEY FINDINGS
   • 3 employees in breach databases
   • 2 subdomains with outdated software
   • Email spoofing possible (no DMARC)
   • GitHub repos contain API endpoints

4. RECOMMENDED ATTACK VECTORS
   • Web application testing (Django app)
   • Phishing campaign (breach-exposed users)
   • Password spraying (identified email pattern)

5. RISK ASSESSMENT
   Overall External Risk: MEDIUM-HIGH
   Primary Concern: Web application exposure
━━━━━━━━━━━━━━━━━━━━━━
```

---

## Summary Table

| Profile Component | Key Information |
|-------------------|----------------|
| **Organization** | Size, industry, locations, partners |
| **Technical** | IPs, tech stack, cloud, security controls |
| **Personnel** | Key employees, email patterns, breaches |
| **Vulnerabilities** | Exposed services, known CVEs, misconfigs |
| **Attack paths** | Prioritized vectors from recon data |

---

## Quick Revision Questions

1. **What components make up a target profile?**
2. **How do you determine the technology stack?**
3. **Why is personnel profiling important for pen testing?**
4. **How do you prioritize attack targets?**
5. **What role do breach databases play in target profiling?**

---

[← Previous: Information Gathering Tools](04-information-gathering-tools.md) | [Next: Attack Surface Mapping →](06-attack-surface-mapping.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*
