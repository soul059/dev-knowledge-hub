# Chapter 5: Security Practices

[← Previous: CI/CD Pipelines](04-ci-cd-pipelines.md) | [Back to README](../README.md) | [Next: Testing Strategies →](06-testing-strategies.md)

---

## Overview

Security in Terraform spans **secrets management**, **state file protection**, **IAM policies**, and **policy-as-code** enforcement. This chapter covers essential security practices for production infrastructure.

---

## 5.1 Secrets Management

```
┌──────────────────────────────────────────────────────────┐
│       SECRETS MANAGEMENT HIERARCHY                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ NEVER DO: Hardcode secrets in .tf files      │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
│  Best → Worst:                                            │
│                                                           │
│  1. ★ Vault / Secrets Manager (dynamic secrets)           │
│     └─ Secrets generated on demand, auto-rotated          │
│                                                           │
│  2. ★ Cloud provider secrets (SSM, Key Vault)             │
│     └─ Retrieved at plan/apply time via data sources      │
│                                                           │
│  3. ● Environment variables                               │
│     └─ TF_VAR_* in CI/CD, not stored in code              │
│                                                           │
│  4. ● Encrypted .tfvars files (SOPS, git-crypt)           │
│     └─ Encrypted at rest, decrypted in pipeline           │
│                                                           │
│  5. ✗ Plain text .tfvars (gitignored)                     │
│     └─ Risk of accidental commit                          │
│                                                           │
│  6. ✗✗ Hardcoded in .tf files                             │
│     └─ NEVER — secrets visible in state AND code          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── GOOD: Using AWS Secrets Manager ─────────────
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/database/password"
}

resource "aws_db_instance" "main" {
  engine               = "postgres"
  instance_class       = "db.r5.large"
  username             = "admin"
  password             = data.aws_secretsmanager_secret_version.db_password.secret_string
  skip_final_snapshot  = false
}

# ─── GOOD: Using HashiCorp Vault ─────────────────
data "vault_generic_secret" "db" {
  path = "secret/data/production/database"
}

resource "aws_db_instance" "main" {
  password = data.vault_generic_secret.db.data["password"]
}

# ─── GOOD: Using environment variables ───────────
variable "db_password" {
  type      = string
  sensitive = true   # Marks as sensitive in plan output
}
# Set via: export TF_VAR_db_password="..."

# ─── BAD: Hardcoded secret ───────────────────────
resource "aws_db_instance" "main" {
  password = "SuperSecret123!"   # NEVER DO THIS
}
```

---

## 5.2 Sensitive Values

```hcl
# ─── Mark variables as sensitive ─────────────────
variable "api_key" {
  type      = string
  sensitive = true
  # Value hidden in plan output, logs, and CLI
}

variable "tls_private_key" {
  type      = string
  sensitive = true
}

# ─── Sensitive outputs ──────────────────────────
output "database_password" {
  value     = aws_db_instance.main.password
  sensitive = true
}

# ─── Sensitive in locals ────────────────────────
locals {
  db_connection_string = "postgresql://${var.db_user}:${var.db_password}@${aws_db_instance.main.endpoint}/mydb"
}

output "connection_string" {
  value     = local.db_connection_string
  sensitive = true
}
```

```
┌──────────────────────────────────────────────────────────┐
│       sensitive = true BEHAVIOR                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  What it does:                                            │
│  ✓ Hides value in terraform plan output                   │
│  ✓ Hides value in terraform apply output                  │
│  ✓ Propagates through expressions (tainted)               │
│                                                           │
│  What it does NOT do:                                     │
│  ✗ Does NOT encrypt the value in state                    │
│  ✗ Does NOT prevent the value from being in state         │
│  ✗ Does NOT encrypt logs on CI/CD platforms               │
│                                                           │
│  → State file encryption is STILL required                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.3 State File Security

```
┌──────────────────────────────────────────────────────────┐
│           STATE FILE SECURITY                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  State files contain ALL resource attributes              │
│  including passwords, keys, and certificates              │
│                                                           │
│  Security measures:                                       │
│  ┌──────────────────────────────────────────┐            │
│  │ 1. Encrypt at rest (S3 SSE, GCS KMS)     │            │
│  │ 2. Encrypt in transit (TLS)               │            │
│  │ 3. Restrict access (IAM policies)         │            │
│  │ 4. Enable versioning (S3 versioning)      │            │
│  │ 5. Enable logging (access logs)           │            │
│  │ 6. Use state locking (DynamoDB)           │            │
│  │ 7. Never store locally in production      │            │
│  └──────────────────────────────────────────┘            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Secure S3 backend configuration ────────────
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true                    # SSE-S3 encryption
    kms_key_id     = "arn:aws:kms:..."       # Customer managed KMS key
    dynamodb_table = "terraform-state-lock"  # State locking
  }
}

# ─── S3 bucket for state (created separately) ───
resource "aws_s3_bucket" "state" {
  bucket = "company-terraform-state"

  lifecycle {
    prevent_destroy = true
  }
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
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.state.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "state" {
  bucket                  = aws_s3_bucket.state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# ─── IAM policy: Restrict state access ──────────
resource "aws_iam_policy" "state_access" {
  name = "terraform-state-access"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:ListBucket"
        ]
        Resource = [
          aws_s3_bucket.state.arn,
          "${aws_s3_bucket.state.arn}/*"
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:DeleteItem"
        ]
        Resource = aws_dynamodb_table.lock.arn
      }
    ]
  })
}
```

---

## 5.4 Least Privilege IAM

```hcl
# ─── Principle: Minimum permissions needed ───────

# BAD: Admin access for Terraform
resource "aws_iam_role" "terraform" {
  name = "terraform-role"
}

resource "aws_iam_role_policy_attachment" "admin" {
  role       = aws_iam_role.terraform.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess"
  # NEVER use admin in production
}

# GOOD: Scoped permissions
resource "aws_iam_role" "terraform" {
  name = "terraform-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:oidc-provider/token.actions.githubusercontent.com"
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
        }
        StringLike = {
          "token.actions.githubusercontent.com:sub" = "repo:myorg/myrepo:*"
        }
      }
    }]
  })
}

# Separate roles for plan vs apply
resource "aws_iam_role" "terraform_plan" {
  name = "terraform-plan-role"
  # Read-only permissions for plan
}

resource "aws_iam_role" "terraform_apply" {
  name = "terraform-apply-role"
  # Write permissions for apply (more restricted access)
}
```

---

## 5.5 Policy-as-Code

```
┌──────────────────────────────────────────────────────────┐
│           POLICY-AS-CODE TOOLS                            │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │  Sentinel   │  │    OPA      │  │  Checkov    │      │
│  │  (TF Cloud) │  │  (Open)     │  │  (Open)     │      │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤      │
│  │ HCL-like    │  │ Rego lang   │  │ Python/YAML │      │
│  │ Commercial  │  │ Open Source │  │ Open Source  │      │
│  │ TFE/TFC     │  │ Any CI/CD   │  │ Any CI/CD   │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                                                           │
│  Common Policies:                                         │
│  • All S3 buckets must be encrypted                       │
│  • No public security group rules                         │
│  • All resources must have required tags                  │
│  • Instance types restricted to approved list             │
│  • RDS must have multi-AZ in production                   │
│  • No IAM policies with wildcard permissions              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Sentinel Policy Example (TF Cloud) ─────────
# policy: enforce-encryption.sentinel
import "tfplan/v2" as tfplan

s3_buckets = filter tfplan.resource_changes as _, rc {
  rc.type is "aws_s3_bucket" and
  rc.mode is "managed" and
  (rc.change.actions contains "create" or
   rc.change.actions contains "update")
}

encryption_check = rule {
  all s3_buckets as _, bucket {
    bucket.change.after.server_side_encryption_configuration
      is not null
  }
}

main = rule {
  encryption_check
}
```

```rego
# ─── OPA/Rego Policy Example ────────────────────
# policy: require_tags.rego
package terraform.policies

deny[msg] {
  resource := input.resource_changes[_]
  resource.change.actions[_] == "create"

  required_tags := {"Environment", "Project", "ManagedBy"}
  tags := object.get(resource.change.after, "tags", {})

  missing := required_tags - {key | tags[key]}
  count(missing) > 0

  msg := sprintf("Resource %s missing required tags: %v",
    [resource.address, missing])
}
```

---

## 5.6 Provider Authentication Security

```hcl
# ─── OIDC Authentication (recommended for CI/CD) ─
# No long-lived credentials needed

# AWS: GitHub Actions OIDC
provider "aws" {
  region = "us-east-1"
  # Credentials provided via OIDC role assumption
  # configured in the CI/CD pipeline
}

# Azure: Managed Identity or OIDC
provider "azurerm" {
  features {}
  use_oidc = true
  # Credentials from environment:
  # ARM_CLIENT_ID, ARM_TENANT_ID, ARM_SUBSCRIPTION_ID
}

# GCP: Workload Identity Federation
provider "google" {
  project = var.project_id
  region  = var.region
  # Credentials from workload identity
}
```

```
┌──────────────────────────────────────────────────────────┐
│       AUTHENTICATION METHODS RANKED                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  CI/CD:                                                   │
│  1. ★ OIDC / Workload Identity Federation                 │
│  2. ★ IAM roles (for cloud-hosted runners)                │
│  3. ● Short-lived tokens (STS)                            │
│  4. ✗ Long-lived access keys in secrets                   │
│  5. ✗✗ Hardcoded credentials                              │
│                                                           │
│  Local Development:                                       │
│  1. ★ SSO / Identity Center (aws sso login)               │
│  2. ★ Temporary credentials (aws sts)                     │
│  3. ● Named profiles with session tokens                  │
│  4. ✗ Long-lived access keys in ~/.aws/credentials        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Secrets visible in plan output | Variable not marked sensitive | Add `sensitive = true` to variable and output |
| Secrets in state file | All attributes stored in state | Encrypt state, restrict access, use Vault |
| State file publicly accessible | S3 bucket misconfigured | Enable public access block, bucket policy |
| CI/CD credentials expired | Long-lived keys rotated | Switch to OIDC authentication |
| Policy check fails on valid config | Overly strict policy rules | Review and adjust policy conditions |

---

## Summary Table

| Security Area | Best Practice | Implementation |
|--------------|--------------|----------------|
| **Secrets** | Use secrets manager | Vault, AWS SM, SSM Parameter Store |
| **Variables** | Mark sensitive | `sensitive = true` |
| **State** | Encrypt + restrict | S3 SSE + KMS + IAM policies |
| **Auth** | OIDC / short-lived | Workload Identity Federation |
| **IAM** | Least privilege | Scoped policies, separate plan/apply roles |
| **Policy** | Enforce guardrails | Sentinel, OPA, Checkov |
| **Git** | No secrets in code | `.gitignore`, pre-commit hooks, git-secrets |

---

## Quick Revision Questions

1. **What does `sensitive = true` do and NOT do?**
   > It hides values in CLI output (plan/apply) but does NOT encrypt the value in the state file. State encryption must be configured separately.

2. **What is the best way to manage database passwords in Terraform?**
   > Use a secrets manager (AWS Secrets Manager, HashiCorp Vault) and retrieve values via data sources. Never hardcode passwords in `.tf` files.

3. **How should you secure the Terraform state file?**
   > Use remote backend with encryption at rest (S3 + KMS), encryption in transit (TLS), restricted IAM access, versioning, and state locking.

4. **What is OIDC authentication and why is it preferred?**
   > OpenID Connect allows CI/CD platforms to assume cloud roles using short-lived tokens without storing long-lived access keys, reducing credential exposure risk.

5. **What is policy-as-code?**
   > Automated policy frameworks (Sentinel, OPA, Checkov) that evaluate Terraform plans against security and compliance rules before infrastructure changes are applied.

6. **Why have separate IAM roles for plan and apply?**
   > The plan role only needs read access, while the apply role needs write access. This limits exposure if the plan-only pipeline is compromised.

---

[← Previous: CI/CD Pipelines](04-ci-cd-pipelines.md) | [Back to README](../README.md) | [Next: Testing Strategies →](06-testing-strategies.md)
