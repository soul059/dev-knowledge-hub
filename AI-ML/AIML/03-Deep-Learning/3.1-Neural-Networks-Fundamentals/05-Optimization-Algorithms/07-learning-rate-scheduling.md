# 7. Learning Rate Scheduling

> **Unit 5 · Optimization Algorithms** — Dynamically adjusting the learning rate during training

---

## Chapter Overview

A fixed learning rate faces a fundamental tension: a **large lr** enables fast early progress but prevents fine convergence, while a **small lr** converges precisely but wastes time in early epochs. **Learning rate scheduling** resolves this by systematically varying the learning rate during training — typically starting high and decaying. This chapter covers every major scheduling strategy, from simple step decay to the powerful one-cycle policy.

---

## 1. Why Schedule the Learning Rate?

### The Learning Rate Dilemma

```
  Large LR (α = 0.01):           Small LR (α = 0.0001):
  
  Loss ↑                          Loss ↑
       │╲                              │╲
       │ ╲  ╱╲                         │ ╲
       │  ╲╱  ╲╱╲                      │  ╲
       │        ╲╱╲  ╱╲                │   ╲
       │            ╲╱  ╲ oscillates   │    ╲
       │                               │     ╲
       │                               │      ╲
       └──────────────→ epoch          │       ╲
                                       │        ╲  very slow
  Fast but can't settle                └──────────────→ epoch
                                       Precise but wastes time
  
  Solution: START with large lr, DECAY to small lr over training!
```

### Benefits of LR Scheduling

| Phase | Learning Rate | Purpose |
|---|---|---|
| Early training | Large | Rapid exploration, escape bad init |
| Mid training | Medium | Continued progress, approaching minimum |
| Late training | Small | Fine convergence, settle into minimum |

---

## 2. Step Decay

### Algorithm

Reduce the learning rate by a **fixed factor** every N epochs:

```
  α(t) = α₀ × γ^⌊t/N⌋
```

where γ < 1 is the decay factor (e.g., 0.1), and N is the step interval.

### Example: α₀ = 0.1, γ = 0.1, step every 30 epochs

```
  α ↑
  0.1  │────────────╮
       │            │
  0.01 │            ╰──────────╮
       │                       │
  0.001│                       ╰──────────╮
       │                                  │
 0.0001│                                  ╰──────
       └───────────────────────────────────────→ epoch
        0      30      60      90      120
```

### PyTorch Implementation

```python
import torch.optim as optim

optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9)
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

for epoch in range(120):
    train_one_epoch(model, optimizer, loader)
    scheduler.step()  # call AFTER optimizer.step()
```

### MultiStepLR (Milestones)

Drop lr at specific epochs (common in ResNet training):

```python
scheduler = optim.lr_scheduler.MultiStepLR(
    optimizer, milestones=[30, 60, 90], gamma=0.1
)
# lr: 0.1 → 0.01 at epoch 30 → 0.001 at 60 → 0.0001 at 90
```

---

## 3. Exponential Decay

### Algorithm

```
  α(t) = α₀ × γᵗ     where 0 < γ < 1 (e.g., 0.95)
```

### Visualization

```
  α ↑
  0.1 │╲
      │ ╲
      │  ╲
      │   ╲╲
  0.05│     ╲╲
      │       ╲╲
      │         ╲╲╲
  0.01│            ╲╲╲╲╲
      │                  ╲╲╲╲╲╲╲╲
  0.00│                            ──────
      └──────────────────────────────────→ epoch
       0    20    40    60    80   100
```

### PyTorch Implementation

```python
scheduler = optim.lr_scheduler.ExponentialLR(optimizer, gamma=0.95)
# Every epoch: lr *= 0.95
# After 50 epochs: lr = 0.1 × 0.95^50 = 0.1 × 0.0769 ≈ 0.0077
```

---

## 4. Cosine Annealing

### Algorithm

```
  ηₜ = η_min + ½(η_max - η_min)(1 + cos(πt/T))
```

where:
- η_max = initial (maximum) learning rate
- η_min = minimum learning rate (often 0 or 1e-6)
- T = total number of training steps/epochs
- t = current step/epoch

### Shape

```
  α ↑
  η_max │───╮
        │    ╲
        │     ╲
        │      ╲
        │       ╲
        │        ╲         ← smooth cosine curve
        │         ╲╲
        │           ╲╲
        │             ╲╲╲
  η_min │                ╲╲╲───
        └──────────────────────→ epoch
         0    T/4   T/2  3T/4  T
```

### Why Cosine?

1. **Smooth** — no abrupt lr drops (unlike step decay)
2. **Spends most time at medium lr** — not too fast, not too slow
3. **Gentle final decay** — fine-tunes gradually at the end
4. **Simple** — only 3 hyperparameters (η_max, η_min, T)

### PyTorch Implementation

```python
scheduler = optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=100, eta_min=1e-6
)
```

---

## 5. Linear Warmup

### Motivation

Large learning rates at the start can cause:
- **Training instability** — large initial gradients + large lr = explosion
- **Adam bias correction issues** — moment estimates are inaccurate early on
- **Batch norm statistics** — running stats need time to stabilize

### Algorithm

Linearly increase lr from 0 (or a small value) to the target lr over W warmup steps:

```
  α(t) = α_target × t / W    for t < W
  α(t) = α_target              for t ≥ W (then apply decay)
```

### Visualization

```
  α ↑
        │         ╱────────╲
        │        ╱          ╲╲
        │       ╱             ╲╲
        │      ╱                ╲╲
        │     ╱   warmup +       ╲╲╲
        │    ╱    cosine decay     ╲╲╲
        │   ╱                        ╲╲╲
        │  ╱                            ╲╲
        │ ╱                               ╲
        │╱                                 ╲
        └────────────────────────────────────→ epoch
         0   W   ...                     T
         
         ←─→ warmup phase
```

### PyTorch Implementation

```python
def warmup_cosine_schedule(optimizer, warmup_steps, total_steps):
    def lr_lambda(step):
        if step < warmup_steps:
            return step / warmup_steps  # linear warmup
        # Cosine decay
        progress = (step - warmup_steps) / (total_steps - warmup_steps)
        return 0.5 * (1 + torch.cos(torch.tensor(progress * 3.14159)))
    
    return optim.lr_scheduler.LambdaLR(optimizer, lr_lambda)

scheduler = warmup_cosine_schedule(optimizer, warmup_steps=1000, total_steps=50000)
```

### Using PyTorch's Built-in LinearLR for Warmup

```python
# Warmup from 0.01 to 0.1 over 5 epochs
warmup = optim.lr_scheduler.LinearLR(
    optimizer, start_factor=0.1, end_factor=1.0, total_iters=5
)
# Then cosine decay
cosine = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=95, eta_min=1e-6)

# Combine them
scheduler = optim.lr_scheduler.SequentialLR(
    optimizer, schedulers=[warmup, cosine], milestones=[5]
)
```

---

## 6. ReduceLROnPlateau

### Concept

Instead of a fixed schedule, **monitor a metric** (typically validation loss) and reduce lr when it **plateaus**:

```
  If validation loss hasn't improved for `patience` epochs:
      lr = lr × factor
```

### Visualization

```
  Val Loss ↑                         LR ↑
           │╲                        0.01│──────╮
           │ ╲                            │      │ patience
           │  ╲─── plateau ──╮           │      ╰───╮
           │                  ╰╲   0.001 │          ╰──╮
           │                   ╲╲        │             │
           │                    ╲──      │             ╰──
           └──────────────────→ epoch    └──────────────→ epoch
```

### PyTorch Implementation

```python
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',        # minimize validation loss
    factor=0.1,        # multiply lr by 0.1
    patience=10,       # wait 10 epochs before reducing
    min_lr=1e-6,       # don't go below this
    verbose=True       # print when lr is reduced
)

for epoch in range(200):
    train_loss = train_one_epoch(model, optimizer, train_loader)
    val_loss = validate(model, val_loader)
    scheduler.step(val_loss)  # pass the metric!
```

---

## 7. One-Cycle Policy

### Concept (Smith & Topin, 2019)

The one-cycle policy consists of:
1. **Warmup**: Increase lr from lr_min to lr_max over ~30% of training
2. **Decay**: Decrease lr from lr_max back to lr_min over ~70% of training
3. **Annihilation**: Final very small lr phase (optional)

```
  α ↑
        │           ╱╲
        │          ╱  ╲
  lr_max│         ╱    ╲
        │        ╱      ╲
        │       ╱        ╲
        │      ╱          ╲
        │     ╱            ╲
        │    ╱              ╲
  lr_min│───╱                ╲──╮
        │                       ╰─ annihilation
        └──────────────────────────→ epoch
         0   30%        100%
         
  Momentum (inverse of lr):
  β ↑
  0.95│╲                    ╱──
      │ ╲                  ╱
  0.85│  ╲                ╱
      │   ╲──────────────╱
      └──────────────────────→ epoch
```

Key insight: **Momentum decreases when lr increases** (and vice versa).

### PyTorch Implementation

```python
scheduler = optim.lr_scheduler.OneCycleLR(
    optimizer,
    max_lr=0.01,           # peak learning rate
    total_steps=len(loader) * num_epochs,
    pct_start=0.3,         # 30% warmup
    anneal_strategy='cos', # cosine annealing
    div_factor=25,         # initial_lr = max_lr/25
    final_div_factor=1e4   # final_lr = initial_lr/1e4
)

# Call scheduler.step() after EACH BATCH (not each epoch!)
for epoch in range(num_epochs):
    for batch in loader:
        loss = compute_loss(model, batch)
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        scheduler.step()  # per-batch!
```

---

## 8. Comparison Table

| Schedule | Shape | Hyperparameters | When to Use |
|---|---|---|---|
| **Step Decay** | Staircase | step_size, γ | Simple baseline, ResNet training |
| **Exponential** | Smooth decay | γ | When you want continuous decay |
| **Cosine Annealing** | Cosine curve | T_max, η_min | Modern default, smooth decay |
| **Linear Warmup** | Ramp up | warmup_steps | Large models, Transformers |
| **Warmup + Cosine** | Ramp + cosine | warmup + T_max | BERT, ViT, most modern training |
| **ReduceLROnPlateau** | Adaptive | patience, factor | When you don't know total epochs |
| **One-Cycle** | Triangle | max_lr, pct_start | Fast training, super-convergence |
| **Constant** | Flat | — | Short fine-tuning |

### Schedule Shapes

```
  Step:          Exponential:     Cosine:          One-Cycle:
  ┌──╮           ╲                ╲                  ╱╲
  │  ╰──╮         ╲╲               ╲╲               ╱  ╲
  │     ╰──╮        ╲╲╲              ╲╲╲            ╱    ╲
  │        ╰──        ╲╲╲╲             ╲╲╲╲        ╱      ╲──
  └──────────→      ───────→        ─────────→   ──────────→
```

---

## 9. Practical Guidelines

### Choosing a Schedule

```
  ┌──────────────────────────────────────────────────┐
  │  START HERE: What are you training?              │
  │                                                  │
  │  Transformer (BERT, GPT, ViT)?                   │
  │  └─→ Warmup + Cosine Decay                      │
  │                                                  │
  │  CNN (ResNet, EfficientNet)?                      │
  │  └─→ Cosine Annealing or Step Decay              │
  │                                                  │
  │  Don't know total epochs?                         │
  │  └─→ ReduceLROnPlateau                           │
  │                                                  │
  │  Want fastest possible training?                  │
  │  └─→ One-Cycle Policy                            │
  │                                                  │
  │  Fine-tuning a pretrained model?                  │
  │  └─→ Small constant lr or linear decay           │
  └──────────────────────────────────────────────────┘
```

### Common Recipes

```python
# ── Recipe 1: ImageNet ResNet-50 (90 epochs) ───────────────────
optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=1e-4)
scheduler = optim.lr_scheduler.MultiStepLR(optimizer, milestones=[30, 60, 80], gamma=0.1)

# ── Recipe 2: BERT Fine-tuning (3 epochs) ─────────────────────
optimizer = optim.AdamW(model.parameters(), lr=2e-5, weight_decay=0.01)
total_steps = len(train_loader) * 3
warmup_steps = int(0.1 * total_steps)
scheduler = optim.lr_scheduler.OneCycleLR(optimizer, max_lr=2e-5,
                                           total_steps=total_steps, pct_start=0.1)

# ── Recipe 3: General Training (unknown duration) ──────────────
optimizer = optim.Adam(model.parameters(), lr=1e-3)
scheduler = optim.lr_scheduler.ReduceLROnPlateau(optimizer, patience=5, factor=0.5)
```

---

## 10. Worked Example: Cosine Annealing

### Problem
Compute the learning rate at epochs 0, 25, 50, 75, 100 with cosine annealing:
η_max = 0.1, η_min = 0.001, T = 100.

### Solution

Using: η(t) = η_min + ½(η_max - η_min)(1 + cos(πt/T))

```
t = 0:   η = 0.001 + 0.5·(0.099)·(1 + cos(0))     = 0.001 + 0.0495·2 = 0.1000
t = 25:  η = 0.001 + 0.5·(0.099)·(1 + cos(π/4))    = 0.001 + 0.0495·1.707 = 0.0855
t = 50:  η = 0.001 + 0.5·(0.099)·(1 + cos(π/2))    = 0.001 + 0.0495·1 = 0.0505
t = 75:  η = 0.001 + 0.5·(0.099)·(1 + cos(3π/4))   = 0.001 + 0.0495·0.293 = 0.0155
t = 100: η = 0.001 + 0.5·(0.099)·(1 + cos(π))       = 0.001 + 0.0495·0 = 0.0010
```

```
  α ↑
  0.10 │──╮
       │   ╲
  0.085│    ╲       ← t=25
       │     ╲
  0.050│      ╲     ← t=50 (half-life: exactly midpoint)
       │       ╲╲
  0.015│         ╲  ← t=75
       │          ╲╲
  0.001│            ╲ ← t=100
       └─────────────────→ epoch
        0  25  50  75  100
```

---

## 11. Python Implementation (Full Example)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
import matplotlib.pyplot as plt
import numpy as np

# ── Data & Model ────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(3000, 20)
y = (X @ torch.randn(20, 1) + torch.randn(3000, 1) * 0.5).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Compare Schedules ──────────────────────────────────────────
schedules = {}
epochs = 100

for name, make_scheduler in [
    ("Constant", lambda opt: None),
    ("Step(30, γ=0.1)", lambda opt: optim.lr_scheduler.StepLR(opt, 30, 0.1)),
    ("Exponential(0.97)", lambda opt: optim.lr_scheduler.ExponentialLR(opt, 0.97)),
    ("Cosine", lambda opt: optim.lr_scheduler.CosineAnnealingLR(opt, epochs, 1e-5)),
]:
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(20, 128), nn.ReLU(), nn.Linear(128, 1))
    opt = optim.SGD(model.parameters(), lr=0.1, momentum=0.9)
    sched = make_scheduler(opt)
    
    losses, lrs = [], []
    for epoch in range(epochs):
        lrs.append(opt.param_groups[0]['lr'])
        total = 0
        for xb, yb in loader:
            loss = nn.MSELoss()(model(xb).squeeze(), yb)
            opt.zero_grad(); loss.backward(); opt.step()
            total += loss.item() * len(xb)
        losses.append(total / len(X))
        if sched:
            sched.step()
    
    schedules[name] = (losses, lrs)

# ── Plot ────────────────────────────────────────────────────────
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

for name, (losses, lrs) in schedules.items():
    ax1.plot(losses, label=name)
    ax2.plot(lrs, label=name)

ax1.set_xlabel("Epoch"); ax1.set_ylabel("Loss"); ax1.set_title("Training Loss")
ax1.legend(); ax1.grid(True, alpha=0.3)
ax2.set_xlabel("Epoch"); ax2.set_ylabel("Learning Rate"); ax2.set_title("LR Schedule")
ax2.legend(); ax2.set_yscale("log"); ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig("lr_schedules_comparison.png", dpi=150)
plt.show()
```

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Step decay | α(t) = α₀ × γ^⌊t/N⌋ |
| Exponential decay | α(t) = α₀ × γᵗ |
| Cosine annealing | η = η_min + ½(η_max-η_min)(1+cos(πt/T)) |
| Linear warmup | α(t) = α_target × t/W for t < W |
| ReduceLROnPlateau | Reduce lr when metric plateaus |
| One-cycle | Warmup → peak → decay → annihilation |
| Modern default | Warmup + cosine decay |
| PyTorch | `torch.optim.lr_scheduler.*` classes |

---

## Revision Questions

1. **Compare step decay and cosine annealing.** What are the advantages of the smooth cosine curve over abrupt drops?

2. **Derive the cosine annealing formula.** Show that at t=0 it gives η_max and at t=T it gives η_min, and at t=T/2 it gives the midpoint.

3. **Why is warmup important for Transformer training?** What would happen without it (relate to Adam's bias correction and large gradients)?

4. **Explain the one-cycle policy.** Why does it cycle both learning rate and momentum in opposite directions?

5. **When would you use ReduceLROnPlateau over a fixed schedule?** What are the advantages and disadvantages?

6. **Write a complete training loop** with warmup (10% of steps) + cosine annealing using PyTorch schedulers.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Adam Optimizer](./06-adam-optimizer.md) | [Unit 5: Optimization](./README.md) | [Warm Restarts →](./08-warm-restarts.md) |
