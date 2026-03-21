# Unit 9: Cloud Security Assessment — Topic 2: Configuration Assessment

## Overview

**Cloud configuration assessment** evaluates whether cloud resources are configured according to security best practices and compliance requirements. Misconfigurations are the leading cause of cloud breaches. This topic covers systematic assessment methodologies, CIS benchmarks, automated tools, and remediation strategies for identifying and fixing cloud misconfigurations.

---

## 1. Configuration Assessment Framework

```
ASSESSMENT METHODOLOGY:

  ┌────────────────────────────────────┐
  │    CONFIGURATION ASSESSMENT        │
  │                                    │
  │  1. SCOPE DEFINITION               │
  │     → Accounts, services, regions  │
  │     → Compliance standards         │
  │     → Risk tolerance               │
  │                                    │
  │  2. DATA COLLECTION                │
  │     → API-based configuration read │
  │     → Resource inventory           │
  │     → Policy/permission mapping    │
  │                                    │
  │  3. EVALUATION                     │
  │     → Compare against benchmarks   │
  │     → Identify deviations          │
  │     → Risk scoring                 │
  │                                    │
  │  4. REPORTING                      │
  │     → Findings by severity         │
  │     → Remediation guidance         │
  │     → Evidence for compliance      │
  │                                    │
  │  5. REMEDIATION                    │
  │     → Fix critical issues          │
  │     → Implement guardrails         │
  │     → Verify fixes                 │
  └────────────────────────────────────┘

CIS BENCHMARKS:
  → Center for Internet Security
  → Industry-standard configuration guides
  → Available for AWS, Azure, GCP
  → Level 1: Essential security (all orgs)
  → Level 2: Enhanced security (sensitive environments)
  → Regular updates
  → Machine-readable (can automate)

KEY ASSESSMENT AREAS:
  → Identity and Access Management
  → Logging and Monitoring
  → Networking
  → Storage
  → Compute
  → Database
  → Encryption
  → Compliance-specific controls
```

---

## 2. Assessment by Service Area

```
IAM ASSESSMENT:

  CHECKS:
  → Root/admin account has MFA
  → No access keys for root account
  → Password policy meets requirements
  → Unused credentials identified and removed
  → MFA enabled for all users
  → Least privilege roles assigned
  → No inline policies (use managed)
  → Access keys rotated within 90 days
  → Service accounts least privileged
  → Cross-account access reviewed

  AWS CLI:
  # Generate credential report
  aws iam generate-credential-report
  aws iam get-credential-report --output text \
    --query Content | base64 -d

  # Check MFA
  aws iam list-users --query 'Users[?MFADevices==null]'
  
  # Check access key age
  aws iam list-access-keys --user-name user1

LOGGING ASSESSMENT:

  CHECKS:
  → CloudTrail/Activity Log/Audit Logs enabled
  → Multi-region/global logging
  → Log file encryption
  → Log file validation
  → S3/Storage bucket logging
  → VPC/NSG/VPC Flow Logs enabled
  → Alerts configured for critical events
  → Log retention meets compliance

  AWS:
  # Check CloudTrail
  aws cloudtrail describe-trails
  aws cloudtrail get-trail-status --name default
  
  # Check S3 logging
  aws s3api get-bucket-logging --bucket my-bucket

NETWORKING ASSESSMENT:

  CHECKS:
  → No SSH/RDP from 0.0.0.0/0
  → Default security groups restrictive
  → VPC flow logs enabled
  → No unused security groups
  → No unrestricted outbound rules
  → Private subnets for databases
  → WAF on public applications
  → DDoS protection enabled

  AWS:
  # Find open security groups
  aws ec2 describe-security-groups \
    --filters "Name=ip-permission.cidr,Values=0.0.0.0/0" \
    --query 'SecurityGroups[*].{ID:GroupId,Name:GroupName}'

STORAGE ASSESSMENT:

  CHECKS:
  → No public buckets/containers
  → Encryption at rest enabled
  → Versioning enabled for critical data
  → Access logging enabled
  → Lifecycle policies configured
  → MFA delete enabled for critical buckets
  → Block public access settings enabled

  AWS:
  # Check public access
  aws s3api get-public-access-block --bucket my-bucket
  
  # Check encryption
  aws s3api get-bucket-encryption --bucket my-bucket
```

---

## 3. Automated Assessment Tools

```
TOOL COMPARISON:

AWS CONFIG:
  → Continuous configuration recording
  → Managed rules for compliance
  → Custom rules (Lambda-based)
  → Conformance packs (groups of rules)
  → Auto-remediation via SSM
  
  # List Config rules
  aws configservice describe-config-rules
  
  # Evaluate compliance
  aws configservice describe-compliance-by-config-rule
  
  # Conformance pack (CIS)
  aws configservice put-conformance-pack \
    --conformance-pack-name CIS-AWS \
    --template-body file://cis-pack.yaml

AZURE POLICY:
  → Built-in policy definitions (1000+)
  → Custom policies (JSON)
  → Initiative assignments
  → Compliance dashboard
  → Auto-remediation tasks
  
  # List non-compliant resources
  az policy state list \
    --filter "complianceState eq 'NonCompliant'"

GCP ORGANIZATION POLICY:
  → Constraints at org/folder/project
  → SCC Security Health Analytics
  → Cloud Asset Inventory
  → Policy Analyzer

MULTI-CLOUD TOOLS:
  ScoutSuite:
  → pip install scoutsuite
  → scout aws --report-dir ./aws-report
  → Generates HTML report
  → Categories: danger, warning, good
  
  Prowler:
  → pip install prowler
  → prowler aws --compliance cis_2.0_aws
  → 300+ checks
  → JSON/CSV/HTML output
  
  Steampipe:
  → steampipe check aws_compliance.benchmark.cis_v200
  → SQL-based queries
  → Real-time assessment
  → Multi-cloud mods

  Cloud Custodian:
  → Policy-as-code tool
  → Define rules in YAML
  → Auto-remediation
  → Multi-cloud support
  
  # Example: Find unencrypted EBS volumes
  policies:
    - name: ebs-unencrypted
      resource: ebs
      filters:
        - type: value
          key: Encrypted
          value: false
      actions:
        - type: notify
          to: [security@company.com]
```

---

## 4. Compliance Frameworks

```
COMPLIANCE MAPPING:

CIS BENCHMARK STRUCTURE:
  1. Identity and Access Management
  2. Logging
  3. Monitoring
  4. Networking
  5. Storage
  (Numbered controls: 1.1, 1.2, 2.1, etc.)

FRAMEWORK COMPARISON:
  Control Area    | CIS | NIST 800-53 | PCI DSS | ISO 27001
  MFA             | 1.x | IA-2        | 8.3     | A.9.4
  Logging         | 2.x | AU-2, AU-3  | 10.x    | A.12.4
  Encryption      | 2.x | SC-12, SC-13| 3.4     | A.10.1
  Network         | 4.x | SC-7        | 1.x     | A.13.1
  Access Control  | 1.x | AC-2, AC-6  | 7.x     | A.9.1

COMPLIANCE EVIDENCE:
  → Automated scan reports
  → Configuration snapshots
  → Remediation records
  → Exception documentation
  → Policy documents
  → Training records
  → Audit logs

AUDIT PREPARATION:
  1. Identify applicable frameworks
  2. Map controls to cloud services
  3. Run automated assessment
  4. Document exceptions
  5. Prepare evidence packages
  6. Schedule remediation for gaps
  7. Maintain continuous compliance
```

---

## 5. Remediation and Governance

```
REMEDIATION STRATEGIES:

AUTO-REMEDIATION:
  → Fix misconfigurations automatically
  → Triggered by detection event
  → Examples:
    - Close public bucket → make private
    - Enable encryption → encrypt resource
    - Disable unused port → remove rule
  → Must test before enabling
  → Break-glass for exceptions

GUARDRAILS:
  → Prevent misconfigurations before creation
  → Service Control Policies (AWS)
  → Azure Policy deny rules
  → Organization Policy constraints (GCP)
  → Terraform Sentinel policies
  → OPA (Open Policy Agent)

INFRASTRUCTURE AS CODE SCANNING:
  → Scan Terraform/CloudFormation before deploy
  → Catch misconfigurations pre-deployment
  → Tools: tfsec, Checkov, Terrascan, KICS
  
  # Scan Terraform
  tfsec .
  checkov -d .
  terrascan scan -t terraform

REMEDIATION WORKFLOW:
  1. Detection (automated scan)
  2. Triage (severity assessment)
  3. Assignment (to responsible team)
  4. Remediation (fix or exception)
  5. Verification (re-scan)
  6. Closure (document resolution)

GOVERNANCE CHECKLIST:
  [ ] Configuration standards documented
  [ ] Assessment tools deployed
  [ ] Regular scanning schedule (daily/weekly)
  [ ] Remediation SLAs defined and tracked
  [ ] Exception process established
  [ ] IaC scanning in CI/CD pipeline
  [ ] Guardrails prevent common misconfigs
  [ ] Compliance reporting automated
  [ ] Trend tracking for improvement
  [ ] Regular governance review meetings
```

---

## Summary Table

| Tool | Type | Best For | Output |
|------|------|----------|--------|
| AWS Config | Native | AWS continuous monitoring | Compliance status |
| Azure Policy | Native | Azure governance | Policy compliance |
| GCP SCC | Native | GCP posture | Findings |
| ScoutSuite | Open Source | Multi-cloud audit | HTML report |
| Prowler | Open Source | CIS benchmark | JSON/HTML |
| Checkov | Open Source | IaC scanning | CLI findings |

---

## Revision Questions

1. What are the key areas of cloud configuration assessment?
2. How do CIS benchmarks structure their security controls?
3. What open-source tools can automate configuration assessment?
4. How can guardrails prevent misconfigurations before deployment?
5. What is the recommended remediation workflow for findings?

---

*Previous: [01-cloud-security-posture-management.md](01-cloud-security-posture-management.md) | Next: [03-penetration-testing-in-cloud.md](03-penetration-testing-in-cloud.md)*

---

*[Back to README](../README.md)*
