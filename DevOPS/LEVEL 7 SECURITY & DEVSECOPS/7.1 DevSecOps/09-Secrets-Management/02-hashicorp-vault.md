# Chapter 9.2: HashiCorp Vault

## Overview

HashiCorp Vault is the industry-leading open-source secrets management platform. It provides secure storage, dynamic secrets, encryption as a service, and identity-based access. This chapter covers Vault architecture, setup, operations, and integration patterns for DevSecOps pipelines.

---

## Vault Architecture

```
HASHICORP VAULT ARCHITECTURE:

  ┌─────────────────────────────────────────────────────┐
  │                    VAULT SERVER                       │
  │                                                       │
  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
  │  │  Auth    │  │  Secrets │  │  Audit Devices    │  │
  │  │  Methods │  │  Engines │  │                    │  │
  │  │          │  │          │  │  ┌──────────────┐ │  │
  │  │ • Token  │  │ • KV v2  │  │  │ File log     │ │  │
  │  │ • LDAP   │  │ • AWS    │  │  │ Syslog       │ │  │
  │  │ • OIDC   │  │ • DB     │  │  │ Socket       │ │  │
  │  │ • K8s    │  │ • PKI    │  │  └──────────────┘ │  │
  │  │ • AppRole│  │ • Transit│  │                    │  │
  │  │ • AWS IAM│  │ • SSH    │  │                    │  │
  │  └──────────┘  └──────────┘  └──────────────────┘  │
  │                                                       │
  │  ┌────────────────────────────────────────────────┐  │
  │  │              BARRIER (Encryption)               │  │
  │  │           AES-256-GCM encryption               │  │
  │  └────────────────────────────────────────────────┘  │
  │                                                       │
  │  ┌────────────────────────────────────────────────┐  │
  │  │            STORAGE BACKEND                      │  │
  │  │   Consul │ Raft │ S3 │ DynamoDB │ PostgreSQL   │  │
  │  └────────────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────────┘
         │              │               │
         ▼              ▼               ▼
    ┌─────────┐   ┌─────────┐    ┌──────────┐
    │ CI/CD   │   │ K8s     │    │ Apps     │
    │ Pipeline│   │ Pods    │    │ Services │
    └─────────┘   └─────────┘    └──────────┘
```

---

## Vault Setup (Development Mode)

```bash
# Install Vault
# macOS
brew install vault

# Linux
wget https://releases.hashicorp.com/vault/1.15.4/vault_1.15.4_linux_amd64.zip
unzip vault_1.15.4_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Start dev server (NOT for production!)
vault server -dev
# Root Token: hvs.abc123xyz...
# Unseal Key: abcdef123456...

# Set environment
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='hvs.abc123xyz...'

# Verify
vault status
```

---

## Production Configuration

```hcl
# vault-config.hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
}

listener "tcp" {
  address       = "0.0.0.0:8200"
  tls_cert_file = "/opt/vault/tls/vault-cert.pem"
  tls_key_file  = "/opt/vault/tls/vault-key.pem"
}

api_addr     = "https://vault.example.com:8200"
cluster_addr = "https://vault-1.internal:8201"

ui = true

# Telemetry for monitoring
telemetry {
  prometheus_retention_time = "30s"
  disable_hostname         = true
}

# Audit logging
audit {
  type = "file"
  options {
    file_path = "/var/log/vault/audit.log"
  }
}
```

```bash
# Initialize Vault (production)
vault operator init -key-shares=5 -key-threshold=3

# Output:
# Unseal Key 1: abc123...
# Unseal Key 2: def456...
# Unseal Key 3: ghi789...
# Unseal Key 4: jkl012...
# Unseal Key 5: mno345...
# Initial Root Token: hvs.roottoken123

# Unseal (requires 3 of 5 keys)
vault operator unseal <key-1>
vault operator unseal <key-2>
vault operator unseal <key-3>
```

---

## KV Secrets Engine (v2)

```bash
# Enable KV v2 secrets engine
vault secrets enable -path=secret kv-v2

# Write a secret
vault kv put secret/myapp/database \
    username="dbadmin" \
    password="s3cur3P@ssw0rd"

# Read a secret
vault kv get secret/myapp/database
vault kv get -field=password secret/myapp/database

# List secrets
vault kv list secret/myapp/

# Update (creates new version)
vault kv put secret/myapp/database \
    username="dbadmin" \
    password="n3wP@ssw0rd!"

# Read specific version
vault kv get -version=1 secret/myapp/database

# Delete (soft delete)
vault kv delete secret/myapp/database

# Undelete
vault kv undelete -versions=2 secret/myapp/database

# Permanently destroy
vault kv destroy -versions=1,2 secret/myapp/database
```

---

## Dynamic Secrets

```
DYNAMIC SECRETS FLOW:

  Application ──── Request ────▶ Vault
                                   │
                            ┌──────┴──────┐
                            │ Generate    │
                            │ unique creds│
                            │ on-the-fly  │
                            └──────┬──────┘
                                   │
                            ┌──────┴──────┐
                            │  Create     │
                            │  user in DB │
                            │  with TTL   │
                            └──────┬──────┘
                                   │
  Application ◀───── Lease ────────┘
  (temp creds)        (TTL: 1h)

  After TTL expires:
  Vault automatically revokes the credentials
  and drops the database user.

  Benefits:
  • No shared/static passwords
  • Each app instance gets unique creds
  • Automatic cleanup on expiry
  • Full audit trail per credential
```

### Dynamic Database Secrets

```bash
# Enable database secrets engine
vault secrets enable database

# Configure database connection
vault write database/config/mydb \
    plugin_name=mysql-database-plugin \
    connection_url="{{username}}:{{password}}@tcp(db.example.com:3306)/" \
    allowed_roles="readonly,readwrite" \
    username="vault-admin" \
    password="vault-admin-password"

# Create a role (template for dynamic users)
vault write database/roles/readonly \
    db_name=mydb \
    creation_statements="CREATE USER '{{name}}'@'%' \
      IDENTIFIED BY '{{password}}'; \
      GRANT SELECT ON *.* TO '{{name}}'@'%';" \
    default_ttl="1h" \
    max_ttl="24h"

# Generate dynamic credentials
vault read database/creds/readonly
# Key             Value
# lease_id        database/creds/readonly/abc123
# lease_duration  1h
# username        v-token-readonly-abc123
# password        A1B2C3D4E5-random-generated

# Revoke when done
vault lease revoke database/creds/readonly/abc123
```

### Dynamic AWS Credentials

```bash
# Enable AWS secrets engine
vault secrets enable aws

# Configure root credentials
vault write aws/config/root \
    access_key=AKIAIOSFODNN7EXAMPLE \
    secret_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY \
    region=us-east-1

# Create a role for dynamic IAM users
vault write aws/roles/s3-readonly \
    credential_type=iam_user \
    policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
EOF

# Generate temporary AWS credentials
vault read aws/creds/s3-readonly
# access_key     AKIAEXAMPLE123
# secret_key     wJalr...temporary
# lease_duration 1h
```

---

## Auth Methods

```bash
# --- AppRole (for applications/CI) ---
vault auth enable approle

vault write auth/approle/role/myapp \
    token_ttl=1h \
    token_max_ttl=4h \
    secret_id_ttl=10m \
    token_policies="myapp-policy"

# Get Role ID (public)
vault read auth/approle/role/myapp/role-id

# Generate Secret ID (private, short-lived)
vault write -f auth/approle/role/myapp/secret-id

# Login with AppRole
vault write auth/approle/login \
    role_id="abc-123" \
    secret_id="def-456"

# --- Kubernetes Auth ---
vault auth enable kubernetes

vault write auth/kubernetes/config \
    kubernetes_host="https://kubernetes.default.svc:443" \
    token_reviewer_jwt="@/var/run/secrets/kubernetes.io/serviceaccount/token" \
    kubernetes_ca_cert="@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"

vault write auth/kubernetes/role/myapp \
    bound_service_account_names=myapp-sa \
    bound_service_account_namespaces=production \
    policies=myapp-policy \
    ttl=1h
```

---

## Vault Policies (ACL)

```hcl
# myapp-policy.hcl
# Read-only access to app secrets
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}

# Dynamic database credentials
path "database/creds/readonly" {
  capabilities = ["read"]
}

# No access to other paths (implicit deny)

# Admin policy
path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "sys/policies/acl/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "auth/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

```bash
# Apply policy
vault policy write myapp-policy myapp-policy.hcl

# List policies
vault policy list

# Read policy
vault policy read myapp-policy
```

---

## Transit Secrets Engine (Encryption as a Service)

```bash
# Enable transit engine
vault secrets enable transit

# Create encryption key
vault write -f transit/keys/myapp

# Encrypt data (base64 input)
vault write transit/encrypt/myapp \
    plaintext=$(echo "sensitive data" | base64)
# ciphertext: vault:v1:abc123encrypted...

# Decrypt data
vault write transit/decrypt/myapp \
    ciphertext="vault:v1:abc123encrypted..."
# plaintext: c2Vuc2l0aXZlIGRhdGE=  (base64)

# Rotate encryption key
vault write -f transit/keys/myapp/rotate

# Re-encrypt with new key version
vault write transit/rewrap/myapp \
    ciphertext="vault:v1:abc123encrypted..."
# ciphertext: vault:v2:xyz789reencrypted...
```

---

## Kubernetes Integration

```yaml
# vault-agent-injector deployment
# Install via Helm
# helm install vault hashicorp/vault \
#   --set "injector.enabled=true" \
#   --set "server.enabled=false"

# Pod with Vault Agent Sidecar injection
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "myapp"
        vault.hashicorp.com/agent-inject-secret-db-creds: "secret/data/myapp/database"
        vault.hashicorp.com/agent-inject-template-db-creds: |
          {{- with secret "secret/data/myapp/database" -}}
          export DB_USER="{{ .Data.data.username }}"
          export DB_PASS="{{ .Data.data.password }}"
          {{- end }}
    spec:
      serviceAccountName: myapp-sa
      containers:
        - name: myapp
          image: myapp:1.0
          command: ["/bin/sh", "-c", "source /vault/secrets/db-creds && ./start.sh"]
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Vault sealed after restart | Expected behavior — Vault seals on restart | Provide unseal keys (3 of 5) or use auto-unseal via cloud KMS |
| Permission denied reading secrets | Policy doesn't grant access to path | Check policy with `vault policy read`; ensure path matches exactly (include `data/` for KV v2) |
| Dynamic DB creds fail | Vault can't connect to database | Check database config; ensure Vault IP allowed in DB firewall rules |
| Kubernetes auth fails | ServiceAccount mismatch | Verify `bound_service_account_names` and namespace match exactly |
| AppRole login fails | Secret ID expired | Generate a new Secret ID; check `secret_id_ttl` setting |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| **KV Engine** | Static key-value secret storage with versioning |
| **Dynamic Secrets** | On-demand, short-lived credentials (DB, AWS, Azure) |
| **Transit Engine** | Encryption/decryption as a service without storing data |
| **Auth Methods** | AppRole, Kubernetes, LDAP, OIDC, AWS IAM, GitHub |
| **Policies** | Path-based ACL with fine-grained capabilities |
| **Audit** | Complete log of every secret access |
| **Auto-Unseal** | Cloud KMS integration for automated unsealing |
| **HA Mode** | Raft/Consul storage for high availability |

---

## Quick Revision Questions

1. **What is HashiCorp Vault and what problem does it solve?**
   > Vault is an open-source secrets management platform that provides centralized, encrypted storage for secrets with fine-grained access control, audit logging, dynamic secret generation, and encryption as a service. It solves the problems of secrets sprawl, hardcoded credentials, manual rotation, and lack of audit trails.

2. **What are dynamic secrets and why are they important?**
   > Dynamic secrets are credentials generated on-demand by Vault for each request, with an automatic TTL (time-to-live). When the TTL expires, Vault revokes the credentials (e.g., drops the database user). This eliminates shared/static passwords, ensures each application instance has unique credentials, and reduces the blast radius of compromise.

3. **Explain the Vault seal/unseal mechanism.**
   > Vault encrypts all data with a master key. This master key is split using Shamir's Secret Sharing into multiple unseal keys (e.g., 5 keys, threshold of 3). On startup, Vault is sealed — no data can be accessed. To unseal, operators must provide the threshold number of unseal keys to reconstruct the master key. Auto-unseal via cloud KMS automates this process.

4. **How does AppRole authentication work?**
   > AppRole uses two credentials: a Role ID (like a username, relatively static) and a Secret ID (like a password, short-lived). Applications present both to Vault to authenticate and receive a Vault token. Secret IDs can be configured with TTLs, IP restrictions, and usage limits, making it suitable for CI/CD and automated systems.

5. **What is the Transit secrets engine?**
   > Transit provides encryption as a service — applications send data to Vault for encryption/decryption without Vault storing the data. This centralizes cryptographic operations, supports key rotation without re-encrypting data, and ensures consistent encryption across applications. The application never handles raw encryption keys.

---

[← Previous: 9.1 Secrets Challenges](01-secrets-challenges.md) | [Next: 9.3 Cloud Secrets Managers →](03-cloud-secrets-managers.md)

[Back to Table of Contents](../README.md)
