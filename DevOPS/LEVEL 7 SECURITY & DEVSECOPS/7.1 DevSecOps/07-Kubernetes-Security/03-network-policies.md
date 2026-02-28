# Chapter 7.3: Network Policies

## Overview

By default, all pods in Kubernetes can communicate with every other pod — there are no network restrictions. Network Policies act as firewall rules for pod-to-pod traffic, implementing zero-trust networking at the cluster level. This chapter covers NetworkPolicy design, implementation, and common patterns.

---

## Default Kubernetes Networking

```
DEFAULT: ALL PODS CAN TALK TO ALL PODS (Flat Network)

  Namespace: frontend           Namespace: backend
  ┌───────────────────┐         ┌───────────────────┐
  │  ┌──────┐         │   ✓     │         ┌──────┐  │
  │  │Web UI│ ◀───────┼─────────┼─────────│ API  │  │
  │  └──────┘         │         │         └──────┘  │
  │         │         │         │         ▲         │
  │         │    ✓    │         │    ✓    │         │
  │         ▼         │         │         │         │
  │  ┌──────┐         │   ✓     │  ┌──────┐         │
  │  │Admin │ ◀───────┼─────────┼──│  DB  │         │
  │  └──────┘         │         │  └──────┘         │
  └───────────────────┘         └───────────────────┘

  PROBLEM: If Web UI is compromised, attacker can reach DB directly!

  WITH NETWORK POLICIES: Zero-Trust

  ┌───────────────────┐         ┌───────────────────┐
  │  ┌──────┐         │   ✓     │         ┌──────┐  │
  │  │Web UI│ ────────┼─────────┼────────▶│ API  │  │
  │  └──────┘         │         │         └──────┘  │
  │         │    ✗    │         │         │  ✓      │
  │         ▼         │         │         ▼         │
  │  ┌──────┐    ✗    │         │  ┌──────┐         │
  │  │Admin │ ────────┼────✗────┼──│  DB  │         │
  │  └──────┘         │         │  └──────┘         │
  └───────────────────┘         └───────────────────┘
  
  Only: Web UI → API → DB  (explicit allow rules)
```

---

## NetworkPolicy Basics

```yaml
# Default deny all ingress traffic in a namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}        # Apply to ALL pods in namespace
  policyTypes:
    - Ingress            # Deny all incoming traffic
  # No ingress rules = deny all

---
# Default deny all egress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Egress
  # No egress rules = deny all

---
# Default deny ALL traffic (ingress + egress)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

---

## Allow Specific Traffic

```yaml
# Allow frontend to reach API on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api                    # Apply to API pods
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend       # Only from frontend pods
      ports:
        - protocol: TCP
          port: 8080              # Only on port 8080

---
# Allow API to reach database on port 5432
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432
```

---

## Cross-Namespace Policies

```yaml
# Allow monitoring namespace to scrape metrics from production
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring-scrape
  namespace: production
spec:
  podSelector: {}                  # All pods in production
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              purpose: monitoring  # Namespace labeled 'purpose=monitoring'
          podSelector:
            matchLabels:
              app: prometheus      # Only Prometheus pods
      ports:
        - protocol: TCP
          port: 9090              # Metrics port
```

```bash
# Label the monitoring namespace
kubectl label namespace monitoring purpose=monitoring
```

---

## Egress Policies

```yaml
# Allow pods to reach DNS (required for service discovery!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to: []                      # Any destination
      ports:
        - protocol: UDP
          port: 53                # DNS
        - protocol: TCP
          port: 53                # DNS over TCP

---
# Allow API to reach external payment service
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-external
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Egress
  egress:
    # Allow DNS
    - ports:
        - protocol: UDP
          port: 53
    # Allow HTTPS to payment provider
    - to:
        - ipBlock:
            cidr: 203.0.113.0/24   # Payment provider IP range
      ports:
        - protocol: TCP
          port: 443
    # Allow internal database
    - to:
        - podSelector:
            matchLabels:
              app: database
      ports:
        - protocol: TCP
          port: 5432
```

---

## Complete Microservice Network Policy

```yaml
# Three-tier application: Frontend → API → Database
# With DNS, monitoring, and external access

# 1. Default deny in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: production
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]

---
# 2. Frontend: allow ingress from ingress controller, egress to API + DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: frontend
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              app.kubernetes.io/name: ingress-nginx
      ports:
        - protocol: TCP
          port: 80
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: api
      ports:
        - protocol: TCP
          port: 8080
    - ports:                  # DNS
        - protocol: UDP
          port: 53

---
# 3. API: allow ingress from frontend, egress to DB + external + DNS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: api
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: frontend
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: database
      ports:
        - protocol: TCP
          port: 5432
    - to:                      # External APIs
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8    # Block internal ranges
              - 172.16.0.0/12
              - 192.168.0.0/16
      ports:
        - protocol: TCP
          port: 443
    - ports:                   # DNS
        - protocol: UDP
          port: 53

---
# 4. Database: allow ingress from API only, no egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: api
      ports:
        - protocol: TCP
          port: 5432
  egress:
    - ports:                   # DNS only
        - protocol: UDP
          port: 53
```

---

## CNI Plugin Requirements

```
NETWORK POLICY CNI SUPPORT:

  ┌──────────────┬─────────────────────────────────────┐
  │ CNI Plugin   │ NetworkPolicy Support               │
  ├──────────────┼─────────────────────────────────────┤
  │ Calico       │ ✓ Full (+ extended Calico policies) │
  │ Cilium       │ ✓ Full (+ L7 policies with eBPF)   │
  │ Weave Net    │ ✓ Ingress + Egress                  │
  │ Canal        │ ✓ Full (Flannel + Calico policies)  │
  │ Antrea       │ ✓ Full                               │
  │ Flannel      │ ✗ NO NetworkPolicy support!         │
  │ AWS VPC CNI  │ ✗ Needs Calico addon for policies   │
  └──────────────┴─────────────────────────────────────┘

  IMPORTANT: If your CNI doesn't support NetworkPolicy,
  the policy resources are silently IGNORED!

  Check: kubectl get networkpolicies -A
  If you see policies but traffic isn't blocked → wrong CNI
```

---

## Testing Network Policies

```bash
# Deploy test pods
kubectl run test-frontend --image=busybox --labels="tier=frontend" \
    -n production -- sleep 3600
kubectl run test-api --image=busybox --labels="tier=api" \
    -n production -- sleep 3600
kubectl run test-attacker --image=busybox --labels="tier=attacker" \
    -n production -- sleep 3600

# Test: frontend → API (should work)
kubectl exec test-frontend -n production -- wget -qO- --timeout=3 \
    http://api-service:8080/health
# Expected: 200 OK

# Test: attacker → API (should be blocked)
kubectl exec test-attacker -n production -- wget -qO- --timeout=3 \
    http://api-service:8080/health
# Expected: timeout / connection refused

# Test: attacker → database (should be blocked)
kubectl exec test-attacker -n production -- nc -zv db-service 5432 -w 3
# Expected: connection timed out

# Test: database → external (should be blocked)
kubectl exec test-db -n production -- wget -qO- --timeout=3 \
    https://example.com
# Expected: timeout
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| NetworkPolicy has no effect | CNI doesn't support policies (e.g., Flannel) | Switch to Calico or Cilium; or add Calico alongside current CNI |
| Pods can't resolve DNS after deny-all | Egress policy blocks DNS (UDP 53) | Always include DNS egress rule in every egress policy |
| Cross-namespace traffic still blocked | Namespace not labeled for namespaceSelector | Label target namespace: `kubectl label ns monitoring purpose=monitoring` |
| Ingress controller can't reach pods | Missing ingress rule for ingress NS | Add namespaceSelector for ingress-nginx namespace |
| Policy looks correct but doesn't work | Label mismatch between policy and pods | Verify pod labels match podSelector with `kubectl get pods --show-labels` |

---

## Summary Table

| Concept | Purpose |
|---------|---------|
| **Default deny** | Block all traffic; only allow explicitly permitted flows |
| **podSelector** | Target pods by label for applying the policy |
| **namespaceSelector** | Allow/deny traffic from specific namespaces |
| **ipBlock** | Allow/deny traffic from external IP ranges |
| **Egress DNS rule** | Essential — always allow UDP/TCP 53 for service discovery |
| **CNI requirement** | NetworkPolicy only works with supporting CNIs (Calico, Cilium) |

---

## Quick Revision Questions

1. **What happens if you apply no NetworkPolicies in a namespace?**
   > All pods can communicate with all other pods in the cluster, regardless of namespace. Kubernetes uses flat networking by default — there are no network restrictions until you create NetworkPolicy resources.

2. **Why is a default-deny policy the recommended starting point?**
   > Default-deny blocks all traffic, then you explicitly allow only necessary communication paths. This follows zero-trust principles — no traffic is trusted by default. Without it, any compromised pod can reach any other pod.

3. **Why must you always include a DNS egress rule?**
   > Kubernetes services use DNS for discovery (e.g., `api-service.production.svc.cluster.local`). If egress is denied without allowing UDP/TCP port 53, pods can't resolve service names and all internal communication breaks.

4. **What happens if your CNI doesn't support NetworkPolicy?**
   > The NetworkPolicy resources are created in etcd but completely ignored — no traffic is actually blocked. This creates a false sense of security. Always verify your CNI supports NetworkPolicy (Calico, Cilium, Weave — NOT plain Flannel).

5. **How do you allow cross-namespace communication with NetworkPolicy?**
   > Use `namespaceSelector` to match the source namespace by label. The target namespace must be labeled (e.g., `kubectl label ns monitoring purpose=monitoring`). Combine with `podSelector` to restrict to specific pods within that namespace.

---

[← Previous: 7.2 RBAC Implementation](02-rbac-implementation.md) | [Next: 7.4 Pod Security Standards →](04-pod-security-standards.md)

[Back to Table of Contents](../README.md)
