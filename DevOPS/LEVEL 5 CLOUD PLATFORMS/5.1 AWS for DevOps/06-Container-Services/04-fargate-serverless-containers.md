# Chapter 6.4: AWS Fargate — Serverless Containers

## Overview

AWS Fargate is a **serverless compute engine for containers** that removes the need to manage servers. You define your container's CPU/memory requirements, and Fargate handles the underlying infrastructure for both ECS and EKS.

---

## Fargate Architecture

```
┌──── Traditional (EC2) ──────────┐  ┌──── Fargate (Serverless) ──────┐
│                                  │  │                                 │
│  You manage:                     │  │  You manage:                   │
│  ┌────────────────────────────┐  │  │  ┌────────────────────────┐    │
│  │ EC2 Instances              │  │  │  │ Container Definitions  │    │
│  │ ├── OS patching            │  │  │  │ ├── Image              │    │
│  │ ├── Scaling instances      │  │  │  │ ├── CPU / Memory       │    │
│  │ ├── Capacity planning      │  │  │  │ ├── Networking         │    │
│  │ ├── Security agents        │  │  │  │ └── IAM roles          │    │
│  │ └── Docker daemon          │  │  │  └────────────────────────┘    │
│  │                            │  │  │                                 │
│  │ + Container Definitions    │  │  │  AWS manages:                  │
│  └────────────────────────────┘  │  │  ┌────────────────────────┐    │
│                                  │  │  │ ✓ Server provisioning  │    │
│  ⚠ Pay for EC2 even if idle    │  │  │ ✓ OS patching          │    │
│                                  │  │  │ ✓ Scaling              │    │
│                                  │  │  │ ✓ Security             │    │
│                                  │  │  └────────────────────────┘    │
│                                  │  │                                 │
│                                  │  │  ✓ Pay only when running      │
└──────────────────────────────────┘  └─────────────────────────────────┘
```

---

## Fargate with ECS

```bash
# Task definition for Fargate
# Set requiresCompatibilities to FARGATE
# Set networkMode to awsvpc (required)
# Specify CPU/memory at task level

# Create Fargate service
aws ecs create-service \
  --cluster prod-cluster \
  --service-name api-service \
  --task-definition api-task:1 \
  --desired-count 3 \
  --launch-type FARGATE \
  --platform-version LATEST \
  --network-configuration '{
    "awsvpcConfiguration": {
      "subnets": ["subnet-priv-1a", "subnet-priv-1b"],
      "securityGroups": ["sg-api"],
      "assignPublicIp": "DISABLED"
    }
  }'
```

### Valid Fargate CPU/Memory Combinations

| CPU (vCPU) | Memory (GB) Options |
|------------|-------------------|
| 0.25 | 0.5, 1, 2 |
| 0.5 | 1, 2, 3, 4 |
| 1 | 2, 3, 4, 5, 6, 7, 8 |
| 2 | 4 – 16 (1 GB increments) |
| 4 | 8 – 30 (1 GB increments) |
| 8 | 16 – 60 (4 GB increments) |
| 16 | 32 – 120 (8 GB increments) |

---

## Fargate with EKS

```yaml
# Fargate Profile defines which pods run on Fargate
# Pods matching the namespace + labels run on Fargate

# fargate-profile.yaml (created via eksctl or AWS CLI)
# eksctl create fargateprofile \
#   --cluster prod-cluster \
#   --name api-profile \
#   --namespace production \
#   --labels app=api
```

```bash
# Create Fargate profile for EKS
aws eks create-fargate-profile \
  --cluster-name prod-cluster \
  --fargate-profile-name api-profile \
  --pod-execution-role-arn arn:aws:iam::role/eksFargatePodRole \
  --subnets subnet-priv-1a subnet-priv-1b \
  --selectors namespace=production,labels={app=api}
```

---

## Fargate Pricing Model

```
┌──── Fargate Pricing (per second, 1 min minimum) ───────────┐
│                                                              │
│  vCPU: ~$0.04048 per vCPU per hour                         │
│  Memory: ~$0.004445 per GB per hour                        │
│  Storage: 20 GB ephemeral free, then $0.000111/GB/hour     │
│                                                              │
│  Example: 3 tasks × (1 vCPU + 2 GB) × 24 hours            │
│  = 3 × ($0.04048 + 2 × $0.004445) × 24                    │
│  = 3 × $0.04937 × 24                                       │
│  = $3.55/day ≈ $106/month                                  │
│                                                              │
│  vs. EC2 (3 × t3.small = 3 × $0.0208/hr × 24 × 30)       │
│  = $44.93/month                                             │
│                                                              │
│  ⚠ Fargate is more expensive per-unit but:                 │
│    • No idle capacity waste                                 │
│    • No ops overhead                                        │
│    • Scales to zero                                         │
│    • Best for variable/bursty workloads                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Task can't start | Invalid CPU/memory combination | Use valid Fargate CPU/memory pairs (see table above) |
| Can't pull image | No internet access in private subnet | Add NAT Gateway or ECR VPC endpoint |
| Task timeout | No route to ECR/CloudWatch | Add VPC endpoints for ECR, S3, and CloudWatch Logs |
| Fargate Spot interruption | Spot capacity reclaimed | Use mixed strategy; ensure graceful shutdown handling |
| Storage full | Exceeded 20 GB ephemeral | Use EFS for persistent storage; increase ephemeral storage |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Fargate** | Serverless containers. No instances. Pay per task. |
| **With ECS** | Set `--launch-type FARGATE`. Network mode must be `awsvpc`. |
| **With EKS** | Create Fargate Profile matching namespace/labels. |
| **Pricing** | Per-second billing (1 min minimum). vCPU + Memory + Storage. |
| **Networking** | Each task gets own ENI. Private subnets + NAT or VPC endpoints. |
| **Best for** | Variable workloads, small teams, reducing operational overhead. |

---

## Quick Revision Questions

1. **What is the required network mode for Fargate tasks?**
   > `awsvpc` — each task gets its own elastic network interface with a private IP.

2. **How does Fargate pricing compare to EC2?**
   > Fargate has higher per-unit cost but no idle waste and zero ops overhead. Best for variable workloads; EC2 is cheaper for steady high utilization.

3. **Can Fargate tasks access the internet?**
   > Only if they're in a private subnet with a NAT Gateway, or a public subnet with `assignPublicIp: ENABLED`.

4. **How does Fargate work with EKS?**
   > Create a Fargate Profile that matches pods by namespace and labels. Matching pods automatically run on Fargate instead of EC2 nodes.

5. **What is the maximum storage for a Fargate task?**
   > Up to 200 GB ephemeral storage (20 GB free), plus EFS for persistent shared storage.

---

[← Previous: 6.3 EKS](03-eks-elastic-kubernetes-service.md) | [Next: 6.5 App Runner →](05-app-runner.md)

[← Back to README](../README.md)
