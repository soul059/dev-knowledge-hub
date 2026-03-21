# Unit 7: A07 - Identification and Authentication Failures — Topic 5: MFA Implementation

## Overview

Multi-Factor Authentication (MFA) is the **single most effective defense** against account takeover attacks. It requires users to prove their identity using two or more independent factors from different categories. Even if passwords are compromised through phishing, breaches, or brute force, MFA prevents unauthorized access. This topic covers MFA types, implementation, attacks against MFA, and best practices.

---

## 1. Authentication Factors

```
┌─────────────────────────────────────────────────────────────────┐
│              THREE AUTHENTICATION FACTOR TYPES                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SOMETHING YOU KNOW (Knowledge):                               │
│  ├── Password / passphrase                                     │
│  ├── PIN                                                       │
│  ├── Security questions                                        │
│  └── Pattern lock                                              │
│                                                                 │
│  SOMETHING YOU HAVE (Possession):                              │
│  ├── TOTP app (Google Authenticator, Authy)                    │
│  ├── Hardware security key (YubiKey, Titan)                    │
│  ├── SMS code (weakest — SIM swap attacks)                     │
│  ├── Email code                                                │
│  ├── Push notification (Duo, Microsoft Authenticator)          │
│  └── Smart card / PKI certificate                              │
│                                                                 │
│  SOMETHING YOU ARE (Inherence / Biometric):                    │
│  ├── Fingerprint                                               │
│  ├── Face recognition                                          │
│  ├── Iris scan                                                 │
│  ├── Voice recognition                                         │
│  └── Behavioral biometrics (typing pattern, gait)              │
│                                                                 │
│  TRUE MFA = Factors from 2+ DIFFERENT categories               │
│  Password + PIN = NOT MFA (both knowledge)                     │
│  Password + TOTP = MFA ✅ (knowledge + possession)             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. MFA Methods Comparison

| Method | Security | Usability | Phishing Resistant | Cost |
|--------|----------|-----------|-------------------|------|
| **FIDO2/WebAuthn** | ★★★★★ | ★★★★★ | ✅ Yes | Medium |
| **Hardware Key** | ★★★★★ | ★★★★ | ✅ Yes | High |
| **TOTP App** | ★★★★ | ★★★★ | ❌ No (phishable) | Free |
| **Push Notification** | ★★★★ | ★★★★★ | ❌ No (fatigue) | Medium |
| **SMS Code** | ★★ | ★★★★★ | ❌ No | Low |
| **Email Code** | ★★ | ★★★ | ❌ No | Free |
| **Security Questions** | ★ | ★★★ | ❌ No | Free |

---

## 3. TOTP Implementation

```python
# Time-Based One-Time Password (RFC 6238)
import pyotp
import qrcode
import io

# Setup: Generate secret for user
def setup_mfa(user):
    """Generate TOTP secret and QR code"""
    # Generate random secret
    secret = pyotp.random_base32()
    
    # Store encrypted secret in database
    user.mfa_secret = encrypt(secret)
    user.mfa_enabled = False  # Not active until verified
    db.session.commit()
    
    # Generate provisioning URI for QR code
    totp = pyotp.TOTP(secret)
    uri = totp.provisioning_uri(
        name=user.email,
        issuer_name="MyApp"
    )
    # URI: otpauth://totp/MyApp:user@example.com?secret=BASE32SECRET&issuer=MyApp
    
    # Generate QR code
    qr = qrcode.QRCode(version=1, box_size=10, border=5)
    qr.add_data(uri)
    qr.make(fit=True)
    
    return {
        'secret': secret,         # Show to user as backup
        'qr_code': qr_to_base64(qr),
        'backup_codes': generate_backup_codes(user)
    }

# Verification: Validate TOTP code
def verify_mfa(user, code):
    """Verify TOTP code with time window"""
    secret = decrypt(user.mfa_secret)
    totp = pyotp.TOTP(secret)
    
    # valid_window=1 allows ±30 seconds tolerance
    is_valid = totp.verify(code, valid_window=1)
    
    if is_valid:
        # Prevent replay attacks — check if code was already used
        if is_code_used(user.id, code):
            return False
        mark_code_used(user.id, code)
    
    return is_valid

# Backup codes: One-time use recovery codes
def generate_backup_codes(user, count=10):
    """Generate one-time backup codes"""
    codes = [secrets.token_hex(4) for _ in range(count)]
    
    # Store hashed backup codes
    for code in codes:
        hashed = bcrypt.hashpw(code.encode(), bcrypt.gensalt())
        store_backup_code(user.id, hashed)
    
    return codes  # Show to user once, never again
```

---

## 4. WebAuthn / FIDO2 (Phishing-Resistant)

```
┌─────────────────────────────────────────────────────────────────┐
│              WebAuthn / FIDO2 FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REGISTRATION:                                                 │
│  1. Server sends challenge + relying party info                │
│  2. Browser calls navigator.credentials.create()              │
│  3. Authenticator creates key pair (private stays on device)   │
│  4. Public key + attestation sent to server                    │
│  5. Server stores public key for user                          │
│                                                                 │
│  AUTHENTICATION:                                               │
│  1. Server sends challenge                                     │
│  2. Browser calls navigator.credentials.get()                 │
│  3. Authenticator signs challenge with private key             │
│  4. Signature sent to server                                   │
│  5. Server verifies signature with stored public key           │
│                                                                 │
│  WHY IT'S PHISHING-RESISTANT:                                  │
│  → Credential is bound to the origin (domain)                 │
│  → Cannot be replayed on a different domain                   │
│  → Private key never leaves the authenticator device           │
│  → No shared secret to steal                                   │
└─────────────────────────────────────────────────────────────────┘
```

```javascript
// WebAuthn Registration (client-side)
async function registerWebAuthn() {
    const options = await fetch('/api/webauthn/register/begin', {
        method: 'POST'
    }).then(r => r.json());
    
    // Create credential
    const credential = await navigator.credentials.create({
        publicKey: {
            challenge: Uint8Array.from(options.challenge),
            rp: { name: "MyApp", id: "myapp.com" },
            user: {
                id: Uint8Array.from(options.userId),
                name: options.email,
                displayName: options.name
            },
            pubKeyCredParams: [
                { type: "public-key", alg: -7 },    // ES256
                { type: "public-key", alg: -257 }   // RS256
            ],
            authenticatorSelection: {
                authenticatorAttachment: "cross-platform", // or "platform"
                userVerification: "required",
                residentKey: "preferred"
            },
            timeout: 60000
        }
    });
    
    // Send to server for storage
    await fetch('/api/webauthn/register/complete', {
        method: 'POST',
        body: JSON.stringify(credential)
    });
}
```

---

## 5. Attacks Against MFA

```
ATTACK 1: MFA FATIGUE (Push Bombing)
→ Attacker sends repeated push notifications
→ User accidentally approves to stop alerts
→ Prevention: Number matching, limited attempts

ATTACK 2: SIM SWAPPING
→ Attacker convinces carrier to port number
→ SMS codes go to attacker's SIM
→ Prevention: Use TOTP/WebAuthn instead of SMS

ATTACK 3: REAL-TIME PHISHING (EvilGinx2, Modlishka)
→ Attacker proxies login page in real-time
→ Captures TOTP code as user enters it
→ Prevention: WebAuthn (bound to domain)

ATTACK 4: SS7 INTERCEPTION
→ Exploit SS7 telecom protocol vulnerabilities
→ Intercept SMS in transit
→ Prevention: Don't use SMS for MFA

ATTACK 5: SOCIAL ENGINEERING
→ Call user pretending to be IT support
→ Ask for TOTP code "for verification"
→ Prevention: User education, WebAuthn

ATTACK 6: BACKUP CODE THEFT
→ Steal stored backup codes from user
→ Prevention: Encrypted storage, limited codes
```

---

## 6. MFA Best Practices

```
IMPLEMENTATION CHECKLIST:

□ Offer MFA for ALL users (mandatory for privileged accounts)
□ Support multiple MFA methods (TOTP + WebAuthn minimum)
□ Prefer WebAuthn/FIDO2 for phishing resistance
□ Avoid SMS-based MFA (use as last resort only)
□ Generate and display backup/recovery codes
□ Implement MFA bypass rate limiting
□ Log all MFA events (setup, verify, fail, bypass)
□ Don't allow MFA disable without re-verification
□ Support MFA on all login flows (web, mobile, API)
□ Implement step-up authentication for sensitive operations
□ Number matching for push notifications (not just approve/deny)
□ Grace period for TOTP (±30 seconds)
□ Prevent TOTP replay (track used codes)
□ Encrypted MFA secret storage
□ MFA recovery process with identity verification
```

---

## Summary Table

| MFA Method | Best For | Vulnerability | Phishing Resistant |
|-----------|---------|---------------|-------------------|
| FIDO2/WebAuthn | Highest security | Physical theft | ✅ Yes |
| TOTP | Good balance | Real-time phishing | ❌ No |
| Push | User convenience | MFA fatigue | ❌ No |
| SMS | Legacy compatibility | SIM swap, SS7 | ❌ No |

---

## Revision Questions

1. What are the three authentication factor categories? Why must MFA use factors from different categories?
2. Implement TOTP setup and verification in Python, including backup code generation.
3. How does WebAuthn/FIDO2 achieve phishing resistance? Explain the cryptographic binding.
4. List five attacks against MFA and the prevention for each.
5. Why is SMS-based MFA considered insecure? What are better alternatives?
6. Design an MFA rollout plan for an organization transitioning from password-only authentication.

---

*Previous: [04-password-requirements.md](04-password-requirements.md) | Next: [06-account-recovery.md](06-account-recovery.md)*

---

*[Back to README](../README.md)*
