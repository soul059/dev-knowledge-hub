# Learning Rate Scheduling for RNNs

> **Unit 9, Chapter 3** — Adaptive learning rate strategies for stable and efficient RNN training: warmup, decay, cyclical, and practical recommendations.

---

## 📋 Overview

The learning rate is arguably the **most important hyperparameter** in training RNNs. Too high and gradients explode; too low and training stalls. Unlike CNNs, RNNs are particularly sensitive to learning rate due to their recurrent dynamics — small changes in weights compound over many time steps. **Learning rate scheduling** systematically adjusts the learning rate during training to balance fast convergence with training stability.

---

## 🌡️ Why RNNs Need Special LR Treatment

### Sensitivity Analysis

```
Effect of learning rate on RNN training:

Loss
  │
  │  ╲  LR = 0.1 (too high)
  │   ╲╱╲╱╲╱╲ NaN!
  │
  │   ──╲                      LR = 0.001 (good)
  │      ──╲___
  │            ──────────────── converged
  │
  │   ─────────────╲
  │                 ─────────── LR = 0.00001 (too low)
  │                             still decreasing slowly...
  └──────────────────────────── Epochs

RNNs amplify LR effects because:
• Same weights applied T times (sequence length)
• Effective step size ∝ LR × T  
• Small LR changes → large training behavior changes
```

---

## 📉 Learning Rate Decay Strategies

### 1. Step Decay

Reduce LR by a factor every N epochs:

```
LR(epoch) = LR₀ × γ^⌊epoch / step_size⌋

LR₀ = 0.001, γ = 0.5, step_size = 10

  LR
  │
0.001│────────────┐
     │            │
0.0005│            └────────────┐
     │                         │
0.00025│                        └────────────┐
     │                                      │
     └──────────────────────────────────────── Epoch
     0         10         20         30
```

**PyTorch:**
```python
scheduler = torch.optim.lr_scheduler.StepLR(
    optimizer, step_size=10, gamma=0.5
)
```

### 2. Exponential Decay

Smooth continuous decay:

```
LR(epoch) = LR₀ × γ^epoch

LR₀ = 0.001, γ = 0.95

  LR
  │
0.001│╲
     │ ╲
     │  ╲
     │   ╲__
     │      ╲___
     │          ╲_____
     │                ╲__________
     └──────────────────────────── Epoch
     0    10   20   30   40   50
```

**PyTorch:**
```python
scheduler = torch.optim.lr_scheduler.ExponentialLR(
    optimizer, gamma=0.95
)
```

### 3. Cosine Annealing

Smooth decay following cosine curve, optionally with warm restarts:

```
LR(t) = LR_min + 0.5 × (LR_max - LR_min) × (1 + cos(π × t / T_max))

  LR
  │
LR_max│╲
      │  ╲
      │    ╲
      │      ╲
      │        ╲___
      │             ╲______
LR_min│                     ╲___
      └────────────────────────── Epoch
      0                    T_max
```

**PyTorch:**
```python
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=50, eta_min=1e-6
)
```

### 4. Reduce on Plateau

Reduce LR when a metric stops improving:

```
Monitor validation loss:

  Val Loss
  │╲
  │  ╲___
  │       ╲_________  ← plateau detected (patience=5)
  │                  │ LR reduced by factor
  │                  ╲___
  │                       ╲_________  ← another plateau
  │                                  │ LR reduced again
  └──────────────────────────────────── Epoch

Most practical for RNNs — adapts to actual training dynamics.
```

**PyTorch:**
```python
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',        # minimize validation loss
    factor=0.5,        # multiply LR by 0.5
    patience=5,        # wait 5 epochs of no improvement
    min_lr=1e-6,       # don't go below this
    verbose=True       # print when LR changes
)

# Usage (note: step takes the metric value)
for epoch in range(epochs):
    train_loss = train(model)
    val_loss = validate(model)
    scheduler.step(val_loss)  # Pass the metric!
```

---

## 🔥 Learning Rate Warmup

### Why Warmup Matters for RNNs

At initialization, hidden states are random and gradients are unreliable. Large learning rates early on can push weights into bad regions from which recovery is difficult.

```
Without warmup:                With warmup:
                               
  Loss                           Loss
  │ ╱╲                           │
  │╱  ╲ ╱╲ (unstable start)     │ ╲
  │    ╲╱  ╲                     │  ╲___
  │         ╲___                 │       ╲___
  │              ───             │            ───
  └──────────────── Epoch        └──────────────── Epoch

Warmup lets the model "find its footing" before taking large steps.
```

### Linear Warmup

```
LR(step) = LR_max × min(step / warmup_steps, 1.0)

  LR
  │                    ┌──────────────────
  │                   ╱
  │                 ╱
  │               ╱
  │             ╱
  │           ╱
  │         ╱
  │       ╱
  │     ╱     Warmup phase      Constant phase
  └───────────┬──────────────────── Step
  0     warmup_steps
```

**PyTorch Implementation:**
```python
class WarmupScheduler(torch.optim.lr_scheduler._LRScheduler):
    def __init__(self, optimizer, warmup_steps, base_lr, last_epoch=-1):
        self.warmup_steps = warmup_steps
        self.base_lr = base_lr
        super().__init__(optimizer, last_epoch)
    
    def get_lr(self):
        if self.last_epoch < self.warmup_steps:
            # Linear warmup
            warmup_factor = self.last_epoch / self.warmup_steps
            return [self.base_lr * warmup_factor for _ in self.base_lrs]
        else:
            return [self.base_lr for _ in self.base_lrs]
```

### Warmup + Cosine Decay (Popular Recipe)

```
  LR
  │
LR_max│         ╱╲
      │        ╱  ╲
      │       ╱    ╲
      │      ╱      ╲___
      │     ╱            ╲______
      │    ╱                     ╲___
LR_min│   ╱                          ╲
      └──────────────────────────────── Step
         ↑                            ↑
      warmup                       T_max
```

```python
def warmup_cosine_lr(step, warmup_steps, total_steps, base_lr, min_lr=1e-6):
    if step < warmup_steps:
        return base_lr * step / warmup_steps
    else:
        progress = (step - warmup_steps) / (total_steps - warmup_steps)
        return min_lr + 0.5 * (base_lr - min_lr) * (1 + math.cos(math.pi * progress))

# Using LambdaLR
scheduler = torch.optim.lr_scheduler.LambdaLR(
    optimizer,
    lr_lambda=lambda step: warmup_cosine_lr(step, 1000, 50000, 0.001) / 0.001
)
```

---

## 🔄 Cyclical Learning Rates

### Concept

Instead of monotonically decreasing, oscillate between bounds:

```
  LR
  │
max_lr│    ╱╲      ╱╲      ╱╲
      │   ╱  ╲    ╱  ╲    ╱  ╲
      │  ╱    ╲  ╱    ╲  ╱    ╲
min_lr│ ╱      ╲╱      ╲╱      ╲
      └──────────────────────────── Step
       ← cycle →
```

### Benefits for RNNs

1. **Escapes local minima** — periodic high LR helps jump out of sharp minima
2. **Explores loss landscape** — finds flatter, more generalizable minima
3. **Acts as regularizer** — prevents settling into overfit solutions

**PyTorch:**
```python
scheduler = torch.optim.lr_scheduler.CyclicLR(
    optimizer,
    base_lr=1e-4,
    max_lr=1e-2,
    step_size_up=2000,     # steps to increase LR
    mode='triangular2',     # halve max_lr each cycle
    cycle_momentum=False    # important for Adam
)
```

### One-Cycle Policy

Single large cycle — popular for training to convergence:

```
  LR
  │
max│          ╱╲
   │        ╱    ╲
   │      ╱        ╲
   │    ╱            ╲
   │  ╱                ╲
min│ ╱                    ╲___
   └───────────────────────────── Step
    30% warmup    70% decay
```

```python
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer,
    max_lr=0.01,
    total_steps=total_training_steps,
    pct_start=0.3,         # 30% warmup
    anneal_strategy='cos'
)
```

---

## 🧮 Worked Example: Complete LR Schedule

### Training an LSTM Language Model

```python
import torch
import torch.nn as nn
import math

class LSTMLanguageModel(nn.Module):
    def __init__(self, vocab_size, embed_size, hidden_size, num_layers):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_size)
        self.lstm = nn.LSTM(embed_size, hidden_size, num_layers,
                           batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_size, vocab_size)
        self.hidden_size = hidden_size
        self.num_layers = num_layers
    
    def forward(self, x, hidden=None):
        emb = self.embedding(x)
        out, hidden = self.lstm(emb, hidden)
        out = self.fc(out)
        return out, hidden

# Configuration
model = LSTMLanguageModel(
    vocab_size=10000, embed_size=256,
    hidden_size=512, num_layers=2
)

# Optimizer
optimizer = torch.optim.Adam(model.parameters(), lr=0.001, weight_decay=1e-5)

# Schedule: ReduceLROnPlateau (most practical)
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', factor=0.5, patience=3, min_lr=1e-6
)

# Training loop
best_val_loss = float('inf')
for epoch in range(50):
    model.train()
    train_loss = 0
    for batch_x, batch_y in train_loader:
        optimizer.zero_grad()
        output, _ = model(batch_x)
        loss = nn.functional.cross_entropy(
            output.view(-1, 10000), batch_y.view(-1)
        )
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=5.0)
        optimizer.step()
        train_loss += loss.item()
    
    # Validation
    model.eval()
    val_loss = 0
    with torch.no_grad():
        for batch_x, batch_y in val_loader:
            output, _ = model(batch_x)
            loss = nn.functional.cross_entropy(
                output.view(-1, 10000), batch_y.view(-1)
            )
            val_loss += loss.item()
    
    val_loss /= len(val_loader)
    
    # Step scheduler with validation loss
    scheduler.step(val_loss)
    
    # Log current LR
    current_lr = optimizer.param_groups[0]['lr']
    print(f"Epoch {epoch+1}: Val Loss={val_loss:.4f}, LR={current_lr:.2e}")
    
    # Early stopping
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), 'best_model.pth')
```

---

## 📊 Comparing Schedules for RNNs

| Schedule | Best For | Pros | Cons |
|----------|---------|------|------|
| Step Decay | Baseline experiments | Simple, predictable | Requires manual tuning of step points |
| Exponential Decay | Smooth convergence | No sudden drops | May decay too fast or too slow |
| Cosine Annealing | Fixed training budget | Smooth, well-studied | Must know T_max upfront |
| ReduceOnPlateau | Most practical use | Adapts to training | Requires validation metric |
| Warmup + Decay | Large models | Stable start, good convergence | Extra hyperparameter |
| Cyclical | Exploration | Can find better minima | Noisy training curves |
| One-Cycle | Fast training | Super-convergence | Single run, no early stopping |

### Recommended Defaults

```
┌─────────────────────────────────────────────────────┐
│           Recommended LR Recipes for RNNs           │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Small dataset / Quick experiment:                  │
│    Adam, LR=0.001, ReduceLROnPlateau(patience=5)    │
│                                                     │
│  Medium dataset / Standard training:                │
│    Adam, LR=0.001, warmup=1000 steps                │
│    + Cosine decay to 1e-5                           │
│                                                     │
│  Large dataset / Long training:                     │
│    SGD, LR=1.0, warmup=5000 steps                   │
│    + Cosine decay, gradient clipping=0.25            │
│    (This is the AWD-LSTM recipe)                    │
│                                                     │
│  Research / Exploration:                            │
│    Adam, OneCycleLR(max_lr=0.01)                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## ⚠️ Common Pitfalls

### 1. Forgetting to Step the Scheduler

```python
# WRONG — scheduler never updates LR
for epoch in range(epochs):
    train(model)
    # Missing: scheduler.step()

# CORRECT
for epoch in range(epochs):
    train(model)
    scheduler.step()  # or scheduler.step(val_loss) for ReduceLROnPlateau
```

### 2. Stepping Per-Batch vs Per-Epoch

```python
# Per-epoch schedulers (StepLR, ExponentialLR, ReduceLROnPlateau):
for epoch in range(epochs):
    for batch in dataloader:
        train_step(batch)
    scheduler.step()  # Once per epoch

# Per-step schedulers (OneCycleLR, CyclicLR, warmup):
for epoch in range(epochs):
    for batch in dataloader:
        train_step(batch)
        scheduler.step()  # Every batch!
```

### 3. Using Cyclical LR with Adam's Momentum

```python
# WRONG — cyclical momentum conflicts with Adam
scheduler = CyclicLR(optimizer, base_lr=1e-4, max_lr=1e-2)

# CORRECT — disable momentum cycling for Adam
scheduler = CyclicLR(optimizer, base_lr=1e-4, max_lr=1e-2,
                     cycle_momentum=False)
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| LR sensitivity | RNNs amplify LR effects through recurrence |
| Warmup | Start slow to stabilize early training |
| Step/Exponential decay | Simple, reliable baselines |
| Cosine annealing | Smooth decay, good for fixed budgets |
| ReduceLROnPlateau | Most practical — adapts to training |
| Cyclical LR | Helps exploration, acts as regularizer |
| One-Cycle | Fast convergence in single training run |
| Default recipe | Adam + LR=0.001 + ReduceLROnPlateau |

---

## ❓ Revision Questions

1. **Why are RNNs more sensitive to learning rate than feedforward networks?** How does recurrence amplify the effect?

2. **Compare ReduceLROnPlateau with Cosine Annealing.** When would you prefer each?

3. **Explain why warmup is important for RNN training.** What can go wrong without it?

4. **You're training an LSTM and notice the loss plateaus at epoch 15 with LR=0.001.** Design a learning rate schedule to continue improving.

5. **How does the One-Cycle policy achieve "super-convergence"?** Why does temporarily increasing LR help?

6. **Your RNN trains well with Adam (LR=0.001) but the AWD-LSTM paper uses SGD (LR=30).** Explain the huge LR difference and why both can work.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Gradient Clipping](02-gradient-clipping.md) |
| ⬆️ Unit | [09 Training RNNs](../README.md) |
| ➡️ Next | [Dropout in RNNs](04-dropout-in-rnns.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
