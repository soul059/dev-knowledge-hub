# Unit 5: Services and Networking

[← Previous: Workload Resources](../04-Workload-Resources/01-workload-resources.md) | [Back to README](../README.md) | [Next: Storage →](../06-Storage/01-storage.md)

---

## Chapter Overview

Kubernetes networking is fundamental to how pods communicate with each other and the outside world. This unit covers Service types, service discovery, DNS, Ingress controllers/resources, Network Policies, and CNI plugins. Understanding networking is essential for the CKA/CKAD exams and production operations.

---

## 5.1 Kubernetes Networking Model

Kubernetes imposes three fundamental networking requirements:

```
┌──────────────────────────────────────────────────────────────┐
│             KUBERNETES NETWORKING RULES                       │
│                                                              │
│  1. Every Pod gets its OWN unique IP address                 │
│  2. Pods on ANY node can communicate with ALL pods           │
│     on ANY other node WITHOUT NAT                            │
│  3. Agents on a node can communicate with ALL pods           │
│     on that node                                             │
│                                                              │
│  ┌──────── Node 1 ──────────┐  ┌──────── Node 2 ──────────┐ │
│  │  Pod A (10.244.1.5)      │  │  Pod C (10.244.2.8)      │ │
│  │  Pod B (10.244.1.6)      │  │  Pod D (10.244.2.9)      │ │
│  └──────────────────────────┘  └──────────────────────────┘ │
│                                                              │
│  Pod A → Pod B:  via localhost (same node) ✅                │
│  Pod A → Pod C:  via Pod IP directly (cross-node) ✅         │
│  Pod A → Pod D:  via Pod IP directly (cross-node) ✅         │
│  No NAT required for any communication path                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Service Types

**Services** provide a stable endpoint (IP + DNS) to access a group of pods. Pods are ephemeral — their IPs change. Services solve this.

```
┌──────────────────────────────────────────────────────────────┐
│                   THE PROBLEM SERVICES SOLVE                  │
│                                                              │
│  WITHOUT Services:              WITH Services:               │
│                                                              │
│  Frontend pod needs            Frontend connects to          │
│  to track all backend          a single Service IP           │
│  pod IPs manually              that load-balances            │
│                                                              │
│  Frontend                      Frontend                      │
│     │                             │                          │
│     ├─▶ 10.244.1.5 (dead!)       │                          │
│     ├─▶ 10.244.2.8               ▼                          │
│     └─▶ 10.244.3.3          ┌─────────────┐                 │
│                              │  Service    │                 │
│  ❌ Must track IPs           │ 10.96.0.100 │                 │
│  ❌ No load balancing        │ backend-svc │                 │
│  ❌ Dead pods not removed    └──────┬──────┘                 │
│                                ┌────┼────┐                   │
│                                ▼    ▼    ▼                   │
│                              Pod1  Pod2  Pod3                │
│                              ✅ Stable IP                     │
│                              ✅ Load balanced                 │
│                              ✅ Auto-updated                  │
└──────────────────────────────────────────────────────────────┘
```

### ClusterIP (Default)

Exposes the Service on a **cluster-internal IP**. Only reachable from within the cluster.

```
┌──────────────────────────────────────────────────────────────┐
│                      ClusterIP Service                        │
│                                                              │
│   ┌──────────────────────────────────────────────────┐       │
│   │              Cluster Network                     │       │
│   │                                                  │       │
│   │   Frontend Pod ──▶ backend-svc:80                │       │
│   │                    (ClusterIP: 10.96.0.100)      │       │
│   │                         │                        │       │
│   │                    ┌────┼────┐                    │       │
│   │                    ▼    ▼    ▼                    │       │
│   │                  Pod1  Pod2  Pod3                 │       │
│   │                  :8080 :8080 :8080                │       │
│   │                                                  │       │
│   └──────────────────────────────────────────────────┘       │
│                                                              │
│   ❌ NOT accessible from outside the cluster                  │
│   ✅ Best for internal service-to-service communication       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP             # Default, can be omitted
  selector:
    app: backend              # Matches pod labels
  ports:
  - port: 80                  # Service port
    targetPort: 8080          # Container port
    protocol: TCP
```

### NodePort

Exposes the Service on a **static port on every node's IP**. Accessible from outside the cluster.

```
┌──────────────────────────────────────────────────────────────┐
│                      NodePort Service                         │
│                                                              │
│   External Client                                            │
│        │                                                     │
│        ▼                                                     │
│   <NodeIP>:30080   (any node in the cluster)                 │
│        │                                                     │
│   ┌────▼──────────────────────────────────────────────┐      │
│   │              Cluster                              │      │
│   │                                                   │      │
│   │  ┌─── Node 1 ─────┐  ┌─── Node 2 ─────┐         │      │
│   │  │ :30080 (open)   │  │ :30080 (open)   │         │      │
│   │  │                 │  │                 │         │      │
│   │  │  kube-proxy     │  │  kube-proxy     │         │      │
│   │  │  routes to      │  │  routes to      │         │      │
│   │  │  Service        │  │  Service        │         │      │
│   │  └─────────────────┘  └─────────────────┘         │      │
│   │           │                    │                   │      │
│   │           └────────┬───────────┘                   │      │
│   │                    ▼                               │      │
│   │            Service (ClusterIP)                     │      │
│   │            10.96.0.100:80                          │      │
│   │                    │                               │      │
│   │              ┌─────┼─────┐                         │      │
│   │              ▼     ▼     ▼                         │      │
│   │            Pod1  Pod2  Pod3                        │      │
│   │                                                   │      │
│   └───────────────────────────────────────────────────┘      │
│                                                              │
│   Port range: 30000-32767                                    │
│   ✅ External access without cloud load balancer              │
│   ❌ Non-standard ports, requires node IP knowledge           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80                  # ClusterIP port
    targetPort: 8080          # Container port
    nodePort: 30080           # External port (30000-32767)
    protocol: TCP
```

### LoadBalancer

Exposes the Service using a **cloud provider's load balancer**. The most common way to expose services externally in cloud environments.

```
┌──────────────────────────────────────────────────────────────┐
│                    LoadBalancer Service                        │
│                                                              │
│   Internet                                                   │
│      │                                                       │
│      ▼                                                       │
│   ┌────────────────────────────┐                             │
│   │  Cloud Load Balancer       │                             │
│   │  (AWS ELB / GCP LB / etc) │                             │
│   │  External IP: 52.1.2.3    │                             │
│   │  Port: 80                 │                             │
│   └───────────┬────────────────┘                             │
│               │                                              │
│   ┌───────────▼────────────────────────────────────┐         │
│   │              Cluster                           │         │
│   │                                                │         │
│   │  ┌── Node 1 ──┐  ┌── Node 2 ──┐               │         │
│   │  │  :30080     │  │  :30080     │               │         │
│   │  └──────┬──────┘  └──────┬──────┘               │         │
│   │         └────────┬───────┘                      │         │
│   │                  ▼                              │         │
│   │          Service (ClusterIP)                    │         │
│   │          10.96.0.100:80                         │         │
│   │                  │                              │         │
│   │            ┌─────┼─────┐                        │         │
│   │            ▼     ▼     ▼                        │         │
│   │          Pod1  Pod2  Pod3                       │         │
│   └────────────────────────────────────────────────┘         │
│                                                              │
│   Creates: External LB + NodePort + ClusterIP                │
│   ✅ Standard ports (80, 443)                                 │
│   ✅ Cloud-integrated health checks                           │
│   ❌ One LB per service (can be expensive)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-lb
  annotations:
    # Cloud-specific annotations
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

### ExternalName

Maps a Service to a **DNS name** (CNAME record). No proxying, no ClusterIP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: db.example.com    # Resolves to this DNS name
```

```
# When a pod resolves "external-db.default.svc.cluster.local"
# it gets back CNAME → db.example.com
# Useful for pointing to external databases, APIs, etc.
```

### Service Types Comparison

| Type | Scope | Use Case | Creates |
|------|-------|----------|---------|
| **ClusterIP** | Internal only | Service-to-service | Virtual IP |
| **NodePort** | External via node IP | Dev/testing, bare metal | ClusterIP + NodePort |
| **LoadBalancer** | External via cloud LB | Production, cloud | ClusterIP + NodePort + LB |
| **ExternalName** | DNS alias | External services | CNAME record |

---

## Service Discovery

Kubernetes provides two mechanisms for service discovery:

### 1. Environment Variables

When a pod starts, Kubernetes injects env vars for each active Service:

```bash
# Inside a pod, environment variables are auto-injected:
BACKEND_SVC_SERVICE_HOST=10.96.0.100
BACKEND_SVC_SERVICE_PORT=80
BACKEND_SVC_PORT=tcp://10.96.0.100:80

# Limitation: Service must exist BEFORE the pod is created
```

### 2. DNS (Preferred)

```
┌────────────────────────────────────────────────────────────┐
│               DNS-BASED SERVICE DISCOVERY                   │
│                                                            │
│   Full DNS format:                                         │
│   <service>.<namespace>.svc.cluster.local                  │
│                                                            │
│   Examples:                                                │
│   ┌──────────────────────────────────────────────────┐     │
│   │ Same namespace:     backend-svc                  │     │
│   │ Cross-namespace:    backend-svc.production       │     │
│   │ Full:               backend-svc.production.svc.  │     │
│   │                     cluster.local                │     │
│   └──────────────────────────────────────────────────┘     │
│                                                            │
│   DNS records created:                                     │
│                                                            │
│   ClusterIP Service:                                       │
│   backend-svc.default.svc → A record → 10.96.0.100        │
│                                                            │
│   Headless Service (clusterIP: None):                      │
│   backend-svc.default.svc → A records → Pod IPs            │
│   pod-0.backend-svc.default.svc → A record → Pod 0 IP     │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## DNS in Kubernetes

CoreDNS is the default DNS server in Kubernetes (since v1.13).

```
┌────────────────────────────────────────────────────────────┐
│                    CoreDNS Architecture                     │
│                                                            │
│  Pod ──DNS query──▶ ┌──────────────┐                       │
│  (resolv.conf       │   CoreDNS    │                       │
│   nameserver:       │   Pod(s)     │                       │
│   10.96.0.10)       │              │                       │
│                     │  Corefile:   │                       │
│                     │  ┌────────┐  │                       │
│                     │  │k8s     │──┼──▶ Kubernetes API     │
│                     │  │plugin  │  │   (watch services)    │
│                     │  ├────────┤  │                       │
│                     │  │forward │──┼──▶ Upstream DNS       │
│                     │  │plugin  │  │   (for external)      │
│                     │  ├────────┤  │                       │
│                     │  │cache   │  │                       │
│                     │  │plugin  │  │                       │
│                     │  └────────┘  │                       │
│                     └──────────────┘                       │
│                                                            │
│  DNS Resolution Order:                                     │
│  1. <name>.<namespace>.svc.cluster.local  → Service IP     │
│  2. <name>.<namespace>.svc                → Service IP     │
│  3. <name>                                → if same NS     │
│  4. Forward to upstream DNS               → external       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS ConfigMap
kubectl get configmap coredns -n kube-system -o yaml

# Test DNS from inside a pod
kubectl run dnstest --image=busybox:1.36 --rm -it -- nslookup kubernetes.default.svc.cluster.local
kubectl run dnstest --image=busybox:1.36 --rm -it -- nslookup backend-svc.default.svc.cluster.local

# Check pod's DNS config
kubectl exec my-pod -- cat /etc/resolv.conf
# nameserver 10.96.0.10
# search default.svc.cluster.local svc.cluster.local cluster.local
# ndots:5
```

---

## Ingress

An **Ingress** manages external HTTP/HTTPS access to services in a cluster. It provides routing rules, SSL termination, and virtual hosting — avoiding the need for one LoadBalancer per service.

```
┌────────────────────────────────────────────────────────────────┐
│              WITHOUT INGRESS                                    │
│                                                                │
│  ┌────────┐  ┌────────┐  ┌────────┐                           │
│  │  LB 1  │  │  LB 2  │  │  LB 3  │   3 LoadBalancers = $$$  │
│  │  $$/mo │  │  $$/mo │  │  $$/mo │                           │
│  └───┬────┘  └───┬────┘  └───┬────┘                           │
│      ▼           ▼           ▼                                │
│  web-svc     api-svc     admin-svc                            │
│                                                                │
│              WITH INGRESS                                      │
│                                                                │
│  ┌────────────────────────┐                                    │
│  │  Single Load Balancer  │   1 LB = $ (much cheaper)         │
│  │  (cloud LB)            │                                    │
│  └────────┬───────────────┘                                    │
│           ▼                                                    │
│  ┌────────────────────────┐                                    │
│  │  Ingress Controller    │                                    │
│  │  (nginx/traefik/etc)   │                                    │
│  │                        │                                    │
│  │  Rules:                │                                    │
│  │  /      → web-svc      │                                    │
│  │  /api   → api-svc      │                                    │
│  │  /admin → admin-svc    │                                    │
│  │                        │                                    │
│  └───┬────────┬───────┬───┘                                    │
│      ▼        ▼       ▼                                        │
│  web-svc  api-svc  admin-svc                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Ingress Controller (Must Install Separately!)

The Ingress resource does nothing by itself — you need an **Ingress Controller** running in the cluster.

| Controller | Provider | Features |
|-----------|----------|----------|
| **NGINX Ingress** | Community/F5 | Most popular, feature-rich |
| **Traefik** | Traefik Labs | Auto-discovery, middlewares |
| **HAProxy** | HAProxy | High performance |
| **AWS ALB** | AWS | Native AWS ALB integration |
| **GCE** | Google | Default in GKE |
| **Istio Gateway** | Istio | Service mesh integration |

```bash
# Install NGINX Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/cloud/deploy.yaml

# Verify
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

### Ingress Resource — Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx          # Which controller to use
  rules:
  - host: myapp.example.com       # Virtual host
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 8080
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-svc
            port:
              number: 3000
```

### Ingress with TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret         # TLS certificate secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
---
# Create TLS secret
# kubectl create secret tls tls-secret \
#   --cert=tls.crt --key=tls.key
```

### Host-Based Routing (Multiple Domains)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-svc
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 8080
```

```
┌──────────────────────────────────────────────────────┐
│           HOST-BASED ROUTING                         │
│                                                      │
│  app.example.com ──▶ Ingress ──▶ app-svc             │
│  api.example.com ──▶ Controller ──▶ api-svc          │
│  blog.example.com ──▶          ──▶ blog-svc          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Network Policies

**Network Policies** are firewall rules for pod-to-pod traffic. By default, all pods can communicate with all other pods. Network Policies restrict this.

```
┌──────────────────────────────────────────────────────────────┐
│                   NETWORK POLICIES                            │
│                                                              │
│  DEFAULT (no policies):     WITH Network Policy:             │
│  ┌────────────────────┐     ┌────────────────────┐           │
│  │ All pods can talk  │     │ Only allowed traffic│           │
│  │ to all pods        │     │ is permitted        │           │
│  │                    │     │                     │           │
│  │  Pod A ◄──▶ Pod B  │     │  Pod A ──✅──▶ Pod B │           │
│  │  Pod A ◄──▶ Pod C  │     │  Pod A ──❌──▶ Pod C │           │
│  │  Pod B ◄──▶ Pod C  │     │  Pod C ──❌──▶ Pod B │           │
│  │                    │     │                     │           │
│  └────────────────────┘     └────────────────────┘           │
│                                                              │
│  ⚠ Requires a CNI that supports Network Policies:            │
│    ✅ Calico    ✅ Cilium    ✅ Weave Net                       │
│    ❌ Flannel (no support)                                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Network Policy Examples

#### Deny All Ingress (Default Deny)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}         # Applies to ALL pods in namespace
  policyTypes:
  - Ingress               # Controls incoming traffic
  # No ingress rules = deny all incoming
```

#### Allow Only Specific Pods

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend            # Apply to backend pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend       # Only allow from frontend pods
    ports:
    - protocol: TCP
      port: 8080
```

```
┌──────────────────────────────────────────────────────┐
│                Network Policy Effect                  │
│                                                      │
│  frontend pods ──TCP:8080──✅──▶ backend pods         │
│  other pods    ──TCP:8080──❌──▶ backend pods         │
│  frontend pods ──TCP:9090──❌──▶ backend pods         │
│                                                      │
└──────────────────────────────────────────────────────┘
```

#### Allow from Specific Namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: monitoring    # Allow from monitoring namespace
    ports:
    - protocol: TCP
      port: 9090
```

#### Egress Policy (Control Outgoing Traffic)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:                       # Allow DNS
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

---

## CNI Plugins Overview

**CNI (Container Network Interface)** plugins provide pod networking. The CNI is responsible for:
- Assigning IP addresses to pods
- Enabling pod-to-pod communication across nodes
- Implementing network policies (optional)

```
┌────────────────────────────────────────────────────────────────┐
│                     CNI PLUGIN COMPARISON                       │
│                                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Flannel  │  │  Calico  │  │  Cilium  │  │ Weave Net│      │
│  │          │  │          │  │          │  │          │      │
│  │ Overlay  │  │ BGP/VXLAN│  │ eBPF     │  │ Overlay  │      │
│  │ (VXLAN)  │  │          │  │          │  │ (mesh)   │      │
│  │          │  │          │  │          │  │          │      │
│  │ Simple   │  │ Net      │  │ Advanced │  │ Simple   │      │
│  │ No Net   │  │ Policies │  │ Net Pol  │  │ Net Pol  │      │
│  │ Policy   │  │ Flexible │  │ L7 aware │  │ Encrypt  │      │
│  │          │  │          │  │ Observ.  │  │          │      │
│  │ Learning │  │ Prod.    │  │ Modern   │  │ Small    │      │
│  │          │  │ standard │  │ Prod.    │  │ clusters │      │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

| CNI Plugin | Networking Mode | Network Policy | Performance | Best For |
|-----------|----------------|----------------|-------------|----------|
| **Flannel** | VXLAN overlay | ❌ No | Good | Learning, simple setups |
| **Calico** | BGP / VXLAN / IPIP | ✅ Yes | Excellent | Production, most popular |
| **Cilium** | eBPF | ✅ Yes (L3-L7) | Best | Modern infra, observability |
| **Weave Net** | Mesh overlay | ✅ Yes | Good | Small clusters, encryption |
| **AWS VPC CNI** | Native VPC | ✅ Yes | Excellent | EKS |
| **Azure CNI** | Native VNET | ✅ Yes | Excellent | AKS |

---

## Real-World Scenario: Complete Networking Setup

```yaml
# 1. Backend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: my-api:2.0
        ports:
        - containerPort: 8080
---
# 2. ClusterIP Service (internal)
apiVersion: v1
kind: Service
metadata:
  name: api-svc
  namespace: production
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8080
---
# 3. Ingress (external access with TLS)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.mycompany.com
    secretName: api-tls
  rules:
  - host: api.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 80
---
# 4. Network Policy (only frontend can access API)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx     # Allow from ingress controller
    ports:
    - protocol: TCP
      port: 8080
```

```
┌──────────────────────────────────────────────────────────────┐
│                  COMPLETE NETWORKING FLOW                      │
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  Cloud LB (api.mycompany.com:443)                            │
│     │                                                        │
│     ▼                                                        │
│  Ingress Controller (NGINX) ← TLS termination                │
│     │                                                        │
│     ▼                                                        │
│  api-svc (ClusterIP: 10.96.0.50:80)                          │
│     │                                                        │
│     ├────┬────┐                                              │
│     ▼    ▼    ▼                                              │
│  Pod-1 Pod-2 Pod-3  (app: api, port: 8080)                   │
│     │                                                        │
│  Network Policy: Only frontend + ingress-nginx allowed       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Investigation | Solution |
|-------|---------------|----------|
| Service not reachable | `kubectl get endpoints <svc>` — empty? | Fix selector to match pod labels |
| DNS not resolving | `nslookup` from test pod | Check CoreDNS pods, ConfigMap |
| Ingress 404 | Check path rules, backend service | Verify service name/port, pathType |
| Ingress 502 | Backend pods not ready | Check pod readiness probes, pod logs |
| Network Policy blocks traffic | Policy rules too restrictive | Add necessary `from`/`to` rules, DNS egress |
| NodePort not accessible | Firewall/security group | Open port 30000-32767 on nodes |
| LoadBalancer pending | No cloud provider / no LB support | Use NodePort or MetalLB for bare metal |

```bash
# Debugging commands
kubectl get svc                              # List services
kubectl get endpoints <svc-name>             # Check if pods are listed
kubectl describe svc <svc-name>              # Service details
kubectl get ingress                          # List ingress rules
kubectl describe ingress <name>              # Ingress details + events

# Test connectivity
kubectl run test --image=busybox:1.36 --rm -it -- wget -qO- http://backend-svc:80
kubectl run test --image=nicolaka/netshoot --rm -it -- bash  # Full network tools
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Service** | Stable endpoint (IP + DNS) for accessing a group of pods |
| **ClusterIP** | Internal-only; default type; for service-to-service |
| **NodePort** | Exposes on every node IP:port (30000-32767); for dev/bare metal |
| **LoadBalancer** | Provisions cloud LB; creates NodePort + ClusterIP; for production |
| **ExternalName** | DNS CNAME alias to external service |
| **DNS** | CoreDNS provides `<svc>.<ns>.svc.cluster.local` resolution |
| **Ingress** | HTTP/HTTPS routing rules; needs an Ingress Controller |
| **Ingress Controller** | NGINX, Traefik, etc.; must be installed separately |
| **Network Policy** | Firewall rules for pod traffic; requires supporting CNI |
| **CNI Plugin** | Provides pod networking (Calico, Cilium, Flannel) |

---

## Quick Revision Questions

1. **What is the difference between ClusterIP, NodePort, and LoadBalancer services?**
   <details><summary>Answer</summary>ClusterIP: Internal-only virtual IP, accessible only within the cluster. NodePort: Extends ClusterIP by opening a static port (30000-32767) on every node, accessible externally via NodeIP:NodePort. LoadBalancer: Extends NodePort by provisioning a cloud load balancer with an external IP, routing traffic to the NodePorts.</details>

2. **Why do you need an Ingress Controller? Why doesn't Ingress work by itself?**
   <details><summary>Answer</summary>An Ingress resource is just a set of routing rules (data). The Ingress Controller is the actual software (NGINX, Traefik) that reads those rules and configures itself to route traffic accordingly. Without a controller, the Ingress resource has no effect — it's like writing a recipe without a chef.</details>

3. **What happens if you create a Network Policy that selects pods but has no `ingress` rules?**
   <details><summary>Answer</summary>It becomes a "deny all ingress" policy for the selected pods. In Kubernetes, creating a NetworkPolicy with `policyTypes: [Ingress]` but no `ingress` rules means all incoming traffic to those pods is blocked. Egress is still unrestricted unless you also add an Egress policy.</details>

4. **How does a pod in namespace `dev` reach a service called `api-svc` in namespace `production`?**
   <details><summary>Answer</summary>Using the cross-namespace DNS name: `api-svc.production` or the fully qualified `api-svc.production.svc.cluster.local`. The short name `api-svc` alone would only resolve within the same namespace.</details>

5. **What is a Headless Service and when is it used?**
   <details><summary>Answer</summary>A Headless Service has `clusterIP: None`. Instead of providing a single virtual IP, it returns the IP addresses of all backing pods directly via DNS A records. It's used with StatefulSets to provide stable DNS names for individual pods (e.g., `postgres-0.postgres-svc`).</details>

6. **Why must you allow DNS (port 53) egress when creating egress Network Policies?**
   <details><summary>Answer</summary>When you create an egress NetworkPolicy, all outgoing traffic not explicitly allowed is blocked — including DNS queries. Without DNS, pods can't resolve service names. You must explicitly allow egress to port 53 (UDP and TCP) so pods can still perform DNS lookups via CoreDNS.</details>

---

[← Previous: Workload Resources](../04-Workload-Resources/01-workload-resources.md) | [Back to README](../README.md) | [Next: Storage →](../06-Storage/01-storage.md)
