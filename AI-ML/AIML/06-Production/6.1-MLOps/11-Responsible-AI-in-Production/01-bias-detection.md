# ⚖️ Bias Detection

> **Unit 11 · Responsible AI in Production · Chapter 1**
>
> How to detect and measure bias in ML models — types of bias (selection, measurement, label), statistical fairness metrics (demographic parity, equalized odds, disparate impact), practical bias audits with Fairlearn and AIF360, and building bias detection into your ML pipeline.

---

## Table of Contents

1. [Chapter Overview](#chapter-overview)
2. [Types of Bias in ML Systems](#types-of-bias-in-ml-systems)
3. [Statistical Fairness Metrics](#statistical-fairness-metrics)
4. [Bias Detection with Fairlearn](#bias-detection-with-fairlearn)
5. [Bias Detection with AIF360](#bias-detection-with-aif360)
6. [Building a Bias Audit Pipeline](#building-a-bias-audit-pipeline)
7. [Summary](#summary)
8. [Revision Questions](#revision-questions)
9. [Navigation](#navigation)

---

## Chapter Overview

A model can score 95% accuracy and still systematically harm specific groups. Bias creeps in through data collection, labelling, feature choice, and model optimization. If you don't measure it, you can't fix it. This chapter covers the **types of bias** that affect ML systems, **statistical metrics** used to quantify unfairness, and hands-on Python examples using **Fairlearn** and **AIF360** to audit models before and after deployment.

```
Bias in the ML Lifecycle
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Data Collection  →  Labelling  →  Training  →  Deployment     │
│       │                 │             │             │           │
│  Selection bias    Label bias    Optimization   Feedback       │
│  Measurement bias  Annotator     bias           loop bias      │
│  Sampling bias     disagreement  Proxy features                │
│                                                                │
│  ─────────────  Bias can enter at EVERY stage  ─────────────   │
└────────────────────────────────────────────────────────────────┘
```

---

## Types of Bias in ML Systems

### Taxonomy of Bias

| Bias Type | Definition | Example |
|---|---|---|
| **Selection bias** | Training data doesn't represent the target population | Hiring model trained only on past (mostly male) hires |
| **Measurement bias** | Features are measured differently across groups | Credit scores less reliable for underbanked communities |
| **Label bias** | Labels reflect historical human prejudice | Recidivism labels encoding systemic policing bias |
| **Aggregation bias** | One model for heterogeneous subpopulations | Single diabetes model for all ethnicities |
| **Representation bias** | Under-representation of minority groups in data | Face recognition trained mostly on lighter skin tones |
| **Evaluation bias** | Benchmark doesn't reflect real-world usage | Testing only on English text for a multilingual model |

### How Bias Manifests

```
┌───────────────────────────────────────────────────────────┐
│  Historical Data  ──►  Biased Model  ──►  Biased Outputs  │
│                             │                              │
│                             ▼                              │
│                     Deployed in production                 │
│                             │                              │
│                             ▼                              │
│                     Biased decisions affect people         │
│                             │                              │
│                             ▼                              │
│                     New biased data collected              │
│                             │                              │
│                             ▼                              │
│                     ┌──────────────┐                       │
│                     │  Feedback    │                       │
│                     │  Loop        │◄──── Reinforces       │
│                     │  (amplifies  │      existing bias    │
│                     │   bias)      │                       │
│                     └──────────────┘                       │
└───────────────────────────────────────────────────────────┘
```

### Detecting Bias in Data

```python
# bias_detection/data_audit.py
"""Audit a dataset for representation bias before training."""
import pandas as pd
import numpy as np


def representation_report(
    df: pd.DataFrame,
    sensitive_col: str,
    label_col: str,
) -> pd.DataFrame:
    """Check representation and label rates across groups."""
    report = df.groupby(sensitive_col).agg(
        count=(label_col, "size"),
        positive_rate=(label_col, "mean"),
    )
    report["pct_of_total"] = report["count"] / len(df) * 100
    report["ratio_to_largest"] = report["count"] / report["count"].max()
    return report.round(4)


def detect_label_bias(
    df: pd.DataFrame,
    sensitive_col: str,
    label_col: str,
    threshold: float = 0.8,
) -> dict:
    """Flag groups where positive-label rate ratio falls below threshold."""
    rates = df.groupby(sensitive_col)[label_col].mean()
    max_rate = rates.max()
    ratios = rates / max_rate

    flagged = ratios[ratios < threshold]
    return {
        "group_rates": rates.to_dict(),
        "ratios_to_max": ratios.to_dict(),
        "flagged_groups": flagged.index.tolist(),
        "is_biased": len(flagged) > 0,
    }


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    df = pd.DataFrame({
        "gender": ["M"] * 700 + ["F"] * 300,
        "approved": np.random.binomial(1, 0.7, 700).tolist()
                  + np.random.binomial(1, 0.4, 300).tolist(),
    })

    print("=== Representation Report ===")
    print(representation_report(df, "gender", "approved"))

    print("\n=== Label Bias Detection ===")
    result = detect_label_bias(df, "gender", "approved")
    for k, v in result.items():
        print(f"  {k}: {v}")
```

---

## Statistical Fairness Metrics

### Core Metrics

```
Confusion matrix per group (group = demographic attribute value):

                    Predicted Positive    Predicted Negative
Actual Positive         TP                    FN
Actual Negative         FP                    TN

Key rates:
  TPR (True Positive Rate)  = TP / (TP + FN)  ← "recall"
  FPR (False Positive Rate) = FP / (FP + TN)
  Acceptance Rate           = (TP + FP) / Total
```

| Metric | Formula | Intuition |
|---|---|---|
| **Demographic Parity** | P(Ŷ=1 \| A=a) = P(Ŷ=1 \| A=b) | Equal acceptance rates across groups |
| **Equalized Odds** | TPR and FPR equal across groups | Equal error rates across groups |
| **Equal Opportunity** | TPR equal across groups | Qualified individuals equally likely to be selected |
| **Disparate Impact** | P(Ŷ=1 \| A=minority) / P(Ŷ=1 \| A=majority) ≥ 0.8 | 80% rule from US employment law |
| **Calibration** | P(Y=1 \| Ŷ=p, A=a) = p for all groups | Predicted probabilities match reality per group |
| **Predictive Parity** | PPV equal across groups | Precision equal across groups |

### Metric Relationships

```
┌────────────────────────────────────────────────────────────┐
│              Fairness Metric Decision Tree                  │
│                                                            │
│  What matters most?                                        │
│       │                                                    │
│       ├── Equal selection rates? ──► Demographic Parity    │
│       │                                                    │
│       ├── Equal error rates?                               │
│       │       │                                            │
│       │       ├── Both TPR and FPR? ──► Equalized Odds     │
│       │       └── Only TPR? ──► Equal Opportunity          │
│       │                                                    │
│       ├── Legal compliance? ──► Disparate Impact (80% rule)│
│       │                                                    │
│       └── Calibrated probabilities? ──► Calibration        │
│                                                            │
│  ⚠️  Impossibility theorem: Cannot satisfy all metrics     │
│     simultaneously (except in trivial cases)               │
└────────────────────────────────────────────────────────────┘
```

### Computing Fairness Metrics from Scratch

```python
# bias_detection/fairness_metrics.py
"""Compute core fairness metrics from predictions."""
import numpy as np
from dataclasses import dataclass


@dataclass
class GroupMetrics:
    tpr: float
    fpr: float
    acceptance_rate: float
    ppv: float


def compute_group_metrics(y_true: np.ndarray, y_pred: np.ndarray) -> GroupMetrics:
    """Compute confusion-matrix-derived rates for a single group."""
    tp = np.sum((y_pred == 1) & (y_true == 1))
    fp = np.sum((y_pred == 1) & (y_true == 0))
    fn = np.sum((y_pred == 0) & (y_true == 1))
    tn = np.sum((y_pred == 0) & (y_true == 0))

    tpr = tp / (tp + fn) if (tp + fn) > 0 else 0.0
    fpr = fp / (fp + tn) if (fp + tn) > 0 else 0.0
    acceptance_rate = (tp + fp) / len(y_pred) if len(y_pred) > 0 else 0.0
    ppv = tp / (tp + fp) if (tp + fp) > 0 else 0.0

    return GroupMetrics(tpr=tpr, fpr=fpr, acceptance_rate=acceptance_rate, ppv=ppv)


def demographic_parity_difference(
    y_pred: np.ndarray, sensitive: np.ndarray
) -> float:
    """Max difference in acceptance rate across groups."""
    groups = np.unique(sensitive)
    rates = [y_pred[sensitive == g].mean() for g in groups]
    return max(rates) - min(rates)


def equalized_odds_difference(
    y_true: np.ndarray, y_pred: np.ndarray, sensitive: np.ndarray
) -> dict:
    """Difference in TPR and FPR across groups."""
    groups = np.unique(sensitive)
    metrics = {
        g: compute_group_metrics(y_true[sensitive == g], y_pred[sensitive == g])
        for g in groups
    }
    tprs = [m.tpr for m in metrics.values()]
    fprs = [m.fpr for m in metrics.values()]

    return {
        "tpr_difference": max(tprs) - min(tprs),
        "fpr_difference": max(fprs) - min(fprs),
        "group_metrics": {g: vars(m) for g, m in metrics.items()},
    }


def disparate_impact_ratio(
    y_pred: np.ndarray, sensitive: np.ndarray, reference_group: str
) -> dict:
    """Ratio of acceptance rates (80% rule)."""
    groups = np.unique(sensitive)
    ref_rate = y_pred[sensitive == reference_group].mean()

    ratios = {}
    for g in groups:
        if g == reference_group:
            continue
        g_rate = y_pred[sensitive == g].mean()
        ratios[g] = g_rate / ref_rate if ref_rate > 0 else float("inf")

    return {
        "reference_group": reference_group,
        "reference_rate": ref_rate,
        "ratios": ratios,
        "passes_80pct_rule": all(r >= 0.8 for r in ratios.values()),
    }


# ── Demo ─────────────────────────────────────────────────
if __name__ == "__main__":
    np.random.seed(42)
    n = 1000
    sensitive = np.array(["A"] * 500 + ["B"] * 500)
    y_true = np.random.binomial(1, 0.5, n)
    y_pred = np.where(
        sensitive == "A",
        np.random.binomial(1, 0.7, n),
        np.random.binomial(1, 0.4, n),
    )

    print(f"Demographic parity diff: {demographic_parity_difference(y_pred, sensitive):.4f}")
    print(f"Equalized odds: {equalized_odds_difference(y_true, y_pred, sensitive)}")
    print(f"Disparate impact: {disparate_impact_ratio(y_pred, sensitive, 'A')}")
```

---

## Bias Detection with Fairlearn

### Installation and Setup

```bash
pip install fairlearn scikit-learn pandas
```

### Fairlearn Bias Audit

```python
# bias_detection/fairlearn_audit.py
"""End-to-end bias audit using Fairlearn."""
from fairlearn.metrics import (
    MetricFrame,
    demographic_parity_difference,
    equalized_odds_difference,
    selection_rate,
    true_positive_rate,
    false_positive_rate,
)
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score
import pandas as pd
import numpy as np


def run_bias_audit(
    X: pd.DataFrame,
    y: pd.Series,
    sensitive_features: pd.Series,
    model=None,
) -> dict:
    """
    Train a model and run a comprehensive bias audit.

    Returns dict with overall metrics and per-group breakdowns.
    """
    X_train, X_test, y_train, y_test, sf_train, sf_test = train_test_split(
        X, y, sensitive_features, test_size=0.3, random_state=42
    )

    if model is None:
        model = GradientBoostingClassifier(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)

    # ── Build MetricFrame ────────────────────────────────
    metrics = {
        "accuracy": accuracy_score,
        "selection_rate": selection_rate,
        "true_positive_rate": true_positive_rate,
        "false_positive_rate": false_positive_rate,
    }

    mf = MetricFrame(
        metrics=metrics,
        y_true=y_test,
        y_pred=y_pred,
        sensitive_features=sf_test,
    )

    # ── Compute fairness metrics ─────────────────────────
    dp_diff = demographic_parity_difference(
        y_test, y_pred, sensitive_features=sf_test
    )
    eo_diff = equalized_odds_difference(
        y_test, y_pred, sensitive_features=sf_test
    )

    return {
        "overall": mf.overall.to_dict(),
        "by_group": mf.by_group.to_dict(),
        "difference": mf.difference().to_dict(),
        "demographic_parity_diff": dp_diff,
        "equalized_odds_diff": eo_diff,
        "model": model,
    }


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    np.random.seed(42)
    n = 2000
    df = pd.DataFrame({
        "feature_1": np.random.randn(n),
        "feature_2": np.random.randn(n),
        "feature_3": np.random.randn(n),
        "gender": np.random.choice(["M", "F"], n),
    })
    df["label"] = (
        (df["feature_1"] + df["feature_2"] > 0).astype(int)
        | ((df["gender"] == "M") & (df["feature_3"] > 0)).astype(int)
    )

    result = run_bias_audit(
        X=df[["feature_1", "feature_2", "feature_3"]],
        y=df["label"],
        sensitive_features=df["gender"],
    )

    print("=== Overall Metrics ===")
    for k, v in result["overall"].items():
        print(f"  {k}: {v:.4f}")

    print(f"\n=== Fairness ===")
    print(f"  Demographic parity diff: {result['demographic_parity_diff']:.4f}")
    print(f"  Equalized odds diff:     {result['equalized_odds_diff']:.4f}")

    print(f"\n=== By Group ===")
    for metric, groups in result["by_group"].items():
        print(f"  {metric}: {groups}")
```

### Fairlearn Mitigation (Post-processing)

```python
# bias_detection/fairlearn_mitigation.py
"""Mitigate bias using Fairlearn's ThresholdOptimizer."""
from fairlearn.postprocessing import ThresholdOptimizer
from fairlearn.metrics import demographic_parity_difference
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import numpy as np
import pandas as pd


def mitigate_with_threshold_optimizer(
    X: pd.DataFrame,
    y: pd.Series,
    sensitive_features: pd.Series,
    constraint: str = "demographic_parity",
) -> dict:
    """Apply ThresholdOptimizer to reduce bias."""
    X_tr, X_te, y_tr, y_te, sf_tr, sf_te = train_test_split(
        X, y, sensitive_features, test_size=0.3, random_state=42
    )

    # Train unconstrained model
    base_model = GradientBoostingClassifier(n_estimators=100, random_state=42)
    base_model.fit(X_tr, y_tr)
    y_pred_base = base_model.predict(X_te)

    # Apply ThresholdOptimizer
    mitigator = ThresholdOptimizer(
        estimator=base_model,
        constraints=constraint,
        prefit=True,
    )
    mitigator.fit(X_tr, y_tr, sensitive_features=sf_tr)
    y_pred_mitigated = mitigator.predict(X_te, sensitive_features=sf_te)

    return {
        "before": {
            "accuracy": accuracy_score(y_te, y_pred_base),
            "dp_diff": demographic_parity_difference(
                y_te, y_pred_base, sensitive_features=sf_te
            ),
        },
        "after": {
            "accuracy": accuracy_score(y_te, y_pred_mitigated),
            "dp_diff": demographic_parity_difference(
                y_te, y_pred_mitigated, sensitive_features=sf_te
            ),
        },
    }
```

---

## Bias Detection with AIF360

### Installation

```bash
pip install aif360 scikit-learn
```

### AIF360 Bias Audit

```python
# bias_detection/aif360_audit.py
"""Bias detection using IBM AIF360."""
from aif360.datasets import BinaryLabelDataset
from aif360.metrics import BinaryLabelDatasetMetric, ClassificationMetric
from aif360.algorithms.preprocessing import Reweighing
from sklearn.linear_model import LogisticRegression
import pandas as pd
import numpy as np


def aif360_dataset_audit(
    df: pd.DataFrame,
    label_col: str,
    protected_col: str,
    privileged_value: int = 1,
) -> dict:
    """Audit a dataset for bias using AIF360."""
    dataset = BinaryLabelDataset(
        df=df,
        label_names=[label_col],
        protected_attribute_names=[protected_col],
        privileged_protected_attributes=[[privileged_value]],
    )

    metric = BinaryLabelDatasetMetric(
        dataset,
        unprivileged_groups=[{protected_col: 0}],
        privileged_groups=[{protected_col: privileged_value}],
    )

    return {
        "disparate_impact": metric.disparate_impact(),
        "statistical_parity_difference": metric.statistical_parity_difference(),
        "consistency": metric.consistency().mean(),
        "num_positives_privileged": metric.num_positives(privileged=True),
        "num_positives_unprivileged": metric.num_positives(privileged=False),
        "base_rate_privileged": metric.base_rate(privileged=True),
        "base_rate_unprivileged": metric.base_rate(privileged=False),
    }


def aif360_reweighing(
    df: pd.DataFrame,
    label_col: str,
    protected_col: str,
    privileged_value: int = 1,
) -> pd.DataFrame:
    """Apply reweighing pre-processing to reduce bias."""
    dataset = BinaryLabelDataset(
        df=df,
        label_names=[label_col],
        protected_attribute_names=[protected_col],
        privileged_protected_attributes=[[privileged_value]],
    )

    rw = Reweighing(
        unprivileged_groups=[{protected_col: 0}],
        privileged_groups=[{protected_col: privileged_value}],
    )
    dataset_rw = rw.fit_transform(dataset)

    df_out = df.copy()
    df_out["sample_weight"] = dataset_rw.instance_weights
    return df_out


# ── Usage ────────────────────────────────────────────────
if __name__ == "__main__":
    np.random.seed(42)
    n = 1000
    df = pd.DataFrame({
        "feature_1": np.random.randn(n),
        "protected": np.random.binomial(1, 0.5, n),
    })
    # Inject bias: privileged group gets higher approval rate
    df["label"] = (
        (df["feature_1"] > -0.5).astype(int)
        & ((df["protected"] == 1) | (df["feature_1"] > 0.5)).astype(int)
    )

    audit = aif360_dataset_audit(df, "label", "protected")
    print("=== AIF360 Dataset Audit ===")
    for k, v in audit.items():
        print(f"  {k}: {v:.4f}" if isinstance(v, float) else f"  {k}: {v}")

    df_reweighed = aif360_reweighing(df, "label", "protected")
    print(f"\nSample weights range: [{df_reweighed['sample_weight'].min():.3f}, "
          f"{df_reweighed['sample_weight'].max():.3f}]")
```

---

## Building a Bias Audit Pipeline

### Automated Bias Report

```
┌───────────────────────────────────────────────────────────────┐
│                  Bias Audit Pipeline                           │
│                                                               │
│  Model trained                                                │
│       │                                                       │
│       ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  1. Dataset audit                                       │  │
│  │     • Representation check per group                    │  │
│  │     • Label rate check per group                        │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │  2. Model prediction audit                              │  │
│  │     • Demographic parity                                │  │
│  │     • Equalized odds                                    │  │
│  │     • Disparate impact ratio                            │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │  3. Intersectional analysis                             │  │
│  │     • Cross-cut sensitive attributes (age × gender)     │  │
│  │     • Find worst-performing subgroups                   │  │
│  ├─────────────────────────────────────────────────────────┤  │
│  │  4. Generate report                                     │  │
│  │     • Pass/fail against thresholds                      │  │
│  │     • Recommendations                                   │  │
│  │     • Attach to model registry                          │  │
│  └─────────────────────────────────────────────────────────┘  │
│       │                                                       │
│       ▼                                                       │
│  ┌──────────────┐                                             │
│  │ Pass? ──► Deploy                                           │
│  │ Fail? ──► Block + alert                                    │
│  └──────────────┘                                             │
└───────────────────────────────────────────────────────────────┘
```

### Bias Gate in CI/CD

```python
# ci/bias_gate.py
"""Quality gate that blocks deployment if bias thresholds are exceeded."""
import json
import sys
from pathlib import Path
from fairlearn.metrics import (
    MetricFrame,
    demographic_parity_difference,
    equalized_odds_difference,
    selection_rate,
)
import numpy as np


THRESHOLDS = {
    "demographic_parity_diff": 0.10,
    "equalized_odds_diff": 0.10,
    "min_group_selection_rate": 0.05,
    "disparate_impact_ratio": 0.80,
}


def run_bias_gate(
    y_true: np.ndarray,
    y_pred: np.ndarray,
    sensitive: np.ndarray,
    thresholds: dict = THRESHOLDS,
) -> dict:
    """Run bias checks and return pass/fail with details."""
    dp_diff = demographic_parity_difference(y_true, y_pred, sensitive_features=sensitive)
    eo_diff = equalized_odds_difference(y_true, y_pred, sensitive_features=sensitive)

    mf = MetricFrame(
        metrics={"selection_rate": selection_rate},
        y_true=y_true,
        y_pred=y_pred,
        sensitive_features=sensitive,
    )

    min_rate = mf.by_group["selection_rate"].min()
    max_rate = mf.by_group["selection_rate"].max()
    di_ratio = min_rate / max_rate if max_rate > 0 else 0.0

    checks = {
        "demographic_parity_diff": {
            "value": dp_diff,
            "threshold": thresholds["demographic_parity_diff"],
            "passed": dp_diff <= thresholds["demographic_parity_diff"],
        },
        "equalized_odds_diff": {
            "value": eo_diff,
            "threshold": thresholds["equalized_odds_diff"],
            "passed": eo_diff <= thresholds["equalized_odds_diff"],
        },
        "min_group_selection_rate": {
            "value": min_rate,
            "threshold": thresholds["min_group_selection_rate"],
            "passed": min_rate >= thresholds["min_group_selection_rate"],
        },
        "disparate_impact_ratio": {
            "value": di_ratio,
            "threshold": thresholds["disparate_impact_ratio"],
            "passed": di_ratio >= thresholds["disparate_impact_ratio"],
        },
    }

    all_passed = all(c["passed"] for c in checks.values())

    return {"passed": all_passed, "checks": checks}


if __name__ == "__main__":
    # Load predictions and sensitive features from saved files
    data = np.load("evaluation/bias_data.npz")
    result = run_bias_gate(data["y_true"], data["y_pred"], data["sensitive"])

    print("=== Bias Gate Results ===")
    for name, check in result["checks"].items():
        status = "✅ PASS" if check["passed"] else "❌ FAIL"
        print(f"  {status}  {name}: {check['value']:.4f} (threshold: {check['threshold']})")

    if not result["passed"]:
        print("\n❌ Bias gate FAILED — deployment blocked.")
        sys.exit(1)
    print("\n✅ Bias gate passed.")
```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| **Selection bias** | Training data doesn't represent the target population; check group distributions |
| **Measurement bias** | Features measured differently across groups can encode hidden discrimination |
| **Label bias** | Historical labels can bake in past prejudice; audit label rates per group |
| **Demographic parity** | Equal acceptance rates across groups — simplest but strictest metric |
| **Equalized odds** | Equal TPR and FPR across groups — balances accuracy and fairness |
| **Disparate impact** | 80% rule from employment law — ratio of selection rates must be ≥ 0.8 |
| **Fairlearn** | Microsoft's toolkit for computing metrics (MetricFrame) and mitigation |
| **AIF360** | IBM's toolkit with dataset metrics, bias mitigation algorithms, and explanations |
| **Bias gate** | Automated CI/CD check that blocks deployment if fairness thresholds are violated |

> **Rule of thumb:** Measure bias across at least three metrics — demographic parity, equalized odds, and disparate impact. No single metric captures all dimensions of fairness, and the right metric depends on your use case and regulatory context.

---

## Revision Questions

1. **Explain** the difference between selection bias, measurement bias, and label bias. Give a concrete example of each in a loan approval system.

2. **A model** has TPR of 0.90 for group A and 0.65 for group B, but equal FPR of 0.10 for both. Does it satisfy equalized odds? Does it satisfy equal opportunity? What would you recommend?

3. **Compare** Fairlearn's `MetricFrame` with AIF360's `BinaryLabelDatasetMetric`. When would you choose one library over the other?

4. **Design** a bias audit pipeline for a hiring model. What metrics would you track, what thresholds would you set, and how would you integrate it into CI/CD?

5. **The impossibility theorem** states you cannot satisfy demographic parity, equalized odds, and calibration simultaneously (except in trivial cases). How would you decide which metric to prioritize for a criminal justice risk assessment tool?

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← CI/CD for ML](../10-ML-Testing/06-cicd-for-ml.md) | [📂 Responsible AI](./README.md) | [Fairness Monitoring →](./02-fairness-monitoring.md) |
