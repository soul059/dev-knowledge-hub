# Chapter 5: Import and Migration

[← Previous: State Commands](04-state-commands.md) | [Back to README](../README.md) | [Next: State Security →](06-state-security.md)

---

## Overview

**Terraform import** brings existing infrastructure under Terraform management by adding resources to state. This is essential for adopting Terraform in environments with pre-existing infrastructure or for recovering from state loss.

---

## 5.1 Why Import?

```
┌──────────────────────────────────────────────────────────┐
│              IMPORT USE CASES                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Brownfield adoption                                   │
│     ├── Existing infrastructure created manually          │
│     └── Want to manage with Terraform going forward       │
│                                                           │
│  2. State recovery                                        │
│     ├── State file lost or corrupted                      │
│     └── Re-import resources to rebuild state              │
│                                                           │
│  3. Resource takeover                                     │
│     ├── Resource created by another tool                  │
│     └── Bring under Terraform management                  │
│                                                           │
│  4. Migration                                             │
│     ├── Moving resources between state files              │
│     └── Reorganizing Terraform projects                   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Import Command (CLI)

```
┌──────────────────────────────────────────────────────────┐
│              CLI IMPORT WORKFLOW                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Step 1: Write the resource block in config               │
│           resource "aws_instance" "web" { }               │
│                    │                                      │
│  Step 2: Run terraform import                             │
│           terraform import aws_instance.web i-abc123      │
│                    │                                      │
│  Step 3: Populate config to match actual state            │
│           Inspect with: terraform state show              │
│                    │                                      │
│  Step 4: Verify with terraform plan                       │
│           Plan should show NO changes                     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### Step-by-Step Example

```hcl
# Step 1: Write empty resource block
resource "aws_instance" "web" {
  # Will be filled in after import
}
```

```bash
# Step 2: Import the existing resource
$ terraform import aws_instance.web i-0abc123def456

aws_instance.web: Importing from ID "i-0abc123def456"...
aws_instance.web: Import prepared!
aws_instance.web: Refreshing state...

Import successful!
```

```bash
# Step 3: View imported attributes
$ terraform state show aws_instance.web
# resource "aws_instance" "web" {
#     ami                    = "ami-0c55b159cbfafe1f0"
#     instance_type          = "t3.micro"
#     subnet_id              = "subnet-abc123"
#     vpc_security_group_ids = ["sg-abc123"]
#     tags = {
#         "Name" = "web-server"
#     }
#     ...
# }
```

```hcl
# Step 3 (continued): Fill in the config to match
resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t3.micro"
  subnet_id              = "subnet-abc123"
  vpc_security_group_ids = ["sg-abc123"]

  tags = {
    Name = "web-server"
  }
}
```

```bash
# Step 4: Verify
$ terraform plan
No changes. Your infrastructure matches the configuration.
```

---

## 5.3 Import Block (Terraform 1.5+)

Starting with Terraform 1.5, you can use the declarative `import` block:

```hcl
# import.tf
import {
  to = aws_instance.web
  id = "i-0abc123def456"
}

import {
  to = aws_s3_bucket.assets
  id = "my-assets-bucket"
}

import {
  to = aws_security_group.web_sg
  id = "sg-abc123"
}
```

```bash
# Plan will show what will be imported
$ terraform plan

# aws_instance.web will be imported
  resource "aws_instance" "web" {
    + ami           = "ami-0c55b159cbfafe1f0"
    + instance_type = "t3.micro"
    ...
  }

Plan: 3 to import, 0 to add, 0 to change, 0 to destroy.

# Apply to perform the import
$ terraform apply
```

### Generate Configuration (Terraform 1.5+)

```bash
# Auto-generate config files for imports
$ terraform plan -generate-config-out=generated_resources.tf

# This creates a .tf file with the resource configs
# Review and adjust the generated code
```

```
┌──────────────────────────────────────────────────────────┐
│        CLI IMPORT vs IMPORT BLOCK COMPARISON              │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  CLI Import (legacy)          Import Block (TF 1.5+)      │
│  ┌────────────────────┐      ┌────────────────────────┐   │
│  │ terraform import    │      │ import {               │   │
│  │   resource.name ID  │      │   to = resource.name   │   │
│  │                     │      │   id = "ID"            │   │
│  │ One at a time       │      │ }                      │   │
│  │ No plan preview     │      │                        │   │
│  │ Not in config       │      │ Batch imports          │   │
│  │ Can't auto-generate │      │ Plan preview           │   │
│  └────────────────────┘      │ In config (reviewable)  │   │
│                               │ Auto-generate config   │   │
│                               └────────────────────────┘   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.4 Import IDs by Resource Type

Different resources use different identifiers for import:

| Resource Type | Import ID Format | Example |
|---------------|-----------------|---------|
| `aws_instance` | Instance ID | `i-0abc123def456` |
| `aws_s3_bucket` | Bucket name | `my-bucket-name` |
| `aws_vpc` | VPC ID | `vpc-abc123` |
| `aws_subnet` | Subnet ID | `subnet-abc123` |
| `aws_security_group` | SG ID | `sg-abc123` |
| `aws_iam_role` | Role name | `my-role-name` |
| `aws_route53_zone` | Zone ID | `Z1234567890` |
| `azurerm_resource_group` | Azure Resource ID | `/subscriptions/.../resourceGroups/rg-name` |
| `google_compute_instance` | `project/zone/name` | `my-project/us-central1-a/my-vm` |

> **Tip**: Check the provider documentation for the "Import" section of each resource.

---

## 5.5 Importing Modules

```bash
# Import into a module resource
$ terraform import module.vpc.aws_vpc.main vpc-abc123
$ terraform import module.vpc.aws_subnet.public[0] subnet-abc123

# Import for_each resources
$ terraform import 'aws_iam_user.users["alice"]' alice
$ terraform import 'aws_iam_user.users["bob"]' bob
```

---

## 5.6 Real-World Migration Strategy

```
┌──────────────────────────────────────────────────────────┐
│       BROWNFIELD MIGRATION STRATEGY                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Phase 1: Discovery                                       │
│  ├── Inventory existing resources                         │
│  ├── Map dependencies between resources                   │
│  ├── Identify resource IDs                                │
│  └── Prioritize what to import first                      │
│                                                           │
│  Phase 2: Configuration                                   │
│  ├── Write Terraform configs for each resource            │
│  ├── Use import blocks (TF 1.5+)                          │
│  ├── Generate configs with -generate-config-out           │
│  └── Organize into logical modules                        │
│                                                           │
│  Phase 3: Import                                          │
│  ├── Import in dependency order (VPC → Subnets → EC2)     │
│  ├── Verify each import with terraform plan               │
│  ├── Fix config mismatches                                │
│  └── Iterate until plan shows no changes                  │
│                                                           │
│  Phase 4: Validation                                      │
│  ├── Full terraform plan — expect "No changes"            │
│  ├── Test with a small non-critical change                │
│  ├── Review and approve                                   │
│  └── Merge to main — Terraform now manages everything     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.7 Moved Blocks (Terraform 1.1+)

For refactoring within config (not importing from outside):

```hcl
# Declare a move in config instead of using CLI
moved {
  from = aws_instance.web
  to   = aws_instance.web_server
}

moved {
  from = aws_instance.app
  to   = module.compute.aws_instance.app
}

# Terraform plan will show moves, not destroy/create
# After successful apply, you can remove the moved blocks
```

```
┌────────────────────────────────────────────────────┐
│          state mv vs moved BLOCK                    │
├────────────────────────────────────────────────────┤
│                                                     │
│  terraform state mv          moved { }              │
│  ├── CLI operation           ├── In config          │
│  ├── Immediate               ├── Applied via plan   │
│  ├── Not tracked in VCS      ├── Tracked in VCS     │
│  ├── Error-prone             ├── Reviewable in PR   │
│  └── Legacy approach         └── Recommended (1.1+) │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Resource already managed" | Resource already in state | Check with `state list`; remove if stale |
| Plan shows changes after import | Config doesn't match actual attributes | Update config to match `state show` output |
| "Cannot import" | Resource type doesn't support import | Check provider docs; not all resources support import |
| Wrong import ID format | Different than expected | Check provider docs for import section |
| Import succeeds but plan wants to recreate | Missing required attributes in config | Fill in all required attributes from state show |
| "Error: Invalid import id" | ID format doesn't match provider expectation | Use the exact format from provider documentation |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| **CLI Import** | `terraform import resource.name ID` |
| **Import Block** | `import { to = resource.name; id = "ID" }` (TF 1.5+) |
| **Generate Config** | `terraform plan -generate-config-out=file.tf` |
| **Moved Block** | `moved { from = old; to = new }` (TF 1.1+) |
| **Import ID** | Resource-specific identifier (check provider docs) |
| **Verification** | `terraform plan` should show no changes |

---

## Quick Revision Questions

1. **What is the purpose of `terraform import`?**
   > To bring existing infrastructure under Terraform management by adding it to the state file.

2. **What are the steps for a CLI import?**
   > 1) Write an empty resource block, 2) `terraform import resource.name ID`, 3) fill config to match state, 4) verify with `terraform plan`.

3. **How does the import block (TF 1.5+) differ from CLI import?**
   > Import blocks are declarative (in config), support batch imports, allow plan preview, and can auto-generate configuration.

4. **What is a `moved` block and when would you use it?**
   > A declarative way to refactor resource addresses (renaming, moving to modules) that is tracked in version control and applied via plan/apply.

5. **How do you find the correct import ID for a resource?**
   > Check the "Import" section in the provider's documentation for that resource type.

6. **How do you auto-generate Terraform config for imported resources?**
   > Use `terraform plan -generate-config-out=generated.tf` with import blocks defined (Terraform 1.5+).

---

[← Previous: State Commands](04-state-commands.md) | [Back to README](../README.md) | [Next: State Security →](06-state-security.md)
