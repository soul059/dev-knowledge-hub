# Unit 6: Cloud Security Certifications — Topic 1: AWS Security Specialty

## Overview

The **AWS Certified Security — Specialty** certification validates expertise in securing AWS cloud environments. As the leading cloud provider, AWS security skills are in extremely high demand. This topic covers the exam structure, domains, key services, and preparation strategy.

---

## 1. Exam Specifications

```
AWS SECURITY SPECIALTY:

  Exam Code:       SCS-C02
  Questions:       65
  Question Types:  MCQ + Multiple response
  Duration:        170 minutes
  Passing Score:   750/1000 (scaled)
  Cost:            $300 USD
  Prerequisites:   Recommended:
                   - 5 years IT security experience
                   - 2 years hands-on AWS experience
                   - AWS Solutions Architect Associate
  Validity:        3 years
  Delivery:        Pearson VUE / PSI

EXAM DOMAINS:
  Domain                              | Weight
  1. Threat Detection & IR            | 14%
  2. Security Logging & Monitoring    | 18%
  3. Infrastructure Security          | 20%
  4. Identity & Access Management     | 16%
  5. Data Protection                  | 18%
  6. Management & Security Governance | 14%

WEIGHT VISUALIZATION:
  D1 Threat Detection  [███████░░░░░░░░░░░░░] 14%
  D2 Logging/Monitor   [█████████░░░░░░░░░░░] 18%
  D3 Infrastructure    [██████████░░░░░░░░░░] 20%  ← Heaviest
  D4 IAM               [████████░░░░░░░░░░░░] 16%
  D5 Data Protection   [█████████░░░░░░░░░░░] 18%
  D6 Governance        [███████░░░░░░░░░░░░░] 14%
```

---

## 2. Key AWS Security Services

```
AWS SECURITY SERVICES TO KNOW:

IDENTITY & ACCESS:
  → IAM: Users, groups, roles, policies
  → IAM Identity Center (SSO): Centralized access
  → Organizations: Multi-account management
  → STS: Temporary security credentials
  → Cognito: User authentication for apps
  → Directory Service: Managed AD

DETECTION & MONITORING:
  → CloudTrail: API activity logging
  → CloudWatch: Metrics, logs, alarms
  → GuardDuty: Threat detection (ML-based)
  → Inspector: Vulnerability scanning
  → Detective: Security investigation
  → Security Hub: Central findings dashboard
  → Macie: Sensitive data discovery (S3)

NETWORK SECURITY:
  → VPC: Virtual Private Cloud
  → Security Groups: Instance-level firewall
  → NACLs: Subnet-level firewall
  → WAF: Web Application Firewall
  → Shield: DDoS protection
  → Firewall Manager: Central firewall mgmt
  → Network Firewall: VPC traffic filtering

ENCRYPTION & DATA PROTECTION:
  → KMS: Key Management Service
  → CloudHSM: Hardware security modules
  → ACM: Certificate Manager
  → Secrets Manager: Secret rotation
  → Parameter Store: Config/secret storage

COMPLIANCE:
  → Config: Resource compliance tracking
  → Artifact: Compliance reports
  → Audit Manager: Continuous auditing
  → Control Tower: Multi-account governance
```

---

## 3. Core Concepts

```
IAM DEEP DIVE:

IAM POLICIES:
  → Identity-based: Attached to user/role/group
  → Resource-based: Attached to resource
  → Permission boundary: Maximum permissions
  → SCP: Service Control Policy (Organizations)
  → Session policy: Temporary restrictions
  
  EVALUATION LOGIC:
  1. Explicit DENY → always wins
  2. SCP allows? → must allow
  3. Permission boundary? → must allow
  4. Identity policy? → must allow
  5. Resource policy? → can grant access
  → Default: IMPLICIT DENY

  BEST PRACTICES:
  → Use roles, not long-term credentials
  → Apply least privilege
  → Enable MFA for all users
  → Rotate access keys regularly
  → Use policy conditions (IP, MFA, time)
  → Review access with IAM Access Analyzer

ENCRYPTION AT REST:
  → S3: SSE-S3, SSE-KMS, SSE-C
  → EBS: KMS-managed keys
  → RDS: KMS encryption
  → DynamoDB: AWS-owned or customer-managed
  → Default encryption: Enable everywhere

ENCRYPTION IN TRANSIT:
  → TLS for all AWS API calls
  → VPN for site-to-site
  → ACM for certificate management
  → CloudFront HTTPS
  → ALB/NLB TLS termination
```

---

## 4. Incident Response on AWS

```
AWS INCIDENT RESPONSE:

IR FRAMEWORK:
  Preparation → Detection → Containment →
  Eradication → Recovery → Lessons Learned

AWS-SPECIFIC IR:
  Detection:
  → GuardDuty alerts
  → CloudTrail anomalies
  → CloudWatch alarms
  → Security Hub findings
  → VPC Flow Logs analysis

  Containment:
  → Isolate instances (security group change)
  → Disable compromised credentials
  → Snapshot EBS for forensics
  → Block malicious IPs (WAF/NACL)
  → Enable enhanced logging

  Investigation:
  → CloudTrail: Who did what, when
  → VPC Flow Logs: Network traffic
  → S3 access logs: Data access
  → GuardDuty: Threat intelligence
  → Athena: Query logs at scale
  → Detective: Visualize relationships

  Eradication:
  → Terminate compromised instances
  → Rotate all credentials
  → Patch vulnerabilities
  → Update security groups

  Recovery:
  → Launch clean instances from known good AMI
  → Restore from clean backups
  → Verify security controls
  → Monitor closely post-recovery

AUTOMATION:
  → EventBridge: Trigger on events
  → Lambda: Automated response
  → Step Functions: IR workflows
  → Systems Manager: Automated remediation
  → Example: GuardDuty → EventBridge → 
    Lambda → Isolate instance
```

---

## 5. Study Resources

```
PREPARATION RESOURCES:

OFFICIAL AWS:
  → AWS Skill Builder (free + paid)
  → AWS Security Specialty exam guide
  → AWS Whitepapers:
    - AWS Security Best Practices
    - AWS Well-Architected (Security Pillar)
    - AWS Shared Responsibility Model
  → AWS re:Invent security sessions (YouTube)

COURSES:
  1. A Cloud Guru / Pluralsight
     → Complete security specialty course
     → Hands-on labs
     → Practice exams
  
  2. Tutorials Dojo (Jon Bonso)
     → Practice exams (highest rated)
     → Detailed explanations
     → Cheat sheets
  
  3. Stephane Maarek (Udemy)
     → Clear explanations
     → Well-structured course
     → Wait for Udemy sales

HANDS-ON PRACTICE:
  → AWS Free Tier account
  → Build security architectures
  → Configure IAM policies
  → Set up GuardDuty + Security Hub
  → Practice incident response
  → Create VPCs with security layers
  → Implement encryption everywhere

STUDY TIMELINE (8-12 weeks):
  Week 1-2:  IAM deep dive
  Week 3-4:  Infrastructure security
  Week 5-6:  Data protection + encryption
  Week 7-8:  Logging + monitoring
  Week 9-10: Detection + IR
  Week 11:   Governance + review
  Week 12:   Practice exams

CAREER IMPACT:
  → Salary premium: $10,000-$20,000+
  → High demand in cloud-first organizations
  → Pairs well with: CISSP, CCSP
  → Role: Cloud Security Architect/Engineer
```

---

## Summary Table

| Domain | Weight | Key Services |
|--------|--------|-------------|
| Threat Detection & IR | 14% | GuardDuty, Detective |
| Logging & Monitoring | 18% | CloudTrail, CloudWatch |
| Infrastructure | 20% | VPC, WAF, Shield |
| IAM | 16% | IAM, Organizations, STS |
| Data Protection | 18% | KMS, ACM, Macie |
| Governance | 14% | Config, Control Tower |

---

## Revision Questions

1. What are the six domains of the AWS Security Specialty exam?
2. How does IAM policy evaluation work (priority order)?
3. What AWS services are used for threat detection?
4. How do you perform incident containment on AWS?
5. What is the shared responsibility model and how does it apply?

---

*Previous: None (First topic in this unit) | Next: [02-azure-security-engineer.md](02-azure-security-engineer.md)*

---

*[Back to README](../README.md)*
