# Unit 5: A05 - Security Misconfiguration — Topic 5: Cloud Misconfigurations

## Overview

Cloud misconfigurations are the **leading cause of cloud security breaches**, with studies showing that over 65% of cloud security incidents result from misconfiguration. The shared responsibility model means cloud providers secure the infrastructure, but customers must correctly configure their resources. Misconfigured storage buckets, overly permissive IAM policies, exposed databases, and insecure network settings have led to some of the largest data breaches in history.

---

## 1. Shared Responsibility Model

```
┌─────────────────────────────────────────────────────────────────┐
│           SHARED RESPONSIBILITY MODEL                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CUSTOMER Responsibility ("Security IN the Cloud"):            │
│  ├── Data classification and encryption                        │
│  ├── Identity and Access Management (IAM)                      │
│  ├── Operating system / application patching                   │
│  ├── Network and firewall configuration                        │
│  ├── Client-side encryption                                    │
│  └── Security group / NACL configuration                       │
│                                                                 │
│  ────────────────── Shared Line ──────────────────              │
│                                                                 │
│  PROVIDER Responsibility ("Security OF the Cloud"):            │
│  ├── Physical security of data centers                         │
│  ├── Hardware maintenance                                      │
│  ├── Network infrastructure                                    │
│  ├── Hypervisor security                                       │
│  └── Managed service infrastructure                            │
│                                                                 │
│  Most breaches occur above the line — customer misconfiguration│
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Storage Misconfiguration

### AWS S3 Bucket Misconfigurations

```
Most common cloud misconfiguration — publicly accessible S3 buckets

❌ DANGEROUS S3 Bucket Policy (Public Access):
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": "*",          ← ANYONE in the world
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::company-data/*"
    }]
}

✅ SECURE S3 Bucket Policy:
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {"AWS": "arn:aws:iam::123456789012:role/AppRole"},
        "Action": ["s3:GetObject"],
        "Resource": "arn:aws:s3:::company-data/*",
        "Condition": {
            "StringEquals": {"aws:SourceVpc": "vpc-abc123"}
        }
    }]
}
```

### Scanning for Public Buckets

```bash
# AWS CLI — Check bucket ACL
aws s3api get-bucket-acl --bucket company-data
aws s3api get-bucket-policy --bucket company-data

# Check for public access
aws s3api get-public-access-block --bucket company-data

# Enable public access block (ALWAYS DO THIS)
aws s3api put-public-access-block --bucket company-data \
    --public-access-block-configuration \
    BlockPublicAcls=true,\
    IgnorePublicAcls=true,\
    BlockPublicPolicy=true,\
    RestrictPublicBuckets=true

# Third-party tools
# S3Scanner, bucket_finder, AWSBucketDump
```

### Azure and GCP Equivalents

```
Azure Storage Account Misconfigurations:
❌ Blob container with "Public access level: Container" 
✅ Set to "Private" — require SAS tokens or Azure AD auth

GCP Cloud Storage:
❌ Bucket with "allUsers" or "allAuthenticatedUsers" permission
✅ Use IAM roles with specific service accounts
```

---

## 3. IAM Misconfigurations

```
┌─────────────────────────────────────────────────────────────────┐
│              COMMON IAM MISCONFIGURATIONS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. OVERLY PERMISSIVE POLICIES:                                │
│     ❌ "Action": "*", "Resource": "*"  → God mode              │
│     ✅ "Action": "s3:GetObject", "Resource": "arn:...specific" │
│                                                                 │
│  2. WILDCARD PERMISSIONS:                                      │
│     ❌ "Action": "s3:*"                → All S3 actions        │
│     ✅ "Action": ["s3:GetObject", "s3:ListBucket"]             │
│                                                                 │
│  3. UNUSED CREDENTIALS:                                        │
│     ❌ Access keys not rotated for 2+ years                    │
│     ✅ 90-day rotation policy, disable unused                  │
│                                                                 │
│  4. ROOT ACCOUNT USAGE:                                        │
│     ❌ Using root/owner account for daily operations           │
│     ✅ MFA on root, use IAM users/roles for everything         │
│                                                                 │
│  5. CROSS-ACCOUNT ACCESS:                                      │
│     ❌ Trust policy with "Principal": "*"                      │
│     ✅ Specific account IDs and external ID conditions         │
│                                                                 │
│  6. MISSING MFA:                                               │
│     ❌ No MFA on privileged accounts                           │
│     ✅ MFA required for all console access and API calls       │
└─────────────────────────────────────────────────────────────────┘
```

### IAM Best Practices

```json
// ✅ LEAST PRIVILEGE IAM Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::app-uploads/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                },
                "IpAddress": {
                    "aws:SourceIp": "10.0.0.0/8"
                }
            }
        }
    ]
}
```

---

## 4. Network Misconfigurations

```
❌ DANGEROUS Security Group:
  Inbound: 0.0.0.0/0 → ALL ports (all traffic from internet)
  Outbound: 0.0.0.0/0 → ALL ports

✅ SECURE Security Group (Web Server):
  Inbound:
    Port 443 (HTTPS) from 0.0.0.0/0     → Public web traffic
    Port 22 (SSH) from 10.0.1.0/24      → Bastion host subnet only
  Outbound:
    Port 443 to 0.0.0.0/0               → External HTTPS
    Port 5432 to sg-database             → Database security group only
    Port 53 to VPC DNS                   → DNS resolution

NETWORK SEGMENTATION:
┌──────────────────────────────────────────────────────────┐
│  VPC 10.0.0.0/16                                        │
│  ┌────────────────────┐  ┌────────────────────────────┐ │
│  │ Public Subnet      │  │ Private Subnet             │ │
│  │ 10.0.1.0/24        │  │ 10.0.2.0/24               │ │
│  │                    │  │                            │ │
│  │ [ALB] [Bastion]    │──│ [App Servers]              │ │
│  │                    │  │      │                     │ │
│  └────────────────────┘  │      ▼                     │ │
│                          │ ┌──────────────────────┐   │ │
│                          │ │ Private Subnet       │   │ │
│                          │ │ 10.0.3.0/24          │   │ │
│                          │ │ [RDS] [ElastiCache]  │   │ │
│                          │ └──────────────────────┘   │ │
│                          └────────────────────────────┘ │
│  Databases NEVER in public subnets                      │
└──────────────────────────────────────────────────────────┘
```

---

## 5. Cloud Security Scanning Tools

| Tool | Type | Platform | Purpose |
|------|------|----------|---------|
| **ScoutSuite** | Open source | Multi-cloud | Configuration audit |
| **Prowler** | Open source | AWS | CIS/NIST compliance |
| **CloudSploit** | Open source | Multi-cloud | Misconfig detection |
| **Checkov** | Open source | IaC scanning | Terraform/CF/K8s |
| **tfsec** | Open source | Terraform | Security scanning |
| **AWS Config** | Native | AWS | Continuous compliance |
| **Azure Policy** | Native | Azure | Policy enforcement |
| **GCP Security Command Center** | Native | GCP | Security posture |
| **Prisma Cloud** | Commercial | Multi-cloud | CSPM |
| **Wiz** | Commercial | Multi-cloud | CNAPP |

### Running Prowler (AWS)

```bash
# Install
pip install prowler

# Run full audit
prowler aws

# Run specific checks
prowler aws --checks s3_bucket_public_access

# Generate report
prowler aws --output-formats html csv json
```

---

## 6. Notable Cloud Breaches from Misconfiguration

| Company | Year | Misconfiguration | Impact |
|---------|------|-------------------|--------|
| **Capital One** | 2019 | Misconfigured WAF + SSRF → IAM role access | 100M+ records |
| **Facebook** | 2019 | Public S3 bucket with user data | 540M records |
| **Twitch** | 2021 | Misconfigured internal server | Full source code leak |
| **Microsoft** | 2022 | Misconfigured Azure endpoint | 2.4TB of customer data |
| **Toyota** | 2023 | Public GitHub repo with cloud credentials | Vehicle data exposed |

---

## 7. Cloud Hardening Checklist

```
STORAGE:
□ All buckets/containers set to private
□ Public access block enabled (account level)
□ Server-side encryption enabled (SSE-S3 or SSE-KMS)
□ Versioning and logging enabled
□ Lifecycle policies for data retention

IAM:
□ Root/owner account has MFA, not used for daily ops
□ All policies follow least privilege
□ Access keys rotated every 90 days
□ Unused accounts and keys removed
□ Service accounts use roles, not long-term keys
□ Cross-account access uses external IDs

NETWORK:
□ Databases in private subnets only
□ Security groups follow least privilege
□ VPC flow logs enabled
□ No 0.0.0.0/0 on management ports (SSH, RDP)
□ NACLs configured as additional layer

COMPUTE:
□ IMDSv2 enforced (prevents SSRF → metadata attacks)
□ No public IPs on non-public instances
□ OS and container images regularly patched
□ Secrets in Secrets Manager, not env vars

LOGGING:
□ CloudTrail/Activity Log enabled (all regions)
□ Log file validation enabled
□ Logs sent to separate, locked account
□ Alerts on suspicious activity
```

---

## Summary Table

| Misconfiguration | Risk | Remediation |
|-----------------|------|-------------|
| Public storage buckets | Data exposure | Block public access |
| Overly permissive IAM | Privilege escalation | Least privilege policies |
| Open security groups | Unauthorized access | Restrict to required ports/IPs |
| Missing encryption | Data breach | Enable SSE and TLS |
| No logging | Undetected attacks | Enable CloudTrail/audit logs |
| Exposed metadata | SSRF → credential theft | IMDSv2, network restrictions |

---

## Revision Questions

1. Explain the shared responsibility model. Who is responsible for S3 bucket permissions?
2. What is the most common S3 misconfiguration? Write both insecure and secure bucket policies.
3. List five IAM misconfigurations and their remediation.
4. Design a secure VPC architecture with proper subnet segmentation for a three-tier web application.
5. Name five cloud security scanning tools and explain what each detects.
6. How did the Capital One breach exploit cloud misconfiguration? What controls would have prevented it?

---

*Previous: [04-security-headers.md](04-security-headers.md) | Next: [06-hardening-procedures.md](06-hardening-procedures.md)*

---

*[Back to README](../README.md)*
