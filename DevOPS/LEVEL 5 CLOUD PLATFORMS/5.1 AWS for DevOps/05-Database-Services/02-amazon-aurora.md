# Chapter 5.2: Amazon Aurora

## Overview

Amazon Aurora is AWS's **cloud-native relational database** — compatible with MySQL and PostgreSQL but re-architected for the cloud. It delivers up to **5x MySQL** and **3x PostgreSQL** performance with the simplicity and cost-effectiveness of open-source databases.

---

## Aurora Architecture

```
┌──────────────── Aurora Cluster ──────────────────────────────┐
│                                                               │
│  ┌── Writer Instance ───────┐    ┌── Reader Instance ──────┐ │
│  │  db.r6g.2xlarge          │    │  db.r6g.xlarge          │ │
│  │  AZ-1a                   │    │  AZ-1b                  │ │
│  │  Cluster Endpoint:       │    │  Reader Endpoint:       │ │
│  │  mydb.cluster-xxx.rds..  │    │  mydb.cluster-ro-xxx.. │ │
│  │  (writes + reads)        │    │  (reads, load-balanced) │ │
│  └──────────┬───────────────┘    └──────────┬──────────────┘ │
│             │                               │                │
│             ▼                               ▼                │
│  ┌──── Shared Cluster Volume (distributed storage) ───────┐ │
│  │                                                         │ │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐    │ │
│  │  │Copy1│ │Copy2│ │Copy3│ │Copy4│ │Copy5│ │Copy6│    │ │
│  │  │AZ-a │ │AZ-a │ │AZ-b │ │AZ-b │ │AZ-c │ │AZ-c │    │ │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘    │ │
│  │                                                         │ │
│  │  • 6 copies across 3 AZs (automatic)                   │ │
│  │  • Self-healing (detects + repairs corruption)          │ │
│  │  • Auto-scales up to 128 TB                            │ │
│  │  • Writes need 4/6 copies (survives AZ + 1 failure)    │ │
│  │  • Reads need 3/6 copies                               │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

---

## Aurora vs Standard RDS

| Feature | Standard RDS | Aurora |
|---------|-------------|--------|
| Storage | EBS (single volume) | Distributed (6 copies, 3 AZs) |
| Max storage | 64 TB | 128 TB |
| Replication | Async (read replicas) | Shared storage (faster) |
| Read replicas | Up to 5 | Up to 15 |
| Failover time | ~60-120s | ~30s (faster) |
| Scaling storage | Manual or auto-scale EBS | Automatic (10 GB increments) |
| Performance | 1x baseline | 5x MySQL, 3x PostgreSQL |
| Cost | Lower | ~20% more (but often cheaper at scale) |
| Backtrack | Not available | Rewind DB to point in time (no restore needed) |

---

## Aurora Endpoints

```
┌──── Aurora Cluster Endpoints ──────────────────────────────┐
│                                                             │
│  1. Cluster Endpoint (Writer)                              │
│     mydb.cluster-xxx.us-east-1.rds.amazonaws.com           │
│     → Always points to current writer instance             │
│     → Use for: writes, DDL, transactions                   │
│                                                             │
│  2. Reader Endpoint (Load-Balanced)                        │
│     mydb.cluster-ro-xxx.us-east-1.rds.amazonaws.com        │
│     → Load balances across all read replicas               │
│     → Use for: read queries, reports, analytics            │
│                                                             │
│  3. Instance Endpoints                                     │
│     mydb-instance-1.xxx.us-east-1.rds.amazonaws.com        │
│     → Direct to specific instance                          │
│     → Use for: debugging, specific instance access         │
│                                                             │
│  4. Custom Endpoints                                       │
│     mydb-analytics.cluster-custom-xxx.us-east-1.rds...     │
│     → Group of instances you define                        │
│     → Use for: route analytics to larger instances         │
└─────────────────────────────────────────────────────────────┘
```

---

## Aurora Serverless v2

```
┌──── Aurora Serverless v2 ──────────────────────────────────┐
│                                                             │
│  Capacity (ACUs):                                          │
│                                                             │
│  32 ─────────────────── Max ──────────────────── ●        │
│     │                                                      │
│     │                ●●●●                                  │
│     │              ●●    ●●                                │
│     │            ●●        ●                               │
│     │          ●●           ●●                             │
│     │        ●●               ●●                           │
│  0.5 ── Min ●─────────────────── ● ──────────            │
│     │                                                      │
│     └──────────────── Time ─────────────────────►         │
│                                                             │
│  • Scales from 0.5 to 128 ACUs (Aurora Capacity Units)    │
│  • Each ACU ≈ 2 GB RAM                                    │
│  • Scales in increments of 0.5 ACU                        │
│  • Mix with provisioned instances in same cluster         │
│  • Pay per ACU-hour consumed                              │
│                                                             │
│  Best for:                                                 │
│  • Variable/unpredictable workloads                       │
│  • Dev/test (scales to near-zero)                         │
│  • Multi-tenant SaaS applications                         │
└─────────────────────────────────────────────────────────────┘
```

```bash
# Create Aurora Serverless v2 cluster
aws rds create-db-cluster \
  --db-cluster-identifier my-aurora-sv2 \
  --engine aurora-mysql \
  --engine-version 8.0.mysql_aurora.3.04.0 \
  --master-username admin \
  --master-user-password 'UseSecretsManager!' \
  --serverless-v2-scaling-configuration MinCapacity=0.5,MaxCapacity=16 \
  --storage-encrypted

# Add serverless v2 instance
aws rds create-db-instance \
  --db-instance-identifier my-aurora-sv2-instance-1 \
  --db-cluster-identifier my-aurora-sv2 \
  --db-instance-class db.serverless \
  --engine aurora-mysql
```

---

## Aurora Global Database

```
┌──── Primary Region (us-east-1) ────┐     ┌──── Secondary Region (eu-west-1) ─┐
│                                     │     │                                     │
│  Writer + Readers                   │     │  Read-only Replicas                │
│  ┌────────────┐ ┌────────────┐      │     │  ┌────────────┐ ┌────────────┐    │
│  │  Writer    │ │  Reader    │      │     │  │  Reader    │ │  Reader    │    │
│  └─────┬──────┘ └─────┬──────┘      │     │  └─────┬──────┘ └─────┬──────┘    │
│        │              │             │     │        │              │            │
│  ┌─────▼──────────────▼──────┐      │     │  ┌─────▼──────────────▼──────┐    │
│  │  Cluster Volume           │──────┼────►│  │  Cluster Volume (replica) │    │
│  │  (primary)                │ <1s  │     │  │  Read-only, ~1s lag       │    │
│  └───────────────────────────┘ rep. │     │  └───────────────────────────┘    │
└─────────────────────────────────────┘     └─────────────────────────────────────┘

  • Replication lag: < 1 second cross-region
  • Failover to secondary: ~1 minute (promoted to read/write)
  • Up to 5 secondary regions
  • Use case: Global reads, disaster recovery
```

---

## Aurora Features Summary

```bash
# Backtrack (rewind without restore — Aurora MySQL only)
aws rds backtrack-db-cluster \
  --db-cluster-identifier my-aurora \
  --backtrack-to "2024-01-15T12:00:00Z"

# Clone (instant, copy-on-write — no storage cost until changes)
aws rds restore-db-cluster-to-point-in-time \
  --source-db-cluster-identifier prod-aurora \
  --db-cluster-identifier test-aurora-clone \
  --restore-type copy-on-write \
  --use-latest-restorable-time

# Enable Performance Insights
aws rds modify-db-instance \
  --db-instance-identifier my-aurora-instance \
  --enable-performance-insights \
  --performance-insights-retention-period 7
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Writer failover | Instance failure or AZ issue | Aurora auto-promotes a reader (~30s); set failover priority |
| Reader endpoint returning stale data | Replica lag | Expected with async; use cluster endpoint for consistent reads |
| Serverless scaling too slow | Min ACU too low for spike | Increase `MinCapacity` to keep warm capacity |
| Storage growing but not shrinking | Aurora doesn't reclaim freed space | Use `ALTER TABLE OPTIMIZE` or recreate table |
| Global DB replication lag >1s | Heavy write load or network | Monitor, scale writer, check cross-region bandwidth |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Aurora** | Cloud-native, 5x MySQL perf, 6 copies across 3 AZs |
| **Storage** | Shared, auto-scaling to 128 TB, self-healing |
| **Endpoints** | Cluster (writer), Reader (load-balanced), Custom, Instance |
| **Serverless v2** | 0.5-128 ACUs, auto-scales, pay per use |
| **Global Database** | <1s cross-region replication, ~1 min failover |
| **Backtrack** | Rewind DB to past point without restore (MySQL only) |
| **Cloning** | Instant copy-on-write clone for testing |

---

## Quick Revision Questions

1. **How many copies of data does Aurora maintain?**
   > 6 copies across 3 Availability Zones, automatically.

2. **What is the Aurora Reader Endpoint?**
   > A load-balanced endpoint that distributes read queries across all read replicas in the cluster.

3. **How does Aurora Serverless v2 differ from provisioned Aurora?**
   > Serverless v2 auto-scales capacity (0.5-128 ACUs) based on demand, while provisioned uses fixed instance sizes.

4. **What is Aurora Backtrack?**
   > A feature (MySQL only) that rewinds the database to a specific point in time without creating a new cluster.

5. **How fast is Aurora Global Database cross-region replication?**
   > Typically under 1 second replication lag, with cross-region failover in about 1 minute.

---

[← Previous: 5.1 RDS](01-rds-relational-database-service.md) | [Next: 5.3 DynamoDB →](03-dynamodb.md)

[← Back to README](../README.md)
