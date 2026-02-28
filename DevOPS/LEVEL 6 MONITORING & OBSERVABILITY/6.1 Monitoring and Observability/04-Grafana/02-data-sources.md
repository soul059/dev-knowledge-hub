# Chapter 2: Data Sources

[← Previous: Grafana Concepts](01-grafana-concepts.md) | [Back to README](../README.md) | [Next: Dashboard Creation →](03-dashboard-creation.md)

---

## Overview

Data sources are the backend systems Grafana connects to for querying and visualizing data. Grafana supports dozens of data sources out of the box.

---

## 2.1 Common Data Sources

```
┌──────────────────────────────────────────────────────────┐
│                GRAFANA DATA SOURCES                       │
├──────────────┬───────────────────────────────────────────┤
│ Metrics      │ Prometheus, InfluxDB, Graphite, DataDog   │
│ Logs         │ Loki, Elasticsearch, CloudWatch Logs      │
│ Traces       │ Tempo, Jaeger, Zipkin, X-Ray              │
│ SQL          │ MySQL, PostgreSQL, MSSQL                  │
│ Cloud        │ CloudWatch, Azure Monitor, Stackdriver    │
│ Other        │ TestData, JSON API, CSV                   │
└──────────────┴───────────────────────────────────────────┘
```

---

## 2.2 Adding Prometheus Data Source

### Via UI
1. Go to **Configuration** → **Data Sources** → **Add data source**
2. Select **Prometheus**
3. Set URL: `http://prometheus:9090`
4. Click **Save & Test**

### Via Provisioning (GitOps)

```yaml
# /etc/grafana/provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy         # Grafana backend proxies requests
    url: http://prometheus:9090
    isDefault: true
    jsonData:
      httpMethod: POST    # Better for large queries
      timeInterval: 15s   # Match Prometheus scrape interval
    editable: false        # Prevent UI changes

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    jsonData:
      derivedFields:
        - datasourceUid: tempo
          matcherRegex: "trace_id=(\\w+)"
          name: TraceID
          url: "$${__value.raw}"

  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
```

---

## 2.3 Access Modes

```
┌─────────────────────────────────────────────────────┐
│  PROXY (recommended):                                │
│  Browser → Grafana Server → Data Source             │
│  ✅ Data source not exposed to browsers             │
│  ✅ Can use internal DNS                            │
│                                                      │
│  DIRECT (browser):                                   │
│  Browser → Data Source (directly)                   │
│  ⚠️ Data source must be accessible from browser    │
│  ⚠️ Credentials exposed to client                  │
└─────────────────────────────────────────────────────┘
```

---

## 2.4 Correlating Data Sources

```
┌──────────────────────────────────────────────────────────┐
│     CORRELATED DATA SOURCES (Metrics → Logs → Traces)   │
│                                                           │
│  Prometheus metric spike                                  │
│       │                                                   │
│       ▼ "View logs" button                               │
│  Loki logs filtered by service + time range              │
│       │                                                   │
│       ▼ Click trace_id in log line                       │
│  Tempo trace showing full request flow                   │
│                                                           │
│  Configuration: Derived fields in Loki data source       │
│  link trace_id regex to Tempo data source                │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Data Source | Type | Query Language |
|------------|------|---------------|
| Prometheus | Metrics | PromQL |
| Loki | Logs | LogQL |
| Tempo | Traces | TraceQL |
| Elasticsearch | Logs/Metrics | Lucene/KQL |
| InfluxDB | Metrics | InfluxQL/Flux |
| CloudWatch | Metrics/Logs | CloudWatch query |

---

## Quick Revision Questions

1. **What is the difference between proxy and direct access modes?**
2. **How do you configure a data source using provisioning (as code)?**
3. **How can you link Prometheus metrics to Loki logs in Grafana?**
4. **What are derived fields in Loki and why are they useful?**
5. **Name five data sources Grafana supports out of the box.**

---

[← Previous: Grafana Concepts](01-grafana-concepts.md) | [Back to README](../README.md) | [Next: Dashboard Creation →](03-dashboard-creation.md)
