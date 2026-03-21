# Unit 8: Container and Serverless Security — Topic 1: Container Security in Cloud

## Overview

**Container security in cloud** addresses the unique risks of running containerized workloads on cloud platforms. Containers (Docker, OCI) provide lightweight isolation but introduce security challenges across the image supply chain, runtime environment, and orchestration layer. Understanding container-specific threats and cloud-native container security services is essential for securing modern cloud applications.

---

## 1. Container Security Fundamentals

```
CONTAINER THREAT LANDSCAPE:

  ┌─────────────────────────────────────────┐
  │      CONTAINER SECURITY LAYERS          │
  │                                         │
  │  ┌──────────────────────────────────┐   │
  │  │ IMAGE SECURITY                   │   │
  │  │ Base image, vulnerabilities,     │   │
  │  │ secrets, malware, supply chain   │   │
  │  └──────────────────────────────────┘   │
  │  ┌──────────────────────────────────┐   │
  │  │ REGISTRY SECURITY                │   │
  │  │ Access control, scanning,        │   │
  │  │ signing, immutability            │   │
  │  └──────────────────────────────────┘   │
  │  ┌──────────────────────────────────┐   │
  │  │ RUNTIME SECURITY                 │   │
  │  │ Privileges, capabilities,        │   │
  │  │ network, file system             │   │
  │  └──────────────────────────────────┘   │
  │  ┌──────────────────────────────────┐   │
  │  │ ORCHESTRATION SECURITY           │   │
  │  │ K8s API, RBAC, secrets,          │   │
  │  │ network policies, admission      │   │
  │  └──────────────────────────────────┘   │
  │  ┌──────────────────────────────────┐   │
  │  │ HOST SECURITY                    │   │
  │  │ Node OS, kernel, isolation,      │   │
  │  │ patching, monitoring             │   │
  │  └──────────────────────────────────┘   │
  └─────────────────────────────────────────┘

CONTAINER vs VM ISOLATION:
  Feature        | VM            | Container
  Isolation      | Hardware/HV   | Kernel (cgroups/ns)
  Kernel         | Separate      | Shared
  Attack surface | Smaller       | Larger
  Escape risk    | Very low      | Higher
  Performance    | More overhead | Near-native
  Startup        | Minutes       | Seconds
```

---

## 2. Image Security

```
CONTAINER IMAGE SECURITY:

SECURE BASE IMAGES:
  → Use official/verified base images
  → Minimal images (Alpine, distroless, scratch)
  → Pin specific versions (not :latest)
  → Regularly update base images
  → Scan for vulnerabilities before deployment

VULNERABILITY SCANNING:
  AWS ECR:
  → Built-in scanning (Clair-based)
  → Enhanced scanning (Inspector integration)
  → Scan on push
  → Continuous monitoring
  
  # Enable scan on push
  aws ecr put-image-scanning-configuration \
    --repository-name my-app \
    --image-scanning-configuration scanOnPush=true

  Azure ACR:
  → Defender for Containers scanning
  → Qualys-powered vulnerability assessment
  → Scan on push and periodic

  GCP Artifact Registry:
  → Container Analysis (Artifact Analysis)
  → On-demand and automatic scanning
  → Vulnerability findings in SCC

DOCKERFILE BEST PRACTICES:
  # Good Dockerfile
  FROM python:3.11-slim AS builder
  WORKDIR /app
  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt
  
  FROM python:3.11-slim
  RUN groupadd -r appuser && useradd -r -g appuser appuser
  COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
  COPY . .
  USER appuser
  EXPOSE 8080
  CMD ["python", "app.py"]

  Best Practices:
  [ ] Multi-stage builds (smaller images)
  [ ] Run as non-root user
  [ ] No secrets in Dockerfile or image layers
  [ ] Pin dependency versions
  [ ] Minimize installed packages
  [ ] Use .dockerignore
  [ ] Read-only filesystem where possible
  [ ] No unnecessary tools (curl, wget, etc.)

IMAGE SIGNING:
  → Verify image authenticity
  → Ensure images haven't been tampered
  → Cosign (Sigstore) for signing
  → Notary for Docker Content Trust
  → Binary Authorization (GCP)
```

---

## 3. Cloud Container Services

```
MANAGED CONTAINER SERVICES:

AWS:
  → ECS (Elastic Container Service)
  → EKS (Elastic Kubernetes Service)
  → Fargate (Serverless containers)
  → ECR (Container Registry)
  → App Runner

AZURE:
  → AKS (Azure Kubernetes Service)
  → Azure Container Instances
  → Azure Container Apps
  → ACR (Azure Container Registry)

GCP:
  → GKE (Google Kubernetes Engine)
  → Cloud Run (Serverless containers)
  → Artifact Registry
  → GKE Autopilot

SECURITY FEATURES BY PROVIDER:
  Feature              | AWS EKS     | Azure AKS    | GCP GKE
  Image scanning       | ECR/Inspector| Defender     | Artifact Analysis
  Runtime protection   | GuardDuty   | Defender     | Container Threat Det
  Admission control    | OPA         | Azure Policy | Binary Authorization
  Secrets              | Secrets Mgr | Key Vault    | Secret Manager
  Network policy       | Calico      | Calico/Azure | Calico/Dataplane V2
  Service mesh         | App Mesh    | OSM/Istio    | Anthos SM/Istio
  Node security        | Bottlerocket| Ubuntu/CBL   | Container-Optimized

RUNTIME SECURITY:
  → No privileged containers
  → Drop all capabilities, add only needed
  → Read-only root filesystem
  → No host network/PID/IPC namespace
  → Resource limits (CPU, memory)
  → Seccomp profiles
  → AppArmor/SELinux policies
  → No container-to-host volume mounts (sensitive)
```

---

## 4. Container Runtime Protection

```
RUNTIME SECURITY CONTROLS:

KUBERNETES POD SECURITY:
  # Pod Security Standards
  apiVersion: v1
  kind: Pod
  metadata:
    name: secure-pod
  spec:
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      fsGroup: 2000
      seccompProfile:
        type: RuntimeDefault
    containers:
    - name: app
      image: myapp:1.0
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
      resources:
        limits:
          cpu: "500m"
          memory: "256Mi"
        requests:
          cpu: "250m"
          memory: "128Mi"

CLOUD RUNTIME PROTECTION:
  AWS GuardDuty EKS Runtime Monitoring:
  → Detect suspicious container behavior
  → Crypto mining detection
  → Reverse shell detection
  → Container escape attempts
  
  Azure Defender for Containers:
  → Runtime threat detection
  → Anomalous process execution
  → Suspicious file operations
  → Network anomalies
  
  GCP Container Threat Detection:
  → Binary execution monitoring
  → Reverse shell detection
  → Script execution monitoring
  → Library loading anomalies

THIRD-PARTY TOOLS:
  → Falco: Runtime security monitoring (open source)
  → Sysdig: Container visibility and security
  → Aqua Security: Full lifecycle container security
  → Prisma Cloud: Cloud workload protection
  → StackRox/Red Hat ACS: Kubernetes security
```

---

## 5. Assessment

```
CONTAINER SECURITY ASSESSMENT:

CHECKLIST:
  [ ] Base images are minimal and updated
  [ ] Image scanning enabled (scan on push)
  [ ] No critical/high vulnerabilities in production
  [ ] Images signed and verified
  [ ] Containers run as non-root
  [ ] Privileged containers prohibited
  [ ] Resource limits set on all containers
  [ ] Network policies restrict pod communication
  [ ] Secrets not stored in images or env vars
  [ ] Runtime protection enabled
  [ ] Container logs collected centrally
  [ ] Admission control policies enforced

COMMON FINDINGS:
  Finding                          | Risk
  Running as root                  | High
  Privileged containers            | Critical
  No image scanning                | High
  Base image with known CVEs       | High
  Secrets in environment variables | High
  No network policies              | Medium
  No resource limits               | Medium
  Using :latest tag                | Medium
  No runtime monitoring            | High

TOOLS:
  → Trivy: Image vulnerability scanner
  → Checkov: IaC security scanner
  → kube-bench: CIS K8s benchmark
  → kube-hunter: K8s penetration testing
  → Falco: Runtime security
  → kubeaudit: Kubernetes audit tool
```

---

## Summary Table

| Layer | Threat | Control | Tools |
|-------|--------|---------|-------|
| Image | Vulnerabilities | Scanning, signing | Trivy, ECR, ACR |
| Registry | Unauthorized access | RBAC, scanning | ECR, ACR, AR |
| Runtime | Privilege escalation | Pod security, capabilities | Falco, GuardDuty |
| Orchestration | API abuse, RBAC | Admission control, RBAC | OPA, kube-bench |
| Host | Kernel exploit | Hardened OS, patching | Bottlerocket, COS |

---

## Revision Questions

1. What are the five layers of container security?
2. Why should containers run as non-root users?
3. How do cloud providers handle container image scanning?
4. What Kubernetes pod security settings should be enforced?
5. What runtime threats can cloud-native detection services identify?

---

*Previous: None (First topic in this unit) | Next: [02-kubernetes-security.md](02-kubernetes-security.md)*

---

*[Back to README](../README.md)*
