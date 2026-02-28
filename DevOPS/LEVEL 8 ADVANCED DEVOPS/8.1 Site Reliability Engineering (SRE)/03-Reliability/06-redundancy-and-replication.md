# Chapter 3.6: Redundancy and Replication

[← Previous: Designing for Reliability](05-designing-for-reliability.md) | [Back to README](../README.md) | [Next: Incident Classification →](../04-Incident-Management/01-incident-classification.md)

---

## Overview

Redundancy and replication are foundational strategies for achieving high availability. Redundancy eliminates single points of failure by adding extra capacity, while replication ensures data durability and availability across multiple locations. Together, they form the backbone of reliable distributed systems.

---

## 1. Redundancy Types

```
  ┌──────────────────────────────────────────────────────────┐
  │  TYPE              │ HOW IT WORKS       │ EXAMPLE        │
  ├────────────────────┼────────────────────┼────────────────┤
  │ Active-Active      │ All replicas serve │ 2 LBs both     │
  │                    │ traffic; if one    │ active, traffic │
  │                    │ fails, others      │ shifts auto     │
  │                    │ absorb the load    │                 │
  │                    │                    │                 │
  │ Active-Passive     │ Standby takes over │ DB primary +   │
  │ (Hot Standby)      │ when primary fails │ hot replica    │
  │                    │ (near-zero switch) │                 │
  │                    │                    │                 │
  │ Active-Passive     │ Standby must start │ Cold DR site   │
  │ (Cold Standby)     │ up before serving  │ (minutes to    │
  │                    │ (longer failover)  │ hours)         │
  │                    │                    │                 │
  │ N+1 Redundancy     │ N servers handle   │ 3 app servers  │
  │                    │ load; +1 for       │ need 2 under   │
  │                    │ failure tolerance  │ normal load     │
  │                    │                    │                 │
  │ N+2 Redundancy     │ Can survive 2      │ Critical infra │
  │                    │ simultaneous       │ like DNS, LB   │
  │                    │ failures           │                 │
  └────────────────────┴────────────────────┴────────────────┘
```

---

## 2. Redundancy Architecture Diagram

```
  ACTIVE-ACTIVE with Geographic Redundancy:
  
  ┌─────────── Region A (US-East) ──────────┐
  │                                          │
  │  ┌────┐   ┌────────────┐   ┌─────────┐  │
  │  │ LB │──▶│ App Server │──▶│ DB      │  │
  │  │ A  │   │ Cluster    │   │ Primary │  │
  │  └────┘   └────────────┘   └───┬─────┘  │
  │                                │ sync    │
  └────────────────────────────────┼─────────┘
                                   │
                      ╔════════════╪════════════╗
                      ║  Async replication       ║
                      ╚════════════╪════════════╝
                                   │
  ┌─────────── Region B (EU-West) ─┼─────────┐
  │                                │         │
  │  ┌────┐   ┌────────────┐   ┌──▼──────┐  │
  │  │ LB │──▶│ App Server │──▶│ DB      │  │
  │  │ B  │   │ Cluster    │   │ Replica │  │
  │  └────┘   └────────────┘   └─────────┘  │
  │                                          │
  └──────────────────────────────────────────┘
  
  DNS: Global load balancer routes by latency/geography
  Normal: Both regions serve traffic (active-active)
  Failure: DNS removes failed region (< 60s failover)
```

---

## 3. Database Replication Strategies

```
  ┌──────────────────────────────────────────────────────────────┐
  │  SYNCHRONOUS REPLICATION                                     │
  │                                                              │
  │  Client ──▶ Primary ──write──▶ Replica ──ack──▶ Primary     │
  │                                                ──ack──▶ Client│
  │                                                              │
  │  ✓ Zero data loss (RPO = 0)                                  │
  │  ✗ Higher write latency                                      │
  │  ✗ Replica failure blocks writes                             │
  │  Use for: Financial transactions, critical data              │
  ├──────────────────────────────────────────────────────────────┤
  │  ASYNCHRONOUS REPLICATION                                    │
  │                                                              │
  │  Client ──▶ Primary ──ack──▶ Client                          │
  │               └──write──▶ Replica (eventual)                 │
  │                                                              │
  │  ✓ Lower write latency                                       │
  │  ✓ Primary unaffected by replica issues                      │
  │  ✗ Potential data loss on failover (RPO > 0)                 │
  │  Use for: Analytics, non-critical data, cross-region         │
  ├──────────────────────────────────────────────────────────────┤
  │  SEMI-SYNCHRONOUS REPLICATION                                │
  │                                                              │
  │  Client ──▶ Primary ──write──▶ 1 Replica ack (out of N)     │
  │                                                              │
  │  ✓ Balance between durability and performance                │
  │  Use for: Most production databases                          │
  └──────────────────────────────────────────────────────────────┘
```

---

## 4. Replication Topologies

```
  Single-Leader (Primary-Replica):
  ┌─────────┐
  │ Primary │──write──▶ ┌─────────┐
  │  (R/W)  │──write──▶ │Replica 1│ (read-only)
  │         │──write──▶ ┌─────────┐
  └─────────┘           │Replica 2│ (read-only)
                        └─────────┘
  
  Multi-Leader:
  ┌────────┐◄───sync───▶┌────────┐
  │Leader 1│             │Leader 2│
  │ (R/W)  │             │ (R/W)  │
  └────────┘             └────────┘
  ⚠ Conflict resolution needed!
  
  Leaderless (Dynamo-style):
  ┌──────┐  W=2, R=2 (quorum)
  │Node 1│◄──write──┐
  │      │          │
  ├──────┤   ┌──────┴──┐
  │Node 2│◄──│ Client  │
  │      │   └──────┬──┘
  ├──────┤          │
  │Node 3│◄──write──┘
  └──────┘
  Rule: W + R > N ensures consistency
  (2 + 2 > 3 ✓)
```

---

## 5. Consistency vs Availability (CAP Theorem)

```
  ┌──────────────────────────────────────────────────────────┐
  │         CAP THEOREM: Pick 2 of 3                        │
  │                                                          │
  │              Consistency                                  │
  │                 /\                                        │
  │                /  \                                       │
  │               /    \                                      │
  │       CP ───/   CA  \─── CA                              │
  │            /   (no    \                                   │
  │           /  partition) \                                  │
  │          /    (myth)     \                                 │
  │         /________________\                                │
  │   Availability ──AP── Partition                           │
  │                      Tolerance                            │
  │                                                          │
  │  In practice, P is not optional (networks WILL fail)     │
  │  Real choice: CP or AP                                   │
  │                                                          │
  │  CP: Refuse requests during partition (banking)          │
  │      → MongoDB, HBase, ZooKeeper                         │
  │                                                          │
  │  AP: Serve stale data during partition (social media)    │
  │      → Cassandra, DynamoDB, CouchDB                      │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Infrastructure Redundancy Layers

```
  ┌──────────────────────────────────────────────────────────┐
  │  LAYER           │ REDUNDANCY STRATEGY                   │
  ├──────────────────┼───────────────────────────────────────┤
  │ DNS              │ Multiple providers (Route53 +         │
  │                  │ Cloudflare), low TTL for fast switch  │
  │                  │                                       │
  │ Load Balancer    │ Active-active pair, health checks     │
  │                  │ per backend, cross-zone balancing     │
  │                  │                                       │
  │ Application      │ N+1 instances across AZs, auto-      │
  │                  │ scaling groups, rolling deploy        │
  │                  │                                       │
  │ Database         │ Primary + sync replica (same AZ),     │
  │                  │ async replica (cross-AZ), read        │
  │                  │ replicas for read scaling             │
  │                  │                                       │
  │ Cache            │ Redis Cluster with replicas,          │
  │                  │ multi-AZ, handle cache-miss           │
  │                  │ gracefully                            │
  │                  │                                       │
  │ Storage          │ S3 cross-region replication,          │
  │                  │ RAID, 3-copy minimum                  │
  │                  │                                       │
  │ Network          │ Dual ISP, redundant switches,         │
  │                  │ multiple AZs/regions                  │
  │                  │                                       │
  │ Power            │ UPS + generator + dual power feed     │
  └──────────────────┴───────────────────────────────────────┘
```

---

## 7. Availability Math with Redundancy

```
  Single instance:
  A(single) = 99.9% ──▶ 8.76 hours downtime/year
  
  Active-Passive (2 instances):
  A(pair) = 1 - (1 - 0.999)² = 1 - 0.000001 = 99.9999%
  (assumes perfect failover — unrealistic)
  
  Real-world with failover time:
  A(real) = A(single) + (1 - A(single)) × A(failover)
  
  If failover success rate = 95%:
  A(real) = 0.999 + (0.001 × 0.95) = 0.99995 = 99.995%
  
  ┌──────────────────────────────────────────────────────────┐
  │  Config              │ Theoretical  │ Realistic           │
  ├──────────────────────┼──────────────┼─────────────────────┤
  │ Single instance      │ 99.9%        │ 99.9%               │
  │ Active-Passive (2)   │ 99.9999%     │ 99.99% (failover    │
  │                      │              │ adds downtime)       │
  │ Active-Active (2)    │ 99.9999%     │ 99.995%             │
  │ Active-Active (3 AZ) │ 99.999999%   │ 99.999%             │
  │                      │              │                     │
  │ [!] Failover is never instant — always account for MTTR  │
  └──────────────────────┴──────────────┴─────────────────────┘
```

---

## 8. Kubernetes HA Configuration

```yaml
# Highly Available Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 3                    # N+1 redundancy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    spec:
      # Spread across availability zones
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: payment-service
      
      # Don't schedule on same node as other replicas
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - payment-service
              topologyKey: kubernetes.io/hostname
      
      containers:
        - name: payment
          resources:
            requests:
              cpu: "500m"
              memory: "512Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"
          
          # Readiness: only receive traffic when ready
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 5
            failureThreshold: 3
          
          # Liveness: restart if stuck
          livenessProbe:
            httpGet:
              path: /livez
              port: 8080
            periodSeconds: 10
            failureThreshold: 5
---
# Pod Disruption Budget: always keep >= 2 pods
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: payment-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: payment-service
```

---

## 9. Real-World Scenario

### [SCENARIO] Designing a Redundant E-Commerce Platform

```
  BEFORE (Single Points of Failure):
  ┌────────┐   ┌─────┐   ┌────┐
  │ Client │──▶│ App │──▶│ DB │  ← All single instances
  └────────┘   └─────┘   └────┘     Any failure = outage
  
  AFTER (Full Redundancy):
  
       ┌──── Global DNS (Route53 + failover) ────┐
       ▼                                          ▼
  ┌─── US-East ────────────┐  ┌── EU-West ───────────┐
  │ ┌────────────────────┐ │  │ ┌────────────────────┐│
  │ │   ALB (multi-AZ)   │ │  │ │   ALB (multi-AZ)   ││
  │ └─────────┬──────────┘ │  │ └─────────┬──────────┘│
  │    ┌──────┼──────┐     │  │    ┌──────┼──────┐    │
  │    ▼      ▼      ▼     │  │    ▼      ▼      ▼    │
  │  [App] [App]  [App]    │  │  [App] [App]  [App]   │
  │  AZ-a  AZ-b   AZ-c    │  │  AZ-a  AZ-b   AZ-c   │
  │           │            │  │           │           │
  │    ┌──────┴──────┐     │  │    ┌──────┴──────┐    │
  │    ▼             ▼     │  │    ▼             ▼    │
  │  [DB Primary] [Replica]│  │  [DB Replica]        │
  │   sync ──────▶         │  │  async from US-East  │
  │               ┌──────┐ │  │                      │
  │               │Redis │ │  │  ┌──────┐            │
  │               │Cluster│ │  │  │Redis │            │
  │               └──────┘ │  │  │Cluster│            │
  └────────────────────────┘  │  └──────┘            │
                              └──────────────────────┘
  
  Result:
  ├─ Single AZ failure: N+1 app servers absorb, DB failover
  ├─ Single region failure: DNS routes all traffic to other
  ├─ DB primary failure: Sync replica promotes in < 30s
  ├─ Availability: 99.99%+ (4.4 min/month max downtime)
  └─ RPO: 0 (sync replication), RTO: < 60s
```

---

## 10. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Failover takes too long" | Pre-warm standby instances. Use hot standby instead of cold. Monitor failover time as an SLI. |
| "Replication lag is growing" | Check replica CPU/IO, increase replica resources, consider read-only traffic shedding. |
| "Split-brain after network partition" | Use fencing (STONITH), consensus-based leader election, or cloud-native managed DBs. |
| "Redundancy costs too much" | Use active-active to serve traffic from all replicas. Reserved instances reduce cost. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Active-Active** | All replicas serve traffic; highest availability |
| **Active-Passive** | Standby takes over on failure; simpler but wasteful |
| **Sync Replication** | Zero data loss; higher latency |
| **Async Replication** | Lower latency; risk of data loss on failover |
| **CAP Theorem** | Consistency vs Availability during network partition |
| **Quorum** | W + R > N ensures read-after-write consistency |
| **PDB** | Pod Disruption Budget prevents voluntary eviction |

---

## Quick Revision Questions

1. **What is the difference between active-active and active-passive redundancy?**
   > Active-active: all replicas serve traffic simultaneously. Active-passive: standby sits idle until primary fails, then takes over.

2. **Compare synchronous and asynchronous replication.**
   > Synchronous: primary waits for replica ACK before confirming write (RPO=0, higher latency). Asynchronous: primary confirms immediately, replica catches up eventually (lower latency, potential data loss).

3. **What does the CAP theorem state, and what's the practical implication?**
   > In a distributed system, you can have at most 2 of: Consistency, Availability, Partition Tolerance. Since partitions are inevitable, the real choice is CP (reject requests to stay consistent) or AP (serve possibly stale data).

4. **What is the quorum formula and why does it matter?**
   > W + R > N (Write nodes + Read nodes > Total nodes). Ensures at least one node in a read overlaps with a write, guaranteeing you read the latest data.

5. **Why is theoretical availability always higher than real-world availability?**
   > Theoretical assumes instant, perfect failover. In reality, failover detection, promotion, and DNS propagation all add downtime (MTTR > 0).

6. **What Kubernetes features help achieve high availability?**
   > Pod anti-affinity (spread across nodes), topology spread constraints (spread across AZs), PodDisruptionBudget (min availability during maintenance), readiness/liveness probes, rolling updates.

---

[← Previous: Designing for Reliability](05-designing-for-reliability.md) | [Back to README](../README.md) | [Next: Incident Classification →](../04-Incident-Management/01-incident-classification.md)
