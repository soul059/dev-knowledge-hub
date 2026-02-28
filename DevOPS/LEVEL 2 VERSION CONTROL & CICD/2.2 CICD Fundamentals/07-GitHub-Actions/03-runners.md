# Chapter 7.3: GitHub Actions Runners

[← Previous: Actions & Marketplace](02-actions-marketplace.md) | [Back to README](../README.md) | [Next: Secrets & Variables →](04-secrets-variables.md)

---

## Overview

**Runners** are the machines that execute workflow jobs. GitHub provides hosted runners, and you can also set up self-hosted runners for custom requirements.

---

## Runner Types

```
  RUNNER TYPES
  ════════════

  1. GITHUB-HOSTED RUNNERS
     ├── Managed by GitHub
     ├── Fresh VM per job
     ├── Pre-installed tools
     ├── Free tier included
     └── Labels: ubuntu-latest, windows-latest, macos-latest

  2. SELF-HOSTED RUNNERS
     ├── Your own infrastructure
     ├── Persistent (or ephemeral with config)
     ├── Custom hardware / software
     ├── No per-minute cost
     └── Labels: self-hosted + custom labels

  3. LARGER RUNNERS (GitHub Teams/Enterprise)
     ├── 4-64 core machines
     ├── GPU runners
     ├── macOS M1/M2 runners
     └── Static IPs available
```

---

## GitHub-Hosted Runner Specs

| Runner | OS | vCPU | RAM | Disk | Label |
|---|---|---|---|---|---|
| Linux | Ubuntu 22.04 | 4 | 16 GB | 14 GB SSD | `ubuntu-latest` |
| Windows | Windows Server 2022 | 4 | 16 GB | 14 GB SSD | `windows-latest` |
| macOS (Intel) | macOS 14 | 3 | 14 GB | 14 GB SSD | `macos-latest` |
| macOS (ARM) | macOS 14 (M1) | 3 | 7 GB | 14 GB SSD | `macos-latest-xlarge` |

### Pre-installed Software (Ubuntu)

```
  ubuntu-latest INCLUDES:
  ├── Languages: Node.js, Python, Java, Go, Ruby, .NET, Rust
  ├── Package Managers: npm, pip, Maven, Gradle
  ├── Tools: Docker, Docker Compose, kubectl, Helm
  ├── CLIs: AWS CLI, Azure CLI, GitHub CLI
  ├── Databases: SQLite, PostgreSQL client
  ├── Browsers: Chrome, Firefox (for E2E tests)
  └── Build Tools: Make, CMake, GCC
```

---

## Using Runners

```yaml
jobs:
  # GitHub-hosted
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running on Ubuntu"

  build-windows:
    runs-on: windows-latest
    steps:
      - run: echo "Running on Windows"
        shell: pwsh

  build-macos:
    runs-on: macos-latest
    steps:
      - run: echo "Running on macOS"

  # Self-hosted
  deploy:
    runs-on: [self-hosted, linux, gpu]
    steps:
      - run: nvidia-smi

  # Larger runner
  heavy-build:
    runs-on: ubuntu-latest-16-cores
    steps:
      - run: make build -j16
```

---

## Self-Hosted Runner Setup

```
  SETUP FLOW
  ══════════

  1. Settings → Actions → Runners → New self-hosted runner
  2. Download runner application
  3. Configure and register with token
  4. Start the runner service

  REGISTRATION LEVELS:
  ├── Repository level  → Available to one repo
  ├── Organization level → Available to selected repos
  └── Enterprise level   → Available to all orgs
```

### Installation (Linux)

```bash
# Download
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf actions-runner-linux-x64.tar.gz

# Configure (use token from GitHub UI)
./config.sh \
  --url https://github.com/my-org/my-repo \
  --token REGISTRATION_TOKEN \
  --name my-runner \
  --labels linux,docker,gpu \
  --work _work

# Run as service
sudo ./svc.sh install
sudo ./svc.sh start
```

### Ephemeral Self-Hosted Runners

```bash
# Register as ephemeral (auto-deregisters after one job)
./config.sh \
  --url https://github.com/my-org \
  --token TOKEN \
  --ephemeral \
  --labels ephemeral,linux

# Perfect for autoscaling with Kubernetes
```

### Kubernetes-Based Runners (ARC)

```yaml
# Actions Runner Controller (ARC)
# Helm installation
# helm repo add actions-runner-controller \
#   https://actions-runner-controller.github.io/actions-runner-controller
# helm install arc actions-runner-controller/actions-runner-controller

# RunnerDeployment
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: my-runners
spec:
  replicas: 3
  template:
    spec:
      repository: my-org/my-repo
      labels:
        - k8s-runner
      dockerEnabled: true
      resources:
        requests:
          cpu: "2"
          memory: "4Gi"

---
# HorizontalRunnerAutoscaler
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: my-runners-autoscaler
spec:
  scaleTargetRef:
    kind: RunnerDeployment
    name: my-runners
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
      repositoryNames:
        - my-org/my-repo
```

---

## Runner Groups & Labels

```
  RUNNER ORGANIZATION
  ═══════════════════

  Organization
  └── Runner Groups
      ├── "Default" group
      │   ├── ubuntu-runner-1 [linux, docker]
      │   └── ubuntu-runner-2 [linux, docker]
      ├── "GPU" group
      │   └── gpu-runner-1 [linux, gpu, cuda]
      └── "Production" group (restricted)
          └── prod-runner-1 [linux, prod]
              └── Allowed repos: deploy-service only
```

```yaml
# Target specific runner labels
jobs:
  ml-training:
    runs-on: [self-hosted, gpu, cuda]
    steps:
      - run: python train_model.py

  deploy-prod:
    runs-on: [self-hosted, prod]
    steps:
      - run: ./deploy.sh production
```

---

## Cost Optimization

```
  GITHUB-HOSTED PRICING (per minute):
  ════════════════════════════════════

  ┌──────────────┬────────┬──────────────────────┐
  │ Runner       │ Cost   │ Free tier (monthly)  │
  ├──────────────┼────────┼──────────────────────┤
  │ Linux        │ $0.008 │ 2,000 min            │
  │ Windows      │ $0.016 │ 2,000 min (2x rate)  │
  │ macOS        │ $0.08  │ 2,000 min (10x rate) │
  └──────────────┴────────┴──────────────────────┘

  OPTIMIZATION STRATEGIES:
  ├── Use Linux runners when possible (cheapest)
  ├── Cancel redundant runs (concurrency groups)
  ├── Cache dependencies aggressively
  ├── Skip CI with [skip ci] in commit message
  ├── Use path filters to avoid unnecessary runs
  └── Self-hosted for high-volume workloads
```

```yaml
# Cancel redundant runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

## Real-World Scenario

> **Scenario**: A company needs to run ML model training on GPU hardware and standard CI on regular machines.
>
> **Solution**: Use GitHub-hosted `ubuntu-latest` for regular CI (lint, test, build). Set up self-hosted runners with NVIDIA GPUs in the `gpu` runner group, labeled `[self-hosted, linux, gpu]`. ML training workflow targets these GPU runners. Use Actions Runner Controller on Kubernetes for auto-scaling standard runners based on queue length.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Job queued indefinitely | No matching runner available | Check runner labels match `runs-on:` |
| Self-hosted runner offline | Service not running | Check `sudo ./svc.sh status`; restart service |
| "Runner is not allowed" | Runner group restrictions | Add repo to runner group's allowed list |
| Slow builds on self-hosted | Workspace not cleaned | Enable `cleanWs` or use ephemeral runners |
| Docker not available | Docker not installed on runner | Install Docker on runner or use `dockerEnabled: true` (ARC) |
| Runner version outdated | Auto-update disabled | Update runner binary or enable auto-update |

---

## Summary Table

| Aspect | GitHub-Hosted | Self-Hosted |
|---|---|---|
| **Setup** | Zero setup | Manual install & config |
| **Cost** | Per-minute billing | Your infrastructure cost |
| **Maintenance** | Managed by GitHub | You manage updates |
| **Environment** | Fresh VM per job | Persistent (or ephemeral config) |
| **Customization** | Limited | Full control |
| **Security** | Shared infrastructure | Your network / private |
| **Scaling** | Automatic | Manual or ARC (K8s) |

---

## Quick Revision Questions

1. **What are the two main types of runners?** *GitHub-hosted (managed VMs) and self-hosted (your own machines).*
2. **What does `runs-on: ubuntu-latest` do?** *Runs the job on a GitHub-hosted Ubuntu VM with the latest supported version.*
3. **How do you target a self-hosted runner?** *Use labels: `runs-on: [self-hosted, linux, gpu]` — the job runs on a runner matching all labels.*
4. **What is an ephemeral runner?** *A self-hosted runner that automatically deregisters after completing one job — ideal for clean, autoscaled environments.*
5. **What is Actions Runner Controller (ARC)?** *A Kubernetes operator that manages self-hosted runners as pods, with autoscaling based on workflow queue length.*
6. **How do you reduce GitHub Actions costs?** *Use Linux runners, cancel redundant runs with concurrency groups, cache dependencies, use path filters, and self-host for high volume.*

---

[← Previous: Actions & Marketplace](02-actions-marketplace.md) | [Back to README](../README.md) | [Next: Secrets & Variables →](04-secrets-variables.md)
