# Chapter 6.4: Testing GitOps

[← Previous: PR Workflows](03-pr-workflows.md) | [Back to README](../README.md) | [Next → Security Considerations](05-security-considerations.md)

---

## Overview

Testing GitOps configurations prevents broken manifests from reaching your clusters. This chapter covers the **testing pyramid** for GitOps — from static analysis to integration tests — with tools and CI pipeline examples.

---

## GitOps Testing Pyramid

```
┌──────────────────────────────────────────────────────────┐
│              GITOPS TESTING PYRAMID                      │
│                                                          │
│                    ╱╲                                     │
│                   ╱  ╲                                    │
│                  ╱    ╲                                   │
│                 ╱ E2E  ╲          Deploy to ephemeral     │
│                ╱ Tests  ╲         cluster and validate    │
│               ╱──────────╲                                │
│              ╱            ╲                               │
│             ╱  Integration ╲      Dry-run against         │
│            ╱    Tests       ╲     real cluster API        │
│           ╱──────────────────╲                            │
│          ╱                    ╲                           │
│         ╱   Policy Checks      ╲  OPA/Conftest/Kyverno   │
│        ╱   (compliance)         ╲                        │
│       ╱──────────────────────────╲                       │
│      ╱                            ╲                      │
│     ╱   Schema Validation          ╲ kubeconform/kubeval │
│    ╱   (structure)                  ╲                    │
│   ╱──────────────────────────────────╲                   │
│  ╱                                    ╲                  │
│ ╱   Static Analysis (lint + build)     ╲ yamllint,       │
│╱   (syntax)                             ╲ kustomize build│
│──────────────────────────────────────────                │
│                                                          │
│  Fast ←────────────────────────────────→ Thorough        │
│  Run on every PR         Run on staging/pre-prod         │
└──────────────────────────────────────────────────────────┘
```

---

## Layer 1: Static Analysis

### YAML Linting

```yaml
# .yamllint.yaml
extends: default
rules:
  line-length:
    max: 200
  truthy:
    check-keys: false
  comments:
    min-spaces-from-content: 1
  document-start: disable
```

```bash
# Run YAML lint
yamllint -c .yamllint.yaml .
```

### Kustomize / Helm Build

```bash
# Verify Kustomize builds without errors
kustomize build overlays/dev > /dev/null
kustomize build overlays/staging > /dev/null
kustomize build overlays/production > /dev/null

# Verify Helm template renders
helm template my-app charts/my-app \
  -f charts/my-app/values-production.yaml > /dev/null
```

---

## Layer 2: Schema Validation

```bash
# kubeconform: validate against Kubernetes JSON schemas
kustomize build overlays/production | \
  kubeconform \
    -strict \
    -summary \
    -kubernetes-version 1.28.0 \
    -schema-location default \
    -schema-location 'https://raw.githubusercontent.com/datreeio/CRDs-catalog/main/{{.Group}}/{{.ResourceKind}}_{{.ResourceAPIVersion}}.json'

# Output:
# Summary: 15 resources found - Valid: 15, Invalid: 0, Errors: 0, Skipped: 0
```

### Validating CRDs

```bash
# Download CRD schemas for custom resources
# kubeconform supports custom schema locations for:
# - ArgoCD Application
# - Flux GitRepository, Kustomization, HelmRelease
# - cert-manager Certificate
# - etc.

kubeconform \
  -schema-location default \
  -schema-location 'https://raw.githubusercontent.com/datreeio/CRDs-catalog/main/{{.Group}}/{{.ResourceKind}}_{{.ResourceAPIVersion}}.json' \
  manifests/
```

---

## Layer 3: Policy Checks

### Conftest (OPA)

```bash
# Write policies in Rego
# policies/deployment.rego
```

```rego
# policies/deployment.rego
package main

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.containers[_].resources.limits
  msg := sprintf("Deployment %s must have resource limits", [input.metadata.name])
}

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.containers[_].readinessProbe
  msg := sprintf("Deployment %s must have a readiness probe", [input.metadata.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  container.image
  not contains(container.image, ":")
  msg := sprintf("Container %s must use a tagged image", [container.name])
}

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  endswith(container.image, ":latest")
  msg := sprintf("Container %s must not use :latest tag", [container.name])
}
```

```bash
# Run policy checks
kustomize build overlays/production | conftest test -p policies/ -

# Output:
# FAIL - Deployment my-app must have resource limits
# 2 tests, 1 passed, 1 failed
```

### Kyverno CLI (Validate Offline)

```bash
# Validate manifests against Kyverno policies
kyverno apply policies/ --resource manifests/

# Example policy
```

```yaml
# policies/require-labels.yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-app-label
    match:
      resources:
        kinds:
        - Deployment
        - Service
    validate:
      message: "Label 'app' is required"
      pattern:
        metadata:
          labels:
            app: "?*"
```

---

## Layer 4: Integration Tests

### Dry-Run Against Cluster API

```bash
# Server-side dry-run (validates against live cluster)
kustomize build overlays/production | \
  kubectl apply --dry-run=server -f -

# This checks:
# - API server validates the resources
# - Admission webhooks run
# - Resource quotas checked
# - RBAC permissions verified
# But does NOT actually create resources
```

### Diff Against Current State

```bash
# Show what would change
kustomize build overlays/production | kubectl diff -f -

# ArgoCD diff
argocd app diff my-app --refresh
```

---

## Layer 5: E2E Tests

### Ephemeral Cluster Testing

```yaml
# GitHub Actions with Kind cluster
name: E2E Test
on:
  pull_request:
    branches: [main]
    paths:
    - 'overlays/production/**'

jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Create Kind cluster
      uses: helm/kind-action@v1
      with:
        cluster_name: test

    - name: Apply manifests
      run: |
        kustomize build overlays/production | kubectl apply -f -

    - name: Wait for rollout
      run: |
        kubectl rollout status deployment/my-app -n my-app --timeout=120s

    - name: Run smoke tests
      run: |
        kubectl port-forward svc/my-app 8080:80 -n my-app &
        sleep 5
        curl -f http://localhost:8080/health || exit 1

    - name: Cleanup
      if: always()
      run: kind delete cluster --name test
```

---

## Complete CI Pipeline

```yaml
name: GitOps Config Validation
on:
  pull_request:
    branches: [main]

jobs:
  lint:
    name: YAML Lint
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - run: yamllint -c .yamllint.yaml .

  build:
    name: Kustomize Build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [dev, staging, production]
    steps:
    - uses: actions/checkout@v4
    - run: kustomize build overlays/${{ matrix.env }} > /tmp/${{ matrix.env }}.yaml
    - uses: actions/upload-artifact@v4
      with:
        name: manifests-${{ matrix.env }}
        path: /tmp/${{ matrix.env }}.yaml

  validate:
    name: Schema Validation
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [dev, staging, production]
    steps:
    - uses: actions/download-artifact@v4
    - run: kubeconform -strict -summary manifests-${{ matrix.env }}/${{ matrix.env }}.yaml

  policy:
    name: Policy Check
    needs: build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/download-artifact@v4
    - run: conftest test -p policies/ manifests-production/production.yaml
```

---

## Summary Table

| Layer | Tool | What It Checks |
|-------|------|----------------|
| Static analysis | yamllint, kustomize build | Syntax, structure, template rendering |
| Schema validation | kubeconform, kubeval | K8s API conformance, field types |
| Policy checks | conftest (OPA), kyverno CLI | Security, compliance, best practices |
| Integration | kubectl --dry-run=server | API validation, admission webhooks |
| E2E | Kind/k3d + smoke tests | Full deployment and health |

---

## Quick Revision Questions

1. **What is the GitOps testing pyramid?**
   > Layers from fast to thorough: Static analysis → Schema validation → Policy checks → Integration (dry-run) → E2E tests.

2. **What tool validates Kubernetes manifests against API schemas?**
   > `kubeconform` (or `kubeval`) validates manifests against Kubernetes JSON schemas, including CRDs.

3. **How do you check policy compliance in CI?**
   > Use `conftest` with OPA/Rego policies or `kyverno apply` to validate manifests against organizational rules (resource limits, labels, image tags).

4. **What does `kubectl apply --dry-run=server` do?**
   > It sends the manifests to the API server for validation (including admission webhooks) without actually creating resources.

5. **When should E2E tests run?**
   > On PRs that modify production paths, using ephemeral clusters (Kind/k3d) to deploy and validate the full stack.

---

[← Previous: PR Workflows](03-pr-workflows.md) | [Back to README](../README.md) | [Next → Security Considerations](05-security-considerations.md)
