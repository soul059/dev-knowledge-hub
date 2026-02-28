# Chapter 1.4: IAM Basics (Identity and Access Management)

## Overview

IAM is the **foundation of security in AWS**. It controls **who** (authentication) can do **what** (authorization) on **which** resources. IAM is a **global service** — not region-specific. Every API call to AWS goes through IAM for permission checks.

---

## Core IAM Components

```
┌───────────────────────────────────────────────────────────────────┐
│                        AWS IAM                                    │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │  USERS   │  │  GROUPS  │  │  ROLES   │  │    POLICIES      │ │
│  │ (People) │  │ (Teams)  │  │ (Temp)   │  │ (Permissions)    │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘ │
│       │             │             │                  │           │
│       └─────────────┴─────────────┴──────────────────┘           │
│                           │                                       │
│                    Policies are ATTACHED                          │
│                    to Users, Groups, or Roles                    │
└───────────────────────────────────────────────────────────────────┘
```

---

## IAM Users

An IAM User represents a **person or application** that interacts with AWS.

### User Properties

| Property | Description |
|----------|-------------|
| **Username** | Unique identifier within the account |
| **Console Password** | For browser login (optional) |
| **Access Keys** | For CLI/SDK access (max 2 per user) |
| **MFA Device** | Multi-factor authentication (strongly recommended) |
| **Permissions** | Via attached policies (inline or managed) |

### Creating a User

```bash
# Create user
aws iam create-user --user-name devops-engineer

# Create console login
aws iam create-login-profile \
  --user-name devops-engineer \
  --password "TempP@ss123!" \
  --password-reset-required

# Create access keys (for CLI/SDK)
aws iam create-access-key --user-name devops-engineer
# Output: AccessKeyId + SecretAccessKey (save these!)

# Enable virtual MFA
aws iam enable-mfa-device \
  --user-name devops-engineer \
  --serial-number arn:aws:iam::123456789012:mfa/devops-engineer \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

### Root Account vs IAM Users

```
┌──────────────────────────────────┬──────────────────────────────┐
│         ROOT ACCOUNT             │         IAM USER             │
├──────────────────────────────────┼──────────────────────────────┤
│ Created with AWS account         │ Created by root or admin     │
│ Full, unrestricted access        │ Only permissions granted     │
│ Cannot be restricted by IAM      │ Controlled by IAM policies   │
│ Used for: billing, account       │ Used for: daily operations   │
│ settings, closing account        │                              │
│                                  │                              │
│ ⚠ NEVER use for daily tasks!    │ ✓ Use for all daily work    │
│ ⚠ Enable MFA immediately!      │ ✓ Follow least privilege    │
└──────────────────────────────────┴──────────────────────────────┘
```

---

## IAM Groups

Groups let you **organize users** and **assign permissions collectively**.

```
┌────────────────────────────────────────────────────────┐
│                    IAM GROUPS                           │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐│
│  │  Developers  │  │   DevOps     │  │   Admins     ││
│  │  ──────────  │  │   ──────     │  │   ──────     ││
│  │  • Alice     │  │  • Charlie   │  │  • Eve       ││
│  │  • Bob       │  │  • Dave      │  │              ││
│  │              │  │              │  │              ││
│  │  Policy:     │  │  Policy:     │  │  Policy:     ││
│  │  EC2 Read    │  │  Full EC2    │  │  Admin       ││
│  │  S3 Read     │  │  Full S3     │  │  Access      ││
│  │  Lambda Full │  │  CloudWatch  │  │              ││
│  └──────────────┘  └──────────────┘  └──────────────┘│
│                                                        │
│  ⚠ Groups CANNOT be nested (no groups inside groups)  │
│  ⚠ A user can belong to multiple groups (max 10)      │
│  ⚠ Groups are for USERS only, not for roles           │
└────────────────────────────────────────────────────────┘
```

```bash
# Create a group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group --user-name Alice --group-name Developers

# Attach a managed policy to the group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

# List users in group
aws iam get-group --group-name Developers
```

---

## IAM Policies

Policies are **JSON documents** that define permissions (allow or deny actions on resources).

### Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

### Policy Elements

```
┌─────────────────────────────────────────────────────────┐
│                 POLICY ANATOMY                          │
│                                                         │
│  Version ──────► "2012-10-17" (always use this)        │
│                                                         │
│  Statement[] ──► Array of permission rules             │
│    │                                                    │
│    ├── Sid ────► Statement ID (optional, descriptive)  │
│    │                                                    │
│    ├── Effect ─► "Allow" or "Deny"                     │
│    │                                                    │
│    ├── Action ─► What API actions (e.g., s3:GetObject) │
│    │             Wildcards: s3:* or *                   │
│    │                                                    │
│    ├── Resource► Which resources (ARN format)          │
│    │             arn:aws:s3:::my-bucket/*               │
│    │                                                    │
│    └── Condition► (Optional) When to apply             │
│                  IP range, time, MFA, tags, etc.        │
└─────────────────────────────────────────────────────────┘
```

### Policy Types

| Type | Description | Use Case |
|------|-------------|----------|
| **AWS Managed** | Created and maintained by AWS | Common permissions (e.g., `AmazonS3ReadOnlyAccess`) |
| **Customer Managed** | Created by you | Custom permissions for your org |
| **Inline** | Embedded directly in user/group/role | One-off, tightly coupled permissions |

### Policy Evaluation Logic

```
┌────────────────────────────────────────────────────────┐
│           IAM POLICY EVALUATION FLOW                   │
│                                                        │
│  Request comes in                                      │
│       │                                                │
│       ▼                                                │
│  ┌──────────────────┐                                  │
│  │  Explicit DENY   │──── YES ──► ✗ DENIED            │
│  │  in any policy?  │                                  │
│  └────────┬─────────┘                                  │
│           │ NO                                         │
│           ▼                                            │
│  ┌──────────────────┐                                  │
│  │  Explicit ALLOW  │──── YES ──► ✓ ALLOWED           │
│  │  in any policy?  │                                  │
│  └────────┬─────────┘                                  │
│           │ NO                                         │
│           ▼                                            │
│  ┌──────────────────┐                                  │
│  │  ✗ DENIED        │  (Implicit deny — default)      │
│  └──────────────────┘                                  │
│                                                        │
│  KEY: Explicit Deny > Explicit Allow > Implicit Deny   │
└────────────────────────────────────────────────────────┘
```

### Common Policy Examples

#### 1. Allow Full S3 Access to One Bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-app-bucket",
        "arn:aws:s3:::my-app-bucket/*"
      ]
    }
  ]
}
```

#### 2. Deny All Actions Without MFA

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

#### 3. Allow EC2 Actions Only in Specific Region

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

---

## IAM Roles

Roles provide **temporary credentials** to entities that assume them. Used by AWS services, applications, and cross-account access.

```
┌──────────────────────────────────────────────────────────┐
│                    IAM ROLES                             │
│                                                          │
│  WHO can assume the role?                                │
│  (Defined in the TRUST POLICY)                           │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  EC2 Instance│  │ Lambda Func  │  │ Another AWS  │  │
│  │              │  │              │  │   Account    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                 │                 │           │
│         └─────────────────┼─────────────────┘           │
│                           ▼                              │
│                  ┌──────────────────┐                    │
│                  │    IAM ROLE      │                    │
│                  │  ┌────────────┐  │                    │
│                  │  │Trust Policy│  │  WHO can assume   │
│                  │  └────────────┘  │                    │
│                  │  ┌────────────┐  │                    │
│                  │  │Permissions │  │  WHAT they can do │
│                  │  │  Policy    │  │                    │
│                  │  └────────────┘  │                    │
│                  └──────────────────┘                    │
│                           │                              │
│                           ▼                              │
│                  Temporary Credentials                   │
│                  (auto-rotated by STS)                   │
└──────────────────────────────────────────────────────────┘
```

### Common Role Use Cases

| Scenario | Who Assumes | Example |
|----------|-------------|---------|
| EC2 Instance Role | EC2 service | App on EC2 needs S3 access |
| Lambda Execution Role | Lambda service | Function needs DynamoDB read |
| Cross-Account Role | IAM user in another account | Dev team accesses prod account |
| Service-Linked Role | AWS service (auto-managed) | ELB managing EC2 targets |

### Create an EC2 Instance Role

```bash
# 1. Create the trust policy (who can assume)
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# 2. Create the role
aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document file://trust-policy.json

# 3. Attach permission policy
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# 4. Create instance profile (required for EC2)
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile \
  --role-name EC2-S3-Access

# 5. Attach to EC2 instance
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxx \
  --iam-instance-profile Name=EC2-S3-Access-Profile
```

---

## MFA (Multi-Factor Authentication)

```
┌────────────────────────────────────────────────────────┐
│                   MFA OPTIONS                          │
│                                                        │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │  Virtual    │  │  Hardware   │  │  Universal   │  │
│  │  MFA App    │  │  Token      │  │  2nd Factor  │  │
│  │  (TOTP)     │  │  (Key fob)  │  │  (U2F Key)   │  │
│  │             │  │             │  │              │  │
│  │  Google     │  │  Gemalto    │  │  YubiKey     │  │
│  │  Authentictr│  │             │  │              │  │
│  │  Authy      │  │             │  │              │  │
│  └─────────────┘  └─────────────┘  └──────────────┘  │
│                                                        │
│  ★ Enable MFA on ROOT ACCOUNT first!                  │
│  ★ Enforce MFA via IAM policies for all users         │
└────────────────────────────────────────────────────────┘
```

---

## IAM Best Practices Checklist

```
✓  Lock down the root account (MFA, no access keys)
✓  Create individual IAM users (no shared accounts)
✓  Use Groups to assign permissions (not direct user policies)
✓  Grant least privilege (start with minimum, add as needed)
✓  Use IAM Roles for applications (not access keys)
✓  Enable MFA for all human users
✓  Rotate credentials regularly
✓  Use policy conditions for extra security (IP, MFA, region)
✓  Monitor IAM activity with CloudTrail
✓  Use IAM Access Analyzer to find unused permissions
```

---

## ARN (Amazon Resource Name) Format

```
arn:partition:service:region:account-id:resource-type/resource-id

Examples:
arn:aws:s3:::my-bucket                          # S3 bucket (global, no region/account)
arn:aws:s3:::my-bucket/*                        # All objects in bucket
arn:aws:ec2:us-east-1:123456789012:instance/i-xxx  # Specific EC2 instance
arn:aws:iam::123456789012:user/Alice            # IAM user (global, no region)
arn:aws:lambda:us-east-1:123456789012:function:myFunc  # Lambda function
```

---

## Real-World Scenarios

### Scenario 1: New Development Team Onboarding
1. Create IAM Group `Developers`
2. Attach policies: `AmazonEC2ReadOnlyAccess`, `AmazonS3FullAccess`, `AWSLambda_FullAccess`
3. Create IAM Users for each developer
4. Add users to the `Developers` group
5. Enforce MFA with a conditional deny policy
6. Provide temporary passwords with mandatory reset

### Scenario 2: Application on EC2 Needs S3 Access
- **Don't:** Store access keys in the application code
- **Do:** Create an IAM Role with S3 permissions, attach to the EC2 instance
- The SDK automatically uses the instance role credentials (no keys in code!)

### Scenario 3: Cross-Account CI/CD Pipeline
- Dev account pipeline needs to deploy to Prod account
- Create a role in Prod with trust policy allowing Dev account
- Pipeline assumes the Prod role to deploy resources

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `AccessDenied` | Missing permissions | Use IAM Policy Simulator or CloudTrail to diagnose |
| Can't delete user | User has resources attached | Remove MFA, access keys, policies, groups first |
| Policy not taking effect | Explicit Deny overriding | Check all attached policies for Deny statements |
| Can't assume role | Trust policy wrong | Verify Principal in trust policy matches caller |
| Key rotation needed | Compliance requirement | Create new key → update apps → delete old key |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **IAM User** | Represents a person or app. Has username, optional password, optional access keys. |
| **IAM Group** | Collection of users. Attach policies to groups, not individual users. |
| **IAM Role** | Temporary credentials for AWS services, apps, cross-account access. |
| **IAM Policy** | JSON document defining permissions. Effect + Action + Resource + Condition. |
| **Policy Evaluation** | Explicit Deny > Explicit Allow > Implicit Deny (default deny). |
| **MFA** | Multi-factor auth. Enable on root account first. Enforce via policies. |
| **Best Practices** | Least privilege, use roles not keys, enable MFA, use groups. |

---

## Quick Revision Questions

1. **What is the default permission for a new IAM user?**
   > No permissions (implicit deny). You must explicitly grant permissions via policies.

2. **Can IAM Groups be nested?**
   > No. AWS does not support groups within groups. A user can belong to up to 10 groups.

3. **In IAM policy evaluation, what happens if there's both an Allow and a Deny?**
   > Explicit Deny always wins. The request is denied.

4. **Why should you use IAM Roles instead of access keys for EC2 applications?**
   > Roles provide temporary, automatically-rotated credentials. Access keys are long-lived secrets that can be leaked or stolen.

5. **What is a Trust Policy in an IAM Role?**
   > It defines WHO (which principal — user, service, or account) is allowed to assume the role.

6. **What is the ARN for an S3 bucket named `logs-prod`?**
   > `arn:aws:s3:::logs-prod` (S3 ARNs don't include region or account ID).

---

[← Previous: 1.3 AWS Console, CLI, SDK](03-aws-console-cli-sdk.md) | [Next: 1.5 AWS Pricing Model →](05-aws-pricing-model.md)

[← Back to README](../README.md)
