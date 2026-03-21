# Application Mapping

## Unit 3: Web Reconnaissance — Topic 1

## 🎯 Overview

Application mapping is the process of understanding a web application's structure, functionality, and attack surface before testing. It involves crawling, spidering, and manual exploration to build a complete map of endpoints, parameters, forms, and logic flows. Thorough mapping is the foundation of effective web application penetration testing.

---

## 1. Application Mapping Process

```
┌──────────────────────────────────────────────────────────────┐
│                APPLICATION MAPPING WORKFLOW                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Passive Discovery                                        │
│     └→ Search engines, cached pages, archives               │
│                                                              │
│  2. Active Spidering                                         │
│     └→ Automated crawling of links and forms                │
│                                                              │
│  3. Manual Exploration                                       │
│     └→ Walk through app functionality, note behaviors       │
│                                                              │
│  4. Content Discovery                                        │
│     └→ Directory brute-forcing, hidden files                │
│                                                              │
│  5. Functionality Mapping                                    │
│     └→ Document all features, user roles, workflows         │
│                                                              │
│  6. Entry Point Identification                               │
│     └→ Parameters, headers, cookies, file uploads           │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Automated Spidering

```bash
# Burp Suite Spider
# 1. Configure browser proxy → Burp Suite (127.0.0.1:8080)
# 2. Browse to target → add to scope
# 3. Spider → Crawl target
# 4. Review sitemap in Target → Site map

# OWASP ZAP Spider
# 1. Quick Start → enter URL → Attack
# 2. Spider tab shows discovered URLs
# 3. Review Sites panel for sitemap

# Gospider — CLI crawler
gospider -s https://target.com -d 3 -c 10 --other-source

# Hakrawler — fast crawler
echo "https://target.com" | hakrawler -d 3 -plain

# Katana — advanced crawler
katana -u https://target.com -d 5 -jc -kf -ef css,png,jpg
```

### Spider Limitations

```
┌──────────────────────────────────────────────────────────────┐
│           WHAT SPIDERS MISS                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ✗ JavaScript-rendered content (SPAs)                        │
│  ✗ Pages behind authentication                               │
│  ✗ Multi-step forms (requires specific input sequence)       │
│  ✗ Content requiring specific User-Agent                     │
│  ✗ Time-based functionality (features shown at certain times)│
│  ✗ Role-based content (admin-only pages)                     │
│  ✗ API endpoints not linked from HTML                        │
│  ✗ WebSocket endpoints                                       │
│                                                              │
│  → Manual exploration is ALWAYS needed                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Manual Application Mapping

### Systematic Exploration Checklist

```
┌──────────────────────────────────────────────────────────────┐
│           MANUAL MAPPING CHECKLIST                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  □ Register account (multiple roles if possible)             │
│  □ Complete user profile                                     │
│  □ Test all forms (search, contact, upload, settings)        │
│  □ Navigate all menu items and submenus                      │
│  □ Test password reset flow                                  │
│  □ Test email verification                                   │
│  □ Explore help/documentation pages                          │
│  □ Check error pages (404, 500, access denied)              │
│  □ Try different user roles (admin, user, guest)            │
│  □ Check mobile vs desktop versions                          │
│  □ Review sitemap.xml and robots.txt                        │
│  □ Check API documentation (/docs, /swagger, /api-docs)    │
│  □ Inspect JavaScript files for hidden endpoints            │
│  □ Check WebSocket connections                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Entry Point Identification

```bash
# URL parameters
https://target.com/search?q=test&page=1&sort=date

# POST body parameters
username=admin&password=test&remember=1

# HTTP headers as inputs
Cookie: session=abc123
Authorization: Bearer eyJ...
X-Forwarded-For: 127.0.0.1
Referer: https://target.com/login

# File upload fields
Content-Type: multipart/form-data

# JSON/XML body
{"action": "transfer", "amount": 100, "to": "user2"}

# Hidden form fields
<input type="hidden" name="role" value="user">
<input type="hidden" name="price" value="99.99">
```

### Entry Point Categories

| Category | Examples | Common Vulnerabilities |
|----------|---------|----------------------|
| Query strings | `?id=1&name=test` | SQLi, XSS, IDOR |
| POST data | Form fields, JSON body | SQLi, XSS, command injection |
| URL path | `/user/123/profile` | IDOR, path traversal |
| Cookies | `session=abc123` | Session attacks |
| Headers | `Host`, `Referer`, `UA` | Header injection, SSRF |
| File uploads | Image, document uploads | File upload attacks |
| WebSockets | Real-time messaging | Injection via WS |

---

## 5. Sitemap Construction

```
Target: https://webapp.target.com

├── / (Home)
├── /login
├── /register
├── /forgot-password
├── /dashboard (auth required)
│   ├── /dashboard/profile
│   ├── /dashboard/settings
│   └── /dashboard/orders
│       ├── /dashboard/orders/new
│       └── /dashboard/orders/{id}
├── /admin (admin role required)
│   ├── /admin/users
│   ├── /admin/logs
│   └── /admin/config
├── /api/v2
│   ├── /api/v2/users
│   ├── /api/v2/orders
│   └── /api/v2/products
├── /robots.txt
├── /sitemap.xml
└── /.well-known/
```

```bash
# Check robots.txt for hidden paths
curl https://target.com/robots.txt
# Disallow: /admin/
# Disallow: /backup/
# Disallow: /internal/

# Check sitemap.xml
curl https://target.com/sitemap.xml

# Check .well-known directory
curl https://target.com/.well-known/security.txt
curl https://target.com/.well-known/openid-configuration
```

---

## 6. Scope Documentation

```
┌──────────────────────────────────────────────────────────────┐
│              APPLICATION MAP TEMPLATE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Application: target.com                                     │
│  Technology: React + Node.js + MongoDB                       │
│  Auth: JWT tokens + OAuth (Google)                           │
│                                                              │
│  Roles Identified:                                           │
│  • Guest (unauthenticated)                                   │
│  • User (registered, email verified)                         │
│  • Premium (paid subscription)                               │
│  • Admin (full access)                                       │
│                                                              │
│  Key Functionality:                                          │
│  • User registration/login                                   │
│  • Profile management (file upload)                          │
│  • Search (with filters)                                     │
│  • Payment processing                                        │
│  • Admin panel (user management)                             │
│                                                              │
│  Entry Points: 47 parameters identified                      │
│  API Endpoints: 23 endpoints documented                      │
│  File Uploads: 2 locations (avatar, document)                │
│  External Integrations: Stripe, Google OAuth, SendGrid       │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Mapping Method | Best For | Tool |
|---------------|---------|------|
| Automated spider | Initial discovery | Burp Spider, ZAP |
| Manual browsing | Auth flows, multi-step | Browser + proxy |
| Directory brute-force | Hidden content | Gobuster, ffuf |
| JS analysis | API endpoints | LinkFinder, Katana |
| robots.txt/sitemap | Disclosed paths | cURL |
| Source code review | All endpoints | IDE/grep |

---

## ❓ Revision Questions

1. Why is automated spidering insufficient for complete application mapping?
2. What entry points should a penetration tester document?
3. How does robots.txt help in reconnaissance?
4. What information should be recorded during manual application mapping?
5. How do you identify hidden API endpoints that aren't linked from the UI?
6. Why should application mapping be performed with different user roles?

---

*Next: [02-technology-fingerprinting.md](02-technology-fingerprinting.md)*

---

*[Back to README](../README.md)*
