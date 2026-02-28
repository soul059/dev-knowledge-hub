# Chapter 2: VM Scale Sets

## Overview
**Azure Virtual Machine Scale Sets (VMSS)** let you create and manage a group of identical, auto-scaling VMs. They're the foundation for building highly available, scalable applications and are used by services like AKS under the hood.

---

## 2.1 VMSS Architecture

```
┌──────────────── VM SCALE SET ARCHITECTURE ──────────────────┐
│                                                              │
│   ┌──────────────────────────────────────────────────┐      │
│   │              LOAD BALANCER                        │      │
│   └──────────────────────┬───────────────────────────┘      │
│                          │                                   │
│   ┌──────────────────────┴───────────────────────────┐      │
│   │              VM SCALE SET                         │      │
│   │                                                   │      │
│   │   Instance 0    Instance 1    Instance 2          │      │
│   │   ┌────────┐    ┌────────┐    ┌────────┐         │      │
│   │   │   VM   │    │   VM   │    │   VM   │   ...   │      │
│   │   │  same  │    │  same  │    │  same  │         │      │
│   │   │ config │    │ config │    │ config │         │      │
│   │   └────────┘    └────────┘    └────────┘         │      │
│   │                                                   │      │
│   │   Autoscale: Min=2, Max=10, Default=3            │      │
│   │   Scaling Metric: CPU > 75% → add instance       │      │
│   │                   CPU < 25% → remove instance    │      │
│   │                                                   │      │
│   └───────────────────────────────────────────────────┘      │
│                                                              │
│   KEY FEATURES:                                              │
│   • All VMs from same base image                             │
│   • Automatic scaling based on metrics/schedule              │
│   • Integrated with Load Balancer / App Gateway              │
│   • Up to 1,000 instances (custom image) or 600 (gallery)   │
│   • Supports Availability Zones (zone-redundant)             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Orchestration Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Uniform** | All VMs identical; VMSS manages lifecycle | Traditional auto-scaling workloads |
| **Flexible** | Mix VM sizes; more control over individual VMs | AKS, heterogeneous workloads |

---

## 2.3 Creating a VMSS

```bash
# Create a VMSS with autoscale
az vmss create \
  --resource-group myRG \
  --name myVMSS \
  --image Ubuntu2204 \
  --vm-sku Standard_B2s \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --upgrade-policy-mode automatic \
  --load-balancer myLB

# Configure autoscale
az monitor autoscale create \
  --resource-group myRG \
  --resource myVMSS \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name myAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 3

# Add scale-out rule (CPU > 75% for 5 min)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscale \
  --condition "Percentage CPU > 75 avg 5m" \
  --scale out 1

# Add scale-in rule (CPU < 25% for 5 min)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscale \
  --condition "Percentage CPU < 25 avg 5m" \
  --scale in 1

# Manual scale
az vmss scale --resource-group myRG --name myVMSS --new-capacity 5
```

---

## 2.4 Upgrade Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| **Automatic** | All instances updated immediately | Dev/test environments |
| **Rolling** | Updated in batches with configurable batch size | Production |
| **Manual** | Updates only when explicitly triggered | Critical workloads |

---

## 2.5 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| VMs not scaling | Autoscale rules not configured | Check: `az monitor autoscale show` |
| Instances unhealthy | Health probe failing | Verify load balancer health probe |
| Update stuck | Rolling update batch too large | Reduce `maxBatchInstancePercent` |
| Quota exceeded | Hit regional vCPU limit | Request quota increase |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **VMSS** | Group of identical, auto-scaling VMs |
| **Autoscale** | Metric-based or schedule-based scaling |
| **Orchestration** | Uniform (identical) vs Flexible (mixed) |
| **Upgrade Policies** | Automatic, Rolling, Manual |
| **Limits** | Up to 1,000 instances with custom images |

---

## Quick Revision Questions

1. **What is the difference between Uniform and Flexible orchestration modes?**
   > Uniform requires identical VMs managed entirely by VMSS. Flexible allows mixing VM sizes with more individual control.

2. **What are the three upgrade policies for VMSS?**
   > Automatic (all at once), Rolling (in batches), Manual (explicit trigger).

3. **How do you configure autoscale rules for a VMSS?**
   > Use `az monitor autoscale create` with min/max/default counts, then add rules with `az monitor autoscale rule create` specifying metric conditions.

4. **What is the maximum number of instances in a VMSS?**
   > Up to 1,000 instances with a custom image, or 600 with a gallery image.

5. **How is a VMSS different from manually creating multiple VMs?**
   > VMSS provides identical configuration, automatic scaling, rolling updates, and integrated load balancing — all managed as a single resource.

---

[⬅ Previous: Virtual Machines](01-virtual-machines.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure App Service ➡](03-azure-app-service.md)
