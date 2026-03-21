# Unit 2: A02 - Cryptographic Failures — Topic 1: Data Classification

## Overview

Data classification is the foundation of cryptographic protection — you must understand **what data you have, how sensitive it is, and what regulations govern it** before deciding how to encrypt it. A02 (formerly "Sensitive Data Exposure") was renamed to "Cryptographic Failures" to emphasize that the root cause is often a failure to properly classify data and apply appropriate cryptographic controls. Organizations that skip classification often either over-encrypt (performance cost) or under-encrypt (security gaps).

---

## 1. Data Classification Levels

```
┌─────────────────────────────────────────────────────────────────┐
│                   DATA CLASSIFICATION LEVELS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RESTRICTED / TOP SECRET                                       │
│  ████████████████████████████████  ← Highest protection        │
│  • Encryption keys, root passwords                             │
│  • PII with financial data                                     │
│  • Trade secrets, M&A data                                     │
│  • Requires: AES-256, HSM, audit logging, MFA access          │
│                                                                 │
│  CONFIDENTIAL                                                  │
│  ██████████████████████████       ← Strong protection          │
│  • Customer PII, employee records                              │
│  • Financial reports, contracts                                │
│  • Requires: AES-256, access control, encryption at rest      │
│                                                                 │
│  INTERNAL                                                      │
│  ████████████████████             ← Moderate protection        │
│  • Internal documentation                                      │
│  • Non-sensitive business data                                 │
│  • Requires: HTTPS, access control, basic encryption          │
│                                                                 │
│  PUBLIC                                                        │
│  ████████████████                 ← Integrity only             │
│  • Marketing materials, public APIs                            │
│  • Published documentation                                     │
│  • Requires: Integrity checks (signing), HTTPS                │
└─────────────────────────────────────────────────────────────────┘
```

### Classification Decision Matrix

| Data Type | Classification | Encryption | Access Control | Audit |
|-----------|---------------|------------|----------------|-------|
| Credit card numbers | Restricted | AES-256 + tokenization | MFA + role-based | Full |
| Social Security numbers | Restricted | AES-256 | Strict RBAC | Full |
| Passwords | Restricted | Argon2/bcrypt hash | System only | Full |
| Health records (PHI) | Confidential | AES-256 | RBAC + need-to-know | Full |
| Customer emails | Confidential | AES-256 at rest | RBAC | Yes |
| Employee names | Internal | TLS in transit | Basic auth | Minimal |
| Product catalog | Public | TLS (integrity) | Public | No |

---

## 2. Regulatory Requirements

### Key Regulations and Crypto Requirements

| Regulation | Scope | Key Crypto Requirements |
|-----------|-------|------------------------|
| **PCI DSS** | Payment card data | AES-256, TLS 1.2+, key management, tokenization |
| **GDPR** | EU personal data | Encryption "appropriate to risk", pseudonymization |
| **HIPAA** | US health data | Encryption for PHI, access controls, audit trails |
| **SOX** | Financial reporting | Integrity controls, access restrictions |
| **CCPA** | California consumer data | "Reasonable security" including encryption |
| **NIST 800-53** | US federal systems | FIPS 140-2 validated crypto modules |
| **SOC 2** | Service organizations | Encryption in transit and at rest |

### PCI DSS Crypto Requirements

```
PCI DSS Requirement 3: Protect Stored Cardholder Data
├── 3.4: Render PAN unreadable anywhere it is stored
│   ├── One-way hash (SHA-256 with salt)
│   ├── Truncation (first 6 / last 4 digits only)
│   ├── Index tokens (tokenization)
│   └── Strong encryption (AES-256)
├── 3.5: Protect encryption keys
│   ├── Restrict access to fewest custodians
│   ├── Store keys in fewest locations
│   └── Store keys securely (HSM, encrypted)
└── 3.6: Key management procedures
    ├── Key generation with approved algorithms
    ├── Key distribution securely
    ├── Key rotation annually
    └── Key destruction when no longer needed

PCI DSS Requirement 4: Encrypt Transmission
├── 4.1: Use strong cryptography for transmission
│   ├── TLS 1.2 or higher
│   ├── No SSL, TLS 1.0, TLS 1.1
│   └── Strong cipher suites only
└── 4.2: Never send PAN via unencrypted channels
```

---

## 3. Data Inventory and Discovery

### Building a Data Inventory

```python
# Automated sensitive data scanner
import re
import os

PATTERNS = {
    'credit_card': r'\b(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|3[47][0-9]{13})\b',
    'ssn': r'\b\d{3}-\d{2}-\d{4}\b',
    'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
    'ip_address': r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b',
    'api_key': r'(?:api[_-]?key|apikey|api_secret)["\s:=]+["\']?([a-zA-Z0-9_-]{20,})',
    'password': r'(?:password|passwd|pwd)["\s:=]+["\']?([^\s"\']+)',
    'private_key': r'-----BEGIN (?:RSA |EC )?PRIVATE KEY-----',
}

def scan_file(filepath):
    findings = []
    try:
        with open(filepath, 'r', errors='ignore') as f:
            content = f.read()
            for data_type, pattern in PATTERNS.items():
                matches = re.findall(pattern, content, re.IGNORECASE)
                if matches:
                    findings.append({
                        'file': filepath,
                        'type': data_type,
                        'count': len(matches),
                        'classification': classify(data_type)
                    })
    except:
        pass
    return findings

def classify(data_type):
    classifications = {
        'credit_card': 'RESTRICTED',
        'ssn': 'RESTRICTED',
        'private_key': 'RESTRICTED',
        'password': 'RESTRICTED',
        'api_key': 'CONFIDENTIAL',
        'email': 'CONFIDENTIAL',
        'ip_address': 'INTERNAL',
    }
    return classifications.get(data_type, 'INTERNAL')
```

---

## 4. Data Flow Mapping

```
┌──────────────────────────────────────────────────────────────┐
│                     DATA FLOW DIAGRAM                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Browser ──[TLS 1.3]──▶ Load Balancer                       │
│                              │                               │
│                         [TLS 1.3]                            │
│                              │                               │
│                              ▼                               │
│                         Web Server                           │
│                              │                               │
│              ┌───────────────┼───────────────┐               │
│              │               │               │               │
│         [TLS/mTLS]     [TLS/mTLS]      [TLS/mTLS]          │
│              │               │               │               │
│              ▼               ▼               ▼               │
│          Database       Cache (Redis)   Payment API         │
│          [AES-256       [Encrypted]     [PCI DSS            │
│           at rest]                       compliant]          │
│                                                              │
│  CLASSIFICATION AT EACH POINT:                               │
│  Browser → Server: Confidential (PII in forms)              │
│  Server → DB: Restricted (credit cards, passwords)          │
│  Server → Cache: Confidential (session data)                │
│  Server → Payment: Restricted (card data)                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Data Lifecycle Management

| Phase | Security Controls |
|-------|------------------|
| **Creation** | Classify immediately, apply encryption |
| **Storage** | Encrypt at rest, access controls |
| **Processing** | Minimize exposure, encrypt in memory if possible |
| **Transmission** | TLS 1.2+, certificate validation |
| **Sharing** | Need-to-know, data masking, tokenization |
| **Archival** | Encrypted backups, retention policies |
| **Destruction** | Secure deletion, key destruction, certificate revocation |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Classification** | Categorize data by sensitivity level |
| **Regulatory Mapping** | Match data types to compliance requirements |
| **Data Inventory** | Know what sensitive data exists and where |
| **Data Flow** | Map how data moves between systems |
| **Lifecycle** | Apply controls from creation to destruction |
| **Impact** | Classification drives encryption decisions |

---

## Revision Questions

1. What are the four standard data classification levels? Give two examples of data in each category.
2. How do PCI DSS requirements 3 and 4 specifically mandate cryptographic protections?
3. Why is data classification the first step before implementing encryption? What happens if you skip it?
4. Create a data flow diagram for an e-commerce application and identify the classification at each point.
5. How would you build an automated sensitive data scanner to discover unprotected PII in a codebase?
6. Compare GDPR and HIPAA cryptographic requirements. Where do they overlap?

---

*Previous: None (First topic) | Next: [02-data-in-transit-protection.md](02-data-in-transit-protection.md)*

---

*[Back to README](../README.md)*
