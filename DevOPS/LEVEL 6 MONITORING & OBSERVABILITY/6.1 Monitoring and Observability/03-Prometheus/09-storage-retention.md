# Chapter 9: Storage and Retention

[← Previous: Federation](08-federation.md) | [Back to README](../README.md) | [Next: Unit 4 — Grafana Concepts →](../04-Grafana/01-grafana-concepts.md)

---

## Overview

Understanding Prometheus storage internals helps with capacity planning, performance tuning, and choosing between local storage and long-term solutions.

---

## 9.1 Local Storage Architecture

```
┌──────────────────────────────────────────────────────┐
│                PROMETHEUS DATA DIRECTORY              │
│                                                       │
│  data/                                                │
│  ├── wal/              ← Write-Ahead Log (recent)    │
│  │   ├── 00000001                                    │
│  │   └── 00000002                                    │
│  ├── 01BKGV7JC0RY8.../ ← Completed block (2h)      │
│  │   ├── chunks/                                     │
│  │   │   └── 000001    ← Compressed time series      │
│  │   ├── index         ← Inverted index              │
│  │   ├── meta.json     ← Block metadata              │
│  │   └── tombstones    ← Deletion markers            │
│  ├── 01BKGV7JC0RY9.../ ← Another block              │
│  └── lock                                             │
└──────────────────────────────────────────────────────┘
```

---

## 9.2 Retention Settings

```bash
# Time-based retention (default: 15 days)
./prometheus --storage.tsdb.retention.time=30d

# Size-based retention
./prometheus --storage.tsdb.retention.size=50GB

# Both (whichever is hit first triggers cleanup)
./prometheus \
  --storage.tsdb.retention.time=90d \
  --storage.tsdb.retention.size=100GB
```

---

## 9.3 Storage Sizing

```
Rule of thumb:
  Storage per sample ≈ 1-2 bytes (compressed)
  
  Storage = samples_per_second × bytes_per_sample × retention_seconds
  
  Example:
  • 100,000 time series
  • 15s scrape interval → 100,000/15 ≈ 6,667 samples/sec
  • 2 bytes per sample
  • 30 days retention
  
  Storage ≈ 6,667 × 2 × 30 × 86,400 ≈ 34 GB

  Add 20% overhead for WAL and index ≈ ~41 GB total
```

---

## 9.4 Long-Term Storage Solutions

```
┌──────────────────────────────────────────────────────┐
│           LONG-TERM STORAGE OPTIONS                   │
│                                                       │
│  Prometheus ──remote_write──► ┌─────────────────┐    │
│  (short-term)                 │ Long-term store │    │
│                               │                 │    │
│                               │ • Thanos        │    │
│  Grafana ◄──── query ────────│ • Cortex/Mimir  │    │
│                               │ • VictoriaM.    │    │
│                               │ • M3DB          │    │
│                               └─────────────────┘    │
└──────────────────────────────────────────────────────┘
```

### Remote Write Configuration

```yaml
# prometheus.yml
remote_write:
  - url: "http://mimir:9009/api/v1/push"
    queue_config:
      max_samples_per_send: 1000
      batch_send_deadline: 5s
      max_shards: 200

remote_read:
  - url: "http://mimir:9009/prometheus/api/v1/read"
    read_recent: false  # Only read older data from remote
```

### Comparison of Long-Term Options

```
┌──────────────┬──────────┬───────────┬──────────┬──────────┐
│ Solution     │ Backend  │ Scale     │ Multi-   │ License  │
│              │ Storage  │           │ tenant   │          │
├──────────────┼──────────┼───────────┼──────────┼──────────┤
│ Thanos       │ Object   │ Very high │ Limited  │ Apache 2 │
│              │ store    │           │          │          │
│ Grafana Mimir│ Object   │ Very high │ Yes      │ AGPL 3   │
│              │ store    │           │          │          │
│ VictoriaM.   │ Custom   │ Very high │ Yes      │ Apache 2 │
│ M3DB         │ Custom   │ High      │ Yes      │ Apache 2 │
│ InfluxDB     │ Custom   │ Medium    │ No       │ MIT/Comm │
└──────────────┴──────────┴───────────┴──────────┴──────────┘
```

---

## 9.5 Performance Monitoring

```promql
# Current active time series
prometheus_tsdb_head_series

# Storage size
prometheus_tsdb_storage_size_bytes

# Compaction duration
prometheus_tsdb_compaction_duration_seconds

# WAL corruptions
prometheus_tsdb_wal_corruptions_total

# Samples ingested per second
rate(prometheus_tsdb_head_samples_appended_total[5m])
```

---

## 9.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| High disk usage | Too many series or long retention | Reduce cardinality, lower retention |
| OOM kills | Too many active series in HEAD | Check `prometheus_tsdb_head_series` |
| Slow startup | Large WAL replay | Enable WAL compression |
| Data gaps | Unreliable remote write | Check queue/error metrics |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Default retention** | 15 days |
| **Storage per sample** | ~1-2 bytes compressed |
| **Block size** | 2 hours (compacted over time) |
| **WAL** | Write-ahead log for crash recovery |
| **Long-term options** | Thanos, Mimir, VictoriaMetrics |
| **Remote Write** | Push data to external TSDB |

---

## Quick Revision Questions

1. **How does Prometheus store data locally? Describe the block structure.**
2. **Calculate the storage needed for 50K time series, 15s scrape interval, 60-day retention.**
3. **What are the two types of retention limits you can configure?**
4. **When should you use a long-term storage solution instead of local storage?**
5. **Name three long-term storage backends for Prometheus and compare them.**

---

[← Previous: Federation](08-federation.md) | [Back to README](../README.md) | [Next: Unit 4 — Grafana Concepts →](../04-Grafana/01-grafana-concepts.md)
