# Session Management

## Unit 4: Authentication Testing — Topic 5

## 🎯 Overview

Session management controls how user state is maintained after authentication. Weak session management is consistently in the OWASP Top 10 and can lead to account takeover, privilege escalation, and unauthorized access. This topic covers session token analysis, lifecycle testing, and exploitation techniques.

---

## 1. Session Token Analysis

```
┌──────────────────────────────────────────────────────────────┐
│              SESSION TOKEN QUALITY CHECKS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Randomness:    Is the token cryptographically random?       │
│  Length:        Is it long enough? (128+ bits recommended)   │
│  Entropy:       Is the character set diverse?                │
│  Predictability: Can next token be guessed from previous?    │
│  Format:        Is it a JWT, opaque token, or encoded value? │
│                                                              │
│  Good:   9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c (32 hex chars)  │
│  Bad:    user123_1705312200 (username + timestamp)           │
│  Bad:    000001, 000002, 000003 (sequential)                 │
│  Bad:    YWRtaW4= (Base64 of "admin")                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Collect multiple session tokens
for i in $(seq 1 50); do
  curl -s -c - https://target.com/login \
    -X POST -d "user=test&pass=test123" 2>/dev/null | \
    grep -oP 'session=\K[^\s;]+'
done > tokens.txt

# Analyze with Burp Sequencer
# Proxy → HTTP History → find Set-Cookie response
# Right-click → Send to Sequencer → Start live capture
# Analyze token randomness

# Manual analysis
# Check for patterns, sequential parts, timestamps
sort tokens.txt | uniq -c | sort -rn | head
# If duplicates found → weak randomness!
```

---

## 2. Session Lifecycle Testing

```
┌──────────────────────────────────────────────────────────────┐
│              SESSION LIFECYCLE                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Creation ──→ Active ──→ Idle ──→ Expired ──→ Destroyed     │
│     │           │         │          │            │          │
│  Login      Requests   No activity  Timeout    Logout       │
│  New token  Token sent  Start timer  Invalidate  Server-side│
│  Set flags  each req   (15-30 min)  on server   delete      │
│                                                              │
│  Test each transition:                                       │
│  • Is token regenerated after login?                         │
│  • Is idle timeout enforced?                                 │
│  • Is absolute timeout enforced?                             │
│  • Is session destroyed on logout?                           │
│  • Can expired session be reused?                            │
└──────────────────────────────────────────────────────────────┘
```

### Testing Session Regeneration

```bash
# Get session before login
PRE_SESSION=$(curl -s -c - https://target.com/login | \
  grep session | awk '{print $NF}')

# Login
POST_SESSION=$(curl -s -c - -X POST https://target.com/login \
  -d "user=test&pass=test123" | \
  grep session | awk '{print $NF}')

echo "Before login: $PRE_SESSION"
echo "After login:  $POST_SESSION"
# If same → session fixation vulnerability!
```

### Testing Logout Invalidation

```bash
# Get authenticated session
SESSION=$(curl -s -c - -X POST https://target.com/login \
  -d "user=test&pass=test123" -D - | \
  grep -oP 'session=\K[^\s;]+')

# Verify it works
curl -b "session=$SESSION" https://target.com/dashboard
# Should return dashboard

# Logout
curl -b "session=$SESSION" https://target.com/logout

# Try using old session
curl -b "session=$SESSION" https://target.com/dashboard
# If dashboard still accessible → session not invalidated!
```

---

## 3. Session Fixation

```
┌──────────────────────────────────────────────────────────────┐
│              SESSION FIXATION ATTACK                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker                Victim                Server        │
│     │                      │                     │           │
│  1. Get session ID         │                     │           │
│     │──GET /──────────────────────────────────→  │           │
│     │←─Set-Cookie: session=ATTACKER_SID────────  │           │
│     │                      │                     │           │
│  2. Force victim to use attacker's session       │           │
│     │──Link: target.com/                         │           │
│     │  ?session=ATTACKER_SID                     │           │
│     │──────────────────→  │                      │           │
│     │                   3. Victim logs in        │           │
│     │                      │──POST /login───→    │           │
│     │                      │  Cookie: session=   │           │
│     │                      │  ATTACKER_SID       │           │
│     │                      │                     │           │
│  4. Attacker uses same session                   │           │
│     │──GET /dashboard──────────────────────────→ │           │
│     │  Cookie: session=ATTACKER_SID              │           │
│     │←─Victim's dashboard!────────────────────── │           │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test session fixation
# 1. Get a session
OLD_SID=$(curl -s -c - https://target.com | grep session | awk '{print $NF}')

# 2. Login with that session
curl -b "session=$OLD_SID" -X POST https://target.com/login \
  -d "user=test&pass=test123" -v 2>&1 | grep Set-Cookie

# 3. Check if session ID changed after login
# If same → vulnerable to session fixation

# Also check:
# Can session be set via URL? (/page?PHPSESSID=xxx)
# Can session be set via POST parameter?
# Can session be set via meta tag or JavaScript?
```

---

## 4. Concurrent Session Testing

```bash
# Test: Can user have multiple active sessions?
# Login from "device 1"
SESSION1=$(curl -s -X POST https://target.com/login \
  -d "user=test&pass=test123" -c - | grep session | awk '{print $NF}')

# Login from "device 2"
SESSION2=$(curl -s -X POST https://target.com/login \
  -d "user=test&pass=test123" -c - | grep session | awk '{print $NF}')

# Test both sessions
curl -b "session=$SESSION1" https://target.com/dashboard
curl -b "session=$SESSION2" https://target.com/dashboard

# Security consideration:
# If both work → no concurrent session limit
# For high-security apps, only one session should be valid
# Banking apps should invalidate old sessions on new login
```

---

## 5. Session Cookie Security Flags

```bash
# Check all cookie attributes
curl -sI https://target.com/login -X POST \
  -d "user=test&pass=test123" | grep -i set-cookie

# Required flags:
# Secure     → only sent over HTTPS
# HttpOnly   → not accessible via JavaScript
# SameSite   → controls cross-site sending
# Path=/     → appropriate scope
# Domain     → appropriate scope

# Test missing Secure flag
curl http://target.com/dashboard -b "session=$SESSION"
# If cookie sent over HTTP → Secure flag missing

# Test missing HttpOnly
# In browser console: document.cookie
# If session visible → HttpOnly missing

# Test SameSite
# Create page on evil.com with form posting to target.com
# If request includes session cookie → SameSite not enforced
```

---

## 6. Session Token in URL

```bash
# Check if session token appears in URL
# /page?session=abc123&user=admin
# /page;jsessionid=abc123

# Why it's dangerous:
# • Visible in browser history
# • Visible in Referer header (leaked to external sites)
# • Visible in server logs
# • Can be shared accidentally
# • Bookmarkable with session

# Test:
# 1. Login and check URL for session parameters
# 2. Click external link, check Referer header on other site
# 3. Check if session in URL works without cookie
```

---

## 📊 Summary Table

| Vulnerability | Test Method | Impact |
|--------------|------------|--------|
| Weak randomness | Collect & analyze tokens | Session prediction |
| No regeneration | Compare pre/post login | Session fixation |
| No expiration | Wait & reuse token | Persistent access |
| No logout invalidation | Use token after logout | Persistent access |
| Missing Secure flag | Access via HTTP | Session hijacking |
| Missing HttpOnly | document.cookie | XSS cookie theft |
| Session in URL | Check URL & Referer | Session leakage |
| No concurrent limits | Multiple logins | Undetected sharing |

---

## ❓ Revision Questions

1. How do you test session token randomness and predictability?
2. What is session fixation and how is it prevented?
3. Why must sessions be regenerated after authentication?
4. How do you verify that sessions are properly invalidated on logout?
5. What is the risk of session tokens appearing in URLs?
6. Why should concurrent sessions be limited for high-security applications?

---

*Previous: [04-mfa-testing.md](04-mfa-testing.md) | Next: [06-oauth-sso-testing.md](06-oauth-sso-testing.md)*

---

*[Back to README](../README.md)*
