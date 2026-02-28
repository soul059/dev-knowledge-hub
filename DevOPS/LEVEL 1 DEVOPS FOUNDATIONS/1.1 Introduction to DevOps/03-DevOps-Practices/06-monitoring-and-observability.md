# Chapter 3.6 — Monitoring & Observability

[← Previous: Configuration Management](05-configuration-management.md) | [Next: Unit 4 — Version Control Systems →](../04-DevOps-Tools-Landscape/01-version-control-systems.md)

---

## Overview

**Monitoring** tells you WHEN something is wrong. **Observability** tells you WHY. Together, they form the feedback loop that keeps systems reliable. Without monitoring and observability, you're flying blind — you'll only discover problems when users complain.

---

## Monitoring vs Observability

```
┌──────────────────────────────────────────────────────────┐
│  MONITORING vs OBSERVABILITY                             │
│                                                          │
│  MONITORING                      OBSERVABILITY           │
│  ══════════                      ═════════════           │
│  "Is the system working?"        "Why is it broken?"     │
│                                                          │
│  ├── Dashboard alerts            ├── Trace requests      │
│  ├── Threshold-based             ├── Correlate events    │
│  ├── Known failure modes         ├── Unknown unknowns    │
│  ├── Predefined queries          ├── Ad-hoc exploration  │
│  └── "CPU > 90% → alert"        └── "Why is checkout     │
│                                       slow for users     │
│                                       in Europe?"        │
│                                                          │
│  Monitoring ⊂ Observability                              │
│  (Monitoring is a subset of Observability)               │
└──────────────────────────────────────────────────────────┘
```

---

## Three Pillars of Observability

```
┌──────────────────────────────────────────────────────────┐
│  THE THREE PILLARS                                       │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    LOGS      │  │   METRICS    │  │   TRACES     │  │
│  │              │  │              │  │              │  │
│  │  What        │  │  How much /  │  │  Where does  │  │
│  │  happened?   │  │  How many?   │  │  time go?    │  │
│  │              │  │              │  │              │  │
│  │  Text events │  │  Numbers     │  │  Request     │  │
│  │  with        │  │  over time   │  │  journey     │  │
│  │  timestamps  │  │  (counters,  │  │  across      │  │
│  │              │  │  gauges,     │  │  services    │  │
│  │  ELK Stack   │  │  histograms) │  │              │  │
│  │  Loki        │  │              │  │  Jaeger      │  │
│  │  Splunk      │  │  Prometheus  │  │  Zipkin      │  │
│  │  CloudWatch  │  │  Datadog     │  │  OpenTelemetry│ │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  Best Practice: CORRELATE all three!                     │
│  Log → Metric → Trace → Root Cause                      │
└──────────────────────────────────────────────────────────┘
```

---

## Pillar 1: Logs

### Structured Logging

```json
// ❌ Unstructured (hard to parse)
"ERROR: Failed to process order 12345 for user john@example.com"

// ✅ Structured (machine-readable, searchable)
{
  "timestamp": "2024-01-15T10:23:45.123Z",
  "level": "ERROR",
  "service": "order-service",
  "message": "Failed to process order",
  "order_id": "12345",
  "user_email": "john@example.com",
  "error_code": "PAYMENT_DECLINED",
  "trace_id": "abc123def456",
  "duration_ms": 2340
}
```

### Log Levels

```
┌──────────────────────────────────────────────────────────┐
│  LOG LEVELS (from most to least severe)                  │
│                                                          │
│  FATAL   ████████████████  System is unusable            │
│  ERROR   ██████████████    Something failed              │
│  WARN    ████████████      Something unexpected          │
│  INFO    ██████████        Normal operations             │
│  DEBUG   ████████          Detailed debugging info       │
│  TRACE   ██████            Very detailed tracing         │
│                                                          │
│  Production: INFO and above                              │
│  Debugging:  DEBUG and above                             │
│  Never in production: TRACE (too verbose)                │
└──────────────────────────────────────────────────────────┘
```

### ELK Stack (Elasticsearch, Logstash, Kibana)

```
┌──────────────────────────────────────────────────────────┐
│  ELK STACK PIPELINE                                      │
│                                                          │
│  ┌─────────┐   ┌──────────┐   ┌──────────────┐         │
│  │  Apps    │──►│ Logstash │──►│Elasticsearch │         │
│  │  (logs)  │   │ / Beats  │   │  (storage +  │         │
│  └─────────┘   │ (ingest) │   │   indexing)  │         │
│                 └──────────┘   └──────┬───────┘         │
│                                       │                  │
│                                ┌──────▼───────┐         │
│                                │   Kibana     │         │
│                                │  (visualize  │         │
│                                │   + search)  │         │
│                                └──────────────┘         │
│                                                          │
│  Modern Alternative: Loki + Grafana (lighter weight)     │
└──────────────────────────────────────────────────────────┘
```

---

## Pillar 2: Metrics

### Prometheus Example

```yaml
# prometheus.yml - Prometheus configuration
global:
  scrape_interval: 15s      # How often to scrape targets
  evaluation_interval: 15s  # How often to evaluate rules

rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

scrape_configs:
  - job_name: 'web-app'
    static_configs:
      - targets: ['web1:8080', 'web2:8080', 'web3:8080']
    metrics_path: /metrics

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['web1:9100', 'web2:9100', 'web3:9100']
```

### Four Golden Signals (Google SRE)

```
┌──────────────────────────────────────────────────────────┐
│  THE FOUR GOLDEN SIGNALS                                 │
│                                                          │
│  1. LATENCY                                              │
│     How long requests take                               │
│     → histogram: http_request_duration_seconds           │
│                                                          │
│  2. TRAFFIC                                              │
│     How much demand is hitting the system                │
│     → counter: http_requests_total                       │
│                                                          │
│  3. ERRORS                                               │
│     Rate of failed requests                              │
│     → gauge: error_rate = errors / total_requests        │
│                                                          │
│  4. SATURATION                                           │
│     How "full" the system is                             │
│     → gauge: cpu_usage_percent, memory_usage_percent     │
│                                                          │
│  If you can only monitor 4 things, monitor THESE.        │
└──────────────────────────────────────────────────────────┘
```

### Alert Rules

```yaml
# alerts.yml - Prometheus alerting rules
groups:
  - name: web-app-alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} (>5%)"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 2.0
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High p95 latency"
          description: "95th percentile latency is {{ $value }}s (>2s)"

      # Pod not ready
      - alert: PodNotReady
        expr: kube_pod_status_ready{condition="true"} == 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.pod }} is not ready"
```

---

## Pillar 3: Distributed Traces

```
┌──────────────────────────────────────────────────────────┐
│  DISTRIBUTED TRACE: "Place Order" Request                │
│                                                          │
│  Trace ID: abc-123-def-456                               │
│  Total Duration: 1250ms                                  │
│                                                          │
│  ├── API Gateway (50ms)                                  │
│  │   └── Auth Service (30ms)                             │
│  │                                                       │
│  ├── Order Service (200ms)                               │
│  │   ├── Inventory Service (80ms)                        │
│  │   │   └── Database Query (45ms)                       │
│  │   └── Pricing Service (100ms)                         │
│  │       └── Cache Lookup (5ms)                          │
│  │                                                       │
│  └── Payment Service (900ms)  ◄── BOTTLENECK!            │
│      ├── Fraud Check (150ms)                             │
│      ├── Payment Gateway (700ms) ◄── External API slow   │
│      └── Receipt Email (50ms)                            │
│                                                          │
│  Without traces: "Orders are slow"                       │
│  With traces: "Payment gateway is slow (700ms)"          │
└──────────────────────────────────────────────────────────┘
```

### OpenTelemetry (OTel)

```python
# OpenTelemetry instrumentation (Python example)
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

# Setup
provider = TracerProvider()
jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)
provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Usage in application code
@app.route("/api/orders", methods=["POST"])
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("user.id", request.user_id)
        
        with tracer.start_as_current_span("validate_inventory"):
            inventory = check_inventory(request.items)
        
        with tracer.start_as_current_span("process_payment"):
            payment = charge_card(request.payment_info)
            span.set_attribute("payment.amount", payment.amount)
        
        return {"order_id": order.id}
```

---

## Grafana Dashboard

```
┌──────────────────────────────────────────────────────────┐
│  GRAFANA DASHBOARD LAYOUT                                │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  🟢 Service Health: All Systems Operational       │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌────────────────┐  ┌────────────────┐  ┌───────────┐ │
│  │ Request Rate   │  │ Error Rate     │  │  P95      │ │
│  │   12.3k/s      │  │   0.02%        │  │ Latency   │ │
│  │ ▁▂▃▄▅▆▇████   │  │ ▁▁▁▁▁▁▁▁▁▁▁  │  │  142ms   │ │
│  └────────────────┘  └────────────────┘  └───────────┘ │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  CPU Usage by Node                                │   │
│  │  web1: ███████████░░░░░░  65%                    │   │
│  │  web2: ████████░░░░░░░░░  48%                    │   │
│  │  web3: █████████████░░░░  78%  ⚠️                │   │
│  │  db1:  ██████░░░░░░░░░░░  35%                    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Data Sources: Prometheus + Loki + Jaeger                │
└──────────────────────────────────────────────────────────┘
```

---

## Alerting Best Practices

```
┌──────────────────────────────────────────────────────────┐
│  ALERTING PYRAMID                                        │
│                                                          │
│         ╱╲         PAGE (wake someone up)                │
│        ╱  ╲        → Service is DOWN                     │
│       ╱ P  ╲       → Data loss risk                      │
│      ╱──────╲      → SLO breach                          │
│     ╱ TICKET ╲     → Create a ticket for next shift      │
│    ╱──────────╲    → Degraded performance                 │
│   ╱    LOG     ╲   → Log for investigation later         │
│  ╱──────────────╲  → Non-critical anomalies              │
│ ╱───────────────────╲                                    │
│                                                          │
│  Alert Fatigue = Too many alerts = All alerts ignored     │
│                                                          │
│  Rules:                                                  │
│  ├── Every alert must be ACTIONABLE                      │
│  ├── If you can't act on it, it's not an alert           │
│  ├── Tune thresholds to reduce false positives           │
│  └── Use severity levels consistently                    │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Diagnosing a Slow Checkout

```
1. ALERT FIRES:
   "HighLatency: 95th percentile latency is 4.2s (>2s)"

2. CHECK METRICS (Grafana):
   → Latency spike started at 14:23
   → Error rate normal (0.1%)
   → Only checkout endpoint affected

3. CHECK TRACES (Jaeger):
   → Trace for slow checkout request shows:
     ├── API Gateway: 10ms ✅
     ├── Cart Service: 25ms ✅
     ├── Payment Service: 3800ms ❌  ← Found it!
     │   └── External API call: 3750ms ← Root cause
     └── Notification: 15ms ✅

4. CHECK LOGS (Kibana):
   → Filter: service=payment-service AND level=WARN
   → "Payment gateway response time degraded"
   → "Retry attempt 2 of 3 for transaction xyz"

5. ROOT CAUSE:
   → Third-party payment gateway experiencing issues
   → Action: Switch to backup payment provider
   → Resolution time: 8 minutes (MTTD: 3m, MTTR: 5m)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert fatigue | Too many non-actionable alerts | Review and prune alerts; every alert must have a runbook |
| Missing context in logs | Unstructured logging | Adopt structured logging (JSON) with trace IDs |
| Can't correlate events | No shared identifiers | Add trace IDs to logs, metrics labels, and spans |
| Metrics cardinality explosion | Too many unique label values | Avoid high-cardinality labels (user IDs, request IDs) |
| Dashboard overload | Too many panels | Focus on Four Golden Signals; create role-specific views |
| Delayed alerts | Scrape interval too long | Reduce scrape interval; use push-based metrics for critical paths |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Monitoring** | Watching known metrics for known failure modes |
| **Observability** | Understanding system behavior from external outputs |
| **Logs** | Timestamped text events describing what happened |
| **Metrics** | Numerical measurements over time (counters, gauges) |
| **Traces** | Request journey across distributed services |
| **Four Golden Signals** | Latency, Traffic, Errors, Saturation |
| **Prometheus** | Open-source metrics collection and alerting |
| **Grafana** | Visualization and dashboarding tool |
| **ELK Stack** | Elasticsearch + Logstash + Kibana for log management |
| **OpenTelemetry** | Vendor-neutral standard for traces, metrics, and logs |

---

## Quick Revision Questions

1. **What is the difference between monitoring and observability?**
   <details><summary>Answer</summary>Monitoring tells you WHEN something is wrong using predefined dashboards and thresholds (e.g., "CPU > 90%"). Observability tells you WHY something is wrong by letting you explore data with ad-hoc queries. Monitoring handles known unknowns; observability handles unknown unknowns. Monitoring is a subset of observability.</details>

2. **What are the three pillars of observability?**
   <details><summary>Answer</summary>1) Logs — timestamped text records of events (what happened). 2) Metrics — numerical measurements over time like request rate or CPU usage (how much/how many). 3) Traces — the path of a request through distributed services (where time is spent). The key is correlating all three using shared identifiers like trace IDs.</details>

3. **What are the Four Golden Signals from Google SRE?**
   <details><summary>Answer</summary>1) Latency — how long requests take. 2) Traffic — demand on the system (requests per second). 3) Errors — rate of failed requests. 4) Saturation — how full the system is (CPU, memory, disk). If you can only monitor four things, monitor these.</details>

4. **What is alert fatigue, and how do you prevent it?**
   <details><summary>Answer</summary>Alert fatigue occurs when teams receive too many alerts, causing them to ignore or miss critical ones. Prevention: 1) Every alert must be actionable. 2) Tune thresholds to reduce false positives. 3) Use severity levels (page vs ticket vs log). 4) Each alert should have a runbook. 5) Regularly review and prune alert rules.</details>

5. **Why is structured logging preferred over unstructured logging?**
   <details><summary>Answer</summary>Structured logging (JSON format) makes logs machine-readable, enabling: 1) Easy parsing and indexing by log management tools. 2) Filtering by specific fields (user_id, error_code). 3) Correlation with traces via trace_id. 4) Aggregation and analysis at scale. Unstructured logs require regex parsing and are brittle to format changes.</details>

6. **How do distributed traces help diagnose performance issues?**
   <details><summary>Answer</summary>Distributed traces track a single request across multiple services, showing: 1) Which services were called. 2) How long each service took. 3) Where the bottleneck is. For example, a trace might reveal that a "slow checkout" is caused by a third-party payment gateway taking 3.7 seconds, not by your own services.</details>

---

[← Previous: Configuration Management](05-configuration-management.md) | [Next: Unit 4 — Version Control Systems →](../04-DevOps-Tools-Landscape/01-version-control-systems.md)
