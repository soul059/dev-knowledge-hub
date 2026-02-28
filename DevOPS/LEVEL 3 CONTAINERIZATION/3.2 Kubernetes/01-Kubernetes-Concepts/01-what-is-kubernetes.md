# Unit 1: Kubernetes Concepts

[← Back to README](../README.md) | [Next: Kubernetes Architecture →](../02-Kubernetes-Architecture/01-architecture-overview.md)

---

## Chapter Overview

This unit introduces the fundamental concepts behind Kubernetes — what it is, why container orchestration matters, its key features, major distributions, and the broader Cloud Native ecosystem. By the end, you will understand the **"why"** behind Kubernetes and be able to compare different distributions for various use cases.

---

## 1.1 What is Kubernetes?

**Kubernetes** (commonly abbreviated as **K8s** — the "8" represents the eight letters between "K" and "s") is an open-source container orchestration platform originally developed by Google, based on their internal system called **Borg**. It was donated to the **Cloud Native Computing Foundation (CNCF)** in 2015.

### Core Definition

> Kubernetes is a portable, extensible, open-source platform for **managing containerized workloads and services**, that facilitates both **declarative configuration** and **automation**.

### Key Idea

```
┌──────────────────────────────────────────────────────────────┐
│                     WITHOUT KUBERNETES                       │
│                                                              │
│   Developer          Ops Team           Servers              │
│   ┌──────┐          ┌──────┐          ┌──────┐              │
│   │ Build │──deploy──│Manual│──SSH────▶│ VM 1 │              │
│   │ Image │          │Config│──SSH────▶│ VM 2 │              │
│   └──────┘          └──────┘──SSH────▶│ VM 3 │              │
│                                        └──────┘              │
│   ❌ Manual scaling    ❌ No self-healing    ❌ Downtime      │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                      WITH KUBERNETES                         │
│                                                              │
│   Developer          Kubernetes          Cluster             │
│   ┌──────┐          ┌──────────┐        ┌──────┐            │
│   │ Push │──YAML───▶│ Control  │───────▶│Node 1│            │
│   │ Code │          │  Plane   │───────▶│Node 2│            │
│   └──────┘          │(Desired  │───────▶│Node 3│            │
│                     │ State)   │        └──────┘            │
│                     └──────────┘                             │
│   ✅ Auto-scaling   ✅ Self-healing     ✅ Zero-downtime     │
└──────────────────────────────────────────────────────────────┘
```

### How Kubernetes Works — The Declarative Model

Instead of telling Kubernetes **how** to do something (imperative), you tell it **what** you want (declarative):

```yaml
# You declare: "I want 3 replicas of my web app"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3           # Desired state
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Kubernetes continuously **reconciles** the actual state with the desired state:

```
     DESIRED STATE                    ACTUAL STATE
     (Your YAML)                     (Running Pods)
   ┌──────────────┐               ┌──────────────┐
   │ replicas: 3  │               │  2 pods       │
   │ image: v2    │◄──reconcile──▶│  image: v1    │
   └──────────────┘               └──────────────┘
         │                               │
         └───── K8s sees difference ─────┘
                       │
              ┌────────▼────────┐
              │ Creates 1 pod   │
              │ Updates to v2   │
              └─────────────────┘
```

---

## Why Orchestration?

### The Problem: Managing Containers at Scale

Docker solved **packaging** — but in production, you face much bigger challenges:

```
┌─────────────────────────────────────────────────────────────┐
│              CHALLENGES WITHOUT ORCHESTRATION                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. SCALING          "How do I run 100 containers           │
│     Challenge         across 20 servers?"                   │
│                                                             │
│  2. FAILURE          "A server crashed — who restarts       │
│     Recovery          the 15 containers it was running?"    │
│                                                             │
│  3. NETWORKING       "How do containers on different        │
│     Complexity        hosts communicate?"                   │
│                                                             │
│  4. DEPLOYMENT       "How do I update without downtime?"    │
│     Strategy                                                │
│                                                             │
│  5. RESOURCE         "Which server has enough CPU/RAM       │
│     Management        for this container?"                  │
│                                                             │
│  6. SERVICE          "How does container A find             │
│     Discovery         container B?"                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### The Evolution

```
  Bare Metal          Virtual Machines        Containers         Orchestration
  (2000s)              (2005-2015)           (2013+)              (2015+)
 ┌─────────┐         ┌─────────────┐      ┌───────────┐       ┌──────────────┐
 │ App 1   │         │┌───────────┐│      │ ┌───┐┌───┐│       │ Kubernetes   │
 │ App 2   │         ││ VM1: App1 ││      │ │C1 ││C2 ││       │ manages      │
 │ App 3   │         ││ VM2: App2 ││      │ └───┘└───┘│       │ 1000s of     │
 │ on one  │         ││ VM3: App3 ││      │ ┌───┐┌───┐│       │ containers   │
 │ server  │         │└───────────┘│      │ │C3 ││C4 ││       │ across nodes │
 └─────────┘         └─────────────┘      │ └───┘└───┘│       └──────────────┘
                                          └───────────┘
  ❌ Conflicts        ✅ Isolation          ✅ Lightweight       ✅ Automated
  ❌ Waste            ❌ Heavy (~GB)        ✅ Fast (~MB)        ✅ Self-healing
  ❌ Single point     ✅ Better util.       ❌ Manual mgmt       ✅ Scalable
```

### What an Orchestrator Does

| Capability | Description |
|-----------|-------------|
| **Scheduling** | Places containers on appropriate nodes based on resources |
| **Self-healing** | Restarts failed containers, replaces unhealthy ones |
| **Scaling** | Adds/removes container instances based on load |
| **Load balancing** | Distributes traffic across container replicas |
| **Rolling updates** | Updates applications without downtime |
| **Service discovery** | Enables containers to find each other |
| **Secret management** | Manages sensitive configuration securely |
| **Storage orchestration** | Mounts storage systems automatically |

---

## Kubernetes Features

### Core Features at a Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│                     KUBERNETES FEATURES                             │
├──────────────────────┬──────────────────────────────────────────────┤
│                      │                                              │
│  ┌─────────────────┐ │  ┌──────────────────┐  ┌─────────────────┐  │
│  │  Self-Healing   │ │  │  Auto-Scaling     │  │  Load Balancing │  │
│  │  • Restart pods │ │  │  • HPA            │  │  • Services     │  │
│  │  • Reschedule   │ │  │  • VPA            │  │  • Ingress      │  │
│  │  • Health check │ │  │  • Cluster Auto   │  │  • kube-proxy   │  │
│  └─────────────────┘ │  └──────────────────┘  └─────────────────┘  │
│                      │                                              │
│  ┌─────────────────┐ │  ┌──────────────────┐  ┌─────────────────┐  │
│  │ Rolling Updates │ │  │  Storage Orch.    │  │  Config Mgmt    │  │
│  │ • Zero-downtime │ │  │  • PV / PVC      │  │  • ConfigMaps   │  │
│  │ • Rollbacks     │ │  │  • StorageClass   │  │  • Secrets      │  │
│  │ • Canary        │ │  │  • CSI drivers    │  │  • Env vars     │  │
│  └─────────────────┘ │  └──────────────────┘  └─────────────────┘  │
│                      │                                              │
│  ┌─────────────────┐ │  ┌──────────────────┐  ┌─────────────────┐  │
│  │  Networking     │ │  │  Security        │  │  Extensibility  │  │
│  │  • Pod-to-Pod   │ │  │  • RBAC          │  │  • CRDs         │  │
│  │  • DNS          │ │  │  • NetworkPolicy │  │  • Operators    │  │
│  │  • Ingress      │ │  │  • Pod Security  │  │  • Webhooks     │  │
│  └─────────────────┘ │  └──────────────────┘  └─────────────────┘  │
│                      │                                              │
└──────────────────────┴──────────────────────────────────────────────┘
```

### Feature Deep Dive

#### 1. Self-Healing
Kubernetes automatically restarts containers that fail, replaces containers, kills containers that don't respond to health checks, and doesn't advertise them to clients until they are ready.

```bash
# Example: If a pod crashes, K8s restarts it automatically
$ kubectl get pods -w
NAME         READY   STATUS    RESTARTS   AGE
web-app-1    1/1     Running   0          5m
web-app-1    0/1     Error     0          6m     # Pod crashes
web-app-1    1/1     Running   1          6m     # K8s auto-restarts
```

#### 2. Horizontal Scaling
```bash
# Scale manually
kubectl scale deployment web-app --replicas=10

# Or auto-scale based on CPU
kubectl autoscale deployment web-app --min=3 --max=10 --cpu-percent=80
```

#### 3. Rolling Updates & Rollbacks
```bash
# Update image (rolling update - zero downtime)
kubectl set image deployment/web-app web=nginx:1.26

# Rollback if something goes wrong
kubectl rollout undo deployment/web-app

# Check rollout status
kubectl rollout status deployment/web-app
```

#### 4. Service Discovery & Load Balancing
```
┌────────────────────────────────────────┐
│            Kubernetes Service          │
│          (web-app-service)             │
│           10.96.0.100:80               │
│                                        │
│      ┌──────────┬──────────┐           │
│      ▼          ▼          ▼           │
│  ┌───────┐ ┌───────┐ ┌───────┐        │
│  │Pod 1  │ │Pod 2  │ │Pod 3  │        │
│  │:8080  │ │:8080  │ │:8080  │        │
│  └───────┘ └───────┘ └───────┘        │
│                                        │
│  DNS: web-app-service.default.svc      │
└────────────────────────────────────────┘
```

#### 5. Declarative Configuration
Everything in Kubernetes is defined as YAML/JSON objects. The desired state is stored in **etcd**, and Kubernetes controllers continuously work to match actual state to desired state.

---

## Kubernetes Distributions

Kubernetes is **open-source**, but many vendors offer their own managed or customized distributions:

### Distribution Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│                  KUBERNETES DISTRIBUTIONS                        │
├──────────┬──────────┬──────────┬──────────┬──────────┬──────────┤
│ Vanilla  │   EKS    │   AKS    │   GKE    │OpenShift │  k3s     │
│  (OSS)   │ (AWS)    │ (Azure)  │ (Google) │(Red Hat) │(Rancher) │
├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ Full     │ Managed  │ Managed  │ Managed  │Enterprise│ Light-   │
│ control  │ control  │ control  │ control  │ platform │ weight   │
│          │ plane    │ plane    │ plane    │          │          │
│ DIY      │ AWS      │ Azure    │ GCP      │ Any      │ IoT /   │
│ setup    │ integr.  │ integr.  │ integr.  │ cloud    │ Edge     │
│          │          │          │          │          │          │
│ Free     │ $0.10/hr │ Free CP  │ Free CP  │ License  │ Free     │
│          │ per      │ per      │(Autopilot│ fee      │          │
│          │ cluster  │ cluster  │ charged) │          │          │
└──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

### Detailed Comparison

| Feature | Vanilla K8s | EKS (AWS) | AKS (Azure) | GKE (Google) | OpenShift |
|---------|-------------|-----------|-------------|-------------|-----------|
| **Setup** | Manual (kubeadm) | CLI/Console | CLI/Portal | CLI/Console | Installer |
| **Control Plane** | Self-managed | AWS-managed | Azure-managed | Google-managed | Self/managed |
| **Upgrades** | Manual | Semi-auto | Semi-auto | Auto | Semi-auto |
| **Networking** | Choose CNI | VPC CNI | Azure CNI | GKE CNI | OpenShift SDN |
| **Registry** | External | ECR | ACR | GCR/Artifact | Internal |
| **CI/CD** | External | CodePipeline | DevOps | Cloud Build | Built-in |
| **Cost** | Infra only | + $72/mo/cluster | Free CP | Free CP* | License + Infra |
| **Best For** | Learning, full control | AWS shops | Azure shops | K8s-native orgs | Enterprise |

### When to Choose What?

```
"I want full control and I'm learning"           → Vanilla (kubeadm / minikube)
"We're an AWS shop, need production K8s"          → EKS
"We use Azure for everything"                     → AKS
"We want the best K8s experience (GCP origin)"    → GKE
"We need enterprise security + developer portal"  → OpenShift
"I need K8s on a Raspberry Pi / edge device"      → k3s
"I want local multi-node for testing"             → kind
```

---

## The CNCF Ecosystem

The **Cloud Native Computing Foundation (CNCF)** is the home of Kubernetes and 150+ related projects that form the cloud-native ecosystem.

### CNCF Landscape Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                        CNCF LANDSCAPE                              │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  PROVISIONING          RUNTIME              ORCHESTRATION          │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐       │
│  │ Terraform    │     │ containerd   │     │ KUBERNETES   │       │
│  │ Pulumi       │     │ CRI-O        │     │ (Graduated)  │       │
│  │ Crossplane   │     │ gVisor       │     │              │       │
│  └──────────────┘     └──────────────┘     └──────────────┘       │
│                                                                    │
│  NETWORKING            OBSERVABILITY        SECURITY               │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐       │
│  │ Envoy        │     │ Prometheus   │     │ Falco        │       │
│  │ CoreDNS      │     │ Grafana      │     │ OPA          │       │
│  │ Cilium       │     │ Jaeger       │     │ cert-manager │       │
│  │ Istio        │     │ OpenTelemetry│     │ Vault        │       │
│  └──────────────┘     └──────────────┘     └──────────────┘       │
│                                                                    │
│  APP DEFINITION        STORAGE             CI/CD                   │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐       │
│  │ Helm         │     │ Rook         │     │ ArgoCD       │       │
│  │ Kustomize    │     │ Longhorn     │     │ Flux         │       │
│  │ Operator Fwk │     │ MinIO        │     │ Tekton       │       │
│  └──────────────┘     └──────────────┘     └──────────────┘       │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### CNCF Project Maturity Levels

```
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │   SANDBOX    │────▶│  INCUBATING  │────▶│  GRADUATED   │
  │              │     │              │     │              │
  │  Early stage │     │  Growing     │     │  Production  │
  │  Experimental│     │  adoption    │     │  ready       │
  │              │     │              │     │              │
  │  Examples:   │     │  Examples:   │     │  Examples:   │
  │  • Backstage │     │  • Cilium    │     │  • Kubernetes│
  │  • Dapr      │     │  • Argo      │     │  • Prometheus│
  │  • Keptn     │     │  • gRPC      │     │  • Envoy     │
  │              │     │  • Knative   │     │  • CoreDNS   │
  └──────────────┘     └──────────────┘     │  • Helm      │
                                            │  • etcd      │
                                            └──────────────┘
```

### Key CNCF Projects You Should Know

| Project | Category | Purpose |
|---------|----------|---------|
| **Kubernetes** | Orchestration | Container orchestration |
| **Prometheus** | Monitoring | Metrics collection and alerting |
| **Envoy** | Networking | Service proxy / data plane |
| **CoreDNS** | Networking | DNS server for K8s service discovery |
| **etcd** | Storage | Distributed key-value store (K8s state) |
| **Helm** | App Definition | Package manager for K8s |
| **containerd** | Runtime | Container runtime |
| **Istio** | Service Mesh | Traffic management, security, observability |
| **Cilium** | Networking | eBPF-based CNI plugin |
| **ArgoCD** | CI/CD | GitOps continuous delivery |
| **OpenTelemetry** | Observability | Unified telemetry collection |
| **Falco** | Security | Runtime security monitoring |

---

## Real-World Scenario: Why a Company Adopts Kubernetes

### Scenario: E-commerce Platform Migration

```
BEFORE (Monolith on VMs)                AFTER (Microservices on K8s)
┌─────────────────────┐                 ┌──────────────────────────┐
│   Load Balancer     │                 │     Ingress Controller   │
│         │           │                 │           │              │
│   ┌─────▼─────┐     │                 │  ┌────────▼────────┐     │
│   │ Monolith  │     │                 │  │  API Gateway     │     │
│   │ App       │     │                 │  └────────┬────────┘     │
│   │ (8 GB RAM)│     │                 │      ┌────┼────┐        │
│   │           │     │                 │  ┌───▼┐┌──▼─┐┌─▼──┐    │
│   │ - Auth    │     │     ──────▶     │  │Auth││Cart ││Prod│    │
│   │ - Cart    │     │                 │  │x3  ││x5   ││x4  │    │
│   │ - Product │     │                 │  └────┘└─────┘└────┘    │
│   │ - Payment │     │                 │  ┌────┐┌─────┐          │
│   │ - Search  │     │                 │  │Pay ││Srch │          │
│   └───────────┘     │                 │  │x2  ││x3   │          │
│                     │                 │  └────┘└─────┘          │
│ ❌ Scale everything │                 │ ✅ Scale independently  │
│ ❌ Single failure   │                 │ ✅ Fault isolation      │
│ ❌ Deploy all or    │                 │ ✅ Independent deploy   │
│    nothing          │                 │ ✅ Team autonomy        │
└─────────────────────┘                 └──────────────────────────┘
```

**Results after migration:**
- 99.99% uptime (from 99.9%)
- Deployments from 1/week → 50/day
- 40% reduction in infrastructure cost (better utilization)
- Auto-scaling handles Black Friday traffic spikes

---

## Troubleshooting Tips

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| `kubectl` not connecting | Wrong kubeconfig | Check `~/.kube/config` or `$KUBECONFIG` |
| Cluster unreachable | API server down | Check control plane node, systemd services |
| Cannot pull images | Registry auth / network | Check `imagePullSecrets`, network policies |
| Pods stuck in `Pending` | No resources | Check node capacity: `kubectl describe nodes` |
| Version mismatch | kubectl vs cluster skew | Keep kubectl within ±1 minor version of cluster |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Kubernetes** | Open-source container orchestration platform from Google/CNCF |
| **Declarative Model** | You define desired state; K8s reconciles actual → desired |
| **Orchestration** | Automates scheduling, scaling, healing, networking, updates |
| **Self-healing** | Auto-restart, reschedule, replace failed containers |
| **Distributions** | Vanilla, EKS, AKS, GKE, OpenShift, k3s — choose by needs |
| **CNCF** | Foundation governing Kubernetes and 150+ cloud-native projects |
| **Key Features** | Scaling, load balancing, rolling updates, config management |
| **Maturity Levels** | Sandbox → Incubating → Graduated |

---

## Quick Revision Questions

1. **What does the "8" in K8s stand for?**
   <details><summary>Answer</summary>The 8 letters between "K" and "s" in "Kubernetes".</details>

2. **Explain the difference between imperative and declarative approaches in Kubernetes.**
   <details><summary>Answer</summary>Imperative: You tell K8s exact steps ("run this container, expose this port"). Declarative: You define the desired end state in YAML, and K8s figures out how to get there. Declarative is preferred because it's version-controllable, reproducible, and self-documenting.</details>

3. **Name three problems that container orchestration solves.**
   <details><summary>Answer</summary>1) Scaling containers across multiple hosts, 2) Automatic failure recovery (self-healing), 3) Service discovery and load balancing. Others include: rolling updates, resource management, secret management.</details>

4. **When would you choose GKE over EKS?**
   <details><summary>Answer</summary>GKE is built by the creators of Kubernetes and often has the latest features first. Choose GKE if you're on GCP, want the most native K8s experience, or prefer features like Autopilot mode. Choose EKS if your infrastructure is already on AWS.</details>

5. **What are the three CNCF project maturity levels?**
   <details><summary>Answer</summary>Sandbox (early/experimental), Incubating (growing adoption), and Graduated (production-ready, widely adopted).</details>

6. **What is the reconciliation loop in Kubernetes?**
   <details><summary>Answer</summary>Kubernetes controllers continuously compare the desired state (from YAML specs stored in etcd) with the actual state of the cluster. If there's a difference, the controller takes action to bring the actual state in line with the desired state. This is the core mechanism behind self-healing and declarative management.</details>

---

[← Back to README](../README.md) | [Next: Kubernetes Architecture →](../02-Kubernetes-Architecture/01-architecture-overview.md)
