# Model Versioning

## Overview

Model versioning tracks every trained model with a unique identifier, linking it to the exact code, data, hyperparameters, and environment that produced it. Unlike code versioning (Git), model versioning must handle large binary files and rich metadata. This chapter covers why, what, and how to version ML models.

---

## Why Version Models?

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Without versioning:                                         │
  │  "Which model is in production?" → "model_final_v2_FIXED.pkl"│
  │  "What data trained it?"         → "...I don't remember"    │
  │  "Can we rollback?"              → "Rollback to what?"      │
  │                                                                │
  │  With versioning:                                            │
  │  "Which model?"  → churn_classifier v3 (run abc123)         │
  │  "What data?"    → dataset v2.1 (DVC hash: d4e5f6)          │
  │  "Rollback?"     → Promote v2 back to Production stage      │
  │                                                                │
  │  Benefits:                                                   │
  │  ✓ Full traceability (model → training run → data)          │
  │  ✓ Instant rollback to any previous version                 │
  │  ✓ Reproducibility                                          │
  │  ✓ Audit compliance                                         │
  │  ✓ A/B testing between model versions                       │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## What to Version

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Artifact         │ How to version                       │
  ├──────────────────┼──────────────────────────────────────┤
  │ Model weights    │ Model Registry (MLflow, W&B)         │
  │ Model code       │ Git (commit hash linked to model)    │
  │ Training data    │ DVC (hash linked to model)           │
  │ Hyperparameters  │ Experiment tracker (logged per run)  │
  │ Environment      │ Docker image, requirements.txt       │
  │ Preprocessing    │ Serialized objects (scaler.pkl)      │
  │ Feature list     │ Feature manifest / config file       │
  │ Evaluation       │ Metrics stored in experiment tracker │
  └──────────────────┴──────────────────────────────────────┘

  A model version = (model weights + all linked metadata)
```

---

## Model Versioning Strategies

```
  1. FILE-BASED:
     Save model files with version numbers
     models/churn_v1.pkl, models/churn_v2.pkl
     ✗ No metadata, no lifecycle management

  2. GIT + DVC:
     Store model files in DVC, metadata in Git
     git tag model-v3
     ✓ Versioned, but no staging/promotion workflow

  3. MODEL REGISTRY (best):
     Centralized system for model lifecycle
     Register → Stage (Staging) → Promote (Production) → Archive
     ✓ Full lifecycle, metadata, governance
     
  Most teams evolve: 1 → 2 → 3
```

---

## Python: Model Versioning with MLflow

```python
import mlflow
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Register a model from a training run
model_uri = f"runs:/{run_id}/model"
result = mlflow.register_model(model_uri, "churn_classifier")
# Creates version 1, 2, 3, ... automatically

# Add description
client.update_model_version(
    name="churn_classifier",
    version=result.version,
    description="GBM with v3 features, F1=0.87"
)

# List all versions
versions = client.search_model_versions("name='churn_classifier'")
for v in versions:
    print(f"  v{v.version}: stage={v.current_stage}, "
          f"run={v.run_id[:8]}")

# Load a specific version
model = mlflow.pyfunc.load_model("models:/churn_classifier/3")
predictions = model.predict(X_test)
```

---

## Revision Questions

1. **Why is model versioning different from code versioning?**
2. **What artifacts should be linked to a model version?**
3. **Compare file-based, Git+DVC, and Model Registry approaches.**
4. **How does MLflow auto-increment model version numbers?**

---

[Next: 02-mlflow-model-registry.md](02-mlflow-model-registry.md)
