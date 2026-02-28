# Chapter 5.4: Amazon ElastiCache

## Overview

Amazon ElastiCache is a **managed in-memory data store** supporting **Redis** and **Memcached**. It delivers microsecond read latency for caching, session management, real-time analytics, and message queuing — a critical layer in high-performance architectures.

---

## ElastiCache Architecture

```
┌──────────────── Application Architecture ───────────────────┐
│                                                              │
│  ┌─── App Server ──────────────────────────────────────┐   │
│  │                                                      │   │
│  │  1. Check cache for data ──────────────►            │   │
│  │                                         │            │   │
│  │  ┌──────────────────────────────────┐   │            │   │
│  │  │ Cache HIT? Return cached data   │◄──┘            │   │
│  │  │ (microseconds)                   │                │   │
│  │  └──────────────────────────────────┘                │   │
│  │                                                      │   │
│  │  2. Cache MISS? Query database ────────────►        │   │
│  │                                              │       │   │
│  │  ┌──────────────────────────────────────┐   │       │   │
│  │  │ Get from DB, store in cache, return │◄──┘       │   │
│  │  │ (milliseconds)                       │           │   │
│  │  └──────────────────────────────────────┘           │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌── ElastiCache ──┐    ┌── Database ───────────────┐      │
│  │  Redis/Memcached │    │  RDS / DynamoDB           │      │
│  │  Microseconds    │    │  Milliseconds             │      │
│  │  In-memory       │    │  Disk-based               │      │
│  └──────────────────┘    └──────────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

---

## Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data structures** | Strings, Lists, Sets, Sorted Sets, Hashes, Streams | Simple key-value only |
| **Persistence** | Yes (snapshots + AOF) | No |
| **Replication** | Yes (read replicas, Multi-AZ) | No |
| **Clustering** | Yes (data sharding) | Yes (multi-node) |
| **Pub/Sub** | Yes | No |
| **Lua scripting** | Yes | No |
| **Backup/Restore** | Yes | No |
| **Multi-threaded** | Single-threaded (per shard) | Multi-threaded |
| **Max item size** | 512 MB | 1 MB |
| **Use case** | Complex caching, sessions, leaderboards, queues | Simple caching, large-scale horizontal |
| **Recommendation** | **Default choice** | Only if simple key-value + multi-threaded needed |

---

## Redis Cluster Mode

```
┌──── Redis Cluster Mode Enabled ────────────────────────────┐
│                                                             │
│  Shard 1                    Shard 2                        │
│  (slots 0-5460)            (slots 5461-10922)              │
│  ┌─────────────┐           ┌─────────────┐                │
│  │ Primary     │           │ Primary     │                │
│  │ AZ-1a       │           │ AZ-1b       │                │
│  └──────┬──────┘           └──────┬──────┘                │
│         │                         │                        │
│  ┌──────▼──────┐           ┌──────▼──────┐                │
│  │ Replica     │           │ Replica     │                │
│  │ AZ-1b       │           │ AZ-1a       │                │
│  └─────────────┘           └─────────────┘                │
│                                                             │
│  Shard 3                                                   │
│  (slots 10923-16383)                                       │
│  ┌─────────────┐                                           │
│  │ Primary     │     • Data partitioned across shards     │
│  │ AZ-1a       │     • Each shard: 1 primary + N replicas │
│  └──────┬──────┘     • Auto-failover within shard         │
│         │            • Online resharding supported          │
│  ┌──────▼──────┐     • Up to 500 nodes per cluster        │
│  │ Replica     │                                           │
│  │ AZ-1b       │                                           │
│  └─────────────┘                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Creating ElastiCache

```bash
# Create Redis replication group (cluster mode disabled)
aws elasticache create-replication-group \
  --replication-group-id prod-redis \
  --replication-group-description "Production Redis" \
  --engine redis \
  --engine-version 7.0 \
  --cache-node-type cache.r6g.large \
  --num-cache-clusters 3 \
  --multi-az-enabled \
  --automatic-failover-enabled \
  --cache-subnet-group-name my-cache-subnets \
  --security-group-ids sg-cache \
  --at-rest-encryption-enabled \
  --transit-encryption-enabled \
  --snapshot-retention-limit 7 \
  --snapshot-window "03:00-05:00"

# Create subnet group for ElastiCache
aws elasticache create-cache-subnet-group \
  --cache-subnet-group-name my-cache-subnets \
  --cache-subnet-group-description "Private subnets for cache" \
  --subnet-ids subnet-priv-1a subnet-priv-1b
```

---

## Caching Strategies

```
┌──── Lazy Loading (Cache-Aside) ────────────────────────────┐
│                                                             │
│  App → Cache: "Do you have key X?"                         │
│      ├─ HIT: Return cached value                           │
│      └─ MISS: Query DB → Write to cache → Return           │
│                                                             │
│  ✓ Only requested data is cached                           │
│  ✓ Resilient to cache failure                              │
│  ✗ Cache miss = 3 trips (cache + DB + write cache)         │
│  ✗ Stale data possible                                     │
└─────────────────────────────────────────────────────────────┘

┌──── Write-Through ─────────────────────────────────────────┐
│                                                             │
│  App → Write to DB AND cache simultaneously                │
│                                                             │
│  ✓ Cache always up to date                                 │
│  ✓ Read latency always low                                 │
│  ✗ Write penalty (write to 2 places)                       │
│  ✗ Unused data may fill cache                              │
└─────────────────────────────────────────────────────────────┘

┌──── Best Practice: Lazy Loading + TTL ─────────────────────┐
│                                                             │
│  • Use Lazy Loading as default strategy                    │
│  • Add TTL (time-to-live) to prevent stale data            │
│  • Add Write-Through for critical data paths               │
│                                                             │
│  SET user:123 '{"name":"Alice"}' EX 3600  ← 1 hour TTL   │
└─────────────────────────────────────────────────────────────┘
```

---

## Common Use Cases

```
┌──── ElastiCache Use Cases ─────────────────────────────────┐
│                                                             │
│  1. DATABASE CACHE                                         │
│     App → Redis → RDS (offload reads, reduce DB load)      │
│                                                             │
│  2. SESSION STORE                                          │
│     ALB → App Server 1 ──┐                                │
│     ALB → App Server 2 ──┼──► Redis (shared sessions)     │
│     ALB → App Server 3 ──┘    Stateless servers!          │
│                                                             │
│  3. RATE LIMITING                                          │
│     INCR api:user:123:count → Check if > 100/min          │
│                                                             │
│  4. LEADERBOARD                                            │
│     ZADD game:scores 1500 "player1"                       │
│     ZREVRANGE game:scores 0 9  → Top 10 players           │
│                                                             │
│  5. PUB/SUB MESSAGING                                      │
│     PUBLISH notifications '{"event":"deploy"}'             │
│     SUBSCRIBE notifications                                │
└─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| High cache miss rate | Cold cache or bad key design | Pre-warm cache; review key patterns |
| Evictions increasing | Cache too small | Scale up node type or add shards |
| Connection timeout | SG not allowing Redis port 6379 | Add inbound 6379 from app SG to cache SG |
| Stale data | No TTL set, cache never refreshes | Set appropriate TTLs on all cached items |
| Failover taking >30s | Large dataset to sync | Use smaller Redis data set; monitor replication lag |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **ElastiCache** | Managed in-memory cache. Redis or Memcached. Microsecond latency. |
| **Redis** | Rich data structures, persistence, replication, Pub/Sub. Default choice. |
| **Memcached** | Simple key-value, multi-threaded, no persistence. |
| **Cluster Mode** | Data sharding across multiple nodes. Online resharding. |
| **Lazy Loading** | Cache on read miss. Add TTL to prevent stale data. |
| **Write-Through** | Write to cache + DB together. Always fresh, but write penalty. |
| **Session Store** | Store sessions in Redis so app servers can be stateless. |

---

## Quick Revision Questions

1. **When would you choose Memcached over Redis?**
   > When you need simple key-value caching with multi-threaded performance and don't need persistence, replication, or complex data structures.

2. **What is the difference between Lazy Loading and Write-Through?**
   > Lazy Loading caches data on read misses (risk of stale data). Write-Through updates cache on every write (always fresh but higher write latency).

3. **How do you prevent stale data in ElastiCache?**
   > Set TTL (time-to-live) on cached items so they automatically expire and get refreshed.

4. **Why is Redis commonly used for session management?**
   > It allows multiple stateless app servers behind a load balancer to share session data, enabling horizontal scaling.

5. **What port does Redis use?**
   > 6379. Security Groups must allow this from application servers.

---

[← Previous: 5.3 DynamoDB](03-dynamodb.md) | [Next: 5.5 Database Migration →](05-database-migration.md)

[← Back to README](../README.md)
