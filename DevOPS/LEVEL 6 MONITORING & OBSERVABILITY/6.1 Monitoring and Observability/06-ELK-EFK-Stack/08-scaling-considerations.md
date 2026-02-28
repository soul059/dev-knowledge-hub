# Chapter 8: Scaling Considerations

[← Previous: Search & Queries](07-search-queries.md) | [Back to README](../README.md) | [Next: Tracing Concepts →](../07-Distributed-Tracing/01-tracing-concepts.md)

---

## Overview

Scaling the ELK/EFK stack requires understanding bottlenecks at each tier — collection, processing, storage, and query. This chapter covers horizontal and vertical scaling strategies.

---

## 8.1 Scaling Dimensions

```
┌──────────────────────────────────────────────────────────┐
│             SCALING DECISION MATRIX                      │
│                                                          │
│  Bottleneck         │ Scale Vertically  │ Scale Horiz.   │
│  ───────────────────┼───────────────────┼────────────── │
│  Indexing slow      │ More CPU/RAM      │ More data nodes│
│  Queries slow       │ More RAM (heap)   │ More replicas  │
│  Disk full          │ Bigger disks      │ More nodes     │
│  Log volume spike   │ Buffer size       │ More collectors│
│  Processing lag     │ More Logstash RAM │ More pipelines │
│                                                          │
│  Rule of thumb: Scale vertically first (cheaper),       │
│  then horizontally when hitting limits                  │
└──────────────────────────────────────────────────────────┘
```

---

## 8.2 Elasticsearch Scaling

```
┌──────────────────────────────────────────────────────────┐
│  HORIZONTAL SCALING (add nodes)                          │
│                                                          │
│  Before:  3 data nodes × 500 GB = 1.5 TB capacity      │
│  After:   6 data nodes × 500 GB = 3.0 TB capacity      │
│                                                          │
│  Elasticsearch automatically:                            │
│  ├── Rebalances shards across new nodes                 │
│  ├── Distributes replicas away from primaries           │
│  └── Starts indexing to new nodes                       │
│                                                          │
│  VERTICAL SCALING (bigger nodes)                         │
│                                                          │
│  Before:  4 CPU, 16 GB RAM, 500 GB SSD                  │
│  After:   8 CPU, 32 GB RAM, 1 TB NVMe                   │
│                                                          │
│  JVM heap: max(50% RAM, 32 GB)                          │
│  32 GB limit: JVM compressed oops boundary              │
└──────────────────────────────────────────────────────────┘
```

### Hot-Warm-Cold Architecture

```yaml
# Node configuration for tiered storage

# Hot node (fast, expensive)
node.roles: [data_hot, data_content]
node.attr.data: hot
# NVMe SSD, high CPU

# Warm node (moderate)
node.roles: [data_warm]
node.attr.data: warm
# Standard SSD, moderate CPU

# Cold node (cheap, slow)
node.roles: [data_cold]
node.attr.data: cold
# HDD or S3-backed, minimal CPU

# Frozen tier (cheapest)
node.roles: [data_frozen]
# Searchable snapshots from S3
```

---

## 8.3 Collection Tier Scaling

```
  High Volume Strategy:

  ┌─────────────────────────────────────────────┐
  │                                              │
  │  Nodes (100+)                                │
  │  ├── Fluent Bit DaemonSet ──┐                │
  │  │   (lightweight, 15MB)    │                │
  │  │                          ▼                │
  │  │                  ┌──────────────┐         │
  │  │                  │ Kafka Cluster │ Buffer  │
  │  │                  │ (3+ brokers)  │         │
  │  │                  └──────┬───────┘         │
  │  │                         │                  │
  │  │                         ▼                  │
  │  │                  ┌──────────────┐         │
  │  │                  │ Logstash Pool │ Process │
  │  │                  │ (auto-scaled) │         │
  │  │                  └──────┬───────┘         │
  │  │                         │                  │
  │  │                         ▼                  │
  │  │                  ┌──────────────┐         │
  │  │                  │ Elasticsearch│ Store   │
  │  │                  └──────────────┘         │
  │  └──────────────────────────────────         │
  └─────────────────────────────────────────────┘

  Kafka provides:
  ✅ Absorbs burst traffic
  ✅ Decouples producers/consumers
  ✅ Message replay capability
  ✅ Multiple consumer groups
```

---

## 8.4 Cost Optimization

| Strategy | Savings | Trade-off |
|----------|---------|-----------|
| Tiered storage (hot/warm/cold) | 50-70% | Slower queries on old data |
| Log filtering (drop health checks) | 10-30% | Lost data |
| Reduce replica count (warm/cold) | 30-50% | Lower redundancy |
| Compression (best_compression) | 20-30% | More CPU for indexing |
| Shorter retention | Linear | Lost historical data |
| Sampling verbose logs | 20-40% | Missing some detail |
| Searchable snapshots (frozen tier) | 80-90% | Much slower queries |

### Cost Comparison: ELK vs Loki

```
100 GB/day log ingestion, 30 days retention:

ELK Stack:
  ES Hot:   3 nodes × $500/mo  = $1,500/mo
  ES Warm:  2 nodes × $300/mo  = $600/mo
  Logstash: 2 nodes × $200/mo  = $400/mo
  Kibana:   1 node  × $200/mo  = $200/mo
  Storage:  3 TB    × $0.10/GB = $300/mo
  Total:    ~$3,000/mo

Loki + Grafana:
  Loki:     2 nodes × $300/mo  = $600/mo
  S3:       3 TB    × $0.023/GB= $69/mo
  Grafana:  1 node  × $200/mo  = $200/mo
  Total:    ~$870/mo  (71% cheaper)

Trade-off: Loki has limited full-text search
```

---

## 8.5 Monitoring the Stack Itself

```yaml
# Key metrics to monitor on your ELK stack:

# Elasticsearch
- elasticsearch_cluster_health_status
- elasticsearch_indices_indexing_index_total (rate)
- elasticsearch_jvm_memory_used_bytes / max_bytes
- elasticsearch_indices_search_query_time_seconds
- elasticsearch_fs_total_available_in_bytes
- elasticsearch_cluster_health_unassigned_shards

# Logstash
- logstash_node_pipeline_events_filtered_total (rate)
- logstash_node_pipeline_events_in_total (rate)
- logstash_node_jvm_memory_heap_used_percent
- logstash_node_pipeline_queue_events

# Filebeat
- filebeat_harvester_running
- filebeat_input_log_files_truncated_total
- filebeat_output_events_acked_total (rate)
```

---

## 8.6 ELK vs Alternatives Summary

| Feature | ELK | Loki | Datadog | Splunk |
|---------|-----|------|---------|--------|
| Cost | Medium | Low | High | Very High |
| Full-text search | Excellent | Limited | Good | Excellent |
| Scaling complexity | High | Medium | Managed | Managed |
| Self-hosted | Yes | Yes | No | Hybrid |
| Kubernetes native | Moderate | Excellent | Good | Moderate |
| Community | Large | Growing | — | — |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Cluster unstable at scale | Too many shards, small heap | Reduce shard count, increase heap to 32 GB |
| Indexing bottleneck | Slow disks or single pipeline | Use NVMe SSDs, parallel pipelines |
| Query performance degrades | Index too large, no replicas | Add replicas, use routing, add coordinating nodes |
| Cost spiraling | Logging everything, long retention | Filter, sample, tiered storage, shorter retention |
| Uneven shard distribution | Shard allocation awareness not set | Configure `cluster.routing.allocation.awareness` |

---

## Quick Revision Questions

1. **What are the main scaling dimensions for an ELK stack?**
2. **How does the hot-warm-cold architecture reduce costs?**
3. **Why is Kafka useful in a high-volume log pipeline?**
4. **What is the JVM heap 32 GB limit and why does it matter?**
5. **Compare the cost and trade-offs of ELK vs Loki.**
6. **What metrics should you monitor on your ELK stack itself?**

---

[← Previous: Search & Queries](07-search-queries.md) | [Back to README](../README.md) | [Next: Tracing Concepts →](../07-Distributed-Tracing/01-tracing-concepts.md)
