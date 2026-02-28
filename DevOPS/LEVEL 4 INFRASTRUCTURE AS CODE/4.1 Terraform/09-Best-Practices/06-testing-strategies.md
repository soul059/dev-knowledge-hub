# Chapter 6: Testing Strategies

[← Previous: Security Practices](05-security-practices.md) | [Back to README](../README.md) | [Next: Documentation →](07-documentation.md)

---

## Overview

Testing Terraform code ensures infrastructure is **correct**, **secure**, and **reliable** before deployment. This chapter covers the testing pyramid from static analysis to end-to-end integration tests.

---

## 6.1 Testing Pyramid

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM TESTING PYRAMID                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│                      ╱╲                                   │
│                     ╱  ╲      End-to-End Tests            │
│                    ╱ E2E╲     (real infrastructure)        │
│                   ╱──────╲                                │
│                  ╱        ╲   Integration Tests            │
│                 ╱ Contract ╲  (Terratest, plan checks)     │
│                ╱────────────╲                              │
│               ╱              ╲  Unit Tests                 │
│              ╱   Unit Tests   ╲ (terraform test, validate)  │
│             ╱──────────────────╲                           │
│            ╱                    ╲  Static Analysis          │
│           ╱   Static Analysis   ╲ (fmt, tflint, tfsec)     │
│          ╱──────────────────────╲                          │
│                                                           │
│  Faster ← Speed    → Slower                               │
│  Cheaper ← Cost   → More expensive                        │
│  More ← Quantity   → Fewer                                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Static Analysis

```bash
# ─── Layer 1: Built-in tools ─────────────────────

# Format check
terraform fmt -check -recursive -diff
# Returns non-zero if files need formatting

# Syntax and configuration validation
terraform init -backend=false
terraform validate
# Checks types, required arguments, references

# ─── Layer 2: Linting (TFLint) ───────────────────
tflint --init
tflint --recursive

# .tflint.hcl
plugin "terraform" {
  enabled = true
  preset  = "recommended"
}

plugin "aws" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

# Custom rules
rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

# ─── Layer 3: Security scanning ──────────────────
# Trivy
trivy config . --severity HIGH,CRITICAL

# Checkov
checkov -d . --framework terraform
checkov -d . --check CKV_AWS_18    # Specific check

# tfsec (now part of Trivy)
tfsec . --minimum-severity HIGH
```

---

## 6.3 terraform test (Terraform 1.6+)

```
┌──────────────────────────────────────────────────────────┐
│      NATIVE TERRAFORM TEST FRAMEWORK                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  project/                                                 │
│  ├── main.tf                                              │
│  ├── variables.tf                                         │
│  ├── outputs.tf                                           │
│  └── tests/                                               │
│      ├── basic.tftest.hcl       ← Unit tests              │
│      ├── validation.tftest.hcl  ← Input validation tests  │
│      └── integration.tftest.hcl ← Integration tests       │
│                                                           │
│  Run: terraform test                                      │
│  Run specific: terraform test -filter=tests/basic.tftest  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── tests/basic.tftest.hcl ─────────────────────

# Variables for test
variables {
  project     = "test-project"
  environment = "dev"
  vpc_cidr    = "10.0.0.0/16"
}

# ─── Unit test: Plan-only (no real resources) ────
run "verify_vpc_configuration" {
  command = plan   # Only generates plan, no apply

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block is incorrect"
  }

  assert {
    condition     = aws_vpc.main.enable_dns_hostnames == true
    error_message = "DNS hostnames should be enabled"
  }

  assert {
    condition     = aws_vpc.main.tags["Environment"] == "dev"
    error_message = "Environment tag is incorrect"
  }
}

# ─── Test with variable overrides ────────────────
run "verify_prod_settings" {
  command = plan

  variables {
    environment   = "prod"
    instance_type = "m5.xlarge"
  }

  assert {
    condition     = aws_instance.app.instance_type == "m5.xlarge"
    error_message = "Prod should use m5.xlarge"
  }
}

# ─── Integration test: Apply real resources ──────
run "create_and_verify" {
  command = apply   # Creates real infrastructure

  assert {
    condition     = output.vpc_id != ""
    error_message = "VPC ID should not be empty"
  }

  assert {
    condition     = length(output.subnet_ids) == 3
    error_message = "Should create 3 subnets"
  }
}
# Resources are automatically destroyed after test
```

```hcl
# ─── tests/validation.tftest.hcl ─────────────────

# Test that invalid inputs are rejected
run "reject_invalid_environment" {
  command = plan

  variables {
    environment = "invalid"
  }

  expect_failures = [
    var.environment   # Expect validation to fail
  ]
}

run "reject_small_cidr" {
  command = plan

  variables {
    vpc_cidr = "10.0.0.0/28"
  }

  expect_failures = [
    var.vpc_cidr
  ]
}

# Test precondition failures
run "reject_wrong_ami_architecture" {
  command = plan

  expect_failures = [
    aws_instance.app   # Expect precondition to fail
  ]
}
```

```hcl
# ─── tests/module_test.tftest.hcl ────────────────

# Test a module with mock providers
run "test_networking_module" {
  command = plan

  module {
    source = "./modules/networking"
  }

  variables {
    vpc_cidr           = "10.0.0.0/16"
    availability_zones = ["us-east-1a", "us-east-1b"]
  }

  assert {
    condition     = length(module.networking.public_subnet_ids) == 2
    error_message = "Should create 2 public subnets"
  }
}
```

---

## 6.4 Terratest (Go)

```
┌──────────────────────────────────────────────────────────┐
│      TERRATEST: GO-BASED INTEGRATION TESTS                │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  project/                                                 │
│  ├── main.tf                                              │
│  ├── variables.tf                                         │
│  ├── outputs.tf                                           │
│  └── test/                                                │
│      ├── go.mod                                           │
│      ├── go.sum                                           │
│      └── terraform_test.go                                │
│                                                           │
│  Run: cd test && go test -v -timeout 30m                  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```go
// test/terraform_test.go
package test

import (
    "testing"
    "fmt"
    "net/http"

    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/gruntwork-io/terratest/modules/aws"
    "github.com/stretchr/testify/assert"
)

func TestVPCModule(t *testing.T) {
    t.Parallel()

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../",
        Vars: map[string]interface{}{
            "project":     "test",
            "environment": "test",
            "vpc_cidr":    "10.99.0.0/16",
        },
    })

    // Clean up after test
    defer terraform.Destroy(t, terraformOptions)

    // Apply infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcID := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcID)

    subnetIDs := terraform.OutputList(t, terraformOptions, "subnet_ids")
    assert.Equal(t, 3, len(subnetIDs))

    // Validate actual AWS resources
    vpc := aws.GetVpcById(t, vpcID, "us-east-1")
    assert.Equal(t, "10.99.0.0/16", *vpc.CidrBlock)

    // Validate tags
    tags := aws.GetTagsForVpc(t, vpcID, "us-east-1")
    assert.Equal(t, "test", tags["Project"])
}

func TestWebServerResponds(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../",
    })

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    // Get the URL from outputs
    url := terraform.Output(t, terraformOptions, "app_url")

    // HTTP request to verify app is running
    resp, err := http.Get(fmt.Sprintf("http://%s/health", url))
    assert.NoError(t, err)
    assert.Equal(t, 200, resp.StatusCode)
}
```

---

## 6.5 Plan-Based Testing

```hcl
# ─── Using terraform plan for validation ─────────

# Generate plan in JSON format
# terraform plan -out=tfplan
# terraform show -json tfplan > plan.json

# Python script to validate plan
# test_plan.py
```

```python
# test_plan.py — Validate Terraform plan output
import json
import sys

def test_plan():
    with open("plan.json") as f:
        plan = json.load(f)

    changes = plan.get("resource_changes", [])

    # Check no resources are destroyed
    destroys = [c for c in changes if "delete" in c["change"]["actions"]]
    assert len(destroys) == 0, f"Unexpected destroys: {[d['address'] for d in destroys]}"

    # Check required tags
    for change in changes:
        if change["change"]["actions"] == ["no-op"]:
            continue
        after = change["change"].get("after", {})
        tags = after.get("tags", {})
        if change["type"] not in ["aws_iam_policy", "aws_iam_role"]:
            assert "Environment" in tags, \
                f"{change['address']} missing Environment tag"

    # Check instance types are approved
    approved_types = ["t3.micro", "t3.small", "t3.medium", "m5.large"]
    instances = [c for c in changes if c["type"] == "aws_instance"]
    for inst in instances:
        itype = inst["change"]["after"]["instance_type"]
        assert itype in approved_types, \
            f"{inst['address']} uses unapproved type: {itype}"

    print("All plan validations passed!")

if __name__ == "__main__":
    test_plan()
```

---

## 6.6 Testing Strategy by Project Size

```
┌──────────────────────────────────────────────────────────┐
│           TESTING RECOMMENDATIONS                         │
├──────────────┬───────────────┬───────────────────────────┤
│ Project Size │ Tests         │ Tools                      │
├──────────────┼───────────────┼───────────────────────────┤
│              │ terraform fmt │ Built-in                   │
│ Small        │ validate      │ Built-in                   │
│ (personal)   │ TFLint        │ TFLint                     │
│              │               │                            │
├──────────────┼───────────────┼───────────────────────────┤
│              │ + security    │ Trivy, Checkov             │
│ Medium       │ + plan tests  │ terraform test (plan)      │
│ (team)       │ + var checks  │ terraform test (expect)    │
│              │               │                            │
├──────────────┼───────────────┼───────────────────────────┤
│              │ + integration │ terraform test (apply)     │
│ Large        │ + contract    │ Terratest                  │
│ (enterprise) │ + policy      │ Sentinel / OPA             │
│              │ + E2E         │ Custom scripts             │
└──────────────┴───────────────┴───────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Tests too slow | Real infrastructure creation | Use `command = plan` for unit tests |
| Terratest timeout | Large infrastructure deploy | Increase `-timeout` flag (e.g., 60m) |
| Test resources not cleaned up | Test crashed mid-run | Use `defer terraform.Destroy()`, run cleanup scripts |
| Flaky tests | Eventual consistency | Add retries, wait for resource readiness |
| `terraform test` not finding tests | Wrong directory | Tests must be in `tests/` directory with `.tftest.hcl` extension |

---

## Summary Table

| Test Type | Tool | Speed | Cost | Confidence |
|-----------|------|-------|------|------------|
| **Format** | `terraform fmt` | Instant | Free | Low |
| **Validate** | `terraform validate` | Seconds | Free | Low-Medium |
| **Lint** | TFLint | Seconds | Free | Medium |
| **Security** | Trivy/Checkov | Seconds | Free | Medium |
| **Unit (plan)** | `terraform test` | Seconds | Free | Medium-High |
| **Integration** | `terraform test` (apply) | Minutes | Cloud costs | High |
| **E2E** | Terratest | Minutes-Hours | Cloud costs | Very High |

---

## Quick Revision Questions

1. **What command runs native Terraform tests?**
   > `terraform test` — runs all `.tftest.hcl` files in the `tests/` directory. Use `-filter` for specific tests.

2. **What is the difference between `command = plan` and `command = apply` in terraform test?**
   > `plan` only generates a plan (no real resources, fast, free). `apply` creates real infrastructure (slower, costs money, higher confidence). Resources are auto-destroyed after apply tests.

3. **How do you test that an invalid variable is properly rejected?**
   > Use `expect_failures` in a test run block to assert that a validation rule or precondition produces an error.

4. **What is Terratest?**
   > A Go library for writing integration tests that deploy real infrastructure, validate it, and clean up. Provides helpers for AWS, Azure, GCP, Kubernetes, and HTTP testing.

5. **What is the recommended testing strategy for most teams?**
   > Static analysis (fmt, validate, TFLint, security scan) for all projects. Add `terraform test` with plan-based tests for modules. Use integration tests (apply) for critical infrastructure modules only.

---

[← Previous: Security Practices](05-security-practices.md) | [Back to README](../README.md) | [Next: Documentation →](07-documentation.md)
