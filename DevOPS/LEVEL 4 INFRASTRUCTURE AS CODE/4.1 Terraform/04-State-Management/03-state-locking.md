# Chapter 3: State Locking

[← Previous: Remote State](02-remote-state.md) | [Back to README](../README.md) | [Next: State Commands →](04-state-commands.md)

---

## Overview

**State locking** prevents concurrent operations on the same state file, protecting against race conditions and state corruption. When one user runs `terraform apply`, others are blocked until the operation completes.

---

## 3.1 Why State Locking Matters

```
┌──────────────────────────────────────────────────────────┐
│           WITHOUT STATE LOCKING (DANGEROUS)               │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Time    User A                 User B                    │
│  ─────────────────────────────────────────────────        │
│  T1      terraform plan         terraform plan            │
│  T2      (reads state)          (reads state)             │
│  T3      terraform apply        terraform apply           │
│  T4      (writes state) ──┐ ┌── (writes state)           │
│  T5                       │ │                             │
│          ⚠ CONFLICT! Both wrote different state ⚠         │
│          → State corruption / resource drift              │
│          → Orphaned resources                             │
│          → Duplicate resources                            │
│                                                           │
├──────────────────────────────────────────────────────────┤
│           WITH STATE LOCKING (SAFE)                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Time    User A                 User B                    │
│  ─────────────────────────────────────────────────        │
│  T1      terraform apply                                  │
│  T2      🔒 LOCK acquired                                │
│  T3      (making changes)       terraform apply           │
│  T4      (still working)        ❌ Error: state locked   │
│  T5      (done)                 "Lock held by User A"     │
│  T6      🔓 LOCK released                                │
│  T7                             terraform apply           │
│  T8                             🔒 LOCK acquired          │
│  T9                             (making changes)          │
│  T10                            🔓 LOCK released          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 How Locking Works by Backend

### AWS S3 + DynamoDB

```
┌──────────────────────────────────────────────────┐
│           S3 + DYNAMODB LOCKING                   │
├──────────────────────────────────────────────────┤
│                                                   │
│  terraform apply                                  │
│      │                                            │
│      ├──1──► DynamoDB: Write lock record           │
│      │       { LockID: "bucket/key",              │
│      │         Info: "user@host",                 │
│      │         Created: "2024-01-15T10:00:00Z" } │
│      │                                            │
│      ├──2──► S3: Read current state               │
│      │                                            │
│      ├──3──► Apply changes to infrastructure      │
│      │                                            │
│      ├──4──► S3: Write updated state              │
│      │                                            │
│      └──5──► DynamoDB: Delete lock record          │
│                                                   │
└──────────────────────────────────────────────────┘
```

```hcl
# DynamoDB table for locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform State Lock Table"
  }
}

# Backend that uses it
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"   # ← Enables locking
    encrypt        = true
  }
}
```

### Azure Blob Storage

```hcl
# Azure uses blob leases for locking (built-in, no extra setup)
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
    # Locking is AUTOMATIC via blob leases
  }
}
```

### GCS

```hcl
# GCS has built-in locking
terraform {
  backend "gcs" {
    bucket = "my-terraform-state"
    prefix = "prod"
    # Locking is AUTOMATIC
  }
}
```

---

## 3.3 Lock Information

When a lock is active, Terraform provides details:

```
$ terraform apply

Error: Error locking state: Error acquiring the state lock: 
ConditionalCheckFailedException: The conditional request failed

Lock Info:
  ID:        a1b2c3d4-e5f6-7890-abcd-ef1234567890
  Path:      my-terraform-state/prod/terraform.tfstate
  Operation: OperationTypeApply
  Who:       alice@macbook-pro
  Version:   1.6.0
  Created:   2024-01-15 10:30:00.000000 UTC
  Info:
```

---

## 3.4 Force Unlocking

```bash
# CAUTION: Only force-unlock if you are CERTAIN 
# no operation is running

# Use the Lock ID from the error message
terraform force-unlock a1b2c3d4-e5f6-7890-abcd-ef1234567890

# You'll be prompted to confirm
Do you really want to force-unlock?
  Terraform will remove the lock on the remote state.
  This will allow local Terraform commands to modify this state,
  even though it may still be in use. Only 'yes' will be accepted.

  Enter a value: yes

Terraform state has been successfully unlocked!
```

```
┌──────────────────────────────────────────────────────┐
│            WHEN TO FORCE-UNLOCK                       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  ✓ Safe to force-unlock when:                        │
│  ├── Process crashed / was killed                     │
│  ├── Network disconnection during apply               │
│  ├── CI/CD pipeline failed/timed out                  │
│  └── You've verified NO apply is running              │
│                                                       │
│  ✗ DO NOT force-unlock when:                         │
│  ├── Another user is actively running apply           │
│  ├── CI/CD pipeline is still in progress              │
│  └── You're not sure what's happening                 │
│                                                       │
│  ⚠ Always investigate before force-unlocking!        │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## 3.5 Disabling Locking

```bash
# Not recommended, but possible for specific operations
terraform plan -lock=false
terraform apply -lock=false

# Set lock timeout (wait for lock instead of failing)
terraform apply -lock-timeout=5m
```

---

## 3.6 Lock Timeout Strategy

```bash
# Default: Fail immediately if locked
terraform apply
# → Error if locked

# Wait up to 5 minutes for lock
terraform apply -lock-timeout=5m
# → Retries until lock available or timeout

# Useful in CI/CD pipelines
terraform apply -lock-timeout=10m -auto-approve
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Error acquiring the state lock" | Another operation is running | Wait for it to finish, or check Lock Info |
| Stale lock after crash | Process terminated without releasing | Verify no operations running, then `force-unlock` |
| DynamoDB "Table not found" | Lock table doesn't exist | Create the DynamoDB table |
| Lock timeout in CI/CD | Multiple pipelines running | Use `-lock-timeout`, serialize pipeline runs |
| "Failed to unlock state" | Permissions or network issue | Check IAM permissions on DynamoDB |
| Lock works locally but not in CI | Different credentials | Ensure CI has DynamoDB permissions |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **State Locking** | Prevents concurrent state modifications |
| **DynamoDB Lock** | AWS S3 backend uses DynamoDB for locking |
| **Blob Lease** | Azure backend uses automatic blob leases |
| **GCS Lock** | GCP backend has built-in locking |
| **Lock Info** | Shows who, when, and what operation holds the lock |
| **Force Unlock** | `terraform force-unlock <LOCK_ID>` — use with caution |
| **Lock Timeout** | `-lock-timeout=5m` — wait instead of failing |
| **Disable Lock** | `-lock=false` — not recommended |

---

## Quick Revision Questions

1. **What problem does state locking solve?**
   > It prevents race conditions when multiple users or processes try to modify the same state simultaneously, avoiding corruption and resource conflicts.

2. **How does the S3 backend implement state locking?**
   > Through a DynamoDB table with a `LockID` hash key. Terraform writes a lock record before operations and deletes it after.

3. **When is it safe to force-unlock state?**
   > Only when you've confirmed no Terraform operation is actively running — e.g., after a crash, network disconnect, or killed process.

4. **What is the `-lock-timeout` flag?**
   > It tells Terraform to wait for a specified duration (e.g., `5m`) for the lock to become available instead of failing immediately.

5. **Does Azure Blob storage require a separate locking resource?**
   > No — Azure uses built-in blob leases for locking. No additional resources are needed.

---

[← Previous: Remote State](02-remote-state.md) | [Back to README](../README.md) | [Next: State Commands →](04-state-commands.md)
