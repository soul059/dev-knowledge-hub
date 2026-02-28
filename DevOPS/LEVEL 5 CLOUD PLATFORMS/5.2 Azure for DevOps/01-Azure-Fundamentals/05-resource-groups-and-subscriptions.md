# Chapter 5: Resource Groups and Subscriptions

## Overview
Azure organizes resources into a **management hierarchy**: Management Groups → Subscriptions → Resource Groups → Resources. Understanding this hierarchy is essential for organizing, securing, and costing Azure deployments in a DevOps context.

---

## 5.1 Azure Management Hierarchy

```
┌──────────────────────────────────────────────────────────────────┐
│                AZURE MANAGEMENT HIERARCHY                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────┐               │
│   │           ROOT MANAGEMENT GROUP             │               │
│   │          (Tenant Root Group)                 │               │
│   └──────────────────┬──────────────────────────┘               │
│                      │                                           │
│          ┌───────────┴───────────┐                               │
│          │                       │                               │
│   ┌──────┴──────┐        ┌──────┴──────┐                        │
│   │ Management  │        │ Management  │                        │
│   │  Group:     │        │  Group:     │                        │
│   │ "Production"│        │   "Dev"     │                        │
│   └──────┬──────┘        └──────┬──────┘                        │
│          │                       │                               │
│   ┌──────┴──────┐        ┌──────┴──────┐                        │
│   │Subscription │        │Subscription │                        │
│   │ "Prod Sub"  │        │ "Dev Sub"   │                        │
│   └──────┬──────┘        └──────┬──────┘                        │
│          │                       │                               │
│     ┌────┴────┐            ┌────┴────┐                          │
│     │         │            │         │                          │
│  ┌──┴───┐ ┌──┴───┐    ┌──┴───┐ ┌──┴───┐                       │
│  │RG:   │ │RG:   │    │RG:   │ │RG:   │                       │
│  │web-rg│ │db-rg │    │dev-rg│ │test  │                       │
│  └──┬───┘ └──┬───┘    └──┬───┘ └──┬───┘                       │
│     │        │            │        │                            │
│  ┌──┴──┐ ┌──┴──┐    ┌──┴──┐  ┌──┴──┐                         │
│  │ VM  │ │ SQL │    │ VM  │  │ App │                          │
│  │ App │ │ DB  │    │ Stor│  │ Svc │                          │
│  └─────┘ └─────┘    └─────┘  └─────┘                         │
│                                                                  │
│  Level 1: Management Groups (governance, policies)              │
│  Level 2: Subscriptions (billing, limits)                       │
│  Level 3: Resource Groups (lifecycle, permissions)              │
│  Level 4: Resources (actual services)                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5.2 Subscriptions

A **subscription** is a logical container for Azure resources that's associated with an Azure account. It acts as a **billing boundary** and an **access control boundary**.

### Subscription Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Free** | $200 credit for 30 days; limited free services | Learning and evaluation |
| **Pay-As-You-Go** | Charged based on usage | Small teams, startups |
| **Enterprise Agreement** | Volume licensing with committed spend | Large organizations |
| **CSP** | Partner-managed subscription | Managed service providers |
| **MSDN/Visual Studio** | Monthly Azure credits for subscribers | Development and testing |

### Subscription Limits (Key Defaults)

| Resource | Default Limit | Can Increase? |
|----------|--------------|---------------|
| VMs per region | 25,000 | Yes |
| Storage accounts per region | 250 | Yes |
| Resource groups per subscription | 980 | No |
| VNets per region | 1,000 | Yes |
| Public IPs per region | 1,000 | Yes |
| NSGs per region | 5,000 | Yes |

### Managing Subscriptions

```bash
# List all subscriptions
az account list --output table

# Set active subscription
az account set --subscription "My Subscription"

# Show current subscription
az account show --output table

# Get subscription ID
az account show --query id --output tsv
```

```powershell
# PowerShell equivalents
Get-AzSubscription | Format-Table
Set-AzContext -SubscriptionName "My Subscription"
Get-AzContext
```

---

## 5.3 Resource Groups

A **resource group** is a container that holds related resources for an Azure solution. Resources in a group share the same **lifecycle** — they're deployed, updated, and deleted together.

```
┌────────────────── RESOURCE GROUP CONCEPTS ──────────────────┐
│                                                              │
│   ┌────────────────────────────────────┐                     │
│   │     Resource Group: "webapp-rg"    │                     │
│   │     Location: East US              │                     │
│   │     Tags: env=prod, team=web       │                     │
│   │                                    │                     │
│   │   ┌─────────┐  ┌─────────┐        │                     │
│   │   │App Svc  │  │ SQL DB  │        │                     │
│   │   │(East US)│  │(East US)│        │                     │
│   │   └─────────┘  └─────────┘        │                     │
│   │                                    │                     │
│   │   ┌─────────┐  ┌─────────┐        │                     │
│   │   │ Storage │  │Key Vault│        │                     │
│   │   │(West EU)│  │(East US)│        │                     │
│   │   └─────────┘  └─────────┘        │  Resources CAN be   │
│   │                                    │  in different        │
│   └────────────────────────────────────┘  regions than the RG │
│                                                              │
│   KEY RULES:                                                 │
│   • A resource belongs to exactly ONE resource group         │
│   • RG location = where metadata is stored                   │
│   • Resources in RG can be in ANY region                     │
│   • Resources can be moved between RGs                      │
│   • Deleting RG deletes ALL resources inside                │
│   • RBAC can be applied at RG level                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Resource Group Best Practices

| Practice | Explanation |
|----------|-------------|
| **Group by lifecycle** | Resources deployed/deleted together should share a RG |
| **Use consistent naming** | e.g., `{app}-{env}-{region}-rg` → `myapp-prod-eastus-rg` |
| **Apply tags** | Environment, cost center, team, project |
| **Lock critical RGs** | Use resource locks to prevent accidental deletion |
| **One RG per environment** | Separate dev, staging, and production |

### Managing Resource Groups

```bash
# Create resource group
az group create --name webapp-prod-rg --location eastus --tags env=prod team=web

# List all resource groups
az group list --output table

# List resources in a group
az resource list --resource-group webapp-prod-rg --output table

# Delete resource group (and ALL resources inside)
az group delete --name webapp-prod-rg --yes --no-wait

# Move resource to another group
az resource move \
  --destination-group new-rg \
  --ids "/subscriptions/{sub}/resourceGroups/old-rg/providers/Microsoft.Compute/virtualMachines/myVM"

# Export template from existing resource group
az group export --name webapp-prod-rg > template.json
```

---

## 5.4 Resource Tags

Tags are **name-value pairs** applied to Azure resources for organization, cost management, and automation.

```
┌──────────────── RESOURCE TAGGING STRATEGY ─────────────────┐
│                                                             │
│  Tag Category     Examples                                  │
│  ────────────     ────────                                  │
│  Environment      env: prod | staging | dev | test          │
│  Cost Center      costCenter: CC-1234                       │
│  Team/Owner       team: platform | owner: john@company.com │
│  Project          project: webapp-v2                        │
│  Compliance       dataClass: confidential                   │
│  Automation       autoShutdown: true | startTime: 08:00     │
│                                                             │
│  Tags can be applied at:                                    │
│  • Management Group level                                   │
│  • Subscription level                                       │
│  • Resource Group level                                     │
│  • Individual Resource level                                │
│                                                             │
│  ⚠️ Tags are NOT inherited by default.                     │
│  Use Azure Policy to enforce tag inheritance.               │
│                                                             │
│  Limits: 50 tags per resource, 512 chars name, 256 value   │
└─────────────────────────────────────────────────────────────┘
```

### Tag Management

```bash
# Add tags to a resource group
az group update --name myRG --tags env=prod team=devops project=webapp

# Add tags to a resource
az resource tag --tags env=prod \
  --ids "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM"

# List resources by tag
az resource list --tag env=prod --output table

# Query cost by tag (using cost management)
az costmanagement query \
  --type Usage \
  --scope "/subscriptions/{sub}" \
  --timeframe MonthToDate \
  --dataset-filter "{\"tags\":{\"name\":\"env\",\"values\":[\"prod\"]}}"
```

---

## 5.5 Resource Locks

Resource locks prevent accidental deletion or modification of critical resources.

| Lock Type | Can Read? | Can Modify? | Can Delete? |
|-----------|-----------|-------------|-------------|
| **CanNotDelete** | ✅ | ✅ | ❌ |
| **ReadOnly** | ✅ | ❌ | ❌ |

```bash
# Create a delete lock on a resource group
az lock create \
  --name "prevent-delete" \
  --resource-group myRG \
  --lock-type CanNotDelete \
  --notes "Critical production resources"

# Create a read-only lock
az lock create \
  --name "read-only-lock" \
  --resource-group myRG \
  --lock-type ReadOnly

# List locks
az lock list --resource-group myRG --output table

# Delete a lock (required before deleting the RG)
az lock delete --name "prevent-delete" --resource-group myRG
```

⚠️ **Warning:** ReadOnly locks can have unexpected effects. For example, a ReadOnly lock on a storage account prevents listing keys (since that's a POST operation).

---

## 5.6 Management Groups

Management groups provide a level of scope **above subscriptions** for organizing governance.

```
┌────────────── MANAGEMENT GROUP HIERARCHY ──────────────┐
│                                                         │
│   ┌──────────────────────────┐                          │
│   │     Tenant Root Group    │  ← Auto-created          │
│   └────────────┬─────────────┘                          │
│                │                                        │
│     ┌──────────┴──────────┐                             │
│     │                     │                             │
│  ┌──┴──────────┐   ┌─────┴────────┐                    │
│  │  IT Dept    │   │  Marketing   │                    │
│  │  Mgmt Group │   │  Mgmt Group  │                    │
│  └──┬──────────┘   └─────┬────────┘                    │
│     │                     │                             │
│  ┌──┴──────────┐   ┌─────┴────────┐                    │
│  │ Production  │   │ Marketing    │                    │
│  │ Sub         │   │ Sub          │                    │
│  └─────────────┘   └──────────────┘                    │
│  ┌─────────────┐                                       │
│  │ Development │                                       │
│  │ Sub         │                                       │
│  └─────────────┘                                       │
│                                                         │
│  POLICIES & RBAC applied at Mgmt Group level           │
│  are INHERITED by all child subscriptions              │
│                                                         │
│  Limits:                                               │
│  • 6 levels deep (excluding root and sub levels)       │
│  • 10,000 management groups per tenant                 │
│  • A subscription can only belong to one mgmt group   │
└─────────────────────────────────────────────────────────┘
```

```bash
# Create management group
az account management-group create --name "IT-Production"

# Move subscription into management group
az account management-group subscription add \
  --name "IT-Production" \
  --subscription "prod-sub-id"

# List management groups
az account management-group list --output table
```

---

## 5.7 Naming Conventions

### Recommended Naming Pattern

```
{resource-type}-{workload}-{environment}-{region}-{instance}

Examples:
──────────
vm-webserver-prod-eastus-001       (Virtual Machine)
rg-webapp-dev-westeu               (Resource Group)
st-appdata-prod-eastus             (Storage Account)  
vnet-hub-prod-eastus               (Virtual Network)
kv-myapp-prod-eastus               (Key Vault)
sql-orders-prod-eastus             (SQL Server)
aks-microservices-prod-eastus      (AKS Cluster)
```

### Common Abbreviations

| Resource | Abbreviation |
|----------|-------------|
| Resource Group | `rg` |
| Virtual Machine | `vm` |
| Storage Account | `st` |
| Virtual Network | `vnet` |
| Subnet | `snet` |
| Network Security Group | `nsg` |
| Key Vault | `kv` |
| App Service | `app` |
| SQL Database | `sqldb` |
| AKS Cluster | `aks` |

---

## 5.8 Real-World DevOps Scenario

### Multi-Environment Organization

```bash
#!/bin/bash
# setup-environments.sh - Create structured environment

REGIONS=("eastus" "westeurope")
ENVS=("dev" "staging" "prod")
APP="myapp"

for env in "${ENVS[@]}"; do
  for region in "${REGIONS[@]}"; do
    RG_NAME="rg-${APP}-${env}-${region}"
    
    echo "Creating: $RG_NAME"
    az group create \
      --name "$RG_NAME" \
      --location "$region" \
      --tags env="$env" app="$APP" region="$region"
    
    # Apply delete lock on production
    if [ "$env" == "prod" ]; then
      az lock create \
        --name "no-delete" \
        --resource-group "$RG_NAME" \
        --lock-type CanNotDelete
    fi
  done
done
```

---

## 5.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot delete resource group | Resource lock exists | Remove the lock first: `az lock delete` |
| Hit subscription limit | Too many resources of a type | Request quota increase via portal or support |
| Tag not appearing on resources | Tags don't inherit by default | Use Azure Policy for tag inheritance |
| Cannot move resource | Some resources don't support cross-RG moves | Check move support matrix in Microsoft docs |
| Resource group in wrong region | RG region is metadata-only | Can't change RG region; resources can be anywhere |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Hierarchy** | Management Groups → Subscriptions → Resource Groups → Resources |
| **Subscriptions** | Billing boundary; access control boundary; have resource limits |
| **Resource Groups** | Lifecycle container; resources in one RG share permissions and tags |
| **Tags** | Name-value pairs for organization; 50 per resource; NOT inherited |
| **Locks** | CanNotDelete or ReadOnly; prevent accidental changes |
| **Management Groups** | Governance above subscriptions; RBAC and policies inherit downward |
| **Naming** | Use consistent conventions: `{type}-{workload}-{env}-{region}` |

---

## Quick Revision Questions

1. **Can a resource belong to more than one resource group?**
   > No, each resource belongs to exactly one resource group.

2. **What happens when you delete a resource group?**
   > All resources within the resource group are also deleted.

3. **What are the two types of resource locks?**
   > CanNotDelete (allows read and modify, prevents delete) and ReadOnly (allows read only).

4. **Are tags inherited from resource groups to resources by default?**
   > No, tags are NOT inherited by default. Use Azure Policy to enforce tag inheritance.

5. **What is the maximum number of resource groups per subscription?**
   > 980 resource groups per subscription.

6. **What is the purpose of Management Groups?**
   > Management Groups provide governance above subscriptions, allowing you to apply RBAC and Azure Policy across multiple subscriptions.

---

[⬅ Previous: Azure Resource Manager](04-azure-resource-manager.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Active Directory (Entra ID) ➡](../02-Identity-and-Access/01-azure-active-directory.md)
