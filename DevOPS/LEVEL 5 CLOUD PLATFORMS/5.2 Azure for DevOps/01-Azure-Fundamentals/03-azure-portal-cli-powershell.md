# Chapter 3: Azure Portal, CLI, PowerShell

## Overview
Azure provides multiple management interfaces for provisioning and managing resources. DevOps engineers must be proficient in all three: the **Azure Portal** (GUI), **Azure CLI** (cross-platform command line), and **Azure PowerShell** (PowerShell module). Automation and Infrastructure as Code rely heavily on CLI and PowerShell.

---

## 3.1 Management Interfaces Comparison

```
┌────────────────────────────────────────────────────────────────┐
│             AZURE MANAGEMENT INTERFACES                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────┐    ┌──────────┐    ┌─────────────┐              │
│  │  Azure   │    │  Azure   │    │   Azure     │              │
│  │  Portal  │    │   CLI    │    │ PowerShell  │              │
│  │  (GUI)   │    │  (bash)  │    │   (pwsh)    │              │
│  └────┬─────┘    └────┬─────┘    └──────┬──────┘              │
│       │               │                 │                      │
│       └───────────────┼─────────────────┘                      │
│                       │                                        │
│              ┌────────┴────────┐                               │
│              │  Azure REST API │                               │
│              └────────┬────────┘                               │
│                       │                                        │
│              ┌────────┴────────┐                               │
│              │ Azure Resource  │                               │
│              │    Manager      │                               │
│              └─────────────────┘                               │
│                                                                │
│  All tools ultimately call the same ARM REST API               │
└────────────────────────────────────────────────────────────────┘
```

| Feature | Portal | Azure CLI | Azure PowerShell |
|---------|--------|-----------|-----------------|
| **Interface** | Web GUI | Command-line (bash) | PowerShell cmdlets |
| **Platform** | Any browser | Windows, macOS, Linux | Windows, macOS, Linux |
| **Syntax** | Point-and-click | `az <group> <command>` | `Verb-AzNoun` |
| **Scripting** | Limited | Bash/shell scripts | PowerShell scripts |
| **Output** | Visual | JSON, table, TSV, YAML | Objects (PSObject) |
| **Best For** | Exploration, learning | CI/CD pipelines, Linux automation | Windows automation, complex logic |
| **Cloud Shell** | ✅ Built-in | ✅ Available | ✅ Available |

---

## 3.2 Azure Portal

The Azure Portal (`portal.azure.com`) is Microsoft's web-based management console.

### Key Portal Features

```
┌─────────────────────────────────────────────────────────────┐
│  AZURE PORTAL LAYOUT                                         │
├──────┬──────────────────────────────────────────────────────┤
│      │  ┌────────────────────────────────────────┐          │
│  S   │  │           SEARCH BAR                   │          │
│  I   │  └────────────────────────────────────────┘          │
│  D   │  ┌────────────────────────────────────────┐          │
│  E   │  │                                        │          │
│  B   │  │         MAIN CONTENT AREA              │          │
│  A   │  │                                        │          │
│  R   │  │    ┌──────────┐  ┌──────────┐          │          │
│      │  │    │ Resource │  │ Resource │          │          │
│  •   │  │    │  Card 1  │  │  Card 2  │          │          │
│  D   │  │    └──────────┘  └──────────┘          │          │
│  a   │  │                                        │          │
│  s   │  │    ┌──────────┐  ┌──────────┐          │          │
│  h   │  │    │ Resource │  │ Resource │          │          │
│  b   │  │    │  Card 3  │  │  Card 4  │          │          │
│  o   │  │    └──────────┘  └──────────┘          │          │
│  a   │  │                                        │          │
│  r   │  └────────────────────────────────────────┘          │
│  d   │  ┌────────────────────────────────────────┐          │
│      │  │        NOTIFICATIONS / ACTIVITY         │          │
│      │  └────────────────────────────────────────┘          │
└──────┴──────────────────────────────────────────────────────┘
```

### Portal Tips for DevOps

- **Cloud Shell** — Built-in terminal (Bash or PowerShell) accessible from any browser
- **Resource Explorer** — Shows raw ARM API responses for any resource
- **Cost Analysis** — Monitor spending by resource group, tag, or service
- **Custom Dashboards** — Pin metrics, resources, and queries for quick access
- **Favorites** — Customize the sidebar for frequently used services

---

## 3.3 Azure CLI

Azure CLI is a **cross-platform** command-line tool for managing Azure resources. Written in Python, it uses a consistent `az <group> <subgroup> <command>` pattern.

### Installation

```bash
# Windows (via MSI or winget)
winget install Microsoft.AzureCLI

# macOS (via Homebrew)
brew install azure-cli

# Linux (Ubuntu/Debian)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Docker
docker run -it mcr.microsoft.com/azure-cli

# Verify installation
az version
```

### Authentication

```bash
# Interactive login (browser-based)
az login

# Login with service principal (for CI/CD)
az login --service-principal \
  --username <app-id> \
  --password <secret> \
  --tenant <tenant-id>

# Login with managed identity (from Azure VM)
az login --identity

# Set active subscription
az account set --subscription "My Subscription"
az account show
```

### Common CLI Patterns

```bash
# ── RESOURCE GROUPS ──
az group create --name myRG --location eastus
az group list --output table
az group delete --name myRG --yes --no-wait

# ── VIRTUAL MACHINES ──
az vm create --resource-group myRG --name myVM \
  --image Ubuntu2204 --size Standard_B2s \
  --admin-username azureuser --generate-ssh-keys

az vm list --output table
az vm stop --resource-group myRG --name myVM
az vm start --resource-group myRG --name myVM
az vm delete --resource-group myRG --name myVM --yes

# ── QUERY WITH JMESPATH ──
az vm list --query "[].{Name:name, RG:resourceGroup, Status:powerState}" --output table

# ── OUTPUT FORMATS ──
az vm list --output json      # Default, full detail
az vm list --output table     # Human-readable
az vm list --output tsv       # Tab-separated, for scripts
az vm list --output yaml      # YAML format
```

### CLI Configuration

```bash
# Set defaults to avoid repetition
az configure --defaults group=myRG location=eastus

# After setting defaults, you can omit those parameters
az vm create --name myVM --image Ubuntu2204

# View current configuration
az configure --list-defaults
```

---

## 3.4 Azure PowerShell

Azure PowerShell is a set of **cmdlets** built on the Az module. It follows PowerShell's `Verb-Noun` convention and works with rich .NET objects.

### Installation

```powershell
# Install Az module (PowerShell 7+)
Install-Module -Name Az -Repository PSGallery -Force -AllowClobber

# Update Az module
Update-Module -Name Az

# Import specific sub-module
Import-Module Az.Compute

# Verify installation
Get-Module -Name Az -ListAvailable
```

### Authentication

```powershell
# Interactive login
Connect-AzAccount

# Service principal login
$securePassword = ConvertTo-SecureString "secret" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("app-id", $securePassword)
Connect-AzAccount -ServicePrincipal -Credential $credential -Tenant "tenant-id"

# Managed identity
Connect-AzAccount -Identity

# Set subscription context
Set-AzContext -SubscriptionName "My Subscription"
Get-AzContext
```

### Common PowerShell Patterns

```powershell
# ── RESOURCE GROUPS ──
New-AzResourceGroup -Name "myRG" -Location "eastus"
Get-AzResourceGroup | Format-Table ResourceGroupName, Location
Remove-AzResourceGroup -Name "myRG" -Force -AsJob

# ── VIRTUAL MACHINES ──
New-AzVM -ResourceGroupName "myRG" `
  -Name "myVM" `
  -Location "eastus" `
  -Image "Ubuntu2204" `
  -Size "Standard_B2s" `
  -Credential (Get-Credential)

Get-AzVM | Format-Table Name, ResourceGroupName, Location
Stop-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force
Start-AzVM -ResourceGroupName "myRG" -Name "myVM"
Remove-AzVM -ResourceGroupName "myRG" -Name "myVM" -Force

# ── FILTERING AND PIPING ──
Get-AzVM | Where-Object { $_.Location -eq "eastus" } | 
  Select-Object Name, ResourceGroupName, @{N="Size"; E={$_.HardwareProfile.VmSize}}

# ── PARALLEL OPERATIONS ──
Get-AzVM | ForEach-Object -Parallel {
    Stop-AzVM -ResourceGroupName $_.ResourceGroupName -Name $_.Name -Force
} -ThrottleLimit 5
```

---

## 3.5 Azure Cloud Shell

Cloud Shell is a **browser-based** shell experience built into the Azure Portal. It provides a pre-configured environment with Azure CLI and PowerShell.

```
┌─────────────────────────────────────────────────────────┐
│                 AZURE CLOUD SHELL                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────────────┐                               │
│   │    Azure Portal     │                               │
│   │    ┌──────────┐     │                               │
│   │    │ Cloud    │     │                               │
│   │    │ Shell    │     │                               │
│   │    │ >_      │     │                               │
│   │    └────┬─────┘     │                               │
│   └─────────┼───────────┘                               │
│             │                                           │
│   ┌─────────┴──────────────────────────┐                │
│   │    Backed by:                       │                │
│   │    • Azure Files (persistent)       │                │
│   │    • Linux container                │                │
│   │    • Pre-installed tools:           │                │
│   │      - az CLI                       │                │
│   │      - PowerShell 7                 │                │
│   │      - Terraform, Ansible           │                │
│   │      - kubectl, helm               │                │
│   │      - git, vim, nano              │                │
│   │      - Python, Node.js             │                │
│   └────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

### Cloud Shell Features

| Feature | Detail |
|---------|--------|
| **Persistent Storage** | 5 GB Azure Files share mounted at `$HOME` |
| **Authentication** | Auto-authenticated with your portal session |
| **Pre-installed Tools** | az CLI, PowerShell, Terraform, kubectl, git, etc. |
| **File Upload/Download** | Transfer files to/from Cloud Shell |
| **Timeout** | 20 minutes of inactivity |
| **Cost** | Free compute; only pay for storage (~$0.06/GB/month) |

---

## 3.6 Comparison: CLI vs PowerShell for Common Tasks

| Task | Azure CLI | Azure PowerShell |
|------|-----------|-----------------|
| Create Resource Group | `az group create -n myRG -l eastus` | `New-AzResourceGroup -Name myRG -Location eastus` |
| List VMs | `az vm list -o table` | `Get-AzVM \| Format-Table` |
| Create VM | `az vm create -g myRG -n myVM --image Ubuntu2204` | `New-AzVM -ResourceGroupName myRG -Name myVM -Image Ubuntu2204` |
| Delete RG | `az group delete -n myRG --yes` | `Remove-AzResourceGroup -Name myRG -Force` |
| Get current sub | `az account show` | `Get-AzContext` |
| Login | `az login` | `Connect-AzAccount` |
| Set subscription | `az account set -s <id>` | `Set-AzContext -Subscription <id>` |

---

## 3.7 Real-World DevOps Scenarios

### Scenario 1: CI/CD Pipeline Authentication

```yaml
# Azure DevOps Pipeline - Azure CLI task
steps:
  - task: AzureCLI@2
    displayName: 'Deploy to Azure'
    inputs:
      azureSubscription: 'my-service-connection'
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        az webapp deploy \
          --resource-group myRG \
          --name myWebApp \
          --src-path ./app.zip
```

### Scenario 2: Infrastructure Automation Script

```bash
#!/bin/bash
# deploy-environment.sh - Create a complete dev environment

RESOURCE_GROUP="dev-rg"
LOCATION="eastus"
APP_NAME="myapp-dev"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create App Service Plan
az appservice plan create \
  --name "${APP_NAME}-plan" \
  --resource-group $RESOURCE_GROUP \
  --sku B1 --is-linux

# Create Web App
az webapp create \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --plan "${APP_NAME}-plan" \
  --runtime "NODE:18-lts"

# Create SQL Database
az sql server create \
  --name "${APP_NAME}-sql" \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --admin-user sqladmin \
  --admin-password "SecureP@ss123!"

az sql db create \
  --name "${APP_NAME}-db" \
  --resource-group $RESOURCE_GROUP \
  --server "${APP_NAME}-sql" \
  --service-objective S0

echo "Environment deployed successfully!"
```

---

## 3.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `az: command not found` | CLI not installed or not in PATH | Reinstall CLI; restart terminal |
| `The term 'Connect-AzAccount' is not recognized` | Az module not installed | `Install-Module -Name Az -Force` |
| Login expires in pipeline | Token timeout | Use service principal with `--service-principal` |
| `AuthorizationFailed` | Insufficient permissions | Check RBAC role assignments; ensure correct subscription |
| Cloud Shell won't start | Storage account issue | Recreate Cloud Shell storage in portal settings |
| Slow CLI performance | Large number of subscriptions | Set default sub: `az account set -s <id>` |

💡 **Best Practice:** Use `az --debug` or `$DebugPreference = "Continue"` to get detailed API call logs when troubleshooting.

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Portal** | Web GUI; best for exploration and dashboards |
| **Azure CLI** | Cross-platform; `az` commands; ideal for bash scripting and pipelines |
| **Azure PowerShell** | `Verb-AzNoun` pattern; rich objects; ideal for Windows automation |
| **Cloud Shell** | Browser-based; pre-configured; persistent 5GB storage |
| **Authentication** | Interactive, service principal, or managed identity |
| **All Tools** | Use the same ARM REST API underneath |
| **CI/CD Usage** | Service principals for automated pipelines |

---

## Quick Revision Questions

1. **What is the command syntax pattern for Azure CLI vs Azure PowerShell?**
   > Azure CLI: `az <group> <command> --parameters`. Azure PowerShell: `Verb-AzNoun -Parameters`.

2. **How do you authenticate in a CI/CD pipeline without user interaction?**
   > Use a service principal: `az login --service-principal -u <app-id> -p <secret> --tenant <tenant-id>` or `Connect-AzAccount -ServicePrincipal -Credential $cred -Tenant <tenant-id>`.

3. **What are the four output formats supported by Azure CLI?**
   > JSON (default), table, TSV, and YAML.

4. **What backs Azure Cloud Shell's persistent storage?**
   > An Azure Files share providing 5 GB of persistent storage mounted at `$HOME`.

5. **Which tool should you choose for a Linux-based CI/CD pipeline?**
   > Azure CLI — it's Python-based, cross-platform, and uses familiar bash scripting syntax.

6. **What is the underlying API that all Azure management tools communicate with?**
   > The Azure Resource Manager (ARM) REST API.

---

[⬅ Previous: Regions and Availability Zones](02-regions-and-availability-zones.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Resource Manager ➡](04-azure-resource-manager.md)
