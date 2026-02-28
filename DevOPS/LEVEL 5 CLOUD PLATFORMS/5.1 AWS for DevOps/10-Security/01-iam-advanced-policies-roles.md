# Chapter 10.1: IAM Advanced — Policies, Roles & Federation

## Overview

This chapter covers **advanced IAM concepts** beyond the basics: policy evaluation logic, cross-account roles, identity federation, permissions boundaries, and session policies that DevOps engineers need to secure AWS environments at scale.

---

## Policy Evaluation Logic

```
┌──── IAM Policy Evaluation Order ─────────────────────────┐
│                                                           │
│  Request comes in                                         │
│       │                                                   │
│       ▼                                                   │
│  ┌─────────────────────────────┐                         │
│  │ 1. Explicit DENY?          │── YES → DENY            │
│  └─────────────┬───────────────┘                         │
│                │ NO                                       │
│                ▼                                          │
│  ┌─────────────────────────────┐                         │
│  │ 2. SCP allows?             │── NO → DENY             │
│  │    (Organizations)         │                          │
│  └─────────────┬───────────────┘                         │
│                │ YES                                      │
│                ▼                                          │
│  ┌─────────────────────────────┐                         │
│  │ 3. Resource policy allows? │── YES → ALLOW           │
│  │    (S3, SQS, etc.)        │                          │
│  └─────────────┬───────────────┘                         │
│                │ NO explicit allow                         │
│                ▼                                          │
│  ┌─────────────────────────────┐                         │
│  │ 4. Permissions boundary    │── NO → DENY             │
│  │    allows?                 │                          │
│  └─────────────┬───────────────┘                         │
│                │ YES                                      │
│                ▼                                          │
│  ┌─────────────────────────────┐                         │
│  │ 5. Identity policy allows? │── YES → ALLOW           │
│  │    (user/role policy)      │── NO  → DENY (default)  │
│  └─────────────────────────────┘                         │
│                                                           │
│  Rule: Default DENY. Explicit DENY always wins.          │
└───────────────────────────────────────────────────────────┘
```

---

## Policy Types

```
┌──── Policy Types ────────────────────────────────────────┐
│                                                          │
│  Identity-Based Policies:                               │
│  ├── Managed (AWS or Customer) → attached to user/role  │
│  └── Inline → embedded directly in a single entity      │
│                                                          │
│  Resource-Based Policies:                               │
│  ├── S3 Bucket Policy                                   │
│  ├── SQS/SNS Access Policy                              │
│  ├── Lambda Resource Policy                             │
│  └── KMS Key Policy  (⚠ required for KMS)             │
│                                                          │
│  Permissions Boundary:                                  │
│  └── Maximum permissions an identity can have           │
│      Effective = Identity Policy ∩ Boundary             │
│                                                          │
│  Service Control Policies (SCPs):                       │
│  └── Maximum permissions for an OU/Account              │
│      (Organizations only)                               │
│                                                          │
│  Session Policies:                                      │
│  └── Passed during AssumeRole/Federation                │
│      Further restricts the session                      │
└──────────────────────────────────────────────────────────┘
```

---

## Cross-Account Access

```
┌──── Cross-Account Role Assumption ───────────────────────┐
│                                                           │
│  Account A (111111)           Account B (222222)          │
│  ┌──────────────────┐        ┌──────────────────────┐    │
│  │ IAM User: alice  │        │ IAM Role:            │    │
│  │                  │  STS   │  CrossAccountAdmin   │    │
│  │ Policy:          │───────→│                      │    │
│  │ Allow sts:       │ Assume │ Trust Policy:        │    │
│  │  AssumeRole      │ Role   │ Account A can assume │    │
│  │  on Role ARN     │        │                      │    │
│  └──────────────────┘        │ Permission Policy:   │    │
│                              │ Admin in Account B   │    │
│                              └──────────────────────┘    │
└───────────────────────────────────────────────────────────┘
```

```json
// Trust Policy (on the role in Account B)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::111111111111:root"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {
        "sts:ExternalId": "UniqueSecret123"
      },
      "Bool": {
        "aws:MultiFactorAuthPresent": "true"
      }
    }
  }]
}
```

```bash
# Assume the cross-account role
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountAdmin \
  --role-session-name alice-session \
  --external-id UniqueSecret123

# Returns temporary credentials (AccessKeyId, SecretAccessKey, SessionToken)
```

---

## Permissions Boundaries

```json
// Permissions Boundary — limits what a role CAN do
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyS3AndDynamo",
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*",
        "logs:*",
        "cloudwatch:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "DenyIAMChanges",
      "Effect": "Deny",
      "Action": [
        "iam:CreateUser",
        "iam:DeleteUser",
        "iam:CreateRole",
        "iam:AttachRolePolicy"
      ],
      "Resource": "*"
    }
  ]
}
```

```bash
# Set permissions boundary on a role
aws iam put-role-permissions-boundary \
  --role-name DevTeamRole \
  --permissions-boundary arn:aws:iam::123456789012:policy/DevBoundary
```

```
┌──── Effective Permissions ───────────────────────────────┐
│                                                          │
│  Identity Policy:    S3, DynamoDB, EC2, IAM             │
│  Permissions Boundary: S3, DynamoDB, Logs, CloudWatch   │
│                                                          │
│  Effective = Identity ∩ Boundary                        │
│            = S3, DynamoDB (only overlap)                 │
│                                                          │
│  EC2 — blocked by boundary                              │
│  IAM — blocked by boundary                              │
│  Logs — not in identity policy                          │
└──────────────────────────────────────────────────────────┘
```

---

## Identity Federation

```
┌──── Federation Options ──────────────────────────────────┐
│                                                          │
│  SAML 2.0 (Enterprise SSO):                             │
│  ├── Active Directory → ADFS → AWS STS                  │
│  ├── Azure AD, Okta, OneLogin                           │
│  └── Maps AD groups to IAM roles                        │
│                                                          │
│  OIDC (Web/Mobile):                                     │
│  ├── Google, Facebook, GitHub, Cognito                  │
│  ├── AssumeRoleWithWebIdentity                          │
│  └── Used by GitHub Actions for OIDC federation         │
│                                                          │
│  AWS IAM Identity Center (SSO):                         │
│  ├── Central SSO for all AWS accounts                   │
│  ├── Built-in directory or external IdP                 │
│  ├── Permission Sets → mapped to accounts               │
│  └── Recommended for multi-account management           │
│                                                          │
│  Custom Identity Broker:                                │
│  ├── Your app calls STS directly                        │
│  ├── AssumeRole or GetFederationToken                   │
│  └── For legacy or custom auth systems                  │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `AccessDenied` despite Allow policy | Explicit Deny elsewhere (SCP, boundary, resource policy) | Use IAM Policy Simulator or Access Analyzer |
| Cross-account assume fails | Trust policy missing or ExternalId mismatch | Check principal ARN and conditions in trust policy |
| Permissions boundary too restrictive | Boundary doesn't include needed actions | Update boundary to include required services |
| Federation token expired | Session duration exceeded | Increase `DurationSeconds` (max 12 hours for role) |
| Can't determine effective permissions | Multiple policy types overlapping | Use `aws iam simulate-principal-policy` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Evaluation Order** | Explicit Deny → SCP → Resource Policy → Boundary → Identity |
| **Cross-Account** | Trust policy + AssumeRole + ExternalId for security |
| **Permissions Boundary** | Max permissions ceiling. Effective = Identity ∩ Boundary. |
| **Federation** | SAML 2.0 (enterprise), OIDC (web), IAM Identity Center (SSO) |
| **Session Policies** | Further restrict assumed role sessions |
| **Best Practice** | Least privilege. Use roles, not users. Enable MFA. |

---

## Quick Revision Questions

1. **In what order are IAM policies evaluated?**
   > Explicit Deny (always wins) → SCPs → Resource policies → Permissions boundaries → Identity policies. Default is deny.

2. **What is a permissions boundary?**
   > A managed policy attached to a user/role that defines the maximum permissions they can have. Effective permissions = identity policy ∩ boundary.

3. **How does cross-account role assumption work?**
   > Account B creates a role with a trust policy allowing Account A. Users in Account A call `sts:AssumeRole` to get temporary credentials for Account B.

4. **What is the purpose of ExternalId in cross-account roles?**
   > Prevents the "confused deputy" problem. A third party must know the ExternalId to assume the role, preventing unauthorized access.

5. **What is IAM Identity Center?**
   > The recommended way to manage SSO access across multiple AWS accounts, using a central identity source (built-in directory or external IdP like Active Directory).

---

[← Previous: 9.5 CloudTrail](../09-Monitoring-and-Logging/05-cloudtrail.md) | [Next: 10.2 Organizations & SCPs →](02-organizations-scps.md)

[← Back to README](../README.md)
