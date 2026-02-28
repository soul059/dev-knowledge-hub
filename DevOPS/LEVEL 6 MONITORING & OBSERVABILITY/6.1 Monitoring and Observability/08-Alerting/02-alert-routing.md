# Chapter 2: Alert Routing

[← Previous: Alerting Strategies](01-alerting-strategies.md) | [Back to README](../README.md) | [Next: PagerDuty & OpsGenie →](03-pagerduty-opsgenie.md)

---

## Overview

Alert routing determines which alerts reach which people and through which channels. Effective routing ensures critical alerts wake the right on-call engineer while low-priority alerts go to the appropriate queue. This chapter covers Prometheus Alertmanager routing in depth.

---

## 2.1 Alertmanager Architecture

```
┌──────────────────────────────────────────────────────────┐
│              ALERTMANAGER PIPELINE                       │
│                                                          │
│  Prometheus ──► Alertmanager                            │
│                     │                                    │
│                     ▼                                    │
│              ┌──────────────┐                           │
│              │   ROUTING    │  Match alerts to receivers │
│              │   TREE       │  based on labels          │
│              └──────┬───────┘                           │
│                     │                                    │
│              ┌──────┼──────────┬────────────┐           │
│              ▼      ▼          ▼            ▼           │
│           ┌─────┐┌──────┐ ┌────────┐ ┌──────────┐     │
│  Group    │Grp 1││Grp 2 │ │ Grp 3  │ │ Grp 4    │     │
│           └──┬──┘└──┬───┘ └───┬────┘ └────┬─────┘     │
│              │      │         │            │            │
│              ▼      ▼         ▼            ▼            │
│           Inhibit? Inhibit? Inhibit?   Inhibit?        │
│           Silence? Silence? Silence?   Silence?        │
│              │      │         │            │            │
│              ▼      ▼         ▼            ▼            │
│           ┌─────┐┌──────┐ ┌────────┐ ┌──────────┐     │
│  Notify   │Slack││PD    │ │ Email  │ │ Webhook  │     │
│           └─────┘└──────┘ └────────┘ └──────────┘     │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 Alertmanager Configuration

```yaml
# alertmanager.yml - Complete production example
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

# ─── TEMPLATES ───
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# ─── INHIBITION RULES ───
# Suppress child alerts when parent fires
inhibit_rules:
  # If cluster is down, don't alert on individual services
  - source_matchers:
      - alertname = "ClusterDown"
    target_matchers:
      - severity =~ "warning|critical"
    equal: ['cluster']

  # If a critical fires, suppress warnings for same alertname
  - source_matchers:
      - severity = "critical"
    target_matchers:
      - severity = "warning"
    equal: ['alertname', 'service']

# ─── ROUTING TREE ───
route:
  # Default receiver
  receiver: 'slack-default'

  # How long to wait before sending first notification
  group_wait: 30s

  # How long to wait before sending updates
  group_interval: 5m

  # How long to wait before re-notifying
  repeat_interval: 4h

  # Group alerts by these labels
  group_by: ['alertname', 'cluster', 'service']

  # Child routes (evaluated top-to-bottom, first match wins)
  routes:
    # Critical alerts → PagerDuty immediately
    - matchers:
        - severity = "critical"
      receiver: 'pagerduty-critical'
      group_wait: 10s
      repeat_interval: 1h
      continue: false

    # Warning alerts → Slack channel
    - matchers:
        - severity = "warning"
      receiver: 'slack-warnings'
      group_wait: 1m
      repeat_interval: 4h

    # Platform team alerts
    - matchers:
        - team = "platform"
      receiver: 'slack-platform'
      routes:
        # Platform critical → PagerDuty
        - matchers:
            - severity = "critical"
          receiver: 'pagerduty-platform'

    # Database alerts → DBA team
    - matchers:
        - component = "database"
      receiver: 'slack-dba'

    # Watchdog (dead-man's switch) → separate channel
    - matchers:
        - alertname = "Watchdog"
      receiver: 'watchdog'
      repeat_interval: 1m

# ─── RECEIVERS ───
receivers:
  - name: 'slack-default'
    slack_configs:
      - channel: '#alerts-default'
        send_resolved: true
        title: '{{ .GroupLabels.alertname }}'
        text: >-
          {{ range .Alerts }}
          *{{ .Labels.severity | toUpper }}* - {{ .Annotations.summary }}
          {{ .Annotations.description }}
          {{ end }}

  - name: 'slack-warnings'
    slack_configs:
      - channel: '#alerts-warnings'
        send_resolved: true

  - name: 'slack-platform'
    slack_configs:
      - channel: '#platform-alerts'
        send_resolved: true

  - name: 'slack-dba'
    slack_configs:
      - channel: '#dba-alerts'
        send_resolved: true

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<pagerduty-integration-key>'
        severity: 'critical'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          service: '{{ .GroupLabels.service }}'

  - name: 'pagerduty-platform'
    pagerduty_configs:
      - service_key: '<platform-pagerduty-key>'
        severity: 'critical'

  - name: 'watchdog'
    webhook_configs:
      - url: 'http://deadmansswitch.example.com/ping'
```

---

## 2.3 Routing Tree Visualization

```
┌──────────────────────────────────────────────────────────┐
│  ROUTING TREE - DECISION FLOW                            │
│                                                          │
│  Alert arrives with labels:                             │
│  { alertname="APIErrors", severity="critical",          │
│    team="platform", service="api-gateway" }             │
│                                                          │
│  route (root):                                           │
│  ├─ severity="critical"?                                │
│  │   YES → pagerduty-critical ✓ (MATCH - stop)         │
│  │                                                       │
│  ├─ severity="warning"?                                 │
│  │   (not evaluated - previous matched)                 │
│  │                                                       │
│  ├─ team="platform"?                                    │
│  │   (not evaluated)                                    │
│  │                                                       │
│  └─ component="database"?                               │
│      (not evaluated)                                    │
│                                                          │
│  With continue: true on critical route:                  │
│  ├─ severity="critical"? YES → pagerduty-critical ✓    │
│  │   continue: true → keep evaluating                   │
│  ├─ team="platform"? YES → slack-platform ✓            │
│  │   └─ severity="critical"? YES → pagerduty-platform ✓│
│  │                                                       │
│  Result: Alert sent to pagerduty AND slack              │
└──────────────────────────────────────────────────────────┘
```

---

## 2.4 Grouping, Inhibition, and Silences

### Grouping

```yaml
# Group all alerts from same service together
route:
  group_by: ['alertname', 'service']
  group_wait: 30s       # Wait 30s to collect related alerts
  group_interval: 5m    # Wait 5m before adding new alerts to group
```

```
Without grouping:                  With grouping:
┌────────────────┐                ┌──────────────────────┐
│ Alert: pod-1   │                │ 3 alerts for api-gw  │
│ Alert: pod-2   │  ──────►      │ • pod-1 down         │
│ Alert: pod-3   │                │ • pod-2 down         │
│ (3 messages)   │                │ • pod-3 down         │
└────────────────┘                │ (1 message)          │
                                  └──────────────────────┘
```

### Inhibition Rules

```
┌──────────────────────────────────────────────────────────┐
│  INHIBITION: Suppress dependent alerts                   │
│                                                          │
│  Cluster Down (source)                                  │
│       │                                                  │
│       ▼  inhibits                                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐               │
│  │ Service A│ │ Service B│ │ Service C│  (suppressed)  │
│  │ down     │ │ errors   │ │ timeout  │               │
│  └──────────┘ └──────────┘ └──────────┘               │
│                                                          │
│  Result: 1 alert (ClusterDown) instead of 4 alerts     │
└──────────────────────────────────────────────────────────┘
```

### Silences

```bash
# Create silence via amtool CLI
amtool silence add \
  alertname="DiskSpaceWarning" \
  instance="web-01" \
  --author="keval" \
  --comment="Scheduled disk expansion at 2pm" \
  --duration="2h"

# List active silences
amtool silence query

# Expire a silence early
amtool silence expire <silence-id>
```

---

## 2.5 Alert Templates

```go
{{ /* /etc/alertmanager/templates/slack.tmpl */ }}

{{ define "slack.custom.title" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.alertname }}
{{ end }}

{{ define "slack.custom.text" }}
{{ range .Alerts }}
*Alert:* {{ .Labels.alertname }}
*Severity:* {{ .Labels.severity }}
*Service:* {{ .Labels.service }}
*Summary:* {{ .Annotations.summary }}
*Description:* {{ .Annotations.description }}
*Runbook:* {{ .Annotations.runbook_url }}
*Dashboard:* {{ .Annotations.dashboard_url }}
{{ if .Labels.instance }}*Instance:* {{ .Labels.instance }}{{ end }}
{{ end }}
{{ end }}
```

---

## 2.6 Testing Alert Routing

```bash
# Validate alertmanager config
amtool check-config alertmanager.yml

# Test which receiver an alert would match
amtool config routes test \
  --config.file=alertmanager.yml \
  severity=critical team=platform service=api-gateway

# Show routing tree
amtool config routes show --config.file=alertmanager.yml

# Send a test alert manually
curl -X POST http://alertmanager:9093/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '[{
    "labels": {
      "alertname": "TestAlert",
      "severity": "warning",
      "service": "test-service"
    },
    "annotations": {
      "summary": "This is a test alert"
    }
  }]'
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert goes to wrong receiver | Route order wrong | Routes match top-to-bottom; reorder |
| Alert not grouped | Wrong group_by labels | Add correct labels to group_by |
| Too many notifications | repeat_interval too short | Increase repeat_interval |
| Inhibition not working | `equal` labels don't match | Verify source and target share labels |
| Silence not applying | Labels don't match exactly | Check label matchers match alert labels |

---

## Summary Table

| Concept | Details |
|---------|---------|
| Routing Tree | Label-based matching, top-to-bottom evaluation |
| Grouping | Combine related alerts into one notification |
| Inhibition | Suppress child alerts when parent fires |
| Silences | Temporary muting during maintenance |
| Receivers | Slack, PagerDuty, Email, Webhook, OpsGenie |
| Templates | Go templating for notification formatting |
| `continue` | When true, keeps evaluating after a match |

---

## Quick Revision Questions

1. **How does the Alertmanager routing tree evaluate which receiver to use?**
2. **What is the difference between `group_wait`, `group_interval`, and `repeat_interval`?**
3. **How do inhibition rules prevent alert storms?**
4. **When would you use `continue: true` in a route?**
5. **How do you test which receiver an alert will match without actually sending it?**

---

[← Previous: Alerting Strategies](01-alerting-strategies.md) | [Back to README](../README.md) | [Next: PagerDuty & OpsGenie →](03-pagerduty-opsgenie.md)
