# Chapter 1: Prometheus Architecture

[← Previous: Aggregations](../02-Metrics/05-aggregations.md) | [Back to README](../README.md) | [Next: Installation & Configuration →](02-installation-configuration.md)

---

## Overview

Prometheus is an open-source monitoring and alerting toolkit originally built at SoundCloud. It is now a graduated CNCF project and the de facto standard for metrics monitoring in cloud-native environments.

---

## 1.1 Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                     PROMETHEUS ARCHITECTURE                          │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │                   PROMETHEUS SERVER                      │        │
│  │                                                          │        │
│  │  ┌──────────┐  ┌───────────┐  ┌──────────────┐         │        │
│  │  │ Retrieval │  │   TSDB    │  │  HTTP Server │         │        │
│  │  │ (Scraper) │  │ (Storage) │  │  (PromQL API)│         │        │
│  │  └────┬─────┘  └───────────┘  └──────┬───────┘         │        │
│  │       │                               │                  │        │
│  │  ┌────┴─────┐                   ┌─────┴──────┐          │        │
│  │  │ Service  │                   │  Rule      │          │        │
│  │  │Discovery │                   │  Engine    │          │        │
│  │  └──────────┘                   └────────────┘          │        │
│  └──────┬───────────────────────────────┬──────────────────┘        │
│         │                               │                            │
│    ┌────▼────┐                    ┌─────▼──────┐                    │
│    │ Targets │                    │ Alertmanager│                    │
│    │         │                    │             │                    │
│    │┌──────┐ │                    │ Dedup       │                    │
│    ││App 1 │ │                    │ Group       │──► Email           │
│    │├──────┤ │                    │ Route       │──► Slack           │
│    ││App 2 │ │                    │ Silence     │──► PagerDuty       │
│    │├──────┤ │                    └─────────────┘                    │
│    ││App 3 │ │                                                       │
│    │├──────┤ │     ┌─────────────┐                                  │
│    ││node_ │ │     │   Grafana   │◄── Queries via PromQL           │
│    ││export│ │     │ (Dashboards)│                                  │
│    │└──────┘ │     └─────────────┘                                  │
│    └─────────┘                                                       │
│                                                                      │
│    ┌─────────┐                                                       │
│    │Pushgatew│◄── Short-lived jobs push metrics here                │
│    │ay       │                                                       │
│    └─────────┘                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 1.2 Core Components

| Component | Purpose |
|-----------|---------|
| **Prometheus Server** | Scrapes and stores metrics, evaluates rules |
| **TSDB** | Time-series database for efficient storage |
| **PromQL** | Query language for metrics |
| **Alertmanager** | Handles alert routing, grouping, silencing |
| **Pushgateway** | Accepts metrics from short-lived jobs |
| **Exporters** | Expose metrics from third-party systems (node, MySQL, etc.) |
| **Service Discovery** | Auto-discovers scrape targets |
| **Client Libraries** | Instrument application code (Go, Python, Java, etc.) |

---

## 1.3 Pull-Based Model

```
┌──────────────────────────────────────────────────────────┐
│                 PULL MODEL                                │
│                                                           │
│   Prometheus ──HTTP GET──► Target:port/metrics            │
│                                                           │
│   Every scrape_interval (e.g., 15s):                     │
│                                                           │
│   ┌────────────┐    GET /metrics    ┌─────────────┐      │
│   │ Prometheus │ ─────────────────► │  Target App │      │
│   │            │ ◄───────────────── │  :8080      │      │
│   │            │   metrics payload  │  /metrics   │      │
│   └────────────┘                    └─────────────┘      │
│                                                           │
│   Response example:                                       │
│   # HELP http_requests_total Total HTTP requests         │
│   # TYPE http_requests_total counter                     │
│   http_requests_total{method="GET"} 1234                 │
│   http_requests_total{method="POST"} 567                 │
│   # HELP http_request_duration_seconds Request duration  │
│   # TYPE http_request_duration_seconds histogram         │
│   http_request_duration_seconds_bucket{le="0.1"} 800    │
│   ...                                                     │
└──────────────────────────────────────────────────────────┘
```

### Advantages of Pull Model

- Prometheus controls scrape rate
- Easy to detect when a target is down (scrape fails)
- Targets don't need to know about Prometheus
- Can scrape targets running on developer laptops
- Targets have minimal overhead — just expose an HTTP endpoint

---

## 1.4 Data Flow

```
1. INSTRUMENTATION        2. EXPOSITION         3. SCRAPING
┌──────────────┐       ┌────────────────┐    ┌────────────┐
│ App uses     │       │ App exposes    │    │ Prometheus │
│ client       │──────►│ /metrics       │───►│ scrapes    │
│ library      │       │ endpoint       │    │ targets    │
└──────────────┘       └────────────────┘    └─────┬──────┘
                                                    │
4. STORAGE              5. QUERYING            6. ALERTING
┌──────────────┐       ┌────────────────┐    ┌────────────┐
│ TSDB stores  │       │ PromQL queries │    │ Alert rules│
│ time series  │◄──────│ via API/UI     │    │ evaluated  │──► Alertmanager
│ on disk      │       │ or Grafana     │    │ every eval │
└──────────────┘       └────────────────┘    └────────────┘
```

---

## 1.5 TSDB (Time Series Database)

```
┌──────────────────────────────────────────────────────────┐
│                    TSDB STRUCTURE                         │
│                                                           │
│  Data organized in 2-hour BLOCKS:                        │
│                                                           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │
│  │ Block 1 │  │ Block 2 │  │ Block 3 │  │  HEAD   │   │
│  │ (2 hrs) │  │ (2 hrs) │  │ (2 hrs) │  │ (active)│   │
│  │ compact │  │ compact │  │ compact │  │ in-mem  │   │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │
│  ◄── older                               newer ──►      │
│                                                           │
│  Each block contains:                                    │
│  ├── chunks/   (compressed time series data)            │
│  ├── index     (inverted index for label lookups)       │
│  ├── meta.json (block metadata)                         │
│  └── tombstones (deletion markers)                      │
│                                                           │
│  HEAD block = write-ahead log (WAL) + in-memory         │
│  Compaction merges older blocks for efficiency           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Target shows as "DOWN" | App not running or port wrong | Check `curl target:port/metrics` |
| High memory usage | Too many time series | Check `prometheus_tsdb_head_series` |
| Gaps in data | Network issues or target overloaded | Check scrape duration vs interval |
| WAL corruption | Unclean shutdown | `--storage.tsdb.wal-compression` + proper shutdown |

---

## Summary Table

| Component | Role | Key Detail |
|-----------|------|------------|
| **Server** | Core engine | Scrapes, stores, queries, alerts |
| **TSDB** | Storage | 2-hour blocks, WAL, compaction |
| **Pull model** | Collection | HTTP GET to /metrics endpoints |
| **Alertmanager** | Alert routing | Dedup, group, route, silence |
| **Exporters** | Bridges | Expose third-party metrics |
| **Pushgateway** | Bridge for batch | Short-lived jobs push here |

---

## Quick Revision Questions

1. **Draw and explain the Prometheus architecture, identifying all major components.**
2. **Why does Prometheus use a pull model? What are the advantages over pushing?**
3. **How does the Prometheus TSDB organize data? What is a "block" and what is the "HEAD"?**
4. **What is the role of the Alertmanager? How does it differ from alert rules in Prometheus?**
5. **When would you use the Pushgateway? Why isn't it recommended for long-running services?**

---

[← Previous: Aggregations](../02-Metrics/05-aggregations.md) | [Back to README](../README.md) | [Next: Installation & Configuration →](02-installation-configuration.md)
