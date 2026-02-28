# Chapter 3: Azure Container Apps

## Overview
**Azure Container Apps** is a serverless container platform that enables running microservices and containerized applications without managing Kubernetes infrastructure. It's built on Kubernetes and Dapr but abstracts away the complexity.

---

## 3.1 Container Apps Architecture

```
┌──────── CONTAINER APPS ARCHITECTURE ────────────┐
│                                                   │
│   ┌───────────────────────────────────────────┐   │
│   │  Container Apps Environment                │   │
│   │  (Shared boundary — like a K8s namespace)  │   │
│   │                                            │   │
│   │  ┌──────────────────────────────────────┐  │   │
│   │  │  Container App: frontend             │  │   │
│   │  │  ┌────────┐ ┌────────┐ ┌────────┐   │  │   │
│   │  │  │Rev. 1  │ │Rev. 2  │ │Rev. 3  │   │  │   │
│   │  │  │ 0%     │ │ 20%    │ │ 80%    │   │  │   │
│   │  │  └────────┘ └────────┘ └────────┘   │  │   │
│   │  │  Ingress: external (HTTPS)           │  │   │
│   │  └──────────────────────────────────────┘  │   │
│   │                                            │   │
│   │  ┌──────────────────────────────────────┐  │   │
│   │  │  Container App: api                  │  │   │
│   │  │  Replicas: 1-10 (auto-scale)         │  │   │
│   │  │  Ingress: internal (VNet only)        │  │   │
│   │  └──────────────────────────────────────┘  │   │
│   │                                            │   │
│   │  ┌──────────────────────────────────────┐  │   │
│   │  │  Container App: worker               │  │   │
│   │  │  Scale: queue-based (KEDA)            │  │   │
│   │  │  Ingress: none (background worker)    │  │   │
│   │  └──────────────────────────────────────┘  │   │
│   │                                            │   │
│   └───────────────────────────────────────────┘   │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 3.2 Container Apps vs AKS vs ACI

| Feature | Container Apps | AKS | ACI |
|---------|---------------|-----|-----|
| **Management** | Serverless | You manage cluster | Serverless |
| **Scaling** | Auto (KEDA-based) | Manual + autoscaler | Manual |
| **Networking** | Built-in ingress | You configure | Basic VNet |
| **Revisions** | ✅ Traffic splitting | Via K8s tools | ❌ |
| **Dapr** | ✅ Built-in | You install | ❌ |
| **Best For** | Microservices, APIs | Complex K8s workloads | Simple containers |
| **K8s Knowledge** | Not required | Required | Not required |

---

## 3.3 Key Features

| Feature | Description |
|---------|-------------|
| **Revisions** | Immutable snapshots; traffic splitting between revisions |
| **Scale Rules** | HTTP concurrent requests, CPU, memory, or custom (KEDA) |
| **Dapr Integration** | Service discovery, state, pub/sub, bindings |
| **Ingress** | External (internet) or internal (VNet-only) HTTPS |
| **Secrets** | Injected as env vars or volume mounts |
| **Scale to Zero** | No charges when no traffic (consumption plan) |

---

## 3.4 Creating Container Apps

```bash
# Install extension
az extension add --name containerapp --upgrade

# Create Container Apps environment
az containerapp env create \
  --name myContainerAppEnv \
  --resource-group myRG \
  --location eastus

# Create a container app
az containerapp create \
  --name frontend \
  --resource-group myRG \
  --environment myContainerAppEnv \
  --image mydevopsregistry.azurecr.io/myapp/frontend:v1.0 \
  --target-port 80 \
  --ingress external \
  --min-replicas 1 \
  --max-replicas 10 \
  --cpu 0.5 \
  --memory 1.0Gi \
  --registry-server mydevopsregistry.azurecr.io \
  --registry-identity system

# Update with new image (creates new revision)
az containerapp update \
  --name frontend \
  --resource-group myRG \
  --image mydevopsregistry.azurecr.io/myapp/frontend:v2.0

# Traffic splitting between revisions
az containerapp ingress traffic set \
  --name frontend \
  --resource-group myRG \
  --revision-weight frontend--rev1=20 frontend--rev2=80

# Scale based on HTTP requests
az containerapp update \
  --name frontend \
  --resource-group myRG \
  --scale-rule-name http-rule \
  --scale-rule-type http \
  --scale-rule-http-concurrency 50

# View app URL
az containerapp show \
  --name frontend \
  --resource-group myRG \
  --query properties.configuration.ingress.fqdn \
  --output tsv
```

---

## 3.5 Revisions and Traffic Splitting

```
┌────── TRAFFIC SPLITTING ──────────────────┐
│                                            │
│   Incoming Traffic (100%)                  │
│       │                                    │
│       ▼                                    │
│   ┌──────────────┐                         │
│   │ Ingress       │                         │
│   │ ┌──────────┐ │                         │
│   │ │ 80% ────►│─┼─► Revision 3 (v2.0)    │
│   │ │          │ │                         │
│   │ │ 20% ────►│─┼─► Revision 2 (v1.5)    │
│   │ │          │ │    (canary testing)      │
│   │ └──────────┘ │                         │
│   └──────────────┘                         │
│                                            │
│   Use cases:                               │
│   • Blue/Green deployments                 │
│   • Canary releases                        │
│   • A/B testing                            │
│                                            │
└────────────────────────────────────────────┘
```

---

## 3.6 Scaling with KEDA

```
┌────── KEDA SCALE RULES ──────────────────┐
│                                           │
│  HTTP Scaling:                            │
│  Scale when concurrent requests > 50      │
│                                           │
│  Queue Scaling:                           │
│  Scale based on Azure Queue message count │
│                                           │
│  Custom Scaling:                          │
│  • Azure Service Bus queue length         │
│  • Kafka topic lag                        │
│  • Prometheus metrics                     │
│  • Cron schedule                          │
│                                           │
│  Scale to ZERO:                           │
│  • When no traffic/messages               │
│  • Pay nothing (consumption plan)         │
│                                           │
└───────────────────────────────────────────┘
```

---

## 3.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| App not accessible | Ingress not configured | Set ingress to external with target port |
| Image pull failed | ACR auth not set | Use `--registry-identity system` or service principal |
| Not scaling up | No scale rules defined | Add HTTP or custom scale rules |
| Revision not receiving traffic | Traffic weight is 0% | Update traffic splitting to route to new revision |
| High costs | Min replicas too high | Set `--min-replicas 0` for scale-to-zero |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Serverless Containers** | No K8s management; built on KEDA |
| **Revisions** | Immutable app versions; traffic splitting for canary/blue-green |
| **Scale** | HTTP, queue, CPU, custom metrics; scale to zero |
| **Dapr** | Built-in service mesh for microservices communication |
| **Ingress** | External (internet) or internal (VNet) HTTPS |
| **Cost** | Consumption plan: pay per vCPU-second and GiB-second |

---

## Quick Revision Questions

1. **What is Azure Container Apps?**
   > A serverless container platform for running microservices without managing Kubernetes infrastructure.

2. **What is a revision?**
   > An immutable snapshot of a container app version. Traffic can be split across revisions for canary/blue-green deployment.

3. **How is scaling handled?**
   > KEDA-based autoscaling using HTTP requests, queue messages, CPU, memory, or custom metrics. Can scale to zero.

4. **What is the difference between Container Apps and AKS?**
   > Container Apps is serverless (no cluster management); AKS gives full Kubernetes control. Use Container Apps for simpler microservices.

5. **What is Dapr in Container Apps?**
   > Distributed Application Runtime — provides service invocation, state management, pub/sub, and bindings as built-in sidecars.

---

[⬅ Previous: Azure Kubernetes Service](02-azure-kubernetes-service.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Docker and Container Fundamentals ➡](04-docker-fundamentals.md)
