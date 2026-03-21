# MLflow Model Registry

## Overview

The **MLflow Model Registry** is a centralized hub for managing the full lifecycle of ML models — from registration through staging, production, and archival. It provides version control, stage transitions, annotations, and CI/CD integration for models.

---

## Model Lifecycle Stages

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  None ──► Staging ──► Production ──► Archived                │
  │                                                                │
  │  None:        Just registered, not deployed anywhere         │
  │  Staging:     Under testing / validation                     │
  │  Production:  Serving live traffic                           │
  │  Archived:    Retired, kept for audit/rollback               │
  │                                                                │
  │  Rules:                                                      │
  │  • Only ONE version in Production at a time (per model)      │
  │  • Promoting new version auto-archives the old one           │
  │  • Archived models remain accessible for rollback            │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: Full Registry Workflow

```python
import mlflow
from mlflow.tracking import MlflowClient

client = MlflowClient()

# 1. Register a new model from a run
model_uri = f"runs:/{run_id}/model"
mv = mlflow.register_model(model_uri, "fraud_detector")
print(f"Registered: v{mv.version}")

# 2. Add metadata
client.update_registered_model(
    name="fraud_detector",
    description="Real-time fraud detection model for payments"
)
client.update_model_version(
    name="fraud_detector",
    version=mv.version,
    description="XGBoost, F1=0.92, trained on 2025-Q1 data"
)

# 3. Transition to Staging
client.transition_model_version_stage(
    name="fraud_detector",
    version=mv.version,
    stage="Staging",
)
print(f"v{mv.version} → Staging")

# 4. Run validation tests on Staging model
staging_model = mlflow.pyfunc.load_model("models:/fraud_detector/Staging")
test_predictions = staging_model.predict(X_test)
test_f1 = f1_score(y_test, test_predictions > 0.5)

if test_f1 >= 0.90:
    # 5. Promote to Production
    client.transition_model_version_stage(
        name="fraud_detector",
        version=mv.version,
        stage="Production",
        archive_existing_versions=True,  # auto-archive old production
    )
    print(f"v{mv.version} → Production ✅")
else:
    print(f"v{mv.version} failed validation (F1={test_f1:.3f})")

# 6. Load production model for serving
prod_model = mlflow.pyfunc.load_model("models:/fraud_detector/Production")

# 7. Rollback: transition a previous version back to Production
client.transition_model_version_stage(
    name="fraud_detector",
    version=2,          # previous version
    stage="Production",
    archive_existing_versions=True,
)
print("Rolled back to v2 ✅")
```

---

## Model Registry UI

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Model: fraud_detector                                       │
  │  Description: Real-time fraud detection for payments         │
  │                                                                │
  │  ┌────────┬─────────┬────────┬────────┬────────────────────┐ │
  │  │Version │ Stage   │ F1     │ Created│ Description        │ │
  │  ├────────┼─────────┼────────┼────────┼────────────────────┤ │
  │  │ v4     │ Prod.   │ 0.923  │ Mar 10 │ XGBoost + new feat│ │
  │  │ v3     │ Archived│ 0.915  │ Feb 15 │ XGBoost baseline  │ │
  │  │ v2     │ Archived│ 0.891  │ Jan 20 │ Random forest      │ │
  │  │ v1     │ Archived│ 0.856  │ Jan 05 │ Logistic regression│ │
  │  └────────┴─────────┴────────┴────────┴────────────────────┘ │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What are the four model lifecycle stages in MLflow?**
2. **How does `archive_existing_versions` work during promotion?**
3. **How do you load a model by stage name vs. version number?**
4. **How do you implement rollback using the Model Registry?**

---

[Previous: 01-model-versioning.md](01-model-versioning.md) | [Next: 03-model-metadata.md](03-model-metadata.md)
