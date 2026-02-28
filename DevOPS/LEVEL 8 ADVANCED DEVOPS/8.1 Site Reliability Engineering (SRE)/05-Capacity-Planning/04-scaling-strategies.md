# Chapter 5.4: Scaling Strategies

[← Previous: Performance Modeling](03-performance-modeling.md) | [Back to README](../README.md) | [Next: Cost Optimization →](05-cost-optimization.md)

---

## Overview

Scaling strategies determine how systems grow to meet increasing demand. The choice between vertical scaling (bigger machines) and horizontal scaling (more machines), along with techniques like sharding, caching, and CDNs, fundamentally shapes system architecture, cost, and reliability characteristics.

---

## 1. Vertical vs Horizontal Scaling

```
  ┌──────────────────────────────────────────────────────────┐
  │  VERTICAL SCALING (Scale Up)      │ HORIZONTAL (Scale Out)│
  │  ──────────────────────────       │ ──────────────────── │
  │                                   │                      │
  │  ┌──────────────┐                 │  ┌────┐┌────┐┌────┐ │
  │  │              │                 │  │ S1 ││ S2 ││ S3 │ │
  │  │  BIG SERVER  │                 │  └────┘└────┘└────┘ │
  │  │              │                 │  ┌────┐┌────┐┌────┐ │
  │  │  64 CPU      │                 │  │ S4 ││ S5 ││ S6 │ │
  │  │  256 GB RAM  │                 │  └────┘└────┘└────┘ │
  │  │              │                 │                      │
  │  └──────────────┘                 │  12 × small servers  │
  │                                   │                      │
  │  ✓ Simple (no code change)       │  ✓ Near-infinite     │
  │  ✓ No distributed complexity     │  ✓ Fault tolerant    │
  │  ✗ Hardware limit ceiling        │  ✓ Cost-efficient    │
  │  ✗ Single point of failure       │  ✗ Distributed       │
  │  ✗ Expensive at high end         │    complexity        │
  │  ✗ Downtime for upgrade          │  ✗ Consistency harder│
  │                                   │  ✗ Network overhead  │
  │  Best for: Databases, legacy     │  Best for: Stateless │
  │  apps, monoliths                 │  services, web apps  │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Scaling Decision Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Is the workload stateless?                              │
  │  ├─ YES ──▶ Horizontal scaling (auto-scale groups)       │
  │  └─ NO                                                   │
  │      Is it read-heavy?                                   │
  │      ├─ YES ──▶ Read replicas + caching                  │
  │      └─ NO                                               │
  │          Is it write-heavy?                              │
  │          ├─ YES ──▶ Sharding + queue-based writes        │
  │          └─ NO (mixed)                                   │
  │              Can you separate reads from writes?         │
  │              ├─ YES ──▶ CQRS pattern                     │
  │              └─ NO  ──▶ Vertical first, then shard       │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Auto-Scaling Configuration

```yaml
# AWS Auto Scaling Group
AutoScalingGroup:
  MinSize: 3
  MaxSize: 20
  DesiredCapacity: 5
  
  ScalingPolicies:
    # Target tracking — simplest and most effective
    - PolicyName: "cpu-target-tracking"
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 60.0    # Keep avg CPU at 60%
        ScaleInCooldown: 300  # Wait 5 min before scale-in
        ScaleOutCooldown: 60  # Wait 1 min before scale-out
    
    # Custom metric — RPS per instance
    - PolicyName: "rps-per-target"
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        CustomizedMetricSpecification:
          MetricName: "RequestsPerTarget"
          Namespace: "Custom/App"
          Statistic: Average
        TargetValue: 500.0   # 500 RPS per instance target

# Kubernetes HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicas: 3
  maxReplicas: 30
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 50        # Scale up by 50% at most
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25        # Scale down by 25% at most
          periodSeconds: 120
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Pods
      pods:
        metric:
          name: requests_per_second
        target:
          type: AverageValue
          averageValue: "500"
```

---

## 4. Database Scaling Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  STRATEGY         │ MECHANISM            │ USE CASE       │
  ├───────────────────┼──────────────────────┼────────────────┤
  │ Read replicas     │ Async replication    │ Read-heavy     │
  │                   │ to read-only copies  │ (90:10 R:W)    │
  │                   │                      │                │
  │ Vertical (bigger) │ More CPU/RAM/IOPS    │ Quick fix,     │
  │                   │                      │ single-digit TB│
  │                   │                      │                │
  │ Connection pooling│ PgBouncer, ProxySQL  │ Many clients,  │
  │                   │                      │ limited conns  │
  │                   │                      │                │
  │ Functional        │ Separate DBs per     │ Microservices, │
  │ partitioning      │ domain/service       │ bounded context│
  │                   │                      │                │
  │ Horizontal        │ Shard data across    │ Very large     │
  │ sharding          │ multiple DB instances│ datasets, high │
  │                   │                      │ write throughput│
  └───────────────────┴──────────────────────┴────────────────┘
  
  Sharding Strategies:
  ┌──────────────────────────────────────────────────────────┐
  │  Range-based:     user_id 1-1M → Shard A                │
  │                   user_id 1M-2M → Shard B               │
  │  ✓ Simple queries │ ✗ Hot spots possible                 │
  │                                                          │
  │  Hash-based:      shard = hash(user_id) % num_shards    │
  │  ✓ Even distribution │ ✗ Resharding is complex           │
  │                                                          │
  │  Directory-based: Lookup service maps key → shard        │
  │  ✓ Flexible │ ✗ Lookup is SPOF, adds latency            │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Caching Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  CACHING LAYER POSITIONING                               │
  │                                                          │
  │  Client ──▶ CDN ──▶ App Cache ──▶ DB Cache ──▶ DB       │
  │   (browser)  (edge)  (Redis)     (query cache)           │
  │                                                          │
  │  PATTERN          │ HOW IT WORKS                         │
  │  ─────────────────┼──────────────────────────────────    │
  │  Cache-aside      │ App checks cache first, fetches     │
  │  (lazy loading)   │ from DB on miss, writes to cache    │
  │                   │                                      │
  │  Write-through    │ App writes to cache AND DB at once   │
  │                   │ (consistent but slower writes)       │
  │                   │                                      │
  │  Write-behind     │ App writes to cache, cache async     │
  │  (write-back)     │ writes to DB (fast but risk of loss) │
  │                   │                                      │
  │  Read-through     │ Cache is source of truth for reads   │
  │                   │ Cache itself fetches from DB on miss │
  │                   │                                      │
  │  Cache hit ratio target: > 90% for effective caching     │
  │  Monitor: cache_hits / (cache_hits + cache_misses)       │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. CDN and Edge Scaling

```
  ┌──────────────────────────────────────────────────────────┐
  │               CDN ARCHITECTURE                           │
  │                                                          │
  │  User (Tokyo)──▶ Edge POP (Tokyo) ──cache hit──▶ Done   │
  │                        │                                 │
  │                   cache miss                             │
  │                        │                                 │
  │                        ▼                                 │
  │            Origin Shield (Singapore)                     │
  │                        │                                 │
  │                   cache miss                             │
  │                        │                                 │
  │                        ▼                                 │
  │              Origin Server (US-East)                     │
  │                                                          │
  │  What to cache at CDN:                                   │
  │  ├─ Static assets (JS, CSS, images, fonts)               │
  │  ├─ API responses with Cache-Control headers             │
  │  ├─ Pre-rendered HTML pages                              │
  │  ├─ Video/media content                                  │
  │  └─ DNS responses (low TTL for flexibility)              │
  │                                                          │
  │  CDN can absorb 80-95% of traffic, dramatically         │
  │  reducing origin load and improving latency              │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Real-World Scenario

### [SCENARIO] Scaling an E-Commerce Platform 10x

```
  Current: 5,000 RPS (normal) / 15,000 RPS (peak)
  Target:  50,000 RPS (normal) / 150,000 RPS (peak)
  
  ┌──────────────────────────────────────────────────────────┐
  │  LAYER        │ BEFORE        │ AFTER                    │
  ├───────────────┼───────────────┼──────────────────────────┤
  │ CDN           │ Static only   │ Static + API caching     │
  │               │               │ (absorbs 70% traffic)    │
  │               │               │                          │
  │ Load Balancer │ Single ALB    │ Global LB (multi-region) │
  │               │               │                          │
  │ Web/API       │ 4 instances   │ HPA: 10-80 instances     │
  │               │ manual scale  │ auto-scale on CPU/RPS    │
  │               │               │                          │
  │ Cache         │ Single Redis  │ Redis Cluster (6 nodes)  │
  │               │               │ Hit ratio: 92%           │
  │               │               │                          │
  │ Database      │ Single PG     │ 1 primary + 4 read       │
  │               │ instance      │ replicas + PgBouncer     │
  │               │               │                          │
  │ Search        │ Single ES     │ ES cluster (6 nodes)     │
  │               │ node          │                          │
  │               │               │                          │
  │ Queue         │ Single RabbitMQ│ Kafka cluster (3 brokers)│
  └───────────────┴───────────────┴──────────────────────────┘
  
  Key decisions:
  ├─ CDN caching reduced origin traffic by 70%
  ├─ Read replicas handled 80% of DB queries
  ├─ Redis cache hit ratio of 92% reduced DB load by 9x
  ├─ Async processing via Kafka for non-critical writes
  └─ Result: 10x scale achieved with 4x cost increase
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Auto-scaling is too slow for traffic spikes" | Use predictive scaling or schedule-based pre-scaling for known events. |
| "Cache hit ratio is low (<70%)" | Analyze cache key distribution. Increase TTL. Add cache warming for popular items. |
| "Horizontal scaling doesn't improve throughput" | Bottleneck is likely DB/shared state. Scale the data layer or add caching. |
| "Sharding creates hot spots" | Hash-based sharding distributes more evenly. Monitor per-shard metrics. |

---

## Summary Table

| Strategy | When to Use | Limit |
|----------|------------|-------|
| **Vertical Scaling** | Quick fix, stateful workloads | Hardware ceiling |
| **Horizontal Scaling** | Stateless services | Serialization point |
| **Read Replicas** | Read-heavy workloads (>80% reads) | Replication lag |
| **Sharding** | Large datasets, high write throughput | Query complexity |
| **Caching** | Frequently read, rarely changed data | Staleness, consistency |
| **CDN** | Static content, geographically distributed users | Dynamic content |
| **Auto-Scaling** | Variable load patterns | Scale-up latency |

---

## Quick Revision Questions

1. **When should you choose vertical vs horizontal scaling?**
   > Vertical: databases, legacy apps, quick fixes (limited by hardware ceiling). Horizontal: stateless services, microservices, web apps (limited by serialization points).

2. **Name three database sharding strategies with tradeoffs.**
   > Range-based (simple but hot spots), Hash-based (even distribution but resharding is complex), Directory-based (flexible but adds lookup latency).

3. **What is the cache-aside pattern?**
   > Application checks cache first; on miss, fetches from DB and stores in cache. Most common pattern. Simple but cache can become stale.

4. **Why is auto-scaling cooldown important?**
   > Without cooldown, the system oscillates — rapidly scaling up and down (flapping). Cooldown periods ensure scaling decisions are based on sustained load, not transient spikes.

5. **How does a CDN reduce origin server load?**
   > CDN caches content at edge locations close to users. Requests for cached content are served directly from the edge, never reaching the origin. Can absorb 70-95% of traffic.

6. **What's a common mistake when horizontally scaling services backed by a single database?**
   > Adding more app servers without scaling the database. The DB becomes the bottleneck. Need to also add read replicas, caching, or sharding.

---

[← Previous: Performance Modeling](03-performance-modeling.md) | [Back to README](../README.md) | [Next: Cost Optimization →](05-cost-optimization.md)
