# Unit 2: Kubernetes Architecture

[← Previous: Kubernetes Concepts](../01-Kubernetes-Concepts/01-what-is-kubernetes.md) | [Back to README](../README.md) | [Next: Pods →](../03-Pods/01-pods.md)

---

## Chapter Overview

This unit dives into the **internal architecture** of a Kubernetes cluster. You'll learn what runs on the control plane, what runs on worker nodes, how components communicate, and what add-ons extend the cluster. Understanding architecture is critical for troubleshooting, the CKA exam, and operating production clusters.

---

## 2.1 High-Level Architecture

A Kubernetes cluster consists of two main layers:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KUBERNETES CLUSTER                               │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     CONTROL PLANE (Master)                      │    │
│  │                                                                 │    │
│  │  ┌──────────┐  ┌──────┐  ┌───────────┐  ┌──────────────────┐  │    │
│  │  │   API    │  │ etcd │  │ Scheduler │  │   Controller     │  │    │
│  │  │  Server  │  │      │  │           │  │   Manager        │  │    │
│  │  │          │  │(k-v  │  │(where to  │  │                  │  │    │
│  │  │(gateway) │  │store)│  │place pods)│  │(reconcile loops) │  │    │
│  │  └────┬─────┘  └───┬──┘  └─────┬─────┘  └────────┬─────────┘  │    │
│  │       │            │           │                  │             │    │
│  │  ┌────▼────────────▼───────────▼──────────────────▼──────┐     │    │
│  │  │              Internal Communication Bus               │     │    │
│  │  └───────────────────────────┬────────────────────────────┘     │    │
│  └──────────────────────────────┼──────────────────────────────────┘    │
│                                 │                                       │
│                          ┌──────▼──────┐                                │
│                          │   Network   │                                │
│                          └──────┬──────┘                                │
│              ┌──────────────────┼──────────────────┐                    │
│              │                  │                  │                     │
│  ┌───────────▼──────┐ ┌────────▼────────┐ ┌───────▼──────────┐         │
│  │   WORKER NODE 1  │ │  WORKER NODE 2  │ │  WORKER NODE 3   │         │
│  │                  │ │                 │ │                   │         │
│  │ ┌──────────────┐ │ │ ┌─────────────┐ │ │ ┌──────────────┐ │         │
│  │ │   kubelet    │ │ │ │   kubelet   │ │ │ │   kubelet    │ │         │
│  │ ├──────────────┤ │ │ ├─────────────┤ │ │ ├──────────────┤ │         │
│  │ │  kube-proxy  │ │ │ │  kube-proxy │ │ │ │  kube-proxy  │ │         │
│  │ ├──────────────┤ │ │ ├─────────────┤ │ │ ├──────────────┤ │         │
│  │ │  Container   │ │ │ │  Container  │ │ │ │  Container   │ │         │
│  │ │  Runtime     │ │ │ │  Runtime    │ │ │ │  Runtime     │ │         │
│  │ ├──────────────┤ │ │ ├─────────────┤ │ │ ├──────────────┤ │         │
│  │ │ ┌──┐ ┌──┐   │ │ │ │ ┌──┐ ┌──┐  │ │ │ │ ┌──┐ ┌──┐   │ │         │
│  │ │ │P1│ │P2│   │ │ │ │ │P3│ │P4│  │ │ │ │ │P5│ │P6│   │ │         │
│  │ │ └──┘ └──┘   │ │ │ │ └──┘ └──┘  │ │ │ │ └──┘ └──┘   │ │         │
│  │ └──────────────┘ │ │ └─────────────┘ │ │ └──────────────┘ │         │
│  └──────────────────┘ └─────────────────┘ └──────────────────┘         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Control Plane Components

The control plane makes global decisions about the cluster (e.g., scheduling) and detects/responds to cluster events (e.g., starting a new pod when a deployment's replicas field is unsatisfied).

### kube-apiserver

The **API Server** is the front door to the Kubernetes cluster. Every interaction goes through it.

```
┌──────────────────────────────────────────────────────────┐
│                     kube-apiserver                        │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐             │
│  │  REST    │   │  AuthN   │   │  AuthZ   │             │
│  │  API     │──▶│(who are  │──▶│(what can │             │
│  │  Handler │   │  you?)   │   │ you do?) │             │
│  └──────────┘   └──────────┘   └────┬─────┘             │
│                                     │                    │
│                               ┌─────▼─────┐             │
│                               │ Admission │             │
│                               │ Control   │             │
│                               │(validate/ │             │
│                               │ mutate)   │             │
│                               └─────┬─────┘             │
│                                     │                    │
│                               ┌─────▼─────┐             │
│                               │   etcd    │             │
│                               │  (store)  │             │
│                               └───────────┘             │
└──────────────────────────────────────────────────────────┘
```

**Key Responsibilities:**
- Exposes the Kubernetes API (RESTful)
- Authenticates and authorizes all requests
- Validates and admits API objects
- Serves as the **only** component that talks to etcd
- Serves as the hub for inter-component communication

```bash
# API Server is what kubectl talks to
kubectl get pods
# Translates to: GET https://<api-server>:6443/api/v1/namespaces/default/pods

# Check API server health
kubectl get --raw /healthz

# Explore API resources
kubectl api-resources
kubectl api-versions
```

**Configuration flags (typical):**
```bash
kube-apiserver \
  --advertise-address=192.168.1.10 \
  --etcd-servers=https://127.0.0.1:2379 \
  --service-cluster-ip-range=10.96.0.0/12 \
  --authorization-mode=Node,RBAC \
  --enable-admission-plugins=NodeRestriction \
  --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
  --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```

---

### etcd

**etcd** is a distributed, reliable key-value store that serves as Kubernetes' **backing store for all cluster data**.

```
┌───────────────────────────────────────────────────────┐
│                       etcd Cluster                     │
│                                                       │
│   ┌────────┐      ┌────────┐      ┌────────┐         │
│   │ etcd-1 │◄────▶│ etcd-2 │◄────▶│ etcd-3 │         │
│   │(Leader)│      │(Follow)│      │(Follow)│         │
│   └───┬────┘      └────────┘      └────────┘         │
│       │                                               │
│   All writes go through the leader                    │
│   Uses RAFT consensus algorithm                       │
│                                                       │
│   Stores:                                             │
│   ├── /registry/pods/default/web-app-xyz              │
│   ├── /registry/deployments/default/web-app           │
│   ├── /registry/services/default/web-svc              │
│   ├── /registry/secrets/default/db-creds              │
│   ├── /registry/configmaps/default/app-config         │
│   └── /registry/nodes/worker-1                        │
│                                                       │
└───────────────────────────────────────────────────────┘
```

**Key Facts:**
- **Only the API server** communicates with etcd directly
- Uses the **Raft consensus algorithm** for leader election
- Requires an **odd number** of members (1, 3, 5) for quorum
- Stores all cluster state: pods, services, secrets, configs
- Critical to back up regularly

```bash
# Backup etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-table

# Check etcd cluster health
ETCDCTL_API=3 etcdctl endpoint health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

### kube-scheduler

The **Scheduler** watches for newly created Pods with no assigned node and selects a node for them to run on.

```
┌─────────────────────────────────────────────────────────┐
│                   kube-scheduler                         │
│                                                         │
│   New Pod Created (no node assigned)                    │
│           │                                             │
│           ▼                                             │
│   ┌───────────────┐                                     │
│   │  FILTERING    │  Remove nodes that can't run Pod    │
│   │  (Predicates) │                                     │
│   │               │  ❌ Not enough CPU?                  │
│   │               │  ❌ Not enough memory?               │
│   │               │  ❌ Taint not tolerated?             │
│   │               │  ❌ NodeSelector mismatch?           │
│   │               │  ❌ Port conflict?                   │
│   └───────┬───────┘                                     │
│           │  Feasible nodes                             │
│           ▼                                             │
│   ┌───────────────┐                                     │
│   │  SCORING      │  Rank remaining nodes               │
│   │  (Priorities) │                                     │
│   │               │  ⭐ Resource balance                 │
│   │               │  ⭐ Pod affinity/anti-affinity       │
│   │               │  ⭐ Data locality                    │
│   │               │  ⭐ Inter-pod spreading              │
│   └───────┬───────┘                                     │
│           │  Highest scored node                        │
│           ▼                                             │
│   ┌───────────────┐                                     │
│   │   BINDING     │  Assign Pod → Node                  │
│   │               │  (writes to API server)             │
│   └───────────────┘                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Scheduling Factors:**
| Factor | Description |
|--------|-------------|
| Resource requests | CPU and memory requested by the pod |
| Node selectors | Labels that must match node labels |
| Affinity rules | Prefer or require specific nodes/pods |
| Taints/Tolerations | Nodes repel pods unless tolerated |
| Pod topology | Spread pods across failure domains |
| Priorities | Higher priority pods get scheduled first |

---

### kube-controller-manager

The **Controller Manager** runs controller processes. Each controller is a separate process but compiled into a single binary.

```
┌────────────────────────────────────────────────────────────┐
│               kube-controller-manager                       │
│                                                            │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  Node           │  │  Replication    │                  │
│  │  Controller     │  │  Controller     │                  │
│  │                 │  │                 │                  │
│  │ Monitors node   │  │ Ensures desired │                  │
│  │ health, marks   │  │ replica count   │                  │
│  │ NotReady after  │  │ is maintained   │                  │
│  │ 40s, evicts     │  │                 │                  │
│  │ after 5min      │  │                 │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                            │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  Endpoints      │  │  ServiceAccount │                  │
│  │  Controller     │  │  Controller     │                  │
│  │                 │  │                 │                  │
│  │ Populates       │  │ Creates default │                  │
│  │ Endpoints       │  │ ServiceAccount  │                  │
│  │ objects         │  │ for namespaces  │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                            │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  Deployment     │  │  Job            │                  │
│  │  Controller     │  │  Controller     │                  │
│  │                 │  │                 │                  │
│  │ Manages         │  │ Creates pods    │                  │
│  │ ReplicaSets     │  │ for one-time    │                  │
│  │ for Deployments │  │ tasks           │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                            │
│  + Namespace, PV, Garbage Collection, CronJob, ...         │
│                                                            │
│  ────────────────────────────────────────────               │
│  Each controller runs a RECONCILIATION LOOP:               │
│                                                            │
│    while true:                                             │
│      actual = get_current_state()                          │
│      desired = get_desired_state() # from etcd             │
│      if actual != desired:                                  │
│        take_action(actual, desired)                        │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### cloud-controller-manager (optional)

For cloud-hosted clusters, the **Cloud Controller Manager** interacts with the cloud provider's APIs:

```
┌──────────────────────────────────────┐
│     cloud-controller-manager          │
│                                      │
│  ┌───────────┐  ┌───────────┐        │
│  │   Node    │  │  Route    │        │
│  │Controller │  │Controller │        │
│  │           │  │           │        │
│  │ Check if  │  │ Configure │        │
│  │ VM still  │  │ cloud     │        │
│  │ exists    │  │ routes    │        │
│  └───────────┘  └───────────┘        │
│                                      │
│  ┌───────────┐                       │
│  │  Service  │                       │
│  │Controller │                       │
│  │           │                       │
│  │ Provision │                       │
│  │ cloud LBs │                       │
│  └───────────┘                       │
└──────────────────────────────────────┘
```

---

## Node Components (Data Plane)

Node components run on **every worker node**, maintaining running pods and providing the Kubernetes runtime environment.

### kubelet

The **kubelet** is the primary node agent. It ensures containers are running in a Pod.

```
┌──────────────────────────────────────────────────────────┐
│                        kubelet                            │
│                                                          │
│   ┌──────────────────────────────────────────────┐       │
│   │              Watch API Server                │       │
│   │     (for PodSpecs assigned to this node)     │       │
│   └───────────────────┬──────────────────────────┘       │
│                       │                                  │
│           ┌───────────▼───────────┐                      │
│           │  PodSpec received?    │                      │
│           │  Compare with actual  │                      │
│           └───────────┬───────────┘                      │
│                       │                                  │
│           ┌───────────▼───────────┐                      │
│           │   Container Runtime   │                      │
│           │   Interface (CRI)     │                      │
│           │                       │                      │
│           │  ┌─────────────────┐  │                      │
│           │  │  containerd /   │  │                      │
│           │  │  CRI-O          │  │                      │
│           │  └─────────────────┘  │                      │
│           └───────────┬───────────┘                      │
│                       │                                  │
│           ┌───────────▼───────────┐                      │
│           │  Manage containers   │                      │
│           │  • Start / Stop      │                      │
│           │  • Health checks     │                      │
│           │  • Report status     │                      │
│           │  • Resource monitor  │                      │
│           └──────────────────────┘                       │
│                                                          │
│   Also handles:                                          │
│   • Volume mounting (CSI)                                │
│   • Device plugins (GPU, etc.)                           │
│   • Container logs                                       │
│   • Static pods                                          │
│   • Node registration                                    │
└──────────────────────────────────────────────────────────┘
```

**Key Points:**
- Does **not** manage containers not created by Kubernetes
- Registers the node with the API server
- Receives PodSpecs through the API server
- Reports node and pod status back to the API server
- Runs health checks (liveness, readiness, startup probes)

```bash
# Check kubelet status on a node
sudo systemctl status kubelet

# View kubelet logs
sudo journalctl -u kubelet -f

# Kubelet config location
cat /var/lib/kubelet/config.yaml
```

---

### kube-proxy

**kube-proxy** maintains network rules on nodes, allowing network communication to pods from inside or outside the cluster.

```
┌──────────────────────────────────────────────────────────┐
│                      kube-proxy                           │
│                                                          │
│   Watches Services and Endpoints from API Server         │
│                                                          │
│   MODE 1: iptables (default)                             │
│   ┌────────────────────────────────────────────┐         │
│   │  Service: web-svc (10.96.0.100:80)         │         │
│   │                                            │         │
│   │  iptables rules:                           │         │
│   │  -A KUBE-SERVICES -d 10.96.0.100/32        │         │
│   │   -p tcp --dport 80 -j KUBE-SVC-XXXXX      │         │
│   │                                            │         │
│   │  -A KUBE-SVC-XXXXX -m statistic            │         │
│   │   --probability 0.33 -j KUBE-SEP-POD1      │         │
│   │  -A KUBE-SVC-XXXXX -m statistic            │         │
│   │   --probability 0.50 -j KUBE-SEP-POD2      │         │
│   │  -A KUBE-SVC-XXXXX -j KUBE-SEP-POD3        │         │
│   │                                            │         │
│   │  → Random load balancing via iptables       │         │
│   └────────────────────────────────────────────┘         │
│                                                          │
│   MODE 2: IPVS (better for large clusters)               │
│   ┌────────────────────────────────────────────┐         │
│   │  Uses Linux IPVS (IP Virtual Server)       │         │
│   │  Supports: round-robin, least-connection,  │         │
│   │           source-hash, shortest-expected   │         │
│   │  Better performance at scale (O(1) lookup) │         │
│   └────────────────────────────────────────────┘         │
│                                                          │
│   MODE 3: nftables (newer)                               │
│   ┌────────────────────────────────────────────┐         │
│   │  Successor to iptables, better performance │         │
│   │  Available in K8s 1.29+                    │         │
│   └────────────────────────────────────────────┘         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Key Point:** kube-proxy does NOT directly handle traffic. It sets up rules in the kernel (iptables/IPVS) that handle the actual packet forwarding.

```bash
# Check kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# View iptables rules created by kube-proxy
sudo iptables -t nat -L KUBE-SERVICES -n | head -20
```

---

### Container Runtime

The **Container Runtime** is the software responsible for running containers.

```
┌─────────────────────────────────────────────────────────┐
│                  Container Runtime Stack                  │
│                                                         │
│   kubelet                                               │
│     │                                                   │
│     │ CRI (Container Runtime Interface)                 │
│     │ gRPC protocol                                     │
│     ▼                                                   │
│   ┌─────────────────────────────────────────┐           │
│   │  High-Level Runtime                     │           │
│   │                                         │           │
│   │  containerd          CRI-O              │           │
│   │  (Docker Inc)        (Red Hat)          │           │
│   │  Most popular        OCI-focused        │           │
│   └───────────────────┬─────────────────────┘           │
│                       │                                 │
│                       │ OCI Runtime Spec                │
│                       ▼                                 │
│   ┌─────────────────────────────────────────┐           │
│   │  Low-Level Runtime                      │           │
│   │                                         │           │
│   │  runc (default)    kata-containers      │           │
│   │  (OCI reference)   (VM-based isolation) │           │
│   │                    gVisor (sandboxed)    │           │
│   └─────────────────────────────────────────┘           │
│                                                         │
│   Note: Docker was removed as a runtime in K8s 1.24     │
│   (dockershim deprecation). Docker images still work     │
│   because they're OCI-compliant.                        │
└─────────────────────────────────────────────────────────┘
```

| Runtime | Description | Use Case |
|---------|-------------|----------|
| **containerd** | Industry standard, lightweight | Default for most distros |
| **CRI-O** | OCI-focused, lightweight | OpenShift, minimal footprint |
| **runc** | Low-level OCI reference implementation | Default low-level runtime |
| **kata-containers** | VM-based container isolation | Multi-tenant, high security |
| **gVisor** | Application kernel sandboxing | Untrusted workloads |

---

## Add-ons

Add-ons extend Kubernetes functionality and typically run as pods in the `kube-system` namespace.

### CoreDNS

Provides DNS-based service discovery within the cluster.

```
┌──────────────────────────────────────────────────────────┐
│                       CoreDNS                             │
│                                                          │
│   Pod A wants to reach Service "web-svc"                 │
│                                                          │
│   Pod A ──DNS query──▶ CoreDNS Pod                       │
│    "web-svc.default.svc.cluster.local"                   │
│                           │                              │
│                     ┌─────▼──────┐                       │
│                     │  Lookup    │                       │
│                     │  ClusterIP │                       │
│                     └─────┬──────┘                       │
│                           │                              │
│                     Returns: 10.96.0.100                 │
│                                                          │
│   DNS Format:                                            │
│   <service>.<namespace>.svc.<cluster-domain>             │
│                                                          │
│   Examples:                                              │
│   • web-svc.default.svc.cluster.local                    │
│   • postgres.database.svc.cluster.local                  │
│   • redis.cache.svc.cluster.local                        │
└──────────────────────────────────────────────────────────┘
```

### Kubernetes Dashboard

Web-based UI for managing cluster resources.

```bash
# Install dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Access via proxy
kubectl proxy
# Open: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### CNI (Container Network Interface)

Plugins that provide pod networking:

```
┌────────────────────────────────────────────────────────────┐
│                    CNI Plugins                              │
│                                                            │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│   │  Calico    │  │  Cilium    │  │  Flannel   │          │
│   │            │  │            │  │            │          │
│   │ BGP-based  │  │ eBPF-based │  │ Overlay    │          │
│   │ L3 routing │  │ advanced   │  │ VXLAN      │          │
│   │ Net Policy │  │ Net Policy │  │ Simple     │          │
│   │ Production │  │ Observ.    │  │ Learning   │          │
│   └────────────┘  └────────────┘  └────────────┘          │
│                                                            │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐          │
│   │  Weave Net │  │  AWS VPC   │  │  Azure CNI │          │
│   │            │  │  CNI       │  │            │          │
│   │ Mesh       │  │            │  │ Native VPC │          │
│   │ overlay    │  │ Native VPC │  │ networking │          │
│   │ encryption │  │ networking │  │ for AKS    │          │
│   └────────────┘  └────────────┘  └────────────┘          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Communication Flow

### How `kubectl apply` Works End-to-End

```
 User                API Server           etcd          Scheduler       kubelet
  │                     │                  │               │              │
  │  kubectl apply -f   │                  │               │              │
  │  deployment.yaml    │                  │               │              │
  │────────────────────▶│                  │               │              │
  │                     │                  │               │              │
  │                     │  1. AuthN/AuthZ  │               │              │
  │                     │  2. Admission    │               │              │
  │                     │     Control      │               │              │
  │                     │  3. Validation   │               │              │
  │                     │                  │               │              │
  │                     │──Store object───▶│               │              │
  │                     │                  │               │              │
  │                     │  4. Notify       │               │              │
  │                     │     watchers     │               │              │
  │                     │                  │               │              │
  │                     │  Deployment Controller           │              │
  │                     │  creates ReplicaSet              │              │
  │                     │──────────────────▶│              │              │
  │                     │                  │               │              │
  │                     │  ReplicaSet Controller           │              │
  │                     │  creates Pod objects             │              │
  │                     │──Store Pods─────▶│               │              │
  │                     │                  │               │              │
  │                     │  5. Scheduler watches            │              │
  │                     │     unscheduled pods             │              │
  │                     │─────────────────────────────────▶│              │
  │                     │                                  │              │
  │                     │  6. Scheduler assigns            │              │
  │                     │     node to pod                  │              │
  │                     │◀─────────────────────────────────│              │
  │                     │──Store binding──▶│               │              │
  │                     │                  │               │              │
  │                     │  7. kubelet watches pods         │              │
  │                     │     assigned to its node         │              │
  │                     │─────────────────────────────────────────────────▶│
  │                     │                                                 │
  │                     │  8. kubelet tells container                     │
  │                     │     runtime to start container                  │
  │                     │                                                 │
  │                     │  9. kubelet reports pod status                  │
  │                     │◀────────────────────────────────────────────────│
  │                     │──Store status───▶│               │              │
  │                     │                  │               │              │
  │◀────────────────────│                  │               │              │
  │  Response: created  │                  │               │              │
```

### Component Communication Summary

```
┌────────────────────────────────────────────────────────────────┐
│                COMMUNICATION PATTERNS                          │
│                                                                │
│  ┌──────────┐                                                  │
│  │  kubectl  │───HTTPS/REST──▶ API Server                     │
│  └──────────┘                     │                            │
│                                   │ HTTPS                      │
│                              ┌────┴────┐                       │
│                              ▼         ▼                       │
│                          ┌──────┐  ┌──────────┐                │
│                          │ etcd │  │Scheduler │                │
│                          └──────┘  │Controller│                │
│                                    │Manager   │                │
│                                    └──────────┘                │
│                                                                │
│  API Server ───HTTPS───▶ kubelet (on each node)               │
│  kubelet ───HTTPS───▶ API Server (status reporting)           │
│  kubelet ───gRPC/CRI──▶ Container Runtime                     │
│                                                                │
│  IMPORTANT:                                                    │
│  • ALL communication goes through the API Server               │
│  • Components do NOT talk to each other directly               │
│  • All connections are TLS-encrypted                           │
│  • Components use watch mechanisms (not polling)               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Anatomy of a Production Cluster

### Three-Master HA Setup

```
┌─────────────────────────────────────────────────────────────────┐
│                 PRODUCTION HA CLUSTER                            │
│                                                                 │
│  ┌─────────── Load Balancer (HAProxy/NLB) ──────────┐          │
│  │                  :6443                            │          │
│  │     ┌────────────┼────────────┐                   │          │
│  │     ▼            ▼            ▼                   │          │
│  │ ┌────────┐  ┌────────┐  ┌────────┐               │          │
│  │ │Master 1│  │Master 2│  │Master 3│               │          │
│  │ │API Srv │  │API Srv │  │API Srv │               │          │
│  │ │Sched.  │  │Sched.  │  │Sched.  │  (standby)    │          │
│  │ │Ctrl Mgr│  │Ctrl Mgr│  │Ctrl Mgr│  (standby)    │          │
│  │ │etcd    │  │etcd    │  │etcd    │               │          │
│  │ └────────┘  └────────┘  └────────┘               │          │
│  │      │           │           │                    │          │
│  │      └─── etcd Raft Cluster ─┘                    │          │
│  └───────────────────────────────────────────────────┘          │
│                                                                 │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐   │
│  │Worker 1│  │Worker 2│  │Worker 3│  │Worker 4│  │Worker 5│   │
│  │16 CPU  │  │16 CPU  │  │16 CPU  │  │32 CPU  │  │32 CPU  │   │
│  │64 GB   │  │64 GB   │  │64 GB   │  │128 GB  │  │128 GB  │   │
│  │kubelet │  │kubelet │  │kubelet │  │kubelet │  │kubelet │   │
│  │proxy   │  │proxy   │  │proxy   │  │proxy   │  │proxy   │   │
│  │runtime │  │runtime │  │runtime │  │runtime │  │runtime │   │
│  └────────┘  └────────┘  └────────┘  └────────┘  └────────┘   │
│                                                                 │
│  Workers 1-3: General workloads                                 │
│  Workers 4-5: GPU/Memory-intensive workloads (labeled)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Best Practices for Production:**
- Run **3 or 5** control plane nodes for HA
- Use a **load balancer** in front of API servers
- Place etcd on **fast SSDs** (etcd is I/O sensitive)
- Separate etcd from control plane for very large clusters
- Use **node pools** for different workload types

---

## Troubleshooting Tips

| Symptom | Component | Investigation |
|---------|-----------|---------------|
| `kubectl` commands hang | API Server | Check API server logs, certificates, etcd connectivity |
| Pods stuck in `Pending` | Scheduler | `kubectl describe pod` — check Events for scheduling failures |
| Pods crash-looping | kubelet | `kubectl logs <pod>`, check container runtime logs |
| Services not reachable | kube-proxy | Check kube-proxy pods, iptables rules, CNI health |
| Node `NotReady` | kubelet | SSH to node, check `systemctl status kubelet`, check resources |
| DNS not resolving | CoreDNS | Check CoreDNS pods, `kubectl logs -n kube-system <coredns>` |
| etcd high latency | etcd | Check disk I/O, network latency between etcd members |

```bash
# Quick health check commands
kubectl get componentstatuses            # Overall component health
kubectl get nodes                        # Node status
kubectl get pods -n kube-system          # Control plane pod health
kubectl cluster-info                     # Cluster info
kubectl top nodes                        # Resource usage
```

---

## Summary Table

| Component | Location | Role |
|-----------|----------|------|
| **kube-apiserver** | Control Plane | REST API gateway, authN/authZ, admission control |
| **etcd** | Control Plane | Distributed key-value store for all cluster state |
| **kube-scheduler** | Control Plane | Assigns pods to nodes (filter → score → bind) |
| **kube-controller-manager** | Control Plane | Runs reconciliation loops (node, replication, endpoints, etc.) |
| **cloud-controller-manager** | Control Plane | Interfaces with cloud provider APIs |
| **kubelet** | Every Node | Node agent — ensures containers match PodSpecs |
| **kube-proxy** | Every Node | Maintains network rules for Services (iptables/IPVS) |
| **Container Runtime** | Every Node | Runs containers (containerd, CRI-O) |
| **CoreDNS** | Add-on | DNS-based service discovery |
| **CNI Plugin** | Add-on | Pod-to-pod networking |

---

## Quick Revision Questions

1. **Which component is the only one that directly communicates with etcd?**
   <details><summary>Answer</summary>The kube-apiserver is the only component that directly reads from and writes to etcd. All other components interact with etcd indirectly through the API server.</details>

2. **What happens if the kube-scheduler goes down?**
   <details><summary>Answer</summary>Existing pods continue to run normally. However, new pods that need scheduling will remain in the Pending state until the scheduler recovers. In an HA setup, a standby scheduler takes over via leader election.</details>

3. **Explain the two phases of the scheduling process.**
   <details><summary>Answer</summary>1) **Filtering (Predicates):** Eliminates nodes that cannot run the pod (insufficient resources, taints, selectors). 2) **Scoring (Priorities):** Ranks remaining feasible nodes based on factors like resource balance, affinity, topology. The highest-scored node gets the pod.</details>

4. **Why was Docker (dockershim) removed as a container runtime in Kubernetes 1.24?**
   <details><summary>Answer</summary>Docker is a full platform (CLI + build + swarm + runtime). K8s only needs the runtime portion. dockershim was a translation layer between CRI and Docker, adding unnecessary complexity. containerd (which Docker uses internally) speaks CRI natively and is more efficient. Docker-built images still work because they follow the OCI standard.</details>

5. **What is the difference between kube-proxy iptables mode and IPVS mode?**
   <details><summary>Answer</summary>iptables mode: Uses Linux iptables rules for packet routing; O(n) lookup; random load balancing; suitable for small clusters. IPVS mode: Uses Linux IPVS kernel module; O(1) lookup; supports round-robin, least-connection, and more; better performance for large clusters (1000+ services).</details>

6. **Draw the flow of what happens when you run `kubectl run nginx --image=nginx`.**
   <details><summary>Answer</summary>1) kubectl sends REST request to API server → 2) API server authenticates, authorizes, admits → 3) API server stores Pod object in etcd → 4) Scheduler watches for unassigned pods, selects a node → 5) API server updates Pod with node assignment in etcd → 6) kubelet on that node watches for assigned pods → 7) kubelet instructs container runtime (containerd) via CRI to pull image and start container → 8) kubelet reports pod status to API server → 9) API server stores status in etcd.</details>

---

[← Previous: Kubernetes Concepts](../01-Kubernetes-Concepts/01-what-is-kubernetes.md) | [Back to README](../README.md) | [Next: Pods →](../03-Pods/01-pods.md)
