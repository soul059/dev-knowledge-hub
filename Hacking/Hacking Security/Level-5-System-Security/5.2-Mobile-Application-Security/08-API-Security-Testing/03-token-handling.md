# Unit 8: API Security Testing — Topic 3: Token Handling

## Overview

**Token handling** is critical to mobile API security. Access tokens, refresh tokens, JWTs, session tokens, and API keys must be generated, transmitted, stored, validated, and revoked securely. Mishandling at any stage can lead to session hijacking, account takeover, and unauthorized data access.

---

## 1. Token Types in Mobile Apps

```
TOKEN ECOSYSTEM:

ACCESS TOKEN:
  → Short-lived (15-60 minutes)
  → Sent with every API request
  → Contains user identity/permissions
  → JWT or opaque string
  → If stolen: limited window of abuse

REFRESH TOKEN:
  → Long-lived (days to months)
  → Used to get new access tokens
  → Stored securely on device
  → If stolen: long-term account access!
  → Must be rotated on use

ID TOKEN (OpenID Connect):
  → Contains user identity claims
  → JWT format
  → For client consumption only
  → Should NOT be sent to resource server

API KEY:
  → Identifies the application (not user)
  → Long-lived, static
  → For rate limiting, billing
  → Not for user authentication!

DEVICE TOKEN:
  → Identifies specific device
  → Used for push notifications
  → For device binding
  → Privacy-sensitive

TOKEN FLOW:
  Login → access_token + refresh_token
    │
    ├── API calls with access_token (15-60 min)
    │
    ├── access_token expires
    │
    ├── Use refresh_token → new access_token + new refresh_token
    │   (old refresh_token invalidated = "rotation")
    │
    └── Logout → invalidate both tokens server-side
```

---

## 2. Token Storage Security

```
WHERE TOKENS ARE STORED:

iOS:
┌──────────────────┬───────────┬──────────────────┐
│ Location         │ Security  │ Recommendation   │
├──────────────────┼───────────┼──────────────────┤
│ Keychain (akpu)  │ ★★★★★    │ BEST for tokens  │
│ Keychain (aku)   │ ★★★★     │ Good             │
│ Keychain (ck)    │ ★★★      │ Acceptable       │
│ NSUserDefaults   │ ★        │ NEVER for tokens │
│ Files            │ ★★       │ Only if encrypted│
│ CoreData         │ ★★       │ Only if encrypted│
└──────────────────┴───────────┴──────────────────┘

ANDROID:
┌──────────────────┬───────────┬──────────────────┐
│ Location         │ Security  │ Recommendation   │
├──────────────────┼───────────┼──────────────────┤
│ Android Keystore │ ★★★★★    │ BEST for tokens  │
│ EncryptedPrefs   │ ★★★★     │ Good             │
│ SharedPrefs      │ ★        │ NEVER for tokens │
│ SQLite           │ ★★       │ Only if encrypted│
│ Files            │ ★★       │ Only if encrypted│
└──────────────────┴───────────┴──────────────────┘

TESTING TOKEN STORAGE:
# iOS
objection -g com.app explore
> ios keychain dump          # Find tokens
> ios nsuserdefaults get     # Check for tokens here (bad!)

# Android
adb shell cat /data/data/com.app/shared_prefs/*.xml
adb shell sqlite3 /data/data/com.app/databases/*.db "SELECT * FROM tokens;"
```

---

## 3. Token Validation Testing

```
SERVER-SIDE TOKEN VALIDATION TESTS:

1. MISSING TOKEN:
   → Send request without Authorization header
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = broken auth!

2. INVALID TOKEN:
   → Send random/garbage token
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = no validation!

3. EXPIRED TOKEN:
   → Wait for expiration, replay
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = no expiration check!

4. MODIFIED TOKEN (JWT):
   → Change payload claims (user_id, role)
   → Re-encode without valid signature
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = no signature verification!

5. ALGORITHM CONFUSION (JWT):
   → Change "alg" to "none"
   → Remove signature
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = algorithm confusion!

6. TOKEN FROM ANOTHER USER:
   → Login as User A, get token
   → Use User A's token to access User B's data
   → Expected: 403 Forbidden
   → FINDING: 200 OK with B's data = IDOR!

7. REVOKED TOKEN:
   → Logout (revoke token)
   → Replay the revoked token
   → Expected: 401 Unauthorized
   → FINDING: 200 OK = no revocation!
```

---

## 4. JWT-Specific Testing

```
JWT STRUCTURE:
  Header.Payload.Signature
  
  Header:  {"alg":"RS256","typ":"JWT"}
  Payload: {"sub":"123","role":"user","exp":1700000000}
  Signature: RSASHA256(base64(header) + "." + base64(payload), key)

JWT ATTACKS:

ALGORITHM NONE:
  Header: {"alg":"none"}
  → Remove signature
  → If server accepts: CRITICAL

HS256/RS256 CONFUSION:
  → Server uses RS256 (asymmetric)
  → Attacker changes to HS256
  → Uses PUBLIC KEY as HMAC secret
  → Forges valid token!

WEAK SECRET (HS256):
  → Brute force the HMAC secret
  hashcat -a 0 -m 16500 jwt.txt wordlist.txt
  → If found: forge any token!

CLAIM MANIPULATION:
  → Modify: "role": "user" → "role": "admin"
  → Modify: "user_id": 1 → "user_id": 2
  → Modify: "exp" to far future
  → Re-sign if key is known

JKU/X5U INJECTION:
  → JWT header points to key URL
  → Attacker hosts own key at different URL
  → Change jku/x5u to attacker's URL
  → Server fetches attacker's key → accepts forged token

TOOLS:
  → jwt_tool: Comprehensive JWT testing
  → jwt.io: Online decoder
  → Burp JWT Editor extension
  → hashcat: Secret brute force
```

---

## 5. Token Best Practices

```
SECURE TOKEN HANDLING:

GENERATION:
  → Use cryptographically secure random generator
  → Sufficient entropy (128+ bits)
  → Include expiration claim
  → Bind to device/IP if possible

TRANSMISSION:
  → HTTPS only (never HTTP)
  → Authorization header (not URL parameter)
  → Short-lived access tokens
  → Do not log tokens

STORAGE:
  → iOS: Keychain with kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly
  → Android: Android Keystore or EncryptedSharedPreferences
  → Clear on logout
  → Clear on app uninstall (where possible)

VALIDATION:
  → Verify signature on every request
  → Check expiration claim
  → Validate issuer and audience
  → Check token type (access vs refresh)
  → Use algorithm allowlist (not blocklist)

REVOCATION:
  → Server-side token blocklist
  → Invalidate on logout
  → Invalidate on password change
  → Invalidate all sessions option
  → Refresh token rotation (one-use)
```

---

## Summary Table

| Attack | Target | Impact | Detection |
|--------|--------|--------|-----------|
| Token theft | Storage | Account takeover | Check storage location |
| No expiration | Validation | Unlimited access | Replay after timeout |
| No revocation | Lifecycle | Persistent access | Replay after logout |
| JWT algorithm none | Signature | Auth bypass | Modify header |
| Weak HMAC secret | JWT signature | Token forgery | hashcat brute force |
| Claim manipulation | JWT payload | Privilege escalation | Modify and replay |

---

## Revision Questions

1. What are the different token types and their purposes?
2. Where should tokens be stored on iOS and Android?
3. How do you test server-side token validation?
4. What are the main JWT attack vectors?
5. What is refresh token rotation and why is it important?

---

*Previous: [02-authentication-testing.md](02-authentication-testing.md) | Next: [04-certificate-pinning.md](04-certificate-pinning.md)*

---

*[Back to README](../README.md)*
