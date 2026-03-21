# Unit 1: A01 - Broken Access Control — Topic 3: IDOR Vulnerabilities

## Overview

Insecure Direct Object References (IDOR) occur when an application uses **user-supplied input to directly access objects** (database records, files, resources) without verifying the requesting user is authorized to access that specific object. IDOR is one of the most commonly found and exploited access control flaws, responsible for major data breaches at companies like Facebook, Uber, and Shopify. It consistently ranks as one of the top vulnerability types in bug bounty programs.

---

## 1. How IDOR Works

### Core Concept

```
┌─────────────────────────────────────────────────────────────┐
│                      IDOR VULNERABILITY                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Legitimate Request (User A accessing their own data):      │
│  GET /api/invoices/1001                                     │
│  Cookie: session=userA_token                                │
│  → 200 OK { "id": 1001, "user": "A", "amount": "$500" }   │
│                                                             │
│  IDOR Attack (User A accessing User B's data):             │
│  GET /api/invoices/1002    ← Changed ID from 1001 to 1002  │
│  Cookie: session=userA_token                                │
│  → 200 OK { "id": 1002, "user": "B", "amount": "$2000" }  │
│                                                             │
│  The server authenticated User A but did NOT check if       │
│  User A is authorized to access invoice 1002 (owned by B)  │
└─────────────────────────────────────────────────────────────┘
```

### IDOR Decision Tree

```
User requests resource with ID X
         │
         ▼
  Is user authenticated?
    │              │
   NO             YES
    │              │
    ▼              ▼
  401          Does user OWN resource X?
              (or have explicit access)
                │              │
               NO             YES
                │              │
                ▼              ▼
              403          200 OK
           (IDOR if        (Correct
            returns        behavior)
            200 here)
```

---

## 2. IDOR Types and Locations

### Where IDORs Occur

| Location | Example | Risk |
|----------|---------|------|
| **URL Path** | `/api/users/123/profile` | High — easily discoverable |
| **Query Parameters** | `/download?file_id=456` | High — visible in URL |
| **POST Body** | `{"invoice_id": 789}` | Medium — requires interception |
| **HTTP Headers** | `X-User-Id: 123` | Medium — custom headers |
| **Cookies** | `user_id=123` | Medium — cookie manipulation |
| **File Names** | `/uploads/user_123_resume.pdf` | High — predictable names |
| **GraphQL** | `query { user(id: 123) { ... } }` | High — parameter in query |
| **WebSocket** | `{"action": "getUser", "id": 123}` | Low — less tested |

### IDOR Categories

```
┌─────────────────────────────────────────────────────────────┐
│                     IDOR CATEGORIES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. HORIZONTAL IDOR                                        │
│     User A accesses User B's data (same privilege level)   │
│     /api/users/A/profile → /api/users/B/profile            │
│                                                             │
│  2. VERTICAL IDOR                                          │
│     Regular user accesses admin resources                   │
│     /api/users/me/settings → /api/admin/settings           │
│                                                             │
│  3. DATA REFERENCE IDOR                                    │
│     Accessing data objects (files, records, documents)      │
│     /documents/my_doc.pdf → /documents/secret_doc.pdf      │
│                                                             │
│  4. FUNCTION REFERENCE IDOR                                │
│     Accessing functions/operations not authorized           │
│     /api/orders/123/view → /api/orders/123/refund          │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Identifier Types and Predictability

### Sequential vs Random Identifiers

| ID Type | Example | Predictability | IDOR Risk |
|---------|---------|---------------|-----------|
| **Sequential Integer** | `1, 2, 3, 4, 5` | Trivially predictable | **Critical** |
| **Timestamps** | `1704067200` | Easily guessable | **High** |
| **Encoded IDs** | `MTIz` (base64 of "123") | Decode and enumerate | **High** |
| **Short Hash** | `a1b2c3` | Brute-forceable | **Medium** |
| **UUID v1** | `550e8400-e29b-41d4-a716-446655440000` | Contains timestamp | **Medium** |
| **UUID v4** | `f47ac10b-58cc-4372-a567-0e02b2c3d479` | Random, 2^122 space | **Low** |
| **Hashed IDs** | `Hashids("salt", 123)` → `jR` | Requires salt knowledge | **Low** |

> **Important:** UUIDs reduce IDOR risk but do **NOT eliminate it**. If a UUID leaks through another endpoint (e.g., `/api/users` list), the IDOR is still exploitable.

### Encoded ID Bypass

```python
# Base64 encoded IDs
import base64

# Server uses: /api/users/MTIz (base64 of "123")
for user_id in range(1, 1000):
    encoded = base64.b64encode(str(user_id).encode()).decode()
    print(f"User {user_id} → /api/users/{encoded}")
    # User 1 → /api/users/MQ==
    # User 2 → /api/users/Mg==
    # User 123 → /api/users/MTIz

# Hex encoded IDs
for user_id in range(1, 1000):
    hex_id = hex(user_id)[2:]
    print(f"User {user_id} → /api/users/{hex_id}")

# Hashed IDs (MD5/SHA1 of sequential)
import hashlib
for user_id in range(1, 1000):
    hashed = hashlib.md5(str(user_id).encode()).hexdigest()
    print(f"User {user_id} → /api/users/{hashed}")
```

---

## 4. IDOR Testing Methodology

### Step-by-Step Testing Process

```
┌──────────────────────────────────────────────────────────────┐
│                IDOR TESTING METHODOLOGY                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: CREATE TWO TEST ACCOUNTS                           │
│  ┌──────────┐  ┌──────────┐                                 │
│  │ Account A │  │ Account B │                                │
│  │ (Attacker)│  │ (Victim)  │                                │
│  └──────────┘  └──────────┘                                 │
│                                                              │
│  Step 2: MAP ALL ENDPOINTS THAT USE IDs                     │
│  GET  /api/users/{id}                                        │
│  GET  /api/orders/{id}                                       │
│  PUT  /api/profile/{id}                                      │
│  POST /api/documents/{id}/share                              │
│                                                              │
│  Step 3: CAPTURE Account A's REQUESTS                       │
│  Note all IDs used by Account A                              │
│                                                              │
│  Step 4: REPLACE IDs WITH Account B's IDs                   │
│  Using Account A's session, request Account B's resources    │
│                                                              │
│  Step 5: ANALYZE RESPONSES                                   │
│  - 200 OK with B's data = IDOR CONFIRMED                    │
│  - 403/404 = Properly protected                              │
│  - 200 OK with A's data = Server ignores ID (safe)          │
│                                                              │
│  Step 6: TEST CRUD OPERATIONS                               │
│  Read (GET), Update (PUT/PATCH), Delete (DELETE)             │
│  Each may have different access controls                     │
└──────────────────────────────────────────────────────────────┘
```

### Burp Suite Autorize Extension

```
┌────────────────────────────────────────────────────────────────┐
│                    AUTORIZE WORKFLOW                           │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. Install Autorize from BApp Store                          │
│                                                                │
│  2. Configure:                                                │
│     • Set low-privilege user's cookies in Autorize            │
│     • Enable interception                                     │
│                                                                │
│  3. Browse as HIGH-privilege user (admin)                     │
│     Autorize automatically replays each request with          │
│     low-privilege cookies                                      │
│                                                                │
│  4. Results:                                                   │
│     ┌──────────┬──────────┬──────────┬──────────┐            │
│     │ URL      │ Original │ Modified │ Status   │            │
│     ├──────────┼──────────┼──────────┼──────────┤            │
│     │/api/admin│ 200      │ 200      │ BYPASSED!│            │
│     │/api/users│ 200      │ 403      │ ENFORCED │            │
│     │/api/logs │ 200      │ 200      │ BYPASSED!│            │
│     └──────────┴──────────┴──────────┴──────────┘            │
│                                                                │
│  Color coding: Red = Bypass, Green = Enforced                 │
└────────────────────────────────────────────────────────────────┘
```

### Automated IDOR Testing Script

```python
#!/usr/bin/env python3
"""Automated IDOR tester using two authenticated sessions"""
import requests
import json
import sys

class IDORTester:
    def __init__(self, base_url, token_a, token_b):
        self.base_url = base_url
        self.session_a = requests.Session()
        self.session_b = requests.Session()
        self.session_a.headers['Authorization'] = f'Bearer {token_a}'
        self.session_b.headers['Authorization'] = f'Bearer {token_b}'
        self.session_a.verify = False
        self.session_b.verify = False
        self.findings = []
    
    def test_endpoint(self, method, path, id_param, id_range):
        """Test an endpoint for IDOR"""
        print(f"\n[*] Testing {method} {path} (param: {id_param})")
        
        for obj_id in id_range:
            url = f"{self.base_url}{path}".replace(f'{{{id_param}}}', str(obj_id))
            
            # Request as User A (should only access own resources)
            resp_a = getattr(self.session_a, method.lower())(url)
            
            # Request as User B (cross-check)
            resp_b = getattr(self.session_b, method.lower())(url)
            
            if resp_a.status_code == 200 and resp_b.status_code == 200:
                # Both users can access — check if data differs
                try:
                    data_a = resp_a.json()
                    data_b = resp_b.json()
                    if data_a == data_b:
                        # Same data returned — could be public or IDOR
                        print(f"  [?] ID {obj_id}: Both users get same data (investigate)")
                    else:
                        print(f"  [!] ID {obj_id}: Different data returned — potential IDOR!")
                        self.findings.append({
                            'endpoint': url,
                            'method': method,
                            'id': obj_id,
                            'severity': 'HIGH'
                        })
                except:
                    pass
            elif resp_a.status_code == 200 and resp_b.status_code in (403, 404):
                print(f"  [✓] ID {obj_id}: Properly protected")
    
    def report(self):
        if self.findings:
            print(f"\n{'='*60}")
            print(f"[!] FOUND {len(self.findings)} POTENTIAL IDOR VULNERABILITIES")
            print(f"{'='*60}")
            for f in self.findings:
                print(f"  [{f['severity']}] {f['method']} {f['endpoint']}")
        else:
            print("\n[✓] No IDOR vulnerabilities found")

# Usage
tester = IDORTester(
    'https://target.com',
    'eyJ...token_a...',
    'eyJ...token_b...'
)
tester.test_endpoint('GET', '/api/users/{id}/profile', 'id', range(1, 50))
tester.test_endpoint('GET', '/api/orders/{id}', 'id', range(100, 200))
tester.report()
```

---

## 5. Real-World IDOR Examples

### Facebook (2015) — Photo Album Deletion

```
Vulnerability: Any user could delete any other user's photo album
Endpoint: POST /ajax/photos/album/delete
Parameter: album_id (sequential integer)

Attack:
POST /ajax/photos/album/delete HTTP/1.1
Cookie: [attacker's session]
Content-Type: application/x-www-form-urlencoded

album_id=VICTIM_ALBUM_ID&fb_dtsg=TOKEN

Result: Album deleted without ownership check
Bounty: $12,500
```

### Uber (2019) — Trip History Access

```
Vulnerability: View any rider's trip details via UUID in API
Endpoint: GET /api/riders/{rider_uuid}/trips
Issue: UUID was leaked through another endpoint

Attack flow:
1. Use partner API to enumerate rider UUIDs
2. GET /api/riders/{victim_uuid}/trips
3. Response: Full trip history, routes, payment info

Impact: PII of millions of riders exposed
```

### Shopify (2018) — Store Revenue Data

```
Vulnerability: Access any store's revenue through IDOR
Endpoint: GET /admin/api/2019-01/reports/{report_id}.json
Parameter: report_id (sequential)

Attack:
GET /admin/api/2019-01/reports/1001.json
Authorization: [attacker's token for different store]

Result: Full revenue reports of competitor stores
Bounty: $15,000
```

---

## 6. Advanced IDOR Techniques

### Parameter Pollution

```
# Some frameworks take the last parameter, some the first
GET /api/users?id=attacker_id&id=victim_id
GET /api/users?id=victim_id&id=attacker_id

# JSON parameter pollution
POST /api/update
{"user_id": "attacker", "user_id": "victim"}  ← Parser might use second value

# Array injection
GET /api/users?id[]=attacker_id&id[]=victim_id
```

### ID Wrapping and Type Juggling

```python
# If server uses loose comparison, try type juggling
# Original: /api/users/123
# Test with:
/api/users/123.0       # Float
/api/users/0x7B        # Hex (123)
/api/users/0173        # Octal (123)
/api/users/00123       # Leading zeros
/api/users/123%00      # Null byte
/api/users/{"id":123}  # JSON in URL
```

### Chained IDOR

```
Step 1: Find endpoint that leaks IDs
  GET /api/comments → returns user_ids of commenters

Step 2: Use leaked IDs for IDOR
  GET /api/users/{leaked_user_id}/private-data

Step 3: Chain with state-changing operations
  PUT /api/users/{leaked_user_id}/email
  {"email": "attacker@evil.com"}
  → Account takeover via password reset
```

---

## 7. Prevention and Remediation

### Secure Implementation Pattern

```python
# VULNERABLE — No ownership check
@app.route('/api/documents/<int:doc_id>')
@login_required
def get_document(doc_id):
    doc = Document.query.get_or_404(doc_id)
    return jsonify(doc.to_dict())

# SECURE — Ownership check
@app.route('/api/documents/<int:doc_id>')
@login_required
def get_document(doc_id):
    doc = Document.query.filter_by(
        id=doc_id,
        owner_id=current_user.id  # Filter by authenticated user
    ).first_or_404()
    return jsonify(doc.to_dict())

# EVEN BETTER — Use indirect references
@app.route('/api/documents/<int:index>')
@login_required
def get_document(index):
    # Map user's index to actual document ID
    user_docs = Document.query.filter_by(owner_id=current_user.id).all()
    if index < 0 or index >= len(user_docs):
        abort(404)
    return jsonify(user_docs[index].to_dict())
```

### Prevention Checklist

| Strategy | Description |
|----------|-------------|
| **Server-side ownership checks** | Always verify the requesting user owns/has access to the resource |
| **Indirect object references** | Map user-specific index to actual IDs server-side |
| **UUID v4 identifiers** | Use random UUIDs instead of sequential integers |
| **Row-level security** | Implement database-level access control (PostgreSQL RLS) |
| **Scoped queries** | Always include user_id in database queries |
| **Automated testing** | Use Autorize and custom scripts in CI/CD |
| **Rate limiting** | Limit enumeration attempts |
| **Audit logging** | Log all access to detect enumeration patterns |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **What is IDOR** | Accessing objects by manipulating user-supplied identifiers |
| **Root Cause** | Missing server-side authorization check for resource ownership |
| **Horizontal** | Same-privilege user accessing another user's data |
| **Vertical** | Lower-privilege user accessing higher-privilege resources |
| **Key Tools** | Burp Autorize, custom Python scripts, ffuf |
| **Prevention** | Ownership checks, indirect references, UUIDs, scoped queries |
| **Bug Bounty Avg** | $500–$15,000 depending on impact |

---

## Revision Questions

1. What is the difference between horizontal and vertical IDOR? Give an example of each.
2. Why do UUID v4 identifiers reduce but not eliminate IDOR risk?
3. Describe a step-by-step methodology for testing IDOR using two authenticated accounts.
4. How does the Burp Suite Autorize extension automate IDOR testing? What does it compare?
5. Write a Python script that tests for IDOR on a parameterized API endpoint using two different auth tokens.
6. Explain a chained IDOR attack where leaking IDs from one endpoint enables exploitation of another.

---

*Previous: [02-common-weaknesses.md](02-common-weaknesses.md) | Next: [04-privilege-escalation.md](04-privilege-escalation.md)*

---

*[Back to README](../README.md)*
