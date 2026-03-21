# Unit 2: A02 - Cryptographic Failures — Topic 5: Key Management

## Overview

Even the strongest encryption algorithm is worthless if its **keys are improperly generated, stored, distributed, or rotated**. Key management is often the weakest link in cryptographic systems. Hardcoded keys in source code, keys stored alongside encrypted data, lack of key rotation, and poor key generation are among the most critical mistakes. This topic covers the full key lifecycle, hardware security modules, cloud key management services, envelope encryption, and incident response for key compromise.

---

## 1. Key Lifecycle Management

```
┌─────────────────────────────────────────────────────────────────┐
│                     KEY LIFECYCLE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. GENERATION        2. DISTRIBUTION       3. STORAGE          │
│  ┌──────────────┐    ┌──────────────┐     ┌──────────────┐    │
│  │ CSPRNG       │───▶│ Secure       │────▶│ HSM / KMS    │    │
│  │ Sufficient   │    │ channel      │     │ Vault        │    │
│  │ key length   │    │ (TLS, out    │     │ Encrypted    │    │
│  │ (256-bit+)   │    │  of band)    │     │ at rest      │    │
│  └──────────────┘    └──────────────┘     └──────────────┘    │
│                                                                 │
│  4. USE               5. ROTATION          6. DESTRUCTION       │
│  ┌──────────────┐    ┌──────────────┐     ┌──────────────┐    │
│  │ Access       │───▶│ Scheduled    │────▶│ Crypto erase │    │
│  │ controls     │    │ rotation     │     │ Secure wipe  │    │
│  │ Audit log    │    │ Re-encrypt   │     │ All copies   │    │
│  │ Least priv   │    │ old data     │     │ Audit trail  │    │
│  └──────────────┘    └──────────────┘     └──────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Key Generation Best Practices

### Secure Key Generation

```python
import os
import secrets
from cryptography.hazmat.primitives.asymmetric import rsa, ec
from cryptography.hazmat.primitives import hashes

# ✅ Generate symmetric key with CSPRNG
aes_key = os.urandom(32)  # 256-bit key
aes_key = secrets.token_bytes(32)  # Alternative

# ❌ NEVER use these for key generation
import random
bad_key = bytes([random.randint(0, 255) for _ in range(32)])  # NOT cryptographically secure!
bad_key = b"mysecretkey12345"  # Hardcoded key — CRITICAL vulnerability

# ✅ Generate RSA key pair
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=4096  # Minimum 2048, prefer 4096
)
public_key = private_key.public_key()

# ✅ Generate ECDSA key pair
ec_private = ec.generate_private_key(ec.SECP384R1())
ec_public = ec_private.public_key()
```

### Key Size Requirements

| Algorithm | Minimum Key Size | Recommended | Equivalent Strength |
|-----------|-----------------|-------------|-------------------|
| **AES** | 128-bit | 256-bit | — |
| **RSA** | 2048-bit | 4096-bit | AES-128 (2048) |
| **ECDSA** | P-256 (256-bit) | P-384 (384-bit) | AES-128 (P-256) |
| **HMAC** | 256-bit | 512-bit | — |
| **DH** | 2048-bit | 4096-bit | AES-128 (2048) |
| **Ed25519** | 256-bit (fixed) | 256-bit | AES-128 |

---

## 3. Key Storage

### Where NOT to Store Keys

```python
# ❌ CRITICAL: Key hardcoded in source code
API_KEY = "sk-1234567890abcdef"
DB_PASSWORD = "super_secret_password"
ENCRYPTION_KEY = b'\x01\x02\x03...'

# ❌ Key in configuration file (committed to git)
# config.yml:
# encryption_key: "my-secret-key-do-not-share"

# ❌ Key in environment variable (readable by all processes)
# Better than source code but still not ideal
import os
key = os.environ['ENCRYPTION_KEY']

# ❌ Key stored alongside encrypted data
# encrypted_data.json:
# { "key": "abc123", "data": "encrypted_blob_here" }
```

### Where TO Store Keys

| Storage Method | Security Level | Use Case |
|---------------|---------------|----------|
| **HSM (Hardware Security Module)** | Highest | Banking, government, PKI |
| **Cloud KMS (AWS KMS, Azure Key Vault)** | High | Cloud applications |
| **HashiCorp Vault** | High | Multi-cloud, on-premise |
| **OS Keychain/Credential Manager** | Medium | Desktop applications |
| **Encrypted config with master key** | Medium | Small applications |
| **Environment variables** | Low-Medium | Development, containers |

### HashiCorp Vault Integration

```python
import hvac

# Connect to Vault
client = hvac.Client(url='https://vault.example.com:8200')
client.token = 'hvs.your-vault-token'

# Store a secret
client.secrets.kv.v2.create_or_update_secret(
    path='myapp/encryption',
    secret={'key': 'base64_encoded_key_here'}
)

# Retrieve a secret
secret = client.secrets.kv.v2.read_secret_version(path='myapp/encryption')
encryption_key = secret['data']['data']['key']

# Transit secrets engine (encryption as a service)
# Vault encrypts data without exposing the key
plaintext = base64.b64encode(b"sensitive data").decode()
result = client.secrets.transit.encrypt_data(
    name='my-key',
    plaintext=plaintext
)
ciphertext = result['data']['ciphertext']
# vault:v1:encrypted_data_here
```

---

## 4. Key Rotation

### Why Rotate Keys?

```
Reasons for key rotation:
1. Limit exposure if key is compromised (unknowingly)
2. Compliance requirements (PCI DSS: annual rotation)
3. Reduce amount of data encrypted with a single key
4. Employee departure (who had access to the key)
5. Cryptographic best practice
```

### Rotation Strategies

```
Strategy 1: RE-ENCRYPT (most secure, most expensive)
  ┌─────────────────────────────────────────────────┐
  │ 1. Generate new key (Key_v2)                    │
  │ 2. Decrypt all data with old key (Key_v1)       │
  │ 3. Re-encrypt all data with new key (Key_v2)    │
  │ 4. Destroy old key (Key_v1)                     │
  └─────────────────────────────────────────────────┘

Strategy 2: KEY VERSIONING (practical for large datasets)
  ┌─────────────────────────────────────────────────┐
  │ 1. Generate new key (Key_v2)                    │
  │ 2. New data encrypted with Key_v2               │
  │ 3. Old data stays encrypted with Key_v1         │
  │ 4. Store key version with ciphertext            │
  │ 5. On read: use appropriate key version         │
  │ 6. Gradually re-encrypt old data (background)   │
  └─────────────────────────────────────────────────┘

Strategy 3: ENVELOPE ENCRYPTION with rotation
  ┌─────────────────────────────────────────────────┐
  │ 1. Rotate master key only (in KMS)              │
  │ 2. Re-encrypt data encryption keys (DEKs)       │
  │ 3. Actual data remains unchanged                │
  │ 4. Much faster than re-encrypting all data      │
  └─────────────────────────────────────────────────┘
```

---

## 5. Envelope Encryption

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENVELOPE ENCRYPTION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐                                              │
│  │ Master Key   │  Stored in HSM/KMS (never exported)          │
│  │ (KEK)        │                                              │
│  └──────┬───────┘                                              │
│         │ Encrypts                                              │
│         ▼                                                       │
│  ┌──────────────┐                                              │
│  │ Data Key     │  Generated per file/record/session           │
│  │ (DEK)        │  Stored encrypted alongside data             │
│  └──────┬───────┘                                              │
│         │ Encrypts                                              │
│         ▼                                                       │
│  ┌──────────────┐                                              │
│  │ Actual Data  │  Encrypted with DEK using AES-256-GCM       │
│  └──────────────┘                                              │
│                                                                 │
│  Storage Layout:                                               │
│  ┌──────────────────────────────────────────┐                  │
│  │ Encrypted DEK | Nonce | Encrypted Data   │                  │
│  │ (256 bytes)   | (12B) | (variable)       │                  │
│  └──────────────────────────────────────────┘                  │
│                                                                 │
│  Benefits:                                                      │
│  • Master key never touches data directly                      │
│  • Can rotate master key without re-encrypting data            │
│  • Different DEK per record limits exposure                    │
│  • DEK operations are fast (symmetric)                         │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

class EnvelopeEncryption:
    def __init__(self, kms_client, master_key_id):
        self.kms = kms_client
        self.master_key_id = master_key_id
    
    def encrypt(self, plaintext):
        # 1. Generate data encryption key (DEK)
        dek = AESGCM.generate_key(bit_length=256)
        
        # 2. Encrypt DEK with master key (via KMS)
        encrypted_dek = self.kms.encrypt(
            KeyId=self.master_key_id,
            Plaintext=dek
        )['CiphertextBlob']
        
        # 3. Encrypt data with DEK
        nonce = os.urandom(12)
        aead = AESGCM(dek)
        ciphertext = aead.encrypt(nonce, plaintext, None)
        
        # 4. Destroy plaintext DEK from memory
        dek = None
        
        # 5. Return encrypted DEK + nonce + ciphertext
        return encrypted_dek + nonce + ciphertext
    
    def decrypt(self, envelope):
        # 1. Extract components
        encrypted_dek = envelope[:256]
        nonce = envelope[256:268]
        ciphertext = envelope[268:]
        
        # 2. Decrypt DEK with master key (via KMS)
        dek = self.kms.decrypt(
            CiphertextBlob=encrypted_dek
        )['Plaintext']
        
        # 3. Decrypt data with DEK
        aead = AESGCM(dek)
        plaintext = aead.decrypt(nonce, ciphertext, None)
        
        # 4. Destroy DEK from memory
        dek = None
        
        return plaintext
```

---

## 6. Key Compromise Response

```
┌─────────────────────────────────────────────────────────────────┐
│              KEY COMPROMISE INCIDENT RESPONSE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  IMMEDIATE (0-1 hours):                                        │
│  □ Revoke compromised key                                      │
│  □ Generate new replacement key                                │
│  □ Activate incident response team                             │
│  □ Preserve forensic evidence                                  │
│                                                                 │
│  SHORT-TERM (1-24 hours):                                      │
│  □ Re-encrypt all data with new key                            │
│  □ Rotate all related credentials                              │
│  □ Revoke certificates signed with compromised key             │
│  □ Notify affected parties                                     │
│                                                                 │
│  MEDIUM-TERM (1-7 days):                                       │
│  □ Full forensic investigation                                 │
│  □ Assess scope of data exposure                               │
│  □ Regulatory notifications (GDPR: 72 hours)                  │
│  □ Patch vulnerability that led to compromise                  │
│                                                                 │
│  LONG-TERM:                                                    │
│  □ Review key management procedures                            │
│  □ Implement additional controls                               │
│  □ Update incident response playbook                           │
│  □ Conduct lessons-learned review                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Best Practice | Common Mistake |
|--------|--------------|----------------|
| **Generation** | CSPRNG, adequate key size | `random.randint()`, short keys |
| **Storage** | HSM, KMS, Vault | Hardcoded in source code |
| **Distribution** | TLS, out-of-band | Email, plaintext config files |
| **Rotation** | Annual minimum, envelope encryption | Never rotated |
| **Access** | Least privilege, audit logging | Shared among developers |
| **Destruction** | Crypto erase, all copies | Keys left on decommissioned systems |

---

## Revision Questions

1. Describe the six phases of the key lifecycle. What controls should be applied at each phase?
2. Why should keys never be hardcoded in source code? What are the secure alternatives?
3. Explain envelope encryption and its advantages over direct encryption with a master key.
4. How does HashiCorp Vault's Transit secrets engine provide encryption-as-a-service?
5. What is the incident response procedure when an encryption key is compromised?
6. Compare key rotation strategies: re-encryption vs key versioning. When would you use each?

---

*Previous: [04-weak-cryptographic-algorithms.md](04-weak-cryptographic-algorithms.md) | Next: [06-certificate-validation.md](06-certificate-validation.md)*

---

*[Back to README](../README.md)*
