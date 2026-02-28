# Chapter 4.4: API Security Testing

## Overview

APIs are the primary attack surface for modern applications. This chapter covers API-specific DAST techniques, including REST/GraphQL testing, authentication bypass, rate limiting, and automated API security scanning.

---

## API Attack Surface

```
API ATTACK SURFACE MAP:

  ┌──────────────────────────────────────────────────────────────┐
  │                    API SECURITY RISKS                         │
  │                                                                │
  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
  │  │ AUTH/AUTHZ  │  │ INPUT       │  │ BUSINESS LOGIC      │  │
  │  │             │  │             │  │                      │  │
  │  │ Broken auth │  │ Injection   │  │ IDOR                 │  │
  │  │ Weak tokens │  │ Mass assign │  │ Rate limit bypass    │  │
  │  │ Missing RBAC│  │ Excessive   │  │ Price manipulation   │  │
  │  │ Token leak  │  │ data return │  │ Workflow bypass      │  │
  │  └─────────────┘  └─────────────┘  └─────────────────────┘  │
  │                                                                │
  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
  │  │ CONFIG      │  │ TRANSPORT   │  │ RESOURCE MGMT       │  │
  │  │             │  │             │  │                      │  │
  │  │ CORS miscon │  │ No TLS      │  │ No pagination        │  │
  │  │ Debug mode  │  │ Mixed HTTP  │  │ Memory exhaustion   │  │
  │  │ Verbose err │  │ Weak cipher │  │ Unrestricted upload  │  │
  │  └─────────────┘  └─────────────┘  └─────────────────────┘  │
  └──────────────────────────────────────────────────────────────┘
```

---

## OWASP API Security Top 10 (2023)

```
  ┌────┬───────────────────────────────┬─────────────────────────┐
  │ #  │ Risk                           │ Example                  │
  ├────┼───────────────────────────────┼─────────────────────────┤
  │ 1  │ Broken Object Level Auth      │ GET /api/users/OTHER_ID │
  │ 2  │ Broken Authentication          │ Weak JWT, no lockout    │
  │ 3  │ Broken Object Property Auth   │ Mass assignment attacks  │
  │ 4  │ Unrestricted Resource Usage   │ No rate limits, DoS      │
  │ 5  │ Broken Function Level Auth    │ User calls admin API     │
  │ 6  │ Unrestricted Server-Side Req  │ SSRF via URL params      │
  │ 7  │ Security Misconfiguration      │ CORS *, debug mode       │
  │ 8  │ Lack of Protection from Auto  │ No bot/scraping defense  │
  │    │ Automated Threats              │                          │
  │ 9  │ Improper Inventory Management │ Shadow/zombie APIs       │
  │ 10 │ Unsafe Consumption of APIs    │ Trusting third-party API │
  └────┴───────────────────────────────┴─────────────────────────┘
```

---

## API Scanning with ZAP

```bash
# Scan REST API using OpenAPI spec
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
    -t https://staging.myapp.com/openapi.json \
    -f openapi \
    -r api-report.html \
    -z "-config replacer.full_list(0).enabled=true \
        -config replacer.full_list(0).matchtype=REQ_HEADER \
        -config replacer.full_list(0).matchstr=Authorization \
        -config replacer.full_list(0).replacement='Bearer eyJhbG...'"

# Scan GraphQL API
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
    -t https://staging.myapp.com/graphql \
    -f graphql \
    -r graphql-report.html
```

### ZAP API Scan Automation

```yaml
# zap-api-automation.yaml
env:
  contexts:
    - name: "API Context"
      urls:
        - "https://staging.myapp.com/api"
      technology:
        include:
          - "Language.Python"
          - "Db.PostgreSQL"

jobs:
  - type: openapi
    parameters:
      apiUrl: "https://staging.myapp.com/openapi.json"
      targetUrl: "https://staging.myapp.com"
  
  - type: passiveScan-wait
    parameters:
      maxDuration: 5
  
  - type: activeScan
    parameters:
      maxRuleDurationInMins: 3
      maxScanDurationInMins: 20
      policy: "API-Minimal"
  
  - type: report
    parameters:
      template: "sarif-json"
      reportDir: "/zap/reports/"
      reportFile: "api-scan"
```

---

## Testing Common API Vulnerabilities

### 1. BOLA / IDOR Testing

```bash
# Test IDOR: Access another user's resource
# Authenticated as User A (ID: 100)

# Normal request
curl -H "Authorization: Bearer $TOKEN_USER_A" \
     https://api.app.com/api/users/100/orders
# → 200 OK (own orders)

# IDOR attempt: Access User B's orders
curl -H "Authorization: Bearer $TOKEN_USER_A" \
     https://api.app.com/api/users/200/orders
# VULNERABLE: 200 OK (other user's orders!)
# SECURE:     403 Forbidden

# Automated IDOR check script
for id in $(seq 1 100); do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Authorization: Bearer $TOKEN_USER_A" \
        "https://api.app.com/api/users/$id/profile")
    if [ "$STATUS" = "200" ] && [ "$id" != "100" ]; then
        echo "IDOR FOUND: /api/users/$id accessible by user 100"
    fi
done
```

### 2. Mass Assignment Testing

```bash
# Normal user registration
curl -X POST https://api.app.com/api/register \
    -H "Content-Type: application/json" \
    -d '{"username": "test", "email": "t@t.com", "password": "pass123"}'

# Mass assignment attack: Try setting admin role
curl -X POST https://api.app.com/api/register \
    -H "Content-Type: application/json" \
    -d '{"username": "hacker", "email": "h@h.com", "password": "pass123",
         "role": "admin", "is_admin": true, "isVerified": true}'

# Check if extra fields were accepted
curl -H "Authorization: Bearer $HACKER_TOKEN" \
     https://api.app.com/api/me
# VULNERABLE: {"role": "admin", "is_admin": true}
```

### 3. Rate Limiting Test

```bash
# Test rate limiting with rapid requests
for i in $(seq 1 200); do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
        -X POST https://api.app.com/api/login \
        -H "Content-Type: application/json" \
        -d '{"username": "admin", "password": "attempt'$i'"}')
    echo "Attempt $i: $STATUS"
    if [ "$STATUS" = "429" ]; then
        echo "Rate limit triggered at attempt $i"
        break
    fi
done
# VULNERABLE: Never gets 429 (no rate limiting)
# SECURE: Gets 429 after ~10-20 attempts
```

### 4. JWT Security Testing

```bash
# Decode JWT (without verification)
echo "$JWT_TOKEN" | cut -d'.' -f2 | base64 -d 2>/dev/null | jq .

# Test: Algorithm confusion (none attack)
# Original header: {"alg":"RS256","typ":"JWT"}
# Attack header:   {"alg":"none","typ":"JWT"}
ATTACK_HEADER=$(echo -n '{"alg":"none","typ":"JWT"}' | base64 -w0 | tr '+/' '-_' | tr -d '=')
PAYLOAD=$(echo "$JWT_TOKEN" | cut -d'.' -f2)
ATTACK_TOKEN="${ATTACK_HEADER}.${PAYLOAD}."

curl -H "Authorization: Bearer $ATTACK_TOKEN" \
     https://api.app.com/api/admin
# VULNERABLE: 200 OK (accepted unsigned token!)

# Test: Change user ID in payload
# Decode, modify sub/user_id, re-encode (if using weak secret)
```

---

## Nuclei API Templates

```yaml
# api-security-checks.yaml
id: api-cors-wildcard
info:
  name: CORS Wildcard Check
  severity: high
  tags: api,cors,misconfig

http:
  - method: GET
    path:
      - "{{BaseURL}}/api/"
    headers:
      Origin: "https://evil.attacker.com"
    matchers:
      - type: word
        part: header
        words:
          - "Access-Control-Allow-Origin: *"
          - "Access-Control-Allow-Origin: https://evil.attacker.com"
        condition: or
```

```yaml
# api-excessive-data.yaml
id: api-excessive-data
info:
  name: API Excessive Data Exposure
  severity: medium
  tags: api,data-exposure

http:
  - method: GET
    path:
      - "{{BaseURL}}/api/users/me"
    headers:
      Authorization: "Bearer {{token}}"
    matchers:
      - type: word
        words:
          - "password"
          - "ssn"
          - "credit_card"
          - "secret"
          - "internal_id"
        condition: or
```

---

## GraphQL-Specific Testing

```
GRAPHQL ATTACK VECTORS:

  1. INTROSPECTION (Information Disclosure)
  ┌────────────────────────────────────────────────────┐
  │ Query:                                              │
  │   { __schema { types { name fields { name } } } } │
  │                                                      │
  │ Reveals entire schema → all types, fields, queries  │
  │ Should be DISABLED in production                    │
  └────────────────────────────────────────────────────┘

  2. BATCH QUERY ATTACK (DoS)
  ┌────────────────────────────────────────────────────┐
  │ Query: [{query1}, {query2}, ... {query1000}]       │
  │ Bypasses rate limiting (1 HTTP request, 1000 ops)  │
  └────────────────────────────────────────────────────┘

  3. DEEP QUERY (DoS)
  ┌────────────────────────────────────────────────────┐
  │ { user { friends { friends { friends { ... } } } } │
  │ Exponential database load                           │
  │ Mitigate: Query depth limiting                      │
  └────────────────────────────────────────────────────┘
```

```bash
# Test GraphQL introspection
curl -X POST https://api.app.com/graphql \
    -H "Content-Type: application/json" \
    -d '{"query": "{ __schema { types { name } } }"}'
# VULNERABLE: Returns full schema
# SECURE: Returns error "Introspection disabled"

# Test batch queries
curl -X POST https://api.app.com/graphql \
    -H "Content-Type: application/json" \
    -d '[
        {"query": "{ user(id:1) { email } }"},
        {"query": "{ user(id:2) { email } }"},
        {"query": "{ user(id:3) { email } }"}
    ]'
# VULNERABLE: Returns all three responses
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ZAP can't parse OpenAPI spec | Invalid or v2 spec format | Validate spec with Swagger editor; convert v2 → v3 |
| Scanner misses authenticated endpoints | Token not passed correctly | Use ZAP replacer or Nuclei `-H` flag for auth headers |
| GraphQL scanning fails | ZAP doesn't understand GraphQL well | Use InQL Burp extension or dedicated GraphQL scanners |
| IDOR testing produces false positives | Public endpoints returning 200 for all IDs | Verify response body differs between users, not just status code |
| Rate limit tests block scanner IP | Scanner IP gets rate-limited | Whitelist scanner IP on staging; test rate limits separately |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **BOLA / IDOR** | Accessing other users' resources by changing IDs |
| **Mass Assignment** | Setting unauthorized fields in API requests |
| **Rate Limiting** | Restricting request frequency to prevent abuse |
| **JWT Testing** | Checking token algorithm, expiry, and signature |
| **CORS Testing** | Verifying cross-origin policy configuration |
| **GraphQL Introspection** | Schema disclosure via `__schema` query |
| **API Specification** | OpenAPI/Swagger doc driving automated API scanning |

---

## Quick Revision Questions

1. **What is BOLA and how do you test for it?**
   > Broken Object Level Authorization (BOLA), also called IDOR, occurs when an API doesn't verify that a user has permission to access the requested object. Test by authenticating as User A and requesting User B's resources (e.g., `/api/users/B/orders`). A vulnerable API returns 200 instead of 403.

2. **Why are APIs harder to test with traditional DAST?**
   > APIs don't have HTML links for crawlers to follow, use JSON instead of forms, require authentication tokens, may use GraphQL or gRPC instead of REST, and their attack surface is defined by the API specification rather than visible UI elements.

3. **What is a mass assignment vulnerability?**
   > When an API endpoint blindly binds request body fields to internal data models, allowing attackers to set fields they shouldn't (e.g., `role: admin`). Prevent by using explicit allowlists of accepted fields (DTOs).

4. **How should GraphQL APIs be secured against DoS?**
   > Disable introspection in production, implement query depth limiting, set query complexity limits, restrict batch queries, use persisted queries, and apply rate limiting per operation (not just per HTTP request).

5. **What is the OWASP API Security Top 10 #1 risk?**
   > Broken Object Level Authorization (BOLA) — APIs failing to verify that the authenticated user has permission to access the specific object they're requesting. It's the most common and exploited API vulnerability.

---

[← Previous: 4.3 Automated Scanning](03-automated-scanning.md) | [Next: 4.5 DAST Pipeline Integration →](05-dast-pipeline-integration.md)

[Back to Table of Contents](../README.md)
