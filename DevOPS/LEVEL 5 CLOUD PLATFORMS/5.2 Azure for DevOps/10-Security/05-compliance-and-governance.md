# Chapter 5: Compliance and Governance

## Overview
Azure provides a governance framework using **Azure Policy**, **Blueprints**, **Management Groups**, **Resource Locks**, and **Cost Management** to enforce organisational standards, ensure regulatory compliance, and maintain control over cloud resources at scale.

---

## 5.1 Governance Hierarchy

```
┌────────── AZURE GOVERNANCE HIERARCHY ────────────┐
│                                                    │
│  Root Management Group                             │
│  └── Management Group: "Company"                   │
│      ├── Management Group: "Production"            │
│      │   ├── Subscription: "Prod-UK"               │
│      │   │   ├── Resource Group: "webapp-prod"      │
│      │   │   │   ├── App Service                    │
│      │   │   │   ├── SQL Database                   │
│      │   │   │   └── Key Vault                      │
│      │   │   └── Resource Group: "data-prod"        │
│      │   └── Subscription: "Prod-EU"               │
│      │                                              │
│      ├── Management Group: "Development"            │
│      │   └── Subscription: "Dev-All"               │
│      │                                              │
│      └── Management Group: "Sandbox"                │
│          └── Subscription: "Sandbox"               │
│                                                    │
│  Policies applied at any level inherit DOWN         │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 5.2 Azure Policy

```
┌────── AZURE POLICY ───────────────────────┐
│                                            │
│  Policy Definition:                        │
│  ┌──────────────────────────────────────┐  │
│  │  WHAT to enforce                    │  │
│  │  (JSON rule: if condition → effect)  │  │
│  └──────────────────────────────────────┘  │
│       │                                    │
│       ▼                                    │
│  Policy Assignment:                        │
│  ┌──────────────────────────────────────┐  │
│  │  WHERE to enforce                   │  │
│  │  (Management Group, Subscription,   │  │
│  │   Resource Group)                   │  │
│  └──────────────────────────────────────┘  │
│       │                                    │
│       ▼                                    │
│  Policy Effects:                           │
│  ┌──────────────────────────────────────┐  │
│  │  Deny       → Block non-compliant   │  │
│  │  Audit      → Log but allow         │  │
│  │  AuditIfNotExists → Check related   │  │
│  │  DeployIfNotExists → Auto-remediate │  │
│  │  Modify     → Add/change tags       │  │
│  │  Disabled   → Policy off            │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

### Common Built-in Policies

| Policy | Effect | Purpose |
|--------|--------|---------|
| Allowed locations | Deny | Restrict resource deployment regions |
| Require tag on resources | Deny | Enforce tagging standards |
| Allowed VM SKUs | Deny | Prevent oversized VMs |
| Inherit tag from RG | Modify | Auto-tag resources from parent RG |
| Enable diagnostic settings | DeployIfNotExists | Auto-enable monitoring |
| HTTPS only for App Service | Deny | Enforce TLS |
| SQL TDE enabled | AuditIfNotExists | Verify encryption |

### Custom Policy Example

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "field": "Microsoft.Compute/virtualMachines/sku.name",
          "notIn": ["Standard_B2s", "Standard_D2s_v5", "Standard_D4s_v5"]
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

```bash
# Assign built-in policy: Allowed Locations
az policy assignment create \
  --name "allowed-locations" \
  --policy "e56962a6-4747-49cd-b67b-bf8b01975c4c" \
  --scope "/subscriptions/<sub-id>" \
  --params '{"listOfAllowedLocations": {"value": ["uksouth", "ukwest"]}}'

# Check compliance
az policy state summarize \
  --subscription "<sub-id>"

# List non-compliant resources
az policy state list \
  --filter "complianceState eq 'NonCompliant'" \
  --subscription "<sub-id>" \
  --query "[].{Resource:resourceId, Policy:policyAssignmentName}" -o table
```

---

## 5.3 Policy Initiatives (Policy Sets)

```
┌────── POLICY INITIATIVE ──────────────────┐
│                                            │
│  Initiative: "Security Baseline"           │
│  ┌──────────────────────────────────────┐  │
│  │  Policy 1: Require HTTPS            │  │
│  │  Policy 2: Enable TDE on SQL        │  │
│  │  Policy 3: Require encryption       │  │
│  │  Policy 4: Enable diagnostics       │  │
│  │  Policy 5: Restrict public IPs      │  │
│  │  Policy 6: Require MFA              │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Assign ONE initiative instead of          │
│  assigning 6 policies individually         │
│                                            │
│  Built-in initiatives:                     │
│  ├── Azure Security Benchmark             │
│  ├── CIS Microsoft Azure Foundations      │
│  ├── ISO 27001                             │
│  ├── PCI DSS                               │
│  └── NIST SP 800-53                        │
│                                            │
└────────────────────────────────────────────┘
```

---

## 5.4 Resource Locks

```
┌────── RESOURCE LOCKS ─────────────────────┐
│                                            │
│  CanNotDelete Lock:                        │
│  ┌──────────────────────────────────────┐  │
│  │  ✅ Read    ✅ Modify    ❌ Delete   │  │
│  │  Can't delete, but can change config │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  ReadOnly Lock:                            │
│  ┌──────────────────────────────────────┐  │
│  │  ✅ Read    ❌ Modify    ❌ Delete   │  │
│  │  Nothing can be changed or deleted   │  │
│  │  ⚠️ Can break things (e.g., can't   │  │
│  │     rotate storage keys)            │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Applied at: Subscription, Resource Group, │
│  or individual Resource level              │
│  Inherited by child resources              │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create a CanNotDelete lock
az lock create \
  --name "prevent-delete" \
  --lock-type CanNotDelete \
  --resource-group production-rg \
  --notes "Protect production resources"

# Create a ReadOnly lock on a resource
az lock create \
  --name "readonly-db" \
  --lock-type ReadOnly \
  --resource-group production-rg \
  --resource-name mydb \
  --resource-type Microsoft.Sql/servers

# List locks
az lock list --resource-group production-rg -o table

# Delete a lock (requires Owner/Lock Contributor role)
az lock delete --name "prevent-delete" --resource-group production-rg
```

---

## 5.5 Cost Management

```
┌────── COST MANAGEMENT ────────────────────┐
│                                            │
│  Cost Analysis:                            │
│  ┌──────────────────────────────────────┐  │
│  │  View costs by:                     │  │
│  │  ├── Resource group                 │  │
│  │  ├── Service type                   │  │
│  │  ├── Tag                            │  │
│  │  ├── Region                         │  │
│  │  └── Time period                    │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Budgets:                                  │
│  ┌──────────────────────────────────────┐  │
│  │  Set monthly budget: £5,000         │  │
│  │  Alert at: 50%, 75%, 90%, 100%      │  │
│  │  Action: Email finance team         │  │
│  │  Action: Trigger automation to      │  │
│  │          shutdown non-prod VMs      │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  Advisor Recommendations:                  │
│  ├── Right-size underutilised VMs         │
│  ├── Use Reserved Instances (1yr/3yr)     │
│  ├── Delete unused resources              │
│  └── Use Spot VMs for batch workloads     │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Create a budget
az consumption budget create \
  --budget-name "monthly-budget" \
  --amount 5000 \
  --category cost \
  --time-grain monthly \
  --start-date 2024-01-01 \
  --end-date 2025-01-01

# View current costs
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --query "[].{Name:instanceName, Cost:pretaxCost}" -o table
```

---

## 5.6 Tagging Strategy

| Tag | Purpose | Example |
|-----|---------|---------|
| **Environment** | Identify lifecycle stage | `prod`, `dev`, `staging` |
| **CostCenter** | Charge back to business unit | `CC-1234` |
| **Owner** | Identify responsible team | `platform-team` |
| **Application** | Group resources by app | `myapp-frontend` |
| **DataClassification** | Security classification | `confidential`, `public` |
| **CreatedBy** | Track who created resource | `terraform`, `manual` |

```bash
# Apply tags to a resource group
az group update \
  --name production-rg \
  --tags Environment=Production CostCenter=CC-1234 Owner=platform-team

# Apply tags to a resource
az resource tag \
  --ids "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Web/sites/myapp" \
  --tags Environment=Production Application=myapp
```

---

## 5.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Resource creation blocked | Azure Policy Deny effect | Check compliance, request policy exemption |
| Can't delete resource | Resource Lock in place | Remove lock (requires Owner/Lock Contributor) |
| Policy shows non-compliant | Existing resources pre-date policy | Remediate existing resources or create remediation task |
| Cost overrun | No budgets configured | Create budgets with alerts at 75% and 90% |
| Tags missing on resources | No tag enforcement policy | Assign "Require tag" policy with Deny effect |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Management Groups** | Hierarchy above subscriptions for policy inheritance |
| **Azure Policy** | Enforce rules (Deny, Audit, DeployIfNotExists, Modify) |
| **Initiatives** | Group multiple policies (e.g., ISO 27001, CIS, PCI DSS) |
| **Resource Locks** | CanNotDelete or ReadOnly — prevent accidental changes |
| **Cost Management** | Budgets, alerts, cost analysis by tag/RG/service |
| **Tagging** | Metadata for cost, ownership, environment, compliance |

---

## Quick Revision Questions

1. **What are the Azure Policy effects?**
   > Deny (block), Audit (log), AuditIfNotExists (check related resource), DeployIfNotExists (auto-remediate), Modify (change tags/properties), Disabled (off).

2. **What is the difference between CanNotDelete and ReadOnly locks?**
   > CanNotDelete allows reads and modifications but blocks deletion. ReadOnly blocks all modifications and deletion — only reads allowed.

3. **What is a Policy Initiative?**
   > A collection of related policies grouped together (e.g., "Security Baseline" with 10+ policies) that can be assigned as a single unit.

4. **What is the Management Group hierarchy?**
   > Root → Management Groups → Subscriptions → Resource Groups → Resources. Policies at higher levels inherit downward.

5. **How do you prevent unexpected Azure costs?**
   > Set budgets with alerts at thresholds (75%, 90%), use Azure Advisor to right-size resources, enforce allowed VM SKUs with policy, and use tags for cost tracking.

6. **Why is tagging important for governance?**
   > Tags enable cost allocation (chargeback), ownership tracking, environment identification, and compliance reporting across resources.

---

[⬅ Previous: Encryption and Certificates](04-encryption-and-certificates.md) | [⬆ Back to Table of Contents](../README.md)
