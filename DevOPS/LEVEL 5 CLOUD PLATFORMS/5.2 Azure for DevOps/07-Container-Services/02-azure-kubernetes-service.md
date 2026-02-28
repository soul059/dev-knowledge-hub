# Chapter 2: Azure Kubernetes Service (AKS)

## Overview
**Azure Kubernetes Service (AKS)** is a managed Kubernetes container orchestration service. Azure manages the control plane (API server, etcd, scheduler), while you manage the worker nodes that run your applications.

---

## 2.1 AKS Architecture

```
┌──────────────── AKS ARCHITECTURE ──────────────────┐
│                                                      │
│   ┌─────────────────────────────────────────────┐    │
│   │  CONTROL PLANE (Managed by Azure — FREE)    │    │
│   │  ┌──────────┐ ┌──────┐ ┌──────────────┐    │    │
│   │  │API Server│ │ etcd │ │  Scheduler   │    │    │
│   │  └──────────┘ └──────┘ └──────────────┘    │    │
│   │  ┌────────────────────┐                     │    │
│   │  │ Controller Manager │                     │    │
│   │  └────────────────────┘                     │    │
│   └──────────────────┬──────────────────────────┘    │
│                      │                                │
│   ┌──────────────────▼──────────────────────────┐    │
│   │  NODE POOL (Managed by you — BILLED)        │    │
│   │                                              │    │
│   │  ┌─────────── Node 1 ──────────────┐        │    │
│   │  │ ┌──────┐ ┌──────┐ ┌──────┐     │        │    │
│   │  │ │Pod-A │ │Pod-B │ │Pod-C │     │        │    │
│   │  │ │nginx │ │api   │ │worker│     │        │    │
│   │  │ └──────┘ └──────┘ └──────┘     │        │    │
│   │  │ kubelet  kube-proxy  runtime    │        │    │
│   │  └─────────────────────────────────┘        │    │
│   │                                              │    │
│   │  ┌─────────── Node 2 ──────────────┐        │    │
│   │  │ ┌──────┐ ┌──────┐               │        │    │
│   │  │ │Pod-D │ │Pod-E │               │        │    │
│   │  │ │redis │ │api   │               │        │    │
│   │  │ └──────┘ └──────┘               │        │    │
│   │  └─────────────────────────────────┘        │    │
│   └──────────────────────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 2.2 Key Concepts

| Concept | Description |
|---------|-------------|
| **Pod** | Smallest deployable unit; one or more containers |
| **Deployment** | Desired state for pods (replicas, image, rolling updates) |
| **Service** | Stable network endpoint for a set of pods |
| **Namespace** | Virtual cluster for resource isolation |
| **Node Pool** | Group of VMs with same configuration |
| **Ingress** | HTTP/HTTPS routing rules to services |

---

## 2.3 Creating an AKS Cluster

```bash
# Create AKS cluster
az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --location eastus \
  --node-count 3 \
  --node-vm-size Standard_DS2_v2 \
  --enable-managed-identity \
  --network-plugin azure \
  --generate-ssh-keys \
  --attach-acr mydevopsregistry

# Get credentials for kubectl
az aks get-credentials \
  --resource-group myRG \
  --name myAKSCluster

# Verify cluster
kubectl get nodes
kubectl cluster-info
```

---

## 2.4 Deploying Applications

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: mydevopsregistry.azurecr.io/myapp/frontend:v1.0
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "250m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

```bash
# Deploy
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Check status
kubectl get deployments
kubectl get pods
kubectl get services

# Scale
kubectl scale deployment web-app --replicas=5

# Rolling update
kubectl set image deployment/web-app web=mydevopsregistry.azurecr.io/myapp/frontend:v2.0

# Rollback
kubectl rollout undo deployment/web-app
```

---

## 2.5 Node Pools

```
┌────── NODE POOLS ──────────────────────────┐
│                                             │
│  SYSTEM NODE POOL (required)                │
│  ┌─────────────────────────────────┐        │
│  │ Runs system pods (CoreDNS, etc.)│        │
│  │ Standard_DS2_v2 × 3 nodes      │        │
│  └─────────────────────────────────┘        │
│                                             │
│  USER NODE POOL (optional)                  │
│  ┌─────────────────────────────────┐        │
│  │ Runs application workloads      │        │
│  │ Standard_DS4_v2 × 5 nodes      │        │
│  └─────────────────────────────────┘        │
│                                             │
│  GPU NODE POOL (optional)                   │
│  ┌─────────────────────────────────┐        │
│  │ ML/AI workloads                 │        │
│  │ Standard_NC6 × 2 nodes          │        │
│  └─────────────────────────────────┘        │
│                                             │
└─────────────────────────────────────────────┘
```

```bash
# Add a node pool
az aks nodepool add \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name gpupool \
  --node-count 2 \
  --node-vm-size Standard_NC6 \
  --labels workload=gpu

# Scale a node pool
az aks nodepool scale \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name gpupool \
  --node-count 4

# Enable cluster autoscaler
az aks nodepool update \
  --resource-group myRG \
  --cluster-name myAKSCluster \
  --name nodepool1 \
  --enable-cluster-autoscaler \
  --min-count 2 \
  --max-count 10
```

---

## 2.6 Networking Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **kubenet** | Pods get IPs from a separate CIDR, NAT to VNet | Simple, fewer IPs needed |
| **Azure CNI** | Pods get IPs directly from VNet subnet | VNet integration, network policies |
| **Azure CNI Overlay** | Pod IPs from overlay network, node IPs from VNet | Large clusters, IP conservation |

---

## 2.7 AKS + DevOps Integration

```
┌────── CI/CD WITH AKS ──────────────────────┐
│                                              │
│  1. Developer pushes code to repo            │
│  2. Pipeline triggers:                       │
│     a. Build container image                 │
│     b. Push to ACR                           │
│     c. Update K8s manifests                  │
│     d. Deploy to AKS                         │
│                                              │
│  ┌──────┐    ┌─────┐    ┌─────┐    ┌─────┐ │
│  │ Git  │───►│Build│───►│ ACR │───►│ AKS │ │
│  │ Push │    │Image│    │Push │    │Deploy│ │
│  └──────┘    └─────┘    └─────┘    └─────┘ │
│                                              │
│  Pipeline step:                              │
│  - task: KubernetesManifest@0                │
│    inputs:                                   │
│      action: deploy                          │
│      kubernetesServiceConnection: aks-conn   │
│      manifests: k8s/*.yaml                   │
│      containers: $(ACR)/myapp:$(tag)         │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 2.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Pod CrashLoopBackOff | App error or missing config | Check `kubectl logs <pod>` |
| ImagePullBackOff | ACR auth or wrong image name | Verify ACR attach and image path |
| Pending pods | Insufficient resources | Scale node pool or check resource requests |
| Service no external IP | LoadBalancer stuck pending | Check NSG rules on AKS subnet |
| Node NotReady | Node health issue | Check `kubectl describe node <name>` |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Control Plane** | Managed by Azure (free), includes API server and etcd |
| **Node Pools** | System (required) + User pools; can mix VM sizes |
| **Autoscaler** | Cluster autoscaler adjusts node count based on demand |
| **Networking** | kubenet, Azure CNI, Azure CNI Overlay |
| **ACR Integration** | `--attach-acr` for seamless image pulling |
| **Deployments** | Declarative desired state with rolling updates & rollbacks |

---

## Quick Revision Questions

1. **What does Azure manage in AKS?**
   > The control plane: API server, etcd, scheduler, and controller manager — provided free of charge.

2. **What is a node pool?**
   > A group of VMs with the same configuration. System pools run K8s components; user pools run workloads.

3. **How does AKS autoscaling work?**
   > The cluster autoscaler automatically adjusts the number of nodes based on pod resource requests when pods are pending.

4. **What is the difference between kubenet and Azure CNI?**
   > kubenet uses NAT with a separate pod CIDR; Azure CNI assigns pods IPs directly from the VNet subnet.

5. **How do you roll back a deployment?**
   > `kubectl rollout undo deployment/<name>` reverts to the previous revision.

---

[⬅ Previous: Container Registry](01-azure-container-registry.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Container Apps ➡](03-azure-container-apps.md)
