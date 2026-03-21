# Unit 7: Cloud Data Security — Topic 6: Database Security

## Overview

**Database security** in cloud environments protects data at rest and in transit within managed database services. Cloud databases (RDS, Azure SQL, Cloud SQL) offer built-in security features including encryption, access controls, auditing, and network isolation. Understanding these features and their proper configuration is essential for securing the most valuable asset — data.

---

## 1. Cloud Database Services

```
MANAGED DATABASE SERVICES:

AWS:
  → RDS (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server)
  → Aurora (MySQL/PostgreSQL compatible)
  → DynamoDB (NoSQL)
  → Redshift (Data warehouse)
  → ElastiCache (Redis/Memcached)
  → DocumentDB (MongoDB compatible)

AZURE:
  → Azure SQL Database
  → Azure Database for MySQL/PostgreSQL/MariaDB
  → Cosmos DB (NoSQL)
  → Azure Synapse Analytics
  → Azure Cache for Redis
  → SQL Managed Instance

GCP:
  → Cloud SQL (MySQL, PostgreSQL, SQL Server)
  → Cloud Spanner (Globally distributed)
  → Bigtable (NoSQL)
  → BigQuery (Data warehouse)
  → Memorystore (Redis/Memcached)
  → Firestore (NoSQL)

SHARED RESPONSIBILITY:
  Cloud Provider Manages:
  → Physical security
  → OS patching
  → Database engine patching
  → High availability infrastructure
  → Backup infrastructure
  
  Customer Manages:
  → Database configuration
  → Access controls and IAM
  → Encryption key management
  → Network security (security groups)
  → Application-level security
  → Data classification
  → Backup strategy and retention
```

---

## 2. Database Security Controls

```
ENCRYPTION:

  AT REST:
  AWS RDS:
  → Enable at creation (cannot change after)
  → AES-256 encryption
  → KMS key management
  → Encrypted snapshots
  
  Azure SQL:
  → TDE (Transparent Data Encryption) — default ON
  → Service-managed or customer-managed key
  → Always Encrypted (column-level, client-side)
  
  GCP Cloud SQL:
  → Encrypted by default
  → CMEK via Cloud KMS
  → Customer-managed encryption keys

  IN TRANSIT:
  → SSL/TLS connections required
  → Certificate validation
  → Force SSL parameter
  
  AWS RDS:
  → rds.force_ssl=1 (PostgreSQL)
  → require_secure_transport=ON (MySQL)
  
  Azure SQL:
  → Minimum TLS version configuration
  → Encrypted connections default
  
  GCP Cloud SQL:
  → --require-ssl flag
  → Server CA certificate verification

NETWORK ISOLATION:
  → Private subnets (no public IP)
  → Security groups restricting access
  → No direct internet access
  → Access through bastion or VPN
  → Private endpoints for PaaS access
  
  Best Practice Architecture:
  Application → Private Subnet → DB (Private Subnet)
  Admin → VPN → Bastion → DB
  
  # AWS: Disable public access
  aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --no-publicly-accessible
  
  # Azure: Deny public access
  az sql server update \
    --name myserver -g myRG \
    --enable-public-network false
  
  # GCP: No public IP
  gcloud sql instances patch mydb \
    --no-assign-ip
```

---

## 3. Access Controls

```
DATABASE ACCESS MANAGEMENT:

IAM-BASED AUTHENTICATION:
  AWS RDS:
  → IAM database authentication
  → Generate auth token via AWS API
  → Token-based login (15-min validity)
  → No password management
  
  # Enable IAM auth
  aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --enable-iam-database-authentication
  
  # Generate auth token
  aws rds generate-db-auth-token \
    --hostname mydb.xxx.us-east-1.rds.amazonaws.com \
    --port 3306 --username iam_user

  Azure SQL:
  → Azure AD authentication
  → Set AD admin for server
  → Token-based login
  → Managed identity support
  
  GCP Cloud SQL:
  → IAM database authentication
  → Automatic credential management
  → Service account authentication

DATABASE-LEVEL CONTROLS:
  → Principle of least privilege
  → Application-specific users
  → Read-only users for reporting
  → No shared accounts
  → Strong password requirements
  → No default/admin accounts for applications
  
  Best Practice:
  User: app_user → Permissions: SELECT, INSERT, UPDATE on app_db
  User: report_user → Permissions: SELECT on app_db
  User: admin_user → Permissions: ALL (via PAM/JIT only)
  User: root → Disabled or strong password + MFA

DYNAMIC DATA MASKING:
  Azure SQL:
  → Mask sensitive columns
  → Partial masking, random, custom
  → Transparently applied to non-privileged users
  
  Example:
  → Email: aXXX@XXXX.com
  → Credit card: XXXX-XXXX-XXXX-1234
  → SSN: XXX-XX-6789
```

---

## 4. Auditing and Monitoring

```
DATABASE AUDITING:

AWS RDS:
  → Enhanced Monitoring (OS metrics)
  → Performance Insights (query analysis)
  → CloudWatch Logs (database logs)
  → Audit plugin (MySQL/MariaDB)
  → pgAudit extension (PostgreSQL)
  → CloudTrail for API operations
  
  # Enable audit log
  aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --cloudwatch-logs-exports '["audit","error","general","slowquery"]'

AZURE SQL:
  → SQL Auditing (to Log Analytics, Storage, Event Hub)
  → Advanced Threat Protection
  → Vulnerability Assessment
  → Data Discovery & Classification
  
  Threat Detection Alerts:
  → SQL injection attempt
  → Anomalous database access
  → Potential brute force attack
  → Access from unusual location
  → Access from suspicious application
  
  # Enable auditing
  az sql server audit-policy update \
    --server myserver -g myRG \
    --state Enabled \
    --storage-account mystorageaccount

GCP CLOUD SQL:
  → Cloud Audit Logs (API operations)
  → Query Insights (performance)
  → Database flags for logging
  → PostgreSQL: log_connections, log_disconnections
  → MySQL: general_log, slow_query_log
  → Integration with Cloud Monitoring

KEY EVENTS TO MONITOR:
  → Failed login attempts (brute force)
  → Privilege escalation
  → Schema changes (DDL)
  → Data export operations
  → Large query results
  → Access from new IPs/locations
  → After-hours access
  → Admin operations
```

---

## 5. Assessment

```
DATABASE SECURITY ASSESSMENT:

CHECKLIST:
  [ ] Encryption at rest enabled
  [ ] SSL/TLS connections enforced
  [ ] No public IP / no public access
  [ ] Security groups restrict to app subnets
  [ ] IAM authentication enabled
  [ ] Application uses least privilege accounts
  [ ] No default passwords
  [ ] Audit logging enabled
  [ ] Threat detection active
  [ ] Automated backups configured
  [ ] Backup encryption enabled
  [ ] Version is current (patched)
  [ ] Parameter groups hardened
  [ ] Multi-AZ for production

COMMON FINDINGS:
  Finding                          | Risk
  Database publicly accessible     | Critical
  No encryption at rest            | Critical
  SSL not enforced                 | High
  Default admin password           | Critical
  No audit logging                 | High
  Application uses admin account   | High
  Security group allows 0.0.0.0/0  | Critical
  No automated backups             | High
  Unpatched database version       | Medium
  No threat detection              | Medium

TESTING:
  # Check connectivity from internet
  nmap -sV -p 3306,5432,1433,27017 <public-ip>
  
  # Check SSL
  openssl s_client -connect <db-host>:3306 -starttls mysql
  
  # List databases (if authenticated)
  mysql -h <host> -u <user> -p -e "SHOW DATABASES;"
  
  # Check permissions
  mysql -e "SELECT user, host, authentication_string FROM mysql.user;"

TOOLS:
  → dbsake (MySQL security assessment)
  → pgsafecheck (PostgreSQL security)
  → SQLMap (SQL injection testing)
  → Database-specific CIS benchmarks
  → Cloud-native threat detection
  → ScoutSuite (multi-cloud audit)
```

---

## Summary Table

| Feature | AWS RDS | Azure SQL | GCP Cloud SQL |
|---------|---------|-----------|--------------|
| Encryption at rest | KMS (enable at creation) | TDE (default ON) | Default (CMEK optional) |
| Force SSL | Parameter group | Server setting | Instance flag |
| IAM Auth | IAM tokens | Azure AD | IAM auth |
| Audit | CloudWatch + plugins | SQL Auditing | Audit Logs + flags |
| Threat Detection | GuardDuty | ATP | SCC |

---

## Revision Questions

1. What is the shared responsibility model for cloud database security?
2. How does IAM-based database authentication work?
3. Why should databases never have public IP addresses?
4. What database events should be monitored for security?
5. What common findings indicate poor database security?

---

*Previous: [05-data-loss-prevention.md](05-data-loss-prevention.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
