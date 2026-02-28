# Chapter 5.3: Amazon DynamoDB

## Overview

DynamoDB is a **fully managed NoSQL key-value and document database** that delivers single-digit millisecond performance at any scale. It's serverless — no instances to manage, no patching, and capacity can auto-scale to handle millions of requests per second.

---

## DynamoDB Data Model

```
┌──────────── DynamoDB Table: Orders ─────────────────────────┐
│                                                              │
│  Primary Key:                                                │
│  ┌───────────────────────────────────────────────────┐      │
│  │  Partition Key (PK)  │  Sort Key (SK)             │      │
│  │  customer_id          │  order_date#order_id       │      │
│  └───────────────────────────────────────────────────┘      │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ PK: CUST#123   SK: 2024-01-15#ORD001                 │  │
│  │ { amount: 99.99, status: "shipped", items: [...] }    │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │ PK: CUST#123   SK: 2024-01-20#ORD002                 │  │
│  │ { amount: 45.00, status: "pending", items: [...] }    │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │ PK: CUST#456   SK: 2024-01-18#ORD003                 │  │
│  │ { amount: 200.00, status: "delivered", items: [...] } │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Partition Key: Determines which partition stores the data   │
│  Sort Key:      Orders items within a partition              │
│  Together:      They form a unique primary key               │
└──────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Table** | Collection of items (rows) |
| **Item** | A single record (max 400 KB) |
| **Attribute** | A column/field (schemaless — each item can differ) |
| **Partition Key** | Hash key. Determines data distribution. Must be in every query. |
| **Sort Key** | Range key. Optional. Enables range queries within a partition. |
| **GSI** | Global Secondary Index. Different PK/SK. Eventually consistent. |
| **LSI** | Local Secondary Index. Same PK, different SK. Created at table creation only. |

---

## Capacity Modes

```
┌──── On-Demand Mode ────────────────────┐  ┌──── Provisioned Mode ────────────────┐
│                                         │  │                                       │
│  • Pay per request                     │  │  • Set Read/Write Capacity Units      │
│  • No capacity planning                │  │  • RCU: 1 strongly consistent read/s  │
│  •   auto-scales to millions/s         │  │         (4 KB per RCU)                │
│  • Higher per-request cost             │  │  • WCU: 1 write/s (1 KB per WCU)     │
│  • Good for: unpredictable workloads   │  │  • Auto Scaling available             │
│  •                                     │  │  • Good for: predictable workloads    │
│  │  Cost: ~$1.25/million writes        │  │  • Cheaper at scale                   │
│  │        ~$0.25/million reads         │  │  • Reserved capacity: up to 77% off   │
└─────────────────────────────────────────┘  └───────────────────────────────────────┘
```

---

## DynamoDB CLI Operations

```bash
# Create table
aws dynamodb create-table \
  --table-name Orders \
  --attribute-definitions \
    AttributeName=customer_id,AttributeType=S \
    AttributeName=order_date_id,AttributeType=S \
  --key-schema \
    AttributeName=customer_id,KeyType=HASH \
    AttributeName=order_date_id,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST

# Put item
aws dynamodb put-item \
  --table-name Orders \
  --item '{
    "customer_id": {"S": "CUST#123"},
    "order_date_id": {"S": "2024-01-15#ORD001"},
    "amount": {"N": "99.99"},
    "status": {"S": "shipped"}
  }'

# Get item (exact key)
aws dynamodb get-item \
  --table-name Orders \
  --key '{
    "customer_id": {"S": "CUST#123"},
    "order_date_id": {"S": "2024-01-15#ORD001"}
  }' \
  --consistent-read

# Query (all orders for a customer, sorted)
aws dynamodb query \
  --table-name Orders \
  --key-condition-expression "customer_id = :cid AND begins_with(order_date_id, :prefix)" \
  --expression-attribute-values '{
    ":cid": {"S": "CUST#123"},
    ":prefix": {"S": "2024-01"}
  }'

# Update item
aws dynamodb update-item \
  --table-name Orders \
  --key '{
    "customer_id": {"S": "CUST#123"},
    "order_date_id": {"S": "2024-01-15#ORD001"}
  }' \
  --update-expression "SET #s = :status" \
  --expression-attribute-names '{"#s": "status"}' \
  --expression-attribute-values '{":status": {"S": "delivered"}}'

# Delete item
aws dynamodb delete-item \
  --table-name Orders \
  --key '{
    "customer_id": {"S": "CUST#123"},
    "order_date_id": {"S": "2024-01-15#ORD001"}
  }'
```

---

## DynamoDB Streams & Event-Driven

```
┌──── DynamoDB Streams ──────────────────────────────────────┐
│                                                             │
│  Table Change                                               │
│      │                                                      │
│      ▼                                                      │
│  DynamoDB Stream (ordered log of changes)                  │
│      │                                                      │
│      ├──► Lambda Function                                  │
│      │    • Send notification on order status change        │
│      │    • Aggregate analytics                             │
│      │    • Replicate to another table                      │
│      │                                                      │
│      └──► Kinesis Data Streams                             │
│           • Long-term processing                            │
│           • Multiple consumers                              │
│                                                             │
│  Stream records contain:                                    │
│  • KEYS_ONLY: Just the keys of modified items              │
│  • NEW_IMAGE: Complete new item                            │
│  • OLD_IMAGE: Complete old item                            │
│  • NEW_AND_OLD_IMAGES: Both (most useful for comparison)   │
│                                                             │
│  Retention: 24 hours                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## Global Tables (Multi-Region)

```
┌──── DynamoDB Global Tables ────────────────────────────────┐
│                                                             │
│  us-east-1              eu-west-1            ap-southeast-1│
│  ┌────────────┐        ┌────────────┐       ┌────────────┐│
│  │ Orders     │◄──────►│ Orders     │◄─────►│ Orders     ││
│  │ (replica)  │  Async │ (replica)  │ Async │ (replica)  ││
│  │ Read/Write │        │ Read/Write │       │ Read/Write ││
│  └────────────┘        └────────────┘       └────────────┘│
│                                                             │
│  • Active-active: read AND write from any region           │
│  • Eventual consistency across regions                     │
│  • Conflict resolution: last writer wins                   │
│  • Requires DynamoDB Streams enabled                       │
│  • Sub-second replication in most cases                    │
└─────────────────────────────────────────────────────────────┘
```

---

## DynamoDB Accelerator (DAX)

```
┌──── DAX — In-Memory Cache ─────────────────────────────────┐
│                                                              │
│  Application                                                │
│      │                                                      │
│      ▼                                                      │
│  ┌──── DAX Cluster ────────────┐                           │
│  │  ┌───────┐ ┌───────┐       │  Microsecond reads        │
│  │  │ Node  │ │ Node  │ ...   │  Drop-in replacement      │
│  │  └───────┘ └───────┘       │  Same API as DynamoDB     │
│  └────────────┬────────────────┘                           │
│               │ Cache miss                                 │
│               ▼                                            │
│  ┌──── DynamoDB Table ────────┐                            │
│  │  Orders                     │  Millisecond reads       │
│  └─────────────────────────────┘                           │
│                                                             │
│  Write-through: writes go to DynamoDB, then cached         │
│  TTL: configurable per-item cache expiration               │
│  Not for: write-heavy or strongly consistent workloads     │
└─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Throttling (ProvisionedThroughputExceeded) | Hot partition or insufficient capacity | Use better partition key distribution; switch to on-demand |
| High read costs | Scanning entire table | Use Query instead of Scan; create GSI for access patterns |
| Item too large | >400 KB limit | Store large data in S3, keep S3 key in DynamoDB |
| GSI throttling | GSI has less capacity than base table | Ensure GSI WCU/RCU matches or exceeds write patterns |
| Stale data | Eventually consistent reads | Use `--consistent-read` for GetItem/Query |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **DynamoDB** | Fully managed NoSQL. Key-value + document. Serverless. |
| **Primary Key** | Partition key (required) + Sort key (optional). Together = unique. |
| **On-Demand** | Pay per request. No capacity planning. Good for unpredictable. |
| **Provisioned** | Set RCU/WCU. Auto-scaling available. Cheaper at scale. |
| **Streams** | Change data capture. Triggers Lambda. 24-hour retention. |
| **Global Tables** | Active-active multi-region replication. Last writer wins. |
| **DAX** | In-memory cache. Microsecond reads. Same DynamoDB API. |

---

## Quick Revision Questions

1. **What is the maximum item size in DynamoDB?**
   > 400 KB. Store larger data in S3 and keep the S3 reference in DynamoDB.

2. **What is the difference between Query and Scan?**
   > Query reads items by primary key (efficient). Scan reads every item in the table (expensive, avoid in production).

3. **When should you use On-Demand vs Provisioned capacity?**
   > On-Demand for unpredictable/spiky workloads. Provisioned for steady, predictable workloads (with auto-scaling).

4. **How do Global Tables handle write conflicts?**
   > Last writer wins — the most recent write to any replica is the one that persists.

5. **What is DAX and when should you NOT use it?**
   > DAX is an in-memory cache for DynamoDB. Don't use it for write-heavy workloads or when strong consistency is required.

---

[← Previous: 5.2 Aurora](02-amazon-aurora.md) | [Next: 5.4 ElastiCache →](04-elasticache.md)

[← Back to README](../README.md)
