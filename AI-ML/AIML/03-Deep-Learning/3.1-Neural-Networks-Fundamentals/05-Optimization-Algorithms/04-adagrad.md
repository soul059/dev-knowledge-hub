# 4. AdaGrad — Adaptive Gradient Algorithm

> **Unit 5 · Optimization Algorithms** — Per-parameter adaptive learning rates from accumulated gradients

---

## Chapter Overview

All optimizers discussed so far use a **single global learning rate** for every parameter. But different parameters may need different step sizes: parameters associated with **frequently occurring features** already get many gradient updates (and should learn slowly), while parameters for **rare features** get few updates (and should learn quickly). **AdaGrad** (Duchi et al., 2011) automatically adapts the learning rate **per parameter** by dividing by the square root of accumulated squared gradients.

---

## 1. Motivation: Why Adaptive Learning Rates?

### The Problem

Consider training a word embedding model:
- The word "the" appears in almost every sentence → its embedding gets **many** gradient updates
- The word "serendipity" appears rarely → its embedding gets **few** gradient updates

With a fixed learning rate:
- If α is tuned for frequent words → rare word updates are too small
- If α is tuned for rare words → frequent word updates are too large and unstable

```
  Fixed Learning Rate Problem:
  
  Parameter    Gradient      What We Want        What We Get (fixed α)
  ─────────────────────────────────────────────────────────────────────
  w_"the"      Large, frequent   Small steps    ✗ Large steps → unstable
  w_"serendip" Small, rare       Large steps    ✗ Small steps → slow
  w_bias       Medium            Medium steps   ≈ OK
```

### The Solution: Scale by Gradient History

```
  Adaptive Learning Rate:
  
  Parameter with many large gradients → LARGE accumulated sum → SMALL effective lr
  Parameter with few small gradients  → SMALL accumulated sum → LARGE effective lr
  
  effective_lr = α / √(accumulated_squared_gradients)
```

---

## 2. AdaGrad Algorithm

### Full Algorithm

```
Initialize:
    G₀ = 0      (accumulated squared gradients, same shape as θ)
    ε = 1e-8     (small constant for numerical stability)

For each step t = 1, 2, ...:
    1. Compute gradient:    gₜ = ∇θ J(θₜ)
    
    2. Accumulate squares:  Gₜ = Gₜ₋₁ + gₜ ⊙ gₜ        ← element-wise square
    
    3. Update parameters:   θₜ₊₁ = θₜ - α/√(Gₜ + ε) ⊙ gₜ   ← element-wise division
```

### Per-Parameter View

For each parameter θⱼ:

```
  Gₜ,ⱼ = Σₛ₌₁ᵗ (gₛ,ⱼ)²           ← sum of all past squared gradients for param j

  θₜ₊₁,ⱼ = θₜ,ⱼ - (α / √(Gₜ,ⱼ + ε)) · gₜ,ⱼ
```

### Effective Learning Rate

```
  αₑff,ⱼ = α / √(Gₜ,ⱼ + ε)
```

This is different for **each parameter** and **decreases over time** (since Gₜ only grows).

---

## 3. Mathematical Details

### Why Square Root?

The squared gradients Gₜ have units of (gradient)². Taking the square root gives units of (gradient), so:

```
  α / √G · g  has units of  (learning_rate / gradient) · gradient = learning_rate
```

This ensures the effective step size is on the same scale as the original learning rate.

### Relationship to Gradient Scale

If a parameter consistently has large gradients |gⱼ| ≈ L:

```
  After t steps: Gₜ,ⱼ ≈ t · L²
  Effective lr:  α / √(tL²) = α / (L√t)
```

The effective learning rate:
1. Is **inversely proportional** to gradient magnitude L → large-gradient params get smaller updates
2. **Decays as 1/√t** → all learning rates decrease over time

---

## 4. Visualizing AdaGrad's Behavior

### Adaptive Steps in 2D

```
  Consider f(x,y) = 50x² + y²  (steep in x, gentle in y)
  
  Gradients: ∂f/∂x = 100x (large), ∂f/∂y = 2y (small)
  
  SGD path:                          AdaGrad path:
  y ↑                                y ↑
    │\  /\  /\                         │\
    │ \/  \/  \                        │ \──
    │          \                       │    \──
    │           ★                      │       \──★
    └──────────→ x                     └──────────→ x
    
  SGD: oscillates in x (large grad)   AdaGrad: x-lr shrinks fast,
       slow progress in y                      y-lr stays large
                                               → balanced progress
```

### Learning Rate Evolution per Parameter

```
  Effective Learning Rate over Time:
  
  α_eff ↑
       │
  0.10 │ ──── y-parameter (small gradients → slow G accumulation)
       │    ──────
  0.05 │          ──────
       │     ─ ─ ─      ──────
  0.02 │         ─ ─ ─ ─      ────── ← still learning
       │ ── x-param       ─ ─ ─ ─ ─
  0.01 │     ──── (large gradients → fast G accumulation)
       │          ────
  0.00 │               ────────── ← essentially stopped
       └──────────────────────────────→ Steps
        0    50   100  150  200  250
```

---

## 5. Sparse Features and NLP

### Why AdaGrad Excels with Sparse Data

In NLP/recommendation systems, feature vectors are often **sparse** — most entries are zero:

```
  Document: "The cat sat"
  Vocabulary: [the, cat, dog, sat, ran, bird, ...]
  Bag-of-words: [1,   1,   0,   1,   0,   0,  ...]
                 ↑         ↑              ↑
                 active    inactive       inactive
```

For inactive features, gradient = 0 → Gₜ doesn't increase → learning rate stays high → when the feature **does** appear, it gets a **large update**. This is exactly the desired behavior.

### Comparison

| Feature Type | SGD (fixed α) | AdaGrad |
|---|---|---|
| Frequent (e.g., "the") | Same α (may be too large) | α decreases → small, stable updates |
| Rare (e.g., "serendipity") | Same α (may be too small) | α stays high → large, impactful updates |
| Inactive (gradient = 0) | No update | G unchanged → lr preserved for future |

---

## 6. The Learning Rate Decay Problem

### The Fatal Flaw

Since Gₜ = Σ gₛ² is a **monotonically increasing** sum:

```
  Gₜ ≥ Gₜ₋₁  for all t   (adding non-negative gₜ² ≥ 0)
```

Therefore:

```
  αₑff,ⱼ(t) = α / √(Gₜ,ⱼ + ε)  is monotonically DECREASING
```

**Problem**: Eventually, Gₜ becomes so large that αₑff → 0, and learning **stops entirely**:

```
  Training Progress:
  
  Loss ↑                              Learning Rate ↑
       │╲                                  │────╮
       │ ╲                                 │    ╰──╮
       │  ╲                                │       ╰──╮
       │   ╲──╮                            │          ╰──╮
       │      ╰──── plateau!               │             ╰──→ 0
       │           (lr too small)          │
       └────────────────→ t               └────────────────→ t
       
  Learning STOPS because accumulated G is too large!
```

### Mathematical Convergence of Learning Rate

After T steps with average squared gradient σ²:

```
  Gₜ ≈ T · σ²
  αₑff ≈ α / (σ√T)  → 0 as T → ∞
```

This is **too aggressive** for non-convex optimization where we need to continue learning.

---

## 7. Limitations of AdaGrad

| Limitation | Explanation |
|---|---|
| **Monotonic lr decay** | Learning rate can only decrease, never recover |
| **Premature stopping** | For long training, lr becomes negligibly small |
| **Not suited for non-convex** | Deep networks need continued learning |
| **Memory** | Must store G (same size as parameters) |
| **Sensitive to initial lr** | α must be set carefully (often higher than SGD) |

### When AdaGrad is Still Good

Despite its limitations, AdaGrad works well when:
- Training is **short** (lr doesn't decay too much)
- Features are **sparse** (NLP, recommendation systems)
- Problem is **convex** (where aggressive lr decay is actually desirable)
- Used for **online learning** where data arrives streaming

---

## 8. Worked Example

### Problem
Minimize f(x, y) = 50x² + y² starting at (2, 10) with AdaGrad, α = 0.5, ε = 1e-8.

### Solution

**Step 0:** θ = (2, 10), G = (0, 0)
```
g = (200, 20)
G = (0, 0) + (200², 20²) = (40000, 400)
θ = (2, 10) - 0.5/(√(40000+ε), √(400+ε)) · (200, 20)
  = (2, 10) - (0.5/200, 0.5/20) · (200, 20)
  = (2, 10) - (0.5, 0.5)
  = (1.5, 9.5)
```

Note: First step is identical in both dimensions (equal effective step of 0.5)!

**Step 1:** θ = (1.5, 9.5), G = (40000, 400)
```
g = (150, 19)
G = (40000 + 22500, 400 + 361) = (62500, 761)
θ = (1.5, 9.5) - 0.5/(250, 27.59) · (150, 19)
  = (1.5, 9.5) - (0.3, 0.344)
  = (1.2, 9.156)
```

Now the x-direction gets a smaller effective lr (0.002) vs y-direction (0.018) — AdaGrad adapts!

**Step 2:** θ = (1.2, 9.156), G = (62500, 761)
```
g = (120, 18.31)
G = (62500 + 14400, 761 + 335.3) = (76900, 1096.3)
θ = (1.2, 9.156) - 0.5/(277.3, 33.11) · (120, 18.31)
  = (1.2, 9.156) - (0.216, 0.277)
  = (0.984, 8.879)
```

The x-lr keeps shrinking faster (large gradients accumulate faster).

---

## 9. Python Implementation (PyTorch)

### Using PyTorch's AdaGrad

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Sparse-like data (simulating NLP embeddings) ───────────────
torch.manual_seed(42)
N, D = 5000, 100
X = torch.zeros(N, D)
for i in range(N):
    active = torch.randperm(D)[:5]  # only 5 of 100 features active
    X[i, active] = torch.randn(5)
y = (X @ torch.randn(D, 1)).squeeze()

loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Compare SGD vs AdaGrad on sparse data ──────────────────────
def train(opt_cls, opt_kwargs, epochs=30):
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(D, 64), nn.ReLU(), nn.Linear(64, 1))
    criterion = nn.MSELoss()
    optimizer = opt_cls(model.parameters(), **opt_kwargs)
    
    history = []
    for epoch in range(epochs):
        total_loss = 0
        for xb, yb in loader:
            pred = model(xb).squeeze()
            loss = criterion(pred, yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total_loss += loss.item() * len(xb)
        history.append(total_loss / N)
    return history

sgd_loss = train(torch.optim.SGD, {"lr": 0.01})
adagrad_loss = train(torch.optim.Adagrad, {"lr": 0.01})

print(f"SGD final loss:     {sgd_loss[-1]:.4f}")
print(f"AdaGrad final loss: {adagrad_loss[-1]:.4f}")
```

### Manual AdaGrad Implementation

```python
class ManualAdaGrad:
    """AdaGrad optimizer from scratch."""
    
    def __init__(self, params, lr=0.01, eps=1e-8):
        self.params = list(params)
        self.lr = lr
        self.eps = eps
        # Accumulated squared gradients
        self.G = [torch.zeros_like(p) for p in self.params]
    
    def zero_grad(self):
        for p in self.params:
            if p.grad is not None:
                p.grad.zero_()
    
    def step(self):
        for i, p in enumerate(self.params):
            if p.grad is None:
                continue
            g = p.grad.data
            
            # Accumulate squared gradients
            self.G[i] += g * g
            
            # Adaptive update
            p.data -= self.lr / (torch.sqrt(self.G[i]) + self.eps) * g

# Verify against PyTorch
model1 = nn.Linear(10, 1); model2 = nn.Linear(10, 1)
model2.load_state_dict(model1.state_dict())

opt1 = torch.optim.Adagrad(model1.parameters(), lr=0.01)
opt2 = ManualAdaGrad(model2.parameters(), lr=0.01)

x = torch.randn(16, 10)
for _ in range(20):
    loss1 = model1(x).sum(); opt1.zero_grad(); loss1.backward(); opt1.step()
    loss2 = model2(x).sum(); opt2.zero_grad(); loss2.backward(); opt2.step()

diff = (model1.weight.data - model2.weight.data).abs().max()
print(f"Max weight difference: {diff:.1e}")  # ≈ 0
```

### Visualizing Learning Rate Decay

```python
import matplotlib.pyplot as plt
import numpy as np

# Simulate AdaGrad lr decay for two parameters
steps = 500
alpha = 0.1
G_freq = 0.0   # parameter with large gradients
G_rare = 0.0   # parameter with small gradients

lr_freq, lr_rare = [], []
for t in range(1, steps + 1):
    g_freq = np.random.normal(10, 2)   # large, frequent gradients
    g_rare = np.random.normal(0.5, 0.3) if np.random.rand() < 0.1 else 0  # sparse
    
    G_freq += g_freq**2
    G_rare += g_rare**2
    
    lr_freq.append(alpha / np.sqrt(G_freq + 1e-8))
    lr_rare.append(alpha / np.sqrt(G_rare + 1e-8) if G_rare > 0 else alpha)

fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(lr_freq, label='Frequent feature (large grads)', color='red')
ax.plot(lr_rare, label='Rare feature (sparse grads)', color='blue')
ax.set_xlabel('Step')
ax.set_ylabel('Effective Learning Rate')
ax.set_title('AdaGrad: Adaptive Learning Rate Per Parameter')
ax.legend()
ax.set_yscale('log')
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig("adagrad_lr_decay.png", dpi=150)
plt.show()
```

---

## 10. Applications

| Application | Why AdaGrad Fits |
|---|---|
| **Word embeddings (Word2Vec, GloVe)** | Sparse word co-occurrence → rare words get large updates |
| **Recommendation systems** | Sparse user-item interactions |
| **Online learning** | Streaming data, short training windows |
| **Convex optimization** | Aggressive lr decay is actually optimal |
| **Click-through rate prediction** | Sparse categorical features |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Core innovation | Per-parameter adaptive learning rate |
| Accumulator | Gₜ = Gₜ₋₁ + gₜ² (sum of squared gradients) |
| Update rule | θ = θ - α/√(G+ε) · g |
| Effective lr | αₑff = α / √(Gₜ + ε) — per parameter |
| Benefit for sparse data | Rare features retain high lr |
| Main limitation | Monotonic lr decay → learning can stop |
| Fix | → RMSProp (next chapter) |
| PyTorch | `torch.optim.Adagrad(params, lr=0.01)` |

---

## Revision Questions

1. **Derive the effective learning rate for a parameter** that receives constant gradients of magnitude G for T steps. How does it decay with T?

2. **Explain why AdaGrad is particularly suited for sparse data.** Give a concrete NLP example with rare vs frequent words.

3. **What is the fundamental limitation of AdaGrad**, and why does it matter more for deep learning than for convex optimization?

4. **Compare AdaGrad's per-parameter adaptation with momentum's approach.** Do they solve the same problem? Can they be combined?

5. **In the worked example, why is the first AdaGrad step identical in both directions?** Show this mathematically.

6. **Implement AdaGrad from scratch** and verify it produces the same results as `torch.optim.Adagrad`.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Nesterov Accelerated Gradient](./03-nesterov-accelerated-gradient.md) | [Unit 5: Optimization](./README.md) | [RMSProp →](./05-rmsprop.md) |
