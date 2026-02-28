# Chapter 4: Grafana Tempo

[← Previous: Jaeger](03-jaeger.md) | [Back to README](../README.md) | [Next: Instrumentation Best Practices →](05-instrumentation-best-practices.md)

---

## Overview

Grafana Tempo is a high-scale, cost-effective distributed tracing backend. Unlike Jaeger or Zipkin, Tempo only requires object storage (S3, GCS, Azure Blob), making it dramatically cheaper to operate at scale.

---

## 4.1 Tempo Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    GRAFANA TEMPO                              │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Service A │  │ Service B │  │ Service C │  Applications   │
│  │ (OTLP)   │  │ (Zipkin)  │  │ (Jaeger)  │                │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
│       └──────────────┼─────────────┘                         │
│                      ▼                                       │
│  ┌──────────────────────────────────────────────────┐       │
│  │               TEMPO DISTRIBUTOR                   │       │
│  │  Accepts spans via OTLP, Jaeger, Zipkin, etc.    │       │
│  │  Distributes to ingesters based on trace ID      │       │
│  └────────────────────┬─────────────────────────────┘       │
│                       ▼                                      │
│  ┌──────────────────────────────────────────────────┐       │
│  │                TEMPO INGESTER                     │       │
│  │  Batches spans in-memory → writes blocks         │       │
│  │  WAL (Write-Ahead Log) for crash recovery        │       │
│  └────────────────────┬─────────────────────────────┘       │
│                       ▼                                      │
│  ┌──────────────────────────────────────────────────┐       │
│  │               COMPACTOR                           │       │
│  │  Merges small blocks → large blocks              │       │
│  │  Performs bloom filter creation                   │       │
│  └────────────────────┬─────────────────────────────┘       │
│                       ▼                                      │
│  ┌──────────────────────────────────────────────────┐       │
│  │          OBJECT STORAGE (Backend)                 │       │
│  │  S3 / GCS / Azure Blob / MinIO / Local FS        │       │
│  │  Stores traces as columnar blocks                 │       │
│  └──────────────────────────────────────────────────┘       │
│                       ▲                                      │
│  ┌──────────────────────────────────────────────────┐       │
│  │              TEMPO QUERIER                        │       │
│  │  Fetches from ingesters + backend                │       │
│  │  Supports TraceQL for querying                   │       │
│  └──────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Tempo vs Jaeger Comparison

| Feature | Jaeger | Tempo |
|---------|--------|-------|
| Storage | Elasticsearch/Cassandra | Object storage (S3/GCS) |
| Cost | High (indexed storage) | Low (cheap blob storage) |
| Indexing | Full-text search on spans | Trace ID lookup + TraceQL |
| Scaling | Complex (ES cluster) | Simple (stateless + object store) |
| Native Protocol | Jaeger Thrift + OTLP | OTLP + Jaeger + Zipkin |
| Query Language | Tags & filters | TraceQL |
| Grafana Integration | Plugin | Native |
| Setup Complexity | Medium-High | Low |

---

## 4.3 Deployment

### Docker Compose

```yaml
version: "3.8"
services:
  tempo:
    image: grafana/tempo:2.3.1
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
      - tempo-data:/var/tempo
    ports:
      - "3200:3200"   # Tempo HTTP
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP

  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"
    environment:
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: Admin

volumes:
  tempo-data:
```

### Tempo Configuration

```yaml
# tempo.yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: "0.0.0.0:4317"
        http:
          endpoint: "0.0.0.0:4318"
    jaeger:
      protocols:
        thrift_compact:
          endpoint: "0.0.0.0:6831"
    zipkin:
      endpoint: "0.0.0.0:9411"

ingester:
  max_block_duration: 5m

compactor:
  compaction:
    block_retention: 72h        # Keep traces for 3 days

storage:
  trace:
    backend: local              # Use s3/gcs/azure for production
    local:
      path: /var/tempo/traces
    wal:
      path: /var/tempo/wal

# Production S3 example:
# storage:
#   trace:
#     backend: s3
#     s3:
#       bucket: my-tempo-traces
#       endpoint: s3.amazonaws.com
#       region: us-east-1
#       access_key: ${AWS_ACCESS_KEY}
#       secret_key: ${AWS_SECRET_KEY}

metrics_generator:
  registry:
    external_labels:
      source: tempo
  storage:
    path: /var/tempo/generator/wal
    remote_write:
      - url: http://prometheus:9090/api/v1/write
```

### Kubernetes with Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install tempo grafana/tempo-distributed \
  --namespace tracing --create-namespace \
  --set storage.trace.backend=s3 \
  --set storage.trace.s3.bucket=my-tempo-traces \
  --set storage.trace.s3.endpoint=s3.amazonaws.com \
  --set storage.trace.s3.region=us-east-1
```

---

## 4.4 TraceQL

TraceQL is Tempo's native query language for searching traces by span attributes.

```
┌──────────────────────────────────────────────────┐
│               TraceQL BASICS                     │
│                                                  │
│  Syntax:   { <spanset filter> }                 │
│                                                  │
│  Scopes:   resource.  span.  .name  .duration   │
│                                                  │
│  Operators: =  !=  >  <  >=  <=  =~  !~        │
│                                                  │
│  Combiners: &&  ||  >>  ~                       │
│              AND  OR  descendant  sibling       │
└──────────────────────────────────────────────────┘
```

### TraceQL Examples

```bash
# Find traces with a specific service
{ resource.service.name = "api-gateway" }

# Find error spans
{ span.http.status_code >= 500 }

# Find slow database queries (> 1s)
{ span.db.system = "postgresql" && duration > 1s }

# Find traces where api-gateway calls payment-service
{ resource.service.name = "api-gateway" } >> { resource.service.name = "payment-service" }

# Find traces with errors in a specific operation
{ .name = "HTTP GET /checkout" && status = error }

# Duration-based search
{ resource.service.name = "order-service" && duration > 500ms }

# Regex match on span name
{ .name =~ "GET /api/v[0-9]+/users.*" }

# Combine conditions across spans
{ resource.service.name = "frontend" } >> { span.db.statement =~ "SELECT.*FROM orders" }

# Find traces longer than 2 seconds total
{ } | traceDuration > 2s

# Count spans matching criteria
{ status = error } | count() > 3

# Structural queries - parent/child
{ .name = "HTTP GET" } > { span.db.system = "redis" }
```

---

## 4.5 Tempo Metrics Generator

Tempo can automatically generate RED metrics from traces:

```yaml
metrics_generator:
  registry:
    external_labels:
      source: tempo
      cluster: production
  processor:
    # Generate span metrics
    span_metrics:
      dimensions:
        - service.name
        - span.name
        - http.method
        - http.status_code
    # Generate service graph metrics
    service_graphs:
      dimensions:
        - service.name
  storage:
    path: /var/tempo/generator/wal
    remote_write:
      - url: http://prometheus:9090/api/v1/write
        send_exemplars: true
```

```
┌──────────────────────────────────────────────────┐
│        METRICS FROM TRACES FLOW                  │
│                                                  │
│  Traces ──► Tempo Metrics ──► Prometheus         │
│              Generator                           │
│                                                  │
│  Generated Metrics:                              │
│  • traces_spanmetrics_calls_total                │
│  • traces_spanmetrics_latency_bucket             │
│  • traces_service_graph_request_total            │
│  • traces_service_graph_request_failed_total     │
│  • traces_service_graph_request_server_seconds   │
│                                                  │
│  Use Case: Get RED metrics without               │
│  manual instrumentation                          │
└──────────────────────────────────────────────────┘
```

---

## 4.6 Grafana Integration

### Data Source Configuration

```yaml
# Grafana provisioning: datasources.yaml
apiVersion: 1
datasources:
  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    jsonData:
      tracesToMetrics:
        datasourceUid: prometheus
        spanStartTimeShift: "-1h"
        spanEndTimeShift: "1h"
        tags:
          - key: service.name
            value: service
      tracesToLogs:
        datasourceUid: loki
        tags:
          - key: service.name
            value: service
        mappedTags:
          - key: service.name
            value: service
        mapTagNamesEnabled: true
        spanStartTimeShift: "-1h"
        spanEndTimeShift: "1h"
        filterByTraceID: true
        filterBySpanID: true
      serviceMap:
        datasourceUid: prometheus
      nodeGraph:
        enabled: true
      search:
        hide: false
      traceQuery:
        timeShiftEnabled: true
        spanStartTimeShift: "1h"
        spanEndTimeShift: "-1h"
```

### Correlating Signals in Grafana

```
┌──────────────────────────────────────────────────────────┐
│  UNIFIED OBSERVABILITY IN GRAFANA                        │
│                                                          │
│  ┌──────────┐  exemplars  ┌──────────┐                  │
│  │ Prometheus├────────────►│  Tempo   │                  │
│  │ (Metrics) │             │ (Traces) │                  │
│  └─────┬────┘             └─────┬────┘                  │
│        │                        │                        │
│        │ derived metrics        │ trace-to-logs          │
│        │ (metrics generator)    │ (service.name match)   │
│        │                        │                        │
│        ▼                        ▼                        │
│  ┌──────────────────────────────────┐                   │
│  │         GRAFANA DASHBOARD        │                   │
│  │  Metrics ◄──► Traces ◄──► Logs  │                   │
│  │                                  │                   │
│  │  Click metric spike              │                   │
│  │  → Jump to exemplar trace        │                   │
│  │  → Jump to correlated logs       │                   │
│  └──────────────────────────────────┘                   │
│                       ▲                                  │
│                       │ trace ID                         │
│                ┌──────┴─────┐                           │
│                │   Loki     │                            │
│                │  (Logs)    │                            │
│                └────────────┘                            │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Trace not found | Trace not yet flushed | Wait for ingester flush (max_block_duration) |
| Missing spans | Different trace backends | Ensure all services send to same Tempo instance |
| TraceQL returns empty | Wrong attribute scope | Use `resource.` vs `span.` prefix correctly |
| High ingester memory | Too many concurrent traces | Increase replicas, tune max_block_duration |
| Compactor errors | Object storage permissions | Check S3/GCS IAM roles and bucket policies |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Purpose | Cost-effective distributed tracing backend |
| Storage | Object storage (S3/GCS/Azure Blob) |
| Query Language | TraceQL |
| Protocols | OTLP, Jaeger, Zipkin |
| Components | Distributor → Ingester → Compactor → Querier |
| Grafana Integration | Native with traces-to-metrics/logs correlation |
| Metrics Generator | Auto-generates RED metrics from traces |

---

## Quick Revision Questions

1. **How does Tempo's storage model differ from Jaeger's, and why is it cheaper?**
2. **What is TraceQL and how do you query for error spans in a specific service?**
3. **What is the Tempo Metrics Generator and what metrics does it produce?**
4. **How do you correlate traces with metrics and logs in Grafana?**
5. **When would you choose Tempo over Jaeger for your tracing backend?**

---

[← Previous: Jaeger](03-jaeger.md) | [Back to README](../README.md) | [Next: Instrumentation Best Practices →](05-instrumentation-best-practices.md)
