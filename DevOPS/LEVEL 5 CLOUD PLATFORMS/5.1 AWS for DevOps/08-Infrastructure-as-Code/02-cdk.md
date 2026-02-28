# Chapter 8.2: AWS CDK — Cloud Development Kit

## Overview

The AWS CDK (Cloud Development Kit) lets you define cloud infrastructure using **familiar programming languages** (TypeScript, Python, Java, C#, Go). CDK code synthesizes into CloudFormation templates, giving you the power of a real programming language with the reliability of CloudFormation.

---

## CDK Architecture

```
┌──── CDK Workflow ────────────────────────────────────────┐
│                                                           │
│  CDK Code              Synth              Deploy          │
│  (TypeScript,          (Generate)          (Provision)    │
│   Python, etc.)                                           │
│  ┌──────────────┐    ┌──────────────┐    ┌────────────┐  │
│  │ Constructs   │──→ │ CloudForm.   │──→ │ AWS        │  │
│  │ (L1, L2, L3) │    │ Template     │    │ Resources  │  │
│  │              │    │ (YAML/JSON)  │    │            │  │
│  └──────────────┘    └──────────────┘    └────────────┘  │
│                                                           │
│  cdk synth            cdk deploy          Live infra     │
│                                                           │
│  CDK App                                                  │
│  └── Stack A                                              │
│      ├── Construct 1 (VPC)                                │
│      ├── Construct 2 (ECS Cluster)                        │
│      └── Construct 3 (RDS Instance)                       │
│  └── Stack B                                              │
│      └── Construct 4 (Lambda + API Gateway)               │
└───────────────────────────────────────────────────────────┘
```

---

## Construct Levels

```
┌──── Construct Levels ────────────────────────────────────┐
│                                                          │
│  L1 (Cfn Resources)    — Direct CloudFormation mapping   │
│  ├── CfnBucket, CfnVPC, CfnInstance                     │
│  ├── 1:1 with CloudFormation resource types              │
│  └── You set every property manually                     │
│                                                          │
│  L2 (Curated)          — Opinionated defaults            │
│  ├── Bucket, Vpc, Function                               │
│  ├── Sensible defaults + helper methods                  │
│  ├── bucket.grantRead(lambdaFn) ← auto IAM              │
│  └── Most commonly used level                            │
│                                                          │
│  L3 (Patterns)         — Multi-resource patterns         │
│  ├── ApplicationLoadBalancedFargateService               │
│  ├── LambdaRestApi                                       │
│  └── Provisions ALB + Fargate + SG + IAM in one call    │
└──────────────────────────────────────────────────────────┘
```

---

## Getting Started

```bash
# Install CDK CLI
npm install -g aws-cdk

# Verify installation
cdk --version

# Create a new CDK project (TypeScript)
mkdir my-cdk-app && cd my-cdk-app
cdk init app --language typescript

# Project structure:
# my-cdk-app/
# ├── bin/
# │   └── my-cdk-app.ts        ← App entry point
# ├── lib/
# │   └── my-cdk-app-stack.ts  ← Stack definition
# ├── test/
# ├── cdk.json                  ← CDK config
# ├── package.json
# └── tsconfig.json

# Bootstrap CDK (one-time per account/region)
cdk bootstrap aws://123456789012/us-east-1
```

---

## Example: VPC + ECS Fargate Service

```typescript
// lib/my-app-stack.ts
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ecs_patterns from 'aws-cdk-lib/aws-ecs-patterns';
import { Construct } from 'constructs';

export class MyAppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // L2 Construct — VPC with sensible defaults
    const vpc = new ec2.Vpc(this, 'AppVpc', {
      maxAzs: 2,
      natGateways: 1,
    });

    // L2 Construct — ECS Cluster
    const cluster = new ecs.Cluster(this, 'AppCluster', {
      vpc,
      containerInsights: true,
    });

    // L3 Construct (Pattern) — ALB + Fargate + IAM + SG
    const service = new ecs_patterns.ApplicationLoadBalancedFargateService(
      this, 'ApiService', {
        cluster,
        cpu: 512,
        memoryLimitMiB: 1024,
        desiredCount: 2,
        taskImageOptions: {
          image: ecs.ContainerImage.fromRegistry('amazon/amazon-ecs-sample'),
          containerPort: 80,
          environment: {
            NODE_ENV: 'production',
          },
        },
        publicLoadBalancer: true,
      }
    );

    // Auto scaling
    const scaling = service.service.autoScaleTaskCount({
      minCapacity: 2,
      maxCapacity: 10,
    });
    scaling.scaleOnCpuUtilization('CpuScaling', {
      targetUtilizationPercent: 70,
    });

    // Output the ALB URL
    new cdk.CfnOutput(this, 'LoadBalancerDNS', {
      value: service.loadBalancer.loadBalancerDnsName,
    });
  }
}
```

---

## Example: S3 + Lambda + API Gateway

```typescript
import * as cdk from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';

export class ApiStack extends cdk.Stack {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    // S3 bucket with lifecycle rules
    const bucket = new s3.Bucket(this, 'DataBucket', {
      versioned: true,
      removalPolicy: cdk.RemovalPolicy.RETAIN,
      lifecycleRules: [{
        transitions: [{
          storageClass: s3.StorageClass.INFREQUENT_ACCESS,
          transitionAfter: cdk.Duration.days(90),
        }],
      }],
    });

    // Lambda function
    const fn = new lambda.Function(this, 'ApiHandler', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda'),
      environment: {
        BUCKET_NAME: bucket.bucketName,
      },
      timeout: cdk.Duration.seconds(30),
    });

    // Grant Lambda read/write to S3 (auto-generates IAM policy)
    bucket.grantReadWrite(fn);

    // API Gateway
    new apigateway.LambdaRestApi(this, 'ApiEndpoint', {
      handler: fn,
      proxy: true,
    });
  }
}
```

---

## CDK CLI Commands

```bash
# Synthesize CloudFormation template (preview)
cdk synth

# Compare deployed stack with local changes
cdk diff

# Deploy stack
cdk deploy

# Deploy specific stack
cdk deploy MyAppStack

# Deploy all stacks
cdk deploy --all

# Deploy with auto-approve (no confirmation prompt)
cdk deploy --require-approval never

# Destroy stack
cdk destroy

# List stacks
cdk list
```

---

## CDK vs CloudFormation vs Terraform

| Feature | CloudFormation | CDK | Terraform |
|---------|---------------|-----|-----------|
| Language | YAML/JSON | TypeScript, Python, Java, etc. | HCL |
| Abstraction | Low (resources) | High (L2/L3 constructs) | Medium (modules) |
| State | Managed by AWS | Managed by AWS (via CF) | State file (S3 + DynamoDB) |
| Multi-cloud | AWS only | AWS only | Multi-cloud |
| Loops/Conditions | Limited | Full programming language | HCL expressions |
| Testing | TaskCat | Jest, pytest, etc. | Terratest |
| Learning curve | Medium | Low (if you know the language) | Medium |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `CDKToolkit stack not found` | Bootstrap not run | Run `cdk bootstrap aws://ACCOUNT/REGION` |
| `No stacks found` | Missing app entry point | Check `bin/` file and `cdk.json` app field |
| Circular dependency | Constructs reference each other | Refactor to pass values via props or use Lazy |
| Template too large | > 51,200 bytes | Use nested stacks or assets |
| Drift after manual changes | Resource changed outside CDK | Use `cdk diff` to detect; redeploy to correct |
| `cdk diff` shows unexpected changes | Construct IDs changed | Keep construct IDs stable across refactors |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CDK** | Define infrastructure in TypeScript, Python, Java, C#, Go |
| **Constructs** | L1 (raw CF), L2 (opinionated defaults), L3 (patterns) |
| **Synth** | Generates CloudFormation template from CDK code |
| **Bootstrap** | One-time setup per account/region for CDK deployments |
| **grant Methods** | Auto-generate IAM policies: `bucket.grantRead(fn)` |
| **Patterns** | L3 constructs like `ApplicationLoadBalancedFargateService` |

---

## Quick Revision Questions

1. **What are the three levels of CDK constructs?**
   > L1 (Cfn — raw CloudFormation), L2 (curated with defaults and helpers), L3 (patterns combining multiple resources).

2. **What does `cdk synth` do?**
   > Synthesizes the CDK code into a CloudFormation template without deploying it.

3. **What is CDK bootstrap?**
   > A one-time setup per account/region that creates an S3 bucket and IAM roles needed for CDK deployments.

4. **How does CDK handle IAM permissions?**
   > L2 constructs have `grant*` methods (e.g., `bucket.grantRead(fn)`) that automatically generate the minimum required IAM policies.

5. **What does CDK generate under the hood?**
   > CloudFormation templates. CDK is an abstraction layer on top of CloudFormation.

---

[← Previous: 8.1 CloudFormation](01-cloudformation.md) | [Next: 8.3 SAM →](03-sam.md)

[← Back to README](../README.md)
