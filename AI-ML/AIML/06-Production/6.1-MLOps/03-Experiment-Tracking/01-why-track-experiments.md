# Why Track Experiments?

## Overview

Experiment tracking is the practice of recording every ML training run — its parameters, metrics, code version, data version, and artifacts. Without tracking, teams lose track of what worked, can't reproduce results, and waste time repeating failed experiments. Experiment tracking is one of the first and highest-ROI MLOps practices to adopt.

---

## The Problem Without Tracking

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  Data Scientist's Notebook (3 months in):                    │
  │                                                                │
  │  # Model v1 - lr=0.01, epochs=10, acc=0.82                  │
  │  # Model v2 - lr=0.001, epochs=20, acc=0.85                 │
  │  # Model v3 - added dropout, acc=0.84 (wait, or was it 0.86?)│
  │  # Model v4 - which dataset did I use??                      │
  │  # Model v5 - can't remember the preprocessing              │
  │  # Model v6 - "the good one" (which params???)              │
  │                                                                │
  │  "My best model got 0.92 accuracy last month."              │
  │  "Can you reproduce it?"                                     │
  │  "...no."                                                    │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘

  Sound familiar? This is EVERY ML team without experiment tracking.
```

---

## What to Track

```
  For every training run, record:

  ┌──────────────────┬──────────────────────────────────────┐
  │ Category         │ Examples                             │
  ├──────────────────┼──────────────────────────────────────┤
  │ Parameters       │ learning_rate, batch_size, epochs,   │
  │ (inputs)         │ model_type, num_layers, dropout      │
  ├──────────────────┼──────────────────────────────────────┤
  │ Metrics          │ accuracy, loss, F1, AUC, latency,    │
  │ (outputs)        │ training_time                        │
  ├──────────────────┼──────────────────────────────────────┤
  │ Artifacts        │ model.pkl, confusion_matrix.png,     │
  │                  │ feature_importance.csv               │
  ├──────────────────┼──────────────────────────────────────┤
  │ Code version     │ Git commit hash, branch name         │
  ├──────────────────┼──────────────────────────────────────┤
  │ Data version     │ DVC hash, dataset path, row count    │
  ├──────────────────┼──────────────────────────────────────┤
  │ Environment      │ Python version, library versions,    │
  │                  │ GPU type, OS                         │
  ├──────────────────┼──────────────────────────────────────┤
  │ Tags / Notes     │ "baseline", "with_augmentation",     │
  │                  │ "ready_for_review"                   │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Benefits of Experiment Tracking

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. REPRODUCIBILITY                                          │
  │     Exact params + code + data → reproduce any result        │
  │                                                                │
  │  2. COMPARISON                                               │
  │     Side-by-side comparison of runs                          │
  │     "Model A vs B: which had better F1?"                    │
  │                                                                │
  │  3. COLLABORATION                                            │
  │     Team members see each other's experiments                │
  │     No duplicate work, shared learnings                      │
  │                                                                │
  │  4. AUDITABILITY                                             │
  │     "What model is in production? How was it trained?"      │
  │     Full lineage for compliance                              │
  │                                                                │
  │  5. DECISION MAKING                                          │
  │     Data-driven model selection, not gut feeling             │
  │                                                                │
  │  6. KNOWLEDGE PRESERVATION                                   │
  │     When team members leave, experiments remain              │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Tool Landscape

```
  ┌─────────────────┬──────────┬──────────┬──────────┬──────────┐
  │ Feature         │ MLflow   │ W&B      │ Neptune  │ ClearML  │
  ├─────────────────┼──────────┼──────────┼──────────┼──────────┤
  │ Open-source     │ ✓        │          │          │ ✓        │
  │ Free tier       │ ✓ (self) │ ✓        │ ✓        │ ✓        │
  │ Hosted option   │ Databrick│ ✓        │ ✓        │ ✓        │
  │ Param tracking  │ ✓        │ ✓        │ ✓        │ ✓        │
  │ Metric tracking │ ✓        │ ✓        │ ✓        │ ✓        │
  │ Artifact store  │ ✓        │ ✓        │ ✓        │ ✓        │
  │ Model registry  │ ✓        │ ✓        │ ✓        │ ✓        │
  │ Visualization   │ Basic    │ ✓✓✓      │ ✓✓       │ ✓✓       │
  │ Collaboration   │ Basic    │ ✓✓✓      │ ✓✓       │ ✓✓       │
  │ Hyperparameter  │ External │ ✓ Sweeps │ ✓        │ ✓        │
  │   optimization  │          │          │          │          │
  │ Learning curve  │ Low      │ Low      │ Medium   │ Medium   │
  └─────────────────┴──────────┴──────────┴──────────┴──────────┘

  Most common: MLflow (open-source) or W&B (managed)
```

---

## Quick Example: Tracking Without a Tool vs. With MLflow

```python
# WITHOUT tracking (bad!)
model = train_model(lr=0.001, epochs=50, dropout=0.3)
accuracy = evaluate(model, test_data)
print(f"Accuracy: {accuracy}")  # Then you forget everything...


# WITH MLflow tracking (good!)
import mlflow

mlflow.set_experiment("churn_prediction")

with mlflow.start_run(run_name="dropout_experiment"):
    # Log parameters
    mlflow.log_param("learning_rate", 0.001)
    mlflow.log_param("epochs", 50)
    mlflow.log_param("dropout", 0.3)
    mlflow.log_param("model_type", "random_forest")
    
    # Train
    model = train_model(lr=0.001, epochs=50, dropout=0.3)
    
    # Log metrics
    accuracy = evaluate(model, test_data)
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("f1_score", f1)
    mlflow.log_metric("training_time", elapsed)
    
    # Log artifacts
    mlflow.log_artifact("confusion_matrix.png")
    mlflow.sklearn.log_model(model, "model")
    
    # Log tags
    mlflow.set_tag("dataset_version", "v3")
    mlflow.set_tag("status", "candidate")

# Now EVERY run is recorded with full context!
# Compare runs in the MLflow UI: http://localhost:5000
```

---

## Revision Questions

1. **What problems occur when experiments are not tracked?**
2. **What six categories of information should be tracked per run?**
3. **How does experiment tracking enable reproducibility?**
4. **Compare MLflow and Weights & Biases — key differences?**
5. **Why is experiment tracking considered one of the first MLOps practices to adopt?**

---

[Previous: ../02-Data-Management/05-data-quality-monitoring.md](../02-Data-Management/05-data-quality-monitoring.md) | [Next: 02-mlflow-tracking.md](02-mlflow-tracking.md)
