# Chapter 1: Azure Active Directory (Entra ID)

## Overview
Azure Active Directory (now **Microsoft Entra ID**) is Microsoft's cloud-based **identity and access management** service. It is the backbone of authentication and authorization for Azure resources, Microsoft 365, and thousands of SaaS applications. For DevOps engineers, Entra ID is fundamental to securing pipelines, managing service accounts, and implementing zero-trust architectures.

---

## 1.1 What is Azure AD / Entra ID?

```
┌────────────────────────────────────────────────────────────────┐
│                   MICROSOFT ENTRA ID                            │
│              (formerly Azure Active Directory)                  │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   ┌──────────────────────────────────────────┐                 │
│   │              TENANT                       │                 │
│   │    (Dedicated instance of Entra ID)       │                 │
│   │                                           │                 │
│   │   ┌─────────┐  ┌─────────┐  ┌─────────┐  │                 │
│   │   │  Users  │  │ Groups  │  │  Apps   │  │                 │
│   │   │ (human) │  │         │  │(registr)│  │                 │
│   │   └─────────┘  └─────────┘  └─────────┘  │                 │
│   │                                           │                 │
│   │   ┌─────────┐  ┌─────────┐  ┌─────────┐  │                 │
│   │   │ Service │  │ Managed │  │Conditional│ │                 │
│   │   │Principals│ │Identities│ │ Access   │  │                 │
│   │   └─────────┘  └─────────┘  └─────────┘  │                 │
│   │                                           │                 │
│   └──────────────────────────────────────────┘                 │
│                        │                                        │
│         ┌──────────────┼──────────────┐                         │
│         ▼              ▼              ▼                         │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                    │
│   │  Azure   │  │Microsoft │  │  SaaS    │                    │
│   │Resources │  │   365    │  │  Apps    │                    │
│   └──────────┘  └──────────┘  └──────────┘                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Azure AD vs On-Premises AD

| Feature | On-Premises AD (AD DS) | Azure AD (Entra ID) |
|---------|----------------------|---------------------|
| **Protocol** | Kerberos, LDAP, NTLM | OAuth 2.0, OIDC, SAML |
| **Structure** | Forests, Domains, OUs | Flat (tenants) |
| **Scope** | Internal network | Cloud + Internet |
| **Devices** | Domain-joined PCs | Azure AD joined/registered |
| **Group Policy** | GPOs | Conditional Access, Intune |
| **Federation** | AD FS | Built-in (B2B, B2C) |
| **Management** | ADUC, PowerShell | Portal, Graph API, CLI |

---

## 1.2 Key Concepts

### Tenant
A **tenant** is a dedicated instance of Entra ID that an organization receives when they sign up for a Microsoft cloud service. Each tenant is distinct and separate.

### Identities in Entra ID

```
┌──────────────── IDENTITY TYPES ────────────────┐
│                                                 │
│   ┌───────────────┐    ┌───────────────┐        │
│   │   USER         │    │   SERVICE     │        │
│   │  IDENTITIES    │    │  IDENTITIES   │        │
│   ├───────────────┤    ├───────────────┤        │
│   │ Cloud users   │    │ Service       │        │
│   │ Synced users  │    │ Principals    │        │
│   │ Guest users   │    │              │        │
│   │ (B2B)         │    │ Managed      │        │
│   └───────────────┘    │ Identities   │        │
│                         │              │        │
│   ┌───────────────┐    │ App          │        │
│   │   DEVICE       │    │ Registrations│        │
│   │  IDENTITIES    │    └───────────────┘        │
│   ├───────────────┤                             │
│   │ Azure AD      │    ┌───────────────┐        │
│   │   joined      │    │   GROUP       │        │
│   │ Hybrid joined │    │  IDENTITIES   │        │
│   │ Registered    │    ├───────────────┤        │
│   └───────────────┘    │ Security      │        │
│                         │ Microsoft 365 │        │
│                         │ Dynamic       │        │
│                         └───────────────┘        │
└─────────────────────────────────────────────────┘
```

---

## 1.3 Authentication Protocols

```
┌────────────── AUTHENTICATION FLOW ──────────────┐
│                                                   │
│   Client App                   Entra ID           │
│   ──────────                   ────────           │
│                                                   │
│   ┌─────────┐    1. Auth Request    ┌──────────┐ │
│   │         │ ──────────────────── ▶│          │ │
│   │  User / │                       │  Entra   │ │
│   │  App    │    2. Credentials     │   ID     │ │
│   │         │ ──────────────────── ▶│          │ │
│   │         │                       │          │ │
│   │         │    3. Token (JWT)     │          │ │
│   │         │ ◀──────────────────── │          │ │
│   └────┬────┘                       └──────────┘ │
│        │                                          │
│        │     4. Access with Token                 │
│        │                                          │
│   ┌────┴────────────────────────────┐             │
│   │        Azure Resource           │             │
│   │    (validates token via         │             │
│   │     Entra ID public keys)       │             │
│   └─────────────────────────────────┘             │
│                                                   │
│   Protocols Used:                                 │
│   • OAuth 2.0 — Authorization (access tokens)    │
│   • OpenID Connect — Authentication (ID tokens)  │
│   • SAML 2.0 — Enterprise SSO                    │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 1.4 Entra ID Editions

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| **User/Group Management** | ✅ | ✅ | ✅ |
| **SSO (unlimited apps)** | ✅ | ✅ | ✅ |
| **MFA** | ✅ (defaults) | ✅ (conditional) | ✅ (risk-based) |
| **Conditional Access** | ❌ | ✅ | ✅ |
| **Self-service password reset** | Cloud only | ✅ (hybrid) | ✅ |
| **Identity Protection** | ❌ | ❌ | ✅ |
| **PIM (Privileged Identity Mgmt)** | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ✅ |

---

## 1.5 Conditional Access

Conditional Access policies are **if-then** rules that enforce access controls based on signals.

```
┌────────────── CONDITIONAL ACCESS FLOW ──────────────┐
│                                                       │
│  SIGNALS (if...)           DECISIONS (then...)        │
│  ──────────────            ────────────────           │
│                                                       │
│  ┌─────────────┐          ┌──────────────────┐       │
│  │ • User/Group│          │ • Allow access   │       │
│  │ • Location  │──────── ▶│ • Block access   │       │
│  │ • Device    │          │ • Require MFA    │       │
│  │ • App       │          │ • Require device │       │
│  │ • Risk level│          │   compliance     │       │
│  │ • Platform  │          │ • Require managed│       │
│  └─────────────┘          │   device         │       │
│                            │ • Session control│       │
│                            └──────────────────┘       │
│                                                       │
│  Example Policy:                                      │
│  IF user is in "DevOps-Team" group                    │
│  AND accessing from outside corporate network         │
│  AND device is not compliant                          │
│  THEN require MFA + compliant device                  │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## 1.6 Azure AD Connect (Hybrid Identity)

For organizations with on-premises AD, **Azure AD Connect** synchronizes identities to the cloud.

```
┌──────────────── HYBRID IDENTITY ────────────────┐
│                                                   │
│   ON-PREMISES                    CLOUD            │
│   ────────────                   ─────            │
│                                                   │
│   ┌───────────┐   Azure AD     ┌───────────┐     │
│   │           │   Connect      │           │     │
│   │  Active   │ ═══════════▶   │  Entra    │     │
│   │ Directory │   (sync)       │    ID     │     │
│   │   (AD DS) │                │           │     │
│   │           │                │           │     │
│   └───────────┘                └───────────┘     │
│                                                   │
│   Sync Methods:                                   │
│   ─────────────                                   │
│   • Password Hash Sync (PHS) ← Recommended       │
│   • Pass-through Auth (PTA)                       │
│   • Federation (AD FS)                            │
│                                                   │
│   ┌────────────────────────────────────────┐      │
│   │ PHS: Hash of password hash synced      │      │
│   │ PTA: Auth request forwarded to on-prem │      │
│   │ Fed: Redirected to ADFS for auth       │      │
│   └────────────────────────────────────────┘      │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 1.7 Managing Users and Groups

```bash
# Create a user
az ad user create \
  --display-name "John DevOps" \
  --user-principal-name "john@contoso.onmicrosoft.com" \
  --password "TempP@ss123!" \
  --force-change-password-next-sign-in true

# List users
az ad user list --query "[].{Name:displayName, UPN:userPrincipalName}" --output table

# Create a security group
az ad group create \
  --display-name "DevOps-Engineers" \
  --mail-nickname "devops-eng"

# Add user to group
az ad group member add \
  --group "DevOps-Engineers" \
  --member-id "<user-object-id>"

# List group members
az ad group member list --group "DevOps-Engineers" --output table
```

---

## 1.8 Real-World DevOps Scenarios

### Scenario: Setting Up SSO for Azure DevOps

```
┌──────────────── SSO ARCHITECTURE ────────────────┐
│                                                    │
│  Developer                Entra ID                 │
│  ─────────                ────────                 │
│  ┌────────┐  1. Login   ┌──────────┐              │
│  │Browser │────────────▶│ Entra ID │              │
│  └────┬───┘             │ (auth)   │              │
│       │                 └─────┬────┘              │
│       │    2. Token           │                    │
│       │◀──────────────────────┘                    │
│       │                                            │
│       │    3. Access with Token                    │
│       │                                            │
│  ┌────┴──────┐  ┌───────────┐  ┌────────────┐    │
│  │   Azure   │  │  Azure    │  │  Azure     │    │
│  │  DevOps   │  │  Portal   │  │   Apps     │    │
│  └───────────┘  └───────────┘  └────────────┘    │
│                                                    │
│  Single token → access to all connected services   │
└────────────────────────────────────────────────────┘
```

---

## 1.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| User can't sign in | Account disabled or deleted | Check user status in Entra ID portal |
| MFA prompt not appearing | Conditional Access policy misconfigured | Review CA policy conditions and assignments |
| Sync not working | Azure AD Connect service stopped | Check Azure AD Connect health in portal |
| Guest user access denied | B2B invitation not redeemed | Resend invitation or check guest user status |
| Token expired | Default token lifetime exceeded | Use refresh tokens; check token lifetime policies |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Entra ID** | Cloud identity service; flat structure; uses OAuth/OIDC/SAML |
| **Tenant** | Dedicated instance per organization |
| **Identity Types** | Users, Groups, Service Principals, Managed Identities |
| **Conditional Access** | If-then policies based on signals (P1 required) |
| **Hybrid Identity** | Azure AD Connect syncs on-prem AD (PHS, PTA, or Federation) |
| **Editions** | Free, P1 (Conditional Access), P2 (Identity Protection, PIM) |
| **Protocols** | OAuth 2.0, OpenID Connect, SAML 2.0 |

---

## Quick Revision Questions

1. **What is the difference between Azure AD and on-premises Active Directory?**
   > Azure AD (Entra ID) is a cloud-based identity service using OAuth/OIDC/SAML with a flat structure, while on-premises AD uses Kerberos/LDAP with forests, domains, and OUs.

2. **What are the three hybrid identity sync methods?**
   > Password Hash Sync (PHS), Pass-through Authentication (PTA), and Federation (AD FS).

3. **What does Conditional Access require in terms of licensing?**
   > Conditional Access requires Azure AD P1 or higher license.

4. **What is a tenant in Entra ID?**
   > A tenant is a dedicated, isolated instance of Entra ID that an organization receives, representing their directory of identities.

5. **Name three types of identities managed in Entra ID.**
   > User identities (cloud, synced, guest), service principals, and managed identities.

6. **What authentication protocols does Entra ID support?**
   > OAuth 2.0 (authorization), OpenID Connect (authentication), and SAML 2.0 (enterprise SSO).

---

[⬅ Previous: Resource Groups and Subscriptions](../01-Azure-Fundamentals/05-resource-groups-and-subscriptions.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Service Principals ➡](02-service-principals.md)
