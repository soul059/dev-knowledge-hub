# Chapter 4: Public Registry Modules

[← Previous: Module Inputs and Outputs](03-module-inputs-outputs.md) | [Back to README](../README.md) | [Next: Writing Reusable Modules →](05-writing-reusable-modules.md)

---

## Overview

The **Terraform Registry** (registry.terraform.io) hosts thousands of community and official modules. Using vetted registry modules accelerates development and provides battle-tested infrastructure patterns.

---

## 4.1 Terraform Registry

```
┌──────────────────────────────────────────────────────────┐
│              TERRAFORM REGISTRY                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  registry.terraform.io                                    │
│  │                                                        │
│  ├── Providers        (AWS, Azure, GCP, etc.)             │
│  │                                                        │
│  ├── Modules                                              │
│  │   ├── Official     (by HashiCorp)                      │
│  │   ├── Partner      (by cloud vendors)                  │
│  │   └── Community    (by anyone)                         │
│  │                                                        │
│  └── Policies         (Sentinel policies)                 │
│                                                           │
│  Module naming: <NAMESPACE>/<NAME>/<PROVIDER>             │
│  Example:       terraform-aws-modules/vpc/aws             │
│                                                           │
│  Key signals of quality:                                  │
│  ├── ✓ Verified badge (partner/official)                 │
│  ├── ✓ High download count                              │
│  ├── ✓ Recent updates                                   │
│  ├── ✓ Good documentation                               │
│  └── ✓ Active GitHub repository                         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Using Registry Modules

```hcl
# Basic usage
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "production-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true

  tags = {
    Environment = "prod"
    ManagedBy   = "terraform"
  }
}

# Access outputs
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "private_subnets" {
  value = module.vpc.private_subnets
}
```

---

## 4.3 Popular Registry Modules

### AWS VPC

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.11.0/24", "10.0.12.0/24"]

  enable_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }
}
```

### AWS EKS

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.28"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    default = {
      min_size     = 1
      max_size     = 3
      desired_size = 2
      instance_types = ["m5.large"]
    }
  }
}
```

### AWS Security Group

```hcl
module "web_sg" {
  source  = "terraform-aws-modules/security-group/aws"
  version = "~> 5.0"

  name        = "web-sg"
  description = "Security group for web servers"
  vpc_id      = module.vpc.vpc_id

  ingress_with_cidr_blocks = [
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = "0.0.0.0/0"
      description = "HTTP"
    },
    {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = "0.0.0.0/0"
      description = "HTTPS"
    }
  ]
}
```

### AWS S3 Bucket

```hcl
module "s3_bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "~> 3.0"

  bucket = "my-app-assets"
  acl    = "private"

  versioning = {
    enabled = true
  }

  server_side_encryption_configuration = {
    rule = {
      apply_server_side_encryption_by_default = {
        sse_algorithm = "aws:kms"
      }
    }
  }
}
```

---

## 4.4 Version Constraints

```hcl
# Exact version
version = "5.1.0"

# Pessimistic constraint (recommended)
version = "~> 5.1"        # >= 5.1, < 6.0
version = "~> 5.1.0"      # >= 5.1.0, < 5.2.0

# Range
version = ">= 4.0, < 6.0"

# Greater than or equal
version = ">= 5.0"
```

```
┌──────────────────────────────────────────────────┐
│         VERSION CONSTRAINT STRATEGIES             │
├──────────────────────────────────────────────────┤
│                                                   │
│  Strategy       Constraint     Risk Level         │
│  ─────────────────────────────────────────────    │
│  Pin exact      = "5.1.0"      Lowest risk        │
│  Pin minor      ~> "5.1"       Low risk           │
│  Pin major      ~> "5.0"       Medium risk        │
│  Range          >= 4, < 6      Higher risk        │
│  Unconstrained  (none)         Highest risk ⚠     │
│                                                   │
│  Recommendation: Use ~> for modules               │
│  Pin exact in production for stability            │
│                                                   │
└──────────────────────────────────────────────────┘
```

---

## 4.5 Private Registry

```hcl
# Terraform Cloud/Enterprise private registry
module "vpc" {
  source  = "app.terraform.io/my-org/vpc/aws"
  version = "1.0.0"
}

# Generic private registry
module "vpc" {
  source  = "registry.example.com/modules/vpc/aws"
  version = "1.0.0"
}
```

---

## 4.6 Evaluating Registry Modules

```
┌──────────────────────────────────────────────────────────┐
│         MODULE EVALUATION CHECKLIST                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Before adopting a registry module, check:                │
│                                                           │
│  [ ] Verified/official badge?                             │
│  [ ] Downloads count (popularity)                         │
│  [ ] Last updated (actively maintained?)                  │
│  [ ] Open issues/PRs on GitHub                            │
│  [ ] License compatibility                                │
│  [ ] Provider version requirements                        │
│  [ ] Input/output documentation                           │
│  [ ] Examples provided                                    │
│  [ ] Changelog available                                  │
│  [ ] Security: no hardcoded secrets                       │
│  [ ] Does it follow module structure conventions?         │
│  [ ] Is the scope appropriate (not too broad/narrow)?     │
│                                                           │
│  When in doubt, consider writing your own module          │
│  with only the features you need.                         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Module version not found" | Version doesn't exist | Check registry for available versions |
| "Could not download module" | Network or auth issue | Check internet connection; configure credentials |
| Too many inputs to manage | Complex module | Read docs; start with examples |
| Module does more than needed | Over-featured module | Consider a simpler module or write your own |
| Breaking changes on upgrade | Major version bump | Read changelog; test in dev first |
| "Unsupported attribute" | Module version changed outputs | Check outputs in the specific version's docs |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Terraform Registry** | Public module repository at registry.terraform.io |
| **Source Format** | `namespace/name/provider` |
| **Version Constraint** | `version = "~> 5.0"` |
| **Verified Modules** | Official/partner modules with verified badge |
| **Install** | `terraform init` downloads registry modules |
| **Upgrade** | `terraform init -upgrade` updates modules |
| **Private Registry** | Terraform Cloud or self-hosted registry |

---

## Quick Revision Questions

1. **What is the source format for a registry module?**
   > `<NAMESPACE>/<NAME>/<PROVIDER>`, e.g., `terraform-aws-modules/vpc/aws`.

2. **What does `version = "~> 5.1"` mean?**
   > Accept any version >= 5.1.0 and < 6.0.0 (pessimistic constraint allowing minor/patch updates within major version 5).

3. **How do you update a registry module to a newer version?**
   > Change the version constraint and run `terraform init -upgrade`.

4. **What should you check before adopting a registry module?**
   > Verified badge, download count, last update date, documentation quality, license, open issues, and whether it fits your needs.

5. **Can you use private registry modules?**
   > Yes — Terraform Cloud/Enterprise offers private registries, and you can configure generic private registries.

---

[← Previous: Module Inputs and Outputs](03-module-inputs-outputs.md) | [Back to README](../README.md) | [Next: Writing Reusable Modules →](05-writing-reusable-modules.md)
