# Gradient Clipping

> **Unit 3, Chapter 5** — Gradient norm clipping and value clipping to prevent exploding gradients, with PyTorch `torch.nn.utils.clip_grad_norm_`.

---

## 📋 Overview

**Gradient clipping** is a simple but essential technique that prevents exploding gradients by capping the gradient magnitude before updating weights. It's used in virtually all RNN training pipelines and is the standard solution to gradient explosion.

---

## ✂️ Two Types of Clipping

### 1. Gradient Norm Clipping (Recommended)

```
┌─────────────────────────────────────────────────────────────┐
│              GRADIENT NORM CLIPPING                         │
│                                                             │
│  Given: gradient g, threshold τ                            │
│                                                             │
│  if ||g|| > τ:                                             │
│      g ← g × (τ / ||g||)                                  │
│                                                             │
│  This RESCALES the gradient to have norm exactly τ          │
│  while PRESERVING its direction.                           │
│                                                             │
│  Before clipping:    g = [100, -200, 50]                   │
│                      ||g|| = 229.1                         │
│  After (τ=5):        g = [2.18, -4.36, 1.09]              │
│                      ||g|| = 5.0    ✓                      │
│                                                             │
│  Visualization:                                             │
│           ╱                          ╱                      │
│          ╱  Before       →         ╱  After                │
│         ╱  (||g||=229)            ╱  (||g||=5)             │
│        ●─────────▶              ●──▶                       │
│       ╱                        ╱  Same direction,          │
│                                   bounded magnitude        │
└─────────────────────────────────────────────────────────────┘
```

### 2. Gradient Value Clipping

```
┌─────────────────────────────────────────────────────────────┐
│              GRADIENT VALUE CLIPPING                        │
│                                                             │
│  For each element g_i:                                     │
│      g_i ← clip(g_i, -τ, τ)                               │
│      g_i ← max(-τ, min(τ, g_i))                           │
│                                                             │
│  Before: g = [100, -200, 0.5, 50]                          │
│  After (τ=5):  g = [5, -5, 0.5, 5]                        │
│                                                             │
│  ⚠️ WARNING: Changes gradient direction!                   │
│  Less commonly used than norm clipping.                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧮 Mathematics

### Norm Clipping Formula

```
g_clipped = g × min(1, τ / ||g||₂)

where ||g||₂ = √(Σᵢ gᵢ²)  is the L2 norm

Properties:
  • If ||g|| ≤ τ: g_clipped = g    (no change)
  • If ||g|| > τ: g_clipped = g × (τ/||g||)  (rescale to norm τ)
  • Direction always preserved ✓
  • Magnitude bounded by τ ✓

Global norm (across all parameters):
  ||g||_global = √(Σ_layer ||g_layer||²)

  All gradients scaled by the same factor:
  g_layer ← g_layer × min(1, τ / ||g||_global)
```

---

## 📐 Choosing the Threshold τ

```
Threshold selection strategies:

1. Monitor gradient norms during training (without clipping)
   Plot histogram → choose τ at ~95th percentile
   
   Gradient norm distribution:
   │  ████
   │  █████
   │  ██████
   │  ████████
   │  █████████
   │  ███████████░░░░░░     ← τ here (95th percentile)
   │  ██████████████░░░░░░░░░░░░ (tail = explosions)
   └─────────────────────────────
    0    2    4    6    8   10  ...

2. Common default values:
   • τ = 1.0  (conservative, very common)
   • τ = 5.0  (moderate)
   • τ = 10.0 (permissive)
   
3. Rule of thumb:
   Start with τ = 1.0, increase if training is too slow
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

# ─── Method 1: torch.nn.utils.clip_grad_norm_ (RECOMMENDED) ───
model = nn.RNN(input_size=10, hidden_size=50, batch_first=True)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

x = torch.randn(32, 20, 10)    # batch=32, seq_len=20
target = torch.randn(32, 20, 50)

output, _ = model(x)
loss = ((output - target) ** 2).mean()

optimizer.zero_grad()
loss.backward()

# CLIP before optimizer step!
max_norm = 1.0
grad_norm = torch.nn.utils.clip_grad_norm_(
    model.parameters(),
    max_norm=max_norm
)
print(f"Gradient norm before clipping: {grad_norm:.4f}")
print(f"Clipped to max_norm: {max_norm}")

optimizer.step()

# ─── Method 2: clip_grad_value_ ───
optimizer.zero_grad()
loss.backward()
torch.nn.utils.clip_grad_value_(model.parameters(), clip_value=0.5)
optimizer.step()

# ─── Complete Training Loop with Gradient Clipping ───
class RNNTrainer:
    def __init__(self, model, lr=0.001, max_grad_norm=1.0):
        self.model = model
        self.optimizer = torch.optim.Adam(model.parameters(), lr=lr)
        self.criterion = nn.MSELoss()
        self.max_grad_norm = max_grad_norm
    
    def train_step(self, x, target):
        self.model.train()
        output, _ = self.model(x)
        loss = self.criterion(output, target)
        
        self.optimizer.zero_grad()
        loss.backward()
        
        # Monitor gradient norm
        grad_norm = torch.nn.utils.clip_grad_norm_(
            self.model.parameters(),
            max_norm=self.max_grad_norm
        )
        
        self.optimizer.step()
        
        return loss.item(), grad_norm.item()

# Training
model = nn.RNN(10, 50, batch_first=True)
trainer = RNNTrainer(model, max_grad_norm=5.0)

print("\n=== Training with Gradient Clipping ===")
for epoch in range(10):
    x = torch.randn(16, 30, 10)
    target = torch.randn(16, 30, 50)
    loss, grad_norm = trainer.train_step(x, target)
    clipped = "CLIPPED" if grad_norm > 5.0 else "ok"
    print(f"Epoch {epoch+1}: loss={loss:.4f}, grad_norm={grad_norm:.4f} [{clipped}]")
```

---

## 📊 Effect of Gradient Clipping

```
WITHOUT clipping:                    WITH clipping (τ=5):
                                     
Loss                                 Loss
│          ╱╲                        │
│     ╱╲  ╱  ╲                       │
│    ╱  ╲╱    ╲     → NaN           │ ╲
│   ╱         ╲                      │  ╲
│  ╱           ╲                     │   ╲__
│ ╱                                  │      ╲___
│╱                                   │          ╲________
└────────────────────                └────────────────────
 Unstable, diverges                   Stable convergence

Gradient Norms                        Gradient Norms (clipped)
│     ╱╲                              │         τ=5
│    ╱  ╲    ╱╲                       │─────────────────────
│   ╱    ╲  ╱  ╲                      │  ╱╲    ╱╲
│  ╱      ╲╱    ╲                     │ ╱  ╲  ╱  ╲  (capped)
│ ╱                                   │╱    ╲╱    ╲_
└────────────────────                └────────────────────
 Spikes → instability                 Bounded → stable
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Norm Clipping | Rescale gradient if \|\|g\|\| > τ, preserves direction |
| Value Clipping | Clip each element to [-τ, τ], changes direction |
| Recommended | Norm clipping (clip_grad_norm_) |
| Common τ | 1.0 (conservative) to 10.0 (permissive) |
| When to Apply | After loss.backward(), before optimizer.step() |
| PyTorch Function | `torch.nn.utils.clip_grad_norm_(params, max_norm)` |
| Returns | Original gradient norm (for monitoring) |
| Does NOT Fix | Vanishing gradients (need LSTM/GRU for that) |

---

## ❓ Revision Questions

1. **What is the difference between gradient norm clipping and gradient value clipping? Which preserves gradient direction?**

2. **Given gradient g = [3, 4] and threshold τ = 2.5, compute the clipped gradient using norm clipping.**

3. **Why should gradient clipping be applied AFTER backward() but BEFORE optimizer.step()?**

4. **A training run shows loss values: 2.3, 1.8, 1.5, 1.2, NaN. What likely happened, and how would gradient clipping help?**

5. **How would you choose a good clipping threshold τ for a new model?**

6. **Gradient clipping fixes exploding gradients. Why doesn't it fix vanishing gradients?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Exploding Gradient Problem](04-exploding-gradient-problem.md) |
| ➡️ Next | [LSTM Motivation](../04-LSTM/01-lstm-motivation.md) |
| ⬆️ Unit Overview | [README](../README.md) |
