# Chapter 2: Provider Configuration

[← Previous: Provider Basics](01-provider-basics.md) | [Back to README](../README.md) | [Next: Multi-Provider →](03-multi-provider.md)

---

## Overview

**Provider configuration** defines how Terraform authenticates and connects to APIs. This chapter covers authentication methods, configuration options, and best practices for securely configuring providers.

---

## 2.1 Provider Block Syntax

```hcl
provider "<NAME>" {
  # Configuration arguments
  argument1 = "value1"
  argument2 = "value2"
}
```

```
┌──────────────────────────────────────────────────────────┐
│          PROVIDER CONFIGURATION FLOW                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Declare ──► required_providers { }                   │
│                  source + version                        │
│                                                           │
│  2. Configure ──► provider "<name>" { }                  │
│                    region, auth, features                │
│                                                           │
│  3. Initialize ──► terraform init                        │
│                     downloads plugin                     │
│                                                           │
│  4. Use ──► resource/data blocks                         │
│              automatically linked to provider            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 AWS Provider Configuration

```hcl
# ─── Basic Configuration ─────────────────────────
provider "aws" {
  region = "us-east-1"
}

# ─── With explicit credentials (NOT recommended) ─
provider "aws" {
  region     = "us-east-1"
  access_key = "AKIAIOSFODNN7EXAMPLE"        # DON'T hardcode!
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxR"  # DON'T hardcode!
}

# ─── With assume role ────────────────────────────
provider "aws" {
  region = "us-east-1"
  assume_role {
    role_arn     = "arn:aws:iam::123456789012:role/TerraformRole"
    session_name = "TerraformSession"
    external_id  = "my-external-id"
  }
}

# ─── With profile ────────────────────────────────
provider "aws" {
  region  = "us-east-1"
  profile = "production"
}

# ─── With default tags ───────────────────────────
provider "aws" {
  region = "us-east-1"

  default_tags {
    tags = {
      Environment = "production"
      ManagedBy   = "terraform"
      Project     = "myapp"
    }
  }
}
```

### AWS Authentication Order

```
┌──────────────────────────────────────────────────────────┐
│       AWS PROVIDER AUTHENTICATION ORDER                   │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  The AWS provider checks credentials in this order:      │
│                                                           │
│  1. Provider block arguments (access_key/secret_key)     │
│  2. Environment variables                                │
│     ├── AWS_ACCESS_KEY_ID                                │
│     ├── AWS_SECRET_ACCESS_KEY                            │
│     └── AWS_SESSION_TOKEN (if using STS)                 │
│  3. Shared credentials file (~/.aws/credentials)         │
│  4. Shared config file (~/.aws/config)                   │
│  5. Container credentials (ECS task role)                │
│  6. Instance metadata (EC2 instance profile)             │
│                                                           │
│  BEST PRACTICE: Use environment variables or IAM roles   │
│  Never hardcode credentials in .tf files!                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.3 Azure Provider Configuration

```hcl
# ─── Basic Configuration ─────────────────────────
provider "azurerm" {
  features {}
}

# ─── With subscription ───────────────────────────
provider "azurerm" {
  features {}
  subscription_id = "00000000-0000-0000-0000-000000000000"
}

# ─── With Service Principal ──────────────────────
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
  client_id       = var.client_id
  client_secret   = var.client_secret
  tenant_id       = var.tenant_id
}

# ─── With Managed Identity ───────────────────────
provider "azurerm" {
  features {}
  use_msi = true
}

# ─── With feature flags ──────────────────────────
provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
    key_vault {
      purge_soft_delete_on_destroy = false
      recover_soft_deleted_key_vaults = true
    }
    virtual_machine {
      delete_os_disk_on_deletion     = true
      graceful_shutdown              = false
    }
  }
}
```

### Azure Authentication Methods

```bash
# Method 1: Azure CLI (dev/local)
az login

# Method 2: Service Principal (CI/CD)
export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
export ARM_CLIENT_SECRET="your-secret"
export ARM_SUBSCRIPTION_ID="00000000-0000-0000-0000-000000000000"
export ARM_TENANT_ID="00000000-0000-0000-0000-000000000000"

# Method 3: Managed Identity (Azure VMs/App Service)
# Set use_msi = true in provider block

# Method 4: OIDC (GitHub Actions, GitLab CI)
export ARM_USE_OIDC=true
```

---

## 2.4 GCP Provider Configuration

```hcl
# ─── Basic Configuration ─────────────────────────
provider "google" {
  project = "my-project-id"
  region  = "us-central1"
  zone    = "us-central1-a"
}

# ─── With credentials file ───────────────────────
provider "google" {
  project     = "my-project-id"
  region      = "us-central1"
  credentials = file("service-account.json")  # Not recommended
}

# ─── With impersonation ──────────────────────────
provider "google" {
  project = "my-project-id"
  region  = "us-central1"

  impersonate_service_account = "terraform@my-project.iam.gserviceaccount.com"
}
```

```bash
# Authentication via environment
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account.json"

# Or via gcloud
gcloud auth application-default login
```

---

## 2.5 Environment Variables for Providers

```bash
# ─── AWS ──────────────────────────────────────────
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG"
export AWS_DEFAULT_REGION="us-east-1"

# ─── Azure ────────────────────────────────────────
export ARM_CLIENT_ID="..."
export ARM_CLIENT_SECRET="..."
export ARM_SUBSCRIPTION_ID="..."
export ARM_TENANT_ID="..."

# ─── GCP ──────────────────────────────────────────
export GOOGLE_PROJECT="my-project"
export GOOGLE_REGION="us-central1"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"

# ─── Generic Terraform variables ──────────────────
export TF_VAR_instance_type="t3.micro"
export TF_VAR_environment="dev"
```

---

## 2.6 Configuration Best Practices

```
┌──────────────────────────────────────────────────────────┐
│       PROVIDER CONFIGURATION BEST PRACTICES               │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. NEVER hardcode credentials                           │
│     ├── Use environment variables                        │
│     ├── Use IAM roles / managed identities               │
│     └── Use vault / secrets manager in CI/CD             │
│                                                           │
│  2. Pin provider versions                                │
│     └── version = "~> 5.0"                               │
│                                                           │
│  3. Use default_tags (AWS)                               │
│     └── Apply consistent tags across all resources       │
│                                                           │
│  4. Set required_version for Terraform                   │
│     └── required_version = ">= 1.5.0"                   │
│                                                           │
│  5. Keep provider config in separate file                │
│     └── providers.tf or versions.tf                      │
│                                                           │
│  6. Use assume_role for cross-account access             │
│     └── Principle of least privilege                     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `no valid credential sources found` | Missing credentials | Set env vars or configure AWS CLI profile |
| `UnauthorizedAccess` | Insufficient permissions | Check IAM policy attached to role/user |
| Azure `features {}` missing | Required by azurerm | Always include `features {}` even if empty |
| `project required` (GCP) | No project set | Set `project` in provider or `GOOGLE_PROJECT` env var |
| Credentials in state file | Sensitive values in outputs | Mark outputs as `sensitive = true` |
| Wrong region used | Region not specified | Always set region explicitly |

---

## Summary Table

| Provider | Auth Best Practice | Key Config |
|----------|-------------------|------------|
| **AWS** | Env vars or IAM roles | `region`, `assume_role`, `default_tags` |
| **Azure** | Managed Identity or SP env vars | `features {}`, `subscription_id` |
| **GCP** | Application Default Credentials | `project`, `region`, `zone` |
| **All** | Never hardcode secrets | `version`, env variables |

---

## Quick Revision Questions

1. **Why should you never hardcode credentials in provider blocks?**
   > They would be visible in version control, state files, and plan outputs. Use environment variables, IAM roles, or managed identities instead.

2. **What is the AWS provider's credential lookup order?**
   > Provider arguments → environment variables → shared credentials file → shared config → container credentials → instance metadata.

3. **What is `default_tags` in the AWS provider?**
   > A block that automatically applies specified tags to all AWS resources created by that provider, reducing repetition.

4. **Why is `features {}` required in the Azure provider?**
   > It's a mandatory block that controls provider behavior features like preventing resource group deletion or soft-delete recovery for key vaults.

5. **What is `assume_role` in the AWS provider?**
   > It allows Terraform to assume an IAM role in another AWS account, enabling cross-account infrastructure management with least-privilege access.

---

[← Previous: Provider Basics](01-provider-basics.md) | [Back to README](../README.md) | [Next: Multi-Provider →](03-multi-provider.md)
