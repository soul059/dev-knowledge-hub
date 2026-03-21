# 🚨 Alerting Systems

> **Unit 8 · Chapter 4** — Setting up alerts for model degradation: PagerDuty/Slack integration, alert fatigue prevention, severity levels, escalation policies, and operational runbooks.

---

## 📋 Chapter Overview

Monitoring without alerting is just data collection. **Alerting** transforms monitoring data into actionable signals — it ensures that when a model degrades, the right people are notified at the right time with the right context. But poorly designed alerting is worse than no alerting: too many false alarms cause **alert fatigue**, and teams start ignoring notifications, missing the real incidents.

Effective ML alerting requires understanding severity levels, designing escalation policies, integrating with incident management tools, and writing runbooks that enable on-call engineers to respond quickly — even if they didn't build the model.

This chapter covers:

- Alert design principles for ML systems
- Severity levels and escalation policies
- PagerDuty, Slack, and email integration
- Alert fatigue prevention strategies
- Writing effective runbooks for ML incidents

---

## 🏗️ ML Alerting Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    ALERTING PIPELINE                             │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────────┐  │
  │  │                   SIGNAL SOURCES                           │  │
  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐ │  │
  │  │  │ Model    │ │ System   │ │ Data     │ │ Business     │ │  │
  │  │  │ Quality  │ │ Health   │ │ Quality  │ │ KPIs         │ │  │
  │  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └──────┬───────┘ │  │
  │  └───────┼────────────┼────────────┼───────────────┼─────────┘  │
  │          └────────────┴──────┬─────┴───────────────┘            │
  │                              ▼                                   │
  │                  ┌────────────────────────┐                     │
  │                  │   ALERT EVALUATOR      │                     │
  │                  │   ────────────────     │                     │
  │                  │   • Threshold check    │                     │
  │                  │   • Duration check     │                     │
  │                  │   • Deduplication      │                     │
  │                  │   • Grouping           │                     │
  │                  └────────────┬───────────┘                     │
  │                               │                                 │
  │                  ┌────────────┼────────────┐                    │
  │                  ▼            ▼            ▼                    │
  │          ┌────────────┐ ┌──────────┐ ┌──────────────┐          │
  │          │  CRITICAL  │ │ WARNING  │ │    INFO      │          │
  │          │  ────────  │ │ ───────  │ │    ────      │          │
  │          │  PagerDuty │ │  Slack   │ │  Dashboard   │          │
  │          │  + Slack   │ │  #alerts │ │  Log only    │          │
  │          │  + Phone   │ │          │ │              │          │
  │          └─────┬──────┘ └────┬─────┘ └──────┬───────┘          │
  │                │             │              │                   │
  │                ▼             ▼              ▼                   │
  │          ┌──────────────────────────────────────┐               │
  │          │         INCIDENT MANAGEMENT          │               │
  │          │  • Acknowledge / Escalate / Resolve  │               │
  │          │  • Runbook links                     │               │
  │          │  • Post-mortem tracking               │               │
  │          └──────────────────────────────────────┘               │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 📊 Severity Levels

```
  ┌────────────────────────────────────────────────────────────────┐
  │  SEVERITY LEVELS FOR ML ALERTS                                │
  │                                                                │
  │  ╔════════════════════════════════════════════════════════════╗ │
  │  ║  P1 — CRITICAL         Response: < 15 min                ║ │
  │  ║  Model completely down, serving errors, or producing      ║ │
  │  ║  dangerous predictions (e.g., safety-critical system)     ║ │
  │  ╚════════════════════════════════════════════════════════════╝ │
  │                                                                │
  │  ┌────────────────────────────────────────────────────────────┐ │
  │  │  P2 — HIGH              Response: < 1 hour               │ │
  │  │  Significant accuracy degradation, high error rate,       │ │
  │  │  major data drift affecting predictions                   │ │
  │  └────────────────────────────────────────────────────────────┘ │
  │                                                                │
  │  ┌────────────────────────────────────────────────────────────┐ │
  │  │  P3 — MEDIUM            Response: < 4 hours              │ │
  │  │  Moderate drift detected, latency approaching SLO,        │ │
  │  │  minor quality degradation                                │ │
  │  └────────────────────────────────────────────────────────────┘ │
  │                                                                │
  │  ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ │
  │    P4 — LOW                Response: Next business day         │
  │    Informational drift, slight metric changes, scheduled       │
  │    retraining recommendations                                  │
  │  └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘ │
  └────────────────────────────────────────────────────────────────┘
```

### Severity Matrix for ML Alerts

| Signal | P1 Critical | P2 High | P3 Medium | P4 Low |
|--------|------------|---------|-----------|--------|
| **Error Rate** | > 10% | > 5% | > 2% | > 1% |
| **Accuracy Drop** | > 20% below SLO | > 10% below SLO | > 5% below SLO | Trending down |
| **Latency (p99)** | > 5× baseline | > 3× baseline | > 2× baseline | > 1.5× baseline |
| **Data Drift (PSI)** | — | PSI > 0.25 for all features | PSI > 0.2 for key features | PSI > 0.1 |
| **Null Predictions** | > 1% | > 0.1% | > 0.01% | Any detected |
| **Model Crash** | Any | — | — | — |

---

## 🐍 Python Implementation

### Alert Manager

```python
import logging
import json
from enum import Enum
from dataclasses import dataclass, field
from datetime import datetime, timedelta
from typing import Dict, List, Optional, Callable
from collections import defaultdict

logger = logging.getLogger("ml_alerting")


class Severity(Enum):
    CRITICAL = "P1"
    HIGH = "P2"
    MEDIUM = "P3"
    LOW = "P4"


@dataclass
class Alert:
    name: str
    severity: Severity
    message: str
    metric_value: float
    threshold: float
    model_name: str
    timestamp: datetime = field(default_factory=datetime.utcnow)
    runbook_url: Optional[str] = None
    labels: Dict[str, str] = field(default_factory=dict)
    acknowledged: bool = False
    resolved: bool = False

    def to_dict(self) -> dict:
        return {
            "name": self.name,
            "severity": self.severity.value,
            "message": self.message,
            "metric_value": self.metric_value,
            "threshold": self.threshold,
            "model_name": self.model_name,
            "timestamp": self.timestamp.isoformat(),
            "runbook_url": self.runbook_url,
            "labels": self.labels,
        }


class AlertManager:
    """
    Manages ML alerts with deduplication, grouping,
    severity routing, and fatigue prevention.
    """

    def __init__(self):
        self.active_alerts: Dict[str, Alert] = {}
        self.alert_history: List[Alert] = []
        self.channels: Dict[Severity, List[Callable]] = defaultdict(list)
        self.cooldowns: Dict[str, datetime] = {}
        self.cooldown_duration = timedelta(minutes=30)

    def register_channel(self, severity: Severity, callback: Callable):
        """Register a notification channel for a severity level."""
        self.channels[severity].append(callback)

    def fire(self, alert: Alert) -> bool:
        """Fire an alert with deduplication and cooldown checks."""
        alert_key = f"{alert.name}:{alert.model_name}"

        # Deduplication: don't fire if same alert is already active
        if alert_key in self.active_alerts:
            existing = self.active_alerts[alert_key]
            if not existing.resolved:
                logger.debug(f"Alert {alert_key} already active, skipping")
                return False

        # Cooldown: prevent re-firing too quickly after resolution
        if alert_key in self.cooldowns:
            if datetime.utcnow() < self.cooldowns[alert_key]:
                logger.debug(f"Alert {alert_key} in cooldown, skipping")
                return False

        # Fire the alert
        self.active_alerts[alert_key] = alert
        self.alert_history.append(alert)

        # Route to appropriate channels
        for channel_fn in self.channels.get(alert.severity, []):
            try:
                channel_fn(alert)
            except Exception as e:
                logger.error(f"Failed to send alert via channel: {e}")

        logger.warning(
            f"🚨 [{alert.severity.value}] {alert.name}: {alert.message}"
        )
        return True

    def resolve(self, alert_name: str, model_name: str):
        """Resolve an active alert."""
        alert_key = f"{alert_name}:{model_name}"
        if alert_key in self.active_alerts:
            self.active_alerts[alert_key].resolved = True
            self.cooldowns[alert_key] = (
                datetime.utcnow() + self.cooldown_duration
            )
            logger.info(f"✅ Alert resolved: {alert_key}")

    def get_active_alerts(self) -> List[Alert]:
        """Return all unresolved alerts."""
        return [a for a in self.active_alerts.values() if not a.resolved]

    def summary(self) -> dict:
        """Return alert summary statistics."""
        active = self.get_active_alerts()
        return {
            "total_active": len(active),
            "by_severity": {
                sev.value: len([a for a in active if a.severity == sev])
                for sev in Severity
            },
            "total_fired_all_time": len(self.alert_history),
        }
```

### Slack Integration

```python
import requests
from typing import Optional


class SlackAlertChannel:
    """Send ML alerts to Slack via webhook."""

    SEVERITY_EMOJI = {
        Severity.CRITICAL: "🔴",
        Severity.HIGH: "🟠",
        Severity.MEDIUM: "🟡",
        Severity.LOW: "🔵",
    }

    def __init__(self, webhook_url: str, channel: str = "#ml-alerts"):
        self.webhook_url = webhook_url
        self.channel = channel

    def send(self, alert: Alert):
        """Format and send alert to Slack."""
        emoji = self.SEVERITY_EMOJI.get(alert.severity, "⚪")

        blocks = [
            {
                "type": "header",
                "text": {
                    "type": "plain_text",
                    "text": f"{emoji} [{alert.severity.value}] {alert.name}",
                },
            },
            {
                "type": "section",
                "fields": [
                    {
                        "type": "mrkdwn",
                        "text": f"*Model:*\n{alert.model_name}",
                    },
                    {
                        "type": "mrkdwn",
                        "text": f"*Severity:*\n{alert.severity.value}",
                    },
                    {
                        "type": "mrkdwn",
                        "text": (
                            f"*Metric Value:*\n{alert.metric_value:.4f}"
                        ),
                    },
                    {
                        "type": "mrkdwn",
                        "text": f"*Threshold:*\n{alert.threshold:.4f}",
                    },
                ],
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*Message:*\n{alert.message}",
                },
            },
        ]

        if alert.runbook_url:
            blocks.append({
                "type": "actions",
                "elements": [
                    {
                        "type": "button",
                        "text": {
                            "type": "plain_text",
                            "text": "📖 View Runbook",
                        },
                        "url": alert.runbook_url,
                    },
                    {
                        "type": "button",
                        "text": {
                            "type": "plain_text",
                            "text": "✅ Acknowledge",
                        },
                        "action_id": f"ack_{alert.name}",
                    },
                ],
            })

        payload = {"channel": self.channel, "blocks": blocks}
        response = requests.post(self.webhook_url, json=payload)
        response.raise_for_status()


class PagerDutyChannel:
    """Send critical ML alerts to PagerDuty."""

    def __init__(self, routing_key: str):
        self.routing_key = routing_key
        self.api_url = "https://events.pagerduty.com/v2/enqueue"

    def send(self, alert: Alert):
        """Create a PagerDuty incident."""
        severity_map = {
            Severity.CRITICAL: "critical",
            Severity.HIGH: "error",
            Severity.MEDIUM: "warning",
            Severity.LOW: "info",
        }

        payload = {
            "routing_key": self.routing_key,
            "event_action": "trigger",
            "payload": {
                "summary": f"[{alert.severity.value}] {alert.name} — {alert.model_name}",
                "severity": severity_map[alert.severity],
                "source": f"ml-monitor-{alert.model_name}",
                "component": alert.model_name,
                "custom_details": {
                    "message": alert.message,
                    "metric_value": alert.metric_value,
                    "threshold": alert.threshold,
                    "runbook": alert.runbook_url,
                },
            },
        }

        if alert.runbook_url:
            payload["links"] = [
                {"href": alert.runbook_url, "text": "Runbook"},
            ]

        response = requests.post(self.api_url, json=payload)
        response.raise_for_status()
```

### Wiring It All Together

```python
# --- Setup ---
alert_manager = AlertManager()

# Configure Slack for all severity levels
slack = SlackAlertChannel(
    webhook_url="https://hooks.slack.com/services/T.../B.../xxx"
)
for severity in Severity:
    alert_manager.register_channel(severity, slack.send)

# Configure PagerDuty for critical and high only
pagerduty = PagerDutyChannel(routing_key="your-pagerduty-routing-key")
alert_manager.register_channel(Severity.CRITICAL, pagerduty.send)
alert_manager.register_channel(Severity.HIGH, pagerduty.send)


# --- Example: Model monitoring loop fires alerts ---
def check_and_alert(metrics: dict, model_name: str = "fraud_detector"):
    """Check metrics and fire alerts as needed."""

    if metrics["error_rate"] > 0.10:
        alert_manager.fire(Alert(
            name="model_error_rate_critical",
            severity=Severity.CRITICAL,
            message=f"Error rate at {metrics['error_rate']:.1%}",
            metric_value=metrics["error_rate"],
            threshold=0.10,
            model_name=model_name,
            runbook_url="https://wiki.internal/runbooks/ml-error-rate",
        ))
    elif metrics["error_rate"] > 0.05:
        alert_manager.fire(Alert(
            name="model_error_rate_high",
            severity=Severity.HIGH,
            message=f"Error rate at {metrics['error_rate']:.1%}",
            metric_value=metrics["error_rate"],
            threshold=0.05,
            model_name=model_name,
            runbook_url="https://wiki.internal/runbooks/ml-error-rate",
        ))

    if metrics["accuracy"] < 0.85:
        alert_manager.fire(Alert(
            name="model_accuracy_critical",
            severity=Severity.CRITICAL,
            message=f"Accuracy dropped to {metrics['accuracy']:.1%}",
            metric_value=metrics["accuracy"],
            threshold=0.85,
            model_name=model_name,
            runbook_url="https://wiki.internal/runbooks/ml-accuracy-drop",
        ))

# --- Example call ---
check_and_alert({
    "error_rate": 0.07,
    "accuracy": 0.88,
    "p99_latency_ms": 180,
})

print(alert_manager.summary())
```

---

## 🛡️ Alert Fatigue Prevention

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                 ALERT FATIGUE PREVENTION                         │
  │                                                                  │
  │  Problem: Too many alerts → team ignores all alerts → real       │
  │           incidents get missed → outage                          │
  │                                                                  │
  │  ┌────────────────────────────────────────────────────────────┐  │
  │  │  STRATEGY              │  HOW                             │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  1. Deduplication      │  Same alert fires once until     │  │
  │  │                        │  resolved (don't repeat)         │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  2. Cooldown periods   │  After resolution, wait 30 min  │  │
  │  │                        │  before re-firing same alert     │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  3. Duration filters   │  Only alert if condition holds   │  │
  │  │                        │  for N minutes (not transient)   │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  4. Alert grouping     │  Group related alerts into one   │  │
  │  │                        │  notification                    │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  5. Threshold tuning   │  Review and adjust thresholds    │  │
  │  │                        │  monthly based on false alarm    │  │
  │  │                        │  rate                            │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  6. Actionable only    │  Every alert must have a clear   │  │
  │  │                        │  action — if not, delete it      │  │
  │  ├────────────────────────┼──────────────────────────────────┤  │
  │  │  7. Severity routing   │  P4/P3 → dashboard only         │  │
  │  │                        │  P2 → Slack                      │  │
  │  │                        │  P1 → PagerDuty + phone          │  │
  │  └────────────────────────┴──────────────────────────────────┘  │
  │                                                                  │
  │  Rule of thumb: If an alert fires and the response is            │
  │  "ignore it", the alert should be deleted or re-thresholded.     │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 📋 ML Incident Runbook Template

```
  ╔════════════════════════════════════════════════════════════════╗
  ║               RUNBOOK: Model Accuracy Drop                    ║
  ╠════════════════════════════════════════════════════════════════╣
  ║                                                                ║
  ║  ALERT: model_accuracy_critical                                ║
  ║  SEVERITY: P1 / P2                                             ║
  ║  ON-CALL: ML Platform Team                                     ║
  ║                                                                ║
  ║  ── STEP 1: TRIAGE (2 min) ──                                 ║
  ║  □ Check: Is the model still serving? (health endpoint)        ║
  ║  □ Check: Error rate on Grafana dashboard                      ║
  ║  □ Check: Recent deployments (any new model version?)          ║
  ║                                                                ║
  ║  ── STEP 2: DIAGNOSE (10 min) ──                              ║
  ║  □ Check data drift dashboard — any feature drift?             ║
  ║  □ Check upstream data pipelines — any failures?               ║
  ║  □ Check prediction distribution — shape change?               ║
  ║  □ Compare with shadow model (if available)                    ║
  ║                                                                ║
  ║  ── STEP 3: MITIGATE ──                                       ║
  ║  □ If model crash → rollback to previous version               ║
  ║  □ If data issue → fix pipeline, re-run batch                  ║
  ║  □ If drift → trigger retraining pipeline                      ║
  ║  □ If unclear → enable fallback / rule-based system            ║
  ║                                                                ║
  ║  ── STEP 4: VERIFY (15 min) ──                                ║
  ║  □ Confirm accuracy returning to SLO                           ║
  ║  □ Monitor for 30 min for stability                            ║
  ║  □ Update incident channel with status                         ║
  ║                                                                ║
  ║  ── STEP 5: POST-MORTEM ──                                    ║
  ║  □ Document root cause                                         ║
  ║  □ File action items to prevent recurrence                     ║
  ║  □ Update this runbook if needed                               ║
  ║                                                                ║
  ║  LINKS:                                                        ║
  ║  • Grafana Dashboard: https://grafana.internal/ml-dashboard    ║
  ║  • Model Registry: https://mlflow.internal/models              ║
  ║  • Rollback Guide: ../07-Model-Deployment/06-rollback.md       ║
  ╚════════════════════════════════════════════════════════════════╝
```

---

## 📊 Alert Routing Summary

| Severity | Channel | Response Time | Escalation | Example Trigger |
|----------|---------|---------------|------------|----------------|
| **P1 Critical** | PagerDuty + Slack + Phone | < 15 min | Auto-escalate after 15 min | Model crash, > 10% error rate |
| **P2 High** | PagerDuty + Slack | < 1 hour | Auto-escalate after 1 hour | Accuracy 10%+ below SLO |
| **P3 Medium** | Slack #ml-alerts | < 4 hours | Manual escalation | Moderate drift, latency warning |
| **P4 Low** | Dashboard / email digest | Next business day | No escalation | Minor drift, scheduled retrain |

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Every alert must be actionable** — if there's no clear action to take, the alert shouldn't exist or should be downgraded to a dashboard metric. |
| 2 | **Route by severity** — P1 pages the on-call engineer; P4 is a dashboard note. Don't send everything to the same channel. |
| 3 | **Deduplication + cooldowns prevent alert storms** — without them, a single failing metric can generate hundreds of notifications per hour. |
| 4 | **Runbooks are force multipliers** — they enable any on-call engineer to respond to ML incidents, not just the model author. |
| 5 | **Review alert health monthly** — track false positive rate, time-to-acknowledge, and whether alerts were actionable. Delete or tune alerts that aren't working. |

---

## ❓ Revision Questions

1. **What is alert fatigue, and why is it dangerous for ML systems?**
   > Alert fatigue occurs when teams receive so many alerts that they begin ignoring them. It's especially dangerous for ML systems because model degradation is often gradual and subtle — if teams are desensitized to alerts, they'll miss the early warnings of accuracy drops or drift, leading to prolonged periods of poor predictions that are only caught when business metrics suffer.

2. **Why should PagerDuty alerts be reserved for P1/P2 severity only?**
   > PagerDuty triggers phone calls and interrupts engineers (potentially at 3 AM). If low-severity alerts use PagerDuty, on-call engineers learn to dismiss pages, defeating the purpose. Reserving PagerDuty for genuinely critical alerts ensures that when a page fires, the engineer knows it demands immediate attention. Lower severity alerts should go to Slack or dashboards.

3. **How does alert deduplication work, and why is it necessary?**
   > Deduplication tracks active alerts by a unique key (typically alert name + model name). If the same alert is already active and unresolved, new firings of that alert are suppressed. Without deduplication, a failing metric checked every 15 seconds would generate 240 notifications per hour for the same issue, overwhelming notification channels.

4. **What should an ML incident runbook include that a standard software runbook might not?**
   > ML runbooks need: (1) data drift dashboard checks, (2) upstream data pipeline verification, (3) prediction distribution analysis, (4) model rollback procedures specific to ML registries, (5) retraining trigger instructions, and (6) shadow model comparison steps. Standard runbooks focus on service restarts and infrastructure, while ML runbooks must also diagnose whether the issue is data, model, or infrastructure related.

5. **How would you set alert thresholds for a model you've never deployed before?**
   > Start with conservative thresholds based on offline evaluation metrics and domain expertise. Deploy the model in shadow mode for 1-2 weeks to establish production baselines. Set initial warning thresholds at 1.5× the observed variance and critical at 2×. Review and tighten thresholds after the first month based on actual alert volumes and false positive rates. Use the "for" duration clause to filter out transient spikes.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Performance Monitoring](./03-performance-monitoring.md) | [📂 Model Monitoring](./README.md) | [Logging and Debugging →](./05-logging-and-debugging.md) |

---

*Unit 8 · Chapter 4 · Alerting Systems — MLOps Course Notes*
