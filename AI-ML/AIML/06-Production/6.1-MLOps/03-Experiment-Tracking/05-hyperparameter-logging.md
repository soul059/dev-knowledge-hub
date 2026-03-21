# Hyperparameter Logging

## Overview

Hyperparameter logging is the systematic recording of all configuration choices that affect model training — from learning rates and batch sizes to architectural decisions and preprocessing options. Proper logging enables reproducibility, hyperparameter optimization, and understanding which settings drive performance.

---

## What Are Hyperparameters?

```
  Parameters (learned):        Hyperparameters (set by you):
  • Model weights              • Learning rate
  • Biases                     • Batch size
  • Embeddings                 • Number of epochs
  (learned during training)    • Architecture choices
                               • Regularization settings
                               • Data preprocessing options
                               (set BEFORE training)

  Categories:
  ┌──────────────────┬──────────────────────────────────────┐
  │ Category         │ Examples                             │
  ├──────────────────┼──────────────────────────────────────┤
  │ Optimization     │ lr, optimizer, scheduler, momentum   │
  │ Architecture     │ num_layers, hidden_size, dropout     │
  │ Regularization   │ weight_decay, dropout_rate, L1/L2    │
  │ Data             │ augmentation, sampling, batch_size   │
  │ Training         │ epochs, early_stopping_patience      │
  │ Preprocessing    │ normalization, tokenizer, max_length │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Config Management Best Practices

```
  DON'T hardcode hyperparameters in training code!

  # BAD ❌
  model = Model(hidden_size=256, lr=0.001, dropout=0.3)

  # GOOD ✅ — use config files
  # config.yaml
  model:
    hidden_size: 256
    num_layers: 4
    dropout: 0.3
  training:
    lr: 0.001
    batch_size: 32
    epochs: 100
    optimizer: adam
  data:
    augmentation: true
    max_length: 512
```

---

## Python: Structured Config with Hydra

```python
# config/config.yaml
# model:
#   type: transformer
#   hidden_size: 256
#   num_layers: 4
#   dropout: 0.3
# training:
#   lr: 0.001
#   batch_size: 32
#   epochs: 100
#   optimizer: adam
# data:
#   path: data/train.csv
#   max_length: 512

import hydra
from omegaconf import DictConfig
import mlflow


@hydra.main(config_path="config", config_name="config", version_base=None)
def train(cfg: DictConfig):
    with mlflow.start_run():
        # Log ALL hyperparameters automatically
        mlflow.log_params({
            "model_type": cfg.model.type,
            "hidden_size": cfg.model.hidden_size,
            "num_layers": cfg.model.num_layers,
            "dropout": cfg.model.dropout,
            "lr": cfg.training.lr,
            "batch_size": cfg.training.batch_size,
            "epochs": cfg.training.epochs,
            "optimizer": cfg.training.optimizer,
            "data_path": cfg.data.path,
            "max_length": cfg.data.max_length,
        })
        
        # Also log the full config as artifact
        mlflow.log_dict(dict(cfg), "config.yaml")
        
        # Train...
        model = build_model(cfg.model)
        train_model(model, cfg.training, cfg.data)


if __name__ == "__main__":
    train()

# Override from command line:
# python train.py training.lr=0.0001 model.dropout=0.5
# python train.py --multirun training.lr=0.001,0.0001,0.00001
```

---

## Python: Logging with MLflow and W&B

```python
import mlflow
import wandb

# Comprehensive hyperparameter logging
config = {
    # Model architecture
    "model_type": "xgboost",
    "n_estimators": 500,
    "max_depth": 8,
    "learning_rate": 0.01,
    "subsample": 0.8,
    "colsample_bytree": 0.8,
    "min_child_weight": 3,
    "reg_alpha": 0.1,
    "reg_lambda": 1.0,
    
    # Data preprocessing
    "feature_selection": "mutual_info_top_50",
    "normalization": "standard_scaler",
    "handle_imbalance": "smote",
    "test_size": 0.2,
    "random_state": 42,
    
    # Training
    "early_stopping_rounds": 50,
    "eval_metric": "logloss",
    
    # Environment
    "python_version": "3.11.5",
    "xgboost_version": "2.0.3",
}

# MLflow
with mlflow.start_run():
    mlflow.log_params(config)
    # ... train and evaluate

# W&B
wandb.init(project="churn", config=config)
# wandb.config automatically accessible everywhere
```

---

## Hyperparameter Search Logging

```python
import optuna
import mlflow

def objective(trial):
    """Optuna objective with MLflow logging."""
    # Sample hyperparameters
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 50, 500),
        "max_depth": trial.suggest_int("max_depth", 3, 15),
        "learning_rate": trial.suggest_float("learning_rate", 1e-4, 0.1, log=True),
        "subsample": trial.suggest_float("subsample", 0.5, 1.0),
        "min_child_weight": trial.suggest_int("min_child_weight", 1, 10),
    }
    
    # Log each trial as a separate MLflow run
    with mlflow.start_run(nested=True, run_name=f"trial_{trial.number}"):
        mlflow.log_params(params)
        
        model = XGBClassifier(**params)
        model.fit(X_train, y_train)
        
        f1 = f1_score(y_val, model.predict(X_val))
        mlflow.log_metric("f1_score", f1)
        mlflow.set_tag("trial_number", trial.number)
        
        return f1

# Run optimization
with mlflow.start_run(run_name="optuna_sweep"):
    study = optuna.create_study(direction="maximize")
    study.optimize(objective, n_trials=100)
    
    # Log best params
    mlflow.log_params({f"best_{k}": v for k, v in study.best_params.items()})
    mlflow.log_metric("best_f1", study.best_value)
    
    print(f"Best F1: {study.best_value:.4f}")
    print(f"Best params: {study.best_params}")
```

---

## What NOT to Forget

```
  Commonly overlooked hyperparameters:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ✗ Random seed (affects reproducibility!)        │
  │  ✗ Data split strategy and seed                  │
  │  ✗ Feature preprocessing steps                   │
  │  ✗ Class weights / sampling strategy             │
  │  ✗ Early stopping patience                       │
  │  ✗ Gradient clipping value                       │
  │  ✗ Warmup steps for scheduler                    │
  │  ✗ Library versions (scikit-learn, PyTorch)      │
  │  ✗ Hardware (GPU type, number of workers)        │
  │  ✗ Data augmentation settings                    │
  │                                                    │
  │  Rule: if changing it could change the result,   │
  │  LOG IT!                                         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What is the difference between parameters and hyperparameters?**
2. **Why should you use config files instead of hardcoded values?**
3. **How does Hydra help with hyperparameter management?**
4. **How can you log Optuna trials as nested runs in MLflow?**
5. **What commonly overlooked settings should always be logged?**

---

[Previous: 04-experiment-comparison.md](04-experiment-comparison.md) | [Next: 06-artifact-management.md](06-artifact-management.md)
