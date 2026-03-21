# Experiment Comparison

## Overview

Running experiments is only half the battle — you need to systematically **compare** them to make informed decisions about which model to deploy. This chapter covers comparison strategies, visualization techniques, and how to use MLflow and W&B to do side-by-side analysis of training runs.

---

## What to Compare

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. METRICS                                                  │
  │     • Primary metric (F1, AUC, accuracy)                     │
  │     • Secondary metrics (latency, model size, memory)        │
  │     • Training curves (loss over epochs)                     │
  │                                                                │
  │  2. HYPERPARAMETERS                                          │
  │     • Which params matter most?                              │
  │     • Interaction effects between params                     │
  │                                                                │
  │  3. RESOURCES                                                │
  │     • Training time, GPU hours, cost                         │
  │     • Inference latency and throughput                       │
  │                                                                │
  │  4. DATA                                                     │
  │     • Which dataset version produced best results?           │
  │     • Impact of data augmentation                            │
  │                                                                │
  │  5. ROBUSTNESS                                               │
  │     • Performance across data slices (demographics, edges)   │
  │     • Variance across random seeds                           │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Comparison Strategies

```
  1. Tabular Comparison:
     Sort all runs by primary metric, filter by constraints

     ┌──────┬────────┬────────┬───────┬────────┬──────────┐
     │ Run  │ Model  │   LR   │  F1   │Latency │  Size    │
     ├──────┼────────┼────────┼───────┼────────┼──────────┤
     │ #12  │ XGBoost│ 0.01   │ 0.891 │  5ms   │  12MB    │
     │ #8   │ RF     │  N/A   │ 0.876 │  3ms   │   8MB    │
     │ #15  │ BERT   │ 2e-5   │ 0.923 │ 45ms   │ 420MB    │
     │ #11  │ LR     │ 0.001  │ 0.812 │  1ms   │  0.1MB   │
     └──────┴────────┴────────┴───────┴────────┴──────────┘

     Best F1? #15 (BERT)
     Best F1 under 10ms latency? #12 (XGBoost)

  2. Parallel Coordinates Plot:
     Visualize how params relate to metrics across many runs
     Each line = one run, axes = params/metrics

  3. Scatter Plots:
     Plot metric A vs metric B (e.g., accuracy vs latency)
     Find the Pareto frontier (best trade-off)

  4. Training Curve Overlay:
     Plot loss/metric curves of multiple runs on same chart
     Spot overfitting, convergence speed differences
```

---

## Python: Comparing Runs in MLflow

```python
import mlflow
import pandas as pd

client = mlflow.tracking.MlflowClient()
experiment = client.get_experiment_by_name("churn_prediction")

# Search all completed runs
runs = client.search_runs(
    experiment_ids=[experiment.experiment_id],
    filter_string="status = 'FINISHED'",
    order_by=["metrics.f1_score DESC"],
)

# Create comparison DataFrame
comparison = []
for run in runs:
    comparison.append({
        "run_id": run.info.run_id[:8],
        "run_name": run.info.run_name,
        "model_type": run.data.params.get("model_type", "unknown"),
        "lr": run.data.params.get("learning_rate"),
        "accuracy": run.data.metrics.get("accuracy"),
        "f1_score": run.data.metrics.get("f1_score"),
        "auc_roc": run.data.metrics.get("auc_roc"),
        "training_time": run.data.metrics.get("training_time"),
    })

df = pd.DataFrame(comparison)
print(df.to_string(index=False))

# Find best run with constraints
best = df[df["training_time"] < 60].sort_values("f1_score", ascending=False).iloc[0]
print(f"\nBest run (< 60s training): {best['run_name']}, F1={best['f1_score']:.4f}")
```

---

## Python: Visualizing Comparisons

```python
import matplotlib.pyplot as plt
import numpy as np

# Scatter plot: F1 vs Latency (find best trade-off)
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# F1 vs Training Time
ax = axes[0]
for _, row in df.iterrows():
    ax.scatter(row["training_time"], row["f1_score"], s=100)
    ax.annotate(row["run_name"], (row["training_time"], row["f1_score"]))
ax.set_xlabel("Training Time (s)")
ax.set_ylabel("F1 Score")
ax.set_title("F1 vs Training Time")

# Grouped bar chart: metrics by model type
ax = axes[1]
model_means = df.groupby("model_type")[["accuracy", "f1_score", "auc_roc"]].mean()
model_means.plot.bar(ax=ax)
ax.set_title("Average Metrics by Model Type")
ax.set_ylabel("Score")
ax.legend(loc="lower right")

plt.tight_layout()
plt.savefig("experiment_comparison.png")
plt.show()


# Statistical significance testing
from scipy import stats

# Compare two models' F1 scores across multiple seeds
model_a_scores = [0.85, 0.87, 0.84, 0.86, 0.85]  # 5 seeds
model_b_scores = [0.88, 0.87, 0.89, 0.88, 0.90]  # 5 seeds

t_stat, p_value = stats.ttest_ind(model_a_scores, model_b_scores)
print(f"t-statistic: {t_stat:.3f}, p-value: {p_value:.4f}")
if p_value < 0.05:
    print("✅ Statistically significant difference!")
else:
    print("⚠️ No significant difference — may be noise")
```

---

## Multi-Objective Comparison

```
  Often you optimize for MULTIPLE objectives:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Accuracy ▲                                       │
  │  0.95 │          ●B                               │
  │  0.90 │    ●A         ●C                         │
  │  0.85 │                    ●D                    │
  │  0.80 │ ●E                                       │
  │       └──────────────────────────►               │
  │        1ms   10ms   50ms  100ms  Latency         │
  │                                                    │
  │  Pareto frontier: A, B, C                        │
  │  (no other run is better in BOTH metrics)        │
  │                                                    │
  │  Choose from Pareto frontier based on:           │
  │  • Business priority (accuracy vs speed)         │
  │  • Deployment constraints (max latency)          │
  │  • Cost (GPU vs CPU serving)                     │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Decision Framework

```
  When comparing experiments, ask:

  1. Is the improvement statistically significant?
     → Run with multiple seeds, compute p-value

  2. Is it practically significant?
     → 0.1% accuracy gain may not matter in practice

  3. What are the trade-offs?
     → Better accuracy but 10× slower? Worth it?

  4. How does it perform on subgroups?
     → Overall accuracy high but fails for minority group?

  5. Is it reproducible?
     → Does it work across data splits and random seeds?

  6. Can it be deployed?
     → Model size, latency, memory constraints
```

---

## Revision Questions

1. **What five dimensions should you compare experiments on?**
2. **What is the Pareto frontier in multi-objective comparison?**
3. **Why should you test for statistical significance when comparing models?**
4. **How do you compare runs programmatically in MLflow?**
5. **What questions should you ask before selecting a model for deployment?**

---

[Previous: 03-weights-and-biases.md](03-weights-and-biases.md) | [Next: 05-hyperparameter-logging.md](05-hyperparameter-logging.md)
