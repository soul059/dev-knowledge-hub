# AD Security Best Practices

## Unit 9 - Topic 6 | AD Hardening

---

## Overview

**AD Security Best Practices** provides a comprehensive checklist of defensive measures for hardening Active Directory environments. This topic consolidates all defensive strategies covered throughout the course into an actionable security framework.

---

## 1. AD Security Checklist

```
AD HARDENING PRIORITY MATRIX:

🔴 CRITICAL (Implement Immediately):
├── Tiered administration model
├── Separate admin accounts per tier
├── Disable LLMNR and NBT-NS
├── Enable SMB signing
├── LAPS deployment
├── Patch management (especially DCs)
├── Strong password policy
└── Disable print spooler on DCs

🟡 HIGH (Implement Within 30 Days):
├── Protected Users group (for DAs)
├── Credential Guard on Tier 0
├── Fine-grained password policies
├── GPP password cleanup
├── Audit ACLs on sensitive objects
├── AdminSDHolder monitoring
├── krbtgt password rotation
└── MachineAccountQuota = 0

🟢 MEDIUM (Implement Within 90 Days):
├── PAW deployment for Tier 0
├── Credential Guard everywhere
├── AD CS template hardening
├── Network segmentation
├── Authentication Policy Silos
├── SIEM with AD-specific detections
├── Sysmon on all systems
└── Advanced auditing configuration
```

---

## 2. Authentication Hardening

```powershell
# === PASSWORD POLICY ===
# Minimum:
# Length: 14+ characters (20+ for admin/service)
# Complexity: Enabled
# History: 24 passwords remembered
# Maximum age: 90 days (admin), 365 days (users)
# Lockout threshold: 5 attempts
# Lockout duration: 30 minutes

# Fine-grained password policy for admins:
New-ADFineGrainedPasswordPolicy -Name "AdminPasswordPolicy" \
  -Precedence 10 -MinPasswordLength 20 \
  -PasswordHistoryCount 24 -MaxPasswordAge 60.00:00:00 \
  -LockoutThreshold 3 -LockoutDuration 00:30:00
Add-ADFineGrainedPasswordPolicySubject -Identity "AdminPasswordPolicy" \
  -Subjects "Domain Admins"

# === DISABLE WEAK PROTOCOLS ===
# Disable NTLM where possible (audit first!):
# GPO: Network security: Restrict NTLM
# Start with: Audit all NTLM authentication
# Then: Deny NTLM for specific accounts/servers

# Disable LM hashes:
# GPO: Network security: Do not store LAN Manager hash

# Disable NTLMv1:
# GPO: Network security: LAN Manager authentication level
# → Send NTLMv2 response only. Refuse LM & NTLM

# === KERBEROS HARDENING ===
# Disable RC4 for Kerberos (use AES):
# GPO: Network security: Configure encryption types
# → Enable AES128 and AES256, disable DES and RC4
# WARNING: Test thoroughly — some systems need RC4

# === gMSA FOR SERVICE ACCOUNTS ===
# Replace regular service accounts with gMSA:
New-ADServiceAccount -Name "gMSA_SQL" \
  -DNSHostName "gmsa-sql.corp.local" \
  -PrincipalsAllowedToRetrieveManagedPassword "SQL_Servers" \
  -KerberosEncryptionType AES128,AES256
# 120-character password, auto-rotated, can't be Kerberoasted
```

---

## 3. Network and Protocol Hardening

```bash
# === DISABLE LLMNR ===
# GPO: Computer Config → Admin Templates → Network →
# DNS Client → Turn off multicast name resolution = Enabled

# === DISABLE NBT-NS ===
# GPO: Computer Config → Admin Templates → Network →
# Network Connections → Prohibit use of Internet Connection Sharing
# Plus: DHCP option to disable NetBIOS

# === ENABLE SMB SIGNING ===
# GPO: Security Settings → Local Policies → Security Options
# "Microsoft network server: Digitally sign communications (always)" = Enabled
# "Microsoft network client: Digitally sign communications (always)" = Enabled
# Prevents SMB relay attacks!

# === DISABLE SPOOLER ON DCs ===
# PrinterBug and PrintNightmare target this service
# GPO applied to Domain Controllers OU:
# Computer Config → Windows Settings → Security Settings →
# System Services → Print Spooler → Disabled

# === LDAP SIGNING ===
# GPO: Domain controller: LDAP server signing requirements
# → Require signing
# Prevents LDAP relay attacks

# === LDAP CHANNEL BINDING ===
# Registry on DCs:
# HKLM\System\CurrentControlSet\Services\NTDS\Parameters
# LdapEnforceChannelBinding = 2

# === NETWORK SEGMENTATION ===
# Tier 0 (DCs): Separate VLAN, strict ACLs
# Tier 1 (Servers): Separate VLAN
# Tier 2 (Workstations): Separate VLAN
# Block: Workstation-to-workstation SMB (port 445)
# Block: Workstation-to-DC direct access (use jump servers)
```

---

## 4. Monitoring and Detection

```
ESSENTIAL AD MONITORING:

EVENT IDS TO ALERT ON:
├── 4720: User account created
├── 4728/4732: Member added to security group
├── 4756: Member added to universal group
├── 4768: Kerberos TGT request (etype 23 = suspicious)
├── 4769: Kerberos TGS request (bulk = Kerberoasting)
├── 4771: Kerberos pre-auth failed
├── 4776: NTLM authentication
├── 4624: Logon (Type 3 from unusual source)
├── 4662: Operation on AD object (DCSync detection)
├── 5136: Directory object modified
├── 5137: Directory object created
├── 7045: New service installed
└── 4697: Service installed on system

SYSMON EVENTS:
├── Event 1: Process creation
├── Event 3: Network connection
├── Event 7: Image loaded (DLL)
├── Event 10: Process access (LSASS access!)
├── Event 11: File creation
├── Event 13: Registry modification
├── Event 20-21: WMI events
└── Event 25: Process tampering

TOOLS FOR AD MONITORING:
├── Microsoft ATA / Azure ATP / Defender for Identity
├── Sysmon + Sigma rules
├── BloodHound (defensive mode)
├── PingCastle (regular assessments)
├── Purple Knight (AD security assessment)
├── Elastic SIEM with AD module
└── Splunk with AD threat detection
```

---

## 5. Regular Maintenance

| Task | Frequency |
|------|:---------:|
| **Patch DCs** | Monthly (priority) |
| **Review DA membership** | Weekly |
| **Run PingCastle assessment** | Monthly |
| **Rotate krbtgt password** | Every 180 days |
| **Review GPO permissions** | Monthly |
| **Audit AdminSDHolder ACL** | Monthly |
| **Review trust relationships** | Quarterly |
| **Review service account SPNs** | Quarterly |
| **LAPS password audit** | Monthly |
| **AD CS template review** | Quarterly |
| **BloodHound defensive scan** | Monthly |
| **Test backup/restore** | Quarterly |

```
AD SECURITY MATURITY LEVELS:

LEVEL 1 — BASICS:
├── Strong password policy
├── LAPS deployed
├── LLMNR/NBT-NS disabled
├── SMB signing enabled
└── Regular patching

LEVEL 2 — INTERMEDIATE:
├── Tiered admin model
├── Protected Users for DAs
├── Credential Guard on DCs/PAWs
├── Fine-grained password policies
├── Basic SIEM monitoring
└── gMSA for service accounts

LEVEL 3 — ADVANCED:
├── Full PAW deployment
├── Authentication Policy Silos
├── AD CS hardened
├── Advanced SIEM with AD rules
├── Regular BloodHound assessments
├── Incident response plan
└── Red team exercises

LEVEL 4 — MATURE:
├── Zero Trust architecture
├── Continuous monitoring
├── Automated response
├── Regular purple team exercises
├── Bastion forest (ESAE)
└── Full compliance framework
```

---

## Summary Table

| Category | Top Priority |
|----------|-------------|
| **Authentication** | Tiered admin, Protected Users, Credential Guard |
| **Credentials** | LAPS, gMSA, strong passwords |
| **Network** | Disable LLMNR, SMB signing, segmentation |
| **Monitoring** | Sysmon, SIEM with AD rules, BloodHound |
| **Maintenance** | Patch DCs, rotate krbtgt, audit ACLs |
| **Assessment** | PingCastle, Purple Knight, red team |

---

## Quick Revision Questions

1. **What are the top 3 most impactful AD hardening measures?**
2. **Why should the Print Spooler be disabled on Domain Controllers?**
3. **How often should the krbtgt password be rotated?**
4. **What Event IDs indicate potential Kerberoasting?**
5. **What is the difference between gMSA and regular service accounts?**

---

[← Previous: Privileged Access Workstations](05-privileged-access-workstations.md)

---

*Unit 9 - Topic 6 of 6 | [Back to README](../README.md)*

---

## 🎓 Subject 3.2: Active Directory Security — COMPLETE

**Congratulations!** You've completed all 9 units covering the full lifecycle of Active Directory security — from fundamentals through attack techniques to hardening best practices.
