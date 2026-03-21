# Model Lineage

## Overview

Model lineage traces the complete provenance chain of a model — from raw data through transformations, feature engineering, training, and deployment. It answers: *"How was this model created?"* and *"What would be affected if this data source changes?"* Lineage is essential for debugging, compliance, and impact analysis.

---

## The Lineage Chain

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Raw Data ──► Transform ──► Features ──► Model ──► Serving  │
  │                                                                │
  │  Full lineage for "fraud_detector v4":                       │
  │                                                                │
  │  ┌───────────────────┐                                       │
  │  │ Data Source        │                                       │
  │  │ transactions DB    │                                       │
  │  │ Date: 2024-01-2025 │                                       │
  │  └────────┬──────────┘                                       │
  │           ▼                                                   │
  │  ┌───────────────────┐                                       │
  │  │ Data Version       │                                       │
  │  │ DVC: abc123        │                                       │
  │  │ 2.3M rows          │                                       │
  │  └────────┬──────────┘                                       │
  │           ▼                                                   │
  │  ┌───────────────────┐                                       │
  │  │ Feature Pipeline   │                                       │
  │  │ Git: commit def456│                                       │
  │  │ 47 features        │                                       │
  │  └────────┬──────────┘                                       │
  │           ▼                                                   │
  │  ┌───────────────────┐                                       │
  │  │ Training Run       │                                       │
  │  │ MLflow: run_789    │                                       │
  │  │ XGBoost, F1=0.92  │                                       │
  │  └────────┬──────────┘                                       │
  │           ▼                                                   │
  │  ┌───────────────────┐                                       │
  │  │ Model Version      │                                       │
  │  │ fraud_detector v4  │                                       │
  │  │ Stage: Production  │                                       │
  │  └────────┬──────────┘                                       │
  │           ▼                                                   │
  │  ┌───────────────────┐                                       │
  │  │ Deployment         │                                       │
  │  │ API endpoint v2    │                                       │
  │  │ Region: us-east-1  │                                       │
  │  └───────────────────┘                                       │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Why Lineage Matters

```
  1. DEBUGGING
     "Model accuracy dropped" → trace back to data source
     → discover upstream schema change

  2. IMPACT ANALYSIS
     "Database table X is being deprecated"
     → Which models depend on it? → lineage tells you

  3. COMPLIANCE
     "Auditor asks: how was this decision made?"
     → Full trace: data → features → model → prediction

  4. REPRODUCIBILITY
     "Reproduce the model from 6 months ago"
     → Lineage points to exact data, code, and config

  5. ROOT CAUSE ANALYSIS
     "Why is model performing differently?"
     → Compare lineage of v3 vs v4 → find what changed
```

---

## Implementing Lineage

```python
import mlflow

with mlflow.start_run() as run:
    # Link to data version
    mlflow.set_tag("data.source", "transactions_db")
    mlflow.set_tag("data.version", "dvc:abc123def")
    mlflow.set_tag("data.rows", "2300000")
    mlflow.set_tag("data.date_range", "2024-01 to 2025-03")
    
    # Link to code version
    mlflow.set_tag("mlflow.source.git.commit", "def456789")
    mlflow.set_tag("mlflow.source.git.branch", "main")
    mlflow.set_tag("code.feature_pipeline", "git:aaa111")
    
    # Link to feature store
    mlflow.set_tag("features.store", "feast")
    mlflow.set_tag("features.view", "user_transaction_features:v3")
    mlflow.set_tag("features.count", "47")
    
    # Link to environment
    mlflow.set_tag("env.docker_image", "ml-train:2025.03.10-gpu")
    mlflow.set_tag("env.python", "3.11.5")
    mlflow.set_tag("env.cuda", "12.1")
    
    # Train and register model
    model.fit(X_train, y_train)
    mlflow.sklearn.log_model(
        model, "model",
        registered_model_name="fraud_detector",
    )
    
    # Now this model version has full lineage:
    # model v4 → run_789 → data abc123 → code def456
```

---

## Lineage Tools

```
  ┌─────────────────┬──────────────────────────────────────┐
  │ Tool            │ Lineage capability                   │
  ├─────────────────┼──────────────────────────────────────┤
  │ MLflow          │ Run → model → data (via tags)        │
  │ DVC             │ Data → pipeline stages → outputs     │
  │ Kubeflow        │ Pipeline → step → artifacts          │
  │ W&B             │ Run → artifacts → model              │
  │ DataHub         │ End-to-end data lineage              │
  │ OpenLineage     │ Open standard for lineage metadata   │
  │ Apache Atlas    │ Enterprise data governance lineage   │
  └─────────────────┴──────────────────────────────────────┘
```

---

## Revision Questions

1. **What is model lineage and what questions does it answer?**
2. **How does lineage help with debugging production model issues?**
3. **What information should be captured at each stage of the lineage chain?**
4. **How can you implement basic lineage using MLflow tags?**

---

[Previous: 03-model-metadata.md](03-model-metadata.md) | [Next: 05-model-promotion.md](05-model-promotion.md)
