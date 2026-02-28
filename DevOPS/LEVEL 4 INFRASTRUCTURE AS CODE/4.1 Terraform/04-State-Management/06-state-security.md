# Chapter 6: State Security

[← Previous: Import and Migration](05-import-and-migration.md) | [Back to README](../README.md) | [Next: Drift Detection →](07-drift-detection.md)

---

## Overview

The Terraform state file contains **all resource attributes**, including sensitive data such as passwords, API keys, and certificates — stored in **plain text**. Securing state is one of the most critical aspects of Terraform operations.

---

## 6.1 What's in State?

```
┌──────────────────────────────────────────────────────────┐
│         SENSITIVE DATA IN STATE FILE                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform.tfstate contains ALL attributes including:     │
│                                                           │
│  ├── Database passwords                                   │
│  │     "password": "SuperSecret123!"                     │
│  │                                                        │
│  ├── API keys and tokens                                  │
│  │     "access_key": "AKIA..."                           │
│  │                                                        │
│  ├── TLS certificates and private keys                    │
│  │     "private_key_pem": "-----BEGIN RSA..."            │
│  │                                                        │
│  ├── Connection strings                                   │
│  │     "connection_string": "Server=...;Password=..."    │
│  │                                                        │
│  └── OAuth tokens, SSH keys, encryption keys              │
│                                                           │
│  ⚠  Even resources marked "sensitive" in config are      │
│     stored in plain text in state                         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 State Security Layers

```
┌──────────────────────────────────────────────────────────┐
│              STATE SECURITY LAYERS                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Layer 1: Storage Encryption (at rest)                    │
│  ├── S3 SSE-S3, SSE-KMS, SSE-C                           │
│  ├── Azure Storage Service Encryption                     │
│  ├── GCS server-side encryption                           │
│  └── Terraform Cloud built-in encryption                  │
│                                                           │
│  Layer 2: Transport Encryption (in transit)               │
│  ├── TLS/HTTPS for all backend communication              │
│  └── Encrypted connections to cloud APIs                  │
│                                                           │
│  Layer 3: Access Control                                  │
│  ├── IAM policies (AWS), RBAC (Azure), IAM (GCP)         │
│  ├── Bucket policies restricting who can read/write       │
│  ├── Terraform Cloud team permissions                     │
│  └── Principle of least privilege                         │
│                                                           │
│  Layer 4: Version Control Protection                      │
│  ├── .gitignore for *.tfstate files                       │
│  ├── Pre-commit hooks to prevent accidental commits       │
│  └── Repository scanning for secrets                      │
│                                                           │
│  Layer 5: Audit and Monitoring                            │
│  ├── S3 access logging / CloudTrail                       │
│  ├── Azure Monitor / Activity Log                         │
│  ├── GCS audit logs                                       │
│  └── Alert on unauthorized access                         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.3 Encrypting State at Rest

### AWS S3 with KMS

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true                    # Enable encryption
    kms_key_id     = "alias/terraform-state" # KMS key (optional)
    dynamodb_table = "terraform-locks"
  }
}

# KMS key for state encryption
resource "aws_kms_key" "terraform_state" {
  description             = "KMS key for Terraform state encryption"
  deletion_window_in_days = 30
  enable_key_rotation     = true

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "AllowStateAccess"
        Effect = "Allow"
        Principal = {
          AWS = [
            "arn:aws:iam::123456789012:role/terraform-role",
            "arn:aws:iam::123456789012:role/ci-cd-role"
          ]
        }
        Action = [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      }
    ]
  })
}

resource "aws_kms_alias" "terraform_state" {
  name          = "alias/terraform-state"
  target_key_id = aws_kms_key.terraform_state.key_id
}
```

### Azure with Customer-Managed Key

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    # Encryption is enabled by default (AES-256)
    # For CMK, configure at the storage account level
  }
}
```

---

## 6.4 Access Control

### AWS S3 Bucket Policy

```hcl
resource "aws_s3_bucket_policy" "state" {
  bucket = aws_s3_bucket.state.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "EnforceTLSOnly"
        Effect = "Deny"
        Principal = "*"
        Action = "s3:*"
        Resource = [
          aws_s3_bucket.state.arn,
          "${aws_s3_bucket.state.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      },
      {
        Sid    = "RestrictAccess"
        Effect = "Allow"
        Principal = {
          AWS = [
            "arn:aws:iam::123456789012:role/terraform-admin",
            "arn:aws:iam::123456789012:role/terraform-ci"
          ]
        }
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:ListBucket"
        ]
        Resource = [
          aws_s3_bucket.state.arn,
          "${aws_s3_bucket.state.arn}/*"
        ]
      }
    ]
  })
}
```

### IAM Policy for Terraform Users

```hcl
resource "aws_iam_policy" "terraform_state" {
  name = "terraform-state-access"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:ListBucket"
        ]
        Resource = "arn:aws:s3:::my-terraform-state"
      },
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = "arn:aws:s3:::my-terraform-state/*"
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:DeleteItem"
        ]
        Resource = "arn:aws:dynamodb:*:*:table/terraform-locks"
      }
    ]
  })
}
```

---

## 6.5 Preventing State in Version Control

```gitignore
# .gitignore — ESSENTIAL
*.tfstate
*.tfstate.*
*.tfstate.backup
.terraform/
.terraform.lock.hcl
*.tfvars        # May contain secrets
!example.tfvars # Keep examples
```

### Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit
# Prevent accidental commit of state files

if git diff --cached --name-only | grep -qE '\.tfstate'; then
  echo "ERROR: Attempting to commit Terraform state files!"
  echo "These contain sensitive data and should NEVER be committed."
  echo ""
  echo "Add to .gitignore:"
  echo "  *.tfstate"
  echo "  *.tfstate.*"
  exit 1
fi
```

---

## 6.6 Sensitive Variables and State

```hcl
variable "db_password" {
  type      = string
  sensitive = true   # Hides from CLI output and logs
}

resource "aws_db_instance" "main" {
  password = var.db_password
  # ...
}

# ⚠ Even with sensitive = true, the password is stored
# in STATE in plain text!
```

```
┌──────────────────────────────────────────────────────────┐
│       "SENSITIVE" PROTECTION SCOPE                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Where "sensitive" DOES protect:                          │
│  ├── terraform plan output      (shows "sensitive")       │
│  ├── terraform apply output     (shows "sensitive")       │
│  ├── terraform output command   (requires -raw/-json)     │
│  └── CI/CD logs                 (masked)                  │
│                                                           │
│  Where "sensitive" does NOT protect:                      │
│  ├── terraform.tfstate file     (plain text!)             │
│  ├── terraform state show       (plain text!)             │
│  ├── terraform state pull       (plain text!)             │
│  └── State file backups         (plain text!)             │
│                                                           │
│  ⚠ Encrypt state storage to protect sensitive values!    │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.7 Security Checklist

```
┌──────────────────────────────────────────────────────────┐
│          STATE SECURITY CHECKLIST                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  [ ] Remote backend configured (not local state)          │
│  [ ] Encryption at rest enabled (SSE/KMS/CMK)             │
│  [ ] State locking enabled (DynamoDB/Blob lease)          │
│  [ ] Versioning enabled on storage bucket                 │
│  [ ] Access restricted to authorized users/roles only     │
│  [ ] Bucket/container is NOT publicly accessible          │
│  [ ] HTTPS/TLS enforced for transit                       │
│  [ ] .gitignore excludes *.tfstate files                  │
│  [ ] Pre-commit hooks prevent state commits               │
│  [ ] Access logging / audit trails enabled                │
│  [ ] KMS key rotation enabled (if using KMS)              │
│  [ ] Separate state per environment                       │
│  [ ] CI/CD uses temporary credentials (not long-lived)    │
│  [ ] Regular access review and credential rotation        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Secrets visible in state | All values stored as plain text | Enable encryption; restrict state access |
| State file committed to Git | Missing .gitignore rules | Add rules, remove from history with `git filter-branch` |
| "Access Denied" on state ops | Insufficient IAM/RBAC | Grant required S3/DynamoDB/Blob permissions |
| KMS decryption failed | Key policy too restrictive | Update KMS key policy to include the calling role |
| State exposed via public bucket | Bucket policy misconfigured | Enable public access block; review bucket policy |

---

## Summary Table

| Security Measure | Implementation |
|-----------------|----------------|
| **Encryption at Rest** | S3 SSE/KMS, Azure AES-256, GCS encryption |
| **Encryption in Transit** | TLS/HTTPS (default for cloud backends) |
| **Access Control** | IAM policies, bucket policies, RBAC |
| **Version Control** | `.gitignore`, pre-commit hooks |
| **Versioning** | S3/Blob/GCS versioning for recovery |
| **Locking** | DynamoDB, blob leases, built-in |
| **Audit** | CloudTrail, Azure Monitor, GCS audit logs |
| **Least Privilege** | Minimal permissions for state access |

---

## Quick Revision Questions

1. **Why is Terraform state a security concern?**
   > It stores all resource attributes in plain text, including passwords, API keys, certificates, and other sensitive data.

2. **Does marking a variable as `sensitive` protect it in state?**
   > No — `sensitive` only hides values from CLI output and logs. State still stores the value in plain text.

3. **What are the key layers of state security?**
   > Encryption at rest, encryption in transit, access control (IAM/RBAC), version control protection (.gitignore), and audit logging.

4. **How do you prevent state files from being committed to Git?**
   > Add `*.tfstate` and `*.tfstate.*` to `.gitignore`, and use pre-commit hooks to catch accidental additions.

5. **What AWS resources help secure state?**
   > KMS key (encryption), S3 bucket policy (access control), public access block (prevent exposure), S3 versioning (recovery), CloudTrail (audit).

---

[← Previous: Import and Migration](05-import-and-migration.md) | [Back to README](../README.md) | [Next: Drift Detection →](07-drift-detection.md)
