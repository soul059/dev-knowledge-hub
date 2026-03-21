# Weights & Biases (W&B)

## Overview

**Weights & Biases (wandb)** is a managed experiment tracking and ML developer platform. Compared to MLflow, W&B provides richer visualizations, built-in collaboration features, hyperparameter sweep support, and a polished hosted UI. It's extremely popular in the research community and increasingly in industry.

---

## Key Features

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. Experiment Tracking:  Log params, metrics, artifacts     │
  │  2. Dashboards:           Interactive charts, custom panels  │
  │  3. Sweeps:               Built-in hyperparameter search     │
  │  4. Artifacts:            Dataset & model versioning         │
  │  5. Tables:               Log & visualize structured data    │
  │  6. Reports:              Shareable documents with live data │
  │  7. Model Registry:       Version and promote models         │
  │  8. Launch:               Job orchestration on any compute   │
  │                                                                │
  │  Unique strengths over MLflow:                               │
  │  • Superior visualization (custom charts, media logging)    │
  │  • Built-in collaboration (teams, reports, comments)        │
  │  • Native sweep (hyperparameter optimization)               │
  │  • System metrics auto-logging (GPU util, memory, etc.)     │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Python: W&B Tracking

```python
import wandb
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, f1_score

# Initialize a run
wandb.init(
    project="churn-prediction",
    name="rf-baseline",
    config={
        "model_type": "random_forest",
        "n_estimators": 100,
        "max_depth": 10,
        "learning_rate": None,
        "dataset_version": "v3",
    },
    tags=["baseline", "production-candidate"],
)

config = wandb.config  # Access config anywhere

# Train model
model = RandomForestClassifier(
    n_estimators=config.n_estimators,
    max_depth=config.max_depth,
)
model.fit(X_train, y_train)

# Log metrics
y_pred = model.predict(X_test)
wandb.log({
    "accuracy": accuracy_score(y_test, y_pred),
    "f1_score": f1_score(y_test, y_pred),
    "train_size": len(X_train),
    "test_size": len(X_test),
})

# Log step-level metrics (e.g., per epoch)
for epoch in range(50):
    train_loss = train_one_epoch(model, epoch)
    val_loss = evaluate(model)
    wandb.log({
        "epoch": epoch,
        "train_loss": train_loss,
        "val_loss": val_loss,
    })

# Log media (images, plots, tables)
import matplotlib.pyplot as plt
fig, ax = plt.subplots()
ax.bar(range(10), model.feature_importances_[:10])
wandb.log({"feature_importance": wandb.Image(fig)})
plt.close()

# Log confusion matrix
wandb.log({
    "confusion_matrix": wandb.plot.confusion_matrix(
        y_true=y_test, preds=y_pred,
        class_names=["not_churn", "churn"]
    )
})

# Log a table of predictions
table = wandb.Table(columns=["input", "true", "predicted"])
for i in range(100):
    table.add_data(str(X_test.iloc[i].values), y_test.iloc[i], y_pred[i])
wandb.log({"predictions": table})

# Finish the run
wandb.finish()
```

---

## W&B Sweeps (Hyperparameter Optimization)

```python
import wandb

# Define sweep configuration
sweep_config = {
    "method": "bayes",       # bayesian optimization
    "metric": {
        "name": "val_f1",
        "goal": "maximize",
    },
    "parameters": {
        "n_estimators": {"values": [50, 100, 200, 500]},
        "max_depth": {"min": 3, "max": 20},
        "min_samples_split": {"min": 2, "max": 20},
        "learning_rate": {
            "distribution": "log_uniform_values",
            "min": 1e-4,
            "max": 1e-1,
        },
    },
}

# Create sweep
sweep_id = wandb.sweep(sweep_config, project="churn-prediction")


# Define training function
def train():
    with wandb.init() as run:
        config = wandb.config
        
        model = GradientBoostingClassifier(
            n_estimators=config.n_estimators,
            max_depth=config.max_depth,
            min_samples_split=config.min_samples_split,
            learning_rate=config.learning_rate,
        )
        model.fit(X_train, y_train)
        
        y_pred = model.predict(X_val)
        wandb.log({"val_f1": f1_score(y_val, y_pred)})


# Run the sweep (launches multiple runs)
wandb.agent(sweep_id, function=train, count=50)

# W&B automatically:
# - Tries 50 hyperparameter combinations
# - Uses Bayesian optimization to focus on promising areas
# - Visualizes results in the sweep dashboard
```

---

## W&B Artifacts (Data & Model Versioning)

```python
import wandb

run = wandb.init(project="churn-prediction")

# Log a dataset as an artifact
artifact = wandb.Artifact("training-data", type="dataset")
artifact.add_file("data/train.csv")
artifact.add_dir("data/processed/")
run.log_artifact(artifact)

# Log a model as an artifact
model_artifact = wandb.Artifact(
    "churn-model",
    type="model",
    metadata={"accuracy": 0.87, "framework": "sklearn"},
)
model_artifact.add_file("models/model.pkl")
run.log_artifact(model_artifact)

# Use an artifact in another run
run = wandb.init(project="churn-prediction")
artifact = run.use_artifact("training-data:v2")
artifact_dir = artifact.download()
# Data is now at artifact_dir/train.csv
```

---

## W&B vs. MLflow Comparison

```
  ┌──────────────────┬──────────────────┬──────────────────┐
  │ Aspect           │ MLflow           │ W&B              │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Hosting          │ Self-hosted      │ Cloud-hosted     │
  │                  │ (or Databricks)  │ (or self-hosted) │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Cost             │ Free (self-host) │ Free tier + paid │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Visualization    │ Basic            │ Rich, interactive│
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Collaboration    │ Limited          │ Built-in (teams, │
  │                  │                  │ reports, comments)│
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Hyperparameter   │ External tools   │ Built-in Sweeps  │
  │ optimization     │ (Optuna, etc.)   │                  │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ System metrics   │ Manual           │ Auto (GPU, mem)  │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Model registry   │ ✓ (strong)       │ ✓                │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Open-source      │ Fully            │ Client only      │
  ├──────────────────┼──────────────────┼──────────────────┤
  │ Best for         │ Full control,    │ Research, teams  │
  │                  │ enterprise       │ wanting ease     │
  └──────────────────┴──────────────────┴──────────────────┘

  Many teams use BOTH: MLflow for production, W&B for research.
```

---

## Revision Questions

1. **What are W&B's unique advantages over MLflow?**
2. **How do W&B Sweeps automate hyperparameter optimization?**
3. **How does W&B Artifacts handle data and model versioning?**
4. **What types of media can W&B log beyond numbers?**
5. **When would you choose W&B over MLflow?**

---

[Previous: 02-mlflow-tracking.md](02-mlflow-tracking.md) | [Next: 04-experiment-comparison.md](04-experiment-comparison.md)
