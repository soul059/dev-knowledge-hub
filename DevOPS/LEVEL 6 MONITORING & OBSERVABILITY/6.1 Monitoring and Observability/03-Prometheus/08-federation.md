# Chapter 8: Federation

[← Previous: Service Discovery](07-service-discovery.md) | [Back to README](../README.md) | [Next: Storage & Retention →](09-storage-retention.md)

---

## Overview

Federation allows one Prometheus server to scrape selected metrics from another. It's essential for scaling Prometheus across multiple clusters or teams.

---

## 8.1 Federation Patterns

```
┌─────────────────────────────────────────────────────────────┐
│               HIERARCHICAL FEDERATION                        │
│                                                              │
│                ┌────────────────┐                            │
│                │ Global Prom    │  Stores aggregated metrics │
│                │ (cross-cluster)│  from all clusters         │
│                └───────┬────────┘                            │
│                        │ scrapes /federate                   │
│            ┌───────────┼───────────┐                        │
│            │           │           │                        │
│     ┌──────▼──┐  ┌─────▼───┐  ┌───▼──────┐                │
│     │Cluster A│  │Cluster B│  │Cluster C │                │
│     │  Prom   │  │  Prom   │  │  Prom    │                │
│     └─────────┘  └─────────┘  └──────────┘                │
│      scrapes      scrapes       scrapes                     │
│      local        local         local                       │
│      targets      targets       targets                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 8.2 Configuration

```yaml
# On the GLOBAL Prometheus — scrape from regional instances
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 60s
    honor_labels: true       # Keep original labels
    metrics_path: '/federate'
    params:
      'match[]':
        # Pull only recording rules (precomputed)
        - '{__name__=~"job:.*"}'
        # Or specific important metrics
        - 'up'
        - '{__name__=~".*:.*:rate5m"}'
    static_configs:
      - targets:
          - 'prometheus-cluster-a:9090'
          - 'prometheus-cluster-b:9090'
          - 'prometheus-cluster-c:9090'
```

---

## 8.3 Federation vs Remote Write vs Thanos

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Aspect       │ Federation   │ Remote Write │ Thanos/Mimir │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Approach     │ Pull subset  │ Push all     │ Query across │
│ Data loss    │ Only summary │ Full data    │ Full data    │
│ Complexity   │ Low          │ Medium       │ High         │
│ Scale        │ Medium       │ Large        │ Very Large   │
│ Best for     │ Simple multi-│ Durable      │ Multi-cluster│
│              │ cluster      │ long-term    │ at scale     │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 8.4 When to Use Federation

| Use Case | Example |
|----------|---------|
| Multi-cluster overview | Aggregate key metrics from 5 K8s clusters |
| Team-level aggregation | Each team runs a Prometheus, central one federates |
| Cross-datacenter | Global view of services in US, EU, APAC |

⚠️ **Limitation:** Federation pulls data at scrape time — if the remote Prometheus is down, you miss that data window. For production at scale, consider Thanos or Grafana Mimir.

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Endpoint** | `/federate` on any Prometheus |
| **Use `match[]`** | Select only needed metrics |
| **`honor_labels`** | Keep original labels from source |
| **Alternative** | Thanos, Cortex, Mimir for full scale |

---

## Quick Revision Questions

1. **What is Prometheus federation and when would you use it?**
2. **Why should you use `match[]` to select only specific metrics for federation?**
3. **What does `honor_labels: true` do in a federation scrape config?**
4. **Compare federation vs Thanos. When is each more appropriate?**
5. **What are the limitations of hierarchical federation?**

---

[← Previous: Service Discovery](07-service-discovery.md) | [Back to README](../README.md) | [Next: Storage & Retention →](09-storage-retention.md)
