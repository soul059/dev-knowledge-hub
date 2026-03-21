# 3. Early Stopping

> **Unit 7 · Regularization Techniques** — Halting training before the model overfits

---

## Chapter Overview

Early stopping is perhaps the **simplest and most effective** regularization technique: monitor the validation loss during training and **stop when it starts increasing**. Since training loss continually decreases but validation loss eventually rises (overfitting), the point where validation loss is minimum represents the best generalization. Early stopping requires no modification to the loss function or architecture — just careful monitoring.

---

## 1. The Overfitting Curve

### Training Dynamics

```
  Loss ↑
       │
       │╲
       │ ╲  ·                          ───── Training Loss
       │  ╲  ·  ·                      · · · Validation Loss
       │   ╲   ·   ·
       │    ╲    ·    ·
       │     ╲     ·     ·
       │      ╲──    ·      ·  ·
       │         ──────·──────────·──·──·──·  ← val loss increases
       │               ──────────────────     ← train loss still drops
       │                    ↑
       │              STOP HERE!
       └──────────────────────────────────────→ Epoch
        0     20     40    60    80    100
       
       ←── underfitting ──→←── overfitting ──→
       ←── learning ──────→←── memorizing ───→
```

### Three Phases of Training

| Phase | Training Loss | Validation Loss | What's Happening |
|---|---|---|---|
| 1. Learning | Decreasing fast | Decreasing fast | Learning useful patterns |
| 2. Transition | Decreasing slowly | Flat / slightly increasing | Near optimal point |
| 3. Overfitting | Still decreasing | Increasing | Memorizing training noise |

---

## 2. Early Stopping Algorithm

### Basic Version

```
Algorithm: Early Stopping

Input: patience P, model, training data, validation data

1. best_loss = ∞
2. best_weights = None
3. patience_counter = 0

4. For each epoch:
   a. Train one epoch
   b. Compute validation loss L_val
   
   c. If L_val < best_loss:
        best_loss = L_val
        best_weights = copy(model.weights)
        patience_counter = 0                 ← reset counter
      Else:
        patience_counter += 1                ← no improvement
   
   d. If patience_counter ≥ P:
        model.weights = best_weights         ← restore best
        STOP training
        
5. Return model with best_weights
```

### Patience Parameter

```
  Val Loss ↑
           │
           │╲
           │ ╲  ╲
           │  ╲   ╲──╮
           │   ╲      ╰──╮     ← temporary plateau (patience helps here!)
           │    ╲         ╰╮
           │     ╲         ╰──╮
           │      ╲            ╰──╮
           │       ╲     ★         ╰─── ← eventually goes lower!
           │        ╲                      (would've missed with patience=0)
           └───────────────────────────────→ Epoch
           
  patience = 0:  stop at first increase → misses the true minimum ★
  patience = 5:  wait 5 epochs of no improvement → finds ★
  patience = 20: very tolerant, may train too long
```

| Patience | Behavior |
|---|---|
| 0–3 | Very aggressive, may stop too early |
| 5–10 | Standard range for most tasks |
| 10–20 | Conservative, for noisy validation curves |
| 20+ | Very patient, close to no early stopping |

---

## 3. Connection to L2 Regularization

### Bishop's Insight (1995)

For simple models (linear regression with gradient descent), early stopping is approximately equivalent to L2 regularization:

```
  Early stopping after t steps ≈ L2 with λ ∝ 1/(α·t)
  
  Fewer steps (early stopping)  ↔  Stronger L2 (larger λ)
  More steps (late stopping)    ↔  Weaker L2 (smaller λ)
```

### Intuition

```
  Initialization: w₀ ≈ 0 (small random)
  
  Without regularization, weights grow large over time:
  
  ||w|| ↑
        │                    ╱╱╱
        │                 ╱╱╱
        │              ╱╱╱
        │           ╱╱╱            ← weights grow larger with more training
        │        ╱╱╱
        │     ╱╱╱
        │  ╱╱╱
        │╱╱
        └──────────────────────→ Epochs
        
  Early stopping keeps ||w|| small → similar to L2 constraint ||w||² ≤ C
```

### Key Differences

| L2 Regularization | Early Stopping |
|---|---|
| Explicit penalty in loss | Implicit constraint on weight growth |
| Requires choosing λ | Requires choosing patience |
| Applies every step | Stops training entirely |
| Can train to convergence | May not reach convergence |
| Weight-specific penalty | Global training-time constraint |

---

## 4. Best Model Checkpointing

### Why Restore Best Weights?

```
  Without restoring:                  With restoring:
  
  Val Loss ↑                          Val Loss ↑
           │ ╲                                 │ ╲
           │  ╲                                │  ╲
           │   ╲    ★ ← best                  │   ╲    ★ ← best
           │    ╲  ╱ ╲                         │    ╲  ╱
           │     ╲╱   ╲                        │     ╲╱
           │      ↑    ╲                       │   RESTORED ←
           │   we stop  ╲ ← worse              │
           │   HERE       ╲                    │
           └──────────────→ Epoch              └──────────────→ Epoch
           
  Final model is at the STOP                  Final model is at the BEST
  point (after patience runs                  point (checkpointed earlier)
  out), which is WORSE than best!
```

---

## 5. Practical Considerations

### What Metric to Monitor

| Metric | When to Use |
|---|---|
| Validation loss | Default choice, most general |
| Validation accuracy | Classification tasks |
| Validation F1 score | Imbalanced classification |
| BLEU score | Machine translation |
| Validation AUC | Ranking/retrieval tasks |

### Monitoring Tips

```
  ┌──────────────────────────────────────────────────────┐
  │  ✓ Always use a HELD-OUT validation set              │
  │  ✓ Don't tune hyperparameters on test set            │
  │  ✓ Use the same validation set throughout training   │
  │  ✓ Monitor at the end of each epoch (not each batch) │
  │  ✓ Save best model to disk (not just memory)         │
  │  ✓ Consider using a minimum improvement threshold    │
  └──────────────────────────────────────────────────────┘
```

### Minimum Improvement Threshold (min_delta)

```
  Without min_delta:
  Tiny improvements (0.0001) reset patience → training runs forever
  
  With min_delta = 0.001:
  Only improvements > 0.001 count as "true" improvements
  Small fluctuations don't reset patience
```

---

## 6. Worked Example

### Problem
Training a classifier. Validation losses by epoch:

| Epoch | Val Loss | Best? | Patience Counter |
|---|---|---|---|
| 1 | 2.50 | Yes (best=2.50) | 0 |
| 2 | 2.10 | Yes (best=2.10) | 0 |
| 3 | 1.80 | Yes (best=1.80) | 0 |
| 4 | 1.75 | Yes (best=1.75) | 0 |
| 5 | 1.78 | No | 1 |
| 6 | 1.82 | No | 2 |
| 7 | 1.71 | Yes (best=1.71) | **0** (reset!) |
| 8 | 1.73 | No | 1 |
| 9 | 1.76 | No | 2 |
| 10 | 1.79 | No | 3 |
| 11 | 1.80 | No | 4 |
| 12 | 1.83 | No | 5 = patience! **STOP** |

**Result**: Restore model from epoch 7 (val loss = 1.71).

---

## 7. Python Implementation (PyTorch)

### EarlyStopping Class

```python
import torch
import copy

class EarlyStopping:
    """Early stopping to halt training when validation loss plateaus.
    
    Args:
        patience: Number of epochs to wait after last improvement
        min_delta: Minimum change to qualify as an improvement
        restore_best: Whether to restore best weights when stopping
    """
    
    def __init__(self, patience=10, min_delta=0.0, restore_best=True):
        self.patience = patience
        self.min_delta = min_delta
        self.restore_best = restore_best
        
        self.best_loss = float('inf')
        self.best_weights = None
        self.counter = 0
        self.should_stop = False
    
    def __call__(self, val_loss, model):
        """Check if training should stop.
        
        Returns:
            bool: True if training should stop
        """
        if val_loss < self.best_loss - self.min_delta:
            # Improvement found
            self.best_loss = val_loss
            self.best_weights = copy.deepcopy(model.state_dict())
            self.counter = 0
        else:
            # No improvement
            self.counter += 1
            if self.counter >= self.patience:
                self.should_stop = True
                if self.restore_best and self.best_weights is not None:
                    model.load_state_dict(self.best_weights)
        
        return self.should_stop
```

### Complete Training Loop

```python
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset, random_split

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(1000, 30)
y = (X[:, :5] @ torch.randn(5, 1)).squeeze() + torch.randn(1000) * 0.5

# Split into train/val/test
dataset = TensorDataset(X, y)
train_set, val_set, test_set = random_split(dataset, [600, 200, 200])
train_loader = DataLoader(train_set, batch_size=32, shuffle=True)
val_loader = DataLoader(val_set, batch_size=200)
test_loader = DataLoader(test_set, batch_size=200)

# ── Model ───────────────────────────────────────────────────────
torch.manual_seed(0)
model = nn.Sequential(
    nn.Linear(30, 256), nn.ReLU(),
    nn.Linear(256, 128), nn.ReLU(),
    nn.Linear(128, 64), nn.ReLU(),
    nn.Linear(64, 1)
)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.MSELoss()
early_stopping = EarlyStopping(patience=10, min_delta=0.001)

# ── Training Loop ──────────────────────────────────────────────
train_losses, val_losses = [], []
max_epochs = 500

for epoch in range(max_epochs):
    # Train
    model.train()
    total_train = 0
    for xb, yb in train_loader:
        loss = criterion(model(xb).squeeze(), yb)
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        total_train += loss.item() * len(xb)
    train_losses.append(total_train / len(train_set))
    
    # Validate
    model.eval()
    with torch.no_grad():
        val_loss = sum(
            criterion(model(xb).squeeze(), yb).item() * len(xb)
            for xb, yb in val_loader
        ) / len(val_set)
    val_losses.append(val_loss)
    
    # Early stopping check
    if early_stopping(val_loss, model):
        print(f"Early stopping at epoch {epoch+1}!")
        print(f"Best val loss: {early_stopping.best_loss:.4f}")
        break
    
    if (epoch + 1) % 20 == 0:
        print(f"Epoch {epoch+1}: train={train_losses[-1]:.4f}, val={val_loss:.4f}")

# ── Evaluate on test set ───────────────────────────────────────
model.eval()
with torch.no_grad():
    test_loss = sum(
        criterion(model(xb).squeeze(), yb).item() * len(xb)
        for xb, yb in test_loader
    ) / len(test_set)
print(f"\nTest loss: {test_loss:.4f}")
```

### Saving Best Model to Disk

```python
class EarlyStoppingWithDisk:
    """Early stopping that saves best model to disk."""
    
    def __init__(self, patience=10, path='best_model.pt'):
        self.patience = patience
        self.path = path
        self.best_loss = float('inf')
        self.counter = 0
    
    def __call__(self, val_loss, model):
        if val_loss < self.best_loss:
            self.best_loss = val_loss
            torch.save(model.state_dict(), self.path)
            self.counter = 0
        else:
            self.counter += 1
        
        if self.counter >= self.patience:
            model.load_state_dict(torch.load(self.path, weights_only=True))
            return True  # stop
        return False
```

### Combining with Learning Rate Scheduling

```python
# A common pattern: reduce LR first, then early stop
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, patience=5, factor=0.5, min_lr=1e-6
)
early_stopping = EarlyStopping(patience=15)  # more patient than scheduler

for epoch in range(max_epochs):
    train_one_epoch(model, optimizer, train_loader)
    val_loss = validate(model, val_loader)
    
    scheduler.step(val_loss)    # reduce LR if plateau
    if early_stopping(val_loss, model):  # stop if still no improvement
        break
```

---

## 8. Visualization

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(train_losses, label='Training Loss', color='blue', linewidth=2)
ax.plot(val_losses, label='Validation Loss', color='orange', linewidth=2)

# Mark best epoch
best_epoch = val_losses.index(min(val_losses))
ax.axvline(x=best_epoch, color='green', linestyle='--', alpha=0.7,
           label=f'Best epoch ({best_epoch+1})')
ax.scatter([best_epoch], [val_losses[best_epoch]], color='green', s=100, zorder=5)

# Mark stop epoch
stop_epoch = len(val_losses) - 1
ax.axvline(x=stop_epoch, color='red', linestyle='--', alpha=0.7,
           label=f'Stop epoch ({stop_epoch+1})')

ax.set_xlabel('Epoch', fontsize=12)
ax.set_ylabel('Loss', fontsize=12)
ax.set_title('Early Stopping: Training vs Validation Loss', fontsize=14)
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("early_stopping.png", dpi=150)
plt.show()
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Core idea | Stop training when validation loss stops improving |
| Patience | Wait P epochs of no improvement before stopping |
| min_delta | Minimum improvement to reset patience counter |
| Best model | Checkpoint weights at lowest validation loss |
| Restore | Load best weights when stopping |
| Connection to L2 | Early stopping ≈ implicit L2 regularization |
| What to monitor | Validation loss (default), accuracy, or any metric |
| Typical patience | 5–20 epochs |
| Advantage | No architecture or loss modification needed |

---

## Revision Questions

1. **Draw the typical training and validation loss curves** and explain why they diverge. When exactly should early stopping trigger?

2. **Explain the connection between early stopping and L2 regularization.** Why does stopping early keep weights small?

3. **What is the role of the patience parameter?** What happens if patience is too small or too large?

4. **Implement an EarlyStopping class** with patience, min_delta, and best weight restoration. Test it on a small overfitting problem.

5. **How does early stopping interact with learning rate scheduling?** Design a practical training procedure that uses both.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Dropout](./02-dropout.md) | [Unit 7: Regularization](./README.md) | [Data Augmentation →](./04-data-augmentation.md) |
