# Chapter 6: Checks and Validation

[← Previous: Terraform Import](05-terraform-import.md) | [Back to README](../README.md) | [Next: Unit 9 - Code Organization →](../09-Best-Practices/01-code-organization.md)

---

## Overview

Terraform provides multiple mechanisms for **validating** infrastructure configuration and state. From input validation to runtime checks, these features help catch errors early and ensure infrastructure meets requirements.

---

## 6.1 Validation Layers

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM VALIDATION PYRAMID                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│                    ┌─────────┐                            │
│                    │ Runtime │  check blocks,             │
│                    │ Checks  │  postconditions            │
│                   ─┴─────────┴─                           │
│                  ┌─────────────┐                          │
│                  │ Plan-Time   │  preconditions,          │
│                  │ Validation  │  lifecycle checks        │
│                 ─┴─────────────┴─                         │
│                ┌───────────────────┐                      │
│                │  Input Validation │  variable validation  │
│               ─┴───────────────────┴─                     │
│              ┌─────────────────────────┐                  │
│              │  Static Analysis        │  fmt, validate,  │
│              │                         │  TFLint, tfsec   │
│             ─┴─────────────────────────┴─                 │
│            ┌───────────────────────────────┐              │
│            │  Type System                  │  HCL types,  │
│            │                               │  constraints │
│            └───────────────────────────────┘              │
│                                                           │
│  Earlier ← More errors caught → Later                    │
│  Cheaper ← Cost to fix      → More expensive             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Variable Validation

```hcl
# ─── Basic validation ────────────────────────────
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# ─── Multiple validation rules ───────────────────
variable "instance_type" {
  type        = string
  description = "EC2 instance type"

  validation {
    condition     = can(regex("^[a-z][0-9][a-z]?\\.", var.instance_type))
    error_message = "Must be a valid AWS instance type format (e.g., t3.micro)."
  }

  validation {
    condition     = !contains(["t2.nano", "t2.micro"], var.instance_type)
    error_message = "t2.nano and t2.micro are not allowed."
  }
}

# ─── Numeric validation ──────────────────────────
variable "port" {
  type        = number
  description = "Application port"

  validation {
    condition     = var.port >= 1 && var.port <= 65535
    error_message = "Port must be between 1 and 65535."
  }
}

# ─── String pattern validation ───────────────────
variable "project_name" {
  type        = string
  description = "Project name for resource naming"

  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{2,20}$", var.project_name))
    error_message = "Project name must be 3-21 chars, lowercase, start with letter, only letters/numbers/hyphens."
  }
}

# ─── CIDR validation ─────────────────────────────
variable "vpc_cidr" {
  type        = string
  description = "VPC CIDR block"

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }

  validation {
    condition     = tonumber(split("/", var.vpc_cidr)[1]) <= 24
    error_message = "CIDR block must be /24 or larger."
  }
}

# ─── List validation ─────────────────────────────
variable "availability_zones" {
  type        = list(string)
  description = "List of AZs"

  validation {
    condition     = length(var.availability_zones) >= 2
    error_message = "At least 2 availability zones required for HA."
  }

  validation {
    condition     = alltrue([for az in var.availability_zones : can(regex("^[a-z]{2}-[a-z]+-[0-9][a-z]$", az))])
    error_message = "Each AZ must be a valid format (e.g., us-east-1a)."
  }
}

# ─── Object validation ───────────────────────────
variable "database_config" {
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    storage_gb     = number
  })

  validation {
    condition     = contains(["postgres", "mysql", "mariadb"], var.database_config.engine)
    error_message = "Database engine must be postgres, mysql, or mariadb."
  }

  validation {
    condition     = var.database_config.storage_gb >= 20 && var.database_config.storage_gb <= 1000
    error_message = "Storage must be between 20 and 1000 GB."
  }
}
```

---

## 6.3 check Blocks (Terraform 1.5+)

```hcl
# ─── Continuous validation checks ────────────────
# check blocks produce WARNINGS, not errors
# They run during plan and apply

check "health_check" {
  data "http" "api_health" {
    url = "https://${aws_lb.main.dns_name}/health"
  }

  assert {
    condition     = data.http.api_health.status_code == 200
    error_message = "Application health check failed!"
  }
}

check "certificate_expiry" {
  data "aws_acm_certificate" "main" {
    domain = "example.com"
  }

  assert {
    condition     = timecmp(data.aws_acm_certificate.main.not_after, timeadd(timestamp(), "720h")) > 0
    error_message = "SSL certificate expires within 30 days!"
  }
}

check "budget_alert" {
  assert {
    condition     = var.instance_count <= 10
    error_message = "Warning: deploying more than 10 instances may exceed budget."
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│      check vs precondition vs postcondition               │
├──────────────────┬──────────────┬────────────────────────┤
│ Feature          │ Behavior     │ Use Case               │
├──────────────────┼──────────────┼────────────────────────┤
│ check + assert   │ WARNING      │ Non-blocking health    │
│                  │ (continues)  │ checks, monitoring     │
│                  │              │                        │
│ precondition     │ ERROR        │ Input validation,      │
│                  │ (stops plan) │ prerequisites          │
│                  │              │                        │
│ postcondition    │ ERROR        │ Output validation,     │
│                  │ (stops apply)│ resource verification  │
└──────────────────┴──────────────┴────────────────────────┘
```

---

## 6.4 Preconditions and Postconditions

```hcl
# ─── Resource preconditions ──────────────────────
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id

  lifecycle {
    precondition {
      condition     = data.aws_ami.selected.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }

    precondition {
      condition     = data.aws_subnet.selected.available_ip_address_count > 0
      error_message = "Subnet has no available IP addresses."
    }

    postcondition {
      condition     = self.public_ip != null
      error_message = "Instance did not receive a public IP."
    }
  }
}

# ─── Data source postconditions ──────────────────
data "aws_ami" "selected" {
  most_recent = true
  owners      = ["self"]

  filter {
    name   = "name"
    values = ["myapp-*"]
  }

  lifecycle {
    postcondition {
      condition     = length(self.id) > 0
      error_message = "No AMI found matching the criteria."
    }
  }
}

# ─── Output preconditions ────────────────────────
output "database_endpoint" {
  value = aws_db_instance.main.endpoint

  precondition {
    condition     = aws_db_instance.main.status == "available"
    error_message = "Database is not yet available."
  }
}
```

---

## 6.5 Static Analysis Tools

```bash
# ─── terraform fmt: Format checking ──────────────
terraform fmt -check -recursive
# Returns exit code 0 if formatted, non-zero if not

terraform fmt -diff
# Shows formatting differences

# ─── terraform validate: Syntax/config checking ──
terraform validate
# Checks syntax, type errors, required arguments

# ─── TFLint: Linting ─────────────────────────────
# Install: https://github.com/terraform-linters/tflint
tflint --init                     # Download plugins
tflint                            # Run linting

# .tflint.hcl configuration
plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
}

# ─── tfsec / Trivy: Security scanning ────────────
# Scans for security misconfigurations
tfsec .
trivy config .

# ─── Checkov: Policy-as-code ─────────────────────
checkov -d .
checkov -f main.tf --check CKV_AWS_79   # Specific check
```

---

## 6.6 CI/CD Validation Pipeline

```yaml
# ─── GitHub Actions: Complete validation ─────────
name: Terraform Validation
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Format Check
        run: terraform fmt -check -recursive

      - name: Init
        run: terraform init -backend=false

      - name: Validate
        run: terraform validate

      - name: TFLint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest
      - run: tflint --init && tflint

      - name: Security Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: config
          scan-ref: .

      - name: Plan
        run: terraform plan -no-color
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Validation runs but doesn't catch issues | Wrong condition logic | Test conditions in `terraform console` |
| `check` block fails on first apply | Data source not yet available | Checks produce warnings, not errors — review after apply |
| `precondition` blocks plan | Invalid input value | Fix input before applying |
| `postcondition` blocks apply | Resource didn't meet criteria | Investigate resource creation; may need config changes |
| CI validation passes, apply fails | Runtime vs static issues | Add postconditions for runtime checks |

---

## Summary Table

| Mechanism | When Runs | Severity | Use Case |
|-----------|-----------|----------|----------|
| **Type system** | Parse time | Error | Basic type correctness |
| **Variable validation** | Plan time | Error | Input constraints |
| **terraform validate** | CLI command | Error | Syntax/config errors |
| **precondition** | Plan time | Error | Prerequisites check |
| **postcondition** | Apply time | Error | Output verification |
| **check + assert** | Plan/Apply | Warning | Health/monitoring checks |
| **TFLint** | CLI tool | Warning/Error | Best practices linting |
| **tfsec/Trivy** | CLI tool | Warning/Error | Security scanning |

---

## Quick Revision Questions

1. **What is the difference between `check` blocks and `precondition`?**
   > `check` blocks produce warnings and don't stop operations. `precondition` produces errors and stops the plan/apply if the condition fails.

2. **How do you validate variable input in Terraform?**
   > Use `validation` blocks inside `variable` blocks with a `condition` expression and `error_message`. A variable can have multiple validation rules.

3. **What does `terraform validate` check?**
   > It checks configuration syntax, type correctness, required arguments, and internal consistency — but does not access remote state or APIs.

4. **What are postconditions used for?**
   > Verifying that a resource was created correctly by checking its actual attributes (via `self`) after creation. Errors stop the apply.

5. **Name three static analysis tools for Terraform.**
   > TFLint (linting and best practices), tfsec/Trivy (security misconfiguration scanning), and Checkov (policy-as-code compliance checking).

---

[← Previous: Terraform Import](05-terraform-import.md) | [Back to README](../README.md) | [Next: Unit 9 - Code Organization →](../09-Best-Practices/01-code-organization.md)
