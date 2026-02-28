# Chapter 8: Provisioning Dashboards as Code

[← Previous: Dashboard Best Practices](07-dashboard-best-practices.md) | [Back to README](../README.md) | [Next: Log Management Concepts →](../05-Logging/01-log-management-concepts.md)

---

## Overview

Dashboard provisioning allows you to manage Grafana dashboards, data sources, and alert rules as code — enabling version control, CI/CD, and consistent environments.

---

## 8.1 Provisioning Architecture

```
┌──────────────────────────────────────────────────────────┐
│              GRAFANA PROVISIONING FLOW                    │
│                                                          │
│  Git Repository                                          │
│  ├── datasources/                                        │
│  │   └── prometheus.yml                                  │
│  ├── dashboards/                                         │
│  │   ├── provider.yml                                    │
│  │   └── json/                                           │
│  │       ├── overview.json                               │
│  │       └── service-detail.json                         │
│  ├── alerting/                                           │
│  │   └── rules.yml                                       │
│  └── notifiers/                                          │
│      └── slack.yml                                       │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────┐                                  │
│  │ /etc/grafana/       │   Grafana reads on startup      │
│  │  provisioning/      │   and watches for changes       │
│  │  ├── datasources/   │                                 │
│  │  ├── dashboards/    │                                 │
│  │  └── alerting/      │                                 │
│  └────────────────────┘                                  │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────┐                                  │
│  │    GRAFANA          │   Dashboards loaded              │
│  │    (read-only)      │   from provisioned files        │
│  └────────────────────┘                                  │
└──────────────────────────────────────────────────────────┘
```

---

## 8.2 Data Source Provisioning

```yaml
# /etc/grafana/provisioning/datasources/all.yml
apiVersion: 1

deleteDatasources:
  - name: Old-Prometheus
    orgId: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
    jsonData:
      httpMethod: POST
      exemplarTraceIdDestinations:
        - name: traceID
          datasourceUid: tempo

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100

  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    uid: tempo
```

---

## 8.3 Dashboard Provisioning

### Step 1: Dashboard Provider Config

```yaml
# /etc/grafana/provisioning/dashboards/provider.yml
apiVersion: 1

providers:
  - name: default
    orgId: 1
    folder: "Provisioned"
    folderUid: provisioned
    type: file
    disableDeletion: true    # Prevent deletion via UI
    updateIntervalSeconds: 30  # Check for changes every 30s
    allowUiUpdates: false    # Read-only in UI
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true  # Auto-create folders
```

### Step 2: Dashboard JSON Files

```json
{
  "uid": "api-overview",
  "title": "API Overview",
  "tags": ["production", "api"],
  "timezone": "browser",
  "editable": false,
  "panels": [
    {
      "id": 1,
      "type": "stat",
      "title": "Current RPS",
      "gridPos": { "h": 4, "w": 6, "x": 0, "y": 0 },
      "targets": [{
        "expr": "sum(rate(http_requests_total{job=\"api\"}[5m]))",
        "legendFormat": "RPS"
      }],
      "fieldConfig": {
        "defaults": {
          "unit": "reqps",
          "thresholds": {
            "steps": [
              { "color": "green", "value": null },
              { "color": "yellow", "value": 500 },
              { "color": "red", "value": 1000 }
            ]
          }
        }
      }
    }
  ],
  "templating": {
    "list": [
      {
        "name": "env",
        "type": "query",
        "query": "label_values(http_requests_total, env)",
        "datasource": "Prometheus"
      }
    ]
  },
  "time": { "from": "now-1h", "to": "now" },
  "refresh": "30s"
}
```

---

## 8.4 Generating Dashboards with Grafonnet

Grafonnet is a Jsonnet library for generating Grafana dashboards programmatically:

```jsonnet
// dashboard.jsonnet
local grafana = import 'grafonnet/grafana.libsonnet';
local dashboard = grafana.dashboard;
local prometheus = grafana.prometheus;
local graphPanel = grafana.graphPanel;

dashboard.new(
  'API Overview',
  tags=['production', 'api'],
  time_from='now-1h',
  refresh='30s',
)
.addPanel(
  graphPanel.new(
    'Request Rate',
    datasource='Prometheus',
  )
  .addTarget(
    prometheus.target(
      'sum(rate(http_requests_total{job="api"}[5m]))',
      legendFormat='{{method}} {{status}}',
    )
  ),
  gridPos={ h: 8, w: 12, x: 0, y: 0 },
)
```

Build:
```bash
# Install jsonnet
brew install jsonnet

# Install grafonnet
jb install github.com/grafana/grafonnet-lib/grafonnet

# Generate JSON
jsonnet -J vendor dashboard.jsonnet > dashboard.json
```

---

## 8.5 Docker Compose with Provisioning

```yaml
# docker-compose.yml
version: "3.8"
services:
  grafana:
    image: grafana/grafana:10.2.0
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin123
      GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH: /var/lib/grafana/dashboards/overview.json
    volumes:
      - ./provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./dashboards:/var/lib/grafana/dashboards
    depends_on:
      - prometheus

  prometheus:
    image: prom/prometheus:v2.48.0
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

---

## 8.6 Kubernetes ConfigMap Provisioning

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
  labels:
    grafana_dashboard: "1"    # Sidecar picks this up
data:
  api-overview.json: |
    {
      "uid": "api-overview",
      "title": "API Overview",
      "panels": [...]
    }
---
# Grafana Helm values.yaml
grafana:
  sidecar:
    dashboards:
      enabled: true
      label: grafana_dashboard
      searchNamespace: ALL
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Prometheus
          type: prometheus
          url: http://prometheus-server:9090
          isDefault: true
```

---

## 8.7 CI/CD Pipeline for Dashboards

```
┌────────────┐     ┌────────────┐     ┌────────────┐
│ Developer  │     │ CI Pipeline│     │ Grafana    │
│ edits JSON │────▶│ Validate   │────▶│ Deploy     │
│ in Git     │     │ & Lint     │     │ Provisioned│
└────────────┘     └────────────┘     └────────────┘

Pipeline Steps:
  1. Lint JSON: jsonlint dashboards/*.json
  2. Validate: grafana-toolkit dashboard:validate
  3. Test: Deploy to staging Grafana
  4. Deploy: Copy to production provisioning path
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Dashboard not appearing | Wrong path in provider.yml | Verify `options.path` matches volume mount |
| "Dashboard is read-only" | `allowUiUpdates: false` | Change setting or copy dashboard to make editable |
| Changes not picked up | Update interval too long | Decrease `updateIntervalSeconds` or restart |
| Duplicate dashboards | Same UID in multiple files | Ensure unique UIDs across all JSON files |
| Data source "not found" | Data sources load after dashboards | Set correct provisioning order or use names |

---

## Quick Revision Questions

1. **What are the three types of resources Grafana can provision?**
2. **How does the dashboard provider file work?**
3. **What is Grafonnet and why would you use it?**
4. **How do you provision dashboards in Kubernetes using Helm?**
5. **Why should you version control dashboard JSON files?**
6. **What CI/CD steps would you include for dashboard deployment?**

---

[← Previous: Dashboard Best Practices](07-dashboard-best-practices.md) | [Back to README](../README.md) | [Next: Log Management Concepts →](../05-Logging/01-log-management-concepts.md)
