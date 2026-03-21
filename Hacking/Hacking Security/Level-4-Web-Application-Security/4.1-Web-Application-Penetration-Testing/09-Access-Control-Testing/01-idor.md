# 01 — Insecure Direct Object References (IDOR)

## Overview

An **Insecure Direct Object Reference (IDOR)** occurs when an application exposes a direct reference to an internal implementation object—such as a database key, filename, or directory path—without proper authorization checks. Attackers manipulate these references to access resources belonging to other users or at higher privilege levels. IDOR consistently ranks among the most impactful web-application vulnerabilities and is a staple of bug-bounty programs. It maps directly to **OWASP Top 10 A01:2021 – Broken Access Control** and **CWE-639: Authorization Bypass Through User-Controlled Key**.

---

## 1. IDOR Fundamentals

### 1.1 What Is an Object Reference?

Every time a web application must retrieve, update, or delete a resource it needs an **identifier**. That identifier is the *object reference*.

```
+──────────────────────────────────────────────────────────+
│                    Object Reference Flow                 │
+──────────────────────────────────────────────────────────+
│                                                          │
│   Browser ──► Request with Reference ──► Server          │
│                                                          │
│   GET /api/invoices/4827                                 │
│                  ▲                                       │
│                  │                                       │
│        Object Reference (4827)                           │
│        exposed in the URL                                │
│                                                          │
│   Server ──► DB Query ──► Returns Invoice 4827           │
│   (No ownership check!)                                  │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 1.2 Why IDOR Happens

| Root Cause | Description |
|---|---|
| **Missing authorization check** | Server retrieves the object without verifying the requesting user owns or may access it. |
| **Predictable identifiers** | Auto-incrementing integers (`id=1, 2, 3…`) make enumeration trivial. |
| **Client-side trust** | Application trusts the identifier that the client sends without validation. |
| **Lack of indirect mapping** | No abstraction layer between the user-facing reference and the internal key. |

### 1.3 IDOR vs Other Access-Control Flaws

| Aspect | IDOR | Broken Access Control (general) | Path Traversal |
|---|---|---|---|
| **Target** | Database records / API objects | Any restricted function or data | Files on the server filesystem |
| **Vector** | Change a parameter value | Missing checks on endpoints | Manipulate file paths (`../`) |
| **CWE** | CWE-639 | CWE-284 | CWE-22 |
| **Example** | `/user/102` → `/user/103` | Access `/admin/dashboard` as normal user | `/download?file=../../../etc/passwd` |

---

## 2. Types of IDOR

### 2.1 Horizontal IDOR

Access resources belonging to **another user at the same privilege level**.

```
+────────────────────────────────────────────────+
│              Horizontal IDOR                   │
+────────────────────────────────────────────────+
│                                                │
│  User A (id=101)                               │
│    Session: abc123                              │
│    Request: GET /api/profile/102   ◄── change  │
│                                                │
│  Server returns User B's (id=102) profile      │
│  WITHOUT checking that abc123 belongs to 102   │
│                                                │
+────────────────────────────────────────────────+
```

**Real-World Example — Facebook (2015):**  
A researcher changed the `photo_id` parameter in a deletion request and successfully deleted other users' photos, earning a $12,500 bounty.

### 2.2 Vertical IDOR

Access resources at a **higher privilege level** (e.g., standard user reads admin data).

```
+────────────────────────────────────────────────+
│              Vertical IDOR                     │
+────────────────────────────────────────────────+
│                                                │
│  Regular User (role=user)                      │
│    Request: GET /api/admin/reports/55           │
│                                                │
│  Server returns admin-only report #55          │
│  because it checks authentication but NOT      │
│  authorization (role).                         │
│                                                │
+────────────────────────────────────────────────+
```

### 2.3 Object-Level Authorization (BOLA)

OWASP API Security names this **Broken Object Level Authorization (BOLA)** — the API equivalent of IDOR. It is the #1 risk in the OWASP API Top 10.

```
+────────────────────────────────────────────────────────+
│                    BOLA in REST APIs                   │
+────────────────────────────────────────────────────────+
│                                                        │
│  Authenticated User A (JWT token)                      │
│                                                        │
│  GET  /api/v2/orders/9001   → User A's order    ✓     │
│  GET  /api/v2/orders/9002   → User B's order    ✗ BUG │
│  PUT  /api/v2/orders/9002   → Modify B's order  ✗ BUG │
│  DELETE /api/v2/orders/9002 → Delete B's order  ✗ BUG │
│                                                        │
│  All operations succeed because the API checks         │
│  "is the user authenticated?" but NOT                  │
│  "does this user own order 9002?"                      │
│                                                        │
+────────────────────────────────────────────────────────+
```

### 2.4 IDOR Type Comparison

| Type | Privilege Change | Impact | Example |
|---|---|---|---|
| Horizontal | Same level → different user | Data leak / modification of peer data | View another customer's order |
| Vertical | Lower → higher privilege | Full privilege escalation | Regular user reads admin logs |
| BOLA (API) | Either | Mass data exfiltration via API | Enumerate all user records via API |

---

## 3. Where IDOR Appears — Attack Surfaces

### 3.1 URL Parameters (GET)

```http
GET /account/settings?user_id=1042 HTTP/1.1
Host: example.com
Cookie: session=eyJhbGciOi...
```

Change `user_id=1042` → `user_id=1043` and observe if another user's settings are returned.

### 3.2 POST / PUT Body Parameters

```http
PUT /api/v1/address HTTP/1.1
Content-Type: application/json
Authorization: Bearer eyJ...

{
  "address_id": 554,
  "street": "123 Hacker Lane",
  "city": "Pwned City"
}
```

Modify `address_id` to another user's address record.

### 3.3 REST API Path Segments

```
GET /api/users/782/documents/19       ← your document
GET /api/users/782/documents/20       ← another user's document (same user context)
GET /api/users/783/documents/19       ← another user entirely
```

### 3.4 File References

```http
GET /download?file=report_1042.pdf
GET /download?file=report_1043.pdf     ← another user's report
```

### 3.5 GraphQL Queries

```graphql
query {
  user(id: "1043") {
    email
    phone
    ssn
  }
}
```

### 3.6 Hidden Form Fields

```html
<form action="/update-profile" method="POST">
  <input type="hidden" name="uid" value="1042">
  <input type="text" name="email" value="me@example.com">
  <button type="submit">Save</button>
</form>
```

Modify `uid` in Burp or browser DevTools before submission.

### 3.7 Cookie / Header Values

```http
GET /dashboard HTTP/1.1
Cookie: session=abc123; user_id=1042
```

Some applications embed the object reference **in a cookie** rather than the URL.

### 3.8 Attack Surface Summary

| Location | HTTP Verb | Detection Difficulty | Frequency |
|---|---|---|---|
| URL query parameter | GET | Easy | Very Common |
| Request body (JSON/XML) | POST / PUT | Medium | Common |
| URL path segment (REST) | Any | Easy | Common |
| File reference parameter | GET | Easy | Moderate |
| GraphQL argument | POST | Medium | Growing |
| Hidden form field | POST | Easy | Common |
| Cookie value | Any | Medium | Rare |
| Custom HTTP header | Any | Hard | Rare |

---

## 4. Testing Methodology

### 4.1 Manual Testing Workflow

```
+──────────────────────────────────────────────────────────────+
│                IDOR Testing Workflow                         │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Step 1: Map the Application                                 │
│    ├── Identify every parameter that references an object    │
│    ├── Note parameter names: id, uid, user_id, doc_id, etc.  │
│    └── Record legitimate values for your test accounts       │
│                                                              │
│  Step 2: Create Multiple Test Accounts                       │
│    ├── Account A (attacker-controlled)                       │
│    ├── Account B (victim simulation)                         │
│    └── Record each account's object IDs                      │
│                                                              │
│  Step 3: Swap References                                     │
│    ├── Use Account A's session                               │
│    ├── Replace A's object IDs with B's IDs                   │
│    └── Observe response (data leak? modification?)           │
│                                                              │
│  Step 4: Enumerate                                           │
│    ├── Increment/decrement numeric IDs                       │
│    ├── Try common UUIDs or pattern-based guesses             │
│    └── Use Burp Intruder for automated enumeration           │
│                                                              │
│  Step 5: Test All HTTP Methods                               │
│    ├── GET   → Read other user's data                        │
│    ├── PUT   → Modify other user's data                      │
│    ├── DELETE → Delete other user's data                     │
│    └── PATCH  → Partial modification                         │
│                                                              │
│  Step 6: Document & Report                                   │
│    ├── Capture request/response pairs                        │
│    ├── Calculate severity (CVSS)                             │
│    └── Recommend remediation                                 │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 4.2 Parameter Discovery Checklist

| Parameter Name Pattern | Example | Notes |
|---|---|---|
| `id`, `uid`, `user_id` | `?user_id=42` | Most common |
| `account`, `acct` | `?acct=10045` | Financial apps |
| `order_id`, `invoice_id` | `/orders/9001` | E-commerce |
| `doc`, `file`, `report` | `?doc=annual_2024.pdf` | Document management |
| `email` | `?email=victim@corp.com` | Sometimes used as key |
| `phone`, `ssn` | `?phone=5551234567` | PII-based lookups |
| Encoded / hashed IDs | `?ref=MTAyMw==` (base64) | Decode and manipulate |

### 4.3 ID Format Analysis

```
+────────────────────────────────────────────────────────────+
│                   ID Format Decision Tree                  │
+────────────────────────────────────────────────────────────+
│                                                            │
│   Identify the parameter value format                      │
│                                                            │
│   ┌─ Numeric (e.g., 1042)                                  │
│   │   └── Try: 1041, 1043, 1, 0, -1, 99999                │
│   │                                                        │
│   ├─ UUID (e.g., 550e8400-e29b-41d4-a716-446655440000)     │
│   │   └── Harvest UUIDs from other endpoints / responses   │
│   │       Check for UUID v1 (timestamp-based, predictable) │
│   │                                                        │
│   ├─ Base64 (e.g., MTAyMw==)                               │
│   │   └── Decode: echo MTAyMw== | base64 -d  → "1023"     │
│   │       Encode new value: echo -n 1024 | base64          │
│   │                                                        │
│   ├─ Hash (e.g., MD5 / SHA1)                               │
│   │   └── Determine if hash of sequential integer          │
│   │       hashcat -m 0 hash.txt -a 3 ?d?d?d?d              │
│   │                                                        │
│   └─ Custom / Encoded                                      │
│       └── Reverse-engineer pattern from multiple samples   │
│                                                            │
+────────────────────────────────────────────────────────────+
```

---

## 5. Tool-Based Testing

### 5.1 Burp Suite — Intruder for IDOR Enumeration

**Step-by-step:**

1. Capture a request in Burp Proxy that contains an object reference.
2. Send it to **Intruder** (`Ctrl+I`).
3. Mark the object-reference parameter as a payload position.
4. Configure payload:

```
Payload type: Numbers
From: 1
To: 10000
Step: 1
```

5. In **Options → Grep – Extract**, add a pattern that distinguishes success (e.g., `"email":"`).
6. Start attack and sort results by response length or grep column.

**Identifying Vulnerable Responses:**

| Indicator | Meaning |
|---|---|
| Response length varies | Different data returned → likely IDOR |
| HTTP 200 for foreign IDs | Server returned data without auth check |
| HTTP 403 for some, 200 for others | Inconsistent access control |
| Same response for every ID | Likely not vulnerable (or static data) |

### 5.2 Burp Suite — Autorize Extension

Autorize automates authorization testing by replaying requests with a lower-privilege session.

```
+──────────────────────────────────────────────────────────+
│                  Autorize Workflow                       │
+──────────────────────────────────────────────────────────+
│                                                          │
│  1. Install Autorize from BApp Store                     │
│  2. Log in as LOW-PRIVILEGE user                         │
│  3. Copy that user's cookies/token into Autorize config  │
│  4. Browse the app as HIGH-PRIVILEGE user                │
│  5. Autorize automatically replays every request with    │
│     the low-privilege token                              │
│  6. Results:                                             │
│     ┌────────────────────────────────────────────┐       │
│     │ Request URL       │ Orig │ Modified │ Auth │       │
│     ├───────────────────┼──────┼──────────┼──────┤       │
│     │ /api/admin/users  │ 200  │ 200      │ FAIL │       │
│     │ /api/user/profile │ 200  │ 200      │ FAIL │       │
│     │ /api/admin/config │ 200  │ 403      │ PASS │       │
│     └────────────────────────────────────────────┘       │
│                                                          │
│  "FAIL" = authorization is BYPASSED (vulnerability!)     │
│  "PASS" = access correctly denied                        │
│                                                          │
+──────────────────────────────────────────────────────────+
```

**Autorize Configuration Tips:**

```
# In Autorize → Configuration tab:

# 1. Set the replacement header (low-priv session)
Cookie: session=LOW_PRIV_SESSION_TOKEN_HERE

# 2. Enforcement Detector — add patterns for "access denied"
#    Autorize uses these to decide PASS vs FAIL
"error": "unauthorized"
"error": "forbidden"
HTTP/1.1 403
HTTP/1.1 401

# 3. Enable "Check unauthenticated" to also test with NO session
```

### 5.3 ffuf — Fuzzing Object References

```bash
# Enumerate numeric user IDs
ffuf -u "https://target.com/api/users/FUZZ/profile" \
     -w <(seq 1 10000) \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -mc 200 \
     -o idor_results.json

# Enumerate with a wordlist of known UUIDs
ffuf -u "https://target.com/api/documents/FUZZ" \
     -w uuids.txt \
     -H "Cookie: session=abc123" \
     -mc 200 \
     -fs 0
```

### 5.4 Custom Python Script

```python
import requests

BASE_URL = "https://target.com/api/users/{}/profile"
ATTACKER_TOKEN = "Bearer eyJhbGciOiJIUzI1NiIs..."
HEADERS = {"Authorization": ATTACKER_TOKEN}

# Test IDs 1 through 500
for user_id in range(1, 501):
    url = BASE_URL.format(user_id)
    resp = requests.get(url, headers=HEADERS)
    
    if resp.status_code == 200:
        data = resp.json()
        print(f"[IDOR] ID={user_id} | Email={data.get('email','N/A')} | "
              f"Name={data.get('name','N/A')}")
    elif resp.status_code == 403:
        pass  # Access correctly denied
    else:
        print(f"[INFO] ID={user_id} | Status={resp.status_code}")
```

### 5.5 Postman — Collection Runner

1. Create a collection with the vulnerable request.
2. Use a variable `{{user_id}}` for the object reference.
3. Supply a CSV data file:

```csv
user_id
100
101
102
103
```

4. Run the Collection Runner and inspect each response.

---

## 6. Advanced IDOR Techniques

### 6.1 Bypassing UUID-Based Protection

UUIDs are not secret. Harvest them from:

| Source | Technique |
|---|---|
| Other API responses | List endpoints sometimes leak UUIDs of other users |
| Public profiles | UUID in profile URL |
| Error messages | Stack traces may include UUIDs |
| UUID v1 | Timestamp + MAC-based — generate adjacent UUIDs |
| Websocket messages | Real-time features may broadcast UUIDs |
| HTML source / JS files | Hardcoded references |
| Password reset tokens | May correlate with user UUIDs |

**UUID v1 Time-Based Prediction:**

```python
import uuid

# UUID v1 encodes timestamp — if you know one, predict neighbors
known = uuid.UUID("6ba7b810-9dad-11d1-80b4-00c04fd430c8")
timestamp = known.time  # 100-ns intervals since Oct 15, 1582

# Generate adjacent UUIDs by adding/subtracting time intervals
for offset in range(-100, 101):
    predicted = uuid.UUID(int=known.int + offset, version=1)
    print(predicted)
```

### 6.2 Encoded / Hashed Reference Manipulation

```bash
# Base64 encoded IDs
echo -n "user_1042" | base64
# dXNlcl8xMDQy

echo -n "user_1043" | base64
# dXNlcl8xMDQz

# URL: /api/data?ref=dXNlcl8xMDQy → change to dXNlcl8xMDQz

# MD5 hashed IDs (if pattern is known)
echo -n "1042" | md5sum
# → 3def184ad8f4755ff269862ea77393dd

echo -n "1043" | md5sum
# → use this hash as the parameter value
```

### 6.3 IDOR in Multi-Step Processes

```
+──────────────────────────────────────────────────────────+
│             Multi-Step IDOR Attack                       │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Step 1: POST /checkout/init                             │
│          Body: { "cart_id": 5001 }                       │
│          → Server validates: cart 5001 belongs to you ✓  │
│                                                          │
│  Step 2: POST /checkout/confirm                          │
│          Body: { "cart_id": 5001, "address_id": 302 }    │
│          → Server validates cart_id again ✓              │
│          → But does NOT validate address_id!             │
│          → Change address_id to 303 (victim's address)   │
│          → Order shipped to attacker's address!          │
│                                                          │
│  Step 3: POST /checkout/pay                              │
│          Body: { "order_id": 7890, "payment_id": 44 }    │
│          → Change payment_id to 45 (victim's card)       │
│          → Victim is charged!                            │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 6.4 IDOR via HTTP Method Switching

```http
# GET is protected
GET /api/users/1043/profile HTTP/1.1   → 403 Forbidden

# But PUT is not
PUT /api/users/1043/profile HTTP/1.1   → 200 OK
{ "email": "attacker@evil.com" }

# Or try lesser-used methods
PATCH /api/users/1043/profile HTTP/1.1 → 200 OK
```

### 6.5 IDOR via Parameter Pollution

```http
# Original request
GET /api/profile?id=1042

# HTTP Parameter Pollution — add duplicate parameter
GET /api/profile?id=1042&id=1043

# Some frameworks take the first, others take the last
# PHP: last value → id=1043
# ASP.NET: comma-joined → id=1042,1043
# Flask: first value → id=1042
```

| Framework | Behavior with `?id=1042&id=1043` |
|---|---|
| PHP | Uses **last** value: `1043` |
| ASP.NET | Joins: `1042,1043` |
| Node.js (Express) | Returns array: `['1042','1043']` |
| Python (Flask) | Uses **first** value: `1042` |
| Java (Spring) | Uses **first** value: `1042` |
| Ruby on Rails | Uses **last** value: `1043` |

---

## 7. Real-World Bug Bounty Examples

### 7.1 Notable IDOR Disclosures

| Program | Vulnerability | Impact | Bounty |
|---|---|---|---|
| **Facebook** | Delete any user's photo via `photo_id` manipulation | Mass photo deletion | $12,500 |
| **Uber** | View any driver's trip history by changing `driver_id` | PII exposure of all drivers | $5,000 |
| **Shopify** | Change `shop_id` in admin API to access other merchants | Full store data access | $15,000 |
| **Twitter** | View any user's ad analytics by modifying account ID | Business data exposure | $2,520 |
| **US Dept. of Defense** | Access other soldiers' records via sequential IDs | Military PII exposure | Hall of Fame |
| **GitLab** | Enumerate private snippets by incrementing snippet ID | Source code leakage | $3,000 |
| **Airbnb** | Access other hosts' earnings via `user_id` in API call | Financial data exposure | $3,500 |

### 7.2 Detailed Case Study — Uber IDOR

```
+──────────────────────────────────────────────────────────+
│              Uber IDOR — Trip History Leak               │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Endpoint: GET /api/v1/drivers/{driver_id}/trips         │
│                                                          │
│  1. Researcher signed up as Uber driver → driver_id=X    │
│  2. Noticed GET /api/v1/drivers/X/trips returned trips   │
│  3. Changed X to X+1, X+2, etc.                         │
│  4. Each request returned OTHER drivers' full trip data:  │
│     - Pickup/dropoff addresses                           │
│     - Rider names and ratings                            │
│     - Fare amounts                                       │
│     - Trip timestamps                                    │
│  5. No rate limiting → bulk enumeration possible         │
│                                                          │
│  Root Cause: API validated JWT (authentication) but      │
│  did NOT check if the JWT's user owned the driver_id     │
│  (missing authorization check).                          │
│                                                          │
│  Fix: Added server-side ownership verification:          │
│  if (jwt.user_id != driver_id) return 403;               │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## 8. Prevention & Remediation

### 8.1 Indirect Object References

Replace direct database IDs with per-session random mappings:

```python
# VULNERABLE — direct reference
@app.route('/api/documents/<int:doc_id>')
def get_document(doc_id):
    doc = Document.query.get(doc_id)       # No ownership check!
    return jsonify(doc.to_dict())

# SECURE — indirect reference map
import secrets

# On login, build a mapping for this session
def build_reference_map(user):
    docs = Document.query.filter_by(owner_id=user.id).all()
    ref_map = {}
    for doc in docs:
        token = secrets.token_urlsafe(16)
        ref_map[token] = doc.id
    session['doc_refs'] = ref_map
    return ref_map

@app.route('/api/documents/<string:ref_token>')
def get_document(ref_token):
    doc_refs = session.get('doc_refs', {})
    real_id = doc_refs.get(ref_token)
    if real_id is None:
        abort(404)
    doc = Document.query.get(real_id)
    return jsonify(doc.to_dict())
```

### 8.2 Server-Side Ownership Verification

```python
# SECURE — ownership check on every request
@app.route('/api/documents/<int:doc_id>')
@login_required
def get_document(doc_id):
    doc = Document.query.filter_by(
        id=doc_id,
        owner_id=current_user.id          # ← ownership check
    ).first_or_404()
    return jsonify(doc.to_dict())
```

```javascript
// Node.js / Express equivalent
app.get('/api/documents/:docId', auth, async (req, res) => {
  const doc = await Document.findOne({
    _id: req.params.docId,
    owner: req.user._id               // ← ownership check
  });
  if (!doc) return res.status(404).json({ error: 'Not found' });
  res.json(doc);
});
```

### 8.3 Authorization Middleware Pattern

```python
# Centralized authorization decorator
def authorize_resource(model, id_param='id', owner_field='owner_id'):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            resource_id = kwargs.get(id_param) or request.args.get(id_param)
            resource = model.query.get_or_404(resource_id)
            if getattr(resource, owner_field) != current_user.id:
                abort(403)
            kwargs['resource'] = resource
            return f(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@app.route('/api/invoices/<int:id>')
@login_required
@authorize_resource(Invoice, id_param='id', owner_field='owner_id')
def get_invoice(id, resource):
    return jsonify(resource.to_dict())
```

### 8.4 Use UUIDs (Defense-in-Depth, Not Sole Protection)

```sql
-- Use UUID primary keys instead of auto-increment
CREATE TABLE documents (
    id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    owner_id    UUID NOT NULL REFERENCES users(id),
    title       VARCHAR(255),
    content     TEXT,
    created_at  TIMESTAMP DEFAULT NOW()
);
```

> ⚠️ **UUIDs alone are NOT sufficient.** They reduce enumeration risk but do not replace proper authorization checks.

### 8.5 Prevention Summary

| Control | Effectiveness | Complexity | Notes |
|---|---|---|---|
| **Server-side ownership check** | ★★★★★ | Low | Essential — always implement |
| **Indirect references (per-session)** | ★★★★☆ | Medium | Strong but adds mapping overhead |
| **UUID primary keys** | ★★★☆☆ | Low | Defense-in-depth only |
| **Rate limiting** | ★★☆☆☆ | Low | Slows enumeration, doesn't prevent |
| **Centralized auth middleware** | ★★★★★ | Medium | Best for consistency across endpoints |
| **Automated auth testing in CI/CD** | ★★★★☆ | High | Catches regressions early |

---

## 9. IDOR Testing Checklist

```
+──────────────────────────────────────────────────────────+
│               IDOR Testing Checklist                     │
+──────────────────────────────────────────────────────────+
│                                                          │
│  [ ] Identify all parameters referencing objects         │
│  [ ] Create 2+ test accounts at each privilege level     │
│  [ ] Test horizontal access (swap user IDs)              │
│  [ ] Test vertical access (use low-priv token on         │
│      high-priv endpoints)                                │
│  [ ] Test all HTTP methods (GET, POST, PUT, DELETE,      │
│      PATCH, OPTIONS)                                     │
│  [ ] Test encoded/hashed IDs (decode, re-encode)         │
│  [ ] Test parameter pollution                            │
│  [ ] Test UUID predictability (v1 vs v4)                 │
│  [ ] Check for IDOR in multi-step processes              │
│  [ ] Test file download/upload references                │
│  [ ] Test GraphQL queries with foreign IDs               │
│  [ ] Run Autorize for automated auth testing             │
│  [ ] Verify rate limiting on enumeration                 │
│  [ ] Document all findings with request/response pairs   │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## Summary Table

| Topic | Key Point |
|---|---|
| **Definition** | Application exposes internal object reference without authorization |
| **OWASP Mapping** | A01:2021 Broken Access Control / CWE-639 |
| **Horizontal IDOR** | Access peer user's data at the same privilege level |
| **Vertical IDOR** | Access data at a higher privilege level |
| **BOLA** | API-specific term for IDOR (OWASP API #1) |
| **Common Locations** | URL params, request body, path segments, hidden fields, cookies |
| **Key Tools** | Burp Intruder, Autorize, ffuf, custom scripts |
| **Primary Fix** | Server-side ownership verification on every request |
| **Defense-in-Depth** | UUIDs + indirect references + rate limiting + centralized middleware |
| **Testing Minimum** | Two accounts, swap IDs, test all HTTP methods |

---

## Revision Questions

1. **What is the fundamental difference between a horizontal IDOR and a vertical IDOR? Provide a concrete example of each.**

2. **Explain how the Burp Suite Autorize extension automates IDOR detection. What configuration is required, and how do you interpret its results?**

3. **A web application uses Base64-encoded user IDs (e.g., `ref=dXNlcl8xMDQy`). Describe the step-by-step process to test this endpoint for IDOR vulnerabilities.**

4. **Why are UUIDs alone insufficient protection against IDOR? Under what circumstances can UUIDs still be exploited?**

5. **Design a centralized authorization middleware that prevents IDOR across all endpoints in a REST API. What inputs does it need, and what checks does it perform?**

6. **A multi-step checkout process validates `cart_id` at step 1 but not `address_id` at step 2. Explain how an attacker could exploit this and what the impact would be.**

---

*Next: [02-horizontal-privilege-escalation.md](02-horizontal-privilege-escalation.md)*

---

*[Back to README](../README.md)*
