# Chapter 5: Learning Rate Scheduling

> **Unit 6 · Optimization · Module 5 of 7**

---

## 5.1 Overview

The learning rate is arguably the **single most important hyperparameter** in training ML models. A static learning rate is rarely optimal: we want to start large (fast progress) and reduce it over time (fine-grained convergence). Learning rate scheduling provides systematic strategies to adjust α during training.

---

## 5.2 Why the Learning Rate Matters

### Learning Rate Too High vs Too Low

```
  Loss (α too high)           Loss (α just right)          Loss (α too low)
   |   ╱╲   ╱╲                |                             |
   |  ╱  ╲ ╱  ╲ DIVERGE       | ╲                           | ╲
   | ╱    ╳    ╲               |   ╲                         |  ╲
   |╱      ╲    → ∞            |    ╲__                      |   ╲
   |        ╲                  |       ╲____                 |    ╲
   +—————————————→             |            ╲________        |     ╲
                               +—————————————————→           |      ╲___
                                                             |          ╲______
                               Converges in ~50 epochs       +——————————————————→
                                                             Takes 500+ epochs
```

### The Dilemma

- **Early training**: Large α → fast exploration of parameter space
- **Late training**: Small α → precise convergence near optimum

**Solution**: Start with a large LR and **decay** it during training.

---

## 5.3 Step Decay

Reduce the learning rate by a factor every N epochs:

```
α_t = α₀ × γ^(⌊epoch / step_size⌋)
```

Where `γ` is the decay factor (e.g., 0.1) and `step_size` is the decay interval.

```
  α
  0.1  |────────────╮
       |            │
  0.01 |            ╰──────────╮
       |                       │
  0.001|                       ╰──────────
       +—————————————————————————————→ Epoch
       0         30         60        90

  Example: α₀=0.1, γ=0.1, step_size=30
  Common in ResNet training
```

```python
# PyTorch
scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)
```

---

## 5.4 Exponential Decay

Smooth, continuous decay:

```
α_t = α₀ × e^(−k·t)

or equivalently:  α_t = α₀ × γ^t    where γ = e^(−k) ∈ (0, 1)
```

```
  α
  0.1  |╲
       | ╲
  0.05 |  ╲___
       |      ╲____
  0.01 |           ╲________
       |                     ╲______________
       +——————————————————————————————————→ Epoch

  Smoother than step decay, but less control over when drops happen
```

```python
scheduler = torch.optim.lr_scheduler.ExponentialLR(optimizer, gamma=0.95)
```

---

## 5.5 Cosine Annealing

Reduces LR following a cosine curve, providing warm restarts:

```
α_t = α_min + ½(α_max − α_min)(1 + cos(π · t / T))
```

```
  α
  α_max |╲          ╱╲          ╱╲
        | ╲        ╱  ╲        ╱  ╲
        |  ╲      ╱    ╲      ╱    ╲      With warm restarts
        |   ╲    ╱      ╲    ╱      ╲     (SGDR)
  α_min |    ╲__╱        ╲__╱        ╲__
        +——————————————————————————————→ Epoch
        0     T_0    2T_0   3T_0

  Without warm restarts:
  α_max |╲
        | ╲
        |   ╲
        |     ╲___
  α_min |          ╲___________
        +——————————————————————→ Epoch
        0              T
```

```python
# Basic cosine annealing
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=100, eta_min=1e-6
)

# Cosine annealing with warm restarts (SGDR)
scheduler = torch.optim.lr_scheduler.CosineAnnealingWarmRestarts(
    optimizer, T_0=10, T_mult=2
)
```

---

## 5.6 Warmup Strategy

Start with a very small LR and linearly increase to the target LR over a warmup period:

```
  α
  α_target |         ╱────────────╲___
           |       ╱                   ╲______
           |     ╱    warmup + cosine decay
           |   ╱
           | ╱
  0        +——————————————————————————————→ Epoch
           0   warmup        training
```

**Why warmup?**
- At initialization, gradients can be very large and unstable
- Large LR + large gradients = catastrophic early updates
- Warmup lets the model stabilize before aggressive learning

```python
def warmup_lr(epoch, warmup_epochs=5, base_lr=0.1):
    if epoch < warmup_epochs:
        return base_lr * (epoch + 1) / warmup_epochs
    return base_lr

# PyTorch with LambdaLR
scheduler = torch.optim.lr_scheduler.LambdaLR(
    optimizer,
    lr_lambda=lambda epoch: min(1.0, (epoch + 1) / warmup_epochs)
)
```

---

## 5.7 Cyclical Learning Rates (CLR)

Oscillate the LR between a minimum and maximum value:

```
  α
  α_max  |    ╱╲        ╱╲        ╱╲
         |   ╱  ╲      ╱  ╲      ╱  ╲
         |  ╱    ╲    ╱    ╲    ╱    ╲
         | ╱      ╲  ╱      ╲  ╱      ╲
  α_min  |╱        ╲╱        ╲╱        ╲
         +——————————————————————————————→ Iteration
         |← cycle →|

  Triangular policy (Smith, 2015)
```

**Benefits**:
- Can escape local minima at high LR
- Fine-tune at low LR
- Implicit super-convergence for some architectures

```python
scheduler = torch.optim.lr_scheduler.CyclicLR(
    optimizer,
    base_lr=1e-4,
    max_lr=1e-2,
    step_size_up=2000,
    mode='triangular2'  # halves the max_lr each cycle
)
```

---

## 5.8 One-Cycle Policy

A single cycle: ramp up LR then ramp down, combined with inverse momentum schedule:

```
  Learning Rate:                    Momentum:
  α_max |       ╱╲                  β_max |╲               ╱
        |      ╱  ╲                       | ╲             ╱
        |     ╱    ╲                      |  ╲           ╱
        |    ╱      ╲                     |   ╲         ╱
  α_min |___╱        ╲____               β_min|    ╲_______╱
        +—————————————————→               +—————————————————→
         warm  peak  anneal                warm  peak  anneal
```

**Key insight**: High LR acts as regularization (prevents overfitting).

```python
scheduler = torch.optim.lr_scheduler.OneCycleLR(
    optimizer,
    max_lr=0.01,
    total_steps=total_steps,
    pct_start=0.3,        # 30% warmup
    anneal_strategy='cos'
)
```

---

## 5.9 Learning Rate Finder

Systematically find a good learning rate by training with exponentially increasing LR and plotting loss:

```
  Loss
   |
   |──────╮                 Good LR range
   |      │╲               ◄────────────►
   |      │ ╲
   |      │  ╲___
   |      │      ╲___╱╲
   |      │            ╲╱╲     ← loss explodes
   |      │                ╲
   +——————┼——————┼———————┼————→ Learning Rate (log scale)
        1e-7   1e-4    1e-1   1e0

  Pick LR where loss is decreasing fastest
  (steepest negative slope, typically 1 order of magnitude before minimum)
```

```python
# Using PyTorch Lightning or fastai
from torch_lr_finder import LRFinder

lr_finder = LRFinder(model, optimizer, criterion)
lr_finder.range_test(train_loader, start_lr=1e-7, end_lr=1, num_iter=100)
lr_finder.plot()  # Inspect the plot
# Pick LR at steepest descent point (e.g., 3e-3)
```

---

## 5.10 ReduceLROnPlateau

Reduce LR when a monitored metric stops improving:

```
  Validation Loss
   |
   | ╲
   |   ╲___
   |       ╲_________ plateau detected → reduce LR
   |                 ╲
   |                   ╲___
   |                       ╲________ plateau → reduce again
   |                                ╲____
   +——————————————————————————————————————→ Epoch

  LR:  0.01 ──────→ 0.001 ──────→ 0.0001
```

```python
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',        # minimize val_loss
    factor=0.1,        # multiply LR by 0.1
    patience=10,       # wait 10 epochs before reducing
    min_lr=1e-7
)

# In training loop:
for epoch in range(epochs):
    train(...)
    val_loss = validate(...)
    scheduler.step(val_loss)  # Pass the monitored metric
```

---

## 5.11 Step-by-Step Example: Cosine Annealing

**Settings**: α₀ = 0.1, α_min = 0.001, T = 100 epochs

```
α_t = 0.001 + 0.5 × (0.1 − 0.001) × (1 + cos(π × t / 100))
```

| Epoch (t) | cos(πt/100) | α_t |
|-----------|-------------|-------|
| 0 | 1.000 | 0.1000 |
| 10 | 0.809 | 0.0810 |
| 25 | 0.000 | 0.0505 |
| 50 | −1.000 | 0.0010 |
| 75 | 0.000 | 0.0505 |
| 100 | 1.000 | 0.1000 |

*(With warm restarts, T resets — values repeat with larger periods)*

---

## 5.12 Comparison of All Schedules

| Schedule | When to Use | Pros | Cons |
|----------|------------|------|------|
| **Constant** | Quick experiments | Simple | Suboptimal |
| **Step Decay** | Standard training (ResNet) | Predictable drops | Manual tuning of milestones |
| **Exponential** | Smooth decay needed | Smooth | Hard to control end LR |
| **Cosine Annealing** | Modern deep learning | Smooth, principled | Requires T_max |
| **Warmup + Cosine** | Large models (Transformers) | Stable early training | Extra hyperparameter |
| **Cyclical LR** | Exploring LR ranges | Super-convergence | Complex schedule |
| **One-Cycle** | Fast training | Often best results | Needs total step count |
| **ReduceLROnPlateau** | Adaptive, metric-based | No manual schedule | Reactive, not proactive |

---

## 5.13 Practical Guidelines

1. **Start with LR finder** to determine the right order of magnitude
2. **Use warmup** for large batch sizes or transformer models (5–10% of total steps)
3. **Cosine annealing** is a strong default for most modern architectures
4. **One-cycle policy** often gives the fastest training
5. **ReduceLROnPlateau** is good for fine-tuning or when training length is unknown
6. **Step decay** at milestones (e.g., epochs 30, 60, 90) for reproducing classic results
7. **Don't tune LR and schedule simultaneously** — fix one, then optimize the other

---

## 5.14 Real-World ML Applications

| Model/Task | LR Schedule Used |
|-----------|-----------------|
| **ResNet (He et al.)** | Step decay (÷10 at epochs 30, 60, 90) |
| **BERT / GPT** | Linear warmup + linear/cosine decay |
| **EfficientNet** | Cosine annealing with warmup |
| **YOLOv5** | Cosine annealing |
| **Stable Diffusion** | Constant + warmup |
| **Super-convergence** | One-cycle policy |

---

## 5.15 Summary Table

| Concept | Key Detail |
|---------|-----------|
| Learning rate | Most critical hyperparameter |
| LR too high | Divergence or oscillation |
| LR too low | Slow convergence, may get stuck |
| Step decay | `α = α₀ × γ^(epoch // step)` |
| Cosine annealing | `α = α_min + ½(α_max−α_min)(1+cos(πt/T))` |
| Warmup | Linear ramp-up over first few epochs |
| One-cycle | Single ramp-up then ramp-down |
| LR finder | Exponentially increase LR, plot loss |
| ReduceLROnPlateau | Reduce LR when metric plateaus |
| Best default | Cosine annealing with warmup |

---

## 5.16 Quick Revision Questions

1. **Why is a fixed learning rate suboptimal?**
   → Early training benefits from large LR (fast exploration), late training needs small LR (precise convergence).

2. **What is the cosine annealing formula?**
   → α_t = α_min + ½(α_max − α_min)(1 + cos(πt/T)).

3. **Why is warmup important for large-batch training?**
   → Large batches produce large gradient estimates; a small initial LR prevents unstable early updates.

4. **How does the LR finder work?**
   → Train with exponentially increasing LR; pick the LR where loss decreases fastest (before it diverges).

5. **What schedule does ResNet typically use?**
   → Step decay: divide LR by 10 at epochs 30, 60, and 90.

6. **When should you use ReduceLROnPlateau?**
   → When you want adaptive scheduling based on validation performance, especially during fine-tuning.

---

| [← Previous: Momentum](04-momentum.md) | [Next Chapter: Convex Optimization →](06-convex-optimization.md) |
|:---|---:|
