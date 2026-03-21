# Unit 2: AWS Security — Topic 2: IAM (Users, Roles, Policies)

## Overview

**AWS Identity and Access Management (IAM)** is the foundation of AWS security. It controls who can access what resources and under what conditions. IAM misconfigurations are the most common source of AWS security incidents, making a deep understanding of users, groups, roles, and policies essential for both defenders and penetration testers.

---

## 1. IAM Fundamentals

```
IAM COMPONENTS:

  ┌──────────────────────────────────────────┐
  │              AWS IAM                     │
  │                                          │
  │  ┌──────────┐  ┌──────────┐             │
  │  │  Users   │  │  Groups  │             │
  │  │          │  │          │             │
  │  │ Person / │  │ Collect  │             │
  │  │ Service  │  │ of users │             │
  │  └─────┬────┘  └────┬─────┘             │
  │        │             │                   │
  │        ▼             ▼                   │
  │  ┌─────────────────────────────────┐     │
  │  │          POLICIES               │     │
  │  │  (Define permissions)           │     │
  │  │  Allow/Deny + Actions +         │     │
  │  │  Resources + Conditions         │     │
  │  └─────────────────────────────────┘     │
  │        ▲                                 │
  │        │                                 │
  │  ┌─────┴────┐                            │
  │  │  Roles   │                            │
  │  │          │                            │
  │  │ Assumed  │                            │
  │  │ by users,│                            │
  │  │ services │                            │
  │  └──────────┘                            │
  └──────────────────────────────────────────┘

USERS:
  → Individual identity with credentials
  → Console password + MFA
  → Access keys (programmatic)
  → Max 2 access keys per user
  → Should map to real people
  → NOT for applications (use roles instead)

GROUPS:
  → Collection of IAM users
  → Policies attached to group
  → Users inherit group permissions
  → Can't nest groups
  → Use for role-based access (Admins, Developers, ReadOnly)

ROLES:
  → Temporary credentials (STS)
  → No permanent credentials
  → Assumed by: users, services, applications
  → Cross-account access
  → Best practice for applications
  → Trust policy defines who can assume

POLICIES:
  → JSON documents defining permissions
  → Attached to users, groups, or roles
  → Types: AWS managed, customer managed, inline
  → Evaluated together — explicit deny wins
```

---

## 2. IAM Policies Deep Dive

```
POLICY STRUCTURE:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Read",
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
          "aws:SourceIp": "10.0.0.0/8"
        }
      }
    }
  ]
}

COMPONENTS:
  Version:   Always "2012-10-17" (current version)
  Statement: Array of permission rules
  Sid:       Statement ID (optional, descriptive)
  Effect:    "Allow" or "Deny"
  Action:    API operations (service:action)
  Resource:  ARN of target resource
  Condition: When the policy applies

POLICY TYPES:
  AWS Managed:
  → Pre-built by AWS
  → e.g., AdministratorAccess, ReadOnlyAccess
  → Updated by AWS
  → Convenient but often over-permissive
  
  Customer Managed:
  → Created by your organization
  → Customized to your needs
  → Reusable across users/roles
  → Best for custom permissions
  
  Inline:
  → Embedded directly in user/group/role
  → Not reusable
  → Deleted with the entity
  → Use sparingly (hard to manage)

POLICY EVALUATION LOGIC:
  1. Start with implicit DENY (default)
  2. Evaluate all applicable policies
  3. If any explicit DENY → DENIED
  4. If any Allow → ALLOWED
  5. Otherwise → DENIED (implicit)
  
  Deny ALWAYS wins over Allow
  
  Decision flow:
  Request → SCP check → Resource policy →
  Permission boundary → Identity policy → Result

DANGEROUS POLICIES:
  # Full admin (avoid!)
  "Action": "*", "Resource": "*"
  
  # Privilege escalation paths
  "Action": "iam:CreateUser"          → Create new admin
  "Action": "iam:AttachUserPolicy"    → Attach admin policy
  "Action": "iam:CreateAccessKey"     → Create keys for any user
  "Action": "iam:PassRole"            → Pass powerful role
  "Action": "sts:AssumeRole"          → Assume powerful role
  "Action": "lambda:CreateFunction"   → Create Lambda with role
```

---

## 3. IAM Roles and Assumption

```
ROLE TRUST POLICIES:

TRUST POLICY (Who can assume the role):
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "ec2.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}

COMMON PRINCIPALS:
  Service Role:
  → "Service": "ec2.amazonaws.com"       (EC2 instances)
  → "Service": "lambda.amazonaws.com"    (Lambda functions)
  → "Service": "ecs-tasks.amazonaws.com" (ECS tasks)
  
  Cross-Account:
  → "AWS": "arn:aws:iam::123456789012:root"  (entire account)
  → "AWS": "arn:aws:iam::123456789012:user/admin"
  
  Federated:
  → "Federated": "cognito-identity.amazonaws.com"
  → "Federated": "arn:aws:iam::123456789012:saml-provider/ADFS"

ASSUMING A ROLE:
  # AWS CLI
  aws sts assume-role \
    --role-arn "arn:aws:iam::123456789012:role/AdminRole" \
    --role-session-name "MySession"
  
  # Returns:
  # AccessKeyId, SecretAccessKey, SessionToken
  # Temporary credentials (default 1 hour)
  
  # Use credentials
  export AWS_ACCESS_KEY_ID=ASIA...
  export AWS_SECRET_ACCESS_KEY=...
  export AWS_SESSION_TOKEN=...

INSTANCE PROFILES:
  → EC2 instances assume roles automatically
  → Credentials available via metadata service
  → No hardcoded keys needed
  → Rotated automatically
  → ALWAYS use instead of access keys on EC2
```

---

## 4. IAM Security Best Practices

```
SECURITY HARDENING:

1. LEAST PRIVILEGE:
   → Start with no permissions
   → Grant only what's needed
   → Use IAM Access Analyzer to find unused
   → Regularly review and reduce
   → Use permission boundaries
   
   # IAM Access Analyzer
   aws accessanalyzer list-findings

2. MFA EVERYWHERE:
   → Root account: Hardware MFA
   → All IAM users: Virtual MFA minimum
   → Console access: Enforce MFA
   → CLI access: MFA with STS
   → Privileged operations: MFA condition

3. NO ROOT ACCESS KEYS:
   → Delete root access keys
   → Use IAM users for daily operations
   → Root only for account-level tasks
   → Monitor root usage with CloudTrail

4. ACCESS KEY ROTATION:
   → Rotate keys every 90 days
   → Use IAM credential report
   → Automate rotation
   → Disable unused keys

5. PERMISSION BOUNDARIES:
   → Cap maximum permissions for role
   → Even if policy grants more, boundary limits
   → Useful for delegating role creation
   
   # Effective permissions = policy ∩ boundary
   # If policy allows s3:* but boundary allows s3:GetObject
   # → only s3:GetObject is effective

6. CONDITIONS:
   → IP-based restrictions
   → MFA requirements
   → Time-based access
   → Tag-based conditions
   
   "Condition": {
     "Bool": {"aws:MultiFactorAuthPresent": "true"},
     "IpAddress": {"aws:SourceIp": "10.0.0.0/8"},
     "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"}
   }

ENUMERATION (Assessment):
  # List all users
  aws iam list-users
  
  # List user's policies
  aws iam list-attached-user-policies --user-name admin
  aws iam list-user-policies --user-name admin
  
  # Get policy details
  aws iam get-policy-version --policy-arn arn:... --version-id v1
  
  # Check for access keys
  aws iam list-access-keys --user-name admin
  
  # Credential report
  aws iam generate-credential-report
  aws iam get-credential-report
  
  # Tools
  → Pacu: AWS exploitation framework
  → enumerate-iam: Find available permissions
  → Principal Mapper: Visualize privilege escalation
```

---

## 5. Common IAM Attacks

```
IAM PRIVILEGE ESCALATION:

KNOWN ESCALATION PATHS:
  1. iam:CreateAccessKey → Create keys for admin user
  2. iam:AttachUserPolicy → Attach AdministratorAccess
  3. iam:PutUserPolicy → Add inline admin policy
  4. iam:PassRole + lambda:CreateFunction → Admin Lambda
  5. iam:PassRole + ec2:RunInstances → Admin EC2
  6. sts:AssumeRole → Assume admin role
  7. iam:CreateUser + iam:AttachUserPolicy → New admin

PACU USAGE:
  # AWS exploitation framework
  pacu
  > import_keys <access_key> <secret_key>
  > run iam__enum_permissions
  > run iam__privesc_scan
  > run iam__bruteforce_permissions

DETECTION:
  → CloudTrail logging all IAM API calls
  → GuardDuty IAM findings
  → Access Analyzer for external access
  → Config rules for compliance
  → Custom CloudWatch alerts
```

---

## Summary Table

| Component | Purpose | Best Practice |
|-----------|---------|--------------|
| Users | Individual identity | MFA, least privilege |
| Groups | Permission grouping | Role-based assignment |
| Roles | Temporary access | Use for services/apps |
| Policies | Permission definition | Customer-managed, specific |
| Access Keys | Programmatic access | Rotate, use roles instead |
| SCPs | Account guardrails | Deny dangerous actions |

---

## Revision Questions

1. What are the four main IAM components?
2. How does IAM policy evaluation work when multiple policies conflict?
3. What are the most common IAM privilege escalation paths?
4. Why should IAM roles be preferred over access keys?
5. What tools help identify excessive IAM permissions?

---

*Previous: [01-aws-security-architecture.md](01-aws-security-architecture.md) | Next: [03-s3-security.md](03-s3-security.md)*

---

*[Back to README](../README.md)*
