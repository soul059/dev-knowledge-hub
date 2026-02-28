# Chapter 3: Cost Optimization

[← Previous: Observability Pipeline Design](02-observability-pipeline-design.md) | [Back to README](../README.md) | [Next: Security & Compliance →](04-security-compliance.md)

---

## Overview

Observability costs can grow exponentially with scale. Metrics cardinality, log volume, and trace storage are the primary cost drivers. This chapter covers strategies to reduce costs without losing critical visibility.

---

## 3.1 Cost Anatomy

```
┌──────────────────────────────────────────────────────────┐
│  OBSERVABILITY COST BREAKDOWN                            │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ INGESTION (40-60% of cost)                      │    │
│  │ • Metrics: per active time series               │    │
│  │ • Logs:    per GB ingested                      │    │
│  │ • Traces:  per GB or per span ingested          │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ STORAGE (20-30% of cost)                        │    │
│  │ • Hot storage: fast queries, high cost           │    │
│  │ • Warm storage: slower, moderate cost            │    │
│  │ • Cold/archive: cheapest (S3, GCS)              │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ QUERY (10-20% of cost)                          │    │
│  │ • CPU for queries                                │    │
│  │ • Memory for aggregations                        │    │
│  │ • Network for data transfer                      │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ LICENSE / SEATS (varies)                         │    │
│  │ • Per-host, per-user, per-container pricing      │    │
│  └─────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Metrics Cost Optimization

### Cardinality Management

```
┌──────────────────────────────────────────────────────────┐
│  THE CARDINALITY PROBLEM                                 │
│                                                          │
│  Metric: http_requests_total                             │
│  Labels:                                                 │
│    method:  3 values  (GET, POST, PUT)                  │
│    status:  5 values  (200, 201, 400, 404, 500)         │
│    path:    1000 values  ← DANGER!                      │
│    user_id: 100,000 values  ← CATASTROPHE!              │
│                                                          │
│  3 × 5 × 1000 × 100,000 = 1,500,000,000 series!!       │
│                                                          │
│  FIX:                                                    │
│    path: 10 buckets (normalized)                         │
│    user_id: REMOVED (not a label)                        │
│                                                          │
│  3 × 5 × 10 = 150 series ✓                             │
└──────────────────────────────────────────────────────────┘
```

### Strategies

```yaml
# 1. Recording rules to pre-aggregate
groups:
  - name: cost-optimization
    interval: 1m
    rules:
      # Pre-aggregate at service level (drop pod/instance labels)
      - record: service:http_requests:rate5m
        expr: sum by (service, method, status_code) (
          rate(http_requests_total[5m])
        )

# 2. Drop unused metrics at collection
# OTel Collector filter processor
processors:
  filter/metrics:
    metrics:
      exclude:
        match_type: regexp
        metric_names:
          - "go_.*"              # Go runtime (if not needed)
          - "process_.*"         # Process metrics
          - "promhttp_.*"        # Prometheus client metrics
          - ".*_bucket"          # Drop histogram buckets (keep sum/count)

# 3. Reduce scrape frequency for non-critical metrics
scrape_configs:
  - job_name: "low-priority"
    scrape_interval: 60s        # Instead of default 15s
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: "(node_disk_io_time_seconds_total|node_network_.*)"
        action: keep
```

---

## 3.3 Log Cost Optimization

```
┌──────────────────────────────────────────────────────────┐
│  LOG VOLUME REDUCTION STRATEGIES                         │
│                                                          │
│  BEFORE:  10 TB/day  ──────────────────────────  $$$$$  │
│                                                          │
│  Step 1: Drop health/debug   -40%  ──────────── $$$$   │
│  Step 2: Sample verbose logs -20%  ─────────── $$$    │
│  Step 3: Compress            -30%  ──────── $$       │
│  Step 4: Tiered retention    -20%  ────── $         │
│                                                          │
│  AFTER: ~1.7 TB/day effective cost  ──── $              │
└──────────────────────────────────────────────────────────┘
```

### Filtering and Sampling

```yaml
# OTel Collector: drop debug logs
processors:
  filter/logs:
    logs:
      exclude:
        match_type: strict
        severity_texts:
          - "DEBUG"
          - "TRACE"
      exclude:
        match_type: regexp
        bodies:
          - "health.*check"
          - "GET /ready"
          - "GET /alive"

# Loki: retention policies (different per tenant/stream)
# loki-config.yaml
limits_config:
  retention_period: 30d          # Default 30 days
  per_stream_rate_limit: 3MB

overrides:
  team-frontend:
    retention_period: 14d        # Less retention for verbose logs
  team-platform:
    retention_period: 90d        # More retention for infra logs
```

### Log Level Strategy

```
┌──────────────────────────────────────────────────────┐
│  PRODUCTION LOG LEVEL RECOMMENDATIONS                │
│                                                      │
│  Level     │ Store?  │ Retention │ Notes             │
│  ──────────┼─────────┼───────────┼─────────────────  │
│  ERROR     │ Always  │ 90 days   │ Full context      │
│  WARN      │ Always  │ 30 days   │ Important signals  │
│  INFO      │ Sample  │ 14 days   │ 1:10 sample rate  │
│  DEBUG     │ Never*  │  -        │ Dev only          │
│  TRACE     │ Never   │  -        │ Dev only          │
│                                                      │
│  * Enable temporarily via dynamic log level          │
└──────────────────────────────────────────────────────┘
```

---

## 3.4 Trace Cost Optimization

### Sampling Strategies

```
┌──────────────────────────────────────────────────────────┐
│  SAMPLING DECISION TREE                                  │
│                                                          │
│  Incoming Trace                                          │
│       │                                                  │
│       ▼                                                  │
│  ┌─ Has error? ──► YES ──► KEEP (100%)                  │
│  │                                                       │
│  └─ NO                                                   │
│       │                                                  │
│       ▼                                                  │
│  ┌─ Latency > 2s? ──► YES ──► KEEP (100%)              │
│  │                                                       │
│  └─ NO                                                   │
│       │                                                  │
│       ▼                                                  │
│  ┌─ Priority endpoint? ──► YES ──► KEEP (50%)          │
│  │   (/checkout, /payment)                               │
│  │                                                       │
│  └─ NO ──► KEEP (5%)  ← Probabilistic sample           │
│                                                          │
│  Result: ~90% cost reduction while keeping              │
│          all interesting traces                           │
└──────────────────────────────────────────────────────────┘
```

### Tail Sampling Config

```yaml
# OTel Collector Gateway - tail sampling
processors:
  tail_sampling:
    decision_wait: 30s
    num_traces: 100000
    policies:
      # Always keep errors
      - name: errors-policy
        type: status_code
        status_code:
          status_codes: [ERROR]
      # Always keep slow requests
      - name: latency-policy
        type: latency
        latency:
          threshold_ms: 2000
      # Keep more of critical paths
      - name: critical-paths
        type: string_attribute
        string_attribute:
          key: http.route
          values: ["/api/checkout", "/api/payment"]
      # Sample everything else at 5%
      - name: probabilistic-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 5
```

---

## 3.5 Storage Tiering

```
┌──────────────────────────────────────────────────────────┐
│  STORAGE TIERING                                         │
│                                                          │
│  Time         Cost    Access     Storage                 │
│  ────────────────────────────────────────────            │
│                                                          │
│  0-7 days     $$$$   Instant    SSD / Hot              │
│                                  (Local, EBS gp3)       │
│                                                          │
│  7-30 days    $$     Fast       Object Store / Warm     │
│                                  (S3, GCS standard)     │
│                                                          │
│  30-90 days   $      Slow       Cold                    │
│                                  (S3 IA, GCS Nearline)  │
│                                                          │
│  90+ days     ¢      Archive    Deep Archive            │
│                                  (Glacier, Archive)      │
│                                                          │
│  Example cost reduction:                                 │
│    All hot (90 days): $10,000/month                     │
│    Tiered:            $2,500/month (75% savings)        │
└──────────────────────────────────────────────────────────┘
```

### Mimir/Loki/Tempo with S3 Backend

```yaml
# Mimir storage config
storage:
  backend: s3
  s3:
    bucket_name: metrics-data
    endpoint: s3.amazonaws.com
    region: us-east-1

# Block storage with lifecycle
compactor:
  block_ranges: [2h, 12h, 24h]     # Compact progressively
  deletion_delay: 12h

# S3 lifecycle rules (via Terraform)
resource "aws_s3_bucket_lifecycle_configuration" "telemetry" {
  bucket = aws_s3_bucket.telemetry.id

  rule {
    id     = "transition-to-ia"
    status = "Enabled"
    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }
    transition {
      days          = 90
      storage_class = "GLACIER"
    }
    expiration {
      days = 365
    }
  }
}
```

---

## 3.6 Cost Monitoring & Governance

```yaml
# Dashboard queries to monitor observability cost drivers

# Metric cardinality
prometheus_tsdb_head_series              # Total active series
count by (__name__) ({__name__=~".+"})   # Series per metric name

# Log ingestion rate
sum(rate(loki_distributor_bytes_received_total[1h])) * 3600
# → bytes per hour ingested

# Trace ingestion rate
sum(rate(tempo_distributor_spans_received_total[1h])) * 3600
# → spans per hour ingested

# Per-team/namespace breakdown
sum by (namespace) (
  rate(loki_distributor_bytes_received_total[1h])
)
```

### Cost Alert Example

```yaml
groups:
  - name: cost-alerts
    rules:
      - alert: MetricCardinalityExplosion
        expr: prometheus_tsdb_head_series > 2000000
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Metric cardinality exceeded 2M series"

      - alert: LogIngestionSpike
        expr: |
          sum(rate(loki_distributor_bytes_received_total[1h])) * 3600 
          > 100e9  # 100GB/hour
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Log ingestion rate exceeds 100GB/hour"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Bill spike | Cardinality explosion | Find metric with highest series count, add relabel to drop |
| Log costs too high | Debug logs in prod | Filter at collector level, set log level to INFO |
| Can't reduce traces | Need all traces | Implement tail sampling — keep errors+slow, sample rest |
| Storage growing fast | No retention policy | Configure retention per signal, add S3 lifecycle |
| Cost not attributable | No per-team tracking | Add team/namespace labels, use multi-tenant backends |

---

## Summary Table

| Strategy | Signal | Savings |
|----------|--------|---------|
| Drop unused metrics | Metrics | 20-50% |
| Pre-aggregate with recording rules | Metrics | 30-60% |
| Filter debug/health logs | Logs | 30-50% |
| Tail sampling | Traces | 80-95% |
| Storage tiering | All | 50-75% |
| Reduce scrape frequency | Metrics | 10-30% |
| Compression | All | 20-40% |

---

## Quick Revision Questions

1. **What are the three main cost drivers in observability?**
2. **How does cardinality affect metrics cost, and how do you control it?**
3. **What is tail sampling, and why does it save more money than head sampling?**
4. **Describe a storage tiering strategy with estimated savings.**
5. **How do you monitor and alert on observability costs themselves?**
6. **What is the difference between dropping metrics at collection vs. using recording rules?**

---

[← Previous: Observability Pipeline Design](02-observability-pipeline-design.md) | [Back to README](../README.md) | [Next: Security & Compliance →](04-security-compliance.md)
