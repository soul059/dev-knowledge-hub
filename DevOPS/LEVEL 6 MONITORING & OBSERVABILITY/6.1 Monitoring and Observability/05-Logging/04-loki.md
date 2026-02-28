# Chapter 4: Loki — Log Aggregation

[← Previous: Logging in Kubernetes](03-logging-in-kubernetes.md) | [Back to README](../README.md) | [Next: Log Aggregation Patterns →](05-log-aggregation-patterns.md)

---

## Overview

Grafana Loki is a horizontally scalable, multi-tenant log aggregation system inspired by Prometheus. Unlike Elasticsearch, Loki indexes only labels (metadata), not the full log content, making it cost-effective.

---

## 4.1 Loki Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   LOKI ARCHITECTURE                      │
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │ Promtail  │  │Fluent Bit│  │ Fluentd  │  Agents     │
│  └─────┬────┘  └────┬─────┘  └────┬─────┘              │
│        │             │             │                     │
│        └─────────────┼─────────────┘                     │
│                      ▼                                   │
│  ┌──────────────────────────────────────────────────┐   │
│  │                LOKI CLUSTER                       │   │
│  │                                                    │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  │   │
│  │  │ Distributor │─▶│  Ingester  │─▶│  Compactor │  │   │
│  │  │ (receives   │  │ (writes    │  │ (merges    │  │   │
│  │  │  log lines) │  │  chunks)   │  │  chunks)   │  │   │
│  │  └────────────┘  └──────┬─────┘  └────────────┘  │   │
│  │                         │                          │   │
│  │                         ▼                          │   │
│  │              ┌──────────────────┐                  │   │
│  │              │ Object Storage   │                  │   │
│  │              │ (S3/GCS/MinIO)   │                  │   │
│  │              │ - Chunks (log    │                  │   │
│  │              │   data)          │                  │   │
│  │              │ - Index (labels) │                  │   │
│  │              └──────────────────┘                  │   │
│  │                                                    │   │
│  │  ┌────────────┐                                   │   │
│  │  │  Querier   │  Reads chunks & index for queries │   │
│  │  └────────────┘                                   │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Loki vs Elasticsearch

| Feature | Loki | Elasticsearch |
|---------|------|---------------|
| Indexing | Labels only | Full-text index |
| Storage cost | Low (10-100x cheaper) | High (full index) |
| Query language | LogQL | Lucene/KQL |
| Query speed | Fast for label queries | Fast for any text search |
| Complexity | Simple | Complex (cluster mgmt) |
| Ideal for | Cloud-native, Kubernetes | Enterprise, complex search |
| Correlation | Native Grafana integration | via Kibana |

---

## 4.3 Installation

### Docker Compose

```yaml
version: "3.8"
services:
  loki:
    image: grafana/loki:2.9.0
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/config.yaml
      - loki-data:/loki
    command: -config.file=/etc/loki/config.yaml

  promtail:
    image: grafana/promtail:2.9.0
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yml:/etc/promtail/config.yaml
    command: -config.file=/etc/promtail/config.yaml

  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"

volumes:
  loki-data:
```

### Loki Config

```yaml
# loki-config.yml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v12
      index:
        prefix: index_
        period: 24h

limits_config:
  max_query_length: 721h      # 30 days
  max_entries_limit_per_query: 5000

compactor:
  working_directory: /loki/compactor
  retention_enabled: true
  retention_delete_delay: 2h
  delete_request_store: filesystem

chunk_store_config:
  max_look_back_period: 720h   # 30 days retention
```

### Promtail Config

```yaml
# promtail-config.yml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: containerlogs
          __path__: /var/log/containers/*.log

  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog
```

---

## 4.4 LogQL — Loki Query Language

### Log Stream Selectors

```logql
# Select by exact label match
{job="api-server"}

# Multiple labels
{job="api-server", namespace="production"}

# Regex match
{job=~"api.*"}

# Negation
{job="api-server", level!="debug"}
```

### Log Pipeline (Filter & Parse)

```logql
# Line filter (contains)
{job="api-server"} |= "error"

# Line filter (does not contain)
{job="api-server"} != "health"

# Regex filter
{job="api-server"} |~ "status=[45]\\d{2}"

# JSON parsing
{job="api-server"} | json | status >= 400

# Pattern parsing
{job="nginx"} | pattern `<ip> - - [<_>] "<method> <path> <_>" <status> <size>`

# Logfmt parsing
{job="api"} | logfmt | duration > 1s

# Label filter after parsing
{job="api-server"} | json | level="error" | line_format "{{.message}}"
```

### Metric Queries (from logs)

```logql
# Count of error logs per minute
count_over_time({job="api-server"} |= "error" [1m])

# Rate of log lines
rate({job="api-server"}[5m])

# Bytes rate
bytes_rate({job="api-server"}[5m])

# Quantile of extracted duration
{job="api-server"} | json | unwrap duration_ms
quantile_over_time(0.99, {job="api-server"} | json | unwrap duration_ms [5m])

# Top 5 namespaces by log volume
topk(5, sum by (namespace) (bytes_rate({job="containers"}[5m])))
```

---

## 4.5 Loki in Grafana

```
Grafana Explore:
  1. Select Loki data source
  2. Use Log Browser to pick labels
  3. Write LogQL query
  4. View: 
     - Log lines (raw text)
     - Log volume graph (histogram)
     - Log context (surrounding lines)
     - Detected fields (parsed JSON fields)
  5. Click trace_id → jump to Tempo trace
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Too many streams" | High cardinality labels | Remove dynamic labels (e.g., request_id) |
| Slow queries | Querying too many streams or long range | Narrow label selectors and time range |
| Missing logs | Promtail not scraping path | Check `__path__` config and permissions |
| Out of order | Logs arriving with old timestamps | Enable `unordered_writes` in ingester |
| High storage | No retention configured | Set retention in compactor config |

---

## Quick Revision Questions

1. **How does Loki's indexing strategy differ from Elasticsearch?**
2. **What are the main components of the Loki architecture?**
3. **Write a LogQL query to find all error logs from the api-server in production.**
4. **How do you extract metrics from logs using LogQL?**
5. **What causes "too many streams" errors and how do you fix it?**
6. **How does Promtail discover and ship logs to Loki?**

---

[← Previous: Logging in Kubernetes](03-logging-in-kubernetes.md) | [Back to README](../README.md) | [Next: Log Aggregation Patterns →](05-log-aggregation-patterns.md)
