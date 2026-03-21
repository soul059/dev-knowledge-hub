# Unit 2: AWS Security — Topic 6: AWS Security Services

## Overview

AWS provides a comprehensive suite of security services covering threat detection, vulnerability management, data protection, and compliance. These managed services reduce the operational burden on security teams while providing cloud-native detection and response capabilities. This topic covers the key AWS security services and how they work together.

---

## 1. Threat Detection Services

```
AMAZON GUARDDUTY:
  → Intelligent threat detection
  → Analyzes: CloudTrail, VPC Flow Logs, DNS Logs, EKS, S3
  → Machine learning-based
  → No agents needed
  → Findings categorized by severity

  Finding Types:
  → UnauthorizedAccess: Unusual API calls
  → Recon: Port scanning, API enumeration
  → Trojan: EC2 communicating with known C2
  → CryptoCurrency: EC2 mining cryptocurrency
  → Stealth: CloudTrail disabled
  → Persistence: IAM user anomalies
  → PrivilegeEscalation: Unusual role assumption

  # Enable GuardDuty
  aws guardduty create-detector --enable
  
  # List findings
  aws guardduty list-findings --detector-id <id>

AWS SECURITY HUB:
  → Central security dashboard
  → Aggregates findings from:
    - GuardDuty
    - Inspector
    - Macie
    - Firewall Manager
    - IAM Access Analyzer
    - Third-party tools
  → Security standards compliance:
    - AWS Foundational Security Best Practices
    - CIS AWS Foundations Benchmark
    - PCI DSS
  → Security score
  → Automated remediation actions

AMAZON DETECTIVE:
  → Security investigation service
  → Automatically collects and analyzes data
  → Visualizes relationships and timelines
  → Identifies root cause
  → Integrates with GuardDuty findings
  → Graph-based analysis

AMAZON INSPECTOR:
  → Automated vulnerability assessment
  → Scans EC2, containers (ECR), Lambda
  → CVE-based vulnerability detection
  → Network reachability analysis
  → Agentless scanning (SSM-based)
  → Continuous scanning
  
  # Enable Inspector
  aws inspector2 enable --resource-types EC2 ECR LAMBDA
```

---

## 2. Data Protection Services

```
AWS KMS (Key Management Service):
  → Create and manage encryption keys
  → Integrates with most AWS services
  → AWS managed keys vs customer managed keys
  → Key policies for access control
  → Automatic key rotation (yearly)
  → Audit via CloudTrail
  
  # Create KMS key
  aws kms create-key --description "Data encryption key"
  
  # Encrypt data
  aws kms encrypt --key-id alias/my-key \
    --plaintext fileb://secret.txt
  
  # Key Policy
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::123456789012:role/AppRole"},
      "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*"
    }]
  }

AWS SECRETS MANAGER:
  → Store and rotate secrets
  → Database credentials
  → API keys
  → Connection strings
  → Automatic rotation via Lambda
  → Fine-grained access control
  
  # Store secret
  aws secretsmanager create-secret \
    --name prod/database/password \
    --secret-string "MySecretPassword123"
  
  # Retrieve secret
  aws secretsmanager get-secret-value \
    --secret-id prod/database/password

AMAZON MACIE:
  → Sensitive data discovery
  → Machine learning classification
  → Scans S3 for PII, PHI, financial data
  → Alerts on exposed sensitive data
  → Custom data identifiers
  → Automated scanning schedules

AWS CERTIFICATE MANAGER (ACM):
  → Free SSL/TLS certificates
  → Automatic renewal
  → Integration with ALB, CloudFront, API Gateway
  → No manual cert management needed
```

---

## 3. Network Security Services

```
AWS WAF (Web Application Firewall):
  → Protect web applications
  → Layer 7 filtering
  → SQL injection protection
  → XSS protection
  → Rate limiting
  → IP reputation lists
  → Bot detection
  → Custom rules
  → Managed rule groups (AWS, partners)
  
  Deployment points:
  → CloudFront
  → Application Load Balancer (ALB)
  → API Gateway
  → AppSync

AWS SHIELD:
  Shield Standard:
  → Free, automatic DDoS protection
  → Layer 3/4 protection
  → All AWS customers
  → Protects CloudFront, Route 53, ELB
  
  Shield Advanced:
  → Enhanced DDoS protection
  → Layer 7 protection
  → DDoS Response Team (DRT)
  → Cost protection during attacks
  → Real-time metrics
  → Additional cost (~$3000/month)

AWS NETWORK FIREWALL:
  → Managed stateful network firewall
  → VPC-level traffic filtering
  → IPS/IDS capabilities
  → Domain filtering
  → Suricata-compatible rules
  → Integration with Firewall Manager

AWS FIREWALL MANAGER:
  → Central firewall management
  → Manage WAF, Shield, Security Groups
  → Across multiple accounts
  → Automatic policy enforcement
  → Compliance reporting
```

---

## 4. Compliance and Governance

```
AWS CONFIG:
  → Track resource configurations
  → Record configuration changes
  → Evaluate compliance rules
  → Auto-remediation
  
  Config Rules:
  → s3-bucket-public-read-prohibited
  → ec2-instance-no-public-ip
  → iam-user-mfa-enabled
  → encrypted-volumes
  → restricted-ssh
  → rds-instance-public-access-check
  
  # Custom rule example
  aws configservice put-config-rule --config-rule '{
    "ConfigRuleName": "required-tags",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "REQUIRED_TAGS"
    },
    "InputParameters": "{\"tag1Key\":\"Environment\"}"
  }'

AWS CONTROL TOWER:
  → Multi-account governance
  → Landing zone setup
  → Guardrails (preventive + detective)
  → Account factory
  → Dashboard for compliance
  → Based on Organizations + Config + CloudTrail

AWS AUDIT MANAGER:
  → Automated audit evidence collection
  → Pre-built frameworks:
    - SOC 2
    - HIPAA
    - PCI DSS
    - GDPR
    - CIS Benchmarks
  → Custom frameworks
  → Evidence linking to controls

IAM ACCESS ANALYZER:
  → Find resources shared externally
  → Identifies: S3, IAM roles, KMS, Lambda, SQS
  → Shared with external accounts/internet
  → Generates findings for review
  → Policy validation
  → Policy generation from CloudTrail
```

---

## 5. Security Service Integration

```
INTEGRATED SECURITY ARCHITECTURE:

  ┌────────────────────────────────────────┐
  │         AWS SECURITY HUB              │
  │    (Central aggregation & scoring)     │
  ├────────┬──────────┬──────────┬─────────┤
  │GuardDuty│Inspector │  Macie  │ Config  │
  │Threats  │Vulns     │Data     │Compliance│
  ├────────┴──────────┴──────────┴─────────┤
  │           CloudTrail                    │
  │    (API logging — feeds everything)     │
  ├─────────────────────────────────────────┤
  │    CloudWatch / EventBridge             │
  │    (Alerting and automation)            │
  ├────────┬──────────┬────────────────────┤
  │  SNS   │  Lambda  │  Step Functions    │
  │ Alerts │  Auto-   │  Workflow          │
  │        │  remediate│  automation        │
  └────────┴──────────┴────────────────────┘

RECOMMENDED SERVICES TO ENABLE:
  Priority 1 (Must-Have):
  → CloudTrail (all regions)
  → GuardDuty
  → Security Hub
  → Config
  → IAM Access Analyzer
  
  Priority 2 (Should-Have):
  → Inspector
  → Macie (for S3 data)
  → WAF (for web apps)
  → Secrets Manager
  
  Priority 3 (Nice-to-Have):
  → Detective
  → Shield Advanced
  → Network Firewall
  → Audit Manager

COST CONSIDERATIONS:
  Service       | Cost Model
  CloudTrail    | Free (management events), data events extra
  GuardDuty     | Per-GB analyzed (~$4/GB first 500GB)
  Security Hub  | Per check ($0.0010/check/region)
  Config        | Per rule evaluation ($0.003/eval)
  Inspector     | Per instance/image scanned
  Macie         | Per GB scanned
  WAF           | Per web ACL + per million requests
```

---

## Summary Table

| Service | Category | Key Capability |
|---------|----------|---------------|
| GuardDuty | Threat Detection | ML-based threat finding |
| Security Hub | Aggregation | Central security dashboard |
| Inspector | Vulnerability | CVE scanning |
| Macie | Data Protection | Sensitive data discovery |
| KMS | Encryption | Key management |
| Config | Compliance | Configuration tracking |
| WAF | Network | Web app protection |
| Shield | Network | DDoS protection |

---

## Revision Questions

1. What data sources does GuardDuty analyze for threat detection?
2. How does Security Hub aggregate security findings?
3. What is the difference between Shield Standard and Shield Advanced?
4. How can AWS Config rules automate compliance checking?
5. What is the recommended priority for enabling AWS security services?

---

*Previous: [05-cloudtrail-and-cloudwatch.md](05-cloudtrail-and-cloudwatch.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
