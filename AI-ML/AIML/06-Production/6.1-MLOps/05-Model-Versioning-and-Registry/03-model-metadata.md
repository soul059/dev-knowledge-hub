# Model Metadata

## Overview

Model metadata is all the information that describes a model beyond its weights — training data provenance, performance metrics, input/output schema, intended use, known limitations, and owner. Comprehensive metadata enables informed deployment decisions, regulatory compliance, and debugging in production.

---

## Essential Model Metadata

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Category         │ What to record                       │
  ├──────────────────┼──────────────────────────────────────┤
  │ Identity         │ Model name, version, unique ID       │
  │ Training         │ Run ID, date, duration, framework    │
  │ Data             │ Dataset version, size, schema, hash  │
  │ Performance      │ All metrics (accuracy, F1, AUC, etc.)│
  │ Hyperparameters  │ All config values                    │
  │ Schema           │ Input features (names, types, ranges)│
  │                  │ Output format and meaning            │
  │ Environment      │ Python version, library versions,    │
  │                  │ Docker image, GPU type               │
  │ Ownership        │ Team, creator, reviewer              │
  │ Governance       │ Intended use, known limitations,     │
  │                  │ fairness evaluation, approval status │
  │ Operational      │ Latency, throughput, model size      │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Model Cards

```
  Model Cards (Mitchell et al., 2019, Google):
  Standardized documentation for ML models

  ┌──────────────────────────────────────────────────┐
  │  MODEL CARD: Churn Prediction v4                 │
  │                                                    │
  │  Model Details:                                  │
  │    Type: XGBoost classifier                      │
  │    Version: 4                                    │
  │    Owner: Growth team                            │
  │    Date: 2025-03-10                              │
  │                                                    │
  │  Intended Use:                                   │
  │    Predict 30-day churn for subscription users   │
  │    NOT for: credit decisions, user blocking      │
  │                                                    │
  │  Training Data:                                  │
  │    180 days of user behavior (2024-09 to 2025-03)│
  │    500,000 users, 12% churn rate                 │
  │                                                    │
  │  Performance:                                    │
  │    Overall: F1=0.87, AUC=0.93                    │
  │    By segment:                                   │
  │      Free users:    F1=0.89                      │
  │      Premium users: F1=0.84                      │
  │      New users:     F1=0.79 (lower!)             │
  │                                                    │
  │  Limitations:                                    │
  │    • Lower accuracy for users < 30 days old      │
  │    • Not tested on enterprise accounts           │
  │    • Assumes stable pricing structure            │
  │                                                    │
  │  Ethical Considerations:                         │
  │    • No demographic features used                │
  │    • Fairness tested across regions              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Recording Model Metadata

```python
import mlflow

with mlflow.start_run() as run:
    # Training metadata
    mlflow.log_params(hyperparameters)
    mlflow.log_metrics(evaluation_metrics)
    
    # Model signature (input/output schema)
    from mlflow.models import infer_signature
    signature = infer_signature(X_train, model.predict(X_train))
    
    # Log model with rich metadata
    mlflow.sklearn.log_model(
        model,
        artifact_path="model",
        signature=signature,
        input_example=X_train.head(5),  # sample input
        registered_model_name="churn_classifier",
    )
    
    # Custom metadata tags
    mlflow.set_tag("model_card_url", "https://wiki/model-cards/churn-v4")
    mlflow.set_tag("owner", "growth-team")
    mlflow.set_tag("intended_use", "30-day churn prediction for subscription users")
    mlflow.set_tag("limitations", "Lower accuracy for users < 30 days old")
    mlflow.set_tag("data_version", "dvc:abc123")
    mlflow.set_tag("docker_image", "ml-train:2025.03.10")
    
    # Log model card as artifact
    model_card = """
    # Churn Prediction Model v4
    ## Intended Use: Predict 30-day churn
    ## Performance: F1=0.87, AUC=0.93
    ## Limitations: Lower accuracy for new users
    """
    mlflow.log_text(model_card, "MODEL_CARD.md")
```

---

## Revision Questions

1. **What categories of metadata should be recorded for every model?**
2. **What is a Model Card and why was it proposed?**
3. **How does MLflow's model signature help catch input errors?**
4. **Why is recording model limitations important?**

---

[Previous: 02-mlflow-model-registry.md](02-mlflow-model-registry.md) | [Next: 04-model-lineage.md](04-model-lineage.md)
