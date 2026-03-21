# Unit 8: API Security Testing — Topic 5: API Vulnerabilities

## Overview

Mobile app APIs are susceptible to the same vulnerabilities as web APIs — plus additional issues unique to the mobile context. This topic covers the most critical API vulnerabilities found during mobile penetration tests, including IDOR, mass assignment, rate limiting failures, injection attacks, and business logic flaws.

---

## 1. IDOR (Insecure Direct Object Reference)

```
IDOR IN MOBILE APIs:

THE MOST COMMON MOBILE API VULNERABILITY!

CONCEPT:
  → API uses predictable identifiers (user_id, order_id)
  → Server doesn't verify requesting user owns the resource
  → Change ID in request → access other users' data

EXAMPLES:
  GET /api/users/123/profile     → Your profile
  GET /api/users/124/profile     → Someone else's profile!
  
  GET /api/orders/5001           → Your order
  GET /api/orders/5002           → Someone else's order!
  
  DELETE /api/messages/789       → Delete someone else's message!

TESTING:
  1. Login as User A
  2. Capture requests with resource IDs
  3. Login as User B
  4. Replay User A's requests with User B's token
  5. If data returned: IDOR confirmed!

AUTOMATED TESTING:
  → Burp Suite Autorize extension
  → Two browser sessions (two users)
  → Autorize replays requests with other user's token
  → Flags successful cross-user access

TYPES:
  → Horizontal IDOR: User A → User B (same role)
  → Vertical IDOR: User → Admin resources
  → Object reference: /users/123 (numeric, sequential)
  → UUID reference: /users/uuid (harder but still testable)
```

---

## 2. Mass Assignment / Excessive Data

```
MASS ASSIGNMENT:

CONCEPT:
  → API accepts more parameters than intended
  → Attacker adds extra fields to modify protected data

EXAMPLE:
  Normal request:
  PUT /api/users/me
  {"name": "John", "email": "john@example.com"}

  Attack request:
  PUT /api/users/me
  {"name": "John", "email": "john@example.com", 
   "role": "admin", "is_premium": true, "balance": 99999}
  
  If server binds ALL fields: attacker becomes admin!

TESTING:
  1. Send normal update request
  2. Add hidden fields found in GET response
  3. Add common privileged fields:
     role, is_admin, admin, premium, verified,
     balance, credits, permissions, group_id

EXCESSIVE DATA EXPOSURE:

  GET /api/users/me returns:
  {
    "id": 123,
    "name": "John",
    "email": "john@example.com",
    "password_hash": "bcrypt$...",    ← CRITICAL!
    "ssn": "123-45-6789",            ← PII LEAK!
    "internal_role": "user",          ← INFO LEAK
    "admin_flag": false,              ← INFO LEAK
    "api_key": "sk_live_...",         ← KEY LEAK!
    "created_ip": "192.168.1.1"      ← PRIVACY
  }
  → App only shows name and email
  → All other data exposed in API response!
```

---

## 3. Injection Attacks

```
SQL INJECTION:
  GET /api/users?search=admin' OR 1=1--
  GET /api/products?category=1 UNION SELECT password FROM users--
  
  TESTING:
  → Add ' to string parameters
  → Add arithmetic to numeric (id=1+1)
  → Time-based: id=1 AND SLEEP(5)
  → Union-based: id=1 UNION SELECT 1,2,3

NoSQL INJECTION (MongoDB):
  POST /api/login
  {"username": {"$gt": ""}, "password": {"$gt": ""}}
  → Matches any non-empty username and password!
  
  {"username": "admin", "password": {"$ne": "wrong"}}
  → Password not equal to "wrong" = true for any password!

COMMAND INJECTION:
  POST /api/tools/ping
  {"host": "8.8.8.8; cat /etc/passwd"}
  → If server executes: command injection!

SERVER-SIDE REQUEST FORGERY (SSRF):
  POST /api/fetch-url
  {"url": "http://169.254.169.254/latest/meta-data/"}
  → Access cloud metadata (AWS keys, etc.)
  
  {"url": "http://internal-service:8080/admin"}
  → Access internal services!
```

---

## 4. Rate Limiting and Business Logic

```
RATE LIMITING FAILURES:

ENDPOINTS REQUIRING RATE LIMITING:
  □ /auth/login           → Brute force prevention
  □ /auth/forgot-password → Email/SMS bombing
  □ /auth/verify-otp      → OTP brute force
  □ /users/search          → Enumeration
  □ /transactions          → Transaction flooding
  □ /referral/apply        → Referral abuse

TESTING:
  → Send 100+ requests in quick succession
  → Check for 429 Too Many Requests
  → Check if limit is per-IP, per-user, or per-session
  → Test bypass: X-Forwarded-For header spoofing
  → Test bypass: Different case in endpoint path

BUSINESS LOGIC FLAWS:

PRICE MANIPULATION:
  POST /api/orders
  {"product_id": 1, "quantity": 1, "price": 0.01}
  → Server accepts client-provided price!

NEGATIVE VALUES:
  POST /api/transfer
  {"to": "attacker", "amount": -1000}
  → Negative transfer = money TO attacker!

RACE CONDITIONS:
  → Send multiple redemption requests simultaneously
  → Coupon/gift card used multiple times
  → Balance check → deduct race condition
  
  # Test with concurrent requests
  for i in {1..10}; do
    curl -X POST /api/redeem/COUPON123 &
  done
  wait
  # Check: Was coupon applied 10 times?

WORKFLOW BYPASS:
  → Skip steps in multi-step process
  → Go directly to final step without validation
  → Example: Skip payment → access premium content
```

---

## 5. API Security Checklist

```
COMPREHENSIVE API TESTING CHECKLIST:

AUTHENTICATION:
  □ All endpoints require authentication
  □ Tokens properly validated
  □ Session management secure
  □ Brute force protection

AUTHORIZATION:
  □ IDOR tested on all resource endpoints
  □ Horizontal privilege tested
  □ Vertical privilege tested (user→admin)
  □ Function-level authorization

INPUT VALIDATION:
  □ SQL injection tested
  □ NoSQL injection tested
  □ Command injection tested
  □ XSS (stored) in API inputs
  □ SSRF in URL parameters

DATA EXPOSURE:
  □ API responses checked for excessive data
  □ Error messages don't leak internals
  □ Debug endpoints removed
  □ Stack traces suppressed

TRANSPORT:
  □ HTTPS enforced
  □ Certificate pinning implemented
  □ No sensitive data in URLs
  □ Security headers present

BUSINESS LOGIC:
  □ Price/quantity server-validated
  □ Race conditions tested
  □ Workflow steps enforced
  □ Rate limiting on sensitive endpoints
```

---

## Summary Table

| Vulnerability | OWASP API | Severity | Test Method |
|--------------|-----------|----------|-------------|
| IDOR | API1 Broken Auth | High-Critical | Change resource IDs |
| Mass Assignment | API6 | High | Add extra fields |
| Excessive Data | API3 | Medium-High | Review responses |
| SQL Injection | API8 Injection | Critical | Special characters |
| Missing Rate Limit | API4 | Medium | Flood requests |
| Business Logic | API10 | Varies | Manual testing |
| SSRF | API7 | High | Internal URLs |

---

## Revision Questions

1. What is IDOR and how do you test for it systematically?
2. How does mass assignment work and what fields should you test?
3. What injection types are common in mobile APIs?
4. How do you test for rate limiting failures?
5. What business logic flaws are specific to mobile commerce apps?
6. What is the OWASP API Security Top 10?

---

*Previous: [04-certificate-pinning.md](04-certificate-pinning.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
