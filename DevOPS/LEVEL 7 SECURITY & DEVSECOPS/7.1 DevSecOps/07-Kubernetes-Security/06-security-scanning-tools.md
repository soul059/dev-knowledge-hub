# Chapter 7.6: Kubernetes Security Scanning Tools

## Overview

Kubernetes security scanning tools check cluster configuration, YAML manifests, and running workloads for misconfigurations, vulnerabilities, and policy violations. This chapter covers kubescape, kube-bench, Trivy for K8s, and Polaris — the core toolkit for K8s security posture management.

---

## Tool Landscape

```
K8S SECURITY SCANNING TOOLS:

  ┌─────────────────────────────────────────────────────────┐
  │                                                           │
  │  PRE-DEPLOY (CI/CD)           IN-CLUSTER (Runtime)       │
  │  ┌─────────────────┐         ┌─────────────────┐         │
  │  │ Manifest Scan   │         │ Cluster Scan    │         │
  │  │ ┌─────────────┐ │         │ ┌─────────────┐ │         │
  │  │ │ kubescape   │ │         │ │ kube-bench  │ │         │
  │  │ │ Trivy       │ │         │ │ kubescape   │ │         │
  │  │ │ Polaris     │ │         │ │ kubeaudit   │ │         │
  │  │ │ Checkov     │ │         │ │ Trivy K8s   │ │         │
  │  │ └─────────────┘ │         │ └─────────────┘ │         │
  │  │                 │         │                 │         │
  │  │ Helm Chart Scan │         │ Compliance      │         │
  │  │ ┌─────────────┐ │         │ ┌─────────────┐ │         │
  │  │ │ kubescape   │ │         │ │ CIS Bench   │ │         │
  │  │ │ Checkov     │ │         │ │ NSA/CISA    │ │         │
  │  │ │ Pluto       │ │         │ │ MITRE ATT&CK│ │         │
  │  │ └─────────────┘ │         │ └─────────────┘ │         │
  │  └─────────────────┘         └─────────────────┘         │
  └─────────────────────────────────────────────────────────┘
```

---

## Kubescape

```bash
# Install kubescape
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | bash

# Scan cluster with NSA/CISA framework
kubescape scan framework nsa --submit

# Scan with CIS Benchmark
kubescape scan framework cis-v1.23-t1.0.1

# Scan with MITRE ATT&CK
kubescape scan framework mitre

# Scan specific YAML manifests
kubescape scan *.yaml

# Scan Helm chart
kubescape scan helm charts/myapp/

# Scan with specific controls
kubescape scan control "Privileged container" "HostPath mount"

# JSON output for CI/CD
kubescape scan framework nsa \
    --format json \
    --output results.json \
    --compliance-threshold 80   # Fail if score < 80%
```

### Kubescape in CI/CD

```yaml
# GitHub Actions
name: K8s Security Scan
on:
  pull_request:
    paths:
      - 'k8s/**'
      - 'charts/**'

jobs:
  kubescape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Kubescape Scan
        uses: kubescape/github-action@main
        with:
          files: "k8s/*.yaml"
          framework: nsa,mitre
          complianceThreshold: 75
          format: sarif
          outputFile: kubescape.sarif

      - name: Upload Results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: kubescape.sarif
```

---

## kube-bench (CIS Benchmark)

```bash
# Run kube-bench checks
# On master node
kube-bench run --targets=master

# On worker node
kube-bench run --targets=node

# Specific checks
kube-bench run --check=1.1.1,1.1.2,1.2.1

# EKS-specific
kube-bench run --benchmark=eks-1.2.0

# JSON output
kube-bench run --json > kube-bench-results.json

# As Kubernetes Job
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs job/kube-bench

# Example output:
# [INFO] 1 Control Plane Security Configuration
# [INFO] 1.1 Control Plane Node Configuration Files
# [PASS] 1.1.1 Ensure API server pod specification file permissions are set to 644
# [FAIL] 1.1.2 Ensure API server pod specification file ownership is set to root:root
# [PASS] 1.1.3 Ensure controller manager pod specification file permissions are set
# ...
# == Summary total ==
# 42 checks PASS
# 8 checks FAIL
# 12 checks WARN
# 0 checks INFO
```

---

## Trivy for Kubernetes

```bash
# Scan entire cluster
trivy k8s --report summary cluster

# Detailed vulnerability report
trivy k8s --report all --severity HIGH,CRITICAL -o results.json cluster

# Scan specific namespace
trivy k8s --namespace production --report summary cluster

# Scan specific workload
trivy k8s --report all deployment/myapp -n production

# Compliance scan
trivy k8s --compliance k8s-cis --report summary cluster
trivy k8s --compliance k8s-nsa --report summary cluster

# Generate SBOM for all images in cluster
trivy k8s --format cyclonedx --output cluster-sbom.json cluster

# CI/CD: Scan manifests before deploy
trivy config k8s/
trivy config --severity HIGH,CRITICAL --exit-code 1 k8s/
```

### Trivy Operator (Continuous Scanning)

```bash
# Install Trivy Operator
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm install trivy-operator aqua/trivy-operator \
    -n trivy-system --create-namespace \
    --set "trivy.severity=HIGH\,CRITICAL"

# Check vulnerability reports
kubectl get vulnerabilityreports -A -o wide

# Check config audit reports
kubectl get configauditreports -A

# Example: Get vulnerabilities for a specific deployment
kubectl get vulnerabilityreports -n production \
    -l trivy-operator.resource.name=myapp -o yaml
```

---

## Polaris

```bash
# Install Polaris
brew install fairwindsops/tap/polaris

# Audit cluster
polaris audit --format=pretty

# Audit specific YAML
polaris audit --audit-path k8s/ --format=pretty

# Dashboard (web UI)
polaris dashboard --port 8080

# JSON output for CI/CD
polaris audit --format=json --output-file polaris-results.json

# CI/CD with score threshold
polaris audit --audit-path k8s/ --set-exit-code-on-danger --set-exit-code-below-score 80
```

```yaml
# Custom Polaris config
checks:
  # Security
  hostIPCSet: danger
  hostPIDSet: danger
  hostNetworkSet: danger
  privilegeEscalationAllowed: danger
  runAsRootAllowed: danger
  runAsPrivileged: danger
  insecureCapabilities: danger

  # Best practices
  cpuRequestsMissing: warning
  memoryRequestsMissing: warning
  cpuLimitsMissing: warning
  memoryLimitsMissing: warning
  readinessProbeMissing: warning
  livenessProbeMissing: warning
  tagNotSpecified: danger
  pullPolicyNotAlways: warning

exemptions:
  - namespace: kube-system
    controllerNames:
      - kube-proxy
      - coredns
    rules:
      - hostNetworkSet
      - runAsRootAllowed
```

---

## Tool Comparison

| Tool | Scans | Frameworks | CI/CD | In-Cluster | SARIF |
|------|-------|-----------|-------|------------|-------|
| **kubescape** | Manifests + Cluster | NSA, CIS, MITRE | ✓ | ✓ | ✓ |
| **kube-bench** | Cluster config only | CIS only | ✓ | ✓ (Job) | ✗ |
| **Trivy K8s** | Images + Config + Cluster | CIS, NSA | ✓ | ✓ (Operator) | ✓ |
| **Polaris** | Manifests + Cluster | Best practices | ✓ | ✓ (Dashboard) | ✗ |
| **Checkov** | IaC + K8s manifests | CIS, custom | ✓ | ✗ | ✓ |

---

## Unified Scanning Strategy

```yaml
# Complete K8s security scanning pipeline
name: Kubernetes Security
on:
  pull_request:
    paths: ['k8s/**', 'charts/**']

jobs:
  # 1. Manifest scanning (pre-deploy)
  manifest-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Kubescape
        uses: kubescape/github-action@main
        with:
          files: "k8s/*.yaml"
          framework: nsa
          complianceThreshold: 75
      - name: Trivy Config
        run: |
          trivy config --exit-code 1 --severity HIGH,CRITICAL k8s/
      - name: Polaris
        run: |
          polaris audit --audit-path k8s/ --set-exit-code-on-danger

  # 2. Image scanning (build time)
  image-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: docker build -t myapp:${{ github.sha }} .
      - name: Trivy Image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          severity: HIGH,CRITICAL
          exit-code: 1

  # 3. In-cluster scanning (post-deploy, scheduled)
  # Run kube-bench + kubescape as CronJobs
  # Run Trivy Operator continuously
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| kubescape score is very low | Many default config issues | Prioritize `danger` level findings; fix incrementally |
| kube-bench can't access config files | Running outside cluster | Run as K8s Job with hostPath mounts to /etc/kubernetes |
| Trivy Operator not generating reports | Insufficient RBAC | Check operator service account permissions; verify installation |
| Polaris flags all pods | No resource limits set | Add CPU/memory requests and limits to all containers |
| False positives on system components | Scanning kube-system pods | Exempt kube-system namespace in tool configuration |

---

## Summary Table

| Tool | Best For |
|------|---------|
| **kubescape** | Multi-framework compliance scanning (NSA, MITRE, CIS) |
| **kube-bench** | CIS Kubernetes Benchmark verification (node-level) |
| **Trivy K8s** | All-in-one: vulnerabilities + misconfig + SBOM |
| **Trivy Operator** | Continuous in-cluster vulnerability monitoring |
| **Polaris** | Best-practice validation with dashboard |

---

## Quick Revision Questions

1. **What frameworks does kubescape support?**
   > kubescape supports NSA/CISA Kubernetes Hardening Guide, CIS Kubernetes Benchmark, MITRE ATT&CK for Containers, and custom frameworks. It can scan both live clusters and YAML manifests, making it useful for both CI/CD and runtime compliance checking.

2. **What is the difference between kube-bench and kubescape?**
   > kube-bench checks node-level configuration against the CIS Kubernetes Benchmark — it inspects API server flags, kubelet config, file permissions. kubescape checks workload-level security — pod configurations, RBAC, network policies, image settings. They complement each other.

3. **How does Trivy Operator provide continuous scanning?**
   > Trivy Operator runs as a controller in the cluster. It automatically scans new workloads when they're deployed and periodically rescans existing ones. It creates CRDs (VulnerabilityReport, ConfigAuditReport) that can be queried with kubectl, integrated with dashboards, or used in admission policies.

4. **What is the recommended K8s security scanning strategy?**
   > Three layers: (1) Pre-deploy: scan manifests with kubescape/Trivy config in CI/CD, (2) Build-time: scan images with Trivy, (3) Runtime: run Trivy Operator continuously and kube-bench periodically. This covers misconfigurations, vulnerabilities, and CIS compliance.

5. **Why use multiple scanning tools instead of just one?**
   > Each tool has different strengths. kube-bench excels at node config. kubescape covers multiple compliance frameworks. Trivy scans both images and configs. Polaris focuses on best practices. Using multiple tools catches more issues and provides defense in depth.

---

[← Previous: 7.5 Kubernetes Secrets Management](05-k8s-secrets-management.md) | [Next: 7.7 Runtime Protection with Falco →](07-runtime-protection-falco.md)

[Back to Table of Contents](../README.md)
