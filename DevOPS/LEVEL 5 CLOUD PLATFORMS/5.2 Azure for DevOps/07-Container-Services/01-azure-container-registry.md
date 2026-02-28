# Chapter 1: Azure Container Registry (ACR)

## Overview
**Azure Container Registry (ACR)** is a managed Docker container registry for storing and managing container images, Helm charts, and OCI artifacts. It integrates natively with AKS, App Service, Azure DevOps, and other Azure services.

---

## 1.1 ACR Architecture

```
┌──────────── ACR ARCHITECTURE ──────────────────┐
│                                                  │
│   Developer / CI Pipeline                        │
│       │                                          │
│       │ docker push                              │
│       ▼                                          │
│   ┌──────────────────────────────────────┐       │
│   │  Azure Container Registry            │       │
│   │  myregistry.azurecr.io               │       │
│   │                                      │       │
│   │  ┌──────────────────────────────┐    │       │
│   │  │ Repository: myapp/frontend   │    │       │
│   │  │ Tags: latest, v1.0, v1.1     │    │       │
│   │  └──────────────────────────────┘    │       │
│   │  ┌──────────────────────────────┐    │       │
│   │  │ Repository: myapp/backend    │    │       │
│   │  │ Tags: latest, v2.0           │    │       │
│   │  └──────────────────────────────┘    │       │
│   │  ┌──────────────────────────────┐    │       │
│   │  │ Repository: myapp/worker     │    │       │
│   │  │ Tags: latest, v1.3           │    │       │
│   │  └──────────────────────────────┘    │       │
│   └─────────────┬────────────────────────┘       │
│                 │ docker pull                     │
│       ┌─────────┼──────────┐                     │
│       ▼         ▼          ▼                     │
│   ┌──────┐  ┌──────┐  ┌──────┐                  │
│   │ AKS  │  │ ACI  │  │ App  │                  │
│   │      │  │      │  │Service│                  │
│   └──────┘  └──────┘  └──────┘                  │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 1.2 ACR SKUs

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| **Storage** | 10 GiB | 100 GiB | 500 GiB |
| **Webhooks** | 2 | 10 | 500 |
| **Geo-replication** | ❌ | ❌ | ✅ |
| **Private link** | ❌ | ❌ | ✅ |
| **Content trust** | ❌ | ❌ | ✅ |
| **Customer-managed keys** | ❌ | ❌ | ✅ |
| **ACR Tasks** | ✅ | ✅ | ✅ |
| **Throughput** | Lower | Medium | Highest |

---

## 1.3 Creating and Using ACR

```bash
# Create ACR
az acr create \
  --resource-group myRG \
  --name mydevopsregistry \
  --sku Standard \
  --location eastus

# Login to ACR
az acr login --name mydevopsregistry

# Tag and push an image
docker tag myapp:v1.0 mydevopsregistry.azurecr.io/myapp/frontend:v1.0
docker push mydevopsregistry.azurecr.io/myapp/frontend:v1.0

# Pull an image
docker pull mydevopsregistry.azurecr.io/myapp/frontend:v1.0

# List repositories
az acr repository list --name mydevopsregistry --output table

# List tags for a repository
az acr repository show-tags \
  --name mydevopsregistry \
  --repository myapp/frontend \
  --output table

# Delete a specific tag
az acr repository delete \
  --name mydevopsregistry \
  --image myapp/frontend:v1.0 \
  --yes
```

---

## 1.4 ACR Tasks (Build in Cloud)

```bash
# Quick build (no local Docker needed!)
az acr build \
  --registry mydevopsregistry \
  --image myapp/frontend:v1.0 \
  --file Dockerfile \
  .

# Multi-step task from YAML
az acr task create \
  --registry mydevopsregistry \
  --name build-and-push \
  --file acr-task.yaml \
  --context https://github.com/myorg/myapp.git \
  --git-access-token $GIT_TOKEN

# Trigger task on base image update
az acr task create \
  --registry mydevopsregistry \
  --name auto-rebuild \
  --image myapp/frontend:{{.Run.ID}} \
  --file Dockerfile \
  --context https://github.com/myorg/myapp.git \
  --git-access-token $GIT_TOKEN \
  --base-image-trigger-enabled true
```

---

## 1.5 Authentication Methods

| Method | Use Case |
|--------|----------|
| **Azure AD (az acr login)** | Developer interactive login |
| **Service Principal** | CI/CD pipelines |
| **Managed Identity** | AKS, App Service, ACI |
| **Admin Account** | Quick testing only (NOT for production) |
| **Repository-scoped Token** | Fine-grained access to specific repos |

```bash
# Attach ACR to AKS (uses managed identity)
az aks update \
  --resource-group myRG \
  --name myAKSCluster \
  --attach-acr mydevopsregistry

# Create service principal for CI/CD
ACR_ID=$(az acr show --name mydevopsregistry --query id --output tsv)
az ad sp create-for-rbac \
  --name acr-push-sp \
  --role AcrPush \
  --scopes $ACR_ID
```

---

## 1.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Unauthorized (401) | Not logged in or wrong credentials | Run `az acr login` or check SP credentials |
| Image not found | Wrong registry/repo/tag name | Verify with `az acr repository list` |
| Push denied | Missing AcrPush role | Assign `AcrPush` RBAC role |
| Slow pulls | Registry far from compute | Use geo-replication (Premium) |
| Storage full | Too many images/tags | Set up purge/retention policies |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **ACR** | Managed Docker registry with Azure AD integration |
| **SKUs** | Basic, Standard, Premium (geo-replication, private link) |
| **ACR Tasks** | Build images in the cloud without local Docker |
| **Auth** | Azure AD, Service Principal, Managed Identity, Admin |
| **AKS Integration** | `az aks update --attach-acr` for seamless pulling |
| **Naming** | `<registry>.azurecr.io/<repo>:<tag>` |

---

## Quick Revision Questions

1. **What is ACR?**
   > Azure Container Registry — a managed Docker/OCI registry for storing and distributing container images.

2. **Which SKU supports geo-replication?**
   > Premium SKU — replicates registry to multiple Azure regions for lower latency pulls.

3. **What are ACR Tasks?**
   > Cloud-based image builds — no local Docker engine needed. Can auto-rebuild on source or base image changes.

4. **How does AKS authenticate to ACR?**
   > Using managed identity via `az aks update --attach-acr`, which grants the AKS kubelet identity the AcrPull role.

5. **Should you use the admin account in production?**
   > No. Admin accounts have full access and can't be scoped. Use service principals or managed identities.

---

[⬅ Previous: Azure Redis Cache](../06-Database-Services/04-azure-redis-cache.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Kubernetes Service ➡](02-azure-kubernetes-service.md)
