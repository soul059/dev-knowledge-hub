# Chapter 2: APM Tools

[← Previous: APM Concepts](01-apm-concepts.md) | [Back to README](../README.md) | [Next: Synthetic Monitoring →](03-synthetic-monitoring.md)

---

## Overview

This chapter compares major APM tools — both commercial (Datadog, New Relic, Dynatrace) and open-source (Elastic APM, SigNoz, Grafana Stack) — covering their strengths, architecture, and when to use each.

---

## 2.1 APM Tool Landscape

```
┌──────────────────────────────────────────────────────────┐
│             APM TOOL LANDSCAPE                           │
│                                                          │
│  COMMERCIAL (Full-Stack, SaaS):                         │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐  │
│  │ Datadog │ │ New Relic│ │ Dynatrace │ │Splunk APM│  │
│  │         │ │          │ │           │ │          │  │
│  │All-in-1 │ │Full stack│ │AI-powered │ │Log+APM   │  │
│  │platform │ │observ.   │ │auto-disc. │ │combined  │  │
│  └─────────┘ └──────────┘ └───────────┘ └──────────┘  │
│                                                          │
│  OPEN-SOURCE (Self-Hosted):                             │
│  ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐  │
│  │ Grafana │ │ Elastic  │ │ SigNoz   │ │ Uptrace  │  │
│  │ Stack   │ │ APM      │ │          │ │          │  │
│  │Prom+    │ │ELK+APM  │ │OTel-     │ │OTel +    │  │
│  │Tempo+   │ │agents    │ │native    │ │ClickHouse│  │
│  │Loki     │ │          │ │          │ │          │  │
│  └─────────┘ └──────────┘ └───────────┘ └──────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 Tool Comparison

| Feature | Datadog | New Relic | Dynatrace | Grafana Stack | SigNoz |
|---------|---------|-----------|-----------|---------------|--------|
| Hosting | SaaS | SaaS | SaaS/On-prem | Self-hosted | Self-hosted |
| Pricing Model | Per host + usage | Per GB ingested | Per host (full-stack) | Free (infra costs) | Free (infra costs) |
| Traces | Yes | Yes | Yes (PurePath) | Tempo | ClickHouse |
| Metrics | Yes | Yes | Yes | Prometheus | ClickHouse |
| Logs | Yes | Yes | Yes | Loki | ClickHouse |
| RUM | Yes | Yes | Yes | Grafana Faro | Planned |
| Profiling | Continuous | Code-level | PurePath | Pyroscope | No |
| AI/ML | Watchdog | Applied Intelligence | Davis AI | No | No |
| OTel Support | Good | Excellent | Good | Native | Native |
| Setup Effort | Low | Low | Low | High | Medium |
| Vendor Lock-in | High | Medium | High | None | Low |
| Best For | All-in-one | Cost-predictable | Enterprise, complex | Full control | OTel-first |

---

## 2.3 Datadog APM

```
┌──────────────────────────────────────────────────────────┐
│  DATADOG APM ARCHITECTURE                                │
│                                                          │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ App +    │───►│ Datadog Agent│───►│ Datadog SaaS │  │
│  │ DD Tracer│    │ (Host-level) │    │ Backend      │  │
│  └──────────┘    └──────────────┘    └──────┬───────┘  │
│                                             │           │
│  Features:                                   ▼          │
│  • Auto-instrumentation (15+ languages)  ┌────────┐    │
│  • Continuous profiler                   │Datadog │    │
│  • Trace → Metric → Log correlation     │  UI    │    │
│  • Watchdog AI anomaly detection         └────────┘    │
│  • Service catalog                                      │
│  • Error tracking                                       │
└──────────────────────────────────────────────────────────┘
```

### Datadog Agent Setup (Kubernetes)

```yaml
# Datadog Helm values
datadog:
  apiKey: <DATADOG_API_KEY>
  appKey: <DATADOG_APP_KEY>
  site: datadoghq.com
  
  apm:
    portEnabled: true
    socketEnabled: true
  
  logs:
    enabled: true
    containerCollectAll: true
  
  processAgent:
    enabled: true
    processCollection: true

  dogstatsd:
    useHostPort: true

agents:
  tolerations:
    - operator: Exists
```

---

## 2.4 Grafana Full-Stack (Open Source)

```
┌──────────────────────────────────────────────────────────┐
│  GRAFANA OBSERVABILITY STACK                             │
│                                                          │
│  ┌──────────┐    ┌──────────────────┐                   │
│  │ App +    │───►│ OTel Collector   │                   │
│  │ OTel SDK │    └────┬───┬────┬───┘                   │
│  └──────────┘         │   │    │                        │
│                       ▼   ▼    ▼                        │
│           ┌────────┐ ┌──┐ ┌────────┐ ┌─────────┐      │
│           │Promethe│ │Te│ │  Loki  │ │Pyroscope│      │
│           │us/Mimir│ │mp│ │ (Logs) │ │(Profile)│      │
│           │(Metric)│ │o │ │        │ │         │      │
│           └───┬────┘ └┬─┘ └───┬────┘ └────┬────┘      │
│               └───────┼───────┼───────────┘            │
│                       ▼                                 │
│              ┌──────────────┐                          │
│              │   GRAFANA    │                          │
│              │ Unified UI   │                          │
│              │              │                          │
│              │ Metrics ←→ Traces ←→ Logs ←→ Profiles  │
│              └──────────────┘                          │
│                                                          │
│  Advantages: No vendor lock-in, OTel-native,           │
│  cost-effective at scale, full customization            │
│                                                          │
│  Challenges: Operational overhead, no AI features,     │
│  requires expertise to set up                           │
└──────────────────────────────────────────────────────────┘
```

### Grafana Stack Docker Compose

```yaml
version: "3.8"
services:
  # Metrics
  prometheus:
    image: prom/prometheus:v2.48.0
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  # Traces
  tempo:
    image: grafana/tempo:2.3.1
    command: ["-config.file=/etc/tempo.yaml"]
    volumes:
      - ./tempo.yaml:/etc/tempo.yaml
    ports:
      - "3200:3200"
      - "4317:4317"

  # Logs
  loki:
    image: grafana/loki:2.9.3
    ports:
      - "3100:3100"

  # Profiles
  pyroscope:
    image: grafana/pyroscope:1.2.0
    ports:
      - "4040:4040"

  # Collector
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.91.0
    volumes:
      - ./otel-collector.yaml:/etc/otelcol/config.yaml
    ports:
      - "4318:4318"

  # UI
  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"
    environment:
      GF_AUTH_ANONYMOUS_ENABLED: "true"
```

---

## 2.5 Cost Comparison

```
┌──────────────────────────────────────────────────────────┐
│  APM COST COMPARISON (100 services, ~50M spans/day)      │
│                                                          │
│  Platform          Monthly Cost    Notes                 │
│  ──────────────────────────────────────────────────      │
│  Datadog           $15,000-25,000  Per host + traces     │
│  New Relic         $5,000-10,000   Per GB ingested       │
│  Dynatrace         $20,000-30,000  Per host full-stack   │
│                                                          │
│  Grafana Stack     $2,000-5,000    Infrastructure only   │
│  (self-hosted)                     + engineering time    │
│                                                          │
│  Grafana Cloud     $3,000-8,000    Managed, usage-based  │
│  (managed)                                               │
│                                                          │
│  SigNoz Cloud      $2,000-4,000    Usage-based           │
│                                                          │
│  Key insight: Open-source costs less in $$$ but more    │
│  in engineering time. Break-even at ~50+ hosts.         │
└──────────────────────────────────────────────────────────┘
```

---

## 2.6 Choosing the Right Tool

```
┌──────────────────────────────────────────────────────────┐
│  APM TOOL DECISION TREE                                  │
│                                                          │
│  Do you have dedicated platform/SRE team?               │
│  ├─ NO → Commercial SaaS (Datadog/New Relic)            │
│  └─ YES ──► Budget constraints?                         │
│             ├─ YES → Open-source (Grafana Stack/SigNoz) │
│             └─ NO ──► Need AI anomaly detection?        │
│                       ├─ YES → Dynatrace or Datadog     │
│                       └─ NO ──► Want vendor freedom?    │
│                                ├─ YES → Grafana Stack   │
│                                └─ NO → Datadog/New Relic│
│                                                          │
│  Migration Path:                                        │
│  Start with OTel instrumentation (vendor-neutral)       │
│  → Send to any backend → Switch vendors easily          │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| APM costs too high | Sending all data | Implement sampling, focus on critical services first |
| Agent slowing app | Heavy profiling | Reduce profiling frequency, use async profiler |
| Missing service in map | Service not instrumented | Add OTel SDK or APM agent |
| Wrong latency numbers | Clock skew between hosts | Use NTP sync, prefer relative timing |
| Tool migration painful | Vendor-specific instrumentation | Use OpenTelemetry for vendor-neutral instrumentation |

---

## Summary Table

| Tool | Type | Strength | Best For |
|------|------|----------|----------|
| Datadog | Commercial SaaS | All-in-one, AI | Teams wanting easy setup |
| New Relic | Commercial SaaS | Predictable pricing | Cost-conscious orgs |
| Dynatrace | Commercial | AI-powered, auto-discovery | Enterprise, complex environments |
| Grafana Stack | Open Source | No vendor lock-in, customizable | Teams with platform expertise |
| SigNoz | Open Source | OTel-native, ClickHouse | OTel-first environments |
| Elastic APM | Open Source | Integrated with ELK | Existing ELK users |

---

## Quick Revision Questions

1. **What are the main differences between commercial and open-source APM tools?**
2. **How does the Grafana observability stack compare to Datadog?**
3. **Why is OpenTelemetry important for APM tool selection?**
4. **When does self-hosted APM become more cost-effective than SaaS?**
5. **What factors should you consider when choosing an APM tool?**

---

[← Previous: APM Concepts](01-apm-concepts.md) | [Back to README](../README.md) | [Next: Synthetic Monitoring →](03-synthetic-monitoring.md)
