# Password Policy

## Unit 6 - Topic 4 | Security Policies

---

## Overview

A **password policy** defines the requirements for creating, managing, and protecting passwords across an organization. With passwords remaining the primary authentication method, a strong password policy is critical for preventing unauthorized access.

---

## 1. Modern Password Requirements

### NIST SP 800-63B Guidelines (Current Best Practice)

| Recommendation | Old Approach ❌ | Modern Approach ✅ |
|---------------|----------------|-------------------|
| **Length** | 8 characters minimum | **12-16+ characters minimum** |
| **Complexity** | Must have upper+lower+number+special | **Length > complexity** |
| **Rotation** | Change every 30-90 days | **Only change if compromised** |
| **History** | Remember last 10 passwords | **Check against breached passwords** |
| **Lockout** | Lock after 3 attempts | **Progressive delays + CAPTCHA** |
| **MFA** | Optional | **Required wherever possible** |

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD STRENGTH COMPARISON                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  WEAK:       P@ssw0rd         (8 chars, predictable) ❌      │
│  MEDIUM:     MyD0g$Name!2     (12 chars, some complexity) ⚠️ │
│  STRONG:     correct-horse-battery-staple (28 chars) ✅      │
│  STRONGEST:  Password manager generated (random 20+) ✅✅    │
│                                                              │
│  KEY INSIGHT: Length beats complexity every time.             │
│  A 20-char passphrase > an 8-char complex password.          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Password Policy Components

| Component | Requirement |
|-----------|------------|
| **Minimum Length** | 12 characters (16+ for privileged accounts) |
| **Maximum Length** | At least 64 characters allowed |
| **Passphrase Support** | Allow spaces and long phrases |
| **Breached Password Check** | Check against known breached password databases |
| **No Forced Rotation** | Only change if evidence of compromise |
| **MFA Required** | Enforce MFA for all accounts (especially admin) |
| **Account Lockout** | Progressive delays after failed attempts |
| **Password Managers** | Encouraged and supported |
| **Secure Storage** | Passwords stored as salted hashes (bcrypt/Argon2) |
| **No Password Hints** | Remove password hints (information leakage) |

---

## 3. Password Attack Reference

| Attack | Description | What Defeats It |
|--------|-------------|-----------------|
| **Brute Force** | Try every combination | Long passwords, lockout |
| **Dictionary** | Try common words | Avoid dictionary words |
| **Rainbow Table** | Pre-computed hash lookup | Salted hashes |
| **Credential Stuffing** | Use breached credentials | Unique passwords, MFA |
| **Password Spraying** | Few passwords, many accounts | Monitoring, lockout |
| **Keylogging** | Record keystrokes | MFA, anti-malware |
| **Phishing** | Trick user into revealing | MFA, training |
| **Shoulder Surfing** | Watch someone type | Privacy screens |

---

## 4. Privileged Account Password Requirements

| Requirement | Standard Accounts | Privileged Accounts |
|-------------|------------------|-------------------|
| **Minimum Length** | 12 characters | 16+ characters |
| **MFA** | Required | Required (hardware key preferred) |
| **Password Manager** | Recommended | Required |
| **Monitoring** | Basic | Enhanced (PAM solution) |
| **Session Recording** | No | Yes |
| **Just-In-Time Access** | No | Yes |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Modern Standard** | NIST SP 800-63B |
| **Length vs Complexity** | Length wins — 12+ chars minimum |
| **Forced Rotation** | Only change if compromised |
| **MFA** | Required, not optional |
| **Breached Passwords** | Check against known breach databases |
| **Password Managers** | Encourage and support their use |

---

## Quick Revision Questions

1. **Why does NIST no longer recommend forced password rotation?**
2. **Why is a 28-character passphrase stronger than an 8-character complex password?**
3. **What is credential stuffing and how does MFA defeat it?**
4. **What additional requirements should privileged accounts have?**
5. **How should passwords be stored in a database?**

---

[← Previous: Incident Response Policy](03-incident-response-policy.md) | [Next: Data Classification →](05-data-classification.md)

---

*Unit 6 - Topic 4 of 6 | [Back to README](../README.md)*
