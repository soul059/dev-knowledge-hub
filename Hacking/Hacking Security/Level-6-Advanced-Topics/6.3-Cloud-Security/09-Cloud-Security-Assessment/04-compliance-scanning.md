# Unit 9: Cloud Security Assessment — Topic 4: Compliance Scanning

## Overview

**Compliance scanning** in cloud environments automates the evaluation of cloud resources against regulatory frameworks and industry standards. With regulations like PCI DSS, HIPAA, SOC 2, GDPR, and NIST requiring continuous compliance evidence, automated scanning is essential for maintaining compliance at cloud scale. This topic covers compliance frameworks, automated scanning tools, and evidence collection strategies.

---

## 1. Compliance Frameworks

```
MAJOR COMPLIANCE FRAMEWORKS:

PCI DSS (Payment Card Industry):
  → Applies to: Organizations handling card data
  → Key Requirements:
    - Network segmentation
    - Encryption of cardholder data
    - Access controls and MFA
    - Vulnerability scanning
    - Logging and monitoring
    - Penetration testing
  → Cloud-specific: Shared responsibility, PCI SSC guidance

HIPAA (Health Insurance Portability):
  → Applies to: Healthcare organizations (covered entities)
  → Key Requirements:
    - Administrative safeguards
    - Physical safeguards
    - Technical safeguards
    - Encryption of PHI
    - Access controls
    - Audit logging
  → Cloud-specific: Business Associate Agreement (BAA)

SOC 2 (Service Organization Controls):
  → Applies to: Service organizations
  → Trust Service Criteria:
    - Security
    - Availability
    - Processing Integrity
    - Confidentiality
    - Privacy
  → Cloud-specific: Shared responsibility documentation

NIST 800-53:
  → Applies to: Federal agencies and contractors
  → Control families:
    - Access Control (AC)
    - Audit (AU)
    - Configuration Management (CM)
    - Identification/Auth (IA)
    - System Protection (SC)
    - Risk Assessment (RA)
  → Cloud-specific: FedRAMP for cloud providers

GDPR (General Data Protection Regulation):
  → Applies to: Organizations processing EU data
  → Key Requirements:
    - Data protection by design
    - Data minimization
    - Right to erasure
    - Data breach notification (72h)
    - Data residency considerations
  → Cloud-specific: Data location, transfer mechanisms

CIS BENCHMARKS:
  → Industry consensus security configurations
  → Cloud-specific benchmarks (AWS, Azure, GCP)
  → Level 1 (essential) and Level 2 (enhanced)
  → Machine-readable for automation
  → Free and widely adopted
```

---

## 2. Automated Compliance Scanning

```
CLOUD-NATIVE COMPLIANCE:

AWS:
  AWS Config + Conformance Packs:
  → Pre-built compliance templates
  → CIS AWS Foundations
  → PCI DSS
  → NIST 800-53
  → Operational best practices
  
  AWS Security Hub:
  → Compliance standards dashboard
  → Automated checks
  → Continuous monitoring
  → Cross-account aggregation
  
  AWS Audit Manager:
  → Automated evidence collection
  → Pre-built frameworks
  → Custom frameworks
  → Assessment reports
  → Delegated administrator support
  
  # Enable compliance standard
  aws securityhub batch-enable-standards \
    --standards-subscription-requests '[{
      "StandardsArn": "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.4.0"
    }]'

AZURE:
  Defender for Cloud Compliance:
  → Regulatory compliance dashboard
  → Azure Security Benchmark
  → CIS, NIST, PCI, ISO, SOC 2
  → Custom initiatives
  → Export compliance reports
  
  Azure Policy:
  → Built-in compliance initiatives
  → Custom policy definitions
  → Non-compliant resource identification
  → Remediation tasks

GCP:
  Security Command Center:
  → Compliance monitoring
  → CIS GCP Benchmark
  → PCI DSS, NIST
  → Custom compliance requirements
  
  Assured Workloads:
  → Compliance-specific environments
  → FedRAMP, IL4, CJIS
  → Enforced controls
  → Restricted regions and services
```

---

## 3. Third-Party Compliance Tools

```
OPEN SOURCE:

  PROWLER:
  → prowler aws --compliance cis_2.0_aws
  → prowler aws --compliance pci_3.2.1_aws
  → prowler aws --compliance hipaa_aws
  → prowler aws --compliance nist_800_53_revision_5_aws
  → HTML, JSON, CSV output
  → CI/CD integration

  STEAMPIPE:
  → steampipe check aws_compliance.benchmark.cis_v200
  → steampipe check aws_compliance.benchmark.pci_v321
  → steampipe check aws_compliance.benchmark.nist_800_53_rev5
  → SQL-based, real-time
  → Multi-cloud benchmarks

  CLOUD CUSTODIAN:
  → Policy-as-code compliance
  → Define compliance rules in YAML
  → Auto-remediation
  → Scheduled scanning
  → Multi-cloud

  INSPEC:
  → Infrastructure testing framework
  → CIS benchmark profiles available
  → Ruby-based DSL
  → inspec exec aws-cis-benchmark

COMMERCIAL:
  → Prisma Cloud (Palo Alto): Multi-cloud compliance
  → Wiz: Agentless compliance scanning
  → Lacework: Compliance automation
  → Qualys CloudView: Cloud compliance
  → DivvyCloud (Rapid7): Policy guardrails
  → Orca Security: Agentless scanning
```

---

## 4. Evidence Collection

```
COMPLIANCE EVIDENCE:

TYPES OF EVIDENCE:
  → Configuration snapshots
  → Automated scan reports
  → Audit logs
  → Access control lists
  → Encryption status reports
  → Vulnerability scan results
  → Penetration test reports
  → Policy documents
  → Training records
  → Incident response records

AUTOMATED EVIDENCE:
  AWS Audit Manager:
  → Automatically collects evidence
  → Maps to compliance controls
  → Categories:
    - Configuration data (AWS Config snapshots)
    - User activity (CloudTrail events)
    - Compliance check (Config rules, Security Hub)
  
  # Create assessment
  aws auditmanager create-assessment \
    --name "SOC2-Assessment-2024" \
    --framework-id "FRAMEWORK_ID" \
    --assessment-reports-destination '{
      "destinationType": "S3",
      "destination": "s3://compliance-evidence-bucket"
    }'

EVIDENCE MANAGEMENT:
  → Centralized evidence repository
  → Tamper-proof storage (immutable)
  → Retention per compliance requirement
  → Version control
  → Access controls on evidence
  → Automated collection schedules
  → Gap analysis for missing evidence

CONTINUOUS COMPLIANCE:
  → Not just point-in-time assessment
  → Real-time monitoring
  → Alert on compliance drift
  → Automated remediation
  → Dashboard for compliance status
  → Regular reporting cadence
```

---

## 5. Assessment

```
COMPLIANCE SCANNING ASSESSMENT:

IMPLEMENTATION CHECKLIST:
  [ ] Applicable frameworks identified
  [ ] Cloud-native compliance tools enabled
  [ ] Compliance standards configured
  [ ] Automated scanning scheduled
  [ ] Evidence collection automated
  [ ] Non-compliant resources tracked
  [ ] Remediation SLAs defined
  [ ] Exception process documented
  [ ] Compliance reports generated regularly
  [ ] Audit-ready evidence maintained
  [ ] Cross-framework mapping completed
  [ ] Third-party audit preparation

COMMON COMPLIANCE GAPS:
  Gap                              | Frameworks
  MFA not enforced                 | PCI, HIPAA, SOC 2, NIST
  Logging incomplete               | All frameworks
  Encryption not enabled           | PCI, HIPAA, NIST
  Access reviews not conducted     | SOC 2, PCI, HIPAA
  No vulnerability scanning        | PCI, NIST
  No incident response plan        | All frameworks
  No data classification           | GDPR, HIPAA
  No penetration testing           | PCI, NIST

REMEDIATION PRIORITY:
  1. Critical compliance gaps (audit blockers)
  2. High-risk findings across frameworks
  3. Common gaps (fix once, satisfy multiple)
  4. Framework-specific requirements
  5. Enhancement opportunities

AUDIT PREPARATION:
  → 90 days before: Run full compliance scan
  → 60 days before: Remediate critical gaps
  → 30 days before: Evidence review and packaging
  → 14 days before: Internal dry-run audit
  → 7 days before: Final evidence refresh
  → Day of: Evidence repository access for auditors
```

---

## Summary Table

| Framework | Focus | Key Cloud Requirements |
|-----------|-------|----------------------|
| PCI DSS | Payment data | Encryption, segmentation, scanning |
| HIPAA | Health data | Encryption, BAA, access controls |
| SOC 2 | Service orgs | Trust criteria, monitoring |
| NIST 800-53 | Federal | Comprehensive controls |
| CIS | Best practice | Configuration benchmarks |
| GDPR | EU data | Residency, consent, erasure |

---

## Revision Questions

1. How do major compliance frameworks differ in their cloud requirements?
2. What cloud-native tools support automated compliance scanning?
3. How does AWS Audit Manager automate evidence collection?
4. What is continuous compliance and why is it important?
5. How should an organization prepare for a cloud compliance audit?

---

*Previous: [03-penetration-testing-in-cloud.md](03-penetration-testing-in-cloud.md) | Next: [05-cloud-security-tools.md](05-cloud-security-tools.md)*

---

*[Back to README](../README.md)*
