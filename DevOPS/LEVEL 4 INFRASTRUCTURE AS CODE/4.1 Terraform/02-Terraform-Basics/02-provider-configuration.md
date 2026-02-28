# Chapter 2: Provider Configuration

[← Previous: Installation and Setup](01-installation-and-setup.md) | [Back to README](../README.md) | [Next: HCL Syntax Basics →](03-hcl-syntax-basics.md)

---

## Overview

Providers are **plugins** that let Terraform interact with cloud platforms, SaaS services, and other APIs. Every resource in Terraform belongs to a provider. This chapter covers how to configure, version, and manage providers.

---

## 2.1 What is a Provider?

```
┌──────────────────────────────────────────────────────────────┐
│                    PROVIDER ARCHITECTURE                     │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   ┌──────────────┐        ┌──────────────┐                   │
│   │  Terraform   │        │   Provider   │                   │
│   │  Core        │◀──────▶│   Plugin     │                   │
│   │  (CLI)       │  gRPC  │   (Binary)   │                   │
│   └──────────────┘        └──────┬───────┘                   │
│                                  │                            │
│                           ┌──────┴───────┐                   │
│                           │  Cloud API   │                   │
│                           │  (AWS, Azure,│                   │
│                           │   GCP, etc.) │                   │
│                           └──────────────┘                   │
│                                                               │
│   Provider responsibilities:                                 │
│   • Authenticate with the cloud API                          │
│   • Translate HCL → API calls                               │
│   • Handle CRUD operations on resources                      │
│   • Report resource attributes back to Terraform             │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Provider Configuration Syntax

```hcl
# Basic provider configuration
provider "aws" {
  region = "us-east-1"
}

# Provider with more options
provider "aws" {
  region  = "us-east-1"
  profile = "production"    # AWS CLI profile name

  default_tags {
    tags = {
      ManagedBy   = "terraform"
      Environment = "production"
    }
  }
}
```

### Required Providers Block

```hcl
# versions.tf — Always specify required providers
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"      # Registry source
      version = "~> 5.0"             # Version constraint
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0, < 4.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}
```

### Provider Source Naming

```
  Provider Source Format:
  
  [hostname/]namespace/type
  
  Examples:
  ┌────────────────────────────┬────────────────────────────┐
  │  Source                    │  Meaning                   │
  ├────────────────────────────┼────────────────────────────┤
  │  hashicorp/aws             │  HashiCorp's AWS provider  │
  │  hashicorp/azurerm         │  HashiCorp's Azure provider│
  │  hashicorp/google          │  HashiCorp's GCP provider  │
  │  hashicorp/kubernetes      │  Kubernetes provider       │
  │  integrations/github       │  GitHub provider           │
  │  cloudflare/cloudflare     │  Cloudflare provider       │
  │  mongodb/mongodbatlas      │  MongoDB Atlas provider    │
  └────────────────────────────┴────────────────────────────┘
  
  Default hostname: registry.terraform.io
```

---

## 2.3 Version Constraints

```
┌─────────────────────────────────────────────────────────────┐
│                VERSION CONSTRAINT OPERATORS                 │
├──────────┬──────────────────────────────────────────────────┤
│ Operator │ Meaning                                         │
├──────────┼──────────────────────────────────────────────────┤
│  = 5.0   │ Exactly version 5.0                             │
│  != 5.0  │ Any version except 5.0                          │
│  > 5.0   │ Greater than 5.0                                │
│  >= 5.0  │ Greater than or equal to 5.0                    │
│  < 6.0   │ Less than 6.0                                   │
│  <= 6.0  │ Less than or equal to 6.0                       │
│  ~> 5.0  │ >= 5.0 AND < 6.0 (pessimistic — RECOMMENDED)   │
│  ~> 5.31 │ >= 5.31 AND < 5.32                              │
├──────────┼──────────────────────────────────────────────────┤
│ Combined │ >= 3.0, < 4.0  (multiple constraints)           │
└──────────┴──────────────────────────────────────────────────┘
```

### Best Practice: Pessimistic Constraint (`~>`)

```hcl
# ~> allows only patch/minor updates — prevents breaking changes

version = "~> 5.0"     # Allows 5.0, 5.1, 5.31 ... but NOT 6.0
version = "~> 5.31"    # Allows 5.31, 5.31.1 ... but NOT 5.32
version = "~> 5.31.0"  # Allows 5.31.0, 5.31.1 ... but NOT 5.32.0
```

---

## 2.4 Multi-Cloud Provider Configuration

```hcl
# Using AWS + Azure + GCP in the same project

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
}

provider "google" {
  project = "my-project-id"
  region  = "us-central1"
}

# Now use resources from any provider
resource "aws_s3_bucket" "backup" {
  bucket = "cross-cloud-backup"
}

resource "azurerm_resource_group" "main" {
  name     = "main-rg"
  location = "East US"
}

resource "google_storage_bucket" "archive" {
  name     = "archive-bucket"
  location = "US"
}
```

```
  ┌──────────────────────────────────────────────────────────┐
  │              MULTI-CLOUD ARCHITECTURE                    │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │            ┌──────────────────┐                          │
  │            │   Terraform      │                          │
  │            │   Configuration  │                          │
  │            └────────┬─────────┘                          │
  │                     │                                     │
  │          ┌──────────┼──────────┐                         │
  │          ▼          ▼          ▼                          │
  │   ┌──────────┐┌──────────┐┌──────────┐                  │
  │   │   AWS    ││  Azure   ││   GCP    │                  │
  │   │ Provider ││ Provider ││ Provider │                  │
  │   └────┬─────┘└────┬─────┘└────┬─────┘                  │
  │        ▼           ▼           ▼                          │
  │   ┌──────────┐┌──────────┐┌──────────┐                  │
  │   │ S3 Bucket││ Resource ││ GCS      │                  │
  │   │          ││ Group    ││ Bucket   │                  │
  │   └──────────┘└──────────┘└──────────┘                  │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.5 Provider Aliases (Multiple Regions)

```hcl
# Deploy resources in multiple AWS regions

provider "aws" {
  region = "us-east-1"     # Default provider
}

provider "aws" {
  alias  = "west"          # Named alias
  region = "us-west-2"
}

provider "aws" {
  alias  = "eu"
  region = "eu-west-1"
}

# Use default provider
resource "aws_instance" "east_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

# Use aliased provider
resource "aws_instance" "west_server" {
  provider      = aws.west              # ← Reference alias
  ami           = "ami-0a1b2c3d4e5f6789"
  instance_type = "t3.micro"
}

resource "aws_instance" "eu_server" {
  provider      = aws.eu
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
}
```

---

## 2.6 Popular Providers

```
┌──────────────────────────────────────────────────────────────┐
│              TOP TERRAFORM PROVIDERS                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  CLOUD PLATFORMS                                             │
│  ├── aws        (hashicorp/aws)        — Amazon Web Services│
│  ├── azurerm    (hashicorp/azurerm)    — Microsoft Azure    │
│  ├── google     (hashicorp/google)     — Google Cloud       │
│  ├── oci        (oracle/oci)           — Oracle Cloud       │
│  └── alicloud   (aliyun/alicloud)      — Alibaba Cloud     │
│                                                               │
│  INFRASTRUCTURE                                              │
│  ├── kubernetes (hashicorp/kubernetes) — K8s resources      │
│  ├── helm       (hashicorp/helm)       — Helm charts       │
│  ├── docker     (kreuzwerker/docker)   — Docker containers  │
│  └── vsphere    (hashicorp/vsphere)    — VMware vSphere    │
│                                                               │
│  SERVICES                                                    │
│  ├── github     (integrations/github)  — GitHub repos/teams │
│  ├── cloudflare (cloudflare/cloudflare)— DNS, CDN          │
│  ├── datadog    (DataDog/datadog)      — Monitoring        │
│  └── pagerduty  (PagerDuty/pagerduty) — Incident mgmt     │
│                                                               │
│  DATABASE                                                    │
│  ├── mongodbatlas (mongodb/mongodbatlas)— MongoDB Atlas     │
│  └── mysql      (petoju/mysql)         — MySQL databases    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.7 Provider Authentication Best Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │         AUTHENTICATION SECURITY HIERARCHY               │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │   BEST ──────────────────────────────────── WORST        │
  │                                                           │
  │   ┌────────────┐  ┌────────────┐  ┌────────────┐        │
  │   │  Managed   │  │   Vault    │  │  Env Vars  │        │
  │   │  Identity  │  │  Dynamic   │  │            │        │
  │   │  / IAM Role│  │  Secrets   │  │            │        │
  │   └────────────┘  └────────────┘  └────────────┘        │
  │   No credentials  Short-lived     Requires manual        │
  │   to manage       auto-rotated    management             │
  │                                                           │
  │                   ┌────────────┐  ┌────────────┐        │
  │                   │  Shared    │  │ Hardcoded  │        │
  │                   │  Cred File │  │ in .tf     │        │
  │                   └────────────┘  └────────────┘        │
  │                   Persists on     NEVER DO THIS!        │
  │                   disk                                   │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

```hcl
# ✗ NEVER DO THIS
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"          # EXPOSED IN GIT!
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfi"  # SECURITY BREACH!
  region     = "us-east-1"
}

# ✓ DO THIS — Let Terraform find credentials automatically
provider "aws" {
  region = "us-east-1"
  # Credentials from: env vars → shared cred file → instance profile
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Error: Failed to install provider` | Network issue or wrong source | Check internet; verify source in `required_providers` |
| `Error: Incompatible provider version` | Version constraint too strict | Widen constraint or update provider |
| `Error: No valid credential sources` | Auth not configured | Set env vars or run `aws configure` |
| Provider plugin cached from old version | `.terraform/` has stale plugins | Run `terraform init -upgrade` |
| Multiple providers conflict | Duplicate provider blocks | Use aliases for multiple instances |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Provider** | Plugin that connects Terraform to a cloud API |
| **Source** | Registry path: `namespace/type` (e.g., `hashicorp/aws`) |
| **Version Constraint** | Controls which provider versions are acceptable |
| **~> (Pessimistic)** | Allows minor/patch updates only — recommended default |
| **Alias** | Named provider instance for multi-region/multi-account |
| **required_providers** | Block in `terraform {}` declaring provider sources and versions |
| **.terraform.lock.hcl** | Lock file pinning exact provider versions |
| **Authentication** | Use IAM roles/managed identity > env vars > shared files |

---

## Quick Revision Questions

1. **What is a Terraform provider, and what does it do?**
   > A provider is a plugin that translates HCL configuration into API calls for a specific cloud or service. It handles authentication, CRUD operations, and attribute management.

2. **What does the `~>` version constraint operator mean?**
   > It's the pessimistic constraint. `~> 5.0` means ≥5.0 and <6.0. It allows minor/patch updates but blocks major version changes.

3. **How do you deploy resources in multiple AWS regions using Terraform?**
   > Use provider aliases: define multiple `provider "aws"` blocks with different `alias` and `region` values, then specify `provider = aws.alias_name` in resources.

4. **Where should cloud credentials be stored? Where should they NEVER be?**
   > Best: IAM roles/managed identity. Acceptable: environment variables, shared credential files. Never: hardcoded in `.tf` files.

5. **What command refreshes provider plugins to the latest allowed version?**
   > `terraform init -upgrade` — downloads the latest provider version matching your constraints.

6. **What is the `.terraform.lock.hcl` file used for?**
   > It records the exact provider versions and hashes used, ensuring all team members use identical provider versions.

---

[← Previous: Installation and Setup](01-installation-and-setup.md) | [Back to README](../README.md) | [Next: HCL Syntax Basics →](03-hcl-syntax-basics.md)
