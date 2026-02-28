# Chapter 5.4: Secrets Management

[← Previous: Environment Promotion](03-environment-promotion.md) | [Back to README](../README.md) | [Next → Drift Detection](05-drift-detection.md)

---

## Overview

Secrets management is one of the biggest challenges in GitOps: you want **everything in Git**, but you can't store plaintext secrets. This chapter covers the major approaches — Sealed Secrets, SOPS, External Secrets, and Vault — with their trade-offs.

---

## The GitOps Secrets Problem

```
┌──────────────────────────────────────────────────────────┐
│              THE SECRETS DILEMMA                         │
│                                                          │
│  GitOps principle:                                       │
│    "Git is the single source of truth"                   │
│                                                          │
│  Security principle:                                     │
│    "Never commit secrets to Git"                         │
│                                                          │
│  Solutions:                                              │
│  ┌────────────────────────────────────┐                  │
│  │ 1. Encrypt secrets IN Git          │                  │
│  │    (Sealed Secrets, SOPS)          │                  │
│  │                                    │                  │
│  │ 2. Reference secrets FROM Git      │                  │
│  │    (External Secrets Operator)     │                  │
│  │                                    │                  │
│  │ 3. Inject secrets AT RUNTIME       │                  │
│  │    (Vault Agent, CSI Driver)       │                  │
│  └────────────────────────────────────┘                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Approach 1: Sealed Secrets

Encrypts secrets **client-side** so only the cluster controller can decrypt them.

```
┌──────────────────────────────────────────────────────────┐
│           SEALED SECRETS FLOW                            │
│                                                          │
│  ┌──────────┐    kubeseal     ┌──────────────┐          │
│  │ Secret   │───────────────→ │ SealedSecret │          │
│  │ (plain)  │  (encrypts     │ (encrypted)  │          │
│  └──────────┘   with public  └──────┬───────┘          │
│                 key)                 │                    │
│                               ┌─────▼────────┐          │
│                               │   Git Repo   │          │
│                               │  (safe to    │          │
│                               │   commit)    │          │
│                               └─────┬────────┘          │
│                                     │                    │
│                               ┌─────▼────────┐          │
│                               │ Sealed       │          │
│                               │ Secrets      │          │
│                               │ Controller   │          │
│                               │ (decrypts    │          │
│                               │  with priv   │          │
│                               │  key)        │          │
│                               └─────┬────────┘          │
│                                     │                    │
│                               ┌─────▼────────┐          │
│                               │ K8s Secret   │          │
│                               │ (created in  │          │
│                               │  cluster)    │          │
│                               └──────────────┘          │
└──────────────────────────────────────────────────────────┘
```

### Usage

```bash
# Install Sealed Secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.25.0/controller.yaml

# Install kubeseal CLI
brew install kubeseal

# Create a regular secret
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=s3cret \
  --dry-run=client -o yaml > secret.yaml

# Seal it (encrypts with controller's public key)
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Commit the sealed secret to Git
git add sealed-secret.yaml
git commit -m "add encrypted db credentials"
git push
```

### SealedSecret YAML

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: db-creds
  namespace: my-app
spec:
  encryptedData:
    username: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEq...
    password: AgCtr2OJSWK+WEtySYMMA8sPi43cGFRt...
  template:
    metadata:
      name: db-creds
      namespace: my-app
    type: Opaque
```

---

## Approach 2: SOPS (Secrets OPerationS)

Mozilla SOPS encrypts values **in-place** within YAML/JSON files using KMS, PGP, or age keys.

```bash
# Install SOPS
brew install sops

# Create .sops.yaml configuration
cat > .sops.yaml << EOF
creation_rules:
  - path_regex: .*\.enc\.yaml$
    encrypted_regex: ^(data|stringData)$
    age: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
EOF

# Encrypt a secret file
sops --encrypt secret.yaml > secret.enc.yaml

# Commit encrypted file
git add secret.enc.yaml .sops.yaml
git commit -m "add encrypted secret"
```

### Encrypted SOPS File

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
  namespace: my-app
type: Opaque
stringData:
  username: ENC[AES256_GCM,data:aGVsbG8=,iv:...,tag:...,type:str]
  password: ENC[AES256_GCM,data:c2VjcmV0,iv:...,tag:...,type:str]
sops:
  age:
  - recipient: age1ql3z7hjy54pw3hyw...
    enc: |
      -----BEGIN AGE ENCRYPTED FILE-----
      ...
      -----END AGE ENCRYPTED FILE-----
  lastmodified: "2024-01-15T10:30:00Z"
  version: 3.8.1
```

### Flux SOPS Integration

```yaml
# Flux Kustomization with SOPS decryption
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-app
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: my-app
  path: ./deploy
  prune: true
  decryption:
    provider: sops               # Enable SOPS decryption
    secretRef:
      name: sops-age             # Secret containing age private key
```

```bash
# Create the age key secret for Flux
cat age.agekey |
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=/dev/stdin
```

---

## Approach 3: External Secrets Operator (ESO)

References secrets from **external providers** (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager, Azure Key Vault).

```
┌──────────────────────────────────────────────────────────┐
│      EXTERNAL SECRETS OPERATOR FLOW                      │
│                                                          │
│  ┌─────────────────┐     ┌─────────────────┐            │
│  │ ExternalSecret  │────→│ External Secrets │            │
│  │ (in Git)        │     │ Operator         │            │
│  │                 │     └────────┬────────┘            │
│  │ References:     │              │                      │
│  │  store: vault   │     ┌────────▼────────┐            │
│  │  key: db/pass   │     │ AWS Secrets Mgr │            │
│  └─────────────────┘     │ / Vault / GCP   │            │
│                          │ / Azure KV      │            │
│                          └────────┬────────┘            │
│                                   │                      │
│                          ┌────────▼────────┐            │
│                          │ K8s Secret      │            │
│                          │ (auto-created)  │            │
│                          └─────────────────┘            │
└──────────────────────────────────────────────────────────┘
```

### SecretStore Configuration

```yaml
# Connect to AWS Secrets Manager
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: my-app
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
```

### ExternalSecret Resource

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-creds
  namespace: my-app
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets
    kind: SecretStore
  target:
    name: db-creds                   # K8s Secret to create
    creationPolicy: Owner
  data:
  - secretKey: username              # Key in K8s Secret
    remoteRef:
      key: prod/db-credentials       # Path in Secrets Manager
      property: username             # JSON key
  - secretKey: password
    remoteRef:
      key: prod/db-credentials
      property: password
```

---

## Approach 4: HashiCorp Vault with Sidecar

```yaml
# Pod with Vault Agent sidecar injection
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "my-app"
        vault.hashicorp.com/agent-inject-secret-db: "secret/data/db-creds"
        vault.hashicorp.com/agent-inject-template-db: |
          {{- with secret "secret/data/db-creds" -}}
          DB_USERNAME={{ .Data.data.username }}
          DB_PASSWORD={{ .Data.data.password }}
          {{- end }}
    spec:
      serviceAccountName: my-app
      containers:
      - name: my-app
        image: myorg/my-app:v1.0.0
        # Secret available at /vault/secrets/db
```

---

## Comparison of Approaches

| Feature | Sealed Secrets | SOPS | External Secrets | Vault Sidecar |
|---------|---------------|------|-----------------|---------------|
| Secrets in Git | Encrypted | Encrypted | Reference only | Reference only |
| External dependency | Controller only | KMS/age key | Secret provider | Vault server |
| Rotation | Manual re-seal | Manual re-encrypt | Auto-refresh | Auto-renew |
| Multi-cloud | No | Yes (KMS) | Yes (any provider) | Vault only |
| Complexity | Low | Medium | Medium | High |
| Flux integration | Apply SealedSecret | Native `decryption` | Apply ExternalSecret | Vault injector |
| ArgoCD integration | Apply SealedSecret | argocd-vault-plugin | Apply ExternalSecret | Vault injector |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Sealed Secrets | Encrypt secrets client-side, decrypt in-cluster |
| SOPS | In-place encryption using KMS/age/PGP, Flux-native |
| External Secrets | Reference secrets from external providers (AWS, Vault, GCP) |
| Vault Sidecar | Inject secrets at pod runtime via sidecar |
| Never in plaintext | Never commit unencrypted secrets to Git |
| Rotation | ESO and Vault support automatic rotation |
| Recommended for Flux | SOPS (native decryption support) |
| Recommended for multi-cloud | External Secrets Operator |

---

## Quick Revision Questions

1. **Why can't you store plaintext Kubernetes Secrets in Git?**
   > Kubernetes Secrets are only base64-encoded (not encrypted). Anyone with repo access can decode them. Git history preserves them even after deletion.

2. **How does Sealed Secrets work?**
   > `kubeseal` encrypts a Secret using the controller's public key. The SealedSecret is committed to Git. The in-cluster controller decrypts it and creates a regular K8s Secret.

3. **What advantage does SOPS have over Sealed Secrets?**
   > SOPS encrypts values in-place (you can see the structure), supports multiple KMS providers, and Flux has native SOPS decryption support.

4. **When would you choose External Secrets Operator?**
   > When secrets are already in a cloud provider's secret manager (AWS, GCP, Azure) and you want automatic synchronization and rotation.

5. **What is the recommended approach for Flux users?**
   > SOPS with age keys — Flux has native `decryption.provider: sops` support in Kustomization resources.

6. **How do you handle secret rotation in GitOps?**
   > With Sealed Secrets/SOPS, update the encrypted file in Git. With ESO, the operator auto-refreshes at `refreshInterval`. With Vault, leases auto-renew.

---

[← Previous: Environment Promotion](03-environment-promotion.md) | [Back to README](../README.md) | [Next → Drift Detection](05-drift-detection.md)
