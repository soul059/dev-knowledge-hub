# Chapter 6.5: Security Considerations

[← Previous: Testing GitOps](04-testing-gitops.md) | [Back to README](../README.md) | [Next → Monitoring GitOps](06-monitoring-gitops.md)

---

## Overview

GitOps improves security by reducing direct cluster access and providing an audit trail, but it also introduces new attack surfaces — the Git repository, container registries, and the GitOps operator itself. This chapter covers the full security model for GitOps.

---

## GitOps Security Model

```
┌──────────────────────────────────────────────────────────────┐
│              GITOPS SECURITY LAYERS                          │
│                                                              │
│  ┌────────────────────────────────────────┐                  │
│  │ Layer 1: Git Repository Security       │                  │
│  │ ├── Branch protection                  │                  │
│  │ ├── Signed commits (GPG/SSH)           │                  │
│  │ ├── CODEOWNERS + PR reviews            │                  │
│  │ ├── Access control (least privilege)   │                  │
│  │ └── Audit logging                      │                  │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │ Layer 2: Container Image Security      │                  │
│  │ ├── Image scanning (Trivy, Snyk)       │                  │
│  │ ├── Image signing (Cosign/Notation)    │                  │
│  │ ├── Trusted registries only            │                  │
│  │ └── No :latest tag                     │                  │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │ Layer 3: GitOps Operator Security      │                  │
│  │ ├── Operator RBAC (least privilege)    │                  │
│  │ ├── Network policies                   │                  │
│  │ ├── Sealed/encrypted secrets           │                  │
│  │ └── Admission controllers              │                  │
│  └────────────────────────────────────────┘                  │
│  ┌────────────────────────────────────────┐                  │
│  │ Layer 4: Cluster Security              │                  │
│  │ ├── Pod Security Standards             │                  │
│  │ ├── Network Policies                   │                  │
│  │ ├── RBAC (no direct kubectl for devs)  │                  │
│  │ └── Audit logging                      │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Principle: Least Privilege Cluster Access

```
┌──────────────────────────────────────────────────────────┐
│         TRADITIONAL vs GITOPS ACCESS MODEL               │
│                                                          │
│  Traditional:                                            │
│  ├── 10 developers with cluster-admin kubeconfig         │
│  ├── CI pipeline with cluster-admin token                │
│  └── Ops team with kubectl access to production          │
│  → Large blast radius, hard to audit                     │
│                                                          │
│  GitOps:                                                 │
│  ├── Developers: Git write access only                   │
│  │   (no kubectl, no kubeconfig)                         │
│  ├── GitOps operator: cluster access                     │
│  │   (single service account, scoped)                    │
│  ├── Ops team: read-only kubectl                         │
│  │   (for debugging only)                                │
│  └── CI pipeline: Git push only                          │
│      (never touches cluster)                             │
│  → Minimal blast radius, full audit trail                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Git Repository Security

### Commit Signing

```bash
# Generate GPG key
gpg --full-generate-key

# Configure Git to use GPG
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true

# Signed commit
git commit -S -m "signed: update production image"

# Verify signature
git verify-commit HEAD
```

### SSH Signing (Git 2.34+)

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub

git commit -S -m "ssh-signed commit"
```

### Enforce in ArgoCD

```yaml
# Require signed commits for an ArgoCD project
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
spec:
  signatureKeys:
  - keyID: <GPG_KEY_ID>     # Only signed commits are accepted
```

---

## Container Image Security

### Image Signing with Cosign

```bash
# Sign an image after build
cosign sign --key cosign.key docker.io/myorg/my-app:v1.2.0

# Verify signature
cosign verify --key cosign.pub docker.io/myorg/my-app:v1.2.0
```

### Enforce Signed Images with Kyverno

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  rules:
  - name: verify-cosign-signature
    match:
      resources:
        kinds:
        - Pod
    verifyImages:
    - imageReferences:
      - "docker.io/myorg/*"
      attestors:
      - entries:
        - keys:
            publicKeys: |-
              -----BEGIN PUBLIC KEY-----
              MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE...
              -----END PUBLIC KEY-----
```

### Image Scanning in CI

```yaml
# GitHub Actions: scan before deploy
- name: Scan image with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myorg/my-app:${{ github.sha }}
    format: table
    exit-code: 1
    severity: CRITICAL,HIGH
```

---

## GitOps Operator Security

### ArgoCD Hardening

```yaml
# Restrict ArgoCD API access
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Disable anonymous access
  users.anonymous.enabled: "false"
  
  # Set session timeout
  timeout.session: "24h"
  
  # Restrict admin password
  admin.enabled: "false"         # Disable local admin after SSO setup
```

### Network Policies for GitOps Operator

```yaml
# Restrict ArgoCD network access
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: argocd-server
  namespace: argocd
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: argocd-server
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx     # Only from ingress
    ports:
    - port: 8080
  egress:
  - to: []         # Allow outbound (Git, cluster API)
    ports:
    - port: 443
    - port: 22
```

---

## Policy Enforcement

### Admission Control Pipeline

```
┌──────────────────────────────────────────────────────────┐
│         POLICY ENFORCEMENT LAYERS                        │
│                                                          │
│  Pre-commit:                                             │
│  └── git hooks validate YAML                             │
│                                                          │
│  PR CI:                                                  │
│  └── conftest / kyverno CLI check policies               │
│                                                          │
│  Admission (in-cluster):                                 │
│  └── OPA Gatekeeper / Kyverno validating webhook         │
│                                                          │
│  Runtime:                                                │
│  └── Falco / audit logging for violations                │
│                                                          │
│  Defense in depth:                                       │
│  Even if a policy passes CI, admission controllers       │
│  provide a second gate before resources are created      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Essential Policies

```yaml
# Policy: No privileged containers
# Policy: No root containers
# Policy: Resource limits required
# Policy: No :latest tags
# Policy: Trusted registries only
# Policy: Required labels (app, team, env)
# Policy: No NodePort services in production
# Policy: Ingress must have TLS
```

---

## Secrets Security Checklist

| Check | Status |
|-------|--------|
| No plaintext secrets in Git | Required |
| Encrypted secrets (SOPS/Sealed Secrets) or external references (ESO) | Required |
| Git credential rotation schedule | Quarterly |
| GitOps operator service account scoped (not cluster-admin) | Recommended |
| Image pull secrets managed via ESO or SOPS | Recommended |
| Audit log for secret access | Required for compliance |

---

## Summary Table

| Security Layer | Controls |
|---------------|----------|
| Git access | Branch protection, CODEOWNERS, signed commits |
| Image supply chain | Scanning (Trivy), signing (Cosign), trusted registries |
| Cluster access | No direct kubectl for devs, GitOps operator handles deploys |
| Operator hardening | Disable admin, SSO, network policies, RBAC |
| Admission control | OPA/Kyverno enforces policies at API level |
| Secrets | SOPS, Sealed Secrets, or ESO — never plaintext |
| Audit | Git history + K8s audit logs = full trail |

---

## Quick Revision Questions

1. **How does GitOps improve security compared to traditional deployments?**
   > Developers don't need direct cluster access (no kubeconfig). All changes go through Git (auditable PRs). The blast radius is reduced to the GitOps operator's permissions.

2. **What is commit signing and why is it important?**
   > GPG/SSH signing proves the commit author's identity. In GitOps, it prevents someone impersonating a developer to push malicious config.

3. **How do you enforce that only signed container images are deployed?**
   > Sign images with Cosign in CI, then use Kyverno or OPA Gatekeeper admission policies to require valid signatures on all Pod images.

4. **What is defense in depth for GitOps policies?**
   > Validate in CI (conftest), enforce at admission (Kyverno/Gatekeeper), and monitor at runtime (Falco/audit logs) — a policy must pass all layers.

5. **Why should the CI pipeline never have direct cluster access in GitOps?**
   > CI only pushes to Git. The GitOps operator handles cluster deployment. This eliminates CI as an attack vector for cluster compromise.

6. **What secrets should never be in plaintext in a GitOps repo?**
   > Database passwords, API keys, TLS certificates, service account tokens, registry credentials — anything sensitive must be encrypted or externally referenced.

---

[← Previous: Testing GitOps](04-testing-gitops.md) | [Back to README](../README.md) | [Next → Monitoring GitOps](06-monitoring-gitops.md)
