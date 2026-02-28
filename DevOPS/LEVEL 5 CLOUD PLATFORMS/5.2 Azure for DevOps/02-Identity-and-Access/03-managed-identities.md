# Chapter 3: Managed Identities

## Overview
**Managed identities** eliminate the need to manage credentials for Azure service-to-service authentication. Azure automatically manages the identity lifecycle, making it the most secure and convenient way to authenticate between Azure services.

---

## 3.1 What are Managed Identities?

```
┌────────────────────────────────────────────────────────────────┐
│                    MANAGED IDENTITIES                           │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   WITHOUT Managed Identity:                                    │
│   ┌────────┐  credentials   ┌───────────┐                     │
│   │  VM /  │──stored in ───▶│ Key Vault │                     │
│   │  App   │  code/config   │ / SQL DB  │                     │
│   └────────┘  ⚠️ RISK!      └───────────┘                     │
│                                                                │
│   WITH Managed Identity:                                       │
│   ┌────────┐  auto token    ┌───────────┐                     │
│   │  VM /  │──from Azure───▶│ Key Vault │                     │
│   │  App   │  ✅ NO CREDS   │ / SQL DB  │                     │
│   └────────┘  in code!      └───────────┘                     │
│                                                                │
│   Azure handles:                                               │
│   • Creating the identity                                      │
│   • Issuing tokens (via IMDS endpoint)                        │
│   • Rotating credentials automatically                         │
│   • Deleting the identity when resource is deleted             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 3.2 Types of Managed Identities

```
┌──────────────── SYSTEM-ASSIGNED vs USER-ASSIGNED ────────────────┐
│                                                                   │
│   SYSTEM-ASSIGNED                    USER-ASSIGNED                │
│   ───────────────                    ─────────────                │
│                                                                   │
│   ┌──────────┐ 1:1 ┌──────────┐    ┌──────────┐ M:N ┌────────┐ │
│   │   VM A   │────▶│Identity A│    │   VM A   │────▶│Identity│ │
│   └──────────┘     └──────────┘    │   VM B   │────▶│   X    │ │
│                                     │   App C  │────▶│        │ │
│   ┌──────────┐ 1:1 ┌──────────┐    └──────────┘     └────────┘ │
│   │   VM B   │────▶│Identity B│                                  │
│   └──────────┘     └──────────┘    • Created as standalone       │
│                                     • Shared across resources     │
│   • Created WITH the resource       • Independent lifecycle       │
│   • Deleted WITH the resource       • Must be manually deleted    │
│   • Cannot be shared                • Reusable                    │
│   • Unique to the resource                                        │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

| Feature | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Created** | With the resource | Independently |
| **Lifecycle** | Tied to resource | Independent |
| **Sharing** | One-to-one only | Many-to-many |
| **Deletion** | Auto-deleted with resource | Must manually delete |
| **Use Case** | Single resource, simple | Multiple resources sharing identity |

---

## 3.3 How Managed Identities Work

```
┌──────────── MANAGED IDENTITY TOKEN FLOW ──────────────┐
│                                                         │
│  1. Azure Resource (VM, App Service, etc.)              │
│     requests token from IMDS endpoint                   │
│                                                         │
│  ┌──────────┐    GET /token     ┌───────────────┐      │
│  │  Your    │──────────────────▶│    IMDS        │      │
│  │  Code    │                   │  (Instance     │      │
│  │          │                   │   Metadata     │      │
│  │          │                   │   Service)     │      │
│  │          │    JWT Token      │  169.254.169.254│     │
│  │          │◀──────────────────│               │      │
│  └────┬─────┘                   └───────────────┘      │
│       │                                                 │
│       │  2. Use token to access Azure resource          │
│       │                                                 │
│  ┌────┴─────────┐     Authorization: Bearer <token>    │
│  │  Azure       │                                       │
│  │  Resource    │  (Key Vault, SQL, Storage, etc.)      │
│  └──────────────┘                                       │
│                                                         │
│  IMDS Endpoint: http://169.254.169.254/metadata/        │
│                 identity/oauth2/token                    │
│  This endpoint is ONLY accessible from within Azure     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3.4 Enabling Managed Identities

### System-Assigned Identity

```bash
# Enable system-assigned identity on a VM
az vm identity assign \
  --resource-group myRG \
  --name myVM

# Enable on App Service
az webapp identity assign \
  --resource-group myRG \
  --name myWebApp

# Enable on Azure Function
az functionapp identity assign \
  --resource-group myRG \
  --name myFunctionApp

# Get the principal ID
az vm identity show \
  --resource-group myRG \
  --name myVM \
  --query principalId --output tsv
```

### User-Assigned Identity

```bash
# Create user-assigned identity
az identity create \
  --resource-group myRG \
  --name myAppIdentity \
  --location eastus

# Get the identity's resource ID and client ID
az identity show \
  --resource-group myRG \
  --name myAppIdentity \
  --query "{ResourceID:id, ClientID:clientId, PrincipalID:principalId}" \
  --output table

# Assign to a VM
az vm identity assign \
  --resource-group myRG \
  --name myVM \
  --identities "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myAppIdentity"

# Assign to App Service
az webapp identity assign \
  --resource-group myRG \
  --name myWebApp \
  --identities "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myAppIdentity"
```

---

## 3.5 Granting Access to Azure Resources

After enabling managed identity, grant it RBAC roles on target resources.

```bash
# Grant VM's managed identity access to Key Vault
az role assignment create \
  --assignee <principal-id> \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myKeyVault"

# Grant access to Storage Account
az role assignment create \
  --assignee <principal-id> \
  --role "Storage Blob Data Reader" \
  --scope "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/myStorage"

# Grant access to SQL Database
az role assignment create \
  --assignee <principal-id> \
  --role "SQL DB Contributor" \
  --scope "/subscriptions/{sub}/resourceGroups/myRG/providers/Microsoft.Sql/servers/myServer"
```

---

## 3.6 Using Managed Identity in Code

### Python Example

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# DefaultAzureCredential automatically uses managed identity on Azure
credential = DefaultAzureCredential()

# Access Key Vault - no connection strings or passwords!
client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net",
    credential=credential
)

secret = client.get_secret("my-secret")
print(f"Secret value: {secret.value}")
```

### .NET Example

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Automatically uses managed identity on Azure
var credential = new DefaultAzureCredential();

var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    credential
);

KeyVaultSecret secret = client.GetSecret("my-secret");
Console.WriteLine($"Secret value: {secret.Value}");
```

### Bash (Direct Token Request)

```bash
# Request token from IMDS endpoint (run ON the Azure VM/App Service)
TOKEN=$(curl -s 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net' \
  -H 'Metadata: true' | jq -r '.access_token')

# Use token to access Key Vault
curl -s "https://mykeyvault.vault.azure.net/secrets/my-secret?api-version=7.4" \
  -H "Authorization: Bearer $TOKEN"
```

---

## 3.7 Supported Azure Services

| Service | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| Virtual Machines | ✅ | ✅ |
| VM Scale Sets | ✅ | ✅ |
| App Service / Functions | ✅ | ✅ |
| Azure Kubernetes Service | ✅ | ✅ |
| Container Instances | ❌ | ✅ |
| Logic Apps | ✅ | ✅ |
| Data Factory | ✅ | ✅ |
| Azure DevOps Pipelines | ❌ | ❌ (use service connections) |
| API Management | ✅ | ✅ |

---

## 3.8 Real-World DevOps Scenario

### App Service Accessing Key Vault and SQL

```
┌────────────── MANAGED IDENTITY ARCHITECTURE ──────────────┐
│                                                            │
│   ┌──────────────┐     RBAC: Key Vault     ┌───────────┐  │
│   │              │     Secrets User        │           │  │
│   │  App Service │─────────────────────────▶│ Key Vault │  │
│   │              │                          │           │  │
│   │  System MI:  │     RBAC: SQL DB        └───────────┘  │
│   │  Enabled ✅  │     Contributor                         │
│   │              │─────────────────────────▶┌───────────┐  │
│   │              │                          │  SQL DB   │  │
│   │              │     RBAC: Blob Data     │           │  │
│   │              │     Reader              └───────────┘  │
│   │              │─────────────────────────▶┌───────────┐  │
│   │              │                          │  Storage  │  │
│   └──────────────┘                          └───────────┘  │
│                                                            │
│   NO passwords, NO connection strings, NO secrets          │
│   stored in code or configuration!                         │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 3.9 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `ManagedIdentityCredential authentication unavailable` | MI not enabled on resource | Enable identity: `az vm identity assign` |
| `AADSTS700024` | Client assertion audience is wrong | Verify the resource URL in token request |
| `Forbidden (403)` | MI lacks RBAC role on target resource | Assign appropriate role: `az role assignment create` |
| Token request fails locally | IMDS endpoint not available outside Azure | Use `DefaultAzureCredential` (falls back to CLI/VS) |
| `DefaultAzureCredential` slow locally | Trying multiple auth methods in sequence | Set `AZURE_CLIENT_ID` env var or use specific credential |

💡 **Best Practice:** Use `DefaultAzureCredential` in code — it tries managed identity first (on Azure), then falls back to Azure CLI or Visual Studio credentials (for local development).

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Managed Identity** | Azure-managed credentials; no secrets in code |
| **System-Assigned** | 1:1 with resource; auto-deleted; simpler |
| **User-Assigned** | Independent lifecycle; shareable across resources |
| **IMDS** | Instance Metadata Service at 169.254.169.254 |
| **DefaultAzureCredential** | SDK class that auto-selects best auth method |
| **RBAC Required** | MI needs role assignments to access target resources |
| **No Rotation** | Azure handles all credential rotation automatically |

---

## Quick Revision Questions

1. **What is the main security advantage of managed identities over service principals?**
   > Managed identities eliminate the need to store and manage credentials — Azure handles all secret rotation automatically, reducing the risk of credential leaks.

2. **What are the two types of managed identities?**
   > System-assigned (tied 1:1 to a resource, auto-deleted) and user-assigned (independent lifecycle, shareable across resources).

3. **What is the IMDS endpoint and its IP address?**
   > The Instance Metadata Service at 169.254.169.254 — it provides tokens to Azure resources using managed identity authentication.

4. **What SDK class should you use for managed identity authentication?**
   > `DefaultAzureCredential` — it automatically uses managed identity on Azure and falls back to CLI/VS credentials for local development.

5. **What must you do after enabling a managed identity before it can access resources?**
   > Assign RBAC roles on the target resources (e.g., "Key Vault Secrets User" on a Key Vault).

6. **Can managed identities be used from outside Azure?**
   > No, managed identities only work on Azure resources where the IMDS endpoint is accessible.

---

[⬅ Previous: Service Principals](02-service-principals.md) | [⬆ Back to Table of Contents](../README.md) | [Next: RBAC Roles ➡](04-rbac-roles.md)
