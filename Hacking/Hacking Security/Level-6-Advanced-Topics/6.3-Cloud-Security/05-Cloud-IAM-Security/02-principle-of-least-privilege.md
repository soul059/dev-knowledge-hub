# Unit 5: Cloud IAM Security — Topic 2: Principle of Least Privilege

## Overview

The **Principle of Least Privilege (PoLP)** is the foundational security concept of granting users, applications, and systems only the minimum permissions necessary to perform their required functions. In cloud environments, where overpermissioning is epidemic, implementing least privilege is critical for reducing attack surface, limiting blast radius, and meeting compliance requirements.

---

## 1. Least Privilege Fundamentals

```
PRINCIPLE OF LEAST PRIVILEGE:

DEFINITION:
  "Every program and every privileged user should 
   operate using the least amount of privilege necessary 
   to complete the job." — Jerome Saltzer

WHY IT MATTERS IN CLOUD:
  → Cloud permissions are additive and broad
  → Default policies often too permissive
  → Managed policies grant more than needed
  → Service accounts often over-privileged
  → Compromise impact is proportional to access

THE PROBLEM:
  Traditional approach:
  User needs access → Give broad role → Works → Move on
  
  Result:
  → 95% of IAM roles grant more than needed
  → Average cloud identity has 40x more permissions than used
  → Standing privileges persist indefinitely
  → Attack blast radius is maximized

THE SOLUTION:
  Least privilege approach:
  1. Identify required actions
  2. Grant minimum role covering those actions
  3. Scope to specific resources
  4. Apply time-based restrictions
  5. Review and adjust regularly

LEAST PRIVILEGE DIMENSIONS:
  
  ┌─────────────────────────────────────┐
  │        LEAST PRIVILEGE              │
  │                                     │
  │  WHO → Minimum identities           │
  │  WHAT → Minimum permissions         │
  │  WHERE → Minimum resource scope     │
  │  WHEN → Minimum time duration       │
  │  HOW → Minimum access method        │
  └─────────────────────────────────────┘
```

---

## 2. Cloud Provider Implementation

```
AWS LEAST PRIVILEGE:

  IAM Policy Best Practices:
  → Use customer-managed policies (not inline)
  → Specify exact actions (not Action: "*")
  → Specify exact resources (not Resource: "*")
  → Use conditions for additional constraints
  → Enable IAM Access Analyzer
  
  GOOD:
  {
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::my-bucket/uploads/*",
    "Condition": {
      "IpAddress": {"aws:SourceIp": "10.0.0.0/8"}
    }
  }
  
  BAD:
  {
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "*"
  }

  Tools:
  → IAM Access Analyzer: Analyze external access
  → IAM Access Advisor: View last-used permissions
  → Policy Simulator: Test policy effects
  → CloudTrail: Audit actual permission usage

AZURE LEAST PRIVILEGE:
  → Use predefined roles over Owner/Contributor
  → Scope at resource level, not subscription
  → Use PIM for just-in-time access
  → Managed identities over service principals
  → Conditional Access for context-based access
  
  # Check role assignments
  Get-AzRoleAssignment | Where-Object {
    $_.RoleDefinitionName -in @("Owner", "Contributor")
  } | Select DisplayName, Scope

GCP LEAST PRIVILEGE:
  → Avoid primitive roles (Owner/Editor/Viewer)
  → Use predefined or custom roles
  → IAM Recommender for right-sizing
  → IAM Conditions for ABAC
  → Organization policies for guardrails
  
  # Check IAM recommendations
  gcloud recommender recommendations list \
    --recommender=google.iam.policy.Recommender \
    --project=PROJECT_ID --location=global
```

---

## 3. Implementation Strategies

```
STRATEGIES FOR LEAST PRIVILEGE:

1. START WITH ZERO ACCESS:
   → Don't assign any access by default
   → Users request access as needed
   → Approve through workflow
   → Document justification

2. ROLE RIGHT-SIZING:
   → Analyze actual permission usage
   → Remove unused permissions
   → Create custom roles matching actual needs
   → Regular review cadence
   
   AWS: CloudTrail → Access Advisor → Reduce
   Azure: Sign-in logs → PIM → Reduce
   GCP: Audit logs → IAM Recommender → Reduce

3. JUST-IN-TIME ACCESS:
   → No standing privileges
   → Request access when needed
   → Time-limited (hours, not days)
   → Approval required
   → Auto-revocation
   
   AWS: AWS SSO temporary credentials
   Azure: PIM activation
   GCP: Temporary IAM bindings

4. JUST-ENOUGH ACCESS:
   → Granular permissions per task
   → Resource-level restrictions
   → Condition-based access
   → Separate read/write roles

5. ACCESS CERTIFICATION:
   → Regular review of all access
   → Managers certify access is needed
   → Remove access that's not justified
   → Quarterly or monthly cadence
   → Automated revocation of uncertified access

IMPLEMENTATION PHASES:
  Phase 1: Discover
  → Inventory all identities and permissions
  → Map actual permission usage
  → Identify overprivileged accounts
  
  Phase 2: Analyze
  → Compare granted vs used permissions
  → Identify unused roles and policies
  → Calculate risk scores
  
  Phase 3: Right-size
  → Remove unused permissions
  → Replace broad roles with specific ones
  → Implement conditions and constraints
  
  Phase 4: Maintain
  → Automated access reviews
  → Just-in-time access workflows
  → Continuous monitoring
  → Regular re-assessment
```

---

## 4. Automation and Tooling

```
TOOLS FOR LEAST PRIVILEGE:

CLOUD-NATIVE:
  AWS:
  → IAM Access Analyzer
  → IAM Access Advisor (last accessed)
  → IAM Policy Simulator
  → CloudTrail log analysis
  → AWS SSO permission sets
  
  Azure:
  → PIM (Privileged Identity Management)
  → Access Reviews
  → Identity Governance
  → Conditional Access
  → Azure AD Identity Protection
  
  GCP:
  → IAM Recommender
  → Policy Analyzer
  → Policy Troubleshooter
  → Asset Inventory
  → Organization Policy Service

THIRD-PARTY:
  → CloudKnox/Entra Permissions Management (Microsoft)
  → Ermetic
  → Sonrai Security
  → Bridgecrew/Prisma Cloud
  → Lacework
  → CrowdStrike Cloud Security

POLICY AS CODE:
  → Define permissions in version control
  → Review changes through pull requests
  → Automated testing of policies
  → Drift detection
  → Terraform/Pulumi for IAM management
  
  Example (Terraform):
  resource "aws_iam_policy" "app_s3" {
    name = "app-s3-read"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [{
        Effect   = "Allow"
        Action   = ["s3:GetObject"]
        Resource = "arn:aws:s3:::app-data/*"
      }]
    })
  }
```

---

## 5. Measuring and Monitoring

```
METRICS FOR LEAST PRIVILEGE:

KEY METRICS:
  → Permission usage ratio (used/granted)
  → Number of admin/owner assignments
  → Standing privilege count
  → Average time to revoke access
  → Orphaned accounts/roles
  → Service accounts with keys
  → Cross-account access count

MONITORING:
  [ ] Alert on new admin/owner assignments
  [ ] Alert on wildcard permission grants
  [ ] Track permission usage trends
  [ ] Monitor for privilege escalation
  [ ] Detect dormant privileged accounts
  [ ] Track access review completion rates

COMMON FINDINGS:
  Finding                    | Risk    | Remediation
  Admin access at org level  | Critical| Scope to specific resource
  Unused admin accounts      | High    | Remove or disable
  SA with owner role         | High    | Create specific custom role
  Wildcard permissions       | High    | Specify exact actions
  No access reviews          | Medium  | Implement quarterly reviews
  No JIT access              | Medium  | Enable PIM/temporary access
  Shared credentials         | High    | Individual accounts + MFA

CONTINUOUS IMPROVEMENT:
  → Monthly permission usage reports
  → Quarterly access reviews
  → Annual IAM architecture review
  → Incident post-mortems review access
  → Benchmark against industry standards
```

---

## Summary Table

| Strategy | Description | Tools |
|----------|-------------|-------|
| Zero Standing Access | No permanent privileges | PIM, JIT systems |
| Role Right-sizing | Match roles to actual usage | IAM Recommender, Access Advisor |
| Resource Scoping | Narrow to specific resources | Conditions, resource policies |
| Time-bounding | Temporary access only | PIM, STS, temporary bindings |
| Access Reviews | Regular certification | Identity Governance tools |
| Policy as Code | Version-controlled IAM | Terraform, CloudFormation |

---

## Revision Questions

1. Why is least privilege especially important in cloud environments?
2. What is the difference between just-in-time and just-enough access?
3. How can cloud-native tools help identify overprivileged accounts?
4. What are the phases of implementing least privilege?
5. What metrics can measure the effectiveness of a least privilege program?

---

*Previous: [01-identity-federation.md](01-identity-federation.md) | Next: [03-service-accounts.md](03-service-accounts.md)*

---

*[Back to README](../README.md)*
