# Unit 8: Scheduling

[← Previous: Configuration](../07-Configuration/01-configuration.md) | [Back to README](../README.md) | [Next: Security →](../09-Security/01-security.md)

---

## Chapter Overview

Kubernetes scheduling is the process of assigning Pods to Nodes. The **kube-scheduler** selects optimal nodes based on resource requirements, constraints, and policies. This unit covers node selectors, affinity/anti-affinity, taints and tolerations, resource management, and priority/preemption.

---

## 8.1 The Scheduling Process

```
┌──────────────────────────────────────────────────────────────┐
│               KUBERNETES SCHEDULING PIPELINE                  │
│                                                              │
│  Pod Created                                                 │
│  (nodeName = "")                                             │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────┐                                            │
│  │  Scheduling  │   1. Filter: Remove nodes that don't       │
│  │    Queue     │      meet requirements (resources,         │
│  │              │      taints, affinity, etc.)                │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐               ┌──────────────────┐         │
│  │   FILTER     │──────────────▶│ Feasible Nodes   │         │
│  │   Phase      │  Remove unfit │ [node1, node3,   │         │
│  │              │  nodes        │  node5]           │         │
│  └──────────────┘               └────────┬─────────┘         │
│                                          │                   │
│                                          ▼                   │
│                                 ┌──────────────────┐         │
│                                 │   SCORE Phase    │         │
│                                 │ Rank remaining   │         │
│                                 │ nodes by fitness │         │
│                                 └────────┬─────────┘         │
│                                          │                   │
│                                          ▼                   │
│                                 ┌──────────────────┐         │
│                                 │ Best Node: node3 │         │
│                                 │ (highest score)  │         │
│                                 └────────┬─────────┘         │
│                                          │                   │
│                                          ▼                   │
│                                 ┌──────────────────┐         │
│                                 │   BIND Phase     │         │
│                                 │ Set pod.spec.    │         │
│                                 │   nodeName=node3 │         │
│                                 └──────────────────┘         │
│                                                              │
│  Filter Plugins: NodeResources, NodePorts, NodeAffinity,     │
│    TaintToleration, PodTopologySpread, etc.                  │
│  Score Plugins: LeastAllocated, BalancedAllocation,           │
│    ImageLocality, NodeAffinity, etc.                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.2 Node Selectors

The simplest way to constrain pods to nodes with specific **labels**.

```bash
# Label a node
kubectl label nodes worker-1 disktype=ssd
kubectl label nodes worker-2 disktype=hdd
kubectl label nodes worker-1 gpu=nvidia

# View node labels
kubectl get nodes --show-labels
kubectl get nodes -L disktype,gpu
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  nodeSelector:                   # Simple label matching
    disktype: ssd                 # Must match node label exactly
  containers:
  - name: app
    image: my-app:1.0
```

```
┌──────────────────────────────────────────────────────────────┐
│  nodeSelector: disktype=ssd                                  │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ worker-1 │  │ worker-2 │  │ worker-3 │                   │
│  │ ssd ✓    │  │ hdd ✗    │  │ no label │                   │
│  │          │  │          │  │    ✗     │                    │
│  │ ◄── Pod  │  │          │  │          │                    │
│  │  placed  │  │          │  │          │                    │
│  │  here    │  │          │  │          │                    │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.3 Node Affinity / Anti-Affinity

More expressive than nodeSelector — supports operators, soft/hard preferences.

```
┌──────────────────────────────────────────────────────────────┐
│               NODE AFFINITY TYPES                             │
│                                                              │
│  requiredDuringSchedulingIgnoredDuringExecution               │
│  ├── HARD constraint (must be satisfied)                     │
│  └── "IgnoredDuringExecution" = running pods stay if         │
│       node labels change                                     │
│                                                              │
│  preferredDuringSchedulingIgnoredDuringExecution               │
│  ├── SOFT constraint (prefer, but not required)              │
│  └── Scheduler tries to match, falls back if not possible   │
│                                                              │
│  Operators:                                                  │
│  ┌────────┬─────────────────────────────────────────┐        │
│  │ In     │ Label value is in the set               │        │
│  │ NotIn  │ Label value is NOT in the set           │        │
│  │ Exists │ Label key exists (any value)            │        │
│  │ DoesNotExist │ Label key does not exist          │        │
│  │ Gt     │ Label value > specified (numeric)       │        │
│  │ Lt     │ Label value < specified (numeric)       │        │
│  └────────┴─────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Node Affinity YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      # HARD: Must be on a node in us-east-1a or us-east-1b
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - us-east-1a
            - us-east-1b

      # SOFT: Prefer nodes with SSD (weight 1-100)
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80                 # Higher = stronger preference
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      - weight: 20
        preference:
          matchExpressions:
          - key: gpu
            operator: Exists

  containers:
  - name: app
    image: my-app:1.0
```

### Node Anti-Affinity

```yaml
# Keep pod AWAY from specific nodes
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-role
          operator: NotIn           # Anti-affinity via NotIn
          values:
          - database-dedicated
        - key: maintenance
          operator: DoesNotExist    # Not on nodes marked for maintenance
```

---

## 8.4 Pod Affinity / Anti-Affinity

Controls pod placement **relative to other pods** (co-locate or spread apart).

```
┌──────────────────────────────────────────────────────────────┐
│            POD AFFINITY vs ANTI-AFFINITY                      │
│                                                              │
│  Pod Affinity (co-locate):                                   │
│  "Schedule me on the SAME node/zone as pods with label X"    │
│                                                              │
│  ┌── Node 1 ──────┐  ┌── Node 2 ──────┐                     │
│  │  ┌─────────┐   │  │                │                     │
│  │  │ cache   │   │  │                │                     │
│  │  │ app=    │   │  │                │                     │
│  │  │ redis   │   │  │                │                     │
│  │  └─────────┘   │  │                │                     │
│  │  ┌─────────┐   │  │                │                     │
│  │  │ web-app │   │  │                │                     │
│  │  │ (affin- │◄──┼──┤ "Put me near  │                     │
│  │  │  ity to │   │  │  app=redis"   │                     │
│  │  │  redis) │   │  │                │                     │
│  │  └─────────┘   │  │                │                     │
│  └─────────────────┘  └────────────────┘                     │
│                                                              │
│  Pod Anti-Affinity (spread apart):                           │
│  "Don't put me on the SAME node/zone as pods with label X"   │
│                                                              │
│  ┌── Node 1 ──────┐  ┌── Node 2 ──────┐                     │
│  │  ┌─────────┐   │  │  ┌─────────┐   │                     │
│  │  │ web-1   │   │  │  │ web-2   │   │                     │
│  │  │ app=web │   │  │  │ app=web │   │                     │
│  │  └─────────┘   │  │  └─────────┘   │                     │
│  │                │  │                │                     │
│  │  anti-affinity │  │  anti-affinity │                     │
│  │  ensures these │  │  are on        │                     │
│  │  separate nodes│  │  separate nodes│                     │
│  └─────────────────┘  └────────────────┘                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Pod Affinity YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        # Co-locate with cache pods on same node
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis-cache
            topologyKey: kubernetes.io/hostname  # Same node

        # Spread web replicas across nodes
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - web
              topologyKey: kubernetes.io/hostname  # Different nodes

      containers:
      - name: web
        image: web-app:1.0
```

### topologyKey Explained

```
┌──────────────────────────────────────────────────────────────┐
│                     topologyKey                               │
│                                                              │
│  Defines the "domain" for affinity/anti-affinity:            │
│                                                              │
│  ┌──────────────────────────────┬──────────────────────────┐ │
│  │ topologyKey                  │ Meaning                  │ │
│  ├──────────────────────────────┼──────────────────────────┤ │
│  │ kubernetes.io/hostname       │ Same / different NODE    │ │
│  │ topology.kubernetes.io/zone  │ Same / different ZONE    │ │
│  │ topology.kubernetes.io/region│ Same / different REGION  │ │
│  │ custom-label-key             │ Same / different value   │ │
│  └──────────────────────────────┴──────────────────────────┘ │
│                                                              │
│  Example: topologyKey = zone                                 │
│  ┌───── Zone A ──────┐  ┌───── Zone B ──────┐               │
│  │ Node1    Node2     │  │ Node3    Node4     │               │
│  │ ┌────┐  ┌────┐    │  │ ┌────┐  ┌────┐    │               │
│  │ │pod1│  │pod2│    │  │ │    │  │    │    │               │
│  │ └────┘  └────┘    │  │ └────┘  └────┘    │               │
│  │                    │  │                    │               │
│  │ Anti-affinity      │  │                    │               │
│  │ (zone) keeps pod3  │  │ ◄── pod3 placed   │               │
│  │ out of Zone A      │  │     in Zone B      │               │
│  └────────────────────┘  └────────────────────┘               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.5 Taints and Tolerations

Taints are applied to **nodes** to repel pods. Tolerations are applied to **pods** to allow scheduling on tainted nodes.

```
┌──────────────────────────────────────────────────────────────┐
│           TAINTS AND TOLERATIONS                              │
│                                                              │
│  Taint on Node:    "I repel pods unless they tolerate me"    │
│  Toleration on Pod: "I can tolerate this taint"              │
│                                                              │
│  ┌──────── Node (tainted) ────────┐                          │
│  │                                │                          │
│  │  Taint: gpu=nvidia:NoSchedule  │                          │
│  │                                │                          │
│  │  ┌─────────┐   ┌──────────┐   │                          │
│  │  │ Pod A   │   │ Pod B    │   │                          │
│  │  │ ✗ No    │   │ ✓ Has    │   │                          │
│  │  │tolera-  │   │tolera-   │   │                          │
│  │  │tion     │   │tion for  │   │                          │
│  │  │REJECTED │   │gpu=nvidia│   │                          │
│  │  └─────────┘   │ACCEPTED  │   │                          │
│  │                │          │   │                          │
│  │                └──────────┘   │                          │
│  └────────────────────────────────┘                          │
│                                                              │
│  Taint Effects:                                              │
│  ┌──────────────────┬────────────────────────────────────┐   │
│  │ NoSchedule       │ Don't schedule new pods without    │   │
│  │                  │ matching toleration                │   │
│  │ PreferNoSchedule │ Try to avoid, but allow if needed  │   │
│  │ NoExecute        │ Evict existing + block new pods    │   │
│  │                  │ without toleration                 │   │
│  └──────────────────┴────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Managing Taints

```bash
# Add taint
kubectl taint nodes worker-1 gpu=nvidia:NoSchedule

# Add taint with no value
kubectl taint nodes worker-2 dedicated:NoSchedule

# View taints on a node
kubectl describe node worker-1 | grep -A5 Taints

# Remove taint (add minus at end)
kubectl taint nodes worker-1 gpu=nvidia:NoSchedule-

# Master/control-plane taint (default)
# node-role.kubernetes.io/control-plane:NoSchedule
# This is why pods don't run on master by default
```

### Tolerations in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  tolerations:
  # Exact match
  - key: "gpu"
    operator: "Equal"
    value: "nvidia"
    effect: "NoSchedule"

  # Key exists (any value)
  - key: "dedicated"
    operator: "Exists"
    effect: "NoSchedule"

  # Tolerate NoExecute with time limit
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 300    # Tolerate for 5 min, then evict

  # Tolerate ALL taints (dangerous!)
  # - operator: "Exists"

  containers:
  - name: gpu-app
    image: gpu-app:1.0
```

### Common Built-in Taints

```
┌────────────────────────────────────────────────────────────┐
│  Automatically Applied Taints (by Node Controller)         │
│                                                            │
│  node.kubernetes.io/not-ready        → Node not ready      │
│  node.kubernetes.io/unreachable      → Node unreachable    │
│  node.kubernetes.io/memory-pressure  → Node low on memory  │
│  node.kubernetes.io/disk-pressure    → Node low on disk    │
│  node.kubernetes.io/pid-pressure     → Too many processes  │
│  node.kubernetes.io/network-unavailable → Network down     │
│  node.kubernetes.io/unschedulable    → Node cordoned       │
│                                                            │
│  K8s automatically adds tolerations to all pods:           │
│  - not-ready: tolerationSeconds=300 (5 min)                │
│  - unreachable: tolerationSeconds=300 (5 min)              │
│  This gives nodes time to recover before evicting pods     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 8.6 Resource Requests and Limits

```
┌──────────────────────────────────────────────────────────────┐
│          RESOURCE REQUESTS vs LIMITS                          │
│                                                              │
│  Requests: Guaranteed minimum resources                      │
│  Limits: Maximum resources allowed                           │
│                                                              │
│  ┌─────── Container ────────────────────────┐                │
│  │                                          │                │
│  │  CPU:                                    │                │
│  │  ├── Request: 250m ─────┤                │                │
│  │  │   (guaranteed)       │                │                │
│  │  ├── Actual use: 400m ──┤                │                │
│  │  │   (burst allowed)    │                │                │
│  │  ├── Limit: 500m ───────┤                │                │
│  │  │   (throttled here)   │                │                │
│  │  └──────────────────────┘                │                │
│  │                                          │                │
│  │  Memory:                                 │                │
│  │  ├── Request: 128Mi ────┤                │                │
│  │  │   (guaranteed)       │                │                │
│  │  ├── Actual use: 200Mi ─┤                │                │
│  │  │                      │                │                │
│  │  ├── Limit: 256Mi ──────┤                │                │
│  │  │   (OOMKilled above!) │                │                │
│  │  └──────────────────────┘                │                │
│  │                                          │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
│  CPU: Throttled (not killed) when exceeding limit            │
│  Memory: OOMKilled when exceeding limit                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Resource Units

```
┌──────────────────────────────────────────────────────────────┐
│  CPU Units:                                                  │
│  ├── 1 CPU = 1 vCPU (AWS) = 1 Core (GCP) = 1 hyperthread   │
│  ├── 1000m (millicores) = 1 CPU                             │
│  ├── 500m = 0.5 CPU                                         │
│  ├── 250m = 0.25 CPU (quarter core)                         │
│  └── 100m = 0.1 CPU (minimum practical value)               │
│                                                              │
│  Memory Units:                                               │
│  ├── Ki = Kibibyte (1024 bytes)    │ K = Kilobyte (1000)    │
│  ├── Mi = Mebibyte (1024² bytes)   │ M = Megabyte           │
│  ├── Gi = Gibibyte (1024³ bytes)   │ G = Gigabyte           │
│  └── Ti = Tebibyte (1024⁴ bytes)   │ T = Terabyte           │
│                                                              │
│  Best practice: Use Mi/Gi (binary) for consistency           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Setting Resources

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    resources:
      requests:                    # Minimum guaranteed
        cpu: "250m"                # 0.25 CPU core
        memory: "128Mi"            # 128 MiB
        ephemeral-storage: "1Gi"   # 1 GiB local storage
      limits:                      # Maximum allowed
        cpu: "500m"                # 0.5 CPU core
        memory: "256Mi"            # 256 MiB
        ephemeral-storage: "2Gi"   # 2 GiB local storage
```

### QoS Classes

Kubernetes assigns Quality of Service classes based on resource settings:

```
┌──────────────────────────────────────────────────────────────┐
│                QoS CLASSES                                    │
│                                                              │
│  ┌────────────┬────────────────────┬─────────────────────┐   │
│  │   Class    │    Condition       │  Eviction Priority   │   │
│  ├────────────┼────────────────────┼─────────────────────┤   │
│  │Guaranteed  │ requests = limits  │ Last to be evicted   │   │
│  │            │ (for ALL containers│ (highest priority)   │   │
│  │            │  in the pod)       │                     │   │
│  ├────────────┼────────────────────┼─────────────────────┤   │
│  │Burstable   │ requests < limits  │ Middle priority      │   │
│  │            │ (at least one      │                     │   │
│  │            │  request set)      │                     │   │
│  ├────────────┼────────────────────┼─────────────────────┤   │
│  │BestEffort  │ No requests or     │ First to be evicted  │   │
│  │            │ limits set         │ (lowest priority)    │   │
│  └────────────┴────────────────────┴─────────────────────┘   │
│                                                              │
│  Under memory pressure, K8s evicts: BestEffort first,        │
│  then Burstable, then Guaranteed last.                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Guaranteed QoS (requests = limits)
resources:
  requests:
    cpu: "500m"
    memory: "256Mi"
  limits:
    cpu: "500m"        # Same as request
    memory: "256Mi"    # Same as request

# Burstable QoS (requests < limits)
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"        # Higher than request
    memory: "256Mi"    # Higher than request

# BestEffort QoS (no resources specified)
# Just omit the resources section entirely
```

### LimitRange (Namespace Defaults)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: dev
spec:
  limits:
  - type: Container
    default:               # Default limits if not specified
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:        # Default requests if not specified
      cpu: "100m"
      memory: "128Mi"
    max:                   # Maximum allowed
      cpu: "2"
      memory: "2Gi"
    min:                   # Minimum allowed
      cpu: "50m"
      memory: "64Mi"
  - type: Pod
    max:
      cpu: "4"
      memory: "4Gi"
```

### ResourceQuota (Namespace Totals)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: dev
spec:
  hard:
    # Compute resources
    requests.cpu: "10"            # Total CPU requests
    requests.memory: "20Gi"       # Total memory requests
    limits.cpu: "20"              # Total CPU limits
    limits.memory: "40Gi"         # Total memory limits

    # Object counts
    pods: "50"                    # Max pods
    services: "20"                # Max services
    secrets: "100"                # Max secrets
    configmaps: "100"             # Max configmaps
    persistentvolumeclaims: "30"  # Max PVCs
    services.loadbalancers: "2"   # Max LB services
    services.nodeports: "5"       # Max NodePort services
```

```bash
# View quota usage
kubectl describe resourcequota team-quota -n dev
kubectl get resourcequota -n dev
```

---

## 8.7 Priority and Preemption

Higher-priority pods can **preempt** (evict) lower-priority pods when resources are scarce.

```
┌──────────────────────────────────────────────────────────────┐
│              PRIORITY AND PREEMPTION                          │
│                                                              │
│  PriorityClass defines a priority value (0 - 1,000,000,000) │
│                                                              │
│  Scenario: Node is full, high-priority pod arrives           │
│                                                              │
│  Before:                         After:                      │
│  ┌────────── Node ──────┐       ┌────────── Node ──────┐    │
│  │ ┌──────┐ ┌──────┐   │       │ ┌──────┐ ┌──────┐   │    │
│  │ │Low-P │ │Low-P │   │       │ │Med-P │ │HIGH-P│   │    │
│  │ │Pod A │ │Pod B │   │       │ │Pod A │ │Pod C │   │    │
│  │ │pri=10│ │pri=10│   │       │ │pri=50│ │pri=  │   │    │
│  │ └──────┘ └──────┘   │       │ └──────┘ │1000  │   │    │
│  │ ┌──────┐             │       │          │(new) │   │    │
│  │ │Med-P │  NO ROOM    │       │          └──────┘   │    │
│  │ │Pod A │  for new    │       │ Low-P Pod B evicted │    │
│  │ │pri=50│  high-pri   │       │ to make room        │    │
│  │ └──────┘  pod        │       └─────────────────────┘    │
│  └──────────────────────┘                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Define PriorityClass
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000                # Higher = more important
globalDefault: false          # Not default for all pods
preemptionPolicy: PreemptLowerPriority  # or Never
description: "For critical production workloads"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100
globalDefault: false
description: "For batch and non-critical jobs"
---
# Use PriorityClass in Pod
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: critical-app:1.0
    resources:
      requests:
        cpu: "1"
        memory: "1Gi"
```

---

## 8.8 Pod Topology Spread Constraints

Distribute pods across topology domains (zones, nodes, regions) more evenly.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 6
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      topologySpreadConstraints:
      - maxSkew: 1                         # Max diff between zones
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule   # or ScheduleAnyway
        labelSelector:
          matchLabels:
            app: web
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway  # Soft constraint
        labelSelector:
          matchLabels:
            app: web
      containers:
      - name: web
        image: web:1.0
```

```
┌──────────────────────────────────────────────────────────────┐
│  topologySpreadConstraints: maxSkew=1, topologyKey=zone      │
│                                                              │
│  ┌─── Zone A ───┐  ┌─── Zone B ───┐  ┌─── Zone C ───┐      │
│  │  web-1       │  │  web-2       │  │  web-3       │      │
│  │  web-4       │  │  web-5       │  │  web-6       │      │
│  │  (2 pods)    │  │  (2 pods)    │  │  (2 pods)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  maxSkew=1 ensures difference between                        │
│  any two zones is at most 1 pod                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.9 Manual Scheduling and nodeName

```yaml
# Bypass scheduler entirely — assign to specific node
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  nodeName: worker-2            # Direct assignment (no scheduler)
  containers:
  - name: app
    image: my-app:1.0
```

```bash
# Cordon/Uncordon + Drain
kubectl cordon worker-1          # Mark node unschedulable (no new pods)
kubectl uncordon worker-1        # Mark node schedulable again
kubectl drain worker-1 \         # Evict all pods + cordon
  --ignore-daemonsets \
  --delete-emptydir-data
```

---

## Real-World Scenario: Production Scheduling Strategy

```yaml
# High-priority web app: spread across zones, co-locate with cache,
# use SSD nodes, with proper resource management
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: production-critical
value: 100000
description: "Production-critical workloads"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-web
spec:
  replicas: 6
  selector:
    matchLabels:
      app: prod-web
  template:
    metadata:
      labels:
        app: prod-web
        tier: frontend
    spec:
      priorityClassName: production-critical

      affinity:
        # Prefer SSD nodes
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 80
            preference:
              matchExpressions:
              - key: disktype
                operator: In
                values: ["ssd"]

        # Co-locate with redis cache
        podAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 50
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values: ["redis-cache"]
              topologyKey: kubernetes.io/hostname

        # Spread replicas across nodes
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: ["prod-web"]
            topologyKey: kubernetes.io/hostname

      # Spread across zones
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: prod-web

      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "frontend"
        effect: "NoSchedule"

      containers:
      - name: web
        image: prod-web:2.0
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
          limits:
            cpu: "1"
            memory: "1Gi"
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| Pod stuck in Pending | `kubectl describe pod <name>` → Events | Check node resources, taints, affinity rules |
| No nodes match nodeSelector | `kubectl get nodes --show-labels` | Add missing label to node or fix selector |
| Insufficient CPU/memory | `kubectl describe node` → Allocated resources | Scale cluster, reduce requests, or evict pods |
| Taint prevents scheduling | `kubectl describe node \| grep Taint` | Add toleration to pod or remove taint |
| Pods not spreading evenly | Check topologySpreadConstraints | Verify labels, maxSkew, topologyKey |
| Pod evicted (OOMKilled) | `kubectl describe pod` → LastState | Increase memory limit or fix memory leak |
| Preemption not working | Check PriorityClass values | Ensure higher value and `preemptionPolicy: PreemptLowerPriority` |

```bash
# Useful debugging commands
kubectl describe pod <pod> | grep -A20 Events
kubectl get pod <pod> -o yaml | grep -A5 conditions
kubectl describe node <node> | grep -A20 "Allocated resources"
kubectl top nodes                        # Node resource usage
kubectl top pods                         # Pod resource usage
kubectl get priorityclass                # List priority classes
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **nodeSelector** | Simple label-based node selection (must match exactly) |
| **Node Affinity** | Expressive node selection with operators (In, NotIn, Exists, Gt, Lt) |
| **required...** | Hard constraint — pod won't schedule without it |
| **preferred...** | Soft constraint — scheduler tries to match, falls back if needed |
| **Pod Affinity** | Co-locate pods with specific labels on same node/zone |
| **Pod Anti-Affinity** | Spread pods apart from pods with specific labels |
| **topologyKey** | Defines domain for affinity: hostname, zone, region |
| **Taints** | Applied to nodes to repel pods (NoSchedule, PreferNoSchedule, NoExecute) |
| **Tolerations** | Applied to pods to allow scheduling on tainted nodes |
| **Requests** | Minimum guaranteed resources (used for scheduling) |
| **Limits** | Maximum allowed (CPU throttled, memory OOMKilled) |
| **QoS Classes** | Guaranteed > Burstable > BestEffort (eviction order) |
| **LimitRange** | Default and max/min resource constraints per namespace |
| **ResourceQuota** | Total resource budget for a namespace |
| **PriorityClass** | Pod priority for preemption (higher value = higher priority) |
| **TopologySpread** | Distribute pods evenly across topology domains |

---

## Quick Revision Questions

1. **What is the difference between `nodeSelector` and `nodeAffinity`?**
   <details><summary>Answer</summary>nodeSelector is a simple key-value label match — the pod MUST go to a node with that exact label. nodeAffinity is more expressive: it supports operators (In, NotIn, Exists, Gt, Lt), can be required (hard) or preferred (soft), and supports multiple expressions with AND/OR logic. nodeAffinity is the recommended approach for most use cases.</details>

2. **A pod is stuck in Pending with the event "0/3 nodes are available: 1 node had taint that the pod didn't tolerate, 2 insufficient cpu." What do you do?**
   <details><summary>Answer</summary>Two issues: 1) One node has a taint — either add a matching toleration to the pod or remove the taint from the node. 2) Two nodes lack sufficient CPU — either reduce the pod's CPU request, free resources by removing/scaling down other pods, or add more nodes to the cluster. Check with: `kubectl describe node` for taints and allocated resources.</details>

3. **What happens when a container exceeds its CPU limit vs its memory limit?**
   <details><summary>Answer</summary>CPU: The container is throttled (CPU cycles are restricted) but NOT killed. It just runs slower. Memory: The container is OOMKilled (Out of Memory Killed) and restarted according to its restartPolicy. This is why setting appropriate memory limits is critical.</details>

4. **How do taints and tolerations differ from node affinity?**
   <details><summary>Answer</summary>Node affinity attracts pods TO specific nodes ("schedule me HERE"). Taints and tolerations repel pods FROM nodes ("stay AWAY unless you tolerate me"). They serve opposite purposes. Taints are set on nodes, tolerations on pods. For dedicated nodes, use both: taint the node (to repel others) AND add nodeSelector/affinity (to attract the right pods).</details>

5. **What are the three QoS classes and when is each assigned?**
   <details><summary>Answer</summary>1) **Guaranteed**: All containers have equal requests and limits for both CPU and memory. Last to be evicted. 2) **Burstable**: At least one container has a request or limit set, but requests ≠ limits. Middle priority. 3) **BestEffort**: No resources specified at all. First to be evicted under pressure. For critical workloads, always use Guaranteed QoS.</details>

6. **How does `topologySpreadConstraints` with `maxSkew: 1` work?**
   <details><summary>Answer</summary>maxSkew defines the maximum allowed difference in pod count between any two topology domains. With maxSkew=1 and topologyKey=zone, if Zone A has 3 pods and Zone B has 2, the skew is 1 (allowed). A new pod would go to Zone B (or a new zone) to maintain even distribution. If `whenUnsatisfiable: DoNotSchedule`, the pod stays Pending if placement would violate maxSkew.</details>

---

[← Previous: Configuration](../07-Configuration/01-configuration.md) | [Back to README](../README.md) | [Next: Security →](../09-Security/01-security.md)
