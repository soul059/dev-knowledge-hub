# Chapter 6.2: Amazon ECS — Elastic Container Service

## Overview

Amazon ECS is AWS's **container orchestration service** that runs Docker containers at scale. It manages scheduling, scaling, and networking of containers on a cluster of EC2 instances or Fargate (serverless).

---

## ECS Architecture

```
┌──────────── ECS Architecture ──────────────────────────────┐
│                                                             │
│  ┌── ECS Cluster ──────────────────────────────────────┐   │
│  │                                                      │   │
│  │  ┌── Service: web-app (desired: 3) ──────────────┐  │   │
│  │  │                                                │  │   │
│  │  │  ┌── Task ──┐  ┌── Task ──┐  ┌── Task ──┐    │  │   │
│  │  │  │ nginx    │  │ nginx    │  │ nginx    │    │  │   │
│  │  │  │ app      │  │ app      │  │ app      │    │  │   │
│  │  │  │ (2 cont.)│  │ (2 cont.)│  │ (2 cont.)│    │  │   │
│  │  │  └──────────┘  └──────────┘  └──────────┘    │  │   │
│  │  │                                                │  │   │
│  │  │  Task Definition (blueprint):                 │  │   │
│  │  │  • Container images (ECR)                     │  │   │
│  │  │  • CPU/Memory                                 │  │   │
│  │  │  • Port mappings                              │  │   │
│  │  │  • Environment variables                      │  │   │
│  │  │  • IAM Task Role                              │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  │                                                      │   │
│  │  Launch Type:                                        │   │
│  │  ├── EC2:     You manage the instances              │   │
│  │  └── Fargate: AWS manages infrastructure (serverless)│   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### ECS Concepts

| Concept | Description |
|---------|-------------|
| **Cluster** | Logical grouping of tasks or services |
| **Task Definition** | Blueprint: image, CPU, memory, ports, volumes, IAM role |
| **Task** | Running instance of a task definition (1+ containers) |
| **Service** | Maintains desired count of tasks, integrates with ALB |
| **Container** | Individual Docker container within a task |

---

## Task Definition

```json
{
  "family": "web-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::role/ecsAppRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "NODE_ENV", "value": "production"}
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/web-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "app"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3
      }
    }
  ]
}
```

---

## ECS with ALB

```
┌──── ECS Service + ALB ─────────────────────────────────────┐
│                                                             │
│          Internet                                           │
│              │                                              │
│       ┌──────▼──────┐                                      │
│       │    ALB      │  Listener: 443 → Target Group       │
│       └──────┬──────┘                                      │
│              │                                              │
│     ┌────────┼────────┐  Target Group (IP type)            │
│     │        │        │                                    │
│  ┌──▼──┐ ┌──▼──┐ ┌──▼──┐                                │
│  │Task │ │Task │ │Task │  ECS auto-registers tasks       │
│  │:8080│ │:8080│ │:8080│  with ALB target group           │
│  └─────┘ └─────┘ └─────┘                                │
│                                                             │
│  Key: With awsvpc network mode, each task gets its own     │
│  ENI and private IP → ALB routes to task IPs directly     │
└─────────────────────────────────────────────────────────────┘
```

---

## ECS CLI Commands

```bash
# Create cluster
aws ecs create-cluster --cluster-name prod-cluster

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-def.json

# Create service with ALB
aws ecs create-service \
  --cluster prod-cluster \
  --service-name web-app \
  --task-definition web-app:3 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration '{
    "awsvpcConfiguration": {
      "subnets": ["subnet-priv-1a", "subnet-priv-1b"],
      "securityGroups": ["sg-app"],
      "assignPublicIp": "DISABLED"
    }
  }' \
  --load-balancers '[{
    "targetGroupArn": "arn:aws:elasticloadbalancing:...:targetgroup/web-tg/xxx",
    "containerName": "app",
    "containerPort": 8080
  }]'

# Update service (rolling deployment)
aws ecs update-service \
  --cluster prod-cluster \
  --service web-app \
  --task-definition web-app:4 \
  --desired-count 3

# Scale service
aws ecs update-service \
  --cluster prod-cluster \
  --service web-app \
  --desired-count 5

# View running tasks
aws ecs list-tasks --cluster prod-cluster --service-name web-app
```

---

## EC2 vs Fargate Launch Type

| Feature | EC2 | Fargate |
|---------|-----|---------|
| Infrastructure | You manage EC2 instances | AWS manages everything |
| Pricing | EC2 instance cost (even if idle) | Per task vCPU + memory/second |
| Scaling | Scale instances + tasks | Scale tasks only |
| OS access | Yes (SSH, agents, monitoring) | No |
| GPU support | Yes | Limited |
| Storage | Instance store + EBS | Ephemeral (20 GB) + EFS |
| Best for | Cost optimization at scale, GPU, custom AMI | Simplicity, variable workloads |

---

## ECS Auto Scaling

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/prod-cluster/web-app \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 2 \
  --max-capacity 20

# Target tracking: maintain 70% CPU
aws application-autoscaling put-scaling-policy \
  --service-namespace ecs \
  --resource-id service/prod-cluster/web-app \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 120
  }'
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Task keeps restarting | Health check failing or app crash | Check CloudWatch logs; verify health check endpoint |
| Service stuck at 0 tasks | No resources or SG blocking | Check Fargate/EC2 capacity; verify SG + subnet config |
| Can't pull ECR image | Task execution role missing ECR perms | Attach `AmazonECSTaskExecutionRolePolicy` |
| Task can't reach internet | Fargate in private subnet without NAT | Add NAT Gateway or use VPC endpoints |
| ALB returns 502 | Container not ready, health check timing | Increase health check grace period; fix startup time |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **ECS** | AWS container orchestrator. Manages tasks across clusters. |
| **Task Definition** | Blueprint: image, CPU, memory, ports, IAM roles, logging. |
| **Service** | Maintains desired count, integrates ALB, rolling deploys. |
| **Fargate** | Serverless. No instances. Pay per task. Simplest option. |
| **EC2 Launch** | Manage instances yourself. Cheaper at scale. GPU support. |
| **awsvpc** | Each task gets its own ENI/IP. Required for Fargate. |

---

## Quick Revision Questions

1. **What is the difference between a Task Definition and a Task?**
   > Task Definition is the blueprint (JSON); a Task is a running instance of that definition.

2. **When should you use Fargate vs EC2 launch type?**
   > Fargate for simplicity and variable workloads. EC2 for cost optimization at scale, GPU needs, or when you need OS-level access.

3. **How does ECS integrate with ALB?**
   > ECS automatically registers/deregisters task IPs in the ALB target group as tasks start/stop.

4. **What IAM roles does an ECS task need?**
   > Task Execution Role (for pulling images, writing logs) and Task Role (for the application's AWS API calls).

5. **How do you deploy a new version in ECS?**
   > Update the task definition with the new image tag, then update the service to use the new revision. ECS performs a rolling deployment.

---

[← Previous: 6.1 ECR](01-ecr-elastic-container-registry.md) | [Next: 6.3 EKS →](03-eks-elastic-kubernetes-service.md)

[← Back to README](../README.md)
