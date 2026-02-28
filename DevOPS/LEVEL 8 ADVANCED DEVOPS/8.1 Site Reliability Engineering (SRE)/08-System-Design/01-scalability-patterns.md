# Chapter 8.1: Scalability Patterns

[← Previous: Change Management](../07-Release-Engineering/06-change-management.md) | [Back to README](../README.md) | [Next: High Availability Patterns →](02-high-availability-patterns.md)

---

## Overview

Scalability is the ability of a system to handle increased load by adding resources. SRE teams design systems that scale predictably, cost-effectively, and without reliability degradation. This chapter covers the fundamental patterns for building scalable distributed systems.

---

## 1. Scalability Dimensions

```
  ┌──────────────────────────────────────────────────────────┐
  │  THREE DIMENSIONS OF SCALABILITY                         │
  │                                                          │
  │  X-AXIS: Horizontal Duplication                          │
  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                         │
  │  │ A │ │ A │ │ A │ │ A │ │ A │  ← Clone everything     │
  │  └───┘ └───┘ └───┘ └───┘ └───┘    (each instance        │
  │  All instances identical, LB distributes                 │
  │                                                          │
  │  Y-AXIS: Functional Decomposition                        │
  │  ┌──────┐ ┌──────┐ ┌──────┐                             │
  │  │ Auth │ │ Cart │ │ Pay  │  ← Split by function        │
  │  └──────┘ └──────┘ └──────┘    (microservices)          │
  │  Each service scales independently                       │
  │                                                          │
  │  Z-AXIS: Data Partitioning (Sharding)                    │
  │  ┌──────┐ ┌──────┐ ┌──────┐                             │
  │  │ A-H  │ │ I-P  │ │ Q-Z  │  ← Split by data           │
  │  └──────┘ └──────┘ └──────┘    (each shard handles     │
  │  Same code, different data subsets                        │
  │                                                          │
  │  Most systems use ALL THREE for full scalability.        │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Load Balancing Patterns

```
  ┌──────────────────────────────────────────────────────────┐
  │  CLIENT REQUEST FLOW                                     │
  │                                                          │
  │  ┌────────┐     ┌─────────┐     ┌──────────────────┐    │
  │  │ Client │────▶│ DNS LB  │────▶│ L4 Load Balancer │    │
  │  └────────┘     │(GeoDNS) │     │   (TCP/UDP)      │    │
  │                 └─────────┘     └────────┬─────────┘    │
  │                                          │               │
  │                                 ┌────────▼─────────┐    │
  │                                 │ L7 Load Balancer │    │
  │                                 │ (HTTP/gRPC)      │    │
  │                                 └────────┬─────────┘    │
  │                                          │               │
  │                  ┌───────────────┬───────┴────────┐     │
  │                  ▼               ▼                ▼     │
  │              ┌──────┐       ┌──────┐         ┌──────┐  │
  │              │ Pod1 │       │ Pod2 │         │ Pod3 │  │
  │              └──────┘       └──────┘         └──────┘  │
  └──────────────────────────────────────────────────────────┘

  LOAD BALANCING ALGORITHMS:
  ┌─────────────────────┬────────────────────────────────────┐
  │ Algorithm           │ Best For                           │
  ├─────────────────────┼────────────────────────────────────┤
  │ Round Robin         │ Homogeneous instances, even load   │
  │ Weighted Round Rob. │ Mixed instance sizes               │
  │ Least Connections   │ Long-lived connections (WebSocket) │
  │ Random              │ Simple, surprisingly effective     │
  │ Hash-based          │ Session affinity, cache locality   │
  │ Power of 2 choices  │ Low-latency (pick best of 2 rand) │
  └─────────────────────┴────────────────────────────────────┘
```

---

## 3. Caching Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  CACHING LAYERS                                          │
  │                                                          │
  │  ┌────────────┐  Hit rate: ~95%  Latency: <1ms          │
  │  │ L1: In-app │  (process memory, e.g., Guava cache)    │
  │  │   memory   │  Limitation: per-instance, not shared   │
  │  └──────┬─────┘                                          │
  │         │ miss                                           │
  │  ┌──────▼─────┐  Hit rate: ~90%  Latency: 1-5ms         │
  │  │ L2: Redis/ │  (shared cache, e.g., Redis cluster)    │
  │  │ Memcached  │  Limitation: network hop, serialization │
  │  └──────┬─────┘                                          │
  │         │ miss                                           │
  │  ┌──────▼─────┐  Hit rate: ~80%  Latency: 10-50ms       │
  │  │ L3: CDN    │  (edge cache, e.g., CloudFront)         │
  │  │            │  Limitation: staleness, cache keys      │
  │  └──────┬─────┘                                          │
  │         │ miss                                           │
  │  ┌──────▼─────┐                  Latency: 50-500ms      │
  │  │ Origin DB  │  (source of truth)                      │
  │  └────────────┘                                          │
  └──────────────────────────────────────────────────────────┘

  CACHE PATTERNS:
  ┌──────────────────┬───────────────────────────────────────┐
  │ Pattern          │ How It Works                          │
  ├──────────────────┼───────────────────────────────────────┤
  │ Cache-Aside      │ App checks cache → miss → query DB → │
  │ (Lazy Loading)   │ write to cache → return              │
  │                  │                                       │
  │ Write-Through    │ App writes to cache → cache writes   │
  │                  │ to DB synchronously                  │
  │                  │                                       │
  │ Write-Behind     │ App writes to cache → cache writes   │
  │ (Write-Back)     │ to DB asynchronously (batch)         │
  │                  │                                       │
  │ Read-Through     │ Cache automatically fetches from DB  │
  │                  │ on miss (cache manages reads)        │
  └──────────────────┴───────────────────────────────────────┘
```

---

## 4. Asynchronous Processing

```
  ┌──────────────────────────────────────────────────────────┐
  │  SYNCHRONOUS vs ASYNCHRONOUS                             │
  │                                                          │
  │  SYNC (tight coupling):                                  │
  │  ┌──────┐───▶┌──────┐───▶┌──────┐                      │
  │  │ API  │    │ Pay  │    │ Email│  Total: 200+300+100   │
  │  │ 200ms│◀───│ 300ms│◀───│ 100ms│  = 600ms to user     │
  │  └──────┘    └──────┘    └──────┘                        │
  │  If Email is down, entire request fails!                 │
  │                                                          │
  │  ASYNC (loose coupling):                                 │
  │  ┌──────┐───▶┌──────┐                                   │
  │  │ API  │    │ Pay  │    Total: 200+300 = 500ms to user │
  │  │ 200ms│◀───│ 300ms│                                   │
  │  └──────┘    └──────┘                                    │
  │       │                                                  │
  │       ▼ publish                                          │
  │  ┌──────────┐     ┌──────┐                              │
  │  │  Queue   │────▶│ Email│  Processed asynchronously    │
  │  │ (Kafka)  │     │ 100ms│  If down, messages wait     │
  │  └──────────┘     └──────┘                              │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Kafka topic configuration for order processing
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: order-events
  labels:
    strimzi.io/cluster: production
spec:
  partitions: 12          # Parallelism = 12 consumers max
  replicas: 3             # Durability: 3 copies
  config:
    retention.ms: 604800000       # 7-day retention
    cleanup.policy: delete
    min.insync.replicas: 2        # At least 2 replicas must ACK
    compression.type: lz4         # Save bandwidth
    max.message.bytes: 1048576    # 1MB max message size
```

---

## 5. Database Scalability

```
  ┌──────────────────────────────────────────────────────────┐
  │  DATABASE SCALING HIERARCHY                              │
  │                                                          │
  │  Level 1: OPTIMIZE (do this first!)                     │
  │  ├─ Query optimization (indexes, EXPLAIN)               │
  │  ├─ Connection pooling (PgBouncer, ProxySQL)            │
  │  ├─ Schema optimization (normalization/denormalization)  │
  │  └─ Cost: LOW   Impact: 2-10x improvement              │
  │                                                          │
  │  Level 2: VERTICAL SCALE                                 │
  │  ├─ Bigger machines (more CPU, RAM, SSD)                │
  │  ├─ Simple, no code changes required                    │
  │  └─ Cost: MEDIUM   Limit: Largest available instance    │
  │                                                          │
  │  Level 3: READ REPLICAS                                  │
  │  ├─ Write to primary, read from replicas                │
  │  ├─ Great for read-heavy workloads (80/20 rule)         │
  │  └─ Cost: MEDIUM   Limit: Write bottleneck remains     │
  │                                                          │
  │  Level 4: SHARDING (partitioning)                       │
  │  ├─ Split data across multiple databases                │
  │  ├─ Complex: cross-shard queries, rebalancing           │
  │  └─ Cost: HIGH   Scale: Near unlimited (horizontal)    │
  │                                                          │
  │  Level 5: POLYGLOT PERSISTENCE                          │
  │  ├─ Use the right DB for each workload                  │
  │  ├─ RDBMS + Redis + Elasticsearch + S3                  │
  │  └─ Cost: HIGH   Complexity: Very high                 │
  │                                                          │
  │  RULE: Exhaust each level before moving to the next.    │
  └──────────────────────────────────────────────────────────┘
```

```
  READ REPLICA ARCHITECTURE:
  
  ┌──────────┐  writes   ┌───────────┐
  │ App      │──────────▶│ Primary   │
  │ (write)  │           │ (master)  │
  └──────────┘           └─────┬─────┘
                               │ replication
                     ┌─────────┼─────────┐
                     ▼         ▼         ▼
                ┌────────┐ ┌────────┐ ┌────────┐
                │Replica1│ │Replica2│ │Replica3│
                └────┬───┘ └────┬───┘ └────┬───┘
                     │         │         │
  ┌──────────┐       │         │         │
  │ App      │───────┴─────────┴─────────┘
  │ (read)   │  reads (load balanced)
  └──────────┘
  
  CAUTION: Replication lag can cause stale reads!
  Use read-your-writes consistency for critical paths.
```

---

## 6. Rate Limiting and Backpressure

```
  ┌──────────────────────────────────────────────────────────┐
  │  RATE LIMITING ALGORITHMS                                │
  │                                                          │
  │  TOKEN BUCKET:                                           │
  │  ┌──────────────────────────────┐                        │
  │  │  Bucket (capacity: 100)     │                        │
  │  │  ████████████████░░░░░░░░░░ │ ← 60 tokens left      │
  │  │                              │                        │
  │  │  Refill: 10 tokens/second   │                        │
  │  │  Request uses 1 token       │                        │
  │  │  Empty = reject (429)       │                        │
  │  └──────────────────────────────┘                        │
  │  Best for: Bursty traffic with sustained rate limit     │
  │                                                          │
  │  SLIDING WINDOW:                                         │
  │  ┌──────────────────────────────┐                        │
  │  │  |←── 1 minute window ──▶|  │                        │
  │  │  ■ ■ ■ ■ ■ ■ ■ ■ · · · ·  │ ← 8 of 10 used       │
  │  │  Count requests in window   │                        │
  │  └──────────────────────────────┘                        │
  │  Best for: Smooth rate enforcement                      │
  │                                                          │
  │  BACKPRESSURE MECHANISMS:                                │
  │  ├─ Queue length limits (reject when full)              │
  │  ├─ Load shedding (drop low-priority requests)          │
  │  ├─ Circuit breaker (stop calling failing service)      │
  │  ├─ Exponential backoff (client slows down)             │
  │  └─ Admission control (reject at ingress when overload) │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Content Delivery Networks (CDN)

```
  ┌──────────────────────────────────────────────────────────┐
  │  CDN ARCHITECTURE                                        │
  │                                                          │
  │  User (London)                                           │
  │       │                                                  │
  │       ▼                                                  │
  │  ┌──────────┐  Cache HIT → serve directly (5ms)        │
  │  │ Edge PoP │                                            │
  │  │ (London) │  Cache MISS ↓                             │
  │  └────┬─────┘                                            │
  │       │                                                  │
  │       ▼                                                  │
  │  ┌──────────┐  Regional cache layer                     │
  │  │ Regional │                                            │
  │  │ (EU)     │  Cache MISS ↓                             │
  │  └────┬─────┘                                            │
  │       │                                                  │
  │       ▼                                                  │
  │  ┌──────────┐                                            │
  │  │ Origin   │  200ms+ (cross-ocean)                     │
  │  │ (US-East)│                                            │
  │  └──────────┘                                            │
  │                                                          │
  │  WHAT TO CACHE:                                          │
  │  ├─ Static assets (JS, CSS, images) — TTL: days/weeks  │
  │  ├─ API responses (GET) — TTL: seconds/minutes         │
  │  ├─ Dynamic pages (personalized) — Edge compute        │
  │  └─ Video/media — Chunked, adaptive bitrate            │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Scaling an E-commerce Platform for Flash Sales

```
  CHALLENGE: Online retailer experiences 50x normal traffic 
  during flash sales (from 5K rps to 250K rps in 2 minutes)
  
  PREVIOUS INCIDENTS:
  ├─ Flash sale 1: Site crashed at 3x load
  ├─ Flash sale 2: Database deadlocks at 10x load
  └─ Flash sale 3: Partial success, 40% of users saw errors
  
  ┌──────────────────────────────────────────────────────────┐
  │  SCALABILITY REDESIGN                                    │
  │                                                          │
  │  LAYER 1: CDN + Static Assets                           │
  │  ├─ Product images/CSS/JS served from CDN edge          │
  │  ├─ Product catalog cached at edge (30s TTL)            │
  │  └─ Reduced origin traffic by 80%                       │
  │                                                          │
  │  LAYER 2: API Gateway + Rate Limiting                   │
  │  ├─ Token bucket: 10 req/s per user                     │
  │  ├─ Priority queue: checkout > browse > search          │
  │  └─ Bot protection: CAPTCHA after 20 req/min            │
  │                                                          │
  │  LAYER 3: Auto-scaling Application Tier                 │
  │  ├─ Pre-scaled 2 hours before sale (20 → 200 pods)     │
  │  ├─ HPA for organic scaling during sale                 │
  │  └─ Warm pod pool (pre-initialized, ready in <30s)     │
  │                                                          │
  │  LAYER 4: Async Order Queue                             │
  │  ├─ "Add to cart" = synchronous (fast, cached)          │
  │  ├─ "Place order" = async queue (Kafka + workers)       │
  │  ├─ User sees "Order received, processing..."          │
  │  └─ Decouples spike from payment processing             │
  │                                                          │
  │  LAYER 5: Database Protection                           │
  │  ├─ Read replicas for product catalog (8 replicas)      │
  │  ├─ Redis cache for inventory counts (write-through)    │
  │  ├─ Connection pooling (PgBouncer, 200 connections)     │
  │  └─ Inventory reservation via Redis DECR (atomic)       │
  │                                                          │
  │  RESULTS:                                                │
  │  ├─ Successfully handled 300K rps (60x normal)          │
  │  ├─ p99 latency: 180ms (was 2.5s under load)           │
  │  ├─ Zero downtime during flash sale                     │
  │  └─ Revenue increased 340% over previous flash sale     │
  └──────────────────────────────────────────────────────────┘
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Auto-scaling too slow for traffic spikes" | Pre-scale before anticipated events. Use warm pod pools. Keep minimum replica counts high enough for baseline. |
| "Cache stampede on popular items" | Use cache locks (only 1 request fetches, others wait). Stagger TTL with jitter. Implement request coalescing. |
| "Database connection exhaustion" | Use connection pooling (PgBouncer). Reduce connection timeout. Implement queue-based backpressure. |
| "Sharding hotspots (uneven distribution)" | Use consistent hashing. Monitor shard load distribution. Implement virtual shards for rebalancing. |
| "Read replicas falling behind (replication lag)" | Monitor lag metrics. Route critical reads to primary. Increase replica resources. Consider synchronous replication for critical data. |

---

## Summary Table

| Pattern | Use Case | Trade-off |
|---------|----------|-----------|
| **X-Axis (Cloning)** | Stateless services | Simple but all instances same size |
| **Y-Axis (Decomposition)** | Independent scaling per function | Complexity of distributed systems |
| **Z-Axis (Sharding)** | Large datasets, write-heavy | Cross-shard queries are expensive |
| **Caching** | Read-heavy workloads | Staleness, cache invalidation |
| **Async Processing** | Decouple spikes from processing | Eventual consistency, debugging |
| **Rate Limiting** | Protect services from overload | May reject legitimate traffic |
| **CDN** | Global user base, static content | Cache invalidation, cost |

---

## Quick Revision Questions

1. **What are the three scalability axes (X, Y, Z)?**
   > X-axis: horizontal duplication (clone instances behind a load balancer). Y-axis: functional decomposition (split into microservices). Z-axis: data partitioning/sharding (split data across instances running the same code).

2. **In what order should you approach database scalability?**
   > Optimize queries/indexes first → vertical scaling → read replicas → sharding → polyglot persistence. Always exhaust cheaper/simpler options before moving to complex solutions.

3. **What is cache stampede and how do you prevent it?**
   > When a popular cache key expires, many concurrent requests all try to regenerate it simultaneously, overwhelming the origin. Prevention: cache locks (one request regenerates, others wait), staggered TTL with jitter, and request coalescing.

4. **Why is asynchronous processing important for scalability?**
   > It decouples request acceptance from processing, allowing the system to absorb traffic spikes by queuing work. If a downstream service is slow or down, messages wait in the queue instead of failing the user's request.

5. **What load balancing algorithm would you choose for WebSocket connections?**
   > Least Connections, because WebSocket connections are long-lived and you want to distribute new connections to the server with the fewest active connections, ensuring even load distribution.

6. **How does rate limiting protect system scalability?**
   > Rate limiting caps the number of requests per client, preventing any single user or bot from consuming disproportionate resources. It protects backend services from overload and ensures fair access during traffic spikes.

---

[← Previous: Change Management](../07-Release-Engineering/06-change-management.md) | [Back to README](../README.md) | [Next: High Availability Patterns →](02-high-availability-patterns.md)
