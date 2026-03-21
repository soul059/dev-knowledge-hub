# Technology Fingerprinting

## Unit 5 - Topic 2 | Web Reconnaissance

---

## Overview

**Technology fingerprinting** identifies the specific software, frameworks, libraries, and services running on a target web application. Knowing the exact technology stack allows targeted vulnerability research and exploit selection.

---

## 1. Technology Stack Components

```
WEB APPLICATION TECHNOLOGY STACK:

┌──────────────────────────────────────────┐
│  FRONTEND                                │
│  ├── Framework: React, Angular, Vue      │
│  ├── CSS: Bootstrap, Tailwind            │
│  └── JavaScript: jQuery, lodash          │
├──────────────────────────────────────────┤
│  BACKEND                                 │
│  ├── Language: PHP, Python, Java, .NET   │
│  ├── Framework: Laravel, Django, Spring  │
│  └── CMS: WordPress, Drupal, Joomla     │
├──────────────────────────────────────────┤
│  SERVER                                  │
│  ├── Web Server: Apache, nginx, IIS      │
│  ├── App Server: Tomcat, Gunicorn        │
│  └── OS: Linux, Windows Server           │
├──────────────────────────────────────────┤
│  DATABASE                                │
│  ├── SQL: MySQL, PostgreSQL, MSSQL       │
│  └── NoSQL: MongoDB, Redis               │
├──────────────────────────────────────────┤
│  INFRASTRUCTURE                          │
│  ├── Cloud: AWS, Azure, GCP              │
│  ├── CDN: Cloudflare, Akamai             │
│  └── Container: Docker, Kubernetes       │
└──────────────────────────────────────────┘
```

---

## 2. Fingerprinting Tools

```bash
# === WAPPALYZER (Browser Extension / CLI) ===
# Most popular — identifies 1000+ technologies
# Detects: CMS, frameworks, analytics, CDN, etc.
# Available as browser extension and npm package
wappalyzer https://target.com

# === WHATWEB ===
whatweb https://target.com
# Output: Apache[2.4.49], Bootstrap, jQuery[3.6.0],
#         PHP[8.1], WordPress[6.3], MySQL

whatweb -a 3 https://target.com  # Aggressive mode (more requests)

# === BUILTWITH ===
# builtwith.com — comprehensive technology profiling
# Shows: frameworks, analytics, hosting, advertising, etc.

# === NETCRAFT ===
# toolbar.netcraft.com/site_report
# Shows: server technology, hosting history, risk rating

# === RETIRE.JS (Vulnerable JS Libraries) ===
retire --js https://target.com
# Detects outdated JavaScript libraries with known CVEs

# === NIKTO (Web Server Scanner) ===
nikto -h https://target.com
# Checks: server config, dangerous files, outdated software
```

---

## 3. Manual Fingerprinting Techniques

```bash
# === CMS DETECTION ===
# WordPress:
curl -s https://target.com | grep "wp-content"
curl -s https://target.com/wp-login.php
curl -s https://target.com/wp-json/wp/v2/users

# Drupal:
curl -s https://target.com/CHANGELOG.txt
curl -s https://target.com | grep "drupal"

# Joomla:
curl -s https://target.com/administrator/
curl -s https://target.com | grep "joomla"

# === FRAMEWORK DETECTION ===
# ASP.NET: .aspx extensions, ViewState, __EVENTVALIDATION
# PHP: .php extensions, PHPSESSID cookie
# Java: JSESSIONID cookie, .jsp/.do extensions
# Python/Django: csrfmiddlewaretoken, django admin
# Ruby on Rails: _session_id cookie, /rails/ paths

# === FAVICON HASH (Shodan) ===
# Calculate favicon hash:
python3 -c "
import mmh3, requests, codecs
response = requests.get('https://target.com/favicon.ico')
favicon = codecs.encode(response.content, 'base64')
hash = mmh3.hash(favicon)
print(f'Shodan query: http.favicon.hash:{hash}')
"
# Search Shodan for other servers with same favicon
```

---

## 4. Version-Specific Fingerprinting

| Technology | Version Detection Method |
|------------|------------------------|
| **Apache** | `Server:` header, `/server-status` |
| **nginx** | `Server:` header, error pages |
| **IIS** | `Server:` header, `.aspx` handling |
| **PHP** | `X-Powered-By`, `phpinfo()` page |
| **WordPress** | `/readme.html`, meta generator tag |
| **jQuery** | `jQuery.fn.jquery` in console |
| **React** | `__REACT_DEVTOOLS` global |
| **Angular** | `ng-version` attribute |

---

## 5. Technology → Vulnerability Mapping

```
TECHNOLOGY IDENTIFIED          VULNERABILITY RESEARCH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Apache 2.4.49                → CVE-2021-41773 (path traversal)
WordPress 6.3                → Check wpscan vulnerability DB
jQuery 3.4.1                 → CVE-2020-11022 (XSS)
PHP 8.0.x                   → Check PHP security advisories
OpenSSL 1.1.1               → Check OpenSSL changelog
Bootstrap 4.x               → Generally safe (CSS framework)
MySQL 5.7                   → Check MySQL CVE database

TOOLS FOR CVE LOOKUP:
├── cvedetails.com
├── nvd.nist.gov
├── exploit-db.com
├── searchsploit <technology>
└── vulners.com
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Fingerprinting** | Identify exact software versions |
| **Wappalyzer** | Browser extension for tech detection |
| **WhatWeb** | CLI tool for technology identification |
| **Manual** | Headers, cookies, file extensions, source |
| **Favicon hash** | Shodan search for similar servers |
| **CVE mapping** | Match versions to known vulnerabilities |

---

## Quick Revision Questions

1. **What tools detect web technologies automatically?**
2. **How do you manually identify the CMS in use?**
3. **What does the favicon hash technique reveal?**
4. **Why is version-specific fingerprinting important?**
5. **How do you map detected technologies to vulnerabilities?**

---

[← Previous: Website Analysis](01-website-analysis.md) | [Next: Directory Enumeration →](03-directory-enumeration.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
