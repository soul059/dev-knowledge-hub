# Unit 2: A02 - Cryptographic Failures — Topic 3: Data at Rest Protection

## Overview

Data at rest refers to data stored on disks, databases, backups, and archives. Protecting data at rest through encryption ensures that even if physical media or database dumps are stolen, the data remains **unreadable without the encryption keys**. This topic covers full-disk encryption, database encryption (TDE), column-level encryption, file-level encryption, cloud key management services, and — critically — password hashing with modern algorithms (bcrypt, scrypt, Argon2).

---

## 1. Encryption at Rest Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                 ENCRYPTION AT REST LAYERS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Layer 1: FULL-DISK ENCRYPTION (FDE)                           │
│  ┌─────────────────────────────────────────────────────┐       │
│  │  Entire disk encrypted (BitLocker, LUKS, FileVault) │       │
│  │  Protects: Physical theft, decommissioned drives    │       │
│  │  Weakness: Data accessible when system is running   │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
│  Layer 2: DATABASE ENCRYPTION (TDE)                            │
│  ┌─────────────────────────────────────────────────────┐       │
│  │  Database engine encrypts data files automatically  │       │
│  │  Protects: Database file theft, backup theft        │       │
│  │  Weakness: Data decrypted in memory during queries  │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
│  Layer 3: COLUMN-LEVEL ENCRYPTION                              │
│  ┌─────────────────────────────────────────────────────┐       │
│  │  Individual columns encrypted (SSN, credit cards)   │       │
│  │  Protects: SQL injection data extraction, DBA access│       │
│  │  Weakness: Performance overhead, can't search       │       │
│  └─────────────────────────────────────────────────────┘       │
│                                                                 │
│  Layer 4: APPLICATION-LEVEL ENCRYPTION                         │
│  ┌─────────────────────────────────────────────────────┐       │
│  │  Application encrypts before storing               │       │
│  │  Protects: All layers below, including DB admins   │       │
│  │  Weakness: Key management complexity               │       │
│  └─────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

### Strategy Comparison

| Strategy | Protection Level | Performance Impact | Key Management | Use Case |
|----------|-----------------|-------------------|----------------|----------|
| **FDE** | Physical theft | Minimal | OS-managed | All servers |
| **TDE** | File-level theft | Low | DB-managed | Database servers |
| **Column** | Field-level | Medium | Application | PII, PCI data |
| **Application** | End-to-end | High | Complex | Highest sensitivity |

---

## 2. Database Encryption

### Transparent Data Encryption (TDE)

```sql
-- SQL Server TDE
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongP@ssw0rd!';
CREATE CERTIFICATE TDE_Cert WITH SUBJECT = 'TDE Certificate';

USE MyDatabase;
CREATE DATABASE ENCRYPTION KEY
    WITH ALGORITHM = AES_256
    ENCRYPTION BY SERVER CERTIFICATE TDE_Cert;

ALTER DATABASE MyDatabase SET ENCRYPTION ON;

-- PostgreSQL: Use pgcrypto for column-level
CREATE EXTENSION pgcrypto;

-- Encrypt a column value
INSERT INTO users (name, ssn_encrypted)
VALUES ('John', pgp_sym_encrypt('123-45-6789', 'encryption_key'));

-- Decrypt
SELECT name, pgp_sym_decrypt(ssn_encrypted::bytea, 'encryption_key') AS ssn
FROM users;
```

### Column-Level Encryption Example

```python
# Application-level column encryption with Python
from cryptography.fernet import Fernet
import base64

class FieldEncryptor:
    def __init__(self, key):
        self.fernet = Fernet(key)
    
    def encrypt(self, plaintext):
        """Encrypt a field value"""
        if plaintext is None:
            return None
        return self.fernet.encrypt(plaintext.encode()).decode()
    
    def decrypt(self, ciphertext):
        """Decrypt a field value"""
        if ciphertext is None:
            return None
        return self.fernet.decrypt(ciphertext.encode()).decode()

# Generate key (store securely — NOT in code!)
key = Fernet.generate_key()  # Store in KMS/Vault
encryptor = FieldEncryptor(key)

# Encrypt before storing
ssn_encrypted = encryptor.encrypt("123-45-6789")
# Store ssn_encrypted in database

# Decrypt after retrieval
ssn_plain = encryptor.decrypt(ssn_encrypted)
```

---

## 3. Password Hashing

### Why Hash, Not Encrypt?

```
ENCRYPTION (reversible):
  plaintext ──[key]──▶ ciphertext ──[key]──▶ plaintext
  If key is compromised, ALL passwords are exposed

HASHING (one-way):
  plaintext ──[hash function]──▶ hash digest
  Cannot reverse. Verify by hashing input and comparing.
  
  password123 ──▶ $2b$12$LJ3m4ys2... (bcrypt)
  
  Login check:
  hash(user_input) == stored_hash ? → Allow
```

### Algorithm Comparison

| Algorithm | Secure? | Salt | Cost Factor | Memory-Hard | Recommended |
|-----------|---------|------|-------------|-------------|-------------|
| **MD5** | ❌ NO | No | None | No | NEVER |
| **SHA-1** | ❌ NO | No | None | No | NEVER |
| **SHA-256** | ⚠️ Weak | Manual | None | No | Only with PBKDF2 |
| **PBKDF2** | ✅ | Yes | Iterations | No | Acceptable |
| **bcrypt** | ✅ | Yes | Cost factor | No | ✅ Yes |
| **scrypt** | ✅ | Yes | CPU + Memory | Yes | ✅ Yes |
| **Argon2id** | ✅✅ | Yes | Time + Memory + Parallelism | Yes | ✅✅ Best |

### Secure Password Hashing Implementation

```python
# ❌ VULNERABLE — Plain MD5
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()
# Can be cracked in seconds with rainbow tables

# ❌ VULNERABLE — SHA-256 without salt
password_hash = hashlib.sha256(password.encode()).hexdigest()
# Same passwords produce same hashes → rainbow tables work

# ✅ SECURE — bcrypt
import bcrypt

def hash_password(password):
    salt = bcrypt.gensalt(rounds=12)  # Cost factor 12 (~250ms)
    return bcrypt.hashpw(password.encode(), salt)

def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode(), hashed)

# ✅✅ BEST — Argon2id
from argon2 import PasswordHasher

ph = PasswordHasher(
    time_cost=3,        # Number of iterations
    memory_cost=65536,  # 64 MB memory usage
    parallelism=4,      # 4 threads
    hash_len=32,        # Output hash length
    salt_len=16         # Salt length
)

def hash_password(password):
    return ph.hash(password)

def verify_password(password, hash):
    try:
        return ph.verify(hash, password)
    except Exception:
        return False

# Usage
hashed = hash_password("MyP@ssw0rd!")
# $argon2id$v=19$m=65536,t=3,p=4$c2FsdHNhbHQ$...
is_valid = verify_password("MyP@ssw0rd!", hashed)  # True
```

### OWASP Password Hashing Recommendations

```
Recommended (in order of preference):
1. Argon2id (minimum: 19 MB memory, 2 iterations, 1 parallelism)
2. bcrypt (minimum: cost factor 10, preferably 12+)
3. scrypt (minimum: N=2^14, r=8, p=5)
4. PBKDF2 with HMAC-SHA256 (minimum: 600,000 iterations)

NEVER use:
× MD5, SHA-1, SHA-256/512 alone
× Unsalted hashes
× Custom/homegrown hashing
× Encryption instead of hashing
× Single iteration of any hash
```

---

## 4. Cloud Encryption Services

### AWS Key Management

```
┌────────────────────────────────────────────────────────────────┐
│                AWS ENCRYPTION ARCHITECTURE                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────┐     ┌──────────┐     ┌──────────────────┐      │
│  │   App    │     │ AWS KMS  │     │  AWS CloudHSM    │      │
│  │          │────▶│          │────▶│  (FIPS 140-2 L3) │      │
│  └──────────┘     └──────────┘     └──────────────────┘      │
│                         │                                      │
│              ┌──────────┼──────────┐                          │
│              ▼          ▼          ▼                           │
│         ┌────────┐ ┌────────┐ ┌────────┐                     │
│         │  S3    │ │  EBS   │ │  RDS   │                     │
│         │(SSE-S3)│ │(AES256)│ │(TDE)   │                     │
│         └────────┘ └────────┘ └────────┘                     │
│                                                                │
│  Envelope Encryption:                                         │
│  CMK (Customer Master Key) → encrypts → Data Key             │
│  Data Key → encrypts → Actual Data                            │
│  Only encrypted Data Key stored with data                     │
│  CMK never leaves KMS                                         │
└────────────────────────────────────────────────────────────────┘
```

### Service-Level Encryption Options

| AWS Service | Encryption Option | Key Management |
|------------|-------------------|----------------|
| **S3** | SSE-S3, SSE-KMS, SSE-C | AWS-managed or customer |
| **EBS** | AES-256 | KMS |
| **RDS** | TDE (Oracle/SQL Server) | KMS |
| **DynamoDB** | Default encryption | AWS-owned or customer KMS |
| **Lambda** | Environment variable encryption | KMS |

---

## 5. File-Level Encryption

```python
# File encryption with AES-256-GCM
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

def encrypt_file(input_path, output_path, key):
    """Encrypt a file with AES-256-GCM"""
    nonce = os.urandom(12)
    aead = AESGCM(key)
    
    with open(input_path, 'rb') as f:
        plaintext = f.read()
    
    ciphertext = aead.encrypt(nonce, plaintext, None)
    
    with open(output_path, 'wb') as f:
        f.write(nonce + ciphertext)  # Prepend nonce

def decrypt_file(input_path, output_path, key):
    """Decrypt a file encrypted with AES-256-GCM"""
    with open(input_path, 'rb') as f:
        data = f.read()
    
    nonce = data[:12]
    ciphertext = data[12:]
    aead = AESGCM(key)
    plaintext = aead.decrypt(nonce, ciphertext, None)
    
    with open(output_path, 'wb') as f:
        f.write(plaintext)

# Generate a 256-bit key
key = AESGCM.generate_key(bit_length=256)
encrypt_file('sensitive.pdf', 'sensitive.pdf.enc', key)
```

---

## Summary Table

| Protection | Layer | Protects Against | Key Tool |
|-----------|-------|-----------------|----------|
| **FDE** | Disk | Physical theft | BitLocker, LUKS |
| **TDE** | Database | DB file theft | SQL Server TDE, Oracle TDE |
| **Column encryption** | Field | SQLi data extraction | pgcrypto, application code |
| **App encryption** | Application | All infrastructure access | Fernet, AES-GCM |
| **Password hashing** | Application | Credential database breach | Argon2id, bcrypt |
| **Cloud KMS** | Cloud | Key compromise | AWS KMS, Azure Key Vault |

---

## Revision Questions

1. Compare the four layers of encryption at rest (FDE, TDE, column, application). When would you use each?
2. Why must passwords be hashed rather than encrypted? What is the difference?
3. Compare Argon2id, bcrypt, and PBKDF2. Why is Argon2id considered the strongest?
4. Explain envelope encryption as used by AWS KMS. Why not encrypt data directly with the master key?
5. Write a Python implementation that encrypts sensitive database columns using AES-256-GCM with proper key management.
6. What are the OWASP-recommended minimum parameters for bcrypt and Argon2id?

---

*Previous: [02-data-in-transit-protection.md](02-data-in-transit-protection.md) | Next: [04-weak-cryptographic-algorithms.md](04-weak-cryptographic-algorithms.md)*

---

*[Back to README](../README.md)*
