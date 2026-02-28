# Chapter 10.2: AWS Organizations & Service Control Policies (SCPs)

## Overview

AWS Organizations lets you **centrally manage multiple AWS accounts** under one organization. Service Control Policies (SCPs) define the **maximum permissions** for accounts and organizational units (OUs), creating guardrails that even account administrators cannot override.

---

## Organizations Structure

```
┌──── AWS Organization ────────────────────────────────────┐
│                                                           │
│  Management Account (root)                                │
│  └── Organization Root                                    │
│      ├── OU: Security                                     │
│      │   ├── Account: Log Archive (333333)               │
│      │   └── Account: Security Tooling (444444)          │
│      │                                                    │
│      ├── OU: Infrastructure                               │
│      │   ├── Account: Networking (555555)                 │
│      │   └── Account: Shared Services (666666)           │
│      │                                                    │
│      ├── OU: Workloads                                    │
│      │   ├── OU: Production                               │
│      │   │   ├── Account: Prod App A (777777)            │
│      │   │   └── Account: Prod App B (888888)            │
│      │   └── OU: Non-Production                           │
│      │       ├── Account: Dev (999999)                   │
│      │       └── Account: Staging (101010)               │
│      │                                                    │
│      └── OU: Sandbox                                      │
│          └── Account: Experiments (121212)                │
│                                                           │
│  Management Account: billing, org management only        │
│  ⚠ Never run workloads in the management account        │
└───────────────────────────────────────────────────────────┘
```

---

## Organization Setup

```bash
# Create organization
aws organizations create-organization --feature-set ALL

# Create organizational unit
aws organizations create-organizational-unit \
  --parent-id r-abc1 \
  --name "Workloads"

# Create member account
aws organizations create-account \
  --email prod-app@company.com \
  --account-name "Production App A"

# Move account to OU
aws organizations move-account \
  --account-id 777777777777 \
  --source-parent-id r-abc1 \
  --destination-parent-id ou-abc1-prod

# Enable all features (if created with consolidated billing only)
aws organizations enable-all-features

# List accounts
aws organizations list-accounts

# List OUs under parent
aws organizations list-organizational-units-for-parent \
  --parent-id r-abc1
```

---

## Service Control Policies (SCPs)

```
┌──── SCP Behavior ────────────────────────────────────────┐
│                                                          │
│  SCPs define MAXIMUM permissions (ceiling, not floor)   │
│                                                          │
│  ┌────────────────────────────────────┐                  │
│  │ SCP: Allow s3, ec2, lambda        │ ← ceiling        │
│  │ ┌──────────────────────────────┐   │                  │
│  │ │ IAM Policy: Allow s3, iam   │   │ ← requested     │
│  │ │ ┌────────────────────────┐   │   │                  │
│  │ │ │ Effective: s3 only    │   │   │ ← intersection  │
│  │ │ │ (iam blocked by SCP)  │   │   │                  │
│  │ │ └────────────────────────┘   │   │                  │
│  │ └──────────────────────────────┘   │                  │
│  └────────────────────────────────────┘                  │
│                                                          │
│  ⚠ SCPs do NOT grant permissions                        │
│  ⚠ SCPs do NOT affect the management account            │
│  ⚠ SCPs affect all users and roles (including root)     │
│  ✓ SCPs are inherited down the OU tree                  │
└──────────────────────────────────────────────────────────┘
```

---

## Common SCP Examples

### Deny Region Access (Region Lock)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2",
            "eu-west-1"
          ]
        },
        "ArnNotLike": {
          "aws:PrincipalARN": [
            "arn:aws:iam::*:role/OrganizationAdmin"
          ]
        }
      }
    }
  ]
}
```

### Prevent Leaving Organization

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyLeaveOrg",
    "Effect": "Deny",
    "Action": "organizations:LeaveOrganization",
    "Resource": "*"
  }]
}
```

### Require Tags on Resources

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "RequireEnvironmentTag",
    "Effect": "Deny",
    "Action": [
      "ec2:RunInstances",
      "ec2:CreateVolume",
      "rds:CreateDBInstance"
    ],
    "Resource": "*",
    "Condition": {
      "Null": {
        "aws:RequestTag/Environment": "true"
      }
    }
  }]
}
```

### Deny Root User Actions

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyRootUser",
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringLike": {
        "aws:PrincipalArn": "arn:aws:iam::*:root"
      }
    }
  }]
}
```

### Protect CloudTrail & GuardDuty

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ProtectSecurityServices",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "guardduty:DeleteDetector",
        "guardduty:DisassociateFromMasterAccount",
        "config:StopConfigurationRecorder",
        "config:DeleteConfigurationRecorder"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## SCP Attachment

```bash
# Create SCP
aws organizations create-policy \
  --name "DenyNonApprovedRegions" \
  --description "Restrict to approved AWS regions" \
  --type SERVICE_CONTROL_POLICY \
  --content file://region-lock-scp.json

# Attach SCP to OU
aws organizations attach-policy \
  --policy-id p-abc123 \
  --target-id ou-abc1-workloads

# List policies for target
aws organizations list-policies-for-target \
  --target-id ou-abc1-workloads \
  --filter SERVICE_CONTROL_POLICY
```

---

## SCP Inheritance

```
┌──── SCP Inheritance ─────────────────────────────────────┐
│                                                          │
│  Root                          SCP: FullAWSAccess       │
│  ├── OU: Workloads             SCP: DenyRegions         │
│  │   ├── OU: Production        SCP: ProtectSecurity     │
│  │   │   └── Account: Prod A                            │
│  │   │       Effective SCPs =                           │
│  │   │       FullAWSAccess ∩ DenyRegions ∩ ProtectSec  │
│  │   │                                                   │
│  │   └── OU: Non-Prod          (no additional SCP)      │
│  │       └── Account: Dev                               │
│  │           Effective SCPs =                           │
│  │           FullAWSAccess ∩ DenyRegions                │
│  │                                                       │
│  └── OU: Sandbox               SCP: RestrictedAccess    │
│      └── Account: Sandbox                               │
│          Effective SCPs =                               │
│          FullAWSAccess ∩ RestrictedAccess                │
│                                                          │
│  ⚠ Default FullAWSAccess SCP must remain attached      │
│     (removing it denies everything)                     │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Action denied despite IAM Allow | SCP blocking the action | Check SCPs at OU and account level |
| Management account unaffected | SCPs don't apply to mgmt account | This is by design; minimize mgmt account usage |
| Removing FullAWSAccess SCP | All actions denied | Always keep FullAWSAccess; add Deny SCPs for restrictions |
| Can't determine effective permissions | Multiple SCPs layered | Use Organizations policy simulator or IAM Access Analyzer |
| Account can't be moved | Target OU doesn't exist or permissions | Verify OU ID and management account permissions |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Organizations** | Multi-account management. Consolidated billing. |
| **OUs** | Group accounts by function: Security, Workloads, Sandbox |
| **SCPs** | Maximum permissions ceiling. Don't grant — only restrict. |
| **SCP Inheritance** | Inherited down OU tree. Intersection of parent + child. |
| **Management Account** | Not affected by SCPs. Use only for org management. |
| **Best Practice** | Deny SCPs for guardrails: region lock, tag enforcement |

---

## Quick Revision Questions

1. **Do SCPs grant permissions?**
   > No. SCPs only define the maximum boundary. You still need IAM policies to grant actual permissions within the SCP limits.

2. **Does an SCP affect the management account?**
   > No. SCPs never apply to the management account. This is why workloads should not run in it.

3. **What happens if you detach the FullAWSAccess SCP?**
   > All actions are implicitly denied for that OU/account since there is no longer any SCP allowing actions.

4. **How do you restrict AWS usage to specific regions?**
   > Create a Deny SCP with `aws:RequestedRegion` condition that denies all actions outside approved regions.

5. **How are SCPs inherited?**
   > SCPs are inherited from parent OUs to child OUs and accounts. Effective permissions are the intersection of all SCPs in the chain.

---

[← Previous: 10.1 IAM Advanced](01-iam-advanced-policies-roles.md) | [Next: 10.3 Secrets Manager & KMS →](03-secrets-manager-kms.md)

[← Back to README](../README.md)
