# Chapter 1: Instrumentation Guidelines

[← Previous: Real User Monitoring](../09-APM/05-real-user-monitoring.md) | [Back to README](../README.md) | [Next: Observability Pipeline Design →](02-observability-pipeline-design.md)

---

## Overview

Instrumentation is the foundation of observability. This chapter provides comprehensive guidelines for instrumenting applications with metrics, logs, and traces — covering what to instrument, naming conventions, and organizational standards.

---

## 1.1 Instrumentation Strategy

```
┌──────────────────────────────────────────────────────────┐
│         INSTRUMENTATION MATURITY LEVELS                  │
│                                                          │
│  Level 1: BASIC                                         │
│  ├─ Health check endpoints (/health, /ready)            │
│  ├─ HTTP request rate, errors, duration (RED)           │
│  ├─ Application logs (structured, JSON)                 │
│  └─ Infrastructure metrics (CPU, memory)                │
│                                                          │
│  Level 2: STANDARD                                      │
│  ├─ Distributed traces (auto-instrumentation)           │
│  ├─ Database query metrics                              │
│  ├─ Cache hit/miss rates                                │
│  ├─ Queue depth and processing time                     │
│  └─ Error tracking with stack traces                    │
│                                                          │
│  Level 3: ADVANCED                                      │
│  ├─ Custom business metrics (orders/sec, revenue)       │
│  ├─ Custom trace spans for business operations          │
│  ├─ SLI/SLO instrumentation                            │
│  ├─ Feature flag tracking                               │
│  └─ Continuous profiling                                │
│                                                          │
│  Level 4: EXPERT                                        │
│  ├─ Exemplars linking metrics → traces                 │
│  ├─ Real User Monitoring (RUM)                          │
│  ├─ Synthetic monitoring                                │
│  ├─ Trace-driven testing                                │
│  └─ Observability-driven development                    │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Metrics Instrumentation Standards

### What Every Service MUST Expose

```yaml
# Required metrics for every service
required_metrics:
  # RED metrics (Request-oriented)
  - name: http_requests_total
    type: counter
    labels: [method, path, status_code]
    description: "Total HTTP requests"

  - name: http_request_duration_seconds
    type: histogram
    labels: [method, path]
    buckets: [0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
    description: "HTTP request duration"

  - name: http_requests_in_flight
    type: gauge
    description: "Current in-flight requests"

  # USE metrics (Resource-oriented)
  - name: process_cpu_seconds_total
    type: counter
    description: "CPU time consumed"

  - name: process_resident_memory_bytes
    type: gauge
    description: "Resident memory size"

  # Application health
  - name: app_info
    type: gauge
    labels: [version, commit, build_time]
    description: "Application metadata (always 1)"

  # Dependency health
  - name: dependency_up
    type: gauge
    labels: [dependency, type]
    description: "1 if dependency reachable, 0 if not"
```

### Naming Convention Cheat Sheet

```
┌──────────────────────────────────────────────────────────┐
│  METRIC NAMING RULES                                     │
│                                                          │
│  Format: <namespace>_<subsystem>_<name>_<unit>          │
│                                                          │
│  Examples:                                               │
│  ✓ http_request_duration_seconds                        │
│  ✓ order_service_orders_created_total                   │
│  ✓ db_pool_connections_active                           │
│  ✓ cache_hit_total                                      │
│                                                          │
│  ✗ requestDuration (camelCase)                          │
│  ✗ http.request.latency (dots)                          │
│  ✗ requests (no unit, ambiguous)                        │
│                                                          │
│  Unit suffixes:                                          │
│  _seconds, _bytes, _total, _ratio, _info               │
│                                                          │
│  Label rules:                                            │
│  ✓ method="GET", status_code="200"                      │
│  ✗ user_id="12345" (high cardinality!)                  │
│  ✗ email="user@example.com" (PII!)                      │
└──────────────────────────────────────────────────────────┘
```

---

## 1.3 Logging Standards

### Structured Log Format

```json
{
  "timestamp": "2024-01-15T14:05:23.456Z",
  "level": "error",
  "message": "Failed to process order",
  "service": "order-service",
  "version": "v2.3.1",
  "environment": "production",
  "trace_id": "abc123def456",
  "span_id": "789ghi012",
  "request_id": "req-xxx-yyy",
  "error": {
    "type": "PaymentDeclinedError",
    "message": "Insufficient funds",
    "stack": "..."
  },
  "context": {
    "order_id": "ORD-2024-001",
    "customer_tier": "premium",
    "amount": 99.99
  }
}
```

### Log Level Guidelines

| Level | When to Use | Example |
|-------|------------|---------|
| ERROR | Unexpected failure, needs attention | Database connection failed |
| WARN | Recoverable issue, potential problem | Retry attempt 2 of 3 |
| INFO | Business events, state changes | Order created, user logged in |
| DEBUG | Diagnostic details for debugging | Query parameters, cache key |
| TRACE | Very detailed, rarely used in prod | Function entry/exit |

### What NOT to Log

```
┌──────────────────────────────────────────┐
│  NEVER LOG THESE:                        │
│                                          │
│  ✗ Passwords or secrets                 │
│  ✗ Credit card numbers                  │
│  ✗ Social security numbers              │
│  ✗ Full API keys                        │
│  ✗ Personal health information          │
│  ✗ Unmasked email addresses (in some cases) │
│  ✗ Session tokens                       │
│                                          │
│  MASK/REDACT:                            │
│  • Card: ****-****-****-1234            │
│  • API key: sk_live_****abcd            │
│  • Email: k***@example.com             │
└──────────────────────────────────────────┘
```

---

## 1.4 Tracing Standards

### Required Resource Attributes

```python
from opentelemetry.sdk.resources import Resource

resource = Resource.create({
    "service.name": "order-service",          # REQUIRED
    "service.version": "v2.3.1",              # REQUIRED
    "deployment.environment": "production",    # REQUIRED
    "service.namespace": "ecommerce",          # Recommended
    "service.instance.id": "pod-abc123",       # Recommended
    "cloud.provider": "aws",                   # If applicable
    "cloud.region": "us-east-1",               # If applicable
    "k8s.pod.name": "order-svc-abc123",        # If K8s
    "k8s.namespace.name": "production",        # If K8s
})
```

### Span Creation Guidelines

```
┌──────────────────────────────────────────────────────────┐
│  WHEN TO CREATE A SPAN                                   │
│                                                          │
│  ALWAYS create spans for:                               │
│  ✓ Incoming HTTP/gRPC requests (auto-instrumented)      │
│  ✓ Outgoing HTTP/gRPC calls (auto-instrumented)         │
│  ✓ Database queries (auto-instrumented)                 │
│  ✓ Message queue publish/consume                        │
│  ✓ External service calls                               │
│  ✓ Key business operations (manual)                     │
│                                                          │
│  NEVER create spans for:                                │
│  ✗ Every function call                                  │
│  ✗ In-memory operations                                │
│  ✗ Simple getter/setter methods                         │
│  ✗ Operations completing in < 1ms                       │
│                                                          │
│  Rule: If it crosses a boundary (network, disk, queue)  │
│  OR is a key business operation → CREATE A SPAN         │
└──────────────────────────────────────────────────────────┘
```

---

## 1.5 Service Instrumentation Template

```python
# Standard service instrumentation template (Python/FastAPI)
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor
from opentelemetry.instrumentation.redis import RedisInstrumentor
from opentelemetry.instrumentation.httpx import HTTPXClientInstrumentor
import structlog
import logging

# === TRACES ===
trace.set_tracer_provider(TracerProvider(resource=resource))
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="otel-collector:4317"))
)

# === METRICS ===
metrics.set_meter_provider(MeterProvider(resource=resource))
meter = metrics.get_meter("order-service")

# Custom business metrics
orders_counter = meter.create_counter(
    "orders_created_total",
    description="Total orders created"
)
order_value = meter.create_histogram(
    "order_value_dollars",
    description="Order value in dollars"
)

# === LOGS ===
structlog.configure(
    processors=[
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        # Add trace context to logs
        add_trace_context,
        structlog.processors.JSONRenderer()
    ]
)

# === AUTO-INSTRUMENTATION ===
FastAPIInstrumentor.instrument_app(app)
SQLAlchemyInstrumentor().instrument()
RedisInstrumentor().instrument()
HTTPXClientInstrumentor().instrument()
```

---

## 1.6 Organizational Standards

| Area | Standard | Enforcement |
|------|----------|-------------|
| Metric naming | snake_case with unit suffix | Linting in CI |
| Log format | Structured JSON | Logger library config |
| Trace resource attrs | service.name, version, env required | OTel SDK validation |
| Health endpoints | `/health`, `/ready` on all services | K8s probe config |
| Cardinality limit | Max 10 labels, no user IDs | Review process |
| Error handling | record_exception() on all errors | Code review checklist |
| Sampling | Minimum 10% in production | OTel Collector config |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Inconsistent metrics | No naming standard | Define and enforce naming conventions |
| Missing trace context in logs | Log library not configured | Add OTel trace context processor |
| Cardinality explosion | User ID in labels | Move dynamic values to log/trace attributes |
| Too many metrics | No review process | Quarterly metric review, remove unused |
| Instrumentation gaps | No service checklist | Create and enforce instrumentation template |

---

## Quick Revision Questions

1. **What metrics should every service expose at minimum?**
2. **What are the rules for metric naming and labels?**
3. **What should a structured log entry include?**
4. **When should you create a custom trace span vs rely on auto-instrumentation?**
5. **How do you enforce instrumentation standards across an organization?**
6. **What sensitive data should never appear in logs or trace attributes?**

---

[← Previous: Real User Monitoring](../09-APM/05-real-user-monitoring.md) | [Back to README](../README.md) | [Next: Observability Pipeline Design →](02-observability-pipeline-design.md)
