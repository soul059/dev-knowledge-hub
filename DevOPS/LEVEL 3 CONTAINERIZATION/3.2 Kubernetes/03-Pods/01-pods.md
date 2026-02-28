# Unit 3: Pods

[← Previous: Kubernetes Architecture](../02-Kubernetes-Architecture/01-architecture-overview.md) | [Back to README](../README.md) | [Next: Workload Resources →](../04-Workload-Resources/01-workload-resources.md)

---

## Chapter Overview

The **Pod** is the smallest deployable unit in Kubernetes — not a container, but a **wrapper around one or more containers** that share networking and storage. This unit covers pod concepts, lifecycle, multi-container patterns, YAML structure, management commands, and static pods.

---

## 3.1 Pod Concepts

### What is a Pod?

A Pod represents a **single instance of a running process** in your cluster. It encapsulates one or more containers that:

- Share the **same network namespace** (same IP, same ports)
- Share the **same storage volumes**
- Are co-located and co-scheduled on the **same node**
- Share the **same lifecycle** (created and destroyed together)

```
┌─────────────────────────────────────────────────────────┐
│                          NODE                            │
│                                                         │
│   ┌─────────────────── POD ──────────────────────┐      │
│   │                                              │      │
│   │  Shared Network Namespace (IP: 10.244.1.5)   │      │
│   │  ┌────────────────────────────────────┐       │      │
│   │  │ eth0: 10.244.1.5                  │       │      │
│   │  │ lo:   127.0.0.1                   │       │      │
│   │  └────────────────────────────────────┘       │      │
│   │                                              │      │
│   │  ┌──────────────┐    ┌──────────────┐        │      │
│   │  │ Container 1  │    │ Container 2  │        │      │
│   │  │ (app)        │    │ (sidecar)    │        │      │
│   │  │ Port: 8080   │    │ Port: 9090   │        │      │
│   │  └──────────────┘    └──────────────┘        │      │
│   │                                              │      │
│   │  Shared Volumes                              │      │
│   │  ┌──────────────────────────────────┐        │      │
│   │  │  /var/log  (emptyDir)            │        │      │
│   │  │  /data     (persistentVolume)    │        │      │
│   │  └──────────────────────────────────┘        │      │
│   │                                              │      │
│   └──────────────────────────────────────────────┘      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Pods vs Containers

| Aspect | Container | Pod |
|--------|-----------|-----|
| **Definition** | Single process in isolation | Group of 1+ containers |
| **Networking** | Own network namespace | Shared network namespace |
| **Scheduling** | N/A in K8s | Unit of scheduling |
| **IP Address** | N/A (pod gets the IP) | One IP per pod |
| **Communication** | Cross-pod via network | Within pod via localhost |
| **Lifecycle** | Managed by pod | K8s manages as a unit |

### When to Use Multiple Containers in a Pod

```
  ONE container per pod (MOST cases):        Multiple containers per pod:
  ┌─────────┐  ┌─────────┐                  ┌──────────────────────────┐
  │  Pod 1  │  │  Pod 2  │                  │        Pod               │
  │ ┌─────┐ │  │ ┌─────┐ │                  │ ┌─────┐  ┌────────────┐ │
  │ │ Web │ │  │ │ API │ │                  │ │ App │  │ Log Agent  │ │
  │ └─────┘ │  │ └─────┘ │                  │ └─────┘  └────────────┘ │
  └─────────┘  └─────────┘                  │   Tightly coupled       │
                                             └──────────────────────────┘
  Scale independently                        Must scale together
  Different lifecycles                       Same lifecycle
```

**Rule of thumb:** Use multiple containers in a pod only when they are **tightly coupled** and must share resources (volumes, network).

---

## Pod Lifecycle

### Pod Phases

```
┌──────────────────────────────────────────────────────────────────┐
│                        POD LIFECYCLE                              │
│                                                                  │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │ Pending  │───▶│ Running  │───▶│Succeeded │    │  Failed  │   │
│  │          │    │          │    │(exit 0)  │    │(exit ≠0) │   │
│  └──────────┘    └────┬─────┘    └──────────┘    └──────────┘   │
│       │               │                                         │
│       │               │  Container crashes                      │
│       │               └──────────┐                              │
│       │                          ▼                              │
│       │               ┌──────────────────┐                      │
│       │               │  CrashLoopBackOff│                      │
│       │               │  (restart with   │                      │
│       │               │   backoff delay) │                      │
│       │               └──────────────────┘                      │
│       │                                                         │
│       └── Reasons for Pending:                                  │
│           • Image being pulled                                  │
│           • No node with enough resources                       │
│           • Node selector/affinity unmet                        │
│           • PVC not bound                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Pod Phases Explained

| Phase | Description |
|-------|-------------|
| **Pending** | Pod accepted, but containers not yet running. Scheduling or image pulling in progress. |
| **Running** | Pod bound to a node. At least one container is running or starting. |
| **Succeeded** | All containers terminated successfully (exit code 0). Won't be restarted. |
| **Failed** | All containers terminated, at least one with non-zero exit code. |
| **Unknown** | Pod state cannot be determined (usually node communication failure). |

### Container States

```
┌───────────┐     ┌───────────┐     ┌────────────┐
│  Waiting  │────▶│  Running  │────▶│ Terminated │
│           │     │           │     │            │
│ • Pulling │     │ • Running │     │ • Completed│
│   image   │     │   normally│     │ • Error    │
│ • Creating│     │           │     │ • OOM      │
│   sandbox │     │           │     │            │
└───────────┘     └─────┬─────┘     └────────────┘
                        │
                  Container crashes
                        │
                  ┌─────▼─────┐
                  │ Restart   │
                  │ Policy    │
                  │ applies   │
                  └───────────┘
```

### Restart Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| `Always` (default) | Always restart on exit | Long-running services |
| `OnFailure` | Restart only on non-zero exit | Jobs, batch processing |
| `Never` | Never restart | One-shot tasks, debugging |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  restartPolicy: OnFailure  # Always | OnFailure | Never
  containers:
  - name: worker
    image: busybox
    command: ["sh", "-c", "echo done"]
```

---

## Multi-Container Pods

### Common Patterns

```
┌───────────────────────────────────────────────────────────────────┐
│                   MULTI-CONTAINER POD PATTERNS                    │
│                                                                   │
│  1. SIDECAR PATTERN                                              │
│  ┌──────────────────────────────────┐                            │
│  │            Pod                   │                            │
│  │  ┌──────────┐  ┌──────────────┐  │                            │
│  │  │   App    │  │   Sidecar    │  │                            │
│  │  │ Container│  │  Container   │  │                            │
│  │  │          │──▶  Log shipper │  │  Extends app functionality │
│  │  │  Writes  │  │  (fluentd)  │  │  without modifying it      │
│  │  │  logs to │  │             │  │                            │
│  │  │  /var/log│  │  Reads from │  │                            │
│  │  └──────────┘  │  /var/log   │  │                            │
│  │        │       └──────────────┘  │                            │
│  │        └──shared volume──┘       │                            │
│  └──────────────────────────────────┘                            │
│                                                                   │
│  2. AMBASSADOR PATTERN                                           │
│  ┌──────────────────────────────────┐                            │
│  │            Pod                   │                            │
│  │  ┌──────────┐  ┌──────────────┐  │                            │
│  │  │   App    │  │  Ambassador  │  │                            │
│  │  │ Container│  │  Container   │  │                            │
│  │  │          │──▶  DB Proxy    │──▶ External DB                │
│  │  │ connects │  │  (pgbouncer) │  │  Proxies connections       │
│  │  │ to       │  │             │  │  to external services      │
│  │  │ localhost│  │             │  │                            │
│  │  └──────────┘  └──────────────┘  │                            │
│  └──────────────────────────────────┘                            │
│                                                                   │
│  3. ADAPTER PATTERN                                              │
│  ┌──────────────────────────────────┐                            │
│  │            Pod                   │                            │
│  │  ┌──────────┐  ┌──────────────┐  │                            │
│  │  │   App    │  │   Adapter    │  │                            │
│  │  │ Container│  │  Container   │  │                            │
│  │  │          │──▶  Prometheus  │──▶ Monitoring                 │
│  │  │ Custom   │  │  Exporter   │  │  Transforms output to      │
│  │  │ metrics  │  │             │  │  a standard format         │
│  │  └──────────┘  └──────────────┘  │                            │
│  └──────────────────────────────────┘                            │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### Init Containers

Init containers run **before** the main application containers start. They run to completion, one after another.

```
┌──────────────────────────────────────────────────────────────┐
│                    INIT CONTAINERS                            │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────────┐    │
│  │  Init 1  │───▶│  Init 2  │───▶│  Main Containers    │    │
│  │          │    │          │    │  (start together)    │    │
│  │ Wait for │    │ Clone    │    │                      │    │
│  │ DB ready │    │ config   │    │  App + Sidecar       │    │
│  │          │    │ repo     │    │                      │    │
│  └──────────┘    └──────────┘    └──────────────────────┘    │
│                                                              │
│  Rules:                                                      │
│  • Run sequentially (not in parallel)                        │
│  • Each must complete successfully before next starts        │
│  • If any fails, pod restarts (per restartPolicy)            │
│  • Run every time a pod starts (not just the first time)     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Init Container Use Cases:**
- Wait for a dependent service (database, API) to be ready
- Clone a Git repository into a shared volume
- Register with a remote service
- Delay startup until preconditions are met
- Generate config files from templates

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.36
    command: ['sh', '-c', 'until nc -z postgres-svc 5432; do echo waiting for db; sleep 2; done']
  - name: clone-repo
    image: alpine/git
    command: ['git', 'clone', 'https://github.com/example/config.git', '/config']
    volumeMounts:
    - name: config-vol
      mountPath: /config
  containers:
  - name: app
    image: my-app:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: config-vol
      mountPath: /app/config
  volumes:
  - name: config-vol
    emptyDir: {}
```

### Sidecar Container Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logging
spec:
  containers:
  - name: web
    image: nginx:1.25
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-shipper
    image: fluentd:v1.16
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
      readOnly: true
    - name: fluentd-config
      mountPath: /fluentd/etc
  volumes:
  - name: shared-logs
    emptyDir: {}
  - name: fluentd-config
    configMap:
      name: fluentd-config
```

---

## Pod YAML Structure

### Complete Pod Specification Anatomy

```yaml
apiVersion: v1                     # API group/version
kind: Pod                          # Resource type
metadata:                          # ── METADATA SECTION ──
  name: my-app                     # Pod name (required)
  namespace: production            # Namespace (default: "default")
  labels:                          # Key-value pairs for selection
    app: web
    environment: prod
    version: v2.1
  annotations:                     # Non-identifying metadata
    description: "Production web server"
    owner: "team-platform"
spec:                              # ── SPEC SECTION ──
  restartPolicy: Always            # Always | OnFailure | Never
  terminationGracePeriodSeconds: 30

  # Node selection
  nodeSelector:
    disktype: ssd

  # Init containers (run first, sequentially)
  initContainers:
  - name: init-db-check
    image: busybox:1.36
    command: ['sh', '-c', 'until nc -z db-svc 5432; do sleep 2; done']

  # Main containers
  containers:
  - name: web                      # Container name (required)
    image: nginx:1.25              # Image (required)
    imagePullPolicy: IfNotPresent  # Always | IfNotPresent | Never

    ports:
    - containerPort: 80            # Informational, not enforced
      protocol: TCP
      name: http

    # Environment variables
    env:
    - name: DB_HOST
      value: "postgres-svc"
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password

    # Resource management
    resources:
      requests:                    # Minimum guaranteed
        cpu: "250m"                # 250 millicores = 0.25 CPU
        memory: "128Mi"            # 128 Mebibytes
      limits:                      # Maximum allowed
        cpu: "500m"
        memory: "256Mi"

    # Health checks
    livenessProbe:                 # Is container alive?
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 10
    readinessProbe:                # Is container ready for traffic?
      httpGet:
        path: /ready
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:                  # Has container started?
      httpGet:
        path: /healthz
        port: 80
      failureThreshold: 30
      periodSeconds: 10

    # Volume mounts
    volumeMounts:
    - name: app-data
      mountPath: /data
    - name: config-volume
      mountPath: /etc/config
      readOnly: true

    # Security context (container level)
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false

  # Volumes
  volumes:
  - name: app-data
    persistentVolumeClaim:
      claimName: app-pvc
  - name: config-volume
    configMap:
      name: app-config

  # Image pull credentials
  imagePullSecrets:
  - name: registry-creds

  # DNS settings
  dnsPolicy: ClusterFirst

  # Service account
  serviceAccountName: app-sa

  # Security context (pod level)
  securityContext:
    fsGroup: 2000
```

---

## Pod Management Commands

### Essential kubectl Commands for Pods

```bash
# ═══════════════════════════════════════════════════
# CREATING PODS
# ═══════════════════════════════════════════════════

# Create a pod imperatively (quick testing)
kubectl run nginx --image=nginx:1.25
kubectl run nginx --image=nginx:1.25 --port=80
kubectl run nginx --image=nginx:1.25 --dry-run=client -o yaml > pod.yaml

# Create from YAML file
kubectl apply -f pod.yaml
kubectl create -f pod.yaml     # fails if already exists

# ═══════════════════════════════════════════════════
# VIEWING PODS
# ═══════════════════════════════════════════════════

# List pods
kubectl get pods                         # Current namespace
kubectl get pods -A                      # All namespaces
kubectl get pods -n kube-system          # Specific namespace
kubectl get pods -o wide                 # Show node, IP
kubectl get pods -o yaml                 # Full YAML output
kubectl get pods --show-labels           # Show labels
kubectl get pods -l app=web              # Filter by label
kubectl get pods -w                      # Watch changes live
kubectl get pods --sort-by=.status.startTime  # Sort by start time
kubectl get pods --field-selector=status.phase=Running  # Field filter

# Detailed info
kubectl describe pod nginx               # Full details + events
kubectl get pod nginx -o jsonpath='{.status.podIP}'  # Extract specific field

# ═══════════════════════════════════════════════════
# DEBUGGING PODS
# ═══════════════════════════════════════════════════

# View logs
kubectl logs nginx                       # Current logs
kubectl logs nginx -f                    # Follow logs (tail -f)
kubectl logs nginx --previous            # Previous container's logs
kubectl logs nginx -c sidecar            # Specific container in multi-container pod
kubectl logs nginx --since=1h            # Logs from last hour
kubectl logs nginx --tail=50             # Last 50 lines

# Execute commands inside a pod
kubectl exec nginx -- ls /etc            # Run a command
kubectl exec -it nginx -- /bin/bash      # Interactive shell
kubectl exec -it nginx -c sidecar -- sh  # Specific container

# Port forwarding (local development/debugging)
kubectl port-forward nginx 8080:80       # localhost:8080 → pod:80
kubectl port-forward pod/nginx 8080:80 9090:9090  # Multiple ports

# Copy files
kubectl cp nginx:/var/log/nginx/access.log ./access.log  # From pod
kubectl cp ./config.txt nginx:/etc/config/config.txt      # To pod

# ═══════════════════════════════════════════════════
# MODIFYING PODS
# ═══════════════════════════════════════════════════

# Edit a pod (limited fields can be changed)
kubectl edit pod nginx

# Add/update labels
kubectl label pod nginx env=production
kubectl label pod nginx env=staging --overwrite

# Add annotations
kubectl annotate pod nginx description="web server"

# ═══════════════════════════════════════════════════
# DELETING PODS
# ═══════════════════════════════════════════════════

# Delete a pod
kubectl delete pod nginx
kubectl delete pod nginx --grace-period=0 --force  # Force delete
kubectl delete pods --all                           # All pods in namespace
kubectl delete -f pod.yaml                          # Delete from file
```

### Quick Reference: Pod Status Meanings

```
┌──────────────────┬──────────────────────────────────────────────┐
│ STATUS           │ MEANING                                      │
├──────────────────┼──────────────────────────────────────────────┤
│ Pending          │ Waiting for scheduling or image pull          │
│ ContainerCreating│ Container being created (image pulled)        │
│ Running          │ At least one container is running             │
│ Completed        │ All containers exited with code 0             │
│ Error            │ Container exited with error                   │
│ CrashLoopBackOff │ Container crashed, waiting to restart         │
│ ImagePullBackOff │ Cannot pull the container image               │
│ ErrImagePull     │ Error pulling image (first attempt)           │
│ OOMKilled        │ Container killed due to out-of-memory         │
│ Evicted          │ Pod evicted due to node resource pressure     │
│ Terminating      │ Pod is being deleted (grace period)           │
└──────────────────┴──────────────────────────────────────────────┘
```

---

## Static Pods

Static pods are managed **directly by the kubelet** on a specific node, without the API server observing them. The kubelet watches a specific directory for pod manifests.

```
┌────────────────────────────────────────────────────────────────┐
│                       STATIC PODS                              │
│                                                                │
│   Normal Pod:                                                  │
│   User ──▶ API Server ──▶ Scheduler ──▶ kubelet ──▶ Container │
│                                                                │
│   Static Pod:                                                  │
│   Manifest file ──▶ kubelet ──▶ Container                     │
│   (on disk)          (watches directory)                       │
│                                                                │
│   Location: /etc/kubernetes/manifests/  (default)              │
│                                                                │
│   ┌─────────────────────────────────┐                          │
│   │  /etc/kubernetes/manifests/     │                          │
│   │  ├── kube-apiserver.yaml        │  ← Control plane        │
│   │  ├── kube-controller-manager.yaml│    components run       │
│   │  ├── kube-scheduler.yaml        │    as static pods!      │
│   │  └── etcd.yaml                  │                          │
│   └─────────────────────────────────┘                          │
│                                                                │
│   Key facts:                                                   │
│   • kubelet creates a "mirror pod" in API server               │
│   • Mirror pod is read-only (can't delete via API)             │
│   • Static pod name = <pod-name>-<node-name>                   │
│   • Used for control plane components (kubeadm)                │
│   • Useful when API server is unavailable                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Creating a Static Pod

```bash
# Step 1: Find the static pod path
cat /var/lib/kubelet/config.yaml | grep staticPodPath
# staticPodPath: /etc/kubernetes/manifests

# Step 2: Create a manifest in that directory
cat <<EOF > /etc/kubernetes/manifests/static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: static
spec:
  containers:
  - name: web
    image: nginx:1.25
    ports:
    - containerPort: 80
EOF

# Step 3: kubelet automatically detects and starts the pod
kubectl get pods
# NAME                     READY   STATUS    AGE
# static-web-worker-1      1/1     Running   10s

# Step 4: To delete, remove the file
rm /etc/kubernetes/manifests/static-web.yaml
```

---

## Real-World Scenarios

### Scenario 1: Debugging a CrashLoopBackOff Pod

```bash
# 1. Check pod status
kubectl get pods
# NAME        READY   STATUS             RESTARTS   AGE
# my-app      0/1     CrashLoopBackOff   5          3m

# 2. Describe the pod for events
kubectl describe pod my-app
# Events:
#   Warning  BackOff  kubelet  Back-off restarting failed container

# 3. Check logs from the crashed container
kubectl logs my-app --previous
# Error: cannot connect to database at db-host:5432

# 4. Root cause: Database service name is wrong
# Fix the environment variable and redeploy
```

### Scenario 2: Multi-Container Pod for Web App with Metrics

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-metrics
  labels:
    app: web
spec:
  containers:
  # Main application
  - name: web
    image: my-web-app:2.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: tmp
      mountPath: /tmp

  # Prometheus metrics exporter (adapter pattern)
  - name: metrics-exporter
    image: prom/nginx-exporter:0.11
    args:
    - '-nginx.scrape-uri=http://localhost:8080/metrics'
    ports:
    - containerPort: 9113
      name: metrics

  # Log collector (sidecar pattern)
  - name: log-collector
    image: fluentbit:2.1
    volumeMounts:
    - name: app-logs
      mountPath: /var/log/app
      readOnly: true

  volumes:
  - name: tmp
    emptyDir: {}
  - name: app-logs
    emptyDir: {}
```

---

## Troubleshooting Tips

| Problem | Check | Solution |
|---------|-------|----------|
| Pod in `Pending` | `kubectl describe pod` Events | Check resources, node selectors, PVC bindings |
| `ImagePullBackOff` | Image name, tag, registry | Fix image name, add imagePullSecrets |
| `CrashLoopBackOff` | `kubectl logs --previous` | Fix application error, check env vars, mounts |
| `OOMKilled` | Container memory limit | Increase memory limit or optimize app |
| Pod `Evicted` | Node resource pressure | Check node disk/memory, add resource requests |
| Container won't start | `kubectl describe pod` | Check command/args, security context, volumes |
| Can't exec into pod | Container has no shell | Use debug containers: `kubectl debug` |
| Init container stuck | `kubectl logs <pod> -c <init>` | Check init container logic, dependent services |

```bash
# The ultimate debugging flow:
kubectl get pods                          # 1. What's the status?
kubectl describe pod <name>               # 2. What do events say?
kubectl logs <name>                       # 3. What do logs say?
kubectl logs <name> --previous            # 4. Previous container logs?
kubectl exec -it <name> -- sh             # 5. Can I shell in?
kubectl get events --sort-by=.lastTimestamp # 6. Cluster events?
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Pod** | Smallest deployable unit; wraps 1+ containers sharing network & storage |
| **Pod IP** | Each pod gets a unique IP; containers in a pod share it |
| **Lifecycle** | Pending → Running → Succeeded/Failed |
| **Restart Policy** | Always (default), OnFailure, Never |
| **Init Containers** | Run before main containers, sequentially, must succeed |
| **Sidecar** | Extends app functionality (logging, proxying, monitoring) |
| **Ambassador** | Proxies outbound connections to external services |
| **Adapter** | Transforms output to a standard format |
| **Static Pods** | Managed by kubelet directly, not by API server |
| **Mirror Pods** | API server's read-only representation of static pods |
| **Labels** | Key-value pairs for grouping and selecting pods |
| **Annotations** | Non-identifying metadata (descriptions, tooling info) |

---

## Quick Revision Questions

1. **Can two containers in the same pod listen on the same port?**
   <details><summary>Answer</summary>No. Since containers in the same pod share the network namespace (same IP), two containers cannot bind to the same port. They communicate via localhost but must use different ports.</details>

2. **What is the difference between init containers and sidecar containers?**
   <details><summary>Answer</summary>Init containers run sequentially before main containers start and must complete successfully. They're used for setup tasks. Sidecar containers run alongside the main container for the entire pod lifetime and provide supporting functionality (logging, proxying, etc.).</details>

3. **A pod is stuck in `Pending` — list three possible causes.**
   <details><summary>Answer</summary>1) Insufficient CPU/memory on any node (no node can satisfy resource requests). 2) PersistentVolumeClaim not bound. 3) nodeSelector or affinity rules don't match any available node. Also: taints without tolerations, resource quotas exceeded.</details>

4. **How are static pods different from regular pods?**
   <details><summary>Answer</summary>Static pods are managed directly by the kubelet from manifest files in a directory (e.g., /etc/kubernetes/manifests/), not by the API server. The kubelet creates a "mirror pod" in the API server for visibility, but it's read-only. Control plane components (api-server, scheduler, etc.) typically run as static pods in kubeadm clusters.</details>

5. **What happens when a container exceeds its memory limit?**
   <details><summary>Answer</summary>The container is killed by the kernel's OOM (Out of Memory) killer. The pod status shows OOMKilled. Based on the restart policy, Kubernetes may restart the container (with exponential backoff if it keeps crashing — CrashLoopBackOff).</details>

6. **Write a `kubectl` command to get the IP address of a pod named `web-app`.**
   <details><summary>Answer</summary>`kubectl get pod web-app -o jsonpath='{.status.podIP}'` or `kubectl get pod web-app -o wide` (shows IP in the output table).</details>

---

[← Previous: Kubernetes Architecture](../02-Kubernetes-Architecture/01-architecture-overview.md) | [Back to README](../README.md) | [Next: Workload Resources →](../04-Workload-Resources/01-workload-resources.md)
