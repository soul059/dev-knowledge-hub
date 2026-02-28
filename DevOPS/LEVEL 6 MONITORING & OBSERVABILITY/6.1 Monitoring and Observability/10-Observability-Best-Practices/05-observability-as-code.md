# Chapter 5: Observability as Code

[← Previous: Security & Compliance](04-security-compliance.md) | [Back to README](../README.md) | [Next: Building SRE Culture →](06-building-sre-culture.md)

---

## Overview

Observability as Code (OaC) applies software engineering practices — version control, code review, CI/CD, and testing — to your observability configuration. Dashboards, alerts, recording rules, and data source configs are all managed as code, ensuring consistency, reproducibility, and auditability across environments.

---

## 5.1 Why Observability as Code

```
┌──────────────────────────────────────────────────────────┐
│  MANUAL vs AS-CODE                                       │
│                                                          │
│  MANUAL (ClickOps)              AS-CODE (GitOps)        │
│  ────────────────               ────────────────        │
│  Click to create dashboard      Dashboard in JSON/YAML  │
│  No change history              Git commit log           │
│  "Who changed the alert?"       PR review + approval    │
│  Differs per environment        Same across dev/prod    │
│  Disaster = start from scratch  Disaster = git apply    │
│  Hard to replicate              Template and reuse      │
│  Single editor, no review       Team collaboration      │
│                                                          │
│  ┌────────────────────────────────────────────────┐     │
│  │  GIT REPO (source of truth)                    │     │
│  │  ├── dashboards/                               │     │
│  │  │   ├── service-overview.jsonnet              │     │
│  │  │   └── slo-dashboard.jsonnet                 │     │
│  │  ├── alerts/                                   │     │
│  │  │   ├── slo-burn-rate.yml                     │     │
│  │  │   └── infrastructure.yml                    │     │
│  │  ├── recording-rules/                          │     │
│  │  │   └── aggregations.yml                      │     │
│  │  ├── datasources/                              │     │
│  │  │   └── prometheus.yml                        │     │
│  │  └── terraform/                                │     │
│  │      ├── pagerduty.tf                          │     │
│  │      └── grafana.tf                            │     │
│  └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Grafana Dashboard as Code

### Jsonnet (Grafonnet)

```jsonnet
// service-overview.jsonnet
local grafana = import 'grafonnet/grafana.libsonnet';
local dashboard = grafana.dashboard;
local prometheus = grafana.prometheus;
local graphPanel = grafana.graphPanel;
local row = grafana.row;
local template = grafana.template;

dashboard.new(
  title='Service Overview',
  schemaVersion=27,
  tags=['generated', 'service'],
  time_from='now-1h',
  editable=false,                    // Prevent manual edits
)
.addTemplate(
  template.datasource(
    name='datasource',
    query='prometheus',
    current='Prometheus',
  )
)
.addTemplate(
  template.new(
    name='service',
    datasource='$datasource',
    query='label_values(up, service)',
    refresh='time',
  )
)
.addRow(
  row.new(title='RED Metrics')
  .addPanel(
    graphPanel.new(
      title='Request Rate',
      datasource='$datasource',
    )
    .addTarget(
      prometheus.target(
        'sum(rate(http_requests_total{service="$service"}[5m]))',
        legendFormat='{{method}} {{status_code}}',
      )
    ),
    gridPos={ x: 0, y: 0, w: 8, h: 8 }
  )
  .addPanel(
    graphPanel.new(
      title='Error Rate',
      datasource='$datasource',
    )
    .addTarget(
      prometheus.target(
        'sum(rate(http_requests_total{service="$service",status_code=~"5.."}[5m])) / sum(rate(http_requests_total{service="$service"}[5m]))',
        legendFormat='Error Rate',
      )
    ),
    gridPos={ x: 8, y: 0, w: 8, h: 8 }
  )
)
```

### Build and Deploy

```bash
# Install Jsonnet and Grafonnet
go install github.com/google/go-jsonnet/cmd/jsonnet@latest
jb init
jb install github.com/grafana/grafonnet-lib/grafonnet

# Generate JSON from Jsonnet
jsonnet -J vendor service-overview.jsonnet > service-overview.json

# Deploy via Grafana API
curl -X POST http://grafana:3000/api/dashboards/db \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_API_KEY" \
  -d "{\"dashboard\": $(cat service-overview.json), \"overwrite\": true}"
```

### Grafana Provisioning (File-based)

```yaml
# /etc/grafana/provisioning/dashboards/default.yaml
apiVersion: 1
providers:
  - name: default
    orgId: 1
    folder: "Provisioned"
    type: file
    disableDeletion: true       # Prevent UI deletion
    editable: false             # Prevent UI edits
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true

# /etc/grafana/provisioning/datasources/prometheus.yaml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

---

## 5.3 Alerts as Code

### Prometheus Alert Rules

```yaml
# alerts/slo-burn-rate.yml
groups:
  - name: slo-burn-rate
    rules:
      - alert: SLOBudgetBurn_Fast
        expr: |
          (
            sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
            / sum(rate(http_requests_total[5m]))
          ) > (14.4 * 0.001)
        for: 2m
        labels:
          severity: critical
          team: "{{ $labels.service }}"
        annotations:
          summary: "Fast SLO budget burn for {{ $labels.service }}"
          dashboard: "https://grafana.company.com/d/slo/{{ $labels.service }}"
          runbook: "https://wiki.company.com/runbooks/slo-burn"

      - alert: SLOBudgetBurn_Slow
        expr: |
          (
            sum(rate(http_requests_total{status_code=~"5.."}[1h])) 
            / sum(rate(http_requests_total[1h]))
          ) > (1 * 0.001)
        for: 1h
        labels:
          severity: warning
          team: "{{ $labels.service }}"
        annotations:
          summary: "Slow SLO budget burn for {{ $labels.service }}"
```

### Alertmanager Config as Code

```yaml
# alertmanager/alertmanager.yml (in Git)
global:
  resolve_timeout: 5m

route:
  receiver: default-receiver
  group_by: ['alertname', 'service']
  routes:
    - match:
        severity: critical
      receiver: pagerduty-critical
    - match:
        severity: warning
      receiver: slack-warnings

receivers:
  - name: default-receiver
    slack_configs:
      - channel: '#alerts-default'
  - name: pagerduty-critical
    pagerduty_configs:
      - service_key: ${PD_SERVICE_KEY}
  - name: slack-warnings
    slack_configs:
      - channel: '#alerts-warnings'
```

---

## 5.4 Terraform for Observability

```hcl
# terraform/grafana.tf
terraform {
  required_providers {
    grafana = {
      source  = "grafana/grafana"
      version = "~> 2.0"
    }
  }
}

provider "grafana" {
  url  = "https://grafana.company.com"
  auth = var.grafana_api_key
}

# Create folder structure
resource "grafana_folder" "services" {
  title = "Service Dashboards"
}

resource "grafana_folder" "infra" {
  title = "Infrastructure"
}

# Deploy dashboard from JSON
resource "grafana_dashboard" "service_overview" {
  folder      = grafana_folder.services.id
  config_json = file("${path.module}/dashboards/service-overview.json")
  overwrite   = true
}

# Alert notification policy
resource "grafana_notification_policy" "main" {
  group_by      = ["alertname", "service"]
  contact_point = grafana_contact_point.slack.name

  policy {
    matcher {
      label = "severity"
      match = "="
      value = "critical"
    }
    contact_point = grafana_contact_point.pagerduty.name
    group_wait    = "30s"
    group_interval = "5m"
  }
}

resource "grafana_contact_point" "slack" {
  name = "slack-default"
  slack {
    url     = var.slack_webhook_url
    channel = "#alerts"
  }
}

resource "grafana_contact_point" "pagerduty" {
  name = "pagerduty-critical"
  pagerduty {
    integration_key = var.pd_integration_key
    severity        = "critical"
  }
}
```

```hcl
# terraform/pagerduty.tf
provider "pagerduty" {
  token = var.pagerduty_token
}

resource "pagerduty_service" "api" {
  name              = "API Service"
  escalation_policy = pagerduty_escalation_policy.default.id
  alert_creation    = "create_alerts_and_incidents"

  incident_urgency_rule {
    type    = "constant"
    urgency = "high"
  }
}

resource "pagerduty_schedule" "primary" {
  name      = "Primary On-Call"
  time_zone = "America/New_York"

  layer {
    name                         = "Weekly Rotation"
    start                        = "2024-01-01T08:00:00-05:00"
    rotation_virtual_start       = "2024-01-01T08:00:00-05:00"
    rotation_turn_length_seconds = 604800  # 1 week

    users = [
      pagerduty_user.alice.id,
      pagerduty_user.bob.id,
      pagerduty_user.charlie.id,
    ]
  }
}
```

---

## 5.5 CI/CD Pipeline for Observability

```
┌──────────────────────────────────────────────────────────┐
│  OBSERVABILITY CI/CD PIPELINE                            │
│                                                          │
│  Developer                                               │
│     │                                                    │
│     ▼                                                    │
│  ┌────────────┐                                          │
│  │ Git Push   │  (branch: feature/add-slo-dashboard)    │
│  └─────┬──────┘                                          │
│        ▼                                                 │
│  ┌────────────────────────────────────────────────┐     │
│  │ CI: VALIDATE                                   │     │
│  │ • Lint Jsonnet syntax                          │     │
│  │ • Validate Prometheus rules (promtool)         │     │
│  │ • Validate Alertmanager config (amtool)        │     │
│  │ • Terraform plan (dry-run)                     │     │
│  │ • Unit test alert expressions                  │     │
│  └─────────────────────┬──────────────────────────┘     │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐     │
│  │ PR Review                                      │     │
│  │ • Team reviews dashboard/alert changes         │     │
│  │ • Preview dashboard renders (optional)         │     │
│  └─────────────────────┬──────────────────────────┘     │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐     │
│  │ CD: DEPLOY                                     │     │
│  │ • Merge to main                                │     │
│  │ • Deploy to staging → verify → deploy to prod │     │
│  │ • Terraform apply                              │     │
│  │ • Grafana API sync                             │     │
│  └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

### GitHub Actions Example

```yaml
# .github/workflows/observability.yml
name: Observability CI/CD

on:
  push:
    paths: ['observability/**']
  pull_request:
    paths: ['observability/**']

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Validate Prometheus rules
      - name: Install promtool
        run: |
          wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz
          tar xzf prometheus-*.tar.gz
          sudo mv prometheus-*/promtool /usr/local/bin/

      - name: Validate alert rules
        run: promtool check rules observability/alerts/*.yml

      - name: Validate recording rules
        run: promtool check rules observability/recording-rules/*.yml

      # Validate Alertmanager config
      - name: Install amtool
        run: |
          wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
          tar xzf alertmanager-*.tar.gz
          sudo mv alertmanager-*/amtool /usr/local/bin/

      - name: Validate Alertmanager config
        run: amtool check-config observability/alertmanager/alertmanager.yml

      # Unit test alert rules
      - name: Test alert rules
        run: promtool test rules observability/tests/*.yml

      # Terraform validation
      - name: Terraform validate
        run: |
          cd observability/terraform
          terraform init
          terraform validate
          terraform plan -out=plan.out

  deploy:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy dashboards
        run: |
          for f in observability/dashboards/*.json; do
            curl -X POST $GRAFANA_URL/api/dashboards/db \
              -H "Authorization: Bearer ${{ secrets.GRAFANA_TOKEN }}" \
              -H "Content-Type: application/json" \
              -d "{\"dashboard\": $(cat $f), \"overwrite\": true}"
          done

      - name: Apply Terraform
        run: |
          cd observability/terraform
          terraform init
          terraform apply -auto-approve
```

### Alert Rule Unit Tests

```yaml
# observability/tests/slo-rules-test.yml
rule_files:
  - ../alerts/slo-burn-rate.yml

evaluation_interval: 1m

tests:
  - interval: 1m
    input_series:
      - series: 'http_requests_total{service="api",status_code="200"}'
        values: '0+100x60'        # 100 req/min, all success
      - series: 'http_requests_total{service="api",status_code="500"}'
        values: '0+0x60'          # No errors

    alert_rule_test:
      - eval_time: 5m
        alertname: SLOBudgetBurn_Fast
        exp_alerts: []            # No alert expected

  - interval: 1m
    input_series:
      - series: 'http_requests_total{service="api",status_code="200"}'
        values: '0+90x60'         # 90 req/min success
      - series: 'http_requests_total{service="api",status_code="500"}'
        values: '0+10x60'         # 10 req/min errors = 10% error rate

    alert_rule_test:
      - eval_time: 5m
        alertname: SLOBudgetBurn_Fast
        exp_alerts:
          - exp_labels:
              severity: critical
              service: api
```

---

## 5.6 Drift Detection

```bash
#!/bin/bash
# detect-drift.sh — Check if live dashboards differ from Git

GRAFANA_URL="https://grafana.company.com"
DASHBOARD_DIR="./dashboards"

for file in "$DASHBOARD_DIR"/*.json; do
  uid=$(jq -r '.uid' "$file")
  
  # Fetch live dashboard
  live=$(curl -s -H "Authorization: Bearer $TOKEN" \
    "$GRAFANA_URL/api/dashboards/uid/$uid" | jq '.dashboard')
  
  # Compare (ignore version, id fields)
  stored=$(jq 'del(.version, .id)' "$file")
  live_clean=$(echo "$live" | jq 'del(.version, .id)')
  
  if ! diff <(echo "$stored") <(echo "$live_clean") > /dev/null; then
    echo "DRIFT DETECTED: $uid ($file)"
    echo "Someone edited dashboard in UI. Please update Git or revert."
  fi
done
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Dashboard edits lost | Provisioned dashboards overwrite UI changes | Set editable=false, train teams on Git workflow |
| Alert rule syntax error | No CI validation | Add promtool check rules to CI pipeline |
| Config differs between envs | Manual per-env config | Use Terraform workspaces or Jsonnet environments |
| Drift from git | Manual UI edits | Run drift detection, use read-only provisioned dashboards |
| Complex dashboard templating | Raw JSON is hard to maintain | Use Jsonnet/Grafonnet for dashboard generation |

---

## Summary Table

| Aspect | Tool / Approach |
|--------|----------------|
| Dashboards | Jsonnet/Grafonnet → JSON → Grafana API/provisioning |
| Alert Rules | YAML in Git → promtool validate → deploy via ConfigMap |
| Alertmanager | YAML in Git → amtool validate → deploy |
| Infrastructure | Terraform (Grafana, PagerDuty, Opsgenie providers) |
| CI/CD | GitHub Actions / GitLab CI — lint, test, plan, apply |
| Drift Detection | Compare live state vs Git, block UI edits |

---

## Quick Revision Questions

1. **What is Observability as Code and why does it matter?**
2. **How do you prevent manual UI edits from overriding provisioned dashboards?**
3. **How do you unit test Prometheus alert rules before deploying?**
4. **What is the role of Terraform in managing observability infrastructure?**
5. **Describe a CI/CD pipeline for deploying alert rules and dashboards.**
6. **What is drift detection and how do you implement it?**

---

[← Previous: Security & Compliance](04-security-compliance.md) | [Back to README](../README.md) | [Next: Building SRE Culture →](06-building-sre-culture.md)
