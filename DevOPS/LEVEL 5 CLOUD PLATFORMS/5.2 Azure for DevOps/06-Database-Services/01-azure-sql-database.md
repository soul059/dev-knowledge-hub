# Chapter 1: Azure SQL Database

## Overview
**Azure SQL Database** is a fully managed relational database service based on SQL Server. It provides built-in high availability, automated backups, intelligent performance tuning, and elastic scaling — without managing the underlying infrastructure.

---

## 1.1 Azure SQL Family

```
┌──────────── AZURE SQL OPTIONS ──────────────────┐
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │  Azure SQL Database                          │  │
│  │  • Fully managed PaaS                        │  │
│  │  • Single DB or Elastic Pool                 │  │
│  │  • Best for cloud-native apps                │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │  Azure SQL Managed Instance                  │  │
│  │  • Near 100% SQL Server compatibility        │  │
│  │  • VNet integration                          │  │
│  │  • Best for lift-and-shift migrations        │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ┌─────────────────────────────────────────────┐  │
│  │  SQL Server on Azure VMs                     │  │
│  │  • Full SQL Server on IaaS                   │  │
│  │  • You manage OS + SQL Server                │  │
│  │  • Best for full control / legacy apps       │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  Management ◄───────────────────────► Control     │
│  (PaaS)         Managed Instance       (IaaS)     │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 1.2 Purchasing Models

| Model | Description | Best For |
|-------|-------------|----------|
| **DTU (Database Transaction Unit)** | Bundled compute + storage | Simple, predictable workloads |
| **vCore** | Independently scale compute & storage | Flexible, migration from on-prem |
| **Serverless** | Auto-pause & auto-scale | Intermittent, unpredictable usage |

### DTU Tiers

| Tier | DTUs | Storage | Use Case |
|------|------|---------|----------|
| Basic | 5 | 2 GB | Dev/test |
| Standard | 10-3000 | 1 TB | General production |
| Premium | 125-4000 | 4 TB | IO-intensive, OLTP |

### vCore Tiers

| Tier | Description | Use Case |
|------|-------------|----------|
| **General Purpose** | Balanced compute/memory | Most production workloads |
| **Business Critical** | High IOPS, built-in HA replicas | Mission-critical applications |
| **Hyperscale** | Up to 100 TB, rapid scale-out | Very large databases |

---

## 1.3 Elastic Pools

```
┌────── ELASTIC POOLS ───────────────────────┐
│                                             │
│  WITHOUT Pool:     WITH Pool:               │
│  ┌────┐ ┌────┐    ┌────────────────────┐   │
│  │DB1 │ │DB2 │    │  Elastic Pool      │   │
│  │50  │ │50  │    │  100 eDTUs shared  │   │
│  │DTU │ │DTU │    │                    │   │
│  └────┘ └────┘    │  ┌────┐ ┌────┐    │   │
│  Total: 100 DTU   │  │DB1 │ │DB2 │    │   │
│  (often idle)      │  │ 80 │ │ 20 │    │   │
│                    │  │    │ │    │    │   │
│  Cost: $$$$        │  └────┘ └────┘    │   │
│                    └────────────────────┘   │
│                    Cost: $$                  │
│                    (databases share pool)    │
│                                             │
│  Best for: SaaS apps with many DBs that     │
│  have unpredictable, variable usage          │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 1.4 Creating Azure SQL Database

```bash
# Create SQL Server (logical server)
az sql server create \
  --name mydevopssqlserver \
  --resource-group myRG \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'P@ssw0rd123!'

# Create database (vCore, General Purpose)
az sql db create \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name myAppDB \
  --edition GeneralPurpose \
  --family Gen5 \
  --capacity 2 \
  --max-size 32GB

# Create database (DTU-based)
az sql db create \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name myDevDB \
  --edition Standard \
  --capacity 50

# Create serverless database
az sql db create \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name myServerlessDB \
  --edition GeneralPurpose \
  --family Gen5 \
  --compute-model Serverless \
  --min-capacity 0.5 \
  --max-capacity 4 \
  --auto-pause-delay 60

# Configure firewall rule (allow Azure services)
az sql server firewall-rule create \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Configure firewall (allow specific IP)
az sql server firewall-rule create \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name AllowMyIP \
  --start-ip-address 203.0.113.5 \
  --end-ip-address 203.0.113.5
```

---

## 1.5 High Availability

```
┌────── HIGH AVAILABILITY OPTIONS ────────────┐
│                                              │
│  GENERAL PURPOSE:                            │
│  ┌──────────┐    ┌──────────────┐            │
│  │ Compute  │    │ Azure Storage│            │
│  │ (single) │───►│ (replicated) │            │
│  └──────────┘    └──────────────┘            │
│  • Storage-level redundancy                  │
│  • ~30s failover time                        │
│                                              │
│  BUSINESS CRITICAL:                          │
│  ┌────────┐  ┌────────┐  ┌────────┐         │
│  │Primary │  │Replica │  │Replica │         │
│  │  ✅    │  │  Sync  │  │  Sync  │         │
│  │        │──►│        │──►│        │         │
│  └────────┘  └────────┘  └────────┘         │
│  • Always On Availability Groups             │
│  • <30s failover, read replicas              │
│  • Free readable secondary                   │
│                                              │
│  HYPERSCALE:                                 │
│  • Up to 4 read replicas                     │
│  • Named replicas for workload isolation     │
│  • Near-instant backups (snapshots)          │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 1.6 Backup and Restore

```bash
# Backups are AUTOMATIC:
# • Full backup: weekly
# • Differential: every 12-24 hours
# • Transaction log: every 5-10 minutes
#
# Retention:
# • Point-in-time restore: 7-35 days (default: 7)
# • Long-term retention (LTR): up to 10 years

# Configure short-term retention
az sql db str-policy set \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name myAppDB \
  --retention-days 14

# Restore to point in time
az sql db restore \
  --resource-group myRG \
  --server mydevopssqlserver \
  --name myAppDB-restored \
  --dest-name myAppDB-restored \
  --time "2024-01-15T10:30:00Z"
```

---

## 1.7 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot connect | Firewall blocking | Add client IP or Azure services to firewall |
| DTU exhausted | Workload exceeds tier | Scale up DTUs or switch to vCore |
| Slow queries | Missing indexes, bad plans | Use Performance Insights, add indexes |
| Serverless cold start | DB auto-paused | Increase auto-pause delay or use provisioned |
| Login failed | Wrong credentials or Azure AD | Verify admin credentials and auth method |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **SQL Database** | Fully managed PaaS, best for cloud-native |
| **Managed Instance** | Near-full SQL Server compatibility, lift-and-shift |
| **DTU** | Bundled compute model, simple pricing |
| **vCore** | Flexible compute/storage, migration-friendly |
| **Serverless** | Auto-pause, auto-scale, pay per actual usage |
| **Elastic Pool** | Share resources across multiple databases |
| **Backups** | Automatic: full weekly, differential daily, log every 5-10 min |

---

## Quick Revision Questions

1. **What are the three Azure SQL deployment options?**
   > Azure SQL Database (PaaS), SQL Managed Instance (near-full compatibility), SQL Server on VMs (IaaS).

2. **What is the difference between DTU and vCore models?**
   > DTU bundles compute and storage together; vCore allows independent scaling and is more flexible.

3. **What is an Elastic Pool?**
   > A pool of shared compute resources (eDTUs or vCores) across multiple databases to optimize costs for variable workloads.

4. **What is the serverless compute model?**
   > Auto-scales compute based on demand and auto-pauses when idle — you pay only for actual usage.

5. **How does automatic backup work?**
   > Azure automatically takes full (weekly), differential (12-24h), and transaction log (5-10 min) backups with 7-35 day retention.

---

[⬅ Previous: Storage Tiers](../05-Storage-Services/05-storage-tiers.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Cosmos DB ➡](02-cosmos-db.md)
