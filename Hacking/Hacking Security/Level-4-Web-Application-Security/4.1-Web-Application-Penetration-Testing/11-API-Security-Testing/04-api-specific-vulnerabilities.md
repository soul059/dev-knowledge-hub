# Unit 11: API Security Testing - Topic 4: API-Specific Vulnerabilities

## Overview

APIs introduce unique vulnerability classes that differ from traditional web application flaws. These include **Broken Object Level Authorization (BOLA)**, **Broken Function Level Authorization**, **Mass Assignment**, **Excessive Data Exposure**, and **Security Misconfiguration**. The OWASP API Security Top 10 (2023) specifically addresses these API-centric risks. Understanding these vulnerabilities is essential because APIs are the backbone of modern applications, microservices, and mobile backends.

---

## 1. OWASP API Security Top 10 (2023)

### Complete List

| # | Risk | Description |
|---|------|-------------|
| **API1** | Broken Object Level Authorization | Accessing other users' objects via ID manipulation |
| **API2** | Broken Authentication | Flawed auth mechanisms in APIs |
| **API3** | Broken Object Property Level Auth | Mass assignment + excessive data exposure |
| **API4** | Unrestricted Resource Consumption | No rate limiting, resource exhaustion |
| **API5** | Broken Function Level Authorization | Accessing admin functions as regular user |
| **API6** | Unrestricted Access to Sensitive Business Flows | Automated abuse of business logic |
| **API7** | Server-Side Request Forgery | SSRF via API parameters |
| **API8** | Security Misconfiguration | Default configs, unnecessary features enabled |
| **API9** | Improper Inventory Management | Unmanaged API versions, shadow APIs |
| **API10** | Unsafe Consumption of Third-Party APIs | Trusting third-party API responses |

---

## 2. Broken Object Level Authorization (BOLA / IDOR)

### The #1 API Vulnerability

```
BOLA Attack Flow:
┌──────────┐                         ┌──────────┐
│  User A  │  GET /api/orders/1001   │          │
│  (owns   │ ──────────────────────► │   API    │ → Returns User A's order ✓
│  order   │                         │  Server  │
│  1001)   │                         │          │
└──────────┘                         └──────────┘

┌──────────┐                         ┌──────────┐
│  User B  │  GET /api/orders/1001   │          │
│  (does   │ ──────────────────────► │   API    │ → Returns User A's order ✗
│  NOT own │                         │  Server  │   BOLA VULNERABILITY!
│  1001)   │                         │          │
└──────────┘                         └──────────┘

The API doesn't verify that the authenticated user
owns or has permission to access the requested object.
```

### BOLA Testing Methodology

```bash
# Step 1: Authenticate as User A and capture their object IDs
GET /api/users/me → {"id": 1001, "name": "User A"}
GET /api/orders → [{"id": 5001}, {"id": 5002}]
GET /api/documents/8001 → User A's document

# Step 2: Authenticate as User B (different account)
# Use User B's token to access User A's resources

# Test sequential IDs
curl -H "Authorization: Bearer USER_B_TOKEN" \
  http://target.com/api/orders/5001   # User A's order

# Test on all object-referencing endpoints
curl -H "Authorization: Bearer USER_B_TOKEN" \
  http://target.com/api/users/1001
curl -H "Authorization: Bearer USER_B_TOKEN" \
  http://target.com/api/documents/8001
curl -H "Authorization: Bearer USER_B_TOKEN" \
  http://target.com/api/invoices/3001

# Step 3: Test CRUD operations
# Can User B modify User A's data?
PUT  /api/orders/5001  {"status": "cancelled"}
DELETE /api/orders/5001
PATCH  /api/users/1001 {"email": "attacker@evil.com"}

# Step 4: Test with different identifier types
# UUID-based (harder to guess but still testable)
GET /api/users/550e8400-e29b-41d4-a716-446655440000
# Try incrementing last digits or using known UUIDs

# Encoded IDs
# Base64: GET /api/data/MTAwMQ== (decodes to "1001")
echo -n "1002" | base64  # MTAwMg==
GET /api/data/MTAwMg==
```

### Automated BOLA Testing

```bash
# Autorize (Burp Extension)
# 1. Install Autorize from BApp Store
# 2. Set User B's session token in Autorize
# 3. Browse application as User A
# 4. Autorize automatically replays requests with User B's token
# 5. Highlights where User B can access User A's resources

# Custom Python script
import requests

USER_A_TOKEN = "token_a"
USER_B_TOKEN = "token_b"
BASE_URL = "http://target.com/api"

# Get User A's resources
endpoints = [
    f"/orders/{i}" for i in range(5000, 5010)
] + [
    f"/users/{i}" for i in range(1000, 1010)
]

for endpoint in endpoints:
    # Access with User B's token
    resp = requests.get(
        f"{BASE_URL}{endpoint}",
        headers={"Authorization": f"Bearer {USER_B_TOKEN}"}
    )
    if resp.status_code == 200:
        print(f"BOLA FOUND: {endpoint} - {resp.status_code}")
    else:
        print(f"Protected:  {endpoint} - {resp.status_code}")
```

---

## 3. Mass Assignment

### How Mass Assignment Works

```
Mass Assignment Vulnerability:

Expected request:                     Actual server processing:
POST /api/users                       user = User()
{                                     user.name = data['name']       ✓
  "name": "John",                     user.email = data['email']     ✓
  "email": "john@example.com"         user.role = data['role']       ✗ DANGER!
}                                     user.admin = data['admin']     ✗ DANGER!
                                      user.balance = data['balance'] ✗ DANGER!

Attack: Add extra fields to request:
POST /api/users
{
  "name": "John",
  "email": "john@example.com",
  "role": "admin",          ← Mass assigned!
  "admin": true,            ← Mass assigned!
  "balance": 999999         ← Mass assigned!
}
```

### Finding Mass Assignment Vulnerabilities

```bash
# Step 1: Get current object properties
GET /api/users/me
{
  "id": 1001,
  "name": "John",
  "email": "john@example.com",
  "role": "user",
  "verified": false,
  "balance": 0,
  "created_at": "2024-01-01"
}

# Step 2: Try updating read-only/admin-only properties
PUT /api/users/me
{
  "name": "John",
  "role": "admin"        # Try escalating role
}

PATCH /api/users/me
{
  "verified": true       # Try self-verifying
}

PUT /api/users/me
{
  "balance": 999999      # Try adding balance
}

# Step 3: Try during registration
POST /api/register
{
  "name": "attacker",
  "email": "attacker@evil.com",
  "password": "password123",
  "role": "admin",
  "is_admin": true,
  "isAdmin": true,
  "admin": 1,
  "level": 99,
  "permissions": ["admin", "superuser"]
}

# Step 4: Try during password reset
POST /api/reset-password
{
  "token": "valid_token",
  "new_password": "newpass123",
  "email": "victim@example.com",   # Change email during reset?
  "role": "admin"                  # Escalate during reset?
}

# Common mass-assignable properties to try:
# role, admin, isAdmin, is_admin, permissions, level, type
# verified, active, approved, status
# balance, credits, price, discount
# password, password_hash, api_key, token
# created_at, updated_at, id
```

### Vulnerable Code Example

```python
# VULNERABLE - Assigns all incoming fields
@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    user = User(**data)  # Mass assignment!
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict())

# SECURE - Whitelist allowed fields
@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    allowed_fields = ['name', 'email', 'password']
    filtered = {k: v for k, v in data.items() if k in allowed_fields}
    user = User(**filtered)
    user.role = 'user'  # Explicitly set defaults
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict())
```

---

## 4. Excessive Data Exposure

### API Returns Too Much Data

```bash
# Vulnerable: API returns all user fields
GET /api/users/1001
{
  "id": 1001,
  "name": "John Doe",
  "email": "john@example.com",
  "password_hash": "$2b$12$...",        # EXPOSED!
  "ssn": "123-45-6789",                 # EXPOSED!
  "credit_card": "4111111111111111",     # EXPOSED!
  "internal_notes": "VIP customer",     # EXPOSED!
  "api_key": "sk_live_abc123",          # EXPOSED!
  "role": "admin",
  "last_login_ip": "192.168.1.100"
}

# The frontend only displays name and email
# But the API sends EVERYTHING
# Attacker just reads the raw API response!
```

### Testing for Excessive Data Exposure

```bash
# Compare API response with UI display
# What the UI shows vs what the API returns

# Test different response formats
GET /api/users/1001
GET /api/users/1001?format=json
GET /api/users/1001?format=xml
GET /api/users/1001?fields=all
GET /api/users/1001?include=*
GET /api/users/1001?expand=true
GET /api/users/1001?verbose=true
GET /api/users/1001?debug=true

# Test list endpoints (often expose more)
GET /api/users          # May return all fields for all users
GET /api/users?limit=1  # Even single record may have extra data

# Test export/download endpoints
GET /api/users/export.csv
GET /api/users/export.json

# Test GraphQL (returns exactly what you query)
query { user(id: 1001) { 
  password_hash  # Will it return this?
  ssn
  credit_card
  internal_notes
}}

# Test error responses (may leak data)
GET /api/users/999999
# Error: "User not found in table 'prod.users'" ← leaks DB info
```

---

## 5. Improper Inventory Management (Shadow APIs)

### Discovering Unmanaged APIs

```
Shadow APIs:
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Production APIs (Managed):                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│  │ /api/v2 │ │ /api/v2 │ │ /api/v2 │              │
│  │ /users  │ │ /orders │ │ /search │              │
│  └─────────┘ └─────────┘ └─────────┘              │
│                                                     │
│  Shadow APIs (Unmanaged):                           │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│  │ /api/v1 │ │ /api    │ │ /debug  │ ← No auth!   │
│  │ /users  │ │ /test   │ │ /status │ ← No rate    │
│  └─────────┘ └─────────┘ └─────────┘   limit!     │
│      ↑             ↑          ↑                     │
│   Old version   Dev/Test   Internal                 │
│   still active  endpoint   endpoint                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

```bash
# Find old API versions
for v in v0 v1 v2 v3 v4 beta dev staging test internal; do
    code=$(curl -s -o /dev/null -w "%{http_code}" \
      "http://target.com/api/$v/users")
    [ "$code" != "404" ] && echo "FOUND: /api/$v/users ($code)"
done

# Find development/debug endpoints
ffuf -u "http://target.com/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,301,401,403

# Common shadow API paths
/api/debug
/api/test
/api/internal
/api/admin
/api/health
/api/status
/api/metrics
/api/swagger
/api/docs
/api/graphql
/internal/
/debug/
/actuator/  (Spring Boot)
/env
/info
/trace
/mappings
```

---

## 6. Unsafe Consumption of Third-Party APIs

### Trusting External API Responses

```python
# VULNERABLE: Trusting third-party API response
@app.route('/api/enrich-user', methods=['POST'])
def enrich_user():
    user_id = request.json['user_id']
    
    # Call third-party API
    resp = requests.get(f"https://third-party.com/api/users/{user_id}")
    external_data = resp.json()
    
    # Directly use external data without validation!
    user = User.query.get(user_id)
    user.address = external_data['address']        # Could contain XSS
    user.bio = external_data['bio']                # Could contain SQLi
    user.avatar_url = external_data['avatar_url']  # Could be SSRF
    db.session.commit()

# SECURE: Validate and sanitize third-party data
@app.route('/api/enrich-user', methods=['POST'])
def enrich_user():
    user_id = request.json['user_id']
    
    resp = requests.get(
        f"https://third-party.com/api/users/{user_id}",
        timeout=5  # Prevent hanging
    )
    
    if resp.status_code != 200:
        return "External service error", 502
    
    external_data = resp.json()
    
    # Validate and sanitize each field
    user = User.query.get(user_id)
    user.address = sanitize_html(external_data.get('address', '')[:500])
    user.bio = sanitize_html(external_data.get('bio', '')[:1000])
    
    avatar = external_data.get('avatar_url', '')
    if is_valid_image_url(avatar):
        user.avatar_url = avatar
    
    db.session.commit()
```

---

## 7. API Security Testing Methodology

### Comprehensive API Testing Workflow

```
┌───────────────────────────────────────────────────────┐
│             API Security Testing Workflow              │
├───────────────────────────────────────────────────────┤
│                                                       │
│ 1. DISCOVERY                                          │
│    □ Find all API endpoints                           │
│    □ Discover API documentation                       │
│    □ Identify old/shadow APIs                         │
│    □ Map authentication mechanisms                    │
│                                                       │
│ 2. AUTHENTICATION TESTING                             │
│    □ Test auth bypass (OWASP API2)                    │
│    □ Test token manipulation                          │
│    □ Test brute force resistance                      │
│                                                       │
│ 3. AUTHORIZATION TESTING                              │
│    □ Test BOLA/IDOR (OWASP API1)                      │
│    □ Test function-level access (OWASP API5)          │
│    □ Test mass assignment (OWASP API3)                │
│                                                       │
│ 4. DATA EXPOSURE TESTING                              │
│    □ Check for excessive data in responses            │
│    □ Test error message information leakage           │
│    □ Check for sensitive data in logs/URLs            │
│                                                       │
│ 5. INPUT VALIDATION                                   │
│    □ Test injection attacks (SQLi, NoSQLi, cmd)       │
│    □ Test SSRF via API parameters (OWASP API7)        │
│    □ Test payload size limits                         │
│                                                       │
│ 6. RATE LIMITING & RESOURCES                          │
│    □ Test rate limiting (OWASP API4)                   │
│    □ Test resource exhaustion                         │
│    □ Test business logic abuse (OWASP API6)            │
│                                                       │
│ 7. CONFIGURATION                                      │
│    □ Check security headers                           │
│    □ Test CORS configuration                          │
│    □ Check TLS configuration                          │
│    □ Test debug/error modes (OWASP API8)              │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## 8. Prevention Strategies

### API Security Best Practices

| Category | Best Practice |
|----------|--------------|
| **Authorization** | Implement object-level authorization checks on every endpoint |
| **Authentication** | Use OAuth 2.0/OIDC, short-lived tokens, MFA |
| **Data exposure** | Return only required fields, use DTOs/response schemas |
| **Mass assignment** | Whitelist allowed fields, use DTOs for input |
| **Rate limiting** | Per-user, per-endpoint, per-IP rate limits |
| **Input validation** | Validate all input, use JSON Schema validation |
| **API gateway** | Centralized auth, rate limiting, logging |
| **Versioning** | Deprecate old versions, remove unused endpoints |
| **Monitoring** | Log all API calls, alert on anomalies |
| **Documentation** | Keep OpenAPI specs current, use as validation source |

---

## Summary Table

| Vulnerability | OWASP API # | Impact | Testing Approach |
|---------------|-------------|--------|------------------|
| **BOLA/IDOR** | API1 | Data breach, unauthorized access | Two-account testing, ID manipulation |
| **Broken Auth** | API2 | Account takeover | Token attacks, brute force |
| **Mass Assignment** | API3 | Privilege escalation | Add extra fields to requests |
| **Resource Exhaustion** | API4 | DoS, abuse | Large payloads, rapid requests |
| **Broken Function Auth** | API5 | Admin access | Test admin endpoints as user |
| **Business Logic Abuse** | API6 | Financial loss | Automated workflow abuse |
| **SSRF** | API7 | Internal access | URL parameters to internal IPs |
| **Misconfiguration** | API8 | Info leak | Debug endpoints, default configs |
| **Shadow APIs** | API9 | Unprotected access | Version enumeration, discovery |
| **Third-Party Trust** | API10 | Injection via data | Validate all external data |

---

## Revision Questions

1. What is BOLA (Broken Object Level Authorization) and why is it the #1 API vulnerability? How would you test for it using two accounts?
2. Explain mass assignment attacks. How can an attacker escalate privileges during user registration?
3. What is excessive data exposure in APIs? Why doesn't client-side filtering solve the problem?
4. Describe the concept of "shadow APIs." How do you discover unmanaged API endpoints?
5. How does the OWASP API Security Top 10 differ from the OWASP Web Application Top 10?
6. Design a secure API architecture that addresses the top 5 API security risks.

---

*Previous: [03-rate-limiting.md](03-rate-limiting.md) | Next: [05-graphql-security.md](05-graphql-security.md)*

---

*[Back to README](../README.md)*
