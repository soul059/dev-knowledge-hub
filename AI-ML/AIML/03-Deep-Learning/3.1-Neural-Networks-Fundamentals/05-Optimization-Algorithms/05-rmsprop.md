# 5. RMSProp — Root Mean Square Propagation

> **Unit 5 · Optimization Algorithms** — Fixing AdaGrad's decay with exponential moving averages

---

## Chapter Overview

AdaGrad's fatal flaw is its **monotonically decreasing learning rate** — the accumulated squared gradients Gₜ only grow, eventually driving the effective learning rate to zero. **RMSProp** (Hinton, 2012, unpublished lecture notes) elegantly fixes this by replacing the ever-growing sum with an **exponential moving average** (EMA) of squared gradients. This allows the learning rate to adapt based on **recent** gradient history rather than the entire past, preventing premature learning rate collapse.

---

## 1. From AdaGrad to RMSProp

### The Fix: Replace Sum with EMA

```
  AdaGrad:   Gₜ = Gₜ₋₁ + gₜ²              ← monotonically increasing (sum)
  RMSProp:   Eₜ = ρ·Eₜ₋₁ + (1-ρ)·gₜ²      ← bounded (exponential moving average)
```

| Aspect | AdaGrad | RMSProp |
|---|---|---|
| Squared gradient tracker | Gₜ = Σ gₛ² (cumulative sum) | Eₜ = EMA of gₛ² |
| Behavior over time | Only increases | Can increase or decrease |
| Learning rate | Monotonically decays → 0 | Adapts to recent gradient scale |
| Long training | Eventually stops learning | Continues learning |

### Visualization

```
  Accumulated Squared Gradients:
  
  G ↑                              E ↑
    │               ╱╱ AdaGrad      │       ╱╲
    │             ╱╱                │      ╱  ╲   ╱╲  RMSProp
    │           ╱╱                  │     ╱    ╲ ╱  ╲──╱──
    │         ╱╱                    │    ╱      ╲
    │       ╱╱                      │   ╱
    │     ╱╱                        │  ╱
    │   ╱╱  ← grows forever        │ ╱ ← tracks recent magnitude
    │ ╱╱                            │╱
    └───────────────→ t             └───────────────→ t
    
  Result: AdaGrad lr → 0            Result: RMSProp lr stays adaptive
```

---

## 2. RMSProp Algorithm

### Full Algorithm

```
Initialize:
    E₀ = 0        (exponential moving average of squared gradients)
    ρ = 0.9        (decay rate, also written as γ or α in some texts)
    ε = 1e-8       (numerical stability)
    α = learning rate

For each step t = 1, 2, ...:
    1. Compute gradient:           gₜ = ∇θ J(θₜ)
    
    2. Update moving average:      Eₜ = ρ · Eₜ₋₁ + (1-ρ) · gₜ²
    
    3. Update parameters:          θₜ₊₁ = θₜ - (α / √(Eₜ + ε)) · gₜ
```

### Per-Parameter View

For each parameter θⱼ:

```
  Eₜ,ⱼ = ρ · Eₜ₋₁,ⱼ + (1-ρ) · (gₜ,ⱼ)²

  θₜ₊₁,ⱼ = θₜ,ⱼ − α / √(Eₜ,ⱼ + ε) · gₜ,ⱼ
```

### Why It Works

The EMA Eₜ approximates the **mean of recent squared gradients** over a window of ~1/(1-ρ) steps:

```
  Eₜ ≈ E[gₜ²]_recent
  √(Eₜ) ≈ RMS(recent gradients)    ← hence the name "Root Mean Square Prop"
```

The effective learning rate becomes:

```
  αₑff = α / RMS(recent gradients)
```

If recent gradients are **large** → αₑff is **small** (careful steps)
If recent gradients are **small** → αₑff is **large** (aggressive steps)

---

## 3. The Decay Rate ρ

### Role of ρ

ρ controls the **memory** of the EMA:

| ρ | Window ≈ 1/(1-ρ) | Behavior |
|---|---|---|
| 0.9 | ~10 steps | Adapts quickly (recommended default) |
| 0.95 | ~20 steps | Moderate memory |
| 0.99 | ~100 steps | Long memory, slow adaptation |
| 0.5 | ~2 steps | Very short memory, noisy estimate |

### Effect on Stability

```
  ρ too small (0.5):             ρ just right (0.9):         ρ too large (0.999):
  
  αₑff ↑                        αₑff ↑                      αₑff ↑
       │  ╱╲╱╲╱╲╱╲               │   ╱╲                          │──────────╮
       │╱╲        ╱╲              │  ╱  ╲  ╱╲                    │          ╰──╮
       │          ╲╱              │ ╱    ╲╱  ╲──                 │             ╰───
       │                         │╱             ──              │
       └──────────→ t            └──────────→ t                 └──────────→ t
       
  Too jerky                      Smooth adaptation              Too slow to adapt
```

---

## 4. Mathematical Analysis

### Steady-State Effective Learning Rate

If the gradient has constant magnitude |g| for many steps:

```
  Eₜ → (1-ρ)|g|² · [1 + ρ + ρ² + ...] = (1-ρ)|g|² · 1/(1-ρ) = |g|²
  
  αₑff = α / √(|g|² + ε) ≈ α / |g|
```

So RMSProp effectively **normalizes** each parameter's update by its RMS gradient magnitude.

### Comparison with AdaGrad

After T steps with constant gradient |g|:

| Optimizer | Denominator | Effective lr |
|---|---|---|
| AdaGrad | √(T · g²) = |g|√T | α/(|g|√T) → 0 |
| RMSProp | √(g²) = |g| | α/|g| = constant |

RMSProp's effective lr stabilizes; AdaGrad's goes to zero.

### Connection to Natural Gradient

RMSProp approximately implements a **diagonal approximation** to the natural gradient:

```
  Natural gradient: θ -= α · F⁻¹ · g    where F = Fisher information matrix
  
  RMSProp ≈ diagonal of F⁻¹ · g  (when F_diag ≈ E[g²])
```

This connection explains why adaptive methods can be so effective.

---

## 5. RMSProp on Elongated Loss Surfaces

### ASCII Optimization Path

```
  f(x,y) = 100x² + y²  (very elongated ellipse)
  
  SGD path:                          RMSProp path:
  y ↑                                y ↑
    │\  /\  /\  /\                     │\
    │ \/  \/  \/  \                    │ \
    │              \                   │  \──
    │      ★        \                  │     \──
    │              /                   │        \──★
    │ /\  /\  /\  /                    │
    └──────────────→ x                 └──────────────→ x
    
  SGD oscillates wildly               RMSProp adapts per-axis:
  in steep x-direction                 x-lr shrinks, y-lr stays large
                                       → diagonal path to minimum
```

### Why It Works

```
  x-direction: |∂f/∂x| = 200|x| (large)  →  E_x large  →  α_x small
  y-direction: |∂f/∂y| = 2|y|   (small)   →  E_y small  →  α_y large
  
  Effective step sizes become balanced across dimensions!
```

---

## 6. Worked Example

### Problem
Minimize f(x, y) = 50x² + y² with RMSProp, α=0.1, ρ=0.9, starting at (2, 10).

### Solution

**Step 0:** θ = (2, 10), E = (0, 0)
```
g = (200, 20)
E = 0.9·(0,0) + 0.1·(200², 20²) = (4000, 40)
θ = (2, 10) - 0.1·(200/√4000, 20/√40)
  = (2, 10) - 0.1·(3.162, 3.162)
  = (2, 10) - (0.316, 0.316)
  = (1.684, 9.684)
```

Note: RMSProp equalizes the step size in both directions!

**Step 1:** θ = (1.684, 9.684), E = (4000, 40)
```
g = (168.4, 19.368)
E = 0.9·(4000, 40) + 0.1·(168.4², 19.368²) = (6434.6, 73.51)
θ = (1.684, 9.684) - 0.1·(168.4/√6434.6, 19.368/√73.51)
  = (1.684, 9.684) - 0.1·(2.098, 2.259)
  = (1.684, 9.684) - (0.210, 0.226)
  = (1.474, 9.458)
```

Compare with AdaGrad Step 1: θ = (1.5, 9.5) — similar early behavior, but RMSProp won't slow down as much later.

---

## 7. Python Implementation (PyTorch)

### Using PyTorch's RMSprop

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(3000, 20)
y = (X @ torch.randn(20, 1) + 0.3 * torch.randn(3000, 1)).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Train function ──────────────────────────────────────────────
def train(opt_cls, kwargs, epochs=50):
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(20, 128), nn.ReLU(),
                          nn.Linear(128, 64), nn.ReLU(),
                          nn.Linear(64, 1))
    criterion = nn.MSELoss()
    optimizer = opt_cls(model.parameters(), **kwargs)
    history = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = criterion(model(xb).squeeze(), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        history.append(total / len(X))
    return history

# Compare optimizers
sgd_loss = train(torch.optim.SGD, {"lr": 0.01, "momentum": 0.9})
adagrad_loss = train(torch.optim.Adagrad, {"lr": 0.01})
rmsprop_loss = train(torch.optim.RMSprop, {"lr": 0.001, "alpha": 0.9})

print(f"SGD+Mom:  {sgd_loss[-1]:.4f}")
print(f"AdaGrad:  {adagrad_loss[-1]:.4f}")
print(f"RMSProp:  {rmsprop_loss[-1]:.4f}")
```

### Manual RMSProp Implementation

```python
class ManualRMSProp:
    """RMSProp optimizer implemented from scratch."""
    
    def __init__(self, params, lr=0.001, rho=0.9, eps=1e-8):
        self.params = list(params)
        self.lr = lr
        self.rho = rho
        self.eps = eps
        self.E = [torch.zeros_like(p) for p in self.params]
    
    def zero_grad(self):
        for p in self.params:
            if p.grad is not None:
                p.grad.zero_()
    
    def step(self):
        for i, p in enumerate(self.params):
            if p.grad is None:
                continue
            g = p.grad.data
            
            # Update EMA of squared gradients
            self.E[i] = self.rho * self.E[i] + (1 - self.rho) * g * g
            
            # Adaptive update
            p.data -= self.lr / (torch.sqrt(self.E[i]) + self.eps) * g

# Usage
model = nn.Sequential(nn.Linear(20, 64), nn.ReLU(), nn.Linear(64, 1))
optimizer = ManualRMSProp(model.parameters(), lr=0.001, rho=0.9)
```

### Comparing AdaGrad vs RMSProp Over Long Training

```python
import matplotlib.pyplot as plt

epochs = 100
adagrad_long = train(torch.optim.Adagrad, {"lr": 0.01}, epochs=epochs)
rmsprop_long = train(torch.optim.RMSprop, {"lr": 0.001, "alpha": 0.9}, epochs=epochs)

fig, ax = plt.subplots(figsize=(10, 5))
ax.plot(adagrad_long, label='AdaGrad', color='orange', linewidth=2)
ax.plot(rmsprop_long, label='RMSProp', color='blue', linewidth=2)
ax.set_xlabel('Epoch')
ax.set_ylabel('Loss')
ax.set_title('AdaGrad vs RMSProp: Long Training')
ax.legend()
ax.grid(True, alpha=0.3)
ax.annotate('AdaGrad lr decays →\nlearning stalls',
            xy=(70, adagrad_long[70]), fontsize=10,
            arrowprops=dict(arrowstyle='->'))
plt.tight_layout()
plt.savefig("adagrad_vs_rmsprop.png", dpi=150)
plt.show()
```

---

## 8. RMSProp with Momentum

PyTorch's RMSProp supports an optional **momentum** parameter:

```python
optimizer = torch.optim.RMSprop(
    model.parameters(),
    lr=0.001,
    alpha=0.9,       # ρ (EMA decay for squared gradients)
    eps=1e-8,
    momentum=0.9     # optional: add momentum on top
)
```

The combined update:
```
Eₜ = ρ · Eₜ₋₁ + (1-ρ) · gₜ²
buffₜ = momentum · buffₜ₋₁ + gₜ / √(Eₜ + ε)
θₜ₊₁ = θₜ - α · buffₜ
```

This combines adaptive learning rates (RMSProp) with gradient smoothing (momentum).

---

## 9. Comparison: AdaGrad vs RMSProp

```
  Training Loss over 100 Epochs:
  
  Loss ↑
  2.0 │╲
      │ ╲·
  1.5 │  ╲ ·
      │   ──╲·
  1.0 │      ─╲·  ·  · ·  · ·  · ·    ← AdaGrad plateaus (lr → 0)
      │        ──╲
  0.5 │           ──╲
      │              ──╲
  0.2 │                 ──────────────  ← RMSProp continues improving
      │
  0.0 │
      └──────────────────────────────────→ Epoch
       0    20    40    60    80   100
```

| Feature | AdaGrad | RMSProp |
|---|---|---|
| Squared gradient tracking | Cumulative sum | Exponential moving average |
| Learning rate behavior | Monotonically decreasing | Adapts to recent gradients |
| Long training | Stops learning | Continues learning |
| Sparse data | Excellent | Good |
| Non-convex optimization | Poor (lr dies) | Good |
| Default ρ | N/A | 0.9 |
| Memory overhead | Same (one buffer) | Same (one buffer) |
| Inventor | Duchi et al. (2011) | Hinton (2012, lecture) |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Core fix | Replace AdaGrad's sum with exponential moving average |
| EMA update | Eₜ = ρ·Eₜ₋₁ + (1-ρ)·gₜ² |
| Parameter update | θ = θ - α/√(Eₜ+ε) · g |
| Decay rate ρ | Typically 0.9; controls memory window ≈ 1/(1-ρ) |
| Effective lr | α / RMS(recent gradients) — bounded, not decaying |
| vs AdaGrad | Non-monotonic lr → can train longer |
| Name meaning | **R**oot **M**ean **S**quare **Prop**agation |
| PyTorch | `torch.optim.RMSprop(params, lr=0.001, alpha=0.9)` |

---

## Revision Questions

1. **What is the fundamental problem with AdaGrad that RMSProp solves?** Show mathematically why AdaGrad's learning rate goes to zero and RMSProp's doesn't.

2. **Derive the steady-state effective learning rate of RMSProp** for a parameter with constant gradient magnitude |g|.

3. **Explain the role of ρ in RMSProp.** What happens with ρ=0 (no memory) and ρ=1 (infinite memory)?

4. **How does RMSProp relate to natural gradient methods?** What approximation does it make?

5. **Compare RMSProp with SGD+Momentum.** Do they solve different problems? Could they be combined? (Hint: yes, and it leads toward Adam.)

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← AdaGrad](./04-adagrad.md) | [Unit 5: Optimization](./README.md) | [Adam Optimizer →](./06-adam-optimizer.md) |
