# Automated Training

## Overview

Automated training removes manual steps from the model training process — automatically triggering retraining when data changes, drift is detected, or on a schedule. A fully automated training pipeline ingests new data, validates it, trains the model, evaluates against quality gates, and promotes the model to the registry without human intervention.

---

## Trigger Types

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Trigger          │ When it fires                        │
  ├──────────────────┼──────────────────────────────────────┤
  │ Scheduled        │ Cron-based (daily, weekly, monthly)  │
  │                  │ Simplest, most common starting point │
  ├──────────────────┼──────────────────────────────────────┤
  │ Data-driven      │ New data batch arrives               │
  │                  │ Data volume exceeds threshold        │
  ├──────────────────┼──────────────────────────────────────┤
  │ Drift-driven     │ Monitoring detects data/concept drift│
  │                  │ Performance drops below threshold    │
  ├──────────────────┼──────────────────────────────────────┤
  │ On-demand        │ Manual trigger by ML engineer        │
  │                  │ API call from another system         │
  ├──────────────────┼──────────────────────────────────────┤
  │ Event-driven     │ Upstream pipeline completes          │
  │                  │ Feature store updated                │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Automated Training Pipeline

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Trigger ──► Data     ──► Validate ──► Feature ──► Train    │
  │              Fetch         Data         Eng.       Model    │
  │                                                     │        │
  │                                                     ▼        │
  │            ┌──────────────────────────────────────────────┐  │
  │            │         Quality Gate                         │  │
  │            │  • Accuracy > threshold?                     │  │
  │            │  • F1 > minimum?                            │  │
  │            │  • No fairness regression?                  │  │
  │            │  • Latency within bounds?                   │  │
  │            │  • Better than current production model?    │  │
  │            └────────┬───────────────────┬────────────────┘  │
  │                     │                   │                    │
  │                 PASS ▼               FAIL ▼                  │
  │            Register model         Alert team               │
  │            in Model Registry      Log failure              │
  │            Auto-deploy (optional)  Keep current model      │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: Automated Training Script

```python
import mlflow
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import f1_score, accuracy_score
import pandas as pd
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Configuration
EXPERIMENT_NAME = "churn_automated"
MODEL_NAME = "churn_classifier"
F1_THRESHOLD = 0.80
ACCURACY_THRESHOLD = 0.82


def get_current_production_f1() -> float:
    """Get F1 score of the current production model."""
    client = mlflow.tracking.MlflowClient()
    try:
        versions = client.get_latest_versions(MODEL_NAME, stages=["Production"])
        if versions:
            run = client.get_run(versions[0].run_id)
            return run.data.metrics.get("f1_score", 0.0)
    except Exception:
        pass
    return 0.0


def automated_training():
    """Full automated training pipeline."""
    mlflow.set_experiment(EXPERIMENT_NAME)
    
    # Step 1: Load and validate data
    logger.info("Loading data...")
    df = pd.read_parquet("data/latest/features.parquet")
    assert len(df) > 5000, f"Insufficient data: {len(df)} rows"
    
    X = df.drop("target", axis=1)
    y = df["target"]
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, stratify=y, random_state=42
    )
    
    # Step 2: Train
    logger.info("Training model...")
    with mlflow.start_run(run_name="automated_retrain") as run:
        params = {
            "n_estimators": 300,
            "max_depth": 8,
            "learning_rate": 0.05,
            "subsample": 0.8,
        }
        mlflow.log_params(params)
        mlflow.log_param("train_rows", len(X_train))
        mlflow.set_tag("trigger", "scheduled")
        
        model = GradientBoostingClassifier(**params)
        model.fit(X_train, y_train)
        
        # Step 3: Evaluate
        y_pred = model.predict(X_test)
        metrics = {
            "accuracy": accuracy_score(y_test, y_pred),
            "f1_score": f1_score(y_test, y_pred),
        }
        mlflow.log_metrics(metrics)
        logger.info(f"Metrics: {metrics}")
        
        # Step 4: Quality gate
        current_prod_f1 = get_current_production_f1()
        
        passes_gate = (
            metrics["f1_score"] >= F1_THRESHOLD
            and metrics["accuracy"] >= ACCURACY_THRESHOLD
            and metrics["f1_score"] >= current_prod_f1
        )
        
        if passes_gate:
            logger.info("✅ Quality gate PASSED — registering model")
            mlflow.sklearn.log_model(
                model, "model",
                registered_model_name=MODEL_NAME,
            )
            mlflow.set_tag("status", "promoted")
        else:
            logger.warning(
                f"❌ Quality gate FAILED — "
                f"F1={metrics['f1_score']:.3f} "
                f"(threshold={F1_THRESHOLD}, prod={current_prod_f1:.3f})"
            )
            mlflow.set_tag("status", "rejected")


if __name__ == "__main__":
    automated_training()
```

---

## Scheduling Strategies

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Conservative: retrain weekly/monthly                        │
  │    + Low compute cost                                        │
  │    + Simple to manage                                        │
  │    - May miss fast-moving drift                              │
  │                                                                │
  │  Moderate: retrain daily + on drift detection                │
  │    + Good balance of freshness and cost                      │
  │    + Catches most drift                                      │
  │    - Requires monitoring infrastructure                      │
  │                                                                │
  │  Aggressive: retrain on every data batch                    │
  │    + Always up-to-date model                                 │
  │    - High compute cost                                       │
  │    - Risk of training on bad data                            │
  │    - Requires robust validation                              │
  │                                                                │
  │  Online learning: update model continuously                  │
  │    + Real-time adaptation                                    │
  │    - Complex to implement and monitor                        │
  │    - Stability concerns                                      │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What are the five trigger types for automated retraining?**
2. **What checks should a quality gate include?**
3. **Why should the new model be compared against the current production model?**
4. **Compare conservative vs. aggressive retraining schedules.**

---

[Previous: 03-apache-airflow-for-ml.md](03-apache-airflow-for-ml.md) | [Next: 05-distributed-training.md](05-distributed-training.md)
