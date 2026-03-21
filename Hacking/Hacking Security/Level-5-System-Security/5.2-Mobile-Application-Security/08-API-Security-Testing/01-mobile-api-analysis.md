# Unit 8: API Security Testing — Topic 1: Mobile API Analysis

## Overview

**Mobile API analysis** focuses on understanding and testing the backend APIs that mobile apps communicate with. Mobile apps are thick clients that expose API endpoints, authentication flows, data structures, and business logic through their network traffic. Analyzing these APIs reveals server-side vulnerabilities that affect all users, not just the mobile app.

---

## 1. API Discovery and Mapping

```
API DISCOVERY PROCESS:

1. TRAFFIC CAPTURE:
   → Configure proxy (Burp Suite)
   → Bypass SSL pinning if needed
   → Exercise ALL app functionality
   → Capture complete API surface

2. STATIC EXTRACTION:
   → Decompile app binary
   → Search for API endpoints in code
   → Extract from config files
   → Find hidden/debug endpoints

3. API MAP:
┌──────────────────────────────────────────────┐
│  MOBILE API MAP                              │
│                                              │
│  AUTH ENDPOINTS:                             │
│  POST /api/v1/auth/login                     │
│  POST /api/v1/auth/register                  │
│  POST /api/v1/auth/forgot-password           │
│  POST /api/v1/auth/refresh-token             │
│  DELETE /api/v1/auth/logout                  │
│                                              │
│  USER ENDPOINTS:                             │
│  GET  /api/v1/users/me                       │
│  PUT  /api/v1/users/me                       │
│  GET  /api/v1/users/{id}     ← IDOR?        │
│                                              │
│  DATA ENDPOINTS:                             │
│  GET  /api/v1/transactions                   │
│  POST /api/v1/transactions                   │
│  GET  /api/v1/transactions/{id}  ← IDOR?    │
│                                              │
│  ADMIN ENDPOINTS (hidden):                   │
│  GET  /api/v1/admin/users                    │
│  POST /api/v1/admin/config                   │
│  GET  /api/v1/debug/info     ← DEBUG LEAK!  │
└──────────────────────────────────────────────┘
```

---

## 2. API Analysis Techniques

```bash
# === EXTRACT APIs FROM BINARY ===

# iOS
strings MyApp.app/MyApp | grep -E "/api/|/v[0-9]/"
class-dump MyApp.app/MyApp | grep -i "endpoint\|url\|api"

# Android
apktool d app.apk
grep -rn "api\|endpoint\|base.url\|/v[0-9]/" --include="*.smali" decompiled/
grep -rn "https\?://" --include="*.java" decompiled/
# Check Retrofit/OkHttp service interfaces

# === BURP SUITE ANALYSIS ===

# Build Site Map
# Proxy → HTTP History → Sort by URL
# Identify: API version, auth mechanism, response format

# API Documentation Discovery
# Try: /api/docs, /swagger, /swagger.json, /openapi.json
# Try: /api/v1/docs, /graphql (GraphQL introspection)

# === API REVERSE ENGINEERING ===
# From captured traffic, document:
# □ Base URL and versioning
# □ Authentication mechanism (JWT, OAuth, API key)
# □ Request/response format (JSON, XML, protobuf)
# □ Common headers (X-API-Key, Authorization)
# □ Rate limiting behavior
# □ Error response format
# □ Pagination mechanism
```

---

## 3. Mobile-Specific API Issues

```
MOBILE API VULNERABILITIES:

1. EXCESSIVE DATA EXPOSURE:
   → API returns more data than app displays
   → Example: GET /users/me returns ALL user fields
     including internal_id, role, admin_flag, ssn
   → Mobile app only shows name and email
   → Attacker sees everything in proxy!

2. BROKEN FUNCTION AUTHORIZATION:
   → Admin endpoints accessible to regular users
   → App hides admin UI but API doesn't check role
   → GET /admin/users → returns all users!
   → POST /admin/config → modifies server config!

3. CLIENT-SIDE BUSINESS LOGIC:
   → Price calculated on client
   → Discount applied on client
   → Quantity limits enforced on client
   → Attacker modifies request → server accepts

4. API VERSIONING ISSUES:
   → /api/v2/ has fixes, /api/v1/ still vulnerable
   → Old versions not properly deprecated
   → Test all discovered versions

5. HIDDEN ENDPOINTS:
   → Debug endpoints left in production
   → /api/debug/*, /api/test/*
   → May bypass authentication
   → May expose internal data

6. GRAPHQL ISSUES:
   → Introspection enabled → full schema exposed
   → No query depth limiting → DoS
   → No rate limiting per query
   → Batch queries bypass rate limits
```

---

## 4. API Testing Methodology

```
SYSTEMATIC API TESTING:

FOR EACH ENDPOINT:
  1. AUTHENTICATION:
     □ Remove auth header → should get 401
     □ Use expired token → should get 401
     □ Use invalid token → should get 401
     
  2. AUTHORIZATION:
     □ Access other user's resources (IDOR)
     □ Access admin endpoints as regular user
     □ Modify role/permissions in request
     
  3. INPUT VALIDATION:
     □ SQL injection in parameters
     □ NoSQL injection (MongoDB operators)
     □ Command injection
     □ XSS in stored values
     □ Path traversal in file parameters
     
  4. BUSINESS LOGIC:
     □ Negative values (negative price/quantity)
     □ Boundary values (0, MAX_INT)
     □ Race conditions (concurrent requests)
     □ Workflow bypass (skip steps)
     
  5. RATE LIMITING:
     □ Brute force login
     □ Enumeration attacks
     □ SMS/email bombing
     □ Password reset flooding
```

---

## Summary Table

| Issue | Test Method | Impact |
|-------|------------|--------|
| Excessive data exposure | Check API response vs UI | Data leakage |
| IDOR | Change user/resource ID | Unauthorized access |
| Missing auth on admin | Access admin endpoints | Full compromise |
| Client-side logic | Modify request values | Business logic bypass |
| Hidden endpoints | Brute force / code review | Info disclosure |
| API version issues | Test old versions | Exploit fixed vulns |

---

## Revision Questions

1. How do you discover and map all API endpoints from a mobile app?
2. What is excessive data exposure and why is it common in mobile APIs?
3. How do you test for broken function-level authorization?
4. What mobile-specific API vulnerabilities should you look for?
5. Why should old API versions be tested during assessments?

---

*Previous: None (First topic in this unit) | Next: [02-authentication-testing.md](02-authentication-testing.md)*

---

*[Back to README](../README.md)*
