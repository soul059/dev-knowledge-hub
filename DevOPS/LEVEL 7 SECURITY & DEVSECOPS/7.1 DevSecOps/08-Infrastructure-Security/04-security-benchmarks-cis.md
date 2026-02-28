# Chapter 8.4: Security Benchmarks — CIS

## Overview

Center for Internet Security (CIS) Benchmarks are industry-standard hardening guides for operating systems, cloud providers, databases, containers, and Kubernetes. They provide specific, actionable configuration recommendations with scoring profiles. This chapter covers key CIS Benchmarks relevant to DevSecOps.

---

## CIS Benchmark Structure

```
CIS BENCHMARK ANATOMY:

  ┌──────────────────────────────────────────────────────┐
  │              CIS BENCHMARK DOCUMENT                    │
  │                                                        │
  │  ┌──────────────────────────────────────────────────┐ │
  │  │ Section: Network Configuration                    │ │
  │  │                                                    │ │
  │  │ Recommendation 3.1.1:                              │ │
  │  │ ┌────────────────────────────────────────────────┐│ │
  │  │ │ Title: Ensure IP forwarding is disabled        ││ │
  │  │ │                                                 ││ │
  │  │ │ Profile: Level 1 (essential)                   ││ │
  │  │ │ Scored: Yes (pass/fail)                        ││ │
  │  │ │                                                 ││ │
  │  │ │ Description: Why this matters                  ││ │
  │  │ │ Rationale: Risk if not implemented             ││ │
  │  │ │ Audit: How to check (command)                  ││ │
  │  │ │ Remediation: How to fix (command)              ││ │
  │  │ │ Impact: Side effects of applying               ││ │
  │  │ │ References: CVEs, standards                    ││ │
  │  │ └────────────────────────────────────────────────┘│ │
  │  └──────────────────────────────────────────────────┘ │
  └──────────────────────────────────────────────────────┘

  Scoring Profiles:
  ┌────────────┐  ┌────────────┐
  │  Level 1   │  │  Level 2   │
  │ Essential  │  │ Defense in │
  │ security;  │  │ depth; may │
  │ minimal    │  │ impact     │
  │ impact     │  │ performance│
  └────────────┘  └────────────┘
```

---

## Key CIS Benchmarks for DevSecOps

| Benchmark | Version | Checks | Automated Tool |
|-----------|---------|--------|---------------|
| **CIS AWS Foundations** | 3.0 | 60+ controls | Prowler, ScoutSuite |
| **CIS Azure Foundations** | 2.1 | 80+ controls | Prowler, ScoutSuite |
| **CIS GCP Foundations** | 2.0 | 60+ controls | Prowler, ScoutSuite |
| **CIS Kubernetes** | 1.8 | 100+ controls | kube-bench |
| **CIS Docker** | 1.6 | 80+ controls | Docker Bench, Dockle |
| **CIS Ubuntu Linux** | 22.04 | 200+ controls | Lynis, InSpec |
| **CIS Amazon Linux** | 2.0 | 150+ controls | Lynis, InSpec |

---

## CIS AWS Foundations Benchmark — Key Controls

```
CIS AWS FOUNDATIONS v3.0 — CRITICAL CONTROLS:

  SECTION 1: IAM
  ┌────────────────────────────────────────────────────┐
  │ 1.4  Ensure no root access keys exist              │
  │ 1.5  Ensure MFA enabled for root account           │
  │ 1.10 Ensure MFA enabled for console access         │
  │ 1.13 Ensure credentials unused 45+ days disabled   │
  │ 1.16 Ensure IAM policies attached to groups/roles  │
  │ 1.17 Ensure no inline policies on IAM users        │
  └────────────────────────────────────────────────────┘

  SECTION 2: STORAGE
  ┌────────────────────────────────────────────────────┐
  │ 2.1.1 Ensure S3 buckets deny HTTP requests         │
  │ 2.1.2 Ensure MFA Delete on S3 buckets             │
  │ 2.1.4 Ensure S3 public access is blocked           │
  │ 2.3.1 Ensure RDS encryption at rest enabled        │
  │ 2.3.2 Ensure auto minor version upgrade enabled    │
  └────────────────────────────────────────────────────┘

  SECTION 3: LOGGING
  ┌────────────────────────────────────────────────────┐
  │ 3.1  Ensure CloudTrail enabled in all regions      │
  │ 3.2  Ensure CloudTrail log file validation enabled │
  │ 3.4  Ensure CloudTrail integrated with CloudWatch  │
  │ 3.7  Ensure VPC Flow Logs enabled                  │
  └────────────────────────────────────────────────────┘

  SECTION 4: MONITORING
  ┌────────────────────────────────────────────────────┐
  │ 4.1  Ensure log metric filter: unauthorized calls  │
  │ 4.3  Ensure alarm: root account usage              │
  │ 4.4  Ensure alarm: IAM policy changes              │
  │ 4.5  Ensure alarm: CloudTrail config changes       │
  │ 4.14 Ensure alarm: VPC changes                     │
  └────────────────────────────────────────────────────┘

  SECTION 5: NETWORKING
  ┌────────────────────────────────────────────────────┐
  │ 5.1  Ensure no SG allows 0.0.0.0/0 to port 22     │
  │ 5.2  Ensure no SG allows 0.0.0.0/0 to port 3389   │
  │ 5.3  Ensure default SG restricts all traffic       │
  │ 5.4  Ensure VPC peering connections are planned     │
  └────────────────────────────────────────────────────┘
```

---

## Automating CIS Checks

### Prowler for CIS AWS

```bash
# Run full CIS v3.0 check
prowler aws --compliance cis_3.0_aws

# CIS with specific sections
prowler aws --compliance cis_3.0_aws \
    --checks cis_3_0_aws_1_4 cis_3_0_aws_1_5 cis_3_0_aws_3_1

# Output compliance report
prowler aws --compliance cis_3.0_aws \
    --output-formats html,json-ocsf,csv
```

### kube-bench for CIS Kubernetes

```bash
# CIS Kubernetes Benchmark
kube-bench run --targets=master,node

# Specific section
kube-bench run --targets=master --check=1.1,1.2

# EKS-specific
kube-bench run --benchmark=eks-1.2.0

# Example: Check 1.2.1 — anonymous auth disabled
# Audit command:
ps -ef | grep kube-apiserver | grep -- '--anonymous-auth=false'
# If not found: FAIL
# Remediation: Add --anonymous-auth=false to API server config
```

### Docker Bench for CIS Docker

```bash
# Run Docker Bench Security
docker run --rm --net host --pid host --userns host \
    --cap-add audit_control \
    -v /etc:/etc:ro \
    -v /var/lib:/var/lib:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    docker/docker-bench-security

# Key checks:
# 1.1 - Ensure Docker is up to date
# 2.1 - Ensure network traffic is restricted between containers
# 4.1 - Ensure a user for the container has been created
# 4.5 - Ensure Content trust is enabled
# 5.2 - Ensure SELinux/AppArmor profile is set
# 5.10 - Ensure memory usage is limited
# 5.12 - Ensure the container is restricted from acquiring new privileges
```

---

## CIS Hardened Images

```
CIS HARDENED IMAGES (pre-hardened OS images):

  ┌──────────────────────────────────────────────┐
  │                                                │
  │  Available in cloud marketplaces:              │
  │                                                │
  │  AWS:   CIS Amazon Linux 2 Benchmark          │
  │         CIS Ubuntu 22.04 Benchmark            │
  │                                                │
  │  Azure: CIS Microsoft Azure Foundations        │
  │         CIS Ubuntu/RHEL images                 │
  │                                                │
  │  GCP:   CIS images via Marketplace             │
  │                                                │
  │  Benefits:                                     │
  │  • Pre-configured to CIS Level 1/2            │
  │  • Regular updates from CIS                    │
  │  • Reduces hardening effort                    │
  │  • Compliance starting point                   │
  └──────────────────────────────────────────────┘
```

```hcl
# Terraform — Use CIS hardened AMI
data "aws_ami" "cis_ubuntu" {
  most_recent = true
  owners      = ["679593333241"]  # CIS AMI owner

  filter {
    name   = "name"
    values = ["CIS Ubuntu Linux 22.04*Level 1*"]
  }
}

resource "aws_instance" "hardened" {
  ami           = data.aws_ami.cis_ubuntu.id
  instance_type = "t3.medium"

  metadata_options {
    http_tokens   = "required"   # IMDSv2
    http_endpoint = "enabled"
  }
}
```

---

## Compliance Reporting

```bash
# Generate CIS compliance report with Prowler
prowler aws --compliance cis_3.0_aws \
    --output-formats html \
    --output-directory ./reports/

# Report structure:
# ├── Overall compliance score: 78%
# ├── Section 1 (IAM): 85%
# ├── Section 2 (Storage): 72%
# ├── Section 3 (Logging): 90%
# ├── Section 4 (Monitoring): 65%
# └── Section 5 (Networking): 80%
#
# Each finding includes:
# - Check ID and title
# - Status: PASS / FAIL / MANUAL
# - Affected resource ARN
# - Remediation steps
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| CIS score is very low initially | Default cloud configs are insecure | Prioritize Level 1 first; use CIS hardened images for new resources |
| Some checks show MANUAL | Can't be automated (process-based) | Document manual checks; create manual review schedule |
| kube-bench can't run on managed K8s | No access to control plane nodes | Use managed-specific benchmark (`--benchmark eks`); focus on worker node checks |
| Docker Bench false positives | Checks don't apply to your environment | Document exceptions; skip non-applicable checks with rationale |
| Conflicting CIS recommendations | Different versions or platforms | Always use latest version for your exact OS/platform version |

---

## Summary Table

| Benchmark | Target | Automated Tool |
|-----------|--------|---------------|
| **CIS AWS Foundations** | AWS account configuration | Prowler, ScoutSuite, AWS Security Hub |
| **CIS Kubernetes** | K8s cluster configuration | kube-bench |
| **CIS Docker** | Docker host and daemon | Docker Bench, Dockle |
| **CIS Linux** | OS hardening | Lynis, InSpec |
| **CIS Hardened Images** | Pre-configured OS AMIs | Cloud marketplace |

---

## Quick Revision Questions

1. **What are CIS Benchmarks?**
   > CIS Benchmarks are consensus-based, best-practice security configuration guides developed by the Center for Internet Security. They provide specific, actionable hardening recommendations for operating systems, cloud providers, containers, databases, and applications. Each recommendation includes audit commands and remediation steps.

2. **What is the difference between CIS Level 1 and Level 2?**
   > Level 1 is essential security hardening with minimal performance impact — suitable for all systems. Level 2 provides defense-in-depth with stricter controls that may impact functionality or performance. Start with Level 1; apply Level 2 for high-security environments.

3. **How can CIS compliance be automated?**
   > Use Prowler for cloud CIS (AWS/Azure/GCP), kube-bench for Kubernetes CIS, Docker Bench for Docker CIS, and Lynis/InSpec for OS CIS. Run these tools in CI/CD pipelines and on schedules. Generate compliance reports automatically and track scores over time.

4. **What are CIS hardened images?**
   > Pre-configured virtual machine images (AMIs, VM images) that have CIS Benchmark recommendations already applied. Available in cloud marketplaces from CIS. They provide a secure baseline, reducing manual hardening effort and ensuring consistent compliance across instances.

5. **What are the most critical CIS AWS controls?**
   > Root account security (no access keys, MFA enabled), S3 public access blocked, CloudTrail enabled in all regions, security groups restricting SSH/RDP access, IAM policies using least privilege (no wildcards), encryption enabled on RDS/EBS/S3, and VPC Flow Logs enabled.

---

[← Previous: 8.3 Compliance as Code](03-compliance-as-code.md) | [Next: 8.5 Infrastructure Security Tools →](05-infrastructure-security-tools.md)

[Back to Table of Contents](../README.md)
