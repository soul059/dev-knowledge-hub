# Chapter 4.6 — Monitoring Tools

[← Previous: IaC Tools](05-iac-tools.md) | [Next: Cloud Platforms →](07-cloud-platforms.md)

---

## Overview

Monitoring tools collect, store, visualize, and alert on system and application data. This chapter covers the most important tools in the DevOps monitoring ecosystem: **Prometheus, Grafana, ELK Stack, Datadog**, and more.

---

## Monitoring Tools Ecosystem

```
┌──────────────────────────────────────────────────────────┐
│  MONITORING TOOLS BY CATEGORY                            │
│                                                          │
│  METRICS              LOGS               TRACES          │
│  ├── Prometheus       ├── ELK Stack      ├── Jaeger      │
│  ├── Datadog          ├── Loki           ├── Zipkin      │
│  ├── New Relic        ├── Splunk         ├── Tempo       │
│  ├── CloudWatch       ├── Datadog Logs   └── Datadog APM │
│  └── InfluxDB         └── CloudWatch Logs                │
│                                                          │
│  VISUALIZATION        ALERTING           UPTIME          │
│  ├── Grafana          ├── PagerDuty      ├── Uptime Robot │
│  ├── Kibana           ├── OpsGenie       ├── Pingdom     │
│  ├── Datadog Dash     ├── Alertmanager   └── StatusCake  │
│  └── CloudWatch Dash  └── VictorOps                      │
│                                                          │
│  ALL-IN-ONE: Datadog, New Relic, Dynatrace               │
│  OPEN-SOURCE STACK: Prometheus + Grafana + Loki + Tempo  │
└──────────────────────────────────────────────────────────┘
```

---

## Prometheus + Grafana Stack

```
┌──────────────────────────────────────────────────────────┐
│  PROMETHEUS + GRAFANA ARCHITECTURE                       │
│                                                          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                   │
│  │  App 1  │ │  App 2  │ │  App 3  │                   │
│  │ /metrics│ │ /metrics│ │ /metrics│  ← expose metrics  │
│  └────┬────┘ └────┬────┘ └────┬────┘                   │
│       │           │           │      PULL (scrape)      │
│  ┌────▼───────────▼───────────▼────┐                    │
│  │         Prometheus              │                    │
│  │  ├── Scrapes targets            │                    │
│  │  ├── Stores time-series data    │                    │
│  │  ├── Evaluates alert rules      │                    │
│  │  └── PromQL query language      │                    │
│  └───────────┬─────────┬───────────┘                    │
│              │         │                                 │
│  ┌───────────▼──┐ ┌────▼────────────┐                   │
│  │   Grafana    │ │ Alertmanager   │                   │
│  │  (visualize) │ │ (route alerts) │                   │
│  │              │ │  → Slack       │                   │
│  │ Dashboards   │ │  → PagerDuty  │                   │
│  │ Panels       │ │  → Email      │                   │
│  └──────────────┘ └────────────────┘                   │
└──────────────────────────────────────────────────────────┘
```

### PromQL Examples

```promql
# Request rate (requests per second over 5 minutes)
rate(http_requests_total[5m])

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m])) * 100

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

# Memory usage percentage
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
/ node_memory_MemTotal_bytes * 100

# Top 5 endpoints by request count
topk(5, sum by (endpoint) (rate(http_requests_total[1h])))
```

---

## ELK Stack (Elasticsearch + Logstash + Kibana)

```
┌──────────────────────────────────────────────────────────┐
│  ELK STACK — LOG MANAGEMENT                              │
│                                                          │
│  ┌────────────────────────────────────┐                  │
│  │  Applications / Servers            │                  │
│  │  ├── App logs (JSON)               │                  │
│  │  ├── Access logs (Nginx)           │                  │
│  │  └── System logs (syslog)          │                  │
│  └───────────────┬────────────────────┘                  │
│                  │                                       │
│  ┌───────────────▼────────────────────┐                  │
│  │  Beats (lightweight shippers)      │                  │
│  │  ├── Filebeat (logs)               │                  │
│  │  ├── Metricbeat (metrics)          │                  │
│  │  └── Packetbeat (network)          │                  │
│  └───────────────┬────────────────────┘                  │
│                  │                                       │
│  ┌───────────────▼────────────────────┐                  │
│  │  Logstash (process & transform)    │                  │
│  │  ├── Input: receive from Beats     │                  │
│  │  ├── Filter: parse, enrich, drop   │                  │
│  │  └── Output: send to Elasticsearch │                  │
│  └───────────────┬────────────────────┘                  │
│                  │                                       │
│  ┌───────────────▼────────────────────┐                  │
│  │  Elasticsearch (store & search)    │                  │
│  │  ├── Full-text search engine       │                  │
│  │  ├── Distributed and scalable      │                  │
│  │  └── RESTful API                   │                  │
│  └───────────────┬────────────────────┘                  │
│                  │                                       │
│  ┌───────────────▼────────────────────┐                  │
│  │  Kibana (visualize & explore)      │                  │
│  │  ├── Dashboards and charts         │                  │
│  │  ├── Log search and filtering      │                  │
│  │  └── Alerting rules                │                  │
│  └────────────────────────────────────┘                  │
└──────────────────────────────────────────────────────────┘
```

### Logstash Configuration Example

```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  # Parse JSON logs
  json {
    source => "message"
  }
  
  # Add geo-IP data
  geoip {
    source => "client_ip"
  }
  
  # Drop debug logs in production
  if [level] == "DEBUG" {
    drop {}
  }
  
  # Parse timestamps
  date {
    match => ["timestamp", "ISO8601"]
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

---

## Open-Source vs Commercial Comparison

| Feature | Prometheus + Grafana | ELK Stack | Datadog | New Relic |
|---------|---------------------|-----------|---------|-----------|
| **Type** | Open source | Open source | SaaS | SaaS |
| **Cost** | Free | Free | Per host/month | Per GB/month |
| **Setup** | Self-hosted | Self-hosted | Agent install | Agent install |
| **Metrics** | ✅ Native | Via Metricbeat | ✅ Native | ✅ Native |
| **Logs** | Via Loki | ✅ Native | ✅ Native | ✅ Native |
| **Traces** | Via Tempo/Jaeger | Via APM | ✅ Native | ✅ Native |
| **Maintenance** | You manage | You manage | Managed | Managed |
| **Best For** | K8s / cost-sensitive | Log-heavy analysis | Enterprise / ease | Full-stack observability |

---

## Choosing a Monitoring Stack

```
┌──────────────────────────────────────────────────────────┐
│  DECISION TREE FOR MONITORING TOOLS                      │
│                                                          │
│  "Budget?"                                               │
│  ├── Limited / Prefer Open Source                        │
│  │   ├── Metrics? → Prometheus + Grafana                 │
│  │   ├── Logs? → Loki + Grafana (or ELK)                │
│  │   └── Traces? → Jaeger or Tempo                       │
│  │                                                       │
│  └── Budget Available / Prefer Managed                   │
│      ├── "All-in-one?" → Datadog or New Relic            │
│      ├── AWS-native? → CloudWatch + X-Ray                │
│      ├── GCP-native? → Cloud Monitoring + Trace          │
│      └── Azure-native? → Azure Monitor + App Insights    │
│                                                          │
│  Universal: Use OpenTelemetry (OTel) as the              │
│  instrumentation layer — it works with ANY backend.      │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Building a Monitoring Stack from Scratch

```
COMPANY: Growing startup, 15 microservices on Kubernetes

REQUIREMENTS:
├── Monitor all 15 services (metrics + logs + traces)
├── Dashboard for SRE team and developers
├── Alert on SLO violations
├── Budget: $0 for tooling (use open source)
└── Must handle 50k metrics/sec and 10GB logs/day

SOLUTION: Prometheus + Grafana + Loki + Tempo

Deployment (via Helm charts):
  helm install prometheus prometheus-community/kube-prometheus-stack
  helm install loki grafana/loki-stack
  helm install tempo grafana/tempo

Architecture:
  ┌─────────────────────────────────────────────┐
  │ Kubernetes Cluster                          │
  │                                             │
  │  Apps ──► Prometheus (metrics)              │
  │   │   ──► Loki       (logs via Promtail)    │
  │   └─────► Tempo      (traces via OTel)      │
  │                │          │         │       │
  │                └──────────┼─────────┘       │
  │                     ┌─────▼──────┐          │
  │                     │  Grafana   │          │
  │                     │ (unified   │          │
  │                     │ dashboard) │          │
  │                     └────────────┘          │
  └─────────────────────────────────────────────┘

RESULT:
├── Single Grafana dashboard for all three pillars
├── Alert rules via Prometheus Alertmanager → Slack
├── Developers self-serve via Grafana explore
├── Total cost: $0 (open-source tools)
└── Scales to handle growth for next 2+ years
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Prometheus running out of disk | High cardinality metrics | Reduce label cardinality; set retention period |
| Grafana dashboard slow | Too many panels / long time range | Reduce panels; use `$__interval`; limit data range |
| Elasticsearch out of heap | Index too large | Implement index lifecycle management (ILM); add nodes |
| Missing metrics | Scrape target unreachable | Check service discovery; verify `/metrics` endpoint |
| Log entries missing | Logstash pipeline bottleneck | Scale Logstash horizontally; use Beats directly to ES |
| Alert not firing | Wrong PromQL expression or `for` duration | Test expression in Prometheus UI; check Alertmanager routing |

---

## Summary Table

| Monitoring Tool | Description |
|----------------|-------------|
| **Prometheus** | Open-source metrics collection with pull model and PromQL |
| **Grafana** | Visualization tool supporting multiple data sources |
| **ELK Stack** | Log management: Elasticsearch + Logstash + Kibana |
| **Loki** | Lightweight log aggregation (like Prometheus for logs) |
| **Jaeger / Tempo** | Distributed tracing backends |
| **Datadog** | Commercial all-in-one observability platform |
| **OpenTelemetry** | Vendor-neutral instrumentation standard |
| **Alertmanager** | Routes and deduplicates Prometheus alerts |

---

## Quick Revision Questions

1. **How does Prometheus collect metrics, and why is this called a "pull" model?**
   <details><summary>Answer</summary>Prometheus actively scrapes (pulls) metrics from HTTP endpoints (/metrics) on target applications at configured intervals (e.g., every 15 seconds). This contrasts with a "push" model where applications send metrics to the monitoring system. Pull advantages: Prometheus controls the scrape rate, can detect if a target is down (no response), and targets don't need to know about Prometheus.</details>

2. **What is PromQL, and give an example of a useful query?**
   <details><summary>Answer</summary>PromQL (Prometheus Query Language) is used to query time-series data in Prometheus. Example: `rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100` calculates the error rate percentage over the last 5 minutes by dividing 5xx requests by total requests.</details>

3. **What is the difference between the ELK stack and the Loki + Grafana approach?**
   <details><summary>Answer</summary>ELK (Elasticsearch + Logstash + Kibana): Full-text indexes all log content — powerful searching but resource-intensive, requires cluster management. Loki + Grafana: Only indexes labels/metadata (not log content) — much lighter, costs less to run, uses Grafana for querying. Loki is often called "Prometheus for logs." Choose ELK for heavy log analysis; choose Loki for cost-effective log storage.</details>

4. **When would you choose a commercial tool like Datadog over open-source options?**
   <details><summary>Answer</summary>Choose commercial when: 1) Team doesn't have capacity to manage monitoring infrastructure. 2) Need all-in-one solution (metrics + logs + traces + APM). 3) Want pre-built integrations for 500+ technologies. 4) Require enterprise support and SLAs. 5) Speed of setup matters more than cost. Choose open-source when budget is constrained and team has operational expertise.</details>

5. **What is OpenTelemetry, and why is it important?**
   <details><summary>Answer</summary>OpenTelemetry (OTel) is a vendor-neutral, open-source standard for collecting telemetry data (traces, metrics, logs). It provides SDKs and a collector that works with any backend (Jaeger, Prometheus, Datadog, etc.). Importance: instrument once, switch backends without code changes. It prevents vendor lock-in and is becoming the universal instrumentation standard.</details>

6. **How do you prevent Prometheus from running out of disk space?**
   <details><summary>Answer</summary>1) Set a retention period: `--storage.tsdb.retention.time=30d`. 2) Set a storage size limit: `--storage.tsdb.retention.size=50GB`. 3) Reduce metric cardinality — avoid high-cardinality labels (user IDs, request IDs). 4) Use recording rules to pre-aggregate expensive queries. 5) Use remote write to long-term storage (Thanos, Cortex) for historical data.</details>

---

[← Previous: IaC Tools](05-iac-tools.md) | [Next: Cloud Platforms →](07-cloud-platforms.md)
