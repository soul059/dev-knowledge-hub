# Chapter 7: Search & Queries

[← Previous: Index Management](06-index-management.md) | [Back to README](../README.md) | [Next: Scaling Considerations →](08-scaling-considerations.md)

---

## Overview

Elasticsearch provides powerful query capabilities including full-text search, structured queries, aggregations, and complex boolean logic. Mastering the Query DSL is essential for effective log analysis.

---

## 7.1 Query DSL Structure

```
┌──────────────────────────────────────────────────────────┐
│            QUERY DSL ANATOMY                             │
│                                                          │
│  GET /index/_search                                      │
│  {                                                       │
│    "query": { ... },       ← What to find               │
│    "sort": [ ... ],        ← How to order                │
│    "from": 0, "size": 10,  ← Pagination                 │
│    "aggs": { ... },        ← Aggregations                │
│    "_source": [ ... ],     ← Fields to return            │
│    "highlight": { ... }    ← Highlight matches           │
│  }                                                       │
│                                                          │
│  Two contexts:                                           │
│  ├── Query context:  "How well does this match?"        │
│  │   (scoring, relevance)                                │
│  └── Filter context: "Does this match? Yes/No"         │
│      (no scoring, faster, cacheable)                     │
└──────────────────────────────────────────────────────────┘
```

---

## 7.2 Common Query Types

### Match Query (Full-Text)
```json
GET /logs-*/_search
{
  "query": {
    "match": {
      "message": "payment timeout"
    }
  }
}
```

### Term Query (Exact Match)
```json
{
  "query": {
    "term": {
      "level": "ERROR"
    }
  }
}
```

### Bool Query (Compound)
```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "message": "timeout" } }
      ],
      "filter": [
        { "term": { "level": "ERROR" } },
        { "term": { "service": "api-server" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ],
      "must_not": [
        { "term": { "service": "health-checker" } }
      ],
      "should": [
        { "term": { "environment": "production" } }
      ],
      "minimum_should_match": 1
    }
  }
}
```

### Range Query
```json
{
  "query": {
    "range": {
      "duration_ms": {
        "gte": 1000,
        "lt": 5000
      }
    }
  }
}
```

### Wildcard & Regex
```json
{
  "query": {
    "wildcard": { "service": "api-*" }
  }
}

{
  "query": {
    "regexp": { "trace_id": "[a-f0-9]{32}" }
  }
}
```

---

## 7.3 Aggregations

```json
GET /logs-*/_search
{
  "size": 0,
  "query": {
    "range": { "@timestamp": { "gte": "now-24h" } }
  },
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "service",
        "size": 10,
        "order": { "_count": "desc" }
      },
      "aggs": {
        "error_types": {
          "terms": { "field": "error_type", "size": 5 }
        },
        "avg_duration": {
          "avg": { "field": "duration_ms" }
        }
      }
    },
    "errors_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "1h"
      }
    },
    "latency_percentiles": {
      "percentiles": {
        "field": "duration_ms",
        "percents": [50, 90, 95, 99]
      }
    }
  }
}
```

### Aggregation Types

| Type | Purpose | Example |
|------|---------|---------|
| `terms` | Group by field value | Errors by service |
| `date_histogram` | Group by time buckets | Errors per hour |
| `avg/sum/min/max` | Numeric calculations | Avg duration |
| `percentiles` | Distribution stats | P50, P95, P99 |
| `cardinality` | Unique count (approx) | Unique users |
| `top_hits` | Sample docs per bucket | Latest error per service |
| `filters` | Named filter buckets | Success vs failure |

---

## 7.4 Practical Query Patterns

```json
// 1. Find all 5xx errors in the last hour
{
  "query": {
    "bool": {
      "filter": [
        { "range": { "status_code": { "gte": 500, "lt": 600 } } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "sort": [{ "@timestamp": "desc" }]
}

// 2. Top 10 slowest endpoints
{
  "size": 0,
  "aggs": {
    "slow_endpoints": {
      "terms": { "field": "http.path", "size": 10 },
      "aggs": {
        "p99_duration": {
          "percentiles": { 
            "field": "duration_ms", 
            "percents": [99] 
          }
        },
        "sort_by_p99": {
          "bucket_sort": {
            "sort": [{ "p99_duration.99.0": { "order": "desc" } }]
          }
        }
      }
    }
  }
}

// 3. Error rate trend
{
  "size": 0,
  "aggs": {
    "timeline": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "5m"
      },
      "aggs": {
        "total": { "value_count": { "field": "_id" } },
        "errors": {
          "filter": { "term": { "level": "ERROR" } }
        },
        "error_rate": {
          "bucket_script": {
            "buckets_path": { "err": "errors._count", "tot": "total" },
            "script": "params.err / params.tot * 100"
          }
        }
      }
    }
  }
}
```

---

## 7.5 Search Performance Tips

```
┌──────────────────────────────────────────────────────────┐
│  QUERY PERFORMANCE OPTIMIZATION                          │
│                                                          │
│  1. Use filters over queries (cacheable, no scoring)     │
│     ✅ "filter": [{ "term": { "level": "ERROR" } }]    │
│     ❌ "must": [{ "match": { "level": "ERROR" } }]     │
│                                                          │
│  2. Narrow time range (less data to scan)                │
│     ✅ "range": { "@timestamp": { "gte": "now-1h" } }  │
│     ❌ Search all data without time filter               │
│                                                          │
│  3. Use keyword fields for exact match                   │
│     ✅ "term": { "service.keyword": "api" }             │
│     ❌ "match": { "service": "api" }                    │
│                                                          │
│  4. Limit returned fields                                │
│     ✅ "_source": ["@timestamp", "message", "level"]    │
│     ❌ Return full documents                            │
│                                                          │
│  5. Use routing for query isolation                      │
│     Route by tenant/service to target specific shards    │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Query timeout | Scanning too much data | Add time range filter, use smaller index |
| No results | Wrong field type (text vs keyword) | Use `.keyword` for exact match on text fields |
| Slow aggregations | High cardinality terms | Increase shard count, use `execution_hint: map` |
| Partial results | Some shards failing | Check `_shards.failures` in response |
| Memory errors | Large aggregations | Use `composite` aggregation for pagination |

---

## Quick Revision Questions

1. **What is the difference between query context and filter context?**
2. **When should you use `term` vs `match` queries?**
3. **Write a bool query to find ERROR logs from api-server in the last hour.**
4. **What are the main aggregation types and when would you use each?**
5. **How do you calculate error rate over time using aggregations?**
6. **What techniques improve Elasticsearch query performance?**

---

[← Previous: Index Management](06-index-management.md) | [Back to README](../README.md) | [Next: Scaling Considerations →](08-scaling-considerations.md)
