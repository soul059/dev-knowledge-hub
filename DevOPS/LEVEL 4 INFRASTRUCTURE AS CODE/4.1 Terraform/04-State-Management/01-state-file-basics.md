# Chapter 1: State File Basics

[← Previous: Unit 3 - Conditionals and Loops](../03-Terraform-Language/07-conditionals-and-loops.md) | [Back to README](../README.md) | [Next: Remote State →](02-remote-state.md)

---

## Overview

The **Terraform state file** is the cornerstone of Terraform's operation. It maps your configuration to real-world resources, tracks metadata, and enables Terraform to determine what changes to make. Understanding state is critical for working with Terraform safely and effectively.

---

## 1.1 What is Terraform State?

```
┌──────────────────────────────────────────────────────────┐
│                   TERRAFORM STATE                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│   Configuration (.tf)     State File          Real World  │
│   ┌───────────────┐    ┌──────────────┐    ┌───────────┐ │
│   │ resource "ec2"│    │ ec2 → i-abc  │    │ i-abc123  │ │
│   │   type=t3.mic │◄──►│ s3  → my-bkt │◄──►│ my-bucket │ │
│   │ resource "s3" │    │ rds → mydb   │    │ mydb-inst │ │
│   │   bucket=my.. │    │              │    │           │ │
│   └───────────────┘    └──────────────┘    └───────────┘ │
│                                                           │
│   "What I want"     "What Terraform       "What actually  │
│                      thinks exists"        exists"         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

State serves several purposes:
- **Mapping** configuration to real infrastructure
- **Tracking** resource metadata (IDs, attributes)
- **Performance** caching to avoid querying every resource
- **Dependency** resolution for destroy ordering

---

## 1.2 State File Structure

```json
{
  "version": 4,
  "terraform_version": "1.6.0",
  "serial": 12,
  "lineage": "a1b2c3d4-e5f6-...",
  "outputs": {
    "instance_ip": {
      "value": "54.123.45.67",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "id": "i-0abc123def456",
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t3.micro",
            "public_ip": "54.123.45.67",
            "tags": {
              "Name": "web-server"
            }
          }
        }
      ]
    }
  ]
}
```

```
┌────────────────────────────────────────────────────┐
│              STATE FILE ANATOMY                     │
├────────────────────────────────────────────────────┤
│                                                     │
│  version ──────── State format version (currently 4)│
│  terraform_version  TF version that wrote this      │
│  serial ───────── Increments on every write         │
│  lineage ──────── Unique ID for this state's history│
│  outputs ──────── Current output values             │
│  resources ────── Array of tracked resources        │
│    ├ mode ──────── managed (resource) or data       │
│    ├ type ──────── Resource type (aws_instance)     │
│    ├ name ──────── Resource name (web)              │
│    ├ provider ──── Provider responsible             │
│    └ instances ─── Actual instance data             │
│       ├ attributes  All resource attributes         │
│       └ schema_version  Provider schema version     │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

## 1.3 State File Location

```
┌────────────────────────────────────────────────────┐
│              DEFAULT STATE LOCATIONS                │
├────────────────────────────────────────────────────┤
│                                                     │
│  Local (default):                                   │
│  project/                                           │
│  ├── main.tf                                        │
│  ├── variables.tf                                   │
│  ├── terraform.tfstate          ◄── Current state   │
│  └── terraform.tfstate.backup   ◄── Previous state  │
│                                                     │
│  Remote (recommended for teams):                    │
│  ┌─────────────────────────────────────────────┐    │
│  │  S3 Bucket / Azure Blob / GCS / TF Cloud   │    │
│  │  ┌─────────────────────────────────────┐    │    │
│  │  │ terraform.tfstate                   │    │    │
│  │  │ (encrypted, versioned, locked)      │    │    │
│  │  └─────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

## 1.4 How Terraform Uses State

```
┌──────────────────────────────────────────────────────┐
│            terraform plan / apply FLOW                │
├──────────────────────────────────────────────────────┤
│                                                       │
│  1. Read Configuration (.tf files)                    │
│     │                                                 │
│  2. Read Current State (terraform.tfstate)            │
│     │                                                 │
│  3. Refresh State (query real infrastructure)         │
│     │    └── Update state with actual values          │
│     │                                                 │
│  4. Compare Desired (config) vs Actual (state)        │
│     │                                                 │
│  5. Generate Execution Plan                           │
│     │    ├── + Create (in config, not in state)       │
│     │    ├── ~ Update (config differs from state)     │
│     │    ├── - Destroy (in state, not in config)      │
│     │    └── (no change)                              │
│     │                                                 │
│  6. Apply Changes (if approved)                       │
│     │                                                 │
│  7. Update State File                                 │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## 1.5 Viewing State

```bash
# List all resources in state
terraform state list

# Output:
# aws_instance.web
# aws_s3_bucket.assets
# aws_security_group.web_sg

# Show details of a specific resource
terraform state show aws_instance.web

# Output:
# resource "aws_instance" "web" {
#     ami                    = "ami-0c55b159cbfafe1f0"
#     instance_type          = "t3.micro"
#     id                     = "i-0abc123def456"
#     public_ip              = "54.123.45.67"
#     ...
# }

# Show full state (JSON)
terraform show -json | jq .

# Pull remote state locally (for inspection)
terraform state pull > state_backup.json
```

---

## 1.6 State and the Plan

```hcl
# Example: You have this configuration
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.small"    # Changed from t3.micro
}
```

```
┌─────────────────────────────────────────────────────┐
│           HOW PLAN DETECTS CHANGES                   │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Config says:        State says:                     │
│  instance_type =     instance_type =                 │
│  "t3.small"          "t3.micro"                      │
│       │                   │                          │
│       └───────┬───────────┘                          │
│               │                                      │
│          DIFFERENT!                                   │
│               │                                      │
│     Plan: ~ Update in-place                          │
│     instance_type: "t3.micro" → "t3.small"           │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 1.7 Sensitive Data in State

> **WARNING**: Terraform state contains ALL attribute values in **plain text**, including passwords, API keys, and secrets.

```
┌──────────────────────────────────────────────────┐
│      ⚠  STATE FILE SECURITY CONCERNS  ⚠         │
├──────────────────────────────────────────────────┤
│                                                   │
│  State file may contain:                          │
│  ├── Database passwords                           │
│  ├── API keys and tokens                          │
│  ├── TLS private keys                             │
│  ├── SSH keys                                     │
│  └── Any attribute marked "sensitive"             │
│                                                   │
│  Protection measures:                             │
│  ├── Use REMOTE state (encrypted at rest)         │
│  ├── Enable ENCRYPTION (S3 SSE, Azure CMK)        │
│  ├── Restrict ACCESS (IAM policies, RBAC)         │
│  ├── NEVER commit state to version control        │
│  ├── Enable VERSIONING for recovery               │
│  └── Use STATE LOCKING to prevent corruption      │
│                                                   │
└──────────────────────────────────────────────────┘
```

```gitignore
# .gitignore — ALWAYS exclude state files
*.tfstate
*.tfstate.*
*.tfstate.backup
.terraform/
```

---

## 1.8 State Serial and Lineage

```
Serial: Incremented every time state is modified
┌────────────────────────────────────────────┐
│  Apply #1  →  serial: 1                   │
│  Apply #2  →  serial: 2                   │
│  Apply #3  →  serial: 3                   │
│  ...                                       │
│  Used to detect conflicts in remote state  │
└────────────────────────────────────────────┘

Lineage: UUID generated when state is created
┌────────────────────────────────────────────┐
│  lineage: "a1b2c3d4-e5f6-7890-abcd-..."  │
│                                            │
│  Prevents accidentally using state from    │
│  a different project or environment        │
│  Terraform will REFUSE if lineage differs  │
└────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "No state file found" | State deleted or wrong directory | Check path; restore from backup or re-import resources |
| State shows resources that don't exist | Resources deleted outside Terraform | Run `terraform refresh` or `terraform plan` (auto-refresh) |
| State out of sync | Manual changes to infrastructure | Import or refresh; avoid manual changes |
| Sensitive data exposed | State stored in plain text | Use remote backend with encryption |
| State file corrupted | Incomplete write or concurrent access | Restore from `.backup` or versioned storage |
| "state serial mismatch" | Concurrent modifications | Enable state locking; resolve conflict manually |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **State File** | JSON file mapping config to real resources |
| **Default Location** | `terraform.tfstate` in project directory |
| **Backup** | `terraform.tfstate.backup` (previous state) |
| **Serial** | Incrementing version number |
| **Lineage** | Unique UUID identifying state history |
| **Refresh** | Sync state with real infrastructure |
| **Sensitive Data** | State stores ALL values in plain text |
| **Best Practice** | Use remote state, enable encryption & locking |

---

## Quick Revision Questions

1. **What is the purpose of the Terraform state file?**
   > It maps configuration resources to real-world infrastructure objects by storing resource IDs, attributes, and metadata.

2. **Where is state stored by default?**
   > In a local file called `terraform.tfstate` in the working directory. A backup is kept in `terraform.tfstate.backup`.

3. **Why should state files never be committed to version control?**
   > State files contain sensitive data in plain text (passwords, keys, secrets) and could expose infrastructure details.

4. **What do the `serial` and `lineage` fields in state represent?**
   > `serial` is an incrementing version number (conflict detection). `lineage` is a UUID identifying the state chain (prevents accidental cross-environment state usage).

5. **How does Terraform determine what changes to make?**
   > It compares the desired state (configuration) with the current state (state file), refreshing from real infrastructure, and generates a plan for any differences.

---

[← Previous: Unit 3 - Conditionals and Loops](../03-Terraform-Language/07-conditionals-and-loops.md) | [Back to README](../README.md) | [Next: Remote State →](02-remote-state.md)
