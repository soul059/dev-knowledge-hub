# Chapter 3: Jaeger

[← Previous: OpenTelemetry](02-opentelemetry.md) | [Back to README](../README.md) | [Next: Grafana Tempo →](04-grafana-tempo.md)

---

## Overview

Jaeger is a CNCF graduated project for distributed tracing, originally developed by Uber. It provides trace collection, storage, and a UI for exploring traces.

---

## 3.1 Jaeger Architecture

```
┌──────────────────────────────────────────────────────────┐
│                 JAEGER ARCHITECTURE                      │
│                                                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│  │ Service A│  │ Service B│  │ Service C│ Applications   │
│  │ (OTel    │  │ (OTel    │  │ (Jaeger  │                │
│  │  SDK)    │  │  SDK)    │  │  client) │                │
│  └────┬────┘  └────┬────┘  └────┬────┘                 │
│       └──────┬─────┘             │                      │
│              │                    │                      │
│              ▼                    ▼                      │
│  ┌────────────────────────────────────────┐             │
│  │           JAEGER COLLECTOR             │             │
│  │  Validates, indexes, transforms spans  │             │
│  │  (horizontally scalable)               │             │
│  └──────────────┬─────────────────────────┘             │
│                 │                                        │
│          ┌──────┼──────┐                                │
│          ▼             ▼                                │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │ Elasticsearch│  │  Cassandra   │  Storage            │
│  │ OR Kafka     │  │  OR Memory   │                    │
│  └──────┬───────┘  └──────────────┘                    │
│         │                                               │
│         ▼                                               │
│  ┌──────────────┐                                      │
│  │ Jaeger Query │  Serves UI and API                   │
│  │ Service      │                                      │
│  └──────┬───────┘                                      │
│         │                                               │
│         ▼                                               │
│  ┌──────────────┐                                      │
│  │  Jaeger UI   │  Web interface for trace exploration │
│  │  :16686      │                                      │
│  └──────────────┘                                      │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Deployment Options

### All-in-One (Development)

```bash
# Docker - all components in one container
docker run -d --name jaeger \
  -p 6831:6831/udp \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 4317:4317 \
  -p 4318:4318 \
  jaegertracing/all-in-one:1.52

# Ports:
# 4317  - OTLP gRPC receiver
# 4318  - OTLP HTTP receiver
# 6831  - Jaeger Thrift compact (UDP)
# 14268 - Jaeger Thrift HTTP
# 16686 - Jaeger UI
```

### Production: Docker Compose

```yaml
version: "3.8"
services:
  jaeger-collector:
    image: jaegertracing/jaeger-collector:1.52
    environment:
      SPAN_STORAGE_TYPE: elasticsearch
      ES_SERVER_URLS: http://elasticsearch:9200
    ports:
      - "4317:4317"
      - "4318:4318"
      - "14268:14268"

  jaeger-query:
    image: jaegertracing/jaeger-query:1.52
    environment:
      SPAN_STORAGE_TYPE: elasticsearch
      ES_SERVER_URLS: http://elasticsearch:9200
    ports:
      - "16686:16686"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ports:
      - "9200:9200"
```

### Production: Kubernetes with Helm

```bash
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm install jaeger jaegertracing/jaeger \
  --namespace tracing --create-namespace \
  --set storage.type=elasticsearch \
  --set storage.elasticsearch.host=elasticsearch.elastic.svc \
  --set collector.replicaCount=3 \
  --set query.replicaCount=2
```

---

## 3.3 Jaeger UI Features

```
┌──────────────────────────────────────────────────────────┐
│  JAEGER UI - TRACE SEARCH                                │
│                                                          │
│  Service: [api-gateway ▼]  Operation: [GET /checkout ▼] │
│  Tags: error=true           Lookback: [Last 1 Hour ▼]   │
│  Min Duration: 500ms        Max Results: 20              │
│                                                          │
│  [Find Traces]                                           │
│                                                          │
│  Results:                                                │
│  ┌────────────────────────────────────────────────────┐  │
│  │ Trace abc123  │ 5 spans │ 450ms │ 3 services      │  │
│  │ ████████████████████████████████████████████       │  │
│  ├────────────────────────────────────────────────────┤  │
│  │ Trace def456  │ 8 spans │ 1.2s  │ 5 services      │  │
│  │ ██████████████████████████████████████████████████ │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  TRACE DETAIL VIEW:                                      │
│  api-gateway      ████████████████████████████ 450ms     │
│  ├─ auth-service  ████ 45ms                              │
│  ├─ order-service ████████████ 120ms                     │
│  │  ├─ db-query   ██ 25ms                                │
│  │  └─ cache-get  █ 8ms                                  │
│  └─ payment-svc   █████████████████████ 280ms            │
│     └─ stripe-api ████████████████████ 270ms ⚠ SLOW     │
│                                                          │
│  Features:                                               │
│  • Timeline view (Gantt chart)                          │
│  • Span detail (attributes, logs, warnings)             │
│  • Compare traces side-by-side                          │
│  • Service dependency graph (DAG)                       │
│  • Deep linking by trace ID                             │
└──────────────────────────────────────────────────────────┘
```

---

## 3.4 Storage Backends

| Backend | Pros | Cons | Best For |
|---------|------|------|----------|
| Memory | Zero setup | No persistence, limited | Dev/testing |
| Elasticsearch | Full-text search, ILM | Resource hungry | Production (search-heavy) |
| Cassandra | Linear scalability | No full-text search | High-volume production |
| Kafka + ES | Buffered, reliable | Complex setup | Enterprise |
| Badger | Embedded, lightweight | Single-node only | Small deployments |

---

## 3.5 Jaeger with OpenTelemetry

```yaml
# OTel Collector config sending traces to Jaeger
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"

exporters:
  jaeger:
    endpoint: "jaeger-collector:14250"
    tls:
      insecure: true

  # OR use OTLP (Jaeger supports OTLP natively since v1.35)
  otlp/jaeger:
    endpoint: "jaeger-collector:4317"
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [otlp/jaeger]
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| No traces in UI | Wrong collector endpoint | Verify endpoint and port in SDK/OTel config |
| Traces incomplete | Not all services instrumented | Add OTel to all services in the call chain |
| UI slow | ES storage overloaded | Add resources, configure ILM retention |
| "Service unknown" | Service name not set | Set `service.name` in OTel resource |
| Collector dropping spans | Collector overwhelmed | Scale collector horizontally, add Kafka buffer |

---

## Quick Revision Questions

1. **What are the main components of the Jaeger architecture?**
2. **What storage backends does Jaeger support and when would you use each?**
3. **How do you deploy Jaeger in development vs production?**
4. **What features does the Jaeger UI provide for trace analysis?**
5. **How does Jaeger integrate with OpenTelemetry?**
6. **How would you scale Jaeger for high-volume trace collection?**

---

[← Previous: OpenTelemetry](02-opentelemetry.md) | [Back to README](../README.md) | [Next: Grafana Tempo →](04-grafana-tempo.md)
