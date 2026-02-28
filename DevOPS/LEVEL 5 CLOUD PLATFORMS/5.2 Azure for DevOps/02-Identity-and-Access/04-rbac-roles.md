# Chapter 4: RBAC Roles

## Overview
**Role-Based Access Control (RBAC)** is Azure's authorization system for managing who has access to Azure resources, what they can do with those resources, and what areas they have access to. RBAC is essential for implementing least-privilege access in DevOps environments.

---

## 4.1 How RBAC Works

```
┌──────────────────── RBAC COMPONENTS ────────────────────┐
│                                                          │
│   WHO?           WHAT?           WHERE?                  │
│   (Security     (Role           (Scope)                  │
│    Principal)    Definition)                              │
│                                                          │
│   ┌─────────┐   ┌─────────┐   ┌─────────────────┐      │
│   │  User   │   │  Owner  │   │ Management Group│      │
│   │  Group  │ + │  Contrib│ + │ Subscription    │      │
│   │  SP     │   │  Reader │   │ Resource Group  │      │
│   │  MI     │   │  Custom │   │ Resource        │      │
│   └─────────┘   └─────────┘   └─────────────────┘      │
│       │              │               │                   │
│       └──────────────┴───────────────┘                   │
│                      │                                   │
│              ┌───────┴───────┐                           │
│              │ ROLE          │                           │
│              │ ASSIGNMENT    │                           │
│              │               │                           │
│              │ "User X has   │                           │
│              │  Contributor  │                           │
│              │  role on      │                           │
│              │  Resource     │                           │
│              │  Group Y"     │                           │
│              └───────────────┘                           │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### RBAC Inheritance

```
┌────────── RBAC SCOPE INHERITANCE ──────────┐
│                                             │
│   Management Group ◄── Widest scope         │
│        │                                    │
│        ├── Subscription                     │
│        │       │                            │
│        │       ├── Resource Group            │
│        │       │       │                    │
│        │       │       ├── Resource          │
│        │       │       │   (VM, DB, etc.)   │
│        │       │       │                    │
│        │       │       └── Most specific    │
│                                             │
│   Roles assigned at a higher level are      │
│   INHERITED by all child scopes.            │
│                                             │
│   Example:                                  │
│   "Contributor" on Subscription =           │
│   Contributor on ALL RGs and resources      │
│   within that subscription                  │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 4.2 Built-in Roles

### Fundamental Roles

| Role | Description | Actions |
|------|-------------|---------|
| **Owner** | Full access + can assign roles | `*` |
| **Contributor** | Full access except role assignments | `*` minus `Authorization/*` |
| **Reader** | View all resources | `*/read` |
| **User Access Administrator** | Manage user access to resources | `Authorization/*` |

### Common Service-Specific Roles

| Role | Scope | Description |
|------|-------|-------------|
| Virtual Machine Contributor | Compute | Manage VMs but not network or storage |
| Network Contributor | Network | Manage networks |
| Storage Blob Data Reader | Storage | Read blob data |
| Storage Blob Data Contributor | Storage | Read/write blob data |
| Key Vault Secrets User | Key Vault | Read secrets |
| Key Vault Administrator | Key Vault | Full Key Vault management |
| SQL DB Contributor | SQL | Manage SQL databases |
| AcrPush | ACR | Push images to container registry |
| AcrPull | ACR | Pull images from container registry |
| Azure Kubernetes Service Cluster Admin | AKS | Full cluster admin |

---

## 4.3 Managing Role Assignments

### Azure CLI

```bash
# List all role assignments for a resource group
az role assignment list \
  --resource-group myRG \
  --output table

# Assign a role to a user
az role assignment create \
  --assignee "john@contoso.com" \
  --role "Contributor" \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG"

# Assign role to a service principal
az role assignment create \
  --assignee <app-id> \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault"

# Assign role to a group
az role assignment create \
  --assignee <group-object-id> \
  --role "Reader" \
  --scope "/subscriptions/{sub-id}"

# Remove a role assignment
az role assignment delete \
  --assignee "john@contoso.com" \
  --role "Contributor" \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG"

# List all built-in roles
az role definition list --output table

# Get details of a specific role
az role definition list --name "Contributor" --output json
```

### Azure PowerShell

```powershell
# List role assignments
Get-AzRoleAssignment -ResourceGroupName "myRG" | Format-Table

# Assign role
New-AzRoleAssignment -SignInName "john@contoso.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "myRG"

# Remove role assignment
Remove-AzRoleAssignment -SignInName "john@contoso.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "myRG"
```

---

## 4.4 Custom Roles

When built-in roles don't match your needs, create custom roles.

```json
{
  "Name": "DevOps Deploy Operator",
  "IsCustom": true,
  "Description": "Can deploy web apps and manage app settings, but not delete resources",
  "Actions": [
    "Microsoft.Web/sites/read",
    "Microsoft.Web/sites/write",
    "Microsoft.Web/sites/config/*",
    "Microsoft.Web/sites/slots/*",
    "Microsoft.Web/sites/restart/action",
    "Microsoft.Web/serverfarms/read",
    "Microsoft.Resources/deployments/*",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [
    "Microsoft.Web/sites/delete"
  ],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

```bash
# Create custom role
az role definition create --role-definition @custom-role.json

# Update custom role
az role definition update --role-definition @custom-role-updated.json

# Delete custom role
az role definition delete --name "DevOps Deploy Operator"

# List custom roles
az role definition list --custom-role-only --output table
```

### Actions vs DataActions

```
┌────────── ACTIONS vs DATA ACTIONS ──────────┐
│                                              │
│  ACTIONS (Management Plane)                  │
│  ───────────────────────                     │
│  • Control the resource itself               │
│  • Create/delete/configure resources         │
│  • Example: Create a storage account         │
│    Microsoft.Storage/storageAccounts/write    │
│                                              │
│  DATA ACTIONS (Data Plane)                   │
│  ─────────────────────────                   │
│  • Operate on data WITHIN the resource       │
│  • Read/write data stored in the resource    │
│  • Example: Read blobs in storage            │
│    Microsoft.Storage/storageAccounts/         │
│    blobServices/containers/blobs/read        │
│                                              │
│  ┌─────────────────────────────────────┐     │
│  │  Storage Account                    │     │
│  │  ┌───────────────┐                  │     │
│  │  │  Actions:     │  Create/delete   │     │
│  │  │  (mgmt plane) │  the account     │     │
│  │  └───────────────┘                  │     │
│  │  ┌───────────────┐                  │     │
│  │  │  DataActions: │  Read/write      │     │
│  │  │  (data plane) │  blob data       │     │
│  │  └───────────────┘                  │     │
│  └─────────────────────────────────────┘     │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 4.5 Deny Assignments

Deny assignments **block** specific actions even if a role assignment grants access.

```
┌──────────── ROLE EVALUATION ORDER ────────────┐
│                                                 │
│   Request Comes In                              │
│        │                                        │
│        ▼                                        │
│   ┌─────────────┐                               │
│   │ Check Deny  │── YES ──▶ ACCESS DENIED       │
│   │ Assignments │                               │
│   └──────┬──────┘                               │
│          │ NO                                   │
│          ▼                                      │
│   ┌─────────────┐                               │
│   │ Check Role  │── NO ──▶ ACCESS DENIED        │
│   │ Assignments │        (implicit deny)        │
│   └──────┬──────┘                               │
│          │ YES                                  │
│          ▼                                      │
│   ACCESS GRANTED                                │
│                                                 │
│   Priority: Deny > Allow > Implicit Deny        │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 4.6 DevOps Team RBAC Strategy

```
┌────────────── TEAM RBAC STRATEGY ──────────────┐
│                                                  │
│   PRODUCTION SUBSCRIPTION                        │
│   ┌────────────────────────────────────────┐     │
│   │                                        │     │
│   │  DevOps Engineers Group:               │     │
│   │  └── Contributor on prod-rg            │     │
│   │                                        │     │
│   │  Developers Group:                     │     │
│   │  └── Reader on prod-rg                 │     │
│   │                                        │     │
│   │  CI/CD Service Principal:              │     │
│   │  └── Custom "Deploy Operator" on prod  │     │
│   │                                        │     │
│   │  Security Team:                        │     │
│   │  └── Security Reader on subscription   │     │
│   │                                        │     │
│   └────────────────────────────────────────┘     │
│                                                  │
│   DEV SUBSCRIPTION                               │
│   ┌────────────────────────────────────────┐     │
│   │                                        │     │
│   │  Developers Group:                     │     │
│   │  └── Contributor on dev-rg             │     │
│   │                                        │     │
│   │  DevOps Engineers Group:               │     │
│   │  └── Owner on dev subscription         │     │
│   │                                        │     │
│   └────────────────────────────────────────┘     │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 4.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `AuthorizationFailed` | Missing role assignment | Check: `az role assignment list --assignee <id>` |
| User has role but still denied | Deny assignment in effect | Check deny assignments at higher scopes |
| Role assigned but not working | Wrong scope (too narrow or wrong resource) | Verify scope matches the target resource path |
| Cannot assign roles | Lacking `User Access Administrator` or `Owner` | Request elevated permissions or use PIM |
| Custom role not visible | `AssignableScopes` too restrictive | Ensure scope includes current subscription |
| Permissions take time | RBAC cache delay (~5 minutes) | Wait a few minutes; changes propagate eventually |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **RBAC** | Who + What + Where = Role Assignment |
| **Scope Inheritance** | Management Group → Subscription → RG → Resource |
| **Fundamental Roles** | Owner, Contributor, Reader, User Access Admin |
| **Custom Roles** | Define specific Actions, NotActions, DataActions |
| **Actions vs DataActions** | Management plane vs data plane operations |
| **Deny Assignments** | Override allow; priority: Deny > Allow > Implicit Deny |
| **Best Practice** | Use groups, least privilege, narrowest scope |

---

## Quick Revision Questions

1. **What are the three elements of an RBAC role assignment?**
   > Security principal (who), role definition (what), and scope (where).

2. **What is the difference between Owner and Contributor roles?**
   > Owner can do everything including assign roles to others. Contributor can do everything except manage role assignments.

3. **How does RBAC scope inheritance work?**
   > Roles assigned at a higher level (e.g., subscription) are inherited by all child scopes (resource groups, resources).

4. **What is the difference between Actions and DataActions?**
   > Actions control the management plane (create/delete resources). DataActions control the data plane (read/write data within resources).

5. **In what order does Azure evaluate access?**
   > Deny assignments first (explicit deny), then role assignments (allow), then implicit deny if no matching assignment.

6. **When should you create a custom role?**
   > When built-in roles are either too permissive or too restrictive for your specific use case, such as allowing deployment but not deletion.

---

[⬅ Previous: Managed Identities](03-managed-identities.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Policy ➡](05-azure-policy.md)
