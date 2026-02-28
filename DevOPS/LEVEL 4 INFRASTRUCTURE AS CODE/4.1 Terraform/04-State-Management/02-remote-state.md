# Chapter 2: Remote State

[← Previous: State File Basics](01-state-file-basics.md) | [Back to README](../README.md) | [Next: State Locking →](03-state-locking.md)

---

## Overview

**Remote state** stores the Terraform state file in a shared, centralized location instead of on a local filesystem. This is essential for team collaboration, security, and reliability.

---

## 2.1 Why Remote State?

```
┌────────────────────────────────────────────────────────────┐
│              LOCAL vs REMOTE STATE                          │
├────────────────────────┬───────────────────────────────────┤
│     LOCAL STATE        │     REMOTE STATE                  │
├────────────────────────┼───────────────────────────────────┤
│ On developer machine   │ In shared storage (S3, Blob, GCS)│
│ No encryption by default│ Encrypted at rest                │
│ No locking             │ State locking available           │
│ No versioning          │ Versioning enabled                │
│ Single user only       │ Team collaboration                │
│ Risk of data loss      │ Durable & backed up               │
│ No access control      │ IAM/RBAC access control           │
│ Sensitive data on disk │ Secure storage                    │
└────────────────────────┴───────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│              REMOTE STATE ARCHITECTURE                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Developer A ─────┐                                      │
│                    │     ┌─────────────────────┐          │
│  Developer B ─────┼────►│   Remote Backend    │          │
│                    │     │  ┌───────────────┐  │          │
│  CI/CD Pipeline ──┘     │  │ tfstate (enc) │  │          │
│                          │  └───────────────┘  │          │
│                          │  ┌───────────────┐  │          │
│                          │  │  Lock Table   │  │          │
│                          │  └───────────────┘  │          │
│                          └─────────────────────┘          │
│                                                          │
│  Everyone reads/writes the SAME state                    │
│  Locking prevents concurrent modifications               │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 2.2 Backend Configuration

### AWS S3 Backend

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/networking/terraform.tfstate"
    region         = "us-east-1"
    
    # Encryption
    encrypt        = true
    kms_key_id     = "alias/terraform-state"  # Optional KMS key
    
    # State locking via DynamoDB
    dynamodb_table = "terraform-locks"
    
    # Access control
    # Uses AWS credentials from environment/profile
  }
}
```

#### S3 Backend Prerequisites

```hcl
# Bootstrap resources (create manually or with separate TF config)

resource "aws_s3_bucket" "state" {
  bucket = "my-terraform-state"
}

resource "aws_s3_bucket_versioning" "state" {
  bucket = aws_s3_bucket.state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "state" {
  bucket = aws_s3_bucket.state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "state" {
  bucket = aws_s3_bucket.state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Azure Storage Backend

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    
    # Uses Azure CLI credentials or service principal
  }
}
```

### GCS Backend

```hcl
terraform {
  backend "gcs" {
    bucket  = "my-terraform-state"
    prefix  = "prod/networking"
    
    # Uses Application Default Credentials
  }
}
```

### Terraform Cloud Backend

```hcl
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "prod-networking"
    }
  }
}
```

---

## 2.3 State File Organization

```
┌──────────────────────────────────────────────────────────┐
│          RECOMMENDED STATE FILE ORGANIZATION              │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  S3 Bucket: my-terraform-state                            │
│  │                                                        │
│  ├── global/                                              │
│  │   ├── iam/terraform.tfstate                            │
│  │   └── dns/terraform.tfstate                            │
│  │                                                        │
│  ├── prod/                                                │
│  │   ├── networking/terraform.tfstate                     │
│  │   ├── compute/terraform.tfstate                        │
│  │   ├── database/terraform.tfstate                       │
│  │   └── monitoring/terraform.tfstate                     │
│  │                                                        │
│  ├── staging/                                             │
│  │   ├── networking/terraform.tfstate                     │
│  │   ├── compute/terraform.tfstate                        │
│  │   └── database/terraform.tfstate                       │
│  │                                                        │
│  └── dev/                                                 │
│      ├── networking/terraform.tfstate                     │
│      └── compute/terraform.tfstate                        │
│                                                           │
│  Benefits:                                                │
│  ├── Blast radius limited per component                   │
│  ├── Independent state locking                            │
│  ├── Faster plan/apply (fewer resources per state)        │
│  └── Granular access control                              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.4 Migrating State

### Local to Remote

```bash
# 1. Add backend configuration to your .tf files
# 2. Run terraform init — it detects the change
$ terraform init

Initializing the backend...
Do you want to copy existing state to the new backend?
  Enter a value: yes

Successfully configured the backend "s3"!
```

### Remote to Local

```bash
# 1. Remove backend block from .tf files
# 2. Run terraform init
$ terraform init -migrate-state

Terraform has detected you're unconfiguring your previously set "s3" backend.
Do you want to copy existing state to the new backend?
  Enter a value: yes
```

### Between Remote Backends

```bash
# Change backend config in .tf files, then:
$ terraform init -migrate-state

# Terraform will:
# 1. Read state from old backend
# 2. Write state to new backend
# 3. Remove state from old backend (if supported)
```

---

## 2.5 Partial Backend Configuration

```hcl
# main.tf — Partial config (no secrets)
terraform {
  backend "s3" {
    key = "prod/terraform.tfstate"
  }
}
```

```bash
# Supply remaining config at init time
terraform init \
  -backend-config="bucket=my-terraform-state" \
  -backend-config="region=us-east-1" \
  -backend-config="dynamodb_table=terraform-locks" \
  -backend-config="encrypt=true"

# Or use a file
terraform init -backend-config=backend.hcl
```

```hcl
# backend.hcl (NOT committed to version control)
bucket         = "my-terraform-state"
region         = "us-east-1"
dynamodb_table = "terraform-locks"
encrypt        = true
```

---

## 2.6 Accessing Remote State Data

```hcl
# Read state from ANOTHER Terraform project
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "prod/networking/terraform.tfstate"
    region = "us-east-1"
  }
}

# Use outputs from the remote state
resource "aws_instance" "web" {
  subnet_id = data.terraform_remote_state.networking.outputs.public_subnet_id
  vpc_security_group_ids = [
    data.terraform_remote_state.networking.outputs.web_sg_id
  ]
}
```

```
┌──────────────────────────────────────────────────────────┐
│         CROSS-PROJECT STATE REFERENCES                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Networking Project          Compute Project              │
│  ┌─────────────────┐        ┌─────────────────┐          │
│  │ VPC, Subnets,   │        │ EC2 Instances,  │          │
│  │ Security Groups │        │ Load Balancers  │          │
│  │                 │        │                 │          │
│  │ output "vpc_id" │◄───────│ data "remote_   │          │
│  │ output "sub_id" │  reads │   state" "net"  │          │
│  │ output "sg_id"  │        │                 │          │
│  └───────┬─────────┘        └─────────────────┘          │
│          │                                                │
│  ┌───────▼─────────┐                                     │
│  │ networking/      │                                     │
│  │ terraform.tfstate│                                     │
│  └─────────────────┘                                     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.7 Backend Types Comparison

| Backend | Locking | Encryption | Versioning | Free Tier |
|---------|---------|------------|------------|-----------|
| **Local** | No | No | No | Yes |
| **S3** | Via DynamoDB | SSE/KMS | S3 Versioning | Essentially |
| **Azure Blob** | Via Lease | AES-256 | Blob Versioning | Essentially |
| **GCS** | Built-in | AES-256 | GCS Versioning | Essentially |
| **Terraform Cloud** | Built-in | Built-in | Built-in | Up to 500 resources |
| **Consul** | Built-in | TLS | No | Self-hosted |
| **HTTP** | Optional | Depends | Depends | Depends |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Error configuring backend" | Missing credentials or permissions | Check IAM roles, credentials, bucket existence |
| "Backend configuration changed" | Modified backend block | Run `terraform init -reconfigure` or `-migrate-state` |
| "state data in S3 has a different lineage" | Wrong state file or bucket | Verify key path matches the correct project |
| "Failed to save state" | Network issue or permissions | Check connectivity and write permissions |
| State migration hangs | Large state file or slow network | Be patient; check network; consider state push/pull |
| "Backend not initialized" | Ran command before `terraform init` | Always run `terraform init` first |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Remote State** | State stored in shared remote storage |
| **Backend Block** | `terraform { backend "type" { ... } }` |
| **S3 Backend** | AWS S3 bucket + DynamoDB for locking |
| **Azure Backend** | Azure Blob storage with lease locking |
| **State Migration** | `terraform init -migrate-state` |
| **Partial Config** | `-backend-config` for secrets at init |
| **Remote State Data** | `data "terraform_remote_state"` to read other states |
| **Organization** | Separate state per environment and component |

---

## Quick Revision Questions

1. **Why is remote state preferred over local state for teams?**
   > Remote state provides shared access, state locking (prevents concurrent modifications), encryption at rest, versioning for recovery, and access control.

2. **What two AWS resources are needed for an S3 backend with locking?**
   > An S3 bucket (stores state) and a DynamoDB table (provides state locking).

3. **How do you migrate state from local to remote?**
   > Add a `backend` block to your config, then run `terraform init`. Terraform will detect the change and offer to copy existing state.

4. **What is partial backend configuration and why use it?**
   > Supplying some backend settings via `-backend-config` at init time, keeping secrets (keys, credentials) out of committed code.

5. **How can one Terraform project reference another project's state?**
   > Using `data "terraform_remote_state"` to read outputs from another project's state file.

---

[← Previous: State File Basics](01-state-file-basics.md) | [Back to README](../README.md) | [Next: State Locking →](03-state-locking.md)
