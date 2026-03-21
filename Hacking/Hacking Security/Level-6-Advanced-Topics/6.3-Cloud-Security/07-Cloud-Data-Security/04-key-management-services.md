# Unit 7: Cloud Data Security — Topic 4: Key Management Services

## Overview

**Key Management Services (KMS)** in cloud environments provide centralized creation, management, rotation, and auditing of cryptographic keys. KMS is the backbone of cloud encryption — controlling who can encrypt/decrypt data, when keys rotate, and maintaining a complete audit trail. Proper KMS configuration is essential for data security and compliance.

---

## 1. KMS Architecture

```
KEY MANAGEMENT HIERARCHY:

  ┌─────────────────────────────────────┐
  │ ROOT OF TRUST                       │
  │ Hardware Security Module (HSM)      │
  │ → FIPS 140-2 Level 2/3             │
  │ → Tamper-resistant hardware         │
  │ → Keys never leave HSM in plaintext │
  └──────────────────┬──────────────────┘
                     │
  ┌──────────────────▼──────────────────┐
  │ MASTER KEY (Root Key / CMK)         │
  │ → Stored in KMS                     │
  │ → Never exported in plaintext       │
  │ → Used to wrap (encrypt) DEKs       │
  │ → Subject to key policies           │
  └──────────────────┬──────────────────┘
                     │
  ┌──────────────────▼──────────────────┐
  │ DATA ENCRYPTION KEY (DEK)           │
  │ → Generated per data object         │
  │ → Encrypts actual data              │
  │ → Wrapped (encrypted) by master key │
  │ → Stored alongside encrypted data   │
  └─────────────────────────────────────┘

ENVELOPE ENCRYPTION FLOW:
  Encryption:
  1. Application requests DEK from KMS
  2. KMS returns plaintext DEK + encrypted DEK
  3. Application encrypts data with plaintext DEK
  4. Application stores encrypted data + encrypted DEK
  5. Application discards plaintext DEK from memory

  Decryption:
  1. Application reads encrypted data + encrypted DEK
  2. Sends encrypted DEK to KMS for decryption
  3. KMS returns plaintext DEK
  4. Application decrypts data
  5. Discards plaintext DEK from memory

  Benefits:
  → Only small DEK sent to KMS (not large data)
  → Reduces KMS API calls and latency
  → Master key never leaves KMS
  → Compromise of one DEK doesn't affect others
```

---

## 2. Cloud KMS Services

```
AWS KMS:

  KEY TYPES:
  → AWS managed keys (aws/service)
  → Customer managed keys (CMK)
  → AWS owned keys (internal use)
  
  KEY SPEC:
  → Symmetric: AES-256-GCM (default)
  → Asymmetric RSA: 2048/3072/4096
  → Asymmetric ECC: P-256/P-384/P-521
  → HMAC: SHA-256/384/512

  # Create CMK
  aws kms create-key --description "Production encryption key"
  
  # Create alias
  aws kms create-alias \
    --alias-name alias/prod-key \
    --target-key-id 1234abcd-...
  
  # Enable rotation
  aws kms enable-key-rotation --key-id 1234abcd-...
  
  # Encrypt data
  aws kms encrypt \
    --key-id alias/prod-key \
    --plaintext fileb://data.txt \
    --output text --query CiphertextBlob
  
  # Decrypt data
  aws kms decrypt \
    --ciphertext-blob fileb://encrypted.txt \
    --output text --query Plaintext

  KEY POLICY:
  {
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456:role/AppRole"},
      "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*"
    }]
  }

AZURE KEY VAULT:

  KEY TYPES:
  → Software-protected (Standard)
  → HSM-protected (Premium) — FIPS 140-2 Level 2
  → Managed HSM — FIPS 140-2 Level 3
  
  # Create key
  az keyvault key create \
    --vault-name myVault \
    --name prod-key \
    --kty RSA --size 2048
  
  # Encrypt
  az keyvault key encrypt \
    --vault-name myVault --name prod-key \
    --algorithm RSA-OAEP --value "base64data"

GCP CLOUD KMS:

  PROTECTION LEVELS:
  → SOFTWARE: Software-backed
  → HSM: FIPS 140-2 Level 3 HSM
  → EXTERNAL: External key manager (EKM)
  → EXTERNAL_VPC: EKM via VPC
  
  KEY RING → KEY → KEY VERSION
  
  # Create key ring
  gcloud kms keyrings create prod-ring \
    --location=us-central1
  
  # Create key
  gcloud kms keys create prod-key \
    --keyring=prod-ring \
    --location=us-central1 \
    --purpose=encryption \
    --rotation-period=90d \
    --next-rotation-time=$(date -u +%Y-%m-%dT%H:%M:%SZ -d "+90 days")
  
  # Encrypt
  gcloud kms encrypt \
    --key=prod-key \
    --keyring=prod-ring \
    --location=us-central1 \
    --plaintext-file=data.txt \
    --ciphertext-file=data.enc
```

---

## 3. Key Rotation

```
KEY ROTATION:

WHY ROTATE:
  → Limit exposure if key is compromised
  → Compliance requirements
  → Cryptographic best practice
  → Reduce amount of data encrypted with single key

AUTOMATIC ROTATION:
  AWS KMS:
  → Annual rotation (365 days)
  → New key material generated
  → Old versions retained for decryption
  → Same key ID, new backing key
  → Enable per CMK
  
  Azure Key Vault:
  → Configurable expiration notifications
  → Rotation via Azure Automation or Event Grid
  → Near-expiry event triggers rotation
  → New version created, old retained
  
  GCP Cloud KMS:
  → Configurable rotation period (min 1 day)
  → New primary version on rotation
  → Old versions available for decryption
  → Can be disabled for specific keys

MANUAL ROTATION:
  → Create new key version
  → Update applications to use new version
  → Decommission old version after migration
  → Used when automatic rotation not suitable

ROTATION BEST PRACTICES:
  [ ] Enable automatic rotation for symmetric keys
  [ ] Rotation period: 90-365 days
  [ ] Test rotation process in non-production
  [ ] Monitor rotation failures
  [ ] Plan for asymmetric key rotation (more complex)
  [ ] Document rotation procedures
  [ ] Audit rotation compliance
```

---

## 4. Key Security

```
PROTECTING KMS KEYS:

ACCESS CONTROL:
  → Separate key admin from key user roles
  → Principle of least privilege
  → Audit all key operations
  → Restrict key deletion

  Key Admin: Create, rotate, enable/disable, delete
  Key User: Encrypt, decrypt, generate data keys
  → NEVER give same identity both roles

KEY POLICIES:
  Restrict who can:
  → Use key for encryption/decryption
  → Manage key (rotate, disable, delete)
  → Grant access to others
  → View key metadata

DELETION PROTECTION:
  AWS:
  → Scheduled deletion (7-30 day waiting period)
  → Cannot use key during waiting period
  → Can cancel deletion
  → CloudTrail alerts on schedule-delete

  Azure:
  → Soft delete (recoverable for 7-90 days)
  → Purge protection (mandatory retention)
  → Cannot purge during retention period
  
  GCP:
  → Scheduled destruction (24h minimum)
  → Can restore before destruction
  → Destroyed versions cannot be recovered

MONITORING:
  → Log every key API call
  → Alert on:
    - Key deletion/disable
    - Policy changes
    - Unusual encryption/decryption volume
    - Key access from unexpected identities
    - Cross-account key usage

COMPLIANCE MATRIX:
  Requirement        | Standard | Premium/HSM
  FIPS 140-2 Level 2 | KMS      | HSM
  FIPS 140-2 Level 3 | —        | Dedicated HSM
  PCI DSS            | KMS+CMEK | HSM recommended
  HIPAA              | KMS+CMEK | HSM for BAA
  FedRAMP            | KMS+CMEK | HSM required
```

---

## 5. Assessment

```
KMS SECURITY ASSESSMENT:

CHECKLIST:
  [ ] All encryption uses KMS-managed keys
  [ ] CMEK used for sensitive/regulated data
  [ ] Key rotation enabled and verified
  [ ] Key policies follow least privilege
  [ ] Key admin and user roles separated
  [ ] Deletion protection enabled
  [ ] Key usage audited via cloud logs
  [ ] Alerts configured for key operations
  [ ] HSM used where compliance requires
  [ ] Key inventory documented
  [ ] Cross-account key sharing reviewed

COMMON FINDINGS:
  Finding                         | Risk
  No key rotation                 | High
  Overly permissive key policies  | High
  No deletion protection          | Critical
  Key admin and user same role    | High
  Unused keys not disabled        | Low
  No monitoring on key operations | High
  Default keys for regulated data | Medium
  Keys shared across environments | Medium

CLI VERIFICATION:
  # AWS: Check rotation
  aws kms get-key-rotation-status --key-id KEY_ID
  
  # AWS: Check key policy
  aws kms get-key-policy --key-id KEY_ID --policy-name default
  
  # Azure: Check Key Vault
  az keyvault key list --vault-name myVault
  
  # GCP: Check key rotation
  gcloud kms keys describe KEY \
    --keyring=RING --location=LOC \
    --format="get(rotationPeriod,nextRotationTime)"
```

---

## Summary Table

| Feature | AWS KMS | Azure Key Vault | GCP Cloud KMS |
|---------|---------|----------------|---------------|
| HSM Option | CloudHSM | Premium/Managed HSM | HSM protection level |
| Auto Rotation | Annual | Via automation | Configurable |
| FIPS Level | Level 2 | Level 2/3 | Level 3 (HSM) |
| Pricing | $1/key/mo | Per operation | Per key version |
| External Keys | No | BYOK | EKM |

---

## Revision Questions

1. How does envelope encryption work and why is it used?
2. What are the different key types available in each cloud KMS?
3. Why is key rotation important and how is it implemented?
4. How should key admin and key user roles be separated?
5. What KMS settings should be verified during a security assessment?

---

*Previous: [03-encryption-in-transit.md](03-encryption-in-transit.md) | Next: [05-data-loss-prevention.md](05-data-loss-prevention.md)*

---

*[Back to README](../README.md)*
