# Chapter 7.6: GitHub Actions with AWS

## Overview

GitHub Actions is a popular CI/CD platform that integrates seamlessly with AWS services. Using **OIDC federation** (recommended) or IAM access keys, workflows can build, test, and deploy to AWS without storing long-lived credentials.

---

## Authentication: OIDC vs Access Keys

```
┌──── OIDC Federation (Recommended) ──────────────────────┐
│                                                          │
│  GitHub Actions         AWS IAM                         │
│  ┌──────────────┐      ┌────────────────────────┐       │
│  │ Workflow runs │      │ OIDC Identity Provider │       │
│  │              │─────→│ (GitHub)                │       │
│  │ Requests     │ JWT  │                        │       │
│  │ short-lived  │ token│ Trust Policy:           │       │
│  │ credentials  │      │ repo:myorg/myrepo:*    │       │
│  └──────────────┘      └──────────┬─────────────┘       │
│                                   │ AssumeRoleWithWebId │
│                        ┌──────────▼─────────────┐       │
│                        │ IAM Role               │       │
│                        │ (short-lived creds)     │       │
│                        └────────────────────────┘       │
│                                                          │
│  ✓ No stored secrets                                    │
│  ✓ Short-lived credentials                              │
│  ✓ Scoped to specific repos/branches                    │
│                                                          │
├──── Access Keys (Legacy) ────────────────────────────────┤
│                                                          │
│  ⚠ Long-lived credentials stored as GitHub Secrets      │
│  ⚠ Must rotate regularly                               │
│  ⚠ Risk of credential exposure                         │
└──────────────────────────────────────────────────────────┘
```

---

## Setting Up OIDC Federation

### Step 1: Create IAM Identity Provider

```bash
# Create OIDC provider in AWS
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

### Step 2: Create IAM Role with Trust Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:myorg/my-app:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

---

## Complete Workflow: Build and Deploy to ECS

```yaml
# .github/workflows/deploy.yml
name: Deploy to ECS

on:
  push:
    branches: [main]

permissions:
  id-token: write    # Required for OIDC
  contents: read

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: my-app
  ECS_CLUSTER: prod-cluster
  ECS_SERVICE: api-service
  TASK_DEFINITION: task-def.json
  CONTAINER_NAME: api

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # 1. Checkout code
      - name: Checkout
        uses: actions/checkout@v4

      # 2. Configure AWS credentials (OIDC)
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::role/GitHubActionsDeployRole
          aws-region: ${{ env.AWS_REGION }}

      # 3. Login to ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # 4. Build and push Docker image
      - name: Build, tag, and push image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      # 5. Update ECS task definition
      - name: Update task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: ${{ env.TASK_DEFINITION }}
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}

      # 6. Deploy to ECS
      - name: Deploy to Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

---

## Other Common AWS Workflows

### Deploy to S3 (Static Website)

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::role/GitHubS3DeployRole
          aws-region: us-east-1

      - name: Build
        run: npm ci && npm run build

      - name: Deploy to S3
        run: aws s3 sync ./dist s3://my-website-bucket --delete

      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id E1234567890 \
            --paths "/*"
```

### Deploy Lambda Function

```yaml
jobs:
  deploy-lambda:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::role/GitHubLambdaRole
          aws-region: us-east-1

      - name: Package and deploy
        run: |
          zip -r function.zip . -x ".git/*" ".github/*"
          aws lambda update-function-code \
            --function-name my-function \
            --zip-file fileb://function.zip
```

### Run CloudFormation

```yaml
      - name: Deploy CloudFormation stack
        uses: aws-actions/aws-cloudformation-github-deploy@v1
        with:
          name: my-stack
          template: infrastructure/template.yml
          parameter-overrides: "Env=prod,InstanceType=t3.medium"
          no-fail-on-empty-changeset: "1"
```

---

## Official AWS GitHub Actions

| Action | Purpose |
|--------|---------|
| `aws-actions/configure-aws-credentials` | OIDC or access key authentication |
| `aws-actions/amazon-ecr-login` | Authenticate Docker to ECR |
| `aws-actions/amazon-ecs-render-task-definition` | Update image in task definition JSON |
| `aws-actions/amazon-ecs-deploy-task-definition` | Deploy task definition to ECS service |
| `aws-actions/aws-cloudformation-github-deploy` | Deploy CloudFormation stacks |
| `aws-actions/setup-sam` | Install AWS SAM CLI |
| `aws-actions/aws-codebuild-run-build` | Trigger CodeBuild from GitHub |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Not authorized to perform sts:AssumeRoleWithWebIdentity` | Trust policy misconfigured | Check OIDC subject claim matches repo/branch |
| `id-token: write` error | Missing permissions block | Add `permissions: id-token: write` to workflow |
| ECR push fails | Role missing ECR permissions | Add `ecr:GetAuthorizationToken`, `ecr:PutImage` to role |
| Workflow not triggering | Wrong branch filter | Verify `on.push.branches` matches your branch |
| Credentials expired mid-job | Session duration too short | Set `role-duration-seconds` (default 1 hour, max 12) |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **OIDC** | No stored secrets. Short-lived creds. Scoped to repo/branch. |
| **AWS Actions** | Pre-built actions for ECR, ECS, CloudFormation, SAM |
| **ECS Deploy** | Build image → push ECR → render task def → deploy service |
| **S3 Deploy** | `aws s3 sync` + CloudFront invalidation |
| **Trust Policy** | Controls which repos/branches can assume the IAM role |
| **Best Practice** | Always use OIDC over access keys for GitHub Actions |

---

## Quick Revision Questions

1. **Why is OIDC federation preferred over access keys for GitHub Actions?**
   > OIDC uses short-lived credentials without storing secrets. No rotation needed. Can be scoped to specific repositories and branches.

2. **What GitHub Actions permission is required for OIDC?**
   > `id-token: write` must be set in the workflow's `permissions` block.

3. **How do you restrict which repos/branches can assume an AWS role?**
   > Use the `Condition` block in the IAM trust policy with `token.actions.githubusercontent.com:sub` matching `repo:org/repo:ref:refs/heads/branch`.

4. **What is the typical ECS deployment workflow?**
   > Checkout → Configure AWS creds → Login to ECR → Build & push image → Update task definition → Deploy to ECS service.

5. **Name three official AWS GitHub Actions.**
   > `configure-aws-credentials` (auth), `amazon-ecr-login` (Docker auth to ECR), `amazon-ecs-deploy-task-definition` (ECS deploy).

---

[← Previous: 7.5 CodeArtifact](05-codeartifact.md) | [Next: 8.1 CloudFormation →](../08-Infrastructure-as-Code/01-cloudformation.md)

[← Back to README](../README.md)
