# Unit 5: Cloud IAM Security — Topic 5: Secrets Management

## Overview

**Secrets management** is the practice of securely storing, distributing, and managing sensitive credentials such as passwords, API keys, database connection strings, encryption keys, and certificates. In cloud environments, where applications, microservices, and automated pipelines all require credentials, a centralized secrets management strategy is essential for preventing credential exposure and unauthorized access.

---

## 1. Secrets Management Fundamentals

```
WHAT ARE SECRETS:
  → Database passwords
  → API keys and tokens
  → SSH keys
  → TLS certificates
  → Encryption keys
  → Connection strings
  → OAuth client secrets
  → Service account credentials
  → Webhook signing secrets

WHERE SECRETS SHOULD NOT BE:
  ✗ Hardcoded in source code
  ✗ In configuration files committed to Git
  ✗ In environment variables (visible in process list)
  ✗ In container images
  ✗ In CI/CD pipeline logs
  ✗ In shared documents/wikis
  ✗ In plaintext config management
  ✗ In email/chat messages

SECRETS LIFECYCLE:

  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Creation │──►│ Storage  │──►│ Access   │
  └──────────┘   └──────────┘   └──────────┘
       │              │              │
       ▼              ▼              ▼
  Generate       Encrypted      Auth Required
  Strong         At Rest        Least Privilege
  Random         Versioned      Audited
  Unique         Backed Up      Cached Safely
       │              │              │
       ▼              ▼              ▼
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │ Rotation │──►│ Audit    │──►│ Revocation│
  └──────────┘   └──────────┘   └──────────┘
  Automated      All Access     Immediate
  Scheduled      Logged         On Compromise
  Zero-downtime  Alerting       Propagated
```

---

## 2. Cloud-Native Secrets Managers

```
AWS SECRETS MANAGER:
  → Store and rotate secrets
  → Automatic rotation (Lambda-based)
  → RDS, Redshift, DocumentDB integration
  → Fine-grained IAM access control
  → Cross-account sharing
  → Versioning
  → $0.40/secret/month + $0.05/10K API calls

  # Create secret
  aws secretsmanager create-secret \
    --name prod/db/password \
    --secret-string '{"username":"admin","password":"P@ssw0rd!"}'

  # Retrieve secret
  aws secretsmanager get-secret-value \
    --secret-id prod/db/password

  # Enable rotation
  aws secretsmanager rotate-secret \
    --secret-id prod/db/password \
    --rotation-lambda-arn arn:aws:lambda:...:function:rotate \
    --rotation-rules AutomaticallyAfterDays=30

AWS SYSTEMS MANAGER PARAMETER STORE:
  → Free (Standard tier)
  → Hierarchical storage (/app/prod/db/password)
  → SecureString type (KMS encrypted)
  → Less features than Secrets Manager
  → Good for configuration + secrets

  # Store parameter
  aws ssm put-parameter \
    --name /prod/db/password \
    --type SecureString \
    --value "P@ssw0rd!"

AZURE KEY VAULT:
  → Secrets, keys, and certificates
  → HSM-backed (Premium tier)
  → Soft delete + purge protection
  → RBAC or access policies
  → Managed identity integration
  → Certificate lifecycle management

  # Create secret
  az keyvault secret set \
    --vault-name production-vault \
    --name db-password \
    --value "P@ssw0rd!"

  # Retrieve
  az keyvault secret show \
    --vault-name production-vault \
    --name db-password

GCP SECRET MANAGER:
  → Automatic replication (regional/global)
  → IAM-based access control
  → Version management
  → Audit logging
  → Rotation notifications
  → $0.06/10K operations

  # Create secret
  echo -n "P@ssw0rd!" | \
    gcloud secrets create db-password --data-file=-

  # Access secret
  gcloud secrets versions access latest \
    --secret=db-password
```

---

## 3. HashiCorp Vault

```
HASHICORP VAULT:

OVERVIEW:
  → Industry-standard secrets management
  → Multi-cloud and on-premises
  → Dynamic secrets (generated on demand)
  → Encryption as a service
  → Identity-based access
  → Open source (with enterprise version)

KEY FEATURES:

  DYNAMIC SECRETS:
  → Generated on demand for each request
  → Unique per consumer
  → Automatically revoked after TTL
  → Supported: databases, AWS, Azure, GCP, PKI
  
  # Enable database secrets engine
  vault secrets enable database
  
  # Configure database connection
  vault write database/config/mydb \
    plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(db:3306)/" \
    allowed_roles="readonly" \
    username="vault_admin" \
    password="admin_password"
  
  # Create role
  vault write database/roles/readonly \
    db_name=mydb \
    creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="24h"
  
  # Get dynamic credentials
  vault read database/creds/readonly
  # Returns unique username/password valid for 1 hour

  TRANSIT ENGINE (Encryption as a Service):
  → Encrypt/decrypt data without managing keys
  → Key versioning and rotation
  → Supported operations: encrypt, decrypt, sign, verify
  
  vault write transit/encrypt/my-key \
    plaintext=$(echo "sensitive data" | base64)

  AUTH METHODS:
  → Token
  → LDAP/Active Directory
  → OIDC/JWT
  → AWS IAM
  → Azure Managed Identity
  → GCP IAM
  → Kubernetes Service Account
  → TLS Certificates

  ARCHITECTURE:
  ┌─────────────────────────────────┐
  │         VAULT SERVER            │
  │                                 │
  │  ┌──────┐  ┌──────┐  ┌──────┐ │
  │  │Secret│  │Transit│  │Auth  │ │
  │  │Engine│  │Engine │  │Method│ │
  │  └──────┘  └──────┘  └──────┘ │
  │                                 │
  │  Storage Backend:               │
  │  Consul | Raft | DynamoDB | GCS │
  └─────────────────────────────────┘
```

---

## 4. Application Integration

```
INTEGRATING SECRETS IN APPLICATIONS:

SDK-BASED RETRIEVAL:
  # Python - AWS Secrets Manager
  import boto3
  
  client = boto3.client('secretsmanager')
  response = client.get_secret_value(SecretId='prod/db/password')
  secret = json.loads(response['SecretString'])
  db_password = secret['password']

  # Python - Azure Key Vault
  from azure.keyvault.secrets import SecretClient
  from azure.identity import DefaultAzureCredential
  
  client = SecretClient(
      vault_url="https://myvault.vault.azure.net",
      credential=DefaultAzureCredential()
  )
  secret = client.get_secret("db-password")
  db_password = secret.value

  # Python - GCP Secret Manager
  from google.cloud import secretmanager
  
  client = secretmanager.SecretManagerServiceClient()
  name = "projects/my-project/secrets/db-password/versions/latest"
  response = client.access_secret_version(request={"name": name})
  db_password = response.payload.data.decode("UTF-8")

KUBERNETES INTEGRATION:
  # External Secrets Operator
  apiVersion: external-secrets.io/v1beta1
  kind: ExternalSecret
  metadata:
    name: db-credentials
  spec:
    refreshInterval: 1h
    secretStoreRef:
      name: aws-secrets-manager
      kind: SecretStore
    target:
      name: db-credentials
    data:
    - secretKey: password
      remoteRef:
        key: prod/db/password
        property: password

  # CSI Secrets Store Driver
  apiVersion: secrets-store.csi.x-k8s.io/v1
  kind: SecretProviderClass
  metadata:
    name: azure-keyvault
  spec:
    provider: azure
    parameters:
      keyvaultName: "myvault"
      objects: |
        array:
          - objectName: db-password
            objectType: secret

CI/CD INTEGRATION:
  → GitHub Actions: secrets.MY_SECRET
  → GitLab CI: CI/CD variables (masked)
  → Jenkins: Credentials plugin
  → Azure DevOps: Variable groups + Key Vault
  → Use OIDC for cloud auth (no stored secrets)
```

---

## 5. Best Practices and Auditing

```
SECRETS MANAGEMENT BEST PRACTICES:

STORAGE:
  [ ] Centralized secrets manager (not scattered)
  [ ] Encryption at rest (KMS-backed)
  [ ] Access control (least privilege)
  [ ] Versioning enabled
  [ ] Backup and disaster recovery

ROTATION:
  [ ] Automated rotation schedule
  [ ] Zero-downtime rotation
  [ ] Rotation tested regularly
  [ ] Rotation alerts on failure
  [ ] Grace period for old versions

ACCESS:
  [ ] Authentication required for all access
  [ ] Least privilege (read-only where possible)
  [ ] No human access to production secrets
  [ ] Audit logging on all access
  [ ] Alert on unusual access patterns

DEVELOPMENT:
  [ ] Pre-commit hooks to prevent secret commits
  [ ] .gitignore for secret files (.env, *.key)
  [ ] Secret scanning in CI/CD pipeline
  [ ] Developer training on secure practices
  [ ] Use mock secrets in development

MONITORING:
  [ ] All secret access logged
  [ ] Alerts for unauthorized access attempts
  [ ] Secret rotation compliance dashboard
  [ ] Expiring secrets alerts
  [ ] Cross-reference with threat intelligence

AUDIT CHECKLIST:
  Question                              | Finding
  Where are secrets stored?             | Secrets Manager ✓ / Code ✗
  How are secrets accessed?             | SDK ✓ / Env vars ✗
  How often are secrets rotated?        | 30 days ✓ / Never ✗
  Who has access to production secrets? | App only ✓ / Everyone ✗
  Are secret access events logged?      | Yes ✓ / No ✗
  Is there a secret scanning process?   | CI/CD ✓ / None ✗
  What happens when a secret is leaked? | Playbook ✓ / Panic ✗
```

---

## Summary Table

| Service | Provider | Dynamic Secrets | Key Features |
|---------|----------|----------------|--------------|
| Secrets Manager | AWS | Via Lambda | Auto-rotation, RDS integration |
| Key Vault | Azure | No | HSM, certificates, RBAC |
| Secret Manager | GCP | No | Replication, versioning |
| Parameter Store | AWS | No | Free tier, hierarchy |
| HashiCorp Vault | Multi-cloud | Yes | Dynamic secrets, transit engine |

---

## Revision Questions

1. Why should secrets never be stored in source code or environment variables?
2. How does automatic secret rotation work in cloud-native services?
3. What advantages do dynamic secrets in HashiCorp Vault provide?
4. How can secrets be securely integrated into Kubernetes workloads?
5. What should a secrets management audit cover?

---

*Previous: [04-api-key-management.md](04-api-key-management.md) | Next: [06-privileged-access-management.md](06-privileged-access-management.md)*

---

*[Back to README](../README.md)*
