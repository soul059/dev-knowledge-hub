# Digital Signatures

## Unit 7 - Topic 4 | Cryptography Basics

---

## Overview

A **digital signature** is a cryptographic technique that provides **authentication** (who sent it), **integrity** (it hasn't been modified), and **non-repudiation** (the sender can't deny sending it). It uses asymmetric cryptography — the sender signs with their private key, and anyone can verify with the sender's public key.

---

## 1. How Digital Signatures Work

```
┌──────────────────────────────────────────────────────────────────┐
│                  DIGITAL SIGNATURE PROCESS                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SIGNING (Sender - Alice):                                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌─────────────┐ │
│  │ Original │──►│  HASH    │──►│  Hash    │──►│ ENCRYPT     │ │
│  │ Message  │   │ (SHA-256)│   │  Digest  │   │ hash with   │ │
│  └──────────┘   └──────────┘   └──────────┘   │ Alice's     │ │
│                                                │ PRIVATE key │ │
│                                                └──────┬──────┘ │
│                                                       │        │
│                                                       ▼        │
│                                              ┌─────────────┐   │
│  SEND: Original Message + Digital Signature  │  SIGNATURE  │   │
│                                              └─────────────┘   │
│                                                                  │
│  VERIFICATION (Receiver - Bob):                                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                    │
│  │ Received │──►│  HASH    │──►│  Hash 1  │──┐                 │
│  │ Message  │   │ (SHA-256)│   │          │  │                 │
│  └──────────┘   └──────────┘   └──────────┘  │ COMPARE        │
│                                               │                 │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐  │                 │
│  │Signature │──►│ DECRYPT  │──►│  Hash 2  │──┘                 │
│  │          │   │ with     │   │          │                     │
│  └──────────┘   │ Alice's  │   └──────────┘                     │
│                 │ PUBLIC   │                                     │
│                 │ key      │   Match ✅ = Authentic + Intact     │
│                 └──────────┘   No Match ❌ = Tampered or Fake    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. What Digital Signatures Provide

| Property | Description |
|----------|-------------|
| **Authentication** | Proves the message came from the claimed sender |
| **Integrity** | Proves the message was not modified after signing |
| **Non-repudiation** | Sender cannot deny having signed the message |

```
Digital Signature ≠ Encryption

ENCRYPTION:     Protects CONFIDENTIALITY (data is secret)
DIGITAL SIGN:   Protects AUTHENTICATION + INTEGRITY (data is genuine)

You can use BOTH together for full protection.
```

---

## 3. Real-World Applications

| Application | How Digital Signatures Are Used |
|-------------|-------------------------------|
| **Software Updates** | OS and app updates are signed to prevent tampering |
| **Code Signing** | Developers sign executables to prove legitimacy |
| **Email (S/MIME, PGP)** | Sign emails to verify sender and integrity |
| **SSL/TLS Certificates** | CA signs certificates to verify website identity |
| **Legal Documents** | Electronic signatures for contracts (DocuSign, etc.) |
| **Blockchain** | Every transaction is digitally signed |
| **PDF Documents** | Signed PDFs for official documents |

---

## 4. Algorithms Used for Digital Signatures

| Algorithm | Based On | Key Size | Status |
|-----------|----------|----------|--------|
| **RSA** | Integer factorization | 2048-4096 bit | ✅ Widely used |
| **DSA** | Discrete logarithm | 2048-3072 bit | ⚠️ Being phased out |
| **ECDSA** | Elliptic curve | 256-384 bit | ✅ Modern, efficient |
| **Ed25519** | Edwards curve | 256 bit | ✅ Fast, modern (SSH, crypto) |

---

## 5. Digital Signature vs Electronic Signature

| Aspect | Digital Signature | Electronic Signature |
|--------|------------------|---------------------|
| **Technology** | Cryptographic (PKI-based) | Any electronic method |
| **Security** | Very high (tamper-proof) | Varies |
| **Verification** | Mathematical verification | Visual/process verification |
| **Examples** | Code signing, S/MIME | Typed name, scanned signature, DocuSign |
| **Non-repudiation** | ✅ Strong | ⚠️ Weak |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **How it works** | Hash message → Encrypt hash with private key = Signature |
| **Provides** | Authentication + Integrity + Non-repudiation |
| **Sign with** | Sender's PRIVATE key |
| **Verify with** | Sender's PUBLIC key |
| **Algorithms** | RSA, ECDSA, Ed25519 |
| **Used in** | Software updates, TLS certificates, email, blockchain |

---

## Quick Revision Questions

1. **What three properties does a digital signature provide?**
2. **Which key is used to sign, and which to verify?**
3. **How does a digital signature differ from encryption?**
4. **Why is hashing used as part of the signing process?**
5. **Name 4 real-world applications of digital signatures.**
6. **What is the difference between a digital signature and an electronic signature?**

---

[← Previous: Hashing](03-hashing.md) | [Next: PKI Fundamentals →](05-pki-fundamentals.md)

---

*Unit 7 - Topic 4 of 6 | [Back to README](../README.md)*
