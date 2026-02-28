# Chapter 8.2: Cloud Security Posture Management (CSPM)

## Overview

CSPM continuously monitors cloud environments (AWS, Azure, GCP) for misconfigurations, compliance violations, and security risks. While IaC scanning checks *before* deployment, CSPM checks *after* — catching drift, manual changes, and runtime misconfigurations.

---

## CSPM Architecture

```
CSPM — CONTINUOUS CLOUD MONITORING:

  ┌──────────────────────────────────────────────────────┐
  │                 CLOUD ENVIRONMENTS                    │
  │                                                       │
  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
  │  │   AWS    │  │  Azure   │  │   GCP    │           │
  │  └────┬─────┘  └────┬─────┘  └────┬─────┘           │
  │       │              │              │                 │
  └───────┼──────────────┼──────────────┼─────────────────┘
          │              │              │
          ▼              ▼              ▼
  ┌──────────────────────────────────────────────────────┐
  │                  CSPM PLATFORM                        │
  │                                                       │
  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
  │  │ Asset       │  │ Policy       │  │ Compliance  │ │
  │  │ Inventory   │  │ Engine       │  │ Reporting   │ │
  │  ├─────────────┤  ├──────────────┤  ├─────────────┤ │
  │  │ Drift       │  │ Risk         │  │ Remediation │ │
  │  │ Detection   │  │ Scoring      │  │ Guidance    │ │
  │  └─────────────┘  └──────────────┘  └─────────────┘ │
  │                                                       │
  │  Outputs: Alerts, Dashboards, Compliance Reports      │
  └──────────────────────────────────────────────────────┘
```

---

## IaC Scanning vs CSPM

```
IaC SCANNING (Pre-Deploy)      vs      CSPM (Post-Deploy)

  ┌─────────────────────┐        ┌─────────────────────┐
  │ Scans Terraform,    │        │ Scans live cloud     │
  │ CloudFormation code │        │ resources via APIs   │
  │                     │        │                      │
  │ Catches:            │        │ Catches:             │
  │ • Code-level issues │        │ • Configuration drift│
  │ • Before deployment │        │ • Manual changes     │
  │                     │        │ • Runtime state      │
  │ Tools:              │        │ • Cross-resource     │
  │ Checkov, Trivy,     │        │   relationships      │
  │ tfsec               │        │                      │
  │                     │        │ Tools:               │
  │ Runs: CI/CD         │        │ AWS Config, Prowler  │
  │                     │        │ ScoutSuite, Prisma   │
  └─────────────────────┘        └─────────────────────┘

  BEST PRACTICE: Use BOTH for full coverage
```

---

## Prowler (AWS/Azure/GCP)

```bash
# Install Prowler
pip install prowler

# Full AWS security assessment
prowler aws

# Specific checks
prowler aws --checks s3_bucket_public_access \
    ec2_security_group_default_restrict_traffic \
    iam_root_access_key_present

# Compliance framework
prowler aws --compliance cis_3.0_aws
prowler aws --compliance aws_well_architected_framework_security_pillar

# Specific region
prowler aws --region us-east-1,eu-west-1

# Output formats
prowler aws --output-formats json-ocsf,html,csv

# Azure scan
prowler azure

# GCP scan
prowler gcp

# Severity filter
prowler aws --severity critical high

# Categories
prowler aws --category internet-exposed
```

### Prowler in CI/CD

```yaml
# Scheduled CSPM scan
name: Cloud Security Posture
on:
  schedule:
    - cron: '0 6 * * 1'   # Every Monday at 6am

jobs:
  prowler:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/prowler-role
          aws-region: us-east-1

      - name: Run Prowler
        run: |
          pip install prowler
          prowler aws --severity critical high \
              --compliance cis_3.0_aws \
              --output-formats json-ocsf,html \
              --output-directory prowler-results/

      - uses: actions/upload-artifact@v4
        with:
          name: prowler-report
          path: prowler-results/
```

---

## ScoutSuite (Multi-Cloud)

```bash
# Install ScoutSuite
pip install scoutsuite

# AWS scan
scout aws

# Azure scan (interactive login)
scout azure --cli

# GCP scan
scout gcp --user-account

# AWS with specific services
scout aws --services s3 ec2 iam rds lambda

# Generate HTML report
# Report is automatically generated at scoutsuite-report/

# Custom ruleset
scout aws --ruleset custom-rules.json
```

---

## AWS Config Rules

```hcl
# Terraform — AWS Config managed rules
resource "aws_config_config_rule" "s3_bucket_public_read" {
  name = "s3-bucket-public-read-prohibited"
  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }
}

resource "aws_config_config_rule" "encrypted_volumes" {
  name = "encrypted-volumes"
  source {
    owner             = "AWS"
    source_identifier = "ENCRYPTED_VOLUMES"
  }
}

resource "aws_config_config_rule" "root_mfa" {
  name = "root-account-mfa-enabled"
  source {
    owner             = "AWS"
    source_identifier = "ROOT_ACCOUNT_MFA_ENABLED"
  }
}

resource "aws_config_config_rule" "rds_encryption" {
  name = "rds-storage-encrypted"
  source {
    owner             = "AWS"
    source_identifier = "RDS_STORAGE_ENCRYPTED"
  }
}

# Custom Config Rule (Lambda-based)
resource "aws_config_config_rule" "custom_sg_check" {
  name = "custom-security-group-check"
  source {
    owner             = "CUSTOM_LAMBDA"
    source_identifier = aws_lambda_function.config_rule.arn
    source_detail {
      event_source = "aws.config"
      message_type = "ConfigurationItemChangeNotification"
    }
  }
  scope {
    compliance_resource_types = ["AWS::EC2::SecurityGroup"]
  }
}
```

---

## AWS Security Hub

```bash
# Enable Security Hub
aws securityhub enable-security-hub \
    --enable-default-standards

# Enable specific standards
aws securityhub batch-enable-standards \
    --standards-subscription-requests '[
      {"StandardsArn": "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/3.0.0"},
      {"StandardsArn": "arn:aws:securityhub:us-east-1::standards/aws-foundational-security-best-practices/v/1.0.0"}
    ]'

# Get findings
aws securityhub get-findings \
    --filters '{"SeverityLabel": [{"Value": "CRITICAL", "Comparison": "EQUALS"}]}' \
    --max-items 10

# Aggregate from multiple accounts (Organizations)
aws securityhub create-finding-aggregator \
    --region us-east-1 \
    --region-linking-mode ALL_REGIONS
```

---

## Common CSPM Findings

| Finding | Risk | Remediation |
|---------|------|-------------|
| S3 bucket publicly accessible | Data exposure | Enable public access block on all buckets |
| Security group allows 0.0.0.0/0 on port 22 | SSH brute force | Restrict to VPN/bastion CIDR ranges |
| RDS instance not encrypted | Data at rest exposure | Enable encryption (requires recreation) |
| Root account has access keys | Full account compromise | Delete root keys; use IAM roles instead |
| CloudTrail logging disabled | No audit trail | Enable CloudTrail in all regions |
| MFA not enabled for IAM users | Account takeover risk | Enforce MFA via IAM policy |
| EBS volumes unencrypted | Data exposure if volume stolen | Enable default EBS encryption per region |
| VPC flow logs disabled | No network visibility | Enable flow logs for all VPCs |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Prowler shows too many findings | Scanning all services in all regions | Filter by severity and specific services/regions |
| AWS Config rules not evaluating | Config recorder not enabled | Enable AWS Config recorder in each region |
| CSPM shows drift from IaC | Manual changes in console | Use Terraform state to detect drift; enforce CI/CD-only changes |
| ScoutSuite crashes on large accounts | Memory exhaustion | Scan specific services: `--services s3 iam ec2` |
| Security Hub cost too high | Evaluating all standards | Disable unused standards; focus on CIS + AWS Best Practices |

---

## Summary Table

| Tool | Type | Coverage |
|------|------|---------|
| **Prowler** | OSS CSPM | AWS, Azure, GCP; CIS/compliance frameworks |
| **ScoutSuite** | OSS CSPM | AWS, Azure, GCP; HTML report |
| **AWS Config** | Native | AWS resources; managed + custom rules |
| **AWS Security Hub** | Native | Aggregated findings; multi-standard compliance |
| **Azure Defender for Cloud** | Native | Azure CSPM + workload protection |
| **GCP Security Command Center** | Native | GCP assets + vulnerability findings |

---

## Quick Revision Questions

1. **What is the difference between IaC scanning and CSPM?**
   > IaC scanning checks infrastructure code (Terraform, CloudFormation) before deployment in CI/CD. CSPM checks live cloud resources after deployment via APIs. IaC scanning catches code-level misconfigs; CSPM catches drift, manual changes, and runtime state issues. Use both for full coverage.

2. **What does Prowler check for?**
   > Prowler runs 300+ checks across AWS, Azure, and GCP covering: IAM best practices, S3 security, encryption, network security, logging, monitoring, CIS Benchmark compliance, and more. It outputs findings with severity ratings and remediation guidance.

3. **Why is configuration drift dangerous?**
   > Drift occurs when cloud resources differ from their IaC definitions — usually from manual console changes. This creates untracked security gaps: a developer might open a security group for debugging and forget to close it. CSPM detects these deviations.

4. **How does AWS Security Hub aggregate security findings?**
   > Security Hub collects findings from AWS Config, GuardDuty, Inspector, Macie, Firewall Manager, and third-party tools into a centralized dashboard. It evaluates compliance against standards (CIS, AWS Best Practices) and provides a security score.

5. **What is the recommended CSPM scanning frequency?**
   > Continuous for critical environments (AWS Config triggers on changes). Weekly full assessments with Prowler for comprehensive coverage. Daily for compliance-sensitive workloads. The key is balancing coverage with alert fatigue.

---

[← Previous: 8.1 IaC Security Scanning](01-iac-security-scanning.md) | [Next: 8.3 Compliance as Code →](03-compliance-as-code.md)

[Back to Table of Contents](../README.md)
