# Brute Force Attacks

## Unit 4 - Topic 2 | Password Attacks

---

## Overview

**Brute force attacks** systematically try every possible character combination until the correct password is found. While guaranteed to succeed given enough time, they're typically the slowest approach and are used as a last resort or against short passwords.

---

## 1. How Brute Force Works

```
BRUTE FORCE SEQUENCE (4-char, lowercase only):

aaaa → aaab → aaac → ... → aaaz
aaba → aabb → ... → aazz
abaa → ... → azzz
baaa → ... → zzzz

TOTAL COMBINATIONS:
├── 4 chars, lowercase (26):     456,976
├── 4 chars, mixed case (52):    7,311,616
├── 4 chars, alphanum (62):      14,776,336
├── 8 chars, lowercase (26):     208 billion
├── 8 chars, all printable (95): 6.6 quadrillion
└── 12 chars, all printable:     540 sextillion!

TIME ESTIMATE (1 billion guesses/sec — Hashcat):
├── 6 chars, lowercase:  ~0.3 seconds
├── 8 chars, lowercase:  ~3.5 minutes
├── 8 chars, mixed+num:  ~2.5 days
├── 10 chars, all chars: ~2,530 years
└── 12 chars, all chars: ~22 million years
```

---

## 2. Online Brute Force

```bash
# === HYDRA (Online Brute Force) ===
# SSH brute force:
hydra -l admin -x 4:6:aA1 target.com ssh
# -l: username
# -x: generate passwords (min:max:charset)
# a=lowercase, A=uppercase, 1=digits

# With specific charset:
hydra -l admin -x 6:8:abcdefghijklmnop0123456789 target.com ssh

# FTP brute force:
hydra -l admin -x 4:6:aA1 target.com ftp

# HTTP form brute force:
hydra -l admin -x 4:6:aA1 \
  target.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid"

# ⚠️ ONLINE BRUTE FORCE IS:
# ├── Very slow (network latency)
# ├── Easily detected (many failed logins)
# ├── Often blocked (account lockout)
# └── Rarely practical for passwords > 6 chars
```

---

## 3. Offline Brute Force

```bash
# === HASHCAT (GPU-Based — FAST) ===
# Brute force MD5 hash:
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a
# -m 0: MD5 hash type
# -a 3: brute force (mask) attack
# ?a: all printable characters
# 6 characters of ?a

# Custom masks:
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?l?d?d
# Pattern: Uppercase + 4 lowercase + 2 digits
# Example: Admin12, Passw99

# Charset options:
# ?l = lowercase (a-z)
# ?u = uppercase (A-Z)
# ?d = digits (0-9)
# ?s = special characters
# ?a = all printable
# ?h = hex lowercase

# Incremental length:
hashcat -m 0 -a 3 hash.txt --increment --increment-min 4 --increment-max 8 ?a?a?a?a?a?a?a?a

# === JOHN THE RIPPER ===
john --incremental hash.txt
# Uses built-in incremental mode
# Automatically tries all combinations
```

---

## 4. Optimizing Brute Force

| Optimization | Description |
|-------------|-------------|
| **Mask attack** | Only try known patterns (?u?l?l?d?d) |
| **Markov chains** | Statistical letter frequency |
| **GPU acceleration** | Hashcat uses GPU (1000x faster) |
| **Distributed** | Spread across multiple machines |
| **Charset reduction** | Only try likely character sets |
| **Length limiting** | Start with short passwords |

```bash
# === SMART MASK EXAMPLES ===
# Company passwords often follow patterns:
hashcat -m 1000 hash.txt -a 3 "?u?l?l?l?l?l?d?d?s"
# Pattern: Capital + 5 lower + 2 digits + special
# Matches: Winter23!, Summer22@, Target24#

# PIN codes:
hashcat -m 0 hash.txt -a 3 "?d?d?d?d"        # 4-digit PIN
hashcat -m 0 hash.txt -a 3 "?d?d?d?d?d?d"    # 6-digit PIN
```

---

## 5. Brute Force Limitations

```
WHEN BRUTE FORCE IS PRACTICAL:
├── Short passwords (≤ 6 characters)
├── Limited character set (digits only = PIN)
├── Known pattern with mask
├── Fast hash type (MD5, NTLM)
├── Powerful GPU available
└── No lockout policy (online)

WHEN BRUTE FORCE IS IMPRACTICAL:
├── Long passwords (≥ 10 characters)
├── Full character set (95 printable)
├── Slow hash (bcrypt, Argon2)
├── Account lockout (online)
├── Rate limiting (online)
└── MFA enabled
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Brute force** | Try every possible combination |
| **Online** | Hydra — slow, detectable, lockout risk |
| **Offline** | Hashcat — GPU-accelerated, billions/sec |
| **Mask attack** | Smart patterns (?u?l?l?d?d) |
| **Limitation** | Impractical for long/complex passwords |
| **Last resort** | Try dictionary/rules first |

---

## Quick Revision Questions

1. **How does a brute force attack differ from a dictionary attack?**
2. **Why is offline brute force faster than online?**
3. **What is a mask attack in Hashcat?**
4. **When is brute force practical?**
5. **How do you estimate brute force time?**

---

[← Previous: Password Attack Types](01-password-attack-types.md) | [Next: Dictionary Attacks →](03-dictionary-attacks.md)

---

*Unit 4 - Topic 2 of 7 | [Back to README](../README.md)*
