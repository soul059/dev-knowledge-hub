# Chapter 4: State Commands

[← Previous: State Locking](03-state-locking.md) | [Back to README](../README.md) | [Next: Import and Migration →](05-import-and-migration.md)

---

## Overview

Terraform provides a set of `terraform state` subcommands for advanced state management — listing, inspecting, moving, removing, and renaming resources within the state file. These are essential for refactoring, troubleshooting, and managing state.

---

## 4.1 State Command Overview

```
┌──────────────────────────────────────────────────────────┐
│              TERRAFORM STATE COMMANDS                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  READ OPERATIONS                                          │
│  ├── terraform state list     List all resources          │
│  ├── terraform state show     Show resource details       │
│  └── terraform state pull     Download remote state       │
│                                                           │
│  WRITE OPERATIONS (modify state)                          │
│  ├── terraform state mv       Move/rename resource        │
│  ├── terraform state rm       Remove from state           │
│  ├── terraform state push     Upload state to remote      │
│  └── terraform state replace-provider  Change provider    │
│                                                           │
│  ⚠ Write operations modify state directly                │
│  ⚠ Always backup state before write operations           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 terraform state list

```bash
# List all resources
$ terraform state list
aws_instance.web
aws_instance.api
aws_s3_bucket.assets
aws_security_group.web_sg
module.vpc.aws_vpc.main
module.vpc.aws_subnet.public[0]
module.vpc.aws_subnet.public[1]

# Filter with address pattern
$ terraform state list aws_instance
aws_instance.web
aws_instance.api

# Filter module resources
$ terraform state list module.vpc
module.vpc.aws_vpc.main
module.vpc.aws_subnet.public[0]
module.vpc.aws_subnet.public[1]
```

---

## 4.3 terraform state show

```bash
# Show detailed attributes of a resource
$ terraform state show aws_instance.web

# resource "aws_instance" "web" {
#     ami                          = "ami-0c55b159cbfafe1f0"
#     arn                          = "arn:aws:ec2:us-east-1:123456789:instance/i-abc123"
#     associate_public_ip_address  = true
#     availability_zone            = "us-east-1a"
#     id                           = "i-abc123def456"
#     instance_state               = "running"
#     instance_type                = "t3.micro"
#     public_ip                    = "54.123.45.67"
#     private_ip                   = "10.0.1.50"
#     subnet_id                    = "subnet-abc123"
#     tags                         = {
#         "Name" = "web-server"
#     }
#     vpc_security_group_ids       = [
#         "sg-abc123",
#     ]
# }

# Show indexed resource
$ terraform state show 'aws_instance.workers[0]'

# Show resource in module
$ terraform state show 'module.vpc.aws_subnet.public[0]'
```

---

## 4.4 terraform state mv

```
┌────────────────────────────────────────────────────┐
│              STATE MOVE SCENARIOS                   │
├────────────────────────────────────────────────────┤
│                                                     │
│  1. Rename a resource                               │
│     old_name ──────► new_name                       │
│                                                     │
│  2. Move into a module                              │
│     resource ──────► module.name.resource            │
│                                                     │
│  3. Move between modules                            │
│     module.a.res ──► module.b.res                   │
│                                                     │
│  4. Move to different state file                    │
│     state_a ───────► state_b                        │
│                                                     │
└────────────────────────────────────────────────────┘
```

### Rename a Resource

```bash
# Rename in config: aws_instance.web → aws_instance.web_server
# Then update state to match:

$ terraform state mv aws_instance.web aws_instance.web_server
Move "aws_instance.web" to "aws_instance.web_server"
Successfully moved 1 object(s).

# Without this, Terraform would DESTROY web and CREATE web_server
```

### Move Into a Module

```bash
# Moving a resource into a newly created module
$ terraform state mv aws_vpc.main module.networking.aws_vpc.main
$ terraform state mv aws_subnet.public module.networking.aws_subnet.public
```

### Move with Count/For-Each

```bash
# Move indexed resources
$ terraform state mv 'aws_instance.web[0]' 'aws_instance.web["primary"]'

# Move from count to for_each
$ terraform state mv 'aws_subnet.public[0]' 'aws_subnet.public["us-east-1a"]'
$ terraform state mv 'aws_subnet.public[1]' 'aws_subnet.public["us-east-1b"]'
```

### Move to Another State

```bash
# Move resource to a different state file
$ terraform state mv -state-out=../other-project/terraform.tfstate \
    aws_s3_bucket.shared \
    aws_s3_bucket.shared
```

---

## 4.5 terraform state rm

```bash
# Remove resource from state WITHOUT destroying it
$ terraform state rm aws_instance.legacy
Removed aws_instance.legacy
Successfully removed 1 resource instance(s).

# Remove module from state
$ terraform state rm module.old_module

# Remove indexed resource
$ terraform state rm 'aws_instance.workers[2]'
```

```
┌──────────────────────────────────────────────────────────┐
│                STATE RM BEHAVIOR                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform state rm aws_instance.legacy                   │
│                                                           │
│  ┌───────────┐    ┌──────────┐    ┌───────────────────┐  │
│  │  Config   │    │  State   │    │  Real World       │  │
│  │           │    │  (before)│    │                   │  │
│  │  (empty   │    │  i-abc123│    │  i-abc123 running │  │
│  │   or      │    │          │    │                   │  │
│  │  removed) │    │  (after) │    │  i-abc123 running │  │
│  │           │    │  (empty) │    │  ↑ STILL EXISTS!  │  │
│  └───────────┘    └──────────┘    └───────────────────┘  │
│                                                           │
│  Resource is "forgotten" — NOT destroyed                  │
│  You'll need to manage it manually or re-import           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.6 terraform state pull / push

```bash
# Download remote state to stdout
$ terraform state pull > backup.tfstate

# Inspect with jq
$ terraform state pull | jq '.resources[] | .type + "." + .name'
"aws_instance.web"
"aws_s3_bucket.assets"

# Upload state to remote (DANGEROUS — use with extreme caution)
$ terraform state push backup.tfstate
```

---

## 4.7 terraform state replace-provider

```bash
# When a provider changes its registry address
$ terraform state replace-provider \
    "registry.terraform.io/hashicorp/aws" \
    "registry.terraform.io/custom/aws"

# Common use: migrating from hashicorp/aws to a forked provider
```

---

## 4.8 Common Workflows

### Refactoring: Rename Resource

```hcl
# Step 1: Change the name in config
# BEFORE:
resource "aws_instance" "web" { ... }
# AFTER:
resource "aws_instance" "web_server" { ... }
```

```bash
# Step 2: Update state
terraform state mv aws_instance.web aws_instance.web_server

# Step 3: Verify — plan should show NO changes
terraform plan
# No changes. Your infrastructure matches the configuration.
```

### Refactoring: Extract to Module

```bash
# 1. Create the module and move .tf code
# 2. Update state for each resource
terraform state mv aws_vpc.main module.network.aws_vpc.main
terraform state mv aws_subnet.public module.network.aws_subnet.public
terraform state mv aws_security_group.web module.network.aws_security_group.web

# 3. Verify
terraform plan
# No changes.
```

### Removing Unmanaged Resources

```bash
# Resource was created by Terraform but should now be managed externally
terraform state rm aws_instance.legacy

# Plan will show it wants to create the resource (since it's still in config)
# Remove from config too, then plan shows no changes
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Resource not found in state" | Wrong address or already removed | Use `terraform state list` to find correct address |
| Move shows plan changes | Config doesn't match new address | Update config to use new names/module paths |
| "Cannot move to existing address" | Target already exists in state | Remove target first or use different name |
| State push rejected | Serial number conflict | Use `-force` flag (dangerous) or resolve conflict |
| Quote issues on Windows | Shell escaping differences | Use single quotes or escape brackets: `\"` |

---

## Summary Table

| Command | Purpose | Modifies State? |
|---------|---------|:---------------:|
| `state list` | List all tracked resources | No |
| `state show` | Show resource attributes | No |
| `state pull` | Download state to stdout | No |
| `state mv` | Rename/move within state | **Yes** |
| `state rm` | Remove from state (not infra) | **Yes** |
| `state push` | Upload state to remote | **Yes** |
| `state replace-provider` | Change provider address | **Yes** |

---

## Quick Revision Questions

1. **What does `terraform state mv` do and when would you use it?**
   > It renames or moves a resource within state, used when refactoring code (renaming resources, moving into modules) to prevent destroy/recreate.

2. **What happens to real infrastructure when you `terraform state rm` a resource?**
   > Nothing — the resource continues to exist but is "forgotten" by Terraform. It won't be managed or destroyed.

3. **How do you verify that a state operation was successful?**
   > Run `terraform plan` — it should show "No changes" if the state matches the configuration.

4. **What is the difference between `state pull` and `state push`?**
   > `pull` downloads remote state to stdout (read-only). `push` uploads a local state file to the remote backend (write — use with extreme caution).

5. **How do you move a resource from count-based indexing to for_each?**
   > Use `terraform state mv 'resource[0]' 'resource["key_name"]'` for each instance, mapping numeric indices to string keys.

---

[← Previous: State Locking](03-state-locking.md) | [Back to README](../README.md) | [Next: Import and Migration →](05-import-and-migration.md)
