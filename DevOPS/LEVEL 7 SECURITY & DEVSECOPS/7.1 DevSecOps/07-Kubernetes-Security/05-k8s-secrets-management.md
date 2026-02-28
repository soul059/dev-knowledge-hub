# Chapter 7.5: Kubernetes Secrets Management

## Overview

Kubernetes Secrets store sensitive data like passwords, tokens, and keys. By default, Secrets are base64-encoded (NOT encrypted) and stored in etcd. This chapter covers the risks of default Secrets, encryption at rest, external secret stores, and best practices.

---

## The Problem with Default Secrets

```
DEFAULT KUBERNETES SECRETS — NOT SECURE:

  ┌──────────────────────────────────────────────────┐
  │ kubectl create secret generic db-creds \          │
  │   --from-literal=password=SuperSecret123          │
  │                                                    │
  │ Stored as base64 (NOT encryption!):                │
  │ password: U3VwZXJTZWNyZXQxMjM=                    │
  │                                                    │
  │ Decode: echo "U3VwZXJTZWNyZXQxMjM=" | base64 -d   │
  │ Result: SuperSecret123                             │
  └──────────────────────────────────────────────────┘

  RISKS:
  ┌────────────────────────────────────────────────────┐
  │ 1. Secrets in etcd are plaintext by default        │
  │ 2. Anyone with RBAC read access can decode them    │
  │ 3. Secrets in YAML files end up in Git repos       │
  │ 4. Mounted secrets are readable in the container   │
  │ 5. No automatic rotation mechanism                 │
  │ 6. No audit trail of who read which secret         │
  └────────────────────────────────────────────────────┘
```

---

## Encryption at Rest

```yaml
# /etc/kubernetes/encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      # Primary: encrypt with AES-CBC
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-32-byte-key>
      # Fallback: read unencrypted secrets (migration)
      - identity: {}
```

```bash
# Generate encryption key
head -c 32 /dev/urandom | base64

# Apply to API server
# Add flag: --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
# Restart API server

# Re-encrypt all existing secrets
kubectl get secrets -A -o json | kubectl replace -f -

# Verify encryption
ETCDCTL_API=3 etcdctl get /registry/secrets/default/my-secret \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/peer.crt \
    --key=/etc/kubernetes/pki/etcd/peer.key | hexdump -C
# Should show encrypted bytes, not plaintext
```

---

## External Secrets Operator

```
EXTERNAL SECRETS OPERATOR (ESO):

  External Secret Store            Kubernetes
  ┌──────────────────┐           ┌────────────────────┐
  │ AWS Secrets Mgr  │           │                    │
  │ Azure Key Vault  │◀──sync──▶│ ExternalSecret CR  │
  │ GCP Secret Mgr   │           │       │            │
  │ HashiCorp Vault   │           │       ▼            │
  │ 1Password         │           │ Kubernetes Secret  │
  └──────────────────┘           │ (auto-generated)   │
                                  └────────────────────┘
  
  Benefits:
  • Secrets never stored in Git
  • Centralized management
  • Automatic rotation sync
  • Audit trail in secret store
```

### Install and Configure ESO

```bash
# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
    -n external-secrets --create-namespace
```

### AWS Secrets Manager Integration

```yaml
# 1. SecretStore — connection to AWS
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
# 2. ExternalSecret — what to sync
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: production
spec:
  refreshInterval: 1h        # Sync every hour
  secretStoreRef:
    name: aws-secrets
    kind: SecretStore
  target:
    name: db-credentials      # K8s secret name created
    creationPolicy: Owner
  data:
    - secretKey: username     # Key in K8s secret
      remoteRef:
        key: prod/database    # AWS secret name
        property: username    # JSON key in AWS secret
    - secretKey: password
      remoteRef:
        key: prod/database
        property: password

---
# 3. Use in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  template:
    spec:
      containers:
        - name: api
          image: myapi:1.0.0
          envFrom:
            - secretRef:
                name: db-credentials   # Auto-synced from AWS
```

### Azure Key Vault Integration

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: azure-kv
  namespace: production
spec:
  provider:
    azurekv:
      vaultUrl: "https://my-vault.vault.azure.net"
      authType: WorkloadIdentity
      serviceAccountRef:
        name: external-secrets-sa

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: api-keys
  namespace: production
spec:
  refreshInterval: 30m
  secretStoreRef:
    name: azure-kv
    kind: SecretStore
  target:
    name: api-keys
  data:
    - secretKey: api-key
      remoteRef:
        key: production-api-key
```

---

## Sealed Secrets (GitOps-Compatible)

```
SEALED SECRETS WORKFLOW:

  Developer                     Git Repo              Cluster
  ┌──────┐    kubeseal     ┌──────────┐    GitOps   ┌──────────┐
  │ Plain │ ──────────────▶│ Encrypted│ ──────────▶ │ Decrypted│
  │Secret │ (public key)   │ Sealed   │ (Flux/Argo)│ K8s      │
  │       │                │ Secret   │             │ Secret   │
  └──────┘                 └──────────┘             └──────────┘

  Only the cluster controller has the private key
  to decrypt → safe to store in Git!
```

```bash
# Install Sealed Secrets controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets \
    -n kube-system

# Install kubeseal CLI
brew install kubeseal

# Create a regular secret
kubectl create secret generic db-password \
    --from-literal=password=MySecret123 \
    --dry-run=client -o yaml > secret.yaml

# Seal it (encrypt with cluster's public key)
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# The sealed secret is safe for Git
cat sealed-secret.yaml
```

```yaml
# sealed-secret.yaml — safe to commit to Git
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: db-password
  namespace: production
spec:
  encryptedData:
    password: AgBxs2Q3...  # Encrypted with cluster public key
  template:
    metadata:
      name: db-password
      namespace: production
```

---

## Secret Best Practices

```
KUBERNETES SECRETS BEST PRACTICES:

  ┌────────────────────────────────────────────────────────┐
  │                                                          │
  │  ✓  Enable encryption at rest for etcd                   │
  │  ✓  Use External Secrets Operator for production         │
  │  ✓  Use Sealed Secrets for GitOps workflows              │
  │  ✓  Restrict RBAC: limit who can read secrets            │
  │  ✓  Mount as files, not env vars (env shows in logs)     │
  │  ✓  Use short-lived tokens where possible                │
  │  ✓  Enable audit logging for secret access               │
  │  ✓  Rotate secrets regularly                              │
  │                                                          │
  │  ✗  Don't store secrets in plain YAML in Git             │
  │  ✗  Don't use the default SA token for secret access     │
  │  ✗  Don't grant wildcard access to secrets               │
  │  ✗  Don't log secrets or print them in pod output        │
  └────────────────────────────────────────────────────────┘
```

### Mount Secrets as Files (Preferred)

```yaml
# PREFERRED: Mount as file (not visible in `docker inspect` or process env)
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: myapp:1.0.0
      volumeMounts:
        - name: secrets
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secrets
      secret:
        secretName: db-credentials
        defaultMode: 0400    # Read-only by owner only

# NOT RECOMMENDED: Environment variables
# - Visible in kubectl describe pod
# - Visible in /proc/1/environ
# - May leak in crash dumps and logs
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ExternalSecret stuck "SecretSyncedError" | Wrong IAM permissions or vault path | Check SecretStore connection; verify IAM role has read access |
| Sealed Secret won't decrypt | Wrong namespace or cluster key rotated | Re-seal with current cluster key; check namespace scope |
| Secret visible in `kubectl describe pod` | Used `env` instead of volume mount | Mount secrets as files with `volumeMounts` |
| etcd secrets still plaintext | Encryption config not applied or old secrets | Apply encryption config; re-encrypt: `kubectl get secrets -A -o json \| kubectl replace -f -` |
| ESO not syncing updates | `refreshInterval` too long | Set shorter `refreshInterval` (e.g., `5m`) |

---

## Summary Table

| Approach | Use Case | Git-Safe? |
|----------|----------|-----------|
| **Default K8s Secrets** | Dev/test only | No |
| **Encryption at rest** | Minimum production requirement | N/A (etcd level) |
| **External Secrets Operator** | Production with cloud secret stores | Yes (no secrets in Git) |
| **Sealed Secrets** | GitOps workflows (Flux/ArgoCD) | Yes (encrypted in Git) |
| **CSI Secret Store Driver** | Direct mount from vault, no K8s Secret created | Yes |

---

## Quick Revision Questions

1. **Why are default Kubernetes Secrets not secure?**
   > They are base64-encoded, not encrypted. Anyone with RBAC read access to secrets can decode them. Secrets stored in YAML files often end up in Git repos. By default, etcd stores them in plaintext — an etcd backup exposes all secrets.

2. **What does encryption at rest protect against?**
   > It encrypts secrets in the etcd datastore. If an attacker gains access to etcd data files, backups, or the etcd API, they cannot read secret values without the encryption key. It does NOT protect against RBAC-authorized reads via the Kubernetes API.

3. **How does External Secrets Operator work?**
   > ESO syncs secrets from external stores (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault) into Kubernetes Secrets. You define an `ExternalSecret` resource that references a remote secret. ESO periodically syncs changes, keeping K8s secrets up to date with the source of truth.

4. **When would you use Sealed Secrets vs External Secrets Operator?**
   > Sealed Secrets is ideal for GitOps: secrets are encrypted and stored in Git, decrypted only by the cluster controller. ESO is better when you already have a centralized secret store (AWS SM, Vault) and want automatic rotation sync. They can complement each other.

5. **Why should secrets be mounted as files instead of environment variables?**
   > Environment variables are visible in `kubectl describe pod`, `/proc/1/environ`, crash dumps, and logging frameworks that auto-capture env. File mounts are not exposed through these channels and can have restrictive file permissions (e.g., 0400).

---

[← Previous: 7.4 Pod Security Standards](04-pod-security-standards.md) | [Next: 7.6 Security Scanning Tools →](06-security-scanning-tools.md)

[Back to Table of Contents](../README.md)
