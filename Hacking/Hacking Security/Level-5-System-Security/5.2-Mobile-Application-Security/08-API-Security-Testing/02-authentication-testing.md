# Unit 8: API Security Testing — Topic 2: Authentication Testing

## Overview

**Authentication testing** for mobile APIs focuses on verifying that the backend properly validates user identity across all endpoints. Mobile apps commonly use OAuth 2.0, JWT tokens, API keys, or custom authentication — each with unique attack vectors including token theft, credential stuffing, brute force, and authentication bypass.

---

## 1. Mobile Authentication Mechanisms

```
AUTHENTICATION TYPES:

1. USERNAME/PASSWORD:
   POST /api/auth/login
   Body: { "username": "user", "password": "pass" }
   Response: { "access_token": "...", "refresh_token": "..." }
   
   TESTS:
   □ Credentials in request body (not URL)
   □ Rate limiting on login
   □ Account lockout after failures
   □ Error messages (don't reveal valid users)

2. OAuth 2.0:
   → Authorization Code flow (preferred)
   → PKCE extension for mobile (required!)
   → Implicit flow (deprecated, insecure)
   
   TESTS:
   □ PKCE code_verifier validation
   □ State parameter (CSRF protection)
   □ Redirect URI validation
   □ Token scope enforcement

3. JWT (JSON Web Tokens):
   Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
   
   TESTS:
   □ Algorithm confusion (none, HS256→RS256)
   □ Token expiration validation
   □ Signature verification
   □ Sensitive data in payload

4. API KEYS:
   X-API-Key: sk_live_abc123...
   
   TESTS:
   □ Key transmitted over HTTPS only
   □ Key not embedded in client code
   □ Key rotation mechanism
   □ Key scope limitations

5. CERTIFICATE-BASED:
   → Mutual TLS (mTLS)
   → Client certificate authentication
   → Strongest but complex
```

---

## 2. Authentication Attack Vectors

```bash
# === BRUTE FORCE ===
# Test login endpoint for rate limiting
for i in $(seq 1 100); do
    curl -s -o /dev/null -w "%{http_code}" \
        -X POST https://api.example.com/auth/login \
        -H "Content-Type: application/json" \
        -d '{"username":"admin","password":"attempt'$i'"}'
    echo " - Attempt $i"
done
# Expected: 429 (Too Many Requests) after threshold
# FINDING if: No rate limiting, no lockout

# === CREDENTIAL STUFFING ===
# Test with known leaked credentials
# Tools: Burp Intruder, hydra, custom scripts

# === PASSWORD RESET BYPASS ===
# Test password reset flow:
# □ Predictable reset tokens
# □ No rate limiting on OTP
# □ OTP brute force (4-6 digit = feasible)
# □ Reset link valid after password change
# □ Token reuse after expiration

# === JWT TESTING ===
# Decode JWT (jwt.io or command line)
echo "eyJhbG..." | base64 -d | python3 -m json.tool

# Test algorithm 'none'
# Header: {"alg":"none","typ":"JWT"}
# Remove signature → send to API

# Test HS256 with known secret
# Try common secrets: "secret", "password", app name
# Tool: jwt_tool, hashcat

# === IDOR VIA AUTHENTICATION ===
# After login as User A:
GET /api/users/123    → User A's data
GET /api/users/124    → User B's data? IDOR!
```

---

## 3. OAuth 2.0 Mobile Testing

```
OAuth 2.0 WITH PKCE (Proof Key for Code Exchange):

FLOW:
  1. App generates code_verifier (random string)
  2. App creates code_challenge = SHA256(code_verifier)
  3. App sends code_challenge to auth server
  4. User authenticates in browser
  5. Auth server returns authorization_code
  6. App sends code + code_verifier to token endpoint
  7. Server verifies SHA256(code_verifier) == code_challenge
  8. Server returns access_token + refresh_token

TESTING:
  □ Is PKCE implemented? (Required for mobile!)
  □ Send token request without code_verifier → Should fail
  □ Send wrong code_verifier → Should fail
  □ Is code_challenge method S256? (plain is weak)
  □ Authorization code reuse → Should fail on second use
  □ Redirect URI exact match validation
  □ Custom scheme hijacking risk

WITHOUT PKCE (VULNERABLE):
  → Authorization code can be intercepted
  → Malicious app with same URL scheme gets code
  → Exchanges code for token → account takeover!
```

---

## 4. Session Management Testing

```
SESSION/TOKEN TESTING:

TOKEN LIFECYCLE:
  □ Token issued on login → check format and entropy
  □ Token expiration → test with expired token
  □ Token refresh → does refresh invalidate old token?
  □ Token revocation → does logout invalidate token?
  □ Concurrent sessions → is there a limit?

TESTING STEPS:
  1. Login → capture access_token
  2. Wait for expiration → replay token → Should fail
  3. Refresh token → replay old access_token → Should fail
  4. Logout → replay access_token → Should fail
  5. Login on 2nd device → check 1st session
  6. Change password → check all sessions invalidated

COMMON FINDINGS:
  → Tokens never expire (infinite session)
  → Logout doesn't invalidate server-side
  → Refresh tokens with no rotation
  → No concurrent session limits
  → Tokens valid after password change
  → Short access token + long refresh token (good pattern)
     but refresh token stored insecurely (bad!)
```

---

## Summary Table

| Test | Expected Behavior | Finding If Failed |
|------|-------------------|-------------------|
| Brute force | Rate limit / lockout | Credential compromise |
| Expired token | 401 Unauthorized | Unlimited session |
| Post-logout token | 401 Unauthorized | Session persistence |
| IDOR | 403 Forbidden | Data breach |
| Algorithm none | Rejected | Auth bypass |
| No PKCE | Required | Code interception |

---

## Revision Questions

1. What authentication mechanisms are common in mobile APIs?
2. How do you test for brute force protection on login endpoints?
3. What is PKCE and why is it required for mobile OAuth 2.0?
4. How do you test JWT tokens for common vulnerabilities?
5. What session management issues are common in mobile APIs?

---

*Previous: [01-mobile-api-analysis.md](01-mobile-api-analysis.md) | Next: [03-token-handling.md](03-token-handling.md)*

---

*[Back to README](../README.md)*
