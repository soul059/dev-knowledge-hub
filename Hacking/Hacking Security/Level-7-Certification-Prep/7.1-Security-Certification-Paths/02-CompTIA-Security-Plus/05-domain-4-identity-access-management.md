# Unit 2: CompTIA Security+ — Topic 5: Domain 4 — Identity and Access Management

## Overview

**Identity and Access Management (IAM)** is a core component of Domain 4: Security Operations (28%). IAM controls who can access what resources and under what conditions. This is one of the most heavily tested areas on the Security+ exam.

---

## 1. Authentication Concepts

```
AUTHENTICATION FACTORS:

  Factor           | Type            | Example
  Something you    | Knowledge       | Password, PIN
  know             |                 |
  Something you    | Possession      | Smart card, token
  have             |                 | Phone (OTP)
  Something you    | Inherence       | Fingerprint, iris
  are              | (Biometric)     | Voice, face
  Somewhere you    | Location        | GPS, IP address
  are              |                 |
  Something you    | Behavior        | Typing pattern
  do               |                 | Gait analysis

MFA (Multi-Factor Authentication):
  → Must use 2+ DIFFERENT factor types
  → Password + OTP = MFA ✓ (know + have)
  → Password + PIN = NOT MFA ✗ (know + know)
  → Fingerprint + face = NOT MFA ✗ (are + are)

AUTHENTICATION PROTOCOLS:
  Protocol    | Use
  Kerberos    | Windows AD authentication
  NTLM        | Legacy Windows auth
  RADIUS      | Network access (AAA)
  TACACS+     | Cisco device admin
  LDAP/LDAPS  | Directory services
  SAML        | Web SSO (XML-based)
  OAuth       | Authorization (not auth)
  OpenID Connect| Authentication layer on OAuth
  FIDO2/WebAuthn| Passwordless auth

EXAM TIP: Know the difference between:
  → Authentication (who you are)
  → Authorization (what you can do)
  → Accounting (what you did)
  → SAML = authentication/SSO
  → OAuth = authorization
```

---

## 2. Access Control Models

```
ACCESS CONTROL MODELS:

  Model | Description         | Example
  DAC   | Owner controls      | File permissions
        | access              | (Windows NTFS)
  MAC   | System enforces     | Military
        | labels/clearances   | classifications
  RBAC  | Role-based access   | AD groups
        |                     | Job functions
  ABAC  | Attribute-based     | Time + location
        | policies            | + role + device
  Rule  | Conditional rules   | Firewall rules
        |                     |

DAC (Discretionary):
  → Owner decides who gets access
  → Flexible but hard to manage at scale
  → NTFS permissions, Linux chmod

MAC (Mandatory):
  → System-enforced, label-based
  → Top Secret > Secret > Confidential
  → Bell-LaPadula (no read up, no write down)
  → Biba (no read down, no write up)
  → Used in government/military

RBAC (Role-Based):
  → Access based on job role
  → Most common in enterprise
  → AD groups = RBAC implementation
  → Easy to manage, scalable

ABAC (Attribute-Based):
  → Policy-based on multiple attributes
  → User + resource + action + environment
  → Most flexible and granular
  → Used in cloud and zero trust

EXAM TIP: Expect questions comparing models.
  → "Owner sets permissions" = DAC
  → "Classification labels" = MAC  
  → "Job function groups" = RBAC
  → "Multiple attributes evaluated" = ABAC
```

---

## 3. Account Management

```
ACCOUNT MANAGEMENT:

ACCOUNT TYPES:
  → User accounts: Standard users
  → Privileged accounts: Admin access
  → Service accounts: Application use
  → Shared accounts: Multiple users (avoid)
  → Guest accounts: Limited temporary
  → Default accounts: Built-in (disable)

ACCOUNT POLICIES:
  → Password complexity requirements
  → Password history (prevent reuse)
  → Password length minimum
  → Account lockout (threshold + duration)
  → Session timeout
  → Account expiration
  → Time-of-day restrictions
  → Geographic restrictions

PRIVILEGED ACCESS:
  → PAM (Privileged Access Management)
  → Just-in-time (JIT) access
  → Credential vaulting
  → Session recording
  → Privileged escalation workflows
  → Emergency/break-glass accounts

ACCOUNT LIFECYCLE:
  Provisioning → Review → Modification → 
  Deprovisioning
  
  → Onboarding: Create accounts, assign roles
  → Transfers: Modify access for new role
  → Offboarding: Disable/delete accounts
  → Access reviews: Regular audit of access
  → Orphaned accounts: Remove unused accounts
```

---

## 4. Federation and SSO

```
FEDERATION AND SSO:

SINGLE SIGN-ON (SSO):
  → One login for multiple applications
  → Reduces password fatigue
  → Centralized authentication
  → Risk: Compromise = access to all

FEDERATION:
  → Trust between organizations
  → Cross-domain authentication
  → Uses standards (SAML, OAuth, OIDC)
  → Identity Provider (IdP) ↔ Service Provider (SP)

SAML (Security Assertion Markup Language):
  → XML-based SSO protocol
  → Web browser SSO
  → IdP authenticates, sends assertion
  → SP trusts assertion, grants access

OAuth 2.0:
  → Authorization framework
  → Delegates access (not authentication)
  → Access tokens
  → Used by APIs (Google, Facebook)

OpenID Connect (OIDC):
  → Authentication layer on OAuth 2.0
  → ID tokens
  → Modern SSO standard
  → Used for "Login with Google/Microsoft"

EXAM KEY DIFFERENCES:
  → SAML = Enterprise SSO, XML, browser
  → OAuth = Authorization, API access
  → OIDC = Authentication on top of OAuth
  → Kerberos = Windows domain SSO
```

---

## 5. Physical Access Control

```
PHYSICAL ACCESS:

PHYSICAL CONTROLS:
  → Badge/card readers
  → Biometric scanners
  → Access control vestibule (mantrap)
  → Security guards
  → Visitor management
  → Locks (cipher, electronic, biometric)
  → Bollards and barriers
  → Fencing

BIOMETRIC TYPES:
  Type        | Measurement    | Accuracy
  Fingerprint | Ridge patterns | High
  Iris scan   | Eye pattern    | Very high
  Retina scan | Blood vessels  | Very high
  Facial      | Face geometry  | Medium-High
  Voice       | Speech pattern | Medium
  Palm vein   | Vein pattern   | High
  Gait        | Walking style  | Medium

BIOMETRIC ERRORS:
  → FAR (False Acceptance Rate): Wrong person accepted
  → FRR (False Rejection Rate): Right person rejected
  → CER/EER (Crossover Error Rate): FAR=FRR point
  → Lower CER = more accurate system

EXAM TIP:
  → FAR = Type II error (accepting impostor)
  → FRR = Type I error (rejecting legitimate)
  → CER is where FAR and FRR are equal
  → Most secure: Low FAR (fewer false accepts)
```

---

## Summary Table

| Topic | Key Concepts | Exam Weight |
|-------|-------------|-------------|
| Authentication | MFA factors, protocols | Heavy |
| Access Control | DAC, MAC, RBAC, ABAC | Heavy |
| Account Mgmt | Lifecycle, PAM, policies | Medium |
| Federation/SSO | SAML, OAuth, OIDC | Medium |
| Physical Access | Biometrics, FAR/FRR | Medium |

---

## Revision Questions

1. What three factors constitute true MFA?
2. How does RBAC differ from ABAC?
3. What is the difference between SAML and OAuth?
4. What is the Crossover Error Rate (CER) in biometrics?
5. What account management practices reduce insider threat risk?

---

*Previous: [04-domain-3-architecture-and-design.md](04-domain-3-architecture-and-design.md) | Next: [06-domain-5-risk-management.md](06-domain-5-risk-management.md)*

---

*[Back to README](../README.md)*
