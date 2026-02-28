# Chapter 4: Azure Resource Manager (ARM)

## Overview
Azure Resource Manager (ARM) is the **deployment and management service** for Azure. Every request to create, update, delete, or manage an Azure resource goes through ARM. Understanding ARM is essential for DevOps engineers implementing Infrastructure as Code (IaC).

---

## 4.1 What is Azure Resource Manager?

ARM provides a consistent management layer that handles all requests from Azure Portal, CLI, PowerShell, SDKs, and REST APIs.

```
┌─────────────────────────────────────────────────────────────────┐
│                AZURE RESOURCE MANAGER (ARM)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐       │
│   │ Portal │ │  CLI   │ │  PS    │ │  SDK   │ │  REST  │       │
│   └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘       │
│       │          │          │          │          │              │
│       └──────────┴──────────┴──────────┴──────────┘              │
│                             │                                    │
│                    ┌────────┴────────┐                           │
│                    │  Authentication │                           │
│                    │  (Azure AD /    │                           │
│                    │   Entra ID)     │                           │
│                    └────────┬────────┘                           │
│                             │                                    │
│                    ┌────────┴────────┐                           │
│                    │    ARM Layer    │                           │
│                    │                 │                           │
│                    │  • Validates    │                           │
│                    │  • Authorizes   │                           │
│                    │  • Routes       │                           │
│                    │  • Orchestrates │                           │
│                    └────────┬────────┘                           │
│                             │                                    │
│          ┌──────────────────┼──────────────────┐                │
│          │                  │                  │                │
│   ┌──────┴──────┐  ┌───────┴──────┐  ┌────────┴─────┐         │
│   │  Compute    │  │   Storage    │  │  Networking  │         │
│   │  Provider   │  │   Provider   │  │  Provider    │         │
│   └─────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### ARM Key Functions

| Function | Description |
|----------|-------------|
| **Authentication** | Validates identity via Azure AD/Entra ID |
| **Authorization** | Checks RBAC permissions for the operation |
| **Validation** | Ensures the request is well-formed |
| **Routing** | Sends requests to the appropriate resource provider |
| **Orchestration** | Manages dependencies between resources during deployment |
| **Idempotency** | Same template deployed multiple times = same result |

---

## 4.2 Resource Providers

Resource providers are the services that supply Azure resources. Each provider must be registered before use.

```
┌──────────────────────────────────────────────────────────┐
│              RESOURCE PROVIDERS                           │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Microsoft.Compute ──── VMs, Scale Sets, Disks          │
│  Microsoft.Storage ──── Storage Accounts, Blobs         │
│  Microsoft.Network ──── VNets, Load Balancers, NSGs     │
│  Microsoft.Web ──────── App Services, Functions         │
│  Microsoft.Sql ──────── SQL Databases                   │
│  Microsoft.KeyVault ─── Key Vaults, Secrets             │
│  Microsoft.ContainerService ── AKS                      │
│  Microsoft.ContainerRegistry ─ ACR                      │
│                                                          │
│  Resource ID Format:                                     │
│  /subscriptions/{sub-id}                                │
│    /resourceGroups/{rg-name}                            │
│      /providers/Microsoft.Compute                       │
│        /virtualMachines/{vm-name}                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Managing Resource Providers

```bash
# List all registered providers
az provider list --query "[?registrationState=='Registered'].{Provider:namespace}" --output table

# Register a provider
az provider register --namespace Microsoft.ContainerService

# Check registration status
az provider show --namespace Microsoft.ContainerService --query "registrationState"
```

---

## 4.3 ARM Templates (JSON)

ARM templates are **JSON files** that define the infrastructure and configuration for your Azure deployment. They enable Infrastructure as Code.

### Template Structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageName": {
      "type": "string",
      "minLength": 3,
      "maxLength": 24,
      "metadata": {
        "description": "Name of the storage account"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "sku": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ]
    }
  },
  "variables": {
    "uniqueStorageName": "[concat(parameters('storageName'), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "[variables('uniqueStorageName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true,
        "minimumTlsVersion": "TLS1_2"
      }
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "string",
      "value": "[reference(variables('uniqueStorageName')).primaryEndpoints.blob]"
    }
  }
}
```

### Template Sections Explained

```
┌──────────────────────────────────────────────────────┐
│              ARM TEMPLATE STRUCTURE                    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────┐                                     │
│  │   $schema   │  Specifies template language version│
│  └──────┬──────┘                                     │
│         ▼                                            │
│  ┌─────────────┐                                     │
│  │ parameters  │  Input values at deployment time    │
│  └──────┬──────┘                                     │
│         ▼                                            │
│  ┌─────────────┐                                     │
│  │  variables  │  Reusable values within template    │
│  └──────┬──────┘                                     │
│         ▼                                            │
│  ┌─────────────┐                                     │
│  │  functions  │  User-defined template functions    │
│  └──────┬──────┘  (optional)                         │
│         ▼                                            │
│  ┌─────────────┐                                     │
│  │  resources  │  Azure resources to deploy          │
│  └──────┬──────┘  (REQUIRED)                         │
│         ▼                                            │
│  ┌─────────────┐                                     │
│  │   outputs   │  Values returned after deployment   │
│  └─────────────┘                                     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 4.4 Bicep (Modern ARM Templates)

**Bicep** is a domain-specific language (DSL) that simplifies ARM template authoring. It compiles to ARM JSON.

### Bicep vs ARM JSON Comparison

```
ARM JSON (verbose)                    Bicep (clean)
──────────────────                    ─────────────
{                                     param location string = 
  "parameters": {                       resourceGroup().location
    "location": {                     param storageName string
      "type": "string",              
      "defaultValue":                 resource storage 'Microsoft.Storage/
        "[resourceGroup()               storageAccounts@2023-01-01' = {
          .location]"                   name: storageName
    }                                   location: location
  },                                    sku: { name: 'Standard_LRS' }
  "resources": [{                       kind: 'StorageV2'
    "type": "Microsoft.                 properties: {
      Storage/                            supportsHttpsTrafficOnly: true
      storageAccounts",                 }
    "apiVersion":                     }
      "2023-01-01",                   
    ...                               output endpoint string = 
  }]                                    storage.properties.
}                                         primaryEndpoints.blob
```

### Complete Bicep Example

```bicep
// main.bicep - Deploy a web app with SQL database

@description('Location for all resources')
param location string = resourceGroup().location

@description('App name (must be globally unique)')
@minLength(3)
@maxLength(24)
param appName string

@description('SQL admin password')
@secure()
param sqlAdminPassword string

// Variables
var appServicePlanName = '${appName}-plan'
var sqlServerName = '${appName}-sql'
var sqlDatabaseName = '${appName}-db'

// App Service Plan
resource appServicePlan 'Microsoft.Web/serverfarms@2023-01-01' = {
  name: appServicePlanName
  location: location
  sku: {
    name: 'B1'
    tier: 'Basic'
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

// Web App
resource webApp 'Microsoft.Web/sites@2023-01-01' = {
  name: appName
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: 'NODE|18-lts'
    }
  }
}

// SQL Server
resource sqlServer 'Microsoft.Sql/servers@2023-05-01-preview' = {
  name: sqlServerName
  location: location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: sqlAdminPassword
  }
}

// SQL Database
resource sqlDatabase 'Microsoft.Sql/servers/databases@2023-05-01-preview' = {
  parent: sqlServer
  name: sqlDatabaseName
  location: location
  sku: {
    name: 'S0'
    tier: 'Standard'
  }
}

// Outputs
output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
```

### Deploying Bicep/ARM Templates

```bash
# Deploy ARM template
az deployment group create \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters storageName=mystgaccount

# Deploy Bicep file
az deployment group create \
  --resource-group myRG \
  --template-file main.bicep \
  --parameters appName=myapp sqlAdminPassword=SecureP@ss123!

# What-If operation (preview changes)
az deployment group what-if \
  --resource-group myRG \
  --template-file main.bicep \
  --parameters appName=myapp

# Validate template
az deployment group validate \
  --resource-group myRG \
  --template-file main.bicep

# Deploy at subscription level
az deployment sub create \
  --location eastus \
  --template-file main.bicep
```

---

## 4.5 Deployment Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| **Incremental** (default) | Adds/updates resources in template; leaves existing resources untouched | Safe, standard deployments |
| **Complete** | Depletes resources NOT in template; use with caution | Full environment reset |

```
┌────────── INCREMENTAL MODE ──────────┐    ┌─────────── COMPLETE MODE ───────────┐
│                                       │    │                                      │
│  Existing RG:     Template:           │    │  Existing RG:     Template:          │
│  ┌──────┐         ┌──────┐           │    │  ┌──────┐         ┌──────┐          │
│  │ VM-A │         │ VM-A │ (update)  │    │  │ VM-A │         │ VM-A │ (update) │
│  │ VM-B │         │ VM-C │ (create)  │    │  │ VM-B │         │ VM-C │ (create) │
│  └──────┘         └──────┘           │    │  └──────┘         └──────┘          │
│                                       │    │                                      │
│  Result:                              │    │  Result:                             │
│  ┌──────┐                             │    │  ┌──────┐                            │
│  │ VM-A │ (updated)                   │    │  │ VM-A │ (updated)                  │
│  │ VM-B │ (unchanged)                 │    │  │ VM-B │ ← DELETED!                │
│  │ VM-C │ (created)                   │    │  │ VM-C │ (created)                  │
│  └──────┘                             │    │  └──────┘                            │
└───────────────────────────────────────┘    └──────────────────────────────────────┘
```

⚠️ **Warning:** Complete mode deletes resources not in the template. Always use `--mode Incremental` (default) unless you explicitly want to purge unmanaged resources.

---

## 4.6 Template Specs and Linked Templates

### Template Specs
Template specs let you store ARM templates as Azure resources for versioning and sharing.

```bash
# Create a template spec
az ts create \
  --name webAppTemplate \
  --version 1.0 \
  --resource-group myRG \
  --template-file main.bicep

# Deploy from template spec
az deployment group create \
  --resource-group targetRG \
  --template-spec "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.Resources/templateSpecs/webAppTemplate/versions/1.0"
```

### Linked Templates (Modules in Bicep)

```bicep
// modules/storage.bicep
param name string
param location string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: name
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

output id string = storageAccount.id

// main.bicep
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    name: 'mystorageacct'
    location: resourceGroup().location
  }
}
```

---

## 4.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `InvalidTemplate` error | Syntax error in JSON/Bicep | Use `az deployment group validate` before deploying |
| `ResourceNotFound` | Resource provider not registered | `az provider register --namespace <provider>` |
| Deployment stuck | Dependency loop or long-running operation | Check deployment operations: `az deployment group show` |
| Complete mode deleted resources | Resources not in template were removed | Always use Incremental mode; review with `what-if` first |
| Parameter validation failed | Value doesn't meet constraints | Check `allowedValues`, `minLength`, `maxLength` |
| Template too large | ARM JSON limit is 4 MB | Use linked templates or Bicep modules |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **ARM** | Central management layer; authenticates, authorizes, and routes all requests |
| **Resource Providers** | Services that supply resources (e.g., Microsoft.Compute) |
| **ARM Templates** | JSON-based IaC; declarative, idempotent |
| **Bicep** | Cleaner DSL for ARM; compiles to JSON; recommended over raw JSON |
| **Deployment Modes** | Incremental (safe, default) vs Complete (deletes unmanaged resources) |
| **What-If** | Preview deployment changes before applying |
| **Template Specs** | Store and version templates as Azure resources |
| **Modules** | Reusable Bicep components for modular IaC |

---

## Quick Revision Questions

1. **What does ARM stand for and what is its primary role?**
   > Azure Resource Manager — it's the deployment and management service that handles all API requests for Azure resources.

2. **What is the difference between Incremental and Complete deployment modes?**
   > Incremental adds/updates resources without touching existing ones. Complete deletes any resources in the resource group that are not in the template.

3. **How does Bicep relate to ARM templates?**
   > Bicep is a DSL that compiles to ARM JSON. It provides cleaner syntax with the same functionality.

4. **What is a resource provider and how do you register one?**
   > A resource provider supplies Azure resources (e.g., Microsoft.Compute for VMs). Register with `az provider register --namespace <provider>`.

5. **What CLI command lets you preview changes before deploying?**
   > `az deployment group what-if --template-file main.bicep` shows what would change without deploying.

6. **What is the maximum size of an ARM template file?**
   > 4 MB. For larger deployments, use linked templates or Bicep modules.

---

[⬅ Previous: Azure Portal, CLI, PowerShell](03-azure-portal-cli-powershell.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Resource Groups and Subscriptions ➡](05-resource-groups-and-subscriptions.md)
