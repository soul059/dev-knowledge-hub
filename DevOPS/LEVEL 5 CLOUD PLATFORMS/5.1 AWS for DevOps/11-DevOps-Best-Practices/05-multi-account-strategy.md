# Chapter 11.5: Multi-Account Strategy

## Overview

A **multi-account strategy** is an AWS best practice where workloads, teams, and environments are separated into distinct AWS accounts. This provides strong security boundaries, blast-radius isolation, and simplified billing. **AWS Control Tower** automates the setup and governance of a secure, multi-account AWS environment following AWS best practices.

---

## Why Multi-Account?

```
┌──── Single Account vs Multi-Account ─────────────────────┐
│                                                           │
│  Single Account (anti-pattern):                          │
│  ┌───────────────────────────────────────────┐           │
│  │  Account 123456789012                      │           │
│  │  ├── Dev resources                         │           │
│  │  ├── Staging resources     ← Blast radius │           │
│  │  ├── Production resources    affects ALL   │           │
│  │  ├── Shared services                       │           │
│  │  └── Security tools                        │           │
│  │  IAM policies = only isolation boundary    │           │
│  │  Service quotas shared across everything   │           │
│  └───────────────────────────────────────────┘           │
│                                                           │
│  Multi-Account (best practice):                          │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │  Dev   │ │Staging │ │  Prod  │ │Security│           │
│  │Account │ │Account │ │Account │ │Account │           │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│  Each account:                                           │
│  ├── Hard security boundary (IAM is per-account)        │
│  ├── Independent service quotas                          │
│  ├── Separate billing visibility                         │
│  ├── Isolated blast radius                               │
│  └── Fine-grained SCPs per OU                           │
└───────────────────────────────────────────────────────────┘
```

---

## Recommended Account Structure

```
┌──── AWS Landing Zone Architecture ───────────────────────┐
│                                                           │
│  Organization Root                                       │
│  │                                                        │
│  ├── Management Account (org management + billing only) │
│  │   ⚠ No workloads here                               │
│  │                                                        │
│  ├── OU: Security                                        │
│  │   ├── Log Archive Account                             │
│  │   │   ├── CloudTrail (org trail)                     │
│  │   │   ├── Config aggregator                          │
│  │   │   ├── VPC Flow Logs (centralized)                │
│  │   │   └── S3 buckets (immutable, lifecycle)          │
│  │   │                                                    │
│  │   └── Security Tooling Account                        │
│  │       ├── GuardDuty (delegated admin)                 │
│  │       ├── Security Hub (delegated admin)              │
│  │       ├── Inspector (delegated admin)                 │
│  │       ├── Detective                                   │
│  │       └── Macie (delegated admin)                     │
│  │                                                        │
│  ├── OU: Infrastructure                                  │
│  │   ├── Network Account                                 │
│  │   │   ├── Transit Gateway                             │
│  │   │   ├── Route 53 Hosted Zones                      │
│  │   │   ├── Direct Connect                             │
│  │   │   └── VPN connections                             │
│  │   │                                                    │
│  │   └── Shared Services Account                         │
│  │       ├── Active Directory                            │
│  │       ├── CI/CD pipelines                             │
│  │       ├── Container registries (ECR)                  │
│  │       └── Artifact repositories                       │
│  │                                                        │
│  ├── OU: Workloads                                       │
│  │   ├── OU: Production                                  │
│  │   │   ├── Prod App A Account                         │
│  │   │   ├── Prod App B Account                         │
│  │   │   └── Prod Data Account                          │
│  │   │                                                    │
│  │   └── OU: Non-Production                              │
│  │       ├── Dev Account                                 │
│  │       ├── Staging Account                             │
│  │       └── QA Account                                  │
│  │                                                        │
│  └── OU: Sandbox                                         │
│      └── Experimentation Account(s)                      │
│          (limited budget, restricted SCPs)               │
└───────────────────────────────────────────────────────────┘
```

---

## AWS Control Tower

```
┌──── Control Tower ───────────────────────────────────────┐
│                                                           │
│  Control Tower automates:                                │
│  ┌─────────────────────────────────────────────┐        │
│  │  1. Landing Zone Setup                       │        │
│  │     ├── Management Account                   │        │
│  │     ├── Log Archive Account (auto-created)   │        │
│  │     └── Audit Account (auto-created)         │        │
│  │                                               │        │
│  │  2. Guardrails (Controls)                    │        │
│  │     ├── Preventive → SCPs                    │        │
│  │     │   (e.g., deny root access)            │        │
│  │     ├── Detective → AWS Config Rules         │        │
│  │     │   (e.g., detect public S3)            │        │
│  │     └── Proactive → CloudFormation Hooks     │        │
│  │         (e.g., block before deploy)          │        │
│  │                                               │        │
│  │  3. Account Factory                          │        │
│  │     ├── Standardized account provisioning    │        │
│  │     ├── Pre-configured VPC, IAM, logging     │        │
│  │     └── Service Catalog integration          │        │
│  │                                               │        │
│  │  4. Dashboard                                │        │
│  │     ├── Compliance status across accounts    │        │
│  │     └── Guardrail violations                 │        │
│  └─────────────────────────────────────────────┘        │
│                                                           │
│  ⚠ Control Tower manages ~20 SCPs + Config rules       │
│    automatically — do not modify these manually          │
└───────────────────────────────────────────────────────────┘
```

---

## Account Factory for Terraform (AFT)

```
┌──── AFT Architecture ────────────────────────────────────┐
│                                                           │
│  Developer                                               │
│    │                                                      │
│    │ Submit account request                              │
│    ▼ (Terraform / Git PR)                                │
│  ┌──────────────────────┐                                │
│  │ AFT Management Acct  │                                │
│  │ ┌─────────────────┐  │     ┌──────────────────────┐  │
│  │ │ CodePipeline     │──┼────►│ Control Tower        │  │
│  │ │ (account vending)│  │     │ Account Factory API  │  │
│  │ └─────────────────┘  │     └──────────┬───────────┘  │
│  │ ┌─────────────────┐  │                │              │
│  │ │ Terraform State  │  │                ▼              │
│  │ │ (S3 + DynamoDB)  │  │     ┌──────────────────────┐  │
│  │ └─────────────────┘  │     │ New Account Created   │  │
│  └──────────────────────┘     │ ├── VPC configured    │  │
│                                │ ├── IAM roles set up  │  │
│                                │ ├── Logging enabled   │  │
│                                │ └── Guardrails applied│  │
│                                └──────────────────────┘  │
│                                                           │
│  AFT = GitOps-driven account provisioning               │
│  Supports account customizations via Terraform           │
└───────────────────────────────────────────────────────────┘
```

---

## Cross-Account Access Pattern

```
┌──── Cross-Account Role Assumption ───────────────────────┐
│                                                           │
│  Dev Account (111111111111)     Prod Account (222222222)  │
│  ┌──────────────────────┐      ┌─────────────────────┐  │
│  │                      │      │                     │  │
│  │  Developer Role      │      │  DeployRole         │  │
│  │  Trust: IAM users    │      │  Trust: Dev Account │  │
│  │                      │      │                     │  │
│  │  Policy:             │      │  Policy:            │  │
│  │  Allow sts:          │ ──►  │  Allow deploy       │  │
│  │  AssumeRole on       │      │  actions only       │  │
│  │  Prod DeployRole     │      │                     │  │
│  │                      │      │                     │  │
│  └──────────────────────┘      └─────────────────────┘  │
│                                                           │
│  Pipeline in Dev account assumes DeployRole in Prod      │
│  to deploy application (least privilege)                 │
└───────────────────────────────────────────────────────────┘
```

```bash
# Trust policy on Prod DeployRole
cat <<'EOF' > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::111111111111:role/CICDPipelineRole"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {
        "sts:ExternalId": "deploy-pipeline-2024"
      }
    }
  }]
}
EOF

aws iam create-role \
  --role-name DeployRole \
  --assume-role-policy-document file://trust-policy.json

# Assume role from pipeline
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/DeployRole \
  --role-session-name pipeline-deploy \
  --external-id deploy-pipeline-2024
```

---

## Centralized Networking

```
┌──── Hub-Spoke Network (Transit Gateway) ─────────────────┐
│                                                           │
│  Network Account (Hub)                                   │
│  ┌─────────────────────────────────────────────┐        │
│  │              Transit Gateway                 │        │
│  │  ┌─────────┐ ┌─────────┐ ┌────────────────┐ │        │
│  │  │ Shared   │ │ Prod    │ │ Non-Prod      │ │        │
│  │  │ Svc RT   │ │ RT      │ │ RT            │ │        │
│  │  └────┬────┘ └────┬────┘ └──────┬─────────┘ │        │
│  └───────┼──────────┬┼─────────────┼───────────┘        │
│          │          ││             │                      │
│     ┌────┘    ┌─────┘└────┐    ┌──┘                     │
│     ▼         ▼           ▼    ▼                         │
│  Shared    Prod App A  Prod B  Dev                      │
│  Services  VPC         VPC     VPC                      │
│  VPC                                                     │
│                                                           │
│  ⚠ Prod cannot route to Non-Prod (segmentation)        │
│  ✓ All accounts share Transit Gateway via RAM           │
└───────────────────────────────────────────────────────────┘
```

```bash
# Share Transit Gateway via Resource Access Manager (RAM)
aws ram create-resource-share \
  --name "TGW-Share" \
  --resource-arns "arn:aws:ec2:us-east-1:555555555555:transit-gateway/tgw-abc123" \
  --principals "arn:aws:organizations::123456789012:organization/o-abc123" \
  --allow-external-principals false
```

---

## IAM Identity Center (SSO)

```
┌──── Centralized Access ──────────────────────────────────┐
│                                                           │
│  IAM Identity Center (formerly AWS SSO)                  │
│  ┌──────────────────────────────────────────────┐       │
│  │                                               │       │
│  │  Identity Source:                             │       │
│  │  ├── Identity Center directory               │       │
│  │  ├── Active Directory (AD Connector / AWS AD)│       │
│  │  └── External IdP (Okta, Azure AD)           │       │
│  │                                               │       │
│  │  Permission Sets:                             │       │
│  │  ├── AdministratorAccess                     │       │
│  │  ├── ReadOnlyAccess                          │       │
│  │  ├── DatabaseAdmin (custom)                  │       │
│  │  └── DeveloperAccess (custom)                │       │
│  │                                               │       │
│  │  Assignments:                                 │       │
│  │  ├── DevTeam → DeveloperAccess → Dev Account │       │
│  │  ├── DevTeam → ReadOnlyAccess → Prod Account │       │
│  │  ├── DBAs → DatabaseAdmin → Prod Account     │       │
│  │  └── Admins → Administrator → All Accounts   │       │
│  └──────────────────────────────────────────────┘       │
│                                                           │
│  Users get SSO portal: https://company.awsapps.com      │
│  One login → select account → assume role                │
│  No long-term credentials in member accounts             │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Can't access cross-account resource | Trust policy missing or wrong principal | Verify trust policy ARN matches source role exactly |
| Control Tower drift detected | Manual changes to managed resources | Use Control Tower console to repair; avoid manual edits |
| Account Factory creation slow | Provisioning pipeline stalled | Check Service Catalog provisioned product status |
| Too many accounts to manage | No automation for account vending | Use Account Factory or AFT for standardized provisioning |
| SSO permissions not working | Permission set not assigned to account | Verify assignment: user/group → permission set → account |
| VPC overlap across accounts | No IP address planning | Define CIDR allocation strategy per account/environment |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Multi-Account** | Security isolation, blast-radius containment, independent quotas, clear billing |
| **Control Tower** | Automated landing zone. Guardrails (preventive SCPs + detective Config rules) |
| **Account Factory** | Standardized account provisioning. Pre-configured VPC, IAM, logging. |
| **Landing Zone** | Management + Log Archive + Security Tooling + Workload accounts |
| **Cross-Account** | IAM role assumption with trust policies. Least privilege. |
| **Identity Center** | Centralized SSO. Permission sets assigned to users/groups per account. |
| **Networking** | Transit Gateway shared via RAM. Route table segmentation for isolation. |

---

## Quick Revision Questions

1. **Why use multiple AWS accounts instead of one?**
   > Multiple accounts provide hard security boundaries (IAM is per-account), blast-radius isolation (a misconfiguration affects only one account), independent service quotas, clear cost visibility per team/project, and fine-grained governance via SCPs per OU.

2. **What does AWS Control Tower provide?**
   > Control Tower automates landing zone setup with a management account, log archive account, and audit account. It applies guardrails (preventive SCPs and detective Config rules), provides Account Factory for standardized provisioning, and shows compliance status on a dashboard.

3. **What are the three types of Control Tower guardrails?**
   > Preventive (SCPs — prevent actions), Detective (AWS Config rules — detect non-compliance), and Proactive (CloudFormation Hooks — block non-compliant resources before deployment).

4. **How do pipelines deploy across accounts?**
   > The pipeline in the CI/CD account assumes an IAM role in the target account using `sts:AssumeRole`. The target role has a trust policy that only allows the pipeline role to assume it, following least privilege.

5. **What is IAM Identity Center?**
   > Formerly AWS SSO. It provides centralized authentication for all accounts in an organization. Users log in once to a portal, select an account, and assume a pre-configured permission set — no long-term credentials needed in member accounts.

---

[← Previous: 11.4 Disaster Recovery](04-disaster-recovery.md)

[← Back to README](../README.md)
