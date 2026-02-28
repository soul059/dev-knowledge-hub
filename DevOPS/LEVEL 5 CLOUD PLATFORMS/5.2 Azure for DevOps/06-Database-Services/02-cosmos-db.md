# Chapter 2: Azure Cosmos DB

## Overview
**Azure Cosmos DB** is a globally distributed, multi-model NoSQL database service. It offers single-digit millisecond latency, 99.999% availability, and turnkey global distribution with multiple consistency models.

---

## 2.1 Cosmos DB Architecture

```
┌──────────── COSMOS DB ARCHITECTURE ─────────────┐
│                                                   │
│   ┌─────────────────────────────────────────┐     │
│   │  Cosmos DB Account                       │     │
│   │                                          │     │
│   │  ┌──────────────────────────────────┐    │     │
│   │  │  Database: myAppDB               │    │     │
│   │  │                                  │    │     │
│   │  │  ┌────────────────────────────┐  │    │     │
│   │  │  │  Container: users          │  │    │     │
│   │  │  │  Partition Key: /userId    │  │    │     │
│   │  │  │  ┌──────┐ ┌──────┐        │  │    │     │
│   │  │  │  │Item 1│ │Item 2│ ...    │  │    │     │
│   │  │  │  │(JSON)│ │(JSON)│        │  │    │     │
│   │  │  │  └──────┘ └──────┘        │  │    │     │
│   │  │  └────────────────────────────┘  │    │     │
│   │  │                                  │    │     │
│   │  │  ┌────────────────────────────┐  │    │     │
│   │  │  │  Container: orders         │  │    │     │
│   │  │  │  Partition Key: /customerId│  │    │     │
│   │  │  └────────────────────────────┘  │    │     │
│   │  └──────────────────────────────────┘    │     │
│   └─────────────────────────────────────────┘     │
│                                                   │
│   Global Distribution:                             │
│   ┌────────┐  ┌────────┐  ┌────────┐              │
│   │East US │  │West EU │  │SE Asia │              │
│   │ Write  │──►│ Read   │──►│ Read   │              │
│   │ Region │  │ Replica│  │ Replica│              │
│   └────────┘  └────────┘  └────────┘              │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 2.2 API Models (Multi-Model)

| API | Data Model | Use Case |
|-----|-----------|----------|
| **NoSQL (Core)** | JSON documents | General purpose, most features |
| **MongoDB** | BSON documents | MongoDB app migration |
| **Cassandra** | Wide-column | Cassandra workload migration |
| **Gremlin** | Graph | Social networks, recommendations |
| **Table** | Key-value | Azure Table Storage replacement |
| **PostgreSQL** | Relational (distributed) | Distributed PostgreSQL |

> ✅ **NoSQL (Core) API** is recommended for new projects — it gets features first.

---

## 2.3 Consistency Levels

```
┌────── CONSISTENCY SPECTRUM ──────────────────┐
│                                               │
│  STRONG ◄──────────────────────► EVENTUAL     │
│  (Most consistent)        (Most available)    │
│                                               │
│  ┌─────────────┐                              │
│  │   STRONG    │  Linearizable reads          │
│  │             │  Always latest write          │
│  │             │  Higher latency               │
│  └─────────────┘                              │
│  ┌─────────────┐                              │
│  │  BOUNDED    │  Reads may lag by K versions  │
│  │  STALENESS  │  or T time interval           │
│  └─────────────┘                              │
│  ┌─────────────┐                              │
│  │   SESSION   │  ← DEFAULT ✅                 │
│  │   (Default) │  Consistent within a session  │
│  │             │  Monotonic reads/writes        │
│  └─────────────┘                              │
│  ┌─────────────┐                              │
│  │  CONSISTENT │  Read your own writes          │
│  │  PREFIX     │  Ordered, but may be stale     │
│  └─────────────┘                              │
│  ┌─────────────┐                              │
│  │  EVENTUAL   │  No ordering guarantee         │
│  │             │  Lowest latency, highest avail │
│  └─────────────┘                              │
│                                               │
└───────────────────────────────────────────────┘
```

---

## 2.4 Request Units (RUs)

| Operation | Approximate RU Cost |
|-----------|-------------------|
| Point read (1 KB item by ID + partition key) | **1 RU** |
| Write (1 KB item) | ~5 RUs |
| Query (returns 1 item) | ~3 RUs |
| Query (returns 100 items) | ~30 RUs |

```
┌────── THROUGHPUT MODES ──────────────────────┐
│                                               │
│  1. PROVISIONED THROUGHPUT                     │
│     • Set RU/s manually (e.g., 400 RU/s)      │
│     • Billed hourly for provisioned amount     │
│     • Autoscale option (100-max RU/s)          │
│                                               │
│  2. SERVERLESS                                 │
│     • Pay per RU consumed                      │
│     • No capacity planning                     │
│     • Best for dev/test, sporadic traffic      │
│     • Max 5,000 RU/s burst                     │
│                                               │
└───────────────────────────────────────────────┘
```

---

## 2.5 Creating Cosmos DB

```bash
# Create Cosmos DB account (NoSQL API)
az cosmosdb create \
  --name mydevopscosmosdb \
  --resource-group myRG \
  --default-consistency-level Session \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1 \
  --enable-automatic-failover true

# Create database
az cosmosdb sql database create \
  --account-name mydevopscosmosdb \
  --resource-group myRG \
  --name myAppDB

# Create container with partition key
az cosmosdb sql container create \
  --account-name mydevopscosmosdb \
  --resource-group myRG \
  --database-name myAppDB \
  --name users \
  --partition-key-path "/userId" \
  --throughput 400

# Create container with autoscale
az cosmosdb sql container create \
  --account-name mydevopscosmosdb \
  --resource-group myRG \
  --database-name myAppDB \
  --name orders \
  --partition-key-path "/customerId" \
  --max-throughput 4000

# Get connection string
az cosmosdb keys list \
  --name mydevopscosmosdb \
  --resource-group myRG \
  --type connection-strings \
  --output table
```

---

## 2.6 Partition Keys

```
┌────── PARTITION KEY SELECTION ──────────────┐
│                                              │
│  GOOD partition key:                         │
│  ✅ High cardinality (many distinct values)  │
│  ✅ Even distribution of data                │
│  ✅ Used in most queries                     │
│                                              │
│  BAD partition key:                          │
│  ❌ Low cardinality (few values like status) │
│  ❌ Skewed distribution (hot partition)      │
│  ❌ Never used in queries                    │
│                                              │
│  Examples:                                   │
│  Users collection:   /userId    ✅           │
│  Orders collection:  /customerId ✅          │
│  Logs collection:    /date       ❌ (hot)    │
│  Logs collection:    /deviceId   ✅          │
│                                              │
│  ⚠️ Cannot change partition key after        │
│     container creation!                      │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 2.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 429 Too Many Requests | RU/s exceeded | Increase throughput or enable autoscale |
| High RU charges | Cross-partition queries | Design queries to target single partition |
| Hot partition | Poor partition key choice | Redesign with higher-cardinality key |
| High latency reads | Reading from distant region | Add read replica in user's region |
| Cannot change partition key | Set at creation | Create new container with new key, migrate data |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Multi-model** | NoSQL, MongoDB, Cassandra, Gremlin, Table, PostgreSQL APIs |
| **Global Distribution** | Multi-region writes/reads, automatic failover |
| **Consistency** | 5 levels: Strong → Bounded → Session (default) → Prefix → Eventual |
| **Request Units** | 1 RU = 1 point read of 1 KB item |
| **Throughput** | Provisioned (fixed/autoscale) or Serverless |
| **Partition Key** | Must be high cardinality, evenly distributed, used in queries |

---

## Quick Revision Questions

1. **What is the default consistency level in Cosmos DB?**
   > Session consistency — provides monotonic reads, monotonic writes, and read-your-own-writes within a session.

2. **What is a Request Unit (RU)?**
   > A normalized measure of throughput cost. 1 RU = one point read (by id + partition key) of a 1 KB item.

3. **Why is partition key selection important?**
   > It determines data distribution and query performance. A bad key causes hot partitions and cross-partition queries.

4. **What APIs does Cosmos DB support?**
   > NoSQL (Core), MongoDB, Cassandra, Gremlin (graph), Table, and PostgreSQL.

5. **What is the difference between provisioned and serverless?**
   > Provisioned: you set RU/s capacity. Serverless: pay per RU consumed, no pre-provisioning needed.

---

[⬅ Previous: Azure SQL Database](01-azure-sql-database.md) | [⬆ Back to Table of Contents](../README.md) | [Next: MySQL and PostgreSQL ➡](03-mysql-and-postgresql.md)
