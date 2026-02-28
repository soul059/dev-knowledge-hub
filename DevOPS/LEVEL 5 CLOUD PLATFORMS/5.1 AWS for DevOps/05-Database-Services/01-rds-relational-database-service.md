# Chapter 5.1: Amazon RDS — Relational Database Service

## Overview

Amazon RDS is a **managed relational database** service supporting six engines. AWS handles provisioning, patching, backups, and failover — letting DevOps teams focus on schema design and application logic rather than database administration.

---

## RDS Architecture

```
┌──────────────── VPC ─────────────────────────────────────────┐
│                                                               │
│  ┌─── Private Subnet (AZ-1a) ────┐  ┌─── Private Subnet    │
│  │                                │  │     (AZ-1b) ────────┐│
│  │  ┌──── RDS Primary ────────┐  │  │  ┌── RDS Standby ──┐││
│  │  │  db.r6g.xlarge          │  │  │  │  Synchronous     │││
│  │  │  MySQL 8.0              │  │  │  │  Replication     │││
│  │  │                         │◄─┼──┼─►│                  │││
│  │  │  Endpoint:              │  │  │  │  Auto failover   │││
│  │  │  mydb.xxx.rds.amazonaws │  │  │  │  (< 60 seconds)  │││
│  │  │  Port: 3306             │  │  │  │                  │││
│  │  └─────────────────────────┘  │  │  └──────────────────┘││
│  │                                │  │                      ││
│  │  ┌──── Read Replica ───────┐  │  └──────────────────────┘│
│  │  │  Async replication      │  │                           │
│  │  │  Read-only endpoint     │  │                           │
│  │  │  Can be cross-region    │  │                           │
│  │  └─────────────────────────┘  │                           │
│  └────────────────────────────────┘                           │
│                                                               │
│  ┌── DB Subnet Group ────────────────────────────────────┐   │
│  │  subnet-1a (10.0.5.0/24) + subnet-1b (10.0.6.0/24)  │   │
│  │  Must span at least 2 AZs for Multi-AZ               │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                               │
│  Security Group: sg-db (inbound 3306 from sg-app only)      │
└───────────────────────────────────────────────────────────────┘
```

---

## Supported Engines

| Engine | Versions | Max Storage | Use Case |
|--------|---------|------------|----------|
| **MySQL** | 5.7, 8.0 | 64 TB | Web apps, CMS |
| **PostgreSQL** | 13-16 | 64 TB | GIS, analytics, complex queries |
| **MariaDB** | 10.x | 64 TB | MySQL-compatible, open source |
| **Oracle** | 19c, 21c | 64 TB | Enterprise, legacy apps |
| **SQL Server** | 2019, 2022 | 16 TB | .NET apps, Windows workloads |
| **Aurora** | MySQL/PostgreSQL | 128 TB | High-performance (see Ch 5.2) |

---

## Creating RDS Instance

```bash
# Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name prod-db-subnets \
  --db-subnet-group-description "Production DB subnets" \
  --subnet-ids subnet-data-1a subnet-data-1b

# Create RDS instance with Multi-AZ
aws rds create-db-instance \
  --db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.xlarge \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password 'UseSecretsManager!' \
  --allocated-storage 100 \
  --max-allocated-storage 500 \
  --storage-type gp3 \
  --multi-az \
  --db-subnet-group-name prod-db-subnets \
  --vpc-security-group-ids sg-db \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "Mon:04:00-Mon:05:00" \
  --storage-encrypted \
  --kms-key-id alias/rds-key \
  --deletion-protection \
  --copy-tags-to-snapshot \
  --tags Key=Environment,Value=production

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier prod-mysql-read1 \
  --source-db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --availability-zone us-east-1b

# Cross-region read replica (for DR)
aws rds create-db-instance-read-replica \
  --db-instance-identifier prod-mysql-dr \
  --source-db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --region eu-west-1
```

---

## Multi-AZ vs Read Replicas

```
┌──── Multi-AZ (High Availability) ─────────────────────────┐
│                                                             │
│  Purpose: FAILOVER (not scaling)                           │
│                                                             │
│  Primary (AZ-1a)                Standby (AZ-1b)           │
│  ┌────────────┐   Synchronous  ┌────────────┐             │
│  │ Read/Write │───────────────►│ No traffic │             │
│  │ Endpoint   │   Replication  │ (standby)  │             │
│  └────────────┘                └────────────┘             │
│       │                              │                     │
│       └── If primary fails ──────────┘                     │
│           DNS endpoint flips to standby                    │
│           Failover: ~60 seconds                            │
└─────────────────────────────────────────────────────────────┘

┌──── Read Replicas (Scaling) ──────────────────────────────┐
│                                                             │
│  Purpose: READ SCALING (not failover)                      │
│                                                             │
│  Primary                Read Replica 1    Read Replica 2   │
│  ┌────────────┐        ┌────────────┐    ┌────────────┐   │
│  │ Read/Write │──Async►│ Read-only  │    │ Read-only  │   │
│  │            │──Async►│            │    │            │   │
│  │            │        │ Own        │    │ Cross-     │   │
│  │            │        │ endpoint   │    │ region     │   │
│  └────────────┘        └────────────┘    └────────────┘   │
│                                                             │
│  • Up to 15 read replicas (Aurora) / 5 (others)           │
│  • Async = eventual consistency                            │
│  • Can be promoted to standalone (manual)                  │
│  • Cross-region supported                                  │
└─────────────────────────────────────────────────────────────┘
```

| Feature | Multi-AZ | Read Replica |
|---------|---------|--------------|
| Purpose | High availability | Read scaling |
| Replication | Synchronous | Asynchronous |
| Reads from standby | No | Yes (separate endpoint) |
| Automatic failover | Yes (~60s) | No (manual promote) |
| Cross-region | No | Yes |
| Cost | ~2x (standby instance) | Per replica instance |

---

## Automated Backups & Snapshots

```bash
# Automated backups (configured at creation)
# Retention: 1-35 days (set to 7 in our example)
# Window: 03:00-04:00 UTC
# Point-in-time recovery: any second within retention period

# Restore to point in time
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier prod-mysql \
  --target-db-instance-identifier prod-mysql-restored \
  --restore-time "2024-01-15T12:30:00Z" \
  --db-instance-class db.r6g.xlarge

# Manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier prod-mysql \
  --db-snapshot-identifier prod-mysql-pre-migration

# Copy snapshot to another region
aws rds copy-db-snapshot \
  --source-db-snapshot-identifier arn:aws:rds:us-east-1:123456:snapshot:prod-mysql-pre-migration \
  --target-db-snapshot-identifier prod-mysql-dr-copy \
  --region eu-west-1

# List snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier prod-mysql \
  --query 'DBSnapshots[*].[DBSnapshotIdentifier,SnapshotCreateTime,Status]' \
  --output table
```

---

## RDS Parameter Groups

```bash
# Create custom parameter group
aws rds create-db-parameter-group \
  --db-parameter-group-name prod-mysql-params \
  --db-parameter-group-family mysql8.0 \
  --description "Production MySQL parameters"

# Modify parameters
aws rds modify-db-parameter-group \
  --db-parameter-group-name prod-mysql-params \
  --parameters \
    "ParameterName=max_connections,ParameterValue=500,ApplyMethod=immediate" \
    "ParameterName=slow_query_log,ParameterValue=1,ApplyMethod=immediate" \
    "ParameterName=long_query_time,ParameterValue=2,ApplyMethod=immediate"

# Apply to instance
aws rds modify-db-instance \
  --db-instance-identifier prod-mysql \
  --db-parameter-group-name prod-mysql-params \
  --apply-immediately
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Can't connect to RDS | SG not allowing app SG, or wrong subnet | Check SG rules; ensure DB is in private subnet |
| Slow queries | Missing indexes, small instance | Enable slow query log; right-size instance class |
| Failover taking long | Large transaction rollback | Optimize write patterns; use smaller transactions |
| Storage full | Auto-scaling not enabled | Set `--max-allocated-storage` for auto-scaling |
| Read replica lag | Heavy writes, slow network | Use larger replica instance; consider Aurora |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **RDS** | Managed relational DB. 6 engines. Automated backups, patching, failover. |
| **Multi-AZ** | Synchronous standby for HA. Auto failover ~60s. Not for read scaling. |
| **Read Replica** | Async read copies. Up to 5 (15 Aurora). Own endpoint. Can promote. |
| **Backups** | Automated (1-35 day retention) + manual snapshots. PITR supported. |
| **Subnet Group** | Must span ≥2 AZs. Use data/private subnets. |
| **Encryption** | At-rest (KMS) + in-transit (SSL). Enable at creation. |

---

## Quick Revision Questions

1. **What is the difference between Multi-AZ and Read Replicas?**
   > Multi-AZ provides HA via synchronous replication with auto failover. Read Replicas provide read scaling via async replication with separate endpoints.

2. **Can you read from a Multi-AZ standby instance?**
   > No. The standby is only for failover. For read scaling, use Read Replicas.

3. **What is the maximum backup retention period for RDS?**
   > 35 days for automated backups. Manual snapshots persist until you delete them.

4. **How does RDS storage auto-scaling work?**
   > Set `max-allocated-storage`. RDS auto-increases storage when free space drops below 10%, up to the max.

5. **Can you encrypt an existing unencrypted RDS instance?**
   > Not directly. Snapshot → copy with encryption → restore from encrypted snapshot.

---

[Next: 5.2 Amazon Aurora →](02-amazon-aurora.md)

[← Back to README](../README.md)
