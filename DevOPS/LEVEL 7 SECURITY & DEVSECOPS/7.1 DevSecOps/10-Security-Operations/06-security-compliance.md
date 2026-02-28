# Chapter 10.6: Security Compliance

## Overview

Security compliance ensures that an organization meets regulatory, industry, and contractual security requirements. In DevSecOps, compliance is automated, continuous, and embedded into the SDLC — replacing manual audits with policy-as-code, automated evidence collection, and real-time compliance dashboards. This chapter covers major compliance frameworks, automation strategies, and audit preparation.

---

## Major Compliance Frameworks

```
COMPLIANCE FRAMEWORKS LANDSCAPE:

  REGULATORY (Government-mandated):
  ┌────────────────────────────────────────────────────┐
  │ GDPR     — EU data protection (privacy)            │
  │ HIPAA    — US healthcare data                      │
  │ SOX      — US financial reporting                  │
  │ FedRAMP  — US federal cloud services               │
  │ PCI DSS  — Payment card industry                   │
  └────────────────────────────────────────────────────┘

  INDUSTRY STANDARDS (Voluntary/Contractual):
  ┌────────────────────────────────────────────────────┐
  │ SOC 2    — Service organization controls           │
  │ ISO 27001— Information security management         │
  │ NIST CSF — Cybersecurity framework                 │
  │ CIS      — Center for Internet Security benchmarks │
  └────────────────────────────────────────────────────┘

  CLOUD-SPECIFIC:
  ┌────────────────────────────────────────────────────┐
  │ AWS Well-Architected (Security Pillar)             │
  │ Azure Security Benchmark                           │
  │ GCP Security Best Practices                        │
  └────────────────────────────────────────────────────┘
```

---

## Key Framework Requirements

### SOC 2 Trust Service Criteria

```
SOC 2 — FIVE TRUST SERVICE CRITERIA:

  ┌──────────────┐
  │  SECURITY    │  (Required — Common Criteria)
  │              │  • Access controls
  │  CC1-CC9     │  • Logical/physical access
  │              │  • System operations
  │              │  • Change management
  │              │  • Risk mitigation
  └──────────────┘
  ┌──────────────┐
  │ AVAILABILITY │  (Optional)
  │              │  • Uptime SLAs
  │  A1          │  • DR/BCP
  └──────────────┘
  ┌──────────────┐
  │ PROCESSING   │  (Optional)
  │ INTEGRITY    │  • Data accuracy
  │  PI1         │  • Error handling
  └──────────────┘
  ┌──────────────┐
  │CONFIDENTIAL- │  (Optional)
  │    ITY       │  • Data classification
  │  C1          │  • Encryption
  └──────────────┘
  ┌──────────────┐
  │   PRIVACY    │  (Optional)
  │              │  • PII handling
  │  P1-P8       │  • Consent
  └──────────────┘
```

### PCI DSS v4.0 Key Requirements

```
PCI DSS v4.0 — 12 REQUIREMENTS:

  ┌──────────────────────────────────────────────────────┐
  │ BUILD & MAINTAIN SECURE NETWORK                        │
  │  1. Install and maintain network security controls    │
  │  2. Apply secure configurations                       │
  ├──────────────────────────────────────────────────────┤
  │ PROTECT ACCOUNT DATA                                   │
  │  3. Protect stored account data (encryption)          │
  │  4. Encrypt transmission of cardholder data           │
  ├──────────────────────────────────────────────────────┤
  │ MAINTAIN VULNERABILITY MANAGEMENT                      │
  │  5. Protect against malicious software                │
  │  6. Develop and maintain secure systems               │
  ├──────────────────────────────────────────────────────┤
  │ IMPLEMENT ACCESS CONTROL                               │
  │  7. Restrict access by business need-to-know          │
  │  8. Identify users and authenticate access            │
  │  9. Restrict physical access                          │
  ├──────────────────────────────────────────────────────┤
  │ MONITOR AND TEST NETWORKS                              │
  │  10. Log and monitor all access                       │
  │  11. Test security systems regularly                  │
  ├──────────────────────────────────────────────────────┤
  │ MAINTAIN INFORMATION SECURITY POLICY                   │
  │  12. Support information security with policies       │
  └──────────────────────────────────────────────────────┘
```

---

## Compliance as Code

```
AUTOMATED COMPLIANCE PIPELINE:

  ┌───────────────────────────────────────────────────────┐
  │                                                         │
  │  Policy Definition        Continuous Checking           │
  │  ┌──────────────┐        ┌──────────────────────────┐ │
  │  │  OPA/Rego    │        │  Pipeline runs on:       │ │
  │  │  InSpec      │───────▶│  • Every commit          │ │
  │  │  Sentinel    │        │  • Scheduled (daily)     │ │
  │  │  Custom YAML │        │  • Infrastructure change │ │
  │  └──────────────┘        └───────────┬──────────────┘ │
  │                                       │                 │
  │                                       ▼                 │
  │  Evidence Collection     Compliance Dashboard          │
  │  ┌──────────────┐        ┌──────────────────────────┐ │
  │  │ Auto-generate│        │  SOC 2: 94% compliant    │ │
  │  │ screenshots  │        │  PCI DSS: 97% compliant  │ │
  │  │ Log exports  │◀───────│  ISO 27001: 91%          │ │
  │  │ Config dumps │        │                          │ │
  │  │ Scan reports │        │  [View Gaps] [Export]     │ │
  │  └──────────────┘        └──────────────────────────┘ │
  │                                                         │
  └───────────────────────────────────────────────────────┘
```

### InSpec Compliance Profiles

```ruby
# SOC 2 Access Control checks
# soc2-access-control.rb

control 'soc2-cc6.1' do
  title 'Logical Access Controls'
  desc 'Ensure MFA is enabled for all users'
  impact 1.0
  tag compliance: 'SOC 2'
  tag criteria: 'CC6.1'

  describe aws_iam_users.where(mfa_enabled: false) do
    it { should_not exist }
    its('entries.count') { should eq 0 }
  end
end

control 'soc2-cc6.2' do
  title 'User Access Reviews'
  desc 'Ensure no inactive users exist'
  impact 0.7
  tag compliance: 'SOC 2'
  tag criteria: 'CC6.2'

  describe aws_iam_users
      .where { last_sign_in_at < (Time.now - 90*24*60*60) } do
    it { should_not exist }
  end
end

control 'soc2-cc6.3' do
  title 'Encryption at Rest'
  desc 'Ensure all S3 buckets have encryption enabled'
  impact 1.0
  tag compliance: 'SOC 2'
  tag criteria: 'CC6.3'

  aws_s3_buckets.bucket_names.each do |bucket_name|
    describe aws_s3_bucket(bucket_name) do
      it { should have_default_encryption_enabled }
    end
  end
end
```

### OPA/Rego Compliance Policies

```rego
# pci-dss-policies.rego
package pci_dss

# PCI DSS Requirement 2: No default passwords
deny[msg] {
    input.resource_type == "aws_rds_instance"
    input.resource.master_password == "admin"
    msg := sprintf(
        "PCI DSS 2.1: Default password detected on RDS instance '%s'",
        [input.resource.identifier]
    )
}

# PCI DSS Requirement 3: Encrypt stored data
deny[msg] {
    input.resource_type == "aws_rds_instance"
    not input.resource.storage_encrypted
    msg := sprintf(
        "PCI DSS 3.4: RDS instance '%s' must have storage encryption enabled",
        [input.resource.identifier]
    )
}

# PCI DSS Requirement 4: Encrypt data in transit
deny[msg] {
    input.resource_type == "aws_lb_listener"
    input.resource.protocol != "HTTPS"
    msg := sprintf(
        "PCI DSS 4.1: Load balancer listener must use HTTPS, got '%s'",
        [input.resource.protocol]
    )
}

# PCI DSS Requirement 10: Enable logging
deny[msg] {
    input.resource_type == "aws_cloudtrail"
    not input.resource.is_multi_region_trail
    msg := "PCI DSS 10.2: CloudTrail must be enabled in all regions"
}
```

---

## Automated Evidence Collection

```yaml
# .github/workflows/compliance-evidence.yml
name: Compliance Evidence Collection
on:
  schedule:
    - cron: '0 6 * * 1'  # Weekly Monday 6am

jobs:
  collect-evidence:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # SOC 2 CC6.1 — Access control evidence
      - name: Collect IAM Evidence
        run: |
          mkdir -p evidence/$(date +%Y-%m-%d)
          
          # MFA status for all users
          aws iam generate-credential-report
          sleep 10
          aws iam get-credential-report \
            --query Content --output text | base64 -d \
            > evidence/$(date +%Y-%m-%d)/iam-credential-report.csv
          
          # Active access keys
          aws iam list-users | jq '.Users[].UserName' -r | \
            while read user; do
              aws iam list-access-keys --user-name "$user"
            done > evidence/$(date +%Y-%m-%d)/access-keys.json

      # SOC 2 CC7.2 — Vulnerability scanning evidence
      - name: Collect Scan Evidence
        run: |
          # Prowler compliance report
          prowler aws --compliance soc2_aws \
            --output-formats json,html \
            --output-directory evidence/$(date +%Y-%m-%d)/prowler/

      # SOC 2 CC8.1 — Change management evidence
      - name: Collect Change Management Evidence
        run: |
          # Git log as change record
          git log --since="7 days ago" --format="%H|%an|%ae|%ai|%s" \
            > evidence/$(date +%Y-%m-%d)/change-log.csv
          
          # PR merge history
          gh pr list --state merged --limit 50 --json number,title,mergedAt,author \
            > evidence/$(date +%Y-%m-%d)/merged-prs.json

      # Upload evidence to secure storage
      - name: Archive Evidence
        run: |
          tar czf evidence-$(date +%Y-%m-%d).tar.gz evidence/
          aws s3 cp evidence-$(date +%Y-%m-%d).tar.gz \
            s3://compliance-evidence-bucket/$(date +%Y)/
```

---

## Compliance Dashboard

```
COMPLIANCE MONITORING DASHBOARD:

  ┌──────────────────────────────────────────────────────┐
  │  COMPLIANCE OVERVIEW                    Last scan: 2h │
  │                                                        │
  │  SOC 2 Type II    ████████████░░░░  94%  ↑2%         │
  │  PCI DSS v4.0     █████████████░░░  97%  ─           │
  │  ISO 27001        ████████████░░░░  91%  ↑5%         │
  │  CIS AWS v3.0     ██████████░░░░░░  82%  ↑3%         │
  │  HIPAA            █████████████░░░  96%  ↑1%         │
  │                                                        │
  ├──────────────────────────────────────────────────────┤
  │  OPEN FINDINGS              TREND (30 days)           │
  │                                                        │
  │  Critical:  2  ●●                ╲  ╱                 │
  │  High:      8  ●●●●●●●●          ╲╱                  │
  │  Medium:   15  ●●●●●●●●●●●●●●●         ────         │
  │  Low:      23  ●●●●●●●●●●●●●●●●●●      trending ↓   │
  │                                                        │
  ├──────────────────────────────────────────────────────┤
  │  TOP GAPS                                              │
  │                                                        │
  │  1. 3 S3 buckets without versioning (PCI 10.5)        │
  │  2. 2 users without MFA (SOC2 CC6.1)                  │
  │  3. 5 SGs with 0.0.0.0/0 SSH (CIS 5.1)               │
  │  4. CloudTrail not in all regions (PCI 10.2)          │
  │                                                        │
  └──────────────────────────────────────────────────────┘
```

---

## Audit Preparation Checklist

```
AUDIT PREPARATION (SOC 2 Type II):

  3 MONTHS BEFORE:
  ┌────────────────────────────────────────────────────┐
  │ □ Gap assessment against trust service criteria     │
  │ □ Remediate identified gaps                         │
  │ □ Document all policies and procedures              │
  │ □ Verify evidence collection automation             │
  │ □ Review and update risk assessment                 │
  └────────────────────────────────────────────────────┘

  1 MONTH BEFORE:
  ┌────────────────────────────────────────────────────┐
  │ □ Run full compliance scan (Prowler, InSpec)        │
  │ □ Verify all controls have evidence                 │
  │ □ Conduct access review                             │
  │ □ Test incident response plan                       │
  │ □ Review vendor security assessments                │
  └────────────────────────────────────────────────────┘

  DURING AUDIT:
  ┌────────────────────────────────────────────────────┐
  │ □ Provide auditor access to evidence portal         │
  │ □ Assign dedicated audit liaison                    │
  │ □ Respond to information requests within SLA        │
  │ □ Document any exceptions with compensating controls│
  │ □ Track audit findings in real time                 │
  └────────────────────────────────────────────────────┘

  AFTER AUDIT:
  ┌────────────────────────────────────────────────────┐
  │ □ Review and accept audit report                    │
  │ □ Remediate any findings                            │
  │ □ Update controls based on recommendations          │
  │ □ Plan for next audit period                        │
  │ □ Share report with customers (if applicable)       │
  └────────────────────────────────────────────────────┘
```

---

## DevSecOps Controls Mapping

| DevSecOps Practice | SOC 2 | PCI DSS | ISO 27001 | NIST CSF |
|-------------------|-------|---------|-----------|----------|
| **Code review (PR)** | CC8.1 | Req 6 | A.14.2 | PR.IP-2 |
| **SAST scanning** | CC7.1 | Req 6.3 | A.14.2 | DE.CM-4 |
| **DAST scanning** | CC7.1 | Req 11.3 | A.14.2 | DE.CM-8 |
| **SCA/dependency scan** | CC7.1 | Req 6.3 | A.14.2 | ID.SC-2 |
| **Container scanning** | CC7.1 | Req 6.3 | A.14.2 | DE.CM-4 |
| **Secrets management** | CC6.1 | Req 3,8 | A.10.1 | PR.AC-1 |
| **IaC scanning** | CC8.1 | Req 2 | A.14.2 | PR.IP-1 |
| **Monitoring/logging** | CC7.2 | Req 10 | A.12.4 | DE.AE-3 |
| **Incident response** | CC7.3 | Req 12.10 | A.16.1 | RS.RP-1 |
| **Access control** | CC6.1-3 | Req 7,8 | A.9.1-4 | PR.AC-1 |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Compliance score drops suddenly | New resources created without controls | Enforce compliance checks in CI/CD; use service control policies to prevent non-compliant resources |
| Auditor requests evidence not collected | Gaps in automated evidence collection | Map every control to evidence source; test evidence collection before audit period |
| Multiple frameworks with overlapping controls | Different frameworks require similar controls | Use unified control mapping; implement once, map to multiple frameworks; use GRC tools |
| Teams don't understand compliance requirements | Controls are written in audit language | Translate controls into engineering requirements; create developer-friendly compliance guides |
| Compliance is seen as a blocker | Too many manual gates and approvals | Automate compliance checks in CI/CD pipeline; provide fast feedback; make compliance self-service |

---

## Summary Table

| Framework | Scope | Audit Frequency | Key DevSecOps Controls |
|-----------|-------|----------------|----------------------|
| **SOC 2** | Service organizations | Annual (Type II) | Access control, change management, monitoring |
| **PCI DSS** | Payment card data | Annual + quarterly scans | Encryption, vulnerability management, logging |
| **ISO 27001** | Information security | Annual surveillance | Risk management, access control, incident response |
| **HIPAA** | Healthcare data (US) | Self-assessment + audits | Data encryption, access control, audit trails |
| **GDPR** | Personal data (EU) | Ongoing | Data protection, consent, breach notification |
| **NIST CSF** | Cybersecurity | Self-assessment | Identify, Protect, Detect, Respond, Recover |

---

## Quick Revision Questions

1. **What is the difference between SOC 2 Type I and Type II?**
   > SOC 2 Type I evaluates the design of controls at a specific point in time — are the right controls in place? Type II evaluates the operating effectiveness of controls over a period (typically 6-12 months) — are the controls actually working consistently? Type II is more valuable and is what most customers require.

2. **How does compliance-as-code work?**
   > Compliance requirements are translated into executable policies (OPA/Rego, InSpec, Sentinel). These policies run automatically in CI/CD pipelines and on schedules to continuously verify that infrastructure, applications, and configurations meet regulatory requirements. Results generate evidence for auditors and feed compliance dashboards. This replaces periodic manual audits with continuous automated verification.

3. **How do DevSecOps practices map to compliance frameworks?**
   > SAST/DAST/SCA scanning satisfies PCI DSS Req 6 (secure development) and SOC 2 CC7.1 (vulnerability management). Secrets management maps to PCI DSS Req 3/8 and SOC 2 CC6.1. CI/CD with code review satisfies change management (SOC 2 CC8.1, PCI Req 6). Monitoring/logging satisfies PCI Req 10 and SOC 2 CC7.2.

4. **What is automated evidence collection and why is it important?**
   > Automated evidence collection uses scripts and pipelines to gather audit evidence — IAM reports, scan results, change logs, access reviews — on a schedule. It's important because manual evidence gathering is time-consuming, error-prone, and often incomplete. Automation ensures evidence is collected consistently, timestamped, and stored securely, reducing audit preparation from weeks to hours.

5. **How should an organization handle multiple compliance frameworks?**
   > Use a unified control framework that maps controls across standards (e.g., one encryption control satisfies PCI Req 3, SOC 2 CC6.3, and ISO 27001 A.10.1). Implement controls once, then map to multiple frameworks. Use GRC (Governance, Risk, Compliance) tools to manage mappings. This "comply once, certify many" approach reduces duplication and cost.

6. **What is the role of continuous compliance monitoring?**
   > Continuous monitoring replaces point-in-time audits with real-time compliance dashboards. Tools like Prowler, Steampipe, and InSpec run on schedules to check infrastructure against compliance requirements. Alerts fire when compliance drifts (new non-compliant resources, policy violations). This ensures compliance is maintained between audits, not just demonstrated during them.

---

[← Previous: 10.5 Bug Bounty Programs](05-bug-bounty-programs.md)

[Back to Table of Contents](../README.md)
