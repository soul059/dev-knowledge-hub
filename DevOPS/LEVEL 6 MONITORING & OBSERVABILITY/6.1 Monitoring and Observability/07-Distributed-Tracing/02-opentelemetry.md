# Chapter 2: OpenTelemetry

[← Previous: Tracing Concepts](01-tracing-concepts.md) | [Back to README](../README.md) | [Next: Jaeger →](03-jaeger.md)

---

## Overview

OpenTelemetry (OTel) is a CNCF project that provides a unified, vendor-neutral standard for collecting metrics, logs, and traces. It is the merger of OpenTracing and OpenCensus and is becoming the industry standard for observability instrumentation.

---

## 2.1 OpenTelemetry Architecture

```
┌──────────────────────────────────────────────────────────┐
│              OPENTELEMETRY ARCHITECTURE                  │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              APPLICATION                         │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │    │
│  │  │ OTel SDK │ │ Auto-inst│ │ Manual   │        │    │
│  │  │ (API +   │ │ Agent    │ │ Spans    │        │    │
│  │  │ SDK)     │ │ (Java,   │ │          │        │    │
│  │  │          │ │  .NET)   │ │          │        │    │
│  │  └──────┬───┘ └──────┬──┘ └─────┬────┘        │    │
│  │         └──────┬──────┘         │              │    │
│  │                ▼                ▼              │    │
│  │         ┌──────────────────────────┐           │    │
│  │         │      OTLP Exporter       │           │    │
│  │         │ (gRPC or HTTP/protobuf)  │           │    │
│  │         └──────────┬───────────────┘           │    │
│  └────────────────────┼──────────────────────────┘    │
│                       │                                │
│                       ▼                                │
│  ┌──────────────────────────────────────────────┐     │
│  │         OTEL COLLECTOR (optional)             │     │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐     │     │
│  │  │Receivers │→│Processors│→│Exporters │     │     │
│  │  │(OTLP,    │ │(batch,   │ │(Jaeger,  │     │     │
│  │  │ Jaeger,  │ │ filter,  │ │ Prom,    │     │     │
│  │  │ Prom)    │ │ sample)  │ │ Loki)    │     │     │
│  │  └──────────┘ └──────────┘ └──────────┘     │     │
│  └──────────────────┬───────────────────────────┘     │
│                     │                                  │
│          ┌──────────┼──────────┐                      │
│          ▼          ▼          ▼                      │
│     ┌────────┐ ┌────────┐ ┌────────┐                 │
│     │ Jaeger │ │Promethe│ │  Loki  │  Backends       │
│     │ /Tempo │ │ us     │ │        │                 │
│     └────────┘ └────────┘ └────────┘                 │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 OTel Components

| Component | Purpose |
|-----------|---------|
| **API** | Stable interfaces for instrumentation (vendor-agnostic) |
| **SDK** | Implementation of the API (configurable) |
| **Instrumentation** | Auto or manual code to create telemetry |
| **OTLP** | OpenTelemetry Protocol — standard wire format |
| **Collector** | Agent/gateway for receiving, processing, exporting telemetry |
| **Semantic Conventions** | Standard attribute names (e.g., `http.method`, `db.system`) |

---

## 2.3 Auto-Instrumentation (Python Example)

```bash
# Install OpenTelemetry packages
pip install opentelemetry-api \
            opentelemetry-sdk \
            opentelemetry-instrumentation-flask \
            opentelemetry-instrumentation-requests \
            opentelemetry-instrumentation-sqlalchemy \
            opentelemetry-exporter-otlp
```

```python
# app.py — Auto-instrumentation with zero code changes
from flask import Flask
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.sdk.resources import Resource

# Configure resource
resource = Resource.create({
    "service.name": "order-service",
    "service.version": "1.2.0",
    "deployment.environment": "production"
})

# Configure tracer provider
provider = TracerProvider(resource=resource)
processor = BatchSpanProcessor(
    OTLPSpanExporter(endpoint="http://otel-collector:4317")
)
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

# Auto-instrument
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)  # Auto-trace all routes
RequestsInstrumentor().instrument()       # Auto-trace HTTP calls

@app.route("/api/orders/<order_id>")
def get_order(order_id):
    # This is automatically traced!
    order = fetch_order(order_id)
    return order
```

### Zero-Code Auto-Instrumentation

```bash
# Java (agent-based, no code changes)
java -javaagent:opentelemetry-javaagent.jar \
  -Dotel.service.name=order-service \
  -Dotel.exporter.otlp.endpoint=http://otel-collector:4317 \
  -jar myapp.jar

# Python (CLI-based)
opentelemetry-instrument \
  --service_name order-service \
  --exporter_otlp_endpoint http://otel-collector:4317 \
  python app.py

# Node.js
node --require @opentelemetry/auto-instrumentations-node/register app.js
```

---

## 2.4 Manual Instrumentation

```python
from opentelemetry import trace

tracer = trace.get_tracer("order-service")

def process_order(order_id):
    # Create a custom span
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.id", order_id)
        
        # Nested span
        with tracer.start_as_current_span("validate_inventory") as child:
            child.set_attribute("items.count", 3)
            result = check_inventory()
            
            if not result:
                child.set_status(trace.StatusCode.ERROR, "Out of stock")
                child.record_exception(Exception("Inventory check failed"))
                raise Exception("Out of stock")
        
        with tracer.start_as_current_span("charge_payment") as child:
            child.set_attribute("payment.method", "credit_card")
            child.set_attribute("payment.amount", 99.99)
            charge_result = process_payment()
            
        span.set_attribute("order.status", "completed")
        return {"status": "success"}
```

---

## 2.5 OTel Collector Configuration

```yaml
# otel-collector-config.yaml

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
      http:
        endpoint: "0.0.0.0:4318"
  
  prometheus:
    config:
      scrape_configs:
        - job_name: 'otel-collector'
          scrape_interval: 15s
          static_configs:
            - targets: ['localhost:8888']

processors:
  batch:
    send_batch_size: 1024
    timeout: 5s
  
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
    spike_limit_mib: 128

  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors
        type: status_code
        status_code: { status_codes: [ERROR] }
      - name: slow-requests
        type: latency
        latency: { threshold_ms: 1000 }
      - name: probabilistic
        type: probabilistic
        probabilistic: { sampling_percentage: 10 }

exporters:
  otlp/tempo:
    endpoint: "tempo:4317"
    tls:
      insecure: true

  prometheus:
    endpoint: "0.0.0.0:8889"

  loki:
    endpoint: "http://loki:3100/loki/api/v1/push"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, tail_sampling, batch]
      exporters: [otlp/tempo]
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [loki]
```

---

## 2.6 Deployment Patterns

```
Pattern 1: Agent (DaemonSet)
  Each node has a collector agent
  App → Agent → Backend

Pattern 2: Gateway
  Central collector cluster
  App → Gateway (load balanced) → Backend

Pattern 3: Agent + Gateway (recommended)
  App → Agent (DaemonSet) → Gateway → Backend
  Agent: buffering, basic processing
  Gateway: sampling, routing, export
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| No traces appearing | Exporter endpoint wrong | Check endpoint URL and collector logs |
| Missing spans in trace | Library not instrumented | Add instrumentation for HTTP/DB/cache |
| High overhead | Tracing 100% of requests | Enable sampling (probabilistic or tail) |
| Trace context lost | Missing propagator | Configure W3C TraceContext propagator |
| Collector dropping spans | Memory limit reached | Increase memory_limiter or add batch processor |

---

## Quick Revision Questions

1. **What is OpenTelemetry and why was it created?**
2. **What are the main components of the OTel architecture?**
3. **How does auto-instrumentation differ from manual instrumentation?**
4. **What is the OTel Collector and what are its three pipeline stages?**
5. **What is OTLP and why is it important?**
6. **What deployment patterns are available for the OTel Collector?**

---

[← Previous: Tracing Concepts](01-tracing-concepts.md) | [Back to README](../README.md) | [Next: Jaeger →](03-jaeger.md)
