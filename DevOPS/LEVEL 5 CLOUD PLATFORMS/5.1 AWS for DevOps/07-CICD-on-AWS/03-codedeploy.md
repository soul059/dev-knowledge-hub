# Chapter 7.3: AWS CodeDeploy — Automated Deployment Service

## Overview

AWS CodeDeploy automates application deployments to **EC2 instances**, **on-premises servers**, **Lambda functions**, and **ECS services**. It supports rolling updates, blue/green deployments, and automatic rollback on failure.

---

## CodeDeploy Architecture

```
┌──── CodeDeploy Flow ─────────────────────────────────────┐
│                                                           │
│  Revision (S3/GitHub)                                     │
│  ┌──────────────────┐                                     │
│  │ appspec.yml      │                                     │
│  │ + app bundle     │                                     │
│  └────────┬─────────┘                                     │
│           │                                               │
│           ▼                                               │
│  ┌──────────────────┐     ┌─────────────────────────┐     │
│  │ CodeDeploy       │     │ Deployment Group        │     │
│  │ Application      │────→│ ┌─────┐ ┌─────┐ ┌─────┐│     │
│  │                  │     │ │ i-1 │ │ i-2 │ │ i-3 ││     │
│  └──────────────────┘     │ └──┬──┘ └──┬──┘ └──┬──┘│     │
│                           │    │       │       │   │     │
│  Deployment Config:       │ CodeDeploy Agent running│     │
│  • OneAtATime             │ on each instance        │     │
│  • HalfAtATime            └─────────────────────────┘     │
│  • AllAtOnce                                              │
│  • Custom (25% at a time)                                 │
└───────────────────────────────────────────────────────────┘
```

---

## appspec.yml — EC2/On-Premises

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /opt/myapp
    overwrite: yes

permissions:
  - object: /opt/myapp
    owner: appuser
    group: appuser
    mode: 755
    type:
      - directory
  - object: /opt/myapp/bin
    owner: appuser
    group: appuser
    mode: 755
    type:
      - file

hooks:
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 120
      runas: root

  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 120
      runas: root

  AfterInstall:
    - location: scripts/after_install.sh
      timeout: 120
      runas: root

  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 120
      runas: root

  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

### Lifecycle Event Order (EC2)

```
┌──── In-Place Deployment Hooks ──────────────────────────┐
│                                                          │
│  ┌─────────────────────┐                                │
│  │ ApplicationStop     │ ← Stop current app             │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ DownloadBundle      │ ← (automatic — no hook)        │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ BeforeInstall       │ ← Decrypt files, backup        │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ Install             │ ← (automatic — copy files)     │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ AfterInstall        │ ← Configure, set permissions   │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ ApplicationStart    │ ← Start new app version        │
│  └─────────┬───────────┘                                │
│            ▼                                             │
│  ┌─────────────────────┐                                │
│  │ ValidateService     │ ← Health check / smoke test    │
│  └─────────────────────┘                                │
└──────────────────────────────────────────────────────────┘
```

---

## Deployment Strategies

### In-Place Deployment

```
┌──── In-Place (HalfAtATime) ─────────────────────────────┐
│                                                          │
│  Step 1: Deploy to 50%         Step 2: Deploy to rest    │
│  ┌────┐ ┌────┐ ┌────┐        ┌────┐ ┌────┐ ┌────┐     │
│  │v2  │ │v1  │ │v1  │   →    │v2  │ │v2  │ │v2  │     │
│  │ ✓  │ │    │ │    │        │ ✓  │ │ ✓  │ │ ✓  │     │
│  └────┘ └────┘ └────┘        └────┘ └────┘ └────┘     │
│                                                          │
│  ⚠ Downtime per instance during update                  │
│  ✓ No extra infrastructure cost                         │
└──────────────────────────────────────────────────────────┘
```

### Blue/Green Deployment

```
┌──── Blue/Green ──────────────────────────────────────────┐
│                                                          │
│  Step 1: Create Green          Step 2: Switch traffic    │
│                                                          │
│  BLUE (current)   GREEN (new)  BLUE        GREEN        │
│  ┌────┐ ┌────┐   ┌────┐       ┌────┐      ┌────┐      │
│  │v1  │ │v1  │   │v2  │       │v1  │      │v2  │      │
│  └──┬─┘ └──┬─┘   └──┬─┘       └────┘      └──┬─┘      │
│     └───┬──┘        │                         │         │
│         │           │              ┌──────────┘         │
│    ┌────┴────┐      │         ┌────┴────┐               │
│    │   ALB   │      │         │   ALB   │               │
│    └─────────┘      │         └─────────┘               │
│         ↑           │              ↑                    │
│      traffic        │           traffic                 │
│                                                          │
│  ✓ Zero downtime                                        │
│  ✓ Instant rollback (switch back to Blue)               │
│  ⚠ Double infrastructure during deployment              │
└──────────────────────────────────────────────────────────┘
```

### Deployment Configurations

| Configuration | Description |
|--------------|-------------|
| `CodeDeployDefault.OneAtATime` | One instance at a time |
| `CodeDeployDefault.HalfAtATime` | 50% of instances at a time |
| `CodeDeployDefault.AllAtOnce` | All instances simultaneously |
| Custom | Define min healthy percentage |

---

## appspec.yml — Lambda

```yaml
version: 0.0
Resources:
  - MyFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: my-function
        Alias: live
        CurrentVersion: 1
        TargetVersion: 2

Hooks:
  - BeforeAllowTraffic: CodeDeployHook_PreTrafficTest
  - AfterAllowTraffic: CodeDeployHook_PostTrafficTest
```

### Lambda Deployment Types

```
┌──── Lambda Traffic Shifting ────────────────────────────┐
│                                                          │
│  Linear:     10% every 10 minutes until 100%            │
│  ├── LambdaLinear10PercentEvery1Minute                  │
│  ├── LambdaLinear10PercentEvery2Minutes                 │
│  └── LambdaLinear10PercentEvery10Minutes                │
│                                                          │
│  Canary:     10% first, then 90% after X minutes        │
│  ├── LambdaCanary10Percent5Minutes                      │
│  ├── LambdaCanary10Percent10Minutes                     │
│  └── LambdaCanary10Percent30Minutes                     │
│                                                          │
│  AllAtOnce:  Immediate 100% shift                       │
│  └── LambdaAllAtOnce                                    │
└──────────────────────────────────────────────────────────┘
```

---

## appspec.yml — ECS

```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: "arn:aws:ecs:us-east-1:123456789012:task-definition/my-app:2"
        LoadBalancerInfo:
          ContainerName: "my-app"
          ContainerPort: 8080
        PlatformVersion: "LATEST"

Hooks:
  - BeforeInstall: "LambdaFunctionForSanityCheck"
  - AfterInstall: "LambdaFunctionForIntegrationTest"
  - BeforeAllowTraffic: "LambdaFunctionForSmokeTest"
  - AfterAllowTraffic: "LambdaFunctionForPostDeployValidation"
```

---

## CLI Commands

```bash
# Create application
aws deploy create-application \
  --application-name my-app \
  --compute-platform Server

# Create deployment group
aws deploy create-deployment-group \
  --application-name my-app \
  --deployment-group-name prod \
  --service-role-arn arn:aws:iam::role/CodeDeployServiceRole \
  --deployment-config-name CodeDeployDefault.HalfAtATime \
  --auto-scaling-groups my-asg \
  --auto-rollback-configuration '{
    "enabled": true,
    "events": ["DEPLOYMENT_FAILURE", "DEPLOYMENT_STOP_ON_ALARM"]
  }' \
  --alarm-configuration '{
    "enabled": true,
    "alarms": [{"name": "HighCPU"}, {"name": "High5xx"}]
  }'

# Create deployment
aws deploy create-deployment \
  --application-name my-app \
  --deployment-group-name prod \
  --s3-location bucket=my-artifacts,key=app.zip,bundleType=zip
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Agent not reporting | CodeDeploy agent not installed/running | Install agent; check `codedeploy-agent` service status |
| `AllowTraffic` timeout | Health check failing on new instances | Verify ALB health check path and port |
| Script failed | Permissions or missing dependencies | Check hook script logs in `/var/log/aws/codedeploy-agent/` |
| Rollback triggered | Deployment or CloudWatch alarm failure | Check alarm thresholds; review deployment logs |
| Stuck in pending | No healthy instances in deployment group | Verify ASG instances are InService with agent running |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CodeDeploy** | Automated deployments to EC2, Lambda, ECS, on-premises |
| **appspec.yml** | Defines files, permissions, lifecycle hooks per platform |
| **In-Place** | Update existing instances; brief downtime per instance |
| **Blue/Green** | New instances → switch traffic → zero downtime |
| **Rollback** | Automatic on failure or CloudWatch alarm breach |
| **Lambda** | Linear, Canary, or AllAtOnce traffic shifting |

---

## Quick Revision Questions

1. **What are the three compute platforms CodeDeploy supports?**
   > EC2/On-Premises (Server), AWS Lambda, and Amazon ECS.

2. **What is the difference between in-place and blue/green deployments?**
   > In-place updates existing instances (brief downtime). Blue/green creates new instances and switches traffic (zero downtime, instant rollback).

3. **What is appspec.yml?**
   > The deployment specification file that defines which files to copy, permissions, and lifecycle hook scripts for each deployment phase.

4. **What hook runs last in an EC2 in-place deployment?**
   > `ValidateService` — used for health checks and smoke tests after the application starts.

5. **How does automatic rollback work?**
   > CodeDeploy can automatically rollback to the last known good revision when a deployment fails or a linked CloudWatch alarm triggers.

---

[← Previous: 7.2 CodeBuild](02-codebuild.md) | [Next: 7.4 CodePipeline →](04-codepipeline.md)

[← Back to README](../README.md)
