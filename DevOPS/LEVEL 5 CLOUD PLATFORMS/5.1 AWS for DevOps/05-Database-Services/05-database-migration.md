# Chapter 5.5: Database Migration (DMS & SCT)

## Overview

**AWS Database Migration Service (DMS)** enables you to migrate databases to AWS with minimal downtime. Combined with the **Schema Conversion Tool (SCT)**, it supports both homogeneous migrations (same engine) and heterogeneous migrations (different engines).

---

## Migration Types

```
┌──── Homogeneous Migration (Same Engine) ───────────────────┐
│                                                             │
│  On-Prem MySQL  ─────── DMS ──────►  Amazon RDS MySQL     │
│  On-Prem PostgreSQL ─── DMS ──────►  Amazon RDS PostgreSQL│
│  On-Prem Oracle ──────── DMS ──────►  Amazon RDS Oracle   │
│                                                             │
│  • No schema conversion needed                             │
│  • DMS handles data migration directly                     │
│  • Simplest migration path                                 │
└─────────────────────────────────────────────────────────────┘

┌──── Heterogeneous Migration (Different Engine) ────────────┐
│                                                             │
│  Step 1: Schema Conversion                                 │
│  On-Prem Oracle ──── SCT ──────► Aurora PostgreSQL Schema  │
│                                                             │
│  Step 2: Data Migration                                    │
│  On-Prem Oracle ──── DMS ──────► Aurora PostgreSQL Data    │
│                                                             │
│  • SCT converts schema, views, stored procedures           │
│  • DMS migrates the actual data                            │
│  • May require manual conversion for complex objects       │
└─────────────────────────────────────────────────────────────┘
```

---

## DMS Architecture

```
┌──────────────── DMS Architecture ──────────────────────────┐
│                                                             │
│  Source Database              DMS Replication              │
│  (On-Prem/Cloud)             Instance                      │
│  ┌─────────────┐          ┌──────────────┐                │
│  │ MySQL       │  ──────► │ EC2 Instance │                │
│  │ PostgreSQL  │  Source  │ running DMS  │                │
│  │ Oracle      │  Endpt   │              │                │
│  │ SQL Server  │          │ Tasks:       │                │
│  │ MongoDB     │          │ • Full load  │                │
│  │ S3          │          │ • CDC        │                │
│  └─────────────┘          │ • Full+CDC   │                │
│                            └──────┬───────┘                │
│                                   │ Target                 │
│  Target Database                  │ Endpoint               │
│  ┌─────────────┐                  │                        │
│  │ RDS         │  ◄──────────────┘                        │
│  │ Aurora      │                                           │
│  │ DynamoDB    │  Migration Types:                         │
│  │ S3          │  • Full Load: one-time bulk migration     │
│  │ Redshift    │  • CDC: ongoing change replication        │
│  │ OpenSearch  │  • Full Load + CDC: bulk + ongoing sync   │
│  └─────────────┘                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## Migration Workflow

```
┌──────── Step-by-Step Migration ────────────────────────────┐
│                                                             │
│  1. ASSESS                                                 │
│     └── Run SCT Assessment Report                          │
│         ├── Identify conversion complexity                 │
│         ├── Estimate manual effort                         │
│         └── Flag unsupported features                      │
│                                                             │
│  2. CONVERT SCHEMA (heterogeneous only)                    │
│     └── Use SCT to convert                                 │
│         ├── Tables, indexes, views                         │
│         ├── Stored procedures, functions                   │
│         └── Fix items SCT can't auto-convert               │
│                                                             │
│  3. MIGRATE DATA                                           │
│     └── Create DMS resources                               │
│         ├── Replication Instance (EC2)                      │
│         ├── Source Endpoint                                 │
│         ├── Target Endpoint                                │
│         └── Migration Task (Full Load + CDC)               │
│                                                             │
│  4. TEST                                                   │
│     └── Validate data integrity                            │
│         ├── Row counts comparison                          │
│         ├── Data validation task (DMS built-in)            │
│         └── Application testing on target                  │
│                                                             │
│  5. CUTOVER                                                │
│     └── Switch application to target                       │
│         ├── Stop writes to source                          │
│         ├── Wait for CDC to catch up (lag = 0)             │
│         ├── Update connection strings                      │
│         └── Verify application functionality               │
└─────────────────────────────────────────────────────────────┘
```

---

## DMS CLI Commands

```bash
# Create replication instance
aws dms create-replication-instance \
  --replication-instance-identifier prod-dms \
  --replication-instance-class dms.r5.xlarge \
  --allocated-storage 100 \
  --multi-az \
  --vpc-security-group-ids sg-dms

# Create source endpoint
aws dms create-endpoint \
  --endpoint-identifier source-mysql \
  --endpoint-type source \
  --engine-name mysql \
  --server-name db.onprem.company.com \
  --port 3306 \
  --username dms_user \
  --password 'SecurePassword'

# Create target endpoint
aws dms create-endpoint \
  --endpoint-identifier target-aurora \
  --endpoint-type target \
  --engine-name aurora \
  --server-name mydb.cluster-xxx.us-east-1.rds.amazonaws.com \
  --port 3306 \
  --username admin \
  --password 'SecurePassword'

# Create migration task (Full Load + CDC)
aws dms create-replication-task \
  --replication-task-identifier mysql-to-aurora \
  --source-endpoint-arn arn:aws:dms:...:endpoint:source-mysql \
  --target-endpoint-arn arn:aws:dms:...:endpoint:target-aurora \
  --replication-instance-arn arn:aws:dms:...:rep:prod-dms \
  --migration-type full-load-and-cdc \
  --table-mappings '{
    "rules": [{
      "rule-type": "selection",
      "rule-id": "1",
      "rule-name": "all-tables",
      "object-locator": {"schema-name": "myapp", "table-name": "%"},
      "rule-action": "include"
    }]
  }'

# Start task
aws dms start-replication-task \
  --replication-task-arn arn:aws:dms:...:task:mysql-to-aurora \
  --start-replication-task-type start-replication
```

---

## Supported Source → Target Paths

| Source | Target Options |
|--------|---------------|
| Oracle | RDS Oracle, Aurora PostgreSQL, RDS PostgreSQL, DynamoDB, S3 |
| SQL Server | RDS SQL Server, Aurora MySQL/PG, RDS MySQL/PG, S3 |
| MySQL | RDS MySQL, Aurora MySQL, S3, DynamoDB |
| PostgreSQL | RDS PostgreSQL, Aurora PostgreSQL, S3 |
| MongoDB | DocumentDB, DynamoDB, S3 |
| S3 | RDS any, Aurora, DynamoDB, Redshift, OpenSearch |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Connection test fails | Network/SG blocking | Ensure DMS can reach source (VPN/DX) and target (VPC SG) |
| CDC not capturing changes | Binary logging not enabled (MySQL) | Enable `binlog_format=ROW` on source |
| Replication lag increasing | Insufficient DMS instance size | Scale up replication instance; optimize heavy tables |
| LOB column errors | Default LOB mode truncates large objects | Use `full-lob-mode` or `limited-lob-mode` with increased max size |
| Data validation mismatches | Schema differences or ongoing writes | Review validation report; recheck after CDC catches up |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **DMS** | Managed data migration. Full load, CDC, or both. Minimal downtime. |
| **SCT** | Schema Conversion Tool. Convert between different DB engines. |
| **Homogeneous** | Same engine migration (MySQL→MySQL). DMS only. |
| **Heterogeneous** | Different engines (Oracle→Aurora). SCT + DMS. |
| **CDC** | Change Data Capture. Ongoing replication after initial load. |
| **Cutover** | Wait for CDC lag = 0, switch app connections, verify. |

---

## Quick Revision Questions

1. **What is the difference between DMS and SCT?**
   > DMS migrates data between databases. SCT converts database schemas between different engines. For heterogeneous migrations, use SCT first, then DMS.

2. **What does CDC stand for and what does it do?**
   > Change Data Capture — it continuously replicates changes from the source to target after the initial full load, enabling near-zero-downtime migration.

3. **Do you need SCT for a MySQL-to-Aurora MySQL migration?**
   > No. This is a homogeneous migration (same engine family), so DMS alone handles it.

4. **What is the cutover process?**
   > Stop writes to source → wait for CDC replication lag to reach zero → switch application connection strings to target → verify.

5. **Can DMS migrate to non-relational targets?**
   > Yes. DMS can migrate to DynamoDB, S3, OpenSearch, Kinesis, and Redshift.

---

[← Previous: 5.4 ElastiCache](04-elasticache.md)

[← Back to README](../README.md)
