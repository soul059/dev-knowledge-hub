# Website Analysis

## Unit 5 - Topic 1 | Web Reconnaissance

---

## Overview

**Website analysis** is the systematic examination of a target's web presence to gather intelligence about technologies, structure, functionality, and potential vulnerabilities. This forms the foundation of web-focused reconnaissance.

---

## 1. Initial Website Review

```
MANUAL WEBSITE ANALYSIS CHECKLIST:

VISUAL INSPECTION:
├── Home page layout and design framework
├── Navigation structure (menus, links)
├── User registration / login forms
├── Contact forms and input fields
├── File upload functionality
├── Error pages (404, 500 — information disclosure?)
├── Comments in HTML source
└── JavaScript files and API endpoints

FUNCTIONAL ANALYSIS:
├── Authentication mechanisms (login, SSO, OAuth)
├── Search functionality (SQL injection vector?)
├── Payment processing (PCI scope?)
├── User-generated content (XSS vector?)
├── API endpoints (REST, GraphQL?)
├── Mobile app integration
└── Third-party integrations
```

---

## 2. Source Code Inspection

```bash
# === VIEW PAGE SOURCE ===
curl -s https://target.com | head -100

# === LOOK FOR COMMENTS ===
curl -s https://target.com | grep -i "<!--"
# Developer comments may reveal:
# <!-- TODO: remove admin debug mode -->
# <!-- API endpoint: /api/v2/users -->
# <!-- Last updated by john.smith@target.com -->

# === FIND JAVASCRIPT FILES ===
curl -s https://target.com | grep -oP 'src="[^"]*\.js"'
# app.js, main.bundle.js, vendor.js

# === EXTRACT API ENDPOINTS FROM JS ===
curl -s https://target.com/js/app.js | grep -oP '"/api/[^"]*"'
# "/api/v1/users"
# "/api/v1/admin/settings"
# "/api/v1/upload"

# === FIND HIDDEN PATHS IN JS ===
# LinkFinder — extracts URLs from JavaScript
python3 linkfinder.py -i https://target.com/js/app.js -o cli

# === EXTRACT EMAILS ===
curl -s https://target.com | grep -oP '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
```

---

## 3. HTTP Header Analysis

```bash
# === RESPONSE HEADERS ===
curl -I https://target.com

# INFORMATION FROM HEADERS:
# Server: Apache/2.4.49        → Web server + version
# X-Powered-By: PHP/8.1        → Backend language
# X-AspNet-Version: 4.0.30319  → .NET version
# Set-Cookie: PHPSESSID=...    → PHP session (confirms PHP)
# Set-Cookie: JSESSIONID=...   → Java application
# X-Frame-Options: DENY        → Clickjacking protection
# Content-Security-Policy: ... → CSP configuration
# Strict-Transport-Security:   → HSTS enabled

# SECURITY HEADERS CHECK:
# ✅ Good: X-Frame-Options, CSP, HSTS present
# ❌ Bad: Server version disclosed, no security headers

# === SUPPORTED HTTP METHODS ===
curl -X OPTIONS https://target.com -I
# Allow: GET, POST, OPTIONS, PUT, DELETE
# PUT and DELETE may be dangerous!

# === AUTOMATED HEADER SCAN ===
# securityheaders.com — grades security headers
```

---

## 4. Website Metadata

| Analysis Area | What to Look For |
|--------------|-----------------|
| **Meta tags** | Description, keywords, author, generator |
| **Favicon** | Framework identification (Shodan favicon hash) |
| **Cookies** | Session management technology |
| **SSL cert** | Organization name, subdomains (SAN) |
| **Error messages** | Stack traces, file paths, version numbers |
| **Robots.txt** | Hidden directories and restricted paths |
| **Sitemap.xml** | Complete URL structure |

---

## 5. Website Profiling Report

```
WEBSITE ANALYSIS REPORT: target.com

BASIC INFO:
├── Title: "Target Corporation - Welcome"
├── CMS: WordPress 6.3
├── Server: nginx/1.24 on Ubuntu
├── Backend: PHP 8.1
├── Database: MySQL (inferred from WP)
└── CDN: Cloudflare

FUNCTIONALITY:
├── User login: /wp-login.php
├── REST API: /wp-json/wp/v2/
├── Search: /?s=query
├── Admin: /wp-admin/
├── Contact form: /contact/ (Gravity Forms)
└── E-commerce: WooCommerce plugin

SECURITY POSTURE:
├── ✅ HSTS enabled
├── ✅ X-Frame-Options: DENY
├── ❌ Server version disclosed
├── ❌ PHP version in headers
├── ❌ WordPress login page accessible
└── ❌ XML-RPC enabled (/xmlrpc.php)

POTENTIAL VECTORS:
├── WordPress known vulnerabilities
├── Plugin vulnerabilities
├── XML-RPC brute force
├── REST API user enumeration
└── wp-login.php brute force
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Source code** | HTML comments, JS files, API endpoints |
| **Headers** | Server version, technology stack, security |
| **Cookies** | Session management technology |
| **SSL cert** | Organization info, subdomains |
| **Error pages** | Information disclosure |
| **Tools** | curl, LinkFinder, Wappalyzer |

---

## Quick Revision Questions

1. **What can HTML comments reveal during reconnaissance?**
2. **How do HTTP headers disclose technology stack?**
3. **What is LinkFinder used for?**
4. **Name 3 security headers to check for.**
5. **Why inspect JavaScript files during web reconnaissance?**

---

[Next: Technology Fingerprinting →](02-technology-fingerprinting.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
