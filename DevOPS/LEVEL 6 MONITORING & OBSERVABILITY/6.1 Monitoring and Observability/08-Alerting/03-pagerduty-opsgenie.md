# Chapter 3: PagerDuty & OpsGenie

[← Previous: Alert Routing](02-alert-routing.md) | [Back to README](../README.md) | [Next: On-Call Management →](04-on-call-management.md)

---

## Overview

PagerDuty and OpsGenie are incident management platforms that handle on-call scheduling, alert escalation, and incident response coordination. They sit between alerting tools (Prometheus, Grafana) and the human responders.

---

## 3.1 Incident Management Platforms

```
┌──────────────────────────────────────────────────────────┐
│       INCIDENT MANAGEMENT PLATFORM FLOW                  │
│                                                          │
│  ┌───────────┐  ┌──────────┐  ┌──────────────┐         │
│  │Prometheus │  │ Grafana  │  │ CloudWatch   │         │
│  │Alertmgr   │  │ Alerting │  │ / Datadog    │         │
│  └─────┬─────┘  └─────┬────┘  └──────┬───────┘         │
│        └───────────────┼──────────────┘                  │
│                        ▼                                 │
│  ┌──────────────────────────────────────────────┐       │
│  │     PAGERDUTY / OPSGENIE                      │       │
│  │                                               │       │
│  │  ┌─────────────┐  ┌───────────────────────┐  │       │
│  │  │  Services   │  │  On-Call Schedules     │  │       │
│  │  │  (routing)  │  │  (who to notify)       │  │       │
│  │  └──────┬──────┘  └───────────┬───────────┘  │       │
│  │         │                     │               │       │
│  │         ▼                     ▼               │       │
│  │  ┌─────────────────────────────────────────┐ │       │
│  │  │  ESCALATION POLICY                       │ │       │
│  │  │  Level 1 → Primary on-call (5 min)      │ │       │
│  │  │  Level 2 → Secondary on-call (10 min)   │ │       │
│  │  │  Level 3 → Team lead (15 min)           │ │       │
│  │  │  Level 4 → Engineering manager (30 min) │ │       │
│  │  └──────────────┬──────────────────────────┘ │       │
│  │                 │                             │       │
│  │                 ▼                             │       │
│  │  ┌─────────────────────────────────────────┐ │       │
│  │  │  NOTIFICATION CHANNELS                   │ │       │
│  │  │  📱 Push notification                    │ │       │
│  │  │  📞 Phone call                           │ │       │
│  │  │  💬 SMS                                  │ │       │
│  │  │  📧 Email                                │ │       │
│  │  │  💻 Slack / Teams                        │ │       │
│  │  └─────────────────────────────────────────┘ │       │
│  └──────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 PagerDuty Setup

### Integration with Alertmanager

```yaml
# alertmanager.yml
receivers:
  - name: 'pagerduty-critical'
    pagerduty_configs:
      - routing_key: '<pagerduty-integration-key>'
        severity: '{{ if eq .GroupLabels.severity "critical" }}critical{{ else }}warning{{ end }}'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
        client: 'Prometheus Alertmanager'
        client_url: 'http://alertmanager.example.com'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          resolved: '{{ .Alerts.Resolved | len }}'
          cluster: '{{ .GroupLabels.cluster }}'
          service: '{{ .GroupLabels.service }}'
        links:
          - href: '{{ .CommonAnnotations.dashboard_url }}'
            text: 'Grafana Dashboard'
          - href: '{{ .CommonAnnotations.runbook_url }}'
            text: 'Runbook'
```

### PagerDuty Service Configuration

```
┌──────────────────────────────────────────────────────────┐
│  PAGERDUTY SERVICE HIERARCHY                             │
│                                                          │
│  Business Service: "E-Commerce Platform"                │
│  ├── Technical Service: "API Gateway"                   │
│  │   └── Integration: Prometheus Alertmanager           │
│  │       └── Escalation: Platform Team                  │
│  ├── Technical Service: "Payment Service"               │
│  │   └── Integration: Grafana Alerts                    │
│  │       └── Escalation: Payments Team                  │
│  ├── Technical Service: "Database"                      │
│  │   └── Integration: CloudWatch                        │
│  │       └── Escalation: DBA Team                       │
│  └── Technical Service: "Infrastructure"                │
│      └── Integration: Datadog                           │
│          └── Escalation: SRE Team                       │
│                                                          │
│  Each service has:                                       │
│  • Urgency rules (critical/warning mapping)             │
│  • Support hours                                        │
│  • Escalation policy                                    │
│  • Integrations (what sends alerts)                     │
└──────────────────────────────────────────────────────────┘
```

---

## 3.3 OpsGenie Setup

### Integration with Alertmanager

```yaml
# alertmanager.yml
receivers:
  - name: 'opsgenie-critical'
    opsgenie_configs:
      - api_key: '<opsgenie-api-key>'
        message: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'
        description: '{{ .CommonAnnotations.description }}'
        priority: '{{ if eq .GroupLabels.severity "critical" }}P1{{ else if eq .GroupLabels.severity "warning" }}P3{{ else }}P5{{ end }}'
        tags: 'monitoring,{{ .GroupLabels.service }}'
        responders:
          - name: 'platform-team'
            type: 'team'
        details:
          service: '{{ .GroupLabels.service }}'
          dashboard: '{{ .CommonAnnotations.dashboard_url }}'
          runbook: '{{ .CommonAnnotations.runbook_url }}'
```

### OpsGenie Alert Priority Mapping

| OpsGenie Priority | Alertmanager Severity | Notification |
|-------------------|----------------------|--------------|
| P1 (Critical) | critical | Phone, SMS, Push, Email |
| P2 (High) | high | Push, SMS, Email |
| P3 (Moderate) | warning | Push, Email |
| P4 (Low) | info | Email |
| P5 (Informational) | none | Email (optional) |

---

## 3.4 PagerDuty vs OpsGenie Comparison

| Feature | PagerDuty | OpsGenie |
|---------|-----------|----------|
| Pricing | Per user/month (higher) | Per user/month (lower) |
| Owned By | Independent | Atlassian (Jira/Confluence) |
| Jira Integration | Good | Native (same vendor) |
| Scheduling | Excellent | Very good |
| Event Intelligence | AI-powered grouping | Alert deduplication |
| Status Page | Built-in (StatusPage) | Included |
| Stakeholder Comms | Yes | Yes |
| Terraform Provider | Yes | Yes |
| Free Tier | Yes (limited) | Yes (5 users) |
| Best For | Large orgs, mature SRE | Atlassian shops, cost-sensitive |

---

## 3.5 Escalation Policies

```
┌──────────────────────────────────────────────────────────┐
│  ESCALATION POLICY EXAMPLE                               │
│                                                          │
│  t=0min   ──► Level 1: Primary On-Call                  │
│               • Push notification                        │
│               • SMS after 1 min                          │
│               • Phone call after 3 min                   │
│                                                          │
│  t=5min   ──► Level 2: Secondary On-Call                │
│  (if no       • Same notification sequence              │
│   ack)                                                   │
│                                                          │
│  t=15min  ──► Level 3: Team Lead                        │
│  (if no       • Phone call immediately                  │
│   ack)                                                   │
│                                                          │
│  t=30min  ──► Level 4: Engineering Manager              │
│  (if no       • Phone call immediately                  │
│   ack)        • Slack #incident-war-room                │
│                                                          │
│  t=60min  ──► Level 5: VP Engineering                   │
│  (if no       • Phone + SMS                             │
│   ack)        • All hands notification                  │
│                                                          │
│  Key: Each level = "if previous didn't acknowledge"     │
└──────────────────────────────────────────────────────────┘
```

---

## 3.6 PagerDuty Event API v2

```bash
# Trigger an incident via API
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -H "Content-Type: application/json" \
  -d '{
    "routing_key": "YOUR_INTEGRATION_KEY",
    "event_action": "trigger",
    "dedup_key": "api-gateway-high-error-rate",
    "payload": {
      "summary": "API Gateway error rate above 5%",
      "severity": "critical",
      "source": "prometheus",
      "component": "api-gateway",
      "group": "production",
      "class": "error-rate",
      "custom_details": {
        "current_value": "5.2%",
        "threshold": "1%",
        "dashboard": "https://grafana.example.com/d/api",
        "runbook": "https://wiki.example.com/runbooks/api-errors"
      }
    },
    "links": [
      {"href": "https://grafana.example.com/d/api", "text": "Dashboard"}
    ]
  }'

# Acknowledge the incident
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -H "Content-Type: application/json" \
  -d '{
    "routing_key": "YOUR_INTEGRATION_KEY",
    "event_action": "acknowledge",
    "dedup_key": "api-gateway-high-error-rate"
  }'

# Resolve the incident
curl -X POST https://events.pagerduty.com/v2/enqueue \
  -H "Content-Type: application/json" \
  -d '{
    "routing_key": "YOUR_INTEGRATION_KEY",
    "event_action": "resolve",
    "dedup_key": "api-gateway-high-error-rate"
  }'
```

---

## 3.7 Infrastructure as Code

### Terraform - PagerDuty

```hcl
provider "pagerduty" {
  token = var.pagerduty_token
}

resource "pagerduty_team" "platform" {
  name        = "Platform Team"
  description = "Platform engineering team"
}

resource "pagerduty_user" "oncall_engineer" {
  name  = "Keval Engineer"
  email = "keval@example.com"
  role  = "user"
}

resource "pagerduty_schedule" "platform_rotation" {
  name      = "Platform On-Call"
  time_zone = "America/New_York"

  layer {
    name                         = "Primary"
    start                        = "2024-01-01T00:00:00-05:00"
    rotation_virtual_start       = "2024-01-01T00:00:00-05:00"
    rotation_turn_length_seconds = 604800  # 1 week
    users                        = [pagerduty_user.oncall_engineer.id]
  }
}

resource "pagerduty_escalation_policy" "platform" {
  name      = "Platform Escalation"
  num_loops = 2

  rule {
    escalation_delay_in_minutes = 5
    target {
      type = "schedule_reference"
      id   = pagerduty_schedule.platform_rotation.id
    }
  }

  rule {
    escalation_delay_in_minutes = 15
    target {
      type = "user_reference"
      id   = pagerduty_user.oncall_engineer.id
    }
  }
}

resource "pagerduty_service" "api_gateway" {
  name              = "API Gateway"
  escalation_policy = pagerduty_escalation_policy.platform.id
  alert_creation    = "create_alerts_and_incidents"

  incident_urgency_rule {
    type    = "constant"
    urgency = "high"
  }
}

resource "pagerduty_service_integration" "prometheus" {
  name    = "Prometheus Alertmanager"
  service = pagerduty_service.api_gateway.id
  vendor  = data.pagerduty_vendor.prometheus.id
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert not reaching PagerDuty | Wrong routing_key | Verify integration key in service settings |
| Duplicate incidents | Missing dedup_key | Set dedup_key to group related alerts |
| No auto-resolve | Alertmanager not sending resolved | Set `send_resolved: true` |
| Escalation not working | Schedule gaps | Ensure 24/7 coverage in schedule |
| Wrong person paged | Schedule override missing | Add schedule overrides for swaps |

---

## Summary Table

| Concept | Details |
|---------|---------|
| PagerDuty | Enterprise incident management, AI grouping |
| OpsGenie | Atlassian-integrated, cost-effective |
| Escalation | Multi-level, time-based notification escalation |
| Integration | Alertmanager → PagerDuty/OpsGenie via config |
| Event API | trigger → acknowledge → resolve lifecycle |
| IaC | Terraform providers for both platforms |

---

## Quick Revision Questions

1. **What is the alert lifecycle in PagerDuty (trigger → acknowledge → resolve)?**
2. **How does an escalation policy work and why do you need multiple levels?**
3. **How do you integrate Prometheus Alertmanager with PagerDuty?**
4. **What is a dedup_key and why is it important for preventing duplicate incidents?**
5. **When would you choose PagerDuty over OpsGenie or vice versa?**

---

[← Previous: Alert Routing](02-alert-routing.md) | [Back to README](../README.md) | [Next: On-Call Management →](04-on-call-management.md)
