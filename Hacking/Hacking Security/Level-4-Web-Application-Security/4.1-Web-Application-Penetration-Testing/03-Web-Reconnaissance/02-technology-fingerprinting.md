# Technology Fingerprinting

## Unit 3: Web Reconnaissance — Topic 2

## 🎯 Overview

Technology fingerprinting identifies the software stack powering a web application — web server, framework, CMS, libraries, and third-party services. Knowing the technology stack enables targeted vulnerability research, exploit selection, and default credential testing. This is one of the first steps in web application reconnaissance.

---

## 1. Fingerprinting Methodology

```
┌──────────────────────────────────────────────────────────────┐
│              FINGERPRINTING LAYERS                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: Web Server    │  Apache, nginx, IIS, LiteSpeed    │
│  Layer 2: Framework     │  Django, Rails, Spring, Express   │
│  Layer 3: CMS           │  WordPress, Drupal, Joomla        │
│  Layer 4: Language      │  PHP, Python, Java, Node.js       │
│  Layer 5: Libraries     │  jQuery, React, Bootstrap         │
│  Layer 6: Database      │  MySQL, PostgreSQL, MongoDB       │
│  Layer 7: CDN/WAF       │  Cloudflare, Akamai, AWS WAF     │
│  Layer 8: OS            │  Linux, Windows Server            │
│  Layer 9: Services      │  Analytics, payment, auth         │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Passive Fingerprinting

```bash
# HTTP response headers
curl -sI https://target.com | head -20

# Key headers to check:
# Server: nginx/1.18.0
# X-Powered-By: PHP/8.1
# X-AspNet-Version: 4.0.30319
# X-Generator: WordPress 6.4
# Set-Cookie: JSESSIONID=xxx (Java)
# Set-Cookie: PHPSESSID=xxx (PHP)
# Set-Cookie: ASP.NET_SessionId=xxx (.NET)
# Set-Cookie: connect.sid=xxx (Express)

# HTML meta tags and comments
curl -s https://target.com | grep -iE "(generator|powered|cms|framework)"
# <meta name="generator" content="WordPress 6.4.2" />

# JavaScript libraries (view source)
curl -s https://target.com | grep -oE 'jquery[.-][0-9.]+'
curl -s https://target.com | grep -oE '(react|angular|vue)[.-][0-9.]+'
```

---

## 3. Active Fingerprinting Tools

```bash
# WhatWeb — detailed fingerprinting
whatweb https://target.com
whatweb -v https://target.com    # Verbose output
whatweb -a 3 https://target.com  # Aggressive mode

# Wappalyzer (browser extension)
# Identifies frameworks, CMS, analytics, CDN, etc.
# Available as CLI: wappalyzer https://target.com

# Nmap HTTP fingerprinting
nmap -sV -p 80,443 target.com
nmap --script http-headers -p 80,443 target.com
nmap --script http-generator -p 80,443 target.com

# BuiltWith API
curl "https://api.builtwith.com/free1/api.json?KEY=xxx&LOOKUP=target.com"

# Netcraft
# https://sitereport.netcraft.com/?url=target.com
```

---

## 4. CMS Detection

```bash
# WordPress detection
curl -s https://target.com/wp-login.php -o /dev/null -w "%{http_code}"
curl -s https://target.com/wp-content/ -o /dev/null -w "%{http_code}"
curl -s https://target.com/wp-json/wp/v2/users  # User enumeration

# WPScan for WordPress
wpscan --url https://target.com --enumerate u,p,t

# Drupal detection
curl -s https://target.com/CHANGELOG.txt
curl -s https://target.com/core/CHANGELOG.txt
droopescan scan drupal -u https://target.com

# Joomla detection
curl -s https://target.com/administrator/
curl -s https://target.com/configuration.php -o /dev/null -w "%{http_code}"
joomscan -u https://target.com
```

### CMS Fingerprint Indicators

| CMS | Indicators |
|-----|-----------|
| WordPress | `/wp-content/`, `/wp-admin/`, `wp-json`, meta generator |
| Drupal | `/core/`, `/sites/default/`, `X-Drupal-Cache` header |
| Joomla | `/administrator/`, `/components/`, meta generator |
| Magento | `/skin/frontend/`, `/downloader/`, cookies |
| Shopify | `cdn.shopify.com`, `myshopify.com` |
| Ghost | `/ghost/`, `X-Ghost-Version` header |

---

## 5. Framework Fingerprinting

```bash
# Error page analysis
curl https://target.com/nonexistent-page-xyz

# Django: Yellow debug page or "Page not found (404)"
# Rails: "Routing Error" or "Action Controller Exception"
# Spring: "Whitelabel Error Page"
# Express: "Cannot GET /path"
# Laravel: Orange/red debug page (Ignition)
# ASP.NET: Detailed yellow screen of death

# Default files and paths
# Django: /admin/ (Django admin)
# Rails: /rails/info (development mode)
# Spring: /actuator/ (Spring Boot)
# Laravel: /telescope (debug tool)
# Express: /swagger or /api-docs
```

### Technology-Specific Paths

| Framework | Default Paths |
|-----------|--------------|
| Django | `/admin/`, `/static/`, `/__debug__/` |
| Rails | `/rails/info/`, `/assets/`, `/sidekiq/` |
| Spring | `/actuator/`, `/h2-console/`, `/swagger-ui/` |
| Laravel | `/telescope/`, `/horizon/`, `/_debugbar/` |
| Express | `/api-docs/`, `/swagger/`, `/graphql` |
| ASP.NET | `/elmah.axd`, `/trace.axd`, `/web.config` |

---

## 6. WAF Detection

```bash
# Identify Web Application Firewall
wafw00f https://target.com

# Manual WAF detection
# Send malicious payload and observe response
curl "https://target.com/?q=<script>alert(1)</script>"
# WAF responses: 403, custom block page, CAPTCHA

# Common WAF identifiers:
# Cloudflare: cf-ray header, __cfduid cookie
# AWS WAF: x-amzn-requestid header
# Akamai: AkamaiGHost header
# ModSecurity: Server: Apache + 403 on attacks
# Imperva: visid_incap cookie

# Bypass WAF for testing
# Change encoding, case, padding
# Use alternative syntax
# Test from different IPs
```

---

## 7. Fingerprinting Cheat Sheet

```bash
# One-liner comprehensive fingerprint
curl -sI https://target.com && \
  echo "---" && \
  curl -s https://target.com | grep -iE \
    "(generator|powered|jquery|react|angular|vue|bootstrap)" && \
  echo "---" && \
  for path in robots.txt sitemap.xml wp-login.php \
    administrator/ actuator/ admin/ .git/HEAD; do
    code=$(curl -s -o /dev/null -w "%{http_code}" \
      "https://target.com/$path")
    [ "$code" != "404" ] && echo "$path → $code"
  done
```

---

## 📊 Summary Table

| Method | Detects | Tools |
|--------|---------|-------|
| Response headers | Server, framework, language | cURL, Nmap |
| HTML analysis | CMS, libraries, version | View source, grep |
| Default paths | Framework, admin panels | Gobuster, manual |
| Error pages | Framework, language | Manual testing |
| Cookies | Technology stack | Browser, Burp |
| WAF detection | Security products | wafw00f, manual |

---

## ❓ Revision Questions

1. What HTTP headers commonly reveal the technology stack?
2. How do you identify a CMS from default files and paths?
3. What is the difference between passive and active fingerprinting?
4. How can error pages reveal the backend framework?
5. Why is WAF detection important before testing?
6. What tools automate technology fingerprinting?

---

*Previous: [01-application-mapping.md](01-application-mapping.md) | Next: [03-directory-enumeration.md](03-directory-enumeration.md)*

---

*[Back to README](../README.md)*
