# PAM Configuration

## Unit 2 - Topic 5 | Linux Hardening

---

## Overview

**PAM (Pluggable Authentication Modules)** is the authentication framework used by Linux to manage how users authenticate and what happens during login. PAM controls **password policies**, **account lockout**, **login restrictions**, and **session management** — making it a critical component for system hardening.

---

## 1. PAM Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              PAM WORKFLOW                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Application (sshd, login, su, sudo)                         │
│       │                                                      │
│       ▼                                                      │
│  PAM Configuration (/etc/pam.d/<service>)                    │
│       │                                                      │
│       ▼                                                      │
│  PAM Module Stack:                                           │
│  ┌─────────────────────────────────────────┐                │
│  │ auth       → Verify identity (password) │                │
│  │ account    → Check account validity     │                │
│  │ password   → Manage password changes    │                │
│  │ session    → Setup/teardown session     │                │
│  └─────────────────────────────────────────┘                │
│       │                                                      │
│       ▼                                                      │
│  PAM Modules (/lib/security/ or /lib64/security/)           │
│  pam_unix.so, pam_pwquality.so, pam_faillock.so, etc.       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. PAM Control Flags

| Flag | Behavior |
|------|----------|
| **required** | Must succeed, but continue checking other modules |
| **requisite** | Must succeed, fail immediately if not |
| **sufficient** | If succeeds, skip remaining modules (pass) |
| **optional** | Result only matters if it's the only module |
| **include** | Include another PAM config file |

---

## 3. Password Policy (pam_pwquality)

```bash
# /etc/pam.d/common-password (Debian/Ubuntu)
# or /etc/pam.d/system-auth (RHEL/CentOS)

password requisite pam_pwquality.so retry=3 \
    minlen=14 \
    dcredit=-1 \
    ucredit=-1 \
    ocredit=-1 \
    lcredit=-1 \
    maxrepeat=3 \
    reject_username \
    enforce_for_root

# Configuration file: /etc/security/pwquality.conf
# minlen = 14         # Minimum password length
# dcredit = -1        # At least 1 digit
# ucredit = -1        # At least 1 uppercase
# ocredit = -1        # At least 1 special character
# lcredit = -1        # At least 1 lowercase
# maxrepeat = 3       # Max 3 repeated characters
# reject_username     # Can't contain username
# enforce_for_root    # Apply rules to root too
```

---

## 4. Account Lockout (pam_faillock)

```bash
# /etc/pam.d/common-auth (Debian/Ubuntu)

# Lock account after 5 failed attempts for 900 seconds (15 min)
auth required pam_faillock.so preauth silent audit \
    deny=5 unlock_time=900 fail_interval=900

auth [default=die] pam_faillock.so authfail audit \
    deny=5 unlock_time=900

# Configuration: /etc/security/faillock.conf
# deny = 5              # Lock after 5 failures
# unlock_time = 900     # Unlock after 15 minutes
# fail_interval = 900   # Count failures within 15 minutes
# even_deny_root        # Lock root account too (use carefully!)

# View failed login attempts
faillock --user alice

# Manually unlock
faillock --user alice --reset
```

---

## 5. Other Important PAM Modules

| Module | Purpose | Configuration |
|--------|---------|--------------|
| **pam_unix.so** | Standard password authentication | Default auth module |
| **pam_limits.so** | Resource limits per user | `/etc/security/limits.conf` |
| **pam_access.so** | Login access control | `/etc/security/access.conf` |
| **pam_time.so** | Time-based access control | `/etc/security/time.conf` |
| **pam_securetty.so** | Restrict root login to specific terminals | `/etc/securetty` |
| **pam_nologin.so** | Prevent non-root logins | Create `/etc/nologin` |
| **pam_google_authenticator.so** | TOTP MFA | Google Authenticator |

```bash
# /etc/security/limits.conf — Resource limits
# Prevent fork bombs
* hard nproc 500
* hard nofile 65535

# /etc/security/access.conf — Access control
# Only allow admin group from specific network
+ : admin : 10.0.0.0/24
- : ALL : ALL
```

---

## 6. Password Aging (login.defs + chage)

```bash
# /etc/login.defs — System-wide password policies
PASS_MAX_DAYS   90      # Maximum password age
PASS_MIN_DAYS   7       # Minimum days between changes
PASS_WARN_AGE   14      # Warning before expiry
PASS_MIN_LEN    14      # Minimum length (also set in PAM)

# Per-user with chage
chage -M 90 alice       # Max 90 days
chage -m 7 alice        # Min 7 days between changes
chage -W 14 alice       # 14-day warning
chage -l alice          # View policy
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **PAM** | Pluggable authentication framework |
| **pam_pwquality** | Enforce password complexity |
| **pam_faillock** | Account lockout after failed attempts |
| **pam_limits** | Resource limits (prevent fork bombs) |
| **Password aging** | `/etc/login.defs` + `chage` |
| **MFA** | pam_google_authenticator for TOTP |

---

## Quick Revision Questions

1. **What are the four PAM module types?**
2. **How do you enforce a minimum password length of 14 characters?**
3. **How do you lock an account after 5 failed login attempts?**
4. **What is the difference between `required` and `requisite`?**
5. **How do you set password maximum age to 90 days?**

---

[← Previous: Kernel Hardening](04-kernel-hardening.md) | [Next: SSH Hardening →](06-ssh-hardening.md)

---

*Unit 2 - Topic 5 of 7 | [Back to README](../README.md)*
