# Chapter 7: Documentation

[← Previous: Testing Strategies](06-testing-strategies.md) | [Back to README](../README.md) | [Next: Unit 10 - Multi-Account Architecture →](../10-Real-World-Patterns/01-multi-account-architecture.md)

---

## Overview

Good documentation makes Terraform code **maintainable** and **accessible** to the whole team. This chapter covers documentation tools, README conventions, inline commenting, and automated documentation generation.

---

## 7.1 Documentation Layers

```
┌──────────────────────────────────────────────────────────┐
│           DOCUMENTATION LAYERS                            │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Layer 1: Inline Comments                                 │
│  └─ Explain WHY, not WHAT                                 │
│                                                           │
│  Layer 2: Variable/Output Descriptions                    │
│  └─ Self-documenting inputs and outputs                   │
│                                                           │
│  Layer 3: Module README.md                                │
│  └─ Usage examples, requirements, architecture            │
│                                                           │
│  Layer 4: Project README.md                               │
│  └─ Overview, setup, deployment instructions              │
│                                                           │
│  Layer 5: Architecture Documentation                      │
│  └─ Diagrams, design decisions, runbooks                  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 7.2 Inline Comments

```hcl
# ─── GOOD: Explain WHY ──────────────────────────

# Use t3 instead of t2 for unlimited burst credits
# without additional charges (t2 charges for burst)
variable "instance_type" {
  default = "t3.medium"
}

# NAT Gateway is placed in az-a only to reduce costs.
# For HA, deploy one per AZ (see modules/networking/ha variant)
resource "aws_nat_gateway" "main" {
  subnet_id     = aws_subnet.public[0].id
  allocation_id = aws_eip.nat.id
}

# Ignore AMI changes to prevent instance replacement
# when the golden image pipeline publishes new AMIs.
# AMI updates are handled by a separate rolling deploy process.
resource "aws_instance" "app" {
  ami = var.ami_id

  lifecycle {
    ignore_changes = [ami]
  }
}

# ─── BAD: Stating the obvious ───────────────────

# Create a VPC                    ← OBVIOUS from code
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"     # Set CIDR to 10.0.0.0/16  ← OBVIOUS
}
```

---

## 7.3 Variable and Output Descriptions

```hcl
# ─── Every variable MUST have a description ──────

variable "vpc_cidr" {
  type        = string
  description = "CIDR block for the VPC. Must be /16 or larger for subnet allocation."
  default     = "10.0.0.0/16"

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }
}

variable "enable_nat_gateway" {
  type        = bool
  description = "Whether to create a NAT Gateway for private subnet internet access. Costs ~$32/month per gateway."
  default     = true
}

variable "instance_type" {
  type        = string
  description = "EC2 instance type. Use t3.* for dev, m5.* for prod. See approved list in wiki."
  default     = "t3.medium"
}

variable "tags" {
  type        = map(string)
  description = "Additional tags to apply to all resources. Merged with default tags (Project, Environment, ManagedBy)."
  default     = {}
}

# ─── Outputs with descriptions ──────────────────
output "vpc_id" {
  description = "ID of the created VPC. Used as input for compute and database modules."
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "List of private subnet IDs across all AZs. Pass to EKS, RDS, or ECS modules."
  value       = aws_subnet.private[*].id
}

output "nat_gateway_ip" {
  description = "Public IP of the NAT Gateway. Add to external service allow-lists."
  value       = aws_eip.nat.public_ip
}
```

---

## 7.4 terraform-docs (Automated Documentation)

```
┌──────────────────────────────────────────────────────────┐
│       terraform-docs: AUTO-GENERATED DOCUMENTATION        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Reads .tf files and generates documentation              │
│  for variables, outputs, providers, and resources         │
│                                                           │
│  Install:                                                 │
│  brew install terraform-docs                              │
│  # or download from GitHub releases                       │
│                                                           │
│  Usage:                                                   │
│  terraform-docs markdown table . > README.md              │
│  terraform-docs markdown table --output-file README.md .  │
│                                                           │
│  Auto-inject into existing README:                        │
│  terraform-docs markdown table \                          │
│    --output-file README.md \                              │
│    --output-mode inject .                                 │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```yaml
# ─── .terraform-docs.yml ─────────────────────────
formatter: markdown table

header-from: docs/header.md

sections:
  show:
    - header
    - requirements
    - providers
    - inputs
    - outputs
    - resources

content: |-
  {{ .Header }}

  ## Usage

  ```hcl
  module "vpc" {
    source = "github.com/company/terraform-aws-vpc"

    project     = "myapp"
    environment = "prod"
    vpc_cidr    = "10.0.0.0/16"
  }
  ```

  {{ .Requirements }}
  {{ .Providers }}
  {{ .Resources }}
  {{ .Inputs }}
  {{ .Outputs }}

output:
  file: README.md
  mode: inject
  template: |-
    <!-- BEGIN_TF_DOCS -->
    {{ .Content }}
    <!-- END_TF_DOCS -->

sort:
  enabled: true
  by: required
```

Generated output example:
```markdown
<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.5.0 |
| aws | ~> 5.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| vpc_cidr | CIDR block for the VPC | `string` | `"10.0.0.0/16"` | no |
| environment | Deployment environment | `string` | n/a | yes |
| project | Project name for tagging | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | ID of the created VPC |
| subnet_ids | List of subnet IDs |
<!-- END_TF_DOCS -->
```

---

## 7.5 Module README Template

```markdown
# Module: AWS VPC

## Description
Creates a VPC with public and private subnets, NAT Gateway,
and associated routing.

## Architecture
```
┌─────────────────────────────────────────┐
│  VPC (10.0.0.0/16)                      │
│  ┌──────────┐  ┌──────────┐             │
│  │ Public   │  │ Public   │             │
│  │ Subnet   │  │ Subnet   │             │
│  │ AZ-a     │  │ AZ-b     │             │
│  └────┬─────┘  └────┬─────┘             │
│       │ NAT GW      │                   │
│  ┌────┴─────┐  ┌────┴─────┐             │
│  │ Private  │  │ Private  │             │
│  │ Subnet   │  │ Subnet   │             │
│  │ AZ-a     │  │ AZ-b     │             │
│  └──────────┘  └──────────┘             │
└─────────────────────────────────────────┘
```

## Usage
```hcl
module "vpc" {
  source  = "github.com/company/terraform-aws-vpc?ref=v2.1.0"

  project            = "myapp"
  environment        = "prod"
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b"]
  enable_nat_gateway = true
}
```

## Requirements
- Terraform >= 1.5.0
- AWS Provider ~> 5.0
- AWS credentials with VPC, Subnet, NAT Gateway, EIP permissions

## Inputs
<!-- BEGIN_TF_DOCS -->
(auto-generated by terraform-docs)
<!-- END_TF_DOCS -->

## Outputs
(auto-generated)

## Examples
- [Basic VPC](examples/basic/)
- [VPC with VPN](examples/vpn/)
- [Multi-AZ HA](examples/ha/)

## Known Issues
- NAT Gateway takes 2-3 minutes to become available
- Changing VPC CIDR requires destroy/recreate
```

---

## 7.6 Pre-commit Documentation Hook

```yaml
# ─── Auto-update docs on commit ──────────────────
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-tf
    rev: v1.83.6
    hooks:
      - id: terraform_docs
        name: Generate terraform docs
        args:
          - --hook-config=--path-to-file=README.md
          - --hook-config=--add-to-existing-file=true
          - --args=--config=.terraform-docs.yml
```

```
┌──────────────────────────────────────────────────────────┐
│       DOCUMENTATION AUTOMATION WORKFLOW                   │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Developer edits variables.tf                             │
│       │                                                   │
│       ▼                                                   │
│  git commit                                               │
│       │                                                   │
│       ▼                                                   │
│  pre-commit hook runs terraform-docs                      │
│       │                                                   │
│       ▼                                                   │
│  README.md auto-updated with new variables                │
│       │                                                   │
│       ▼                                                   │
│  Updated README included in commit                        │
│                                                           │
│  Result: Documentation always matches code                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| terraform-docs not updating README | Missing inject markers | Add `<!-- BEGIN_TF_DOCS -->` and `<!-- END_TF_DOCS -->` |
| Descriptions not showing | Missing `description` attribute | Add descriptions to all variables and outputs |
| Pre-commit hook fails | terraform-docs not installed | Install via `brew` or download binary |
| Docs out of sync with code | No automation | Set up pre-commit hook or CI check |
| Complex module hard to understand | No architecture diagram | Add ASCII or Mermaid diagrams to README |

---

## Summary Table

| Documentation Type | Where | How | Automation |
|-------------------|-------|-----|------------|
| **Inline comments** | `.tf` files | Manual | N/A |
| **Variable descriptions** | `variables.tf` | Manual | terraform-docs reads these |
| **Output descriptions** | `outputs.tf` | Manual | terraform-docs reads these |
| **Module README** | `README.md` | Template + auto-gen | terraform-docs + pre-commit |
| **Architecture diagrams** | README / docs/ | ASCII art or Mermaid | Manual creation |
| **Runbooks** | Wiki / docs/ | Manual | N/A |

---

## Quick Revision Questions

1. **What should inline comments explain?**
   > WHY something is done, not WHAT is being done. The code shows what; comments explain context, trade-offs, and non-obvious decisions.

2. **What tool auto-generates Terraform documentation?**
   > `terraform-docs` — reads `.tf` files and generates Markdown tables for variables, outputs, providers, and resources.

3. **How do you keep documentation in sync with code?**
   > Use `terraform-docs` with pre-commit hooks to auto-update README.md on every commit. Use inject markers (`<!-- BEGIN_TF_DOCS -->` / `<!-- END_TF_DOCS -->`) in the README.

4. **What should every variable include?**
   > A `description` attribute explaining what it does, acceptable values, and any cost or operational implications.

5. **What sections should a module README contain?**
   > Description, architecture diagram, usage example, requirements, inputs (auto-generated), outputs (auto-generated), examples directory, and known issues.

---

[← Previous: Testing Strategies](06-testing-strategies.md) | [Back to README](../README.md) | [Next: Unit 10 - Multi-Account Architecture →](../10-Real-World-Patterns/01-multi-account-architecture.md)
