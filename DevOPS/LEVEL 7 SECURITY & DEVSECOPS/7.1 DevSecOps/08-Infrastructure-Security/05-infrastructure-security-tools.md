# Chapter 8.5: Infrastructure Security Tools

## Overview

Infrastructure security requires a layered tooling approach covering IaC scanning, cloud posture management, VM hardening, network security, and continuous compliance monitoring. This chapter provides a comprehensive survey of tools for securing infrastructure across the DevSecOps lifecycle.

---

## Infrastructure Security Tool Landscape

```
INFRASTRUCTURE SECURITY TOOLING ECOSYSTEM:

  ┌──────────────────────────────────────────────────────┐
  │                    PRE-DEPLOYMENT                      │
  │                                                        │
  │  IaC Scanning        Policy Engines    Secret Detection│
  │  ┌──────────┐       ┌───────────┐     ┌────────────┐ │
  │  │ Checkov  │       │ OPA/Rego  │     │ TruffleHog │ │
  │  │ tfsec    │       │ Sentinel  │     │ GitLeaks   │ │
  │  │ Terrascan│       │ Kyverno   │     │ detect-    │ │
  │  │ KICS     │       │           │     │ secrets    │ │
  │  └──────────┘       └───────────┘     └────────────┘ │
  │                                                        │
  ├──────────────────────────────────────────────────────┤
  │                    DEPLOYMENT                          │
  │                                                        │
  │  Cloud Posture       K8s Security      Container Scan  │
  │  ┌──────────┐       ┌───────────┐     ┌────────────┐ │
  │  │ Prowler  │       │ kube-bench│     │ Trivy      │ │
  │  │ ScoutSte │       │ kubescape │     │ Grype      │ │
  │  │ CloudSplt│       │ Polaris   │     │ Snyk       │ │
  │  │ Steampipe│       │ Datree    │     │ Clair      │ │
  │  └──────────┘       └───────────┘     └────────────┘ │
  │                                                        │
  ├──────────────────────────────────────────────────────┤
  │                    RUNTIME                             │
  │                                                        │
  │  OS Hardening        Network Sec      SIEM/Monitoring  │
  │  ┌──────────┐       ┌───────────┐     ┌────────────┐ │
  │  │ Lynis    │       │ Nmap      │     │ ELK Stack  │ │
  │  │ OpenSCAP │       │ Calico    │     │ Wazuh      │ │
  │  │ Ansible  │       │ Cilium    │     │ Falco      │ │
  │  │ Hardening│       │ Istio     │     │ Prometheus │ │
  │  └──────────┘       └───────────┘     └────────────┘ │
  └──────────────────────────────────────────────────────┘
```

---

## IaC Scanning Tools

### Checkov

```bash
# Install
pip install checkov

# Scan Terraform
checkov -d ./terraform/ --framework terraform

# Scan CloudFormation
checkov -d ./cloudformation/ --framework cloudformation

# Scan Kubernetes manifests
checkov -d ./k8s/ --framework kubernetes

# Skip specific checks
checkov -d ./terraform/ --skip-check CKV_AWS_18,CKV_AWS_21

# Custom policies in Python
checkov -d ./terraform/ --external-checks-dir ./custom_policies/

# JUnit output for CI/CD
checkov -d ./terraform/ -o junitxml > checkov-results.xml
```

### tfsec

```bash
# Install
brew install tfsec

# Scan
tfsec ./terraform/

# With custom severity threshold
tfsec ./terraform/ --minimum-severity HIGH

# Markdown output
tfsec ./terraform/ --format markdown > tfsec-report.md
```

### KICS (Keeping Infrastructure as Code Secure)

```bash
# Run KICS via Docker
docker run -v $(pwd):/path checkmarx/kics:latest scan \
    -p /path/terraform/ \
    -o /path/results/ \
    --report-formats json,html

# Supports: Terraform, CloudFormation, Ansible,
#           Docker, Kubernetes, Helm, OpenAPI
```

---

## Cloud Security Posture Tools

### Prowler (Multi-Cloud)

```bash
# AWS scan
prowler aws

# Azure scan
prowler azure

# GCP scan
prowler gcp

# Compliance-specific checks
prowler aws --compliance cis_3.0_aws
prowler aws --compliance aws_well_architected_framework_security_pillar

# Multiple output formats
prowler aws -M html,json-ocsf,csv \
    --output-directory ./reports/

# Filter by severity
prowler aws --severity critical high

# Specific services only
prowler aws --services s3 iam ec2 rds
```

### Steampipe

```bash
# Install
brew install turbot/tap/steampipe

# Install AWS plugin
steampipe plugin install aws

# Interactive SQL queries against cloud APIs
steampipe query

# Example queries:
# Find public S3 buckets
steampipe query "SELECT name, acl
  FROM aws_s3_bucket
  WHERE bucket_policy_is_public = true"

# Find unencrypted EBS volumes
steampipe query "SELECT volume_id, encrypted
  FROM aws_ebs_volume
  WHERE encrypted = false"

# Run CIS compliance mod
steampipe check aws_compliance.benchmark.cis_v300
```

### CloudSploit

```bash
# Clone and configure
git clone https://github.com/aquasecurity/cloudsploit.git
cd cloudsploit
npm install

# Configure credentials
cp config_example.js config.js
# Edit config.js with your cloud credentials

# Run all checks
node index.js

# Run specific plugins
node index.js --plugin s3BucketAllUsersPolicy
```

---

## OS Hardening Tools

### Lynis

```bash
# Install
apt install lynis

# Full system audit
lynis audit system

# Output:
# ============================================
# Lynis Security Scan Results
# ============================================
# Hardening Index: 72 [############........]
# Tests performed: 256
# Plugins enabled: 2
#
# Sections:
#   Boot and services  [OK]
#   Kernel             [WARNING]
#   Memory and procs   [OK]
#   Users and groups   [WARNING]
#   Shells             [OK]
#   File systems       [WARNING]
#   Networking         [OK]
#   Firewalls          [OK]
#   SSH                [WARNING]
```

### OpenSCAP

```bash
# Install
yum install openscap-scanner scap-security-guide

# List available profiles
oscap info /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

# Run CIS benchmark scan
oscap xccdf eval \
    --profile xccdf_org.ssgproject.content_profile_cis \
    --results results.xml \
    --report report.html \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

# Auto-remediate
oscap xccdf eval \
    --profile cis \
    --remediate \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
```

### Ansible Hardening Playbooks

```yaml
# ansible-hardening.yml
---
- hosts: all
  become: true
  roles:
    - role: devsec.hardening.os_hardening
      vars:
        os_auth_retries: 5
        os_auth_lockout_time: 600
        os_security_users_allow: ['deploy', 'admin']
        sysctl_overwrite:
          net.ipv4.ip_forward: 0
          net.ipv4.conf.all.send_redirects: 0
          net.ipv4.conf.default.accept_source_route: 0

    - role: devsec.hardening.ssh_hardening
      vars:
        ssh_permit_root_login: "no"
        ssh_password_authentication: "no"
        ssh_max_auth_retries: 3
        ssh_allow_users: "deploy admin"
        ssh_kex:
          - curve25519-sha256
          - diffie-hellman-group-exchange-sha256
```

```bash
# Install hardening roles
ansible-galaxy install devsec.hardening

# Run hardening
ansible-playbook -i inventory ansible-hardening.yml
```

---

## CI/CD Pipeline Integration

```yaml
# .github/workflows/infra-security.yml
name: Infrastructure Security Scan
on:
  pull_request:
    paths:
      - 'terraform/**'
      - 'k8s/**'
      - 'cloudformation/**'

jobs:
  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # --- Checkov ---
      - name: Checkov IaC Scan
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: terraform/
          framework: terraform
          soft_fail: false
          output_format: sarif
          output_file_path: checkov.sarif

      # --- tfsec ---
      - name: tfsec Scan
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          working_directory: terraform/
          soft_fail: false

      # --- KICS ---
      - name: KICS Scan
        uses: Checkmarx/kics-github-action@v2
        with:
          path: .
          fail_on: high,medium

      # --- Upload SARIF ---
      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: checkov.sarif

  cloud-posture:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Prowler AWS Scan
        uses: prowler-cloud/prowler-action@v1
        with:
          provider: aws
          compliance: cis_3.0_aws
          severity: critical,high
```

---

## Tool Comparison Matrix

```
INFRASTRUCTURE SECURITY TOOL COMPARISON:

  Feature          │ Checkov │ tfsec  │ KICS   │ Prowler│ Steampipe
  ─────────────────┼─────────┼────────┼────────┼────────┼──────────
  IaC Scanning     │  ✅     │  ✅    │  ✅    │  ❌    │  ❌
  Terraform        │  ✅     │  ✅    │  ✅    │  ❌    │  ❌
  CloudFormation   │  ✅     │  ❌    │  ✅    │  ❌    │  ❌
  Kubernetes       │  ✅     │  ❌    │  ✅    │  ❌    │  ❌
  Dockerfile       │  ✅     │  ❌    │  ✅    │  ❌    │  ❌
  Helm             │  ✅     │  ❌    │  ✅    │  ❌    │  ❌
  Cloud Runtime    │  ❌     │  ❌    │  ❌    │  ✅    │  ✅
  CIS Compliance   │  ✅     │  ❌    │  ❌    │  ✅    │  ✅
  SQL Interface    │  ❌     │  ❌    │  ❌    │  ❌    │  ✅
  Custom Policies  │  ✅     │  ✅    │  ✅    │  ✅    │  ✅
  Multi-Cloud      │  ✅     │  ❌    │  ✅    │  ✅    │  ✅
  SARIF Output     │  ✅     │  ✅    │  ✅    │  ❌    │  ❌
  Free / OSS       │  ✅     │  ✅    │  ✅    │  ✅    │  ✅
```

---

## Real-World Scenario: Multi-Layer Infrastructure Scanning

```
SCENARIO: E-commerce platform security pipeline

  Developer pushes Terraform + K8s manifests
  │
  ▼
  ┌─────────────────────────────────────────────┐
  │ STAGE 1: Pre-commit                          │
  │ • tfsec local scan (fast feedback)           │
  │ • detect-secrets for embedded secrets        │
  └─────────────────┬───────────────────────────┘
                    │
                    ▼
  ┌─────────────────────────────────────────────┐
  │ STAGE 2: CI Pipeline                         │
  │ • Checkov full scan (IaC + K8s)             │
  │ • KICS for Helm charts                       │
  │ • OPA policy gate (custom rules)            │
  │ • Block on HIGH/CRITICAL findings            │
  └─────────────────┬───────────────────────────┘
                    │
                    ▼
  ┌─────────────────────────────────────────────┐
  │ STAGE 3: Post-deploy (scheduled)             │
  │ • Prowler CIS scan (daily)                  │
  │ • Steampipe drift detection                  │
  │ • kube-bench on K8s clusters                │
  │ • Lynis on EC2 instances                     │
  └─────────────────┬───────────────────────────┘
                    │
                    ▼
  ┌─────────────────────────────────────────────┐
  │ STAGE 4: Reporting                           │
  │ • Compliance dashboard (Grafana)            │
  │ • JIRA tickets for failed checks             │
  │ • Executive compliance report (monthly)     │
  └─────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many false positives in IaC scans | Overly strict defaults; policies not tuned | Create `.checkov.yaml` / `.tfsec.yaml` to skip non-applicable checks with documented rationale |
| Prowler scan takes too long | Scanning all services in large accounts | Use `--services` flag to target specific services; run full scans on schedule |
| Steampipe query errors | Plugin not installed or credentials expired | Run `steampipe plugin install aws`; refresh cloud credentials |
| OpenSCAP profile not found | Wrong content package version | Install matching `scap-security-guide` for your OS version |
| Tool output not integrating with SIEM | Incompatible output format | Use SARIF, JSON-OCSF, or CSV outputs; configure log shipping to SIEM |

---

## Summary Table

| Tool Category | Tools | Purpose |
|---------------|-------|---------|
| **IaC Scanning** | Checkov, tfsec, KICS, Terrascan | Scan infrastructure code before deployment |
| **Cloud Posture** | Prowler, ScoutSuite, Steampipe | Audit live cloud configuration against benchmarks |
| **OS Hardening** | Lynis, OpenSCAP, Ansible Hardening | Audit and harden operating system configuration |
| **K8s Security** | kube-bench, kubescape, Polaris | Scan Kubernetes cluster and manifest security |
| **Network** | Calico, Cilium, Istio | Network segmentation and zero-trust networking |
| **Secret Detection** | TruffleHog, GitLeaks, detect-secrets | Find secrets in code and IaC files |

---

## Quick Revision Questions

1. **What is Checkov and what does it scan?**
   > Checkov is an open-source static analysis tool for infrastructure as code. It scans Terraform, CloudFormation, Kubernetes manifests, Dockerfiles, Helm charts, and ARM templates for security misconfigurations using 1000+ built-in policies. It also supports custom policies in Python and YAML.

2. **How does Steampipe differ from traditional scanning tools?**
   > Steampipe provides a SQL-based interface to query live cloud infrastructure APIs as if they were database tables. Instead of running predefined checks, you write SQL queries to explore cloud resources, detect misconfigurations, and build custom compliance checks. It supports 100+ plugins for different cloud and SaaS providers.

3. **What is the recommended multi-layer scanning strategy?**
   > Pre-commit hooks for fast local scanning (tfsec, detect-secrets), CI pipeline gates with comprehensive IaC scanning (Checkov, KICS, OPA policies), post-deployment cloud posture assessment (Prowler, kube-bench) on a schedule, and continuous runtime monitoring with SIEM integration and compliance dashboards.

4. **How do you reduce false positives in IaC scanning?**
   > Create configuration files (`.checkov.yaml`, `.tfsec.yaml`) to skip non-applicable checks with documented rationale. Use inline skip annotations (`#checkov:skip=CKV_AWS_18:Reason`) for intentional exceptions. Tune severity thresholds to focus on HIGH/CRITICAL. Regularly review and update skip lists.

5. **What is OpenSCAP and how is it used?**
   > OpenSCAP is an open-source tool implementing the Security Content Automation Protocol (SCAP). It evaluates system configurations against compliance profiles (CIS, STIG, PCI-DSS) using XCCDF/OVAL content. It generates HTML reports and can auto-remediate findings. Used primarily for OS-level compliance on RHEL/CentOS systems.

6. **How can Ansible be used for infrastructure hardening?**
   > Ansible hardening roles (like `devsec.hardening`) provide idempotent playbooks that configure OS, SSH, and network settings according to security benchmarks. They can enforce password policies, disable unused services, configure firewalls, and harden SSH. Runs on schedules to maintain compliance drift prevention.

---

[← Previous: 8.4 Security Benchmarks CIS](04-security-benchmarks-cis.md) | [Next: 9.1 Secrets Challenges →](../09-Secrets-Management/01-secrets-challenges.md)

[Back to Table of Contents](../README.md)
