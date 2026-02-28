# Chapter 7.4: AWS CodePipeline — Continuous Delivery Pipeline

## Overview

AWS CodePipeline is a **fully managed continuous delivery service** that orchestrates build, test, and deployment workflows. It connects Source → Build → Test → Deploy stages, automating your release process.

---

## CodePipeline Architecture

```
┌──── CodePipeline ────────────────────────────────────────┐
│                                                           │
│  ┌─────────────┐  ┌────────────┐  ┌────────────────────┐ │
│  │   SOURCE     │  │   BUILD    │  │    DEPLOY          │ │
│  │             │  │            │  │                    │ │
│  │ CodeCommit  │→ │ CodeBuild  │→ │ CodeDeploy         │ │
│  │ GitHub      │  │ Jenkins    │  │ ECS                │ │
│  │ S3          │  │            │  │ CloudFormation     │ │
│  │ Bitbucket   │  │            │  │ Elastic Beanstalk  │ │
│  │ ECR         │  │            │  │ S3                 │ │
│  └─────────────┘  └────────────┘  └────────────────────┘ │
│                                                           │
│  Optional Stages:                                         │
│  ┌─────────────┐  ┌────────────┐  ┌────────────────────┐ │
│  │   TEST       │  │  APPROVAL  │  │   INVOKE           │ │
│  │             │  │            │  │                    │ │
│  │ CodeBuild   │  │ Manual     │  │ Lambda Function    │ │
│  │ (run tests) │  │ Review     │  │ Step Functions     │ │
│  └─────────────┘  └────────────┘  └────────────────────┘ │
└───────────────────────────────────────────────────────────┘
```

---

## Multi-Stage Pipeline Example

```
┌──── Production Pipeline ─────────────────────────────────┐
│                                                           │
│  Source       Build        Test         Approval  Deploy  │
│  ┌─────┐    ┌─────┐    ┌──────┐    ┌─────┐    ┌─────┐  │
│  │ Git │ →  │Build│ →  │ QA   │ →  │ ✋  │ →  │Prod │  │
│  │push │    │+Test│    │Deploy│    │Wait │    │Blue/│  │
│  │     │    │+Scan│    │+E2E  │    │     │    │Green│  │
│  └─────┘    └─────┘    └──────┘    └─────┘    └─────┘  │
│                                                           │
│  Artifacts flow through S3 between stages                │
└───────────────────────────────────────────────────────────┘
```

---

## Pipeline Definition (CLI)

```bash
# Create pipeline
aws codepipeline create-pipeline --cli-input-json file://pipeline.json

# Start pipeline execution
aws codepipeline start-pipeline-execution --name my-pipeline

# Get pipeline state
aws codepipeline get-pipeline-state --name my-pipeline

# List pipeline executions
aws codepipeline list-pipeline-executions --pipeline-name my-pipeline
```

### Pipeline JSON Structure

```json
{
  "pipeline": {
    "name": "my-app-pipeline",
    "roleArn": "arn:aws:iam::role/CodePipelineServiceRole",
    "artifactStore": {
      "type": "S3",
      "location": "my-pipeline-artifacts-bucket"
    },
    "stages": [
      {
        "name": "Source",
        "actions": [{
          "name": "SourceAction",
          "actionTypeId": {
            "category": "Source",
            "owner": "AWS",
            "provider": "CodeCommit",
            "version": "1"
          },
          "outputArtifacts": [{"name": "SourceOutput"}],
          "configuration": {
            "RepositoryName": "my-app",
            "BranchName": "main",
            "PollForSourceChanges": "false"
          }
        }]
      },
      {
        "name": "Build",
        "actions": [{
          "name": "BuildAction",
          "actionTypeId": {
            "category": "Build",
            "owner": "AWS",
            "provider": "CodeBuild",
            "version": "1"
          },
          "inputArtifacts": [{"name": "SourceOutput"}],
          "outputArtifacts": [{"name": "BuildOutput"}],
          "configuration": {
            "ProjectName": "my-api-build"
          }
        }]
      },
      {
        "name": "ManualApproval",
        "actions": [{
          "name": "ApprovalAction",
          "actionTypeId": {
            "category": "Approval",
            "owner": "AWS",
            "provider": "Manual",
            "version": "1"
          },
          "configuration": {
            "NotificationArn": "arn:aws:sns:us-east-1:123456789012:approval-topic",
            "CustomData": "Review changes and approve for production deployment"
          }
        }]
      },
      {
        "name": "Deploy",
        "actions": [{
          "name": "DeployAction",
          "actionTypeId": {
            "category": "Deploy",
            "owner": "AWS",
            "provider": "CodeDeploy",
            "version": "1"
          },
          "inputArtifacts": [{"name": "BuildOutput"}],
          "configuration": {
            "ApplicationName": "my-app",
            "DeploymentGroupName": "prod"
          }
        }]
      }
    ]
  }
}
```

---

## Parallel and Sequential Actions

```
┌──── Parallel Actions in a Stage ────────────────────────┐
│                                                          │
│  Stage: "Build"                                          │
│                                                          │
│  runOrder: 1 (parallel)     runOrder: 2 (after both)   │
│  ┌──────────┐ ┌──────────┐  ┌──────────────────────┐   │
│  │ Build    │ │ Lint     │  │ Integration Tests    │   │
│  │ App      │ │ Check    │  │ (depends on build)   │   │
│  └──────────┘ └──────────┘  └──────────────────────┘   │
│                                                          │
│  Actions with same runOrder execute in parallel.         │
│  Actions with higher runOrder wait for lower to finish. │
└──────────────────────────────────────────────────────────┘
```

---

## EventBridge Integration (Triggers)

```bash
# Use EventBridge (not polling) for pipeline triggers
# EventBridge rule to detect CodeCommit push → start pipeline

# EventBridge rule created automatically when
# PollForSourceChanges = false (recommended)

# Custom EventBridge rule example:
aws events put-rule \
  --name "pipeline-trigger" \
  --event-pattern '{
    "source": ["aws.codecommit"],
    "detail-type": ["CodeCommit Repository State Change"],
    "resources": ["arn:aws:codecommit:us-east-1:123456789012:my-app"],
    "detail": {
      "event": ["referenceUpdated"],
      "referenceName": ["main"]
    }
  }'
```

---

## Cross-Region and Cross-Account

```
┌──── Cross-Account Pipeline ─────────────────────────────┐
│                                                          │
│  Account A (Dev/Tools)        Account B (Production)    │
│  ┌──────────────────┐        ┌──────────────────────┐   │
│  │ Source → Build    │───────→│ Deploy               │   │
│  │                  │        │                      │   │
│  │ Pipeline runs    │ Cross- │ CodeDeploy runs      │   │
│  │ here             │ acct   │ with assumed role    │   │
│  │                  │ role   │                      │   │
│  │ Artifact S3      │ ←KMS──→│ Access artifacts    │   │
│  │ (encrypted)      │        │ via cross-acct key  │   │
│  └──────────────────┘        └──────────────────────┘   │
│                                                          │
│  Requirements:                                           │
│  • Cross-account IAM roles with trust policies           │
│  • KMS key accessible from both accounts                │
│  • S3 bucket policy allowing cross-account access       │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pipeline not triggering | Using polling instead of EventBridge | Set `PollForSourceChanges: false`; verify EventBridge rule |
| Stage stuck at Approval | No one approved/rejected | Check SNS notification delivery; approve/reject in console |
| Artifact not found | Mismatched artifact names between stages | Ensure `outputArtifacts` name matches next stage's `inputArtifacts` |
| Cross-account deploy fails | IAM trust policy missing | Add pipeline account to deploy role's trust policy |
| Insufficient permissions | Pipeline role too restrictive | Add required actions to CodePipeline service role |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CodePipeline** | Orchestrates CI/CD workflow: Source → Build → Test → Deploy |
| **Stages** | Source, Build, Test, Approval, Deploy; sequential order |
| **Actions** | Same `runOrder` = parallel. Higher `runOrder` = sequential. |
| **Artifacts** | Passed via S3 between stages using named references |
| **Triggers** | EventBridge (recommended) or polling |
| **Cross-Account** | IAM roles + KMS + S3 bucket policies for multi-account |

---

## Quick Revision Questions

1. **What role does CodePipeline play in CI/CD?**
   > It orchestrates the entire CI/CD workflow, connecting source, build, test, approval, and deployment stages together.

2. **How are artifacts passed between stages?**
   > Through S3. Each action defines `outputArtifacts` that downstream actions reference as `inputArtifacts`.

3. **How do you implement manual approval gates?**
   > Add a Manual Approval action with an optional SNS notification. The pipeline pauses until someone approves or rejects in the console/CLI.

4. **What triggers a pipeline execution?**
   > EventBridge rules (recommended) detect source changes and start the pipeline. Polling or manual starts are also possible.

5. **Can actions within a stage run in parallel?**
   > Yes. Actions with the same `runOrder` value run in parallel. Actions with different `runOrder` values run sequentially.

---

[← Previous: 7.3 CodeDeploy](03-codedeploy.md) | [Next: 7.5 CodeArtifact →](05-codeartifact.md)

[← Back to README](../README.md)
