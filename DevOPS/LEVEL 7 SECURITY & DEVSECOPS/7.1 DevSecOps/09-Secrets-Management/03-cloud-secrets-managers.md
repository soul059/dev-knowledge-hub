# Chapter 9.3: Cloud Secrets Managers

## Overview

Every major cloud provider offers a managed secrets management service. These services provide encryption, access control, rotation, and native integration with cloud resources — often simpler to adopt than self-hosted solutions like Vault. This chapter covers AWS Secrets Manager, Azure Key Vault, and GCP Secret Manager.

---

## Cloud Secrets Managers Comparison

```
CLOUD SECRETS MANAGERS AT A GLANCE:

  ┌──────────────────┬──────────────────┬──────────────────┐
  │   AWS SECRETS    │   AZURE KEY      │   GCP SECRET     │
  │   MANAGER        │   VAULT          │   MANAGER        │
  ├──────────────────┼──────────────────┼──────────────────┤
  │                  │                  │                  │
  │ Secrets +        │ Secrets + Keys + │ Secrets          │
  │ Rotation         │ Certificates     │ + Versioning     │
  │                  │                  │                  │
  │ $/secret/month   │ $/operation      │ $/secret/month   │
  │ + $/10K API call │ + $/key/month    │ + $/10K access   │
  │                  │                  │                  │
  │ Lambda rotation  │ Managed HSM      │ Cloud Functions  │
  │ RDS native       │ RBAC + policies  │ rotation         │
  │ integration      │ Key rotation     │ IAM integration  │
  │                  │                  │                  │
  │ AWS IAM          │ Azure AD RBAC    │ GCP IAM          │
  │ Resource policy  │ Access policies  │ Resource-level   │
  │ VPC endpoint     │ Private endpoint │ VPC SC support   │
  └──────────────────┴──────────────────┴──────────────────┘
```

---

## AWS Secrets Manager

### Store and Retrieve Secrets

```bash
# Create a secret
aws secretsmanager create-secret \
    --name prod/myapp/database \
    --description "Production database credentials" \
    --secret-string '{"username":"dbadmin","password":"s3cur3P@ss!"}'

# Retrieve secret
aws secretsmanager get-secret-value \
    --secret-id prod/myapp/database \
    --query SecretString --output text

# Update secret
aws secretsmanager update-secret \
    --secret-id prod/myapp/database \
    --secret-string '{"username":"dbadmin","password":"n3wP@ss!"}'

# List secrets
aws secretsmanager list-secrets \
    --filters Key=name,Values=prod/myapp
```

### Automatic Rotation (Lambda)

```python
# rotation_lambda.py — AWS Secrets Manager rotation function
import boto3
import json
import string
import random

def lambda_handler(event, context):
    """Rotate database password in AWS Secrets Manager."""
    secret_id = event['SecretId']
    step = event['Step']
    token = event['ClientRequestToken']

    client = boto3.client('secretsmanager')

    if step == "createSecret":
        # Generate new password
        new_password = ''.join(
            random.SystemRandom().choice(
                string.ascii_letters + string.digits + "!@#$%"
            ) for _ in range(32)
        )

        # Get current secret
        current = client.get_secret_value(SecretId=secret_id)
        secret_dict = json.loads(current['SecretString'])
        secret_dict['password'] = new_password

        # Store as pending
        client.put_secret_value(
            SecretId=secret_id,
            ClientRequestToken=token,
            SecretString=json.dumps(secret_dict),
            VersionStages=['AWSPENDING']
        )

    elif step == "setSecret":
        # Update password in the database
        pending = client.get_secret_value(
            SecretId=secret_id,
            VersionStage='AWSPENDING'
        )
        secret_dict = json.loads(pending['SecretString'])
        # Connect to DB and ALTER USER password here

    elif step == "testSecret":
        # Verify new credentials work
        pending = client.get_secret_value(
            SecretId=secret_id,
            VersionStage='AWSPENDING'
        )
        # Test DB connection with new creds

    elif step == "finishSecret":
        # Promote AWSPENDING to AWSCURRENT
        client.update_secret_version_stage(
            SecretId=secret_id,
            VersionStage='AWSCURRENT',
            MoveToVersionId=token,
            RemoveFromVersionId=get_current_version(client, secret_id)
        )
```

```bash
# Enable rotation
aws secretsmanager rotate-secret \
    --secret-id prod/myapp/database \
    --rotation-lambda-arn arn:aws:lambda:us-east-1:123456:function:rotate-db \
    --rotation-rules '{"AutomaticallyAfterDays": 30}'
```

### Terraform Integration

```hcl
# AWS Secrets Manager with Terraform
resource "aws_secretsmanager_secret" "db_creds" {
  name        = "prod/myapp/database"
  description = "Production database credentials"

  tags = {
    Environment = "production"
    Application = "myapp"
  }
}

resource "aws_secretsmanager_secret_version" "db_creds" {
  secret_id = aws_secretsmanager_secret.db_creds.id
  secret_string = jsonencode({
    username = "dbadmin"
    password = random_password.db.result
    host     = aws_db_instance.main.address
    port     = 3306
    dbname   = "myapp"
  })
}

# Reference in ECS task
resource "aws_ecs_task_definition" "myapp" {
  family = "myapp"

  container_definitions = jsonencode([{
    name  = "myapp"
    image = "myapp:latest"
    secrets = [
      {
        name      = "DB_USERNAME"
        valueFrom = "${aws_secretsmanager_secret.db_creds.arn}:username::"
      },
      {
        name      = "DB_PASSWORD"
        valueFrom = "${aws_secretsmanager_secret.db_creds.arn}:password::"
      }
    ]
  }])
}
```

---

## Azure Key Vault

### Setup and Secret Operations

```bash
# Create Key Vault
az keyvault create \
    --name myapp-vault \
    --resource-group myapp-rg \
    --location eastus \
    --sku standard \
    --enable-soft-delete true \
    --enable-purge-protection true

# Store secret
az keyvault secret set \
    --vault-name myapp-vault \
    --name "DatabasePassword" \
    --value "s3cur3P@ss!"

# Retrieve secret
az keyvault secret show \
    --vault-name myapp-vault \
    --name "DatabasePassword" \
    --query value -o tsv

# List secrets
az keyvault secret list --vault-name myapp-vault

# Set access policy
az keyvault set-policy \
    --name myapp-vault \
    --object-id <service-principal-id> \
    --secret-permissions get list
```

### Azure Key Vault with Managed Identity

```python
# Python — Access Azure Key Vault with Managed Identity
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://myapp-vault.vault.azure.net/",
    credential=credential
)

# Retrieve secret
secret = client.get_secret("DatabasePassword")
print(f"Secret value: {secret.value}")

# Set secret
client.set_secret("ApiKey", "new-api-key-value")

# List secrets
for secret_properties in client.list_properties_of_secrets():
    print(f"{secret_properties.name}: enabled={secret_properties.enabled}")
```

### Terraform Integration

```hcl
# Azure Key Vault with Terraform
resource "azurerm_key_vault" "main" {
  name                       = "myapp-vault"
  location                   = azurerm_resource_group.main.location
  resource_group_name        = azurerm_resource_group.main.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 90
  purge_protection_enabled   = true

  access_policy {
    tenant_id = data.azurerm_client_config.current.tenant_id
    object_id = data.azurerm_client_config.current.object_id

    secret_permissions = ["Get", "List", "Set", "Delete"]
    key_permissions    = ["Get", "List", "Create"]
  }
}

resource "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  value        = random_password.db.result
  key_vault_id = azurerm_key_vault.main.id
}
```

---

## GCP Secret Manager

### Setup and Operations

```bash
# Enable Secret Manager API
gcloud services enable secretmanager.googleapis.com

# Create a secret
echo -n "s3cur3P@ss!" | \
    gcloud secrets create db-password \
    --data-file=- \
    --replication-policy="automatic"

# Access secret (latest version)
gcloud secrets versions access latest --secret=db-password

# Add new version
echo -n "n3wP@ss!" | \
    gcloud secrets versions add db-password --data-file=-

# List secrets
gcloud secrets list

# Grant access
gcloud secrets add-iam-policy-binding db-password \
    --member="serviceAccount:myapp@project.iam.gserviceaccount.com" \
    --role="roles/secretmanager.secretAccessor"
```

### Python Integration

```python
# Python — Access GCP Secret Manager
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()

# Access secret
name = "projects/my-project/secrets/db-password/versions/latest"
response = client.access_secret_version(request={"name": name})
secret_value = response.payload.data.decode("UTF-8")

# Create a new secret
parent = "projects/my-project"
secret = client.create_secret(
    request={
        "parent": parent,
        "secret_id": "api-key",
        "secret": {"replication": {"automatic": {}}},
    }
)

# Add secret version
client.add_secret_version(
    request={
        "parent": secret.name,
        "payload": {"data": b"my-api-key-value"},
    }
)
```

---

## CI/CD Integration Patterns

```yaml
# GitHub Actions — Multi-cloud secret access
name: Deploy with Secrets
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write     # OIDC token
      contents: read

    steps:
      # --- AWS ---
      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456:role/github-actions
          aws-region: us-east-1

      - name: Get AWS Secret
        run: |
          DB_PASS=$(aws secretsmanager get-secret-value \
            --secret-id prod/myapp/database \
            --query SecretString --output text | jq -r .password)
          echo "::add-mask::$DB_PASS"

      # --- Azure ---
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Get Azure Secret
        run: |
          az keyvault secret show \
            --vault-name myapp-vault \
            --name DatabasePassword \
            --query value -o tsv

      # --- GCP ---
      - name: GCP Auth (Workload Identity)
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123/locations/global/...'
          service_account: 'github@project.iam.gserviceaccount.com'

      - name: Get GCP Secret
        run: |
          gcloud secrets versions access latest --secret=db-password
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Access denied to secret | IAM/RBAC policy missing | Grant `secretsmanager:GetSecretValue` (AWS), Secret `Get` permission (Azure), `secretAccessor` role (GCP) |
| Rotation Lambda fails | Lambda can't reach DB | Ensure Lambda in same VPC as DB; check security groups and NAT gateway |
| Azure Key Vault soft-deleted | Vault deleted but in retention | Use `az keyvault recover` to recover; or wait for purge retention period |
| Secret version not found | Referencing old/disabled version | Use `latest` or `AWSCURRENT` version stage; check version isn't disabled |
| Cross-account access denied | Missing resource policy | Add resource policy allowing cross-account access; use assume-role pattern |

---

## Summary Table

| Feature | AWS Secrets Manager | Azure Key Vault | GCP Secret Manager |
|---------|--------------------|-----------------|--------------------|
| **Secret Storage** | ✅ | ✅ | ✅ |
| **Key Management** | ❌ (use KMS) | ✅ (built-in) | ❌ (use Cloud KMS) |
| **Certificate Mgmt** | ❌ (use ACM) | ✅ (built-in) | ❌ (use Certificate Manager) |
| **Auto Rotation** | ✅ (Lambda) | ✅ (Event Grid) | ✅ (Cloud Functions) |
| **Versioning** | ✅ | ✅ | ✅ |
| **Audit Logging** | CloudTrail | Azure Monitor | Cloud Audit Logs |
| **Pricing Model** | Per secret + API calls | Per operation + key | Per secret + API calls |

---

## Quick Revision Questions

1. **How do cloud secrets managers differ from HashiCorp Vault?**
   > Cloud secrets managers are managed services — no infrastructure to run, automatic HA, native cloud IAM integration. Vault is self-hosted (or HCP Vault), offers dynamic secrets, Transit encryption engine, and multi-cloud support. Cloud managers are simpler for single-cloud; Vault is better for multi-cloud, dynamic secrets, and advanced use cases.

2. **How does AWS Secrets Manager rotation work?**
   > Rotation is powered by a Lambda function that executes four steps: createSecret (generate new password, store as AWSPENDING), setSecret (update the actual database password), testSecret (verify new credentials work), finishSecret (promote AWSPENDING to AWSCURRENT). Rotation is triggered on a schedule (e.g., every 30 days).

3. **What are the key security features of Azure Key Vault?**
   > Soft delete and purge protection prevent accidental deletion. Hardware Security Modules (HSMs) protect keys. Azure AD RBAC and access policies control access. Private endpoints restrict network access. Managed identities eliminate credential management for Azure services. Diagnostic logging provides audit trails.

4. **How should secrets be accessed in CI/CD pipelines?**
   > Use OIDC/Workload Identity Federation instead of long-lived service account keys. Authenticate to cloud providers using short-lived tokens, then retrieve secrets via API calls. Mask secrets in logs using `::add-mask::`. Never store cloud secrets as GitHub/CI secrets — fetch them dynamically from the secrets manager.

5. **What is the advantage of secret versioning?**
   > Versioning allows rollback to previous secret values if a rotation fails or breaks an application. It enables gradual migration — old version remains active while new version is tested. Version staging (AWSCURRENT, AWSPENDING, AWSPREVIOUS) ensures zero-downtime rotation.

---

[← Previous: 9.2 HashiCorp Vault](02-hashicorp-vault.md) | [Next: 9.4 Secret Rotation →](04-secret-rotation.md)

[Back to Table of Contents](../README.md)
