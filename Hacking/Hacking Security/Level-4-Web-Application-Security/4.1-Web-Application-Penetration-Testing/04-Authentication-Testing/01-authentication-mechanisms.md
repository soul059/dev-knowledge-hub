# Authentication Mechanisms

## Unit 4: Authentication Testing — Topic 1

## 🎯 Overview

Authentication is the process of verifying a user's identity — "who are you?" It is one of the most critical security controls in any web application. Weaknesses in authentication lead to unauthorized access, account takeover, and data breaches. This topic covers the major authentication mechanisms and their security implications.

---

## 1. Authentication Types

```
┌──────────────────────────────────────────────────────────────┐
│              AUTHENTICATION MECHANISMS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Something You Know     │  Password, PIN, security question  │
│  Something You Have     │  Token, phone, smart card, key     │
│  Something You Are      │  Fingerprint, face, iris, voice    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Single Factor        →  Password only               │    │
│  │  Two Factor (2FA)     →  Password + OTP              │    │
│  │  Multi Factor (MFA)   →  Password + OTP + Biometric  │    │
│  └─────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Form-Based Authentication

```
┌──────────┐     POST /login          ┌──────────┐
│  Browser │ ───────────────────────→ │  Server  │
│          │  username=admin          │          │
│          │  password=P@ssw0rd       │ Verify   │
│          │                          │ against  │
│          │ ←───────────────────────  │ database │
│          │  Set-Cookie: session=abc │          │
└──────────┘                          └──────────┘

Security Concerns:
• Credentials in POST body (never GET — visible in logs)
• HTTPS required (plaintext credentials otherwise)
• Rate limiting needed (brute force protection)
• Account lockout policy
• Error messages (don't reveal which field is wrong)
```

### Testing Form Authentication

```bash
# Test for username enumeration
curl -X POST https://target.com/login \
  -d "username=admin&password=wrong"
# "Invalid password" → username exists!
# "Invalid username or password" → safe error message

# Test for timing-based enumeration
# Valid user: 250ms (hashes password)
# Invalid user: 50ms (returns immediately)
time curl -X POST https://target.com/login \
  -d "username=validuser&password=wrong"
time curl -X POST https://target.com/login \
  -d "username=invaliduser&password=wrong"
```

---

## 3. Token-Based Authentication (JWT)

```
┌──────────────────────────────────────────────────────────────┐
│                    JWT AUTHENTICATION                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Login → Server returns JWT                               │
│  2. Client stores JWT (localStorage or cookie)               │
│  3. Each request: Authorization: Bearer <JWT>                │
│                                                              │
│  JWT Structure:                                              │
│  eyJhbGciOiJIUzI1NiJ9.                  ← Header (Base64)   │
│  eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ.  ← Payload  │
│  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV     ← Signature        │
│                                                              │
│  Decoded Payload:                                            │
│  {"user": "admin", "role": "admin", "exp": 1705398600}      │
└──────────────────────────────────────────────────────────────┘
```

### JWT Attack Vectors

```bash
# Decode JWT
echo "eyJ1c2VyIjoiYWRtaW4ifQ" | base64 -d

# Attack 1: Algorithm None
# Change header to {"alg":"none"} → no signature required
python3 jwt_tool.py <token> -X a

# Attack 2: Algorithm confusion (RS256 → HS256)
# Use public key as HMAC secret
python3 jwt_tool.py <token> -X k -pk public_key.pem

# Attack 3: Weak secret brute-force
hashcat -m 16500 jwt.txt wordlist.txt
john jwt.txt --wordlist=wordlist.txt --format=HMAC-SHA256

# Attack 4: Modify payload
python3 jwt_tool.py <token> -T  # Tamper mode
# Change "role": "user" → "role": "admin"
```

---

## 4. Basic and Digest Authentication

```
┌──────────────────────────────────────────────────────────────┐
│  HTTP Basic Authentication                                    │
│  ────────────────────────                                     │
│  Authorization: Basic YWRtaW46cGFzc3dvcmQ=                  │
│                       └── Base64(admin:password)              │
│                                                              │
│  ⚠ Credentials sent with EVERY request                      │
│  ⚠ Base64 is encoding, NOT encryption                       │
│  ⚠ Must use HTTPS to protect credentials                    │
├──────────────────────────────────────────────────────────────┤
│  HTTP Digest Authentication                                   │
│  ──────────────────────────                                   │
│  Uses challenge-response with nonce                           │
│  Better than Basic but still has weaknesses                  │
│  Vulnerable to MitM if not over HTTPS                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test Basic auth
curl -u admin:password https://target.com/api
curl -H "Authorization: Basic YWRtaW46cGFzc3dvcmQ=" https://target.com/api

# Brute force Basic auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-get /protected/ -f
```

---

## 5. API Key Authentication

```bash
# Common API key locations
# Header: X-API-Key: abc123
# Header: Authorization: ApiKey abc123
# Query:  /api/data?api_key=abc123
# Cookie: api_key=abc123

# Testing API keys:
# 1. Check scope — can the key access admin endpoints?
# 2. Check for key rotation — is it the same key always?
# 3. Check for key in client-side code (JS, mobile apps)
# 4. Test if key is required for all endpoints
# 5. Test if expired/revoked keys still work

curl -H "X-API-Key: found_in_js" https://target.com/api/admin/users
```

---

## 6. Password Reset Flaws

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD RESET ATTACK VECTORS                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Predictable Reset Tokens                                 │
│     Token = MD5(email + timestamp)                           │
│     → Generate and predict token                             │
│                                                              │
│  2. Token Not Expiring                                       │
│     Old reset links still valid                              │
│     → Use previously leaked tokens                           │
│                                                              │
│  3. Host Header Poisoning                                    │
│     POST /reset-password                                     │
│     Host: evil.com                                           │
│     → Reset link sent with evil.com domain                   │
│                                                              │
│  4. No Rate Limiting                                         │
│     Brute-force short reset codes (4-6 digits)              │
│                                                              │
│  5. Password Reset via IDOR                                  │
│     POST /reset?user_id=123 → change to user_id=1 (admin)  │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Mechanism | Strength | Common Weakness |
|-----------|----------|----------------|
| Form-based | Flexible | Brute force, enumeration |
| JWT | Stateless | Algorithm attacks, weak secrets |
| Basic Auth | Simple | Base64 exposure, no logout |
| API Key | Easy to implement | Static, often leaked in code |
| OAuth/OIDC | Delegated | Misconfiguration, redirect bypass |
| MFA | Strong | Bypass via race condition, phishing |

---

## ❓ Revision Questions

1. What are the three factors of authentication?
2. How can username enumeration be performed through error messages?
3. What is the JWT "Algorithm None" attack?
4. Why is HTTP Basic authentication insecure without HTTPS?
5. What are common password reset vulnerabilities?
6. How does timing-based username enumeration work?

---

*Next: [02-brute-force-attacks.md](02-brute-force-attacks.md)*

---

*[Back to README](../README.md)*
