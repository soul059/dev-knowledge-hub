# AWS for DevOps — Comprehensive Study Notes

> **Complete guide to mastering AWS services and DevOps practices on the cloud**

---

## Course Overview

This course covers the essential AWS services and DevOps methodologies needed to build, deploy, monitor, and secure applications on Amazon Web Services. From foundational concepts like IAM and VPC to advanced topics like CI/CD pipelines, Infrastructure as Code, and container orchestration — these notes provide a hands-on, exam-ready reference.

### Who Is This For?

- DevOps engineers transitioning to AWS
- Developers wanting to understand cloud infrastructure
- Students preparing for AWS certifications (Solutions Architect, DevOps Engineer)
- Anyone building production systems on AWS

### How to Use These Notes

1. **Sequential Study** — Follow units in order for a structured learning path
2. **Quick Reference** — Jump to any topic using the Table of Contents
3. **Hands-On Practice** — Use the CLI examples and configurations in your own AWS account
4. **Revision** — Use summary tables and quick revision questions at the end of each chapter

---

## Complete Table of Contents

### Unit 1: AWS Fundamentals
| # | Topic | Link |
|---|-------|------|
| 1.1 | AWS Global Infrastructure | [01-aws-global-infrastructure.md](01-AWS-Fundamentals/01-aws-global-infrastructure.md) |
| 1.2 | Regions and Availability Zones | [02-regions-and-availability-zones.md](01-AWS-Fundamentals/02-regions-and-availability-zones.md) |
| 1.3 | AWS Console, CLI, SDK | [03-aws-console-cli-sdk.md](01-AWS-Fundamentals/03-aws-console-cli-sdk.md) |
| 1.4 | IAM Basics | [04-iam-basics.md](01-AWS-Fundamentals/04-iam-basics.md) |
| 1.5 | AWS Pricing Model | [05-aws-pricing-model.md](01-AWS-Fundamentals/05-aws-pricing-model.md) |

### Unit 2: Compute Services
| # | Topic | Link |
|---|-------|------|
| 2.1 | EC2 Instances | [01-ec2-instances.md](02-Compute-Services/01-ec2-instances.md) |
| 2.2 | EC2 Instance Types and Sizing | [02-ec2-instance-types-and-sizing.md](02-Compute-Services/02-ec2-instance-types-and-sizing.md) |
| 2.3 | AMIs and Launch Templates | [03-amis-and-launch-templates.md](02-Compute-Services/03-amis-and-launch-templates.md) |
| 2.4 | Auto Scaling Groups | [04-auto-scaling-groups.md](02-Compute-Services/04-auto-scaling-groups.md) |
| 2.5 | Elastic Load Balancing (ALB, NLB) | [05-elastic-load-balancing.md](02-Compute-Services/05-elastic-load-balancing.md) |
| 2.6 | Lambda (Serverless) | [06-lambda-serverless.md](02-Compute-Services/06-lambda-serverless.md) |

### Unit 3: Networking (VPC)
| # | Topic | Link |
|---|-------|------|
| 3.1 | VPC Concepts | [01-vpc-concepts.md](03-Networking-VPC/01-vpc-concepts.md) |
| 3.2 | Subnets (Public/Private) | [02-subnets-public-private.md](03-Networking-VPC/02-subnets-public-private.md) |
| 3.3 | Route Tables | [03-route-tables.md](03-Networking-VPC/03-route-tables.md) |
| 3.4 | Internet Gateway and NAT Gateway | [04-internet-gateway-nat-gateway.md](03-Networking-VPC/04-internet-gateway-nat-gateway.md) |
| 3.5 | Security Groups vs NACLs | [05-security-groups-vs-nacls.md](03-Networking-VPC/05-security-groups-vs-nacls.md) |
| 3.6 | VPC Peering and Transit Gateway | [06-vpc-peering-transit-gateway.md](03-Networking-VPC/06-vpc-peering-transit-gateway.md) |

### Unit 4: Storage Services
| # | Topic | Link |
|---|-------|------|
| 4.1 | S3 Buckets, Objects, and Versioning | [01-s3-buckets-objects-versioning.md](04-Storage-Services/01-s3-buckets-objects-versioning.md) |
| 4.2 | S3 Storage Classes | [02-s3-storage-classes.md](04-Storage-Services/02-s3-storage-classes.md) |
| 4.3 | EBS Volumes and Snapshots | [03-ebs-volumes-and-snapshots.md](04-Storage-Services/03-ebs-volumes-and-snapshots.md) |
| 4.4 | EFS (Elastic File System) | [04-efs-elastic-file-system.md](04-Storage-Services/04-efs-elastic-file-system.md) |
| 4.5 | Storage Gateway | [05-storage-gateway.md](04-Storage-Services/05-storage-gateway.md) |

### Unit 5: Database Services
| # | Topic | Link |
|---|-------|------|
| 5.1 | RDS (Relational Databases) | [01-rds-relational-databases.md](05-Database-Services/01-rds-relational-databases.md) |
| 5.2 | Aurora | [02-aurora.md](05-Database-Services/02-aurora.md) |
| 5.3 | DynamoDB (NoSQL) | [03-dynamodb-nosql.md](05-Database-Services/03-dynamodb-nosql.md) |
| 5.4 | ElastiCache | [04-elasticache.md](05-Database-Services/04-elasticache.md) |
| 5.5 | Database Migration | [05-database-migration.md](05-Database-Services/05-database-migration.md) |

### Unit 6: Container Services
| # | Topic | Link |
|---|-------|------|
| 6.1 | ECR (Container Registry) | [01-ecr-container-registry.md](06-Container-Services/01-ecr-container-registry.md) |
| 6.2 | ECS (Elastic Container Service) | [02-ecs-elastic-container-service.md](06-Container-Services/02-ecs-elastic-container-service.md) |
| 6.3 | EKS (Elastic Kubernetes Service) | [03-eks-elastic-kubernetes-service.md](06-Container-Services/03-eks-elastic-kubernetes-service.md) |
| 6.4 | Fargate (Serverless Containers) | [04-fargate-serverless-containers.md](06-Container-Services/04-fargate-serverless-containers.md) |
| 6.5 | App Runner | [05-app-runner.md](06-Container-Services/05-app-runner.md) |

### Unit 7: CI/CD on AWS
| # | Topic | Link |
|---|-------|------|
| 7.1 | CodeCommit | [01-codecommit.md](07-CICD-on-AWS/01-codecommit.md) |
| 7.2 | CodeBuild | [02-codebuild.md](07-CICD-on-AWS/02-codebuild.md) |
| 7.3 | CodeDeploy | [03-codedeploy.md](07-CICD-on-AWS/03-codedeploy.md) |
| 7.4 | CodePipeline | [04-codepipeline.md](07-CICD-on-AWS/04-codepipeline.md) |
| 7.5 | CodeArtifact | [05-codeartifact.md](07-CICD-on-AWS/05-codeartifact.md) |
| 7.6 | Integrating with GitHub Actions | [06-integrating-github-actions.md](07-CICD-on-AWS/06-integrating-github-actions.md) |

### Unit 8: Infrastructure as Code
| # | Topic | Link |
|---|-------|------|
| 8.1 | CloudFormation | [01-cloudformation.md](08-Infrastructure-as-Code/01-cloudformation.md) |
| 8.2 | CDK (Cloud Development Kit) | [02-cdk-cloud-development-kit.md](08-Infrastructure-as-Code/02-cdk-cloud-development-kit.md) |
| 8.3 | SAM (Serverless Application Model) | [03-sam-serverless-application-model.md](08-Infrastructure-as-Code/03-sam-serverless-application-model.md) |
| 8.4 | Terraform with AWS | [04-terraform-with-aws.md](08-Infrastructure-as-Code/04-terraform-with-aws.md) |

### Unit 9: Monitoring and Logging
| # | Topic | Link |
|---|-------|------|
| 9.1 | CloudWatch Logs | [01-cloudwatch-logs.md](09-Monitoring-and-Logging/01-cloudwatch-logs.md) |
| 9.2 | CloudWatch Metrics and Alarms | [02-cloudwatch-metrics-and-alarms.md](09-Monitoring-and-Logging/02-cloudwatch-metrics-and-alarms.md) |
| 9.3 | CloudWatch Dashboards | [03-cloudwatch-dashboards.md](09-Monitoring-and-Logging/03-cloudwatch-dashboards.md) |
| 9.4 | X-Ray (Tracing) | [04-xray-tracing.md](09-Monitoring-and-Logging/04-xray-tracing.md) |
| 9.5 | CloudTrail (Audit) | [05-cloudtrail-audit.md](09-Monitoring-and-Logging/05-cloudtrail-audit.md) |

### Unit 10: Security
| # | Topic | Link |
|---|-------|------|
| 10.1 | IAM Policies and Roles | [01-iam-policies-and-roles.md](10-Security/01-iam-policies-and-roles.md) |
| 10.2 | AWS Organizations | [02-aws-organizations.md](10-Security/02-aws-organizations.md) |
| 10.3 | Secrets Manager and KMS | [03-secrets-manager-and-kms.md](10-Security/03-secrets-manager-and-kms.md) |
| 10.4 | WAF and Shield | [04-waf-and-shield.md](10-Security/04-waf-and-shield.md) |
| 10.5 | Security Hub and GuardDuty | [05-security-hub-and-guardduty.md](10-Security/05-security-hub-and-guardduty.md) |

### Unit 11: DevOps Best Practices
| # | Topic | Link |
|---|-------|------|
| 11.1 | Well-Architected Framework | [01-well-architected-framework.md](11-DevOps-Best-Practices/01-well-architected-framework.md) |
| 11.2 | Cost Optimization | [02-cost-optimization.md](11-DevOps-Best-Practices/02-cost-optimization.md) |
| 11.3 | Automation Strategies | [03-automation-strategies.md](11-DevOps-Best-Practices/03-automation-strategies.md) |
| 11.4 | Disaster Recovery | [04-disaster-recovery.md](11-DevOps-Best-Practices/04-disaster-recovery.md) |
| 11.5 | Multi-Account Strategy | [05-multi-account-strategy.md](11-DevOps-Best-Practices/05-multi-account-strategy.md) |

---

## AWS Services Architecture — Big Picture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              AWS CLOUD PLATFORM                                     │
│                                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐    │
│  │   COMPUTE    │  │  NETWORKING  │  │   STORAGE    │  │     DATABASES        │    │
│  │  ─────────── │  │  ─────────── │  │  ─────────── │  │  ─────────────────── │    │
│  │  EC2         │  │  VPC         │  │  S3          │  │  RDS / Aurora        │    │
│  │  Lambda      │  │  Subnets     │  │  EBS         │  │  DynamoDB            │    │
│  │  ECS / EKS   │  │  Route 53    │  │  EFS         │  │  ElastiCache         │    │
│  │  Fargate     │  │  CloudFront  │  │  Glacier     │  │  Redshift            │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────────┘    │
│                                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐    │
│  │   CI/CD      │  │     IaC      │  │  MONITORING  │  │     SECURITY         │    │
│  │  ─────────── │  │  ─────────── │  │  ─────────── │  │  ─────────────────── │    │
│  │  CodeCommit  │  │  CloudForm.  │  │  CloudWatch  │  │  IAM                 │    │
│  │  CodeBuild   │  │  CDK         │  │  X-Ray       │  │  KMS / Secrets Mgr   │    │
│  │  CodeDeploy  │  │  SAM         │  │  CloudTrail  │  │  WAF / Shield        │    │
│  │  CodePipeline│  │  Terraform   │  │  Config      │  │  GuardDuty           │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────────────┘    │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Quick Start Guide

### Prerequisites

| Tool | Purpose | Install Command |
|------|---------|-----------------|
| AWS CLI v2 | Command-line access | `msiinstaller` / `brew install awscli` |
| Terraform | Infrastructure as Code | `choco install terraform` |
| Docker | Container builds | Docker Desktop |
| kubectl | Kubernetes management | `choco install kubernetes-cli` |
| AWS CDK | IaC in programming languages | `npm install -g aws-cdk` |
| SAM CLI | Serverless local testing | `pip install aws-sam-cli` |

### AWS CLI Configuration

```bash
# Configure default profile
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ****
# Default region name: us-east-1
# Default output format: json

# Verify configuration
aws sts get-caller-identity
```

---

## Legend for Diagrams

| Symbol | Meaning |
|--------|---------|
| `[Box]` | AWS Service or Resource |
| `───>` | Data flow direction |
| `- - >` | Optional / conditional flow |
| `═══>` | Encrypted connection |
| `◉` | Endpoint |
| `☁` | Cloud / Internet |

---

*Created for DevOps Engineering — Level 5 Cloud Platforms*
*Last updated: February 2026*
