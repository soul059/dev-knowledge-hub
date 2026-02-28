# Chapter 7: Service Discovery

[← Previous: Alerting Rules](06-alerting-rules.md) | [Back to README](../README.md) | [Next: Federation →](08-federation.md)

---

## Overview

Service discovery automatically finds scrape targets instead of hardcoding them. Essential in dynamic environments like Kubernetes where pods are created and destroyed constantly.

---

## 7.1 Why Service Discovery?

```
Static Config (fragile):              Service Discovery (dynamic):
┌──────────────────────┐              ┌──────────────────────┐
│ static_configs:      │              │ kubernetes_sd_configs:│
│   - targets:         │              │   - role: pod        │
│     - app1:8080      │              │                      │
│     - app2:8080      │              │ Prometheus auto-     │
│     - app3:8080   ❌ │              │ discovers all pods ✅│
│   # What if app4     │              │                      │
│   # is deployed?     │              │ New pods found       │
│   # Manual update!   │              │ automatically!       │
└──────────────────────┘              └──────────────────────┘
```

---

## 7.2 Discovery Types

```
┌────────────────────┬──────────────────────────────────────┐
│ Type               │ Use Case                             │
├────────────────────┼──────────────────────────────────────┤
│ static_configs     │ Fixed, known targets                 │
│ file_sd_configs    │ Targets from JSON/YAML files         │
│ kubernetes_sd      │ Kubernetes pods/services/nodes       │
│ consul_sd          │ HashiCorp Consul service registry    │
│ ec2_sd             │ AWS EC2 instances                    │
│ azure_sd           │ Azure VMs                            │
│ gce_sd             │ Google Cloud instances               │
│ dns_sd             │ DNS SRV records                      │
│ docker_sd          │ Docker containers                    │
└────────────────────┴──────────────────────────────────────┘
```

---

## 7.3 Kubernetes Service Discovery

```yaml
scrape_configs:
  # Discover all pods with prometheus.io/scrape annotation
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with annotation prometheus.io/scrape=true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      
      # Use custom path from annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      
      # Use custom port from annotation  
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      
      # Add pod name, namespace as labels
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod
```

### Pod Annotations

```yaml
# Add these annotations to your pod/deployment for auto-discovery
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"    # Enable scraping
        prometheus.io/port: "8080"       # Metrics port
        prometheus.io/path: "/metrics"   # Metrics path
```

---

## 7.4 Relabeling

Relabeling transforms labels **before** scraping to control which targets to keep and what labels to apply:

```yaml
relabel_configs:
  # keep — only scrape matching targets
  - source_labels: [__meta_kubernetes_namespace]
    action: keep
    regex: production|staging

  # drop — exclude matching targets
  - source_labels: [__meta_kubernetes_pod_phase]
    action: drop
    regex: (Failed|Succeeded)

  # replace — rename/transform labels
  - source_labels: [__meta_kubernetes_pod_name]
    target_label: pod
    action: replace

  # labelmap — copy patterns of meta labels
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
```

---

## 7.5 File-Based Discovery

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'file-based'
    file_sd_configs:
      - files:
          - 'targets/*.json'
        refresh_interval: 30s

# targets/apps.json — auto-reloaded on change
[
  {
    "targets": ["app1:8080", "app2:8080"],
    "labels": {
      "environment": "production",
      "team": "backend"
    }
  }
]
```

---

## Summary Table

| Method | Environment | Auto-Update |
|--------|-------------|-------------|
| `static_configs` | Fixed infra | No (manual) |
| `file_sd_configs` | Any (file-based) | Yes (on file change) |
| `kubernetes_sd` | Kubernetes | Yes (watch API) |
| `consul_sd` | Consul-based | Yes (consul catalog) |
| `ec2_sd` | AWS | Yes (EC2 API) |

---

## Quick Revision Questions

1. **Why is service discovery necessary in Kubernetes environments?**
2. **What Kubernetes annotations enable automatic Prometheus scraping?**
3. **Explain the purpose of `relabel_configs` in service discovery.**
4. **What is the difference between `keep` and `drop` actions in relabeling?**
5. **When would you choose file-based service discovery over Kubernetes SD?**

---

[← Previous: Alerting Rules](06-alerting-rules.md) | [Back to README](../README.md) | [Next: Federation →](08-federation.md)
