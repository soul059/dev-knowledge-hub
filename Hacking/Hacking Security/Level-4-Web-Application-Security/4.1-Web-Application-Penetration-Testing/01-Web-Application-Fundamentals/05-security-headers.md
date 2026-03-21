# Headers (Security-Relevant)

## Unit 1: Web Application Fundamentals — Topic 5

## 🎯 Overview

HTTP headers control how requests and responses are handled by clients and servers. Security headers defend against common attacks like XSS, clickjacking, and MitM. For penetration testers, missing or misconfigured headers represent easy wins, while custom headers can reveal architecture details and bypass opportunities.

---

## 1. Security Response Headers

### Essential Security Headers

```
┌──────────────────────────────────────────────────────────────┐
│              SECURITY HEADERS CHECKLIST                       │
├──────────────────────────────────────────────────────────────┤
│  ✓ Strict-Transport-Security (HSTS)                         │
│  ✓ Content-Security-Policy (CSP)                            │
│  ✓ X-Content-Type-Options                                   │
│  ✓ X-Frame-Options                                          │
│  ✓ X-XSS-Protection (legacy)                               │
│  ✓ Referrer-Policy                                          │
│  ✓ Permissions-Policy                                       │
│  ✓ Cache-Control                                            │
│  ✓ Set-Cookie (with Secure, HttpOnly, SameSite)             │
└──────────────────────────────────────────────────────────────┘
```

### Header Details

| Header | Purpose | Recommended Value |
|--------|---------|-------------------|
| `Strict-Transport-Security` | Force HTTPS | `max-age=31536000; includeSubDomains; preload` |
| `Content-Security-Policy` | Control resource loading | `default-src 'self'; script-src 'self'` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `X-Frame-Options` | Prevent clickjacking | `DENY` or `SAMEORIGIN` |
| `X-XSS-Protection` | Legacy XSS filter | `0` (disable — rely on CSP) |
| `Referrer-Policy` | Control Referer header | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Restrict browser features | `camera=(), microphone=(), geolocation=()` |

---

## 2. Strict-Transport-Security (HSTS)

```
Without HSTS:
┌────────┐  HTTP (80)   ┌──────────┐  HTTPS (443)  ┌────────┐
│ Client │──────────────→│ Attacker │───────────────→│ Server │
│        │←──────────────│ SSLStrip │←───────────────│        │
└────────┘  Plaintext    └──────────┘                └────────┘

With HSTS:
┌────────┐  HTTPS (443) directly  ┌────────┐
│ Client │════════════════════════→│ Server │
│        │  Browser enforces TLS  │        │
└────────┘  No downgrade possible └────────┘
```

```bash
# Check HSTS header
curl -sI https://target.com | grep -i strict

# Missing HSTS = vulnerable to SSLStrip
# Bad config: max-age=0 (disables HSTS)
# Weak config: max-age too short, missing includeSubDomains
```

---

## 3. Content-Security-Policy (CSP)

```
┌──────────────────────────────────────────────────────────────┐
│                    CSP DIRECTIVES                             │
├──────────────────────────────────────────────────────────────┤
│  default-src  → Fallback for all resource types              │
│  script-src   → JavaScript sources                           │
│  style-src    → CSS sources                                  │
│  img-src      → Image sources                                │
│  connect-src  → AJAX, WebSocket, fetch                       │
│  font-src     → Font file sources                            │
│  frame-src    → iframe sources                               │
│  object-src   → Plugins (Flash, Java)                        │
│  base-uri     → <base> element URLs                          │
│  form-action  → Form submission targets                      │
│  report-uri   → Violation report destination                 │
└──────────────────────────────────────────────────────────────┘
```

### CSP Bypass Techniques

```bash
# Weak CSP: script-src 'unsafe-inline'
# → Inline scripts allowed = XSS still works!

# Weak CSP: script-src 'unsafe-eval'  
# → eval() allowed = DOM XSS possible

# Weak CSP: script-src *.google.com
# → JSONP endpoints on google.com can be abused

# Weak CSP: script-src 'nonce-abc123' (predictable)
# → If nonce is guessable, XSS bypass possible

# CSP report-only mode (no enforcement)
# Content-Security-Policy-Report-Only: ...
# → Only reports violations, doesn't block!

# Analyze CSP
curl -sI https://target.com | grep -i content-security
# Use: https://csp-evaluator.withgoogle.com/
```

---

## 4. Information Disclosure Headers

### Server & Technology Headers

```http
# Headers that leak information
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/8.1.2
X-AspNet-Version: 4.0.30319
X-Generator: WordPress 6.4
X-Drupal-Cache: HIT
Via: 1.1 varnish (Varnish/6.6)
X-Varnish: 12345678
X-Cache: HIT from cloudfront
```

```bash
# Enumerate information headers
curl -sI https://target.com | grep -iE \
  "^(server|x-powered|x-aspnet|x-generator|via|x-cache|x-varnish)"

# Nmap HTTP header scan
nmap --script http-headers -p 80,443 target.com
```

---

## 5. Request Headers for Attacks

### Headers Used in Bypass Attacks

```bash
# IP-based access control bypass
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1

# URL rewrite bypass
X-Original-URL: /admin
X-Rewrite-URL: /admin

# Host header attacks
Host: evil.com           # Host header injection
Host: target.com         
X-Forwarded-Host: evil.com  # Password reset poisoning
```

### CORS Headers

```http
# Request
Origin: https://evil.com

# Vulnerable Response (reflects any origin)
Access-Control-Allow-Origin: https://evil.com
Access-Control-Allow-Credentials: true

# This allows evil.com to read authenticated responses!
```

```bash
# Test CORS configuration
curl -sI https://target.com/api/data \
  -H "Origin: https://evil.com" | grep -i access-control

# Check for wildcard with credentials
# Access-Control-Allow-Origin: *
# Access-Control-Allow-Credentials: true  ← Dangerous combo!
```

---

## 6. Cookie Security Attributes

```
Set-Cookie: session=abc123; Path=/; Domain=.target.com;
            Secure; HttpOnly; SameSite=Strict; Max-Age=3600

┌──────────────────────────────────────────────────────────────┐
│  Attribute     │  Purpose              │  Missing = Risk      │
├────────────────┼───────────────────────┼─────────────────────┤
│  Secure        │  HTTPS only           │  Cookie sent via HTTP│
│  HttpOnly      │  No JS access         │  XSS can steal cookie│
│  SameSite      │  Cross-site control   │  CSRF possible       │
│  Path          │  URL scope            │  Broad cookie scope  │
│  Domain        │  Domain scope         │  Cookie leaks to subs│
│  Max-Age       │  Expiration           │  Persistent sessions │
└────────────────┴───────────────────────┴─────────────────────┘
```

---

## 7. Header Testing Methodology

```bash
# 1. Check all security headers
curl -sI https://target.com | head -30

# 2. Use security header scanner
# securityheaders.com — online scanner
# Nmap script
nmap --script http-security-headers -p 443 target.com

# 3. Check for missing headers
# Missing HSTS → SSLStrip
# Missing CSP → XSS easier
# Missing X-Frame-Options → Clickjacking
# Missing X-Content-Type-Options → MIME confusion

# 4. Test header injection
curl "https://target.com/page?param=value%0d%0aInjected-Header:evil"
# If response contains "Injected-Header: evil" → CRLF injection!
```

---

## 📊 Summary Table

| Header | Missing Impact | Attacker Benefit |
|--------|---------------|-----------------|
| HSTS | SSL downgrade | SSLStrip, MitM |
| CSP | No script control | Easier XSS exploitation |
| X-Frame-Options | Clickjacking | UI redressing attacks |
| X-Content-Type-Options | MIME sniffing | Content-type confusion |
| Referrer-Policy | URL leakage | Token theft via Referer |
| Cookie: Secure | HTTP cookie leak | Session hijacking |
| Cookie: HttpOnly | JS cookie access | XSS cookie theft |
| Cookie: SameSite | Cross-site cookies | CSRF attacks |

---

## ❓ Revision Questions

1. What is HSTS and how does it prevent SSL stripping?
2. How can a weak CSP be exploited for XSS attacks?
3. What information can Server and X-Powered-By headers reveal?
4. How do X-Forwarded-For headers enable access control bypass?
5. What are the three SameSite cookie values and their security implications?
6. How does CORS misconfiguration enable cross-origin data theft?

---

*Previous: [04-status-codes.md](04-status-codes.md) | Next: [06-cookies-and-sessions.md](06-cookies-and-sessions.md)*

---

*[Back to README](../README.md)*
