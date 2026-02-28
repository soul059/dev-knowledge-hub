# Unit 11: Advanced Topics

[← Previous: Observability](../10-Observability/01-observability.md) | [Back to README](../README.md) | [Next: Kubernetes Administration →](../12-Kubernetes-Administration/01-kubernetes-administration.md)

---

## Chapter Overview

This unit explores advanced Kubernetes topics that extend the platform's capabilities. We cover **Helm** (package manager), **Custom Resource Definitions (CRDs)**, **Operators**, **Autoscaling (HPA/VPA/Cluster Autoscaler)**, **GitOps (ArgoCD/Flux)**, and **Service Mesh (Istio)**.

---

## 11.1 Helm — The Kubernetes Package Manager

Helm packages Kubernetes manifests into **Charts** — reusable, versioned, and configurable templates.

```
┌──────────────────────────────────────────────────────────────┐
│                    HELM CONCEPTS                              │
│                                                              │
│  Chart: A package of K8s resource templates                  │
│  Release: An instance of a chart deployed to a cluster       │
│  Repository: A collection of published charts                │
│  Values: Configuration that customizes a chart               │
│                                                              │
│  ┌─────────────┐   values.yaml   ┌──────────────┐           │
│  │             │ ──────────────▶ │              │           │
│  │  Chart      │                 │   Release    │           │
│  │  Templates  │  helm install   │  (deployed   │           │
│  │  + defaults │ ──────────────▶ │   resources) │           │
│  │             │                 │              │           │
│  └─────────────┘                 └──────────────┘           │
│                                                              │
│  Chart Structure:                                            │
│  my-chart/                                                   │
│  ├── Chart.yaml        # Chart metadata (name, version)      │
│  ├── values.yaml       # Default configuration values        │
│  ├── charts/           # Dependencies (sub-charts)           │
│  ├── templates/        # K8s manifest templates              │
│  │   ├── deployment.yaml                                     │
│  │   ├── service.yaml                                        │
│  │   ├── ingress.yaml                                        │
│  │   ├── configmap.yaml                                      │
│  │   ├── _helpers.tpl   # Template helpers                   │
│  │   ├── NOTES.txt      # Post-install instructions          │
│  │   └── tests/         # Test hooks                         │
│  └── .helmignore       # Files to ignore                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Helm Commands

```bash
# ═══════════ Repository Management ═══════════
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus https://prometheus-community.github.io/helm-charts
helm repo update
helm repo list
helm search repo nginx                # Search repos
helm search hub wordpress             # Search Artifact Hub

# ═══════════ Install / Upgrade / Rollback ═══════════
# Install a chart
helm install my-release bitnami/nginx

# Install with custom values
helm install my-release bitnami/nginx -f custom-values.yaml

# Install with --set overrides
helm install my-release bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=LoadBalancer \
  --namespace production --create-namespace

# Upgrade a release
helm upgrade my-release bitnami/nginx -f new-values.yaml

# Install or upgrade (idempotent)
helm upgrade --install my-release bitnami/nginx -f values.yaml

# Rollback to previous revision
helm rollback my-release 1

# ═══════════ Inspect and Manage ═══════════
helm list                              # List all releases
helm list -A                           # All namespaces
helm status my-release                 # Release status
helm history my-release                # Release history
helm get values my-release             # Current values
helm get manifest my-release           # Rendered manifests
helm show values bitnami/nginx         # Chart default values

# ═══════════ Uninstall ═══════════
helm uninstall my-release

# ═══════════ Create Your Own Chart ═══════════
helm create my-app                     # Scaffold new chart
helm template my-app ./my-app          # Render locally (dry-run)
helm lint ./my-app                     # Validate chart
helm package ./my-app                  # Package as .tgz
```

### Helm Templating Basics

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
  labels:
    app: {{ .Values.app.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.app.name }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.port }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
```

```yaml
# values.yaml
replicaCount: 2
app:
  name: my-web-app
image:
  repository: nginx
  tag: "1.25"
service:
  type: ClusterIP
  port: 80
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

---

## 11.2 Custom Resource Definitions (CRDs)

CRDs extend the Kubernetes API with custom resource types.

```
┌──────────────────────────────────────────────────────────────┐
│            CUSTOM RESOURCE DEFINITIONS                        │
│                                                              │
│  Built-in resources:    Custom resources (via CRDs):         │
│  ┌────────────────┐    ┌────────────────┐                    │
│  │ Pod            │    │ Certificate    │ (cert-manager)     │
│  │ Deployment     │    │ VirtualService │ (Istio)            │
│  │ Service        │    │ PrometheusRule │ (Prometheus)       │
│  │ ConfigMap      │    │ Backup         │ (Velero)           │
│  │ Ingress        │    │ Database       │ (custom operator)  │
│  └────────────────┘    └────────────────┘                    │
│                                                              │
│  CRDs allow you to define your own "Kind" of resource        │
│  that the K8s API server manages just like built-in types    │
│                                                              │
│  kubectl get certificates   ← works just like built-in!      │
│  kubectl describe database my-db                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### CRD YAML Example

```yaml
# Define the CRD
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.io
spec:
  group: mycompany.io
  names:
    kind: Database
    listKind: DatabaseList
    plural: databases
    singular: database
    shortNames:
    - db
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: [engine, version, storage]
            properties:
              engine:
                type: string
                enum: [postgres, mysql, mongodb]
              version:
                type: string
              replicas:
                type: integer
                default: 1
              storage:
                type: string
          status:
            type: object
            properties:
              phase:
                type: string
              endpoint:
                type: string
    additionalPrinterColumns:
    - name: Engine
      type: string
      jsonPath: .spec.engine
    - name: Version
      type: string
      jsonPath: .spec.version
    - name: Status
      type: string
      jsonPath: .status.phase
---
# Use the custom resource
apiVersion: mycompany.io/v1
kind: Database
metadata:
  name: orders-db
  namespace: production
spec:
  engine: postgres
  version: "16"
  replicas: 3
  storage: "100Gi"
```

```bash
kubectl apply -f database-crd.yaml     # Register the CRD
kubectl get crd                        # List all CRDs
kubectl get databases                  # List custom resources
kubectl get db orders-db -o yaml       # Shortname works!
```

---

## 11.3 Operators

An **Operator** is a controller that uses CRDs to automate the lifecycle of complex applications. It encodes operational knowledge (deployment, scaling, backup, upgrade, recovery) into software.

```
┌──────────────────────────────────────────────────────────────┐
│                 OPERATOR PATTERN                              │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                  Control Loop                          │  │
│  │                                                        │  │
│  │   ┌──────────┐   Watch    ┌──────────────────────┐    │  │
│  │   │  Custom   │◄─────────│    Operator           │    │  │
│  │   │  Resource │           │    Controller         │    │  │
│  │   │  (CR)     │           │                      │    │  │
│  │   │           │  Observe  │  1. Observe current   │    │  │
│  │   │ Desired:  │──────────▶│     state             │    │  │
│  │   │ 3 replicas│           │  2. Compare to desired│    │  │
│  │   │ v16       │           │  3. Act to reconcile  │    │  │
│  │   │ 100Gi     │           │                      │    │  │
│  │   └──────────┘   Act     │  Creates/manages:     │    │  │
│  │        ▲        ◄────────│  - StatefulSet        │    │  │
│  │        │                  │  - Service            │    │  │
│  │   User updates CR         │  - PVC                │    │  │
│  │   (e.g., replicas: 5)     │  - ConfigMap          │    │  │
│  │                           │  - Backup CronJob     │    │  │
│  │                           └──────────────────────┘    │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Popular Operators:                                          │
│  ┌────────────────────┬──────────────────────────────────┐   │
│  │ Prometheus Operator│ Manages Prometheus, AlertManager  │   │
│  │ cert-manager       │ Automates TLS certificate mgmt   │   │
│  │ Strimzi            │ Manages Apache Kafka clusters     │   │
│  │ Zalando Postgres   │ Manages PostgreSQL clusters       │   │
│  │ MongoDB Operator   │ Manages MongoDB deployments       │   │
│  │ ArgoCD             │ GitOps continuous delivery        │   │
│  │ Velero             │ Backup and disaster recovery      │   │
│  └────────────────────┴──────────────────────────────────┘   │
│                                                              │
│  Find operators: https://operatorhub.io                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 11.4 Autoscaling

### Horizontal Pod Autoscaler (HPA)

HPA automatically scales the **number of pod replicas** based on CPU/memory usage or custom metrics.

```
┌──────────────────────────────────────────────────────────────┐
│          HORIZONTAL POD AUTOSCALER                            │
│                                                              │
│  metrics-server / Prometheus ──▶ HPA controller              │
│                                       │                      │
│                                       ▼                      │
│  Current: 2 pods, CPU = 80%    Target: CPU 50%               │
│                                                              │
│  Formula:                                                    │
│  desiredReplicas = ceil(currentReplicas *                     │
│                    (currentMetric / targetMetric))            │
│                                                              │
│  = ceil(2 * (80 / 50)) = ceil(3.2) = 4 replicas             │
│                                                              │
│  Before:                      After:                         │
│  ┌──────┐  ┌──────┐          ┌──────┐ ┌──────┐              │
│  │Pod 1 │  │Pod 2 │          │Pod 1 │ │Pod 2 │              │
│  │ 80%  │  │ 80%  │    ──▶   │ 40%  │ │ 40%  │              │
│  └──────┘  └──────┘          └──────┘ └──────┘              │
│                               ┌──────┐ ┌──────┐              │
│                               │Pod 3 │ │Pod 4 │              │
│                               │ 40%  │ │ 40%  │              │
│                               └──────┘ └──────┘              │
│                                                              │
│  Scale-up: Immediate when threshold exceeded                 │
│  Scale-down: Gradual (default 5-min stabilization window)    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  # CPU-based scaling
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50       # Target 50% CPU

  # Memory-based scaling
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi          # Target 500Mi per pod

  # Custom metric (e.g., requests per second)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"         # 1000 RPS per pod

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60   # Wait 60s before scaling up
      policies:
      - type: Percent
        value: 100                     # Double replicas max
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5min before scaling down
      policies:
      - type: Pods
        value: 1                       # Remove 1 pod at a time
        periodSeconds: 60
```

```bash
# HPA commands
kubectl autoscale deployment web-app \
  --cpu-percent=50 --min=2 --max=20      # Imperative creation

kubectl get hpa                           # View HPAs
kubectl describe hpa web-hpa              # Detailed status
kubectl top pods                          # Current resource usage
```

### Vertical Pod Autoscaler (VPA)

VPA automatically adjusts **CPU and memory requests/limits** for pods.

```
┌──────────────────────────────────────────────────────────────┐
│          VERTICAL POD AUTOSCALER                              │
│                                                              │
│  Before VPA:               After VPA:                        │
│  ┌──────────────┐          ┌──────────────┐                  │
│  │ Pod          │          │ Pod          │                  │
│  │ req: 100m CPU│          │ req: 350m CPU│  ← Adjusted     │
│  │ lim: 200m   │   ──▶    │ lim: 700m   │    based on      │
│  │ req: 128Mi  │          │ req: 512Mi  │    actual usage   │
│  │ lim: 256Mi  │          │ lim: 1Gi    │                  │
│  │             │          │             │                  │
│  │ Actual use: │          │ Right-sized! │                  │
│  │ CPU: 340m   │          │             │                  │
│  │ Mem: 480Mi  │          │             │                  │
│  └──────────────┘          └──────────────┘                  │
│                                                              │
│  Modes:                                                      │
│  ┌──────────┬──────────────────────────────────────────┐     │
│  │ Off      │ Only calculates recommendations          │     │
│  │ Initial  │ Sets resources only at pod creation      │     │
│  │ Auto     │ Evicts running pods to apply new values  │     │
│  │ Recreate │ Same as Auto (alias)                     │     │
│  └──────────┴──────────────────────────────────────────┘     │
│                                                              │
│  ⚠ VPA and HPA should NOT target the same metric             │
│    (e.g., both on CPU). Use HPA for scaling out,             │
│    VPA for right-sizing, on different metrics.               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  updatePolicy:
    updateMode: "Auto"              # Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: web
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

### Cluster Autoscaler (CA)

The Cluster Autoscaler adds or removes **nodes** from the cluster based on pending pods.

```
┌──────────────────────────────────────────────────────────────┐
│            CLUSTER AUTOSCALER                                 │
│                                                              │
│  Trigger: Pod can't be scheduled (Pending) due to            │
│  insufficient resources                                      │
│                                                              │
│  Scale Up:                                                   │
│  ┌──────┐ ┌──────┐ ┌──────┐   Pending   ┌──────┐           │
│  │Node 1│ │Node 2│ │Node 3│   Pod ──▶   │Node 4│ (NEW!)    │
│  │ FULL │ │ FULL │ │ FULL │             │ Pod  │           │
│  └──────┘ └──────┘ └──────┘   ──▶       │ here │           │
│                                          └──────┘           │
│                                                              │
│  Scale Down:                                                 │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                        │
│  │Node 1│ │Node 2│ │Node 3│ │Node 4│                        │
│  │ busy │ │ busy │ │ 10%  │ │empty │                        │
│  └──────┘ └──────┘ └──────┘ └──┬───┘                        │
│                                 │                            │
│           Pods moved ◄──────────┘ Node removed               │
│           to Node 3    (after 10min idle)                     │
│                                                              │
│  Works with:                                                 │
│  - AWS Auto Scaling Groups (ASG)                             │
│  - GCP Managed Instance Groups (MIG)                         │
│  - Azure VM Scale Sets (VMSS)                                │
│  - Karpenter (AWS-native, faster alternative)                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### KEDA — Kubernetes Event-Driven Autoscaling

```
┌──────────────────────────────────────────────────────────────┐
│  KEDA: Scale based on external events                        │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Event Sources:                                  │        │
│  │  - Kafka topic lag                               │        │
│  │  - RabbitMQ queue length                         │        │
│  │  - AWS SQS messages                              │        │
│  │  - Azure Service Bus                             │        │
│  │  - Prometheus metrics                            │        │
│  │  - Cron schedule                                 │        │
│  │  - HTTP requests                                 │        │
│  │  - 60+ scalers available                         │        │
│  └──────────────────────┬───────────────────────────┘        │
│                         │                                    │
│                         ▼                                    │
│  ┌──────────────────────────────────────────────────┐        │
│  │  KEDA → creates/manages HPA → scales Deployment  │        │
│  │  Can scale to/from ZERO pods!                    │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 11.5 GitOps with ArgoCD and Flux

GitOps uses Git as the single source of truth for declarative infrastructure and applications.

```
┌──────────────────────────────────────────────────────────────┐
│               GITOPS PRINCIPLES                               │
│                                                              │
│  1. Declarative: Entire system described declaratively        │
│  2. Versioned:   Desired state stored in Git                 │
│  3. Automated:   Changes auto-applied to cluster             │
│  4. Reconciled:  Software agents ensure convergence          │
│                                                              │
│  Traditional:                    GitOps:                      │
│  Developer ──▶ kubectl apply    Developer ──▶ git push       │
│  (imperative, manual)            (declarative, automated)     │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │                                                  │        │
│  │  Developer ──▶ Git Push ──▶ Git Repository       │        │
│  │                               │                  │        │
│  │                          Pull / Sync             │        │
│  │                               │                  │        │
│  │                               ▼                  │        │
│  │                          ┌──────────┐            │        │
│  │                          │ ArgoCD / │            │        │
│  │                          │ Flux     │            │        │
│  │                          └─────┬────┘            │        │
│  │                                │                 │        │
│  │                          Apply to cluster        │        │
│  │                                │                 │        │
│  │                                ▼                 │        │
│  │                          ┌──────────┐            │        │
│  │                          │   K8s    │            │        │
│  │                          │ Cluster  │            │        │
│  │                          └──────────┘            │        │
│  │                                                  │        │
│  │  If cluster state drifts from Git → auto-heal    │        │
│  │                                                  │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### ArgoCD

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/org/k8s-manifests.git
    targetRevision: main
    path: apps/my-app/production     # Path in repo

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true                    # Delete resources removed from Git
      selfHeal: true                 # Fix drift automatically
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```bash
# ArgoCD CLI
argocd app list                          # List applications
argocd app get my-app                    # App details
argocd app sync my-app                   # Manual sync
argocd app diff my-app                   # Show diff
argocd app history my-app                # Deployment history
argocd app rollback my-app <revision>    # Rollback
```

### ArgoCD vs Flux

```
┌──────────────────────────────────────────────────────────────┐
│              ArgoCD vs Flux Comparison                        │
│                                                              │
│  ┌─────────────┬──────────────────┬──────────────────┐       │
│  │ Feature     │ ArgoCD           │ Flux             │       │
│  ├─────────────┼──────────────────┼──────────────────┤       │
│  │ UI          │ Rich web UI ✓    │ No built-in UI   │       │
│  │ Architecture│ Centralized      │ Decentralized    │       │
│  │ Multi-cluster│ Built-in ✓      │ Via controllers  │       │
│  │ RBAC        │ Built-in SSO/RBAC│ K8s native RBAC  │       │
│  │ Helm support│ ✓                │ ✓ (HelmRelease)  │       │
│  │ Kustomize   │ ✓                │ ✓ (Kustomization)│       │
│  │ CNCF Status │ Graduated        │ Graduated        │       │
│  │ Best for    │ Multi-cluster    │ Deep K8s native  │       │
│  │             │ with UI needs    │ GitOps           │       │
│  └─────────────┴──────────────────┴──────────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 11.6 Service Mesh (Istio)

A service mesh provides infrastructure-level networking features: traffic management, security (mTLS), and observability — without modifying application code.

```
┌──────────────────────────────────────────────────────────────┐
│               SERVICE MESH ARCHITECTURE                       │
│                                                              │
│  Without mesh:                                               │
│  ┌──────┐ ─── HTTP ──▶ ┌──────┐                             │
│  │App A │               │App B │  No encryption, no          │
│  └──────┘               └──────┘  retries, no tracing        │
│                                                              │
│  With mesh (Istio):                                          │
│  ┌──────────────────┐    ┌──────────────────┐                │
│  │ ┌──────┐ ┌─────┐│    │┌─────┐ ┌──────┐  │                │
│  │ │App A │ │Envoy││    ││Envoy│ │App B │  │                │
│  │ │      │ │proxy││◄──▶││proxy│ │      │  │                │
│  │ └──────┘ └─────┘│mTLS│└─────┘ └──────┘  │                │
│  │   Pod A         │    │   Pod B           │                │
│  └──────────────────┘    └──────────────────┘                │
│         ▲                         ▲                          │
│         │       Control Plane     │                          │
│         │    ┌────────────────┐   │                          │
│         └────│    istiod      │───┘                          │
│              │  (Pilot +      │                              │
│              │   Citadel +    │   Pushes config to           │
│              │   Galley)      │   all Envoy sidecars         │
│              └────────────────┘                              │
│                                                              │
│  Envoy sidecar is auto-injected into every pod               │
│  Handles: mTLS, retries, timeouts, circuit breaking,         │
│           load balancing, tracing, metrics                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Istio Traffic Management

```yaml
# VirtualService — traffic routing rules
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews
  http:
  # Canary: 90% to v1, 10% to v2
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 3s
---
# DestinationRule — define subsets and policies
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
    outlierDetection:              # Circuit breaker
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

### Service Mesh Comparison

```
┌──────────────────────────────────────────────────────────────┐
│  ┌──────────┬──────────────┬──────────────┬──────────────┐   │
│  │Feature   │ Istio        │ Linkerd      │ Cilium       │   │
│  ├──────────┼──────────────┼──────────────┼──────────────┤   │
│  │Complexity│ High         │ Low          │ Medium       │   │
│  │Sidecar   │ Envoy        │ Ultra-light  │ No sidecar   │   │
│  │          │              │ Rust proxy   │ (eBPF-based) │   │
│  │mTLS      │ ✓            │ ✓            │ ✓            │   │
│  │Traffic   │ Very rich    │ Basic        │ Medium       │   │
│  │ mgmt     │              │              │              │   │
│  │Resources │ Heavy        │ Light        │ Light        │   │
│  │CNCF      │ Graduated    │ Graduated    │ Graduated    │   │
│  └──────────┴──────────────┴──────────────┴──────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Canary Deployment with ArgoCD + Istio

```yaml
# ArgoCD deploys the manifests from Git
# Istio handles traffic splitting for canary

# Step 1: Deploy v2 alongside v1
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
      version: v2
  template:
    metadata:
      labels:
        app: web-app
        version: v2
    spec:
      containers:
      - name: web
        image: web-app:2.0
---
# Step 2: Route 10% traffic to v2
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web-app
spec:
  hosts: [web-app]
  http:
  - route:
    - destination:
        host: web-app
        subset: v1
      weight: 90
    - destination:
        host: web-app
        subset: v2
      weight: 10

# Step 3: Monitor metrics, gradually increase to 50%, 100%
# Step 4: Delete v1 deployment when confident
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| Helm install fails | `helm install --debug --dry-run` | Fix template errors; check values.yaml |
| HPA not scaling | `kubectl describe hpa` | Ensure metrics-server is installed and pod has resource requests |
| HPA showing `<unknown>` metrics | `kubectl top pods` | Install metrics-server; check pod resource requests are set |
| ArgoCD shows "OutOfSync" | ArgoCD UI or `argocd app diff` | Sync manually or enable auto-sync |
| Istio sidecar not injected | Check namespace label | `kubectl label ns <ns> istio-injection=enabled` |
| CRD not recognized | `kubectl get crd` | Ensure CRD is applied before creating CR instances |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Helm** | K8s package manager: Charts = templates + values = releases |
| **CRD** | Extend K8s API with custom resource types |
| **Operator** | Controller + CRD that automates complex app lifecycle |
| **HPA** | Scale pod replicas based on CPU/memory/custom metrics |
| **VPA** | Adjust pod resource requests/limits automatically |
| **Cluster Autoscaler** | Add/remove nodes based on pending pod demand |
| **KEDA** | Event-driven autoscaling (queues, topics, scale to zero) |
| **GitOps** | Git as single source of truth; auto-sync to cluster |
| **ArgoCD** | GitOps tool with rich UI and multi-cluster support |
| **Flux** | Lightweight, K8s-native GitOps toolkit |
| **Service Mesh** | Infrastructure-level mTLS, traffic control, observability |
| **Istio** | Most feature-rich mesh; Envoy sidecars; VirtualService/DestinationRule |

---

## Quick Revision Questions

1. **What is a Helm Chart, and what are the key files in a chart?**
   <details><summary>Answer</summary>A Helm Chart is a package of Kubernetes manifest templates that can be customized and versioned. Key files: `Chart.yaml` (metadata — name, version, description), `values.yaml` (default configuration values), `templates/` (K8s manifest templates using Go templating), `charts/` (dependency sub-charts), `.helmignore` (files to ignore). A Release is an instance of a chart deployed to a cluster.</details>

2. **How does an Operator differ from a standard Deployment?**
   <details><summary>Answer</summary>A standard Deployment manages stateless pod replicas. An Operator is a custom controller that uses CRDs to encode application-specific operational knowledge. It watches custom resources and automatically handles complex lifecycle tasks: provisioning, scaling, backup, upgrades, failure recovery. For example, a PostgreSQL Operator creates StatefulSets, Services, PVCs, handles replication, backups, and failover — things a plain Deployment can't automate.</details>

3. **Why might an HPA show `<unknown>` for metrics, and how do you fix it?**
   <details><summary>Answer</summary>Common causes: 1) **metrics-server not installed** — HPA needs it for CPU/memory metrics. Install it. 2) **No resource requests** on the target pods — HPA calculates utilization as a percentage of requests. Add `resources.requests.cpu` to the pod spec. 3) **Metrics not yet available** — wait a few minutes after metrics-server install. Check with: `kubectl top pods` and `kubectl describe hpa`.</details>

4. **What is the key difference between ArgoCD's pull-based GitOps and a traditional CI/CD push model?**
   <details><summary>Answer</summary>**Push model**: CI pipeline runs `kubectl apply` or `helm install` to push changes to the cluster. Requires cluster credentials in CI. **Pull model (GitOps)**: ArgoCD runs inside the cluster and continuously pulls/watches the Git repo for changes. No cluster credentials leak outside. ArgoCD also detects and optionally auto-heals drift (when someone manually changes cluster state). Git becomes the audit trail for all changes.</details>

5. **What problem does a service mesh solve, and what is the sidecar pattern?**
   <details><summary>Answer</summary>A service mesh solves cross-cutting concerns in microservices: mutual TLS (encryption between services), traffic management (canary, retries, timeouts, circuit breaking), and observability (distributed tracing, metrics) — without changing application code. The sidecar pattern injects a proxy container (e.g., Envoy) into every pod. All traffic goes through the proxy, which handles mTLS, routing, retries, and telemetry transparently.</details>

6. **When should you use HPA vs VPA vs Cluster Autoscaler?**
   <details><summary>Answer</summary>**HPA**: Scale the number of pod replicas horizontally based on load (CPU, memory, custom metrics). Best for stateless services. **VPA**: Adjust resource requests/limits vertically (right-size individual pods). Best for workloads where scaling out isn't effective. Don't use VPA and HPA on the same metric. **Cluster Autoscaler**: Adds/removes cluster nodes when pods can't be scheduled (insufficient node resources) or nodes are underutilized. Works alongside HPA — HPA creates pods, CA ensures nodes exist to run them.</details>

---

[← Previous: Observability](../10-Observability/01-observability.md) | [Back to README](../README.md) | [Next: Kubernetes Administration →](../12-Kubernetes-Administration/01-kubernetes-administration.md)
