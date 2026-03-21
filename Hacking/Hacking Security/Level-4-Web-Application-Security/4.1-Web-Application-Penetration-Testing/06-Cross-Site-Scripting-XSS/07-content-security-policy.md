# Unit 6: Cross-Site Scripting (XSS) — Topic 7: Content Security Policy

## Overview

Content Security Policy (CSP) is a powerful HTTP response header that provides a defence-in-depth layer against XSS and other code injection attacks. CSP allows developers to define a whitelist of trusted content sources, preventing browsers from executing inline scripts, loading external resources from unauthorised origins, or running `eval()`. While CSP is not a replacement for proper output encoding, it is a critical mitigation that limits the damage an XSS vulnerability can cause. This topic covers CSP directives, implementation strategies, bypass techniques, and evaluation tools.

---

## 1. CSP Fundamentals

### How CSP Works

```
Server Response:
  HTTP/1.1 200 OK
  Content-Security-Policy: default-src 'self'; script-src 'self'

Browser Behaviour:
  +----------------------------------------------------+
  | Inline <script>alert(1)</script>    →  BLOCKED     |
  | <script src="https://evil.com">     →  BLOCKED     |
  | <script src="/app.js">              →  ALLOWED     |
  | eval("alert(1)")                    →  BLOCKED     |
  | <img src="https://tracker.com">     →  BLOCKED     |
  | <img src="/logo.png">               →  ALLOWED     |
  +----------------------------------------------------+
```

### CSP Delivery Methods

```
Method 1: HTTP Header (recommended)
  Content-Security-Policy: directive1 value1; directive2 value2

Method 2: Meta Tag (limited — cannot use report-uri, frame-ancestors)
  <meta http-equiv="Content-Security-Policy"
        content="default-src 'self'; script-src 'self'">

Method 3: Report-Only (monitoring mode — does not enforce)
  Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report
```

---

## 2. CSP Directives in Detail

### Resource Loading Directives

| Directive            | Controls                                | Example                              |
|----------------------|-----------------------------------------|--------------------------------------|
| `default-src`        | Fallback for all resource types         | `default-src 'self'`                |
| `script-src`         | JavaScript sources                      | `script-src 'self' cdn.example.com` |
| `style-src`          | CSS stylesheets                         | `style-src 'self' 'unsafe-inline'`  |
| `img-src`            | Image sources                           | `img-src 'self' data: https:`       |
| `font-src`           | Web font sources                        | `font-src 'self' fonts.gstatic.com` |
| `connect-src`        | Fetch, XHR, WebSocket targets           | `connect-src 'self' api.example.com`|
| `media-src`          | Audio and video sources                 | `media-src 'self'`                  |
| `object-src`         | Plugins (Flash, Java applets)           | `object-src 'none'`                |
| `frame-src`          | Iframe sources                          | `frame-src 'self'`                  |
| `worker-src`         | Web Worker, Service Worker sources      | `worker-src 'self'`                |
| `child-src`          | Workers and frames (deprecated)         | Use `worker-src` + `frame-src`      |

### Navigation and Document Directives

| Directive            | Controls                                | Example                              |
|----------------------|-----------------------------------------|--------------------------------------|
| `frame-ancestors`    | Who can embed this page in iframe       | `frame-ancestors 'none'`           |
| `form-action`        | Valid form submission targets            | `form-action 'self'`               |
| `base-uri`           | Allowed values for `<base>` element     | `base-uri 'self'`                  |
| `navigate-to`        | URLs the page can navigate to           | `navigate-to 'self'`               |

### Special Values

```
'self'           → Same origin only
'none'           → Block all sources for this directive
'unsafe-inline'  → Allow inline scripts/styles (WEAKENS CSP)
'unsafe-eval'    → Allow eval(), Function(), setTimeout(string)
'strict-dynamic' → Trust scripts loaded by already-trusted scripts
data:            → Allow data: URIs
https:           → Allow any HTTPS source
*                → Allow any source (defeats CSP purpose)
```

---

## 3. Building an Effective CSP

### Starting Point: Strict CSP

```http
Content-Security-Policy:
  default-src 'none';
  script-src 'self';
  style-src 'self';
  img-src 'self';
  font-src 'self';
  connect-src 'self';
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  object-src 'none'
```

### CSP Implementation Workflow

```
+------------------------+
| 1. Deploy Report-Only  |   Monitor what would be blocked
+-----------+------------+
            |
            v
+------------------------+
| 2. Analyse Reports     |   Identify legitimate resources
+-----------+------------+   being flagged
            |
            v
+------------------------+
| 3. Whitelist Sources   |   Add trusted domains to directives
+-----------+------------+
            |
            v
+------------------------+
| 4. Switch to Enforce   |   Change from Report-Only to enforcing
+-----------+------------+
            |
            v
+------------------------+
| 5. Monitor & Iterate   |   Review ongoing reports, tighten policy
+------------------------+
```

### Report-Only Mode

```http
Content-Security-Policy-Report-Only:
  default-src 'self';
  script-src 'self';
  report-uri /csp-violations;
  report-to csp-endpoint

Report-To: {"group":"csp-endpoint","max_age":86400,
            "endpoints":[{"url":"/csp-violations"}]}
```

### Sample Violation Report (JSON)

```json
{
  "csp-report": {
    "document-uri": "https://example.com/page",
    "referrer": "",
    "violated-directive": "script-src 'self'",
    "effective-directive": "script-src",
    "original-policy": "default-src 'self'; script-src 'self'",
    "blocked-uri": "https://evil.com/malicious.js",
    "status-code": 200
  }
}
```

---

## 4. Nonce and Hash-Based CSP

### Nonce-Based CSP

A cryptographic nonce (number used once) allows specific inline scripts while blocking all others.

```http
Content-Security-Policy: script-src 'nonce-r4nd0mV4lu3'
```

```html
<!-- ALLOWED: nonce matches -->
<script nonce="r4nd0mV4lu3">
  console.log("This executes");
</script>

<!-- BLOCKED: no nonce or wrong nonce -->
<script>alert("XSS")</script>
<script nonce="wrong">alert("XSS")</script>
```

**Critical requirements:**
- Nonce must be cryptographically random (min 128 bits)
- Nonce must be regenerated on every page load
- Never reuse nonces across requests

### Hash-Based CSP

Instead of nonces, you can whitelist specific inline scripts by their hash.

```http
Content-Security-Policy: script-src 'sha256-abc123hashvalue...'
```

```bash
# Generate hash of inline script content
echo -n 'console.log("hello")' | openssl dgst -sha256 -binary | openssl base64
# Result: qznLcsROx4GACP2dm0UCKCzCG+HiZ1guq6ZZDob/Tng=

# CSP header
Content-Security-Policy: script-src 'sha256-qznLcsROx4GACP2dm0UCKCzCG+HiZ1guq6ZZDob/Tng='
```

### Nonce vs Hash Comparison

| Feature           | Nonce                          | Hash                            |
|--------------------|--------------------------------|---------------------------------|
| Dynamic scripts    | Yes — add nonce attribute      | No — hash changes with content  |
| Static scripts     | Yes                            | Yes — ideal use case            |
| Server requirement | Generate random nonce per page | Pre-compute hash at build time  |
| Maintenance        | Low                            | Update hash if script changes   |
| `strict-dynamic`   | Compatible                     | Compatible                      |

---

## 5. CSP Bypass Techniques

Understanding bypasses is essential for penetration testers evaluating CSP effectiveness.

### 5.1 JSONP Endpoints on Whitelisted Domains

```
CSP: script-src 'self' cdn.google.com

Bypass:
<script src="https://cdn.google.com/api?callback=alert(1)//"></script>

The JSONP endpoint returns: alert(1)//({...})
→ Executes alert(1) since the domain is whitelisted
```

### 5.2 AngularJS on Whitelisted CDN

```
CSP: script-src 'self' cdnjs.cloudflare.com

Bypass:
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.6.0/angular.min.js">
</script>
<div ng-app ng-csp>{{constructor.constructor('alert(1)')()}}</div>
```

### 5.3 Base URI Injection

```
CSP: script-src 'self'   (but no base-uri directive)

Bypass:
<base href="https://evil.com/">
<!-- Now relative script paths load from evil.com -->
<!-- <script src="/app.js"> loads https://evil.com/app.js -->
```

### 5.4 Unsafe-Inline Weakness

```
CSP: script-src 'self' 'unsafe-inline'

This CSP provides almost NO XSS protection because:
  <script>alert(1)</script>        → ALLOWED (unsafe-inline)
  <img src=x onerror=alert(1)>     → ALLOWED (unsafe-inline)
```

### Common CSP Weaknesses

```
+------------------------------+-----------------------------------+
| Weakness                     | Why It's Dangerous                |
+------------------------------+-----------------------------------+
| 'unsafe-inline'              | Allows all inline scripts         |
| 'unsafe-eval'                | Allows eval(), Function()         |
| Wildcard (*) in script-src   | Any domain can serve scripts      |
| data: in script-src          | Allows data:text/html payloads    |
| Missing object-src           | Flash-based XSS vectors           |
| Missing base-uri             | Base tag injection                |
| Overly broad CDN whitelists  | JSONP / Angular CSP bypasses      |
| No frame-ancestors           | Clickjacking possible             |
+------------------------------+-----------------------------------+
```

---

## 6. CSP Evaluation Tools

| Tool                          | Type          | Purpose                              |
|-------------------------------|---------------|--------------------------------------|
| **Google CSP Evaluator**      | Web tool      | Analyses CSP for weaknesses          |
| **csp-evaluator.withgoogle**  | Web tool      | Rates CSP directives                 |
| **Report URI**                | SaaS          | CSP report collection and analysis   |
| **Laboratory (Firefox ext.)** | Extension     | Auto-generates CSP from browsing     |
| **CSP Scanner (Burp ext.)**   | Burp plugin   | Evaluates CSP in pentest context     |
| **csp-bypass.com**            | Reference     | Known bypasses for common CSPs       |

### Using Google CSP Evaluator

```bash
# Visit: https://csp-evaluator.withgoogle.com/

# Paste your CSP:
default-src 'self'; script-src 'self' https://cdn.jsdelivr.net

# Result will flag:
# ⚠ cdn.jsdelivr.net hosts Angular which can bypass CSP
# ✓ No 'unsafe-inline'
# ✓ No 'unsafe-eval'
```

---

## Summary Table

| CSP Concept          | Key Detail                                              |
|----------------------|---------------------------------------------------------|
| Purpose              | Whitelist trusted content sources, block inline scripts |
| Primary directive    | `script-src` — most critical for XSS prevention        |
| Nonce-based CSP      | Per-request random token for inline script allowlisting |
| Hash-based CSP       | SHA-256/384/512 of inline script content                |
| `strict-dynamic`     | Trust propagation from nonced/hashed scripts            |
| Report-Only mode     | Monitor without blocking — use during rollout           |
| Common weakness      | `unsafe-inline` negates most XSS protection             |
| Bypass via JSONP     | Whitelisted domains with JSONP = CSP bypass             |
| Must-have directives | `object-src 'none'`, `base-uri 'self'`                 |
| Evaluation tool      | csp-evaluator.withgoogle.com                            |

---

## Revision Questions

1. **What is the difference between `Content-Security-Policy` and `Content-Security-Policy-Report-Only`? When would you use each?**

2. **Explain how nonce-based CSP prevents XSS while still allowing legitimate inline scripts. What requirements must the nonce meet?**

3. **A CSP contains `script-src 'self' 'unsafe-inline'`. Why does this provide almost no XSS protection? What should replace `'unsafe-inline'`?**

4. **Describe how an attacker can bypass CSP using a JSONP endpoint on a whitelisted domain. How can this be mitigated?**

5. **What is `strict-dynamic` and how does it simplify CSP deployment for applications that dynamically load scripts?**

6. **You are tasked with implementing CSP for a web application that uses Google Fonts, a CDN for jQuery, and has some inline event handlers. Design a CSP policy and explain the trade-offs.**

---

*Previous: [06-xss-prevention.md](06-xss-prevention.md)*

---

*[Back to README](../README.md)*
