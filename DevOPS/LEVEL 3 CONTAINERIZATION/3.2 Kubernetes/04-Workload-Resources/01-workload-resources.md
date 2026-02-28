# Unit 4: Workload Resources

[← Previous: Pods](../03-Pods/01-pods.md) | [Back to README](../README.md) | [Next: Services and Networking →](../05-Services-And-Networking/01-services-and-networking.md)

---

## Chapter Overview

While Pods are the atomic unit, you rarely create pods directly in production. **Workload resources** are higher-level abstractions that manage pods for you — handling replication, updates, scheduling, and lifecycle. This unit covers ReplicaSets, Deployments, StatefulSets, DaemonSets, Jobs, and CronJobs.

---

## 4.1 ReplicaSets

A **ReplicaSet** ensures that a specified number of pod replicas are running at all times.

```
┌───────────────────────────────────────────────────────────┐
│                      ReplicaSet                           │
│                  (desired replicas: 3)                    │
│                                                          │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐           │
│   │  Pod 1   │    │  Pod 2   │    │  Pod 3   │           │
│   │ app=web  │    │ app=web  │    │ app=web  │           │
│   └──────────┘    └──────────┘    └──────────┘           │
│                                                          │
│   If Pod 2 dies:                                         │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐           │
│   │  Pod 1   │    │  Pod 2   │    │  Pod 3   │           │
│   │ app=web  │    │  DEAD ⚠  │    │ app=web  │           │
│   └──────────┘    └──────────┘    └──────────┘           │
│                         │                                │
│                   ReplicaSet detects                     │
│                   only 2 running                         │
│                         │                                │
│                   ┌─────▼──────┐                         │
│                   │  Pod 2-new │ ← Creates replacement   │
│                   │  app=web   │                         │
│                   └────────────┘                         │
│                                                          │
└───────────────────────────────────────────────────────────┘
```

### ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
  labels:
    app: web
spec:
  replicas: 3
  selector:                    # MUST match template labels
    matchLabels:
      app: web
  template:                    # Pod template
    metadata:
      labels:
        app: web               # MUST match selector
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
```

**Important:** You almost never create ReplicaSets directly. Use **Deployments** instead, which manage ReplicaSets for you.

```bash
# ReplicaSet commands
kubectl get replicasets         # or: kubectl get rs
kubectl describe rs web-rs
kubectl scale rs web-rs --replicas=5
kubectl delete rs web-rs
```

---

## Deployments

A **Deployment** provides declarative updates for Pods and ReplicaSets. It's the **most common workload resource** in Kubernetes.

```
┌────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT                              │
│                    (manages ReplicaSets)                       │
│                                                                │
│   Deployment: web-app                                          │
│   ┌──────────────────────────────────────────────────────────┐ │
│   │                                                          │ │
│   │  ReplicaSet: web-app-6d4f5b7c8  (current - v2)          │ │
│   │  replicas: 3                                             │ │
│   │  ┌──────┐  ┌──────┐  ┌──────┐                           │ │
│   │  │Pod v2│  │Pod v2│  │Pod v2│                           │ │
│   │  └──────┘  └──────┘  └──────┘                           │ │
│   │                                                          │ │
│   └──────────────────────────────────────────────────────────┘ │
│                                                                │
│   ReplicaSet: web-app-5b8c7d3a1  (old - v1, scaled to 0)    │
│   replicas: 0  (kept for rollback)                           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  strategy:
    type: RollingUpdate         # or Recreate
    rollingUpdate:
      maxSurge: 1               # Max pods over desired count
      maxUnavailable: 0         # Max pods that can be down
  revisionHistoryLimit: 10      # Keep 10 old ReplicaSets for rollback
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
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
```

### Deployment Strategies

```
STRATEGY 1: RollingUpdate (default)
──────────────────────────────────────────
Gradually replaces old pods with new ones.

Step 1:  [v1] [v1] [v1]              ← 3 old pods
Step 2:  [v1] [v1] [v1] [v2]         ← 1 new pod created (maxSurge=1)
Step 3:  [v1] [v1] [v2] [v2]         ← 1 old removed, 1 new added
Step 4:  [v1] [v2] [v2] [v2]         ← continue
Step 5:  [v2] [v2] [v2]              ← all updated

✅ Zero downtime
✅ Gradual rollout
❌ Mixed versions temporarily


STRATEGY 2: Recreate
──────────────────────────────────────────
Kills all old pods before creating new ones.

Step 1:  [v1] [v1] [v1]              ← 3 old pods
Step 2:  [ ] [ ] [ ]                 ← ALL killed (DOWNTIME!)
Step 3:  [v2] [v2] [v2]              ← 3 new pods created

✅ No version mixing
✅ Clean switchover
❌ Downtime during transition
Use for: incompatible schema migrations
```

### Deployment Commands

```bash
# ═══════════════════════════════════════════════════
# CREATE
# ═══════════════════════════════════════════════════
kubectl create deployment web --image=nginx:1.25 --replicas=3
kubectl apply -f deployment.yaml

# Generate YAML without creating
kubectl create deployment web --image=nginx:1.25 --dry-run=client -o yaml > deploy.yaml

# ═══════════════════════════════════════════════════
# UPDATE (triggers rolling update)
# ═══════════════════════════════════════════════════
kubectl set image deployment/web-app web=nginx:1.26
kubectl edit deployment web-app
kubectl apply -f deployment.yaml   # after editing file

# ═══════════════════════════════════════════════════
# SCALE
# ═══════════════════════════════════════════════════
kubectl scale deployment web-app --replicas=5

# ═══════════════════════════════════════════════════
# ROLLOUT MANAGEMENT
# ═══════════════════════════════════════════════════
kubectl rollout status deployment/web-app        # Watch rollout progress
kubectl rollout history deployment/web-app        # View revision history
kubectl rollout undo deployment/web-app           # Rollback to previous
kubectl rollout undo deployment/web-app --to-revision=2  # Specific revision
kubectl rollout pause deployment/web-app          # Pause rollout
kubectl rollout resume deployment/web-app         # Resume rollout

# ═══════════════════════════════════════════════════
# VIEW
# ═══════════════════════════════════════════════════
kubectl get deployments
kubectl describe deployment web-app
kubectl get rs                    # See ReplicaSets created by deployment
kubectl get pods --show-labels    # See pods with labels
```

### Rolling Update in Detail

```
┌──────────────────────────────────────────────────────────────┐
│           ROLLING UPDATE (maxSurge=1, maxUnavailable=0)      │
│                                                              │
│  Time 0:  RS-old(3)                      RS-new(0)           │
│           [v1-a][v1-b][v1-c]             []                  │
│                                                              │
│  Time 1:  RS-old(3)                      RS-new(1)           │
│           [v1-a][v1-b][v1-c]             [v2-a]  ← creating │
│           4 total pods (3+1=maxSurge)                        │
│                                                              │
│  Time 2:  RS-old(2)                      RS-new(1)           │
│           [v1-b][v1-c]                   [v2-a✓] ← ready    │
│           v1-a terminated                                    │
│                                                              │
│  Time 3:  RS-old(2)                      RS-new(2)           │
│           [v1-b][v1-c]                   [v2-a][v2-b]        │
│                                                              │
│  Time 4:  RS-old(1)                      RS-new(2)           │
│           [v1-c]                         [v2-a][v2-b✓]       │
│                                                              │
│  Time 5:  RS-old(1)                      RS-new(3)           │
│           [v1-c]                         [v2-a][v2-b][v2-c]  │
│                                                              │
│  Time 6:  RS-old(0)                      RS-new(3)           │
│           []                             [v2-a][v2-b][v2-c✓] │
│                                                              │
│  Done!    Old RS kept (scaled to 0) for rollback             │
└──────────────────────────────────────────────────────────────┘
```

---

## StatefulSets

**StatefulSets** are used for stateful applications that require:
- **Stable network identity** (persistent hostname)
- **Stable persistent storage** (each pod gets its own PVC)
- **Ordered deployment and scaling** (pods created/deleted in order)

```
┌────────────────────────────────────────────────────────────────┐
│                      StatefulSet                               │
│                   Name: postgres                               │
│                                                                │
│   ┌───────────────┐  ┌───────────────┐  ┌───────────────┐     │
│   │  postgres-0   │  │  postgres-1   │  │  postgres-2   │     │
│   │  (ordinal 0)  │  │  (ordinal 1)  │  │  (ordinal 2)  │     │
│   │               │  │               │  │               │     │
│   │  Hostname:    │  │  Hostname:    │  │  Hostname:    │     │
│   │  postgres-0   │  │  postgres-1   │  │  postgres-2   │     │
│   │               │  │               │  │               │     │
│   │  DNS:         │  │  DNS:         │  │  DNS:         │     │
│   │  postgres-0.  │  │  postgres-1.  │  │  postgres-2.  │     │
│   │  postgres-svc │  │  postgres-svc │  │  postgres-svc │     │
│   │               │  │               │  │               │     │
│   │  ┌──────────┐ │  │  ┌──────────┐ │  │  ┌──────────┐ │     │
│   │  │ PVC-0    │ │  │  │ PVC-1    │ │  │  │ PVC-2    │ │     │
│   │  │ (10 Gi)  │ │  │  │ (10 Gi)  │ │  │  │ (10 Gi)  │ │     │
│   │  └──────────┘ │  │  └──────────┘ │  │  └──────────┘ │     │
│   └───────────────┘  └───────────────┘  └───────────────┘     │
│                                                                │
│   Scaling UP:   0 → 1 → 2  (ordered)                          │
│   Scaling DOWN: 2 → 1 → 0  (reverse order)                    │
│   Pod deleted:  PVC is NOT deleted (data persists)             │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| **Pod names** | Random (web-app-6d4f5-x7k2j) | Ordered (postgres-0, postgres-1) |
| **Network identity** | Interchangeable | Stable, predictable hostname |
| **Storage** | Shared (or no PVC) | Each pod gets its own PVC |
| **Scaling order** | Any order | Sequential (0,1,2... or 2,1,0) |
| **Update order** | Any order | Reverse ordinal (2→1→0) by default |
| **PVC cleanup** | N/A | PVCs persist after pod/sts deletion |
| **Use case** | Stateless apps (web, API) | Databases, message queues, etc. |

### StatefulSet YAML

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-svc     # Required: headless service name
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:         # Each pod gets its own PVC
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: standard
      resources:
        requests:
          storage: 10Gi
---
# Headless Service (required for StatefulSet)
apiVersion: v1
kind: Service
metadata:
  name: postgres-svc
spec:
  clusterIP: None               # Headless!
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### Headless Service DNS Records

```
Headless Service creates DNS A records for each pod:

postgres-0.postgres-svc.default.svc.cluster.local → 10.244.1.5
postgres-1.postgres-svc.default.svc.cluster.local → 10.244.2.8
postgres-2.postgres-svc.default.svc.cluster.local → 10.244.3.3

This allows clients to connect to SPECIFIC pods, which is
essential for database replication (primary vs replicas).
```

---

## DaemonSets

A **DaemonSet** ensures a copy of a pod runs on **every** node (or a subset of nodes) in the cluster.

```
┌────────────────────────────────────────────────────────────────┐
│                        DaemonSet                               │
│                    (runs on every node)                        │
│                                                                │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  Node 1    │  │  Node 2    │  │  Node 3    │               │
│  │            │  │            │  │            │               │
│  │  ┌──────┐  │  │  ┌──────┐  │  │  ┌──────┐  │               │
│  │  │log-  │  │  │  │log-  │  │  │  │log-  │  │               │
│  │  │agent │  │  │  │agent │  │  │  │agent │  │               │
│  │  └──────┘  │  │  └──────┘  │  │  └──────┘  │               │
│  │            │  │            │  │            │               │
│  │  [Pod A]   │  │  [Pod B]   │  │  [Pod C]   │               │
│  │  [Pod D]   │  │  [Pod E]   │  │  [Pod F]   │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                │
│  New Node 4 added → DaemonSet auto-creates pod on it          │
│  Node 3 removed   → DaemonSet pod is garbage collected        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Common DaemonSet Use Cases

| Use Case | Example |
|----------|---------|
| **Log collection** | Fluentd, Fluent Bit, Filebeat |
| **Node monitoring** | Prometheus Node Exporter, Datadog Agent |
| **Network plugin** | Calico, Cilium, kube-proxy |
| **Storage daemon** | Ceph, GlusterFS agents |
| **Security agent** | Falco, Sysdig |

### DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      tolerations:                    # Run on ALL nodes including master
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - name: fluentd
        image: fluentd:v1.16
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            memory: 500Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: containers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: containers
        hostPath:
          path: /var/lib/docker/containers
```

```bash
# DaemonSet commands
kubectl get daemonsets -A            # All namespaces
kubectl describe ds log-agent -n kube-system
kubectl rollout status ds/log-agent -n kube-system
```

---

## Jobs and CronJobs

### Jobs

A **Job** creates one or more Pods and ensures a specified number of them successfully terminate.

```
┌────────────────────────────────────────────────────────────────┐
│                          JOB                                   │
│                    Name: data-migration                       │
│                                                                │
│  Sequential (completions: 3, parallelism: 1):                 │
│  ┌───────┐              ┌───────┐              ┌───────┐      │
│  │ Pod 1 │──Completed──▶│ Pod 2 │──Completed──▶│ Pod 3 │      │
│  └───────┘              └───────┘              └───────┘      │
│                                                                │
│  Parallel (completions: 6, parallelism: 3):                   │
│  ┌───────┐ ┌───────┐ ┌───────┐                                │
│  │ Pod 1 │ │ Pod 2 │ │ Pod 3 │  ← Batch 1 (3 parallel)      │
│  └───────┘ └───────┘ └───────┘                                │
│  ┌───────┐ ┌───────┐ ┌───────┐                                │
│  │ Pod 4 │ │ Pod 5 │ │ Pod 6 │  ← Batch 2 (3 parallel)      │
│  └───────┘ └───────┘ └───────┘                                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Job YAML

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-migration
spec:
  completions: 3          # Total successful completions needed
  parallelism: 2          # Max pods running at same time
  backoffLimit: 4         # Max retries before marking failed
  activeDeadlineSeconds: 600  # Timeout for entire job
  ttlSecondsAfterFinished: 100  # Auto-cleanup after completion
  template:
    spec:
      restartPolicy: Never   # Jobs use Never or OnFailure
      containers:
      - name: migrate
        image: my-migration:1.0
        command: ["python", "migrate.py"]
        env:
        - name: BATCH_ID
          value: "production"
```

```bash
# Job commands
kubectl get jobs
kubectl describe job data-migration
kubectl logs job/data-migration           # Logs from job pods
kubectl delete job data-migration         # Also deletes pods
```

### CronJobs

A **CronJob** creates Jobs on a repeating schedule (cron syntax).

```
┌────────────────────────────────────────────────────────────────┐
│                        CronJob                                 │
│                  Schedule: "0 2 * * *"                         │
│                  (Daily at 2:00 AM)                            │
│                                                                │
│  Time ──────────────────────────────────────────────▶          │
│                                                                │
│  Day 1 (2:00 AM)    Day 2 (2:00 AM)    Day 3 (2:00 AM)       │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐          │
│  │  Job 1   │       │  Job 2   │       │  Job 3   │          │
│  │ ┌──────┐ │       │ ┌──────┐ │       │ ┌──────┐ │          │
│  │ │ Pod  │ │       │ │ Pod  │ │       │ │ Pod  │ │          │
│  │ │backup│ │       │ │backup│ │       │ │backup│ │          │
│  │ └──────┘ │       │ └──────┘ │       │ └──────┘ │          │
│  └──────────┘       └──────────┘       └──────────┘          │
│                                                                │
│  Cron Syntax:  ┌───── minute (0-59)                           │
│                │ ┌─── hour (0-23)                             │
│                │ │ ┌─ day of month (1-31)                     │
│                │ │ │ ┌─ month (1-12)                          │
│                │ │ │ │ ┌─ day of week (0-6, Sun=0)            │
│                * * * * *                                      │
│                                                                │
│  Examples:                                                     │
│  "*/5 * * * *"      Every 5 minutes                           │
│  "0 */2 * * *"      Every 2 hours                             │
│  "0 2 * * *"        Daily at 2 AM                             │
│  "0 2 * * 1"        Every Monday at 2 AM                      │
│  "0 0 1 * *"        First day of every month at midnight      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### CronJob YAML

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"          # Daily at 2 AM
  concurrencyPolicy: Forbid      # Don't overlap with previous job
  successfulJobsHistoryLimit: 3   # Keep last 3 successful jobs
  failedJobsHistoryLimit: 1       # Keep last 1 failed job
  startingDeadlineSeconds: 200    # Max delay before skipping
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: postgres:16
            command:
            - /bin/sh
            - -c
            - |
              pg_dump -h postgres-svc -U admin mydb > /backup/dump-$(date +%Y%m%d).sql
              echo "Backup completed successfully"
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
```

**CronJob Concurrency Policies:**

| Policy | Behavior |
|--------|----------|
| `Allow` (default) | Multiple jobs can run concurrently |
| `Forbid` | Skip new job if previous is still running |
| `Replace` | Cancel running job and start new one |

```bash
# CronJob commands
kubectl get cronjobs
kubectl describe cronjob db-backup
kubectl create job --from=cronjob/db-backup manual-backup  # Trigger manually
```

---

## Choosing the Right Workload Type

```
┌──────────────────────────────────────────────────────────────────┐
│              WORKLOAD TYPE DECISION TREE                         │
│                                                                  │
│  Is it a one-time task?                                         │
│  ├── YES → Is it scheduled/recurring?                           │
│  │         ├── YES → CronJob                                    │
│  │         └── NO  → Job                                        │
│  │                                                              │
│  └── NO (long-running) →                                        │
│       │                                                          │
│       ├── Must run on EVERY node?                               │
│       │   └── YES → DaemonSet                                   │
│       │                                                          │
│       ├── Needs stable identity/storage?                        │
│       │   └── YES → StatefulSet                                 │
│       │                                                          │
│       └── Stateless replicas?                                   │
│           └── YES → Deployment  (most common)                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Feature | Deployment | StatefulSet | DaemonSet | Job | CronJob |
|---------|-----------|-------------|-----------|-----|---------|
| **Purpose** | Stateless apps | Stateful apps | Per-node agents | One-time tasks | Scheduled tasks |
| **Replicas** | Configurable | Configurable | 1 per node | completions | Per schedule |
| **Pod Names** | Random | Ordered (0,1,2) | Per node | Random | Random |
| **Scaling** | Any order | Sequential | Auto (nodes) | parallelism | N/A |
| **Updates** | Rolling/Recreate | Rolling (ordered) | Rolling | N/A | N/A |
| **Persistent Storage** | Optional (shared) | Per-pod PVC | Node storage | Optional | Optional |
| **Network Identity** | None | Stable hostname | N/A | N/A | N/A |
| **Examples** | Web servers, APIs | DBs, Kafka, Redis | Log agents, monitors | Migrations, reports | Backups, cleanups |

---

## Real-World Scenario: Complete Application Stack

```yaml
# A typical microservices app uses multiple workload types:

# 1. Deployment — Stateless API service
#    web-api (3 replicas, rolling updates)

# 2. StatefulSet — Database
#    postgres (3 replicas, primary + 2 replicas, each with own PVC)

# 3. DaemonSet — Log collection
#    fluentd (one per node, ships logs to Elasticsearch)

# 4. CronJob — Database backup
#    db-backup (runs daily at 2 AM)

# 5. Job — Database migration
#    db-migrate (runs once during deployment)
```

```
┌──────────────────────────────────────────────────────────────┐
│                APPLICATION ON KUBERNETES                      │
│                                                              │
│  ┌──────────────── Deployment ────────────────┐              │
│  │  web-api (3 replicas)                      │              │
│  │  [Pod] [Pod] [Pod]  ← rolling updates      │              │
│  └──────────────────────────────────────────────┘             │
│                      │                                       │
│                      ▼                                       │
│  ┌──────────────── StatefulSet ────────────────┐             │
│  │  postgres (3 replicas)                      │             │
│  │  [postgres-0] [postgres-1] [postgres-2]     │             │
│  │  [PVC-0]      [PVC-1]      [PVC-2]          │             │
│  └──────────────────────────────────────────────┘             │
│                                                              │
│  ┌──── DaemonSet ─────┐  ┌──── CronJob ──────┐              │
│  │  fluentd            │  │  db-backup         │              │
│  │  [Node1][Node2]     │  │  "0 2 * * *"       │              │
│  │  [Node3]            │  │  → Job → Pod       │              │
│  └─────────────────────┘  └────────────────────┘              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Workload | Solution |
|-------|----------|---------|
| Deployment stuck during rollout | Deployment | `kubectl rollout status`, check pod events, readiness probes |
| Pods not scaling | ReplicaSet/Deployment | Check resource quotas, node capacity |
| StatefulSet pod pending | StatefulSet | Check PVC binding, storage class, headless service |
| DaemonSet missing on some nodes | DaemonSet | Check taints/tolerations, nodeSelector |
| Job keeps retrying | Job | Check backoffLimit, fix container exit code |
| CronJob not triggering | CronJob | Check schedule syntax, startingDeadlineSeconds, suspended field |
| Rollback not working | Deployment | Check revisionHistoryLimit, `kubectl rollout history` |

```bash
# Useful debugging commands for workloads
kubectl get all                              # See all resources
kubectl rollout history deployment/web-app   # Revision history
kubectl get events --sort-by='.lastTimestamp' # Recent events
kubectl get pods -l app=web --watch          # Watch pod changes
```

---

## Summary Table

| Resource | Key Point |
|----------|-----------|
| **ReplicaSet** | Ensures N pod replicas; rarely created directly |
| **Deployment** | Most common; manages ReplicaSets, rolling updates, rollbacks |
| **StatefulSet** | Stable identity, ordered scaling, per-pod PVCs; for databases |
| **DaemonSet** | One pod per node; for agents (logging, monitoring, networking) |
| **Job** | Run-to-completion tasks; configurable parallelism and retries |
| **CronJob** | Scheduled jobs; creates Job objects on a cron schedule |
| **Rolling Update** | Gradually replace old pods; zero-downtime |
| **Recreate** | Kill all old then create new; brief downtime |
| **Headless Service** | Required by StatefulSet; provides per-pod DNS records |

---

## Quick Revision Questions

1. **What is the relationship between a Deployment, a ReplicaSet, and a Pod?**
   <details><summary>Answer</summary>A Deployment manages ReplicaSets. A ReplicaSet manages Pods. When you update a Deployment, it creates a new ReplicaSet (with the new spec) and scales it up while scaling down the old ReplicaSet. The old ReplicaSet is kept (scaled to 0) for potential rollback.</details>

2. **When would you use `Recreate` strategy instead of `RollingUpdate`?**
   <details><summary>Answer</summary>Use Recreate when your application cannot handle running two versions simultaneously — for example, when performing incompatible database schema migrations, or when the old and new versions would conflict on shared resources (like a specific port on a hostPort).</details>

3. **Why does a StatefulSet need a Headless Service?**
   <details><summary>Answer</summary>A Headless Service (clusterIP: None) creates individual DNS A records for each pod (e.g., postgres-0.postgres-svc). This provides stable network identities, which is essential for stateful applications like databases where clients need to connect to specific instances (e.g., primary vs replica).</details>

4. **Name three workloads that are good candidates for a DaemonSet.**
   <details><summary>Answer</summary>1) Log collection agents (Fluentd, Fluent Bit), 2) Node monitoring exporters (Prometheus Node Exporter), 3) Network plugins (Calico, Cilium, kube-proxy). Others: storage daemons, security agents (Falco).</details>

5. **What is the difference between `completions` and `parallelism` in a Job?**
   <details><summary>Answer</summary>`completions` is the total number of pods that must successfully complete for the Job to be considered done. `parallelism` is the maximum number of pods that can run at the same time. For example, completions=6, parallelism=3 means 6 pods total, at most 3 running in parallel.</details>

6. **How do you manually trigger a CronJob?**
   <details><summary>Answer</summary>`kubectl create job --from=cronjob/db-backup manual-backup-run`. This creates a one-time Job from the CronJob's template, named "manual-backup-run".</details>

---

[← Previous: Pods](../03-Pods/01-pods.md) | [Back to README](../README.md) | [Next: Services and Networking →](../05-Services-And-Networking/01-services-and-networking.md)
