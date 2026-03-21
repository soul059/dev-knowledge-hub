# PKI Fundamentals

## Unit 7 - Topic 5 | Cryptography Basics

---

## Overview

**Public Key Infrastructure (PKI)** is a framework of roles, policies, hardware, software, and procedures used to create, manage, distribute, store, and revoke **digital certificates**. PKI is the trust foundation that makes secure internet communication (HTTPS), email signing, code signing, and VPN authentication possible.

---

## 1. How PKI Works

```
┌──────────────────────────────────────────────────────────────────┐
│                        PKI OVERVIEW                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem: How do you TRUST a public key really belongs           │
│  to who they claim to be?                                        │
│                                                                  │
│  Solution: A TRUSTED THIRD PARTY (Certificate Authority)         │
│  vouches for the identity.                                       │
│                                                                  │
│  ┌──────────┐    Request     ┌──────────────┐                   │
│  │  Entity  │───────────────►│  Certificate │                   │
│  │ (Website)│                │  Authority   │                   │
│  └──────────┘                │    (CA)      │                   │
│       │                      └──────┬───────┘                   │
│       │                             │ Verify identity           │
│       │                             │ Issue certificate          │
│       │◄────────────────────────────┘                           │
│       │  Certificate contains:                                   │
│       │  • Entity's public key                                   │
│       │  • Entity's identity info                                │
│       │  • CA's digital signature                                │
│       │  • Validity period                                       │
│       │                                                          │
│  User's browser trusts the CA ──► Trusts the certificate        │
│  ──► Trusts the public key ──► Secure communication             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. PKI Components

| Component | Description |
|-----------|-------------|
| **Certificate Authority (CA)** | Issues and signs digital certificates (trusted root) |
| **Registration Authority (RA)** | Verifies identity before CA issues certificate |
| **Digital Certificate** | Electronic document binding public key to identity |
| **Certificate Revocation List (CRL)** | List of revoked certificates |
| **OCSP** | Online Certificate Status Protocol (real-time cert validation) |
| **Certificate Store** | Where certificates are stored (browser, OS) |

---

## 3. Certificate Chain of Trust

```
┌──────────────────────────────────────────────────────────────┐
│                CHAIN OF TRUST                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐                                        │
│  │  ROOT CA          │  Self-signed, pre-installed in        │
│  │  (Trusted Root)   │  browsers/OS. Ultimate trust anchor.  │
│  └────────┬─────────┘                                        │
│           │ Signs                                            │
│           ▼                                                  │
│  ┌──────────────────┐                                        │
│  │  INTERMEDIATE CA  │  Signed by Root CA.                   │
│  │  (Issuing CA)     │  Actually issues end-entity certs.    │
│  └────────┬─────────┘                                        │
│           │ Signs                                            │
│           ▼                                                  │
│  ┌──────────────────┐                                        │
│  │  END-ENTITY CERT │  The website/server certificate.       │
│  │  (example.com)   │  Contains the server's public key.     │
│  └──────────────────┘                                        │
│                                                              │
│  Browser validates the ENTIRE chain:                         │
│  End cert → Intermediate → Root (trusted in browser)         │
│  If any link breaks → ❌ Certificate error                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. X.509 Certificate Contents

| Field | Description |
|-------|-------------|
| **Version** | X.509 version (usually v3) |
| **Serial Number** | Unique identifier for the certificate |
| **Signature Algorithm** | Algorithm used to sign (e.g., SHA256withRSA) |
| **Issuer** | CA that issued the certificate |
| **Validity** | Not Before / Not After dates |
| **Subject** | Entity the certificate identifies (CN=example.com) |
| **Public Key** | Subject's public key |
| **Extensions** | Key Usage, Subject Alternative Names (SAN), etc. |
| **Signature** | CA's digital signature over the certificate |

---

## 5. Certificate Types

| Type | Validation Level | Use Case |
|------|-----------------|----------|
| **DV (Domain Validation)** | Proves domain ownership | Small websites, blogs |
| **OV (Organization Validation)** | Verifies organization identity | Business websites |
| **EV (Extended Validation)** | Thorough organization verification | Banks, e-commerce |
| **Wildcard** | Covers all subdomains (*.example.com) | Multiple subdomains |
| **SAN (Subject Alternative Name)** | Covers multiple specific domains | Multi-domain hosting |
| **Code Signing** | Signs software executables | Software distribution |
| **Client Certificate** | Authenticates users/devices | VPN, mutual TLS |

---

## 6. Certificate Lifecycle

```
REQUEST ──► VALIDATE ──► ISSUE ──► USE ──► RENEW/REVOKE ──► EXPIRE

Key events:
• Renewal: Before expiry (automate with ACME/Let's Encrypt)
• Revocation: If private key compromised or entity changes
• Expiry: Certificate becomes invalid after "Not After" date
```

---

## 7. Major Certificate Authorities

| CA | Type | Notes |
|----|------|-------|
| **Let's Encrypt** | Free, automated DV | Most popular, ACME protocol |
| **DigiCert** | Commercial | Enterprise, EV certificates |
| **Sectigo (Comodo)** | Commercial | Wide range of products |
| **GlobalSign** | Commercial | Enterprise solutions |
| **Internal/Private CA** | Self-managed | Organizations' internal certificates |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **PKI** | Framework for managing digital certificates and trust |
| **CA** | Trusted third party that issues and signs certificates |
| **Chain of Trust** | Root CA → Intermediate CA → End-Entity certificate |
| **X.509** | Standard format for digital certificates |
| **Certificate Types** | DV, OV, EV — increasing levels of validation |
| **Revocation** | CRL and OCSP for checking if cert is still valid |

---

## Quick Revision Questions

1. **What problem does PKI solve?**
2. **What is the role of a Certificate Authority?**
3. **Explain the Chain of Trust from Root CA to end-entity certificate.**
4. **What is the difference between DV, OV, and EV certificates?**
5. **What are CRL and OCSP, and why are they important?**
6. **What happens when a certificate's private key is compromised?**

---

[← Previous: Digital Signatures](04-digital-signatures.md) | [Next: Common Algorithms →](06-common-algorithms.md)

---

*Unit 7 - Topic 5 of 6 | [Back to README](../README.md)*
