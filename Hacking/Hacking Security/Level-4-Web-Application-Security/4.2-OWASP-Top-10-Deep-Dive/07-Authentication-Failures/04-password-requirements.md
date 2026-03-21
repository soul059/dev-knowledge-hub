# Unit 7: A07 - Identification and Authentication Failures — Topic 4: Password Requirements

## Overview

Password requirements define the **minimum standards for password creation** to resist brute force and dictionary attacks. Modern best practices have evolved significantly — NIST SP 800-63B (2017) fundamentally changed password guidance by deprecating complexity rules and rotation requirements in favor of length, breach checking, and usability.

---

## 1. Modern Password Guidelines (NIST SP 800-63B)

```
┌─────────────────────────────────────────────────────────────────┐
│     OLD vs NEW PASSWORD GUIDELINES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ OLD (Deprecated):                                          │
│  ├── Minimum 8 characters with complexity rules                │
│  ├── Must contain uppercase + lowercase + number + symbol      │
│  ├── Mandatory rotation every 60-90 days                       │
│  ├── Password hints/security questions                         │
│  └── Result: "P@ssw0rd!" — meets rules, easily cracked        │
│                                                                 │
│  ✅ NEW (NIST SP 800-63B):                                     │
│  ├── Minimum 8 characters, recommended 15+                    │
│  ├── Maximum at least 64 characters                            │
│  ├── Allow ALL characters (Unicode, spaces, emoji)             │
│  ├── NO complexity rules (uppercase/number/symbol)             │
│  ├── NO mandatory rotation (only on breach)                    │
│  ├── CHECK against breached password databases                 │
│  ├── CHECK against common/context-specific passwords           │
│  ├── Allow paste in password fields                            │
│  ├── Show password option (for usability)                      │
│  └── Use password strength meter                               │
│                                                                 │
│  WHY THE CHANGE:                                               │
│  Complexity rules → Users create predictable patterns          │
│  Forced rotation → Users increment: Password1, Password2      │
│  Length > Complexity for entropy                                │
│  "correct horse battery staple" >> "P@ssw0rd!"                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Password Entropy

```
Entropy (bits) = log2(characters^length)

Examples:
┌────────────────────────────┬──────┬─────────┬──────────────────┐
│ Password Type              │ Char │ Entropy │ Crack Time (GPU) │
├────────────────────────────┼──────┼─────────┼──────────────────┤
│ 6 digits (PIN)             │ 10^6 │ 20 bits │ < 1 second       │
│ 8 lowercase letters        │ 26^8 │ 38 bits │ Minutes           │
│ 8 mixed (upper+lower+num)  │ 62^8 │ 48 bits │ Hours             │
│ 8 all printable ASCII      │ 95^8 │ 53 bits │ Days              │
│ 12 mixed case + symbols    │ 95^12│ 79 bits │ Centuries          │
│ 4 random words (diceware)  │ ~58  │ ~58 bits│ Decades            │
│ 5 random words (diceware)  │ ~73  │ ~73 bits│ Millennia          │
│ 16 random characters       │ 95^16│ 105 bits│ Heat death of sun │
└────────────────────────────┴──────┴─────────┴──────────────────┘

KEY INSIGHT: Length matters more than complexity
"correcthorsebatterystaple" (25 chars) > "P@$$w0rd!" (9 chars)
```

---

## 3. Password Blocklist

```python
# Passwords that MUST be rejected regardless of length/complexity:

BLOCKED_PASSWORDS = {
    # Top common passwords
    'password', '123456', '12345678', 'qwerty', 'abc123',
    'password1', 'iloveyou', 'admin', 'welcome', 'monkey',
    
    # Context-specific (your company/app)
    'companyname', 'appname', 'companyname123',
    
    # Keyboard patterns
    'qwerty', 'asdfgh', 'zxcvbn', '1qaz2wsx',
    
    # Repetitive patterns
    'aaaaaa', '111111', 'abcabc',
    
    # Dictionary words (single words)
    # Use a comprehensive wordlist
}

def validate_password(password, username, email):
    errors = []
    
    # Minimum length
    if len(password) < 8:
        errors.append("Password must be at least 8 characters")
    
    # Maximum length (prevent DoS with very long passwords)
    if len(password) > 128:
        errors.append("Password must be at most 128 characters")
    
    # Blocklist check
    if password.lower() in BLOCKED_PASSWORDS:
        errors.append("This password is too common")
    
    # Context check (don't use your username/email as password)
    if username.lower() in password.lower():
        errors.append("Password cannot contain your username")
    if email.split('@')[0].lower() in password.lower():
        errors.append("Password cannot contain your email")
    
    # Breached password check (Have I Been Pwned)
    is_breached, count = check_hibp(password)
    if is_breached:
        errors.append(f"This password was found in {count} data breaches")
    
    return len(errors) == 0, errors
```

---

## 4. Password Strength Meter

```javascript
// Client-side password strength estimation using zxcvbn
// Library: https://github.com/dropbox/zxcvbn

const zxcvbn = require('zxcvbn');

function checkPasswordStrength(password, userInputs = []) {
    // userInputs: context-specific words to penalize
    // e.g., username, email, site name
    const result = zxcvbn(password, userInputs);
    
    return {
        score: result.score,        // 0-4 (0=worst, 4=best)
        crackTime: result.crack_times_display.offline_slow_hashing_1e4_per_second,
        feedback: result.feedback,  // Suggestions for improvement
        warning: result.feedback.warning
    };
}

// Score interpretation:
// 0: Too guessable (risky password)
// 1: Very guessable (protection from throttled attacks)
// 2: Somewhat guessable (provides some protection)
// 3: Safely unguessable (moderate protection)
// 4: Very unguessable (strong protection)

// Example:
// "password"     → score: 0, crack: "instant"
// "P@ssw0rd!"    → score: 1, crack: "10 minutes"
// "MyD0g$Name"   → score: 2, crack: "1 day"
// "correct horse" → score: 3, crack: "centuries"
// Random 16-char → score: 4, crack: "centuries+"
```

---

## 5. Secure Password Storage

```python
# OWASP recommendation: Argon2id
from argon2 import PasswordHasher, Type

ph = PasswordHasher(
    time_cost=3,           # Number of iterations
    memory_cost=65536,     # 64 MB memory required
    parallelism=4,         # Number of threads
    hash_len=32,           # Output hash length
    salt_len=16,           # Salt length
    type=Type.ID           # Argon2id (hybrid)
)

def hash_password(password):
    """Hash password for storage"""
    return ph.hash(password)
    # Output: $argon2id$v=19$m=65536,t=3,p=4$salt$hash

def verify_password(stored_hash, password):
    """Verify password against stored hash"""
    try:
        return ph.verify(stored_hash, password)
    except Exception:
        return False

def needs_rehash(stored_hash):
    """Check if hash needs upgrading (params changed)"""
    return ph.check_needs_rehash(stored_hash)

# If Argon2 not available, use bcrypt:
import bcrypt

def hash_password_bcrypt(password):
    return bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt(rounds=12))
```

---

## 6. Password Policy Implementation

```
RECOMMENDED PASSWORD POLICY:

┌──────────────────────────┬──────────────────────────────────────┐
│ Requirement              │ Value                                │
├──────────────────────────┼──────────────────────────────────────┤
│ Minimum length           │ 8 characters (12 recommended)       │
│ Maximum length           │ 128 characters                       │
│ Character types          │ All Unicode allowed                  │
│ Complexity rules         │ None (NIST recommendation)           │
│ Breach check             │ Required (HIBP API)                  │
│ Common password block    │ Top 100,000 + context-specific       │
│ Strength meter           │ Required (zxcvbn or similar)         │
│ Mandatory rotation       │ No (only on breach evidence)         │
│ Password history         │ Last 5 passwords                     │
│ Hashing algorithm        │ Argon2id (bcrypt acceptable)         │
│ Paste in password field  │ Allowed (for password managers)      │
│ Show/hide password       │ Available (accessibility)            │
│ Password manager support │ Autocomplete="new-password"         │
└──────────────────────────┴──────────────────────────────────────┘
```

---

## Summary Table

| Practice | Old Approach | New Approach (NIST) |
|----------|-------------|---------------------|
| Length | Minimum 8 | Minimum 8, recommend 15+ |
| Complexity | Required (upper+lower+num+symbol) | Not required |
| Rotation | Every 60-90 days | Only on breach |
| Breach check | Not done | Required |
| Blocklist | Not used | Common + context passwords |
| Storage | MD5/SHA | Argon2id/bcrypt |
| Paste | Often disabled | Must be allowed |

---

## Revision Questions

1. How do NIST SP 800-63B guidelines differ from traditional password requirements?
2. Why is "correct horse battery staple" more secure than "P@ssw0rd!" despite being simpler?
3. Calculate the entropy for different password types and estimate crack times.
4. Implement a password validation function that checks length, blocklist, context, and HIBP.
5. Why should mandatory password rotation be eliminated? What should trigger a password change?
6. Compare Argon2id, bcrypt, and SHA-256 for password hashing with cracking speed estimates.

---

*Previous: [03-session-management.md](03-session-management.md) | Next: [05-mfa-implementation.md](05-mfa-implementation.md)*

---

*[Back to README](../README.md)*
