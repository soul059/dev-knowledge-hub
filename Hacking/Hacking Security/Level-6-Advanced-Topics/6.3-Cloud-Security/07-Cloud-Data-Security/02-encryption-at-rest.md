# Unit 7: Cloud Data Security — Topic 2: Encryption at Rest

## Overview

**Encryption at rest** protects data stored in cloud services by encrypting it when it is written to disk and decrypting it when authorized users or services read it. All major cloud providers offer default encryption, but understanding the different key management options and their security implications is crucial for meeting compliance requirements and protecting sensitive data.

---

## 1. Encryption at Rest Models

```
KEY MANAGEMENT OPTIONS:

  ┌───────────────────────────────────────────────┐
  │   SERVER-SIDE ENCRYPTION (SSE)                │
  │                                               │
  │   SSE with Provider-Managed Keys:             │
  │   → Cloud provider manages everything         │
  │   → Default for most services                 │
  │   → No configuration needed                   │
  │   → Least operational burden                  │
  │   → AWS: SSE-S3 | Azure: PMK | GCP: Google   │
  │                                               │
  │   SSE with Customer-Managed Keys (CMEK):      │
  │   → Customer controls keys via KMS            │
  │   → Customer controls rotation                │
  │   → Can revoke access by disabling key        │
  │   → Audit key usage                           │
  │   → AWS: SSE-KMS | Azure: CMK | GCP: CMEK    │
  │                                               │
  │   SSE with Customer-Supplied Keys (CSEK):     │
  │   → Customer provides key with each request   │
  │   → Cloud never stores the key                │
  │   → Maximum customer control                  │
  │   → Operational complexity                    │
  │   → AWS: SSE-C | Azure: CSK | GCP: CSEK      │
  │                                               │
  │   CLIENT-SIDE ENCRYPTION:                     │
  │   → Encrypt BEFORE sending to cloud           │
  │   → Cloud never sees plaintext                │
  │   → Maximum security                          │
  │   → Application-managed                       │
  │   → Most operational complexity               │
  └───────────────────────────────────────────────┘

SECURITY COMPARISON:
  Model           | Key Control | Cloud Sees Data | Effort
  Provider-managed| Cloud       | Yes (decrypted) | Lowest
  CMEK            | Customer    | Yes (decrypted) | Medium
  CSEK            | Customer    | No (key per req)| High
  Client-side     | Customer    | No (encrypted)  | Highest

WHEN TO USE:
  Provider-managed: Default, non-regulated data
  CMEK: Regulated data, compliance, audit needs
  CSEK: Maximum control, specific compliance
  Client-side: Ultra-sensitive, zero-trust to cloud
```

---

## 2. Cloud Provider Implementation

```
AWS ENCRYPTION AT REST:

  S3:
  → SSE-S3: AES-256, AWS managed (default)
  → SSE-KMS: AWS KMS managed key
  → SSE-C: Customer-provided key per request
  → Default encryption on bucket
  
  # Enable default encryption (KMS)
  aws s3api put-bucket-encryption \
    --bucket my-bucket \
    --server-side-encryption-configuration '{
      "Rules": [{
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "aws:kms",
          "KMSMasterKeyID": "arn:aws:kms:..."
        }
      }]
    }'

  EBS (Elastic Block Store):
  → Encrypted volumes (AES-256)
  → Default encryption setting per region
  → KMS key for each volume
  → Snapshots inherit encryption
  
  # Enable default EBS encryption
  aws ec2 enable-ebs-encryption-by-default

  RDS:
  → Encryption at rest (AES-256)
  → Must enable at creation (cannot encrypt existing)
  → KMS key per instance
  → Automated backups encrypted
  → Read replicas inherit encryption

AZURE ENCRYPTION AT REST:

  Storage Accounts:
  → AES-256 encryption (always on, cannot disable)
  → Platform-managed keys (PMK) — default
  → Customer-managed keys (CMK) via Key Vault
  → Infrastructure encryption (double encryption)

  Azure SQL:
  → Transparent Data Encryption (TDE) — default
  → Service-managed or customer-managed key
  → Always Encrypted (client-side column encryption)
  → Dynamic Data Masking

  Disk Encryption:
  → Azure Disk Encryption (BitLocker/DM-Crypt)
  → Server-Side Encryption (SSE) with PMK/CMK
  → Encryption at host (encrypt temp disks, caches)

GCP ENCRYPTION AT REST:

  → ALL data encrypted by default (AES-256)
  → Multiple encryption layers (DEK → KEK → KMS)
  → CMEK via Cloud KMS
  → CSEK (customer-supplied keys)
  → Confidential Computing (encryption in use)
  
  GCP Encryption Architecture:
  Data → DEK (Data Encryption Key) → encrypted data
  DEK → KEK (Key Encryption Key) → encrypted DEK
  KEK → Google KMS → protected KEK
  
  # Create CMEK key
  gcloud kms keyrings create my-ring --location=global
  gcloud kms keys create my-key \
    --keyring=my-ring --location=global \
    --purpose=encryption

  # Encrypt storage with CMEK
  gsutil kms encryption \
    -k projects/PROJECT/locations/global/keyRings/my-ring/cryptoKeys/my-key \
    gs://my-bucket
```

---

## 3. Key Management Services

```
CLOUD KMS SERVICES:

AWS KMS:
  → Managed key creation and storage
  → Symmetric and asymmetric keys
  → Automatic annual key rotation
  → Key policies for access control
  → CloudTrail logging of key usage
  → Multi-Region keys
  → $1/month per key + $0.03/10K requests
  → CloudHSM for dedicated HSM ($1.20/hour)

AZURE KEY VAULT:
  → Keys, secrets, and certificates
  → Standard (software) and Premium (HSM)
  → FIPS 140-2 Level 2 (Standard) or Level 3 (HSM)
  → Managed HSM for dedicated HSM pools
  → Soft delete and purge protection
  → RBAC or access policies

GCP CLOUD KMS:
  → Software, HSM, and External keys
  → Automatic or manual rotation
  → IAM-based access control
  → Audit logging
  → Import your own keys
  → EKM (External Key Manager) for external keys

KEY ROTATION:
  AWS KMS:
  → Automatic: Annual rotation (enabled per key)
  → Manual: Create new key version
  → Old versions retained for decryption
  
  Azure Key Vault:
  → Manual rotation or automated via automation
  → Key versioning
  → Near-expiry notifications
  
  GCP Cloud KMS:
  → Automatic: Configurable rotation period
  → Manual: Create new key version
  → Old versions for decryption

KEY HIERARCHY:
  ┌─────────────────────────────┐
  │ Master Key (KMS-protected)  │
  │ → Never leaves KMS          │
  │ → Used to wrap DEKs         │
  ├─────────────────────────────┤
  │ Data Encryption Key (DEK)   │
  │ → Encrypts actual data      │
  │ → Wrapped (encrypted) by MK │
  │ → Stored alongside data     │
  ├─────────────────────────────┤
  │ Envelope Encryption         │
  │ → DEK encrypts data         │
  │ → Master Key wraps DEK      │
  │ → Only wrapped DEK stored   │
  │ → Plaintext DEK in memory   │
  └─────────────────────────────┘
```

---

## 4. Encryption Verification

```
VERIFYING ENCRYPTION:

AWS:
  # Check S3 bucket encryption
  aws s3api get-bucket-encryption --bucket my-bucket
  
  # Check EBS volume encryption
  aws ec2 describe-volumes \
    --volume-ids vol-123 \
    --query "Volumes[*].{ID:VolumeId,Encrypted:Encrypted,KMS:KmsKeyId}"
  
  # Check RDS encryption
  aws rds describe-db-instances \
    --query "DBInstances[*].{ID:DBInstanceIdentifier,Encrypted:StorageEncrypted}"
  
  # Check EBS default encryption
  aws ec2 get-ebs-encryption-by-default

AZURE:
  # Check storage encryption
  az storage account show --name mystorageaccount \
    --query "encryption"
  
  # Check disk encryption
  az disk show --name myDisk -g myRG \
    --query "encryption"
  
  # Check SQL TDE
  az sql db tde show --database myDB \
    --server myServer -g myRG

GCP:
  # Check bucket encryption
  gsutil kms encryption gs://my-bucket
  
  # Check disk encryption
  gcloud compute disks describe my-disk \
    --zone=us-central1-a \
    --format="get(diskEncryptionKey)"
  
  # List KMS keys
  gcloud kms keys list \
    --keyring=my-ring --location=global

COMMON FINDINGS:
  Finding                          | Risk
  Unencrypted storage volumes      | Critical
  Provider-managed keys for PCI    | High
  No key rotation enabled          | Medium
  KMS key accessible to all users  | High
  No encryption at rest policy     | High
  Unencrypted database             | Critical
  Snapshot/backup unencrypted      | High
```

---

## 5. Best Practices

```
ENCRYPTION AT REST BEST PRACTICES:

  [ ] Enable encryption on ALL storage services
  [ ] Use CMEK for regulated/sensitive data
  [ ] Enable automatic key rotation
  [ ] Restrict key access to minimum roles
  [ ] Audit key usage via cloud logging
  [ ] Enable default encryption settings
  [ ] Encrypt backups and snapshots
  [ ] Use HSM-backed keys for highest sensitivity
  [ ] Implement key deletion safeguards
  [ ] Document encryption key inventory
  [ ] Test key rotation procedures
  [ ] Plan for crypto-shredding (key deletion = data destruction)

POLICY ENFORCEMENT:
  AWS:
  → SCP: Deny unencrypted resource creation
  → S3 bucket policy: Deny PutObject without encryption
  → Config Rule: Check encryption compliance
  
  Azure:
  → Azure Policy: Require encryption
  → Deny deployment without encryption
  → Compliance dashboard
  
  GCP:
  → Organization Policy: Require CMEK
  → SCC findings for unencrypted resources
  → Custom policy constraints
```

---

## Summary Table

| Model | Key Control | Security | Operational Effort |
|-------|------------|----------|-------------------|
| Provider-managed | Cloud provider | Baseline | None |
| CMEK | Customer (KMS) | Enhanced | Medium |
| CSEK | Customer (per request) | High | High |
| Client-side | Customer (app) | Maximum | Highest |

---

## Revision Questions

1. What are the four encryption at rest models and when should each be used?
2. How does envelope encryption work?
3. What is the difference between KMS and HSM?
4. How can encryption at rest be verified across cloud services?
5. What policies can enforce encryption at rest compliance?

---

*Previous: [01-data-classification.md](01-data-classification.md) | Next: [03-encryption-in-transit.md](03-encryption-in-transit.md)*

---

*[Back to README](../README.md)*
