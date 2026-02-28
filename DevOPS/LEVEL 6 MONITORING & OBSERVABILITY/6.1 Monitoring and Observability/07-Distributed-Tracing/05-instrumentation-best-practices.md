# Chapter 5: Instrumentation Best Practices

[← Previous: Grafana Tempo](04-grafana-tempo.md) | [Back to README](../README.md) | [Next: Trace Analysis & Optimization →](06-trace-analysis-optimization.md)

---

## Overview

Effective distributed tracing requires thoughtful instrumentation. This chapter covers best practices for instrumenting applications to produce meaningful, actionable traces without overwhelming your tracing infrastructure.

---

## 5.1 Instrumentation Strategy

```
┌──────────────────────────────────────────────────────────┐
│          INSTRUMENTATION PYRAMID                         │
│                                                          │
│                    ▲                                     │
│                   / \     Custom Business Spans          │
│                  / 3 \    (checkout, payment, search)    │
│                 /─────\                                  │
│                / 2     \  Framework Middleware Spans     │
│               /  (auto) \  (HTTP, gRPC, DB drivers)     │
│              /───────────\                               │
│             / 1           \  Auto-Instrumentation       │
│            /  (zero-code)  \ (libraries, agents)        │
│           /─────────────────\                            │
│                                                          │
│  Start from base → add layers as needed                 │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 What to Instrument

### Must Instrument (High Value)

```
┌────────────────────────────────────────────────┐
│  HIGH-VALUE SPAN TARGETS                       │
│                                                │
│  ✓ HTTP/gRPC incoming requests                │
│  ✓ HTTP/gRPC outgoing calls                   │
│  ✓ Database queries                            │
│  ✓ Cache operations (Redis, Memcached)         │
│  ✓ Message queue publish/consume               │
│  ✓ External API calls (payment, email, SMS)    │
│  ✓ Authentication/authorization checks         │
│  ✓ File I/O operations                         │
└────────────────────────────────────────────────┘
```

### Should Instrument (Medium Value)

- Business-critical operations (checkout, order processing)
- Background job execution
- Feature flag evaluation
- Circuit breaker state changes

### Avoid Instrumenting (Low Value / Noise)

- Every function call (creates span explosion)
- In-memory data transformations
- Getter/setter methods
- Logging within an already-traced span

---

## 5.3 Span Naming Conventions

```
┌────────────────────────────────────────────────────────────┐
│  SPAN NAMING BEST PRACTICES                               │
│                                                            │
│  ✓ GOOD Names (low cardinality):                          │
│    • HTTP GET /api/users/{id}    (parameterized)           │
│    • SQL SELECT users            (table name only)         │
│    • redis GET                   (operation type)          │
│    • kafka.publish orders-topic  (topic name)              │
│                                                            │
│  ✗ BAD Names (high cardinality):                           │
│    • HTTP GET /api/users/12345   (includes ID)             │
│    • SELECT * FROM users WHERE id=12345 (full query)       │
│    • ProcessOrder-ORD-2024-001   (includes order number)   │
│                                                            │
│  Rule: Span name = operation type + static resource name   │
│        Dynamic values go in span ATTRIBUTES, not names     │
└────────────────────────────────────────────────────────────┘
```

---

## 5.4 Span Attributes (Tags)

### Semantic Conventions (OpenTelemetry)

```python
from opentelemetry import trace
from opentelemetry.semconv.trace import SpanAttributes

tracer = trace.get_tracer("order-service")

def process_order(order):
    with tracer.start_as_current_span("process_order") as span:
        # Standard semantic attributes
        span.set_attribute("http.method", "POST")
        span.set_attribute("http.url", "/api/orders")
        span.set_attribute("http.status_code", 200)
        
        # Database attributes
        span.set_attribute("db.system", "postgresql")
        span.set_attribute("db.statement", "INSERT INTO orders ...")
        span.set_attribute("db.name", "ecommerce")
        
        # Custom business attributes
        span.set_attribute("order.id", order.id)
        span.set_attribute("order.total", order.total)
        span.set_attribute("order.item_count", len(order.items))
        span.set_attribute("customer.tier", order.customer.tier)
```

### Common Attribute Namespaces

| Namespace | Example Attributes | Description |
|-----------|-------------------|-------------|
| `http.` | method, url, status_code, scheme | HTTP request/response |
| `db.` | system, statement, name, operation | Database operations |
| `rpc.` | system, service, method | RPC calls (gRPC) |
| `messaging.` | system, destination, operation | Message queues |
| `net.` | host.name, host.port, peer.name | Network details |
| `enduser.` | id, role, scope | End-user info |
| `cloud.` | provider, region, availability_zone | Cloud infra |
| `k8s.` | pod.name, namespace.name, deployment.name | Kubernetes |

---

## 5.5 Context Propagation Patterns

```
┌──────────────────────────────────────────────────────────────┐
│        CONTEXT PROPAGATION DO's AND DON'Ts                   │
│                                                              │
│  ✓ DO: Propagate context through HTTP headers                │
│     traceparent: 00-trace_id-span_id-01                     │
│                                                              │
│  ✓ DO: Propagate through message queue metadata              │
│     message.headers["traceparent"] = ctx                    │
│                                                              │
│  ✓ DO: Use W3C Trace Context (standard)                     │
│     Compatible across vendors                                │
│                                                              │
│  ✗ DON'T: Create new root spans for downstream calls         │
│     This breaks the trace chain                              │
│                                                              │
│  ✗ DON'T: Propagate context across batch boundaries          │
│     Batch jobs should start new traces with links             │
└──────────────────────────────────────────────────────────────┘
```

### Propagation Through Message Queues

```python
from opentelemetry import trace, context
from opentelemetry.propagate import inject, extract
import json

tracer = trace.get_tracer("order-service")

# === PRODUCER: Inject context into message ===
def publish_order_event(order):
    with tracer.start_as_current_span("publish_order_created") as span:
        span.set_attribute("messaging.system", "kafka")
        span.set_attribute("messaging.destination", "orders")
        
        # Inject trace context into message headers
        headers = {}
        inject(headers)
        
        message = {
            "order_id": order.id,
            "total": order.total
        }
        kafka_producer.send(
            "orders",
            value=json.dumps(message),
            headers=headers
        )

# === CONSUMER: Extract context from message ===
def consume_order_event(message):
    # Extract trace context from message headers
    ctx = extract(message.headers)
    
    with tracer.start_as_current_span(
        "process_order_event",
        context=ctx,
        kind=trace.SpanKind.CONSUMER
    ) as span:
        span.set_attribute("messaging.system", "kafka")
        span.set_attribute("messaging.operation", "process")
        # Process the order...
```

---

## 5.6 Error Handling in Spans

```python
from opentelemetry import trace
from opentelemetry.trace import StatusCode

tracer = trace.get_tracer("payment-service")

def charge_customer(order):
    with tracer.start_as_current_span("charge_customer") as span:
        try:
            result = payment_gateway.charge(
                amount=order.total,
                currency="USD",
                customer_id=order.customer_id
            )
            span.set_attribute("payment.transaction_id", result.txn_id)
            span.set_status(StatusCode.OK)
            return result
            
        except PaymentDeclinedError as e:
            # Business error - record but don't set ERROR status
            span.set_attribute("payment.declined_reason", str(e))
            span.set_status(StatusCode.OK, "Payment declined - expected")
            span.add_event("payment_declined", {
                "reason": str(e),
                "customer_id": order.customer_id
            })
            raise
            
        except Exception as e:
            # Unexpected error - set ERROR status
            span.set_status(StatusCode.ERROR, str(e))
            span.record_exception(e)
            raise
```

---

## 5.7 Sampling Strategies

```
┌──────────────────────────────────────────────────────────┐
│           SAMPLING DECISION TREE                         │
│                                                          │
│  Is it a health check / readiness probe?                │
│  ├─ YES → Drop (0% sample)                             │
│  └─ NO ──► Is it an error trace?                        │
│            ├─ YES → Keep (100% sample)                  │
│            └─ NO ──► Is it high latency (> p99)?        │
│                      ├─ YES → Keep (100% sample)        │
│                      └─ NO ──► Is it critical path?     │
│                                ├─ YES → 10-50% sample   │
│                                └─ NO ──► 1-5% sample    │
│                                                          │
│  Result: Errors always captured, noise minimized        │
└──────────────────────────────────────────────────────────┘
```

### OTel Collector Tail Sampling Config

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      # Always sample errors
      - name: errors
        type: status_code
        status_code:
          status_codes: [ERROR]
      
      # Always sample slow traces
      - name: slow-traces
        type: latency
        latency:
          threshold_ms: 2000
      
      # Drop health checks entirely
      - name: drop-healthcheck
        type: string_attribute
        string_attribute:
          key: http.target
          values: ["/health", "/ready", "/live"]
          enabled_regex_matching: true
          invert_match: true

      # Sample remaining at 10%
      - name: probabilistic
        type: probabilistic
        probabilistic:
          sampling_percentage: 10
```

---

## 5.8 Span Events vs Child Spans

```
┌──────────────────────────────────────────────────────────┐
│  WHEN TO USE EVENTS vs CHILD SPANS                       │
│                                                          │
│  Use SPAN EVENTS for:                                   │
│  • Point-in-time annotations                            │
│  • "Something happened" moments                         │
│  • Low-overhead recording                               │
│                                                          │
│  Example: span.add_event("cache_miss",                  │
│              {"key": "user:123"})                        │
│                                                          │
│  Use CHILD SPANS for:                                   │
│  • Operations with duration                             │
│  • External calls (HTTP, DB, queue)                     │
│  • Operations you want to measure latency of            │
│                                                          │
│  Example: with tracer.start_as_current_span("db_query") │
│                                                          │
│  Rule of thumb:                                         │
│  "Did it take measurable time?" → CHILD SPAN            │
│  "Did something noteworthy happen?" → EVENT             │
└──────────────────────────────────────────────────────────┘
```

---

## 5.9 Common Anti-Patterns

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| Over-instrumentation | Span explosion, high costs | Instrument boundaries only |
| Missing propagation | Broken traces | Use OTel auto-instrumentation |
| High-cardinality names | Unbounded span names | Parameterize routes |
| No error recording | Can't find failures | Always record_exception() |
| Sync-only tracing | Missing async work | Propagate context to threads/tasks |
| Ignoring baggage | Lost cross-service context | Use OTel baggage for shared data |
| No resource attributes | Can't identify source | Set service.name, version, env |

---

## 5.10 Instrumentation Checklist

```
┌────────────────────────────────────────────────┐
│  SERVICE INSTRUMENTATION CHECKLIST             │
│                                                │
│  □ OTel SDK installed and configured           │
│  □ service.name resource attribute set         │
│  □ service.version attribute set               │
│  □ deployment.environment attribute set        │
│  □ Auto-instrumentation enabled                │
│  □ HTTP client/server spans working            │
│  □ Database spans working                      │
│  □ Message queue spans working                 │
│  □ Custom business spans added                 │
│  □ Error recording with record_exception()     │
│  □ Span names use low cardinality              │
│  □ Sensitive data NOT in attributes            │
│  □ Sampling configured appropriately           │
│  □ Context propagation verified end-to-end     │
│  □ Traces visible in backend (Tempo/Jaeger)    │
└────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Span explosion | Instrumenting every function | Only instrument I/O and business ops |
| Disconnected traces | Context not propagated | Verify headers pass through proxies/LBs |
| Missing service name | Resource not configured | Set `OTEL_SERVICE_NAME` env var |
| Duplicate spans | Multiple instrumentations | Disable overlapping auto-instrumentation |
| High memory usage | Too many in-flight spans | Enable batch span processor, add sampling |

---

## Quick Revision Questions

1. **What are the three levels of the instrumentation pyramid?**
2. **Why should span names be low cardinality? Give an example of a good and bad span name.**
3. **How do you propagate trace context through a message queue?**
4. **When should you use span events instead of child spans?**
5. **What sampling strategy would you use to always capture errors while reducing overall trace volume?**
6. **What are the essential resource attributes every service should set?**

---

[← Previous: Grafana Tempo](04-grafana-tempo.md) | [Back to README](../README.md) | [Next: Trace Analysis & Optimization →](06-trace-analysis-optimization.md)
