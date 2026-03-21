# Unit 2: CompTIA Security+ — Topic 7: Domain 6 — Cryptography and PKI

## Overview

Cryptography is tested across multiple Security+ domains. This topic consolidates all cryptographic concepts including symmetric/asymmetric algorithms, hashing, digital signatures, certificates, and Public Key Infrastructure (PKI).

---

## 1. Cryptographic Fundamentals

```
CORE CONCEPTS:

  Encryption: Convert plaintext → ciphertext
  Decryption: Convert ciphertext → plaintext
  Key: Secret value used in algorithm
  Algorithm: Mathematical process
  Cipher: Encryption algorithm

GOALS OF CRYPTOGRAPHY:
  → Confidentiality: Only authorized can read
  → Integrity: Data not tampered with
  → Authentication: Verify identity
  → Non-repudiation: Cannot deny actions

SYMMETRIC ENCRYPTION:
  → Same key for encrypt and decrypt
  → Fast, efficient for bulk data
  → Challenge: Key distribution
  
  Algorithm    | Key Size    | Status
  AES          | 128/192/256 | Current standard
  3DES         | 168         | Legacy (deprecated)
  DES          | 56          | Broken (never use)
  RC4          | Variable    | Broken
  Blowfish     | 32-448      | Legacy
  Twofish      | 128/192/256 | Secure alternative
  ChaCha20     | 256         | Modern stream cipher

  Block vs Stream:
  → Block: Encrypts fixed-size blocks (AES)
  → Stream: Encrypts one bit/byte at a time (RC4)

ASYMMETRIC ENCRYPTION:
  → Public key + Private key pair
  → Public encrypts, private decrypts
  → Slow, used for key exchange
  → Enables digital signatures
  
  Algorithm    | Key Size     | Use
  RSA          | 2048-4096    | Encryption, signatures
  ECC          | 256-384      | Mobile, IoT
  Diffie-Hellman| 2048+       | Key exchange only
  DSA          | 1024-3072    | Signatures only
  ElGamal      | Variable     | Encryption, signatures

EXAM TIP:
  → Symmetric = fast, same key, bulk data
  → Asymmetric = slow, key pairs, key exchange
  → Hybrid: Use asymmetric to exchange symmetric key
```

---

## 2. Hashing and Digital Signatures

```
HASHING:

  Properties:
  → One-way function (cannot reverse)
  → Fixed-length output regardless of input
  → Any change = completely different hash
  → Used for integrity verification

  Algorithm    | Output Size | Status
  MD5          | 128-bit     | Broken (collisions)
  SHA-1        | 160-bit     | Deprecated
  SHA-256      | 256-bit     | Current standard
  SHA-384      | 384-bit     | More secure
  SHA-512      | 512-bit     | High security
  SHA-3        | Variable    | Newest standard
  RIPEMD       | 160-bit     | Alternative

HMAC (Hash-based Message Authentication Code):
  → Hash + secret key
  → Provides integrity AND authenticity
  → HMAC-SHA256, HMAC-SHA512
  → Used in TLS, API authentication

DIGITAL SIGNATURES:
  → Hash the message
  → Encrypt hash with sender's PRIVATE key
  → Recipient decrypts with sender's PUBLIC key
  → Verifies: Integrity + Authentication + Non-repudiation

  Signing Process:
  Sender: Message → Hash → Encrypt(Private Key) → Signature
  Receiver: Signature → Decrypt(Public Key) → Hash
           Message → Hash → Compare with decrypted hash

PASSWORD STORAGE:
  → Never store plaintext passwords
  → Hash + Salt (random value per user)
  → Key stretching: PBKDF2, bcrypt, scrypt
  → Salt prevents rainbow table attacks
  → Pepper: Secret added to all hashes
```

---

## 3. Public Key Infrastructure (PKI)

```
PKI COMPONENTS:

  Component         | Function
  CA (Certificate   | Issues digital certificates
  Authority)        |
  RA (Registration  | Verifies identity before CA
  Authority)        | issues certificate
  CRL (Certificate  | List of revoked certificates
  Revocation List)  |
  OCSP (Online Cert | Real-time revocation check
  Status Protocol)  |
  CSR (Certificate  | Request for a certificate
  Signing Request)  |

PKI HIERARCHY:
  Root CA (offline, highly protected)
      │
  Intermediate CA (issues end-entity certs)
      │
  End-Entity Certificate (server, user)

CERTIFICATE TYPES:
  → DV (Domain Validation): Basic, domain ownership
  → OV (Organization Validation): Verify organization
  → EV (Extended Validation): Highest assurance
  → Wildcard: *.domain.com
  → SAN (Subject Alternative Name): Multiple domains
  → Self-signed: For testing (not trusted)
  → Code signing: Verify software
  → Email (S/MIME): Encrypt/sign email

CERTIFICATE FORMATS:
  Format | Extension  | Encoding
  PEM    | .pem, .crt | Base64 text
  DER    | .der, .cer | Binary
  PFX    | .pfx, .p12 | Binary (key + cert)
  P7B    | .p7b       | Certificate chain

CERTIFICATE LIFECYCLE:
  Request → Validation → Issuance → Use →
  Renewal → Revocation → Expiration
```

---

## 4. Cryptographic Applications

```
TLS/SSL:
  → TLS 1.2: Current minimum standard
  → TLS 1.3: Latest, faster, more secure
  → SSL: Deprecated (all versions)
  → Provides confidentiality + integrity
  → Uses hybrid encryption:
    Asymmetric for key exchange →
    Symmetric for data transfer

TLS HANDSHAKE (simplified):
  Client → ServerHello
  Server → Certificate + Key Exchange
  Both   → Derive session key
  Both   → Encrypted communication

S/MIME:
  → Secure email
  → Encryption + digital signatures
  → Uses X.509 certificates
  → Alternative: PGP/GPG

VPN ENCRYPTION:
  → IPSec: AH (integrity) + ESP (encryption)
  → IKE: Key exchange for IPSec
  → SSL/TLS VPN: Web-based access
  → WireGuard: Modern protocol

DISK ENCRYPTION:
  → BitLocker: Windows FDE
  → FileVault: macOS FDE
  → LUKS: Linux FDE
  → TPM: Hardware key storage
  → SED: Self-encrypting drives
  → FDE vs FLE (full disk vs file-level)

BLOCKCHAIN:
  → Distributed ledger
  → Cryptographic hashing
  → Immutable records
  → Decentralized trust
```

---

## 5. Key Management and Steganography

```
KEY MANAGEMENT:

KEY EXCHANGE METHODS:
  → Diffie-Hellman: Secure key exchange
  → RSA key exchange
  → ECDH: Elliptic curve DH
  → Out-of-band exchange
  → Key wrapping

KEY LIFECYCLE:
  Generation → Distribution → Storage →
  Use → Rotation → Destruction

KEY STORAGE:
  → HSM (Hardware Security Module)
  → TPM (Trusted Platform Module)
  → Key vault (cloud-based)
  → Smart cards
  → Never in plaintext files

KEY CONCEPTS:
  → Key escrow: Third party holds copy
  → Key recovery: Retrieve lost keys
  → Key rotation: Regular key changes
  → Perfect forward secrecy: Unique session keys
  → Ephemeral keys: Temporary, single-use

STEGANOGRAPHY:
  → Hiding data within other data
  → Image steganography (LSB)
  → Audio steganography
  → Not encryption — obscurity
  → Used with encryption for defense in depth
  → Detection: Steganalysis

OBFUSCATION:
  → Tokenization: Replace data with tokens
  → Data masking: Hide portions (****1234)
  → Pseudonymization: Replace identifiers
  → Homomorphic encryption: Compute on encrypted data

QUANTUM COMPUTING IMPACT:
  → Threatens RSA and ECC
  → Post-quantum cryptography needed
  → NIST standardizing PQC algorithms
  → Symmetric less affected (double key size)
  → Harvest now, decrypt later threat
```

---

## Summary Table

| Topic | Key Concepts | Must Know |
|-------|-------------|----------|
| Symmetric | AES standard, same key | Algorithms + key sizes |
| Asymmetric | RSA, ECC, key pairs | Use cases |
| Hashing | SHA-256, HMAC | Properties + algorithms |
| PKI | CA, certificates, CRL | Components + cert types |
| Applications | TLS, S/MIME, VPN, FDE | Implementation |

---

## Revision Questions

1. What is the difference between symmetric and asymmetric encryption?
2. How does a digital signature provide non-repudiation?
3. What is the role of a Certificate Authority in PKI?
4. What is perfect forward secrecy?
5. How does quantum computing threaten current cryptography?

---

*Previous: [06-domain-5-risk-management.md](06-domain-5-risk-management.md) | Next: [08-study-resources.md](08-study-resources.md)*

---

*[Back to README](../README.md)*
