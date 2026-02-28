# Chapter 4: Network Segmentation

## Overview

**Network segmentation** divides a network into smaller, isolated zones to limit the blast radius of security breaches, improve performance, and enforce access controls. In cloud-native environments, segmentation is the foundation of defense-in-depth — it ensures that compromising one component doesn't expose the entire infrastructure.

---

## 4.1 Why Segment Networks?

```
┌──────────────────────────────────────────────────────────────┐
│          FLAT NETWORK vs SEGMENTED NETWORK                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FLAT NETWORK (Bad):                                         │
│  ┌──────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Web   App   DB   Admin   Dev   CI/CD   Monitoring   │    │
│  │   ↔     ↔     ↔     ↔      ↔     ↔        ↔         │    │
│  │         EVERYTHING CAN TALK TO EVERYTHING            │    │
│  │                                                      │    │
│  └──────────────────────────────────────────────────────┘    │
│  ⚠️ Attacker compromises web → accesses database directly    │
│                                                              │
│  SEGMENTED NETWORK (Good):                                   │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌──────────┐       │
│  │  DMZ   │──▶│  App   │──▶│  Data  │   │  Mgmt    │       │
│  │ (Web)  │   │ Tier   │   │ Tier   │   │ (Admin)  │       │
│  │        │   │        │   │        │   │          │       │
│  │ :443   │   │ :8080  │   │ :5432  │   │ :22 VPN  │       │
│  └────────┘   └────────┘   └────────┘   └──────────┘       │
│   Public       Private      Isolated      Restricted         │
│                                                              │
│  ✅ Web can only reach App tier                               │
│  ✅ Only App tier can reach Database                          │
│  ✅ Admin access only via VPN                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Benefits of Segmentation

| Benefit | Description |
|---------|-------------|
| **Blast radius reduction** | Breach in one zone doesn't spread |
| **Compliance** | PCI-DSS, HIPAA require segmentation |
| **Performance** | Less broadcast traffic per segment |
| **Access control** | Enforce least privilege per zone |
| **Monitoring** | Easier to detect anomalous cross-zone traffic |

---

## 4.2 Segmentation Strategies

```
┌──────────────────────────────────────────────────────────────┐
│         SEGMENTATION APPROACHES (Coarse → Fine)             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Level 1: NETWORK-LEVEL (VLANs/Subnets)                     │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ Separate subnets for web, app, db, management       │     │
│  │ Controlled by: NACLs, route tables, firewall rules  │     │
│  │ Granularity: Subnet/IP-based                        │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  Level 2: HOST-LEVEL (Security Groups / iptables)            │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ Per-instance or per-service firewall rules           │     │
│  │ Controlled by: SGs, host firewalls                  │     │
│  │ Granularity: Instance/port-based                    │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  Level 3: APPLICATION-LEVEL (Service Mesh)                   │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ Policy per service identity (not IP)                │     │
│  │ Controlled by: Istio, Linkerd, Calico               │     │
│  │ Granularity: Service/API-based                      │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  Level 4: MICRO-SEGMENTATION (Zero Trust)                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ Every workload is isolated by default               │     │
│  │ All communication requires explicit authorization   │     │
│  │ Controlled by: Network policies + identity + mTLS   │     │
│  │ Granularity: Process/container-level                │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 AWS VPC Segmentation Architecture

```
┌──────────────────────────────────────────────────────────────┐
│             MULTI-TIER VPC SEGMENTATION                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  VPC: 10.0.0.0/16                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │  Public Subnets (DMZ) — 10.0.1.0/24, 10.0.2.0/24      │  │
│  │  ┌──────────┐  ┌──────────┐                            │  │
│  │  │ ALB      │  │ NAT GW   │  ← Internet-facing        │  │
│  │  │ Bastion  │  │          │                            │  │
│  │  └────┬─────┘  └──────────┘                            │  │
│  │       │  NACL: Allow 443 inbound                       │  │
│  │  ─────┼────────────────────────────────────────────    │  │
│  │       │                                                │  │
│  │  Private Subnets (App) — 10.0.10.0/24, 10.0.11.0/24   │  │
│  │  ┌──────────┐  ┌──────────┐                            │  │
│  │  │ ECS/EKS  │  │ Lambda   │  ← No direct internet     │  │
│  │  │ Services │  │          │                            │  │
│  │  └────┬─────┘  └──────────┘                            │  │
│  │       │  SG: Only from ALB on port 8080                │  │
│  │  ─────┼────────────────────────────────────────────    │  │
│  │       │                                                │  │
│  │  Isolated Subnets (Data) — 10.0.20.0/24, 10.0.21.0/24 │  │
│  │  ┌──────────┐  ┌──────────┐                            │  │
│  │  │ RDS      │  │ ElastiC. │  ← No internet, no NAT    │  │
│  │  │ Postgres │  │ Redis    │                            │  │
│  │  └──────────┘  └──────────┘                            │  │
│  │       SG: Only from App subnets on DB port             │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Terraform Implementation

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = { Name = "production-vpc" }
}

# Subnet tiers
locals {
  azs = ["us-east-1a", "us-east-1b"]
  tiers = {
    public  = { cidrs = ["10.0.1.0/24", "10.0.2.0/24"],   public = true  }
    app     = { cidrs = ["10.0.10.0/24", "10.0.11.0/24"],  public = false }
    data    = { cidrs = ["10.0.20.0/24", "10.0.21.0/24"],  public = false }
  }
}

# Public subnets
resource "aws_subnet" "public" {
  count                   = length(local.azs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = local.tiers.public.cidrs[count.index]
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = true
  tags = { Name = "public-${local.azs[count.index]}" }
}

# App subnets (private)
resource "aws_subnet" "app" {
  count             = length(local.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = local.tiers.app.cidrs[count.index]
  availability_zone = local.azs[count.index]
  tags = { Name = "app-${local.azs[count.index]}" }
}

# Data subnets (isolated)
resource "aws_subnet" "data" {
  count             = length(local.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = local.tiers.data.cidrs[count.index]
  availability_zone = local.azs[count.index]
  tags = { Name = "data-${local.azs[count.index]}" }
}

# Security Group: ALB (public-facing)
resource "aws_security_group" "alb" {
  name_prefix = "alb-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Security Group: App (only from ALB)
resource "aws_security_group" "app" {
  name_prefix = "app-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]  # Only ALB
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Security Group: Database (only from App)
resource "aws_security_group" "db" {
  name_prefix = "db-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]  # Only App tier
  }

  # No egress to internet — fully isolated
  egress {
    from_port       = 0
    to_port         = 0
    protocol        = "-1"
    security_groups = [aws_security_group.app.id]
  }
}

# NACL for data subnets (extra layer)
resource "aws_network_acl" "data" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.data[*].id

  # Allow inbound from app subnets only
  ingress {
    rule_no    = 100
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.10.0/24"
    from_port  = 5432
    to_port    = 5432
  }

  ingress {
    rule_no    = 110
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.11.0/24"
    from_port  = 5432
    to_port    = 5432
  }

  # Deny all other inbound
  ingress {
    rule_no    = 200
    protocol   = "-1"
    action     = "deny"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  # Allow ephemeral ports outbound to app subnets
  egress {
    rule_no    = 100
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.10.0/23"
    from_port  = 1024
    to_port    = 65535
  }

  egress {
    rule_no    = 200
    protocol   = "-1"
    action     = "deny"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }
}
```

---

## 4.4 Kubernetes Network Policies

```
┌──────────────────────────────────────────────────────────────┐
│        KUBERNETES NETWORK SEGMENTATION                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Default: ALL pods can talk to ALL pods (flat network)        │
│                                                              │
│  With NetworkPolicy:                                         │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Namespace: frontend                                │     │
│  │  ┌───────┐                                          │     │
│  │  │ nginx │──── allowed (port 8080) ──────┐          │     │
│  │  └───────┘                               │          │     │
│  └──────────────────────────────────────────┼──────────┘     │
│                                             ▼                │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Namespace: backend                                 │     │
│  │  ┌──────┐                                           │     │
│  │  │ api  │──── allowed (port 5432) ──────┐           │     │
│  │  └──────┘                               │           │     │
│  └─────────────────────────────────────────┼───────────┘     │
│                                            ▼                 │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Namespace: database                                │     │
│  │  ┌──────────┐                                       │     │
│  │  │ postgres │  ← Only accepts from backend ns       │     │
│  │  └──────────┘                                       │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Default deny all ingress in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: backend
spec:
  podSelector: {}    # Applies to ALL pods in namespace
  policyTypes:
    - Ingress
    - Egress

---
# Allow frontend → backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              tier: frontend
        - podSelector:
            matchLabels:
              app: nginx
      ports:
        - protocol: TCP
          port: 8080

---
# Allow backend → database on port 5432
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
  namespace: database
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              tier: backend
      ports:
        - protocol: TCP
          port: 5432

---
# Allow DNS egress for all pods (required!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to: []
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

---

## 4.5 Docker Network Segmentation

```bash
# Create isolated networks for each tier
docker network create --driver bridge frontend-net
docker network create --driver bridge backend-net
docker network create --driver bridge db-net --internal  # No external access

# Web container — only frontend network
docker run -d --name nginx \
  --network frontend-net \
  -p 443:443 \
  nginx

# API container — bridge between frontend and backend
docker run -d --name api \
  --network backend-net \
  myapp/api

# Connect API to frontend network too
docker network connect frontend-net api

# Database — only on isolated db network
docker run -d --name postgres \
  --network db-net \
  -e POSTGRES_PASSWORD=secret \
  postgres

# Connect API to db network
docker network connect db-net api

# Result:
#   nginx  ←→ api  (via frontend-net)
#   api    ←→ postgres (via db-net)
#   nginx  ✗  postgres (no shared network!)
```

### Docker Compose Equivalent

```yaml
version: "3.8"
services:
  nginx:
    image: nginx
    ports:
      - "443:443"
    networks:
      - frontend

  api:
    image: myapp/api
    networks:
      - frontend
      - backend

  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external connectivity
```

---

## 4.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Service can't reach database | Security group missing ingress rule | Add SG rule referencing app tier SG |
| Pods can't resolve DNS after NetworkPolicy | DNS egress blocked | Add allow-dns egress policy |
| Docker containers can't communicate | On different networks without bridge | Connect container to shared network |
| App can't reach internet after segmentation | No NAT Gateway for private subnet | Add NAT GW + route table entry |
| NACL blocking responses | Ephemeral ports not allowed | Allow ports 1024-65535 outbound |
| Cross-namespace K8s traffic blocked | Default deny policy applied | Create explicit ingress NetworkPolicy |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Flat Network | Everything can reach everything — dangerous |
| Network-Level | VLANs, subnets, NACLs — coarse segmentation |
| Host-Level | Security groups, iptables — per-instance rules |
| Application-Level | Service mesh, K8s NetworkPolicy — identity-based |
| Micro-Segmentation | Every workload isolated, explicit allow rules |
| 3-Tier Design | Public → App → Data with decreasing access |
| K8s NetworkPolicy | Default deny + explicit allow per pod/namespace |
| Docker Networks | Separate bridge networks per tier, `internal` for isolation |

---

## Quick Revision Questions

1. **Why is a flat network dangerous in production?**
2. **Design a 3-tier VPC segmentation with appropriate security groups.**
3. **What's the difference between network-level and micro-segmentation?**
4. **Write a Kubernetes NetworkPolicy that allows only frontend pods to reach a backend API on port 8080.**
5. **How do you isolate a Docker database container so only the API can reach it?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← HTTPS Encryption](03-https-encryption.md) | [README](../README.md) | [Zero Trust Networking →](05-zero-trust-networking.md) |
