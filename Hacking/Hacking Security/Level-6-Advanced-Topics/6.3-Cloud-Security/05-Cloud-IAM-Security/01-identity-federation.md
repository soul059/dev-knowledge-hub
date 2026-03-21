# Unit 5: Cloud IAM Security — Topic 1: Identity Federation

## Overview

**Identity Federation** allows organizations to use existing identity systems to authenticate users to cloud services without creating separate cloud accounts. Federation bridges on-premises identity providers (IdPs) with cloud service providers using standards like SAML 2.0, OpenID Connect (OIDC), and OAuth 2.0. Proper federation implementation is critical for secure, scalable cloud identity management.

---

## 1. Federation Concepts

```
IDENTITY FEDERATION MODEL:

  ┌─────────────────┐     Trust     ┌────────────────┐
  │ Identity Provider│◄────────────►│ Service Provider│
  │ (IdP)           │              │ (SP/Cloud)      │
  │                 │   Tokens     │                 │
  │ • On-prem AD    │─────────────►│ • AWS           │
  │ • Okta          │              │ • Azure         │
  │ • Ping Identity │              │ • GCP           │
  │ • Azure AD      │              │ • SaaS Apps     │
  └─────────────────┘              └────────────────┘

FEDERATION FLOW (SAML):
  1. User accesses cloud resource
  2. Cloud redirects to IdP
  3. User authenticates at IdP (password + MFA)
  4. IdP generates SAML assertion
  5. Assertion sent to cloud SP
  6. Cloud validates assertion
  7. User receives temporary credentials
  8. Access granted

  User → Cloud App → Redirect to IdP → Auth → 
  SAML Assertion → Cloud → Temp Credentials → Access

PROTOCOLS:
  SAML 2.0:
  → XML-based
  → Most common for enterprise SSO
  → Browser-based flows
  → Used by AWS, Azure, GCP
  → Supports assertions, attributes, groups
  
  OpenID Connect (OIDC):
  → JSON-based (JWT tokens)
  → Built on OAuth 2.0
  → Modern, API-friendly
  → Used by GCP Workload Identity
  → Mobile and web application friendly
  
  OAuth 2.0:
  → Authorization framework
  → Delegate access without sharing credentials
  → Access tokens, refresh tokens
  → Used for API authorization
  
  WS-Federation:
  → Microsoft ecosystem
  → ADFS-based
  → Being replaced by SAML/OIDC
```

---

## 2. Cloud Provider Federation

```
AWS FEDERATION:

  AWS IAM Identity Center (SSO):
  → Centralized SSO for AWS
  → Integrates with external IdPs
  → Permission sets for AWS accounts
  → Automatic role creation

  SAML 2.0 Federation:
  → Create IAM Identity Provider
  → Configure SAML trust
  → Map IdP groups to IAM roles
  → Users assume roles via federation
  
  # Create SAML provider
  aws iam create-saml-provider \
    --saml-metadata-document file://metadata.xml \
    --name CorporateIdP

  Web Identity Federation:
  → For mobile/web app users
  → Google, Facebook, Amazon as IdPs
  → Cognito for user pools
  → Temporary STS credentials

AZURE FEDERATION:
  → Azure AD as both IdP and SP
  → Hybrid identity with AD Connect
  → B2B for external collaboration
  → B2C for customer identity
  → SAML/OIDC/WS-Fed support
  → Conditional Access with federation

GCP FEDERATION:
  → Workforce Identity Federation
  → Workload Identity Federation
  → Cloud Identity for user management
  → SAML/OIDC support
  
  Workload Identity Federation:
  → No service account keys needed
  → External workloads get GCP access
  → AWS → GCP, Azure → GCP, GitHub → GCP
  → Short-lived credentials
  
  # Configure Workload Identity Pool
  gcloud iam workload-identity-pools create my-pool \
    --location="global" \
    --display-name="External Workloads"
```

---

## 3. Federation Security Risks

```
ATTACK VECTORS:

TOKEN THEFT:
  → Intercepting SAML assertions
  → Stealing OAuth/OIDC tokens
  → Session hijacking
  → Token replay attacks
  → Mitigation: Use HTTPS, short-lived tokens, token binding

GOLDEN SAML:
  → Attacker compromises ADFS signing certificate
  → Can forge any SAML assertion
  → Impersonate any federated user
  → Bypass MFA (assertion already authenticated)
  → Critical: Protect ADFS/IdP signing keys

  Attack Flow:
  1. Compromise ADFS server
  2. Extract token-signing certificate
  3. Forge SAML assertion for any user
  4. Present to cloud SP (AWS/Azure/GCP)
  5. Receive valid cloud credentials
  6. Full access as federated user

  Mitigation:
  → Harden ADFS servers (Tier 0)
  → Monitor certificate access
  → Rotate signing certificates
  → Use certificate HSMs
  → Detect anomalous SAML patterns

CONSENT PHISHING (Azure):
  → Attacker creates malicious app
  → Tricks user into granting permissions
  → App gets access to user's data
  → No password theft needed
  → Mitigation: Restrict app consent, use admin consent

IdP COMPROMISE:
  → If IdP is compromised, ALL federated access is compromised
  → Single point of failure
  → Must protect IdP as highest-value target
  → Implement break-glass accounts (non-federated)
```

---

## 4. Best Practices

```
FEDERATION SECURITY BEST PRACTICES:

ARCHITECTURE:
  [ ] Centralized IdP for all cloud providers
  [ ] Redundant IdP infrastructure
  [ ] Break-glass accounts (not federated)
  [ ] Session timeout policies
  [ ] Single logout implementation

AUTHENTICATION:
  [ ] MFA enforced at IdP
  [ ] Conditional/risk-based authentication
  [ ] Certificate-based authentication for high security
  [ ] Passwordless options (FIDO2)
  [ ] Phishing-resistant MFA

TOKEN SECURITY:
  [ ] Short token lifetimes (1-4 hours)
  [ ] Token encryption in transit
  [ ] Audience restriction on tokens
  [ ] Signed assertions verified
  [ ] Token revocation capabilities

MONITORING:
  [ ] Log all federation events
  [ ] Alert on anomalous federation patterns
  [ ] Monitor IdP signing certificate access
  [ ] Track token issuance volumes
  [ ] Detect impossible travel in federated logins
  [ ] Monitor for Golden SAML indicators

BREAK-GLASS PROCEDURES:
  → Non-federated admin accounts
  → Strong passwords + hardware MFA
  → Stored in secure vault
  → Tested regularly
  → Monitored for any usage
  → Only used when IdP is unavailable
```

---

## 5. Implementation Patterns

```
COMMON FEDERATION PATTERNS:

SINGLE IdP, MULTIPLE CLOUDS:
  
  ┌──────────┐
  │ Corporate │───► AWS (SAML)
  │ IdP       │───► Azure (SAML/OIDC)
  │ (Okta)    │───► GCP (SAML/OIDC)
  └──────────┘───► SaaS Apps

HYBRID IDENTITY:
  
  On-Premises AD
       │
       │ AD Connect / ADFS
       ▼
  Azure AD (Hub IdP)
       │
       ├── AWS (SAML)
       ├── GCP (SAML)
       └── SaaS (OIDC)

WORKLOAD FEDERATION:
  
  External Workload (AWS/GitHub/on-prem)
       │
       │ OIDC Token
       ▼
  GCP Workload Identity Pool
       │
       │ Short-lived GCP credentials
       ▼
  GCP Resources (no SA keys needed)

GROUP-BASED ACCESS:
  IdP Groups → Cloud Roles
  
  Group: Cloud-Admins → AWS: AdministratorAccess
  Group: Cloud-Devs → AWS: DeveloperAccess
  Group: Cloud-Readers → AWS: ReadOnlyAccess
  
  → Changes at IdP automatically reflected in cloud
  → Single source of truth for access
  → Easier access reviews
```

---

## Summary Table

| Protocol | Format | Use Case | Cloud Support |
|----------|--------|----------|---------------|
| SAML 2.0 | XML | Enterprise SSO | All clouds |
| OIDC | JWT | Modern apps, workloads | All clouds |
| OAuth 2.0 | JSON | API authorization | All clouds |
| WS-Fed | XML | Legacy Microsoft | Azure |

---

## Revision Questions

1. How does SAML 2.0 federation work at a high level?
2. What is a Golden SAML attack and how can it be mitigated?
3. Why are break-glass accounts important in federated environments?
4. How does GCP Workload Identity Federation eliminate service account keys?
5. What monitoring should be in place for federated identity?

---

*Previous: None (First topic in this unit) | Next: [02-principle-of-least-privilege.md](02-principle-of-least-privilege.md)*

---

*[Back to README](../README.md)*
