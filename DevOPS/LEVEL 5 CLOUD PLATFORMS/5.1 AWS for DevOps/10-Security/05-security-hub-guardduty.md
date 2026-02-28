# Chapter 10.5: AWS Security Hub & GuardDuty

## Overview

**AWS Security Hub** provides a comprehensive view of your security posture by aggregating findings from multiple AWS security services and third-party tools, checking them against security standards. **Amazon GuardDuty** is an intelligent threat detection service that continuously monitors for malicious activity and unauthorized behavior using machine learning, anomaly detection, and threat intelligence.

---

## Security Hub Architecture

```
┌──── Security Hub ────────────────────────────────────────┐
│                                                           │
│  Finding Sources:                                        │
│  ┌────────────┐ ┌───────────┐ ┌──────────────────┐      │
│  │ GuardDuty  │ │ Inspector │ │ IAM Access       │      │
│  │            │ │           │ │ Analyzer         │      │
│  └─────┬──────┘ └─────┬─────┘ └────────┬─────────┘      │
│        │              │                │                  │
│  ┌─────┴──────┐ ┌─────┴─────┐ ┌───────┴────────┐       │
│  │ Macie      │ │ Firewall  │ │ Config Rules   │       │
│  │            │ │ Manager   │ │                │       │
│  └─────┬──────┘ └─────┬─────┘ └───────┬────────┘       │
│        │              │                │                  │
│        └──────────────┼────────────────┘                  │
│                       ▼                                   │
│        ┌──────────────────────────────┐                  │
│        │      SECURITY HUB           │                  │
│        │  ┌────────────────────────┐  │                  │
│        │  │ AWS Security Finding   │  │                  │
│        │  │ Format (ASFF)          │  │                  │
│        │  └────────────────────────┘  │                  │
│        │                              │                  │
│        │  Security Standards:        │                  │
│        │  ├── AWS Foundational       │                  │
│        │  ├── CIS AWS Foundations    │                  │
│        │  ├── PCI DSS               │                  │
│        │  └── NIST 800-53           │                  │
│        └──────────────┬───────────────┘                  │
│                       │                                   │
│         ┌─────────────┼─────────────┐                    │
│         ▼             ▼             ▼                    │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│   │Dashboard │  │EventBridge│  │Custom    │             │
│   │& Insights│  │Automation │  │Actions   │             │
│   └──────────┘  └──────────┘  └──────────┘             │
└───────────────────────────────────────────────────────────┘
```

---

## Security Hub Setup

```bash
# Enable Security Hub
aws securityhub enable-security-hub \
  --enable-default-standards

# Enable specific standard
aws securityhub batch-enable-standards \
  --standards-subscription-requests '[{
    "StandardsArn": "arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.4.0"
  }]'

# Get findings (critical severity)
aws securityhub get-findings \
  --filters '{
    "SeverityLabel": [{"Value": "CRITICAL", "Comparison": "EQUALS"}],
    "WorkflowStatus": [{"Value": "NEW", "Comparison": "EQUALS"}]
  }' \
  --max-items 10

# Update finding workflow status
aws securityhub batch-update-findings \
  --finding-identifiers '[{
    "Id": "arn:aws:securityhub:us-east-1:123456789012:finding/abc123",
    "ProductArn": "arn:aws:securityhub:us-east-1::product/aws/guardduty"
  }]' \
  --workflow '{"Status": "RESOLVED"}'

# Get security score
aws securityhub get-security-control-definition \
  --security-control-id "IAM.1"
```

---

## Security Standards & Controls

```
┌──── Security Standards ──────────────────────────────────┐
│                                                           │
│  Standard                        Key Checks              │
│  ──────────────────────────────  ──────────────────────  │
│  AWS Foundational Best Practices                         │
│  (FSBP)                          Root MFA enabled        │
│                                  S3 public access off    │
│                                  EBS encryption          │
│                                  CloudTrail enabled      │
│                                                           │
│  CIS AWS Foundations Benchmark   IAM password policy     │
│  (v1.4 / v3.0)                   No root access keys     │
│                                  MFA on all users        │
│                                  VPC flow logs enabled   │
│                                                           │
│  PCI DSS v3.2.1                  Encryption at rest      │
│                                  Network segmentation    │
│                                  Access logging          │
│                                                           │
│  NIST 800-53 Rev 5               Comprehensive controls  │
│                                  Federal compliance      │
│                                                           │
│  Score: percentage of controls passing                   │
│  Controls can be: PASSED / FAILED / NOT_AVAILABLE        │
└───────────────────────────────────────────────────────────┘
```

---

## GuardDuty Architecture

```
┌──── GuardDuty ───────────────────────────────────────────┐
│                                                           │
│  Data Sources (automatically analyzed):                  │
│  ┌──────────────────────────────────────────────┐       │
│  │ VPC Flow Logs   DNS Logs   CloudTrail Events │       │
│  │ S3 Data Events  EKS Audit  Lambda Network    │       │
│  │ RDS Login Activity        EBS Volumes (Malware)│      │
│  └─────────────────────┬────────────────────────┘       │
│                        │                                  │
│                        ▼                                  │
│  ┌──────────────────────────────────────────────┐       │
│  │          GuardDuty Detection Engine          │       │
│  │  ┌───────────────┐  ┌────────────────────┐   │       │
│  │  │    Machine     │  │ Threat Intel Feeds │   │       │
│  │  │    Learning    │  │ (AWS + CrowdStrike │   │       │
│  │  │    Models      │  │  + Proofpoint)     │   │       │
│  │  └───────────────┘  └────────────────────┘   │       │
│  │  ┌───────────────────────────────────────┐   │       │
│  │  │   Anomaly Detection (baseline behavior)│   │       │
│  │  └───────────────────────────────────────┘   │       │
│  └─────────────────────┬────────────────────────┘       │
│                        │                                  │
│                        ▼                                  │
│  ┌──────────────────────────────────────────────┐       │
│  │              Findings                         │       │
│  │  Severity: Low (1-3) / Medium (4-6) /        │       │
│  │            High (7-8) / Critical (9-10)      │       │
│  └─────────────────────┬────────────────────────┘       │
│                        │                                  │
│         ┌──────────────┼──────────────┐                  │
│         ▼              ▼              ▼                  │
│   EventBridge    Security Hub    S3 Export              │
│   (automate)     (aggregate)     (archive)             │
└───────────────────────────────────────────────────────────┘
```

---

## GuardDuty Setup

```bash
# Enable GuardDuty
aws guardduty create-detector \
  --enable \
  --finding-publishing-frequency FIFTEEN_MINUTES

# Enable EKS protection
aws guardduty update-detector \
  --detector-id <detector-id> \
  --features '[{
    "Name": "EKS_AUDIT_LOGS",
    "Status": "ENABLED"
  },{
    "Name": "EBS_MALWARE_PROTECTION",
    "Status": "ENABLED"
  },{
    "Name": "LAMBDA_NETWORK_LOGS",
    "Status": "ENABLED"
  },{
    "Name": "RDS_LOGIN_EVENTS",
    "Status": "ENABLED"
  }]'

# List findings
aws guardduty list-findings \
  --detector-id <detector-id> \
  --finding-criteria '{
    "Criterion": {
      "severity": { "Gte": 7 }
    }
  }'

# Get finding details
aws guardduty get-findings \
  --detector-id <detector-id> \
  --finding-ids '["finding-id-1"]'

# Add trusted IP list (won't generate findings)
aws guardduty create-ip-set \
  --detector-id <detector-id> \
  --name "TrustedIPs" \
  --format TXT \
  --location s3://my-bucket/trusted-ips.txt \
  --activate

# Create threat intel set (force generate findings)
aws guardduty create-threat-intel-set \
  --detector-id <detector-id> \
  --name "CustomThreatIntel" \
  --format TXT \
  --location s3://my-bucket/threat-ips.txt \
  --activate
```

---

## Common GuardDuty Finding Types

```
┌──── Finding Categories ──────────────────────────────────┐
│                                                           │
│  EC2 Findings:                                           │
│  ├── Recon:EC2/PortProbeUnprotectedPort                 │
│  ├── UnauthorizedAccess:EC2/SSHBruteForce               │
│  ├── CryptoCurrency:EC2/BitcoinTool.B!DNS               │
│  ├── Backdoor:EC2/DenialOfService.Tcp                   │
│  └── Trojan:EC2/DNSDataExfiltration                     │
│                                                           │
│  IAM Findings:                                           │
│  ├── UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B   │
│  ├── Recon:IAMUser/UserPermissions                      │
│  ├── Persistence:IAMUser/AnomalousBehavior              │
│  └── CredentialAccess:IAMUser/AnomalousBehavior         │
│                                                           │
│  S3 Findings:                                            │
│  ├── Policy:S3/BucketBlockPublicAccessDisabled          │
│  ├── Exfiltration:S3/AnomalousBehavior                  │
│  └── Impact:S3/MaliciousIPCaller                        │
│                                                           │
│  Kubernetes Findings:                                    │
│  ├── Execution:Kubernetes/ExecInKubeSystemPod           │
│  ├── PrivilegeEscalation:Kubernetes/PrivilegedContainer │
│  └── Discovery:Kubernetes/MaliciousIPCaller             │
│                                                           │
│  Lambda Findings:                                        │
│  ├── UnauthorizedAccess:Lambda/MaliciousIPCaller        │
│  └── CryptoCurrency:Lambda/CryptoMining                 │
└───────────────────────────────────────────────────────────┘
```

---

## Automated Remediation

```
┌──── Auto-Remediation Pipeline ───────────────────────────┐
│                                                           │
│  GuardDuty Finding                                       │
│       │                                                   │
│       ▼                                                   │
│  ┌──────────────┐                                        │
│  │ EventBridge  │──── Rule: Match finding type           │
│  └──────┬───────┘                                        │
│         │                                                 │
│    ┌────┴────┐                                           │
│    ▼         ▼                                           │
│  Lambda    Step Functions                                │
│  (simple)  (complex workflow)                            │
│    │                                                      │
│    ▼ Examples:                                            │
│  • Block IP in NACL / WAF                                │
│  • Isolate EC2 (replace SG with deny-all)               │
│  • Disable compromised IAM access keys                  │
│  • Snapshot EBS for forensics                            │
│  • Send to SNS / Slack / PagerDuty                      │
└───────────────────────────────────────────────────────────┘
```

### EventBridge Rule for GuardDuty

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{ "numeric": [">=", 7] }]
  }
}
```

### Auto-Remediate Compromised EC2 (Lambda)

```python
import boto3

ec2 = boto3.client('ec2')

def handler(event, context):
    finding = event['detail']
    finding_type = finding['type']
    
    if 'EC2' in finding_type:
        instance_id = finding['resource']['instanceDetails']['instanceId']
        
        # Create forensic security group (deny all)
        vpc_id = finding['resource']['instanceDetails']['networkInterfaces'][0]['vpcId']
        
        sg = ec2.create_security_group(
            GroupName=f'forensic-isolate-{instance_id}',
            Description='Isolate compromised instance',
            VpcId=vpc_id
        )
        
        # Remove all rules (deny everything)
        ec2.revoke_security_group_egress(
            GroupId=sg['GroupId'],
            IpPermissions=[{
                'IpProtocol': '-1',
                'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
            }]
        )
        
        # Replace instance SGs with isolation SG
        ec2.modify_instance_attribute(
            InstanceId=instance_id,
            Groups=[sg['GroupId']]
        )
        
        # Tag for investigation
        ec2.create_tags(
            Resources=[instance_id],
            Tags=[{'Key': 'SecurityStatus', 'Value': 'COMPROMISED'}]
        )
        
        return {'status': 'isolated', 'instance': instance_id}
```

---

## Multi-Account Security

```
┌──── Multi-Account Security Architecture ─────────────────┐
│                                                           │
│  Management Account                                      │
│  └── AWS Organizations                                   │
│       │                                                   │
│       ▼                                                   │
│  Security Account (Delegated Admin)                      │
│  ├── Security Hub ──── aggregates all accounts' findings │
│  ├── GuardDuty ──────── admin for all accounts           │
│  ├── Config ─────────── aggregator                       │
│  ├── Inspector ──────── delegated admin                  │
│  └── Macie ──────────── delegated admin                  │
│       │                                                   │
│       ▼                                                   │
│  Member Accounts (auto-enabled via Organizations)        │
│  ├── Account A: GuardDuty + Security Hub findings ──►    │
│  ├── Account B: GuardDuty + Security Hub findings ──►    │
│  └── Account C: GuardDuty + Security Hub findings ──►    │
│                                                           │
│  Log Archive Account                                     │
│  ├── CloudTrail (organization trail)                     │
│  ├── Config history                                      │
│  └── GuardDuty findings export                          │
└───────────────────────────────────────────────────────────┘
```

---

## Other Security Services

```
┌──── Related Security Services ───────────────────────────┐
│                                                           │
│  Service          Purpose                                │
│  ──────────────── ─────────────────────────────────────  │
│  Inspector        Vulnerability scanning (EC2, Lambda,   │
│                   ECR images). CVE detection.            │
│                                                           │
│  Macie            ML-powered discovery of sensitive      │
│                   data (PII, credentials) in S3.         │
│                                                           │
│  IAM Access       Identifies resources shared            │
│  Analyzer         externally. Validates IAM policies.    │
│                                                           │
│  Detective        Investigates findings from GuardDuty   │
│                   using graph models. Root-cause.        │
│                                                           │
│  Config           Track resource configuration changes.  │
│                   Compliance rules. Remediation.         │
│                                                           │
│  Trusted          Best practice checks: cost, security,  │
│  Advisor          performance, fault tolerance, limits.  │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| GuardDuty not generating findings | New deployment; no anomalies yet | Wait 7-14 days for baseline; use sample findings to test |
| Security Hub score low | Many controls failing | Prioritize CRITICAL findings; use automated remediation |
| Findings not aggregating | Multi-account not configured | Set delegated admin + enable auto-enable for new accounts |
| False positives in GuardDuty | Legitimate activity flagged | Add trusted IP list or suppress with archival rules |
| High finding volume | Too many low-severity findings | Use filters; focus on High/Critical; create suppression rules |
| EventBridge rule not triggering | Pattern mismatch | Verify `detail-type` is exactly "GuardDuty Finding" |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Security Hub** | Aggregates findings. Security standards (CIS, FSBP, PCI). Security score. |
| **GuardDuty** | Threat detection via ML + threat intel. Analyzes VPC Flow Logs, DNS, CloudTrail. |
| **Finding Severity** | Low (1-3), Medium (4-6), High (7-8), Critical (9-10) |
| **Auto-Remediation** | GuardDuty → EventBridge → Lambda (isolate EC2, block IP, disable keys) |
| **Multi-Account** | Organizations integration. Delegated admin in Security account. |
| **Inspector** | Vulnerability scanning. Macie: sensitive data. Detective: investigation. |

---

## Quick Revision Questions

1. **What data sources does GuardDuty analyze?**
   > VPC Flow Logs, DNS query logs, CloudTrail management & S3 data events, EKS audit logs, RDS login activity, Lambda network activity, and EBS volumes (malware scanning).

2. **Does GuardDuty need VPC Flow Logs to be enabled?**
   > No. GuardDuty accesses VPC flow log data independently from an internal data source. You don't need to enable VPC Flow Logs in your account.

3. **What is the AWS Security Finding Format (ASFF)?**
   > A standardized JSON format used by Security Hub to normalize findings from different sources (GuardDuty, Inspector, Macie, third-party) into a consistent structure.

4. **How do you automate response to GuardDuty findings?**
   > Create an EventBridge rule matching GuardDuty findings by type/severity, then trigger a Lambda function or Step Functions workflow to perform remediation (isolate instance, block IP, notify team).

5. **What is the difference between GuardDuty and Inspector?**
   > GuardDuty detects threats (unauthorized access, cryptocurrency mining, data exfiltration) using ML and threat intelligence. Inspector scans for software vulnerabilities (CVEs) in EC2 instances, Lambda functions, and ECR container images.

6. **How does Security Hub work across multiple accounts?**
   > Designate a delegated admin account via Organizations. Security Hub auto-enables in member accounts and aggregates all findings into the admin account for centralized visibility.

---

[← Previous: 10.4 WAF & Shield](04-waf-shield.md) | [Next: 11.1 Well-Architected Framework →](../11-DevOps-Best-Practices/01-well-architected-framework.md)

[← Back to README](../README.md)
