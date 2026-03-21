# Unit 3: Azure Security — Topic 3: Azure RBAC

## Overview

**Azure Role-Based Access Control (RBAC)** is the authorization system for managing access to Azure resources. RBAC allows you to grant specific permissions to users, groups, and service principals at different scopes. Properly implemented RBAC ensures least privilege access and is fundamental to Azure security.

---

## 1. RBAC Concepts

```
RBAC COMPONENTS:

SECURITY PRINCIPAL (WHO):
  → User (individual)
  → Group (collection of users)
  → Service Principal (application identity)
  → Managed Identity (Azure resource identity)

ROLE DEFINITION (WHAT):
  → Collection of permissions
  → Actions (allowed operations)
  → NotActions (excluded operations)
  → DataActions (data plane operations)
  → NotDataActions (excluded data operations)

SCOPE (WHERE):
  Management Group → Subscription → Resource Group → Resource
  
  ┌──────────────────────────┐
  │    Management Group      │  Most broad
  │  ┌────────────────────┐  │
  │  │   Subscription     │  │
  │  │  ┌──────────────┐  │  │
  │  │  │ Resource Grp │  │  │
  │  │  │  ┌────────┐  │  │  │
  │  │  │  │Resource│  │  │  │  Most narrow
  │  │  │  └────────┘  │  │  │
  │  │  └──────────────┘  │  │
  │  └────────────────────┘  │
  └──────────────────────────┘
  
  → Permissions INHERIT downward
  → Assignment at subscription → applies to all RGs and resources

ROLE ASSIGNMENT = Principal + Role + Scope

BUILT-IN ROLES:
  Owner:
  → Full access to all resources
  → Can manage access for others
  → DANGEROUS — use sparingly!
  
  Contributor:
  → Can create/manage all resources
  → Cannot manage access (no role assignments)
  → Still powerful
  
  Reader:
  → View all resources
  → Cannot make changes
  → Good for auditors
  
  User Access Administrator:
  → Manage user access to resources
  → Cannot manage resources themselves

SERVICE-SPECIFIC ROLES:
  → Virtual Machine Contributor
  → Storage Blob Data Reader
  → SQL DB Contributor
  → Key Vault Secrets Officer
  → Network Contributor
  → Security Reader
  → Log Analytics Reader
```

---

## 2. Custom Roles

```
CREATING CUSTOM ROLES:

CUSTOM ROLE DEFINITION:
{
  "Name": "VM Restart Operator",
  "Description": "Can restart VMs only",
  "Actions": [
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/12345678-1234-1234-1234-123456789012"
  ]
}

# Create with CLI
az role definition create --role-definition role.json

# List custom roles
az role definition list --custom-role-only true

IMPORTANT PERMISSIONS TO CONTROL:
  Dangerous actions:
  → Microsoft.Authorization/roleAssignments/write
    (Can assign ANY role to anyone)
  → Microsoft.Authorization/roleDefinitions/write
    (Can create custom roles)
  → */write on subscription scope
    (Can modify any resource)
  → Microsoft.Compute/virtualMachines/extensions/write
    (Can execute code on VMs)
```

---

## 3. RBAC Best Practices

```
SECURITY BEST PRACTICES:

1. LEAST PRIVILEGE:
   → Start with Reader role
   → Add specific roles as needed
   → Use resource-level assignments
   → Prefer specific roles over Owner/Contributor
   → Review assignments regularly

2. USE GROUPS:
   → Assign roles to groups, not individual users
   → Easier to manage at scale
   → Single point for access changes
   → Naming convention: AZ-RG-WebApp-Contributor

3. SCOPE APPROPRIATELY:
   → Assign at narrowest scope possible
   → Resource group > Subscription > Management Group
   → Avoid management group assignments except for governance

4. MANAGED IDENTITIES:
   → Use for Azure resource-to-resource access
   → System-assigned: lifecycle tied to resource
   → User-assigned: independent lifecycle, reusable
   → No credentials to manage
   → Automatically rotated

5. AUDIT AND MONITOR:
   # List all role assignments
   az role assignment list --all --output table
   
   # Role assignments for specific user
   az role assignment list --assignee user@domain.com
   
   # Role assignments at subscription level
   az role assignment list --scope /subscriptions/<id>
   
   # Activity log for RBAC changes
   az monitor activity-log list \
     --resource-provider Microsoft.Authorization

COMMON MISCONFIGURATIONS:
  → Owner at subscription level (too broad)
  → Contributor to everyone in subscription
  → Custom roles with wildcard actions
  → Service principals with Owner
  → No regular access reviews
  → Standing privileged access (use PIM)
```

---

## 4. RBAC vs Azure AD Roles

```
TWO ROLE SYSTEMS:

AZURE RBAC ROLES:
  → Control Azure RESOURCE access
  → Scope: MG → Sub → RG → Resource
  → Examples: VM Contributor, Storage Reader
  → Managed in Azure portal (Access Control)
  → Affect what you can do with Azure services

AZURE AD ROLES:
  → Control Azure AD DIRECTORY access
  → Scope: Azure AD tenant
  → Examples: Global Admin, User Admin
  → Managed in Azure AD portal
  → Affect identity management

COMPARISON:
  Feature       | Azure RBAC          | Azure AD Roles
  Controls      | Azure resources     | Azure AD objects
  Scope         | Resource hierarchy  | Tenant-wide
  Examples      | VM Contributor      | Global Administrator
  Assignment    | ARM (Azure Portal)  | Azure AD
  Custom roles  | Yes                 | Yes (P1/P2)
  
KEY RELATIONSHIPS:
  → Global Administrator in Azure AD can gain 
    User Access Administrator on ALL subscriptions
  → This is a privilege escalation path!
  → Protect Global Admin role carefully
  → Use PIM for both Azure AD and Azure RBAC roles
```

---

## 5. Assessment and Enumeration

```
RBAC SECURITY ASSESSMENT:

ENUMERATION:
  # All role assignments in subscription
  az role assignment list --all --include-inherited
  
  # Find Owner assignments
  az role assignment list --role "Owner" --all
  
  # Find assignments with broad scope
  az role assignment list --scope "/subscriptions/<id>"
  
  # Custom role definitions
  az role definition list --custom-role-only
  
  # Service principal assignments
  az role assignment list --assignee <sp-app-id>

POWERSHELL:
  # Connect
  Connect-AzAccount
  
  # List all role assignments
  Get-AzRoleAssignment | Select-Object DisplayName, RoleDefinitionName, Scope
  
  # Find high-privilege assignments
  Get-AzRoleAssignment | Where-Object {
    $_.RoleDefinitionName -in @("Owner","Contributor","User Access Administrator")
  }

TOOLS:
  → AzureHound: Map Azure RBAC and AD relationships
  → ScoutSuite: Multi-cloud security auditing
  → Azucar: Azure security assessment
  → PowerZure: Azure offensive toolkit
  
SECURITY REVIEW CHECKLIST:
  [ ] No individual Owner assignments (use groups)
  [ ] Owner at subscription level justified
  [ ] Custom roles reviewed for excessive permissions
  [ ] Service principals have minimum required roles
  [ ] Managed identities used instead of service principals
  [ ] PIM enabled for privileged RBAC roles
  [ ] Regular access reviews conducted
  [ ] Activity logs monitored for RBAC changes
```

---

## Summary Table

| Role | Access Level | Use Case |
|------|-------------|----------|
| Owner | Full + access mgmt | Last resort, use PIM |
| Contributor | Full, no access mgmt | Resource administrators |
| Reader | Read-only | Auditors, monitoring |
| Custom | Specific actions | Least privilege |
| Managed Identity | Automated access | Azure-to-Azure |

---

## Revision Questions

1. What are the four components of an Azure RBAC role assignment?
2. How does scope inheritance work in Azure RBAC?
3. What is the difference between Azure RBAC roles and Azure AD roles?
4. Why should managed identities be preferred over service principals?
5. How can you identify overprivileged role assignments?

---

*Previous: [02-azure-ad.md](02-azure-ad.md) | Next: [04-network-security-groups.md](04-network-security-groups.md)*

---

*[Back to README](../README.md)*
