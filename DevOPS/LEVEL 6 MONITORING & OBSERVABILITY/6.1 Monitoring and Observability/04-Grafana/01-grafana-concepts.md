# Chapter 1: Grafana Concepts

[← Previous: Prometheus Storage](../03-Prometheus/09-storage-retention.md) | [Back to README](../README.md) | [Next: Data Sources →](02-data-sources.md)

---

## Overview

Grafana is an open-source visualization and analytics platform. It connects to multiple data sources and provides rich dashboards, alerting, and exploration capabilities.

---

## 1.1 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     GRAFANA ARCHITECTURE                     │
│                                                              │
│  ┌──────────────────────────────────────────┐               │
│  │              GRAFANA SERVER               │               │
│  │                                           │               │
│  │  ┌──────────┐  ┌──────────┐  ┌────────┐ │               │
│  │  │Dashboard │  │ Alerting │  │  Auth  │ │               │
│  │  │ Engine   │  │ Engine   │  │(LDAP,  │ │               │
│  │  │          │  │          │  │ OAuth) │ │               │
│  │  └──────────┘  └──────────┘  └────────┘ │               │
│  │                                           │               │
│  │  ┌──────────┐  ┌──────────┐  ┌────────┐ │               │
│  │  │ Plugin   │  │ Explore  │  │  API   │ │               │
│  │  │ System   │  │ Mode     │  │(REST)  │ │               │
│  │  └──────────┘  └──────────┘  └────────┘ │               │
│  └───────────────────┬───────────────────────┘               │
│                      │                                       │
│      ┌───────────────┼───────────────┐                      │
│      │               │               │                      │
│  ┌───▼────┐    ┌─────▼────┐   ┌─────▼──────┐              │
│  │Promethe│    │Elasticse-│   │   Loki     │              │
│  │us/Mimir│    │arch      │   │            │              │
│  └────────┘    └──────────┘   └────────────┘              │
│                                                              │
│  Database: SQLite (default) / MySQL / PostgreSQL            │
│  (stores dashboards, users, orgs, alerts)                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.2 Key Concepts

| Concept | Description |
|---------|-------------|
| **Dashboard** | Collection of panels arranged in a grid layout |
| **Panel** | Single visualization (graph, gauge, table, etc.) |
| **Data Source** | Backend connection (Prometheus, ES, Loki, etc.) |
| **Organization** | Logical tenant — separate dashboards and data sources |
| **Folder** | Group dashboards for access control |
| **Variable** | Dynamic values for reusable dashboards (dropdown selectors) |
| **Annotation** | Event markers on graphs (deploys, incidents) |
| **Explore** | Ad-hoc query mode (no dashboard needed) |
| **Alert Rule** | Condition-triggered notifications |
| **Plugin** | Extend Grafana with new panels, data sources, apps |

---

## 1.3 Installation

```bash
# Docker
docker run -d --name grafana -p 3000:3000 grafana/grafana:10.3.0

# Docker Compose (with persistence)
# grafana:
#   image: grafana/grafana:10.3.0
#   ports: ["3000:3000"]
#   volumes: [grafana_data:/var/lib/grafana]
#   environment:
#     GF_SECURITY_ADMIN_PASSWORD: admin

# Helm (Kubernetes)
helm install grafana grafana/grafana \
  --set adminPassword=admin \
  --set persistence.enabled=true
```

Default login: `admin` / `admin` (change immediately!)

---

## 1.4 Grafana vs Prometheus UI

```
┌──────────────────┬───────────────────┬──────────────────────┐
│ Feature          │ Prometheus UI     │ Grafana              │
├──────────────────┼───────────────────┼──────────────────────┤
│ Querying         │ PromQL only       │ Multi-source         │
│ Visualization    │ Basic line graph  │ Rich: graphs, gauges │
│                  │                   │ heatmaps, tables etc │
│ Dashboards       │ None              │ Full dashboard mgmt  │
│ Alerting         │ Rules only        │ Full alert pipeline  │
│ Auth             │ Basic             │ LDAP, OAuth, SAML    │
│ Sharing          │ URLs              │ Snapshots, embed,    │
│                  │                   │ PDF export           │
│ Variables        │ None              │ Template variables   │
│ Use case         │ Quick debugging   │ Production dashboards│
└──────────────────┴───────────────────┴──────────────────────┘
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Grafana** | Open-source visualization and analytics platform |
| **Dashboard** | Grid of panels showing different visualizations |
| **Data Source** | Backend connection to metrics/logs/traces stores |
| **Explore** | Ad-hoc query mode for investigation |
| **Default port** | 3000 |
| **Database** | SQLite (default), MySQL, or PostgreSQL |

---

## Quick Revision Questions

1. **What is Grafana and what role does it play in an observability stack?**
2. **What database does Grafana use by default? What are the alternatives?**
3. **What is the difference between a Dashboard, Panel, and Data Source?**
4. **How does Grafana's Explore mode differ from dashboards?**
5. **Compare Grafana with the built-in Prometheus UI.**

---

[← Previous: Prometheus Storage](../03-Prometheus/09-storage-retention.md) | [Back to README](../README.md) | [Next: Data Sources →](02-data-sources.md)
