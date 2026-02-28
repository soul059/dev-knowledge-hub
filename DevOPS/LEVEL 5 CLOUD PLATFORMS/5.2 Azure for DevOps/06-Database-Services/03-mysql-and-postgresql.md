# Chapter 3: Azure Database for MySQL and PostgreSQL

## Overview
Azure offers fully managed database services for two popular open-source engines: **MySQL** and **PostgreSQL**. Both provide built-in HA, automated backups, security, and scaling — without the overhead of managing VMs or database installations.

---

## 3.1 Service Architecture

```
┌──────── MANAGED OPEN-SOURCE DATABASES ──────────┐
│                                                   │
│   ┌──────────────────────────────────────────┐    │
│   │  Azure Database for MySQL                │    │
│   │  Flexible Server (recommended)           │    │
│   │                                          │    │
│   │  • MySQL 5.7, 8.0                        │    │
│   │  • Zone-redundant HA                     │    │
│   │  • VNet integration                      │    │
│   │  • Stop/Start for cost savings           │    │
│   └──────────────────────────────────────────┘    │
│                                                   │
│   ┌──────────────────────────────────────────┐    │
│   │  Azure Database for PostgreSQL           │    │
│   │  Flexible Server (recommended)           │    │
│   │                                          │    │
│   │  • PostgreSQL 13, 14, 15, 16             │    │
│   │  • Zone-redundant HA                     │    │
│   │  • Extensions support (PostGIS, etc.)    │    │
│   │  • Stop/Start for cost savings           │    │
│   └──────────────────────────────────────────┘    │
│                                                   │
│   ⚠️ "Single Server" is deprecated — use          │
│      "Flexible Server" for all new deployments    │
│                                                   │
└───────────────────────────────────────────────────┘
```

---

## 3.2 Flexible Server Tiers

| Tier | vCores | Use Case |
|------|--------|----------|
| **Burstable** | 1-20 | Dev/test, low-traffic apps |
| **General Purpose** | 2-96 | Production workloads |
| **Memory Optimized** | 2-96 | In-memory caching, analytics |

---

## 3.3 High Availability Options

```
┌────── HA ARCHITECTURE (FLEXIBLE SERVER) ────┐
│                                              │
│  SAME-ZONE HA:                               │
│  ┌───────────────────────────────┐           │
│  │ Availability Zone 1           │           │
│  │ ┌──────────┐  ┌──────────┐   │           │
│  │ │ Primary  │  │ Standby  │   │           │
│  │ │ Server   │──►│ Server   │   │           │
│  │ │          │  │ (sync)   │   │           │
│  │ └──────────┘  └──────────┘   │           │
│  └───────────────────────────────┘           │
│                                              │
│  ZONE-REDUNDANT HA:                          │
│  ┌──────────────┐  ┌──────────────┐          │
│  │ Zone 1       │  │ Zone 2       │          │
│  │ ┌──────────┐ │  │ ┌──────────┐ │          │
│  │ │ Primary  │ │  │ │ Standby  │ │          │
│  │ │ Server   │─┼──┼►│ Server   │ │          │
│  │ └──────────┘ │  │ └──────────┘ │          │
│  └──────────────┘  └──────────────┘          │
│  Automatic failover: ~60-120 seconds         │
│                                              │
└──────────────────────────────────────────────┘
```

---

## 3.4 Creating MySQL Flexible Server

```bash
# Create MySQL Flexible Server
az mysql flexible-server create \
  --resource-group myRG \
  --name mydevopsmysql \
  --location eastus \
  --admin-user mysqladmin \
  --admin-password 'P@ssw0rd123!' \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --version 8.0

# Create database
az mysql flexible-server db create \
  --resource-group myRG \
  --server-name mydevopsmysql \
  --database-name myappdb

# Configure firewall (allow specific IP)
az mysql flexible-server firewall-rule create \
  --resource-group myRG \
  --name mydevopsmysql \
  --rule-name AllowMyIP \
  --start-ip-address 203.0.113.5 \
  --end-ip-address 203.0.113.5

# Connect using mysql client
mysql -h mydevopsmysql.mysql.database.azure.com \
  -u mysqladmin \
  -p \
  --ssl-mode=REQUIRED

# Stop server (save costs when not in use)
az mysql flexible-server stop \
  --resource-group myRG \
  --name mydevopsmysql

# Start server
az mysql flexible-server start \
  --resource-group myRG \
  --name mydevopsmysql
```

---

## 3.5 Creating PostgreSQL Flexible Server

```bash
# Create PostgreSQL Flexible Server
az postgres flexible-server create \
  --resource-group myRG \
  --name mydevopspostgres \
  --location eastus \
  --admin-user pgadmin \
  --admin-password 'P@ssw0rd123!' \
  --sku-name Standard_B1ms \
  --tier Burstable \
  --storage-size 32 \
  --version 16

# Create database
az postgres flexible-server db create \
  --resource-group myRG \
  --server-name mydevopspostgres \
  --database-name myappdb

# Enable extension (e.g., PostGIS for geospatial)
az postgres flexible-server parameter set \
  --resource-group myRG \
  --server-name mydevopspostgres \
  --name azure.extensions \
  --value "POSTGIS,UUID-OSSP"

# Connect using psql
psql "host=mydevopspostgres.postgres.database.azure.com \
  port=5432 dbname=myappdb user=pgadmin sslmode=require"
```

---

## 3.6 MySQL vs PostgreSQL Comparison

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| **SQL Compliance** | Partial | More complete |
| **JSON Support** | Basic | Advanced (JSONB) |
| **Extensions** | Limited | Rich (PostGIS, pg_stat, etc.) |
| **Replication** | Read replicas | Read replicas + logical replication |
| **Best For** | Web apps, CMS (WordPress) | Complex queries, geospatial, analytics |
| **Popularity** | Very high (LAMP stack) | Growing rapidly |

---

## 3.7 Backup and Restore

```bash
# Backups are automatic:
# • Full: daily
# • Incremental: continuous
# • Retention: 7-35 days (default: 7)

# Configure backup retention
az mysql flexible-server update \
  --resource-group myRG \
  --name mydevopsmysql \
  --backup-retention 14

# Point-in-time restore
az mysql flexible-server restore \
  --resource-group myRG \
  --name mydevopsmysql-restored \
  --source-server mydevopsmysql \
  --restore-time "2024-01-15T10:30:00Z"
```

---

## 3.8 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Cannot connect | Firewall blocking | Add client IP to firewall rules |
| SSL connection required | SSL enforced by default | Use `--ssl-mode=REQUIRED` in connection string |
| Server stopped | Stop was used for cost savings | Start the server |
| Slow queries | Missing indexes or under-provisioned | Check slow query log, scale up tier |
| Extension not available | Not allowlisted | Add extension to `azure.extensions` parameter (PostgreSQL) |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Flexible Server** | Recommended deployment; replaces Single Server (deprecated) |
| **Tiers** | Burstable, General Purpose, Memory Optimized |
| **HA Options** | Same-Zone or Zone-Redundant (automatic failover) |
| **Stop/Start** | Pause server to save costs during off-hours |
| **Backups** | Automatic daily + incremental, 7-35 day retention |
| **SSL** | Enforced by default for all connections |

---

## Quick Revision Questions

1. **What is Flexible Server?**
   > The recommended deployment option for both MySQL and PostgreSQL on Azure, offering zone-redundant HA, better cost control, and VNet integration.

2. **Which deployment option is deprecated?**
   > Single Server — all new deployments should use Flexible Server.

3. **What HA options are available?**
   > Same-zone HA (primary + standby in same zone) and zone-redundant HA (primary and standby in different availability zones).

4. **How do you save costs during non-business hours?**
   > Use the Stop/Start feature to pause the server when not in use.

5. **When would you choose PostgreSQL over MySQL?**
   > When you need advanced JSON support (JSONB), rich extensions (PostGIS), complex queries, or higher SQL standard compliance.

---

[⬅ Previous: Cosmos DB](02-cosmos-db.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Cache for Redis ➡](04-azure-redis-cache.md)
