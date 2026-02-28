# Chapter 4: Azure Cache for Redis

## Overview
**Azure Cache for Redis** is a fully managed in-memory data store based on Redis. It provides sub-millisecond latency for caching, session management, real-time analytics, and message brokering — dramatically improving application performance.

---

## 4.1 Redis Cache Architecture

```
┌──────────── REDIS CACHE ARCHITECTURE ──────────┐
│                                                  │
│   ┌──────────────┐    Cache Miss     ┌────────┐ │
│   │  Application │─────────────────► │Database│ │
│   │              │                   │        │ │
│   │              │◄─────────────────│        │ │
│   │              │  Read from DB +   └────────┘ │
│   │              │  Store in Cache               │
│   │              │                               │
│   │              │    Cache Hit (fast!)          │
│   │              │◄──────────┐                   │
│   └──────────────┘           │                   │
│          │                   │                   │
│          │              ┌────┴────┐              │
│          └─────────────►│  Redis  │              │
│            Check cache  │  Cache  │              │
│            first        │ (< 1ms) │              │
│                         └─────────┘              │
│                                                  │
│   Result: 10-100x faster reads!                  │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 4.2 Common Use Cases

| Use Case | Description |
|----------|-------------|
| **Data Caching** | Cache frequent database queries |
| **Session Store** | Store user sessions across web farm |
| **Message Queue** | Pub/Sub messaging, task queues |
| **Leaderboards** | Sorted sets for gaming/ranking |
| **Rate Limiting** | Track API request counts |
| **Real-time Analytics** | Counters, HyperLogLog for unique counts |

---

## 4.3 Service Tiers

| Tier | Memory | Features | Use Case |
|------|--------|----------|----------|
| **Basic** | 250 MB - 53 GB | Single node, no SLA | Dev/test |
| **Standard** | 250 MB - 53 GB | Primary-replica, 99.9% SLA | Production |
| **Premium** | 6 - 120 GB | Persistence, clustering, VNet, geo-replication | High performance |
| **Enterprise** | 12 - 100 GB | RediSearch, RedisBloom, RedisTimeSeries | Advanced modules |
| **Enterprise Flash** | 300 GB - 4.5 TB | NVMe flash + DRAM | Large datasets |

---

## 4.4 Creating Azure Cache for Redis

```bash
# Create Redis Cache (Standard tier)
az redis create \
  --name mydevopsredis \
  --resource-group myRG \
  --location eastus \
  --sku Standard \
  --vm-size C1 \
  --enable-non-ssl-port false \
  --minimum-tls-version 1.2

# Get connection info
az redis show \
  --name mydevopsredis \
  --resource-group myRG \
  --query "{hostName:hostName, port:sslPort, provisioningState:provisioningState}" \
  --output table

# Get access keys
az redis list-keys \
  --name mydevopsredis \
  --resource-group myRG

# Regenerate key
az redis regenerate-keys \
  --name mydevopsredis \
  --resource-group myRG \
  --key-type Primary
```

---

## 4.5 Connecting to Redis

```python
# Python example using redis-py
import redis

r = redis.StrictRedis(
    host='mydevopsredis.redis.cache.windows.net',
    port=6380,
    password='<access-key>',
    ssl=True,
    decode_responses=True
)

# Set a value (with 1-hour expiry)
r.setex('user:1001:name', 3600, 'John Doe')

# Get a value
name = r.get('user:1001:name')
print(name)  # "John Doe"

# Cache-aside pattern
def get_user(user_id):
    cache_key = f'user:{user_id}'
    
    # Check cache first
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Cache miss — query database
    user = database.query(f"SELECT * FROM users WHERE id = {user_id}")
    
    # Store in cache with 1-hour TTL
    r.setex(cache_key, 3600, json.dumps(user))
    
    return user
```

```bash
# Connect via redis-cli (for testing)
redis-cli -h mydevopsredis.redis.cache.windows.net \
  -p 6380 \
  -a '<access-key>' \
  --tls

# Basic Redis commands
SET greeting "Hello DevOps"
GET greeting
DEL greeting
EXPIRE mykey 3600
TTL mykey
KEYS user:*
```

---

## 4.6 Caching Patterns

```
┌────── CACHING PATTERNS ──────────────────────┐
│                                               │
│  1. CACHE-ASIDE (most common)                 │
│     App checks cache → miss → reads DB        │
│     → stores in cache → returns data          │
│                                               │
│  2. WRITE-THROUGH                             │
│     App writes to cache AND DB simultaneously │
│     Cache always has latest data              │
│                                               │
│  3. WRITE-BEHIND                              │
│     App writes to cache → cache async writes  │
│     to DB → faster writes, eventual consistency│
│                                               │
│  4. READ-THROUGH                              │
│     Cache automatically loads from DB on miss │
│     App only talks to cache                   │
│                                               │
└───────────────────────────────────────────────┘
```

---

## 4.7 Redis Clustering (Premium)

```
┌────── REDIS CLUSTER (PREMIUM) ──────────────┐
│                                              │
│   Data is sharded across multiple nodes      │
│                                              │
│   ┌────────┐  ┌────────┐  ┌────────┐       │
│   │ Shard 0│  │ Shard 1│  │ Shard 2│       │
│   │ Keys   │  │ Keys   │  │ Keys   │       │
│   │ 0-5460 │  │5461-   │  │10923-  │       │
│   │        │  │10922   │  │16383   │       │
│   │Primary │  │Primary │  │Primary │       │
│   │+Replica│  │+Replica│  │+Replica│       │
│   └────────┘  └────────┘  └────────┘       │
│                                              │
│   Benefits:                                  │
│   • Higher throughput (up to 10 shards)      │
│   • Larger datasets (shard memory combined)  │
│   • Automatic data distribution              │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 4.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection timeout | Firewall or wrong port | Use port 6380 (SSL); check firewall rules |
| High latency | Cache too small, evictions | Scale up or reduce data volume |
| Data evicted unexpectedly | Memory pressure | Increase cache size or set shorter TTLs |
| Connection refused | Non-SSL port disabled | Use SSL (port 6380) or enable non-SSL port |
| Cache stampede | Many requests for same expired key | Use lock/mutex pattern or staggered TTLs |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **In-Memory** | Sub-millisecond latency, 10-100x faster than DB |
| **Tiers** | Basic (dev), Standard (prod), Premium (cluster), Enterprise |
| **Cache-Aside** | Most common pattern: check cache → miss → load from DB → cache |
| **TTL** | Set expiry on keys to prevent stale data |
| **SSL** | Port 6380 with TLS; non-SSL on 6379 (disable in prod) |
| **Clustering** | Premium tier: shard data across up to 10 nodes |

---

## Quick Revision Questions

1. **What is the cache-aside pattern?**
   > The app checks the cache first. On a miss, it queries the database, then stores the result in cache before returning.

2. **Which Redis port should you use in production?**
   > Port 6380 with SSL/TLS enabled. Never use non-SSL port 6379 in production.

3. **What tier supports clustering?**
   > Premium tier supports Redis clustering with up to 10 shards for higher throughput and capacity.

4. **How do you prevent stale data in the cache?**
   > Set TTL (Time-to-Live) on cache keys so they automatically expire and get refreshed from the database.

5. **What is a cache stampede?**
   > When a popular cache key expires and many simultaneous requests hit the database. Mitigate with lock patterns or staggered TTLs.

---

[⬅ Previous: MySQL and PostgreSQL](03-mysql-and-postgresql.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Unit 7 - Azure Container Registry ➡](../07-Container-Services/01-azure-container-registry.md)
