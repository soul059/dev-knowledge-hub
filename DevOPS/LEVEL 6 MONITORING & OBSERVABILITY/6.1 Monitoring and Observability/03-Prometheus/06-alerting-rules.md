# Chapter 6: Alerting Rules

[← Previous: Recording Rules](05-recording-rules.md) | [Back to README](../README.md) | [Next: Service Discovery →](07-service-discovery.md)

---

## Overview

Prometheus alerting rules define conditions under which alerts fire. When conditions are met, alerts are sent to Alertmanager for routing, grouping, and notification.

---

## 6.1 Alerting Flow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Prometheus  │───►│ Alertmanager │───►│ Notification │
│  Alert Rules │    │  Route/Group │    │   Channels   │
│              │    │  Silence     │    │              │
│ IF expr true │    │  Inhibit     │    │ • Slack      │
│ FOR duration │    │  Dedup       │    │ • PagerDuty  │
│ THEN fire    │    │              │    │ • Email      │
└──────────────┘    └──────────────┘    └──────────────┘
```

---

## 6.2 Alert Rule Configuration

```yaml
# alert_rules.yml
groups:
  - name: application_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          /
          sum(rate(http_requests_total[5m])) by (service)
          > 0.05
        for: 5m  # Must be true for 5 minutes before firing
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }} (>5%) for 5m"
          runbook_url: "https://wiki.example.com/runbooks/high-error-rate"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 1.0
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "P99 latency high on {{ $labels.service }}"
          description: "P99 latency is {{ $value }}s (>1s)"

  - name: infrastructure_alerts
    rules:
      # Instance down
      - alert: InstanceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} down"

      # Disk space prediction
      - alert: DiskWillFillIn4Hours
        expr: predict_linear(node_filesystem_avail_bytes[6h], 4*3600) < 0
        for: 30m
        labels:
          severity: warning
        annotations:
          summary: "Disk on {{ $labels.instance }} predicted full in 4h"

      # High memory usage
      - alert: HighMemoryUsage
        expr: |
          (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes)
          / node_memory_MemTotal_bytes > 0.90
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Memory usage >90% on {{ $labels.instance }}"
```

---

## 6.3 Alert States

```
┌────────────┐     expr becomes      ┌────────────┐    for duration    ┌────────────┐
│  INACTIVE  │────── true ──────────►│  PENDING   │────── elapsed ───►│   FIRING   │
│            │                        │            │                    │            │
│ (normal)   │◄────── false ─────────│(waiting)   │◄─── false ────────│(alerting)  │
└────────────┘                        └────────────┘                    └────────────┘

INACTIVE: Expression is false, no alert
PENDING:  Expression is true, waiting for `for` duration
FIRING:   Expression has been true for `for` duration → sent to Alertmanager
```

---

## 6.4 Alertmanager Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@example.com'

route:
  group_by: ['alertname', 'service']
  group_wait: 30s           # Wait before sending first notification
  group_interval: 5m        # Wait between notifications for same group
  repeat_interval: 4h       # Re-send if alert still firing
  receiver: 'default-slack'
  
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      group_wait: 10s
      
    - match:
        severity: warning
      receiver: 'slack-warnings'
      
    - match:
        team: database
      receiver: 'slack-database'

receivers:
  - name: 'default-slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: 'your-pagerduty-service-key'

  - name: 'slack-warnings'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#warnings'

  - name: 'slack-database'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
        channel: '#db-alerts'

# Silence specific alerts during maintenance
inhibit_rules:
  - source_match:
      alertname: 'InstanceDown'
    target_match:
      severity: 'warning'
    equal: ['instance']
    # If InstanceDown fires, suppress warnings for same instance
```

---

## 6.5 Best Practices

| Practice | Description |
|----------|-------------|
| Always use `for` | Avoid flapping — require condition to persist |
| Include `runbook_url` | Link to troubleshooting steps |
| Use `severity` labels | Route critical vs warning differently |
| Template annotations | Use `{{ $labels.x }}` and `{{ $value }}` |
| Validate before deploy | `promtool check rules alert_rules.yml` |

---

## Summary Table

| Component | Purpose |
|-----------|---------|
| **Alert Rule** | PromQL condition + `for` duration |
| **`for` clause** | Prevents flapping — wait before firing |
| **Labels** | Routing (severity, team) |
| **Annotations** | Human-readable description + runbook |
| **Alertmanager** | Route, group, silence, notify |
| **Inhibit rules** | Suppress downstream alerts |

---

## Quick Revision Questions

1. **Explain the three alert states: inactive, pending, and firing.**
2. **What is the purpose of the `for` clause in an alert rule?**
3. **Write an alert rule for when CPU usage exceeds 90% for 10 minutes.**
4. **How does Alertmanager's routing tree work?**
5. **What are inhibition rules and when would you use them?**

---

[← Previous: Recording Rules](05-recording-rules.md) | [Back to README](../README.md) | [Next: Service Discovery →](07-service-discovery.md)
