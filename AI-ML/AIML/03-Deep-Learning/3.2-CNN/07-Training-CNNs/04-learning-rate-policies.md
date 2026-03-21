# Learning Rate Policies for CNNs

> **Unit 7 · Training CNNs · Topic 4/5**

---

## Table of Contents

1. [Overview & Importance](#1-overview--importance)
2. [Step Decay](#2-step-decay)
3. [Cosine Annealing](#3-cosine-annealing)
4. [Warmup + Cosine Decay](#4-warmup--cosine-decay)
5. [One-Cycle Policy](#5-one-cycle-policy)
6. [Cyclical Learning Rates](#6-cyclical-learning-rates)
7. [Reduce on Plateau](#7-reduce-on-plateau)
8. [LR Finder for CNNs](#8-lr-finder-for-cnns)
9. [Practical Recommendations](#9-practical-recommendations)
10. [PyTorch Schedulers](#10-pytorch-schedulers)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview & Importance

The learning rate is arguably the **most important hyperparameter** in CNN training. Too high → divergence. Too low → slow convergence and poor local minima.

```
Learning Rate Impact:

Loss │
     │  ╲  lr too high: diverges!
     │   ╲
     │    ╲  ╱╲  ╱╲
     │     ╲╱  ╲╱  ╲╱╲╱╲
     │
     │     ╲
     │      ╲     ← lr just right
     │       ╲
     │        ╲────── converges to good minimum
     │
     │         ╲
     │          ╲
     │           ╲
     │            ╲                 ← lr too low
     │             ╲─────────────── stuck in poor minimum
     └──────────────────────────────────── Iterations

Dynamic schedules outperform fixed LR by adapting throughout training:
  Early training:   Higher LR → fast progress, escape bad regions
  Late training:    Lower LR  → fine-tune, settle into sharp minimum
```

---

## 2. Step Decay

Reduce LR by a fixed factor at predetermined epoch milestones.

```
Step Decay Formula:
  lr(t) = lr_init × γ^⌊t/step_size⌋

  or with milestones:
  lr(epoch) = lr_init × γ^n,  where n = # milestones passed

Typical: lr_init=0.1, γ=0.1, milestones=[30, 60, 90]

LR  │
0.1 ├─────────────┐
    │             │
0.01├─────────────┼─────────────┐
    │             │             │
.001├─────────────┼─────────────┼──────────────┐
    │             │             │              │
    └─────────────┴─────────────┴──────────────┴── Epoch
    0            30            60             90

Standard for ResNet training on ImageNet.
Simple and effective but requires knowing good milestones in advance.
```

---

## 3. Cosine Annealing

Smoothly decay LR following a cosine curve from initial to minimum value.

```
Cosine Annealing Formula:
  lr(t) = lr_min + (lr_max - lr_min) × 0.5 × (1 + cos(π × t / T_max))

  Where:
    t = current epoch/iteration
    T_max = total epochs/iterations
    lr_max = initial learning rate
    lr_min = minimum learning rate (often 0 or 1e-6)

LR  │
    │╲
    │ ╲
    │  ╲
    │   ╲
    │    ╲
    │     ╲                        Smooth, no sudden drops
    │      ╲
    │       ╲
    │        ╲
    │         ╲
    │          ╲─────              Approaches lr_min gradually
    └──────────────────────── Epoch
    0          T_max/2      T_max

Cosine Annealing with Warm Restarts (SGDR):
  After each cycle, reset LR back to lr_max
  
  lr(t) = lr_min + (lr_max - lr_min) × 0.5 × (1 + cos(π × T_cur / T_i))

LR  │
    │╲    ╱╲        ╱╲
    │ ╲  ╱  ╲      ╱  ╲
    │  ╲╱    ╲    ╱    ╲
    │         ╲  ╱      ╲
    │          ╲╱        ╲─────
    └──────────────────────────── Epoch
    
  T₁ cycle   T₂ cycle     T₃ cycle
  (restart at each cycle, potentially find better minima)
```

---

## 4. Warmup + Cosine Decay

Start with a low LR, linearly increase to target, then cosine decay. This is the **modern standard** for training large models.

```
Warmup + Cosine Decay:

Phase 1: Linear Warmup (first few epochs/iterations)
  lr(t) = lr_max × (t / T_warmup)

Phase 2: Cosine Decay (remaining training)
  lr(t) = lr_min + (lr_max - lr_min) × 0.5 × (1 + cos(π × (t - T_warmup) / (T_total - T_warmup)))

LR    │
      │        ╱╲
lr_max│       ╱  ╲
      │      ╱    ╲
      │     ╱      ╲
      │    ╱        ╲
      │   ╱          ╲
      │  ╱            ╲
      │ ╱              ╲
lr_min│╱                ╲──────
      └────────────────────────── Epoch
      0  T_warmup      T_total

Why warmup?
  1. BN statistics are unstable initially (need a few batches)
  2. Adaptive optimizers need to estimate second moments
  3. Prevents large early updates that could be harmful
  4. Especially important for ViT and large batch training

Typical warmup: 5-10 epochs (or 500-2000 iterations)
```

### Mathematical Details

```
Linear Warmup:
  lr(t) = t/T_w × lr_max       for 0 ≤ t < T_w

Cosine Decay:
  lr(t) = lr_min + (lr_max - lr_min)/2 × (1 + cos(π × (t - T_w)/(T - T_w)))
                                         for T_w ≤ t ≤ T

Where:
  T_w = warmup epochs
  T   = total epochs
  
Example values:
  T = 300 epochs, T_w = 5 epochs
  lr_max = 1e-3, lr_min = 1e-6
  
  At epoch 0:    lr = 0
  At epoch 5:    lr = 1e-3 (peak)
  At epoch 152:  lr ≈ 5e-4 (half of peak)
  At epoch 300:  lr = 1e-6 (minimum)
```

---

## 5. One-Cycle Policy

The **1cycle** policy (Smith & Topin, 2018) uses a single super-convergence cycle: warmup to high LR → anneal back down. Trains **faster** and reaches **higher accuracy**.

```
One-Cycle Policy:

Three phases:
  Phase 1 (30% of training): LR ramps UP from lr_max/div_factor to lr_max
  Phase 2 (70% of training): LR ramps DOWN from lr_max to lr_max/div_factor
  Phase 3 (last ~5%):        LR anneals to lr_max/(div_factor × final_div_factor)

LR    │         ╱╲
lr_max│        ╱  ╲
      │       ╱    ╲
      │      ╱      ╲
      │     ╱        ╲
      │    ╱          ╲
lr/25 │   ╱            ╲
      │  ╱              ╲
      │ ╱                ╲
lr/2500│                  ╲___
      └────────────────────────── Epoch
      0%   30%      70%    100%

Momentum (inverse of LR):
      │___
 0.95 │   ╲
      │    ╲
      │     ╲          ╱────
 0.85 │      ╲        ╱
      │       ╲      ╱
      │        ╲    ╱
      │         ╲  ╱
      │          ╲╱
      └────────────────────────── Epoch

Key hyperparameters:
  max_lr = found via LR range test
  div_factor = 25 (initial lr = max_lr / 25)
  final_div_factor = 1e4 (final lr = max_lr / 250000)
  pct_start = 0.3 (30% warmup)
  
Super-convergence: Achieves same accuracy in ~5× fewer epochs!
```

---

## 6. Cyclical Learning Rates

Oscillate LR between bounds, allowing the model to escape local minima.

```
Cyclical LR (CLR) — Smith, 2017:

Triangular policy:
  Linearly increase from base_lr to max_lr, then back

LR    │
max_lr│    ╱╲      ╱╲      ╱╲
      │   ╱  ╲    ╱  ╲    ╱  ╲
      │  ╱    ╲  ╱    ╲  ╱    ╲
base  │ ╱      ╲╱      ╲╱      ╲
      └──────────────────────────── Iteration
       cycle 1  cycle 2  cycle 3

Triangular2 policy (halve amplitude each cycle):
LR    │
      │    ╱╲
      │   ╱  ╲   ╱╲
      │  ╱    ╲ ╱  ╲  ╱╲
base  │ ╱      ╲    ╲╱  ╲╱
      └──────────────────────── Iteration

Exponential range (decay max_lr):
  max_lr(cycle) = max_lr × γ^iteration

Why it works:
  • Increasing LR helps escape saddle points and poor local minima
  • Decreasing LR helps converge to sharper minima
  • Acts as implicit regularization (like SGD noise)
  • Finds wider minima → better generalization
```

---

## 7. Reduce on Plateau

Reduce LR when a monitored metric stops improving.

```
ReduceLROnPlateau:

  if metric hasn't improved for 'patience' epochs:
      lr = lr × factor

LR    │
      ├────────────────┐   Wait for plateau
      │                │
      │                ├───────────┐   Wait again
      │                │           │
      │                │           ├──────────
      │                │           │
      └────────────────┴───────────┴────────── Epoch

Parameters:
  factor = 0.1 (reduce by 10×)
  patience = 10 (wait 10 epochs)
  min_lr = 1e-7 (floor)
  threshold = 1e-4 (minimum improvement to count)
  mode = 'min' (for loss) or 'max' (for accuracy)

Advantages:
  ✓ Adapts to actual training dynamics
  ✓ No need to pre-specify milestones
  ✓ Simple to use

Disadvantages:
  ✗ Reactive, not proactive
  ✗ Can reduce LR too aggressively (hard to recover)
  ✗ Not optimal for large-scale training (cosine is better)
```

---

## 8. LR Finder for CNNs

### LR Range Test (Smith, 2018)

```
Algorithm:
  1. Start with very small LR (e.g., 1e-7)
  2. Increase LR exponentially after each batch
  3. Record the loss at each LR
  4. Plot loss vs LR
  5. Choose LR where loss is decreasing fastest

Loss │
     │╲                                Divergence ↗
     │ ╲
     │  ╲                             ╱
     │   ╲                           ╱
     │    ╲                         ╱
     │     ╲─────╮                 ╱
     │            ╲               ╱
     │             ╲─────╮       ╱
     │                    ╲─────╱
     └──────────────────────────────── log(LR)
      1e-7  1e-5  1e-3  1e-1   1  10
                    ↑         ↑
              Good max_lr    Diverges

Rules of thumb:
  • max_lr = point where loss starts increasing ÷ 10
  • For 1cycle: max_lr is 2-10× smaller than divergence point
  • For cosine: initial_lr = max_lr from LR finder
```

### Implementation

```python
import torch
import matplotlib.pyplot as plt

def lr_finder(model, train_loader, criterion, optimizer,
              start_lr=1e-7, end_lr=10, num_iter=200):
    """
    LR Range Test: exponentially increase LR and track loss.
    """
    lrs, losses = [], []
    lr_mult = (end_lr / start_lr) ** (1 / num_iter)
    lr = start_lr
    
    # Set initial LR
    for param_group in optimizer.param_groups:
        param_group['lr'] = lr
    
    model.train()
    best_loss = float('inf')
    
    for i, (inputs, targets) in enumerate(train_loader):
        if i >= num_iter:
            break
        
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        
        # Record
        lrs.append(lr)
        losses.append(loss.item())
        
        # Stop if loss diverges
        if loss.item() > 4 * best_loss:
            break
        if loss.item() < best_loss:
            best_loss = loss.item()
        
        loss.backward()
        optimizer.step()
        
        # Update LR
        lr *= lr_mult
        for param_group in optimizer.param_groups:
            param_group['lr'] = lr
    
    # Plot
    plt.figure(figsize=(10, 4))
    plt.plot(lrs, losses)
    plt.xscale('log')
    plt.xlabel('Learning Rate')
    plt.ylabel('Loss')
    plt.title('LR Finder')
    plt.savefig('lr_finder.png')
    
    return lrs, losses
```

---

## 9. Practical Recommendations

```
Recommendation Matrix:

┌──────────────────┬─────────────────────────────────────────┐
│ Scenario         │ Recommended Schedule                     │
├──────────────────┼─────────────────────────────────────────┤
│ ImageNet training│ Warmup (5 ep) + Cosine decay (300 ep)   │
│ (from scratch)   │ SGD, lr=0.1, batch=256                  │
├──────────────────┼─────────────────────────────────────────┤
│ Fine-tuning CNN  │ 1cycle or Cosine with warmup            │
│                  │ AdamW, lr=1e-3 (head), 1e-5 (backbone)  │
├──────────────────┼─────────────────────────────────────────┤
│ ViT training     │ Warmup (10 ep) + Cosine decay           │
│                  │ AdamW, lr=1e-3, wd=0.3                  │
├──────────────────┼─────────────────────────────────────────┤
│ Quick experiment │ 1cycle (fast convergence)               │
│                  │ Use LR finder to set max_lr             │
├──────────────────┼─────────────────────────────────────────┤
│ Small dataset    │ ReduceLROnPlateau or step decay         │
│                  │ Conservative LR, early stopping         │
├──────────────────┼─────────────────────────────────────────┤
│ Medical imaging  │ Cosine with warmup                      │
│                  │ Long warmup, strong augmentation        │
└──────────────────┴─────────────────────────────────────────┘

General rules:
  1. Always use warmup for large models or large batch sizes
  2. Cosine decay is the safe default
  3. 1cycle for fastest training
  4. Run LR finder before training
  5. AdamW generally better than SGD for fine-tuning
  6. SGD+momentum still competitive for training from scratch
```

---

## 10. PyTorch Schedulers

### All Schedulers in One Reference

```python
import torch.optim as optim
from torch.optim.lr_scheduler import (
    StepLR, MultiStepLR, ExponentialLR,
    CosineAnnealingLR, CosineAnnealingWarmRestarts,
    OneCycleLR, CyclicLR, ReduceLROnPlateau,
    LinearLR, SequentialLR
)

optimizer = optim.AdamW(model.parameters(), lr=1e-3)

# 1. Step Decay
scheduler = StepLR(optimizer, step_size=30, gamma=0.1)

# 2. Multi-Step Decay
scheduler = MultiStepLR(optimizer, milestones=[30, 60, 90], gamma=0.1)

# 3. Exponential Decay
scheduler = ExponentialLR(optimizer, gamma=0.95)  # decay each epoch

# 4. Cosine Annealing
scheduler = CosineAnnealingLR(optimizer, T_max=100, eta_min=1e-6)

# 5. Cosine Annealing with Warm Restarts
scheduler = CosineAnnealingWarmRestarts(optimizer, T_0=10, T_mult=2)

# 6. One-Cycle Policy
scheduler = OneCycleLR(optimizer, max_lr=1e-3, total_steps=1000,
                       pct_start=0.3, div_factor=25,
                       final_div_factor=1e4)

# 7. Cyclical LR
scheduler = CyclicLR(optimizer, base_lr=1e-5, max_lr=1e-3,
                     step_size_up=2000, mode='triangular2')

# 8. Reduce on Plateau
scheduler = ReduceLROnPlateau(optimizer, mode='min', factor=0.1,
                               patience=10, min_lr=1e-7)

# 9. Linear Warmup + Cosine (using Sequential)
warmup = LinearLR(optimizer, start_factor=0.01, total_iters=5)
cosine = CosineAnnealingLR(optimizer, T_max=95, eta_min=1e-6)
scheduler = SequentialLR(optimizer, [warmup, cosine], milestones=[5])
```

### Training Loop with Scheduler

```python
# For epoch-based schedulers (Step, Cosine, etc.)
for epoch in range(num_epochs):
    model.train()
    for images, labels in train_loader:
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
    
    scheduler.step()  # called once per epoch
    print(f"Epoch {epoch}: LR = {scheduler.get_last_lr()}")

# For iteration-based schedulers (OneCycleLR, CyclicLR)
scheduler = OneCycleLR(optimizer, max_lr=1e-3,
                       steps_per_epoch=len(train_loader),
                       epochs=num_epochs)

for epoch in range(num_epochs):
    model.train()
    for images, labels in train_loader:
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        scheduler.step()  # called once per BATCH

# For ReduceLROnPlateau
scheduler = ReduceLROnPlateau(optimizer, patience=5)
for epoch in range(num_epochs):
    train(...)
    val_loss = validate(...)
    scheduler.step(val_loss)  # pass the monitored metric
```

### Custom Warmup + Cosine

```python
import math

class WarmupCosineScheduler:
    def __init__(self, optimizer, warmup_epochs, total_epochs,
                 min_lr=1e-6):
        self.optimizer = optimizer
        self.warmup_epochs = warmup_epochs
        self.total_epochs = total_epochs
        self.min_lr = min_lr
        self.base_lrs = [g['lr'] for g in optimizer.param_groups]
        
    def step(self, epoch):
        if epoch < self.warmup_epochs:
            # Linear warmup
            scale = epoch / self.warmup_epochs
        else:
            # Cosine decay
            progress = (epoch - self.warmup_epochs) / (
                self.total_epochs - self.warmup_epochs)
            scale = self.min_lr / self.base_lrs[0] + (
                1 - self.min_lr / self.base_lrs[0]) * 0.5 * (
                1 + math.cos(math.pi * progress))
        
        for param_group, base_lr in zip(
                self.optimizer.param_groups, self.base_lrs):
            param_group['lr'] = base_lr * scale

# Usage
scheduler = WarmupCosineScheduler(optimizer, warmup_epochs=5,
                                   total_epochs=100)
```

---

## 11. Summary Table

| Schedule | Formula / Key Idea | Best For |
|---|---|---|
| **Step Decay** | lr × γ at milestones | Classic ImageNet training |
| **Cosine Annealing** | lr_min + ½(lr_max-lr_min)(1+cos(πt/T)) | General purpose, smooth |
| **Warmup + Cosine** | Linear warmup → cosine decay | Modern default for large models |
| **One-Cycle** | Up 30% → Down 70% (super-convergence) | Fast training, best single-cycle |
| **Cyclical LR** | Triangle waves between bounds | Exploration, avoiding local minima |
| **ReduceOnPlateau** | Reduce when metric stagnates | Small datasets, adaptive |
| **SGDR** | Cosine with periodic warm restarts | Finding multiple good minima |
| **LR Finder** | Exponentially increase, find sweet spot | Before any training |

---

## 12. Revision Questions

1. **Compare step decay, cosine annealing, and one-cycle policy.** Draw the LR vs epoch curve for each. Which achieves convergence fastest?

2. **Why is learning rate warmup important?** What problems does it solve? When is warmup NOT needed?

3. **Explain the LR range test (LR finder).** How do you interpret the loss vs LR plot? How do you choose max_lr from it?

4. **Describe the one-cycle policy** in detail. What are the three phases? Why does it also adjust momentum? How does it achieve super-convergence?

5. **You're training a ViT-Base on 50,000 images** for 100 epochs. Design a complete learning rate schedule including warmup, optimizer choice, and scheduler parameters.

6. **When would you prefer ReduceLROnPlateau** over cosine annealing? What are the risks of using plateau-based scheduling?

---

## 13. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Fine-Tuning Strategies](./03-fine-tuning-strategies.md) | [Unit 7 Index](../07-Training-CNNs/) | [Mixed Precision Training →](./05-mixed-precision-training.md) |

---

> **References:**  
> - Smith, L.N. (2017). *Cyclical Learning Rates for Training Neural Networks.* WACV.  
> - Smith, L.N. & Topin, N. (2018). *Super-Convergence: Very Fast Training Using Large Learning Rates.*  
> - Loshchilov, I. & Hutter, F. (2017). *SGDR: Stochastic Gradient Descent with Warm Restarts.* ICLR.  
> - Goyal, P. et al. (2017). *Accurate, Large Minibatch SGD: Training ImageNet in 1 Hour.* (Introduced LR warmup)
