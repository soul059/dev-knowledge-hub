# Unit 4: A04 - Insecure Design — Topic 1: Secure Design Principles

## Overview

Insecure Design is a **fundamentally different category** from implementation bugs. While injection or XSS result from coding mistakes, insecure design means the application **lacks security controls at the architectural level**. A perfectly implemented insecure design is still insecure. This topic covers the foundational principles that must guide secure software architecture from the very beginning of the development lifecycle.

---

## 1. Insecure Design vs Implementation Bugs

```
┌─────────────────────────────────────────────────────────────────┐
│          INSECURE DESIGN vs IMPLEMENTATION BUG                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INSECURE DESIGN:                                              │
│  "We never thought about rate-limiting password resets"         │
│  → No amount of perfect code fixes this                        │
│  → Must redesign the feature                                   │
│                                                                 │
│  IMPLEMENTATION BUG:                                           │
│  "We have rate limiting but forgot to apply it to one route"  │
│  → Fix: Add the rate limit middleware to the route             │
│  → Design was correct, implementation was wrong                │
│                                                                 │
│  KEY DIFFERENCE:                                               │
│  Design = what controls SHOULD exist                           │
│  Implementation = how those controls ARE coded                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Secure Design Principles

### Principle 1: Least Privilege

```
Every component operates with minimum necessary permissions.

Examples:
┌──────────────────────────────┬──────────────────────────────┐
│ ❌ Insecure Design           │ ✅ Secure Design             │
├──────────────────────────────┼──────────────────────────────┤
│ App runs as root/admin       │ App runs as limited user     │
│ DB user has ALL PRIVILEGES   │ DB user has SELECT only      │
│ API key has full access      │ API key scoped per service   │
│ All users see all data       │ Users see own data only      │
│ Single admin role            │ Granular RBAC roles          │
└──────────────────────────────┴──────────────────────────────┘
```

### Principle 2: Defense in Depth

```
Multiple layers of security — if one layer fails, others still protect.

[Internet] → [WAF] → [Load Balancer] → [API Gateway] → [App] → [DB]
                ↓          ↓                  ↓           ↓       ↓
            IP/Rate    TLS Term         AuthN/AuthZ    Input    ACLs
            Limits     Certificate      Schema Valid   Valid    Least
            GeoBlock   Verification     Rate Limits    Escape  Priv

Each layer adds security independently.
Compromise of one layer doesn't compromise all.
```

### Principle 3: Fail Secure (Fail Closed)

```
When errors occur, default to DENY, not ALLOW.

❌ Fail Open:
try:
    if is_authorized(user, resource):
        return resource
except AuthServiceError:
    return resource  # Service down = everyone gets access!

✅ Fail Closed:
try:
    if is_authorized(user, resource):
        return resource
except AuthServiceError:
    log.error("Auth service unavailable")
    return HTTP_503_SERVICE_UNAVAILABLE  # Deny by default
```

### Principle 4: Complete Mediation

```
Every access to every resource must be checked for authorization.

❌ Insecure: Check auth once at login, then trust session forever
✅ Secure:   Check auth on EVERY request to EVERY resource

Example — Direct Object Reference:
GET /api/invoices/12345

Must verify:
1. Is the user authenticated?
2. Is the session valid and not expired?
3. Does user 'X' have permission to access invoice '12345'?
4. Is the resource still active/accessible?

All four checks, every single request.
```

### Principle 5: Separation of Duties

```
No single user/component should control an entire critical process.

Examples:
- Code author ≠ Code reviewer ≠ Deployer
- Funds requester ≠ Funds approver
- User creator ≠ Role assigner
- Data owner ≠ Data custodian

Architecture:
┌──────────┐   ┌──────────┐   ┌──────────┐
│ Developer│──→│ Reviewer │──→│ Deployer │
│ writes   │   │ approves │   │ releases │
│ code     │   │ PR       │   │ to prod  │
└──────────┘   └──────────┘   └──────────┘
No single person can push malicious code to production.
```

### Principle 6: Economy of Mechanism (Keep It Simple)

```
Simple designs are easier to verify, test, and secure.

❌ Complex:
- Custom authentication protocol
- Home-grown encryption algorithm
- 15 microservices for a simple CRUD app
- Custom session management

✅ Simple:
- OAuth 2.0 / OpenID Connect (industry standard)
- AES-256, RSA (proven algorithms)
- Monolith or appropriately-sized services
- Framework session management
```

### Principle 7: Open Design

```
Security should not depend on secrecy of the design.
(Kerckhoffs's Principle)

❌ Security through obscurity:
- Hiding admin panel at /s3cr3t-4dm1n
- Custom "encryption" that XORs with a fixed key
- Assuming attackers won't find undocumented API endpoints

✅ Open design:
- Admin panel at /admin with proper authentication + MFA
- Standard encryption algorithms (AES, ChaCha20)
- All API endpoints require authentication regardless
- Assume attackers know the source code
```

---

## 3. Secure Design Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| **Deny by default** | Block everything, then explicitly allow | Firewall rules, CORS |
| **Validate all inputs** | Never trust any input source | Server-side validation always |
| **Centralized security** | Single auth/authz point | API Gateway, middleware |
| **Audit everything** | Log security-relevant events | Auth attempts, data access |
| **Minimize attack surface** | Remove unnecessary features | Disable unused endpoints |
| **Secure defaults** | Ship secure out of box | HTTPS, strong passwords, MFA |

---

## 4. Business Logic Security Design

```
Common Insecure Design Scenarios:

1. CREDENTIAL STUFFING — No Rate Limiting
   Design Question: "What if someone tries 10,000 passwords?"
   Control: Account lockout, CAPTCHA, rate limiting

2. COUPON ABUSE — No Usage Limits
   Design Question: "What if someone applies a coupon 1000 times?"
   Control: Single-use tokens, per-user limits

3. BOOKING ABUSE — No Reservation Limits
   Design Question: "What if a bot books all seats, then cancels?"
   Control: Payment required to confirm, time-limited holds

4. DATA SCRAPING — No Access Throttling
   Design Question: "What if someone downloads our entire database via API?"
   Control: Pagination limits, rate limiting, anomaly detection

5. PASSWORD RESET — No Verification
   Design Question: "What if someone resets another user's password?"
   Control: Multi-step verification, secure token, email confirmation
```

---

## 5. Security Requirements Engineering

```
┌────────────────────────────────────────────────────────────────┐
│           SECURITY REQUIREMENTS FRAMEWORK                      │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  For each feature, ask:                                       │
│                                                                │
│  1. AUTHENTICATION                                            │
│     Who can access this? How do we verify identity?           │
│                                                                │
│  2. AUTHORIZATION                                             │
│     What can they do? Role-based? Attribute-based?           │
│                                                                │
│  3. INPUT VALIDATION                                          │
│     What input is expected? What are the boundaries?          │
│                                                                │
│  4. DATA PROTECTION                                           │
│     Is data sensitive? Encrypt at rest? In transit?           │
│                                                                │
│  5. RATE LIMITING                                             │
│     How often can this be called? Throttle? Block?           │
│                                                                │
│  6. LOGGING & MONITORING                                      │
│     What events should be logged? What triggers alerts?       │
│                                                                │
│  7. ERROR HANDLING                                            │
│     What happens on failure? Fail open or closed?            │
│                                                                │
│  8. ABUSE CASES                                               │
│     How could an attacker misuse this feature?               │
└────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Principle | Core Idea | Violation Example |
|-----------|-----------|-------------------|
| Least Privilege | Minimum necessary access | App running as root |
| Defense in Depth | Multiple security layers | Single firewall, no app-level checks |
| Fail Secure | Deny on error | Auth error → allow access |
| Complete Mediation | Check every access | Check auth only at login |
| Separation of Duties | No single control point | Developer deploys own code to prod |
| Economy of Mechanism | Keep it simple | Custom crypto algorithm |
| Open Design | No security by obscurity | Hidden admin URL = security |

---

## Revision Questions

1. What is the fundamental difference between insecure design and an implementation bug? Give three examples of each.
2. Explain the principle of "Fail Secure" and why "Fail Open" is dangerous. Provide code examples.
3. How does Defense in Depth apply to a modern web application? List all layers.
4. What security questions should you ask for EVERY new feature during design?
5. Explain Complete Mediation and why checking authorization only at login is insufficient.
6. Why does OWASP consider Insecure Design a separate category from other vulnerabilities?

---

*Previous: None (First topic in this unit) | Next: [02-threat-modeling.md](02-threat-modeling.md)*

---

*[Back to README](../README.md)*
