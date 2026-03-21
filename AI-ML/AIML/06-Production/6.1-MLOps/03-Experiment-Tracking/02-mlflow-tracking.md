# MLflow Tracking

## Overview

**MLflow** is the most popular open-source platform for managing the ML lifecycle. Its **Tracking** component lets you log parameters, metrics, artifacts, and models for every training run. MLflow provides a web UI for comparing runs, a REST API for programmatic access, and integrations with all major ML frameworks. It can be self-hosted or used via Databricks.

---

## MLflow Components

```
  ┌──────────────────────────────────────────────────────────────┐
  │                        MLflow Platform                        │
  │                                                                │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
  │  │  Tracking     │  │  Projects    │  │  Models      │       │
  │  │  Log runs,    │  │  Reproducible│  │  Package &   │       │
  │  │  compare      │  │  ML code     │  │  deploy      │       │
  │  └──────────────┘  └──────────────┘  └──────────────┘       │
  │                                                                │
  │  ┌──────────────┐  ┌──────────────┐                          │
  │  │  Model       │  │  Evaluate    │                          │
  │  │  Registry    │  │  LLM eval,   │                          │
  │  │  Version &   │  │  model eval  │                          │
  │  │  stage       │  │              │                          │
  │  └──────────────┘  └──────────────┘                          │
  │                                                                │
  │  This chapter focuses on TRACKING                            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Setup

```bash
# Install
pip install mlflow

# Start the tracking server (local)
mlflow ui --port 5000

# Or with a database backend (production)
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlflow-artifacts \
  --host 0.0.0.0 --port 5000

# Or with PostgreSQL + S3 (full production)
mlflow server \
  --backend-store-uri postgresql://user:pass@host/mlflow \
  --default-artifact-root s3://my-bucket/mlflow-artifacts \
  --host 0.0.0.0 --port 5000
```

---

## Core Concepts

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Experiment:  A named group of related runs       │
  │               e.g., "churn_prediction"            │
  │                                                    │
  │  Run:         A single training execution         │
  │               Has params, metrics, artifacts      │
  │                                                    │
  │  Parameter:   An input to the run (logged once)   │
  │               e.g., learning_rate=0.001            │
  │                                                    │
  │  Metric:      An output measurement               │
  │               Can be logged multiple times (steps) │
  │               e.g., loss at each epoch             │
  │                                                    │
  │  Artifact:    A file output from the run          │
  │               e.g., model.pkl, plots, data         │
  │                                                    │
  │  Tag:         Metadata label for organizing       │
  │               e.g., "team": "fraud"                │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Complete MLflow Tracking Example

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score
import pandas as pd
import matplotlib.pyplot as plt

# Set tracking server URI (or use local ./mlruns)
mlflow.set_tracking_uri("http://localhost:5000")

# Create or get experiment
mlflow.set_experiment("customer_churn")

# Load data
df = pd.read_csv("data/churn.csv")
X = df.drop("churn", axis=1)
y = df["churn"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Start a tracked run
with mlflow.start_run(run_name="rf_baseline"):
    
    # Define hyperparameters
    params = {
        "n_estimators": 100,
        "max_depth": 10,
        "min_samples_split": 5,
        "random_state": 42,
    }
    
    # Log all parameters
    mlflow.log_params(params)
    
    # Log dataset info
    mlflow.log_param("train_size", len(X_train))
    mlflow.log_param("test_size", len(X_test))
    mlflow.log_param("n_features", X_train.shape[1])
    
    # Train model
    model = RandomForestClassifier(**params)
    model.fit(X_train, y_train)
    
    # Evaluate
    y_pred = model.predict(X_test)
    y_prob = model.predict_proba(X_test)[:, 1]
    
    metrics = {
        "accuracy": accuracy_score(y_test, y_pred),
        "f1_score": f1_score(y_test, y_pred),
        "auc_roc": roc_auc_score(y_test, y_prob),
    }
    
    # Log all metrics
    mlflow.log_metrics(metrics)
    
    # Log metrics at steps (e.g., epoch-level tracking)
    for epoch in range(10):
        mlflow.log_metric("train_loss", 1.0 / (epoch + 1), step=epoch)
    
    # Log feature importance plot as artifact
    importances = model.feature_importances_
    fig, ax = plt.subplots(figsize=(10, 6))
    pd.Series(importances, index=X.columns).sort_values().plot.barh(ax=ax)
    ax.set_title("Feature Importance")
    fig.savefig("feature_importance.png")
    mlflow.log_artifact("feature_importance.png")
    plt.close()
    
    # Log the model itself
    mlflow.sklearn.log_model(
        model, 
        artifact_path="model",
        registered_model_name="churn_classifier",
    )
    
    # Log tags for organization
    mlflow.set_tag("model_type", "random_forest")
    mlflow.set_tag("team", "growth")
    mlflow.set_tag("status", "baseline")
    
    # Log custom artifact (any file)
    with open("run_notes.txt", "w") as f:
        f.write("Baseline RF model with default features")
    mlflow.log_artifact("run_notes.txt")
    
    print(f"Run ID: {mlflow.active_run().info.run_id}")
    print(f"Metrics: {metrics}")
```

---

## Autologging

```python
# MLflow can automatically log params, metrics, and models!
# No manual log_param/log_metric calls needed

import mlflow

# For scikit-learn
mlflow.sklearn.autolog()

# For PyTorch
mlflow.pytorch.autolog()

# For XGBoost
mlflow.xgboost.autolog()

# For TensorFlow/Keras
mlflow.tensorflow.autolog()

# Now just train normally — everything is logged!
from sklearn.ensemble import GradientBoostingClassifier
mlflow.sklearn.autolog()

with mlflow.start_run():
    model = GradientBoostingClassifier(n_estimators=200, max_depth=5)
    model.fit(X_train, y_train)
    # Parameters, metrics, model, and feature importance
    # are ALL automatically logged!
```

---

## Querying Runs Programmatically

```python
import mlflow

client = mlflow.tracking.MlflowClient()

# Search for best run
experiment = client.get_experiment_by_name("customer_churn")
runs = client.search_runs(
    experiment_ids=[experiment.experiment_id],
    filter_string="metrics.f1_score > 0.8",
    order_by=["metrics.f1_score DESC"],
    max_results=5,
)

for run in runs:
    print(f"Run: {run.info.run_id}")
    print(f"  F1: {run.data.metrics['f1_score']:.4f}")
    print(f"  Params: {run.data.params}")
    print()

# Load a model from a specific run
best_run_id = runs[0].info.run_id
model = mlflow.sklearn.load_model(f"runs:/{best_run_id}/model")
```

---

## MLflow UI

```
  The MLflow UI (http://localhost:5000) provides:

  ┌──────────────────────────────────────────────────┐
  │  Experiment View:                                │
  │  ┌────┬───────┬────────┬──────┬──────┬────────┐ │
  │  │ Run│ LR    │ Epochs │ Acc  │ F1   │ Status │ │
  │  ├────┼───────┼────────┼──────┼──────┼────────┤ │
  │  │ #1 │ 0.01  │  10    │ 0.82│ 0.79│ Done   │ │
  │  │ #2 │ 0.001 │  20    │ 0.85│ 0.83│ Done   │ │
  │  │ #3 │ 0.001 │  50    │ 0.87│ 0.85│ Done   │ │
  │  │ #4 │ 0.0001│  50    │ 0.84│ 0.82│ Done   │ │
  │  └────┴───────┴────────┴──────┴──────┴────────┘ │
  │                                                    │
  │  • Sort/filter by any param or metric             │
  │  • Compare selected runs side-by-side             │
  │  • View metric charts over steps/epochs           │
  │  • Download artifacts                             │
  │  • View model details and schema                  │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What are the five main components of the MLflow platform?**
2. **What is the difference between parameters, metrics, and artifacts?**
3. **How does MLflow autologging simplify experiment tracking?**
4. **How do you query MLflow runs programmatically?**
5. **What backend storage options does MLflow support for production?**

---

[Previous: 01-why-track-experiments.md](01-why-track-experiments.md) | [Next: 03-weights-and-biases.md](03-weights-and-biases.md)
