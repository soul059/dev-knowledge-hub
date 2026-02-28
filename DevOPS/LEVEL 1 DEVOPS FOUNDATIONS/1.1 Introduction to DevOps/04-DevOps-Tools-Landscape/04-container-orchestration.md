# Chapter 4.4 — Container Orchestration

[← Previous: Containerization](03-containerization.md) | [Next: IaC Tools →](05-iac-tools.md)

---

## Overview

When you move from running a few containers to hundreds or thousands, you need **container orchestration**. Orchestrators automate deployment, scaling, networking, and self-healing of containerized applications. **Kubernetes (K8s)** is the industry standard.

---

## Why Orchestration?

```
┌──────────────────────────────────────────────────────────┐
│  WITHOUT ORCHESTRATION         WITH ORCHESTRATION        │
│  ══════════════════════        ═══════════════════       │
│                                                          │
│  "Run docker on 50 servers"    "Run 50 replicas"         │
│                                                          │
│  ├── Which server has space?   ├── Scheduler decides     │
│  ├── Container crashed → ???   ├── Auto-restart          │
│  ├── Traffic spike → manual    ├── Auto-scale            │
│  ├── Rolling update → ???      ├── Zero-downtime deploy  │
│  ├── Service discovery → ???   ├── Built-in DNS          │
│  ├── Load balancing → manual   ├── Automatic             │
│  └── Secrets → env vars        └── Secrets management    │
│                                                          │
│  Manual + fragile              Automated + resilient     │
└──────────────────────────────────────────────────────────┘
```

---

## Kubernetes Architecture

```
┌──────────────────────────────────────────────────────────┐
│  KUBERNETES CLUSTER ARCHITECTURE                         │
│                                                          │
│  ┌─────────────── Control Plane ──────────────────┐     │
│  │                                                 │     │
│  │  ┌───────────┐  ┌──────────┐  ┌─────────────┐ │     │
│  │  │API Server │  │Scheduler │  │ Controller  │ │     │
│  │  │(kube-api) │  │          │  │  Manager    │ │     │
│  │  └───────────┘  └──────────┘  └─────────────┘ │     │
│  │  ┌───────────┐                                 │     │
│  │  │   etcd    │  ← Cluster state database       │     │
│  │  └───────────┘                                 │     │
│  └─────────────────────────────────────────────────┘     │
│       │              │              │                    │
│  ┌────▼────┐    ┌────▼────┐    ┌────▼────┐              │
│  │ Worker  │    │ Worker  │    │ Worker  │              │
│  │ Node 1  │    │ Node 2  │    │ Node 3  │              │
│  │         │    │         │    │         │              │
│  │┌──┐┌──┐│    │┌──┐┌──┐│    │┌──┐┌──┐│              │
│  ││P1││P2││    ││P3││P4││    ││P5││P6││              │
│  │└──┘└──┘│    │└──┘└──┘│    │└──┘└──┘│              │
│  │kubelet │    │kubelet │    │kubelet │              │
│  │kube-prx│    │kube-prx│    │kube-prx│              │
│  └────────┘    └────────┘    └────────┘              │
│                                                          │
│  P = Pod (smallest deployable unit)                      │
└──────────────────────────────────────────────────────────┘
```

### Control Plane Components

| Component | Role |
|-----------|------|
| **API Server** | Front door for all K8s operations (REST API) |
| **etcd** | Distributed key-value store for cluster state |
| **Scheduler** | Decides which node runs new pods |
| **Controller Manager** | Ensures desired state matches actual state |

### Worker Node Components

| Component | Role |
|-----------|------|
| **kubelet** | Agent on each node; manages pods |
| **kube-proxy** | Network proxy; handles service routing |
| **Container Runtime** | Runs containers (containerd, CRI-O) |

---

## Key Kubernetes Objects

```
┌──────────────────────────────────────────────────────────┐
│  KUBERNETES OBJECT HIERARCHY                             │
│                                                          │
│  Cluster                                                 │
│  └── Namespace (logical isolation)                       │
│      ├── Deployment (manages ReplicaSets)                │
│      │   └── ReplicaSet (maintains pod count)            │
│      │       └── Pod (one or more containers)            │
│      │           └── Container (your application)        │
│      ├── Service (stable network endpoint)               │
│      ├── ConfigMap (non-secret configuration)            │
│      ├── Secret (sensitive data)                         │
│      ├── Ingress (external HTTP routing)                 │
│      └── PersistentVolumeClaim (storage)                 │
└──────────────────────────────────────────────────────────┘
```

---

## Kubernetes YAML Manifests

### Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
  labels:
    app: web-app
    version: v2.1.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # 1 extra pod during update
      maxUnavailable: 0    # Always keep all pods running
  template:
    metadata:
      labels:
        app: web-app
        version: v2.1.0
    spec:
      containers:
        - name: web-app
          image: registry.io/web-app:v2.1.0
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: "250m"       # 0.25 CPU cores
              memory: "256Mi"   # 256 MB RAM
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
          env:
            - name: NODE_ENV
              value: "production"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secrets
                  key: password
```

### Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 3000
      protocol: TCP
```

### Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-app-service
                port:
                  number: 80
```

---

## Horizontal Pod Autoscaler (HPA)

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

```
┌──────────────────────────────────────────────────────────┐
│  AUTOSCALING IN ACTION                                   │
│                                                          │
│  Normal Load (CPU ~30%):                                 │
│  [Pod 1] [Pod 2] [Pod 3]                                │
│                                                          │
│  Traffic Spike (CPU ~85%):                               │
│  [Pod 1] [Pod 2] [Pod 3] [Pod 4] [Pod 5] [Pod 6]       │
│  HPA detects CPU > 70% → scales to 6 pods               │
│                                                          │
│  Traffic Drops (CPU ~20%):                               │
│  [Pod 1] [Pod 2] [Pod 3]                                │
│  HPA detects CPU < 70% → scales back to 3 pods          │
└──────────────────────────────────────────────────────────┘
```

---

## Essential kubectl Commands

```bash
# ═══ CLUSTER INFO ═══
kubectl cluster-info                          # Cluster details
kubectl get nodes                             # List nodes
kubectl top nodes                             # Node resource usage

# ═══ DEPLOYMENTS ═══
kubectl apply -f deployment.yaml              # Create/update resource
kubectl get deployments -n production         # List deployments
kubectl rollout status deploy/web-app         # Watch rollout
kubectl rollout undo deploy/web-app           # Rollback to previous
kubectl scale deploy/web-app --replicas=5     # Manual scale

# ═══ PODS ═══
kubectl get pods -n production                # List pods
kubectl describe pod <pod-name>               # Detailed pod info
kubectl logs <pod-name> -f                    # Follow pod logs
kubectl exec -it <pod-name> -- /bin/sh        # Shell into pod
kubectl top pods                              # Pod resource usage

# ═══ SERVICES ═══
kubectl get services                          # List services
kubectl get ingress                           # List ingress rules

# ═══ DEBUGGING ═══
kubectl get events --sort-by=.lastTimestamp   # Recent events
kubectl describe pod <pod-name>               # Check pod events
kubectl logs <pod-name> --previous            # Logs from crashed pod
```

---

## Managed Kubernetes Services

| Service | Provider | Description |
|---------|----------|-------------|
| **EKS** | AWS | Elastic Kubernetes Service |
| **AKS** | Azure | Azure Kubernetes Service |
| **GKE** | Google | Google Kubernetes Engine |
| **DOKS** | DigitalOcean | Managed K8s |

All managed services handle the control plane for you — you only manage worker nodes and workloads.

---

## Real-World Scenario: Zero-Downtime Deployment

```
ROLLING UPDATE PROCESS:

Initial state (v1):
┌──────┐ ┌──────┐ ┌──────┐
│Pod v1│ │Pod v1│ │Pod v1│   ← 3 pods serving traffic
└──────┘ └──────┘ └──────┘

Step 1 — Create new pod:
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Pod v1│ │Pod v1│ │Pod v1│ │Pod v2│  ← maxSurge: 1
└──────┘ └──────┘ └──────┘ └──────┘

Step 2 — v2 passes readiness check, terminate one v1:
┌──────┐ ┌──────┐ ┌──────┐
│Pod v1│ │Pod v1│ │Pod v2│
└──────┘ └──────┘ └──────┘

Step 3-4 — Repeat until complete:
┌──────┐ ┌──────┐ ┌──────┐
│Pod v2│ │Pod v2│ │Pod v2│   ← All v2, zero downtime!
└──────┘ └──────┘ └──────┘

If v2 crashes → readiness fails → rollout pauses
kubectl rollout undo deploy/web-app → instant rollback
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pod stuck in `Pending` | No node has enough resources | Check resource requests; add nodes or free resources |
| Pod in `CrashLoopBackOff` | Application crashing on start | Check `kubectl logs`; verify env vars and configs |
| Pod in `ImagePullBackOff` | Wrong image name or no auth | Verify image tag; create registry pull secret |
| Service not routing traffic | Label selector mismatch | Ensure pod labels match service selector |
| HPA not scaling | Metrics server not installed | Install metrics-server; check `kubectl top pods` |
| OOMKilled | Memory limit exceeded | Increase memory limits or optimize application |

---

## Summary Table

| K8s Concept | Description |
|------------|-------------|
| **Pod** | Smallest deployable unit (one or more containers) |
| **Deployment** | Manages pod replicas and rolling updates |
| **Service** | Stable network endpoint for a set of pods |
| **Ingress** | Routes external HTTP traffic to services |
| **Namespace** | Logical isolation within a cluster |
| **ConfigMap / Secret** | External configuration and sensitive data |
| **HPA** | Auto-scales pods based on CPU/memory metrics |
| **kubectl** | CLI tool for interacting with Kubernetes |

---

## Quick Revision Questions

1. **Why do we need container orchestration?**
   <details><summary>Answer</summary>When running many containers, you need automated: scheduling (which host runs each container), scaling (add/remove instances), self-healing (restart crashed containers), load balancing (distribute traffic), service discovery (find other services), rolling updates (zero-downtime deploys), and secrets management. Manual management doesn't scale.</details>

2. **What is the role of the Kubernetes control plane?**
   <details><summary>Answer</summary>The control plane manages the cluster: API Server (handles all requests), Scheduler (assigns pods to nodes), Controller Manager (ensures desired state = actual state), and etcd (stores cluster state). It makes all cluster-level decisions but doesn't run application workloads — those run on worker nodes.</details>

3. **What is a Pod in Kubernetes?**
   <details><summary>Answer</summary>A Pod is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace (localhost) and storage volumes. Pods are ephemeral — they can be replaced at any time. You typically don't create Pods directly; you use Deployments to manage them.</details>

4. **How does a Kubernetes rolling update achieve zero downtime?**
   <details><summary>Answer</summary>A rolling update: 1) Creates new pods with the updated version. 2) Waits for readiness probes to pass. 3) Routes traffic to new pods. 4) Terminates old pods one by one. Using maxSurge and maxUnavailable settings, it ensures at least the desired number of pods are always serving traffic. If new pods fail, the rollout pauses automatically.</details>

5. **What is the difference between a Deployment, a Service, and an Ingress?**
   <details><summary>Answer</summary>Deployment: manages pod replicas and updates (how many instances, which version). Service: provides a stable internal IP and DNS name for pods (internal networking). Ingress: routes external HTTP/HTTPS traffic to services based on hostname and path rules (external access with TLS).</details>

6. **How does the Horizontal Pod Autoscaler (HPA) work?**
   <details><summary>Answer</summary>HPA monitors resource metrics (CPU, memory) for pods. When average utilization exceeds the target (e.g., 70% CPU), it increases replicas up to maxReplicas. When utilization drops below the target, it removes replicas down to minReplicas. It runs a control loop every 15 seconds by default, using the metrics-server for data.</details>

---

[← Previous: Containerization](03-containerization.md) | [Next: IaC Tools →](05-iac-tools.md)
