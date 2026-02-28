# Chapter 6.1: Amazon ECR — Elastic Container Registry

## Overview

Amazon ECR is a **fully managed Docker container registry** that stores, manages, and deploys container images. It integrates natively with ECS, EKS, and Lambda — making it the standard image registry for AWS container workloads.

---

## ECR Architecture

```
┌──────────── ECR Architecture ──────────────────────────────┐
│                                                             │
│  Developer / CI/CD Pipeline                                │
│  ┌─────────────────┐                                      │
│  │ docker build     │                                      │
│  │ docker tag       │                                      │
│  │ docker push      │──────────►  ┌─────────────────────┐ │
│  └─────────────────┘              │  ECR Registry       │ │
│                                    │  123456789012.dkr   │ │
│                                    │  .ecr.region.       │ │
│                                    │  amazonaws.com      │ │
│                                    │                     │ │
│                                    │  ┌── Repository ──┐│ │
│                                    │  │ my-app         ││ │
│                                    │  │  :latest       ││ │
│                                    │  │  :v1.2.3       ││ │
│                                    │  │  :abc123       ││ │
│                                    │  └────────────────┘│ │
│                                    │                     │ │
│  Consumer Services                │  ┌── Repository ──┐│ │
│  ┌──────┐ ┌──────┐ ┌──────┐     │  │ nginx-proxy    ││ │
│  │ ECS  │ │ EKS  │ │Lambda│◄────│  │  :latest       ││ │
│  └──────┘ └──────┘ └──────┘     │  └────────────────┘│ │
│                                    └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## ECR CLI Workflow

```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true \
  --encryption-configuration encryptionType=KMS \
  --image-tag-mutability IMMUTABLE

# Build, tag, and push
docker build -t my-app:v1.2.3 .
docker tag my-app:v1.2.3 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3

# List images
aws ecr describe-images \
  --repository-name my-app \
  --query 'imageDetails[*].[imageTags[0],imagePushedAt,imageSizeInBytes]' \
  --output table

# Scan image for vulnerabilities
aws ecr start-image-scan \
  --repository-name my-app \
  --image-id imageTag=v1.2.3

# Get scan results
aws ecr describe-image-scan-findings \
  --repository-name my-app \
  --image-id imageTag=v1.2.3
```

---

## Lifecycle Policies

```bash
# Automatically clean up old images
aws ecr put-lifecycle-policy \
  --repository-name my-app \
  --lifecycle-policy-text '{
    "rules": [
      {
        "rulePriority": 1,
        "description": "Keep last 10 tagged images",
        "selection": {
          "tagStatus": "tagged",
          "tagPrefixList": ["v"],
          "countType": "imageCountMoreThan",
          "countNumber": 10
        },
        "action": {"type": "expire"}
      },
      {
        "rulePriority": 2,
        "description": "Delete untagged after 1 day",
        "selection": {
          "tagStatus": "untagged",
          "countType": "sinceImagePushed",
          "countUnit": "days",
          "countNumber": 1
        },
        "action": {"type": "expire"}
      }
    ]
  }'
```

---

## ECR Features

| Feature | Description |
|---------|-------------|
| **Image Scanning** | Scan on push for OS and language vulnerabilities (CVEs) |
| **Immutable Tags** | Prevent image tags from being overwritten |
| **Lifecycle Policies** | Auto-clean old/untagged images |
| **Cross-Region Replication** | Replicate images to other regions |
| **Cross-Account Access** | Share repositories via resource policies |
| **Encryption** | AES-256 (default) or KMS |
| **ECR Public** | Public gallery (like Docker Hub but on AWS) |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **ECR** | Managed Docker registry. Per-account, per-region. |
| **Auth** | `aws ecr get-login-password` → `docker login`. Token valid 12 hours. |
| **Scanning** | Scan on push for CVEs. Integrated with Security Hub. |
| **Immutable** | Prevent tag overwrites for production safety. |
| **Lifecycle** | Auto-expire old images to save storage costs. |

---

## Quick Revision Questions

1. **How do you authenticate Docker to ECR?**
   > Use `aws ecr get-login-password` piped to `docker login` with the ECR registry URL.

2. **What is image tag immutability?**
   > Once an image is pushed with a tag, that tag cannot be overwritten. Prevents accidental deployment of different code under the same tag.

3. **What does ECR image scanning detect?**
   > Known OS and programming language vulnerabilities (CVEs) in the container image layers.

---

[Next: 6.2 ECS →](02-ecs-elastic-container-service.md)

[← Back to README](../README.md)
