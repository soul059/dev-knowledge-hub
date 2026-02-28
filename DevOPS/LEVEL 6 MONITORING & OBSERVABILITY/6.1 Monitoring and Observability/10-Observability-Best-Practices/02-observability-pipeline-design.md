# Chapter 2: Observability Pipeline Design

[← Previous: Instrumentation Guidelines](01-instrumentation-guidelines.md) | [Back to README](../README.md) | [Next: Cost Optimization →](03-cost-optimization.md)

---

## Overview

An observability pipeline is the infrastructure that collects, processes, routes, and stores telemetry data. A well-designed pipeline handles scale, reduces costs, ensures reliability, and enables flexibility to change backends without re-instrumenting applications.

---

## 2.1 Pipeline Architecture

```
┌──────────────────────────────────────────────────────────────┐
│           OBSERVABILITY PIPELINE ARCHITECTURE                │
│                                                              │
│  DATA SOURCES          COLLECTION        PROCESSING          │
│  ┌──────────┐         ┌──────────┐      ┌──────────┐       │
│  │ App Metrics│───────►│          │      │          │       │
│  └──────────┘         │  Agent   │─────►│ Central  │       │
│  ┌──────────┐         │  Layer   │      │ Pipeline │       │
│  │ App Logs  │───────►│          │      │          │       │
│  └──────────┘         │ (OTel   │      │ (OTel    │       │
│  ┌──────────┐         │  Coll.  │      │  Coll.   │       │
│  │ App Traces│───────►│  Agent  │      │  Gateway)│       │
│  └──────────┘         │  mode)  │      │          │       │
│  ┌──────────┐         └──────────┘      └────┬─────┘       │
│  │ Infra    │              │                  │             │
│  │ Metrics  │──────────────┘                  │             │
│  └──────────┘                                 │             │
│                                               │             │
│  ROUTING                    STORAGE                         │
│  ┌──────────────────────────┼──────────────────────────┐   │
│  │                          │                          │   │
│  │  ┌───────────────┐       ▼                          │   │
│  │  │               │  ┌──────────┐                    │   │
│  │  │  Metrics ────►│──│Prometheus│                    │   │
│  │  │               │  │/Mimir    │                    │   │
│  │  │  Traces ─────►│  └──────────┘                    │   │
│  │  │               │  ┌──────────┐                    │   │
│  │  │  Logs ───────►│──│ Tempo    │                    │   │
│  │  │               │  └──────────┘                    │   │
│  │  │  Profiles ───►│  ┌──────────┐                    │   │
│  │  │               │──│ Loki     │                    │   │
│  │  └───────────────┘  └──────────┘                    │   │
│  │                     ┌──────────┐                    │   │
│  │                     │Pyroscope │                    │   │
│  │                     └──────────┘                    │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 OTel Collector as Pipeline Foundation

### Agent + Gateway Pattern

```
┌──────────────────────────────────────────────────────────┐
│  TWO-TIER COLLECTOR ARCHITECTURE                         │
│                                                          │
│  Node 1:                    Node 2:                     │
│  ┌────────────┐            ┌────────────┐               │
│  │ App A      │            │ App C      │               │
│  │ App B      │            │ App D      │               │
│  │            │            │            │               │
│  │ ┌────────┐ │            │ ┌────────┐ │               │
│  │ │OTel    │ │            │ │OTel    │ │               │
│  │ │Agent   │ │            │ │Agent   │ │               │
│  │ └───┬────┘ │            │ └───┬────┘ │               │
│  └─────┼──────┘            └─────┼──────┘               │
│        │                         │                       │
│        └─────────┬───────────────┘                       │
│                  ▼                                       │
│  ┌──────────────────────────────────┐                   │
│  │   OTel Collector GATEWAY (HA)    │                   │
│  │   (3 replicas, load balanced)    │                   │
│  │                                   │                   │
│  │   Processing:                     │                   │
│  │   • Batch, compress              │                   │
│  │   • Filter, transform            │                   │
│  │   • Tail sampling (traces)       │                   │
│  │   • Route to backends            │                   │
│  └────────────┬─────────────────────┘                   │
│               │                                          │
│      ┌────────┼────────┬─────────┐                      │
│      ▼        ▼        ▼         ▼                      │
│  Prometheus  Tempo    Loki    S3/Archive                │
└──────────────────────────────────────────────────────────┘
```

### Gateway Configuration

```yaml
# otel-collector-gateway.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
      http:
        endpoint: "0.0.0.0:4318"

processors:
  # Batch for efficiency
  batch:
    send_batch_size: 10000
    timeout: 5s

  # Memory limiter to prevent OOM
  memory_limiter:
    check_interval: 1s
    limit_mib: 4096
    spike_limit_mib: 1024

  # Filter unwanted data
  filter/logs:
    logs:
      exclude:
        match_type: regexp
        bodies:
          - "health check"
          - "readiness probe"

  # Add metadata
  resource:
    attributes:
      - key: pipeline.name
        value: "central-gateway"
        action: upsert

  # Transform attributes
  transform:
    metric_statements:
      - context: datapoint
        statements:
          - truncate_all(attributes, 128)

  # Tail sampling for traces
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors
        type: status_code
        status_code:
          status_codes: [ERROR]
      - name: slow
        type: latency
        latency:
          threshold_ms: 2000
      - name: sample-rest
        type: probabilistic
        probabilistic:
          sampling_percentage: 10

exporters:
  prometheusremotewrite:
    endpoint: "http://mimir:9009/api/v1/push"
  
  otlp/tempo:
    endpoint: "tempo:4317"
    tls:
      insecure: true
  
  loki:
    endpoint: "http://loki:3100/loki/api/v1/push"
  
  # Archive to S3 for compliance
  awss3:
    s3uploader:
      region: us-east-1
      s3_bucket: telemetry-archive
      s3_prefix: raw

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch, transform]
      exporters: [prometheusremotewrite]
    traces:
      receivers: [otlp]
      processors: [memory_limiter, tail_sampling, batch]
      exporters: [otlp/tempo]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, filter/logs, batch]
      exporters: [loki]
```

---

## 2.3 Pipeline Reliability Patterns

```
┌──────────────────────────────────────────────────────────┐
│  RELIABILITY PATTERNS                                    │
│                                                          │
│  1. BUFFERING (Handle backend downtime)                 │
│  Apps ──► Agent (disk buffer) ──► Gateway ──► Backend   │
│                  ↕                                       │
│           Local WAL/queue                                │
│                                                          │
│  2. BACKPRESSURE (Handle overload)                      │
│  Collector returns 429 → Agent retries with backoff    │
│  Memory limiter drops data before OOM                   │
│                                                          │
│  3. KAFKA BUFFER (Enterprise scale)                     │
│  Apps ──► Agent ──► Kafka ──► Consumer ──► Backend     │
│                      ↕                                   │
│               Replay/reprocess                          │
│                                                          │
│  4. DUAL WRITE (Migration / redundancy)                 │
│  Gateway ──► Backend A (primary)                        │
│         └──► Backend B (secondary / new)                │
└──────────────────────────────────────────────────────────┘
```

### Kafka Buffer Pipeline

```yaml
# OTel Collector → Kafka → OTel Collector → Backend
# Producer collector
exporters:
  kafka:
    brokers:
      - kafka:9092
    topic: otlp_spans
    encoding: otlp_proto

# Consumer collector
receivers:
  kafka:
    brokers:
      - kafka:9092
    topic: otlp_spans
    encoding: otlp_proto
    group_id: otel-consumer

exporters:
  otlp/tempo:
    endpoint: "tempo:4317"
```

---

## 2.4 Multi-Tenant Pipeline

```
┌──────────────────────────────────────────────────────────┐
│  MULTI-TENANT PIPELINE                                   │
│                                                          │
│  Team A ──► ┌──────────────────────────┐ ──► Team A DB  │
│             │                          │                 │
│  Team B ──► │   Central Gateway        │ ──► Team B DB  │
│             │   (route by headers/     │                 │
│  Team C ──► │    labels)               │ ──► Team C DB  │
│             │                          │                 │
│  Shared  ──►│                          │ ──► Shared DB  │
│             └──────────────────────────┘                 │
│                                                          │
│  Routing by:                                            │
│  • X-Scope-OrgID header (Mimir/Loki/Tempo)             │
│  • Resource attribute: team.name                        │
│  • Namespace label (Kubernetes)                         │
└──────────────────────────────────────────────────────────┘
```

---

## 2.5 Pipeline Monitoring

```yaml
# Monitor the pipeline itself
# OTel Collector exposes these metrics:
pipeline_metrics:
  - otelcol_receiver_accepted_spans       # Spans received
  - otelcol_receiver_refused_spans        # Spans rejected
  - otelcol_processor_dropped_spans       # Spans dropped
  - otelcol_exporter_sent_spans           # Spans exported
  - otelcol_exporter_send_failed_spans    # Failed exports
  - otelcol_process_memory_rss            # Memory usage
  - otelcol_processor_batch_batch_size_trigger_send  # Batch sizes

# Alert rules for pipeline health
groups:
  - name: pipeline-health
    rules:
      - alert: CollectorDroppingData
        expr: rate(otelcol_processor_dropped_spans[5m]) > 0
        for: 5m
        annotations:
          summary: "OTel Collector is dropping spans"

      - alert: CollectorExportFailure
        expr: rate(otelcol_exporter_send_failed_spans[5m]) > 0
        for: 5m
        annotations:
          summary: "OTel Collector failing to export spans"

      - alert: CollectorHighMemory
        expr: otelcol_process_memory_rss > 3.5e9  # 3.5GB of 4GB limit
        for: 2m
        annotations:
          summary: "OTel Collector memory approaching limit"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Data loss during peaks | No buffer | Add disk buffer or Kafka |
| Collector OOM | No memory limiter | Configure memory_limiter processor |
| Backend slow ingestion | No batching | Configure batch processor |
| Can't change backend | Direct app-to-backend | Add OTel Collector as intermediary |
| Pipeline bottleneck | Single gateway | Scale gateway horizontally |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Pipeline Layers | Agent → Gateway → Backend |
| Key Tool | OpenTelemetry Collector |
| Reliability | Buffering, backpressure, Kafka, dual-write |
| Scaling | Horizontal gateway replicas, Kafka partitions |
| Multi-tenant | Route by headers, labels, or namespace |
| Monitoring | Monitor collector metrics, alert on drops |

---

## Quick Revision Questions

1. **What is the agent + gateway collector pattern and why is it preferred?**
2. **How does the OTel Collector handle backpressure and prevent data loss?**
3. **When should you add Kafka to your observability pipeline?**
4. **How do you monitor the health of the pipeline itself?**
5. **How do you design a multi-tenant observability pipeline?**

---

[← Previous: Instrumentation Guidelines](01-instrumentation-guidelines.md) | [Back to README](../README.md) | [Next: Cost Optimization →](03-cost-optimization.md)
