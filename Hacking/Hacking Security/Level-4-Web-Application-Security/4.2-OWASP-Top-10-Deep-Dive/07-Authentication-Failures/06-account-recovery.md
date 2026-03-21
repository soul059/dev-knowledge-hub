# Unit 7: A07 - Identification and Authentication Failures — Topic 6: Account Recovery

## Overview

Account recovery mechanisms (password reset, account unlock, backup access) are frequently **targeted by attackers** because they bypass normal authentication entirely. A weak recovery process can nullify even the strongest password and MFA policies. This topic covers secure account recovery design, common vulnerabilities, and implementation best practices.

---

## 1. Account Recovery Attack Vectors

```
┌─────────────────────────────────────────────────────────────────┐
│              RECOVERY MECHANISM ATTACKS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. USER ENUMERATION                                           │
│     "Email not found" → reveals which emails are registered    │
│     "Reset link sent" (only for valid emails) → same issue     │
│                                                                 │
│  2. PREDICTABLE RESET TOKENS                                   │
│     Sequential: reset_1001, reset_1002                         │
│     Timestamp: base64(timestamp + user_id)                     │
│     Weak random: rand() without crypto                         │
│                                                                 │
│  3. TOKEN LIFETIME TOO LONG                                    │
│     Token valid for 24-48 hours → extended window for theft    │
│                                                                 │
│  4. NO RATE LIMITING                                           │
│     Brute force reset tokens                                   │
│     Flood user's email with reset requests                     │
│                                                                 │
│  5. INSECURE TRANSPORT                                         │
│     Reset link sent over HTTP, not HTTPS                       │
│     Token in URL query string (logged in server/proxy logs)    │
│                                                                 │
│  6. SECURITY QUESTIONS                                         │
│     Answers easily found on social media                       │
│     "What city were you born in?" → Facebook/LinkedIn          │
│                                                                 │
│  7. ACCOUNT TAKEOVER VIA RECOVERY                              │
│     Change email → reset password → full account control       │
│     SIM swap → intercept SMS code → reset password             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Secure Password Reset Flow

```
┌──────────────────────────────────────────────────────────────────┐
│           SECURE PASSWORD RESET FLOW                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: User requests reset                                    │
│  ├── Enter email address                                        │
│  ├── Rate limit: 3 requests per hour per email                  │
│  ├── CAPTCHA after first request                                │
│  └── ALWAYS respond: "If account exists, email sent"            │
│       (identical response regardless of email validity)         │
│                                                                  │
│  Step 2: Generate secure token                                  │
│  ├── 256-bit cryptographically random token                    │
│  ├── Store hashed token in database (not plaintext!)           │
│  ├── Set expiration: 15-60 minutes maximum                     │
│  └── Invalidate any previous unused tokens                     │
│                                                                  │
│  Step 3: Send reset email                                       │
│  ├── HTTPS link only: https://app.com/reset?token=xxx          │
│  ├── Include: "If you didn't request this, ignore"             │
│  ├── Don't include username or password hints                  │
│  └── From verified sender (SPF/DKIM/DMARC)                    │
│                                                                  │
│  Step 4: User clicks link                                       │
│  ├── Verify token hash matches database                        │
│  ├── Check token not expired                                    │
│  ├── Check token not already used (single-use)                 │
│  └── Present password change form                              │
│                                                                  │
│  Step 5: Set new password                                       │
│  ├── Validate new password (length, breach check, blocklist)   │
│  ├── Hash and store new password                               │
│  ├── Invalidate the reset token                                │
│  ├── Invalidate ALL existing sessions                          │
│  ├── Require MFA re-enrollment if applicable                   │
│  ├── Send confirmation email                                    │
│  └── Log the password change event                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Implementation

```python
import secrets
import hashlib
from datetime import datetime, timedelta

# Generate reset token
def create_password_reset(email):
    """Initiate password reset process"""
    
    # ALWAYS the same response regardless of email validity
    generic_response = {"message": "If an account exists, a reset email has been sent"}
    
    # Rate limiting
    if is_rate_limited(email, limit=3, window=3600):
        return generic_response  # Don't reveal rate limiting applies
    
    user = User.query.filter_by(email=email.lower()).first()
    if not user:
        # Still return same response — prevent enumeration
        # But log the attempt
        log_security_event("password_reset_nonexistent", email=email)
        return generic_response
    
    # Invalidate previous tokens
    PasswordReset.query.filter_by(user_id=user.id, used=False).update({'used': True})
    
    # Generate cryptographically secure token
    raw_token = secrets.token_urlsafe(48)  # 384 bits
    
    # Store HASHED token (like a password)
    token_hash = hashlib.sha256(raw_token.encode()).hexdigest()
    
    reset = PasswordReset(
        user_id=user.id,
        token_hash=token_hash,
        expires_at=datetime.utcnow() + timedelta(minutes=30),
        ip_address=request.remote_addr,
        used=False
    )
    db.session.add(reset)
    db.session.commit()
    
    # Send email with raw token
    send_reset_email(user.email, raw_token)
    
    log_security_event("password_reset_initiated", user_id=user.id)
    return generic_response

# Verify and complete reset
def complete_password_reset(token, new_password):
    """Verify token and set new password"""
    
    # Hash the provided token to compare
    token_hash = hashlib.sha256(token.encode()).hexdigest()
    
    reset = PasswordReset.query.filter_by(
        token_hash=token_hash,
        used=False
    ).first()
    
    if not reset:
        return {"error": "Invalid or expired reset link"}, 400
    
    if reset.expires_at < datetime.utcnow():
        reset.used = True
        db.session.commit()
        return {"error": "Invalid or expired reset link"}, 400
    
    # Validate new password
    valid, errors = validate_password(new_password, reset.user.username, reset.user.email)
    if not valid:
        return {"error": errors}, 400
    
    # Update password
    reset.user.password_hash = hash_password(new_password)
    reset.used = True
    
    # Invalidate ALL existing sessions
    Session.query.filter_by(user_id=reset.user.id).delete()
    
    db.session.commit()
    
    # Send confirmation email
    send_password_changed_notification(reset.user.email)
    
    log_security_event("password_reset_completed", user_id=reset.user.id)
    return {"message": "Password has been reset successfully"}
```

---

## 4. MFA Recovery

```
MFA RECOVERY OPTIONS (ordered by security):

1. BACKUP CODES (Most Common)
   → 8-10 one-time codes generated during MFA setup
   → User stores securely (password manager)
   → Each code usable only once
   → Should be regenerated periodically

2. SECONDARY MFA DEVICE
   → Register multiple authenticators
   → If one lost, use another
   → Best for organizations

3. RECOVERY KEY
   → Long random string generated at MFA setup
   → Similar to backup codes but single long key
   → Must be stored securely by user

4. IDENTITY VERIFICATION (Last Resort)
   → Contact support with identity proof
   → Photo ID + selfie verification
   → Should have waiting period (24-72 hours)
   → Notify user on all channels during waiting period
   → Allow cancellation during waiting period

5. TRUSTED CONTACTS (Social Recovery)
   → Designate trusted contacts who can vouch
   → Requires M of N contacts to approve
   → Used by some crypto wallets and Facebook
```

```python
# MFA Recovery with Backup Codes
def verify_backup_code(user_id, code):
    """Verify and consume a backup code"""
    stored_codes = BackupCode.query.filter_by(
        user_id=user_id, 
        used=False
    ).all()
    
    for stored in stored_codes:
        if bcrypt.checkpw(code.encode(), stored.code_hash):
            # Mark as used (one-time only)
            stored.used = True
            stored.used_at = datetime.utcnow()
            db.session.commit()
            
            # Alert user that a backup code was used
            remaining = BackupCode.query.filter_by(
                user_id=user_id, used=False
            ).count()
            
            send_notification(user_id, 
                f"A backup code was used. {remaining} codes remaining.")
            
            if remaining <= 2:
                send_notification(user_id,
                    "⚠️ You're running low on backup codes. Generate new ones.")
            
            return True
    
    return False
```

---

## 5. Email Change Security

```
EMAIL CHANGE = CRITICAL OPERATION
(Controls password reset → controls entire account)

SECURE EMAIL CHANGE PROCESS:

1. Require current password (re-authentication)
2. Require MFA verification
3. Send confirmation to BOTH old and new email
4. Old email gets: "Your email is being changed. Click to cancel."
5. New email gets: "Confirm this email address"
6. Waiting period: 24 hours before change takes effect
7. Allow cancellation from old email during waiting period
8. Log the event and alert on all channels
9. Invalidate all sessions after email change
10. Force MFA re-enrollment
```

---

## 6. Recovery Security Checklist

```
PASSWORD RESET:
□ Generic response regardless of email existence
□ Cryptographically random tokens (256+ bits)
□ Tokens hashed before storage
□ Short expiration (15-60 minutes)
□ Single-use tokens
□ Rate limited (per email and per IP)
□ HTTPS-only reset links
□ Invalidate all sessions after reset
□ Confirmation email sent after reset
□ New password validated against policy

MFA RECOVERY:
□ Backup codes generated during MFA setup
□ Codes are one-time use
□ User warned when codes running low
□ Identity verification for last-resort recovery
□ Waiting period for account recovery (24-72 hours)
□ Notification on all channels during recovery

EMAIL CHANGE:
□ Re-authentication required
□ MFA verification required
□ Confirmation to old AND new email
□ Cancellation option from old email
□ 24-hour waiting period
□ Session invalidation after change
```

---

## Summary Table

| Recovery Method | Security Level | User Experience | Risk |
|----------------|---------------|-----------------|------|
| Email reset (properly implemented) | High | Good | Token theft, email compromise |
| Backup codes | High | Medium | Physical theft, storage |
| Secondary MFA device | Very High | Medium | Loss of all devices |
| Identity verification | Medium | Poor | Social engineering |
| Security questions | Very Low | Good | OSINT, social media |

---

## Revision Questions

1. Why should password reset responses be identical whether or not the email exists?
2. Implement a complete secure password reset flow with rate limiting, token hashing, and session invalidation.
3. Why should reset tokens be hashed before database storage?
4. Design a MFA recovery system using backup codes with proper security controls.
5. Why is email change a critical security operation? What controls should protect it?
6. What are the risks of security questions for account recovery? Why does NIST deprecate them?

---

*Previous: [05-mfa-implementation.md](05-mfa-implementation.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
