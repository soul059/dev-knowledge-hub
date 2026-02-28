# Chapter 3.2: Measuring Reliability

[← Previous: Availability Calculations](01-availability-calculations.md) | [Back to README](../README.md) | [Next: Risk Assessment →](03-risk-assessment.md)

---

## Overview

Measuring reliability goes beyond simple uptime percentages. It involves capturing user-facing health across multiple dimensions, building dashboards, and creating alerting systems that detect issues before users do.

---

## 1. Reliability Metrics Framework

```
  ┌──────────────────────────────────────────────────────────┐
  │            RELIABILITY METRICS HIERARCHY                 │
  │                                                          │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ DETECTION: How quickly do we know something broke? │  │
  │  │            MTTD (Mean Time To Detect)               │  │
  │  └────────────────────┬───────────────────────────────┘  │
  │                       ▼                                  │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ RESPONSE: How quickly do we start working on it?   │  │
  │  │           MTTA (Mean Time To Acknowledge)           │  │
  │  └────────────────────┬───────────────────────────────┘  │
  │                       ▼                                  │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ DIAGNOSIS: How quickly do we find the root cause?  │  │
  │  │            MTTD (Mean Time To Diagnose)             │  │
  │  └────────────────────┬───────────────────────────────┘  │
  │                       ▼                                  │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ RECOVERY: How quickly do we restore service?       │  │
  │  │           MTTR (Mean Time To Recover)               │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  MTTR = MTTD + MTTA + Diagnosis + Fix time              │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Key Reliability Metrics

| Metric | Formula | What It Measures |
|--------|---------|------------------|
| **MTTD** | Σ(detection time) / incidents | Speed of problem detection |
| **MTTA** | Σ(acknowledge time) / incidents | Speed of human response |
| **MTTR** | Σ(recovery time) / incidents | Speed of service restoration |
| **MTBF** | Total uptime / failures | Frequency of failures |
| **Failure Rate** | Failures / time period | How often things break |
| **Error Budget Burn** | Budget consumed / budget total | SLO health |
| **SLI Compliance** | Current SLI vs SLO target | Are we meeting our targets? |

---

## 3. The Four Golden Signals (Deep Dive)

```
  ┌──────────────────────────────────────────────────────────┐
  │  SIGNAL 1: LATENCY                                      │
  │  ──────────────────                                      │
  │  Measure: Time to serve a request                        │
  │  Key: Distinguish successful from failed request latency │
  │                                                          │
  │  Implementation:                                         │
  │  histogram_quantile(0.50, rate(http_duration_bucket[5m]))│
  │  histogram_quantile(0.95, rate(http_duration_bucket[5m]))│
  │  histogram_quantile(0.99, rate(http_duration_bucket[5m]))│
  │                                                          │
  │  [!] A fast 500 error still counts as bad latency data.  │
  │      Separate success and error latency distributions.   │
  ├──────────────────────────────────────────────────────────┤
  │  SIGNAL 2: TRAFFIC                                      │
  │  ──────────────────                                      │
  │  Measure: Demand on system                               │
  │  Examples: HTTP req/sec, transactions/sec, I/O ops/sec   │
  │                                                          │
  │  Implementation:                                         │
  │  sum(rate(http_requests_total[5m])) by (service)         │
  ├──────────────────────────────────────────────────────────┤
  │  SIGNAL 3: ERRORS                                       │
  │  ──────────────────                                      │
  │  Measure: Rate of failed requests                        │
  │  Types: Explicit (5xx), implicit (wrong content),        │
  │         policy (too slow counts as error)                │
  │                                                          │
  │  Implementation:                                         │
  │  sum(rate(http_requests_total{code=~"5.."}[5m]))         │
  │  / sum(rate(http_requests_total[5m]))                    │
  ├──────────────────────────────────────────────────────────┤
  │  SIGNAL 4: SATURATION                                   │
  │  ──────────────────                                      │
  │  Measure: How "full" is the service                      │
  │  Examples: CPU%, memory%, disk%, queue depth              │
  │  Key: Most services degrade before hitting 100%          │
  │                                                          │
  │  Implementation:                                         │
  │  node_cpu_seconds_total, node_memory_MemAvailable_bytes  │
  │  kube_pod_container_resource_limits                      │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Observability Stack

```
  ┌──────────────────────────────────────────────────────────┐
  │             TYPICAL SRE OBSERVABILITY STACK              │
  │                                                          │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │  DATA COLLECTION                                   │  │
  │  │                                                    │  │
  │  │  Metrics:  Prometheus / Datadog / CloudWatch       │  │
  │  │  Logs:     ELK / Loki / Splunk / CloudWatch Logs  │  │
  │  │  Traces:   Jaeger / Tempo / X-Ray / Zipkin        │  │
  │  │  Events:   PagerDuty / Opsgenie                   │  │
  │  └───────────────────────┬────────────────────────────┘  │
  │                          │                               │
  │                          ▼                               │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │  VISUALIZATION                                     │  │
  │  │                                                    │  │
  │  │  Grafana dashboards                                │  │
  │  │  ├── Service Overview (golden signals)             │  │
  │  │  ├── SLO Dashboard (budget burn, compliance)       │  │
  │  │  ├── Infrastructure (nodes, pods, network)         │  │
  │  │  └── Business Metrics (orders, sign-ups)           │  │
  │  └───────────────────────┬────────────────────────────┘  │
  │                          │                               │
  │                          ▼                               │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │  ALERTING                                          │  │
  │  │                                                    │  │
  │  │  SLO burn rate alerts → PagerDuty → On-call        │  │
  │  │  Symptom-based, not cause-based                    │  │
  │  └────────────────────────────────────────────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Prometheus Configuration Example

```yaml
# prometheus.yml — scrape configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "sli_recording_rules.yml"
  - "slo_alerting_rules.yml"

scrape_configs:
  - job_name: 'api-server'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true

# sli_recording_rules.yml
groups:
- name: sli_rules
  interval: 30s
  rules:
  # Availability SLI
  - record: sli:http_availability:ratio_rate5m
    expr: |
      sum(rate(http_requests_total{code!~"5.."}[5m])) by (service)
      / sum(rate(http_requests_total[5m])) by (service)

  # Latency SLI (% under 200ms)
  - record: sli:http_latency_200ms:ratio_rate5m
    expr: |
      sum(rate(http_request_duration_seconds_bucket{le="0.2"}[5m])) by (service)
      / sum(rate(http_request_duration_seconds_count[5m])) by (service)

  # Error budget remaining (30-day)
  - record: sli:error_budget_remaining:ratio
    expr: |
      1 - (
        (1 - sli:http_availability:ratio_rate5m)
        / (1 - 0.999)
      )
```

---

## 6. Alerting Best Practices

```
  ┌──────────────────────────────────────────────────────────┐
  │          ALERTING PHILOSOPHY                             │
  │                                                          │
  │  SYMPTOM-BASED (preferred)    vs    CAUSE-BASED (avoid)  │
  │  ─────────────────────              ─────────────────    │
  │  "Error rate is 5%"           vs    "CPU is at 90%"     │
  │  "Latency p99 > 2s"          vs    "Disk at 80%"       │
  │  "SLO burn rate > 14x"       vs    "Pod restarted"     │
  │                                                          │
  │  Why symptoms?                                          │
  │  • Directly indicates user impact                       │
  │  • Fewer false positives                                │
  │  • CPU at 90% might be normal under load                │
  │  • A pod restart might be routine                       │
  │                                                          │
  │  ┌─────────────────────────────────────────────────┐    │
  │  │  Alert Routing:                                 │    │
  │  │  PAGE (wake up)  → SLO burn rate critical       │    │
  │  │  TICKET (next day)→ Elevated error rate         │    │
  │  │  LOG (info)       → Individual error occurrence │    │
  │  └─────────────────────────────────────────────────┘    │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Reliability Dashboards

### Dashboard Layout

```
  ┌──────────────────────────────────────────────────────────┐
  │  SERVICE RELIABILITY DASHBOARD — payment-api              │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐│
  │  │ Availability│ │ Error Budget│ │ Current SLO Status  ││
  │  │   99.94%    │ │  67% left   │ │   ✅ COMPLIANT     ││
  │  │  (SLO:99.9%)│ │ (28.9 min)  │ │                     ││
  │  └─────────────┘ └─────────────┘ └─────────────────────┘│
  │                                                          │
  │  ┌── Latency (p50/p95/p99) ──────────────────────────┐  │
  │  │  p50: 42ms    p95: 180ms    p99: 350ms            │  │
  │  │  ───────────────────────────────────────           │  │
  │  │  [graph showing latency over 24h]                  │  │
  │  └───────────────────────────────────────────────────┘  │
  │                                                          │
  │  ┌── Traffic ─────────────┐ ┌── Error Rate ──────────┐  │
  │  │  1,200 req/s           │ │  0.06%                 │  │
  │  │  [graph]               │ │  [graph]               │  │
  │  └────────────────────────┘ └────────────────────────┘  │
  │                                                          │
  │  ┌── Saturation ─────────────────────────────────────┐  │
  │  │  CPU: 45%   Memory: 62%   Pods: 8/12 ready       │  │
  │  └───────────────────────────────────────────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Real-World Scenario

### [SCENARIO] Building a Reliability Measurement System

```
  Company: FinServ Corp — Online Trading Platform
  
  Step 1: Instrument the application
  ┌────────────────────────────────────────────────┐
  │  // Go application with Prometheus client      │
  │  var (                                         │
  │    httpRequests = prometheus.NewCounterVec(     │
  │      prometheus.CounterOpts{                   │
  │        Name: "http_requests_total",            │
  │      }, []string{"method", "path", "status"})  │
  │                                                │
  │    httpDuration = prometheus.NewHistogramVec(   │
  │      prometheus.HistogramOpts{                 │
  │        Name:    "http_request_duration_secs",  │
  │        Buckets: []float64{.01,.05,.1,.2,.5,1}, │
  │      }, []string{"method", "path"})            │
  │  )                                             │
  └────────────────────────────────────────────────┘
  
  Step 2: Create recording rules for SLIs
  Step 3: Build Grafana dashboard (golden signals)
  Step 4: Set up SLO burn rate alerts
  Step 5: Route to PagerDuty for on-call
  
  Result:
  • MTTD reduced from 15 min → 2 min (alerting)
  • MTTR reduced from 45 min → 12 min (dashboards +runbooks)
  • Availability improved: 99.8% → 99.95%
```

---

## 9. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Too many alerts, alert fatigue" | Only alert on symptoms. Remove any alert that doesn't require human action. |
| "Dashboards too complex" | Start with Four Golden Signals per service. Add detail panels below. |
| "Metrics cardinality explosion" | Limit label values. Don't use user IDs as labels. Use recording rules. |
| "MTTR isn't improving" | Break MTTR into MTTD + MTTA + diagnosis + fix. Optimize the slowest component. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **MTTR Components** | MTTD + MTTA + Diagnosis + Fix time |
| **Four Golden Signals** | Latency, Traffic, Errors, Saturation |
| **Three Pillars** | Metrics (Prometheus), Logs (Loki/ELK), Traces (Jaeger) |
| **Alerting Philosophy** | Symptom-based (user impact) not cause-based (infra metrics) |
| **Alert Types** | PAGE (immediate), TICKET (next day), LOG (informational) |
| **Best Practice** | Reduce MTTR by investing in detection and runbooks |

---

## Quick Revision Questions

1. **What are the components of MTTR and which is usually the biggest?**
   > MTTD + MTTA + Diagnosis + Fix. Detection (MTTD) is often the biggest — invest in monitoring.

2. **Name the Four Golden Signals and why they matter.**
   > Latency, Traffic, Errors, Saturation. They directly reflect user experience and service health.

3. **Why is symptom-based alerting preferred over cause-based?**
   > CPU at 90% might be normal; error rate at 5% always means users are affected. Symptoms have fewer false positives.

4. **What is the recommended alert routing hierarchy?**
   > PAGE for immediate action (SLO burn critical), TICKET for next-day (elevated metrics), LOG for informational only.

5. **How do you avoid metrics cardinality explosion?**
   > Limit label values, don't use high-cardinality fields (user IDs) as labels, use recording rules for pre-aggregation.

---

[← Previous: Availability Calculations](01-availability-calculations.md) | [Back to README](../README.md) | [Next: Risk Assessment →](03-risk-assessment.md)
