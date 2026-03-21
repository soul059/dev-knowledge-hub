# HTTP/HTTPS

## Unit 5 - Topic 2 | Application Layer Protocols

---

## Overview

**HTTP (Hypertext Transfer Protocol)** is the foundation of web communication. **HTTPS** adds TLS encryption for security. Understanding HTTP methods, headers, status codes, and common web attacks is essential for web application security and penetration testing.

---

## 1. HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| **Port** | 80 | 443 |
| **Encryption** | ❌ None (cleartext) | ✅ TLS/SSL encrypted |
| **Authentication** | ❌ No server verification | ✅ Certificate validates server |
| **Integrity** | ❌ Data can be modified in transit | ✅ Tamper-proof |
| **Use** | Never for sensitive data | All modern web traffic |

```
HTTP (Cleartext):
Client ──── "GET /login password=secret" ──── Server
                   ↑ Attacker can READ this

HTTPS (Encrypted):
Client ──── [TLS Encrypted Tunnel] ──── Server
                   ↑ Attacker sees only encrypted gibberish
```

---

## 2. HTTP Methods

| Method | Purpose | Security Notes |
|--------|---------|---------------|
| **GET** | Retrieve data | Parameters in URL (visible in logs, history) |
| **POST** | Submit data | Parameters in body (more secure for sensitive data) |
| **PUT** | Upload/replace resource | Can overwrite files if enabled! |
| **DELETE** | Delete resource | Dangerous if unauthorized |
| **PATCH** | Partial update | Same risks as PUT |
| **HEAD** | Get headers only | Recon — check server type/version |
| **OPTIONS** | Show allowed methods | Recon — discover attack surface |
| **TRACE** | Echo request back | XST attack — should be disabled |

```bash
# Check allowed methods
curl -X OPTIONS https://target.com -i

# Banner grabbing
curl -I https://target.com         # HEAD request — shows server info
```

---

## 3. Important HTTP Headers (Security)

| Header | Purpose | Security Use |
|--------|---------|-------------|
| **Content-Security-Policy** | Controls resource loading | Prevents XSS |
| **Strict-Transport-Security** | Forces HTTPS | Prevents SSL stripping |
| **X-Frame-Options** | Controls iframe embedding | Prevents clickjacking |
| **X-Content-Type-Options** | Prevents MIME sniffing | Set to `nosniff` |
| **X-XSS-Protection** | Browser XSS filter | Enable for older browsers |
| **Set-Cookie** | Session management | Use HttpOnly, Secure, SameSite flags |
| **Server** | Reveals server software | Remove/obfuscate (information disclosure) |
| **X-Powered-By** | Reveals technology stack | Remove (information disclosure) |

### Secure Cookie Flags

```
Set-Cookie: session=abc123; 
  HttpOnly;       ← JavaScript can't access (prevents XSS theft)
  Secure;         ← Only sent over HTTPS
  SameSite=Strict; ← Prevents CSRF
  Path=/;
  Max-Age=3600
```

---

## 4. HTTP Status Codes (Security Relevant)

| Code | Meaning | Security Relevance |
|:----:|---------|-------------------|
| **200** | OK | Normal response |
| **301/302** | Redirect | Open redirect vulnerability |
| **400** | Bad Request | Input validation |
| **401** | Unauthorized | Authentication required |
| **403** | Forbidden | Access denied (resource exists!) |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Method restriction |
| **500** | Internal Server Error | May leak stack traces |
| **503** | Service Unavailable | DoS indicator |

---

## 5. Common HTTP/Web Attacks

| Attack | Description | Defense |
|--------|-------------|---------|
| **SSL Stripping** | Downgrade HTTPS → HTTP | HSTS header |
| **MITM** | Intercept HTTP traffic | Use HTTPS everywhere |
| **Session Hijacking** | Steal session cookies | HttpOnly, Secure cookies |
| **Clickjacking** | Invisible iframe overlay | X-Frame-Options: DENY |
| **CORS Misconfiguration** | Overly permissive cross-origin | Strict CORS policy |
| **HTTP Request Smuggling** | Ambiguous request parsing | Normalize HTTP parsing |

---

## 6. TLS/SSL Versions

| Version | Status | Notes |
|---------|--------|-------|
| SSL 2.0 | ❌ Broken | Never use |
| SSL 3.0 | ❌ Broken (POODLE) | Never use |
| TLS 1.0 | ❌ Deprecated | PCI DSS prohibits |
| TLS 1.1 | ❌ Deprecated | Browsers dropping support |
| TLS 1.2 | ✅ Secure | Widely supported, still standard |
| TLS 1.3 | ✅ Best | Faster, more secure, modern default |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **HTTP** | Cleartext web protocol — never for sensitive data |
| **HTTPS** | HTTP + TLS encryption — standard for all web traffic |
| **Security Headers** | CSP, HSTS, X-Frame-Options — must be configured |
| **Cookies** | HttpOnly + Secure + SameSite flags required |
| **TLS Version** | Use TLS 1.2+ only, disable older versions |

---

## Quick Revision Questions

1. **Why should HTTP never be used for sensitive data?**
2. **What security headers should every web application set?**
3. **What are the three important cookie security flags?**
4. **What is SSL stripping and how does HSTS prevent it?**
5. **What is the difference between TLS 1.2 and TLS 1.3?**
6. **What information can an attacker learn from HTTP HEAD requests?**

---

[← Previous: DNS & Attacks](01-dns-and-attacks.md) | [Next: SMTP/IMAP/POP3 →](03-smtp-imap-pop3.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
