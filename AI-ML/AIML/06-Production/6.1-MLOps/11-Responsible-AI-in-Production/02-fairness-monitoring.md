# 📊 Fairness Monitoring in Production

> **Unit 11 · Responsible AI in Production · Chapter 2**
>
> How to continuously monitor fairness in deployed ML systems — real-time fairness dashboards, tracking metrics across demographic groups over time, alerting on fairness degradation, regulatory reporting requirements, and building fairness into your observability stack.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Why Fairness Drifts in Production](#why-fairness-drifts-in-production)
3. [Continuous Fairness Checks](#continuous-fairness-checks)
4. [Fairness Dashboards](#fairness-dashboards)
5. [Alerting on Fairness Degradation](#alerting-on-fairness-degradation)
6. [Regulatory Reporting Requirements](#regulatory-reporting-requirements)
7. [End-to-End Fairness Monitoring System](#end-to-end-fairness-monitoring-system)
8. [Summary](#summary)
9. [Revision Questions](#revision-questions)
10. [Navigation](#navigation)

---

## Chapter Overview

A model that passes a bias audit at deployment can become unfair within weeks. Population shifts, feedback loops, and changing data distributions cause **fairness drift** — gradual or sudden changes in how a model treats different groups. Pre-deployment bias testing is necessary but not sufficient. This chapter covers how to build **continuous fairness monitoring** into your production observability stack: computing fairness metrics on live traffic, building dashboards that slice performance by demographic groups, setting alert thresholds for fairness degradation, and meeting regulatory reporting requirements.

```
Pre-deployment Audit vs Continuous Monitoring
┌──────────────────────────────┐    ┌──────────────────────────────┐
│   Pre-deployment Audit       │    │   Continuous Monitoring       │
│   (Chapter 1)                │    │   (This Chapter)              │
│                              │    │                              │
│  ✅ One-time check           │    │  ✅ Ongoing checks            │
│  ✅ Static test data         │    │  ✅ Live production data      │
│  ✅ Known distribution       │    │  ✅ Detects distribution shift│
│  ❌ Misses production drift  │    │  ✅ Catches feedback loops    │
│  ❌ Snapshot in time         │    │  ✅ Time-series trends        │
│  ❌ No feedback loop detect  │    │  ✅ Regulatory evidence       │
└──────────────────────────────┘    └──────────────────────────────┘
```

---

## Why Fairness Drifts in Production

### Sources of Fairness Drift

```
┌───────────────────────────────────────────────────────────────┐
│             Sources of Fairness Drift                         │
│                                                               │
│  1. Population shift                                          │
│     • Demographic mix of users changes over time              │
│     • Model sees groups not well-represented in training      │
│                                                               │
│  2. Feature drift                                             │
│     • Feature distributions change differently per group      │
│     • e.g., income distributions shift for one group          │
│                                                               │
│  3. Feedback loops                                            │
│     • Model decisions affect future data                      │
│     • Denied loans → less data on repayment → worse model     │
│                                                               │
│  4. Label drift                                               │
│     • Outcome definitions or measurement changes              │
│     • e.g., policy change redefines "default"                 │
│                                                               │
│  5. Concept drift                                             │
│     • Relationship between features and outcomes changes      │
│     • Different rates of change across groups                 │
└───────────────────────────────────────────────────────────────┘
```

| Drift Type | Detection Signal | Example |
|---|---|---|
| **Population shift** | Group proportions change | More applicants from new demographic |
| **Feature drift** | Per-group feature distributions diverge | Income mean shifts for group B but not A |
| **Feedback loop** | Increasing performance gap over time | TPR gap widens each month |
| **Label drift** | Outcome rates change per group | Default rate drops for one group only |
| **Concept drift** | Per-group accuracy degrades at different rates | Model accuracy drops for group B only |

---

## Continuous Fairness Checks

### Fairness Metrics Logger

```python
# monitoring/fairness_logger.py
"""Log fairness metrics for production predictions."""
import time
import json
import logging
from collections import defaultdict
from dataclasses import dataclass, field
from typing import Optional
import numpy as np

logger = logging.getLogger(__name__)


@dataclass
class FairnessWindow:
    """Accumulates predictions over a time window for fairness computation."""
    window_seconds: int = 3600  # 1 hour default
    start_time: float = field(default_factory=time.time)
    predictions: dict = field(default_factory=lambda: defaultdict(list))
    labels: dict = field(default_factory=lambda: defaultdict(list))

    def add(self, group: str, prediction: int, label: Optional[int] = None):
        self.predictions[group].append(prediction)
        if label is not None:
            self.labels[group].append(label)

    def is_expired(self) -> bool:
        return (time.time() - self.start_time) > self.window_seconds

    def compute_metrics(self) -> dict:
        """Compute fairness metrics for the current window."""
        groups = list(self.predictions.keys())
        if len(groups) < 2:
            return {"error": "Need at least 2 groups"}

        # Selection rates per group
        selection_rates = {}
        for g in groups:
            preds = self.predictions[g]
            selection_rates[g] = sum(preds) / len(preds) if preds else 0.0

        rates = list(selection_rates.values())
        dp_diff = max(rates) - min(rates)
        di_ratio = min(rates) / max(rates) if max(rates) > 0 else 0.0

        result = {
            "timestamp": time.time(),
            "window_seconds": self.window_seconds,
            "group_counts": {g: len(self.predictions[g]) for g in groups},
            "selection_rates": selection_rates,
            "demographic_parity_diff": dp_diff,
            "disparate_impact_ratio": di_ratio,
        }

        # If labels are available, compute TPR per group
        if all(len(self.labels[g]) > 0 for g in groups):
            tpr_per_group = {}
            for g in groups:
                y_true = np.array(self.labels[g])
                y_pred = np.array(self.predictions[g][: len(y_true)])
                positives = y_true == 1
                if positives.sum() > 0:
                    tpr_per_group[g] = y_pred[positives].mean()
            if tpr_per_group:
                tprs = list(tpr_per_group.values())
                result["tpr_per_group"] = tpr_per_group
                result["equalized_odds_tpr_diff"] = max(tprs) - min(tprs)

        return result


class FairnessMonitor:
    """Production fairness monitor with rolling windows."""

    def __init__(self, window_seconds: int = 3600, emit_fn=None):
        self.window_seconds = window_seconds
        self.current_window = FairnessWindow(window_seconds=window_seconds)
        self.emit_fn = emit_fn or self._default_emit

    def record(self, group: str, prediction: int, label: Optional[int] = None):
        """Record a prediction for fairness tracking."""
        if self.current_window.is_expired():
            self._flush()
        self.current_window.add(group, prediction, label)

    def _flush(self):
        metrics = self.current_window.compute_metrics()
        self.emit_fn(metrics)
        self.current_window = FairnessWindow(window_seconds=self.window_seconds)

    @staticmethod
    def _default_emit(metrics: dict):
        logger.info("fairness_metrics %s", json.dumps(metrics, default=str))

    def force_flush(self):
        """Force computation and emission of current window metrics."""
        self._flush()


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    monitor = FairnessMonitor(window_seconds=60)

    np.random.seed(42)
    for _ in range(500):
        group = np.random.choice(["A", "B"])
        # Simulate biased predictions
        pred = int(np.random.random() < (0.7 if group == "A" else 0.4))
        label = int(np.random.random() < 0.5)
        monitor.record(group, pred, label)

    monitor.force_flush()
```

### Scheduled Fairness Evaluation

```python
# monitoring/fairness_scheduler.py
"""Run fairness evaluations on a schedule against production data."""
import json
from datetime import datetime, timedelta
from pathlib import Path
from fairlearn.metrics import (
    MetricFrame,
    demographic_parity_difference,
    equalized_odds_difference,
    selection_rate,
    true_positive_rate,
    false_positive_rate,
)
import pandas as pd
import numpy as np


def evaluate_fairness_window(
    predictions_df: pd.DataFrame,
    sensitive_col: str,
    pred_col: str = "prediction",
    label_col: str = "label",
) -> dict:
    """Evaluate fairness metrics for a time window of predictions."""
    metrics = {
        "selection_rate": selection_rate,
        "true_positive_rate": true_positive_rate,
        "false_positive_rate": false_positive_rate,
    }

    mf = MetricFrame(
        metrics=metrics,
        y_true=predictions_df[label_col],
        y_pred=predictions_df[pred_col],
        sensitive_features=predictions_df[sensitive_col],
    )

    return {
        "timestamp": datetime.utcnow().isoformat(),
        "n_predictions": len(predictions_df),
        "overall": mf.overall.to_dict(),
        "by_group": {
            col: mf.by_group[col].to_dict() for col in mf.by_group.columns
        },
        "differences": mf.difference().to_dict(),
        "demographic_parity_diff": demographic_parity_difference(
            predictions_df[label_col],
            predictions_df[pred_col],
            sensitive_features=predictions_df[sensitive_col],
        ),
        "equalized_odds_diff": equalized_odds_difference(
            predictions_df[label_col],
            predictions_df[pred_col],
            sensitive_features=predictions_df[sensitive_col],
        ),
    }


def run_daily_fairness_report(
    data_path: str, output_dir: str, sensitive_col: str
) -> Path:
    """Generate daily fairness report and save to file."""
    df = pd.read_parquet(data_path)
    result = evaluate_fairness_window(df, sensitive_col)

    output_path = Path(output_dir) / f"fairness_{datetime.utcnow():%Y%m%d}.json"
    output_path.parent.mkdir(parents=True, exist_ok=True)
    output_path.write_text(json.dumps(result, indent=2, default=str))

    return output_path
```

---

## Fairness Dashboards

### Dashboard Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                  Fairness Dashboard                            │
│                                                               │
│  ┌──────────────── Overall Health ────────────────────────┐   │
│  │  Demographic Parity Diff:  0.08  ✅  (threshold: 0.10) │   │
│  │  Equalized Odds Diff:      0.12  ⚠️  (threshold: 0.10) │   │
│  │  Disparate Impact Ratio:   0.82  ✅  (threshold: 0.80) │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────── Selection Rate by Group ───────────────────┐   │
│  │                                                        │   │
│  │  Group A  ████████████████████░░░░░  72%               │   │
│  │  Group B  ████████████████░░░░░░░░░  64%               │   │
│  │  Group C  ██████████████░░░░░░░░░░░  58%               │   │
│  │                                                        │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────── Fairness Over Time ────────────────────────┐   │
│  │  DP Diff                                               │   │
│  │  0.15 ┤                          ╭─── ⚠️ alert         │   │
│  │  0.10 ┤─ ─ ─ ─ threshold ─ ─ ─ ╭╯─ ─ ─ ─ ─ ─ ─ ─    │   │
│  │  0.05 ┤    ╭───╮   ╭──╮    ╭──╯                       │   │
│  │  0.00 ┤───╯    ╰──╯   ╰──╯                            │   │
│  │       └───┬────┬────┬────┬────┬────┬────               │   │
│  │         Mon  Tue  Wed  Thu  Fri  Sat  Sun              │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                               │
│  ┌──────────── Per-Group Metrics ─────────────────────────┐   │
│  │  Metric        │ Group A │ Group B │ Group C │ Diff    │   │
│  │  TPR            │  0.88   │  0.76   │  0.80   │  0.12  │   │
│  │  FPR            │  0.08   │  0.11   │  0.09   │  0.03  │   │
│  │  Selection Rate │  0.72   │  0.64   │  0.58   │  0.14  │   │
│  │  Sample Size    │  4521   │  3218   │  1891   │  —     │   │
│  └────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────┘
```

### Prometheus Metrics Exporter

```python
# monitoring/fairness_prometheus.py
"""Export fairness metrics to Prometheus for Grafana dashboards."""
from prometheus_client import Gauge, Counter, Histogram, start_http_server
from collections import defaultdict
import time
import threading
import numpy as np


# ── Prometheus metrics ───────────────────────────────────
fairness_selection_rate = Gauge(
    "ml_fairness_selection_rate",
    "Selection rate per demographic group",
    ["model_name", "group"],
)
fairness_dp_diff = Gauge(
    "ml_fairness_demographic_parity_diff",
    "Demographic parity difference",
    ["model_name"],
)
fairness_di_ratio = Gauge(
    "ml_fairness_disparate_impact_ratio",
    "Disparate impact ratio (80% rule)",
    ["model_name"],
)
fairness_tpr = Gauge(
    "ml_fairness_true_positive_rate",
    "True positive rate per group",
    ["model_name", "group"],
)
prediction_counter = Counter(
    "ml_predictions_total",
    "Total predictions per group",
    ["model_name", "group", "prediction"],
)


class PrometheusFairnessExporter:
    """Collects predictions and periodically exports fairness metrics."""

    def __init__(self, model_name: str, window_seconds: int = 300):
        self.model_name = model_name
        self.window_seconds = window_seconds
        self._predictions = defaultdict(list)
        self._labels = defaultdict(list)
        self._lock = threading.Lock()

    def record(self, group: str, prediction: int, label: int = None):
        prediction_counter.labels(
            model_name=self.model_name,
            group=group,
            prediction=str(prediction),
        ).inc()

        with self._lock:
            self._predictions[group].append(prediction)
            if label is not None:
                self._labels[group].append(label)

    def export(self):
        """Compute and export current fairness metrics to Prometheus."""
        with self._lock:
            groups = list(self._predictions.keys())
            if len(groups) < 2:
                return

            rates = {}
            for g in groups:
                preds = self._predictions[g]
                rate = sum(preds) / len(preds) if preds else 0.0
                rates[g] = rate
                fairness_selection_rate.labels(
                    model_name=self.model_name, group=g
                ).set(rate)

            # Demographic parity difference
            all_rates = list(rates.values())
            dp_diff = max(all_rates) - min(all_rates)
            fairness_dp_diff.labels(model_name=self.model_name).set(dp_diff)

            # Disparate impact ratio
            di = min(all_rates) / max(all_rates) if max(all_rates) > 0 else 0
            fairness_di_ratio.labels(model_name=self.model_name).set(di)

            # TPR per group (if labels available)
            for g in groups:
                if self._labels[g]:
                    y_true = np.array(self._labels[g])
                    y_pred = np.array(self._predictions[g][: len(y_true)])
                    positives = y_true == 1
                    if positives.sum() > 0:
                        tpr = y_pred[positives].mean()
                        fairness_tpr.labels(
                            model_name=self.model_name, group=g
                        ).set(tpr)

            # Reset window
            self._predictions.clear()
            self._labels.clear()

    def start_periodic_export(self):
        """Start a background thread that exports metrics periodically."""
        def _loop():
            while True:
                time.sleep(self.window_seconds)
                self.export()

        t = threading.Thread(target=_loop, daemon=True)
        t.start()


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    start_http_server(8000)
    exporter = PrometheusFairnessExporter("loan_model", window_seconds=60)
    exporter.start_periodic_export()

    print("Fairness metrics available at http://localhost:8000/metrics")
    # In production, record() would be called from your serving code
    while True:
        time.sleep(1)
```

### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "ML Fairness Monitor",
    "panels": [
      {
        "title": "Demographic Parity Difference",
        "type": "gauge",
        "targets": [
          {
            "expr": "ml_fairness_demographic_parity_diff{model_name='loan_model'}",
            "legendFormat": "DP Diff"
          }
        ],
        "thresholds": [
          {"value": 0, "color": "green"},
          {"value": 0.08, "color": "yellow"},
          {"value": 0.10, "color": "red"}
        ]
      },
      {
        "title": "Selection Rate by Group",
        "type": "timeseries",
        "targets": [
          {
            "expr": "ml_fairness_selection_rate{model_name='loan_model'}",
            "legendFormat": "{{group}}"
          }
        ]
      },
      {
        "title": "Disparate Impact Ratio",
        "type": "stat",
        "targets": [
          {
            "expr": "ml_fairness_disparate_impact_ratio{model_name='loan_model'}",
            "legendFormat": "DI Ratio"
          }
        ],
        "thresholds": [
          {"value": 0, "color": "red"},
          {"value": 0.8, "color": "green"}
        ]
      }
    ]
  }
}
```

---

## Alerting on Fairness Degradation

### Alert Strategy

```
┌───────────────────────────────────────────────────────────────┐
│              Fairness Alert Tiers                              │
│                                                               │
│  Tier 1 — INFO (Log + Dashboard)                              │
│  • DP diff > 0.05                                             │
│  • Any group sample size < 100 in window                      │
│                                                               │
│  Tier 2 — WARNING (Slack + On-call page)                      │
│  • DP diff > 0.08                                             │
│  • Equalized odds diff > 0.08                                 │
│  • DI ratio < 0.85                                            │
│                                                               │
│  Tier 3 — CRITICAL (PagerDuty + Auto-rollback)                │
│  • DP diff > 0.15                                             │
│  • DI ratio < 0.70                                            │
│  • TPR for any group drops below absolute minimum             │
│  • Sustained violation for > 2 consecutive windows            │
└───────────────────────────────────────────────────────────────┘
```

### Alert Rules Implementation

```python
# monitoring/fairness_alerts.py
"""Fairness alert system with tiered severity."""
import json
import logging
from dataclasses import dataclass
from enum import Enum
from typing import Callable, Optional

logger = logging.getLogger(__name__)


class Severity(Enum):
    INFO = "info"
    WARNING = "warning"
    CRITICAL = "critical"


@dataclass
class AlertRule:
    name: str
    metric: str
    condition: Callable[[float], bool]
    severity: Severity
    message_template: str


ALERT_RULES = [
    # ── Tier 1 — INFO ────────────────────────────────────
    AlertRule(
        name="dp_elevated",
        metric="demographic_parity_diff",
        condition=lambda v: v > 0.05,
        severity=Severity.INFO,
        message_template="DP diff elevated: {value:.4f} (> 0.05)",
    ),
    # ── Tier 2 — WARNING ─────────────────────────────────
    AlertRule(
        name="dp_warning",
        metric="demographic_parity_diff",
        condition=lambda v: v > 0.08,
        severity=Severity.WARNING,
        message_template="DP diff high: {value:.4f} (> 0.08)",
    ),
    AlertRule(
        name="di_warning",
        metric="disparate_impact_ratio",
        condition=lambda v: v < 0.85,
        severity=Severity.WARNING,
        message_template="DI ratio low: {value:.4f} (< 0.85)",
    ),
    # ── Tier 3 — CRITICAL ────────────────────────────────
    AlertRule(
        name="dp_critical",
        metric="demographic_parity_diff",
        condition=lambda v: v > 0.15,
        severity=Severity.CRITICAL,
        message_template="DP diff critical: {value:.4f} (> 0.15). Consider rollback.",
    ),
    AlertRule(
        name="di_critical",
        metric="disparate_impact_ratio",
        condition=lambda v: v < 0.70,
        severity=Severity.CRITICAL,
        message_template="DI ratio critical: {value:.4f} (< 0.70). Auto-rollback triggered.",
    ),
]


def evaluate_alerts(metrics: dict, rules: list[AlertRule] = ALERT_RULES) -> list[dict]:
    """Evaluate fairness metrics against alert rules."""
    fired = []
    for rule in rules:
        value = metrics.get(rule.metric)
        if value is not None and rule.condition(value):
            alert = {
                "rule": rule.name,
                "severity": rule.severity.value,
                "metric": rule.metric,
                "value": value,
                "message": rule.message_template.format(value=value),
            }
            fired.append(alert)
            logger.log(
                logging.CRITICAL if rule.severity == Severity.CRITICAL
                else logging.WARNING if rule.severity == Severity.WARNING
                else logging.INFO,
                "FAIRNESS ALERT [%s] %s",
                rule.severity.value.upper(),
                alert["message"],
            )
    return fired


def send_alert(alert: dict, channel: str = "slack"):
    """Send alert to notification channel (stub)."""
    if alert["severity"] == "critical":
        # PagerDuty integration
        logger.critical("PAGERDUTY: %s", alert["message"])
    elif alert["severity"] == "warning":
        # Slack integration
        logger.warning("SLACK: %s", alert["message"])
    else:
        logger.info("DASHBOARD: %s", alert["message"])


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)

    sample_metrics = {
        "demographic_parity_diff": 0.12,
        "disparate_impact_ratio": 0.78,
        "equalized_odds_tpr_diff": 0.09,
    }

    alerts = evaluate_alerts(sample_metrics)
    print(f"\n{len(alerts)} alert(s) fired:")
    for a in alerts:
        print(f"  [{a['severity'].upper()}] {a['message']}")
        send_alert(a)
```

---

## Regulatory Reporting Requirements

### Regulatory Landscape

| Regulation | Fairness Requirement | Reporting Cadence |
|---|---|---|
| **ECOA / Reg B** (US) | Cannot discriminate on race, sex, age in credit | Per-decision adverse action notices |
| **EU AI Act** | High-risk AI must monitor for bias | Continuous monitoring + annual audit |
| **NYC Local Law 144** | Automated employment decision tools bias audit | Annual independent audit |
| **GDPR Art. 22** | Right to human review of automated decisions | On request |
| **SR 11-7** (US Banking) | Model risk management for all material models | Ongoing + periodic review |

### Compliance Report Generator

```python
# monitoring/compliance_report.py
"""Generate regulatory compliance reports for fairness monitoring."""
import json
from datetime import datetime
from pathlib import Path
from typing import Optional


def generate_compliance_report(
    model_name: str,
    fairness_metrics: dict,
    thresholds: dict,
    sensitive_attributes: list[str],
    regulation: str = "EU AI Act",
    period_start: Optional[str] = None,
    period_end: Optional[str] = None,
) -> dict:
    """Generate a structured compliance report."""
    now = datetime.utcnow().isoformat()

    # Evaluate pass/fail for each metric
    checks = {}
    for metric, value in fairness_metrics.items():
        if metric in thresholds:
            threshold = thresholds[metric]
            if metric == "disparate_impact_ratio":
                passed = value >= threshold
            else:
                passed = value <= threshold
            checks[metric] = {
                "value": round(value, 4),
                "threshold": threshold,
                "passed": passed,
            }

    all_passed = all(c["passed"] for c in checks.values())

    report = {
        "report_type": "fairness_compliance",
        "regulation": regulation,
        "model_name": model_name,
        "generated_at": now,
        "period": {
            "start": period_start or now,
            "end": period_end or now,
        },
        "sensitive_attributes_monitored": sensitive_attributes,
        "fairness_checks": checks,
        "overall_compliance": "COMPLIANT" if all_passed else "NON-COMPLIANT",
        "recommendations": [],
    }

    if not all_passed:
        failed = [k for k, v in checks.items() if not v["passed"]]
        report["recommendations"] = [
            f"Investigate and remediate: {m}" for m in failed
        ]

    return report


def save_compliance_report(report: dict, output_dir: str) -> Path:
    """Save report to JSON file with timestamp."""
    path = Path(output_dir) / f"compliance_{report['model_name']}_{datetime.utcnow():%Y%m%d_%H%M}.json"
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(json.dumps(report, indent=2))
    return path
```

---

## End-to-End Fairness Monitoring System

### Architecture

```
┌───────────────────────────────────────────────────────────────┐
│           End-to-End Fairness Monitoring Architecture          │
│                                                               │
│  Prediction Request                                           │
│       │                                                       │
│       ▼                                                       │
│  ┌──────────┐    ┌──────────────────────────────────┐         │
│  │  Model   │───►│  Prediction Logger                │         │
│  │  Server  │    │  (prediction + group + timestamp) │         │
│  └──────────┘    └──────────┬───────────────────────┘         │
│                             │                                 │
│                             ▼                                 │
│                  ┌──────────────────────┐                     │
│                  │  Message Queue       │                     │
│                  │  (Kafka / SQS)       │                     │
│                  └──────────┬───────────┘                     │
│                             │                                 │
│              ┌──────────────┼──────────────┐                  │
│              ▼              ▼              ▼                  │
│  ┌───────────────┐ ┌──────────────┐ ┌──────────────┐         │
│  │ Real-time     │ │ Batch        │ │ Compliance   │         │
│  │ Fairness      │ │ Fairness     │ │ Report       │         │
│  │ Calculator    │ │ Evaluator    │ │ Generator    │         │
│  │ (5-min window)│ │ (daily)      │ │ (quarterly)  │         │
│  └──────┬────────┘ └──────┬───────┘ └──────┬───────┘         │
│         │                 │                │                  │
│         ▼                 ▼                ▼                  │
│  ┌──────────┐     ┌──────────────┐  ┌──────────────┐         │
│  │Prometheus│     │  Data Lake   │  │  Compliance  │         │
│  │+ Grafana │     │  (S3/GCS)    │  │  Archive     │         │
│  └──────┬───┘     └──────────────┘  └──────────────┘         │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────┐                                             │
│  │ Alert Manager│                                             │
│  │ (PagerDuty,  │                                             │
│  │  Slack)      │                                             │
│  └──────────────┘                                             │
└───────────────────────────────────────────────────────────────┘
```

### Integration with Model Serving

```python
# serving/fairness_middleware.py
"""FastAPI middleware that logs predictions for fairness monitoring."""
from fastapi import FastAPI, Request
from starlette.middleware.base import BaseHTTPMiddleware
import json
import time

app = FastAPI()


class FairnessLoggingMiddleware(BaseHTTPMiddleware):
    """Middleware to capture predictions and demographic info for fairness."""

    def __init__(self, app, fairness_monitor):
        super().__init__(app)
        self.monitor = fairness_monitor

    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)

        # Extract prediction and group from response context
        if hasattr(request.state, "prediction_context"):
            ctx = request.state.prediction_context
            self.monitor.record(
                group=ctx.get("demographic_group", "unknown"),
                prediction=ctx.get("prediction", 0),
                label=ctx.get("label"),
            )

        return response


# ── Usage in serving code ────────────────────────────────
# from monitoring.fairness_logger import FairnessMonitor
#
# monitor = FairnessMonitor(window_seconds=300)
# app.add_middleware(FairnessLoggingMiddleware, fairness_monitor=monitor)
#
# @app.post("/predict")
# async def predict(request: Request, payload: PredictionRequest):
#     prediction = model.predict(payload.features)
#     request.state.prediction_context = {
#         "prediction": int(prediction),
#         "demographic_group": payload.demographic_group,
#     }
#     return {"prediction": int(prediction)}
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Fairness drift** | Models can become unfair over time due to population shift, feedback loops, and concept drift |
| **Continuous checks** | Rolling-window fairness metrics on live predictions, not just pre-deployment audits |
| **Fairness dashboards** | Grafana + Prometheus for per-group selection rates, DP diff, DI ratio over time |
| **Tiered alerts** | INFO → WARNING → CRITICAL with escalating response (log → Slack → PagerDuty + rollback) |
| **Regulatory reporting** | ECOA, EU AI Act, NYC LL 144 all require different cadences and formats of bias reporting |
| **Architecture** | Prediction logger → message queue → real-time + batch calculators → dashboards + alerts |

> **Rule of thumb:** If you monitor model accuracy in production, you must monitor fairness with equal rigour. A model that degrades for one group while overall metrics stay flat is the most dangerous kind of failure — invisible until it causes real harm.

---

## Revision Questions

1. **Explain** three sources of fairness drift in production. For each, describe what monitoring signal would detect it earliest.

2. **Design** a Grafana dashboard for a loan approval model that serves three demographic groups. What panels would you include, and what alert thresholds would you set?

3. **A model's** overall accuracy stays at 92%, but the TPR gap between groups widens from 0.03 to 0.12 over three months. How would a rolling-window fairness monitor detect this? What alert tier should fire?

4. **Compare** real-time fairness monitoring (5-minute windows) with daily batch evaluation. What are the trade-offs in terms of latency, accuracy, and cost?

5. **Your company** operates under NYC Local Law 144 and the EU AI Act. Design a compliance reporting pipeline that satisfies both. What metrics must you report, how often, and to whom?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Bias Detection](./01-bias-detection.md) | [📂 Responsible AI](./README.md) | [Explainability in Production →](./03-explainability-in-production.md) |
