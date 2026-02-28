# Chapter 6: Alerting in Grafana

[← Previous: Variables & Templating](05-variables-templating.md) | [Back to README](../README.md) | [Next: Dashboard Best Practices →](07-dashboard-best-practices.md)

---

## Overview

Grafana Alerting (unified alerting since v8+) provides a centralized system for defining alert rules, managing notification policies, and routing alerts to contact points (Slack, PagerDuty, email, etc.).

---

## 6.1 Alerting Architecture

```
┌──────────────────────────────────────────────────────────┐
│               GRAFANA ALERTING PIPELINE                  │
│                                                          │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │ Alert Rules   │───▶│ Evaluation   │                   │
│  │ (PromQL/SQL)  │    │ Engine       │                   │
│  └──────────────┘    └──────┬───────┘                   │
│                             │                            │
│                      ┌──────▼───────┐                   │
│                      │ Alert State   │                   │
│                      │ Normal/Pend/  │                   │
│                      │ Firing        │                   │
│                      └──────┬───────┘                   │
│                             │                            │
│                      ┌──────▼───────┐                   │
│                      │ Notification  │                   │
│                      │ Policies      │                   │
│                      │ (Routing)     │                   │
│                      └──────┬───────┘                   │
│                             │                            │
│              ┌──────────────┼──────────────┐            │
│              ▼              ▼              ▼            │
│       ┌──────────┐   ┌──────────┐   ┌──────────┐      │
│       │ Slack     │   │ PagerDuty│   │ Email    │      │
│       │ Channel   │   │          │   │          │      │
│       └──────────┘   └──────────┘   └──────────┘      │
│                                                          │
│       ┌──────────────────────────────────┐              │
│       │ Silences      Mute Timings       │              │
│       │ (Temporary)   (Recurring)        │              │
│       └──────────────────────────────────┘              │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Alert States

```
  Normal ────pending_period────▶ Pending ────▶ Firing
    ▲                                            │
    │                                            │
    └────────── condition no longer met ─────────┘

  States:
  ┌──────────┬────────────────────────────────────┐
  │ Normal   │ Condition is not met               │
  │ Pending  │ Condition met, waiting for duration │
  │ Firing   │ Condition met for full duration     │
  │ NoData   │ Query returned no results           │
  │ Error    │ Query evaluation failed             │
  └──────────┴────────────────────────────────────┘
```

---

## 6.3 Creating Alert Rules

### Grafana-Managed Alert

```yaml
# Alert Rule Configuration
Name: High Error Rate
Folder: Production Alerts
Group: API Alerts
Evaluation interval: 1m
Pending period: 5m         # Must be firing for 5m

# Condition
Query A:
  Type: Prometheus
  Expression: |
    sum(rate(http_requests_total{status=~"5.."}[5m]))
    / sum(rate(http_requests_total[5m])) * 100

Condition B:
  Type: Reduce
  Function: Last
  Input: A

Condition C:
  Type: Threshold
  Input: B
  Is above: 5              # Fire when error rate > 5%

# Labels (for routing)
Labels:
  severity: critical
  team: backend
  service: api-server

# Annotations (for notification content)
Annotations:
  summary: "High error rate on {{ $labels.service }}"
  description: |
    Error rate is {{ $values.B }}% (threshold: 5%)
    Service: {{ $labels.service }}
    Environment: {{ $labels.env }}
  runbook_url: "https://wiki.example.com/runbooks/high-error-rate"
  dashboard_url: "https://grafana.example.com/d/prod-overview"
```

---

## 6.4 Contact Points

```yaml
# Contact Point: Slack
Name: slack-backend-team
Type: Slack
Settings:
  url: https://hooks.slack.com/services/T00/B00/XXX
  channel: "#backend-alerts"
  username: Grafana Alert
  title: |
    {{ .Status | toUpper }} {{ .CommonLabels.alertname }}
  text: |
    **Severity:** {{ .CommonLabels.severity }}
    **Service:** {{ .CommonLabels.service }}
    {{ range .Alerts }}
    - {{ .Annotations.summary }}
    {{ end }}

# Contact Point: PagerDuty
Name: pagerduty-oncall
Type: PagerDuty
Settings:
  integrationKey: "abc123def456"
  severity: "{{ .CommonLabels.severity }}"
  summary: "{{ .CommonAnnotations.summary }}"

# Contact Point: Email
Name: email-ops
Type: Email
Settings:
  addresses: "ops-team@example.com;sre@example.com"
  singleEmail: true
```

---

## 6.5 Notification Policies (Routing)

```yaml
# Notification Policy Tree
Root Policy:
  Contact Point: email-ops          # Default destination
  Group by: [alertname, service]
  Group wait: 30s                   # Wait before first notification
  Group interval: 5m                # Wait between group updates
  Repeat interval: 4h               # Re-notify after this period

  Nested Policies:
    - Match:
        severity: critical
      Contact Point: pagerduty-oncall
      Continue: false               # Stop at first match

    - Match:
        severity: warning
        team: backend
      Contact Point: slack-backend-team
      Continue: false

    - Match:
        severity: warning
        team: frontend
      Contact Point: slack-frontend-team
```

```
  Routing Decision Tree:

  Alert arrives with labels:
  { severity="critical", team="backend", service="api" }
       │
       ▼
  Root: email-ops (default)
       │
       ├── severity=critical? YES → PagerDuty ✓ (STOP)
       ├── severity=warning, team=backend? NO
       └── severity=warning, team=frontend? NO
```

---

## 6.6 Silences & Mute Timings

### Silences (One-Time)
```
Purpose: Suppress alerts during known maintenance
Duration: Fixed start/end time
Example:  Silence all alerts for service=api 
          from 2024-01-15 02:00 to 04:00 UTC
Matchers: service=api, env=prod
```

### Mute Timings (Recurring)
```yaml
# Suppress alerts during weekends
Name: weekends
Time intervals:
  - weekdays: [saturday, sunday]
    times:
      - start_time: "00:00"
        end_time: "24:00"

# Business hours only
Name: outside-business-hours
Time intervals:
  - weekdays: [monday:friday]
    times:
      - start_time: "18:00"
        end_time: "09:00"
    location: America/New_York
```

---

## 6.7 Grafana vs Prometheus Alerting

| Feature | Grafana Alerting | Prometheus Alertmanager |
|---------|-----------------|----------------------|
| Query source | Any data source | Prometheus only |
| Configuration | UI + provisioning | YAML files |
| Multi-condition | Yes (reduce + threshold) | PromQL only |
| High availability | Grafana HA | Alertmanager cluster |
| Silences | UI-based | API/UI |
| Templates | Go templates | Go templates |
| Best for | Multi-source environments | Prometheus-only setups |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert never fires | Pending period too long or condition never true | Reduce pending period, test query in Explore |
| Duplicate notifications | Missing group_by or policies matching multiple | Check routing, add `continue: false` |
| No notification received | Contact point misconfigured | Test contact point with "Test" button |
| "NoData" alerts | Query returns empty result for some series | Configure NoData state handling |
| Alert flapping | Threshold too close to normal values | Add hysteresis or increase pending period |

---

## Quick Revision Questions

1. **What are the three main alert states in Grafana?**
2. **How does the notification policy routing tree work?**
3. **What is the difference between a silence and a mute timing?**
4. **How do you configure alert routing based on severity labels?**
5. **What are annotations in alert rules and how are they used?**
6. **Compare Grafana alerting vs Prometheus Alertmanager — when would you use each?**

---

[← Previous: Variables & Templating](05-variables-templating.md) | [Back to README](../README.md) | [Next: Dashboard Best Practices →](07-dashboard-best-practices.md)
