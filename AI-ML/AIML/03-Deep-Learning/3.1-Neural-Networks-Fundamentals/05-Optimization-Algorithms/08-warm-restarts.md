# 8. Warm Restarts, Cyclic Learning Rates & SWA

> **Unit 5 · Optimization Algorithms** — Restarting and cycling the learning rate for better solutions

---

## Chapter Overview

Standard learning rate schedules monotonically decrease the lr — once it's small, it stays small. **Warm restarts** and **cyclic learning rates** break this pattern by periodically **resetting** the learning rate to a large value, allowing the optimizer to escape local minima and explore new regions of the loss landscape. Combined with **snapshot ensembles** and **stochastic weight averaging (SWA)**, these techniques can dramatically improve generalization without additional training cost.

---

## 1. Cosine Annealing with Warm Restarts (SGDR)

### Concept (Loshchilov & Hutter, 2017)

Instead of a single cosine decay over all training, perform **multiple cosine annealing cycles**, each followed by a "warm restart" that resets the lr to η_max:

```
  α ↑
  η_max│──╮    ╭──╮        ╭──╮              ╭──╮
       │   ╲  ╱    ╲      ╱    ╲            ╱    ╲
       │    ╲╱      ╲    ╱      ╲          ╱      ╲
       │             ╲  ╱        ╲        ╱        ╲
       │              ╲╱          ╲      ╱          ╲
  η_min│                           ╲    ╱            ╲
       │                            ╲  ╱              ╲
       │                             ╲╱                ╲
       └────────────────────────────────────────────────→ epoch
        0   T₀   T₀  ←── T₀·Tₘᵤₗₜ ──→  ← T₀·Tₘᵤₗₜ² →
        
       Cycle 1   Cycle 2 (longer)     Cycle 3 (even longer)
```

### Formula

Within each restart cycle:

```
  ηₜ = η_min + ½(η_max - η_min)(1 + cos(π · T_cur / T_i))
```

where:
- **T_cur** = number of epochs since last restart
- **T_i** = length of current cycle
- **T₀** = initial cycle length
- **T_mult** = cycle length multiplier (T_{i+1} = T_mult × T_i)

### Restart Schedule

| Cycle | Length | Start Epoch | End Epoch |
|---|---|---|---|
| 1 | T₀ | 0 | T₀ |
| 2 | T₀ · T_mult | T₀ | T₀ + T₀·T_mult |
| 3 | T₀ · T_mult² | ... | ... |

With T₀=10, T_mult=2:
- Cycle 1: epochs 0–10 (10 epochs)
- Cycle 2: epochs 10–30 (20 epochs)
- Cycle 3: epochs 30–70 (40 epochs)
- Cycle 4: epochs 70–150 (80 epochs)

### Why Warm Restarts Help

```
  Loss Landscape (1D slice):
  
  Loss ↑
       │       ╱╲
       │      ╱  ╲    ╱╲
       │     ╱    ╲  ╱  ╲     ╱╲
       │    ╱  A   ╲╱  B ╲   ╱  ╲
       │   ╱        ★     ╲ ╱  C ╲
       │  ╱                ╲      ╲
       │ ╱                  ★      ╲
       └──────────────────────────────→ θ
       
  Without restarts: converge to A (first minimum found)
  With restarts: restart from A → high lr → explore → find B or C (potentially better!)
```

1. **Escape suboptimal local minima** — large lr after restart can jump out
2. **Explore wider loss landscape** — each cycle starts fresh exploration
3. **Find flatter minima** — wide minima are more robust and discovered by large lr
4. **Enable snapshot ensembles** — collect model checkpoints at end of each cycle

---

## 2. Snapshot Ensembles

### Concept (Huang et al., 2017)

Since each warm restart cycle converges to a **different local minimum**, save a snapshot of the model at the end of each cycle. The snapshots form an **ensemble** for free!

```
  lr ↑
     │╲   ╱╲     ╱╲
     │ ╲ ╱  ╲   ╱  ╲
     │  ╲╱    ╲ ╱    ╲
     │   📸    ╲╱     📸    ← save snapshot at each lr minimum
     │   ①     📸      ③
     │          ②
     └──────────────────→ epoch
     
  At inference: average predictions from snapshots ①, ②, ③
  
  ŷ_ensemble = (1/M) Σₘ f(x; θₘ)
```

### Benefits

| Property | Value |
|---|---|
| Additional training cost | **Zero** (just save checkpoints) |
| Ensemble diversity | Different local minima → diverse predictions |
| Accuracy improvement | Typically 1–3% over single model |
| Storage | M × model size |

### Implementation

```python
import copy

snapshots = []

for cycle in range(num_cycles):
    for epoch in range(cycle_length):
        train_one_epoch(model, optimizer, loader)
        scheduler.step()
    
    # Save snapshot at the end of each cycle (when lr is minimum)
    snapshots.append(copy.deepcopy(model.state_dict()))

# Ensemble prediction
def ensemble_predict(models, x):
    preds = [model(x) for model in models]
    return torch.stack(preds).mean(dim=0)
```

---

## 3. Stochastic Weight Averaging (SWA)

### Concept (Izmailov et al., 2018)

Instead of using the final model weights, **average** the weights from multiple points near the end of training:

```
  θ_SWA = (1/K) Σₖ θₖ
```

where θₖ are model weights sampled at regular intervals during the final phase of training.

### Why SWA Works

```
  Loss surface cross-section:
  
  Loss ↑
       │     ╱╲   ╱╲   ╱╲
       │    ╱  ╲ ╱  ╲ ╱  ╲
       │   ╱  ●₁ ● ●₂    ╲
       │  ╱   θ₁  SWA θ₂   ╲      SWA point (average) sits at
       │ ╱         ★          ╲     a WIDER minimum than either
       │╱          ↑           ╲    individual point
       │     weight average
       └──────────────────────────→ θ
  
  ●₁, ●₂ = individual SGD solutions (at different times)
  ★ = SWA average → centered in a FLAT minimum → better generalization
```

Key insight: SGD with high lr explores the **boundary** of flat minima, while the average of these points lies at the **center** of the flat region.

### SWA Algorithm

```
  Phase 1: Train normally (e.g., 75% of total epochs)
  Phase 2: SWA phase (last 25%):
      - Use constant or cyclic lr
      - Every c epochs, update the running average:
      
      θ_SWA = (θ_SWA · n_models + θ_current) / (n_models + 1)
      
  After training: Update batch normalization statistics with SWA weights
```

### PyTorch Implementation

```python
from torch.optim.swa_utils import AveragedModel, SWALR

model = MyModel()
optimizer = torch.optim.SGD(model.parameters(), lr=0.1, momentum=0.9)

# SWA model wraps original model
swa_model = AveragedModel(model)
swa_scheduler = SWALR(optimizer, swa_lr=0.05)  # constant lr for SWA phase
normal_scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=100
)

swa_start = 75  # start SWA at epoch 75

for epoch in range(100):
    train_one_epoch(model, optimizer, train_loader)
    
    if epoch >= swa_start:
        swa_model.update_parameters(model)  # update running average
        swa_scheduler.step()
    else:
        normal_scheduler.step()

# IMPORTANT: Update batch norm statistics for SWA model
torch.optim.swa_utils.update_bn(train_loader, swa_model)

# Use swa_model for inference
test_loss = evaluate(swa_model, test_loader)
```

---

## 4. Cyclic Learning Rates

### Concept (Smith, 2017)

Cyclically vary the learning rate between a minimum and maximum value:

```
  Triangular Policy:
  α ↑
     │  ╱╲    ╱╲    ╱╲    ╱╲
     │ ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲
     │╱    ╲╱    ╲╱    ╲╱    ╲
     └──────────────────────────→ iteration
  
  Triangular2 (decaying amplitude):
  α ↑
     │  ╱╲
     │ ╱  ╲  ╱╲
     │╱    ╲╱  ╲╱╲
     │          ╲╱╲╱
     └──────────────────────────→ iteration
```

### Why Cycling Works

1. **Large lr enables exploration** — jumps to new regions
2. **Small lr enables exploitation** — fine-tunes near a minimum
3. **Alternating** — repeatedly refines and explores
4. **Finds wider minima** — passes through narrow minima with high lr

### PyTorch Implementation

```python
scheduler = torch.optim.lr_scheduler.CyclicLR(
    optimizer,
    base_lr=1e-4,           # minimum lr
    max_lr=1e-2,            # maximum lr
    step_size_up=2000,      # iterations to go from base to max
    mode='triangular2',     # decaying triangular
    cycle_momentum=True     # cycle momentum inversely
)
```

---

## 5. Smith's Learning Rate Range Test

### Concept (Smith, 2017)

Systematically find the optimal learning rate range by training for a few epochs with an exponentially increasing lr:

```
  Algorithm:
  1. Start with a very small lr (e.g., 1e-7)
  2. Increase lr exponentially each batch
  3. Record the loss at each lr
  4. Plot loss vs lr → find the sweet spot
  
  Loss ↑
       │           ╱
       │          ╱  ← loss explodes (lr too high)
       │         ╱
       │╲       ╱
       │ ╲     ╱
       │  ╲   ╱
       │   ╲_╱   ← minimum loss (best lr)
       │    ↑
       │    optimal range
       └──────────────────→ lr (log scale)
       1e-7  1e-5  1e-3  1e-1  10
       
  Choose: base_lr ≈ 1/10 × optimal, max_lr ≈ optimal
```

### Implementation

```python
def lr_range_test(model, train_loader, optimizer, start_lr=1e-7, end_lr=10,
                  num_steps=100):
    """Find optimal learning rate range."""
    lrs, losses = [], []
    lr_mult = (end_lr / start_lr) ** (1 / num_steps)
    
    optimizer.param_groups[0]['lr'] = start_lr
    best_loss = float('inf')
    
    for i, (x, y) in enumerate(train_loader):
        if i >= num_steps:
            break
        
        # Forward + backward
        pred = model(x)
        loss = nn.MSELoss()(pred.squeeze(), y)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        # Record
        lrs.append(optimizer.param_groups[0]['lr'])
        losses.append(loss.item())
        
        # Stop if loss explodes
        if loss.item() > 4 * best_loss:
            break
        best_loss = min(best_loss, loss.item())
        
        # Increase lr
        optimizer.param_groups[0]['lr'] *= lr_mult
    
    return lrs, losses

# Plot results
import matplotlib.pyplot as plt

lrs, losses = lr_range_test(model, train_loader, optimizer)
plt.plot(lrs, losses)
plt.xscale('log')
plt.xlabel('Learning Rate')
plt.ylabel('Loss')
plt.title('LR Range Test')
plt.savefig('lr_range_test.png', dpi=150)
plt.show()
```

---

## 6. PyTorch CosineAnnealingWarmRestarts

### API

```python
scheduler = torch.optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer,
    T_0=10,         # initial cycle length (epochs)
    T_mult=2,       # cycle length multiplier
    eta_min=1e-6    # minimum learning rate
)
```

### Complete Training Example

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(5000, 30)
y = (X @ torch.randn(30, 1) + 0.5 * torch.randn(5000, 1)).squeeze()
train_loader = DataLoader(TensorDataset(X[:4000], y[:4000]), batch_size=128, shuffle=True)
val_loader = DataLoader(TensorDataset(X[4000:], y[4000:]), batch_size=256)

# ── Model ───────────────────────────────────────────────────────
torch.manual_seed(0)
model = nn.Sequential(
    nn.Linear(30, 256), nn.ReLU(), nn.BatchNorm1d(256),
    nn.Linear(256, 128), nn.ReLU(), nn.BatchNorm1d(128),
    nn.Linear(128, 1)
)

optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=1e-4)
scheduler = optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer, T_0=10, T_mult=2, eta_min=1e-5
)

# ── Training Loop with Warm Restarts ──────────────────────────
epochs = 70  # cycles: 10 + 20 + 40 = 70
train_losses, val_losses, learning_rates = [], [], []

for epoch in range(epochs):
    # Train
    model.train()
    total_loss = 0
    for xb, yb in train_loader:
        loss = nn.MSELoss()(model(xb).squeeze(), yb)
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        total_loss += loss.item() * len(xb)
    
    train_losses.append(total_loss / len(train_loader.dataset))
    learning_rates.append(optimizer.param_groups[0]['lr'])
    
    # Validate
    model.eval()
    with torch.no_grad():
        val_loss = sum(
            nn.MSELoss()(model(xb).squeeze(), yb).item() * len(xb)
            for xb, yb in val_loader
        ) / len(val_loader.dataset)
    val_losses.append(val_loss)
    
    scheduler.step()

# ── Plot Results ────────────────────────────────────────────────
import matplotlib.pyplot as plt

fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 8), sharex=True)

ax1.plot(train_losses, label='Train', color='blue')
ax1.plot(val_losses, label='Val', color='orange')
ax1.set_ylabel('Loss')
ax1.legend()
ax1.set_title('Cosine Annealing with Warm Restarts (T₀=10, T_mult=2)')
ax1.grid(True, alpha=0.3)

ax2.plot(learning_rates, color='green')
ax2.set_ylabel('Learning Rate')
ax2.set_xlabel('Epoch')
ax2.set_yscale('log')
ax2.grid(True, alpha=0.3)

# Mark restart points
for restart_epoch in [10, 30]:
    ax1.axvline(x=restart_epoch, color='red', linestyle='--', alpha=0.5)
    ax2.axvline(x=restart_epoch, color='red', linestyle='--', alpha=0.5, label='restart')

plt.tight_layout()
plt.savefig("warm_restarts_training.png", dpi=150)
plt.show()
```

---

## 7. Combining Techniques

### Warm Restarts + SWA

```
  lr ↑
     │╲   ╱╲        ╱╲                    Phase 1: Warm restarts
     │ ╲ ╱  ╲      ╱  ╲                   to explore landscape
     │  ╲╱    ╲   ╱    ╲
     │         ╲ ╱      ╲                 Phase 2: SWA to average
     │          ╲        ╲──── constant    solutions found
     │                        ← SWA →
     │                   📸📸📸📸📸📸📸       (collect snapshots)
     └──────────────────────────────→ epoch
```

### Warm Restarts + Snapshot Ensemble

```python
import copy

snapshots = []
scheduler = optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer, T_0=20, T_mult=1
)

for epoch in range(100):
    train_one_epoch(model, optimizer, train_loader)
    scheduler.step()
    
    # Save at cycle boundaries (when lr resets to max)
    if (epoch + 1) % 20 == 0:
        snapshots.append(copy.deepcopy(model))
        print(f"Snapshot {len(snapshots)} saved at epoch {epoch+1}")

# Ensemble inference
def predict_ensemble(snapshots, x):
    with torch.no_grad():
        preds = torch.stack([m(x) for m in snapshots])
    return preds.mean(dim=0)  # average predictions
```

---

## 8. When to Use Each Technique

| Technique | Best For | Key Benefit |
|---|---|---|
| **Cosine + Warm Restarts** | General training | Explore multiple minima |
| **Snapshot Ensembles** | When ensemble is needed but budget is limited | Free ensemble |
| **SWA** | Improving any trained model | +1-2% accuracy, trivial to add |
| **Cyclic LR** | Finding optimal lr range first, then training | Super-convergence |
| **LR Range Test** | Before any training | Calibrate lr bounds |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| SGDR | Cosine annealing with periodic lr resets to η_max |
| Restart formula | ηₜ = η_min + ½(η_max-η_min)(1+cos(π·T_cur/T_i)) |
| T_mult | Cycle length grows: T_{i+1} = T_mult · T_i |
| Snapshot Ensembles | Save model at end of each cycle → free ensemble |
| SWA | Average weights from late training → flatter minimum |
| Cyclic LR | Continuously cycle between lr_min and lr_max |
| LR Range Test | Exponentially increase lr, find optimal range |
| PyTorch SGDR | `CosineAnnealingWarmRestarts(opt, T_0, T_mult)` |
| PyTorch SWA | `AveragedModel(model)` + `SWALR(optimizer)` |
| PyTorch Cyclic | `CyclicLR(opt, base_lr, max_lr)` |

---

## Revision Questions

1. **Explain why warm restarts can find better minima than a single cosine annealing schedule.** Use the loss landscape to illustrate.

2. **How do snapshot ensembles work?** Why are snapshots from different restart cycles more diverse than snapshots from the same cycle?

3. **What is SWA, and why does averaging weights from the boundary of a flat minimum produce a point at its center?**

4. **Describe Smith's LR range test procedure.** How do you interpret the resulting loss-vs-lr plot to choose base_lr and max_lr?

5. **Compare SGDR (warm restarts) with the one-cycle policy.** When would you prefer each?

6. **Implement a training loop with CosineAnnealingWarmRestarts and snapshot ensembles.** Show the ensemble prediction code.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Learning Rate Scheduling](./07-learning-rate-scheduling.md) | [Unit 5: Optimization](./README.md) | [Unit 6: Weight Initialization →](../06-Weight-Initialization/01-why-initialization-matters.md) |
