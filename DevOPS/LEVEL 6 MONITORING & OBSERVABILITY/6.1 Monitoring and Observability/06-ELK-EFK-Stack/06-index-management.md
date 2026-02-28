# Chapter 6: Index Management

[← Previous: ELK Stack Deployment](05-elk-stack-deployment.md) | [Back to README](../README.md) | [Next: Search & Queries →](07-search-queries.md)

---

## Overview

Index management covers strategies for organizing, optimizing, and retaining Elasticsearch indices. Proper management prevents cluster instability and controls storage costs.

---

## 6.1 Index Naming Strategies

```
┌──────────────────────────────────────────────────────────┐
│  INDEX NAMING PATTERNS                                   │
│                                                          │
│  Time-based (most common for logs):                      │
│    logs-2024.01.15                                       │
│    logs-2024.01.16                                       │
│    logs-2024.01.17                                       │
│                                                          │
│  Rollover-based (recommended):                           │
│    logs-000001  (active, writing)                        │
│    logs-000002  (next after rollover)                    │
│    Alias: "logs" → points to active index               │
│                                                          │
│  Category-based:                                         │
│    logs-api-2024.01.15                                   │
│    logs-web-2024.01.15                                   │
│    logs-worker-2024.01.15                                │
│                                                          │
│  Environment-based:                                      │
│    prod-logs-2024.01.15                                  │
│    staging-logs-2024.01.15                               │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Index Templates

```json
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "priority": 100,
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "refresh_interval": "5s",
      "index.lifecycle.name": "logs-policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "mappings": {
      "properties": {
        "@timestamp":  { "type": "date" },
        "level":       { "type": "keyword" },
        "service":     { "type": "keyword" },
        "namespace":   { "type": "keyword" },
        "message":     { "type": "text" },
        "trace_id":    { "type": "keyword" },
        "duration_ms": { "type": "float" },
        "status_code": { "type": "short" },
        "tags":        { "type": "keyword" }
      },
      "dynamic_templates": [
        {
          "strings_as_keywords": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      ]
    },
    "aliases": {
      "logs-read": {}
    }
  }
}
```

---

## 6.3 Index Lifecycle Management (ILM) in Detail

```
┌──────────────────────────────────────────────────────────┐
│               ILM PHASE TRANSITIONS                      │
│                                                          │
│  HOT (0-3 days)                                          │
│  ├── Active indexing (writes)                            │
│  ├── Fast SSDs / NVMe storage                            │
│  ├── Rollover when: >50GB OR >1 day                     │
│  │                                                       │
│  ▼                                                       │
│  WARM (3-30 days)                                        │
│  ├── Read-only (no more writes)                          │
│  ├── Shrink: 3 shards → 1 shard                         │
│  ├── Force merge: many segments → 1 segment             │
│  ├── Move to cheaper storage                             │
│  │                                                       │
│  ▼                                                       │
│  COLD (30-90 days)                                       │
│  ├── Infrequently accessed                               │
│  ├── Frozen / searchable snapshots                       │
│  ├── Cheapest available storage                          │
│  │                                                       │
│  ▼                                                       │
│  DELETE (>90 days)                                        │
│  └── Permanently removed                                 │
└──────────────────────────────────────────────────────────┘
```

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1d"
          },
          "set_priority": { "priority": 100 }
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 },
          "allocate": {
            "require": { "data": "warm" }
          },
          "set_priority": { "priority": 50 }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "require": { "data": "cold" }
          },
          "set_priority": { "priority": 0 }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

---

## 6.4 Shard Management

```
SHARD SIZING RULES OF THUMB:
  ┌──────────────────────────────────────────────┐
  │  Target shard size:  20-50 GB                 │
  │  Max shards per node: ~1000                   │
  │  Max shards per GB heap: 20                   │
  │                                                │
  │  Example:                                      │
  │    Daily log volume: 100 GB                    │
  │    Target shard size: 50 GB                    │
  │    Primary shards needed: 100/50 = 2           │
  │    With 1 replica: 2P + 2R = 4 shards/day     │
  │    30-day retention: 4 × 30 = 120 shards      │
  │    Heap needed: 120/20 = 6 GB minimum          │
  └──────────────────────────────────────────────┘

  OVERSHARDING (too many small shards):
  ✗ Each shard has overhead (~20 MB heap)
  ✗ More shards = more cluster state
  ✗ Slower searches across many shards

  UNDERSIZED (too few large shards):
  ✗ Slower indexing (fewer parallel writes)
  ✗ Recovery takes longer
  ✗ Unbalanced cluster
```

---

## 6.5 Common Index Operations

```bash
# List all indices
GET _cat/indices?v&s=store.size:desc

# Check index health
GET _cat/indices/logs-*?v&h=index,health,pri,rep,docs.count,store.size

# Close index (free resources, no search)
POST /logs-2024.01.01/_close

# Delete old indices
DELETE /logs-2024.01.01

# Reindex (copy data to new index)
POST _reindex
{
  "source": { "index": "logs-old" },
  "dest": { "index": "logs-new" }
}

# Force merge (optimize read performance)
POST /logs-2024.01.14/_forcemerge?max_num_segments=1

# Refresh (make recent docs searchable)
POST /logs-2024.01.15/_refresh

# Check shard allocation
GET _cat/shards/logs-*?v&s=store:desc
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many shards | Daily indices with too many primaries | Use rollover, reduce shard count |
| ILM not progressing | Policy misconfigured or index not linked | Check `GET index/_ilm/explain` |
| Disk watermark reached | 85%/90%/95% thresholds | Add nodes, delete old data, adjust watermarks |
| Mapping explosion | Dynamic mapping with high-cardinality fields | Set `dynamic: strict` or use templates |
| Slow rollover | Index hasn't met rollover conditions | Check conditions, force rollover if needed |

---

## Quick Revision Questions

1. **What are the ILM phases and what actions happen in each?**
2. **What is the recommended shard size and why?**
3. **How do index templates and index patterns work together?**
4. **What is the rollover action and when does it trigger?**
5. **What happens when disk watermarks are reached?**
6. **How do you calculate the number of shards needed for a given data volume?**

---

[← Previous: ELK Stack Deployment](05-elk-stack-deployment.md) | [Back to README](../README.md) | [Next: Search & Queries →](07-search-queries.md)
