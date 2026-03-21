# Request and Response Structure

## Unit 1: Web Application Fundamentals — Topic 2

## 🎯 Overview

Every web application interaction consists of HTTP requests from the client and responses from the server. Understanding the exact structure of these messages is critical for penetration testing — every web exploit involves crafting or manipulating requests and analyzing responses. This topic dissects each component in detail.

---

## 1. HTTP Request Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP REQUEST                              │
├─────────────────────────────────────────────────────────────┤
│  Request Line:   METHOD /path?query HTTP/version            │
├─────────────────────────────────────────────────────────────┤
│  Headers:        Host: example.com                          │
│                  User-Agent: Mozilla/5.0...                  │
│                  Accept: text/html                           │
│                  Cookie: session=abc123                       │
│                  Content-Type: application/json              │
│                  Authorization: Bearer eyJ...                │
├─────────────────────────────────────────────────────────────┤
│  Blank Line:     \r\n                                       │
├─────────────────────────────────────────────────────────────┤
│  Body:           {"username":"admin","password":"pass123"}   │
└─────────────────────────────────────────────────────────────┘
```

### Request Line Components

```
POST /api/login?redirect=/dashboard HTTP/1.1
 │      │           │                  │
 │      │           │                  └── HTTP Version
 │      │           └── Query String (parameters)
 │      └── Path (URI)
 └── Method (verb)
```

### Real Request Example

```http
POST /api/v2/login HTTP/1.1
Host: webapp.target.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0)
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Content-Length: 52
Origin: https://webapp.target.com
Referer: https://webapp.target.com/login
Cookie: _csrf=xYz123; lang=en
Connection: keep-alive

{"username":"admin","password":"P@ssw0rd!","mfa":""}
```

---

## 2. HTTP Response Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP RESPONSE                             │
├─────────────────────────────────────────────────────────────┤
│  Status Line:    HTTP/version StatusCode ReasonPhrase        │
├─────────────────────────────────────────────────────────────┤
│  Headers:        Content-Type: text/html                     │
│                  Set-Cookie: session=xyz789                   │
│                  X-Frame-Options: DENY                       │
│                  Content-Security-Policy: default-src 'self' │
│                  Server: nginx/1.18.0                        │
├─────────────────────────────────────────────────────────────┤
│  Blank Line:     \r\n                                       │
├─────────────────────────────────────────────────────────────┤
│  Body:           <html><body>...</body></html>              │
└─────────────────────────────────────────────────────────────┘
```

### Real Response Example

```http
HTTP/1.1 200 OK
Date: Mon, 15 Jan 2024 10:30:00 GMT
Server: nginx/1.18.0
Content-Type: application/json; charset=utf-8
Content-Length: 186
Set-Cookie: session=eyJhbGc...; Path=/; HttpOnly; Secure; SameSite=Strict
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains

{"status":"success","token":"eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4ifQ.abc","redirect":"/dashboard"}
```

---

## 3. Request/Response Flow Analysis

```
┌────────────┐                                    ┌────────────┐
│   Browser  │                                    │ Web Server │
│            │                                    │            │
│ 1. User    │  ─── GET /page HTTP/1.1 ─────────→ │ 2. Process │
│    clicks  │      Host: target.com              │    request │
│    link    │      Cookie: session=abc           │            │
│            │                                    │ 3. Query   │
│            │                                    │    database│
│            │                                    │            │
│ 5. Render  │ ←── HTTP/1.1 200 OK ─────────────  │ 4. Build   │
│    page    │     Content-Type: text/html        │    response│
│            │     <html>...</html>               │            │
└────────────┘                                    └────────────┘
```

---

## 4. Security-Critical Request Components

### Components Pentesters Focus On

| Component | Location | Security Relevance |
|-----------|----------|-------------------|
| Cookies | Header | Session hijacking, CSRF tokens |
| Authorization | Header | Token manipulation, bypass |
| Content-Type | Header | Injection type selection |
| Referer | Header | CSRF validation bypass |
| Origin | Header | CORS exploitation |
| User-Agent | Header | WAF bypass, client detection |
| X-Forwarded-For | Header | IP spoofing, ACL bypass |
| Query Parameters | URL | Injection points |
| POST Body | Body | Injection points |
| Hidden Fields | Body | Parameter tampering |

### Request Manipulation Techniques

```bash
# Modify request with cURL
curl -X POST https://target.com/api/login \
  -H "Content-Type: application/json" \
  -H "X-Forwarded-For: 127.0.0.1" \
  -H "Cookie: admin=true" \
  -d '{"user":"admin","pass":"test"}'

# Change User-Agent to bypass restrictions
curl -A "Googlebot/2.1" https://target.com/admin

# Add custom headers
curl -H "X-Custom-IP-Authorization: 127.0.0.1" \
     https://target.com/admin/panel
```

---

## 5. Response Analysis for Pentesters

### Information Leakage in Responses

```
HTTP/1.1 200 OK
Server: Apache/2.4.41 (Ubuntu)     ← Server version disclosed
X-Powered-By: PHP/7.4.3            ← Backend technology leaked
X-Debug-Token: abc123               ← Debug mode enabled
X-AspNet-Version: 4.0.30319        ← Framework version exposed

# Check response headers
curl -I https://target.com
curl -v https://target.com 2>&1 | grep -i "< "
```

### Error Response Analysis

```
┌─────────────────────────────────────────────────────┐
│           Error Response Information Leak            │
├─────────────────────────────────────────────────────┤
│  HTTP/1.1 500 Internal Server Error                 │
│                                                     │
│  Stack Trace:                                       │
│  at MySQL.query("SELECT * FROM users               │
│     WHERE id = '1' OR '1'='1'")                    │
│  at /var/www/app/controllers/user.js:42             │
│  at Database.connection(/var/www/app/db.js:15)      │
│                                                     │
│  ⚠ Reveals: DB type, query, file paths, line #s    │
└─────────────────────────────────────────────────────┘
```

---

## 6. Content Types and Encoding

| Content-Type | Use Case | Attack Surface |
|-------------|----------|----------------|
| `application/x-www-form-urlencoded` | Form submissions | SQL injection, XSS |
| `application/json` | API calls | JSON injection |
| `multipart/form-data` | File uploads | File upload attacks |
| `text/xml` / `application/xml` | XML data | XXE, XML injection |
| `text/html` | HTML content | XSS, HTML injection |

### Encoding Manipulation

```bash
# URL encoding
# Space → %20 or +
# < → %3C, > → %3E
# ' → %27, " → %22

# Double URL encoding (WAF bypass)
# < → %3C → %253C
# ' → %27 → %2527

# Unicode encoding
# < → \u003c
# ' → \u0027

# Example: Bypassing input filters
# Normal:  <script>alert(1)</script>
# Encoded: %3Cscript%3Ealert(1)%3C/script%3E
# Double:  %253Cscript%253Ealert(1)%253C/script%253E
```

---

## 📊 Summary Table

| Component | Request | Response |
|-----------|---------|----------|
| First Line | Method + Path + Version | Version + Status Code + Reason |
| Headers | Host, Cookie, Auth, Content-Type | Set-Cookie, Server, Security headers |
| Body | Form data, JSON, XML, file | HTML, JSON, XML, binary |
| Separator | Blank line (\r\n) | Blank line (\r\n) |
| Key for Testing | Injection points | Information disclosure |

---

## ❓ Revision Questions

1. What are the four main components of an HTTP request?
2. How can response headers reveal sensitive server information?
3. What is the significance of Content-Type in attack selection?
4. How does double URL encoding help bypass WAFs?
5. Which request headers are commonly manipulated during penetration testing?
6. What information can be extracted from verbose error responses?

---

*Previous: [01-http-https-protocol.md](01-http-https-protocol.md) | Next: [03-http-methods.md](03-http-methods.md)*

---

*[Back to README](../README.md)*
