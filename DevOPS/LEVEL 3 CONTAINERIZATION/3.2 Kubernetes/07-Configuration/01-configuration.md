# Unit 7: Configuration

[← Previous: Storage](../06-Storage/01-storage.md) | [Back to README](../README.md) | [Next: Scheduling →](../08-Scheduling/01-scheduling.md)

---

## Chapter Overview

Kubernetes provides several objects for externalizing application configuration from container images. This unit covers **ConfigMaps**, **Secrets**, environment variables, volume-based config mounting, and immutable configurations — best practices for keeping your deployments flexible and secure.

---

## 7.1 ConfigMaps

A **ConfigMap** stores non-confidential configuration data as key-value pairs. It decouples configuration from container images, making applications portable.

```
┌──────────────────────────────────────────────────────────────┐
│                      ConfigMap                                │
│                                                              │
│  ┌─────────────────────────────────────────────────┐         │
│  │  ConfigMap: app-config                          │         │
│  │                                                 │         │
│  │  Key-Value Data:                                │         │
│  │  ┌─────────────────┬──────────────────────┐     │         │
│  │  │ DATABASE_HOST   │ postgres-svc          │     │         │
│  │  │ DATABASE_PORT   │ 5432                  │     │         │
│  │  │ LOG_LEVEL       │ info                  │     │         │
│  │  │ MAX_CONNECTIONS │ 100                   │     │         │
│  │  └─────────────────┴──────────────────────┘     │         │
│  │                                                 │         │
│  │  File-like Data:                                │         │
│  │  ┌─────────────────┬──────────────────────┐     │         │
│  │  │ nginx.conf      │ server { ... }        │     │         │
│  │  │ app.properties  │ key1=value1\n...      │     │         │
│  │  └─────────────────┴──────────────────────┘     │         │
│  └─────────────────────────────────────────────────┘         │
│                                                              │
│  Usage:                                                      │
│  ┌──────────────────┐    ┌──────────────────────┐            │
│  │ As ENV variables │    │ As mounted files     │            │
│  │ in containers    │    │ (volume)             │            │
│  └──────────────────┘    └──────────────────────┘            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Creating ConfigMaps

```bash
# ═══════════ Imperative Methods ═══════════

# From literal values
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=postgres-svc \
  --from-literal=DATABASE_PORT=5432 \
  --from-literal=LOG_LEVEL=info

# From a file
kubectl create configmap nginx-config --from-file=nginx.conf

# From a directory (all files become keys)
kubectl create configmap app-configs --from-file=./config-dir/

# From an env file
kubectl create configmap app-env --from-env-file=app.env

# Generate YAML without creating
kubectl create configmap app-config \
  --from-literal=KEY=VALUE \
  --dry-run=client -o yaml > configmap.yaml
```

### ConfigMap YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Simple key-value pairs
  DATABASE_HOST: "postgres-svc"
  DATABASE_PORT: "5432"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"

  # Multi-line file content
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        location / {
            proxy_pass http://backend:8080;
        }
    }

  application.yaml: |
    spring:
      datasource:
        url: jdbc:postgresql://postgres-svc:5432/mydb
      jpa:
        hibernate:
          ddl-auto: update
```

### Using ConfigMaps in Pods

#### Method 1: Environment Variables (Single Keys)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_HOST                    # Env var name in container
      valueFrom:
        configMapKeyRef:
          name: app-config             # ConfigMap name
          key: DATABASE_HOST           # Key in ConfigMap
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_PORT
```

#### Method 2: Environment Variables (All Keys at Once)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: my-app:1.0
    envFrom:
    - configMapRef:
        name: app-config               # All keys become env vars
      prefix: APP_                      # Optional: prefix all vars
    # Result: APP_DATABASE_HOST, APP_DATABASE_PORT, APP_LOG_LEVEL, etc.
```

#### Method 3: Volume Mount (Files)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d      # Each key = a file
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:                            # Optional: select specific keys
      - key: nginx.conf
        path: default.conf             # Rename the file
```

---

## Secrets

A **Secret** stores sensitive data (passwords, tokens, certificates). Similar to ConfigMaps but with base64 encoding and access controls.

```
┌──────────────────────────────────────────────────────────────┐
│                        Secrets                                │
│                                                              │
│  ⚠ Base64 encoding is NOT encryption!                        │
│  It only obscures data. Use external secret managers         │
│  (Vault, AWS Secrets Manager) for real security.             │
│                                                              │
│  Secret Types:                                               │
│  ┌────────────────────────┬────────────────────────────┐     │
│  │ Type                   │ Usage                      │     │
│  ├────────────────────────┼────────────────────────────┤     │
│  │ Opaque (default)       │ Arbitrary user-defined data│     │
│  │ kubernetes.io/tls      │ TLS certificates           │     │
│  │ kubernetes.io/         │ Docker registry creds      │     │
│  │   dockerconfigjson     │                            │     │
│  │ kubernetes.io/         │ Basic auth credentials     │     │
│  │   basic-auth           │                            │     │
│  │ kubernetes.io/         │ SSH credentials            │     │
│  │   ssh-auth             │                            │     │
│  │ kubernetes.io/         │ ServiceAccount tokens      │     │
│  │   service-account-token│                            │     │
│  └────────────────────────┴────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Creating Secrets

```bash
# From literal values (auto base64-encodes)
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password='S3cur3P@ss!'

# From files
kubectl create secret generic tls-certs \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key

# TLS secret (special type)
kubectl create secret tls my-tls \
  --cert=tls.crt --key=tls.key

# Docker registry secret
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# View a secret (decoded)
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

### Secret YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Values must be base64-encoded
  username: YWRtaW4=                # echo -n "admin" | base64
  password: UzNjdXIzUEBzcyE=       # echo -n "S3cur3P@ss!" | base64
---
# Alternative: use stringData (auto-encodes)
apiVersion: v1
kind: Secret
metadata:
  name: db-secret-plain
type: Opaque
stringData:
  username: admin                   # Plain text (K8s encodes it)
  password: "S3cur3P@ss!"
```

### Using Secrets in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: my-app:1.0

    # Method 1: Single env var from secret
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password

    # Method 2: All keys as env vars
    envFrom:
    - secretRef:
        name: db-secret

    # Method 3: Mount as volume (files)
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true

  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
      defaultMode: 0400           # File permissions (read-only)

  # For Docker registry secrets
  imagePullSecrets:
  - name: my-registry
```

---

## Environment Variables

### All Methods of Setting Env Vars

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    # 1. Hardcoded value
    - name: APP_COLOR
      value: "blue"

    # 2. From ConfigMap key
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST

    # 3. From Secret key
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password

    # 4. From Pod field (Downward API)
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName

    # 5. From container resource field
    - name: CPU_LIMIT
      valueFrom:
        resourceFieldRef:
          containerName: app
          resource: limits.cpu
    - name: MEM_REQUEST
      valueFrom:
        resourceFieldRef:
          containerName: app
          resource: requests.memory

    # 6. Bulk import from ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
        prefix: CONF_            # All keys prefixed with CONF_
    - secretRef:
        name: app-secrets
```

```
┌──────────────────────────────────────────────────────────────┐
│               ENVIRONMENT VARIABLE SOURCES                    │
│                                                              │
│   ┌──────────────┐                                           │
│   │   Pod Spec   │                                           │
│   │              │                                           │
│   │  env:        ├──▶ value: "literal"                       │
│   │              ├──▶ configMapKeyRef → ConfigMap             │
│   │              ├──▶ secretKeyRef → Secret                   │
│   │              ├──▶ fieldRef → Pod metadata/status          │
│   │              └──▶ resourceFieldRef → Container resources  │
│   │                                                          │
│   │  envFrom:    ├──▶ configMapRef → All keys from ConfigMap  │
│   │              └──▶ secretRef → All keys from Secret        │
│   │                                                          │
│   └──────────────┘                                           │
│                                                              │
│  ⚠ env vars are set at container START. Changes to           │
│    ConfigMaps/Secrets are NOT reflected until pod restart.    │
│    (Volume mounts auto-update, but not env vars)             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Mounting Configs as Volumes

When you mount a ConfigMap or Secret as a volume, each key becomes a **file** and each value becomes the **file content**.

```
┌────────────────────────────────────────────────────────────────┐
│           CONFIGMAP / SECRET AS VOLUME                         │
│                                                                │
│  ConfigMap: app-config                                         │
│  ┌──────────────────────┐                                      │
│  │ DATABASE_HOST = pg   │                                      │
│  │ LOG_LEVEL = info     │         Pod filesystem:              │
│  │ nginx.conf = server  │                                      │
│  │   { listen 80; ... } │    /etc/config/                      │
│  └──────────┬───────────┘    ├── DATABASE_HOST  → "pg"         │
│             │                ├── LOG_LEVEL       → "info"       │
│        Mount as volume       └── nginx.conf      → "server..." │
│                                                                │
│  With 'items' (selective mount):                               │
│  ┌──────────────────────┐                                      │
│  │ volumes:             │    /etc/nginx/conf.d/                 │
│  │   configMap:         │    └── default.conf  → "server..."   │
│  │     items:           │                                      │
│  │     - key: nginx.conf│    Only selected keys are mounted    │
│  │       path: default. │                                      │
│  │             conf     │                                      │
│  └──────────────────────┘                                      │
│                                                                │
│  With 'subPath' (mount single file without replacing dir):     │
│  ┌──────────────────────┐                                      │
│  │ volumeMounts:        │    /etc/nginx/nginx.conf ← just this │
│  │ - mountPath: /etc/   │       file, other files in /etc/     │
│  │   nginx/nginx.conf   │       nginx/ are preserved           │
│  │   subPath: nginx.conf│                                      │
│  └──────────────────────┘                                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Auto-Update Behavior

```
┌──────────────────────────────────────────────────────────────┐
│            CONFIG UPDATE BEHAVIOR                             │
│                                                              │
│  Method              │ Auto-Update? │ When?                  │
│  ────────────────────┼──────────────┼──────────────────────  │
│  env / envFrom       │ ❌ No         │ Need pod restart       │
│  Volume mount        │ ✅ Yes        │ ~60-90 seconds delay   │
│  Volume + subPath    │ ❌ No         │ Need pod restart       │
│  Immutable ConfigMap │ ❌ No         │ Cannot be changed      │
│                                                              │
│  Volume mount auto-update uses symlinks:                     │
│  /etc/config/                                                │
│    ├── ..data → ..2024_01_15_10_30_00.123456789              │
│    ├── ..2024_01_15_10_30_00.123456789/                      │
│    │   ├── DATABASE_HOST                                     │
│    │   └── LOG_LEVEL                                         │
│    ├── DATABASE_HOST → ..data/DATABASE_HOST                  │
│    └── LOG_LEVEL → ..data/LOG_LEVEL                          │
│                                                              │
│  When updated, K8s atomically swaps the ..data symlink       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Volume Mount Example with subPath

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    # Mount entire ConfigMap as directory
    - name: full-config
      mountPath: /app/config

    # Mount single file without replacing directory
    - name: nginx-config
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf              # Only this key

  volumes:
  - name: full-config
    configMap:
      name: app-config
  - name: nginx-config
    configMap:
      name: nginx-config
```

---

## Immutable ConfigMaps and Secrets

Marking ConfigMaps/Secrets as **immutable** prevents changes after creation. Benefits:

- **Performance:** Kubelet doesn't need to watch for changes
- **Safety:** Prevents accidental modifications in production
- **Scale:** Reduces API server load (no watch streams)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-v2
immutable: true                    # Cannot be changed after creation
data:
  DATABASE_HOST: "postgres-svc"
  LOG_LEVEL: "warn"
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret-v3
immutable: true                    # Cannot be changed after creation
type: Opaque
stringData:
  password: "NewS3cur3P@ss!"
```

### Immutable Config Update Strategy

```
┌──────────────────────────────────────────────────────────────┐
│         IMMUTABLE CONFIG UPDATE WORKFLOW                       │
│                                                              │
│  1. Create new ConfigMap with version suffix                 │
│     app-config-v1 → app-config-v2                            │
│                                                              │
│  2. Update Deployment to reference new ConfigMap             │
│     triggers rolling update                                  │
│                                                              │
│  3. Delete old ConfigMap when rollout is complete             │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐               │
│  │config-v1 │    │config-v2 │    │config-v2 │               │
│  │(in use)  │───▶│(deployed)│───▶│(in use)  │               │
│  │          │    │config-v1 │    │config-v1 │               │
│  │          │    │(being    │    │(deleted) │               │
│  │          │    │replaced) │    │          │               │
│  └──────────┘    └──────────┘    └──────────┘               │
│                                                              │
│  This ensures atomic config updates with rollback capability │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│              CONFIGURATION BEST PRACTICES                     │
│                                                              │
│  1. NEVER hardcode config in container images                │
│     → Use ConfigMaps and Secrets                             │
│                                                              │
│  2. NEVER put sensitive data in ConfigMaps                   │
│     → Use Secrets (or external secret managers)              │
│                                                              │
│  3. Use immutable ConfigMaps/Secrets for production          │
│     → Prevents accidental changes                            │
│                                                              │
│  4. Version your ConfigMaps (app-config-v1, v2, ...)         │
│     → Enables rollback                                       │
│                                                              │
│  5. Use volume mounts when you need auto-updates             │
│     → Env vars don't auto-update                             │
│                                                              │
│  6. Set defaultMode on secret volumes                        │
│     → defaultMode: 0400 (read-only by owner)                 │
│                                                              │
│  7. Use external secret managers for production              │
│     → HashiCorp Vault, AWS Secrets Manager, Azure KeyVault   │
│     → Tools: External Secrets Operator, Sealed Secrets       │
│                                                              │
│  8. Apply RBAC to restrict Secret access                     │
│     → Not everyone should read all secrets                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Application with Full Configuration

```yaml
# ConfigMap for app settings
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
  namespace: production
data:
  APP_ENV: "production"
  LOG_LEVEL: "warn"
  CACHE_TTL: "3600"
  application.yaml: |
    server:
      port: 8080
    spring:
      datasource:
        url: jdbc:postgresql://${DB_HOST}:5432/mydb
---
# Secret for credentials
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secrets
  namespace: production
type: Opaque
stringData:
  DB_USERNAME: "app_user"
  DB_PASSWORD: "Pr0dP@ssw0rd!"
  API_KEY: "sk-abcdef123456"
---
# Deployment using both
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp:2.0
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: webapp-config        # All config as env vars
        env:
        - name: DB_HOST
          value: "postgres.production.svc"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: webapp-secrets
              key: DB_PASSWORD
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: webapp-secrets
              key: API_KEY
        volumeMounts:
        - name: app-config
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: app-config
        configMap:
          name: webapp-config
          items:
          - key: application.yaml
            path: application.yaml
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| Env var not set in container | `kubectl exec pod -- env` | Check ConfigMap/Secret name and key spelling |
| Config file not appearing | `kubectl exec pod -- ls /path` | Verify volumeMounts path and ConfigMap items |
| Config changes not reflected | Env var based | Restart pod/deployment (env vars don't auto-update) |
| Secret not found | `kubectl get secret -n <ns>` | Check namespace matches pod's namespace |
| Permission denied on secret file | Check defaultMode | Set appropriate `defaultMode` in volume spec |
| Base64 decode errors | `echo <value> \| base64 -d` | Ensure values were encoded with `echo -n` (no newline) |

```bash
# Debugging commands
kubectl get configmaps                             # List ConfigMaps
kubectl describe configmap app-config              # View details
kubectl get configmap app-config -o yaml           # Full YAML

kubectl get secrets                                # List Secrets
kubectl get secret db-secret -o yaml               # View (base64 encoded)
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d  # Decode

kubectl exec <pod> -- env | sort                   # Check env vars in pod
kubectl exec <pod> -- cat /etc/config/key          # Check mounted file
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ConfigMap** | Stores non-sensitive config as key-value pairs |
| **Secret** | Stores sensitive data (base64-encoded, not encrypted at rest by default) |
| **env / valueFrom** | Inject single keys as environment variables |
| **envFrom** | Inject all keys from ConfigMap/Secret as env vars |
| **Volume Mount** | Mount ConfigMap/Secret as files in container filesystem |
| **subPath** | Mount single file without replacing the entire directory |
| **Auto-Update** | Volume mounts auto-update (~60-90s); env vars do NOT |
| **Immutable** | Prevents changes; better performance; use versioning for updates |
| **stringData** | Secret field that accepts plain text (auto-encodes to base64) |
| **Downward API** | Expose pod/container metadata as env vars or files |

---

## Quick Revision Questions

1. **What is the key difference between ConfigMaps and Secrets?**
   <details><summary>Answer</summary>ConfigMaps store non-sensitive configuration data in plain text. Secrets store sensitive data (passwords, tokens, keys) that is base64-encoded. Secrets also have additional access control via RBAC and can be encrypted at rest when configured. However, base64 is NOT encryption — Secrets should be combined with external secret managers for true security.</details>

2. **How do you make a ConfigMap change take effect if it's used as an environment variable?**
   <details><summary>Answer</summary>You must restart the pod (or delete it so the controller recreates it). Environment variables are set at container startup and don't auto-update. Alternatively, mount the ConfigMap as a volume — volume mounts auto-update within ~60-90 seconds.</details>

3. **What is the `subPath` field used for in volumeMounts?**
   <details><summary>Answer</summary>subPath lets you mount a single file from a ConfigMap/Secret without replacing the entire target directory. Without subPath, the volume mount replaces all content at the mountPath. With subPath, only the specified file is placed there, preserving other files. Note: subPath mounts do NOT auto-update.</details>

4. **How would you expose a pod's own name and namespace as environment variables?**
   <details><summary>Answer</summary>Using the Downward API with fieldRef:
   ```yaml
   env:
   - name: POD_NAME
     valueFrom:
       fieldRef:
         fieldPath: metadata.name
   - name: POD_NAMESPACE
     valueFrom:
       fieldRef:
         fieldPath: metadata.namespace
   ```</details>

5. **What are the benefits of making a ConfigMap immutable?**
   <details><summary>Answer</summary>1) Prevents accidental modifications in production. 2) Improves performance — kubelet stops watching for changes. 3) Reduces API server load (no watch streams). The tradeoff is that updates require creating a new ConfigMap with a new name and updating the Deployment reference.</details>

6. **Why is base64 encoding NOT considered secure for Secrets?**
   <details><summary>Answer</summary>Base64 is an encoding, not encryption — anyone can decode it (`echo "encoded" | base64 -d`). Kubernetes Secrets are stored unencrypted in etcd by default. For real security: enable encryption at rest for etcd, use RBAC to restrict Secret access, and use external secret managers (Vault, AWS Secrets Manager) for production.</details>

---

[← Previous: Storage](../06-Storage/01-storage.md) | [Back to README](../README.md) | [Next: Scheduling →](../08-Scheduling/01-scheduling.md)
