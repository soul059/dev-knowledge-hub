# Application Mapping

## Unit 8 - Topic 5 | Active Reconnaissance

---

## Overview

**Application mapping** creates a detailed blueprint of a target web application — its pages, parameters, functionality, authentication mechanisms, and data flows. This is the bridge between reconnaissance and targeted exploitation, providing the attack surface inventory for web application testing.

---

## 1. Application Mapping Components

```
APPLICATION MAP STRUCTURE:

ENTRY POINTS:
├── Login pages
├── Registration forms
├── Search functionality
├── Contact forms
├── File upload
├── API endpoints
└── Password reset

PAGES AND FUNCTIONALITY:
├── Public pages (home, about, products)
├── Authenticated pages (dashboard, profile)
├── Admin pages (admin panel, settings)
├── API documentation (swagger, openapi)
└── Error pages (404, 500, debug)

PARAMETERS:
├── URL parameters (?id=1&page=2)
├── POST body parameters
├── Cookies
├── HTTP headers
├── JSON/XML payloads
└── Hidden form fields

AUTHENTICATION:
├── Login mechanism (form, SSO, OAuth)
├── Session management (cookies, tokens)
├── Authorization levels (user, admin)
├── MFA implementation
└── Password policy
```

---

## 2. Mapping Tools

```bash
# === BURP SUITE (Primary Tool) ===
# 1. Configure browser proxy → 127.0.0.1:8080
# 2. Browse the application manually
# 3. Burp Spider/Crawler maps all pages
# 4. Review Site Map for complete structure

# Burp Suite features for mapping:
# ├── Target → Site map      (visual app structure)
# ├── Spider/Crawler         (automated discovery)
# ├── Scanner                (active vulnerability testing)
# ├── Intruder               (parameter fuzzing)
# └── Repeater               (manual request editing)

# === ZAP (OWASP Zed Attack Proxy) ===
# Free alternative to Burp Suite
# 1. Configure proxy
# 2. Spider the application
# 3. Review Sites tree
# 4. Active scan for vulnerabilities

# === CRAWLING TOOLS ===
# Gospider — fast web spider
gospider -s https://target.com -d 3 -o output

# Hakrawler — web crawler for discovery
echo "https://target.com" | hakrawler -d 3

# Katana — fast crawler by ProjectDiscovery
katana -u https://target.com -d 5 -o crawl_results.txt
```

---

## 3. Manual Application Walkthrough

```
MANUAL MAPPING CHECKLIST:

1. AUTHENTICATION FLOW:
   ├── How does login work?
   ├── What parameters are sent?
   ├── How is the session maintained?
   ├── Is there a "remember me" option?
   ├── How does password reset work?
   └── Is registration open?

2. AUTHORIZATION TESTING:
   ├── What roles exist? (user, admin, etc.)
   ├── What can each role access?
   ├── Can you access admin pages as user?
   ├── Are API endpoints properly restricted?
   └── IDOR: Can user 1 access user 2's data?

3. INPUT POINTS:
   ├── All forms (search, contact, profile)
   ├── URL parameters
   ├── File upload functionality
   ├── Comment/review sections
   ├── API request bodies
   └── Each input = potential injection point

4. DATA FLOW:
   ├── Where does user data go?
   ├── Is data reflected in responses? (XSS?)
   ├── Is data used in queries? (SQLi?)
   ├── Is data stored and displayed later?
   └── Are there redirect parameters? (Open redirect?)
```

---

## 4. API Mapping

```bash
# === API ENDPOINT DISCOVERY ===

# From JavaScript analysis:
curl -s https://target.com/js/app.js | grep -oP '"/api/[^"]*"'

# From Swagger/OpenAPI:
curl https://target.com/swagger.json
curl https://target.com/api-docs
curl https://target.com/openapi.json

# From Burp Suite proxy — capture all API calls

# === API MAP EXAMPLE ===
# GET    /api/v1/users          → List users
# GET    /api/v1/users/{id}     → Get specific user
# POST   /api/v1/users          → Create user
# PUT    /api/v1/users/{id}     → Update user
# DELETE /api/v1/users/{id}     → Delete user
# POST   /api/v1/auth/login     → Login
# POST   /api/v1/auth/register  → Register
# GET    /api/v1/admin/settings → Admin settings
# POST   /api/v1/upload         → File upload

# === API TESTING CHECKLIST ===
# ├── Authentication required for each endpoint?
# ├── Authorization checks per endpoint?
# ├── Rate limiting implemented?
# ├── Input validation on all parameters?
# ├── Proper HTTP methods enforced?
# └── Sensitive data in responses?

# === POSTMAN / INSOMNIA ===
# Import Swagger spec → auto-generate API requests
# Test each endpoint systematically
```

---

## 5. Application Map Documentation

```
APPLICATION MAP: target.com

TECHNOLOGY STACK:
├── Frontend: React 18.2
├── Backend: Node.js + Express
├── Database: PostgreSQL
├── Auth: JWT tokens
└── CDN: Cloudflare

SITEMAP:
├── / (home)
├── /login
├── /register
├── /dashboard (auth required)
│   ├── /dashboard/profile
│   ├── /dashboard/settings
│   └── /dashboard/orders
├── /admin (admin role required)
│   ├── /admin/users
│   └── /admin/settings
├── /api/v1/... (REST API)
└── /uploads/ (user-uploaded files)

ATTACK SURFACE:
┌──────────────────┬───────────┬─────────────────────┐
│ Entry Point      │ Type      │ Potential Vuln       │
├──────────────────┼───────────┼─────────────────────┤
│ /login           │ POST form │ Brute force, SQLi    │
│ /register        │ POST form │ Account takeover     │
│ /search?q=       │ GET param │ XSS, SQLi            │
│ /api/v1/users/ID │ REST API  │ IDOR                 │
│ /uploads/        │ File      │ Unrestricted upload  │
│ /admin/          │ Auth page │ Auth bypass, privesc │
└──────────────────┴───────────┴─────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Application mapping** | Blueprint of app functionality |
| **Burp Suite** | Primary tool for web app mapping |
| **Entry points** | Every input = potential vulnerability |
| **API mapping** | Document all endpoints and methods |
| **Authorization** | Test access controls between roles |
| **Documentation** | Map the complete attack surface |

---

## Quick Revision Questions

1. **What is application mapping and why is it important?**
2. **What tools are used for web application mapping?**
3. **What should a manual application walkthrough cover?**
4. **How do you discover API endpoints?**
5. **What makes an entry point a potential vulnerability?**

---

[← Previous: Vulnerability Identification](04-vulnerability-identification.md)

---

*Unit 8 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Reconnaissance Automation →](../09-Reconnaissance-Automation/01-recon-frameworks.md)*
