# Unit 5: A05 - Security Misconfiguration — Topic 4: Security Headers

## Overview

HTTP security headers instruct browsers to enable **built-in security mechanisms** that protect against common attacks like XSS, clickjacking, MIME sniffing, and protocol downgrade attacks. Missing security headers is one of the most prevalent misconfigurations — they are easy to add, have minimal performance impact, and provide significant protection as a defense-in-depth layer.

---

## 1. Essential Security Headers

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURITY HEADERS OVERVIEW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MUST HAVE (Critical):                                         │
│  ├── Content-Security-Policy (CSP)                             │
│  ├── Strict-Transport-Security (HSTS)                          │
│  ├── X-Content-Type-Options                                    │
│  └── X-Frame-Options                                           │
│                                                                 │
│  SHOULD HAVE (Recommended):                                    │
│  ├── Referrer-Policy                                           │
│  ├── Permissions-Policy                                        │
│  └── Cache-Control (for sensitive pages)                       │
│                                                                 │
│  SHOULD REMOVE:                                                │
│  ├── X-Powered-By                                              │
│  ├── Server (detailed version)                                 │
│  └── X-AspNet-Version                                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Content-Security-Policy (CSP)

```
PURPOSE: Controls which resources browsers can load
PREVENTS: XSS, data injection, clickjacking

Directives:
┌──────────────────────┬───────────────────────────────────────────┐
│ Directive            │ Controls                                  │
├──────────────────────┼───────────────────────────────────────────┤
│ default-src          │ Fallback for all resource types           │
│ script-src           │ JavaScript sources                        │
│ style-src            │ CSS sources                               │
│ img-src              │ Image sources                             │
│ font-src             │ Font sources                              │
│ connect-src          │ AJAX, WebSocket, fetch                    │
│ media-src            │ Audio/video sources                       │
│ object-src           │ Plugins (Flash, Java)                     │
│ frame-src            │ iframes                                   │
│ frame-ancestors      │ Who can frame this page                   │
│ base-uri             │ Restricts <base> tag                      │
│ form-action          │ Restricts form submission targets         │
│ upgrade-insecure-req │ Upgrades HTTP to HTTPS                    │
│ report-uri / -to     │ Where to send violation reports           │
└──────────────────────┴───────────────────────────────────────────┘

Values:
'self'          — Same origin only
'none'          — Block all
'unsafe-inline' — Allow inline scripts/styles (AVOID)
'unsafe-eval'   — Allow eval() (AVOID)
'nonce-abc123'  — Allow specific tagged elements
'sha256-...'    — Allow specific script hash
https:          — Only HTTPS sources
data:           — Allow data: URIs
*.example.com   — Specific domain wildcard
```

### CSP Examples

```
# Strict CSP (recommended)
Content-Security-Policy: 
    default-src 'none'; 
    script-src 'self' 'nonce-random123'; 
    style-src 'self'; 
    img-src 'self' data:; 
    font-src 'self'; 
    connect-src 'self' https://api.example.com; 
    frame-ancestors 'none'; 
    base-uri 'self'; 
    form-action 'self';
    upgrade-insecure-requests;
    report-uri /csp-report

# CSP with nonce (prevents inline script execution)
<!-- Server generates random nonce for each response -->
<script nonce="random123">
    // This script runs because nonce matches
    console.log("Allowed");
</script>

<script>
    // This script is BLOCKED — no matching nonce
    alert("XSS"); 
</script>

# Report-Only mode (monitor before enforcing)
Content-Security-Policy-Report-Only: 
    default-src 'self'; 
    report-uri /csp-report
```

---

## 3. Strict-Transport-Security (HSTS)

```
PURPOSE: Forces HTTPS connections
PREVENTS: Protocol downgrade attacks, SSL stripping, cookie hijacking

Header:
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

Parameters:
┌──────────────────────┬────────────────────────────────────────────┐
│ Parameter            │ Purpose                                    │
├──────────────────────┼────────────────────────────────────────────┤
│ max-age=31536000     │ Remember for 1 year (seconds)             │
│ includeSubDomains    │ Apply to all subdomains                    │
│ preload              │ Eligible for browser preload lists         │
└──────────────────────┴────────────────────────────────────────────┘

How HSTS works:
1. User visits https://example.com
2. Server responds with HSTS header
3. Browser remembers for max-age seconds
4. ALL future requests to example.com → automatically HTTPS
5. User types http://example.com → browser upgrades to HTTPS
6. Invalid certificate → HARD FAIL (no user override)

HSTS Preload:
→ Submit to https://hstspreload.org
→ Browser ships with your domain in its preload list
→ HTTPS enforced from the very first visit
→ Cannot be bypassed even on first connection
```

---

## 4. X-Content-Type-Options

```
PURPOSE: Prevents MIME type sniffing
PREVENTS: Attacks where browsers interpret files as different types

Header:
X-Content-Type-Options: nosniff

Attack scenario without this header:
1. Attacker uploads malicious.html renamed as image.jpg
2. Server sends it with Content-Type: image/jpeg
3. Browser "sniffs" the content and detects HTML
4. Browser renders it as HTML → JavaScript executes → XSS!

With nosniff:
→ Browser strictly follows Content-Type header
→ image/jpeg file is NEVER executed as HTML/JS
```

---

## 5. X-Frame-Options

```
PURPOSE: Controls whether page can be loaded in iframes
PREVENTS: Clickjacking attacks

Values:
┌──────────────────┬────────────────────────────────────────────────┐
│ Value            │ Effect                                         │
├──────────────────┼────────────────────────────────────────────────┤
│ DENY             │ Cannot be framed by anyone                     │
│ SAMEORIGIN       │ Can only be framed by same origin              │
│ ALLOW-FROM uri   │ Can be framed by specified URI (deprecated)    │
└──────────────────┴────────────────────────────────────────────────┘

Header:
X-Frame-Options: DENY

Modern replacement (CSP):
Content-Security-Policy: frame-ancestors 'none'     → Same as DENY
Content-Security-Policy: frame-ancestors 'self'     → Same as SAMEORIGIN
Content-Security-Policy: frame-ancestors example.com → Specific origin
```

---

## 6. Other Important Headers

### Referrer-Policy

```
Referrer-Policy: strict-origin-when-cross-origin

Values:
no-referrer                 → Never send Referer header
no-referrer-when-downgrade  → Don't send on HTTPS→HTTP
origin                      → Send only origin (no path)
origin-when-cross-origin    → Full URL same-origin, origin cross-origin
same-origin                 → Only send to same origin
strict-origin               → Origin only, nothing on downgrade
strict-origin-when-cross-origin → Full URL same-origin, origin cross, none on downgrade
unsafe-url                  → Always send full URL (AVOID)
```

### Permissions-Policy

```
# Restrict browser features
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()

# Allow only for self
Permissions-Policy: camera=(self), fullscreen=(self)

# Allow for specific origin
Permissions-Policy: payment=(self "https://payment.example.com")
```

### Cache-Control (for sensitive pages)

```
# Prevent caching of sensitive data
Cache-Control: no-store, no-cache, must-revalidate, private
Pragma: no-cache
Expires: 0

Use on:
- Authentication pages
- Pages with personal data
- Admin pages
- API responses with sensitive data
```

---

## 7. Implementation Examples

### Nginx

```nginx
# Add security headers
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; frame-ancestors 'none'" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

# Remove revealing headers
proxy_hide_header X-Powered-By;
server_tokens off;
```

### Apache

```apache
# Security headers
Header always set Content-Security-Policy "default-src 'self'"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "DENY"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "camera=(), microphone=()"

# Remove revealing headers
Header unset X-Powered-By
ServerTokens Prod
ServerSignature Off
```

### Express.js (Helmet)

```javascript
const helmet = require('helmet');

app.use(helmet());  // Applies sensible defaults

// Or configure individually:
app.use(helmet.contentSecurityPolicy({
    directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'"],
        styleSrc: ["'self'"],
        imgSrc: ["'self'", "data:"],
        frameAncestors: ["'none'"]
    }
}));
app.use(helmet.hsts({ maxAge: 31536000, includeSubDomains: true }));
app.use(helmet.noSniff());
app.use(helmet.frameguard({ action: 'deny' }));
app.use(helmet.referrerPolicy({ policy: 'strict-origin-when-cross-origin' }));
```

---

## 8. Testing Security Headers

```bash
# Check headers with curl
curl -sI https://example.com | grep -i "security\|csp\|strict\|x-frame\|x-content\|referrer\|permissions\|powered\|server"

# Online scanners
# https://securityheaders.com
# https://observatory.mozilla.org
# https://csp-evaluator.withgoogle.com

# Nmap script
nmap --script http-security-headers -p 443 example.com
```

### Security Headers Scoring

| Grade | Criteria |
|-------|----------|
| **A+** | All essential headers present + HSTS preload |
| **A** | All essential headers present |
| **B** | Most headers present, minor gaps |
| **C** | Some headers present |
| **D** | Few headers present |
| **F** | No security headers |

---

## Summary Table

| Header | Prevents | Required Value |
|--------|----------|---------------|
| **CSP** | XSS, injection | `default-src 'self'` minimum |
| **HSTS** | Downgrade, stripping | `max-age=31536000; includeSubDomains` |
| **X-Content-Type-Options** | MIME sniffing | `nosniff` |
| **X-Frame-Options** | Clickjacking | `DENY` |
| **Referrer-Policy** | URL leakage | `strict-origin-when-cross-origin` |
| **Permissions-Policy** | Feature abuse | Deny unused features |

---

## Revision Questions

1. List all essential security headers and explain what each prevents.
2. Write a Content-Security-Policy that allows scripts from 'self' with nonce, styles from 'self', and blocks framing.
3. Explain how HSTS preload works and why it's more secure than regular HSTS.
4. What is MIME type sniffing and how does X-Content-Type-Options prevent it?
5. Implement all security headers in Nginx, Apache, and Express.js.
6. How would you test a website's security headers? What grade would you consider acceptable?

---

*Previous: [03-error-handling.md](03-error-handling.md) | Next: [05-cloud-misconfigurations.md](05-cloud-misconfigurations.md)*

---

*[Back to README](../README.md)*
