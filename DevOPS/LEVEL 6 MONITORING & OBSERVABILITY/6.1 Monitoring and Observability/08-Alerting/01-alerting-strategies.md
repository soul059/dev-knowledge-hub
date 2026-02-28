# Chapter 1: Alerting Strategies

[← Previous: Trace Analysis](../07-Distributed-Tracing/06-trace-analysis-optimization.md) | [Back to README](../README.md) | [Next: Alert Routing →](02-alert-routing.md)

---

## Overview

Alerting is the bridge between observability data and human action. Effective alerting notifies the right people about real problems while minimizing noise. This chapter covers alerting philosophies, strategies, and design principles.

---

## 1.1 Why Alerting Matters

```
┌──────────────────────────────────────────────────────────┐
│          THE ALERTING SPECTRUM                           │
│                                                          │
│  Too Few Alerts          Just Right         Too Many     │
│  ◄──────────────────────────┼──────────────────────────► │
│                             │                            │
│  • Outages go              • Every alert is             • Alert fatigue  │
│    unnoticed                 actionable                 • Noise          │
│  • Slow response           • Right person               • Ignored alerts │
│  • Customer impact           notified                   • Burnout        │
│    before detection        • Timely detection           • Real alerts    │
│                            • Minimal noise                missed         │
│                                                          │
│  Goal: Maximize signal, minimize noise                  │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Alerting Philosophy

### Symptoms vs Causes

```
┌──────────────────────────────────────────────────────────┐
│  ALERT ON SYMPTOMS, NOT CAUSES                           │
│                                                          │
│  ✓ SYMPTOM-BASED (Alert on these):                      │
│  • "Error rate > 1% for 5 minutes"                      │
│  • "p99 latency > 2s for 10 minutes"                    │
│  • "Success rate < 99.9% (SLO breach)"                  │
│  • "Queue depth growing for 15 minutes"                 │
│                                                          │
│  ✗ CAUSE-BASED (Avoid alerting on these):               │
│  • "CPU > 80%" (might not affect users)                 │
│  • "Memory > 90%" (might be normal caching)             │
│  • "1 pod restarted" (self-healing)                     │
│  • "Disk > 75%" (not urgent yet)                        │
│                                                          │
│  Why: Causes without impact = noise                     │
│       Symptoms guarantee user impact                     │
└──────────────────────────────────────────────────────────┘
```

### The Four Golden Signals for Alerting

| Signal | Alert Example | Threshold Example |
|--------|--------------|-------------------|
| Latency | p99 response time too high | p99 > 500ms for 5min |
| Traffic | Unusual traffic drop | Traffic < 50% of baseline |
| Errors | Error rate exceeds threshold | 5xx rate > 1% for 5min |
| Saturation | Resource exhaustion approaching | Queue depth > 10k for 10min |

---

## 1.3 SLO-Based Alerting

```
┌──────────────────────────────────────────────────────────┐
│         SLO-BASED ALERTING (Best Practice)               │
│                                                          │
│  SLO: 99.9% availability (43.8 min downtime/month)      │
│  Error Budget: 0.1% of requests can fail                 │
│                                                          │
│  MULTI-WINDOW BURN RATE ALERTS:                          │
│                                                          │
│  ┌────────────────────────────────────────────────┐      │
│  │ Severity │ Burn Rate │ Long Win │ Short Win   │      │
│  ├──────────┼───────────┼──────────┼─────────────┤      │
│  │ PAGE     │ 14.4x     │ 1 hour   │ 5 minutes  │      │
│  │ PAGE     │ 6x        │ 6 hours  │ 30 minutes │      │
│  │ TICKET   │ 3x        │ 1 day    │ 2 hours    │      │
│  │ TICKET   │ 1x        │ 3 days   │ 6 hours    │      │
│  └────────────────────────────────────────────────┘      │
│                                                          │
│  Burn Rate 14.4x = Entire monthly budget consumed in     │
│  1 hour → PAGE immediately                               │
│                                                          │
│  Burn Rate 1x = Budget consumed at exactly allowed rate  │
│  → TICKET, investigate during business hours             │
└──────────────────────────────────────────────────────────┘
```

### PromQL Burn Rate Example

```yaml
# Multi-window, multi-burn-rate alert
groups:
  - name: slo-alerts
    rules:
      # Critical: 14.4x burn rate over 1h (checked in 5m window)
      - alert: HighErrorBurnRate_Critical
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h]))
            /
            sum(rate(http_requests_total[1h]))
          ) > (14.4 * 0.001)
          AND
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > (14.4 * 0.001)
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error burn rate - SLO at risk"
          description: "Error budget being consumed 14.4x faster than allowed"

      # Warning: 3x burn rate over 1d (checked in 2h window)
      - alert: HighErrorBurnRate_Warning
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1d]))
            /
            sum(rate(http_requests_total[1d]))
          ) > (3 * 0.001)
          AND
          (
            sum(rate(http_requests_total{status=~"5.."}[2h]))
            /
            sum(rate(http_requests_total[2h]))
          ) > (3 * 0.001)
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "Elevated error burn rate"
```

---

## 1.4 Alert Severity Levels

```
┌──────────────────────────────────────────────────────────┐
│  ALERT SEVERITY FRAMEWORK                                │
│                                                          │
│  ┌──────────┬────────────────┬──────────────────────┐   │
│  │ Severity │ Response Time  │ Action Required      │   │
│  ├──────────┼────────────────┼──────────────────────┤   │
│  │ P1/SEV1  │ < 15 minutes   │ Page on-call, all    │   │
│  │ Critical │                │ hands on deck        │   │
│  ├──────────┼────────────────┼──────────────────────┤   │
│  │ P2/SEV2  │ < 1 hour       │ Page on-call,        │   │
│  │ High     │                │ investigate now      │   │
│  ├──────────┼────────────────┼──────────────────────┤   │
│  │ P3/SEV3  │ < 4 hours      │ Ticket, fix during   │   │
│  │ Medium   │                │ business hours       │   │
│  ├──────────┼────────────────┼──────────────────────┤   │
│  │ P4/SEV4  │ Next sprint    │ Backlog item,        │   │
│  │ Low      │                │ improve when able    │   │
│  └──────────┴────────────────┴──────────────────────┘   │
│                                                          │
│  Rule: Only P1/P2 should page (wake people up)          │
│  P3/P4 = Slack/email/ticket only                        │
└──────────────────────────────────────────────────────────┘
```

---

## 1.5 Alert Design Best Practices

### Writing Good Alert Rules

```yaml
# ✓ GOOD ALERT: Actionable, symptom-based, with context
- alert: APIHighErrorRate
  expr: |
    sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
    /
    sum(rate(http_requests_total[5m])) by (service)
    > 0.01
  for: 5m
  labels:
    severity: critical
    team: platform
  annotations:
    summary: "{{ $labels.service }} error rate above 1%"
    description: |
      Service {{ $labels.service }} has {{ $value | humanizePercentage }}
      error rate over the last 5 minutes.
    runbook_url: https://wiki.example.com/runbook/api-high-error-rate
    dashboard_url: https://grafana.example.com/d/api-overview
    impact: "Users may see 500 errors on the API"

# ✗ BAD ALERT: Vague, no context, cause-based
- alert: HighCPU
  expr: node_cpu_usage > 80
  labels:
    severity: critical
  annotations:
    summary: "CPU is high"
```

### Alert Annotation Checklist

```
┌────────────────────────────────────────────────┐
│  EVERY ALERT SHOULD INCLUDE:                   │
│                                                │
│  □ Summary - one-line description              │
│  □ Description - current value, threshold      │
│  □ Impact - what users/services are affected   │
│  □ Runbook URL - link to troubleshooting doc   │
│  □ Dashboard URL - link to relevant dashboard  │
│  □ Team - who owns this alert                  │
│  □ Severity - P1/P2/P3/P4                     │
│  □ Service - which service is affected         │
└────────────────────────────────────────────────┘
```

---

## 1.6 Reducing Alert Noise

| Technique | Description | Implementation |
|-----------|-------------|----------------|
| `for` duration | Wait before firing | `for: 5m` in Prometheus rules |
| Rate over time | Avoid single spike alerts | `rate(errors[5m]) > 0.01` |
| Multi-window | Confirm in short AND long window | Burn rate alerting |
| Grouping | Combine related alerts | Alertmanager `group_by` |
| Inhibition | Suppress dependent alerts | Suppress service alert if cluster is down |
| Silences | Mute during maintenance | Alertmanager silences |
| Dead-man's switch | Alert ONLY if monitoring fails | Watchdog alert that always fires |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Alert fatigue | Too many non-actionable alerts | Review and remove alerts nobody acts on |
| Flapping alerts | Threshold too sensitive | Add `for` duration, widen threshold |
| Missed outages | Symptom not covered | Add SLO-based alerting |
| Alert storms | No grouping/inhibition | Configure Alertmanager grouping |
| Late detection | Evaluation interval too long | Reduce interval, use short windows |

---

## Quick Revision Questions

1. **Why should you alert on symptoms rather than causes?**
2. **What is a burn rate and how does multi-window burn rate alerting work?**
3. **What are the four alert severity levels and when should each page on-call?**
4. **What annotations should every production alert include?**
5. **How do you reduce alert noise without missing real incidents?**
6. **How does SLO-based alerting differ from threshold-based alerting?**

---

[← Previous: Trace Analysis](../07-Distributed-Tracing/06-trace-analysis-optimization.md) | [Back to README](../README.md) | [Next: Alert Routing →](02-alert-routing.md)
