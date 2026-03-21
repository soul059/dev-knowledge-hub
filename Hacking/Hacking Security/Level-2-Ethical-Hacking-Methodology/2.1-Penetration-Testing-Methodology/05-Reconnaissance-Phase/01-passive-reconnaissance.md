# Passive Reconnaissance

## Unit 5 - Topic 1 | Reconnaissance Phase

---

## Overview

**Passive reconnaissance** involves gathering information about a target **without directly interacting** with their systems. Since no packets are sent to the target, passive recon is **undetectable** and leaves no logs. It is the safest starting point for any engagement.

---

## 1. Passive vs Active Reconnaissance

```
┌──────────────────────────────────────────────────────────────┐
│              PASSIVE vs ACTIVE RECON                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PASSIVE RECONNAISSANCE:                                     │
│  ├── No direct interaction with target                       │
│  ├── Uses publicly available information                     │
│  ├── Undetectable by target                                  │
│  ├── Examples: Google, WHOIS, social media                  │
│  └── Risk: NONE                                              │
│                                                              │
│  ACTIVE RECONNAISSANCE:                                      │
│  ├── Direct interaction with target                          │
│  ├── Sends packets to target systems                         │
│  ├── Detectable (logs, IDS alerts)                          │
│  ├── Examples: Port scans, DNS queries                       │
│  └── Risk: Detection, legal issues                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Passive Recon Techniques

| Technique | Tool/Source | Information Gathered |
|-----------|-----------|---------------------|
| **WHOIS lookup** | whois, ICANN | Domain owner, registrar, contacts |
| **DNS records** | dig, nslookup | Subdomains, mail servers, IPs |
| **Search engines** | Google dorks | Exposed files, directories, info |
| **Social media** | LinkedIn, Twitter | Employees, tech stack, culture |
| **Job postings** | Indeed, LinkedIn | Technologies used, team structure |
| **Archive.org** | Wayback Machine | Historical versions of website |
| **Certificate search** | crt.sh | Subdomains from SSL certificates |
| **GitHub/GitLab** | Code repositories | Source code, credentials, configs |
| **Shodan** | IoT search engine | Exposed services, banners |

---

## 3. Google Dorking

```bash
# GOOGLE DORK OPERATORS:

site:target.com                       # Results from target domain only
inurl:admin site:target.com           # Admin pages
intitle:"index of" site:target.com    # Directory listings
filetype:pdf site:target.com          # PDF files
filetype:sql site:target.com          # SQL files (possible DB dumps)
filetype:env site:target.com          # Environment files (credentials!)
filetype:log site:target.com          # Log files

# FIND SENSITIVE INFO:
"password" filetype:log site:target.com
"api_key" OR "api key" site:target.com
"BEGIN RSA PRIVATE KEY" site:target.com
inurl:wp-admin site:target.com        # WordPress admin
inurl:phpinfo site:target.com         # PHP info pages

# FIND EXPOSED SYSTEMS:
intitle:"Apache2 Ubuntu Default Page"
intitle:"Welcome to nginx"
intitle:"Dashboard [Jenkins]"
inurl:"/wp-content/uploads/"
```

---

## 4. WHOIS and DNS Enumeration

```bash
# === WHOIS ===
whois target.com
# Returns: Registrant, registrar, creation date, nameservers
# ⚠️ GDPR has reduced WHOIS data availability

# === DNS RECORDS ===
dig target.com ANY                    # All records
dig target.com A                      # IPv4 addresses
dig target.com MX                     # Mail servers
dig target.com NS                     # Name servers
dig target.com TXT                    # SPF, DKIM, verification records
dig target.com AAAA                   # IPv6 addresses

# === CERTIFICATE TRANSPARENCY ===
# crt.sh — find subdomains from SSL certificates
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
    jq -r '.[].name_value' | sort -u

# === SHODAN ===
# Search for target's exposed services
# shodan search "hostname:target.com"
# shodan host 203.0.113.1
```

---

## 5. Social Media & OSINT

```
INFORMATION FROM SOCIAL MEDIA:

LINKEDIN:
├── Employee names and titles
├── Technology stack (from job descriptions)
├── Office locations
├── Organizational structure
└── Recent projects/changes

TWITTER/X:
├── Employee opinions and complaints
├── Technology mentions
├── Event attendance
└── Personal information

GITHUB:
├── Source code (public repos)
├── Hardcoded credentials
├── API endpoints and documentation
├── Internal project names
├── Developer email addresses

JOB POSTINGS:
├── "Looking for experienced Cisco ASA administrator"
│   → They use Cisco ASA firewalls
├── "Must know AWS, Docker, Kubernetes"
│   → Cloud infrastructure details
└── "Experience with SAP required"
    → Using SAP ERP system
```

---

## Summary Table

| Technique | Tool | Risk Level |
|-----------|------|:----------:|
| **WHOIS** | whois, ICANN | None |
| **Google dorks** | Google search | None |
| **Certificate search** | crt.sh | None |
| **Social media** | LinkedIn, Twitter | None |
| **Shodan** | shodan.io | None |
| **Code repos** | GitHub, GitLab | None |
| **Wayback Machine** | archive.org | None |

---

## Quick Revision Questions

1. **What makes passive recon "passive"?**
2. **How do Google dorks help in reconnaissance?**
3. **What information can WHOIS reveal?**
4. **How can job postings reveal an organization's tech stack?**
5. **What is certificate transparency and how does crt.sh help?**

---

[Next: Active Reconnaissance →](02-active-reconnaissance.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
