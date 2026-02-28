# Chapter 8.1: Infrastructure as Code Security Scanning

## Overview

Infrastructure as Code (IaC) defines cloud resources in version-controlled configuration files — Terraform, CloudFormation, ARM templates, Pulumi, and Ansible. IaC security scanning detects misconfigurations (public S3 buckets, open security groups, unencrypted databases) before deployment, shifting infrastructure security left.

---

## Why Scan IaC?

```
IaC MISCONFIGURATION — THE #1 CLOUD RISK:

  ┌──────────────────────────────────────────────────────┐
  │                                                        │
  │  Developer writes Terraform  ──────▶  terraform apply  │
  │                                            │           │
  │  resource "aws_s3_bucket" "data" {         │           │
  │    bucket = "my-data"                      │           │
  │    # Missing: encryption ✗                  │           │
  │    # Missing: public access block ✗         ▼           │
  │    # Missing: logging ✗              Public S3 Bucket! │
  │  }                                    DATA BREACH!      │
  │                                                        │
  │  WITH IaC SCANNING:                                    │
  │  ┌────────────────────────────────────────────┐        │
  │  │ ✗ FAIL: S3 bucket encryption not enabled   │        │
  │  │ ✗ FAIL: S3 bucket public access not blocked│        │
  │  │ ✗ FAIL: S3 bucket logging not configured   │        │
  │  │                                             │        │
  │  │ Fix these BEFORE terraform apply!           │        │
  │  └────────────────────────────────────────────┘        │
  └──────────────────────────────────────────────────────┘
```

---

## IaC Scanning Tools

| Tool | IaC Support | Policies | License |
|------|------------|----------|---------|
| **Checkov** | Terraform, CloudFormation, K8s, Helm, ARM, Ansible | 1000+ built-in | OSS |
| **tfsec** (now Trivy) | Terraform, CloudFormation | 300+ built-in | OSS |
| **Trivy config** | Terraform, CloudFormation, K8s, Dockerfile | Unified scanner | OSS |
| **Terrascan** | Terraform, K8s, Helm, ARM, CloudFormation | OPA-based | OSS |
| **KICS** | 15+ IaC technologies | 2000+ queries | OSS |
| **Snyk IaC** | Terraform, CloudFormation, K8s, ARM | Commercial rules | Commercial |

---

## Checkov

```bash
# Install Checkov
pip install checkov

# Scan Terraform directory
checkov -d ./terraform/

# Scan specific file
checkov -f main.tf

# Scan with framework filter
checkov -d . --framework terraform

# Output formats
checkov -d . --output json > checkov-results.json
checkov -d . --output sarif > checkov-results.sarif

# Skip specific checks
checkov -d . --skip-check CKV_AWS_18,CKV_AWS_19

# Use Checkov config file
checkov -d . --config-file .checkov.yaml

# Scan Terraform plan (detects dynamic values)
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json
checkov -f plan.json --framework terraform_plan
```

```yaml
# .checkov.yaml
branch: main
compact: true
directory:
  - terraform/
framework:
  - terraform
output:
  - cli
  - sarif
skip-check:
  - CKV_AWS_18    # S3 logging (handled by org policy)
soft-fail-on:
  - CKV_AWS_145   # S3 KMS encryption (using SSE-S3 instead)
```

### Common Checkov Findings

```hcl
# BAD: Public S3 bucket
resource "aws_s3_bucket" "bad" {
  bucket = "my-data"
}
# CKV_AWS_18: Ensure S3 bucket has logging enabled ✗
# CKV_AWS_19: Ensure S3 bucket has encryption enabled ✗
# CKV_AWS_21: Ensure S3 bucket has versioning enabled ✗
# CKV2_AWS_6: Ensure S3 bucket has public access block ✗

# GOOD: Secure S3 bucket
resource "aws_s3_bucket" "good" {
  bucket = "my-data"
}

resource "aws_s3_bucket_versioning" "good" {
  bucket = aws_s3_bucket.good.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "good" {
  bucket = aws_s3_bucket.good.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "good" {
  bucket                  = aws_s3_bucket.good.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_logging" "good" {
  bucket        = aws_s3_bucket.good.id
  target_bucket = aws_s3_bucket.logs.id
  target_prefix = "s3-access-logs/"
}
```

---

## Trivy for IaC

```bash
# Trivy config scanning (replacement for tfsec)
trivy config ./terraform/
trivy config ./cloudformation/
trivy config ./k8s/

# With severity filter
trivy config --severity HIGH,CRITICAL ./terraform/

# Exit code for CI/CD
trivy config --exit-code 1 --severity CRITICAL ./terraform/

# SARIF output
trivy config --format sarif --output trivy-iac.sarif ./terraform/

# Scan specific file
trivy config main.tf
```

---

## CI/CD Integration

```yaml
# GitHub Actions — IaC Security Scanning
name: IaC Security
on:
  pull_request:
    paths:
      - 'terraform/**'
      - 'cloudformation/**'

jobs:
  checkov:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Checkov Scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
          output_format: sarif
          output_file_path: checkov.sarif
          soft_fail: false
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: checkov.sarif

  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Trivy IaC Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: config
          scan-ref: terraform/
          format: sarif
          output: trivy-iac.sarif
          exit-code: 1
          severity: HIGH,CRITICAL
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: trivy-iac.sarif

  # Terraform plan scan (catches dynamic issues)
  plan-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - name: Terraform Plan
        run: |
          cd terraform
          terraform init
          terraform plan -out=plan.tfplan
          terraform show -json plan.tfplan > plan.json
      - name: Checkov Plan Scan
        run: checkov -f terraform/plan.json --framework terraform_plan
```

---

## Custom Policies with Checkov

```python
# custom_checks/s3_encryption.py
from checkov.terraform.checks.resource.base_resource_check import BaseResourceCheck
from checkov.common.models.enums import CheckResult, CheckCategories

class S3BucketKMSEncryption(BaseResourceCheck):
    def __init__(self):
        name = "Ensure S3 bucket uses KMS encryption (not SSE-S3)"
        id = "CKV_CUSTOM_1"
        supported_resources = ["aws_s3_bucket_server_side_encryption_configuration"]
        categories = [CheckCategories.ENCRYPTION]
        super().__init__(name=name, id=id, categories=categories,
                        supported_resources=supported_resources)

    def scan_resource_conf(self, conf):
        rules = conf.get("rule", [{}])
        for rule in rules:
            default_enc = rule.get("apply_server_side_encryption_by_default", [{}])
            for enc in default_enc:
                algorithm = enc.get("sse_algorithm", [""])[0]
                if algorithm == "aws:kms":
                    return CheckResult.PASSED
        return CheckResult.FAILED

check = S3BucketKMSEncryption()
```

```bash
# Run with custom checks
checkov -d terraform/ --external-checks-dir custom_checks/
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Checkov reports too many findings | Default rules are comprehensive | Use `--skip-check` for accepted risks; document rationale |
| tfsec not available | Merged into Trivy | Use `trivy config` instead of tfsec |
| False positive on module output | Can't resolve module variables | Scan Terraform plan JSON instead of HCL files |
| Checkov doesn't detect dynamic values | HCL-level scanning only | Generate and scan Terraform plan JSON for full coverage |
| CI/CD takes too long | Scanning all files every time | Use path filters in CI triggers; cache tool installations |

---

## Summary Table

| Tool | Best For |
|------|---------|
| **Checkov** | Most comprehensive IaC scanning; custom Python policies |
| **Trivy config** | Unified scanner (IaC + containers + code) |
| **Terrascan** | OPA/Rego-based custom policies |
| **KICS** | Widest IaC format support (15+ technologies) |
| **Plan scanning** | Catching dynamic values and module-level issues |

---

## Quick Revision Questions

1. **Why is IaC security scanning important?**
   > IaC defines cloud infrastructure in code — a single misconfiguration (public S3 bucket, open security group, unencrypted database) can expose data or create vulnerabilities. Scanning catches these before `terraform apply`, shifting security left and preventing expensive production incidents.

2. **What is the advantage of scanning Terraform plan vs HCL files?**
   > HCL scanning can't resolve variables, module outputs, or dynamic values. Scanning the Terraform plan JSON (`terraform show -json`) evaluates the actual resolved configuration that will be applied, catching issues that static HCL analysis misses.

3. **How does Checkov differ from Trivy config scanning?**
   > Checkov has 1000+ built-in policies, supports custom Python check development, and scans Terraform plans. Trivy config is part of the unified Trivy scanner (also does images, code, SBOM) and uses Rego for custom policies. Checkov has deeper IaC-specific coverage; Trivy offers tool consolidation.

4. **What are the most common IaC misconfigurations?**
   > Public S3/storage buckets, unencrypted databases and storage, overly permissive security groups (0.0.0.0/0 ingress), disabled logging/monitoring, missing encryption at rest/in transit, default credentials, and overly broad IAM policies.

5. **How should IaC scanning be integrated into CI/CD?**
   > Scan on every PR that modifies infrastructure code (path filtering). Use multiple tools (Checkov + Trivy) for coverage. Scan both HCL files and Terraform plan JSON. Upload SARIF results to GitHub Security. Block merges on CRITICAL findings; warn on HIGH.

---

[← Previous: 7.7 Runtime Protection with Falco](../07-Kubernetes-Security/07-runtime-protection-falco.md) | [Next: 8.2 Cloud Security Posture →](02-cloud-security-posture.md)

[Back to Table of Contents](../README.md)
