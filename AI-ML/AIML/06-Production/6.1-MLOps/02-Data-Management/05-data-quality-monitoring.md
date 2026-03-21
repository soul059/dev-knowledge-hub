# Data Quality Monitoring

## Overview

Data quality monitoring continuously checks production data for anomalies, drift, and degradation. Unlike one-time validation, monitoring runs **continuously** on live data, detecting issues like distribution shifts, missing value spikes, and schema changes before they impact model performance. Tools like **Evidently AI**, **WhyLabs**, and **Great Expectations** provide automated monitoring dashboards and alerts.

---

## What to Monitor

```
  ┌──────────────────────────────────────────────────────────────┐
  │                   Data Quality Dimensions                     │
  │                                                                │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
  │  │ Completeness │  │  Accuracy    │  │ Consistency  │       │
  │  │ Missing vals │  │ Correct vals │  │ Same format  │       │
  │  │ Null rates   │  │ Valid ranges │  │ No conflicts │       │
  │  └──────────────┘  └──────────────┘  └──────────────┘       │
  │                                                                │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
  │  │  Freshness   │  │  Volume      │  │ Distribution │       │
  │  │ How recent?  │  │ Row counts   │  │ Stats shift? │       │
  │  │ Latency SLA  │  │ Expected vol │  │ Drift detect │       │
  │  └──────────────┘  └──────────────┘  └──────────────┘       │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Data Drift Detection

```
  Data drift = input feature distributions change over time

  Reference (training):         Production (current):
  ┌───────────────────┐        ┌───────────────────┐
  │    ▄▄▄▄            │        │         ▄▄▄▄      │
  │   ██████           │        │       ████████    │
  │  ████████          │        │      ██████████   │
  │ ██████████         │        │     ████████████  │
  │████████████        │        │    ██████████████ │
  │ age: mean=35       │        │    age: mean=52   │
  └───────────────────┘        └───────────────────┘

  The age distribution shifted right → model may not work well!

  Statistical tests for drift:
  ┌─────────────────────┬──────────────────────────────────┐
  │ Test                │ Use case                         │
  ├─────────────────────┼──────────────────────────────────┤
  │ KS test             │ Continuous features (compare CDFs)│
  │ Chi-squared test    │ Categorical features             │
  │ PSI                 │ Population Stability Index       │
  │ Jensen-Shannon div. │ Probability distribution distance│
  │ Wasserstein dist.   │ Earth mover's distance           │
  └─────────────────────┴──────────────────────────────────┘
```

---

## Evidently AI

```
  Open-source ML monitoring framework

  Key features:
  • Data drift detection
  • Data quality reports
  • Model performance monitoring
  • Beautiful HTML reports or JSON for dashboards
  • Integration with Grafana, Airflow, MLflow
```

---

## Python: Monitoring with Evidently

```python
import pandas as pd
from evidently import ColumnMapping
from evidently.report import Report
from evidently.metric_preset import (
    DataDriftPreset,
    DataQualityPreset,
    TargetDriftPreset,
)

# Load reference (training) and current (production) data
reference = pd.read_csv("data/train.csv")
current = pd.read_csv("data/production_batch.csv")

column_mapping = ColumnMapping(
    target="churn",
    prediction="prediction",
    numerical_features=["age", "income", "purchase_count"],
    categorical_features=["region", "plan_type"],
)

# Data Drift Report
drift_report = Report(metrics=[DataDriftPreset()])
drift_report.run(
    reference_data=reference,
    current_data=current,
    column_mapping=column_mapping,
)
drift_report.save_html("reports/data_drift.html")

# Check drift programmatically
drift_results = drift_report.as_dict()
dataset_drift = drift_results["metrics"][0]["result"]["dataset_drift"]
print(f"Dataset drift detected: {dataset_drift}")

# Data Quality Report
quality_report = Report(metrics=[DataQualityPreset()])
quality_report.run(
    reference_data=reference,
    current_data=current,
    column_mapping=column_mapping,
)
quality_report.save_html("reports/data_quality.html")


# Evidently Tests (pass/fail checks)
from evidently.test_suite import TestSuite
from evidently.test_preset import DataDriftTestPreset, DataQualityTestPreset

test_suite = TestSuite(tests=[
    DataDriftTestPreset(),
    DataQualityTestPreset(),
])

test_suite.run(reference_data=reference, current_data=current,
               column_mapping=column_mapping)

# Get pass/fail result
if not test_suite.as_dict()["summary"]["all_passed"]:
    print("❌ Data quality tests FAILED — investigate!")
    # Send alert to Slack / PagerDuty
else:
    print("✅ All data quality tests passed")
```

---

## Building a Monitoring Pipeline

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Production data ──→ Monitor ──→ Dashboard ──→ Alert        │
  │       │                │              │            │          │
  │  (every batch         │         Grafana /       Slack /      │
  │   or streaming)       │         custom UI      PagerDuty    │
  │                       │                                      │
  │                       ▼                                      │
  │              ┌──────────────┐                                │
  │              │  Checks:     │                                │
  │              │  • Drift > θ │                                │
  │              │  • Nulls > 5%│                                │
  │              │  • Volume < X│                                │
  │              │  • Schema Δ  │                                │
  │              └──────┬───────┘                                │
  │                     │                                        │
  │              PASS? ─┤                                        │
  │              YES  → continue pipeline                       │
  │              NO   → alert + optionally halt pipeline        │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: Automated Monitoring Job

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset
import pandas as pd
import json
import requests


def run_monitoring(reference_path, current_path, webhook_url=None):
    """Run data quality monitoring and alert on drift."""
    reference = pd.read_csv(reference_path)
    current = pd.read_csv(current_path)
    
    report = Report(metrics=[DataDriftPreset()])
    report.run(reference_data=reference, current_data=current)
    
    results = report.as_dict()
    drift_detected = results["metrics"][0]["result"]["dataset_drift"]
    drift_share = results["metrics"][0]["result"]["share_of_drifted_columns"]
    
    print(f"Drift detected: {drift_detected}")
    print(f"Drifted columns: {drift_share:.1%}")
    
    # Alert if drift exceeds threshold
    if drift_detected and webhook_url:
        payload = {
            "text": (
                f"⚠️ Data drift detected!\n"
                f"Drifted columns: {drift_share:.1%}\n"
                f"Action: investigate and consider retraining."
            )
        }
        requests.post(webhook_url, json=payload)
    
    # Save report
    report.save_html(f"reports/monitoring_{pd.Timestamp.now().date()}.html")
    
    return drift_detected


# Schedule this to run daily via Airflow/Prefect/cron
run_monitoring(
    "data/reference.csv",
    "data/today_production.csv",
    webhook_url="https://hooks.slack.com/services/XXX"
)
```

---

## Monitoring Dashboard Metrics

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Metric           │ What to track                        │
  ├──────────────────┼──────────────────────────────────────┤
  │ Null rate        │ % nulls per column over time         │
  │ Row count        │ Daily volume (detect drops/spikes)   │
  │ Feature means    │ Rolling average of key features      │
  │ Feature variance │ Stability of distributions           │
  │ Drift score      │ PSI or KS stat per feature           │
  │ Schema changes   │ New/removed/renamed columns          │
  │ Freshness        │ Time since last data update          │
  │ Duplicate rate   │ % duplicate rows                     │
  │ Outlier rate     │ % values beyond 3σ from mean         │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Revision Questions

1. **What is the difference between data validation and data monitoring?**
2. **Name three statistical tests used for drift detection.**
3. **How does Evidently AI help with data quality monitoring?**
4. **What should happen when data drift is detected?**
5. **What metrics should a data monitoring dashboard track?**

---

[Previous: 04-feature-stores.md](04-feature-stores.md) | [Next: ../03-Experiment-Tracking/01-why-track-experiments.md](../03-Experiment-Tracking/01-why-track-experiments.md)
