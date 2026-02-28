# Chapter 8.3: Compliance as Code

## Overview

Compliance as Code automates the verification of regulatory and security requirements by encoding policies into machine-readable formats. Instead of manual audits and spreadsheets, compliance checks run automatically in CI/CD pipelines, providing continuous assurance.

---

## Traditional vs Compliance as Code

```
TRADITIONAL COMPLIANCE:

  Quarterly Audit → Manual Checks → Spreadsheet → PDF Report
  ┌───────────┐   ┌────────────┐   ┌──────────┐   ┌────────┐
  │ Auditor   │──▶│ Manual     │──▶│ Excel    │──▶│ Report │
  │ arrives   │   │ screenshots│   │ tracking │   │ (stale │
  │ every 90  │   │ & checks   │   │          │   │  next  │
  │ days      │   │            │   │          │   │  day!) │
  └───────────┘   └────────────┘   └──────────┘   └────────┘

  COMPLIANCE AS CODE:

  Continuous → Automated Checks → Real-time Dashboard → Evidence
  ┌───────────┐   ┌────────────┐   ┌──────────┐   ┌────────┐
  │ Policy as │──▶│ Automated  │──▶│ Live     │──▶│ Audit  │
  │ code in   │   │ scanning   │   │ dashboard│   │ trail  │
  │ Git repo  │   │ every PR / │   │ always   │   │ always │
  │ (Rego/    │   │ deploy     │   │ current  │   │ ready  │
  │  Python)  │   │            │   │          │   │        │
  └───────────┘   └────────────┘   └──────────┘   └────────┘
```

---

## Common Compliance Frameworks

| Framework | Scope | Key Requirements |
|-----------|-------|-----------------|
| **SOC 2** | SaaS / Cloud services | Access control, encryption, monitoring, incident response |
| **PCI DSS** | Payment card data | Network segmentation, encryption, access logging, vulnerability scanning |
| **HIPAA** | Healthcare data | PHI protection, access controls, audit logging, encryption |
| **GDPR** | EU personal data | Data protection, consent, right to erasure, breach notification |
| **CIS Benchmarks** | Infrastructure | OS, cloud, K8s, database hardening baselines |
| **NIST 800-53** | US government | Comprehensive security controls (over 1000) |
| **ISO 27001** | Information security | ISMS framework; risk-based controls |

---

## Open Policy Agent (OPA)

```
OPA ARCHITECTURE:

  ┌──────────────────────────────────────────────────┐
  │                   OPA ENGINE                      │
  │                                                    │
  │  Input (JSON) ──▶ Rego Policy ──▶ Decision (JSON) │
  │                                                    │
  │  "Is this S3     "S3 buckets      "DENY:          │
  │   config         must have         encryption      │
  │   compliant?"    encryption"       not enabled"    │
  └──────────────────────────────────────────────────┘
```

### Rego Policies

```rego
# policy/s3.rego — S3 compliance checks

package aws.s3

# Deny S3 buckets without encryption
deny[msg] {
    input.resource_type == "aws_s3_bucket"
    not input.values.server_side_encryption_configuration
    msg := sprintf("S3 bucket '%s' must have encryption enabled", [input.name])
}

# Deny S3 buckets without versioning
deny[msg] {
    input.resource_type == "aws_s3_bucket"
    not input.values.versioning[_].enabled == true
    msg := sprintf("S3 bucket '%s' must have versioning enabled", [input.name])
}

# Deny S3 buckets without public access block
deny[msg] {
    input.resource_type == "aws_s3_bucket"
    not input.values.block_public_acls == true
    msg := sprintf("S3 bucket '%s' must block public access", [input.name])
}
```

```rego
# policy/iam.rego — IAM compliance

package aws.iam

# Deny IAM policies with wildcard actions
deny[msg] {
    input.resource_type == "aws_iam_policy"
    statement := input.values.policy.Statement[_]
    statement.Effect == "Allow"
    statement.Action[_] == "*"
    msg := sprintf("IAM policy '%s' must not use wildcard actions", [input.name])
}

# Require MFA for console access
deny[msg] {
    input.resource_type == "aws_iam_user"
    not input.values.mfa_enabled
    msg := sprintf("IAM user '%s' must have MFA enabled", [input.name])
}
```

---

## Conftest (OPA for CI/CD)

```bash
# Install conftest
brew install conftest

# Test Terraform files against policies
conftest test terraform/main.tf --policy policy/

# Test Kubernetes manifests
conftest test k8s/deployment.yaml --policy policy/k8s/

# Test Dockerfiles
conftest test Dockerfile --policy policy/docker/

# Multiple formats
conftest test --output json terraform/ --policy policy/
conftest test --output table terraform/ --policy policy/

# Pull policies from remote
conftest pull oci://registry.company.com/policies:latest
conftest test terraform/ --policy policy/
```

### Conftest Policies for Kubernetes

```rego
# policy/k8s/deployment.rego
package k8s.deployment

# Require resource limits
deny[msg] {
    container := input.spec.template.spec.containers[_]
    not container.resources.limits
    msg := sprintf("Container '%s' must have resource limits", [container.name])
}

# Deny privileged containers
deny[msg] {
    container := input.spec.template.spec.containers[_]
    container.securityContext.privileged == true
    msg := sprintf("Container '%s' must not be privileged", [container.name])
}

# Require images from approved registry
deny[msg] {
    container := input.spec.template.spec.containers[_]
    not startswith(container.image, "ghcr.io/company/")
    not startswith(container.image, "registry.company.com/")
    msg := sprintf("Container '%s' uses unapproved image registry: %s",
                   [container.name, container.image])
}

# Require non-root
deny[msg] {
    container := input.spec.template.spec.containers[_]
    not container.securityContext.runAsNonRoot == true
    msg := sprintf("Container '%s' must run as non-root", [container.name])
}
```

---

## InSpec (Infrastructure Compliance Testing)

```ruby
# controls/aws_s3.rb — InSpec compliance tests

control 'aws-s3-1' do
  impact 1.0
  title 'S3 buckets must be encrypted'
  desc 'All S3 buckets should have server-side encryption enabled'

  aws_s3_buckets.bucket_names.each do |bucket_name|
    describe aws_s3_bucket(bucket_name: bucket_name) do
      it { should have_default_encryption_enabled }
    end
  end
end

control 'aws-s3-2' do
  impact 1.0
  title 'S3 buckets must not be public'

  aws_s3_buckets.bucket_names.each do |bucket_name|
    describe aws_s3_bucket(bucket_name: bucket_name) do
      it { should_not be_public }
    end
  end
end

control 'aws-s3-3' do
  impact 0.7
  title 'S3 buckets must have versioning'

  aws_s3_buckets.bucket_names.each do |bucket_name|
    describe aws_s3_bucket(bucket_name: bucket_name) do
      it { should have_versioning_enabled }
    end
  end
end
```

```bash
# Run InSpec against AWS
inspec exec controls/ -t aws://us-east-1

# Output as JSON
inspec exec controls/ -t aws://us-east-1 --reporter json:results.json

# Use compliance profiles
inspec exec https://github.com/dev-sec/cis-aws-foundations-baseline -t aws://
```

---

## Compliance Pipeline

```yaml
# Complete compliance-as-code pipeline
name: Compliance Checks
on:
  pull_request:
    paths: ['terraform/**', 'k8s/**']
  schedule:
    - cron: '0 0 * * *'    # Daily compliance check

jobs:
  # 1. Policy checks on IaC
  policy-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Conftest - Terraform
        run: |
          conftest test terraform/*.tf --policy policy/terraform/
      - name: Conftest - Kubernetes
        run: |
          conftest test k8s/*.yaml --policy policy/k8s/

  # 2. Cloud compliance scan
  cloud-compliance:
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule'
    steps:
      - uses: actions/checkout@v4
      - name: Prowler CIS Compliance
        run: |
          pip install prowler
          prowler aws --compliance cis_3.0_aws \
              --output-formats json-ocsf,html \
              --severity critical high

  # 3. Generate compliance report
  report:
    needs: [policy-check, cloud-compliance]
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule'
    steps:
      - name: Generate Report
        run: |
          echo "## Compliance Report - $(date)" > report.md
          echo "### Policy Violations" >> report.md
          # Aggregate findings from previous jobs
      - uses: actions/upload-artifact@v4
        with:
          name: compliance-report
          path: report.md
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Rego policy not matching resources | Wrong input structure | Debug with `opa eval` and print input; check JSON paths |
| Conftest passes but resources non-compliant | Policy doesn't cover edge case | Add test cases; review with `conftest verify` |
| InSpec timeout on large accounts | Too many resources to check | Filter by tags/regions; run in parallel |
| Too many compliance failures | Policies applied before remediation | Implement in warn mode first; fix incrementally |
| OPA performance degradation | Complex Rego with nested loops | Optimize with indexed rules; cache partial evaluations |

---

## Summary Table

| Tool | Purpose | Language |
|------|---------|---------|
| **OPA** | General-purpose policy engine | Rego |
| **Conftest** | Test structured data against OPA policies | Rego |
| **InSpec** | Infrastructure compliance testing | Ruby DSL |
| **Prowler** | Cloud compliance scanning (CIS, SOC 2) | Python |
| **Checkov** | IaC compliance checks | Python |

---

## Quick Revision Questions

1. **What is Compliance as Code?**
   > Encoding regulatory, security, and organizational policies into machine-readable format (Rego, Ruby, Python) that can be automatically evaluated against infrastructure. This replaces manual audits with continuous automated compliance checking in CI/CD pipelines.

2. **How does OPA/Rego work for compliance?**
   > OPA takes structured input (JSON — Terraform plans, K8s manifests, cloud state) and evaluates it against Rego policies. Policies define `deny` rules that return violation messages when conditions are met. OPA returns a list of violations. Conftest wraps OPA for CLI/CI usage.

3. **What is the advantage of Compliance as Code over manual audits?**
   > Continuous vs point-in-time checking; version-controlled policies (audit trail of policy changes); automated evidence collection; consistent enforcement across all environments; faster feedback (minutes vs months); reduced human error.

4. **How should compliance policies be rolled out?**
   > Start with monitoring/warn mode — log violations without blocking. Fix existing violations incrementally. Then switch to enforce mode that blocks non-compliant resources. Exempt legacy systems temporarily with documented exceptions.

5. **What compliance frameworks can be automated with these tools?**
   > CIS Benchmarks (Prowler, Checkov, InSpec), SOC 2 controls (access, encryption, logging), PCI DSS requirements (network segmentation, encryption), HIPAA (access controls, audit logging), NIST 800-53 controls. Most compliance requirements map to technical checks.

---

[← Previous: 8.2 Cloud Security Posture](02-cloud-security-posture.md) | [Next: 8.4 Security Benchmarks (CIS) →](04-security-benchmarks-cis.md)

[Back to Table of Contents](../README.md)
