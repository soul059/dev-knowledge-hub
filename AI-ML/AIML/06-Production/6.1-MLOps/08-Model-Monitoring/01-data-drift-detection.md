# 📊 Data Drift Detection

> **Unit 8 · Chapter 1** — Detecting shifts in input data distributions: statistical tests (KS, PSI, chi-squared), Python implementation with Evidently, and building drift monitoring dashboards.

---

## 📋 Chapter Overview

Production ML models are trained on a **snapshot of reality**. As the world changes, the data flowing into your model inevitably diverges from the training distribution — this is **data drift**. A model that was 95% accurate at deployment can silently degrade to 70% accuracy without a single line of code changing, simply because the inputs no longer resemble what the model learned from.

Data drift detection is the **early warning system** for ML systems. By catching distribution shifts before they impact predictions, teams can trigger retraining, investigate root causes, and maintain model reliability.

This chapter covers:

- What data drift is and why it matters
- Statistical tests for detecting drift (KS test, PSI, chi-squared)
- Python implementation with the Evidently library
- Building drift monitoring dashboards
- Setting detection thresholds and automation

---

## 🔍 What Is Data Drift?

```
  TRAINING TIME                          PRODUCTION (months later)
  ─────────────                          ────────────────────────

  Feature: "age"                         Feature: "age"

  Frequency                              Frequency
  │                                      │
  │    ██                                │              ██
  │   ████                               │            ████
  │  ██████                              │           ██████
  │ ████████                             │         ████████
  │██████████                            │        ██████████
  └──────────── age                      └──────────────── age
    20  30  40  50                          30  40  50  60  70

  Mean: 35, Std: 8                       Mean: 48, Std: 10
                                         ⚠️  DISTRIBUTION HAS SHIFTED
```

### Types of Data Drift

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    DATA DRIFT TAXONOMY                           │
  ├──────────────────┬───────────────────┬───────────────────────────┤
  │                  │                   │                           │
  │  COVARIATE SHIFT │  PRIOR PROB SHIFT │  FEATURE SCHEMA DRIFT    │
  │  ────────────── │  ─────────────── │  ──────────────────────   │
  │  P(X) changes    │  P(Y) changes     │  Features added/removed  │
  │  P(Y|X) stable   │  P(X|Y) stable    │  Types/ranges change     │
  │                  │                   │                           │
  │  Example:        │  Example:         │  Example:                │
  │  User age dist   │  Fraud rate goes  │  New column added by     │
  │  shifts older    │  from 1% to 5%    │  upstream data pipeline  │
  │                  │                   │                           │
  └──────────────────┴───────────────────┴───────────────────────────┘
```

---

## 📈 Statistical Tests for Drift Detection

### Kolmogorov-Smirnov (KS) Test

The KS test measures the **maximum distance** between two cumulative distribution functions. It works on continuous numerical features.

```
  CDF
  1.0 ┤                              ╭──── Reference (training)
      │                          ╭───╯
      │                     ╭────╯
      │                ╭────╯
  0.5 ┤           ╭────╯          D_max = 0.23
      │      ╭────╯       ◄──── Maximum vertical
      │ ╭────╯                   distance between
      │╭╯                        the two CDFs
  0.0 ┤╯
      └────────────────────────────── Feature value

                              ╭──── Current (production)
                         ╭────╯
                    ╭────╯
              ╭─────╯
         ╭────╯
    ╭────╯
  ──╯

  If D_max > critical value → reject H₀ (distributions are same)
  Typical threshold: p-value < 0.05
```

### Population Stability Index (PSI)

PSI quantifies how much a distribution has shifted by comparing **binned proportions** between reference and current data.

```
  PSI = Σ (Pᵢ - Qᵢ) × ln(Pᵢ / Qᵢ)

  Where:
    Pᵢ = proportion of current data in bin i
    Qᵢ = proportion of reference data in bin i

  ┌─────────────────────────────────────────────┐
  │  PSI Interpretation                         │
  ├──────────────┬──────────────────────────────┤
  │  PSI < 0.1   │  ✅ No significant drift      │
  │  0.1 ≤ PSI   │  ⚠️ Moderate drift — monitor  │
  │    < 0.2     │                              │
  │  PSI ≥ 0.2   │  🚨 Significant drift — act   │
  └──────────────┴──────────────────────────────┘
```

### Chi-Squared Test

Best for **categorical features**. Compares observed vs expected frequencies across categories.

```
  χ² = Σ (Observed - Expected)² / Expected

  Example: Feature "payment_method"
  ┌─────────────┬───────────┬───────────┬──────────────┐
  │ Category    │ Training  │ Production│ Contribution │
  │             │ (Expected)│ (Observed)│ to χ²        │
  ├─────────────┼───────────┼───────────┼──────────────┤
  │ Credit Card │ 45%       │ 42%       │ 0.20         │
  │ Debit Card  │ 30%       │ 28%       │ 0.13         │
  │ Digital     │ 15%       │ 22%       │ 3.27 ⚠️      │
  │ Cash        │ 10%       │  8%       │ 0.40         │
  └─────────────┴───────────┴───────────┴──────────────┘
  χ² = 4.00, df = 3, p-value = 0.26 → No significant drift
```

### Test Comparison

| Test | Feature Type | Strengths | Weaknesses | When to Use |
|------|-------------|-----------|------------|-------------|
| **KS Test** | Continuous | Non-parametric, no binning needed | Only univariate, less sensitive to tail changes | Single numeric feature monitoring |
| **PSI** | Continuous / Ordinal | Intuitive thresholds, industry standard | Sensitive to bin choice, can miss shape changes | Batch monitoring, regulatory reporting |
| **Chi-Squared** | Categorical | Well-understood, handles multi-class | Requires sufficient samples per category | Categorical feature monitoring |
| **Wasserstein** | Continuous | Captures magnitude of shift | Computationally expensive | When shift magnitude matters |
| **Jensen-Shannon** | Any (binned) | Symmetric, bounded [0,1] | Requires binning for continuous | General-purpose drift scoring |

---

## 🏗️ Drift Detection Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                    DRIFT DETECTION PIPELINE                      │
  │                                                                  │
  │  ┌────────────┐    ┌─────────────┐    ┌───────────────────────┐ │
  │  │ Production │    │  Feature    │    │  Reference Dataset    │ │
  │  │ Traffic    │───▶│  Store /   │    │  (training snapshot)  │ │
  │  │            │    │  Logger     │    │                       │ │
  │  └────────────┘    └──────┬──────┘    └───────────┬───────────┘ │
  │                           │                       │             │
  │                           ▼                       ▼             │
  │                    ┌──────────────────────────────────┐         │
  │                    │       DRIFT DETECTOR             │         │
  │                    │  ┌───────┐ ┌─────┐ ┌─────────┐  │         │
  │                    │  │  KS   │ │ PSI │ │ Chi-Sq  │  │         │
  │                    │  │ Test  │ │     │ │  Test   │  │         │
  │                    │  └───┬───┘ └──┬──┘ └────┬────┘  │         │
  │                    │      └────────┼─────────┘       │         │
  │                    └───────────────┼─────────────────┘         │
  │                                    │                           │
  │                                    ▼                           │
  │                    ┌──────────────────────────────┐            │
  │                    │        DRIFT REPORT          │            │
  │                    │  • Per-feature drift scores  │            │
  │                    │  • Overall drift status      │            │
  │                    │  • Visualization / dashboard │            │
  │                    └──────────────┬───────────────┘            │
  │                                   │                            │
  │                    ┌──────────────┼──────────────┐             │
  │                    ▼              ▼              ▼             │
  │              ┌──────────┐  ┌──────────┐  ┌────────────┐       │
  │              │  Alert   │  │ Dashboard│  │  Retrain   │       │
  │              │  System  │  │  Update  │  │  Trigger   │       │
  │              └──────────┘  └──────────┘  └────────────┘       │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 🐍 Python Implementation

### Manual Statistical Tests

```python
import numpy as np
from scipy import stats
from typing import Dict, Tuple


def ks_test_drift(
    reference: np.ndarray, current: np.ndarray, threshold: float = 0.05
) -> Dict:
    """Detect drift using the Kolmogorov-Smirnov test."""
    statistic, p_value = stats.ks_2samp(reference, current)
    return {
        "test": "KS",
        "statistic": round(statistic, 4),
        "p_value": round(p_value, 4),
        "drift_detected": p_value < threshold,
        "threshold": threshold,
    }


def calculate_psi(
    reference: np.ndarray, current: np.ndarray, bins: int = 10
) -> Dict:
    """Calculate Population Stability Index."""
    # Create bins from reference distribution
    breakpoints = np.percentile(reference, np.linspace(0, 100, bins + 1))
    breakpoints[0] = -np.inf
    breakpoints[-1] = np.inf

    ref_counts = np.histogram(reference, bins=breakpoints)[0]
    cur_counts = np.histogram(current, bins=breakpoints)[0]

    # Convert to proportions (add small epsilon to avoid division by zero)
    eps = 1e-6
    ref_proportions = ref_counts / len(reference) + eps
    cur_proportions = cur_counts / len(current) + eps

    psi = np.sum(
        (cur_proportions - ref_proportions)
        * np.log(cur_proportions / ref_proportions)
    )

    return {
        "test": "PSI",
        "psi_value": round(psi, 4),
        "drift_detected": psi >= 0.2,
        "severity": (
            "none" if psi < 0.1
            else "moderate" if psi < 0.2
            else "significant"
        ),
    }


def chi_squared_drift(
    reference: np.ndarray, current: np.ndarray, threshold: float = 0.05
) -> Dict:
    """Detect drift in categorical features using chi-squared test."""
    # Get unique categories from both arrays
    categories = np.union1d(reference, current)

    ref_counts = np.array(
        [np.sum(reference == cat) for cat in categories], dtype=float
    )
    cur_counts = np.array(
        [np.sum(current == cat) for cat in categories], dtype=float
    )

    # Normalize to same total
    ref_counts = ref_counts * (len(current) / len(reference))

    statistic, p_value = stats.chisquare(cur_counts, f_exp=ref_counts)
    return {
        "test": "Chi-Squared",
        "statistic": round(statistic, 4),
        "p_value": round(p_value, 4),
        "drift_detected": p_value < threshold,
    }


# --- Usage example ---
np.random.seed(42)
reference_data = np.random.normal(loc=50, scale=10, size=10000)
current_data = np.random.normal(loc=55, scale=12, size=10000)  # Shifted!

print("KS Test:", ks_test_drift(reference_data, current_data))
# KS Test: {'test': 'KS', 'statistic': 0.1842, 'p_value': 0.0, 'drift_detected': True}

print("PSI:", calculate_psi(reference_data, current_data))
# PSI: {'test': 'PSI', 'psi_value': 0.1137, 'drift_detected': False, 'severity': 'moderate'}
```

### Drift Detection with Evidently

```python
import pandas as pd
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset
from evidently.metrics import (
    DataDriftTable,
    DatasetDriftMetric,
    ColumnDriftMetric,
)


def run_drift_report(
    reference_df: pd.DataFrame,
    current_df: pd.DataFrame,
    output_path: str = "drift_report.html",
) -> dict:
    """Generate a comprehensive drift report using Evidently."""
    report = Report(metrics=[
        DataDriftPreset(),
    ])

    report.run(reference_data=reference_df, current_data=current_df)

    # Save interactive HTML dashboard
    report.save_html(output_path)
    print(f"Drift report saved to {output_path}")

    # Extract results programmatically
    result = report.as_dict()
    return result


def check_column_drift(
    reference_df: pd.DataFrame,
    current_df: pd.DataFrame,
    column_name: str,
) -> dict:
    """Check drift for a specific column."""
    report = Report(metrics=[
        ColumnDriftMetric(column_name=column_name),
    ])

    report.run(reference_data=reference_df, current_data=current_df)
    result = report.as_dict()

    drift_result = result["metrics"][0]["result"]
    return {
        "column": column_name,
        "drift_detected": drift_result["drift_detected"],
        "drift_score": drift_result["drift_score"],
        "stattest_name": drift_result["stattest_name"],
    }


# --- Usage example ---
reference_df = pd.DataFrame({
    "age": np.random.normal(35, 8, 5000),
    "income": np.random.lognormal(10.5, 0.8, 5000),
    "transaction_amount": np.random.exponential(50, 5000),
    "category": np.random.choice(
        ["A", "B", "C", "D"], 5000, p=[0.4, 0.3, 0.2, 0.1]
    ),
})

current_df = pd.DataFrame({
    "age": np.random.normal(42, 10, 5000),        # Shifted!
    "income": np.random.lognormal(10.5, 0.8, 5000),  # Stable
    "transaction_amount": np.random.exponential(65, 5000),  # Shifted!
    "category": np.random.choice(
        ["A", "B", "C", "D"], 5000, p=[0.25, 0.25, 0.3, 0.2]  # Shifted!
    ),
})

# Full drift report
results = run_drift_report(reference_df, current_df)

# Per-column check
for col in ["age", "income", "transaction_amount", "category"]:
    print(check_column_drift(reference_df, current_df, col))
```

### Automated Drift Monitoring Job

```python
import schedule
import time
import json
import logging
from datetime import datetime, timedelta
from typing import Optional

logger = logging.getLogger("drift_monitor")


class DriftMonitor:
    """Scheduled drift monitoring with alerting."""

    def __init__(
        self,
        reference_df: pd.DataFrame,
        data_fetcher,            # Callable that returns current DataFrame
        alert_callback=None,     # Callable for sending alerts
        drift_threshold: float = 0.5,  # Fraction of features drifted
    ):
        self.reference_df = reference_df
        self.data_fetcher = data_fetcher
        self.alert_callback = alert_callback
        self.drift_threshold = drift_threshold
        self.history = []

    def run_check(self) -> dict:
        """Execute a single drift check."""
        current_df = self.data_fetcher()
        timestamp = datetime.utcnow().isoformat()

        report = Report(metrics=[DatasetDriftMetric()])
        report.run(
            reference_data=self.reference_df,
            current_data=current_df,
        )

        result = report.as_dict()
        drift_info = result["metrics"][0]["result"]

        check_result = {
            "timestamp": timestamp,
            "dataset_drift": drift_info["dataset_drift"],
            "drift_share": drift_info["drift_share"],
            "number_of_drifted_columns": drift_info[
                "number_of_drifted_columns"
            ],
            "number_of_columns": drift_info["number_of_columns"],
        }

        self.history.append(check_result)
        logger.info(f"Drift check: {json.dumps(check_result)}")

        # Alert if drift exceeds threshold
        if drift_info["drift_share"] > self.drift_threshold:
            msg = (
                f"🚨 Dataset drift detected! "
                f"{drift_info['number_of_drifted_columns']}/"
                f"{drift_info['number_of_columns']} features drifted "
                f"(share: {drift_info['drift_share']:.1%})"
            )
            logger.warning(msg)
            if self.alert_callback:
                self.alert_callback(msg)

        return check_result

    def start_scheduled(self, interval_minutes: int = 60):
        """Run drift checks on a schedule."""
        schedule.every(interval_minutes).minutes.do(self.run_check)
        logger.info(
            f"Drift monitor started (every {interval_minutes} min)"
        )
        while True:
            schedule.run_pending()
            time.sleep(1)
```

---

## 📊 Dashboard Example

```
  ╔════════════════════════════════════════════════════════════════╗
  ║                 DATA DRIFT DASHBOARD                          ║
  ║  Model: fraud_detector_v3    Period: Last 7 days              ║
  ╠════════════════════════════════════════════════════════════════╣
  ║                                                                ║
  ║  Overall Status: ⚠️  MODERATE DRIFT (3/12 features drifted)    ║
  ║                                                                ║
  ║  ┌─────────────────────────────────────────────────────────┐  ║
  ║  │ Feature            │ PSI    │ Status │ Trend            │  ║
  ║  ├────────────────────┼────────┼────────┼──────────────────┤  ║
  ║  │ transaction_amount │ 0.312  │ 🚨     │ ▁▂▃▅▇ ↑         │  ║
  ║  │ user_age           │ 0.187  │ ⚠️     │ ▁▂▃▃▄ ↑         │  ║
  ║  │ merchant_category  │ 0.156  │ ⚠️     │ ▁▁▂▃▃ ↑         │  ║
  ║  │ device_type        │ 0.089  │ ✅     │ ▂▂▂▂▂ →         │  ║
  ║  │ time_of_day        │ 0.045  │ ✅     │ ▁▁▁▁▂ →         │  ║
  ║  │ ip_country         │ 0.023  │ ✅     │ ▁▁▁▁▁ →         │  ║
  ║  └─────────────────────────────────────────────────────────┘  ║
  ║                                                                ║
  ║  PSI Over Time (last 30 days):                                ║
  ║  0.4│                                            ╭──          ║
  ║     │                                       ╭────╯            ║
  ║  0.2│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─╭────╯ ─ ─ THRESHOLD  ║
  ║     │                             ╭────╯                      ║
  ║  0.0│──────────────────────────────╯                          ║
  ║     └──────────────────────────────────────── days            ║
  ║       1    5    10   15   20   25   30                        ║
  ╚════════════════════════════════════════════════════════════════╝
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Data drift is inevitable** — production data will diverge from training data over time; the question is when and how much. |
| 2 | **Choose the right test for your feature type** — KS/PSI for continuous features, chi-squared for categorical features. |
| 3 | **PSI thresholds are industry standard** — < 0.1 (stable), 0.1–0.2 (investigate), ≥ 0.2 (action required). |
| 4 | **Automate monitoring** — scheduled drift checks with alerting catch problems before users do. |
| 5 | **Drift ≠ degradation** — drift signals a risk, but always validate with performance metrics before retraining. |

---

## ❓ Revision Questions

1. **What is the difference between covariate shift and prior probability shift?**
   > Covariate shift means P(X) changes while P(Y|X) stays the same — the input distribution shifts but the relationship between inputs and outputs is preserved. Prior probability shift means P(Y) changes — the distribution of the target variable shifts (e.g., fraud rate increases from 1% to 5%).

2. **When would you use PSI over the KS test for drift detection?**
   > PSI is preferred for batch monitoring workflows and regulatory reporting because it produces an intuitive single score with well-established thresholds (< 0.1, 0.1–0.2, ≥ 0.2). The KS test is better for real-time univariate monitoring where you need a p-value for statistical significance testing.

3. **Why do you add a small epsilon when calculating PSI?**
   > PSI involves computing ln(Pᵢ / Qᵢ). If any bin has zero observations in either distribution, this results in a division by zero or log(0), which is undefined. Adding epsilon (e.g., 1e-6) prevents numerical errors while having negligible impact on the PSI value.

4. **How does Evidently determine which statistical test to apply to each feature?**
   > Evidently automatically selects the appropriate test based on feature type and sample size. For numerical features with large samples, it typically uses the Wasserstein distance or KS test. For categorical features, it uses chi-squared or Jensen-Shannon divergence. Users can override these defaults.

5. **If drift is detected, should you immediately retrain the model?**
   > Not necessarily. Drift detection signals a distribution shift, but the model may still perform adequately. First, check if performance metrics (accuracy, F1, etc.) have actually degraded. Investigate the root cause — it could be a data pipeline bug, a seasonal pattern, or a genuine shift. Only retrain if performance is impacted and the drift is persistent, not transient.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Rollback Strategies](../07-Model-Deployment/06-rollback-strategies.md) | [📂 Model Monitoring](./README.md) | [Concept Drift →](./02-concept-drift.md) |

---

*Unit 8 · Chapter 1 · Data Drift Detection — MLOps Course Notes*
