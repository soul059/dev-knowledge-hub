# Chapter 6.5: AWS App Runner — Fully Managed Container Service

## Overview

AWS App Runner is a **fully managed service** that makes it easy to deploy containerized web applications and APIs at scale. You provide a container image or source code, and App Runner handles building, deploying, scaling, load balancing, and TLS termination automatically.

---

## App Runner Architecture

```
┌──── App Runner ─────────────────────────────────────────────┐
│                                                              │
│  Source:                        App Runner Service:          │
│  ┌──────────────────┐          ┌─────────────────────┐      │
│  │ ECR Image        │──push──→ │                     │      │
│  │ (or)             │          │  ┌─────┐  ┌─────┐  │      │
│  │ GitHub Repo      │──push──→ │  │Inst │  │Inst │  │      │
│  │ (source code)    │          │  │  1  │  │  2  │  │      │
│  └──────────────────┘          │  └──┬──┘  └──┬──┘  │      │
│                                │     │        │     │      │
│  App Runner handles:           │  ┌──┴────────┴──┐  │      │
│  ✓ Build (if source)          │  │ Load Balancer │  │      │
│  ✓ Deploy                     │  │ (managed)     │  │      │
│  ✓ Scale (0 to N)            │  └───────┬───────┘  │      │
│  ✓ TLS certificate           │          │          │      │
│  ✓ Health checks             └──────────┼──────────┘      │
│  ✓ Logging                              │                  │
│                                ┌────────┴────────┐          │
│                                │ HTTPS endpoint   │          │
│                                │ *.awsapprunner   │          │
│                                │   .com           │          │
│                                └─────────────────┘          │
└──────────────────────────────────────────────────────────────┘
```

---

## Deployment Sources

### Option 1: Container Image (ECR)

```bash
# Create App Runner service from ECR image
aws apprunner create-service \
  --service-name my-api \
  --source-configuration '{
    "AuthenticationConfiguration": {
      "AccessRoleArn": "arn:aws:iam::role/apprunner-ecr-role"
    },
    "AutoDeploymentsEnabled": true,
    "ImageRepository": {
      "ImageIdentifier": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "ImageRepositoryType": "ECR",
      "ImageConfiguration": {
        "Port": "8080",
        "RuntimeEnvironmentVariables": {
          "NODE_ENV": "production",
          "DB_HOST": "mydb.cluster-xxx.us-east-1.rds.amazonaws.com"
        }
      }
    }
  }' \
  --instance-configuration '{
    "Cpu": "1 vCPU",
    "Memory": "2 GB"
  }' \
  --health-check-configuration '{
    "Protocol": "HTTP",
    "Path": "/health",
    "Interval": 10,
    "Timeout": 5,
    "HealthyThreshold": 1,
    "UnhealthyThreshold": 5
  }'
```

### Option 2: Source Code (GitHub)

```bash
# App Runner builds from source using managed runtimes
# Supported: Python 3, Node.js 12/14/16/18, Java 8/11, Go 1, .NET 6, PHP 8, Ruby 3

aws apprunner create-service \
  --service-name my-api \
  --source-configuration '{
    "AuthenticationConfiguration": {
      "ConnectionArn": "arn:aws:apprunner:us-east-1:123456789012:connection/github/xxx"
    },
    "AutoDeploymentsEnabled": true,
    "CodeRepository": {
      "RepositoryUrl": "https://github.com/myorg/my-app",
      "SourceCodeVersion": {
        "Type": "BRANCH",
        "Value": "main"
      },
      "CodeConfiguration": {
        "ConfigurationSource": "API",
        "CodeConfigurationValues": {
          "Runtime": "NODEJS_18",
          "BuildCommand": "npm install",
          "StartCommand": "npm start",
          "Port": "3000"
        }
      }
    }
  }'
```

---

## Auto Scaling Configuration

```bash
# Create auto scaling config (controls min/max instances)
aws apprunner create-auto-scaling-configuration \
  --auto-scaling-configuration-name prod-scaling \
  --max-concurrency 100 \
  --min-size 1 \
  --max-size 10

# MaxConcurrency: requests per instance before scaling
# MinSize: minimum instances (1 = always warm, 0 = scale to zero is NOT supported)
# MaxSize: upper limit
```

```
┌──── Auto Scaling Behavior ──────────────────────────────┐
│                                                          │
│  Requests/sec:  50   100   200   500   50   50          │
│                                                          │
│  Instances:                                              │
│  ┌─┐       ┌─┐  ┌─┐  ┌─┐ ┌─┐ ┌─┐                      │
│  │1│ → → → │1│  │2│  │3│ │5│ │1│  (scale down)         │
│  └─┘       └─┘  └─┘  └─┘ └─┘ └─┘                      │
│                                                          │
│  MaxConcurrency = 100 → new instance per 100 requests   │
│                                                          │
│  Pause instances (low cost) keep container ready         │
│  Active instances ($$$) handle live traffic              │
└──────────────────────────────────────────────────────────┘
```

---

## VPC Connector (Private Resources)

```bash
# Create VPC connector for App Runner to reach private resources
aws apprunner create-vpc-connector \
  --vpc-connector-name prod-vpc \
  --subnets subnet-priv-1a subnet-priv-1b \
  --security-groups sg-app-runner

# Associate with service for access to RDS, ElastiCache, etc.
# Outbound traffic routes through VPC
# Inbound traffic still comes via App Runner's managed endpoint
```

---

## Service Comparison

```
┌──── When to Use What ───────────────────────────────────┐
│                                                          │
│  App Runner                                              │
│  └─ Web apps/APIs, minimal config, auto-scaling          │
│     Best for: Startups, prototypes, simple services      │
│                                                          │
│  ECS Fargate                                             │
│  └─ Full control over task definitions, networking       │
│     Best for: Microservices, batch jobs, complex infra   │
│                                                          │
│  EKS                                                     │
│  └─ Kubernetes-native workloads, multi-cloud portability │
│     Best for: Large teams, K8s ecosystem, hybrid cloud   │
│                                                          │
│  Lambda                                                  │
│  └─ Event-driven, sub-second billing, no containers      │
│     Best for: Glue code, triggers, short-lived tasks     │
│                                                          │
│  Elastic Beanstalk                                       │
│  └─ PaaS with more AWS integration, legacy support       │
│     Best for: Traditional apps, easy migration           │
└──────────────────────────────────────────────────────────┘
```

| Feature | App Runner | ECS Fargate | EKS | Lambda |
|---------|-----------|-------------|-----|--------|
| Setup complexity | Very Low | Medium | High | Low |
| Scaling | Auto | Configurable | Configurable | Auto |
| Min instances | 1 | 0 | 0 | 0 |
| Max timeout | None | None | None | 15 min |
| Container support | Yes | Yes | Yes | Container image |
| VPC integration | VPC Connector | Native | Native | VPC config |
| Pricing | Per instance-hour | Per vCPU/mem/sec | Per node/Fargate | Per request + duration |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Service creation fails | Invalid runtime or build command | Check logs; verify runtime matches your code |
| Can't connect to RDS | No VPC connector | Create VPC connector and associate with service |
| Auto deploy not triggering | Connection not authorized | Re-authorize GitHub connection in App Runner console |
| High latency on first request | Paused instance warming up | Increase `MinSize` for consistent performance |
| Build fails | Missing dependencies | Check `apprunner.yaml` build commands; review build logs |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **App Runner** | Fully managed. Source code or container image → HTTPS endpoint. |
| **Sources** | ECR images or GitHub repos with managed runtimes. |
| **Auto Deploy** | Push to ECR/GitHub triggers automatic redeployment. |
| **Scaling** | Concurrency-based auto scale. Pause idle instances. |
| **VPC Connector** | Connect to private VPC resources (RDS, ElastiCache). |
| **Best For** | Simple web apps, APIs, rapid prototyping, small teams. |

---

## Quick Revision Questions

1. **What are the two source types App Runner supports?**
   > Container images from ECR and source code from GitHub repositories.

2. **How does App Runner auto scaling work?**
   > Based on concurrent requests per instance (MaxConcurrency). When exceeded, new instances are added up to MaxSize.

3. **Can App Runner connect to private VPC resources?**
   > Yes, through a VPC Connector that routes outbound traffic through specified subnets and security groups.

4. **What happens to idle App Runner instances?**
   > They are paused (lower cost) but kept warm to handle traffic quickly when requests arrive.

5. **When would you choose App Runner over ECS Fargate?**
   > When you want the simplest path from code/image to HTTPS endpoint, with minimal infrastructure configuration. ECS Fargate is better for complex microservice architectures.

6. **Does App Runner support automatic deployments?**
   > Yes. Enable `AutoDeploymentsEnabled` to trigger redeployment whenever a new image is pushed to ECR or code is pushed to GitHub.

---

[← Previous: 6.4 Fargate](04-fargate-serverless-containers.md) | [Next: 7.1 CodeCommit →](../07-CICD-on-AWS/01-codecommit.md)

[← Back to README](../README.md)
