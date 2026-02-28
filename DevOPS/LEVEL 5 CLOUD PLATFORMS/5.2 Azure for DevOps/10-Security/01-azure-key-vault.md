# Chapter 1: Azure Key Vault

## Overview
**Azure Key Vault** is a cloud service for securely storing and managing **secrets**, **keys**, and **certificates**. It provides centralised secret management, hardware security module (HSM) backing, access policies, RBAC integration, and audit logging — essential for DevOps pipelines and application security.

---

## 1.1 Key Vault Architecture

```
┌──────────────── AZURE KEY VAULT ─────────────────┐
│                                                    │
│  Key Vault: "myapp-kv"                             │
│  ┌──────────────────────────────────────────────┐  │
│  │                                              │  │
│  │  SECRETS                                     │  │
│  │  ├── db-connection-string                    │  │
│  │  ├── api-key-stripe                          │  │
│  │  ├── storage-account-key                     │  │
│  │  └── redis-password                          │  │
│  │                                              │  │
│  │  KEYS (Cryptographic)                        │  │
│  │  ├── data-encryption-key (RSA 2048)          │  │
│  │  ├── signing-key (EC P-256)                  │  │
│  │  └── wrap-key (RSA 4096)                     │  │
│  │                                              │  │
│  │  CERTIFICATES                                │  │
│  │  ├── myapp.com (TLS/SSL)                     │  │
│  │  ├── api.myapp.com (TLS/SSL)                 │  │
│  │  └── code-signing-cert                       │  │
│  │                                              │  │
│  └──────────────────────────────────────────────┘  │
│                                                    │
│  Access Control:                                   │
│  ├── Vault Access Policy (legacy)                  │
│  └── Azure RBAC (recommended)                      │
│                                                    │
│  Tiers:                                            │
│  ├── Standard: Software-protected keys             │
│  └── Premium: HSM-protected keys (FIPS 140-2 L2)  │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## 1.2 Key Vault Operations

```bash
# Create Key Vault
az keyvault create \
  --resource-group myRG \
  --name myapp-kv \
  --location uksouth \
  --enable-rbac-authorization true \
  --sku standard

# ── SECRETS ──

# Set a secret
az keyvault secret set \
  --vault-name myapp-kv \
  --name "db-connection-string" \
  --value "Server=mydb.database.windows.net;Database=myapp;User Id=admin;Password=S3cur3P@ss!"

# Get a secret
az keyvault secret show \
  --vault-name myapp-kv \
  --name "db-connection-string" \
  --query "value" -o tsv

# List secrets
az keyvault secret list \
  --vault-name myapp-kv \
  --query "[].{Name:name, Enabled:attributes.enabled}" -o table

# Delete a secret (soft-delete, recoverable)
az keyvault secret delete \
  --vault-name myapp-kv \
  --name "old-api-key"

# ── KEYS ──

# Create an RSA key
az keyvault key create \
  --vault-name myapp-kv \
  --name "data-encryption-key" \
  --kty RSA \
  --size 2048

# ── CERTIFICATES ──

# Create a self-signed certificate
az keyvault certificate create \
  --vault-name myapp-kv \
  --name "myapp-cert" \
  --policy "$(az keyvault certificate get-default-policy)"

# Import a PFX certificate
az keyvault certificate import \
  --vault-name myapp-kv \
  --name "imported-cert" \
  --file ./cert.pfx \
  --password "pfxPassword"
```

---

## 1.3 Access Control Models

```
┌────── ACCESS CONTROL ─────────────────────┐
│                                            │
│  VAULT ACCESS POLICY (Legacy):             │
│  ┌──────────────────────────────────────┐  │
│  │  Per-vault policies                 │  │
│  │  Permissions: Get, List, Set, Delete │  │
│  │  Assigned to: User, Group, App      │  │
│  │  Not inherited from subscription    │  │
│  └──────────────────────────────────────┘  │
│                                            │
│  AZURE RBAC (Recommended):                 │
│  ┌──────────────────────────────────────┐  │
│  │  Standard RBAC roles:              │  │
│  │  ├── Key Vault Administrator       │  │
│  │  │   (full control)                │  │
│  │  ├── Key Vault Secrets Officer     │  │
│  │  │   (manage secrets)              │  │
│  │  ├── Key Vault Secrets User        │  │
│  │  │   (read secrets only)           │  │
│  │  ├── Key Vault Crypto Officer      │  │
│  │  │   (manage keys)                 │  │
│  │  └── Key Vault Certificates Officer│  │
│  │      (manage certificates)         │  │
│  │                                    │  │
│  │  Inherited from resource group     │  │
│  │  Conditional Access supported      │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

```bash
# Grant RBAC role for secrets
az role assignment create \
  --assignee "user@company.com" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myapp-kv"

# Grant app's Managed Identity access
az role assignment create \
  --assignee "<managed-identity-object-id>" \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myapp-kv"
```

---

## 1.4 Using Key Vault in Applications

### .NET Application

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Key Vault as configuration source
builder.Configuration.AddAzureKeyVault(
    new Uri("https://myapp-kv.vault.azure.net/"),
    new DefaultAzureCredential()
);

var app = builder.Build();

// Access secret as configuration
var connectionString = app.Configuration["db-connection-string"];
```

### Azure Pipelines Integration

```yaml
# Use Key Vault secrets in pipeline
steps:
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: 'my-azure-connection'
      KeyVaultName: 'myapp-kv'
      SecretsFilter: 'db-connection-string,api-key-stripe'
      RunAsPreJob: true

  - script: |
      echo "Using secret in deployment"
      # Secrets available as pipeline variables
      # $(db-connection-string) — masked in logs
    displayName: 'Deploy with secrets'
```

### App Service Key Vault Reference

```bash
# Reference Key Vault secret in App Service settings
az webapp config appsettings set \
  --resource-group myRG \
  --name myapp \
  --settings "ConnectionString=@Microsoft.KeyVault(SecretUri=https://myapp-kv.vault.azure.net/secrets/db-connection-string/)"
```

---

## 1.5 Key Vault Features

| Feature | Description |
|---------|-------------|
| **Soft Delete** | Deleted items recoverable for 7-90 days (enabled by default) |
| **Purge Protection** | Prevents permanent deletion during retention period |
| **Private Endpoint** | Access Key Vault over private network |
| **Firewall** | Restrict access to specific VNets/IPs |
| **Logging** | Audit all access via Azure Monitor diagnostic settings |
| **Auto-Rotation** | Automatically rotate secrets/certificates |
| **Versioning** | Each secret/key/cert has version history |

---

## 1.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 403 Forbidden | No access policy or RBAC role | Grant appropriate role or access policy |
| Secret not found | Wrong name or deleted | Check name spelling; recover from soft delete |
| App can't access KV | Managed Identity not enabled | Enable System/User Managed Identity |
| Network blocked | Key Vault firewall blocking | Add VNet/IP to firewall or use Private Endpoint |
| Certificate expired | Auto-renewal not configured | Enable auto-renewal in certificate policy |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Key Vault** | Centralised store for secrets, keys, certificates |
| **Secrets** | Connection strings, API keys, passwords |
| **Keys** | Cryptographic keys (RSA, EC) for encrypt/sign |
| **Certificates** | TLS/SSL certs with auto-renewal |
| **Access Control** | Azure RBAC (recommended) or Vault Access Policies |
| **DevOps** | `AzureKeyVault@2` task pulls secrets into pipelines |

---

## Quick Revision Questions

1. **What are the three types of objects in Key Vault?**
   > Secrets (passwords, connection strings), Keys (cryptographic keys for encryption/signing), and Certificates (TLS/SSL certificates).

2. **What is the difference between RBAC and Vault Access Policies?**
   > RBAC uses standard Azure role assignments (inherited, conditional access). Access Policies are per-vault permissions not integrated with Azure RBAC hierarchy.

3. **How does an App Service securely access Key Vault?**
   > Enable Managed Identity on the App Service, grant it "Key Vault Secrets User" role, then reference secrets via Key Vault References in app settings.

4. **What is soft delete?**
   > When a secret/key/cert is deleted, it's retained in a recoverable state for 7-90 days before permanent deletion.

5. **How do you use Key Vault secrets in Azure Pipelines?**
   > Use the `AzureKeyVault@2` task to download secrets from Key Vault into pipeline variables (automatically masked in logs).

---

[⬅ Previous: Diagnostics](../09-Monitoring-and-Logging/06-diagnostics.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Microsoft Defender for Cloud ➡](02-security-center.md)
