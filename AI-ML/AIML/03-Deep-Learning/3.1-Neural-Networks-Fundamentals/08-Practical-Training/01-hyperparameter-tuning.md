# 1. Hyperparameter Tuning

> **Unit 8 · Practical Training** — Systematically finding the best configuration for your neural network

---

## Chapter Overview

Training a neural network involves choosing dozens of **hyperparameters** — values that are *not* learned by gradient descent but profoundly affect model performance. Picking a learning rate of 0.1 vs 0.001, or using 3 hidden layers vs 7, can mean the difference between a model that converges beautifully and one that fails completely. This chapter catalogs the key hyperparameters, explains their effects, and walks through systematic search strategies — from brute-force grid search to sophisticated Bayesian optimization — so you can find winning configurations efficiently.

---

## 1. What Are Hyperparameters?

### Parameters vs Hyperparameters

```
  ┌────────────────────────────────────────────────────┐
  │               Neural Network Config                │
  ├──────────────────────┬─────────────────────────────┤
  │     PARAMETERS       │      HYPERPARAMETERS        │
  │  (learned by SGD)    │  (set before training)      │
  ├──────────────────────┼─────────────────────────────┤
  │  • Weights (W)       │  • Learning rate (α)        │
  │  • Biases (b)        │  • Batch size               │
  │                      │  • Number of layers         │
  │  Adjusted by         │  • Neurons per layer        │
  │  backpropagation     │  • Dropout rate             │
  │  every iteration     │  • Weight decay (λ)         │
  │                      │  • Optimizer choice          │
  │  Millions of them    │  • Activation functions      │
  │                      │  • Learning rate schedule    │
  │                      │                             │
  │  Set AFTER training  │  Set BEFORE training        │
  └──────────────────────┴─────────────────────────────┘
```

### Why Tuning Matters

```
  Performance
  ↑
  │            ╱──╲            Best HP config
  │           ╱    ╲
  │      ╱──╲╱      ╲         HP landscape is
  │     ╱             ╲       NOT smooth!
  │    ╱               ╲──╲
  │   ╱                    ╲
  │──╱                      ╲──
  │
  └──────────────────────────────→ Hyperparameter value
  
  Small changes in hyperparameters can cause
  LARGE changes in performance
```

---

## 2. Key Hyperparameters in Neural Networks

### 2.1 Learning Rate (α)

The single most important hyperparameter.

```
  Learning Rate Too Small          Just Right              Too Large
  ────────────────────────────────────────────────────────────────────
  
  Loss ↑                      Loss ↑                   Loss ↑
       │╲                          │╲                        │╲  ╱╲  ╱╲
       │ ╲                         │ ╲                       │ ╲╱  ╲╱  ╲
       │  ╲                        │  ╲                      │          ╲╱╲
       │   ╲                       │   ╲╲                    │             ╲→ NaN!
       │    ╲╲╲╲╲╲╲╲╲──           │     ╲──── converges!    │
       └──────────────→            └──────────────→          └──────────────→
       Extremely slow!             Fast convergence          Diverges!
  
  Typical range: 1e-5 to 1e-1
  Good default:  1e-3 (Adam), 1e-2 (SGD with momentum)
```

### 2.2 Batch Size

```
  Small Batch (32)                              Large Batch (4096)
  ──────────────────────────────────────────────────────────────────
  ✓ Noisy gradients → regularization effect     ✓ Stable gradients
  ✓ Flat minima → better generalization         ✓ Faster per epoch (GPU efficient)
  ✗ Slower per epoch                            ✗ Sharp minima → worse generalization
  ✗ Poor GPU utilization                        ✗ Needs LR warmup
  
  Common values: 32, 64, 128, 256
  Rule of thumb: Start with 32-128
```

### 2.3 Number of Layers (Depth)

```
  Depth = Number of hidden layers
  
  1 layer:   Simple patterns (linear boundaries + activation)
  2 layers:  Most standard problems
  3-5 layers: Complex patterns, hierarchical features
  50+ layers: Very complex tasks (ResNet, need skip connections)
  
  ┌─────────────────────────────────────────┐
  │  More layers:                           │
  │  ✓ Can learn hierarchical features      │
  │  ✗ Harder to train (vanishing gradient) │
  │  ✗ More prone to overfitting            │
  │  ✗ Slower training and inference        │
  └─────────────────────────────────────────┘
```

### 2.4 Neurons per Layer (Width)

```
  Width = Number of neurons in a hidden layer
  
  Common pattern — "funnel" or "constant":
  
  Funnel:                     Constant:
  Input: 784                  Input: 784
  Hidden 1: 512               Hidden 1: 256
  Hidden 2: 256               Hidden 2: 256
  Hidden 3: 128               Hidden 3: 256
  Output: 10                  Output: 10
  
  Rules of thumb:
  • Start between input size and output size
  • Powers of 2 (64, 128, 256...) for GPU efficiency
  • Start wider, narrow down if overfitting
```

### 2.5 Dropout Rate

```
  Dropout Rate (p) = probability of zeroing a neuron during training
  
  p = 0.0:  No dropout (no regularization)
  p = 0.1-0.3:  Light regularization (common for input layer)
  p = 0.5:  Standard (Hinton's original recommendation)
  p = 0.7+:  Heavy regularization (very large networks)
  p = 1.0:  Drop everything (network learns nothing!)
  
  ┌─────────────────────────────────────────────┐
  │  • Higher dropout → more regularization     │
  │  • Needs more epochs to converge            │
  │  • Usually higher for wider layers          │
  │  • Often 0.2-0.5 in practice               │
  └─────────────────────────────────────────────┘
```

### 2.6 Weight Decay (λ)

```
  λ controls L2 regularization strength
  
  λ = 0:      No regularization
  λ = 1e-5:   Very light (common with Adam)
  λ = 1e-4:   Standard for many tasks
  λ = 1e-3:   Strong regularization
  λ = 1e-2:   Very strong (may underfit)
  
  Note: In Adam optimizer, weight decay ≠ L2 regularization
  Use AdamW for proper decoupled weight decay
```

---

## 3. Hyperparameter Search Strategies

### 3.1 Grid Search

Try every combination of predefined values.

```
  Grid Search: Learning Rate × Batch Size
  
  LR ↑
  1e-1 │  ●────●────●────●
       │  │    │    │    │
  1e-2 │  ●────●────●────●     ● = one experiment
       │  │    │    │    │
  1e-3 │  ●────●────●────●     Total: 4 × 3 = 12 experiments
       │  │    │    │    │
  1e-4 │  ●────●────●────●
       └──┬────┬────┬────┬──→ Batch Size
          32   64  128  256
```

**Pros:**
- Simple to implement
- Exhaustive (won't miss combinations on the grid)

**Cons:**
- Exponential cost: k values × n hyperparameters = k^n experiments
- Wastes compute on unimportant dimensions

```python
from sklearn.model_selection import ParameterGrid

param_grid = {
    'learning_rate': [1e-4, 1e-3, 1e-2, 1e-1],
    'batch_size': [32, 64, 128, 256],
    'hidden_size': [64, 128, 256],
    'dropout': [0.1, 0.3, 0.5]
}

# Total experiments: 4 × 4 × 3 × 3 = 144 !!!
for params in ParameterGrid(param_grid):
    model = build_model(params)
    score = train_and_evaluate(model, params)
```

### 3.2 Random Search

Sample hyperparameters randomly from distributions.

```
  Random Search: Learning Rate × Batch Size
  
  LR ↑
  1e-1 │       ●        ●
       │  ●         ●
  1e-2 │        ●            ●    ● = one experiment
       │    ●          ●
  1e-3 │  ●       ●              Total: 12 experiments
       │             ●    ●      (same budget as grid)
  1e-4 │      ●           
       └────────────────────→ Batch Size
          32   64  128  256
```

### Why Random is Better Than Grid for Neural Networks

```
  KEY INSIGHT (Bergstra & Bengio, 2012):
  
  When some hyperparameters matter MORE than others
  (which is ALWAYS true for neural networks),
  random search explores more unique values of
  important hyperparameters.
  
  Grid Search (9 trials):          Random Search (9 trials):
  ┌─────────────────────┐         ┌─────────────────────┐
  │ ●    ●    ●         │         │    ●   ●       ●    │
  │                     │         │  ●         ●        │
  │ ●    ●    ●         │  vs     │       ●        ●    │
  │                     │         │  ●              ●    │
  │ ●    ●    ●         │         │         ●           │
  └─────────────────────┘         └─────────────────────┘
  Important HP                    Important HP
  ↕ only 3 unique values          ↕ 9 unique values!
  
  Unimportant HP →                Unimportant HP →
  3 unique values                 9 unique values
  
  Grid "wastes" trials on the unimportant dimension
  Random covers the important dimension MUCH better
```

```python
import numpy as np

# Define distributions (not discrete lists!)
param_distributions = {
    'learning_rate': lambda: 10 ** np.random.uniform(-5, -1),  # log-uniform
    'batch_size': lambda: int(2 ** np.random.uniform(5, 9)),   # 32-512
    'hidden_size': lambda: int(2 ** np.random.uniform(6, 9)),  # 64-512
    'dropout': lambda: np.random.uniform(0.1, 0.6),           # uniform
    'weight_decay': lambda: 10 ** np.random.uniform(-6, -2),  # log-uniform
    'n_layers': lambda: np.random.randint(1, 6),              # discrete
}

n_trials = 50
best_score = 0
best_params = None

for trial in range(n_trials):
    params = {k: v() for k, v in param_distributions.items()}
    score = train_and_evaluate(params)
    if score > best_score:
        best_score = score
        best_params = params
        print(f"Trial {trial}: New best = {score:.4f}, params = {params}")
```

**Key point: Use log-uniform for learning rate and weight decay!**

```
  Linear scale:   [0.001, 0.002, 0.003, ..., 0.100]
                   ↑ 99% of samples between 0.01-0.1!
  
  Log scale:      [1e-5, 1e-4, 1e-3, 1e-2, 1e-1]
                   ↑ Equal chance in each order of magnitude ✓
```

### 3.3 Bayesian Optimization

Use a probabilistic model to predict promising hyperparameters.

```
  Bayesian Optimization Loop:
  
  ┌──────────────────────────────────────────────────┐
  │  1. Evaluate a few random configurations         │
  │  2. Build surrogate model of HP → performance    │
  │  3. Use acquisition function to pick next HP     │
  │     (balance exploration vs exploitation)         │
  │  4. Evaluate that configuration                  │
  │  5. Update surrogate model                       │
  │  6. Repeat from step 3                           │
  └──────────────────────────────────────────────────┘
  
  Surrogate Model (Gaussian Process / TPE):
  
  Score ↑     ┌─╮   Predicted mean
       │    ╱─┘ ╰─╮
       │  ╱─       ╰───╮    ╱─── uncertainty
       │╱─   ▒▒▒▒▒   ╰──╱     (explore here!)
       │    ▒▒▒▒▒▒▒▒      
       │  observed points (●)
       │  ●   ●     ●  ●  ●
       └──────────────────────→ Hyperparameter
              ↑
        Next point to try
        (high uncertainty + high predicted score)
```

#### Optuna (Recommended)

```python
import optuna

def objective(trial):
    """Define the search space and training in one function."""
    # Suggest hyperparameters
    lr = trial.suggest_float('lr', 1e-5, 1e-1, log=True)
    batch_size = trial.suggest_categorical('batch_size', [32, 64, 128, 256])
    n_layers = trial.suggest_int('n_layers', 1, 5)
    dropout = trial.suggest_float('dropout', 0.1, 0.5)
    hidden_size = trial.suggest_categorical('hidden_size', [64, 128, 256, 512])
    weight_decay = trial.suggest_float('weight_decay', 1e-6, 1e-2, log=True)
    optimizer_name = trial.suggest_categorical('optimizer', ['Adam', 'AdamW', 'SGD'])
    
    # Build model
    layers = []
    in_features = 784
    for i in range(n_layers):
        layers.append(nn.Linear(in_features, hidden_size))
        layers.append(nn.ReLU())
        layers.append(nn.Dropout(dropout))
        in_features = hidden_size
    layers.append(nn.Linear(in_features, 10))
    model = nn.Sequential(*layers).to(device)
    
    # Choose optimizer
    if optimizer_name == 'Adam':
        optimizer = torch.optim.Adam(model.parameters(), lr=lr, weight_decay=weight_decay)
    elif optimizer_name == 'AdamW':
        optimizer = torch.optim.AdamW(model.parameters(), lr=lr, weight_decay=weight_decay)
    else:
        optimizer = torch.optim.SGD(model.parameters(), lr=lr, 
                                     momentum=0.9, weight_decay=weight_decay)
    
    # Train and evaluate
    for epoch in range(20):
        train_one_epoch(model, optimizer, train_loader, batch_size)
        val_acc = evaluate(model, val_loader)
        
        # Report intermediate value for pruning
        trial.report(val_acc, epoch)
        
        # Prune unpromising trials early!
        if trial.should_prune():
            raise optuna.exceptions.TrialPruned()
    
    return val_acc

# Create study and optimize
study = optuna.create_study(
    direction='maximize',
    pruner=optuna.pruners.MedianPruner(n_warmup_steps=5)
)
study.optimize(objective, n_trials=100, timeout=3600)  # 100 trials or 1 hour

# Results
print(f"Best accuracy: {study.best_value:.4f}")
print(f"Best hyperparameters: {study.best_params}")

# Visualization
optuna.visualization.plot_optimization_history(study)
optuna.visualization.plot_param_importances(study)
```

#### Ray Tune

```python
from ray import tune
from ray.tune.schedulers import ASHAScheduler

config = {
    "lr": tune.loguniform(1e-5, 1e-1),
    "batch_size": tune.choice([32, 64, 128, 256]),
    "n_layers": tune.randint(1, 6),
    "hidden_size": tune.choice([64, 128, 256, 512]),
    "dropout": tune.uniform(0.1, 0.5),
}

scheduler = ASHAScheduler(
    max_t=50,          # max epochs
    grace_period=5,    # min epochs before pruning
    reduction_factor=3
)

result = tune.run(
    train_fn,
    config=config,
    num_samples=100,
    scheduler=scheduler,
    resources_per_trial={"cpu": 2, "gpu": 0.5}
)

print(f"Best config: {result.best_config}")
```

### 3.4 Hyperband

Early stopping on steroids — aggressively prunes bad configurations.

```
  Hyperband: Successive Halving
  
  Round 1: Train 81 configs for 1 epoch each
           Keep top 1/3 → 27 configs
  
  Round 2: Train 27 configs for 3 epochs each
           Keep top 1/3 → 9 configs
  
  Round 3: Train 9 configs for 9 epochs each
           Keep top 1/3 → 3 configs
  
  Round 4: Train 3 configs for 27 epochs each
           Keep top 1/3 → 1 winner!
  
  Total budget: 81×1 + 27×3 + 9×9 + 3×27 = 324 epoch-equivalents
  vs full training: 81 × 27 = 2187 epoch-equivalents
  
  → 6.7x more efficient!
```

### 3.5 Population-Based Training (PBT)

Combines hyperparameter search with training — mutates hyperparameters during training.

```
  Population-Based Training:
  
  Epoch 1    Epoch 5    Epoch 10    Epoch 15    Epoch 20
  ────────────────────────────────────────────────────────
  Agent A ──────●──────────●───────────●────────────●
                │          ↑ copy weights           │
  Agent B ──────●──────────●           ●────────────●
                │          │ mutate HP ↑            │ 
  Agent C ──────●──────────●───────────●────────────● ← BEST
                           │           │
  Agent D ──────●──────────● (pruned — replaced by copy of C)
  
  Key idea: Poorly performing agents are replaced by
  copies of good agents with mutated hyperparameters.
  
  Advantages:
  • HP schedule is adapted DURING training
  • No need to restart from scratch
  • Automatically finds HP schedules (e.g., LR decay)
```

---

## 4. Practical Priority Order for Tuning

Not all hyperparameters are equally important. Tune in this order:

```
  ┌─────────────────────────────────────────────────────────────┐
  │  PRIORITY 1 (Highest Impact)                                │
  │  ─────────────────────────────                              │
  │  1. Learning rate           ← ALWAYS tune this first       │
  │  2. Learning rate schedule  ← step decay, cosine, one-cycle│
  │                                                             │
  │  PRIORITY 2 (High Impact)                                   │
  │  ────────────────────────                                   │
  │  3. Batch size              ← affects LR and generalization│
  │  4. Number of layers        ← model capacity               │
  │  5. Hidden layer size       ← model capacity               │
  │                                                             │
  │  PRIORITY 3 (Medium Impact)                                 │
  │  ─────────────────────────                                  │
  │  6. Optimizer choice        ← Adam vs SGD vs AdamW         │
  │  7. Weight decay            ← regularization               │
  │  8. Dropout rate            ← regularization               │
  │                                                             │
  │  PRIORITY 4 (Lower Impact)                                  │
  │  ─────────────────────────                                  │
  │  9. Activation function     ← ReLU is usually fine         │
  │  10. Weight initialization  ← Kaiming/Xavier default works │
  │  11. Momentum               ← 0.9 is usually fine         │
  └─────────────────────────────────────────────────────────────┘
```

### Practical Tuning Workflow

```
  Step 1: Get a BASELINE working
  ├── Use standard defaults (Adam, lr=1e-3, batch=128)
  ├── Verify training loop works (overfit single batch)
  └── Get baseline validation score
  
  Step 2: Tune LEARNING RATE
  ├── Use LR range test (see next chapter)
  ├── Try 3-5 values: [1e-4, 3e-4, 1e-3, 3e-3, 1e-2]
  └── Pick best, then fine-tune around it
  
  Step 3: Tune ARCHITECTURE
  ├── Try 2-4 depth/width combinations
  └── Balance capacity vs overfitting
  
  Step 4: Tune REGULARIZATION
  ├── Adjust dropout (0.1-0.5)
  ├── Adjust weight decay (1e-5 to 1e-3)
  └── Try data augmentation
  
  Step 5: Tune SCHEDULE
  ├── Compare step decay vs cosine annealing vs one-cycle
  └── Adjust schedule parameters
  
  Step 6: FINAL POLISH
  ├── Run Bayesian optimization on remaining HPs
  └── Ensemble top models if needed
```

---

## 5. Complete Example: Tuning an MLP with Optuna

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, random_split
from torchvision import datasets, transforms
import optuna

# ── Data ──
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

full_train = datasets.MNIST('./data', train=True, download=True, transform=transform)
train_set, val_set = random_split(full_train, [50000, 10000])
test_set = datasets.MNIST('./data', train=False, transform=transform)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')


def build_model(n_layers, hidden_size, dropout):
    """Build an MLP with variable depth and width."""
    layers = []
    in_features = 784
    for _ in range(n_layers):
        layers.extend([
            nn.Linear(in_features, hidden_size),
            nn.ReLU(),
            nn.Dropout(dropout)
        ])
        in_features = hidden_size
    layers.append(nn.Linear(in_features, 10))
    return nn.Sequential(layers[0], *layers[1:])


def objective(trial):
    # ── Hyperparameter search space ──
    lr = trial.suggest_float('lr', 1e-5, 1e-1, log=True)
    batch_size = trial.suggest_categorical('batch_size', [32, 64, 128, 256])
    n_layers = trial.suggest_int('n_layers', 1, 4)
    hidden_size = trial.suggest_categorical('hidden_size', [128, 256, 512])
    dropout = trial.suggest_float('dropout', 0.0, 0.5)
    weight_decay = trial.suggest_float('weight_decay', 1e-6, 1e-2, log=True)
    
    # ── Model, optimizer, data loaders ──
    model = build_model(n_layers, hidden_size, dropout).to(device)
    optimizer = optim.AdamW(model.parameters(), lr=lr, weight_decay=weight_decay)
    criterion = nn.CrossEntropyLoss()
    
    train_loader = DataLoader(train_set, batch_size=batch_size, shuffle=True)
    val_loader = DataLoader(val_set, batch_size=512)
    
    # ── Training loop ──
    for epoch in range(20):
        model.train()
        for X, y in train_loader:
            X, y = X.view(-1, 784).to(device), y.to(device)
            optimizer.zero_grad()
            loss = criterion(model(X), y)
            loss.backward()
            optimizer.step()
        
        # ── Validation ──
        model.eval()
        correct = total = 0
        with torch.no_grad():
            for X, y in val_loader:
                X, y = X.view(-1, 784).to(device), y.to(device)
                preds = model(X).argmax(dim=1)
                correct += (preds == y).sum().item()
                total += y.size(0)
        
        val_acc = correct / total
        trial.report(val_acc, epoch)
        
        if trial.should_prune():
            raise optuna.exceptions.TrialPruned()
    
    return val_acc


# ── Run optimization ──
study = optuna.create_study(
    direction='maximize',
    study_name='mnist-mlp-tuning',
    pruner=optuna.pruners.MedianPruner(n_warmup_steps=5)
)

study.optimize(objective, n_trials=50)

# ── Results ──
print(f"\nBest validation accuracy: {study.best_value:.4f}")
print(f"Best hyperparameters:")
for key, value in study.best_params.items():
    print(f"  {key}: {value}")

# ── Retrain best model on full training data ──
best = study.best_params
model = build_model(best['n_layers'], best['hidden_size'], best['dropout']).to(device)
optimizer = optim.AdamW(model.parameters(), lr=best['lr'], weight_decay=best['weight_decay'])
criterion = nn.CrossEntropyLoss()

# ... train with best params on full training set, evaluate on test set
```

---

## 6. Common Pitfalls

```
  ┌─────────────────────────────────────────────────────────────────┐
  │  PITFALL                          │  FIX                       │
  ├───────────────────────────────────┼────────────────────────────┤
  │  Tuning on test set               │  Always use validation set │
  │  Using linear scale for LR        │  Use log scale!            │
  │  Too many HPs at once             │  Tune in priority order    │
  │  Not enough trials                │  ≥50 for random/Bayesian   │
  │  Ignoring interactions            │  LR and batch size interact│
  │  Not fixing random seeds          │  Fix seeds for fair compare│
  │  Overfitting to validation set    │  Keep separate test set    │
  │  Grid search with many HPs       │  Use random or Bayesian    │
  └───────────────────────────────────┴────────────────────────────┘
```

---

## 7. Applications

| Domain | Key Hyperparameters | Typical Approach |
|--------|-------------------|------------------|
| **Image Classification** | LR, augmentation, architecture depth | Random search + fine-tune LR |
| **NLP / Transformers** | LR, warmup steps, weight decay | Grid on LR, random on rest |
| **Reinforcement Learning** | LR, discount factor, exploration rate | PBT (hyperparameters change during training) |
| **Medical Imaging** | Dropout, weight decay (limited data) | Bayesian optimization |
| **Production Systems** | All (cost-sensitive) | Hyperband for efficiency |

---

## 8. Summary Table

| Strategy | Trials Needed | Best For | Key Advantage | Key Disadvantage |
|----------|:------------:|----------|---------------|------------------|
| **Grid Search** | k^n | Few HPs (≤3) | Exhaustive | Exponential cost |
| **Random Search** | 50-200 | General purpose | Covers important dims | No learning between trials |
| **Bayesian (Optuna)** | 30-100 | Expensive models | Learns from past trials | Overhead, can get stuck |
| **Hyperband** | 50-200 | Fast evaluation | Early stopping saves compute | Assumes early performance predicts late |
| **PBT** | 10-30 agents | RL, long training | Adapts HPs during training | Complex to implement |

---

## 9. Revision Questions

1. **Why is random search generally better than grid search for neural network hyperparameter tuning?** Explain with reference to the Bergstra & Bengio (2012) finding about important vs unimportant hyperparameters.

2. **Why should learning rate be sampled on a log scale rather than a linear scale?** Give an example showing the distribution difference.

3. **Describe how Bayesian optimization differs from random search.** What is the role of the surrogate model and acquisition function?

4. **What is Hyperband and how does successive halving work?** Calculate the computational savings compared to training all configurations for the maximum number of epochs.

5. **In what order should you tune hyperparameters for a neural network, and why?** List the top 5 hyperparameters by impact.

6. **Explain Population-Based Training (PBT) and when it would be preferred over standard Bayesian optimization.** What unique advantage does PBT have regarding learning rate schedules?

---

| [← 07-Regularization-Techniques](../07-Regularization-Techniques/) | [Unit 8 Home](README.md) | [02 → Learning Rate Finder](02-learning-rate-finder.md) |
|:---|:---:|---:|
