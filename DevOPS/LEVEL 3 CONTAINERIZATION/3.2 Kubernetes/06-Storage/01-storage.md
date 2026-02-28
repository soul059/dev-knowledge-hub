# Unit 6: Storage

[← Previous: Services and Networking](../05-Services-And-Networking/01-services-and-networking.md) | [Back to README](../README.md) | [Next: Configuration →](../07-Configuration/01-configuration.md)

---

## Chapter Overview

Containers are **ephemeral** — when a pod restarts, its filesystem is lost. Kubernetes storage abstractions let you persist data beyond the lifetime of a pod. This unit covers Volumes, Persistent Volumes (PV), Persistent Volume Claims (PVC), Storage Classes, dynamic provisioning, and CSI drivers.

---

## 6.1 Volumes

A **Volume** is a directory accessible to containers in a pod. It survives container restarts but is tied to the **pod's lifecycle**.

```
┌─────────────────────────────────────────────────────────────┐
│                    VOLUME TYPES OVERVIEW                     │
│                                                             │
│  ┌──────────────────────────────────────────────────┐       │
│  │  EPHEMERAL (die with pod)                        │       │
│  │  ┌──────────┐  ┌────────────┐                    │       │
│  │  │ emptyDir │  │ configMap  │                    │       │
│  │  │          │  │ secret     │                    │       │
│  │  │ Temp     │  │ downwardAPI│                    │       │
│  │  │ storage  │  │            │                    │       │
│  │  └──────────┘  └────────────┘                    │       │
│  └──────────────────────────────────────────────────┘       │
│                                                             │
│  ┌──────────────────────────────────────────────────┐       │
│  │  PERSISTENT (survive pod deletion)               │       │
│  │  ┌──────────────┐  ┌────────────────┐            │       │
│  │  │ hostPath     │  │ PersistentVol  │            │       │
│  │  │ (node disk)  │  │ Claim (PVC)    │            │       │
│  │  │              │  │                │            │       │
│  │  │ ⚠ Testing    │  │ ✅ Production   │            │       │
│  │  │   only       │  │                │            │       │
│  │  └──────────────┘  └────────────────┘            │       │
│  └──────────────────────────────────────────────────┘       │
│                                                             │
│  ┌──────────────────────────────────────────────────┐       │
│  │  NETWORK STORAGE                                 │       │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌───────┐           │       │
│  │  │ NFS  │ │iSCSI │ │ Ceph │ │AWS EBS│           │       │
│  │  │      │ │      │ │ RBD  │ │GCE PD │           │       │
│  │  │      │ │      │ │      │ │AZ Disk│           │       │
│  │  └──────┘ └──────┘ └──────┘ └───────┘           │       │
│  └──────────────────────────────────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### emptyDir

Created when a pod is assigned to a node, deleted when the pod is removed. Used for temporary storage and sharing data between containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shared-data
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "Hello" > /data/message; sleep 3600']
    volumeMounts:
    - name: shared
      mountPath: /data
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /data/message; sleep 3600']
    volumeMounts:
    - name: shared
      mountPath: /data
  volumes:
  - name: shared
    emptyDir: {}            # Can also use: emptyDir: { medium: Memory }
```

### hostPath

Mounts a file or directory from the **host node's filesystem** into the pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-path-pod
spec:
  containers:
  - name: logger
    image: busybox
    volumeMounts:
    - name: host-logs
      mountPath: /var/log/host
      readOnly: true
  volumes:
  - name: host-logs
    hostPath:
      path: /var/log            # Path on the host node
      type: Directory           # Directory | File | DirectoryOrCreate
```

> **Warning:** `hostPath` is a security risk and should be avoided in production. It ties the pod to a specific node and can access sensitive host files.

---

## Persistent Volumes (PV)

A **PersistentVolume (PV)** is a piece of storage provisioned by an admin (or dynamically). It exists independently of any pod.

```
┌────────────────────────────────────────────────────────────────┐
│                  PV / PVC / POD RELATIONSHIP                    │
│                                                                │
│                                                                │
│  Admin / StorageClass                                          │
│  provisions PV            User creates PVC      Pod uses PVC   │
│                                                                │
│  ┌──────────────┐        ┌──────────────┐     ┌────────────┐  │
│  │  Persistent  │◄──────▶│  Persistent  │◄───▶│    Pod     │  │
│  │  Volume      │ BIND   │  Volume      │ USE │            │  │
│  │  (PV)        │        │  Claim (PVC) │     │ volumeMount│  │
│  │              │        │              │     │ /data      │  │
│  │  100Gi SSD   │        │  Request:    │     │            │  │
│  │  ReadWrite   │        │  50Gi RWO    │     │            │  │
│  │  Once        │        │              │     │            │  │
│  └──────────────┘        └──────────────┘     └────────────┘  │
│                                                                │
│  Think of it as:                                               │
│  PV  = Physical hard drive (provisioned by admin)              │
│  PVC = "I need a drive this big" (requested by developer)      │
│  Pod = "Mount that drive here" (uses the claim)                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### PV YAML

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ssd-100gi
  labels:
    type: ssd
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce             # RWO - one node read/write
  persistentVolumeReclaimPolicy: Retain   # What happens when PVC is deleted
  storageClassName: manual
  hostPath:                   # For testing only
    path: /mnt/data
  # For production, use:
  # csi:
  #   driver: ebs.csi.aws.com
  #   volumeHandle: vol-0a1b2c3d4e5f
```

### Access Modes

| Mode | Abbreviation | Description |
|------|-------------|-------------|
| `ReadWriteOnce` | RWO | Read/write by a **single node** |
| `ReadOnlyMany` | ROX | Read-only by **many nodes** |
| `ReadWriteMany` | RWX | Read/write by **many nodes** |
| `ReadWriteOncePod` | RWOP | Read/write by a **single pod** (K8s 1.22+) |

### Reclaim Policies

```
┌──────────────────────────────────────────────────────────┐
│              RECLAIM POLICIES                             │
│                                                          │
│  When PVC is deleted, what happens to the PV?            │
│                                                          │
│  ┌─────────────────────────────────────┐                 │
│  │  Retain (default for static PVs)    │                 │
│  │  PV remains, data preserved         │                 │
│  │  Admin must manually reclaim        │                 │
│  │  Status: Released                   │                 │
│  └─────────────────────────────────────┘                 │
│                                                          │
│  ┌─────────────────────────────────────┐                 │
│  │  Delete (default for dynamic PVs)   │                 │
│  │  PV and underlying storage deleted  │                 │
│  │  ⚠ Data is lost!                    │                 │
│  └─────────────────────────────────────┘                 │
│                                                          │
│  ┌─────────────────────────────────────┐                 │
│  │  Recycle (deprecated)               │                 │
│  │  Basic scrub (rm -rf /volume/*)     │                 │
│  │  Use dynamic provisioning instead   │                 │
│  └─────────────────────────────────────┘                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### PV Lifecycle

```
  ┌───────────┐     ┌───────────┐     ┌───────────┐     ┌───────────┐
  │ Available │────▶│   Bound   │────▶│ Released  │────▶│  Failed   │
  │           │     │           │     │           │     │           │
  │ PV exists │     │ PVC bound │     │ PVC       │     │ Reclaim   │
  │ no PVC    │     │ to PV     │     │ deleted   │     │ failed    │
  └───────────┘     └───────────┘     └───────────┘     └───────────┘
```

---

## Persistent Volume Claims (PVC)

A **PVC** is a request for storage by a user. Kubernetes matches PVCs to available PVs.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: manual       # Must match PV's storageClassName
  # selector:                    # Optional: select specific PVs
  #   matchLabels:
  #     type: ssd
```

### Using PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - name: data
      mountPath: /app/data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: app-data-pvc    # Reference the PVC
```

### PV-PVC Binding Rules

```
┌─────────────────────────────────────────────────────────────┐
│              PV ↔ PVC MATCHING CRITERIA                       │
│                                                             │
│  PVC requests:                PV must have:                 │
│  ┌──────────────────┐        ┌──────────────────┐           │
│  │ storage: 50Gi    │───────▶│ capacity ≥ 50Gi  │           │
│  │ accessMode: RWO  │───────▶│ accessMode: RWO  │           │
│  │ storageClass: ssd│───────▶│ storageClass: ssd│           │
│  │ selector: type=x │───────▶│ label: type=x    │           │
│  └──────────────────┘        └──────────────────┘           │
│                                                             │
│  Binding is 1:1 — one PVC binds to exactly one PV          │
│  PV capacity may be larger than PVC request                 │
│  If no matching PV exists, PVC stays in Pending state       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Storage Classes

A **StorageClass** defines different classes/tiers of storage (e.g., SSD vs HDD, fast vs slow) and enables **dynamic provisioning**.

```
┌────────────────────────────────────────────────────────────────┐
│                    STORAGE CLASSES                              │
│                                                                │
│  Static Provisioning:           Dynamic Provisioning:          │
│  ┌───────────────────┐          ┌───────────────────┐          │
│  │ 1. Admin creates  │          │ 1. Admin creates  │          │
│  │    PVs manually   │          │    StorageClass    │          │
│  │                   │          │                   │          │
│  │ 2. User creates   │          │ 2. User creates   │          │
│  │    PVC             │          │    PVC with SC    │          │
│  │                   │          │                   │          │
│  │ 3. K8s binds      │          │ 3. SC auto-creates│          │
│  │    PVC → PV       │          │    PV and binds   │          │
│  └───────────────────┘          └───────────────────┘          │
│                                                                │
│  ❌ Manual, tedious             ✅ Automatic, scalable          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### StorageClass Examples

```yaml
# Fast SSD storage class (AWS)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com      # CSI driver
parameters:
  type: gp3
  iops: "5000"
  throughput: "250"
  encrypted: "true"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer  # Wait until pod is scheduled
allowVolumeExpansion: true               # Allow PVC resize
---
# Standard HDD storage class (AWS)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-hdd
provisioner: ebs.csi.aws.com
parameters:
  type: st1
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
---
# NFS storage class (on-prem)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: nfs.csi.k8s.io
parameters:
  server: nfs-server.example.com
  share: /exports/kubernetes
reclaimPolicy: Retain
```

### Volume Binding Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| `Immediate` | PV provisioned immediately when PVC created | All pods in one zone |
| `WaitForFirstConsumer` | PV provisioned when pod using PVC is scheduled | Multi-zone clusters |

---

## Dynamic Provisioning

With dynamic provisioning, you only need a **StorageClass** and **PVC** — the PV is created automatically.

```
┌────────────────────────────────────────────────────────────────┐
│               DYNAMIC PROVISIONING FLOW                        │
│                                                                │
│  1. StorageClass exists                                        │
│     ┌──────────────────┐                                       │
│     │ SC: fast-ssd     │                                       │
│     │ provisioner: ebs │                                       │
│     └──────────────────┘                                       │
│                                                                │
│  2. User creates PVC                                           │
│     ┌──────────────────────┐                                   │
│     │ PVC: my-data         │                                   │
│     │ storageClassName:    │                                   │
│     │   fast-ssd           │                                   │
│     │ storage: 50Gi        │                                   │
│     └──────────┬───────────┘                                   │
│                │                                               │
│  3. SC auto-provisions PV                                      │
│                ▼                                               │
│     ┌──────────────────────┐                                   │
│     │ PV: pvc-abc123       │  ← Auto-created!                 │
│     │ 50Gi, gp3 SSD        │                                   │
│     │ Bound to: my-data    │                                   │
│     └──────────────────────┘                                   │
│                                                                │
│  4. Pod uses PVC                                               │
│     ┌──────────────────────┐                                   │
│     │ Pod mounts my-data   │                                   │
│     │ at /app/data         │                                   │
│     └──────────────────────┘                                   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Complete Dynamic Provisioning Example

```yaml
# StorageClass (created by admin once)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
# PVC (created by developer)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd      # Reference StorageClass
  resources:
    requests:
      storage: 100Gi
---
# Pod using the PVC
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
  - name: postgres
    image: postgres:16
    volumeMounts:
    - name: data
      mountPath: /var/lib/postgresql/data
    env:
    - name: POSTGRES_PASSWORD
      value: mysecretpassword
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: postgres-data
```

### Expanding PVCs

```bash
# Only works if StorageClass has allowVolumeExpansion: true
kubectl patch pvc postgres-data -p '{"spec":{"resources":{"requests":{"storage":"200Gi"}}}}'

# Or edit:
kubectl edit pvc postgres-data
# Change storage: 100Gi → storage: 200Gi

# Note: Shrinking PVCs is NOT supported
```

---

## CSI Drivers

**Container Storage Interface (CSI)** is the standard for exposing storage systems to Kubernetes. CSI drivers replace the old in-tree volume plugins.

```
┌──────────────────────────────────────────────────────────────┐
│                     CSI ARCHITECTURE                          │
│                                                              │
│  Kubernetes                          Storage Provider        │
│  ┌──────────────┐                   ┌──────────────────┐     │
│  │   kubelet    │                   │  Storage Backend  │     │
│  │              │                   │  (AWS EBS, Ceph,  │     │
│  │  ┌────────┐  │     CSI Spec      │   NFS, etc.)      │     │
│  │  │ CSI    │◄─┼──────────────────▶│                  │     │
│  │  │ Plugin │  │     gRPC          │                  │     │
│  │  └────────┘  │                   └──────────────────┘     │
│  └──────────────┘                                            │
│                                                              │
│  CSI Components:                                             │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Controller Plugin (runs as Deployment)          │        │
│  │  • CreateVolume / DeleteVolume                   │        │
│  │  • Attach / Detach                               │        │
│  │  • Snapshot                                      │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Node Plugin (runs as DaemonSet)                 │        │
│  │  • Mount / Unmount on node                       │        │
│  │  • Format volume                                 │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Common CSI Drivers

| CSI Driver | Storage Backend | Features |
|-----------|-----------------|----------|
| **aws-ebs-csi-driver** | AWS EBS | Block storage, snapshots, encryption |
| **aws-efs-csi-driver** | AWS EFS | Shared NFS (ReadWriteMany) |
| **azuredisk-csi-driver** | Azure Disk | Block storage for AKS |
| **gcp-pd-csi-driver** | GCE PD | Block storage for GKE |
| **csi-driver-nfs** | NFS servers | Shared NFS storage |
| **rook-ceph** | Ceph | Block, file, object storage |
| **longhorn** | Longhorn | Distributed block storage (Rancher) |
| **local-path-provisioner** | Local disk | For development/testing |

```bash
# Install AWS EBS CSI driver (EKS)
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable"

# Check CSI driver pods
kubectl get pods -n kube-system -l app=ebs-csi-controller

# List storage classes
kubectl get storageclass

# List PVs and PVCs
kubectl get pv
kubectl get pvc
kubectl get pvc -A
```

---

## Real-World Scenario: Database with Persistent Storage

```yaml
# Complete production database setup with storage

# 1. StorageClass for database storage
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: db-storage
provisioner: ebs.csi.aws.com
parameters:
  type: io2
  iops: "10000"
  encrypted: "true"
reclaimPolicy: Retain          # Don't delete DB data!
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
# 2. StatefulSet with volumeClaimTemplates
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
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
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            cpu: "500m"
            memory: 1Gi
          limits:
            cpu: "2"
            memory: 4Gi
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: db-storage
      resources:
        requests:
          storage: 100Gi
```

```
┌──────────────────────────────────────────────────────────────┐
│          STATEFULSET STORAGE (auto-created PVCs)              │
│                                                              │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  │
│  │  postgres-0    │  │  postgres-1    │  │  postgres-2    │  │
│  │  ┌──────────┐  │  │  ┌──────────┐  │  │  ┌──────────┐  │  │
│  │  │Container │  │  │  │Container │  │  │  │Container │  │  │
│  │  │/data     │  │  │  │/data     │  │  │  │/data     │  │  │
│  │  └────┬─────┘  │  │  └────┬─────┘  │  │  └────┬─────┘  │  │
│  │       │        │  │       │        │  │       │        │  │
│  │  ┌────▼─────┐  │  │  ┌────▼─────┐  │  │  ┌────▼─────┐  │  │
│  │  │PVC:      │  │  │  │PVC:      │  │  │  │PVC:      │  │  │
│  │  │postgres- │  │  │  │postgres- │  │  │  │postgres- │  │  │
│  │  │data-     │  │  │  │data-     │  │  │  │data-     │  │  │
│  │  │postgres-0│  │  │  │postgres-1│  │  │  │postgres-2│  │  │
│  │  └────┬─────┘  │  │  └────┬─────┘  │  │  └────┬─────┘  │  │
│  │       │        │  │       │        │  │       │        │  │
│  │  ┌────▼─────┐  │  │  ┌────▼─────┐  │  │  ┌────▼─────┐  │  │
│  │  │PV: 100Gi │  │  │  │PV: 100Gi │  │  │  │PV: 100Gi │  │  │
│  │  │EBS io2   │  │  │  │EBS io2   │  │  │  │EBS io2   │  │  │
│  │  └──────────┘  │  │  └──────────┘  │  │  └──────────┘  │  │
│  └────────────────┘  └────────────────┘  └────────────────┘  │
│                                                              │
│  If postgres-1 is deleted and recreated:                     │
│  → New pod reattaches to the SAME PVC/PV                     │
│  → Data is preserved!                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| PVC stuck in `Pending` | `kubectl describe pvc` | Check StorageClass exists, capacity available |
| PV stuck in `Released` | PVC deleted, PV has Retain policy | Manually delete PV or remove `claimRef` |
| Pod can't mount volume | Check events: `kubectl describe pod` | Verify PVC is bound, node access |
| Multi-attach error | Volume used by pod on another node | Use `ReadWriteMany` or terminate old pod |
| Volume expansion stuck | Controller not supporting expansion | Check CSI driver, `allowVolumeExpansion` |
| Data loss after pod delete | Using `emptyDir` instead of PVC | Use PersistentVolumeClaim |

```bash
# Storage debugging commands
kubectl get pv                                # List PersistentVolumes
kubectl get pvc                               # List PersistentVolumeClaims
kubectl get storageclass                      # List StorageClasses
kubectl describe pvc <name>                   # PVC details + events
kubectl describe pv <name>                    # PV details
kubectl get events --field-selector reason=FailedMount  # Mount failures
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **emptyDir** | Temporary storage, dies with pod; for scratch space, shared between containers |
| **hostPath** | Mounts node filesystem; testing only; security risk |
| **PersistentVolume (PV)** | Admin-provisioned storage; lifecycle independent of pods |
| **PersistentVolumeClaim (PVC)** | User's request for storage; binds to a matching PV |
| **StorageClass** | Defines storage tiers; enables dynamic provisioning |
| **Dynamic Provisioning** | Auto-creates PV when PVC is created (via StorageClass) |
| **Access Modes** | RWO (one node), ROX (many read), RWX (many write), RWOP (one pod) |
| **Reclaim Policy** | Retain (keep data), Delete (remove all), Recycle (deprecated) |
| **CSI** | Standard interface for storage plugins; replaces in-tree drivers |
| **VolumeClaimTemplates** | StatefulSet feature; creates unique PVC per pod replica |

---

## Quick Revision Questions

1. **What is the difference between `emptyDir` and a PersistentVolumeClaim?**
   <details><summary>Answer</summary>emptyDir is ephemeral — it's created when a pod is assigned to a node and deleted when the pod is removed. Data is lost on pod deletion. A PVC is bound to a PV that persists beyond the pod's lifecycle — data survives pod restarts and rescheduling.</details>

2. **Explain the PV → PVC → Pod binding flow.**
   <details><summary>Answer</summary>1) Admin creates PV (or StorageClass for dynamic provisioning). 2) Developer creates PVC requesting specific size, access mode, and storage class. 3) Kubernetes finds a matching PV and binds it to the PVC (1:1). 4) Pod references PVC in its volumes section and mounts it via volumeMounts. 5) Pod can now read/write to the mounted path.</details>

3. **What happens when a PVC is deleted and the PV has `reclaimPolicy: Retain`?**
   <details><summary>Answer</summary>The PV enters the "Released" state. The data is preserved, but the PV cannot be bound to a new PVC automatically. An admin must manually clean up: either delete the PV and recreate it, or remove the `claimRef` from the PV spec to make it available again.</details>

4. **Why should you use `WaitForFirstConsumer` binding mode in multi-zone clusters?**
   <details><summary>Answer</summary>With `Immediate` binding, the PV might be provisioned in Zone A, but the pod gets scheduled in Zone B — the pod can't access the volume. `WaitForFirstConsumer` delays PV creation until the pod is scheduled, ensuring the volume is created in the same zone as the pod.</details>

5. **How does a StatefulSet handle storage differently from a Deployment?**
   <details><summary>Answer</summary>A StatefulSet uses `volumeClaimTemplates` to create a unique PVC for each pod replica (e.g., data-postgres-0, data-postgres-1). Each pod always reattaches to its own PVC, even after rescheduling. A Deployment shares a single PVC across all replicas (or has no persistent storage). StatefulSet PVCs are NOT deleted when the StatefulSet is scaled down or deleted.</details>

6. **Can you shrink a PersistentVolumeClaim? Why or why not?**
   <details><summary>Answer</summary>No, PVC shrinking is not supported in Kubernetes. You can only expand a PVC (if the StorageClass has `allowVolumeExpansion: true` and the CSI driver supports it). This is because shrinking a volume could lead to data loss if the used space exceeds the new requested size.</details>

---

[← Previous: Services and Networking](../05-Services-And-Networking/01-services-and-networking.md) | [Back to README](../README.md) | [Next: Configuration →](../07-Configuration/01-configuration.md)
