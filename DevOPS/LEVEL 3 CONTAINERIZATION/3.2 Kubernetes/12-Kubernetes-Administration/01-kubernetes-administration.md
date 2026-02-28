# Unit 12: Kubernetes Administration

[← Previous: Advanced Topics](../11-Advanced-Topics/01-advanced-topics.md) | [Back to README](../README.md)

---

## Chapter Overview

This unit covers the operational side of running Kubernetes in production: **cluster setup**, **upgrades**, **backup and disaster recovery**, **troubleshooting**, **performance tuning**, and **multi-tenancy**. These are the skills needed to keep clusters healthy and reliable.

---

## 12.1 Cluster Setup

### Cluster Setup Options

```
┌──────────────────────────────────────────────────────────────┐
│              CLUSTER SETUP OPTIONS                            │
│                                                              │
│  Local / Development:                                        │
│  ┌────────────────┬──────────────────────────────────────┐   │
│  │ minikube       │ Single-node, beginner-friendly        │   │
│  │ kind           │ K8s in Docker containers (CI-friendly)│   │
│  │ k3d            │ k3s in Docker (lightweight)           │   │
│  │ Docker Desktop │ Built-in K8s (Mac/Windows)            │   │
│  └────────────────┴──────────────────────────────────────┘   │
│                                                              │
│  Production (Self-Managed):                                  │
│  ┌────────────────┬──────────────────────────────────────┐   │
│  │ kubeadm        │ Official K8s bootstrap tool           │   │
│  │ kubespray      │ Ansible-based, production-ready       │   │
│  │ kops           │ AWS-focused cluster management        │   │
│  │ RKE2 / k3s     │ Rancher lightweight distributions     │   │
│  └────────────────┴──────────────────────────────────────┘   │
│                                                              │
│  Production (Managed):                                       │
│  ┌────────────────┬──────────────────────────────────────┐   │
│  │ EKS            │ AWS managed K8s                       │   │
│  │ AKS            │ Azure managed K8s                     │   │
│  │ GKE            │ Google managed K8s                    │   │
│  │ OpenShift      │ Red Hat enterprise K8s                │   │
│  └────────────────┴──────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### kubeadm Cluster Setup

```bash
# ═══════════ Prerequisites (all nodes) ═══════════

# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Load kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# Sysctl settings
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# Install containerd (container runtime)
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
# Set SystemdCgroup = true in config.toml
sudo systemctl restart containerd

# Install kubeadm, kubelet, kubectl
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# ═══════════ Control Plane (master node only) ═══════════

# Initialize cluster
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<MASTER_IP> \
  --kubernetes-version=v1.29.0

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install CNI plugin (e.g., Calico)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml

# ═══════════ Worker Nodes ═══════════

# Join cluster (use the token from kubeadm init output)
sudo kubeadm join <MASTER_IP>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>

# If token expired, generate a new one on master:
kubeadm token create --print-join-command
```

### kubeconfig Management

```bash
# View current config
kubectl config view
kubectl config current-context

# Switch context
kubectl config use-context production-cluster
kubectl config get-contexts

# Set namespace for current context
kubectl config set-context --current --namespace=production

# Merge multiple kubeconfigs
export KUBECONFIG=~/.kube/config:~/.kube/other-config
kubectl config view --flatten > ~/.kube/merged-config

# Create context
kubectl config set-context dev --cluster=dev-cluster \
  --user=dev-admin --namespace=development
```

```
┌──────────────────────────────────────────────────────────────┐
│              KUBECONFIG STRUCTURE                              │
│                                                              │
│  ~/.kube/config                                              │
│  ┌────────────────────────────────────────────┐              │
│  │ clusters:                                  │              │
│  │   - cluster: (API server URL, CA cert)     │              │
│  │     name: production                       │              │
│  │                                            │              │
│  │ users:                                     │              │
│  │   - user: (client cert, token, or OIDC)    │              │
│  │     name: admin                            │              │
│  │                                            │              │
│  │ contexts:     ┌──────────┐                 │              │
│  │   - context:  │ cluster  │ = cluster +     │              │
│  │     name: prod│ user     │   user +         │              │
│  │               │ namespace│   namespace      │              │
│  │               └──────────┘                 │              │
│  │                                            │              │
│  │ current-context: prod                      │              │
│  └────────────────────────────────────────────┘              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 12.2 Cluster Upgrades

```
┌──────────────────────────────────────────────────────────────┐
│            CLUSTER UPGRADE STRATEGY                           │
│                                                              │
│  Rules:                                                      │
│  - Upgrade one MINOR version at a time (1.28 → 1.29)        │
│  - Always upgrade control plane FIRST, then workers          │
│  - kubelet must be within 2 minor versions of API server     │
│                                                              │
│  Version Skew Policy:                                        │
│  ┌─────────────────┬─────────────────────────────────┐       │
│  │ Component       │ Allowed skew from API server    │       │
│  ├─────────────────┼─────────────────────────────────┤       │
│  │ kubelet         │ ± 2 minor versions              │       │
│  │ kube-proxy      │ ± 2 minor versions              │       │
│  │ kubectl         │ ± 1 minor version               │       │
│  │ kube-controller │ Must match API server            │       │
│  │ kube-scheduler  │ Must match API server            │       │
│  └─────────────────┴─────────────────────────────────┘       │
│                                                              │
│  Upgrade Order:                                              │
│  1. kubeadm (tool)                                           │
│  2. Control plane (kube-apiserver, etcd, etc.)               │
│  3. kubelet + kubectl on control plane                       │
│  4. Worker nodes (one at a time — drain, upgrade, uncordon)  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### kubeadm Upgrade Steps

```bash
# ═══════════ Step 1: Upgrade kubeadm ═══════════
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm=1.29.0-*
sudo apt-mark hold kubeadm

# Verify
kubeadm version

# ═══════════ Step 2: Upgrade Control Plane ═══════════
# Check upgrade plan
sudo kubeadm upgrade plan

# Apply upgrade
sudo kubeadm upgrade apply v1.29.0

# ═══════════ Step 3: Upgrade kubelet on control plane ═══════════
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.29.0-* kubectl=1.29.0-*
sudo apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# ═══════════ Step 4: Upgrade Worker Nodes (one at a time) ═══════════

# From control plane: drain the worker
kubectl drain worker-1 --ignore-daemonsets --delete-emptydir-data

# On worker-1:
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.29.0-*
sudo kubeadm upgrade node
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet=1.29.0-* kubectl=1.29.0-*
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# From control plane: uncordon the worker
kubectl uncordon worker-1

# Verify
kubectl get nodes   # All should show v1.29.0
```

---

## 12.3 Backup and Disaster Recovery

### etcd Backup

```
┌──────────────────────────────────────────────────────────────┐
│            WHAT TO BACK UP                                    │
│                                                              │
│  Critical:                                                   │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ etcd data         ALL cluster state (resources,       │   │
│  │                   secrets, configmaps, etc.)           │   │
│  │ Certificates      /etc/kubernetes/pki/                │   │
│  │ kubeconfig        /etc/kubernetes/admin.conf          │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  Also important:                                             │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ Persistent Volumes  Application data (separate backup)│   │
│  │ Git repo           Manifests / Helm values (GitOps)   │   │
│  │ External secrets   Vault, cloud secret managers       │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# ═══════════ etcd Backup ═══════════

# Snapshot etcd
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify snapshot
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-snapshot.db --write-table

# ═══════════ etcd Restore ═══════════

# Stop kube-apiserver (move manifests)
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# Restore snapshot
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restored \
  --initial-cluster=master=https://127.0.0.1:2380 \
  --initial-advertise-peer-urls=https://127.0.0.1:2380

# Update etcd manifest to use new data-dir
# Edit /etc/kubernetes/manifests/etcd.yaml
# Change --data-dir to /var/lib/etcd-restored

# Restart kube-apiserver
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

### Velero — Backup & Migration Tool

```
┌──────────────────────────────────────────────────────────────┐
│                    VELERO                                     │
│                                                              │
│  Velero backs up K8s resources AND persistent volumes        │
│  to cloud storage (S3, GCS, Azure Blob)                      │
│                                                              │
│  ┌──── Cluster ────┐      ┌──── Cloud Storage ────┐         │
│  │                 │      │                       │         │
│  │  Resources      │ ────▶│  Backup files         │         │
│  │  (YAML)         │      │  (resources + PV      │         │
│  │                 │      │   snapshots)           │         │
│  │  Persistent     │      │                       │         │
│  │  Volumes        │      │                       │         │
│  └─────────────────┘      └───────────────────────┘         │
│                                                              │
│  Use Cases:                                                  │
│  - Disaster recovery                                         │
│  - Cluster migration                                         │
│  - Namespace backup before changes                           │
│  - Scheduled automated backups                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Install Velero CLI
# Configure with cloud provider

# Create backup
velero backup create my-backup

# Backup specific namespace
velero backup create ns-backup --include-namespaces production

# Backup with label selector
velero backup create app-backup --selector app=web

# Schedule backups
velero schedule create daily-backup --schedule="0 2 * * *"
velero schedule create hourly-prod --schedule="0 * * * *" \
  --include-namespaces production

# List backups
velero backup get

# Restore
velero restore create my-restore --from-backup my-backup
velero restore create --from-backup ns-backup --include-namespaces production
```

---

## 12.4 Troubleshooting

### Systematic Troubleshooting Approach

```
┌──────────────────────────────────────────────────────────────┐
│         TROUBLESHOOTING FLOWCHART                             │
│                                                              │
│  Problem detected                                            │
│       │                                                      │
│       ▼                                                      │
│  ┌────────────┐                                              │
│  │ Pod issue? │──── Yes ──▶ Check Pod status & events        │
│  └─────┬──────┘            kubectl describe pod <name>       │
│        │ No                kubectl logs <pod>                 │
│        ▼                   kubectl logs <pod> --previous     │
│  ┌────────────┐                                              │
│  │ Service    │──── Yes ──▶ Check endpoints & selectors      │
│  │ issue?     │            kubectl get endpoints <svc>       │
│  └─────┬──────┘            kubectl describe svc <svc>        │
│        │ No                                                  │
│        ▼                                                     │
│  ┌────────────┐                                              │
│  │ Node issue?│──── Yes ──▶ Check node status & resources    │
│  └─────┬──────┘            kubectl describe node <name>      │
│        │ No                kubectl top nodes                  │
│        ▼                                                     │
│  ┌────────────┐                                              │
│  │ Cluster    │──── Yes ──▶ Check control plane components   │
│  │ issue?     │            kubectl get componentstatus       │
│  └────────────┘            kubectl get nodes                  │
│                            systemctl status kubelet           │
│                            journalctl -u kubelet              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Common Pod States and Fixes

```
┌──────────────────────────────────────────────────────────────┐
│  POD STATUS TROUBLESHOOTING                                   │
│                                                              │
│  ┌──────────────────┬────────────────────────────────────┐   │
│  │ Status           │ Common Causes & Fixes              │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ Pending          │ - Insufficient resources           │   │
│  │                  │   → Scale cluster or reduce req    │   │
│  │                  │ - No matching node (selector/taint)│   │
│  │                  │   → Fix nodeSelector or add tol    │   │
│  │                  │ - PVC not bound                    │   │
│  │                  │   → Check PV/StorageClass          │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ ImagePullBackOff │ - Wrong image name/tag             │   │
│  │ ErrImagePull     │   → Fix image: field               │   │
│  │                  │ - Private registry, no secret      │   │
│  │                  │   → Add imagePullSecrets            │   │
│  │                  │ - Image doesn't exist              │   │
│  │                  │   → Verify image in registry        │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ CrashLoopBackOff │ - App crashes on start             │   │
│  │                  │   → Check logs: kubectl logs --prev│   │
│  │                  │ - Missing config/secret             │   │
│  │                  │   → Check env vars and mounts       │   │
│  │                  │ - Liveness probe too aggressive     │   │
│  │                  │   → Increase initialDelaySeconds    │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ OOMKilled        │ - Memory limit too low              │   │
│  │                  │   → Increase memory limits          │   │
│  │                  │ - Memory leak in application        │   │
│  │                  │   → Fix app or add monitoring       │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ CreateContainer  │ - ConfigMap/Secret not found        │   │
│  │ Error            │   → Verify CM/Secret exists         │   │
│  │                  │ - Volume mount failure              │   │
│  │                  │   → Check PV/PVC status             │   │
│  ├──────────────────┼────────────────────────────────────┤   │
│  │ Evicted          │ - Node under resource pressure      │   │
│  │                  │   → Check node disk/memory          │   │
│  │                  │   → Set appropriate QoS class       │   │
│  └──────────────────┴────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Essential Troubleshooting Commands

```bash
# ═══════════ Pod Investigation ═══════════
kubectl get pods -A                          # All pods, all namespaces
kubectl get pods -o wide                     # Show node + IP
kubectl describe pod <pod>                   # Full details + events
kubectl logs <pod> -c <container>            # Container logs
kubectl logs <pod> --previous                # Logs from crashed container
kubectl exec -it <pod> -- /bin/sh            # Shell into container
kubectl exec <pod> -- env                    # Check env vars
kubectl exec <pod> -- cat /etc/resolv.conf   # Check DNS
kubectl exec <pod> -- wget -qO- http://svc   # Test connectivity

# ═══════════ Service / Network ═══════════
kubectl get svc                              # List services
kubectl get endpoints <svc>                  # Are pods registered?
kubectl describe svc <svc>                   # Service details
kubectl run tmp --rm -it --image=busybox -- \
  wget -qO- http://<svc>.<ns>.svc.cluster.local  # DNS test

# ═══════════ Node Investigation ═══════════
kubectl get nodes                            # Node status
kubectl describe node <node>                 # Details + conditions
kubectl top nodes                            # Resource usage
kubectl cordon <node>                        # Mark unschedulable
kubectl drain <node> --ignore-daemonsets     # Evict pods
kubectl uncordon <node>                      # Mark schedulable

# ═══════════ Cluster / Control Plane ═══════════
kubectl cluster-info                         # Cluster endpoints
kubectl get events --sort-by='.lastTimestamp' # Recent events
kubectl get cs                               # Component status
sudo systemctl status kubelet                # Kubelet status
sudo journalctl -u kubelet -f               # Kubelet logs

# ═══════════ Resource Investigation ═══════════
kubectl api-resources                        # All resource types
kubectl explain pod.spec.containers          # Field documentation
kubectl get all -n <namespace>               # All resources in ns
kubectl diff -f manifest.yaml                # Diff before apply
```

### Debugging Network Issues

```bash
# Deploy a debug pod
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash

# Inside the debug pod:
nslookup kubernetes.default           # Test DNS
nslookup <service>.<namespace>        # Test service DNS
curl http://<service>:<port>/health   # Test HTTP
ping <pod-ip>                         # Test pod connectivity
traceroute <pod-ip>                   # Trace route
nc -zv <host> <port>                  # Test TCP port
```

---

## 12.5 Performance Tuning

```
┌──────────────────────────────────────────────────────────────┐
│          PERFORMANCE TUNING CHECKLIST                         │
│                                                              │
│  Pod Level:                                                  │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ ✓ Set CPU/memory requests AND limits                  │   │
│  │ ✓ Use resource-appropriate QoS (Guaranteed for prod)  │   │
│  │ ✓ Right-size resources (use VPA recommendations)      │   │
│  │ ✓ Use HPA for automatic horizontal scaling            │   │
│  │ ✓ Set appropriate probe timing (not too aggressive)   │   │
│  │ ✓ Use readinessGates for dependencies                 │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  Cluster Level:                                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ ✓ Enable Cluster Autoscaler for dynamic node scaling  │   │
│  │ ✓ Use LimitRange/ResourceQuota per namespace          │   │
│  │ ✓ Monitor etcd performance (latency < 10ms)           │   │
│  │ ✓ Use SSD for etcd storage                            │   │
│  │ ✓ Spread control plane across AZs                     │   │
│  │ ✓ Set appropriate --max-pods per node (default 110)   │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  Network Level:                                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ ✓ Choose CNI based on needs (Calico for policy,       │   │
│  │   Cilium for eBPF performance)                        │   │
│  │ ✓ Use NodeLocal DNSCache for DNS performance          │   │
│  │ ✓ Set appropriate kube-proxy mode (IPVS > iptables)   │   │
│  │ ✓ Use topology-aware traffic routing                  │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  Storage Level:                                              │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ ✓ Use local SSD for latency-sensitive workloads       │   │
│  │ ✓ Match StorageClass to workload needs                │   │
│  │ ✓ Use volume expansion instead of migration           │   │
│  │ ✓ Set appropriate fsGroup for volume permissions      │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 12.6 Multi-Tenancy

Multi-tenancy allows multiple teams or applications to share a cluster securely.

```
┌──────────────────────────────────────────────────────────────┐
│           MULTI-TENANCY STRATEGIES                            │
│                                                              │
│  Soft Multi-Tenancy (shared cluster):                        │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Cluster                                        │        │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐     │        │
│  │  │ NS: team-a│ │ NS: team-b│ │ NS: team-c│     │        │
│  │  │           │ │           │ │           │     │        │
│  │  │ RBAC      │ │ RBAC      │ │ RBAC      │     │        │
│  │  │ Quotas    │ │ Quotas    │ │ Quotas    │     │        │
│  │  │ NetPol    │ │ NetPol    │ │ NetPol    │     │        │
│  │  │ PSS       │ │ PSS       │ │ PSS       │     │        │
│  │  └───────────┘ └───────────┘ └───────────┘     │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Hard Multi-Tenancy (separate clusters):                     │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │  Cluster A  │ │  Cluster B  │ │  Cluster C  │            │
│  │  (Team A)   │ │  (Team B)   │ │  (Team C)   │            │
│  │  Full       │ │  Full       │ │  Full       │            │
│  │  isolation  │ │  isolation  │ │  isolation  │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                              │
│  Virtual Clusters (vCluster):                                │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Host Cluster                                   │        │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐  │        │
│  │  │ vCluster A │ │ vCluster B │ │ vCluster C │  │        │
│  │  │ Own API    │ │ Own API    │ │ Own API    │  │        │
│  │  │ server,    │ │ server,    │ │ server,    │  │        │
│  │  │ scheduler  │ │ scheduler  │ │ scheduler  │  │        │
│  │  └────────────┘ └────────────┘ └────────────┘  │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Namespace Isolation Checklist

```yaml
# Complete namespace isolation setup for a team

# 1. Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: team-alpha
  labels:
    team: alpha
    pod-security.kubernetes.io/enforce: restricted
---
# 2. ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-alpha-quota
  namespace: team-alpha
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "50"
    services: "20"
    persistentvolumeclaims: "20"
---
# 3. LimitRange
apiVersion: v1
kind: LimitRange
metadata:
  name: team-alpha-limits
  namespace: team-alpha
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    max:
      cpu: "2"
      memory: "4Gi"
---
# 4. NetworkPolicy (default deny)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: team-alpha
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
# 5. Allow DNS + internal traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal-and-dns
  namespace: team-alpha
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: alpha
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          team: alpha
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
---
# 6. RBAC Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-alpha-dev
  namespace: team-alpha
rules:
- apiGroups: ["", "apps", "batch", "networking.k8s.io"]
  resources: ["*"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
# 7. RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-alpha-dev-binding
  namespace: team-alpha
subjects:
- kind: Group
  name: team-alpha-developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: team-alpha-dev
  apiGroup: rbac.authorization.k8s.io
```

---

## 12.7 Useful Administrative Commands Reference

```bash
# ═══════════ Cluster Health ═══════════
kubectl cluster-info
kubectl get nodes -o wide
kubectl get cs                               # Component statuses
kubectl get all -A                           # Everything everywhere
kubectl api-versions                         # Available API versions

# ═══════════ Certificate Management ═══════════
kubeadm certs check-expiration               # Check cert expiry
kubeadm certs renew all                      # Renew all certs
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates

# ═══════════ etcd Operations ═══════════
ETCDCTL_API=3 etcdctl member list            # List etcd members
ETCDCTL_API=3 etcdctl endpoint health        # Check health
ETCDCTL_API=3 etcdctl endpoint status --write-table

# ═══════════ Resource Management ═══════════
kubectl top nodes                            # Node resources
kubectl top pods --containers -A             # All pod resources
kubectl describe node <node> | grep -A20 "Allocated"  # Allocations

# ═══════════ Bulk Operations ═══════════
kubectl delete pods --field-selector status.phase=Failed -A  # Clean failed
kubectl get pods -A | grep -i evict | awk '{print $1,$2}' \
  | xargs -L1 kubectl delete pod -n         # Clean evicted
kubectl rollout restart deployment -n <ns>   # Restart all deployments

# ═══════════ Output Formatting ═══════════
kubectl get pods -o json                     # JSON output
kubectl get pods -o yaml                     # YAML output
kubectl get pods -o wide                     # Extra columns
kubectl get pods -o name                     # Just names
kubectl get pods --sort-by=.status.startTime # Sort
kubectl get pods -o custom-columns=\
  NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|---------|
| Node NotReady | `kubectl describe node` → Conditions | Check kubelet: `systemctl status kubelet`, `journalctl -u kubelet` |
| etcd unhealthy | `etcdctl endpoint health` | Check disk I/O, memory; use SSD for etcd |
| Certificate expired | `kubeadm certs check-expiration` | `kubeadm certs renew all`, restart control plane |
| Cluster upgrade fails | `kubeadm upgrade plan` | Follow version skew policy; upgrade one minor version at a time |
| Can't join worker node | Check token and CA hash | `kubeadm token create --print-join-command` on master |
| DNS resolution fails | `kubectl exec -- nslookup kubernetes` | Check CoreDNS pods: `kubectl get pods -n kube-system -l k8s-app=kube-dns` |
| Disk pressure on node | `kubectl describe node` → DiskPressure | Clean up images: `crictl rmi --prune`; remove unused PVs |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **kubeadm** | Official tool to bootstrap K8s clusters (init, join, upgrade) |
| **kubeconfig** | Config file with clusters, users, contexts for kubectl |
| **Cluster Upgrade** | One minor version at a time; control plane first, then workers |
| **Version Skew** | kubelet ±2 minor from API server; kubectl ±1 |
| **etcd Backup** | `etcdctl snapshot save` — most critical backup for cluster state |
| **etcd Restore** | `etcdctl snapshot restore` — restores to new data directory |
| **Velero** | Backup/restore K8s resources + PVs to cloud storage |
| **Drain** | Evict pods from node before maintenance (`kubectl drain`) |
| **Cordon/Uncordon** | Mark node unschedulable/schedulable |
| **Troubleshooting** | describe → logs → events → exec flow |
| **Multi-Tenancy** | Namespace isolation: RBAC + Quota + LimitRange + NetPol + PSS |
| **Performance** | Right-size resources, use autoscaling, monitor etcd latency |

---

## Quick Revision Questions

1. **What is the correct order for upgrading a Kubernetes cluster?**
   <details><summary>Answer</summary>1) Upgrade kubeadm binary on the control plane node. 2) Run `kubeadm upgrade plan` then `kubeadm upgrade apply v1.XX.0` on the control plane. 3) Upgrade kubelet and kubectl on the control plane node. 4) Drain each worker node (`kubectl drain`), upgrade kubeadm + kubelet + kubectl, run `kubeadm upgrade node`, then uncordon (`kubectl uncordon`). Always upgrade one minor version at a time (e.g., 1.28 → 1.29, never 1.28 → 1.30).</details>

2. **How do you back up and restore etcd?**
   <details><summary>Answer</summary>**Backup**: `ETCDCTL_API=3 etcdctl snapshot save /path/snapshot.db` with --endpoints, --cacert, --cert, and --key flags pointing to etcd TLS certificates (usually in `/etc/kubernetes/pki/etcd/`). **Restore**: Stop the API server, run `etcdctl snapshot restore /path/snapshot.db --data-dir=/var/lib/etcd-new`, update the etcd manifest to use the new data directory, restart the API server. Verify with `etcdctl snapshot status`.</details>

3. **A pod is in CrashLoopBackOff — what steps do you take to diagnose?**
   <details><summary>Answer</summary>1) `kubectl describe pod <name>` — check Events section for error details. 2) `kubectl logs <pod> --previous` — read logs from the crashed container. 3) Check exit code in describe output (Exit Code 137 = OOMKilled, 1 = app error). 4) Verify ConfigMaps/Secrets exist and are correctly referenced. 5) Check if liveness probe is too aggressive (low initialDelaySeconds). 6) Try running the image locally with the same env vars to reproduce.</details>

4. **What resources do you need to create for soft multi-tenancy namespace isolation?**
   <details><summary>Answer</summary>1) **Namespace** with Pod Security Standards labels (enforce: restricted). 2) **ResourceQuota** — limit total CPU, memory, pods, services per namespace. 3) **LimitRange** — set default and max resource requests/limits per container. 4) **NetworkPolicy** — default deny all, then explicit allow rules. 5) **RBAC Role + RoleBinding** — grant team members only the permissions they need. 6) **ServiceAccount** with `automountServiceAccountToken: false` for workloads.</details>

5. **What is the difference between `kubectl cordon` and `kubectl drain`?**
   <details><summary>Answer</summary>`cordon` marks a node as unschedulable — no new pods will be placed on it, but existing pods keep running. `drain` does cordon AND evicts all pods from the node (respecting PodDisruptionBudgets), moving them to other nodes. Use `drain` before node maintenance/upgrades. Use `--ignore-daemonsets` (DaemonSet pods can't move) and `--delete-emptydir-data` (allow deletion of emptyDir data). After maintenance, `uncordon` makes the node schedulable again.</details>

6. **How do you troubleshoot a Service that isn't routing traffic to pods?**
   <details><summary>Answer</summary>1) Check Service selector matches pod labels: `kubectl get svc <svc> -o yaml` and compare with `kubectl get pods --show-labels`. 2) Check endpoints: `kubectl get endpoints <svc>` — should list pod IPs. Empty = selector mismatch. 3) Verify pods are Ready: `kubectl get pods` — Readiness probe must pass. 4) Test from inside cluster: `kubectl exec <pod> -- curl http://<svc>:<port>`. 5) Check targetPort matches container's listening port. 6) For NodePort/LB, check firewall rules and security groups.</details>

---

[← Previous: Advanced Topics](../11-Advanced-Topics/01-advanced-topics.md) | [Back to README](../README.md)
