# Chapter 5: Azure Policy

## Overview
**Azure Policy** enforces organizational standards and assesses compliance at scale. While RBAC controls *who* can do things, Azure Policy controls *what* can be done — regardless of who does it. For DevOps engineers, policies ensure infrastructure guardrails without blocking productivity.

---

## 5.1 Azure Policy vs RBAC

```
┌──────────────── POLICY vs RBAC ────────────────┐
│                                                  │
│  RBAC:  "WHO can do WHAT?"                       │
│  ─────                                           │
│  "DevOps group can create VMs in prod-rg"        │
│                                                  │
│  POLICY: "WHAT is ALLOWED to happen?"            │
│  ──────                                          │
│  "All VMs must use Standard_B2s or larger"       │
│  "All resources must have 'env' tag"             │
│  "Only East US and West Europe regions allowed"  │
│                                                  │
│  ┌──────────┐     ┌──────────┐                   │
│  │   RBAC   │     │  POLICY  │                   │
│  │          │     │          │                   │
│  │  User ───│───▶ │ ── Audit │                   │
│  │  Group   │     │ ── Deny  │                   │
│  │  SP      │     │ ── Modify│                   │
│  │  MI      │     │ ── DINE  │                   │
│  └──────────┘     └──────────┘                   │
│   Controls WHO     Controls WHAT is compliant    │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 5.2 Policy Concepts

```
┌────────────── POLICY HIERARCHY ──────────────┐
│                                               │
│   ┌──────────────────┐                        │
│   │ POLICY DEFINITION │                        │
│   │ • The rule itself  │                        │
│   │ • Conditions       │                        │
│   │ • Effect           │                        │
│   └────────┬───────────┘                        │
│            │                                    │
│            ▼                                    │
│   ┌──────────────────┐                          │
│   │ POLICY ASSIGNMENT │                          │
│   │ • Apply to scope   │                          │
│   │ • Set parameters   │                          │
│   │ • Exclusions       │                          │
│   └────────┬───────────┘                          │
│            │                                      │
│            ▼                                      │
│   ┌──────────────────┐                            │
│   │ POLICY INITIATIVE │ (Policy Set)              │
│   │ • Group of policies│                            │
│   │ • Shared params    │                            │
│   │ • Single assignment│                            │
│   └──────────────────┘                            │
│                                                   │
│   Compliance is evaluated every ~24 hours         │
│   or on resource create/update                    │
│                                                   │
└───────────────────────────────────────────────────┘
```

### Policy Effects

| Effect | Description | Use Case |
|--------|-------------|----------|
| **Audit** | Log non-compliance; don't block | Discovery, reporting |
| **Deny** | Block non-compliant resource creation | Enforce standards |
| **Modify** | Auto-add or change properties | Add missing tags |
| **DeployIfNotExists (DINE)** | Deploy related resource if missing | Auto-enable diagnostics |
| **AuditIfNotExists** | Audit if related resource is missing | Check for missing configs |
| **Append** | Add properties during create/update | Add IP restrictions |
| **Disabled** | Turn off the policy | Testing |

---

## 5.3 Common Built-in Policies

| Policy Name | Effect | Description |
|-------------|--------|-------------|
| Allowed locations | Deny | Restrict regions for resources |
| Require a tag and its value | Deny | Enforce tagging |
| Allowed VM SKUs | Deny | Restrict VM sizes |
| Audit VMs without managed disks | Audit | Check disk type |
| Deploy diagnostics to Log Analytics | DINE | Auto-enable monitoring |
| Inherit a tag from resource group | Modify | Copy RG tags to resources |
| Key Vault should use private endpoint | AuditIfNotExists | Network security |

---

## 5.4 Creating and Assigning Policies

### Using CLI

```bash
# List built-in policy definitions
az policy definition list --query "[?policyType=='BuiltIn'].{Name:displayName, ID:name}" --output table

# Assign built-in policy: Require tag 'env'
az policy assignment create \
  --name "require-env-tag" \
  --display-name "Require env tag on resources" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/871b6d14-10aa-478d-b590-94f262ecfa99" \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG" \
  --params '{"tagName": {"value": "env"}}'

# Assign built-in policy: Allowed locations
az policy assignment create \
  --name "allowed-locations" \
  --display-name "Restrict to US and Europe regions" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c" \
  --params '{"listOfAllowedLocations": {"value": ["eastus", "eastus2", "westeurope", "northeurope"]}}'

# Check compliance
az policy state list \
  --resource-group myRG \
  --query "[].{Resource:resourceId, Compliance:complianceState}" \
  --output table
```

### Custom Policy Definition

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
          "not": {
            "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.managedDisk",
            "exists": true
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {}
}
```

```bash
# Create custom policy
az policy definition create \
  --name "deny-unmanaged-disks" \
  --display-name "Deny VMs without managed disks" \
  --description "Ensures all VMs use managed disks" \
  --rules @policy-rule.json \
  --mode All

# Assign custom policy
az policy assignment create \
  --name "enforce-managed-disks" \
  --policy "deny-unmanaged-disks" \
  --scope "/subscriptions/{sub-id}"
```

---

## 5.5 Policy Initiatives (Policy Sets)

Group related policies into a single **initiative** for easier management.

```
┌────────── POLICY INITIATIVE ──────────┐
│                                        │
│  Initiative: "DevOps Standards"        │
│  ┌──────────────────────────────────┐  │
│  │                                  │  │
│  │  ┌────────────────────────┐      │  │
│  │  │ Policy 1: Require tags │      │  │
│  │  └────────────────────────┘      │  │
│  │  ┌────────────────────────┐      │  │
│  │  │ Policy 2: Allowed SKUs │      │  │
│  │  └────────────────────────┘      │  │
│  │  ┌────────────────────────┐      │  │
│  │  │ Policy 3: Allowed      │      │  │
│  │  │          regions       │      │  │
│  │  └────────────────────────┘      │  │
│  │  ┌────────────────────────┐      │  │
│  │  │ Policy 4: Enable       │      │  │
│  │  │          diagnostics   │      │  │
│  │  └────────────────────────┘      │  │
│  │                                  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Assign once → All 4 policies applied  │
└────────────────────────────────────────┘
```

```bash
# Create initiative definition
az policy set-definition create \
  --name "devops-standards" \
  --display-name "DevOps Standards" \
  --definitions '[
    {"policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/..."},
    {"policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/..."}
  ]'

# Assign initiative
az policy assignment create \
  --name "apply-devops-standards" \
  --policy-set-definition "devops-standards" \
  --scope "/subscriptions/{sub-id}"
```

---

## 5.6 Policy Exemptions

Exempt specific resources from policy enforcement when needed.

```bash
# Create policy exemption
az policy exemption create \
  --name "exempt-dev-vm" \
  --policy-assignment "enforce-managed-disks" \
  --scope "/subscriptions/{sub}/resourceGroups/dev-rg/providers/Microsoft.Compute/virtualMachines/testVM" \
  --exemption-category Waiver \
  --expires-on "2026-06-30"
```

---

## 5.7 Policy in DevOps Pipelines

### Pre-deployment Compliance Check

```yaml
# Azure DevOps Pipeline - Policy compliance gate
stages:
  - stage: ValidateCompliance
    jobs:
      - job: CheckPolicy
        steps:
          - task: AzureCLI@2
            displayName: 'Check policy compliance'
            inputs:
              azureSubscription: 'my-connection'
              scriptType: 'bash'
              inlineScript: |
                # Run what-if to check if deployment would be compliant
                RESULT=$(az deployment group what-if \
                  --resource-group myRG \
                  --template-file main.bicep \
                  --no-pretty-print)
                
                echo "$RESULT"
                
                # Check for policy violations
                NON_COMPLIANT=$(az policy state list \
                  --resource-group myRG \
                  --filter "complianceState eq 'NonCompliant'" \
                  --query "length(@)")
                
                if [ "$NON_COMPLIANT" -gt 0 ]; then
                  echo "##vso[task.logissue type=error]Non-compliant resources found!"
                  exit 1
                fi
```

---

## 5.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Policy doesn't block resources | Effect is "Audit" not "Deny" | Change effect to "Deny" |
| Existing resources not flagged | Compliance scan hasn't run | Trigger scan: `az policy state trigger-scan` |
| Policy too restrictive | Blocking legitimate deployments | Add exemptions or refine conditions |
| DINE policy not deploying | Managed identity missing on assignment | `az policy assignment identity assign` |
| Compliance shows unknown | Policy evaluation in progress | Wait for next evaluation cycle (~24h) |

💡 **Best Practice:** Start with Audit effect to understand impact, then switch to Deny after reviewing compliance reports.

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Policy** | Controls *what* can be done; complements RBAC |
| **Effects** | Audit, Deny, Modify, DeployIfNotExists, Append, Disabled |
| **Initiatives** | Group of policies assigned together |
| **Scope** | Management Group, Subscription, Resource Group |
| **Compliance** | Evaluated every ~24h or on resource changes |
| **Exemptions** | Exclude specific resources with optional expiration |
| **Best Practice** | Start with Audit, then move to Deny |

---

## Quick Revision Questions

1. **What is the difference between Azure Policy and RBAC?**
   > RBAC controls *who* can perform actions, while Azure Policy controls *what* actions are allowed regardless of who performs them.

2. **What are the main policy effects and when would you use each?**
   > Audit (reporting), Deny (block non-compliant), Modify (auto-fix properties), DeployIfNotExists (deploy missing resources), Disabled (testing).

3. **What is a Policy Initiative?**
   > A group of related policy definitions that can be assigned as a single unit for easier management.

4. **How often does Azure evaluate policy compliance?**
   > Approximately every 24 hours automatically, plus on every resource create or update event.

5. **What is the recommended approach when introducing a new policy?**
   > Start with Audit effect to understand the impact, review compliance reports, then switch to Deny once confident.

6. **What is needed for a DeployIfNotExists (DINE) policy to work?**
   > A managed identity must be assigned to the policy assignment, as DINE needs permissions to deploy resources.

---

[⬅ Previous: RBAC Roles](04-rbac-roles.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Virtual Machines ➡](../03-Compute-Services/01-virtual-machines.md)
