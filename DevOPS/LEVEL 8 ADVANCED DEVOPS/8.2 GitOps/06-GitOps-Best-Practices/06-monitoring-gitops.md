# Chapter 6.6: Monitoring GitOps

[← Previous: Security Considerations](05-security-considerations.md) | [Back to README](../README.md)

---

## Overview

Monitoring your GitOps pipeline is critical. Git is your desired state, the cluster is your actual state, and the GitOps operator is the bridge. You need observability into all three to detect drift, failed syncs, and operational issues before they impact users.

---

## What to Monitor

```
┌──────────────────────────────────────────────────────────────┐
│            GITOPS OBSERVABILITY MODEL                        │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐   │
│  │     Git      │    │   Operator   │    │   Cluster    │   │
│  │  (Desired    │───▶│  (ArgoCD /   │───▶│  (Actual     │   │
│  │   State)     │    │   Flux)      │    │   State)     │   │
│  └──────────────┘    └──────────────┘    └──────────────┘   │
│        │                    │                    │           │
│        ▼                    ▼                    ▼           │
│  Webhook events      Sync status          Pod health        │
│  PR merge times      Error logs           Resource usage    │
│  Commit frequency    Reconcile duration   Drift events      │
│  DORA metrics        Queue depth          Rollout status    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## ArgoCD Metrics

ArgoCD exposes Prometheus metrics on each component.

### Key Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `argocd_app_info` | Application sync/health status | health != Healthy |
| `argocd_app_sync_total` | Total sync operations | Spike = issue |
| `argocd_app_reconcile_count` | Reconciliation events | Unusually high = thrashing |
| `argocd_app_reconcile_bucket` | Reconciliation duration | p99 > 30s |
| `argocd_cluster_api_resource_objects` | Resource count per cluster | Capacity planning |
| `argocd_git_request_total` | Git requests (success/fail) | Failures > 0 |
| `argocd_repo_pending_request_total` | Queued repo requests | > 10 = bottleneck |

### ServiceMonitor for ArgoCD

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
  - port: metrics
    interval: 30s
```

### Server Metrics Endpoint

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics
  endpoints:
  - port: metrics
    interval: 30s
```

---

## Flux Metrics

Flux controllers expose metrics via the `/metrics` endpoint.

### Key Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `gotk_reconcile_condition` | Reconciliation result (Ready/Stalled) | condition != Ready |
| `gotk_reconcile_duration_seconds` | Time to reconcile | p99 > 60s |
| `gotk_suspend_status` | Whether a resource is suspended | 1 = suspended |
| `controller_runtime_reconcile_total` | Total reconcile attempts | High error ratio |
| `controller_runtime_reconcile_errors_total` | Failed reconciles | > 0 sustained |

### ServiceMonitor for Flux

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: flux-system
  namespace: flux-system
spec:
  selector:
    matchLabels:
      app: source-controller
  endpoints:
  - port: http-prom
    interval: 30s
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: flux-kustomize
  namespace: flux-system
spec:
  selector:
    matchLabels:
      app: kustomize-controller
  endpoints:
  - port: http-prom
    interval: 30s
```

---

## Alerting Rules

### ArgoCD Alerts

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: argocd-alerts
  namespace: argocd
spec:
  groups:
  - name: argocd
    rules:
    # App out of sync for more than 15 minutes
    - alert: ArgoCDAppOutOfSync
      expr: |
        argocd_app_info{sync_status="OutOfSync"} == 1
      for: 15m
      labels:
        severity: warning
      annotations:
        summary: "{{ $labels.name }} is out of sync"
        description: "Application {{ $labels.name }} has been OutOfSync for 15+ minutes"

    # App unhealthy
    - alert: ArgoCDAppUnhealthy
      expr: |
        argocd_app_info{health_status!~"Healthy|Progressing"} == 1
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.name }} is unhealthy"

    # Sync failures
    - alert: ArgoCDSyncFailed
      expr: |
        increase(argocd_app_sync_total{phase="Failed"}[10m]) > 0
      labels:
        severity: critical
      annotations:
        summary: "Sync failed for {{ $labels.name }}"

    # Git request failures
    - alert: ArgoCDGitRequestFailure
      expr: |
        increase(argocd_git_request_total{request_type="fetch",result="error"}[10m]) > 3
      labels:
        severity: warning
      annotations:
        summary: "Git fetch errors detected"
```

### Flux Alerts

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: flux-alerts
  namespace: flux-system
spec:
  groups:
  - name: flux
    rules:
    # Reconciliation failure
    - alert: FluxReconciliationFailure
      expr: |
        gotk_reconcile_condition{type="Ready",status="False"} == 1
      for: 15m
      labels:
        severity: critical
      annotations:
        summary: "{{ $labels.kind }}/{{ $labels.name }} failing to reconcile"

    # Suspended resources (forgotten?)
    - alert: FluxResourceSuspended
      expr: |
        gotk_suspend_status == 1
      for: 24h
      labels:
        severity: warning
      annotations:
        summary: "{{ $labels.kind }}/{{ $labels.name }} suspended for 24h+"

    # Slow reconciliation
    - alert: FluxSlowReconciliation
      expr: |
        histogram_quantile(0.99,
          rate(gotk_reconcile_duration_seconds_bucket[5m])
        ) > 60
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "Reconciliation taking >60s for {{ $labels.kind }}"
```

---

## Flux Native Notifications

Flux has a built-in Notification Controller for alerting without Prometheus.

```yaml
# Provider: Slack
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  type: slack
  channel: gitops-alerts
  secretRef:
    name: slack-webhook-url
---
# Alert: notify on failures
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: on-failure
  namespace: flux-system
spec:
  providerRef:
    name: slack
  eventSeverity: error
  eventSources:
  - kind: Kustomization
    name: "*"
  - kind: HelmRelease
    name: "*"
  - kind: GitRepository
    name: "*"
```

---

## Grafana Dashboards

### ArgoCD Dashboard

Import the official ArgoCD Grafana dashboard:

```
Dashboard ID: 14584   (ArgoCD Overview)
Dashboard ID: 19993   (ArgoCD Operational)
```

### Key Panels to Include

```
┌────────────────────────────────────────────────────────┐
│                 GITOPS DASHBOARD                       │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Sync Status │  │  Health      │  │  Sync Errors │ │
│  │  ██████ 95%  │  │  ██████ 98%  │  │  ██  2       │ │
│  │  Synced      │  │  Healthy     │  │  Failed      │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                        │
│  ┌──────────────────────────────────┐  ┌────────────┐ │
│  │  Reconciliation Duration (p99)   │  │  Git Fetch │ │
│  │  ▁▂▃▄▅▃▂▁▂▃▄▅▃▂▁               │  │  Latency   │ │
│  │  12s avg | 28s p99               │  │  1.2s avg  │ │
│  └──────────────────────────────────┘  └────────────┘ │
│                                                        │
│  ┌────────────────────────────────────────────────────┐│
│  │  Application List (table)                         ││
│  │  Name     | Sync   | Health  | Last Sync          ││
│  │  frontend | Synced | Healthy | 2 min ago          ││
│  │  backend  | OOS    | Degraded| 15 min ago    ⚠    ││
│  │  db       | Synced | Healthy | 5 min ago          ││
│  └────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────┘
```

---

## DORA Metrics Integration

Track deployment performance with DORA metrics derived from GitOps data.

| DORA Metric | GitOps Data Source |
|-------------|-------------------|
| Deployment Frequency | Git commit/merge frequency to main |
| Lead Time for Changes | Time from commit to sync completion |
| Change Failure Rate | Failed syncs / total syncs |
| Mean Time to Recovery | Time from failure detection to successful resync |

### Collecting Lead Time

```yaml
# Prometheus recording rule
- record: gitops:lead_time_seconds
  expr: |
    argocd_app_sync_total{phase="Succeeded"}
    - on(name) 
    argocd_git_request_total{request_type="fetch"}
```

> **Tip:** Tools like **Sleuth**, **Faros AI**, and **Flux's notification events** can calculate DORA metrics from GitOps activity automatically.

---

## Troubleshooting Checklist

| Symptom | Check |
|---------|-------|
| App stuck OutOfSync | `argocd app diff <app>` / check repo access |
| Reconciliation slow | Dashboard p99 latency / reduce manifest count |
| Git fetch errors | Network policies / SSH key expiry / rate limits |
| Flux not reconciling | `flux get all` / check controller logs |
| Alerts not firing | ServiceMonitor labels match Prometheus selector |
| Dashboard empty | Verify scrape targets in Prometheus UI |

---

## Summary Table

| Area | Tool | Purpose |
|------|------|---------|
| Metrics collection | Prometheus + ServiceMonitor | Scrape ArgoCD/Flux metrics |
| Visualization | Grafana dashboards | Sync status, health, latency |
| Alerting | PrometheusRule / Flux Alerts | Notify on drift, failures |
| Chat notifications | Flux Notification Controller | Slack/Teams/webhook alerts |
| Deployment tracking | DORA metrics | Frequency, lead time, failure rate |
| Log analysis | Loki / ELK | Controller error investigation |

---

## Quick Revision Questions

1. **What are the three things you need to monitor in a GitOps pipeline?**
   > Git (desired state), the GitOps operator (sync status), and the cluster (actual state). Each has different metrics and failure modes.

2. **Name three critical ArgoCD metrics.**
   > `argocd_app_info` (sync/health status), `argocd_app_sync_total` (sync results), `argocd_git_request_total` (Git fetch success/failure).

3. **How does Flux provide alerting without Prometheus?**
   > Flux has a built-in Notification Controller that sends events to Slack, Teams, webhooks, etc. using Provider and Alert CRDs.

4. **What DORA metrics can you derive from GitOps data?**
   > Deployment Frequency (merge rate), Lead Time (commit-to-sync), Change Failure Rate (failed syncs / total), MTTR (time to fix failed sync).

5. **What should you check when a Grafana dashboard shows no ArgoCD data?**
   > Verify ServiceMonitor labels match the Prometheus operator's `serviceMonitorSelector`. Check that Prometheus is scraping the ArgoCD metrics endpoints in the Prometheus targets UI.

6. **Why is reconciliation duration an important metric?**
   > Slow reconciliation means longer drift windows — the time between a Git change and the cluster reflecting it. High p99 values indicate performance issues that delay deployments.

---

[← Previous: Security Considerations](05-security-considerations.md) | [Back to README](../README.md)
