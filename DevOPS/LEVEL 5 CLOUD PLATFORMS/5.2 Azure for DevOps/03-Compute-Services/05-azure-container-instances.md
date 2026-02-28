# Chapter 5: Azure Container Instances

## Overview
**Azure Container Instances (ACI)** is the fastest and simplest way to run a container in Azure without managing servers. It's ideal for quick tasks, build agents, batch jobs, and simple microservices that don't need full orchestration.

---

## 5.1 ACI Architecture

```
┌────────────── AZURE CONTAINER INSTANCES ──────────────┐
│                                                        │
│   ┌────────────────────────────────────────┐          │
│   │         CONTAINER GROUP                 │          │
│   │   (unit of deployment; like a pod)     │          │
│   │                                         │          │
│   │   ┌───────────┐    ┌───────────┐       │          │
│   │   │Container 1│    │Container 2│       │          │
│   │   │ (web app) │    │ (sidecar) │       │          │
│   │   └───────────┘    └───────────┘       │          │
│   │                                         │          │
│   │   Shared: IP, Ports, Storage, Network  │          │
│   └────────────────────────────────────────┘          │
│                                                        │
│   • Serverless containers (no VM management)           │
│   • Per-second billing                                 │
│   • Linux and Windows containers                       │
│   • Up to 4 vCPU, 16 GB RAM per container group       │
│   • Pull from ACR, Docker Hub, or any registry        │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 5.2 Creating Container Instances

```bash
# Simple container from Docker Hub
az container create \
  --resource-group myRG \
  --name mycontainer \
  --image nginx:latest \
  --ports 80 \
  --dns-name-label mycontainer-demo \
  --location eastus

# Container from Azure Container Registry
az container create \
  --resource-group myRG \
  --name myapp \
  --image myacr.azurecr.io/myapp:v1 \
  --registry-login-server myacr.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --ports 8080 \
  --cpu 2 \
  --memory 4 \
  --environment-variables DB_HOST=mydb.mysql.database.azure.com NODE_ENV=production

# Container group with YAML
az container create \
  --resource-group myRG \
  --file container-group.yaml

# Check status
az container show --resource-group myRG --name mycontainer --query instanceView.state

# View logs
az container logs --resource-group myRG --name mycontainer

# Attach to container
az container attach --resource-group myRG --name mycontainer

# Delete
az container delete --resource-group myRG --name mycontainer --yes
```

### Container Group YAML

```yaml
apiVersion: 2021-09-01
location: eastus
name: multi-container-group
properties:
  containers:
    - name: web
      properties:
        image: nginx:latest
        ports:
          - port: 80
        resources:
          requests:
            cpu: 1
            memoryInGb: 1.5
    - name: sidecar
      properties:
        image: myacr.azurecr.io/logger:v1
        resources:
          requests:
            cpu: 0.5
            memoryInGb: 0.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
      - port: 80
type: Microsoft.ContainerInstance/containerGroups
```

---

## 5.3 ACI vs Other Compute Services

| Feature | ACI | App Service | AKS | Functions |
|---------|-----|-------------|-----|-----------|
| **Complexity** | Low | Low | High | Low |
| **Orchestration** | None | Managed | Kubernetes | Managed |
| **Scaling** | Manual | Auto | Auto | Auto |
| **Best For** | Quick tasks, CI/CD agents | Web apps | Microservices at scale | Event-driven |
| **Min Cost** | Per-second | Plan-based | Cluster + nodes | Per-execution |

---

## 5.4 DevOps Use Case: CI/CD Build Agent

```bash
# Run a build container
az container create \
  --resource-group myRG \
  --name build-agent \
  --image myacr.azurecr.io/build-agent:latest \
  --cpu 4 --memory 8 \
  --restart-policy Never \
  --environment-variables \
    BUILD_REPO=https://github.com/org/repo.git \
    BUILD_BRANCH=main
```

---

## 5.5 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Container stuck in "Waiting" | Image pull failure | Verify image name and registry credentials |
| Container exits immediately | Application crashes on start | Check logs: `az container logs` |
| Cannot access container | Port not exposed | Add `--ports` and check DNS/IP |
| Out of memory | Insufficient memory allocation | Increase `--memory` parameter |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **ACI** | Serverless containers; no VM management; per-second billing |
| **Container Group** | Multiple containers sharing network/storage (like a pod) |
| **Limits** | Up to 4 vCPU, 16 GB RAM per group |
| **Use Cases** | Build agents, batch jobs, simple microservices, quick tasks |
| **Registries** | Pull from ACR, Docker Hub, or any registry |

---

## Quick Revision Questions

1. **What is a Container Group in ACI?**
   > A group of containers that share the same lifecycle, network (IP/ports), and storage — similar to a Kubernetes pod.

2. **When should you use ACI instead of AKS?**
   > For short-lived tasks, CI/CD agents, batch processing, or simple containers that don't need orchestration.

3. **What are the resource limits for ACI per container group?**
   > Up to 4 vCPUs and 16 GB of memory per container group.

4. **How is ACI billed?**
   > Per-second billing based on CPU and memory allocation while the container is running.

5. **What restart policies are available for ACI?**
   > Always (default), OnFailure, Never — controlling container restart behavior.

---

[⬅ Previous: Azure Functions](04-azure-functions.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Virtual Networks ➡](../04-Networking/01-virtual-networks.md)
