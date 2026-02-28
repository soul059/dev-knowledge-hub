# Chapter 5: Terraform Import

[← Previous: Moved and Removed Blocks](04-moved-and-removed.md) | [Back to README](../README.md) | [Next: Checks and Validation →](06-checks-and-validation.md)

---

## Overview

**Terraform import** brings existing infrastructure under Terraform management. This is critical for brownfield environments where resources were created manually (via console/CLI) and need to be managed as code going forward.

---

## 5.1 Import Methods

```
┌──────────────────────────────────────────────────────────┐
│          TERRAFORM IMPORT METHODS                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Method 1: CLI Import (any version)                      │
│  terraform import <address> <id>                         │
│  └── Imports into state only; you write the config       │
│                                                           │
│  Method 2: Import Block (TF 1.5+)                        │
│  import {                                                │
│    to = aws_instance.app                                 │
│    id = "i-1234567890abcdef0"                            │
│  }                                                        │
│  └── Declarative; generates config with -generate-config │
│                                                           │
│  Method 3: Config Generation (TF 1.5+)                   │
│  terraform plan -generate-config-out=generated.tf        │
│  └── Auto-generates HCL from imported resources          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 CLI Import (Classic)

### Step-by-Step Process

```
┌──────────────────────────────────────────────────────────┐
│          CLI IMPORT WORKFLOW                               │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Step 1: Write resource block stub                       │
│       resource "aws_instance" "app" { }                  │
│                                                           │
│  Step 2: Run import command                              │
│       terraform import aws_instance.app i-12345          │
│                                                           │
│  Step 3: Run terraform plan                              │
│       See what attributes need to be added               │
│                                                           │
│  Step 4: Fill in resource configuration                  │
│       Match config to actual resource state              │
│                                                           │
│  Step 5: Run terraform plan again                        │
│       Goal: no changes detected                          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```bash
# Step 1: Write empty resource block
cat > import.tf << 'EOF'
resource "aws_instance" "app" {
  # Will be filled after import
}
EOF

# Step 2: Import
terraform import aws_instance.app i-0123456789abcdef0

# Step 3: View current state
terraform state show aws_instance.app

# Step 4: Copy attributes to config
# Edit import.tf to match state output

# Step 5: Verify — should show "No changes"
terraform plan
```

### Import IDs by Resource Type

| Resource | Import ID | Example |
|----------|-----------|---------|
| `aws_instance` | Instance ID | `i-1234567890abcdef0` |
| `aws_s3_bucket` | Bucket name | `my-bucket-name` |
| `aws_security_group` | SG ID | `sg-12345678` |
| `aws_vpc` | VPC ID | `vpc-12345678` |
| `aws_subnet` | Subnet ID | `subnet-12345678` |
| `aws_iam_role` | Role name | `my-role-name` |
| `aws_db_instance` | DB identifier | `my-database` |
| `aws_route53_zone` | Zone ID | `Z1234567890ABC` |
| `azurerm_resource_group` | Resource ID | `/subscriptions/.../resourceGroups/myRG` |
| `google_compute_instance` | `project/zone/name` | `myproj/us-central1-a/myvm` |

---

## 5.3 Import Block (Terraform 1.5+)

```hcl
# ─── Declarative import ──────────────────────────
import {
  to = aws_instance.app
  id = "i-0123456789abcdef0"
}

import {
  to = aws_s3_bucket.data
  id = "my-application-data"
}

import {
  to = aws_security_group.app
  id = "sg-0123456789abcdef0"
}

# Write corresponding resource blocks
resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  # ... fill in to match actual resource
}

resource "aws_s3_bucket" "data" {
  bucket = "my-application-data"
}

resource "aws_security_group" "app" {
  name   = "app-sg"
  vpc_id = "vpc-12345678"
}
```

```bash
# Preview what will be imported
terraform plan

# Execute the import
terraform apply
```

---

## 5.4 Config Generation (Terraform 1.5+)

```hcl
# ─── Step 1: Write import blocks only ────────────
# imports.tf
import {
  to = aws_instance.app
  id = "i-0123456789abcdef0"
}

import {
  to = aws_s3_bucket.data
  id = "my-application-data"
}

import {
  to = aws_vpc.main
  id = "vpc-12345678"
}
```

```bash
# Step 2: Generate configuration
terraform plan -generate-config-out=generated.tf

# This creates generated.tf with all attributes filled in!
# Review and clean up the generated code

# Step 3: Apply the import
terraform apply
```

```hcl
# ─── Example generated.tf output ─────────────────
# __generated__ by Terraform
resource "aws_instance" "app" {
  ami                         = "ami-0c55b159cbfafe1f0"
  instance_type               = "t3.micro"
  key_name                    = "my-key"
  subnet_id                   = "subnet-12345678"
  vpc_security_group_ids      = ["sg-12345678"]
  associate_public_ip_address = true

  tags = {
    Name = "my-app"
  }

  root_block_device {
    volume_size = 20
    volume_type = "gp3"
  }
}
```

> **Tip:** Generated config often includes read-only attributes or defaults. Clean it up to keep only the attributes you want to manage.

---

## 5.5 for_each Import

```hcl
# Import multiple resources with for_each
locals {
  existing_buckets = {
    logs    = "company-logs-bucket"
    data    = "company-data-bucket"
    backups = "company-backups-bucket"
  }
}

import {
  for_each = local.existing_buckets
  to       = aws_s3_bucket.imported[each.key]
  id       = each.value
}

resource "aws_s3_bucket" "imported" {
  for_each = local.existing_buckets
  bucket   = each.value
}
```

---

## 5.6 Brownfield Migration Strategy

```
┌──────────────────────────────────────────────────────────┐
│       BROWNFIELD MIGRATION STRATEGY                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Phase 1: Inventory                                       │
│  ├── List all existing resources                         │
│  ├── Document resource IDs                               │
│  ├── Map dependencies                                    │
│  └── Prioritize by risk/importance                       │
│                                                           │
│  Phase 2: Import Foundation                               │
│  ├── VPCs, Subnets, Route Tables                         │
│  ├── Security Groups                                     │
│  └── IAM Roles and Policies                              │
│                                                           │
│  Phase 3: Import Compute                                  │
│  ├── EC2 Instances, ASGs                                 │
│  ├── Load Balancers                                      │
│  └── ECS/EKS Clusters                                    │
│                                                           │
│  Phase 4: Import Data                                     │
│  ├── RDS Databases                                       │
│  ├── S3 Buckets                                          │
│  └── ElastiCache, DynamoDB                               │
│                                                           │
│  Phase 5: Validate                                        │
│  ├── terraform plan → "No changes"                       │
│  ├── Test in non-prod first                              │
│  └── Gradual rollout to production                       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.7 Import into Modules

```hcl
# Import into a module resource
import {
  to = module.networking.aws_vpc.main
  id = "vpc-12345678"
}

import {
  to = module.networking.aws_subnet.public[0]
  id = "subnet-aaaa1111"
}

import {
  to = module.networking.aws_subnet.public[1]
  id = "subnet-bbbb2222"
}

# CLI equivalent:
# terraform import module.networking.aws_vpc.main vpc-12345678
# terraform import 'module.networking.aws_subnet.public[0]' subnet-aaaa1111
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `resource already managed` | Already in state | Check `terraform state list` |
| Plan shows changes after import | Config doesn't match actual | Run `terraform state show` and match attributes |
| Import ID not found | Wrong format or deleted resource | Check provider docs for ID format |
| Generated config has errors | Read-only attributes included | Remove computed/read-only attributes |
| `import` block not recognized | Terraform < 1.5 | Upgrade or use CLI `terraform import` |

---

## Summary Table

| Method | Min Version | Config Required | Generates Config |
|--------|-------------|----------------|-----------------|
| `terraform import` (CLI) | Any | Yes (manual) | No |
| `import {}` block | 1.5+ | Yes (manual) | No |
| `import {}` + `-generate-config-out` | 1.5+ | No | Yes |
| `import {}` with `for_each` | 1.5+ | Yes | Optional |

---

## Quick Revision Questions

1. **What does `terraform import` do?**
   > It brings an existing resource into Terraform state so Terraform can manage it going forward. It maps a real resource to a resource address in your configuration.

2. **What is the difference between CLI import and import blocks?**
   > CLI import (`terraform import`) is imperative — a one-time command. Import blocks are declarative, tracked in code, reviewable in PRs, and can use `for_each`.

3. **What does `-generate-config-out` do?**
   > It auto-generates HCL resource configuration for import blocks, writing the attributes of the imported resources to a file. The generated code needs cleanup.

4. **What's the goal after importing a resource?**
   > `terraform plan` should show "No changes" — meaning the written configuration exactly matches the real resource state.

5. **In what order should you import brownfield infrastructure?**
   > Foundation first (VPCs, subnets, security groups), then compute (instances, load balancers), then data (databases, storage), validating at each stage.

---

[← Previous: Moved and Removed Blocks](04-moved-and-removed.md) | [Back to README](../README.md) | [Next: Checks and Validation →](06-checks-and-validation.md)
