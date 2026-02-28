# Chapter 11.3: Automation Strategies on AWS

## Overview

Automation is the backbone of DevOps. On AWS, automation spans infrastructure provisioning, application deployment, configuration management, security enforcement, and operational runbooks. This chapter covers patterns and services for comprehensive automation across the DevOps lifecycle.

---

## Automation Layers

```
┌──── DevOps Automation Stack ─────────────────────────────┐
│                                                           │
│  Layer 5: Governance & Compliance                        │
│  ├── AWS Config Rules (auto-remediate)                   │
│  ├── SCPs (preventive guardrails)                        │
│  └── Security Hub (automated standards checks)           │
│                                                           │
│  Layer 4: Monitoring & Response                          │
│  ├── CloudWatch Alarms → Auto Scaling / SNS              │
│  ├── EventBridge Rules → Lambda (event-driven)           │
│  └── GuardDuty → EventBridge → auto-remediate            │
│                                                           │
│  Layer 3: Application Deployment                         │
│  ├── CodePipeline → CodeBuild → CodeDeploy               │
│  ├── GitHub Actions → OIDC → AWS deployment              │
│  └── ArgoCD / Flux (GitOps for Kubernetes)               │
│                                                           │
│  Layer 2: Configuration Management                       │
│  ├── Systems Manager (SSM) → Run Command / State Mgr    │
│  ├── EC2 User Data / Cloud-init                          │
│  └── Parameter Store / Secrets Manager                   │
│                                                           │
│  Layer 1: Infrastructure Provisioning                    │
│  ├── CloudFormation / CDK / SAM                          │
│  ├── Terraform                                           │
│  └── Service Catalog (self-service provisioning)        │
│                                                           │
│  Foundation: Everything as Code in Git                   │
└───────────────────────────────────────────────────────────┘
```

---

## AWS Systems Manager (SSM)

```
┌──── Systems Manager Capabilities ────────────────────────┐
│                                                           │
│  ┌──────────────────────────────────────────────┐       │
│  │           SSM Agent (on EC2 / on-prem)        │       │
│  └──────────────────────┬───────────────────────┘       │
│                         │                                │
│  ┌──────────┬──────────┬┴────────┬──────────┐           │
│  ▼          ▼          ▼         ▼          ▼           │
│ Run       State      Session  Patch     Inventory      │
│ Command   Manager    Manager  Manager                  │
│ (ad-hoc)  (desired   (SSH     (auto     (collect       │
│           state)     replace) patching) SW/HW info)    │
│                                                         │
│ Automation:          Parameter     OpsCenter:           │
│ (runbooks with       Store:        (operational        │
│  approval steps)     (config &     issues tracking)    │
│                      secrets)                           │
└─────────────────────────────────────────────────────────┘
```

```bash
# Run command on multiple instances
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets Key=tag:Environment,Values=production \
  --parameters 'commands=["yum update -y","systemctl restart nginx"]' \
  --comment "Patch and restart nginx"

# Start a Session Manager session (SSH replacement)
aws ssm start-session --target i-1234567890abcdef0

# Create automation runbook execution
aws ssm start-automation-execution \
  --document-name "AWS-RestartEC2Instance" \
  --parameters '{"InstanceId":["i-1234567890abcdef0"]}'

# Patch Manager — create patch baseline
aws ssm create-patch-baseline \
  --name "ProductionPatchBaseline" \
  --operating-system AMAZON_LINUX_2023 \
  --approval-rules '{
    "PatchRules": [{
      "PatchFilterGroup": {
        "PatchFilters": [
          {"Key":"CLASSIFICATION","Values":["Security"]},
          {"Key":"SEVERITY","Values":["Critical","Important"]}
        ]
      },
      "ApproveAfterDays": 7,
      "ComplianceLevel": "CRITICAL"
    }]
  }'
```

---

## Event-Driven Automation with EventBridge

```
┌──── EventBridge Automation Patterns ─────────────────────┐
│                                                           │
│  Pattern 1: Auto-tag resources                           │
│  CloudTrail (RunInstances) → EventBridge → Lambda        │
│  → Tag with creator, timestamp                           │
│                                                           │
│  Pattern 2: Security response                            │
│  GuardDuty Finding → EventBridge → Lambda                │
│  → Isolate EC2 / Block IP / Notify                       │
│                                                           │
│  Pattern 3: CI/CD trigger                                │
│  ECR Image Push → EventBridge → CodePipeline             │
│  → Deploy new container version                          │
│                                                           │
│  Pattern 4: Cost control                                 │
│  EC2 State Change (running) → EventBridge → Lambda       │
│  → Check tags, terminate if untagged after 1 hour        │
│                                                           │
│  Pattern 5: Self-healing                                 │
│  CloudWatch Alarm → EventBridge → SSM Automation         │
│  → Restart service / Replace instance                    │
└───────────────────────────────────────────────────────────┘
```

### Auto-Tag EC2 on Launch

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": {
    "eventSource": ["ec2.amazonaws.com"],
    "eventName": ["RunInstances"]
  }
}
```

```python
# Lambda function to auto-tag
import boto3

ec2 = boto3.client('ec2')

def handler(event, context):
    detail = event['detail']
    user = detail['userIdentity']['arn']
    
    instances = [
        item['instanceId'] 
        for item in detail['responseElements']['instancesSet']['items']
    ]
    
    ec2.create_tags(
        Resources=instances,
        Tags=[
            {'Key': 'CreatedBy', 'Value': user},
            {'Key': 'LaunchDate', 'Value': detail['eventTime']}
        ]
    )
```

---

## AWS Config Rules with Auto-Remediation

```bash
# Create Config rule to check S3 bucket public access
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "s3-bucket-public-read-prohibited",
    "Source": {
      "Owner": "AWS",
      "SourceIdentifier": "S3_BUCKET_PUBLIC_READ_PROHIBITED"
    },
    "Scope": {
      "ComplianceResourceTypes": ["AWS::S3::Bucket"]
    }
  }'

# Set up auto-remediation
aws configservice put-remediation-configurations \
  --remediation-configurations '[{
    "ConfigRuleName": "s3-bucket-public-read-prohibited",
    "TargetType": "SSM_DOCUMENT",
    "TargetId": "AWS-DisableS3BucketPublicReadWrite",
    "Automatic": true,
    "MaximumAutomaticAttempts": 3,
    "RetryAttemptSeconds": 60
  }]'
```

---

## Infrastructure Automation Patterns

```
┌──── IaC Automation Pipeline ─────────────────────────────┐
│                                                           │
│  Git Push                                                │
│    │                                                      │
│    ▼                                                      │
│  ┌──────────────────────┐                                │
│  │ CodePipeline Source   │                                │
│  └──────────┬───────────┘                                │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                │
│  │ CodeBuild: Validate  │ ← cfn-lint, tflint, cdk synth │
│  │ • Lint templates      │                                │
│  │ • Security scan       │ ← cfn-nag, checkov, tfsec    │
│  │ • Unit tests          │                                │
│  └──────────┬───────────┘                                │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                │
│  │ Deploy to Staging     │ ← CloudFormation Change Set   │
│  │ • Create change set   │   or Terraform Plan          │
│  │ • Execute change set  │                                │
│  └──────────┬───────────┘                                │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                │
│  │ Integration Tests     │ ← Validate infra deployed    │
│  └──────────┬───────────┘                                │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                │
│  │ Manual Approval       │ ← Required for production    │
│  └──────────┬───────────┘                                │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                │
│  │ Deploy to Production  │                                │
│  └──────────────────────┘                                │
│                                                           │
│  Key: Never deploy IaC without validation + staging     │
└───────────────────────────────────────────────────────────┘
```

---

## GitOps for Kubernetes

```
┌──── GitOps Pattern ──────────────────────────────────────┐
│                                                           │
│  Developer                                               │
│    │                                                      │
│    │ Push manifest changes                               │
│    ▼                                                      │
│  ┌──────────┐                                            │
│  │ Git Repo │ ← Single source of truth                  │
│  │ (config) │                                            │
│  └─────┬────┘                                            │
│        │                                                  │
│        ▼                                                  │
│  ┌───────────────────┐                                   │
│  │ ArgoCD / Flux     │ ← Watches Git for changes        │
│  │ (GitOps operator) │                                   │
│  └─────────┬─────────┘                                   │
│            │                                              │
│            │ Reconcile (actual → desired)                │
│            ▼                                              │
│  ┌───────────────────┐                                   │
│  │ EKS Cluster       │                                   │
│  │ (desired state)   │                                   │
│  └───────────────────┘                                   │
│                                                           │
│  Benefits:                                               │
│  • Git = audit trail for all changes                    │
│  • Automatic drift detection and correction             │
│  • Pull-based (cluster pulls; no CI needs cluster creds)│
│  • Easy rollback (git revert)                           │
└───────────────────────────────────────────────────────────┘
```

---

## Automation Decision Matrix

```
┌──── What to Automate ────────────────────────────────────┐
│                                                           │
│  Task                    Tool                 Priority   │
│  ─────────────────────── ──────────────────── ────────── │
│  Infra provisioning      CloudFormation/CDK   Day 1     │
│  App deployment          CodePipeline/GH Act  Day 1     │
│  OS patching             SSM Patch Manager    Week 1    │
│  Security compliance     Config Rules         Week 1    │
│  Secret rotation         Secrets Manager      Week 1    │
│  Incident response       EventBridge+Lambda   Week 2    │
│  Cost alerts             Budgets              Week 2    │
│  Resource tagging        EventBridge+Lambda   Week 2    │
│  Backup management       AWS Backup           Month 1   │
│  Drift detection         Config+CF drift      Month 1   │
│  Chaos engineering       FIS                  Month 2   │
│  Compliance reporting    Security Hub         Month 2   │
│                                                           │
│  Priority: Automate highest-risk manual tasks first     │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SSM Run Command fails | SSM Agent not installed or no IAM role | Ensure `AmazonSSMManagedInstanceCore` policy attached |
| EventBridge rule not triggering | Pattern mismatch | Test with sample event in EventBridge console |
| Auto-remediation loop | Config remediates, another process reverts | Investigate conflicting automation or IaC |
| Lambda timeout in automation | Complex remediation takes too long | Use Step Functions for multi-step workflows |
| Automation changes not tracked | No audit trail | Ensure CloudTrail is on; use Git for IaC changes |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Systems Manager** | Run Command, State Manager, Session Manager, Patch Manager, Automation |
| **EventBridge** | Event-driven automation. Route AWS events to Lambda, SSM, SNS, etc. |
| **Config Rules** | Compliance checks with auto-remediation via SSM documents |
| **IaC Pipeline** | Lint → Security scan → Staging deploy → Tests → Approval → Production |
| **GitOps** | Git as single source of truth. ArgoCD/Flux reconcile desired state. |
| **Priority** | Infra + deployment first. Security + patching next. Compliance ongoing. |

---

## Quick Revision Questions

1. **What is AWS Systems Manager Run Command used for?**
   > Executing commands (shell scripts, PowerShell) remotely on managed instances without SSH. It uses the SSM Agent and supports targeting by tags, instance IDs, or resource groups.

2. **How does EventBridge enable automation?**
   > EventBridge routes events from AWS services (EC2 state changes, GuardDuty findings, CloudTrail API calls) to targets like Lambda, SSM Automation, Step Functions, or SNS based on event pattern rules.

3. **What is the GitOps pattern?**
   > Git is the single source of truth for declarative infrastructure and application configuration. A controller (ArgoCD/Flux) watches the Git repo and automatically reconciles the cluster state to match the desired state.

4. **How do you auto-remediate non-compliant resources?**
   > Use AWS Config rules to detect non-compliance, then configure automatic remediation actions that invoke SSM Automation documents to fix the issue (e.g., disable S3 public access, enable encryption).

5. **What should be automated first in a DevOps transformation?**
   > Infrastructure provisioning (IaC) and application deployment (CI/CD pipeline) — these are Day 1 priorities. Follow with security compliance, OS patching, and incident response automation.

---

[← Previous: 11.2 Cost Optimization](02-cost-optimization.md) | [Next: 11.4 Disaster Recovery →](04-disaster-recovery.md)

[← Back to README](../README.md)
