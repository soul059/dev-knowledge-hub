# Unit 4: Android Vulnerabilities — Topic 3: Insecure Authentication

## Overview

**Insecure authentication** in mobile apps occurs when authentication mechanisms can be bypassed, credentials are improperly handled, or session management is weak. Mobile apps often implement authentication differently from web apps — using OAuth tokens, biometrics, and local PIN codes — each introducing unique vulnerability patterns.

---

## 1. Common Authentication Vulnerabilities

```
AUTHENTICATION ISSUES:

CLIENT-SIDE AUTHENTICATION:
  → Auth checks performed ONLY on client
  → Modify client code → bypass authentication
  → Server trusts client's auth decision
  → ALWAYS authenticate on server side!

WEAK LOCAL AUTHENTICATION:
  → 4-digit PIN (only 10,000 combinations)
  → Pattern lock (limited combinations)
  → No lockout after failed attempts
  → Biometric without fallback security

CREDENTIAL STORAGE:
  → Passwords stored in SharedPreferences
  → Tokens with no expiration
  → Remember-me token in plaintext
  → Credentials in SQLite database

SESSION MANAGEMENT:
  → Long-lived tokens (no expiration)
  → No token rotation after privilege change
  → Session not invalidated on logout
  → Token not bound to device/IP

BIOMETRIC BYPASS:
  → Local-only biometric check
  → No server-side verification
  → Frida can bypass BiometricPrompt
  → Biometric result not cryptographically bound
```

---

## 2. Testing Authentication

```bash
# === TEST CLIENT-SIDE AUTH BYPASS ===
# Use Frida to bypass login check
```

```javascript
// Frida: Bypass login check
Java.perform(function() {
    var LoginActivity = Java.use("com.example.app.LoginActivity");
    LoginActivity.validateCredentials.implementation = function(user, pass) {
        console.log("[*] Bypassing auth check");
        return true;  // Always return authenticated
    };
});

// Frida: Bypass biometric check
Java.perform(function() {
    var BiometricCallback = Java.use("com.example.app.BiometricCallback");
    BiometricCallback.onAuthenticationSucceeded.implementation = function(result) {
        console.log("[*] Biometric bypassed!");
        this.onAuthenticationSucceeded(result);
    };
    
    // Force call onAuthenticationSucceeded
    BiometricCallback.onAuthenticationFailed.implementation = function() {
        console.log("[*] Turning failure into success");
        this.onAuthenticationSucceeded(null);
    };
});
```

```
API AUTHENTICATION TESTING:
  □ Remove auth token from request → Server should reject
  □ Use expired token → Server should reject
  □ Use another user's token → Should not access their data
  □ Modify user_id in request → Test for IDOR
  □ Test without token → Server should require auth
  □ Replay old requests → Tokens should expire
  □ Brute force PIN/password → Lockout should trigger
```

---

## 3. Secure Authentication Practices

```
RECOMMENDATIONS:

SERVER-SIDE:
  → ALL authentication decisions on server
  → Use industry standards (OAuth 2.0, OpenID Connect)
  → Implement rate limiting and account lockout
  → Short-lived access tokens + refresh tokens
  → Token binding to device fingerprint

BIOMETRIC:
  → Use crypto-based biometric (CryptoObject)
  → Biometric unlocks a key in Android Keystore
  → Key performs crypto operation (sign/decrypt)
  → Server verifies the crypto result
  → NOT just a local true/false check!

TOKEN MANAGEMENT:
  → Store in Android Keystore (not SharedPreferences)
  → Short expiration (15-30 minutes for access tokens)
  → Rotate tokens on privilege change
  → Invalidate on logout (server-side)
  → Use refresh tokens with rotation

MFA:
  → Implement multi-factor authentication
  → TOTP (Google Authenticator)
  → Push notifications
  → Hardware security keys (FIDO2)
```

---

## Summary Table

| Vulnerability | Impact | Test Method |
|--------------|--------|-------------|
| Client-side auth | Complete bypass | Frida hook, code review |
| Weak PIN | Brute force | Automated testing |
| No token expiration | Permanent access | Token reuse after timeout |
| IDOR | Data access | Modify user_id in API |
| Biometric bypass | Auth bypass | Frida BiometricPrompt hook |
| No server validation | Auth bypass | Remove token from request |

---

## Revision Questions

1. Why is client-side-only authentication dangerous?
2. How can Frida bypass biometric authentication?
3. What makes crypto-based biometric more secure than boolean?
4. How should authentication tokens be stored on Android?
5. What token management best practices prevent session attacks?

---

*Previous: [02-insecure-communication.md](02-insecure-communication.md) | Next: [04-code-tampering.md](04-code-tampering.md)*

---

*[Back to README](../README.md)*
