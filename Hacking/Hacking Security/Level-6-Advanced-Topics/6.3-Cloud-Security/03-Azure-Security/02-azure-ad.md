# Unit 3: Azure Security — Topic 2: Azure AD

## Overview

**Azure Active Directory (Azure AD)**, now rebranded as **Microsoft Entra ID**, is Microsoft's cloud-based identity and access management service. It is the foundation of security for Azure, Microsoft 365, and thousands of SaaS applications. Azure AD handles authentication, authorization, single sign-on, and identity governance for cloud and hybrid environments.

---

## 1. Azure AD Fundamentals

```
AZURE AD COMPONENTS:

TENANT:
  → Dedicated instance of Azure AD
  → Represents an organization
  → Has unique tenant ID
  → Contains all identity objects
  → Isolated from other tenants

IDENTITY OBJECTS:
  → Users (cloud and synced)
  → Groups (security and M365)
  → Applications (registered apps)
  → Service Principals (app identities)
  → Managed Identities (for Azure resources)
  → Devices (registered/joined)

AUTHENTICATION METHODS:
  → Password
  → Multi-Factor Authentication (MFA)
  → Passwordless (FIDO2, Windows Hello, Authenticator)
  → Certificate-based
  → SAML / OAuth 2.0 / OpenID Connect
  → WS-Federation

AZURE AD vs ON-PREMISES AD:
  Feature          | On-Prem AD    | Azure AD
  Protocol         | Kerberos/LDAP | OAuth/SAML/OIDC
  Structure        | Domains/OUs   | Flat (no OUs)
  Group Policy     | Yes           | No (use Intune)
  Authentication   | NTLM/Kerberos | Modern auth
  Directory        | LDAP          | Graph API
  Trust             | Forest trust  | B2B/B2C
  Management       | MMC/PowerShell| Portal/PowerShell/CLI
```

---

## 2. Authentication Security

```
MULTI-FACTOR AUTHENTICATION:

MFA METHODS:
  → Microsoft Authenticator app (push notification)
  → OATH TOTP (time-based codes)
  → SMS (less secure, SIM swapping risk)
  → Phone call
  → FIDO2 security keys (hardware)
  → Windows Hello for Business

CONDITIONAL ACCESS:
  → "If-then" policies for access decisions
  → Based on: user, device, location, risk, app
  → Actions: Allow, Block, Require MFA, Require compliance

  Example Policies:
  IF user is admin
  AND accessing from outside corporate network
  THEN require MFA + compliant device

  IF sign-in risk is high
  THEN block access

  IF user accesses sensitive app
  AND device is not compliant
  THEN block access

  # Policy structure:
  Assignments:
  → Users and groups (who)
  → Cloud apps (what)
  → Conditions (when/where)
    - Sign-in risk
    - Device platforms
    - Locations
    - Client apps

  Access Controls:
  → Grant: MFA, compliant device, approved app
  → Session: App enforced restrictions

IDENTITY PROTECTION:
  → Risk-based detection
  → User risk (compromised account likelihood)
  → Sign-in risk (suspicious sign-in)
  
  Risk Detections:
  → Anonymous IP address
  → Atypical travel
  → Malware-linked IP
  → Unfamiliar sign-in properties
  → Leaked credentials
  → Password spray attack

  Risk Levels: Low, Medium, High
  
  Policies:
  → High risk sign-in → Block or require MFA
  → High risk user → Require password change
  → Medium risk → Require MFA
```

---

## 3. Privileged Identity Management (PIM)

```
AZURE AD PIM:

PURPOSE:
  → Just-in-time privileged access
  → Time-bound role activation
  → Approval workflows
  → Audit trail of privilege use
  → Reduce standing admin access

HOW IT WORKS:
  1. User is ELIGIBLE for a role (not active)
  2. User requests activation when needed
  3. Approval may be required
  4. MFA verification
  5. Role is ACTIVE for limited time (e.g., 8 hours)
  6. Role automatically deactivates

  Without PIM:           With PIM:
  User → Always Admin    User → Eligible
  24/7 admin access      → Request when needed
  No audit of usage      → Approved + MFA
  Permanent risk         → Active for 4-8 hours
                         → Full audit trail

ROLES TO PROTECT:
  → Global Administrator
  → Security Administrator
  → Privileged Role Administrator
  → Exchange Administrator
  → SharePoint Administrator
  → Billing Administrator
  → User Administrator

ACCESS REVIEWS:
  → Periodic review of role assignments
  → Automated or manual
  → Reviewers: self, manager, specific users
  → Remove access if not justified
  → Regular cadence (monthly, quarterly)
```

---

## 4. Azure AD Security Assessment

```
SECURITY ASSESSMENT:

POWERSHELL / CLI ENUMERATION:
  # Azure AD PowerShell
  Connect-AzureAD
  
  # List all users
  Get-AzureADUser -All $true
  
  # List all groups
  Get-AzureADGroup -All $true
  
  # List Global Admins
  Get-AzureADDirectoryRole | Where-Object {$_.DisplayName -eq "Global Administrator"} | Get-AzureADDirectoryRoleMember
  
  # List applications
  Get-AzureADApplication -All $true
  
  # List service principals
  Get-AzureADServicePrincipal -All $true

  # Microsoft Graph PowerShell (modern)
  Connect-MgGraph -Scopes "User.Read.All","Group.Read.All"
  Get-MgUser -All
  Get-MgGroup -All

SECURITY CHECKLIST:
  [ ] MFA enabled for all users (especially admins)
  [ ] Conditional Access policies configured
  [ ] Legacy authentication blocked
  [ ] PIM enabled for privileged roles
  [ ] Access reviews configured
  [ ] Identity Protection enabled
  [ ] Self-service password reset configured securely
  [ ] Guest access restricted
  [ ] App consent policies restrictive
  [ ] Sign-in logs monitored
  [ ] Risky users investigated
  [ ] Named locations configured

COMMON ATTACK VECTORS:
  → Password spraying against Azure AD
  → Token theft (refresh token abuse)
  → Consent phishing (malicious app registration)
  → Guest account abuse
  → Overpermissioned service principals
  → Legacy authentication bypass of MFA
  → Device code phishing

TOOLS:
  → ROADtools: Azure AD enumeration and export
  → AADInternals: Azure AD security toolkit
  → TokenTactics: Token manipulation
  → MicroBurst: Azure security assessment
  → AzureHound: BloodHound for Azure AD
```

---

## 5. Hybrid Identity Security

```
AZURE AD CONNECT:
  → Syncs on-premises AD to Azure AD
  → Password Hash Sync (PHS)
  → Pass-Through Authentication (PTA)
  → Federation (ADFS)
  
  Security Considerations:
  → Azure AD Connect server is Tier 0 (critical)
  → Compromise = full cloud access
  → Must be hardened
  → Limit who can manage it
  → Monitor for changes

SYNC MODES:
  Password Hash Sync (PHS):
  → Hash of hash sent to Azure AD
  → Azure AD authenticates directly
  → Simplest, most resilient
  → Leaked credential detection
  → Recommended by Microsoft
  
  Pass-Through Authentication (PTA):
  → Authentication against on-prem AD
  → Agent on-premises validates
  → Password never leaves on-prem
  → Requires agent availability
  
  Federation (ADFS):
  → On-premises ADFS servers
  → Complex infrastructure
  → Token-based authentication
  → Highest complexity
  → Golden SAML attack risk

SECURITY BEST PRACTICES:
  → Use PHS (simplest, most secure)
  → Enable Seamless SSO carefully
  → Harden Azure AD Connect server
  → Monitor sync account permissions
  → Enable Password Protection (banned passwords)
  → Enable Smart Lockout
```

---

## Summary Table

| Feature | Purpose | Security Impact |
|---------|---------|----------------|
| MFA | Second factor | Prevents 99% of account attacks |
| Conditional Access | Context-based access | Risk-based enforcement |
| PIM | Just-in-time admin | Reduces standing privilege |
| Identity Protection | Risk detection | Automated threat response |
| Access Reviews | Periodic review | Removes stale access |
| Azure AD Connect | Hybrid identity | Sync security critical |

---

## Revision Questions

1. How does Azure AD differ from traditional on-premises Active Directory?
2. What is Conditional Access and how does it improve security?
3. How does Privileged Identity Management reduce risk?
4. What are the common attack vectors against Azure AD?
5. What are the security implications of Azure AD Connect?

---

*Previous: [01-azure-security-architecture.md](01-azure-security-architecture.md) | Next: [03-azure-rbac.md](03-azure-rbac.md)*

---

*[Back to README](../README.md)*
