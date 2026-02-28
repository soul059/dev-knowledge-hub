# Chapter 7.4: Pod Security Standards

## Overview

Pod Security Standards (PSS) replaced the deprecated PodSecurityPolicy (PSP) in Kubernetes v1.25+. PSS defines three security profiles — Privileged, Baseline, and Restricted — enforced via the built-in Pod Security Admission (PSA) controller. This chapter covers each profile, enforcement modes, and migration.

---

## Pod Security Standards Levels

```
POD SECURITY STANDARDS — THREE LEVELS:

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  PRIVILEGED          BASELINE           RESTRICTED         │
  │  ┌──────────┐       ┌──────────┐       ┌──────────┐       │
  │  │          │       │          │       │          │       │
  │  │ No       │       │ Prevent  │       │ Heavily  │       │
  │  │ restrict-│       │ known    │       │ restrict-│       │
  │  │ ions     │       │ privilege│       │ ed; best │       │
  │  │          │       │ escala-  │       │ practice │       │
  │  │ Use for: │       │ tions    │       │ hardened │       │
  │  │ system   │       │          │       │          │       │
  │  │ critical │       │ Use for: │       │ Use for: │       │
  │  │ workloads│       │ most     │       │ security │       │
  │  │ (kube-   │       │ workloads│       │ sensitive│       │
  │  │ system)  │       │          │       │ workloads│       │
  │  └──────────┘       └──────────┘       └──────────┘       │
  │                                                            │
  │  Least Secure ◄──────────────────────► Most Secure        │
  └──────────────────────────────────────────────────────────┘
```

### Profile Comparison

| Control | Privileged | Baseline | Restricted |
|---------|-----------|----------|------------|
| Privileged containers | Allowed | **Blocked** | **Blocked** |
| Host namespaces (PID, IPC, Network) | Allowed | **Blocked** | **Blocked** |
| Host ports | Allowed | Limited | **Blocked** |
| HostPath volumes | Allowed | **Blocked** | **Blocked** |
| Privileged escalation | Allowed | Allowed | **Blocked** |
| Run as non-root | Not required | Not required | **Required** |
| Seccomp profile | Not required | Not required | **Required** (RuntimeDefault) |
| Capabilities | All allowed | Drop some | **Drop ALL** (add limited) |
| Root filesystem | Writable | Writable | Writable (recommend RO) |

---

## Enforcement Modes

```
PSA ENFORCEMENT MODES:

  ┌──────────┬──────────────────────────────────────┐
  │ Mode     │ Behavior                              │
  ├──────────┼──────────────────────────────────────┤
  │ enforce  │ REJECT pod that violates policy       │
  │ audit    │ Allow but LOG violation in audit log   │
  │ warn     │ Allow but WARN user in kubectl output  │
  └──────────┴──────────────────────────────────────┘

  Recommended migration path:
  1. warn   → See what would break
  2. audit  → Log violations 
  3. enforce → Block violations
```

---

## Applying Pod Security Standards

```bash
# Apply PSS to a namespace using labels
# Format: pod-security.kubernetes.io/<MODE>: <LEVEL>

# Enforce baseline + warn on restricted violations
kubectl label namespace production \
    pod-security.kubernetes.io/enforce=baseline \
    pod-security.kubernetes.io/enforce-version=latest \
    pod-security.kubernetes.io/warn=restricted \
    pod-security.kubernetes.io/warn-version=latest \
    pod-security.kubernetes.io/audit=restricted \
    pod-security.kubernetes.io/audit-version=latest

# Enforce restricted for sensitive namespaces
kubectl label namespace pci-workloads \
    pod-security.kubernetes.io/enforce=restricted \
    pod-security.kubernetes.io/enforce-version=latest

# Privileged for system namespaces
kubectl label namespace kube-system \
    pod-security.kubernetes.io/enforce=privileged \
    --overwrite
```

```yaml
# Namespace with PSS labels
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/audit: restricted
```

---

## Baseline-Compliant Pod

```yaml
# This pod passes BASELINE checks
apiVersion: v1
kind: Pod
metadata:
  name: baseline-app
  namespace: production
spec:
  containers:
    - name: app
      image: myapp:1.0.0
      ports:
        - containerPort: 8080
      # Baseline: no privileged, no hostNetwork, no hostPID
      # Baseline: no hostPath volumes
      securityContext:
        allowPrivilegeEscalation: true  # Allowed in baseline
        runAsUser: 1000                 # Good practice but not required
```

---

## Restricted-Compliant Pod

```yaml
# This pod passes RESTRICTED checks (most secure)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      securityContext:
        runAsNonRoot: true              # REQUIRED: restricted
        seccompProfile:
          type: RuntimeDefault          # REQUIRED: restricted
      containers:
        - name: app
          image: myapp:1.0.0
          ports:
            - containerPort: 8080
          securityContext:
            allowPrivilegeEscalation: false  # REQUIRED: restricted
            readOnlyRootFilesystem: true     # Recommended
            runAsNonRoot: true               # REQUIRED: restricted
            runAsUser: 1000
            capabilities:
              drop:
                - ALL                         # REQUIRED: restricted
              # Can add back only: NET_BIND_SERVICE
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
            requests:
              memory: "128Mi"
              cpu: "250m"
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir:
            sizeLimit: 64Mi
      # No hostPath, no hostNetwork, no hostPID, no hostIPC
```

---

## Cluster-Wide PSA Configuration

```yaml
# /etc/kubernetes/psa-config.yaml
# Applied via --admission-control-config-file flag on API server
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: PodSecurity
    configuration:
      apiVersion: pod-security.admission.config.k8s.io/v1
      kind: PodSecurityConfiguration
      defaults:
        enforce: baseline
        enforce-version: latest
        warn: restricted
        warn-version: latest
        audit: restricted
        audit-version: latest
      exemptions:
        # Exempt system namespaces
        namespaces:
          - kube-system
          - kube-public
          - istio-system
        # Exempt specific users
        usernames:
          - system:serviceaccount:kube-system:*
        # Exempt specific runtime classes
        runtimeClasses: []
```

---

## Dry-Run and Migration

```bash
# Check what would be affected BEFORE enforcing
# Dry-run: test restricted level against existing pods
kubectl label --dry-run=server --overwrite namespace production \
    pod-security.kubernetes.io/enforce=restricted

# Output example:
# Warning: existing pods in namespace "production" violate the new 
#   PodSecurity enforce level "restricted":
#   myapp-7d6f4c8b9-abc12: allowPrivilegeEscalation != false,
#     unrestricted capabilities, runAsNonRoot != true,
#     seccompProfile not set

# Check specific pod compliance
kubectl get pod myapp-7d6f4c8b9-abc12 -n production -o yaml | \
    kubectl apply --dry-run=server -f -

# Count violations per namespace
for ns in $(kubectl get ns -o jsonpath='{.items[*].metadata.name}'); do
  echo "=== $ns ==="
  kubectl label --dry-run=server --overwrite namespace "$ns" \
      pod-security.kubernetes.io/enforce=restricted 2>&1 | \
      grep -c "Warning" || echo "0 violations"
done
```

---

## Migration Strategy

```
PSP → PSA MIGRATION PATH:

  Phase 1: AUDIT (Week 1-2)
  ┌─────────────────────────────────────┐
  │ • Label namespaces with audit mode  │
  │ • Collect violation data            │
  │ • Identify non-compliant workloads  │
  └─────────────────────────────────────┘
            │
            ▼
  Phase 2: WARN (Week 3-4)
  ┌─────────────────────────────────────┐
  │ • Add warn mode to namespaces       │
  │ • Developers see warnings           │
  │ • Fix non-compliant deployments     │
  └─────────────────────────────────────┘
            │
            ▼
  Phase 3: ENFORCE (Week 5+)
  ┌─────────────────────────────────────┐
  │ • Enable enforce mode               │
  │ • Non-compliant pods are rejected   │
  │ • Monitor audit logs for issues     │
  └─────────────────────────────────────┘

  TIP: Use enforce=baseline + warn=restricted
       as an intermediate step!
```

---

## Kyverno for Advanced Pod Security

```yaml
# When PSA isn't flexible enough, use Kyverno
# Example: Require specific seccomp profile
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-seccomp
spec:
  validationFailureAction: Enforce
  rules:
    - name: check-seccomp
      match:
        any:
          - resources:
              kinds: ["Pod"]
      exclude:
        any:
          - resources:
              namespaces: ["kube-system"]
      validate:
        message: "Pods must use RuntimeDefault or Localhost seccomp profile"
        pattern:
          spec:
            securityContext:
              seccompProfile:
                type: "RuntimeDefault | Localhost"

---
# Require non-root + read-only root FS
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-security-context
spec:
  validationFailureAction: Enforce
  rules:
    - name: require-run-as-non-root
      match:
        any:
          - resources:
              kinds: ["Pod"]
      validate:
        message: "Containers must run as non-root with read-only root filesystem"
        pattern:
          spec:
            containers:
              - securityContext:
                  runAsNonRoot: true
                  readOnlyRootFilesystem: true
                  allowPrivilegeEscalation: false
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pod rejected: `allowPrivilegeEscalation` | Restricted requires `false` | Set `securityContext.allowPrivilegeEscalation: false` |
| Pod rejected: `runAsNonRoot` | Restricted requires non-root | Set `runAsNonRoot: true` and `runAsUser: 1000`+ |
| Pod rejected: seccomp not set | Restricted requires seccomp | Add `seccompProfile.type: RuntimeDefault` |
| Pod rejected: capabilities | Restricted requires drop ALL | Add `capabilities.drop: ["ALL"]` |
| System pods breaking after enforce | System ns needs privileged | Exempt `kube-system` in PSA config or label as privileged |

---

## Summary Table

| Feature | Description |
|---------|------------|
| **Privileged** | No restrictions — for system components (kube-system) |
| **Baseline** | Prevents known privilege escalations — good default for most workloads |
| **Restricted** | Maximum security — required for security-sensitive workloads |
| **enforce** | Reject non-compliant pods |
| **warn** | Allow but warn user in kubectl output |
| **audit** | Allow but log in audit log |
| **Dry-run** | Test enforcement impact before enabling |

---

## Quick Revision Questions

1. **What are the three Pod Security Standards levels?**
   > Privileged (no restrictions), Baseline (prevents known privilege escalations like hostNetwork, hostPID, privileged containers), and Restricted (heavily locked down — requires non-root, drop all capabilities, seccomp profile, no privilege escalation).

2. **How does PSA replace PodSecurityPolicy?**
   > PSA is a built-in admission controller that enforces Pod Security Standards using namespace labels. Unlike PSP (which was complex, required RBAC binding), PSA is simpler — just label a namespace with the desired level and mode. PSP was removed in Kubernetes v1.25.

3. **What is the recommended migration strategy from no enforcement to restricted?**
   > Start with `audit` mode to log violations without blocking. Then add `warn` mode so developers see warnings. Finally enable `enforce` to block non-compliant pods. An intermediate step is `enforce=baseline` + `warn=restricted` to catch the most critical issues first.

4. **What security settings does the Restricted profile require?**
   > `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `seccompProfile.type: RuntimeDefault`, no hostPath volumes, no host namespaces (PID/IPC/Network), no privileged containers.

5. **When should you use Kyverno instead of built-in PSA?**
   > When you need more granular policies than PSA's three levels offer. Kyverno allows custom validation rules: require specific labels, enforce image registries, mandate resource limits, require read-only rootfs, and more. It also supports mutation (auto-fixing non-compliant pods).

---

[← Previous: 7.3 Network Policies](03-network-policies.md) | [Next: 7.5 Kubernetes Secrets Management →](05-k8s-secrets-management.md)

[Back to Table of Contents](../README.md)
