# Chapter 7.2: AWS CodeBuild — Managed Build Service

## Overview

AWS CodeBuild is a **fully managed continuous integration service** that compiles source code, runs tests, and produces deployable artifacts. It eliminates the need to manage build servers — builds run in isolated containers that scale automatically.

---

## CodeBuild Architecture

```
┌──── CodeBuild Workflow ──────────────────────────────────┐
│                                                           │
│  Source                Build                  Output      │
│  ┌──────────┐   ┌──────────────────┐   ┌──────────────┐ │
│  │CodeCommit│   │ Build Container  │   │ S3 Bucket    │ │
│  │GitHub    │──→│ ┌──────────────┐ │──→│ (Artifacts)  │ │
│  │S3 Bucket │   │ │ buildspec.yml│ │   └──────────────┘ │
│  │Bitbucket │   │ │              │ │                     │
│  └──────────┘   │ │ 1. install   │ │   ┌──────────────┐ │
│                 │ │ 2. pre_build │ │──→│ ECR          │ │
│  Environment    │ │ 3. build     │ │   │ (Docker img) │ │
│  Variables ────→│ │ 4. post_build│ │   └──────────────┘ │
│  (SSM/Secrets)  │ └──────────────┘ │                     │
│                 │                  │   ┌──────────────┐  │
│  VPC Access ───→│  Managed compute │──→│ CloudWatch   │  │
│  (optional)     │  (no servers)    │   │ Logs         │  │
│                 └──────────────────┘   └──────────────┘  │
└───────────────────────────────────────────────────────────┘
```

---

## buildspec.yml

The `buildspec.yml` file defines the build commands. It must be in the root of your source directory (or specified in the project config).

```yaml
version: 0.2

env:
  variables:
    NODE_ENV: "production"
    APP_NAME: "my-api"
  parameter-store:
    DB_PASSWORD: "/myapp/prod/db-password"
  secrets-manager:
    API_KEY: "prod/api-key:API_KEY"

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - echo "Installing dependencies..."
      - npm ci

  pre_build:
    commands:
      - echo "Running linter..."
      - npm run lint
      - echo "Logging into ECR..."
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION |
        docker login --username AWS --password-stdin $ECR_REPO

  build:
    commands:
      - echo "Running tests..."
      - npm test
      - echo "Building application..."
      - npm run build
      - echo "Building Docker image..."
      - docker build -t $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION .
      - docker tag $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION $ECR_REPO:latest

  post_build:
    commands:
      - echo "Pushing Docker image..."
      - docker push $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION
      - docker push $ECR_REPO:latest
      - echo "Writing image definitions..."
      - printf '[{"name":"api","imageUri":"%s"}]' $ECR_REPO:$CODEBUILD_RESOLVED_SOURCE_VERSION > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
    - appspec.yml
    - scripts/**/*
  discard-paths: no

reports:
  jest-reports:
    files:
      - "coverage/clover.xml"
    file-format: CLOVERXML

cache:
  paths:
    - "node_modules/**/*"
```

---

## Build Phase Flow

```
┌──── Build Phases ───────────────────────────────────────┐
│                                                          │
│  SUBMITTED → QUEUED → PROVISIONING → DOWNLOAD_SOURCE    │
│                                                          │
│  → INSTALL → PRE_BUILD → BUILD → POST_BUILD             │
│                                                          │
│  → UPLOAD_ARTIFACTS → FINALIZING → SUCCEEDED            │
│                                                          │
│  Any phase failure → FAILED (remaining phases skipped)  │
│                                                          │
│  ┌─────────┐  ┌──────────┐  ┌───────┐  ┌────────────┐  │
│  │ install │→ │pre_build │→ │ build │→ │ post_build │  │
│  │ deps    │  │ lint/auth│  │ test  │  │ publish    │  │
│  │ tools   │  │ setup    │  │ build │  │ notify     │  │
│  └─────────┘  └──────────┘  └───────┘  └────────────┘  │
│    on-failure   on-failure   on-failure    always*       │
│    commands     commands     commands     (configurable) │
└──────────────────────────────────────────────────────────┘
```

---

## CodeBuild Project Setup

```bash
# Create a CodeBuild project
aws codebuild create-project \
  --name my-api-build \
  --source '{
    "type": "GITHUB",
    "location": "https://github.com/myorg/my-app.git",
    "buildspec": "buildspec.yml"
  }' \
  --environment '{
    "type": "LINUX_CONTAINER",
    "image": "aws/codebuild/amazonlinux2-x86_64-standard:5.0",
    "computeType": "BUILD_GENERAL1_MEDIUM",
    "privilegedMode": true,
    "environmentVariables": [
      {"name": "ECR_REPO", "value": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app"},
      {"name": "AWS_DEFAULT_REGION", "value": "us-east-1"}
    ]
  }' \
  --artifacts '{
    "type": "S3",
    "location": "my-build-artifacts-bucket",
    "packaging": "ZIP"
  }' \
  --service-role arn:aws:iam::role/codebuild-service-role \
  --cache '{"type": "S3", "location": "my-cache-bucket/cache"}'

# Start a build manually
aws codebuild start-build --project-name my-api-build

# View build logs
aws codebuild batch-get-builds --ids my-api-build:build-id-123
```

### Compute Types

| Compute Type | vCPU | Memory | Use Case |
|-------------|------|--------|----------|
| BUILD_GENERAL1_SMALL | 2 | 3 GB | Small projects |
| BUILD_GENERAL1_MEDIUM | 4 | 7 GB | Most builds |
| BUILD_GENERAL1_LARGE | 8 | 15 GB | Large compilations |
| BUILD_GENERAL1_2XLARGE | 72 | 145 GB | Huge monorepos |

---

## Environment Variables

```
┌──── Variable Sources (Priority: highest → lowest) ──────┐
│                                                          │
│  1. Start build override (CLI/API)                      │
│  2. Build project environment variables                  │
│  3. buildspec.yml env section                           │
│                                                          │
│  Built-in Variables:                                     │
│  ┌──────────────────────────────────────────┐            │
│  │ CODEBUILD_BUILD_ID        Build ID       │            │
│  │ CODEBUILD_BUILD_NUMBER    Sequential #   │            │
│  │ CODEBUILD_SRC_DIR         Source dir     │            │
│  │ CODEBUILD_RESOLVED_SOURCE_VERSION        │            │
│  │                           Commit hash    │            │
│  │ AWS_DEFAULT_REGION        Build region   │            │
│  └──────────────────────────────────────────┘            │
│                                                          │
│  Secrets (never store in plaintext):                     │
│  • parameter-store: SSM Parameter Store references       │
│  • secrets-manager: Secrets Manager references           │
└──────────────────────────────────────────────────────────┘
```

---

## VPC Access

```bash
# Build in VPC (access private resources like RDS, ElastiCache)
aws codebuild update-project \
  --name my-api-build \
  --vpc-config '{
    "vpcId": "vpc-abc123",
    "subnets": ["subnet-priv-1a", "subnet-priv-1b"],
    "securityGroupIds": ["sg-build"]
  }'

# ⚠ VPC builds need NAT Gateway for internet access
# (pulling dependencies, pushing to public registries)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `DOWNLOAD_SOURCE` fails | Invalid repo URL or credentials | Check source connection; verify IAM/OAuth token |
| Docker build fails | `privilegedMode` not enabled | Set `privilegedMode: true` in environment config |
| Build timeout | Default 60 min exceeded | Increase `timeoutInMinutes`; optimize build steps |
| Can't access SSM params | Missing IAM permissions | Add `ssm:GetParameters` to CodeBuild service role |
| Artifact upload fails | S3 bucket policy issue | Verify CodeBuild role has `s3:PutObject` on bucket |
| Dependency download slow | No cache configured | Enable S3 or local cache in project settings |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CodeBuild** | Fully managed CI. No build servers. Pay per build minute. |
| **buildspec.yml** | Defines phases: install, pre_build, build, post_build |
| **Artifacts** | Upload to S3; can include imagedefinitions.json for ECS |
| **Secrets** | Use SSM Parameter Store or Secrets Manager references |
| **Docker Builds** | Enable `privilegedMode: true` for Docker-in-Docker |
| **VPC Access** | Optional VPC config for accessing private resources |

---

## Quick Revision Questions

1. **What is the purpose of buildspec.yml?**
   > It defines the build commands, environment variables, artifact locations, and cache settings for a CodeBuild project.

2. **What are the four build phases in buildspec.yml?**
   > `install` (dependencies/tools), `pre_build` (auth/lint), `build` (compile/test), `post_build` (publish/notify).

3. **How do you securely use secrets in CodeBuild?**
   > Reference them from SSM Parameter Store or Secrets Manager in the `env` section of buildspec.yml, never as plaintext env vars.

4. **What is needed to build Docker images in CodeBuild?**
   > Set `privilegedMode: true` in the build environment configuration.

5. **How does CodeBuild pricing work?**
   > Pay per build minute based on the compute type selected. No charge when not building.

6. **Can CodeBuild access private VPC resources?**
   > Yes, by configuring VPC access with subnets and security groups. A NAT Gateway is needed for internet access from VPC builds.

---

[← Previous: 7.1 CodeCommit](01-codecommit.md) | [Next: 7.3 CodeDeploy →](03-codedeploy.md)

[← Back to README](../README.md)
