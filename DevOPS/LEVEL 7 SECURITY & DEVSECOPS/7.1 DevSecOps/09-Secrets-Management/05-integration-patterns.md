# Chapter 9.5: Integration Patterns

## Overview

Secrets management is only effective when secrets are seamlessly delivered to applications, CI/CD pipelines, containers, and cloud services without human intervention or insecure workarounds. This chapter covers proven integration patterns for delivering secrets securely across different platforms and architectures.

---

## Integration Architecture

```
SECRETS DELIVERY PATTERNS:

  ┌──────────────────────────────────────────────────────┐
  │                  SECRETS STORE                         │
  │         (Vault / AWS SM / Azure KV / GCP SM)          │
  └───────┬──────────┬──────────┬──────────┬─────────────┘
          │          │          │          │
          ▼          ▼          ▼          ▼
  ┌───────────┐ ┌─────────┐ ┌─────────┐ ┌──────────────┐
  │  PATTERN  │ │ PATTERN │ │ PATTERN │ │   PATTERN    │
  │     1     │ │    2    │ │    3    │ │      4       │
  │           │ │         │ │         │ │              │
  │  Direct   │ │ Sidecar │ │ Init   │ │  Environment │
  │  API Call │ │ Agent   │ │ Contner │ │  Injection   │
  │           │ │         │ │         │ │              │
  │ App calls │ │ Vault   │ │ Fetch   │ │ Platform     │
  │ Vault API │ │ Agent   │ │ secrets │ │ injects env  │
  │ directly  │ │ sidecar │ │ before  │ │ vars from    │
  │           │ │ writes  │ │ app     │ │ secrets      │
  │           │ │ to file │ │ starts  │ │ manager      │
  └───────────┘ └─────────┘ └─────────┘ └──────────────┘
```

---

## Pattern 1: Direct API Integration

```python
# Python — Direct Vault API integration
import hvac
import os

class VaultClient:
    """Direct Vault integration for applications."""

    def __init__(self):
        self.client = hvac.Client(
            url=os.environ.get('VAULT_ADDR', 'https://vault.example.com:8200')
        )
        # Authenticate using AppRole
        self.client.auth.approle.login(
            role_id=os.environ['VAULT_ROLE_ID'],
            secret_id=os.environ['VAULT_SECRET_ID']
        )

    def get_secret(self, path, key=None):
        """Read a secret from Vault KV v2."""
        response = self.client.secrets.kv.v2.read_secret_version(
            path=path,
            mount_point='secret'
        )
        data = response['data']['data']
        return data[key] if key else data

    def get_database_creds(self, role='readonly'):
        """Get dynamic database credentials."""
        response = self.client.secrets.database.generate_credentials(
            name=role
        )
        return {
            'username': response['data']['username'],
            'password': response['data']['password'],
            'lease_id': response['lease_id'],
            'ttl': response['lease_duration']
        }

# Usage
vault = VaultClient()
db_config = vault.get_secret('myapp/database')
dynamic_creds = vault.get_database_creds('webapp')
```

```go
// Go — Direct AWS Secrets Manager integration
package main

import (
    "context"
    "encoding/json"
    "github.com/aws/aws-sdk-go-v2/config"
    "github.com/aws/aws-sdk-go-v2/service/secretsmanager"
)

type DBConfig struct {
    Username string `json:"username"`
    Password string `json:"password"`
    Host     string `json:"host"`
    Port     int    `json:"port"`
}

func getSecret(secretID string) (*DBConfig, error) {
    cfg, _ := config.LoadDefaultConfig(context.TODO())
    client := secretsmanager.NewFromConfig(cfg)

    result, err := client.GetSecretValue(context.TODO(),
        &secretsmanager.GetSecretValueInput{
            SecretId: &secretID,
        })
    if err != nil {
        return nil, err
    }

    var dbConfig DBConfig
    json.Unmarshal([]byte(*result.SecretString), &dbConfig)
    return &dbConfig, nil
}
```

---

## Pattern 2: Sidecar Agent (Kubernetes)

```yaml
# Vault Agent Sidecar Injector — Kubernetes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
      annotations:
        # Vault Agent Injector annotations
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "webapp"

        # Inject database credentials as file
        vault.hashicorp.com/agent-inject-secret-db.json: "secret/data/webapp/database"
        vault.hashicorp.com/agent-inject-template-db.json: |
          {{- with secret "secret/data/webapp/database" -}}
          {
            "host": "{{ .Data.data.host }}",
            "username": "{{ .Data.data.username }}",
            "password": "{{ .Data.data.password }}",
            "port": {{ .Data.data.port }}
          }
          {{- end }}

        # Inject API key as env-style file
        vault.hashicorp.com/agent-inject-secret-api-key: "secret/data/webapp/api"
        vault.hashicorp.com/agent-inject-template-api-key: |
          {{- with secret "secret/data/webapp/api" -}}
          export API_KEY="{{ .Data.data.key }}"
          {{- end }}

        # Auto-refresh every 5 minutes
        vault.hashicorp.com/agent-cache-enable: "true"
        vault.hashicorp.com/agent-pre-populate-only: "false"

    spec:
      serviceAccountName: webapp-sa
      containers:
        - name: webapp
          image: webapp:1.0
          # Secrets available at /vault/secrets/
          command: ["/bin/sh", "-c"]
          args:
            - |
              source /vault/secrets/api-key
              python app.py --db-config /vault/secrets/db.json
```

---

## Pattern 3: Init Container

```yaml
# Init Container Pattern — fetch secrets before app starts
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  template:
    spec:
      serviceAccountName: webapp-sa

      # Init container fetches secrets
      initContainers:
        - name: fetch-secrets
          image: vault:1.15
          command: ["/bin/sh", "-c"]
          args:
            - |
              vault login -method=kubernetes role=webapp
              vault kv get -format=json secret/webapp/database \
                | jq -r '.data.data' > /secrets/db.json
              vault kv get -field=key secret/webapp/api \
                > /secrets/api-key
          env:
            - name: VAULT_ADDR
              value: "https://vault.internal:8200"
          volumeMounts:
            - name: secrets
              mountPath: /secrets

      # App container uses secrets from shared volume
      containers:
        - name: webapp
          image: webapp:1.0
          env:
            - name: DB_CONFIG_PATH
              value: /secrets/db.json
          volumeMounts:
            - name: secrets
              mountPath: /secrets
              readOnly: true

      volumes:
        - name: secrets
          emptyDir:
            medium: Memory  # tmpfs — never written to disk
```

---

## Pattern 4: Environment Injection (Cloud-Native)

```yaml
# AWS ECS — Inject from Secrets Manager
{
  "containerDefinitions": [
    {
      "name": "webapp",
      "image": "webapp:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456:secret:prod/db:password::"
        },
        {
          "name": "API_KEY",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456:secret:prod/api:key::"
        }
      ]
    }
  ],
  "executionRoleArn": "arn:aws:iam::123456:role/ecsTaskExecutionRole"
}
```

```yaml
# Kubernetes External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: webapp-secrets
spec:
  refreshInterval: 5m
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: webapp-secrets
    creationPolicy: Owner
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: prod/myapp/database
        property: password
    - secretKey: API_KEY
      remoteRef:
        key: prod/myapp/api
        property: key

---
# SecretStore configuration
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
            namespace: external-secrets
```

---

## CI/CD Integration Patterns

```yaml
# GitHub Actions — OIDC-based secret access (no stored credentials!)
name: Deploy
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

      # --- Vault via OIDC ---
      - name: Import Secrets from Vault
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.example.com:8200
          method: jwt
          role: github-actions
          jwtGithubAudience: https://vault.example.com
          secrets: |
            secret/data/myapp/database password | DB_PASSWORD ;
            secret/data/myapp/api key | API_KEY

      # --- AWS via OIDC ---
      - name: Configure AWS (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456:role/github-deploy
          aws-region: us-east-1

      - name: Get AWS Secrets
        run: |
          DB_PASS=$(aws secretsmanager get-secret-value \
            --secret-id prod/myapp/database \
            --query SecretString --output text | jq -r .password)
          echo "::add-mask::$DB_PASS"
          echo "DB_PASSWORD=$DB_PASS" >> $GITHUB_ENV

      # Use secrets in deployment
      - name: Deploy
        run: |
          helm upgrade myapp ./charts/myapp \
            --set database.password=$DB_PASSWORD
```

```yaml
# GitLab CI — Vault integration
variables:
  VAULT_ADDR: https://vault.example.com:8200

deploy:
  stage: deploy
  id_tokens:
    VAULT_ID_TOKEN:
      aud: https://vault.example.com
  secrets:
    DB_PASSWORD:
      vault: secret/data/myapp/database/password@secret
      token: $VAULT_ID_TOKEN
    API_KEY:
      vault: secret/data/myapp/api/key@secret
      token: $VAULT_ID_TOKEN
  script:
    - echo "Deploying with secrets..."
    - helm upgrade myapp ./charts/myapp
```

---

## Pattern Decision Guide

```
CHOOSING THE RIGHT PATTERN:

  ┌─────────────────────────────────────────────────┐
  │         What is your runtime environment?        │
  └──────────────────┬──────────────────────────────┘
                     │
        ┌────────────┼────────────────┐
        ▼            ▼                ▼
  ┌──────────┐ ┌──────────┐    ┌──────────────┐
  │ Kubernetes│ │ Cloud    │    │ VM / Bare    │
  │           │ │ Managed  │    │ Metal        │
  └─────┬────┘ │ (ECS/    │    └──────┬───────┘
        │       │ Lambda)  │           │
        │       └────┬─────┘           │
        │            │                 │
        ▼            ▼                 ▼
  Use Sidecar   Use Cloud         Use Direct
  Agent or      Native Env        API with
  External      Injection         Vault Agent
  Secrets       (SM ARN ref)      (systemd)
  Operator

  Priority Order:
  1. OIDC/Workload Identity (no stored creds at all)
  2. Sidecar/Agent (transparent to app)
  3. Init Container (simple, one-time fetch)
  4. Direct API (flexible, more app coupling)
  5. Environment injection (acceptable for managed platforms)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Vault Agent sidecar not injecting secrets | Missing annotation or wrong role | Verify all annotations present; check `vault.hashicorp.com/role` matches Vault role name |
| External Secrets not syncing | SecretStore misconfigured | Check ESO controller logs; verify IAM permissions; test `SecretStore` status condition |
| Init container times out | Vault unreachable from pod network | Check network policies; verify Vault service endpoint accessible; check DNS resolution |
| Secrets appear as empty in container | Template rendering error | Check Vault Agent logs; verify secret path and JSON property names match |
| OIDC authentication fails in CI/CD | Audience or bound claims mismatch | Verify `aud` claim matches Vault JWT auth config; check `bound_claims` in role |

---

## Summary Table

| Pattern | Complexity | App Changes | Auto-Refresh | Best For |
|---------|-----------|-------------|-------------|----------|
| **Direct API** | Medium | Code changes | Manual polling | Libraries available; fine-grained control |
| **Sidecar Agent** | Low | File reads | Automatic | Kubernetes; transparent to app |
| **Init Container** | Low | File reads | No (restart required) | Simple apps; one-time fetch |
| **Env Injection** | Low | None | No (restart required) | Cloud managed services (ECS, Lambda) |
| **External Secrets** | Medium | None | Automatic (interval) | K8s; multi-cloud; GitOps |
| **OIDC Federation** | Medium | None | Per-request | CI/CD; zero stored credentials |

---

## Quick Revision Questions

1. **What are the main secrets delivery patterns?**
   > Direct API integration (app calls Vault/SM SDK), sidecar agent (Vault Agent injects secrets as files), init container (fetch secrets before app starts), environment injection (platform injects env vars from secrets manager), and External Secrets Operator (syncs cloud secrets to K8s Secrets).

2. **Why is the sidecar agent pattern preferred in Kubernetes?**
   > The sidecar agent runs alongside the app container, transparently fetching and refreshing secrets without any application code changes. It writes secrets to shared files, supports automatic renewal, handles authentication via Kubernetes service accounts, and keeps secrets out of environment variables and Kubernetes Secret objects.

3. **How does OIDC-based secret access work in CI/CD?**
   > The CI/CD platform (GitHub Actions, GitLab CI) issues a short-lived OIDC token identifying the workflow. This token is exchanged with Vault or cloud providers for temporary credentials using federated identity. No long-lived secrets are stored in CI/CD variables — authentication is based on the identity of the pipeline itself.

4. **What is the External Secrets Operator?**
   > ESO is a Kubernetes operator that synchronizes secrets from external managers (AWS SM, Azure KV, GCP SM, Vault) into native Kubernetes Secrets. It polls the external source at configurable intervals and updates K8s Secrets automatically. This enables GitOps workflows where ExternalSecret manifests declare which secrets to sync.

5. **When should you use init containers vs sidecar agents?**
   > Init containers are simpler — they fetch secrets once before the app starts, writing to a shared volume. Use them when secrets rarely change and restarts are acceptable. Sidecar agents continuously run alongside the app, refreshing secrets on a schedule and supporting dynamic secrets with TTLs. Use sidecars when secrets rotate frequently or you need zero-downtime refresh.

---

[← Previous: 9.4 Secret Rotation](04-secret-rotation.md) | [Next: 9.6 Secrets Best Practices →](06-secrets-best-practices.md)

[Back to Table of Contents](../README.md)
