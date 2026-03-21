# Identity Management

## Unit 8 - Topic 6 | Security Architecture

---

## Overview

**Identity and Access Management (IAM)** is the security discipline that ensures the **right people** have the **right access** to the **right resources** at the **right time** for the **right reasons**. IAM is the cornerstone of modern security architecture — especially in cloud and Zero Trust environments where identity has become the new security perimeter.

---

## 1. IAM Components

```
┌──────────────────────────────────────────────────────────────────┐
│                  IDENTITY & ACCESS MANAGEMENT                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐                                           │
│  │ IDENTIFICATION    │  "Who are you?"                          │
│  │ (Username, email) │  Claiming an identity                     │
│  └────────┬─────────┘                                           │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ AUTHENTICATION   │  "Prove it!"                               │
│  │ (Password, MFA,  │  Verifying the claimed identity           │
│  │  biometrics)     │                                           │
│  └────────┬─────────┘                                           │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ AUTHORIZATION    │  "What can you do?"                        │
│  │ (Permissions,    │  Granting access based on policies        │
│  │  roles, groups)  │                                           │
│  └────────┬─────────┘                                           │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ ACCOUNTING       │  "What did you do?"                        │
│  │ (Audit logs,     │  Tracking and logging all access          │
│  │  monitoring)     │                                           │
│  └──────────────────┘                                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Authentication Methods

| Method | Factors | Security | Example |
|--------|:-------:|:--------:|---------|
| **Password** | Knowledge (something you know) | ⚠️ Low | Traditional login |
| **MFA** | 2+ factors | ✅ High | Password + OTP |
| **Biometrics** | Inherence (something you are) | ✅ High | Fingerprint, face |
| **Smart Card** | Possession (something you have) | ✅ High | PIV card, YubiKey |
| **Certificate** | Possession + Knowledge | ✅ High | Client TLS certificate |
| **Passwordless** | Possession/Inherence | ✅ High | FIDO2, passkeys |

### Multi-Factor Authentication (MFA)

```
┌──────────────────────────────────────────────────────────────┐
│              AUTHENTICATION FACTORS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Factor 1: SOMETHING YOU KNOW                                │
│  └── Password, PIN, security question                        │
│                                                              │
│  Factor 2: SOMETHING YOU HAVE                                │
│  └── Phone (SMS/authenticator app), hardware token, smart    │
│      card, YubiKey                                           │
│                                                              │
│  Factor 3: SOMETHING YOU ARE                                 │
│  └── Fingerprint, facial recognition, iris scan, voice       │
│                                                              │
│  Factor 4: SOMEWHERE YOU ARE                                 │
│  └── GPS location, IP address, network                       │
│                                                              │
│  Factor 5: SOMETHING YOU DO                                  │
│  └── Typing pattern, gait, behavioral biometrics             │
│                                                              │
│  MFA = Using 2 or more DIFFERENT factors                     │
│  (Password + PIN is NOT MFA — both are "something you know") │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Authorization Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **DAC** (Discretionary Access Control) | Resource owner decides who gets access | File system permissions |
| **MAC** (Mandatory Access Control) | System enforces policies based on labels/clearances | Military, government |
| **RBAC** (Role-Based Access Control) | Access based on roles assigned to users | Enterprise applications |
| **ABAC** (Attribute-Based Access Control) | Access based on attributes (user, resource, environment) | Cloud, dynamic policies |
| **PBAC** (Policy-Based Access Control) | Centralized policies govern access | Modern IAM platforms |

### RBAC Example

```
┌──────────────────────────────────────────────────────────────┐
│              ROLE-BASED ACCESS CONTROL                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Users       →    Roles        →    Permissions              │
│                                                              │
│  Alice ──────►  Admin ─────────►  Full access               │
│  Bob ────────►  Analyst ───────►  Read-only + reports       │
│  Carol ──────►  Developer ─────►  Code repos + staging      │
│  Dave ───────►  Auditor ───────►  Read logs only            │
│  Eve ────────►  Guest ─────────►  Public resources only     │
│                                                              │
│  Users don't get permissions directly.                       │
│  They get ROLES, and roles have PERMISSIONS.                 │
│  Change a role = change access for all users in that role.   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Identity Federation and SSO

| Technology | Description |
|------------|-------------|
| **SSO (Single Sign-On)** | One login grants access to multiple applications |
| **SAML** | XML-based protocol for SSO between domains |
| **OAuth 2.0** | Authorization framework (API access delegation) |
| **OIDC (OpenID Connect)** | Authentication layer on top of OAuth 2.0 |
| **LDAP** | Directory protocol for identity lookup (Active Directory) |
| **Kerberos** | Ticket-based authentication (Windows domains) |
| **RADIUS** | Network access authentication (Wi-Fi, VPN) |

```
SSO FLOW:
User → Login to Identity Provider (IdP) → Receives Token
     → Access App A (token verified ✅) → No login needed
     → Access App B (token verified ✅) → No login needed
     → Access App C (token verified ✅) → No login needed

One login = access to all authorized applications
```

---

## 5. IAM Best Practices

| Practice | Description |
|----------|-------------|
| **Enforce MFA** | Require MFA for ALL users, especially admins |
| **Least Privilege** | Grant minimum access needed for the job |
| **Regular Access Reviews** | Review and revoke unnecessary access quarterly |
| **Automate Provisioning** | Use SCIM/HR integration for onboarding/offboarding |
| **Privileged Access Management** | Use PAM tools for admin accounts (CyberArk, BeyondTrust) |
| **Monitor & Alert** | Log all authentication events, alert on anomalies |
| **Password Policy** | Minimum 12+ characters, no reuse, breach detection |
| **Offboarding** | Immediately revoke all access when employee leaves |

---

## 6. Identity as the New Perimeter

```
┌──────────────────────────────────────────────────────────────┐
│          IDENTITY = THE NEW SECURITY PERIMETER                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL:                                                │
│  ┌───────────────────────────────┐                           │
│  │  Corporate Network            │                           │
│  │  Inside = Trusted ✅           │                           │
│  │  Firewall = Perimeter         │                           │
│  │  Outside = Untrusted ❌        │                           │
│  └───────────────────────────────┘                           │
│                                                              │
│  MODERN (Cloud, Remote Work, BYOD):                          │
│  Users are EVERYWHERE. There IS no network perimeter.        │
│                                                              │
│  ZERO TRUST:                                                 │
│  "Never trust, always verify"                                │
│  Every access request verified by IDENTITY:                  │
│  • Who is the user? (Authentication)                         │
│  • What device? (Device health)                              │
│  • What are they accessing? (Authorization)                  │
│  • Where are they? (Context)                                 │
│  • Is this normal behavior? (Analytics)                      │
│                                                              │
│  IDENTITY is the control plane for all access decisions.     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IAM** | Right people → right access → right resources → right time |
| **Authentication** | Verify identity (MFA = 2+ different factors) |
| **Authorization** | RBAC (role-based) is most common enterprise model |
| **SSO** | One login for multiple applications |
| **Key Protocols** | SAML, OAuth 2.0, OIDC, Kerberos, LDAP |
| **Modern Approach** | Identity is the new perimeter (Zero Trust) |
| **Best Practice** | MFA everywhere + least privilege + access reviews |

---

## Quick Revision Questions

1. **What are the four components of IAM (IAAA)?**
2. **What makes MFA more secure than a password alone?**
3. **What is the difference between RBAC and ABAC?**
4. **How does SSO improve both security and user experience?**
5. **Why is identity considered the "new perimeter" in modern security?**
6. **What should happen to a user's access immediately when they leave the organization?**

---

[← Previous: Cloud Security Basics](05-cloud-security-basics.md)

---

*Unit 8 - Topic 6 of 6 | [Back to README](../README.md)*

---

## 🎉 Subject 1.1 Complete!

You have completed all 8 units and 48 topics of **Cybersecurity Fundamentals**. Proceed to [Subject 1.2: Networking for Security](../../1.2-Networking-for-Security/) when ready.
