# Chapter 8.4: Data Integrity

[← Previous: Disaster Recovery](03-disaster-recovery.md) | [Back to README](../README.md) | [Next: Service Architecture →](05-service-architecture.md)

---

## Overview

Data integrity ensures that data remains accurate, consistent, and trustworthy throughout its lifecycle. For SRE teams, data loss or corruption is often worse than downtime — you can recover from an outage, but corrupted data that goes undetected can cause irreversible damage. This chapter covers patterns, practices, and safeguards to protect the most valuable asset: your data.

---

## 1. Types of Data Integrity Threats

```
  ┌──────────────────────────────────────────────────────────┐
  │  DATA INTEGRITY THREATS                                  │
  │                                                          │
  │  ┌─────────────────────┐                                 │
  │  │ CORRUPTION          │ Data modified incorrectly       │
  │  │ ├─ Software bugs    │ (e.g., race condition writes)  │
  │  │ ├─ Hardware failure  │ (e.g., bit rot on disk)       │
  │  │ └─ Malicious attack  │ (e.g., SQL injection)        │
  │  └─────────────────────┘                                 │
  │                                                          │
  │  ┌─────────────────────┐                                 │
  │  │ LOSS                │ Data permanently deleted        │
  │  │ ├─ Accidental delete │ (e.g., DROP TABLE)           │
  │  │ ├─ Storage failure   │ (e.g., disk crash, no backup)│
  │  │ └─ Ransomware        │ (encrypted by attacker)      │
  │  └─────────────────────┘                                 │
  │                                                          │
  │  ┌─────────────────────┐                                 │
  │  │ INCONSISTENCY       │ Data disagrees across systems  │
  │  │ ├─ Replication lag   │ (replica behind primary)      │
  │  │ ├─ Split-brain       │ (two primaries diverge)      │
  │  │ └─ Partial failure   │ (half of transaction applied)│
  │  └─────────────────────┘                                 │
  │                                                          │
  │  ┌─────────────────────┐                                 │
  │  │ SILENT CORRUPTION   │ ← MOST DANGEROUS!             │
  │  │ Data is wrong but   │                                │
  │  │ no error is raised  │ Can propagate for days/weeks  │
  │  │ before detection    │ before anyone notices          │
  │  └─────────────────────┘                                 │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Defense in Depth for Data

```
  ┌──────────────────────────────────────────────────────────┐
  │  DATA PROTECTION LAYERS                                  │
  │                                                          │
  │  Layer 1: PREVENTION                                     │
  │  ├─ Input validation (reject bad data at the gate)      │
  │  ├─ Schema constraints (NOT NULL, UNIQUE, FK, CHECK)    │
  │  ├─ Transactions (ACID guarantees)                      │
  │  ├─ Least privilege (limit who can write/delete)        │
  │  └─ Immutable audit logs                                │
  │                                                          │
  │  Layer 2: DETECTION                                      │
  │  ├─ Checksums (verify data hasn't changed)              │
  │  ├─ Data validation pipelines (continuous checks)       │
  │  ├─ Anomaly detection (unusual patterns)                │
  │  ├─ Reconciliation jobs (compare source of truth)       │
  │  └─ Monitoring and alerting on data metrics             │
  │                                                          │
  │  Layer 3: RECOVERY                                       │
  │  ├─ Automated backups (tested regularly!)               │
  │  ├─ Point-in-time recovery (PITR)                       │
  │  ├─ Soft deletes (mark as deleted, don't remove)        │
  │  ├─ Event sourcing (rebuild state from events)          │
  │  └─ Data versioning (keep history of changes)           │
  └──────────────────────────────────────────────────────────┘
```

---

## 3. Database Constraints and Validation

```sql
-- Layer 1: Schema-level data protection

CREATE TABLE orders (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL REFERENCES users(id),
    status          VARCHAR(20) NOT NULL 
                    CHECK (status IN ('pending', 'confirmed', 
                           'shipped', 'delivered', 'cancelled')),
    total_amount    DECIMAL(10,2) NOT NULL CHECK (total_amount > 0),
    currency        CHAR(3) NOT NULL DEFAULT 'USD'
                    CHECK (currency ~ '^[A-Z]{3}$'),
    item_count      INT NOT NULL CHECK (item_count > 0),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Prevent future-dated orders
    CONSTRAINT valid_dates 
        CHECK (created_at <= NOW() + INTERVAL '1 minute'),
    
    -- Total must be reasonable
    CONSTRAINT reasonable_total 
        CHECK (total_amount < 1000000)
);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER orders_update_timestamp
    BEFORE UPDATE ON orders
    FOR EACH ROW EXECUTE FUNCTION update_timestamp();

-- Soft delete pattern (never hard delete)
ALTER TABLE orders ADD COLUMN deleted_at TIMESTAMPTZ;
CREATE INDEX idx_orders_active ON orders(id) 
    WHERE deleted_at IS NULL;
```

---

## 4. Data Validation Pipeline

```
  ┌──────────────────────────────────────────────────────────┐
  │  CONTINUOUS DATA VALIDATION                              │
  │                                                          │
  │  ┌──────────┐     ┌──────────┐     ┌──────────┐         │
  │  │ Source   │────▶│ Validate │────▶│ Alert    │         │
  │  │ Database │     │ Pipeline │     │ if Invalid│        │
  │  └──────────┘     └─────┬────┘     └──────────┘         │
  │                         │                                │
  │                    ┌────▼────┐                            │
  │                    │ Report  │                            │
  │                    │ Dashboard│                           │
  │                    └─────────┘                            │
  │                                                          │
  │  VALIDATION CHECKS:                                      │
  │  ┌───────────────────┬──────────────────────────────┐    │
  │  │ Check Type        │ Example                      │    │
  │  ├───────────────────┼──────────────────────────────┤    │
  │  │ Completeness      │ No NULL in required fields   │    │
  │  │ Uniqueness        │ No duplicate order IDs       │    │
  │  │ Referential       │ All user_ids exist in users  │    │
  │  │ Range             │ Amounts > 0, dates in range  │    │
  │  │ Format            │ Emails match regex pattern   │    │
  │  │ Cross-field       │ ship_date > order_date       │    │
  │  │ Aggregate         │ Daily totals ± 10% of normal │    │
  │  │ Cross-system      │ Orders DB = Payment records  │    │
  │  └───────────────────┴──────────────────────────────┘    │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Data validation job configuration
data_validation:
  schedule: "*/30 * * * *"    # Every 30 minutes
  
  checks:
    - name: "orphaned_orders"
      query: |
        SELECT COUNT(*) FROM orders o
        LEFT JOIN users u ON o.user_id = u.id
        WHERE u.id IS NULL AND o.deleted_at IS NULL
      threshold: 0
      severity: "critical"
      action: "page_oncall"
      
    - name: "negative_balances"
      query: |
        SELECT COUNT(*) FROM accounts
        WHERE balance < 0 AND account_type = 'savings'
      threshold: 0
      severity: "critical"
      action: "page_oncall"
      
    - name: "duplicate_transactions"
      query: |
        SELECT idempotency_key, COUNT(*) 
        FROM transactions
        WHERE created_at > NOW() - INTERVAL '1 hour'
        GROUP BY idempotency_key
        HAVING COUNT(*) > 1
      threshold: 0
      severity: "high"
      action: "alert_slack"
      
    - name: "daily_revenue_anomaly"
      query: |
        SELECT ABS(today.total - avg_30d.avg_total) / 
               avg_30d.avg_total AS deviation
        FROM (SELECT SUM(amount) as total FROM transactions 
              WHERE date = CURRENT_DATE) today,
             (SELECT AVG(daily_total) as avg_total FROM 
              daily_aggregates WHERE date > CURRENT_DATE - 30) avg_30d
      threshold: 0.5     # >50% deviation from 30-day average
      severity: "high"
      action: "alert_slack"
```

---

## 5. Idempotency

```
  ┌──────────────────────────────────────────────────────────┐
  │  IDEMPOTENCY: Same operation applied multiple times      │
  │  produces the same result.                               │
  │                                                          │
  │  WHY IT MATTERS:                                         │
  │  User clicks "Pay" → network timeout → retries          │
  │  WITHOUT idempotency: Charged twice! ($100 × 2)         │
  │  WITH idempotency:    Charged once ($100), 2nd = no-op  │
  │                                                          │
  │  IMPLEMENTATION:                                         │
  │                                                          │
  │  Client                     Server                       │
  │  ┌──────┐                   ┌──────┐                     │
  │  │ POST │  idempotency_key  │      │                     │
  │  │/pay  │──────────────────▶│Check:│                     │
  │  │      │  "pay_abc123"     │Key   │                     │
  │  └──────┘                   │seen? │                     │
  │                             ├──────┤                     │
  │                             │ NO:  │──▶ Process payment  │
  │                             │      │    Store result     │
  │                             │      │    with key         │
  │                             ├──────┤                     │
  │                             │ YES: │──▶ Return stored    │
  │                             │      │    result (no-op)   │
  │                             └──────┘                     │
  │                                                          │
  │  RULES:                                                  │
  │  ├─ Client generates unique key per logical operation   │
  │  ├─ Server stores key → result mapping                  │
  │  ├─ Same key = return cached result (don't re-process)  │
  │  ├─ Keys expire after reasonable time (24-48 hours)     │
  │  └─ Use database UNIQUE constraint on idempotency_key   │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Event Sourcing for Auditability

```
  ┌──────────────────────────────────────────────────────────┐
  │  TRADITIONAL vs EVENT SOURCING                           │
  │                                                          │
  │  TRADITIONAL (mutable state):                            │
  │  ┌─────────────────────────────────┐                     │
  │  │ accounts table                  │                     │
  │  │ user_id │ balance │ updated_at  │                     │
  │  │ U001    │ $850    │ 2024-03-15  │                     │
  │  └─────────────────────────────────┘                     │
  │  Q: How did balance get to $850? 🤷                     │
  │                                                          │
  │  EVENT SOURCING (immutable event log):                   │
  │  ┌──────────────────────────────────────────────┐        │
  │  │ events table (append-only, immutable)        │        │
  │  │ event_id │ type       │ amount │ balance     │        │
  │  │ E001     │ DEPOSIT    │ +1000  │ $1000       │        │
  │  │ E002     │ PURCHASE   │ -50    │ $950        │        │
  │  │ E003     │ PURCHASE   │ -75    │ $875        │        │
  │  │ E004     │ REFUND     │ +25    │ $900        │        │
  │  │ E005     │ PURCHASE   │ -50    │ $850        │        │
  │  └──────────────────────────────────────────────┘        │
  │  Q: How did balance get to $850? → Replay events!       │
  │                                                          │
  │  BENEFITS:                                               │
  │  ├─ Complete audit trail (who did what, when)           │
  │  ├─ Can rebuild state at any point in time              │
  │  ├─ Debug by replaying events                           │
  │  ├─ Detect corruption by re-computing state             │
  │  └─ Natural fit for financial / regulated systems       │
  │                                                          │
  │  COSTS:                                                  │
  │  ├─ Storage grows unbounded (need archival strategy)    │
  │  ├─ Event schema evolution is complex                   │
  │  ├─ Read performance: need materialized views/snapshots │
  │  └─ Complexity: significant learning curve              │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Checksums and Data Verification

```
  ┌──────────────────────────────────────────────────────────┐
  │  DATA VERIFICATION TECHNIQUES                            │
  │                                                          │
  │  1. ROW-LEVEL CHECKSUMS                                  │
  │     Store hash of critical fields with each row          │
  │     Detect silent corruption on read                     │
  │                                                          │
  │     checksum = SHA256(user_id + amount + timestamp)      │
  │     On read: recompute and compare                      │
  │                                                          │
  │  2. TABLE-LEVEL CHECKSUMS                                │
  │     Periodic job computes aggregate checksum             │
  │     Compare across replicas to detect drift              │
  │                                                          │
  │     Primary:  CHECKSUM TABLE orders = 0xA3F2B1          │
  │     Replica1: CHECKSUM TABLE orders = 0xA3F2B1 ✓       │
  │     Replica2: CHECKSUM TABLE orders = 0xC1D4E5 ✗ ALERT │
  │                                                          │
  │  3. RECONCILIATION JOBS                                  │
  │     Compare records between systems                     │
  │                                                          │
  │     Orders DB     ←compare→    Payment Gateway          │
  │     1000 orders                 1000 payments  ✓        │
  │     $50,000 total               $50,000 total  ✓        │
  │     Order #4521                 Missing!        ✗ ALERT │
  │                                                          │
  │  4. END-TO-END DATA INTEGRITY                           │
  │     Generate → Transport → Store → Read → Verify       │
  │     Checksum at each stage, compare at destination      │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Detecting and Recovering from Silent Data Corruption

```
  INCIDENT: Financial platform discovers account balances 
  are incorrect for ~2,000 users (out of 500K)
  
  DETECTION:
  Daily reconciliation job (comparing orders DB with 
  payment gateway) found $47,000 discrepancy.
  
  ┌──────────────────────────────────────────────────────────┐
  │  INVESTIGATION TIMELINE                                  │
  │                                                          │
  │  Day 1: Reconciliation alert fires                       │
  │  ├─ 2,147 accounts have incorrect balances              │
  │  ├─ All affected accounts processed between 3am-5am     │
  │  └─ Discrepancy started 8 days ago                      │
  │                                                          │
  │  Day 2: Root cause identified                            │
  │  ├─ Deploy on Day -8 introduced a race condition        │
  │  ├─ When 2 transactions hit simultaneously:             │
  │  │   Thread A: READ balance = $100                      │
  │  │   Thread B: READ balance = $100                      │
  │  │   Thread A: WRITE balance = $100 + $50 = $150       │
  │  │   Thread B: WRITE balance = $100 - $30 = $70        │
  │  │   Correct: $120   Actual: $70   Lost: $50           │
  │  └─ Only affected concurrent transactions (rare, ~0.4%) │
  │                                                          │
  │  RECOVERY:                                               │
  │  ├─ Event sourcing saved the day!                       │
  │  ├─ All transactions were recorded in immutable event   │
  │  │   log (even though balances were wrong)              │
  │  ├─ Replayed all events for affected accounts           │
  │  ├─ Recalculated correct balances                       │
  │  └─ Applied corrections with full audit trail           │
  │                                                          │
  │  FIX DEPLOYED:                                           │
  │  ├─ SELECT ... FOR UPDATE (row-level locking)           │
  │  ├─ Atomic balance updates: UPDATE SET balance =        │
  │  │   balance + $amount (not read-modify-write)          │
  │  └─ Added real-time balance checksum verification       │
  │                                                          │
  │  PREVENTION IMPROVEMENTS:                                │
  │  ├─ Real-time reconciliation (was daily, now hourly)    │
  │  ├─ Row-level checksums on financial records            │
  │  ├─ Mandatory concurrency testing for financial code    │
  │  └─ Event sourcing → always have ground truth           │
  └──────────────────────────────────────────────────────────┘
  
  KEY INSIGHT: Without event sourcing, recovery would have 
  required manual investigation of 2,147 accounts. With 
  event sourcing, recovery was automated in 2 hours.
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Data inconsistency between microservices" | Implement saga pattern or eventual consistency with reconciliation. Use event-driven architecture with idempotent consumers. |
| "Silent data corruption going undetected" | Add checksums to critical data. Run continuous validation jobs. Implement cross-system reconciliation. Use event sourcing for audit trail. |
| "Accidental data deletion (DROP TABLE, bad WHERE clause)" | Enable soft deletes. Use point-in-time recovery (PITR). Require MFA for destructive operations. Add confirmation delays for bulk deletes. |
| "Duplicate records appearing" | Enforce UNIQUE constraints at DB level. Use idempotency keys for all write operations. Add deduplication checks in application layer. |
| "Backup restoration fails" | Test backup restoration regularly (monthly minimum). Verify backup integrity with checksums. Monitor backup job completion. Store backups in multiple locations. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Data Integrity Threats** | Corruption, loss, inconsistency, silent corruption (most dangerous) |
| **Defense in Depth** | Prevention (constraints) → Detection (validation) → Recovery (backups) |
| **Idempotency** | Same operation applied multiple times = same result. Essential for retries. |
| **Event Sourcing** | Immutable event log as source of truth. Can rebuild state at any point. |
| **Checksums** | Hash-based verification at row, table, and cross-system levels |
| **Reconciliation** | Continuous comparison between systems to detect discrepancies |
| **Soft Deletes** | Mark data as deleted instead of removing. Enables recovery. |

---

## Quick Revision Questions

1. **Why is silent data corruption the most dangerous integrity threat?**
   > Because no error is raised when it occurs. Corrupted data can propagate through the system for days or weeks before detection, making it difficult to identify the scope of damage and the point at which corruption began.

2. **What is idempotency and why is it critical for data integrity?**
   > Idempotency means that performing the same operation multiple times produces the same result as performing it once. It's critical because network failures and retries are inevitable in distributed systems — without idempotency, retried operations (like payments) can be applied multiple times.

3. **How does event sourcing help with data integrity?**
   > Event sourcing stores an immutable log of all state changes. If the current state becomes corrupted, you can rebuild the correct state by replaying events from the beginning. It also provides a complete audit trail showing exactly what happened and when.

4. **What are the three layers of data defense in depth?**
   > (1) Prevention — input validation, schema constraints, transactions, access control. (2) Detection — checksums, validation pipelines, anomaly detection, reconciliation. (3) Recovery — backups, PITR, soft deletes, event sourcing, data versioning.

5. **How can you detect data inconsistency between a database and a payment gateway?**
   > Run reconciliation jobs that compare records between systems — count of transactions, sum of amounts, and individual record matching. Alert when discrepancies exceed a threshold. Run frequently (hourly for financial data) to limit the blast radius of any issues.

---

[← Previous: Disaster Recovery](03-disaster-recovery.md) | [Back to README](../README.md) | [Next: Service Architecture →](05-service-architecture.md)
