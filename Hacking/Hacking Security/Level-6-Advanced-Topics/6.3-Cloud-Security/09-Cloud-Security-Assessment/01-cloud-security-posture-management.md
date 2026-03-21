# Unit 9: Cloud Security Assessment — Topic 1: Cloud Security Posture Management

## Overview

**Cloud Security Posture Management (CSPM)** continuously monitors cloud infrastructure for misconfigurations, compliance violations, and security risks. CSPM tools automatically assess cloud resources against security best practices and compliance frameworks, providing visibility into the security posture across multi-cloud environments. CSPM is the foundation of proactive cloud security.

---

## 1. CSPM Fundamentals

```
WHAT IS CSPM:

  → Automated cloud configuration assessment
  → Continuous monitoring (not point-in-time)
  → Multi-cloud visibility
  → Compliance mapping
  → Remediation guidance
  → Risk prioritization

WHY CSPM IS NEEDED:
  → Cloud misconfigurations cause ~70% of breaches
  → Manual auditing doesn't scale
  → Configuration drift over time
  → Multi-cloud complexity
  → DevOps speed creates security gaps
  → Compliance requires continuous evidence

CSPM CAPABILITIES:

  ┌──────────────────────────────────────┐
  │         CSPM PLATFORM                │
  │                                      │
  │  ┌─────────────────────────────────┐ │
  │  │ ASSET DISCOVERY                 │ │
  │  │ What resources exist?           │ │
  │  └─────────────────────────────────┘ │
  │  ┌─────────────────────────────────┐ │
  │  │ CONFIGURATION ASSESSMENT        │ │
  │  │ Are resources properly configured│ │
  │  └─────────────────────────────────┘ │
  │  ┌─────────────────────────────────┐ │
  │  │ COMPLIANCE MONITORING           │ │
  │  │ Do we meet regulatory standards?│ │
  │  └─────────────────────────────────┘ │
  │  ┌─────────────────────────────────┐ │
  │  │ RISK PRIORITIZATION             │ │
  │  │ What should we fix first?       │ │
  │  └─────────────────────────────────┘ │
  │  ┌─────────────────────────────────┐ │
  │  │ REMEDIATION                     │ │
  │  │ How to fix and auto-remediate?  │ │
  │  └─────────────────────────────────┘ │
  └──────────────────────────────────────┘
```

---

## 2. Cloud-Native CSPM

```
CLOUD PROVIDER CSPM:

AWS SECURITY HUB:
  → Aggregates findings from multiple services
  → AWS Foundational Security Best Practices
  → CIS AWS Foundations Benchmark
  → PCI DSS compliance checks
  → Automated remediation with EventBridge
  → Cross-account, cross-region
  
  Integrations:
  → GuardDuty (threat detection)
  → Inspector (vulnerability scanning)
  → Macie (data discovery)
  → IAM Access Analyzer
  → Config (configuration compliance)
  → Firewall Manager
  
  # Enable Security Hub
  aws securityhub enable-security-hub \
    --enable-default-standards
  
  # Get findings
  aws securityhub get-findings \
    --filters '{"SeverityLabel":[{"Value":"CRITICAL","Comparison":"EQUALS"}]}'

AZURE DEFENDER FOR CLOUD:
  → Secure Score (0-100%)
  → Security recommendations
  → Regulatory compliance dashboard
  → Attack path analysis
  → Cloud security graph
  
  Standards:
  → Azure Security Benchmark
  → CIS Microsoft Azure
  → NIST 800-53
  → PCI DSS
  → ISO 27001

GCP SECURITY COMMAND CENTER:
  → Security Health Analytics
  → Vulnerability scanning
  → Threat detection
  → Asset inventory
  → Compliance monitoring
  → Attack path simulation (Premium)
```

---

## 3. Third-Party CSPM

```
MULTI-CLOUD CSPM TOOLS:

OPEN SOURCE:
  SCOUTSUITE:
  → Multi-cloud security auditing
  → AWS, Azure, GCP, Alibaba, Oracle
  → HTML report with findings
  → Rule-based assessment
  → Open source (Python)
  
  # Install and run
  pip install scoutsuite
  scout aws --report-dir ./report
  scout azure --report-dir ./report
  scout gcp --report-dir ./report

  PROWLER:
  → AWS and Azure security assessment
  → CIS Benchmark checks
  → 300+ checks for AWS
  → CLI-based
  → Machine-readable output
  
  # Run Prowler
  prowler aws
  prowler aws -c check11 -c check12
  prowler aws --compliance cis_2.0_aws

  CLOUDSPLOIT:
  → Multi-cloud scanning
  → Open source core
  → AWS, Azure, GCP, Oracle
  → Remediation steps included

  STEAMPIPE:
  → SQL-based cloud querying
  → Multi-cloud support
  → Compliance mods (CIS, NIST, PCI)
  → Real-time queries
  
  # Query open S3 buckets
  steampipe query "select name, acl 
    from aws_s3_bucket 
    where block_public_acls = false"

COMMERCIAL:
  → Prisma Cloud (Palo Alto)
  → Wiz
  → Orca Security
  → Lacework
  → Check Point CloudGuard
  → Qualys CloudView
  → Rapid7 InsightCloudSec
  → CrowdStrike Cloud Security
```

---

## 4. CSPM Implementation

```
IMPLEMENTING CSPM:

PHASE 1: DISCOVERY
  → Connect all cloud accounts
  → Inventory all resources
  → Identify shadow IT
  → Map resource relationships
  → Baseline current state

PHASE 2: ASSESSMENT
  → Enable security checks
  → Run initial scan
  → Review findings by severity
  → Identify quick wins
  → Prioritize by risk

PHASE 3: REMEDIATION
  → Fix critical findings first
  → Automate common remediations
  → Create exception process
  → Track remediation progress
  → Implement guardrails

PHASE 4: CONTINUOUS MONITORING
  → Real-time configuration monitoring
  → Alert on new findings
  → Track posture over time
  → Regular compliance reporting
  → Integration with ticketing

METRICS:
  → Total findings by severity
  → Mean time to remediate (MTTR)
  → Compliance score trend
  → Number of critical findings
  → Resources assessed coverage
  → Exception count and justification

COMMON FINDINGS ACROSS CLOUDS:
  Category           | Finding
  Storage            | Public buckets/containers
  Network            | SSH/RDP open to internet
  IAM                | Root/admin without MFA
  Encryption         | Unencrypted storage volumes
  Logging            | Audit logging disabled
  Access             | Overpermissioned accounts
  Database           | Publicly accessible databases
  Secrets            | Hardcoded credentials
```

---

## 5. Assessment Methodology

```
CSPM ASSESSMENT CHECKLIST:

  [ ] CSPM tool deployed across all cloud accounts
  [ ] All accounts/subscriptions/projects connected
  [ ] Security standards enabled (CIS, NIST, etc.)
  [ ] Critical findings addressed promptly
  [ ] Remediation SLAs defined:
      - Critical: 24 hours
      - High: 7 days
      - Medium: 30 days
      - Low: 90 days
  [ ] Auto-remediation for common issues
  [ ] Exception process documented
  [ ] Regular posture review meetings
  [ ] Executive reporting established
  [ ] Multi-cloud coverage verified
  [ ] Integration with SIEM/ticketing

REPORTING:
  Executive Dashboard:
  → Overall security score
  → Trend over time (improving/declining)
  → Top risk areas
  → Compliance status
  → Remediation velocity

  Operational Report:
  → New findings since last report
  → Open findings by severity
  → Remediation progress
  → Exception requests
  → Configuration drift events
```

---

## Summary Table

| Tool | Type | Clouds | Key Feature |
|------|------|--------|-------------|
| AWS Security Hub | Native | AWS | Finding aggregation |
| Defender for Cloud | Native | Azure | Secure Score |
| SCC | Native | GCP | Health Analytics |
| ScoutSuite | Open Source | Multi | HTML reports |
| Prowler | Open Source | AWS/Azure | 300+ checks |
| Prisma Cloud | Commercial | Multi | Full CNAPP |
| Wiz | Commercial | Multi | Agentless scanning |

---

## Revision Questions

1. What is CSPM and why is it necessary?
2. How do cloud-native CSPM tools compare to third-party solutions?
3. What are the phases of implementing CSPM?
4. What metrics should be tracked for security posture?
5. What are the most common cloud misconfigurations found by CSPM?

---

*Previous: None (First topic in this unit) | Next: [02-configuration-assessment.md](02-configuration-assessment.md)*

---

*[Back to README](../README.md)*
