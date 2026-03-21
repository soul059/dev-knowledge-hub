# Unit 2: AWS Security — Topic 1: AWS Security Architecture

## Overview

**AWS security architecture** is designed around a defense-in-depth approach with multiple layers of security controls. Understanding how AWS organizes security — from global infrastructure to individual services — is essential for securing workloads, conducting assessments, and identifying misconfigurations. AWS provides a comprehensive security framework with shared responsibility, global infrastructure isolation, and hundreds of security features.

---

## 1. AWS Global Infrastructure Security

```
AWS INFRASTRUCTURE LAYERS:

  ┌─────────────────────────────────────────────┐
  │              AWS GLOBAL                     │
  │                                             │
  │  ┌─────────────────────────────────────┐    │
  │  │          REGION (e.g., us-east-1)   │    │
  │  │                                     │    │
  │  │  ┌───────────┐  ┌───────────┐       │    │
  │  │  │    AZ-1   │  │    AZ-2   │       │    │
  │  │  │           │  │           │       │    │
  │  │  │ ┌───────┐ │  │ ┌───────┐ │       │    │
  │  │  │ │ VPC   │ │  │ │ VPC   │ │       │    │
  │  │  │ │Subnet │ │  │ │Subnet │ │       │    │
  │  │  │ │EC2    │ │  │ │EC2    │ │       │    │
  │  │  │ └───────┘ │  │ └───────┘ │       │    │
  │  │  └───────────┘  └───────────┘       │    │
  │  └─────────────────────────────────────┘    │
  │                                             │
  │  Edge Locations (CloudFront, Route 53)      │
  │  Global Services (IAM, S3, CloudFront)      │
  └─────────────────────────────────────────────┘

REGIONS:
  → 30+ geographic regions worldwide
  → Physically isolated from each other
  → Data does not replicate between regions by default
  → Choose regions based on: compliance, latency, cost
  → Each region is independent for most services

AVAILABILITY ZONES (AZ):
  → 2-6 AZs per region
  → Physically separate data centers
  → Connected via low-latency links
  → Independent power, cooling, networking
  → Use multiple AZs for high availability

EDGE LOCATIONS:
  → 400+ edge locations globally
  → CloudFront CDN
  → Route 53 DNS
  → AWS Shield (DDoS protection)
  → WAF deployment points
```

---

## 2. AWS Account Structure

```
ACCOUNT SECURITY:

AWS ORGANIZATIONS:
  ┌───────────────────────────────────┐
  │     Management Account           │
  │     (Root of organization)       │
  ├───────────────┬───────────────────┤
  │   OU: Prod    │   OU: Dev        │
  │   ┌────────┐  │   ┌────────┐     │
  │   │Acct 1  │  │   │Acct 3  │     │
  │   │Acct 2  │  │   │Acct 4  │     │
  │   └────────┘  │   └────────┘     │
  ├───────────────┼───────────────────┤
  │   OU: Security│   OU: Sandbox    │
  │   ┌────────┐  │   ┌────────┐     │
  │   │Log Acct│  │   │Test    │     │
  │   │SecHub  │  │   │Accts   │     │
  │   └────────┘  │   └────────┘     │
  └───────────────┴───────────────────┘

SERVICE CONTROL POLICIES (SCP):
  → Applied at OU or account level
  → Restrict what services/actions are allowed
  → Cannot grant permissions — only restrict
  → Guardrails for the entire organization
  
  Example SCP — Deny disabling CloudTrail:
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "DenyCloudTrailDisable",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:DeleteTrail",
        "cloudtrail:StopLogging"
      ],
      "Resource": "*"
    }]
  }

ROOT ACCOUNT SECURITY:
  → Enable MFA on root account (hardware key preferred)
  → Never use root for daily operations
  → Remove root access keys
  → Monitor root account usage
  → Lock root in a secure process

BEST PRACTICES:
  → Separate accounts per environment (dev, staging, prod)
  → Centralized security account (logs, security tools)
  → SCPs as guardrails
  → Consolidated billing in management account
  → Cross-account roles for access
```

---

## 3. Network Architecture

```
VPC SECURITY ARCHITECTURE:

  ┌─────────────────────────────────────────────┐
  │                   VPC                       │
  │                                             │
  │  ┌─────────────────┐  ┌─────────────────┐   │
  │  │ Public Subnet    │  │ Public Subnet    │   │
  │  │ (AZ-1)          │  │ (AZ-2)          │   │
  │  │                 │  │                 │   │
  │  │ ┌─────────────┐│  │ ┌─────────────┐│   │
  │  │ │   ALB       ││  │ │   NAT GW    ││   │
  │  │ └─────────────┘│  │ └─────────────┘│   │
  │  └───────┬─────────┘  └───────┬─────────┘   │
  │          │                    │              │
  │  ┌───────▼─────────┐  ┌──────▼──────────┐   │
  │  │ Private Subnet   │  │ Private Subnet   │   │
  │  │ (AZ-1)          │  │ (AZ-2)          │   │
  │  │                 │  │                 │   │
  │  │ ┌─────────────┐│  │ ┌─────────────┐│   │
  │  │ │   EC2       ││  │ │   EC2       ││   │
  │  │ └─────────────┘│  │ └─────────────┘│   │
  │  └───────┬─────────┘  └───────┬─────────┘   │
  │          │                    │              │
  │  ┌───────▼─────────┐  ┌──────▼──────────┐   │
  │  │ Data Subnet      │  │ Data Subnet      │   │
  │  │ (AZ-1)          │  │ (AZ-2)          │   │
  │  │ ┌─────────────┐│  │ ┌─────────────┐│   │
  │  │ │   RDS       ││  │ │   RDS       ││   │
  │  │ └─────────────┘│  │ └─────────────┘│   │
  │  └─────────────────┘  └─────────────────┘   │
  │                                             │
  │  Internet Gateway ←→ Public subnets          │
  │  NAT Gateway → Private subnet outbound       │
  │  No direct internet → Data subnets           │
  └─────────────────────────────────────────────┘

SECURITY LAYERS:
  Layer 1: Security Groups (instance-level firewall)
  Layer 2: Network ACLs (subnet-level firewall)
  Layer 3: VPC Flow Logs (monitoring)
  Layer 4: AWS WAF (application layer)
  Layer 5: AWS Shield (DDoS protection)
```

---

## 4. AWS Security Services Overview

```
SECURITY SERVICE CATEGORIES:

IDENTITY & ACCESS:
  → IAM: Users, groups, roles, policies
  → AWS SSO (Identity Center): Centralized access
  → Cognito: Application user authentication
  → Directory Service: Managed AD
  → Resource Access Manager: Cross-account sharing

DETECTION:
  → CloudTrail: API activity logging
  → CloudWatch: Metrics and logs
  → GuardDuty: Threat detection
  → Security Hub: Security posture dashboard
  → Inspector: Vulnerability assessment
  → Macie: Data discovery and protection
  → Detective: Security investigation

NETWORK PROTECTION:
  → WAF: Web Application Firewall
  → Shield: DDoS protection
  → Firewall Manager: Central firewall management
  → Network Firewall: VPC-level firewall
  → VPC Security Groups & NACLs

DATA PROTECTION:
  → KMS: Key Management Service
  → CloudHSM: Hardware Security Module
  → Certificate Manager: SSL/TLS certificates
  → Secrets Manager: Secret storage and rotation
  → Macie: Sensitive data discovery

COMPLIANCE:
  → Config: Resource compliance tracking
  → Artifact: Compliance reports
  → Audit Manager: Audit management
  → Control Tower: Multi-account governance

INCIDENT RESPONSE:
  → Detective: Investigation
  → SSM (Systems Manager): Automation
  → Lambda: Automated response
  → Step Functions: Workflow automation
```

---

## 5. Security Architecture Best Practices

```
AWS WELL-ARCHITECTED — SECURITY PILLAR:

DESIGN PRINCIPLES:
  1. Implement a strong identity foundation
  2. Enable traceability
  3. Apply security at all layers
  4. Automate security best practices
  5. Protect data in transit and at rest
  6. Keep people away from data
  7. Prepare for security events

REFERENCE ARCHITECTURE:
  → Multi-account strategy (AWS Organizations)
  → Centralized logging (CloudTrail → S3 → Security Lake)
  → Centralized security (Security Hub aggregation)
  → Network segmentation (VPCs, subnets, SGs)
  → Encryption everywhere (KMS, TLS)
  → Least privilege IAM
  → Automated remediation (Config + Lambda)

ENUMERATION (For Assessments):
  # AWS CLI enumeration
  aws sts get-caller-identity         # Who am I?
  aws iam list-users                  # List IAM users
  aws iam list-roles                  # List IAM roles
  aws s3 ls                           # List S3 buckets
  aws ec2 describe-instances          # List EC2 instances
  aws ec2 describe-security-groups    # List security groups
  aws lambda list-functions           # List Lambda functions
  aws rds describe-db-instances       # List RDS databases
```

---

## Summary Table

| Component | Security Feature | Purpose |
|-----------|-----------------|---------|
| Organizations | SCPs | Guardrails for accounts |
| VPC | Security Groups, NACLs | Network isolation |
| IAM | Policies, roles | Access control |
| CloudTrail | API logging | Audit trail |
| GuardDuty | Threat detection | Identify threats |
| KMS | Encryption keys | Data protection |

---

## Revision Questions

1. How is AWS global infrastructure organized for security?
2. What role do Service Control Policies play in AWS Organizations?
3. What is the recommended VPC security architecture?
4. What are the main categories of AWS security services?
5. What are the AWS Well-Architected Security Pillar design principles?

---

*Previous: None (First topic in this unit) | Next: [02-iam-users-roles-policies.md](02-iam-users-roles-policies.md)*

---

*[Back to README](../README.md)*
