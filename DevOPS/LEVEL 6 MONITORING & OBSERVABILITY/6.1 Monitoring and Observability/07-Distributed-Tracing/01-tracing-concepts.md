# Chapter 1: Tracing Concepts

[← Previous: Scaling Considerations](../06-ELK-EFK-Stack/08-scaling-considerations.md) | [Back to README](../README.md) | [Next: OpenTelemetry →](02-opentelemetry.md)

---

## Overview

Distributed tracing tracks a single request as it flows through multiple services in a microservices architecture. It reveals the complete lifecycle of a request — which services were involved, how long each took, and where failures occurred.

---

## 1.1 Why Distributed Tracing?

```
┌──────────────────────────────────────────────────────────┐
│  THE PROBLEM: Debugging in Microservices                 │
│                                                          │
│  User: "The checkout page is slow"                       │
│                                                          │
│  Without tracing:                                        │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐    │
│  │  API  │→│ Auth  │→│ Order │→│Payment│→│ Email │    │
│  │       │ │       │ │       │ │  ???  │ │       │    │
│  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘    │
│  Which service is slow? Where did the error occur?      │
│  Check 5 different dashboards? Grep 5 different logs?   │
│                                                          │
│  With tracing:                                           │
│  Trace ID: abc123 shows EXACTLY:                        │
│  API Gateway    ████████████████████████ 450ms           │
│  ├─ Auth        ██ 20ms                                  │
│  ├─ Order Svc   ████ 40ms                                │
│  │  ├─ DB Query ██ 15ms                                  │
│  │  └─ Cache    █ 5ms                                    │
│  ├─ Payment     ██████████████████ 350ms  ← BOTTLENECK  │
│  │  └─ Stripe   ████████████████ 340ms   ← ROOT CAUSE  │
│  └─ Email       ██ 15ms                                  │
│                                                          │
│  Instantly see: Stripe API is slow (340ms of 450ms)     │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Core Concepts

```
┌──────────────────────────────────────────────────────────┐
│                TRACING TERMINOLOGY                       │
│                                                          │
│  TRACE:  End-to-end journey of a request                │
│          Identified by a unique Trace ID                 │
│          Contains multiple spans                         │
│                                                          │
│  SPAN:   A single unit of work within a trace           │
│          Has: name, start time, duration, status        │
│          Identified by a unique Span ID                  │
│          Has a parent Span ID (except root span)        │
│                                                          │
│  CONTEXT: Propagated metadata (trace_id, span_id)       │
│           Passed via HTTP headers or message metadata    │
│                                                          │
│  Trace ID: abc-123-def-456                              │
│  ┌──────────────────────────────────────────┐           │
│  │ Span A (root): API Gateway               │           │
│  │ ┌────────────────────────────────────┐    │           │
│  │ │ Span B: Auth Service                │    │           │
│  │ └────────────────────────────────────┘    │           │
│  │ ┌────────────────────────────────────┐    │           │
│  │ │ Span C: Order Service               │    │           │
│  │ │ ┌──────────────────┐               │    │           │
│  │ │ │ Span D: DB Query  │               │    │           │
│  │ │ └──────────────────┘               │    │           │
│  │ └────────────────────────────────────┘    │           │
│  └──────────────────────────────────────────┘           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.3 Span Anatomy

```json
{
  "trace_id": "abc123def456",
  "span_id": "span-001",
  "parent_span_id": null,
  "operation_name": "GET /api/checkout",
  "service_name": "api-gateway",
  "start_time": "2024-01-15T10:23:45.000Z",
  "duration_ms": 450,
  "status": "OK",
  "kind": "SERVER",
  "attributes": {
    "http.method": "GET",
    "http.url": "/api/checkout",
    "http.status_code": 200,
    "user.id": "u789"
  },
  "events": [
    {
      "name": "cache.miss",
      "timestamp": "2024-01-15T10:23:45.050Z",
      "attributes": { "key": "cart:u789" }
    }
  ],
  "links": []
}
```

### Span Kinds

| Kind | Description | Example |
|------|------------|---------|
| SERVER | Incoming request handler | HTTP endpoint receiving a request |
| CLIENT | Outgoing request maker | HTTP client calling another service |
| PRODUCER | Message queue producer | Publishing to Kafka |
| CONSUMER | Message queue consumer | Reading from Kafka |
| INTERNAL | Internal operation | DB query, cache lookup |

---

## 1.4 Context Propagation

```
┌──────────────────────────────────────────────────────────┐
│          CONTEXT PROPAGATION                             │
│                                                          │
│  Service A ──HTTP──▶ Service B ──HTTP──▶ Service C      │
│                                                          │
│  HTTP Headers carry trace context:                       │
│                                                          │
│  W3C Trace Context (standard):                          │
│  traceparent: 00-abc123def456-span001-01                │
│  tracestate: vendor=value                               │
│                                                          │
│  B3 (Zipkin format):                                    │
│  X-B3-TraceId: abc123def456                             │
│  X-B3-SpanId: span002                                   │
│  X-B3-ParentSpanId: span001                             │
│  X-B3-Sampled: 1                                        │
│                                                          │
│  Jaeger format:                                         │
│  uber-trace-id: abc123:span002:span001:1                │
│                                                          │
│  Each service:                                          │
│  1. Extracts trace context from incoming headers        │
│  2. Creates a child span                                │
│  3. Injects trace context into outgoing headers         │
└──────────────────────────────────────────────────────────┘
```

---

## 1.5 Sampling Strategies

```
┌──────────────────────────────────────────────────────────┐
│  SAMPLING: Not every request needs to be traced         │
│                                                          │
│  Head-based sampling (decide at entry):                  │
│  ├── Always-on:     100% traces (dev only)              │
│  ├── Probabilistic: 10% of requests                     │
│  ├── Rate-limiting: Max 10 traces/second                │
│  └── Guaranteed:    100% errors, 1% success             │
│                                                          │
│  Tail-based sampling (decide after trace completes):     │
│  ├── Keep all traces with errors                        │
│  ├── Keep all traces with latency > P99                 │
│  ├── Keep traces that hit specific services             │
│  └── More expensive (must buffer all spans)             │
│                                                          │
│  Volume estimation:                                      │
│  1,000 req/s × 10 spans/req × 500 bytes/span = 5 MB/s  │
│  With 10% sampling: 500 KB/s (10x reduction)            │
└──────────────────────────────────────────────────────────┘
```

---

## 1.6 Tracing Tools Landscape

| Tool | Type | Backend | Query Language |
|------|------|---------|---------------|
| Jaeger | Open source (CNCF) | ES/Cassandra/Kafka | Service/Operation |
| Zipkin | Open source | ES/Cassandra/MySQL | Service/Span |
| Tempo | Open source (Grafana) | S3/GCS/Azure Blob | TraceQL |
| X-Ray | AWS managed | AWS | Filter expressions |
| Datadog APM | SaaS | Managed | — |
| New Relic | SaaS | Managed | NRQL |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Missing spans | Context not propagated | Check HTTP headers, ensure instrumentation |
| Broken traces | Service not instrumented | Add tracing SDK/agent to all services |
| Too many traces | No sampling | Enable probabilistic or rate-limiting sampling |
| Wrong parent spans | Incorrect context extraction | Verify propagation format (W3C vs B3) |
| High trace latency | Sync span export | Use async/batch span exporters |

---

## Quick Revision Questions

1. **What is a trace and how does it differ from a span?**
2. **How does context propagation work across services?**
3. **What are the differences between W3C Trace Context and B3 formats?**
4. **Explain head-based vs tail-based sampling and when to use each.**
5. **What are span kinds and give an example of each?**
6. **Why is distributed tracing essential in microservices architectures?**

---

[← Previous: Scaling Considerations](../06-ELK-EFK-Stack/08-scaling-considerations.md) | [Back to README](../README.md) | [Next: OpenTelemetry →](02-opentelemetry.md)
