# Artifact Management

## Overview

Artifacts are the files produced during ML experiments — trained models, evaluation plots, preprocessed datasets, configuration files, and any other outputs. Proper artifact management ensures you can always find, reproduce, and deploy the exact outputs from any experiment. MLflow, W&B, and cloud storage backends all provide artifact tracking capabilities.

---

## Types of ML Artifacts

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Artifact Type    │ Examples                             │
  ├──────────────────┼──────────────────────────────────────┤
  │ Models           │ model.pkl, model.pt, saved_model/,   │
  │                  │ model.onnx                           │
  ├──────────────────┼──────────────────────────────────────┤
  │ Evaluation       │ confusion_matrix.png, roc_curve.png, │
  │                  │ classification_report.json           │
  ├──────────────────┼──────────────────────────────────────┤
  │ Data             │ train.parquet, test.parquet,          │
  │                  │ feature_importance.csv               │
  ├──────────────────┼──────────────────────────────────────┤
  │ Configuration    │ config.yaml, params.json,             │
  │                  │ requirements.txt                     │
  ├──────────────────┼──────────────────────────────────────┤
  │ Logs             │ training_log.txt, tensorboard/       │
  ├──────────────────┼──────────────────────────────────────┤
  │ Preprocessing    │ scaler.pkl, tokenizer.json,           │
  │                  │ label_encoder.pkl                    │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Artifact Storage Architecture

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Experiment Tracker ──── references ────► Artifact Store      │
  │  (MLflow, W&B)                           (S3, GCS, Azure)   │
  │                                                                │
  │  Tracker stores:            Artifact Store stores:           │
  │  • Run ID                   • Actual files                   │
  │  • Artifact URI             • Organized by run_id            │
  │  • Metadata                 • Large binary files             │
  │                                                                │
  │  ┌──────────────────────┐   ┌──────────────────────┐         │
  │  │  MLflow DB           │   │  S3 Bucket           │         │
  │  │  run: abc123         │──►│  /abc123/            │         │
  │  │  artifact: model.pkl │   │    model.pkl         │         │
  │  │  artifact: plots/    │   │    plots/            │         │
  │  └──────────────────────┘   │      roc.png         │         │
  │                              │      cm.png          │         │
  │                              └──────────────────────┘         │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: MLflow Artifact Management

```python
import mlflow
import json
import pickle

with mlflow.start_run(run_name="full_pipeline"):
    
    # --- Log individual files ---
    mlflow.log_artifact("config.yaml")
    mlflow.log_artifact("requirements.txt")
    
    # --- Log directory of files ---
    mlflow.log_artifacts("evaluation_plots/", artifact_path="plots")
    # Creates: artifacts/plots/roc_curve.png, artifacts/plots/cm.png
    
    # --- Log model with MLflow's model format ---
    mlflow.sklearn.log_model(model, "model")
    # Creates: artifacts/model/model.pkl, artifacts/model/MLmodel
    
    # --- Log preprocessing objects ---
    with open("scaler.pkl", "wb") as f:
        pickle.dump(scaler, f)
    mlflow.log_artifact("scaler.pkl", artifact_path="preprocessing")
    
    # --- Log dict as JSON ---
    metrics = {"accuracy": 0.87, "f1": 0.85, "auc": 0.91}
    mlflow.log_dict(metrics, "detailed_metrics.json")
    
    # --- Log text ---
    mlflow.log_text(
        "Model trained with augmented data, v3 features",
        "notes.txt"
    )
    
    # --- Log figure ---
    import matplotlib.pyplot as plt
    fig, ax = plt.subplots()
    ax.plot([1, 2, 3], [0.8, 0.85, 0.87])
    mlflow.log_figure(fig, "training_curve.png")
    plt.close()


# --- Retrieve artifacts from a past run ---
run_id = "abc123def456"
client = mlflow.tracking.MlflowClient()

# List artifacts
artifacts = client.list_artifacts(run_id)
for a in artifacts:
    print(f"  {a.path} ({a.file_size} bytes)")

# Download artifacts
local_path = client.download_artifacts(run_id, "model/model.pkl")
print(f"Downloaded to: {local_path}")

# Load model directly
model = mlflow.sklearn.load_model(f"runs:/{run_id}/model")
predictions = model.predict(X_test)
```

---

## MLflow Model Format

```
  MLflow saves models in a standard format:

  artifacts/model/
  ├── MLmodel              ← metadata (flavor, signature)
  ├── model.pkl            ← serialized model
  ├── conda.yaml           ← conda environment
  ├── python_env.yaml      ← Python environment
  ├── requirements.txt     ← pip requirements
  └── input_example.json   ← sample input for testing

  MLmodel file contents:
  ┌────────────────────────────────────────┐
  │ flavors:                               │
  │   python_function:                     │
  │     model_path: model.pkl              │
  │     loader_module: mlflow.sklearn      │
  │   sklearn:                             │
  │     pickled_model: model.pkl           │
  │     sklearn_version: 1.4.0             │
  │ signature:                             │
  │   inputs:                              │
  │     - name: age, type: float           │
  │     - name: income, type: float        │
  │   outputs:                             │
  │     - type: integer                    │
  └────────────────────────────────────────┘

  Benefits:
  • Framework-agnostic loading (any model via pyfunc)
  • Environment reproducibility (exact library versions)
  • Input/output schema validation
```

---

## Artifact Lifecycle

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. CREATE                                                   │
  │     During training: log model, plots, configs               │
  │                                                                │
  │  2. STORE                                                    │
  │     Artifacts stored in configured backend (S3, GCS, local)  │
  │     Linked to run ID for traceability                       │
  │                                                                │
  │  3. DISCOVER                                                 │
  │     Browse in UI, query via API                              │
  │     Search by run params/metrics to find relevant artifacts  │
  │                                                                │
  │  4. RETRIEVE                                                 │
  │     Download for analysis, comparison, or deployment         │
  │     Load models directly for inference                       │
  │                                                                │
  │  5. PROMOTE                                                  │
  │     Best model artifact → Model Registry → Production        │
  │                                                                │
  │  6. ARCHIVE / DELETE                                         │
  │     Old artifacts consume storage                            │
  │     Set retention policies (keep last N, keep tagged)        │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Best Practices

```
  1. Log EVERYTHING needed to reproduce a run
     • Model, preprocessors, config, requirements.txt

  2. Use consistent artifact paths
     • artifacts/model/, artifacts/plots/, artifacts/data/

  3. Include model signatures
     • Define input/output schemas for validation

  4. Version your environments
     • Log conda.yaml or requirements.txt with every run

  5. Set retention policies
     • Don't keep every artifact forever (cost!)
     • Keep: production models, tagged "best" runs
     • Delete: failed runs, old exploratory runs

  6. Use artifact URIs, not local paths
     • Reference: "runs:/abc123/model" not "/tmp/model.pkl"
```

---

## Revision Questions

1. **What types of artifacts should be logged during ML experiments?**
2. **How does MLflow's model format ensure reproducibility?**
3. **How do you retrieve artifacts from a past MLflow run?**
4. **What is the artifact lifecycle in MLOps?**
5. **Why should you set artifact retention policies?**

---

[Previous: 05-hyperparameter-logging.md](05-hyperparameter-logging.md) | [Next: ../04-Model-Training-Pipelines/01-pipeline-orchestration.md](../04-Model-Training-Pipelines/01-pipeline-orchestration.md)
