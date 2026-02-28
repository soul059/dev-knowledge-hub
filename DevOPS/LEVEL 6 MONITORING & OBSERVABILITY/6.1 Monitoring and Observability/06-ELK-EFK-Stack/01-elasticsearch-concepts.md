# Chapter 1: Elasticsearch Concepts

[← Previous: Log Aggregation Patterns](../05-Logging/05-log-aggregation-patterns.md) | [Back to README](../README.md) | [Next: Logstash & Fluentd →](02-logstash-fluentd.md)

---

## Overview

Elasticsearch is a distributed, RESTful search and analytics engine built on Apache Lucene. It is the core of the ELK (Elasticsearch, Logstash, Kibana) and EFK (Elasticsearch, Fluentd, Kibana) stacks.

---

## 1.1 Elasticsearch Architecture

```
┌──────────────────────────────────────────────────────────┐
│              ELASTICSEARCH CLUSTER                       │
│                                                          │
│  ┌─── Node 1 (Master) ──┐  ┌─── Node 2 (Data) ──────┐ │
│  │ ┌──────────────────┐  │  │ ┌──────────────────┐    │ │
│  │ │ Index: logs-2024  │  │  │ │ Index: logs-2024  │    │ │
│  │ │ ┌─────┐ ┌─────┐  │  │  │ │ ┌─────┐ ┌─────┐  │    │ │
│  │ │ │Shard│ │Shard│  │  │  │ │ │Repli│ │Repli│  │    │ │
│  │ │ │  0  │ │  1  │  │  │  │ │ │ca 0│ │ca 1│  │    │ │
│  │ │ │(P)  │ │(P)  │  │  │  │ │ │(R)  │ │(R)  │  │    │ │
│  │ │ └─────┘ └─────┘  │  │  │ │ └─────┘ └─────┘  │    │ │
│  │ └──────────────────┘  │  │ └──────────────────┘    │ │
│  └───────────────────────┘  └─────────────────────────┘ │
│                                                          │
│  ┌─── Node 3 (Data) ──────┐                            │
│  │ ┌──────────────────┐    │                            │
│  │ │ Index: logs-2024  │    │                            │
│  │ │ ┌─────┐ ┌─────┐  │    │  P = Primary shard       │
│  │ │ │Shard│ │Repli│  │    │  R = Replica shard        │
│  │ │ │  2  │ │ca 2│  │    │                            │
│  │ │ │(P)  │ │(R)  │  │    │                            │
│  │ │ └─────┘ └─────┘  │    │                            │
│  │ └──────────────────┘    │                            │
│  └─────────────────────────┘                            │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Core Concepts

| Concept | Description | Analogy |
|---------|------------|---------|
| **Cluster** | Group of nodes working together | Database cluster |
| **Node** | Single Elasticsearch instance | Database server |
| **Index** | Collection of documents | Database table |
| **Document** | Single JSON record | Table row |
| **Shard** | Horizontal partition of an index | Table partition |
| **Replica** | Copy of a primary shard | Read replica |
| **Mapping** | Schema definition for fields | Table schema |

---

## 1.3 Node Roles

```
┌──────────────────────────────────────────────────────────┐
│  NODE ROLES                                              │
│                                                          │
│  Master Node:           Cluster state management         │
│  ├── Index creation/deletion                             │
│  ├── Shard allocation                                    │
│  └── Node tracking                                       │
│                                                          │
│  Data Node:             Stores data, handles CRUD        │
│  ├── Indexing documents                                  │
│  ├── Searching                                           │
│  └── Aggregations                                        │
│                                                          │
│  Ingest Node:           Pre-process documents            │
│  ├── Pipeline processing                                 │
│  ├── Field extraction                                    │
│  └── Enrichment                                          │
│                                                          │
│  Coordinating Node:     Routes requests, merges results  │
│  ├── Query distribution                                  │
│  ├── Result aggregation                                  │
│  └── Load balancing                                      │
│                                                          │
│  Production Setup:                                       │
│    3 dedicated master nodes (quorum)                     │
│    N data nodes (based on storage needs)                 │
│    2+ coordinating nodes (for heavy queries)             │
└──────────────────────────────────────────────────────────┘
```

---

## 1.4 Index & Document Operations

```bash
# Create an index with mappings
PUT /logs-2024.01.15
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "refresh_interval": "5s"
  },
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "level":     { "type": "keyword" },
      "service":   { "type": "keyword" },
      "message":   { "type": "text" },
      "trace_id":  { "type": "keyword" },
      "duration":  { "type": "float" },
      "status":    { "type": "integer" }
    }
  }
}

# Index a document
POST /logs-2024.01.15/_doc
{
  "timestamp": "2024-01-15T10:23:45Z",
  "level": "ERROR",
  "service": "api-server",
  "message": "Payment processing failed: timeout",
  "trace_id": "abc123",
  "duration": 30.5,
  "status": 500
}

# Search
GET /logs-2024.01.15/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "ERROR" } },
        { "range": { "timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "sort": [{ "timestamp": { "order": "desc" } }],
  "size": 100
}
```

---

## 1.5 Index Lifecycle Management (ILM)

```
┌──────────────────────────────────────────────────────────┐
│              INDEX LIFECYCLE PHASES                       │
│                                                          │
│  Hot ───▶ Warm ───▶ Cold ───▶ Frozen ───▶ Delete       │
│  (0-3d)   (3-7d)   (7-30d)   (30-90d)    (>90d)       │
│                                                          │
│  Hot:    Fast SSDs, actively writing                    │
│  Warm:   Standard disks, read-only                      │
│  Cold:   Cheap storage, infrequent access               │
│  Frozen: Searchable snapshots (S3)                      │
│  Delete: Removed permanently                            │
└──────────────────────────────────────────────────────────┘
```

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

---

## 1.6 Field Types

| Type | Use For | Example |
|------|---------|---------|
| `keyword` | Exact match, aggregations | status codes, service names |
| `text` | Full-text search | log messages |
| `date` | Timestamps | `2024-01-15T10:23:45Z` |
| `integer/long` | Numeric values | HTTP status codes |
| `float/double` | Decimal numbers | Duration, latency |
| `boolean` | True/false | `is_error` |
| `ip` | IP addresses | `192.168.1.1` |
| `geo_point` | Latitude/longitude | Location data |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Cluster RED | Unassigned primary shards | Check disk space, node health |
| Cluster YELLOW | Missing replica shards | Add nodes or reduce replicas |
| Slow indexing | Too many shards, small docs | Use bulk API, optimize shard count |
| Slow queries | Large time ranges, no filters | Add keyword filters, use date ranges |
| Disk full | No ILM or retention | Configure ILM policy, add cold tier |

---

## Quick Revision Questions

1. **What are the core components of an Elasticsearch cluster?**
2. **What is the difference between a primary shard and a replica shard?**
3. **What are the different node roles and when would you use each?**
4. **How does Index Lifecycle Management work?**
5. **When should you use `keyword` vs `text` field types?**
6. **What do RED, YELLOW, and GREEN cluster health statuses mean?**

---

[← Previous: Log Aggregation Patterns](../05-Logging/05-log-aggregation-patterns.md) | [Back to README](../README.md) | [Next: Logstash & Fluentd →](02-logstash-fluentd.md)
