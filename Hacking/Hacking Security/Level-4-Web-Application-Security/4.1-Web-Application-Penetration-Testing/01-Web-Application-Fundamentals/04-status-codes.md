# Status Codes

## Unit 1: Web Application Fundamentals — Topic 4

## 🎯 Overview

HTTP status codes are three-digit numbers returned by the server indicating the result of a request. For penetration testers, status codes reveal application behavior, access control logic, error handling patterns, and potential attack vectors. Understanding these codes is essential for interpreting automated scan results and crafting targeted exploits.

---

## 1. Status Code Categories

```
┌──────────────────────────────────────────────────────────┐
│                 HTTP STATUS CODE CLASSES                   │
├──────┬───────────────────────────────────────────────────┤
│ 1xx  │  Informational — Request received, continuing      │
│ 2xx  │  Success — Request accepted and processed          │
│ 3xx  │  Redirection — Further action needed               │
│ 4xx  │  Client Error — Bad request from client            │
│ 5xx  │  Server Error — Server failed to process           │
└──────┴───────────────────────────────────────────────────┘
```

---

## 2. Key Status Codes for Pentesters

### 2xx — Success Codes

| Code | Name | Security Relevance |
|------|------|-------------------|
| 200 | OK | Request succeeded — standard response |
| 201 | Created | Resource created — confirm object creation |
| 204 | No Content | Success but no body — common for DELETE |
| 206 | Partial Content | Range requests — potential for data extraction |

### 3xx — Redirection Codes

| Code | Name | Security Relevance |
|------|------|-------------------|
| 301 | Moved Permanently | Permanent redirect — check destination |
| 302 | Found | Temporary redirect — open redirect testing |
| 303 | See Other | POST→GET redirect — CSRF token handling |
| 307 | Temporary Redirect | Preserves method — different from 302 |
| 308 | Permanent Redirect | Preserves method — different from 301 |

```
┌──────────────────────────────────────────────────────────┐
│              OPEN REDIRECT ATTACK (3xx)                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  GET /redirect?url=https://evil.com HTTP/1.1             │
│  Host: target.com                                        │
│                                                          │
│  HTTP/1.1 302 Found                                      │
│  Location: https://evil.com    ← Open redirect!          │
│                                                          │
│  Victim trusts target.com → clicks link → evil.com       │
└──────────────────────────────────────────────────────────┘
```

### 4xx — Client Error Codes

| Code | Name | Security Relevance |
|------|------|-------------------|
| 400 | Bad Request | Input validation — check error details |
| 401 | Unauthorized | Authentication required — not logged in |
| 403 | Forbidden | Authorized but no access — access control |
| 404 | Not Found | Resource doesn't exist — directory brute force |
| 405 | Method Not Allowed | Method restricted — try other methods |
| 413 | Payload Too Large | Size limit — DoS testing |
| 414 | URI Too Long | URL length limit — buffer overflow testing |
| 429 | Too Many Requests | Rate limiting — brute force protection |
| 431 | Request Header Fields Too Large | Header limit — header injection |

### 5xx — Server Error Codes

| Code | Name | Security Relevance |
|------|------|-------------------|
| 500 | Internal Server Error | Server crash — potential vulnerability |
| 501 | Not Implemented | Feature not available — method testing |
| 502 | Bad Gateway | Upstream error — backend architecture leak |
| 503 | Service Unavailable | Server overloaded — DoS indicator |

---

## 3. Status Code Analysis in Penetration Testing

### 401 vs 403 — Critical Difference

```
┌─────────────────────────────────────────────────────────────┐
│  401 Unauthorized                    403 Forbidden           │
│  ─────────────────                   ─────────────          │
│  "Who are you?"                      "I know who you are,   │
│                                       but you can't do this"│
│  Authentication issue                Authorization issue     │
│  → Need valid credentials            → Need different role   │
│  → Token missing/expired             → Access control bypass │
│                                                             │
│  Testing:                            Testing:                │
│  • Add/modify Auth header            • Try different methods │
│  • Try default credentials           • Parameter tampering   │
│  • Token manipulation                • Path traversal bypass │
└─────────────────────────────────────────────────────────────┘
```

### Directory Brute Force Response Analysis

```bash
# Different status codes reveal different things
ffuf -u https://target.com/FUZZ -w wordlist.txt \
  -mc 200,301,302,403 -fc 404

# Results interpretation:
# 200 → Accessible resource (check content)
# 301 → Directory exists (follow redirect)
# 302 → Auth redirect (login required)
# 403 → Exists but restricted (try bypass)
# 404 → Does not exist (skip)

# Filter by response size to find real results
ffuf -u https://target.com/FUZZ -w wordlist.txt \
  -fs 0 -mc all
```

### Error-Based Information Disclosure

```
┌─────────────────────────────────────────────────────────────┐
│  500 Internal Server Error — Information Leaks              │
│                                                             │
│  GET /api/user?id=1' HTTP/1.1                              │
│                                                             │
│  Response:                                                  │
│  {                                                          │
│    "error": "MySQLSyntaxError",                            │
│    "message": "You have an error in your SQL syntax...",    │
│    "query": "SELECT * FROM users WHERE id = '1''",         │
│    "file": "/var/www/api/models/user.py",                  │
│    "line": 42,                                             │
│    "stack": ["..."]                                         │
│  }                                                          │
│                                                             │
│  ⚠ Reveals: DB type, query, file paths, framework          │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Status Code Manipulation Techniques

### Bypass 403 Forbidden

```bash
# Path manipulation
curl https://target.com/admin          # 403
curl https://target.com/Admin          # 200? (case change)
curl https://target.com/admin/         # 200? (trailing slash)
curl https://target.com/admin/.        # 200? (dot)
curl https://target.com//admin         # 200? (double slash)
curl https://target.com/./admin        # 200? (path traversal)
curl https://target.com/admin%20       # 200? (URL encoding)
curl https://target.com/admin%09       # 200? (tab character)

# Header-based bypass
curl https://target.com/admin \
  -H "X-Original-URL: /admin"
curl https://target.com/admin \
  -H "X-Forwarded-For: 127.0.0.1"
curl https://target.com/admin \
  -H "X-Custom-IP-Authorization: 127.0.0.1"

# Method-based bypass
curl -X POST https://target.com/admin
curl -X PUT https://target.com/admin
```

### Rate Limit (429) Bypass

```bash
# IP rotation headers
curl https://target.com/login \
  -H "X-Forwarded-For: 10.0.0.$((RANDOM % 255))"
  
# Change request format
# URL-encoded → JSON body
# Add random parameters
curl https://target.com/login?nocache=$RANDOM

# Slow down requests
# Use delays between attempts
```

---

## 5. Automated Status Code Enumeration

```bash
# Scan for interesting status codes across paths
gobuster dir -u https://target.com -w common.txt \
  -s "200,204,301,302,307,401,403" -t 50

# Nuclei template for status code analysis
nuclei -u https://target.com -tags tech-detect

# Custom script: test all status codes
for path in admin panel dashboard config backup; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    "https://target.com/$path")
  echo "$path → $code"
done
```

---

## 📊 Summary Table

| Code Range | Meaning | Pentester Action |
|-----------|---------|-----------------|
| 200 | Success | Analyze content, test parameters |
| 301/302 | Redirect | Check destination for open redirect |
| 401 | Unauthenticated | Try credentials, tokens, defaults |
| 403 | Forbidden | Attempt bypass techniques |
| 404 | Not found | Useful for negative confirmation |
| 405 | Method not allowed | Try other HTTP methods |
| 429 | Rate limited | Rotate IPs, slow requests |
| 500 | Server error | Analyze error for info leak |
| 502/503 | Backend issue | Probe backend architecture |

---

## ❓ Revision Questions

1. What is the security difference between a 401 and 403 response?
2. How can a 500 error response aid in exploitation?
3. What techniques can bypass a 403 Forbidden response?
4. Why are 302 redirects important for security testing?
5. How do you use status codes to filter directory brute-force results?
6. What does a 429 status code indicate and how can it be bypassed?

---

*Previous: [03-http-methods.md](03-http-methods.md) | Next: [05-security-headers.md](05-security-headers.md)*

---

*[Back to README](../README.md)*
