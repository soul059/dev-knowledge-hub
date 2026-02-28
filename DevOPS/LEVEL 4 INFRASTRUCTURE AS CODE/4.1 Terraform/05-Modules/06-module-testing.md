# Chapter 6: Module Testing

[← Previous: Writing Reusable Modules](05-writing-reusable-modules.md) | [Back to README](../README.md) | [Next: Module Composition →](07-module-composition.md)

---

## Overview

Testing Terraform modules ensures they work correctly, handle edge cases, and don't introduce regressions. Terraform 1.6+ introduced **native testing** with `.tftest.hcl` files, complementing external tools like Terratest.

---

## 6.1 Testing Pyramid for Terraform

```
┌──────────────────────────────────────────────────────────┐
│              TERRAFORM TESTING PYRAMID                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│                    ╱╲                                     │
│                   ╱  ╲      End-to-End / Integration      │
│                  ╱ E2E╲     Deploy real infra, validate   │
│                 ╱──────╲    Slow, expensive               │
│                ╱        ╲                                  │
│               ╱ Contract ╲   Module interface tests       │
│              ╱────────────╲  Inputs, outputs, plans       │
│             ╱              ╲                               │
│            ╱   Unit Tests   ╲  terraform validate         │
│           ╱──────────────────╲ terraform plan              │
│          ╱                    ╲ Static analysis            │
│         ╱   Static Analysis   ╲ tflint, checkov, tfsec    │
│        ╱──────────────────────╲                            │
│       ╱     Format / Lint      ╲ terraform fmt             │
│      ╱──────────────────────────╲                          │
│                                                           │
│  Bottom = Fast, cheap, run often                          │
│  Top    = Slow, expensive, run less often                 │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Native Testing (Terraform 1.6+)

### Test File Structure

```
modules/vpc/
├── main.tf
├── variables.tf
├── outputs.tf
└── tests/
    ├── basic.tftest.hcl
    ├── validation.tftest.hcl
    └── complete.tftest.hcl
```

### Basic Test

```hcl
# tests/basic.tftest.hcl

# Variables for the test
variables {
  name       = "test-vpc"
  cidr_block = "10.0.0.0/16"
  azs        = ["us-east-1a", "us-east-1b"]
  public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  environment = "test"
}

# Test: Plan succeeds (no real resources created)
run "plan_succeeds" {
  command = plan     # Only plan, don't apply

  assert {
    condition     = aws_vpc.this.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block should be 10.0.0.0/16"
  }

  assert {
    condition     = length(aws_subnet.public) == 2
    error_message = "Should create 2 public subnets"
  }
}

# Test: Validate outputs
run "outputs_correct" {
  command = plan

  assert {
    condition     = output.vpc_id != null
    error_message = "vpc_id output should not be null"
  }

  assert {
    condition     = length(output.public_subnet_ids) == 2
    error_message = "Should output 2 subnet IDs"
  }
}
```

### Validation Tests

```hcl
# tests/validation.tftest.hcl

# Test: Invalid CIDR should fail
run "invalid_cidr_rejected" {
  command = plan

  variables {
    name       = "test"
    cidr_block = "not-a-cidr"
    azs        = ["us-east-1a"]
    environment = "test"
  }

  expect_failures = [
    var.cidr_block,     # Expect this variable's validation to fail
  ]
}

# Test: Invalid environment should fail
run "invalid_environment_rejected" {
  command = plan

  variables {
    name        = "test"
    cidr_block  = "10.0.0.0/16"
    azs         = ["us-east-1a"]
    environment = "invalid"
  }

  expect_failures = [
    var.environment,
  ]
}
```

### Integration Test (applies real resources)

```hcl
# tests/complete.tftest.hcl

# Provider config for tests
provider "aws" {
  region = "us-east-1"
}

variables {
  name       = "tftest-vpc"
  cidr_block = "10.99.0.0/16"
  azs        = ["us-east-1a"]
  public_subnets = ["10.99.1.0/24"]
  environment = "test"
}

# Apply real resources and validate
run "creates_vpc" {
  command = apply      # Actually create resources

  assert {
    condition     = aws_vpc.this.cidr_block == "10.99.0.0/16"
    error_message = "VPC CIDR mismatch"
  }

  assert {
    condition     = aws_vpc.this.enable_dns_support == true
    error_message = "DNS support should be enabled"
  }
}

# Resources are automatically destroyed after tests complete
```

### Running Tests

```bash
# Run all tests
$ terraform test

# Run tests in a specific directory
$ terraform test -test-directory=tests

# Verbose output
$ terraform test -verbose

# Filter specific test file
$ terraform test -filter=tests/basic.tftest.hcl
```

---

## 6.3 Static Analysis Tools

### terraform validate

```bash
$ terraform validate
Success! The configuration is valid.

# Catches:
# - Syntax errors
# - Invalid references
# - Missing required arguments
# - Type mismatches
```

### terraform fmt

```bash
# Check formatting
$ terraform fmt -check -recursive
# Exit code 0 = formatted, non-zero = needs formatting

# Auto-fix formatting
$ terraform fmt -recursive
```

### TFLint

```bash
# Install
$ brew install tflint    # macOS
$ choco install tflint   # Windows

# Initialize rules
$ tflint --init

# Run
$ tflint
# Catches:
# - Invalid instance types
# - Deprecated resource attributes
# - Naming convention violations
```

```hcl
# .tflint.hcl
plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

rule "terraform_naming_convention" {
  enabled = true
}

rule "terraform_documented_variables" {
  enabled = true
}
```

### Checkov / tfsec

```bash
# Checkov — Security scanning
$ pip install checkov
$ checkov -d .
# Checks for security misconfigurations

# tfsec (now part of Trivy)
$ brew install tfsec
$ tfsec .
# Detects security issues in Terraform code
```

---

## 6.4 Terratest (Go-based Testing)

```go
// test/vpc_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestVpcModule(t *testing.T) {
    t.Parallel()

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "name":       "test-vpc",
            "cidr_block": "10.99.0.0/16",
            "azs":        []string{"us-east-1a"},
        },
    })

    // Clean up after test
    defer terraform.Destroy(t, terraformOptions)

    // Create resources
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)
    assert.Contains(t, vpcId, "vpc-")
}
```

```bash
# Run Terratest
$ cd test
$ go test -v -timeout 30m
```

---

## 6.5 CI/CD Testing Pipeline

```yaml
# .github/workflows/terraform-test.yml
name: Terraform Module Tests
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - name: Format Check
        run: terraform fmt -check -recursive

      - name: Validate
        run: |
          terraform init -backend=false
          terraform validate

      - name: TFLint
        uses: terraform-linters/setup-tflint@v4
        with:
          tflint_version: latest
      - run: tflint --init && tflint

      - name: Security Scan
        uses: aquasecurity/tfsec-action@v1.0.0

  test:
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3

      - name: Terraform Test
        run: terraform test
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "terraform test" not found | TF version < 1.6 | Upgrade to Terraform 1.6+ |
| Tests timeout | Integration tests take too long | Use `command = plan` for fast tests; apply only when needed |
| Leftover resources from failed test | Test crashed before cleanup | Manually clean up; use naming prefixes to identify |
| Provider not configured in tests | Missing provider block | Add provider block in test file |
| TFLint rules too strict | Default rules may not match your conventions | Customize `.tflint.hcl` |

---

## Summary Table

| Tool / Method | Type | Speed | Cost | What It Tests |
|--------------|------|-------|------|---------------|
| `terraform fmt` | Formatting | Instant | Free | Code style |
| `terraform validate` | Syntax | Instant | Free | Config validity |
| TFLint | Static analysis | Fast | Free | Best practices, valid values |
| Checkov / tfsec | Security scan | Fast | Free | Security misconfigurations |
| `terraform test` (plan) | Unit/Contract | Fast | Free | Plan-time assertions |
| `terraform test` (apply) | Integration | Slow | Cloud costs | Real infrastructure |
| Terratest | Integration | Slow | Cloud costs | Real infrastructure (Go) |

---

## Quick Revision Questions

1. **What testing tool was introduced in Terraform 1.6?**
   > Native testing with `.tftest.hcl` files, run via `terraform test`.

2. **What is the difference between `command = plan` and `command = apply` in tests?**
   > `plan` only generates an execution plan (fast, free, no real resources). `apply` creates actual infrastructure (slow, costs money, tests real behavior).

3. **What does `expect_failures` do in a test?**
   > It specifies which validation rules should fail, allowing you to test that invalid inputs are properly rejected.

4. **Name three static analysis tools for Terraform.**
   > `terraform validate` (syntax), TFLint (best practices), Checkov/tfsec (security scanning).

5. **Why should integration tests use unique naming prefixes?**
   > To avoid conflicts with production resources and make it easy to identify and clean up test resources if tests fail.

---

[← Previous: Writing Reusable Modules](05-writing-reusable-modules.md) | [Back to README](../README.md) | [Next: Module Composition →](07-module-composition.md)
