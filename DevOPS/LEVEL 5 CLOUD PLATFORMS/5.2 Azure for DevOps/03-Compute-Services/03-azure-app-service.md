# Chapter 3: Azure App Service

## Overview
**Azure App Service** is a fully managed PaaS for building, deploying, and scaling web applications, REST APIs, and mobile backends. For DevOps, it's the go-to service for deploying applications without managing infrastructure.

---

## 3.1 App Service Architecture

```
┌──────────────── APP SERVICE ARCHITECTURE ──────────────────┐
│                                                             │
│   ┌─────────────────────────────────────────────┐          │
│   │            APP SERVICE PLAN                  │          │
│   │        (Underlying compute)                  │          │
│   │                                              │          │
│   │   ┌──────────┐  ┌──────────┐  ┌──────────┐ │          │
│   │   │  Web App │  │  Web App │  │  API App │ │          │
│   │   │    #1    │  │    #2    │  │    #3    │ │          │
│   │   └──────────┘  └──────────┘  └──────────┘ │          │
│   │                                              │          │
│   │   SKU: B1, S1, P1v3, etc.                   │          │
│   │   OS: Linux or Windows                       │          │
│   │   Scale: Manual or Auto                      │          │
│   └─────────────────────────────────────────────┘          │
│                                                             │
│   Multiple apps share the SAME App Service Plan             │
│   (same compute resources)                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### App Service Plan Tiers

| Tier | SKU | Features | Use Case |
|------|-----|----------|----------|
| **Free** | F1 | 1 GB, no custom domain, shared compute | Testing |
| **Shared** | D1 | Custom domains, shared compute | Dev |
| **Basic** | B1-B3 | Dedicated compute, manual scale | Dev/test |
| **Standard** | S1-S3 | Auto-scale, staging slots, daily backups | Production |
| **Premium** | P1v3-P3v3 | More powerful, VNet integration | High-traffic |
| **Isolated** | I1v2-I3v2 | Dedicated ASE, full network isolation | Enterprise |

---

## 3.2 Creating and Deploying

```bash
# Create App Service Plan
az appservice plan create \
  --name myAppPlan \
  --resource-group myRG \
  --sku S1 \
  --is-linux

# Create Web App
az webapp create \
  --name mywebapp-unique \
  --resource-group myRG \
  --plan myAppPlan \
  --runtime "NODE:18-lts"

# Deploy from ZIP
az webapp deploy \
  --resource-group myRG \
  --name mywebapp-unique \
  --src-path ./app.zip

# Deploy from GitHub
az webapp deployment source config \
  --resource-group myRG \
  --name mywebapp-unique \
  --repo-url https://github.com/user/repo \
  --branch main \
  --manual-integration

# Deploy from container image
az webapp create \
  --resource-group myRG \
  --plan myAppPlan \
  --name mycontainerapp \
  --deployment-container-image-name myacr.azurecr.io/myapp:latest

# Set environment variables (app settings)
az webapp config appsettings set \
  --resource-group myRG \
  --name mywebapp-unique \
  --settings DB_HOST=mydb.mysql.database.azure.com NODE_ENV=production
```

---

## 3.3 Deployment Slots

Deployment slots let you deploy to **staging** and **swap** to production with zero downtime.

```
┌──────────── DEPLOYMENT SLOTS ──────────────┐
│                                             │
│   PRODUCTION SLOT         STAGING SLOT      │
│   ┌─────────────┐        ┌─────────────┐   │
│   │   v1.0      │        │   v1.1      │   │
│   │   (live)    │        │  (testing)  │   │
│   └──────┬──────┘        └──────┬──────┘   │
│          │                      │           │
│          └──────── SWAP ────────┘           │
│                                             │
│   After swap:                               │
│   ┌─────────────┐        ┌─────────────┐   │
│   │   v1.1      │        │   v1.0      │   │
│   │   (live)    │        │  (rollback) │   │
│   └─────────────┘        └─────────────┘   │
│                                             │
│   ✅ Zero downtime                          │
│   ✅ Instant rollback (swap back)           │
│   ✅ Warm up before swap                    │
│                                             │
└─────────────────────────────────────────────┘
```

```bash
# Create staging slot
az webapp deployment slot create \
  --resource-group myRG \
  --name mywebapp \
  --slot staging

# Deploy to staging
az webapp deploy \
  --resource-group myRG \
  --name mywebapp \
  --slot staging \
  --src-path ./app.zip

# Swap staging to production
az webapp deployment slot swap \
  --resource-group myRG \
  --name mywebapp \
  --slot staging \
  --target-slot production
```

---

## 3.4 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| App returns 503 | App Service Plan at capacity | Scale up (bigger SKU) or scale out (more instances) |
| Deployment fails | Wrong runtime version | Check: `az webapp list-runtimes` |
| App settings not applied | Slot-specific settings needed | Mark settings as "slot setting" |
| Slow cold starts | Free/Shared tier limitations | Use Always On (Standard+ tier) |
| CORS errors | Missing CORS configuration | `az webapp cors add --allowed-origins "https://example.com"` |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **App Service** | Fully managed PaaS for web apps and APIs |
| **App Service Plan** | Defines compute resources; multiple apps per plan |
| **Deployment Slots** | Zero-downtime deployments via swap (Standard+ tier) |
| **Deployment Methods** | ZIP, Git, Container, FTP, GitHub Actions |
| **Scaling** | Manual or auto-scale (Standard+ tier) |
| **App Settings** | Environment variables; can be slot-specific |

---

## Quick Revision Questions

1. **What is the relationship between an App Service Plan and Web Apps?**
   > An App Service Plan defines the compute resources (CPU, RAM, features). Multiple web apps can run on the same plan sharing those resources.

2. **What tier is required for deployment slots?**
   > Standard (S1) or higher.

3. **How do deployment slots enable zero-downtime deployments?**
   > You deploy to a staging slot, test it, then swap with production. The swap is a DNS-level change that's effectively instant.

4. **How do you set environment variables for a Web App?**
   > `az webapp config appsettings set --settings KEY=VALUE`. These are exposed as environment variables to the app.

5. **What is the difference between scaling up and scaling out?**
   > Scaling up changes the VM size (more CPU/RAM). Scaling out adds more instances of the same size.

---

[⬅ Previous: VM Scale Sets](02-vm-scale-sets.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Functions ➡](04-azure-functions.md)
