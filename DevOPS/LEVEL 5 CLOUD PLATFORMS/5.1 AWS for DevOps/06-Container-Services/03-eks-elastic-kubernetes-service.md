# Chapter 6.3: Amazon EKS — Elastic Kubernetes Service

## Overview

Amazon EKS is a **managed Kubernetes service** that runs the Kubernetes control plane across multiple AZs. It's the choice for teams with Kubernetes expertise who need portability across clouds or access to the Kubernetes ecosystem.

---

## EKS Architecture

```
┌──────────── EKS Architecture ──────────────────────────────┐
│                                                             │
│  ┌── AWS-Managed Control Plane ──────────────────────────┐ │
│  │  ┌─── AZ-1a ───┐ ┌─── AZ-1b ───┐ ┌─── AZ-1c ───┐   │ │
│  │  │ API Server  │ │ API Server  │ │ API Server  │   │ │
│  │  │ etcd        │ │ etcd        │ │ etcd        │   │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │ │
│  │                                                      │ │
│  │  • AWS manages: upgrades, patching, scaling          │ │
│  │  • Endpoint: https://xxx.eks.amazonaws.com           │ │
│  │  • Cost: $0.10/hour (~$73/month)                    │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌── Your Worker Nodes (Data Plane) ─────────────────────┐ │
│  │                                                        │ │
│  │  Option 1: Managed Node Groups (EC2)                  │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐                │ │
│  │  │ Node    │ │ Node    │ │ Node    │   Auto-scaling  │ │
│  │  │ (EC2)   │ │ (EC2)   │ │ (EC2)   │   AWS manages  │ │
│  │  │ Pods ↓  │ │ Pods ↓  │ │ Pods ↓  │   AMI updates  │ │
│  │  └─────────┘ └─────────┘ └─────────┘                │ │
│  │                                                        │ │
│  │  Option 2: Fargate (Serverless)                       │ │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐               │ │
│  │  │ Pod  │ │ Pod  │ │ Pod  │ │ Pod  │  No nodes!    │ │
│  │  └──────┘ └──────┘ └──────┘ └──────┘               │ │
│  │                                                        │ │
│  │  Option 3: Self-Managed Nodes                         │ │
│  │  (You manage EC2 + Kubernetes agents — not recommended)│ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## ECS vs EKS

| Feature | ECS | EKS |
|---------|-----|-----|
| Orchestrator | AWS proprietary | Kubernetes (open-source) |
| Learning curve | Lower (AWS-native) | Higher (k8s knowledge needed) |
| Portability | AWS only | Multi-cloud (k8s standard) |
| Ecosystem | AWS integrations | Helm, Istio, Argo, etc. |
| Control plane cost | Free | $0.10/hr ($73/mo) |
| Configuration | JSON task definitions | YAML manifests |
| Best for | AWS-only environments | Multi-cloud, k8s teams |

---

## Setting Up EKS

```bash
# Create EKS cluster (using eksctl — recommended)
eksctl create cluster \
  --name prod-cluster \
  --version 1.29 \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.large \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10 \
  --managed \
  --with-oidc

# Or using AWS CLI
aws eks create-cluster \
  --name prod-cluster \
  --role-arn arn:aws:iam::role/eksClusterRole \
  --resources-vpc-config subnetIds=subnet-1a,subnet-1b,securityGroupIds=sg-eks

# Update kubeconfig
aws eks update-kubeconfig --name prod-cluster --region us-east-1

# Verify
kubectl get nodes
kubectl get pods --all-namespaces
```

---

## Kubernetes Deployment Example

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      serviceAccountName: web-app-sa
      containers:
      - name: app
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "256m"
            memory: "512Mi"
          limits:
            cpu: "512m"
            memory: "1Gi"
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 8080
```

```bash
# Deploy
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check status
kubectl get deployments
kubectl get pods
kubectl get svc web-app

# Rolling update
kubectl set image deployment/web-app app=123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.4

# Rollback
kubectl rollout undo deployment/web-app
```

---

## EKS IAM Integration (IRSA)

```
┌──── IAM Roles for Service Accounts (IRSA) ─────────────────┐
│                                                              │
│  Kubernetes Service Account ──► IAM Role                    │
│                                                              │
│  Pod (web-app-sa) ──► AssumeRoleWithWebIdentity             │
│                        ──► IAM Role (S3 access)             │
│                                                              │
│  • Fine-grained: per-pod IAM permissions                    │
│  • No access keys in code or env vars                       │
│  • Uses OIDC federation                                     │
│                                                              │
│  Setup:                                                     │
│  1. Enable OIDC provider on cluster                         │
│  2. Create IAM role with trust policy for OIDC              │
│  3. Create k8s ServiceAccount annotated with IAM role ARN   │
│  4. Reference ServiceAccount in pod spec                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `kubectl` can't connect | kubeconfig not updated | Run `aws eks update-kubeconfig --name cluster` |
| Pods stuck Pending | Insufficient node capacity | Check node resources; enable Cluster Autoscaler |
| Can't pull ECR image | Node IAM role missing ECR permissions | Attach `AmazonEC2ContainerRegistryReadOnly` to node role |
| Pod can't access AWS services | No IRSA configured | Set up IAM Roles for Service Accounts (IRSA) |
| Node groups won't update | PodDisruptionBudget blocking | Review PDB settings; allow disruptions during maintenance |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **EKS** | Managed Kubernetes. AWS runs control plane. You manage workers. |
| **Managed Node Groups** | AWS manages EC2 nodes, AMI updates, scaling. Recommended. |
| **Fargate on EKS** | Serverless pods. No nodes to manage. Per-pod billing. |
| **eksctl** | CLI tool to create/manage EKS clusters easily. |
| **IRSA** | IAM Roles for Service Accounts. Per-pod AWS permissions. |
| **ECS vs EKS** | ECS = simpler, AWS-native. EKS = portable, k8s ecosystem. |

---

## Quick Revision Questions

1. **What does AWS manage in EKS?**
   > The Kubernetes control plane: API servers, etcd, and their availability across multiple AZs.

2. **What are the three options for EKS worker nodes?**
   > Managed Node Groups (recommended), Fargate (serverless), and Self-Managed Nodes.

3. **What is IRSA?**
   > IAM Roles for Service Accounts — it maps a Kubernetes service account to an IAM role, giving pods fine-grained AWS permissions.

4. **How do you perform a rolling update in EKS?**
   > Update the container image in the Deployment manifest or use `kubectl set image`, and Kubernetes performs a rolling update automatically.

5. **When should you choose EKS over ECS?**
   > When you need Kubernetes portability across clouds, want access to the k8s ecosystem (Helm, Istio, Argo), or your team has existing Kubernetes expertise.

---

[← Previous: 6.2 ECS](02-ecs-elastic-container-service.md) | [Next: 6.4 Fargate →](04-fargate-serverless-containers.md)

[← Back to README](../README.md)
