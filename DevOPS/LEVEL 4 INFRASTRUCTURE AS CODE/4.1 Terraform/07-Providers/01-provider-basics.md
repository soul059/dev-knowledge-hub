# Chapter 1: Provider Basics

[← Previous: Workspace Alternatives](../06-Workspaces-and-Environments/04-workspace-alternatives.md) | [Back to README](../README.md) | [Next: Provider Configuration →](02-provider-configuration.md)

---

## Overview

**Providers** are plugins that Terraform uses to interact with cloud platforms, SaaS services, and other APIs. Each provider adds resource types and data sources that Terraform can manage. Without providers, Terraform cannot create any infrastructure.

---

## 1.1 What Is a Provider?

```
┌──────────────────────────────────────────────────────────┐
│               TERRAFORM PROVIDER ARCHITECTURE             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Terraform Core          Providers            APIs        │
│  ┌──────────┐    ┌──────────────────┐   ┌──────────────┐ │
│  │          │    │  AWS Provider     │──►│ AWS API      │ │
│  │          │    └──────────────────┘   └──────────────┘ │
│  │          │                                            │
│  │ Terraform│    ┌──────────────────┐   ┌──────────────┐ │
│  │ Core     │───►│  Azure Provider   │──►│ Azure API    │ │
│  │ Engine   │    └──────────────────┘   └──────────────┘ │
│  │          │                                            │
│  │          │    ┌──────────────────┐   ┌──────────────┐ │
│  │          │    │  GCP Provider     │──►│ GCP API      │ │
│  │          │    └──────────────────┘   └──────────────┘ │
│  └──────────┘                                            │
│                                                           │
│  Core: Reads config, builds graph, manages state         │
│  Provider: Translates HCL → API calls, manages lifecycle │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Provider Types

| Type | Description | Examples |
|------|-------------|----------|
| **Official** | Maintained by HashiCorp | AWS, Azure, GCP, Kubernetes |
| **Partner** | Maintained by technology partners | Datadog, MongoDB Atlas, Cloudflare |
| **Community** | Maintained by individuals | Various niche providers |
| **Built-in** | Part of Terraform core | `terraform_remote_state`, `terraform_data` |

---

## 1.3 Declaring Providers

```hcl
# ─── required_providers block ─────────────────────
terraform {
  required_providers {
    # Official HashiCorp provider
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }

    # Partner provider
    datadog = {
      source  = "DataDog/datadog"
      version = "~> 3.0"
    }

    # Community provider
    uptimerobot = {
      source  = "louy/uptimerobot"
      version = "~> 0.5"
    }
  }

  required_version = ">= 1.5.0"
}
```

### Provider Source Addresses

```
┌──────────────────────────────────────────────────────────┐
│          PROVIDER SOURCE FORMAT                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Format: <HOSTNAME>/<NAMESPACE>/<TYPE>                   │
│                                                           │
│  registry.terraform.io/hashicorp/aws                     │
│  ┌────────────────────┐ ┌────────┐ ┌──┐                  │
│  │   Hostname         │ │Namespace│ │Type│               │
│  │   (default:        │ │(org)   │ │    │               │
│  │    registry.       │ │        │ │    │               │
│  │    terraform.io)   │ │        │ │    │               │
│  └────────────────────┘ └────────┘ └──┘                  │
│                                                           │
│  Examples:                                                │
│  hashicorp/aws          → registry.terraform.io default  │
│  hashicorp/azurerm      → Azure provider (official)      │
│  hashicorp/google       → GCP provider (official)        │
│  DataDog/datadog        → Datadog (partner)              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.4 Version Constraints

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"

      # Exact version
      version = "5.31.0"

      # Minimum version
      version = ">= 5.0.0"

      # Pessimistic constraint (>= 5.31.0, < 5.32.0)
      version = "~> 5.31.0"

      # Major version constraint (>= 5.0, < 6.0)
      version = "~> 5.0"

      # Range
      version = ">= 5.0, < 6.0"

      # Exclude specific version
      version = ">= 5.0, != 5.10.0"
    }
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│          VERSION CONSTRAINT OPERATORS                     │
├────────────┬─────────────────────────────────────────────┤
│ Operator   │ Meaning                                     │
├────────────┼─────────────────────────────────────────────┤
│ = 5.31.0   │ Exactly 5.31.0                              │
│ != 5.10.0  │ Not 5.10.0                                  │
│ > 5.0      │ Greater than 5.0                            │
│ >= 5.0     │ Greater than or equal to 5.0                │
│ < 6.0      │ Less than 6.0                               │
│ <= 5.31.0  │ Less than or equal to 5.31.0                │
│ ~> 5.31.0  │ >= 5.31.0, < 5.32.0 (patch only)           │
│ ~> 5.31    │ >= 5.31.0, < 6.0.0 (minor + patch)         │
│ ~> 5.0     │ >= 5.0.0, < 6.0.0 (minor + patch)          │
└────────────┴─────────────────────────────────────────────┘
```

> **Best Practice:** Use `~> 5.0` for major version pinning in production. This allows minor and patch updates but prevents breaking major version changes.

---

## 1.5 Provider Installation

```bash
# terraform init downloads providers
terraform init

# Output:
# Initializing provider plugins...
# - Finding hashicorp/aws versions matching "~> 5.0"...
# - Installing hashicorp/aws v5.31.0...
# - Installed hashicorp/aws v5.31.0 (signed by HashiCorp)
```

### Lock File (.terraform.lock.hcl)

```hcl
# Auto-generated by terraform init
# Pins exact versions and checksums
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:abc123...",
    "zh:def456...",
  ]
}
```

```
┌──────────────────────────────────────────────────────────┐
│             LOCK FILE BEST PRACTICES                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ✓ Commit .terraform.lock.hcl to version control         │
│    └── Ensures all team members use same versions        │
│                                                           │
│  ✗ Do NOT commit .terraform/ directory                   │
│    └── Contains downloaded provider binaries             │
│                                                           │
│  Update lock file:                                       │
│    terraform init -upgrade                               │
│    └── Updates to latest allowed versions                │
│                                                           │
│  Add platform hashes:                                    │
│    terraform providers lock \                            │
│      -platform=linux_amd64 \                             │
│      -platform=darwin_amd64 \                            │
│      -platform=windows_amd64                             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.6 Popular Providers

| Provider | Source | Description |
|----------|--------|-------------|
| **AWS** | `hashicorp/aws` | Amazon Web Services |
| **Azure** | `hashicorp/azurerm` | Microsoft Azure |
| **GCP** | `hashicorp/google` | Google Cloud Platform |
| **Kubernetes** | `hashicorp/kubernetes` | K8s resources |
| **Helm** | `hashicorp/helm` | Helm charts on K8s |
| **Docker** | `kreuzwerker/docker` | Docker containers |
| **GitHub** | `integrations/github` | GitHub repos, teams |
| **Cloudflare** | `cloudflare/cloudflare` | DNS, CDN, security |
| **Datadog** | `DataDog/datadog` | Monitoring dashboards |
| **Random** | `hashicorp/random` | Random values generation |
| **Null** | `hashicorp/null` | Null resources, provisioners |
| **Local** | `hashicorp/local` | Local file management |
| **TLS** | `hashicorp/tls` | TLS cert generation |

---

## 1.7 Provider Documentation

```bash
# Browse provider docs at:
# https://registry.terraform.io/providers/hashicorp/aws/latest/docs

# Each provider page includes:
# ├── Authentication guide
# ├── Provider argument reference
# ├── Resources (with examples)
# │   ├── aws_instance
# │   ├── aws_s3_bucket
# │   └── ...
# └── Data Sources
#     ├── aws_ami
#     ├── aws_vpc
#     └── ...
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `provider not found` | Missing `required_providers` | Add source and version in `terraform` block |
| `version constraints not met` | Conflicting version requirements | Resolve constraints; run `terraform init -upgrade` |
| `checksum mismatch` | Lock file mismatch | Run `terraform providers lock` with target platforms |
| `failed to query available provider packages` | Network/registry issue | Check connectivity; use a provider mirror |
| Slow `terraform init` | Large provider binary download | Use provider plugin cache: `TF_PLUGIN_CACHE_DIR` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Provider** | Plugin connecting Terraform to an API/service |
| **Source** | `<hostname>/<namespace>/<type>` address |
| **Version** | Constraint operators: `=`, `!=`, `>`, `>=`, `<`, `<=`, `~>` |
| **Lock file** | `.terraform.lock.hcl` — pins exact versions, commit to VCS |
| **required_providers** | Declares provider dependencies in `terraform {}` block |
| **Plugin cache** | `TF_PLUGIN_CACHE_DIR` — share downloads across projects |

---

## Quick Revision Questions

1. **What is a Terraform provider?**
   > A plugin that allows Terraform to interact with a specific API or service. Providers define resources and data sources.

2. **What is the source address format for providers?**
   > `<hostname>/<namespace>/<type>`. For official providers, the hostname defaults to `registry.terraform.io`, e.g., `hashicorp/aws`.

3. **What does `~> 5.31.0` mean as a version constraint?**
   > It allows versions `>= 5.31.0` and `< 5.32.0` — only patch-level updates.

4. **Should you commit the `.terraform.lock.hcl` file?**
   > Yes. It ensures all team members and CI/CD use the exact same provider versions and checksums.

5. **How do you update provider versions?**
   > Run `terraform init -upgrade` to download the latest versions within your constraints and update the lock file.

---

[← Previous: Workspace Alternatives](../06-Workspaces-and-Environments/04-workspace-alternatives.md) | [Back to README](../README.md) | [Next: Provider Configuration →](02-provider-configuration.md)
