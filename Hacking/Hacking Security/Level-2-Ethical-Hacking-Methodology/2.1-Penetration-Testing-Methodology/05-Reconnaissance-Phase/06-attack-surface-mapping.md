# Attack Surface Mapping

## Unit 5 - Topic 6 | Reconnaissance Phase

---

## Overview

**Attack surface mapping** is the final reconnaissance activity where all gathered information is organized into a comprehensive map of every possible entry point into the target. This map drives the **scanning, exploitation, and post-exploitation** phases.

---

## 1. What is an Attack Surface?

```
┌──────────────────────────────────────────────────────────────┐
│              ATTACK SURFACE CATEGORIES                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NETWORK ATTACK SURFACE:                                     │
│  ├── Open ports and services                                 │
│  ├── Exposed protocols                                       │
│  ├── Network devices (routers, firewalls)                   │
│  └── VPN/remote access points                                │
│                                                              │
│  APPLICATION ATTACK SURFACE:                                 │
│  ├── Web applications and APIs                               │
│  ├── Input fields and forms                                  │
│  ├── File upload functionality                               │
│  ├── Authentication mechanisms                               │
│  └── Third-party integrations                                │
│                                                              │
│  HUMAN ATTACK SURFACE:                                       │
│  ├── Employees (phishing targets)                            │
│  ├── Contractors and vendors                                 │
│  ├── Social media exposure                                   │
│  └── Help desk / support channels                            │
│                                                              │
│  PHYSICAL ATTACK SURFACE:                                    │
│  ├── Office locations and entrances                          │
│  ├── Wireless networks                                       │
│  ├── USB ports on accessible systems                         │
│  └── Dumpster diving targets                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Building the Attack Surface Map

```
TARGET: example.com — ATTACK SURFACE MAP

EXTERNAL NETWORK:
├── 203.0.113.10 — Web server (nginx 1.22, port 443)
├── 203.0.113.11 — Mail server (Postfix, port 25/587)
├── 203.0.113.12 — VPN gateway (OpenVPN, port 1194)
├── 203.0.113.13 — DNS server (BIND 9.18, port 53)
└── 203.0.113.14 — FTP server (vsftpd 3.0.5, port 21) ⚠️

WEB APPLICATIONS:
├── https://www.example.com — Main website (Django)
│   ├── /login — Authentication
│   ├── /api/v2/ — REST API (18 endpoints)
│   ├── /upload — File upload ⚠️
│   └── /admin — Admin panel (Django Admin)
├── https://portal.example.com — Customer portal
│   ├── /auth — OAuth 2.0
│   └── /dashboard — User data access
└── https://dev.example.com — Development server ⚠️
    └── HTTP Basic Auth only

EMAIL INFRASTRUCTURE:
├── MX: mail.example.com (Google Workspace)
├── SPF: Configured ✅
├── DMARC: p=none ⚠️ (not enforced!)
└── Employees: ~200 email addresses identified

CLOUD ASSETS:
├── AWS S3: s3.example.com
├── AWS EC2: us-east-1 region
└── GitHub: github.com/example-org (5 public repos)

IDENTIFIED VULNERABILITIES:
├── FTP server exposed externally ⚠️
├── Dev server with weak auth ⚠️
├── DMARC not enforced ⚠️
├── 3 employees in breach databases ⚠️
└── File upload functionality ⚠️
```

---

## 3. Attack Surface Visualization

```
VISUAL ATTACK SURFACE:

                    INTERNET
                       │
         ┌─────────────┼─────────────┐
         │             │             │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
    │  Web    │  │  Mail   │  │   VPN   │
    │ Server  │  │ Server  │  │ Gateway │
    │ :443    │  │ :25/587 │  │ :1194   │
    └────┬────┘  └────┬────┘  └────┬────┘
         │             │             │
         └─────────────┼─────────────┘
                       │
              ┌────────▼────────┐
              │  INTERNAL NET   │
              │  10.0.0.0/24    │
              ├─────────────────┤
              │ • DC (AD)       │
              │ • File server   │
              │ • Database      │
              │ • Dev server    │
              │ • Workstations  │
              └─────────────────┘
```

---

## 4. Attack Surface Reduction

| Finding | Risk | Recommendation |
|---------|:----:|---------------|
| FTP exposed | High | Disable or restrict to VPN |
| Dev server exposed | High | Move behind VPN/firewall |
| DMARC not enforced | Medium | Set to `p=quarantine` or `p=reject` |
| File upload | Medium | Validate file types, scan uploads |
| Breached creds | Medium | Force password reset, enable MFA |

---

## 5. Tools for Attack Surface Mapping

| Tool | Purpose |
|------|---------|
| **Nmap** | Port and service discovery |
| **Amass** | Comprehensive attack surface mapping |
| **OWASP ZAP** | Web application mapping |
| **Burp Suite** | Web app entry point discovery |
| **Masscan** | Fast port scanning at scale |
| **Aquatone** | Visual subdomain inspection |
| **Gowitness** | Screenshot web services |

```bash
# Comprehensive attack surface scan
amass enum -d target.com -o subdomains.txt
cat subdomains.txt | httpx -o live_hosts.txt
cat live_hosts.txt | gowitness file -f - -P screenshots/
nmap -iL live_hosts.txt -sV -oA surface_scan
```

---

## Summary Table

| Surface | Key Elements |
|---------|-------------|
| **Network** | Open ports, exposed services, VPN |
| **Application** | Web apps, APIs, forms, uploads |
| **Human** | Employees, social media, phishing |
| **Physical** | Offices, wireless, USB access |
| **Cloud** | S3 buckets, repos, cloud services |

---

## Quick Revision Questions

1. **What are the 4 categories of attack surface?**
2. **How do you create an attack surface map?**
3. **Why is a development server a high-risk finding?**
4. **What tools help with attack surface discovery?**
5. **How does attack surface mapping connect to the exploitation phase?**

---

[← Previous: Target Profiling](05-target-profiling.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Scanning and Enumeration →](../06-Scanning-and-Enumeration/01-network-scanning.md)*
