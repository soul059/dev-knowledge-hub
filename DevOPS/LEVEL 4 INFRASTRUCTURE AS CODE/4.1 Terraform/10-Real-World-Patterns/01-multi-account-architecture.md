# Chapter 1: Multi-Account Architecture

[← Previous: Unit 9 - Documentation](../09-Best-Practices/07-documentation.md) | [Back to README](../README.md) | [Next: Multi-Region Deployment →](02-multi-region-deployment.md)

---

## Overview

Multi-account architecture is a **production best practice** for cloud environments. It provides security isolation, blast radius reduction, and clear billing separation. This chapter covers how to manage multi-account infrastructure with Terraform.

---

## 1.1 Why Multi-Account?

```
┌──────────────────────────────────────────────────────────┐
│           MULTI-ACCOUNT BENEFITS                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────┐  ┌─────────────────┐                │
│  │ Security        │  │ Blast Radius    │                │
│  │ Isolation       │  │ Reduction       │                │
│  │                 │  │                 │                │
│  │ Dev can't touch │  │ Misconfig in    │                │
│  │ prod resources  │  │ dev won't       │                │
│  │                 │  │ affect prod     │                │
│  └─────────────────┘  └─────────────────┘                │
│                                                           │
│  ┌─────────────────┐  ┌─────────────────┐                │
│  │ Billing         │  │ Service Limits  │                │
│  │ Separation      │  │ Per Account     │                │
│  │                 │  │                 │                │
│  │ Clear cost      │  │ Each account    │                │
│  │ attribution     │  │ has own quotas  │                │
│  │                 │  │                 │                │
│  └─────────────────┘  └─────────────────┘                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 AWS Account Structure

```
┌──────────────────────────────────────────────────────────┐
│       AWS ORGANIZATION STRUCTURE                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Management Account (root)                                │
│  ├── Organizations, Billing, SSO                          │
│  │                                                        │
│  ├── OU: Security                                         │
│  │   ├── Security Hub Account (GuardDuty, Config)         │
│  │   └── Log Archive Account (CloudTrail, VPC Flow Logs)  │
│  │                                                        │
│  ├── OU: Infrastructure                                   │
│  │   ├── Network Account (Transit Gateway, DNS)           │
│  │   └── Shared Services Account (CI/CD, Artifacts)       │
│  │                                                        │
│  ├── OU: Workloads                                        │
│  │   ├── Dev Account                                      │
│  │   ├── Staging Account                                  │
│  │   └── Prod Account                                     │
│  │                                                        │
│  └── OU: Sandbox                                          │
│      └── Sandbox Account (experimentation)                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── AWS Organizations setup ─────────────────────
resource "aws_organizations_organization" "main" {
  feature_set = "ALL"

  enabled_policy_types = [
    "SERVICE_CONTROL_POLICY",
    "TAG_POLICY"
  ]

  aws_service_access_principals = [
    "sso.amazonaws.com",
    "cloudtrail.amazonaws.com",
    "config.amazonaws.com"
  ]
}

resource "aws_organizations_organizational_unit" "workloads" {
  name      = "Workloads"
  parent_id = aws_organizations_organization.main.roots[0].id
}

resource "aws_organizations_account" "prod" {
  name      = "production"
  email     = "aws-prod@company.com"
  parent_id = aws_organizations_organizational_unit.workloads.id
  role_name = "OrganizationAccessRole"

  lifecycle {
    prevent_destroy = true
    ignore_changes  = [role_name]
  }
}
```

---

## 1.3 Cross-Account Access with Terraform

```hcl
# ─── Provider aliases for each account ──────────
provider "aws" {
  alias  = "management"
  region = "us-east-1"
  # Uses default credentials (management account)
}

provider "aws" {
  alias  = "production"
  region = "us-east-1"

  assume_role {
    role_arn     = "arn:aws:iam::${var.prod_account_id}:role/TerraformRole"
    session_name = "terraform-prod"
    external_id  = var.external_id
  }
}

provider "aws" {
  alias  = "staging"
  region = "us-east-1"

  assume_role {
    role_arn     = "arn:aws:iam::${var.staging_account_id}:role/TerraformRole"
    session_name = "terraform-staging"
  }
}

# ─── Resources in different accounts ────────────
resource "aws_vpc" "prod" {
  provider   = aws.production
  cidr_block = "10.1.0.0/16"
  tags       = { Name = "prod-vpc" }
}

resource "aws_vpc" "staging" {
  provider   = aws.staging
  cidr_block = "10.2.0.0/16"
  tags       = { Name = "staging-vpc" }
}
```

---

## 1.4 Cross-Account IAM Roles

```hcl
# ─── In target account: Trust policy ─────────────
resource "aws_iam_role" "terraform" {
  provider = aws.production
  name     = "TerraformRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        AWS = "arn:aws:iam::${var.management_account_id}:root"
      }
      Action = "sts:AssumeRole"
      Condition = {
        StringEquals = {
          "sts:ExternalId" = var.external_id
        }
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "terraform" {
  provider   = aws.production
  role       = aws_iam_role.terraform.name
  policy_arn = aws_iam_policy.terraform_permissions.arn
}

# ─── In CI/CD: OIDC for initial auth ────────────
resource "aws_iam_role" "github_actions" {
  name = "github-actions-terraform"

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
          "token.actions.githubusercontent.com:sub" = "repo:${var.github_org}/${var.github_repo}:*"
        }
      }
    }]
  })
}

# Permission to assume roles in other accounts
resource "aws_iam_role_policy" "assume_target_roles" {
  role = aws_iam_role.github_actions.name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = "sts:AssumeRole"
      Resource = [
        "arn:aws:iam::${var.prod_account_id}:role/TerraformRole",
        "arn:aws:iam::${var.staging_account_id}:role/TerraformRole"
      ]
    }]
  })
}
```

---

## 1.5 Project Structure for Multi-Account

```
┌──────────────────────────────────────────────────────────┐
│   MULTI-ACCOUNT PROJECT STRUCTURE                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  infrastructure/                                          │
│  ├── accounts/                                            │
│  │   ├── management/        ← Organizations, SSO          │
│  │   │   ├── main.tf                                      │
│  │   │   └── terraform.tf                                 │
│  │   ├── security/          ← GuardDuty, Config           │
│  │   │   ├── main.tf                                      │
│  │   │   └── terraform.tf                                 │
│  │   └── network/           ← Transit GW, DNS             │
│  │       ├── main.tf                                      │
│  │       └── terraform.tf                                 │
│  │                                                        │
│  ├── workloads/                                           │
│  │   ├── dev/                                             │
│  │   │   ├── main.tf                                      │
│  │   │   ├── variables.tf                                 │
│  │   │   └── terraform.tf   ← Backend in dev account      │
│  │   ├── staging/                                         │
│  │   │   └── ...                                          │
│  │   └── prod/                                            │
│  │       └── ...                                          │
│  │                                                        │
│  └── modules/               ← Shared modules              │
│      ├── networking/                                      │
│      ├── compute/                                         │
│      └── database/                                        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── workloads/prod/terraform.tf ─────────────────
terraform {
  backend "s3" {
    bucket         = "prod-terraform-state"
    key            = "workloads/terraform.tfstate"
    region         = "us-east-1"
    role_arn       = "arn:aws:iam::PROD_ACCOUNT_ID:role/TerraformStateRole"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

---

## 1.6 Account Vending (Automated Account Creation)

```hcl
# ─── Automated account creation ──────────────────
variable "accounts" {
  type = map(object({
    email = string
    ou    = string
    tags  = map(string)
  }))

  default = {
    "app-team-dev" = {
      email = "aws-app-dev@company.com"
      ou    = "Workloads"
      tags  = { Team = "app-team", Environment = "dev" }
    }
    "app-team-prod" = {
      email = "aws-app-prod@company.com"
      ou    = "Workloads"
      tags  = { Team = "app-team", Environment = "prod" }
    }
  }
}

resource "aws_organizations_account" "accounts" {
  for_each  = var.accounts
  name      = each.key
  email     = each.value.email
  parent_id = aws_organizations_organizational_unit.ous[each.value.ou].id
  role_name = "OrganizationAccessRole"
  tags      = each.value.tags

  lifecycle {
    prevent_destroy = true
    ignore_changes  = [role_name]
  }
}

# Baseline each account with standard resources
module "account_baseline" {
  source   = "../modules/account-baseline"
  for_each = aws_organizations_account.accounts

  providers = {
    aws = aws.target
  }

  account_id  = each.value.id
  account_name = each.key
  # Creates: IAM roles, CloudTrail, Config, GuardDuty, VPC
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `AccessDenied` assuming role | Trust policy misconfigured | Check Principal, ExternalId, and conditions |
| State file in wrong account | Backend role_arn not set | Add `role_arn` to backend config |
| Provider uses wrong account | Missing `provider` argument | Explicitly set `provider = aws.prod` on resources |
| Account limits hit | Service quotas per account | Request quota increases or redistribute workloads |
| Can't destroy Organizations account | AWS restriction | Close account manually, remove from Terraform state |

---

## Summary Table

| Account Type | Purpose | Terraform State | Access |
|-------------|---------|----------------|--------|
| **Management** | Organizations, billing, SSO | Own S3 bucket | Admin only |
| **Security** | GuardDuty, Config, CloudTrail | Own S3 bucket | Security team |
| **Network** | Transit Gateway, Route 53 | Own S3 bucket | Network team |
| **Shared Services** | CI/CD, artifacts, AMIs | Own S3 bucket | Platform team |
| **Workload (dev)** | Development resources | Own S3 bucket | Dev team |
| **Workload (prod)** | Production resources | Own S3 bucket | Restricted |

---

## Quick Revision Questions

1. **Why use multiple AWS accounts instead of one?**
   > Security isolation (dev can't affect prod), blast radius reduction, clear billing, separate service quotas, and easier compliance boundaries.

2. **How does Terraform access resources in a different account?**
   > Using `assume_role` in the provider block to assume an IAM role in the target account. The role must have a trust policy allowing the source account.

3. **What is account vending?**
   > Automated process for creating and baselining new AWS accounts using Terraform `for_each` over a map of account configurations, including standard IAM roles, logging, and security controls.

4. **Where should Terraform state be stored in a multi-account setup?**
   > Each account should have its own state bucket, or use a centralized state account. Use `role_arn` in the backend config to access cross-account state buckets.

5. **What is an SCP and how does it relate to Terraform?**
   > Service Control Policies (SCPs) are organization-level guardrails that restrict what any IAM principal can do in member accounts. Terraform must work within SCP boundaries.

---

[← Previous: Unit 9 - Documentation](../09-Best-Practices/07-documentation.md) | [Back to README](../README.md) | [Next: Multi-Region Deployment →](02-multi-region-deployment.md)
