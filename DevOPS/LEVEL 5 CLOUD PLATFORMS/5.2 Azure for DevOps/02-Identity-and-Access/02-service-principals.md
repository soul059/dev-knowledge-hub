# Chapter 2: Service Principals

## Overview
A **service principal** is an identity used by applications, services, and automation tools to access Azure resources. For DevOps, service principals are essential for CI/CD pipelines, Terraform deployments, and any automated Azure interaction that needs its own identity separate from a human user.

---

## 2.1 What is a Service Principal?

```
┌────────────────────────────────────────────────────────────────┐
│              SERVICE PRINCIPAL CONCEPTS                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   ┌──────────────────┐        ┌───────────────────────┐        │
│   │  App Registration│        │   Service Principal   │        │
│   │  (Global def.)   │───────▶│  (Local instance)     │        │
│   │                  │1:Many  │                       │        │
│   │  • App ID        │        │  • Object ID          │        │
│   │  • Credentials   │        │  • Tenant-specific    │        │
│   │  • API perms     │        │  • RBAC assignments   │        │
│   └──────────────────┘        └───────────────────────┘        │
│                                                                │
│   Think of it as:                                              │
│   App Registration = "class definition"                        │
│   Service Principal = "instance of the class in a tenant"     │
│                                                                │
│   ┌──────────────────────────────────────────────┐             │
│   │              Three Types:                    │             │
│   │                                              │             │
│   │  1. Application  ─── Local representation    │             │
│   │                      of an app registration  │             │
│   │                                              │             │
│   │  2. Managed Identity ── Azure-managed,       │             │
│   │                         no credentials       │             │
│   │                                              │             │
│   │  3. Legacy ──────── Directly created SPs     │             │
│   │                     (az ad sp create)        │             │
│   └──────────────────────────────────────────────┘             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Authentication Methods

| Method | Security | Use Case | Rotation |
|--------|----------|----------|----------|
| **Client Secret** | Medium | Quick setup, dev/test | Manual (set expiry 6-24 months) |
| **Certificate** | High | Production workloads | Certificate renewal |
| **Federated Credential** | Highest | GitHub Actions, AKS | No secrets to rotate |

```
┌────────── AUTHENTICATION METHODS ──────────┐
│                                             │
│  CLIENT SECRET                              │
│  ──────────────                             │
│  App ──[App ID + Secret]──▶ Entra ID        │
│            ⚠️ Secret can leak               │
│                                             │
│  CERTIFICATE                                │
│  ───────────                                │
│  App ──[App ID + Cert]──▶ Entra ID          │
│            ✅ More secure than secrets       │
│                                             │
│  FEDERATED CREDENTIAL (OIDC)                │
│  ───────────────────────                    │
│  GitHub ──[OIDC Token]──▶ Entra ID          │
│            ✅ No secrets stored anywhere     │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 2.3 Creating Service Principals

### Using Azure CLI

```bash
# Create SP with auto-generated secret (Contributor on subscription)
az ad sp create-for-rbac \
  --name "sp-devops-pipeline" \
  --role Contributor \
  --scopes /subscriptions/{subscription-id}

# Output:
# {
#   "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",     ← Client ID
#   "displayName": "sp-devops-pipeline",
#   "password": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",       ← Client Secret
#   "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"     ← Tenant ID
# }

# Create SP with specific role on resource group
az ad sp create-for-rbac \
  --name "sp-webapp-deploy" \
  --role "Web Plan Contributor" \
  --scopes /subscriptions/{sub-id}/resourceGroups/webapp-rg

# Create SP with certificate-based auth
az ad sp create-for-rbac \
  --name "sp-prod-deploy" \
  --cert @/path/to/cert.pem \
  --role Contributor \
  --scopes /subscriptions/{sub-id}

# Create SP with new self-signed certificate
az ad sp create-for-rbac \
  --name "sp-prod-deploy" \
  --create-cert \
  --role Contributor \
  --scopes /subscriptions/{sub-id}
```

### Using Azure PowerShell

```powershell
# Create SP with auto-generated secret
$sp = New-AzADServicePrincipal -DisplayName "sp-devops-pipeline"

# Get the secret
$sp.PasswordCredentials.SecretText

# Create SP with certificate
$cert = New-SelfSignedCertificate -Subject "CN=sp-prod-deploy" `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -KeyExportPolicy Exportable -KeySpec Signature

New-AzADServicePrincipal -DisplayName "sp-prod-deploy" `
  -CertValue ($cert.GetRawCertData())
```

---

## 2.4 Using Service Principals in CI/CD

### Azure DevOps Service Connection

```
┌────────── AZURE DEVOPS SERVICE CONNECTION ──────────┐
│                                                       │
│  Azure DevOps Org                                     │
│  ┌──────────────────────────────────┐                 │
│  │  Project Settings                │                 │
│  │  └── Service Connections         │                 │
│  │      └── Azure Resource Manager  │                 │
│  │          ┌─────────────────────┐ │                 │
│  │          │ Subscription ID     │ │                 │
│  │          │ Service Principal ID│ │                 │
│  │          │ Client Secret/Cert  │ │                 │
│  │          │ Tenant ID           │ │                 │
│  │          └─────────────────────┘ │                 │
│  └───────────────┬──────────────────┘                 │
│                  │                                     │
│                  ▼                                     │
│  Pipeline uses service connection to                   │
│  authenticate with Azure ──▶ Deploy resources          │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### GitHub Actions with Federated Credentials (OIDC)

```bash
# Step 1: Create app registration
az ad app create --display-name "github-actions-deploy"

# Step 2: Create federated credential for GitHub
az ad app federated-credential create \
  --id <app-object-id> \
  --parameters '{
    "name": "github-main-branch",
    "issuer": "https://token.actions.githubusercontent.com",
    "subject": "repo:myorg/myrepo:ref:refs/heads/main",
    "audiences": ["api://AzureADTokenExchange"]
  }'

# Step 3: Create service principal
az ad sp create --id <app-id>

# Step 4: Assign RBAC role
az role assignment create \
  --assignee <app-id> \
  --role Contributor \
  --scope /subscriptions/{sub-id}
```

**GitHub Actions Workflow:**
```yaml
name: Deploy to Azure
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login (OIDC - no secrets!)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy
        run: |
          az webapp deploy --resource-group myRG --name myApp --src-path ./app.zip
```

---

## 2.5 Managing Service Principal Credentials

```bash
# List existing credentials for an SP
az ad sp credential list --id <app-id> --output table

# Reset/rotate client secret
az ad sp credential reset --id <app-id> --years 1

# Add a new client secret (keeping old one for rotation)
az ad app credential reset --id <app-id> --append --years 1

# Delete expired credential
az ad app credential delete --id <app-id> --key-id <key-id>

# Check expiration dates
az ad app credential list --id <app-id> \
  --query "[].{KeyID:keyId, EndDate:endDateTime}" --output table
```

### Secret Rotation Strategy

```
┌──────────────── SECRET ROTATION ────────────────┐
│                                                   │
│   Timeline:                                       │
│                                                   │
│   Day 0: Create Secret A (valid 12 months)        │
│   ├──────────────────────────────────────┤        │
│   │         Secret A in use              │        │
│   ├──────────────────────────────────────┤        │
│   │                                      │        │
│   Day 300: Create Secret B (valid 12 months)      │
│   │         ├──────────────────────────┤ │        │
│   │         │    Secret B ready        │ │        │
│   │         ├──────────────────────────┤ │        │
│   │                                      │        │
│   Day 330: Update all apps to use Secret B        │
│   │                                      │        │
│   Day 365: Secret A expires (safely)     │        │
│   │         Delete Secret A              │        │
│   ├──────────────────────────────────────┤        │
│                                                   │
│   💡 Always have two valid secrets during rotation │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 2.6 Least Privilege Best Practices

| Practice | Description |
|----------|-------------|
| **Narrow scope** | Assign roles at resource group level, not subscription |
| **Specific roles** | Use "Web Plan Contributor" instead of "Contributor" |
| **Short-lived secrets** | Set expiry to 6-12 months maximum |
| **Prefer certificates** | More secure than client secrets |
| **Prefer federated** | OIDC eliminates secrets entirely (GitHub, AKS) |
| **Use managed identities** | When running on Azure, avoid SPs altogether |
| **Audit regularly** | Review SP permissions and last sign-in dates |

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `AADSTS7000215` | Client secret expired | Rotate secret: `az ad sp credential reset` |
| `AADSTS700016` | App not found in tenant | Verify App ID and Tenant ID |
| `AuthorizationFailed` | SP lacks RBAC role | Assign appropriate role: `az role assignment create` |
| Pipeline auth fails | Wrong credentials in service connection | Update service connection with new credentials |
| Federated credential fails | Subject claim mismatch | Verify repo/branch in federated credential matches exactly |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Service Principal** | Identity for apps/automation; tenant-specific instance |
| **App Registration** | Global definition; one-to-many with service principals |
| **Client Secret** | Password-like credential; medium security; needs rotation |
| **Certificate** | Higher security; used for production |
| **Federated Credential** | OIDC-based; no secrets; recommended for GitHub Actions |
| **Scope** | Always use smallest scope needed (RG > Subscription) |
| **Rotation** | Overlap old and new secrets during rotation window |

---

## Quick Revision Questions

1. **What is the relationship between an App Registration and a Service Principal?**
   > An App Registration is a global definition, and a Service Principal is a local instance of that app in a specific tenant. One App Registration can have multiple Service Principals across tenants.

2. **What are the three authentication methods for service principals?**
   > Client secret, certificate, and federated credential (OIDC).

3. **Why are federated credentials preferred for GitHub Actions?**
   > Because they use OIDC token exchange and don't require storing any secrets, eliminating the risk of credential leaks.

4. **What CLI command creates a service principal with Contributor role?**
   > `az ad sp create-for-rbac --name "my-sp" --role Contributor --scopes /subscriptions/{sub-id}`

5. **How should you handle secret rotation for service principals?**
   > Create a new secret before the old one expires, update all applications to use the new secret, then delete the old secret after confirming everything works.

6. **What is the principle of least privilege for service principals?**
   > Assign the most specific role at the narrowest scope needed — e.g., "Web Plan Contributor" on a specific resource group instead of "Contributor" on the subscription.

---

[⬅ Previous: Azure Active Directory](01-azure-active-directory.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Managed Identities ➡](03-managed-identities.md)
