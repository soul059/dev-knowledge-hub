# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 6: Network Segmentation

## Overview

Network segmentation is a **critical infrastructure defense** against SSRF and lateral movement. By dividing the network into isolated segments with controlled access between them, even if an attacker achieves SSRF on a web application, they cannot reach sensitive internal services. This topic covers segmentation strategies, implementation across environments, and how segmentation complements application-level SSRF defenses.

---

## 1. Why Segmentation Matters for SSRF

```
WITHOUT SEGMENTATION:
┌───────────────────────────────────────────────┐
│                FLAT NETWORK                    │
│                                                │
│  [Web App] ←→ [Database] ←→ [Redis]          │
│      ↕            ↕            ↕               │
│  [Admin Panel] ←→ [Docker] ←→ [Metadata]     │
│      ↕            ↕            ↕               │
│  [Jenkins] ←→ [Kubernetes] ←→ [Secrets]       │
│                                                │
│  SSRF from Web App → Can reach EVERYTHING!    │
└───────────────────────────────────────────────┘

WITH SEGMENTATION:
┌─────────────┐  Firewall  ┌─────────────┐  Firewall  ┌─────────────┐
│ DMZ Segment │  ────────→ │ App Segment │  ────────→ │ Data Segment│
│             │  Port 443  │             │  Port 5432 │             │
│  [Web App]  │  only      │ [API Server]│  only      │ [Database]  │
│  [WAF]      │            │ [Workers]   │            │ [Redis]     │
│             │            │             │            │ [Secrets]   │
└─────────────┘            └─────────────┘            └─────────────┘
                                │
                           SSRF from Web App:
                           ✓ Can reach API (port 443)
                           ✗ Cannot reach Database
                           ✗ Cannot reach Redis
                           ✗ Cannot reach Metadata
```

---

## 2. Segmentation Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **VLAN-based** | Layer 2 separation with tagged VLANs | Traditional data centers |
| **Subnet-based** | Different IP subnets with routing controls | Cloud VPCs |
| **Firewall rules** | Layer 3/4 filtering between segments | All environments |
| **Security Groups** | Cloud-native instance-level filtering | AWS, Azure, GCP |
| **Network Policies** | Container-level network controls | Kubernetes |
| **Service Mesh** | mTLS and identity-based access | Microservices |
| **Zero Trust** | Verify every request regardless of network | Modern architecture |

---

## 3. Cloud Implementation

### AWS VPC Segmentation

```
VPC Architecture:
┌─────────────────────────────────────────────────┐
│ VPC: 10.0.0.0/16                                │
│                                                  │
│ ┌──────────────────┐  ┌──────────────────┐      │
│ │ Public Subnet     │  │ Public Subnet     │     │
│ │ 10.0.1.0/24      │  │ 10.0.2.0/24      │     │
│ │ [ALB]             │  │ [NAT Gateway]     │     │
│ └──────────────────┘  └──────────────────┘      │
│                                                  │
│ ┌──────────────────┐  ┌──────────────────┐      │
│ │ Private Subnet    │  │ Private Subnet    │     │
│ │ 10.0.10.0/24     │  │ 10.0.11.0/24     │     │
│ │ [App Servers]     │  │ [App Servers]     │     │
│ └──────────────────┘  └──────────────────┘      │
│                                                  │
│ ┌──────────────────┐  ┌──────────────────┐      │
│ │ Data Subnet       │  │ Data Subnet       │     │
│ │ 10.0.20.0/24     │  │ 10.0.21.0/24     │     │
│ │ [RDS] [Redis]     │  │ [RDS Replica]     │     │
│ └──────────────────┘  └──────────────────┘      │
│                                                  │
│ Security Groups:                                 │
│ ├── sg-alb: Inbound 443 from 0.0.0.0/0         │
│ ├── sg-app: Inbound 8080 from sg-alb only       │
│ ├── sg-data: Inbound 5432 from sg-app only      │
│ └── sg-app egress: DENY to 169.254.169.254      │
└─────────────────────────────────────────────────┘
```

### Kubernetes Network Policies

```yaml
# Deny all ingress and egress by default
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

---
# Allow web app to reach only the API service
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-api
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web-frontend
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: api-server
      ports:
        - protocol: TCP
          port: 8080
    # Allow DNS
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53

---
# Block metadata access from all pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-metadata
  namespace: production
spec:
  podSelector: {}
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 169.254.169.254/32
```

---

## 4. Zero Trust Architecture

```
ZERO TRUST PRINCIPLES:
1. Never trust, always verify
2. Assume breach — limit blast radius
3. Verify explicitly — identity, device, location
4. Least privilege access
5. Continuous monitoring and validation

TRADITIONAL:               ZERO TRUST:
"Inside = Trusted"          "Nothing = Trusted"
Network perimeter           Identity perimeter
IP-based access             Identity-based access
VPN = full access           Per-resource access

IMPLEMENTATION:
├── mTLS between all services
│   (Each service has its own certificate)
│
├── Service Mesh (Istio/Linkerd)
│   ├── Automatic mTLS
│   ├── Authorization policies
│   ├── Traffic encryption
│   └── Observability
│
├── Identity-Aware Proxy
│   ├── Google BeyondCorp / Cloud IAP
│   ├── Azure AD Application Proxy
│   └── Cloudflare Access
│
└── Microsegmentation
    └── Per-workload firewall rules
```

---

## 5. Segmentation Validation

```bash
# Verify segmentation is working

# From web app server, test if internal services are reachable
# These should ALL fail if segmentation is correct:
curl -m 5 http://169.254.169.254/                    # Metadata
curl -m 5 http://10.0.20.10:6379/                     # Redis
curl -m 5 http://10.0.20.11:5432/                     # PostgreSQL
curl -m 5 http://10.0.30.10:8080/                     # Admin panel

# Automated segmentation testing
nmap -sT -p 22,80,443,3306,5432,6379,8080,9200 10.0.20.0/24

# Continuous validation
# Use tools like:
# - Illumio (microsegmentation platform)
# - Guardicore (now Akamai)
# - Network policy testing in CI/CD
```

---

## 6. Segmentation + SSRF Defense Summary

```
LAYERED DEFENSE AGAINST SSRF:

Layer 1: APPLICATION
  ├── URL validation and allowlisting
  ├── DNS pinning
  └── Disable redirects

Layer 2: NETWORK
  ├── Egress filtering from app servers
  ├── Block metadata endpoint
  └── Segment app from data tier

Layer 3: CLOUD
  ├── IMDSv2 enforcement
  ├── Security groups (least privilege)
  └── VPC endpoint policies

Layer 4: SERVICE
  ├── mTLS between services
  ├── Service-level authentication
  └── No reliance on network trust

Layer 5: MONITORING
  ├── Log all outbound connections
  ├── Alert on internal IP access attempts
  └── Regular segmentation audits

RESULT: Even if SSRF exists, impact is minimized
```

---

## Summary Table

| Segmentation Type | Scope | Tools | SSRF Impact |
|-------------------|-------|-------|-------------|
| VPC/Subnet | Network-wide | AWS VPC, Azure VNet | Block internal access |
| Security Groups | Instance-level | Cloud provider SGs | Block per-instance |
| Network Policies | Container-level | Calico, Cilium | Block per-pod |
| Service Mesh | Service-level | Istio, Linkerd | Identity-based access |
| Zero Trust | Organization-wide | BeyondCorp, ZTNA | Eliminate trust assumptions |

---

## Revision Questions

1. Draw a segmented network architecture that prevents SSRF from reaching databases and internal services.
2. Write AWS Security Group rules that implement proper segmentation for a three-tier application.
3. Create Kubernetes NetworkPolicies that block metadata access and limit inter-pod communication.
4. What is Zero Trust Architecture and how does it relate to SSRF prevention?
5. How do you validate that network segmentation is correctly implemented and maintained?
6. Design a complete layered defense strategy against SSRF combining all prevention techniques.

---

*Previous: [05-prevention-techniques.md](05-prevention-techniques.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
