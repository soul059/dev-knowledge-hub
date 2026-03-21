# 2. Momentum

> **Unit 5 · Optimization Algorithms** — Accelerating gradient descent with exponentially weighted moving averages

---

## Chapter Overview

Vanilla SGD treats every gradient independently — each update depends only on the current mini-batch. **Momentum** adds memory: it accumulates a **velocity** vector that summarizes past gradients, allowing the optimizer to build up speed in consistent directions and dampen oscillations in noisy or ill-conditioned directions. The physical analogy is a **ball rolling downhill** — it gains momentum on slopes and coasts through flat regions.

---

## 1. The Problem with Vanilla SGD

### Oscillation in Ill-Conditioned Problems

Consider a loss surface shaped like a narrow valley (different curvatures along different axes):

```
  θ₂ ↑  (high curvature — steep walls)
     │    ╱╲    ╱╲    ╱╲
     │   ╱  ╲  ╱  ╲  ╱  ╲
     │  ╱    ╲╱    ╲╱    ╲    ← SGD oscillates across the valley
     │ ╱                   ╲
     │╱          ★           ╲  ★ = optimum
     └──────────────────────────→ θ₁ (low curvature — gentle slope)
     
  SGD path: zigzags across θ₂, slow progress along θ₁
```

The gradient is **large** across the valley (θ₂ direction) but **small** along the valley floor (θ₁ direction). SGD oscillates violently in θ₂ while making glacial progress in θ₁.

### Why This Happens

For an elongated loss surface with condition number κ = λ_max / λ_min:
- Gradient in the steep direction: large magnitude → large steps → overshoot → oscillation
- Gradient in the gentle direction: small magnitude → tiny steps → slow convergence
- Optimal learning rate is constrained by the steep direction: α < 2/λ_max

---

## 2. Exponentially Weighted Moving Average (EWMA)

### Definition

Given a sequence of values g₁, g₂, g₃, ..., the EWMA is:

```
vₜ = β · vₜ₋₁ + (1 - β) · gₜ
```

where β ∈ [0, 1) is the **decay rate**.

### Interpretation

Expanding the recursion:

```
vₜ = (1-β) · gₜ + (1-β)β · gₜ₋₁ + (1-β)β² · gₜ₋₂ + ...
```

This is a weighted sum of all past values where:
- **Recent values** have higher weight
- **Older values** decay exponentially by factor β

### Effective Window

The EWMA approximately averages the last **1/(1-β)** values:

| β | Effective Window | Memory |
|---|---|---|
| 0.9 | ~10 steps | Short-term |
| 0.95 | ~20 steps | Medium-term |
| 0.99 | ~100 steps | Long-term |

```
  Weight of past gradients (β = 0.9):
  
  1.0 │ ■
      │ ■
  0.8 │ ■
      │ ■
  0.6 │ ■
      │ ■ ■
  0.4 │ ■ ■
      │ ■ ■ ■
  0.2 │ ■ ■ ■ ■
      │ ■ ■ ■ ■ ■ ■
  0.0 │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■
      └─────────────────────────→ Steps ago
       0  1  2  3  4  5  6  7  8  9  10
```

---

## 3. Momentum Update Rule

### Algorithm

```
Initialize: v₀ = 0 (velocity vector, same shape as θ)

For each step t:
    1. Compute gradient:  gₜ = ∇θ J(θₜ)
    2. Update velocity:   vₜ = β · vₜ₋₁ + (1 - β) · gₜ     ← EWMA of gradients
    3. Update parameters: θₜ₊₁ = θₜ - α · vₜ
```

### Alternative (More Common) Formulation

Many frameworks (including PyTorch) use this equivalent form:

```
vₜ = β · vₜ₋₁ + gₜ               ← note: no (1-β) factor
θₜ₊₁ = θₜ - α · vₜ
```

The difference is absorbed into the learning rate α. PyTorch uses this convention.

### Comparison

| Formulation | Velocity Update | Effect |
|---|---|---|
| EWMA form | vₜ = β·vₜ₋₁ + (1-β)·gₜ | vₜ has same scale as gₜ |
| PyTorch form | vₜ = β·vₜ₋₁ + gₜ | vₜ grows up to 1/(1-β) × gₜ |

Both are mathematically equivalent with appropriate α scaling.

---

## 4. Physical Analogy: Ball Rolling Downhill

```
  Loss Surface (side view):
  
  ╲         ╱╲                  Ball with momentum:
   ╲       ╱  ╲                 • Accelerates on downhill slopes
    ╲     ╱    ╲                • Coasts through flat regions
     ╲   ╱  ●→  ╲              • Can overshoot, but dampens
      ╲ ╱  (ball) ╲               oscillations over time
       ╲    with    ╲
        ╲ momentum   ╲
         ╲     ★      ╲        ★ = minimum
          ╰────────────╯

  Without momentum:             With momentum:
  Ball stops at every flat      Ball rolls through flats,
  spot, oscillates on slopes    smoothly descends to minimum
```

### Physics Mapping

| Physics | Optimization |
|---|---|
| Position | Parameters θ |
| Velocity | Velocity vector v |
| Gravity (slope) | Gradient ∇J |
| Friction (drag) | Decay factor β < 1 |
| Mass | Implicit in β |

The decay β < 1 acts as **friction** — without it (β=1), the ball would never stop.

---

## 5. How Momentum Dampens Oscillations

### In the Elongated Valley

```
  Without Momentum (vanilla SGD):        With Momentum (β = 0.9):
  
  θ₂ ↑                                   θ₂ ↑
     │    /\  /\  /\                         │    
     │   /  \/  \/  \                        │   ╲
     │  /              \                     │    ╲___
     │ S      ★         │                   │ S     ╲___
     │  \              /                     │          ╲___★
     │   \  /\  /\  /                        │
     └────────────────→ θ₁                   └────────────────→ θ₁
     
  Oscillations cancel    →  velocity in θ₂ ≈ 0 (gradients alternate ±)
  Consistent gradient    →  velocity in θ₁ builds up (gradients same sign)
```

**Why it works:**
- **θ₂ direction** (oscillating): Gradients alternate sign → EWMA cancels out → small velocity
- **θ₁ direction** (consistent): Gradients point same way → EWMA accumulates → large velocity

### Mathematical Analysis

If the gradient alternates in one direction: gₜ = +G, gₜ₊₁ = -G, gₜ₊₂ = +G, ...

```
Velocity (EWMA form):
  v₁ = (1-β)G
  v₂ = β(1-β)G + (1-β)(-G) = (1-β)G(β-1) = -(1-β)²G  ← nearly zero for β≈0.9
  
With β=0.9: v₂ = -(0.01)G ≈ 0    ✓ Oscillation suppressed!
```

If the gradient is consistent: gₜ = G for all t:

```
v∞ = (1-β)G · [1 + β + β² + ...] = (1-β)G · 1/(1-β) = G

With PyTorch form: v∞ = G/(1-β) = 10G for β=0.9    ✓ 10× acceleration!
```

---

## 6. Convergence Speed Analysis

### Terminal Velocity

In the PyTorch formulation, with a constant gradient g:

```
v∞ = g / (1 - β)
```

Effective step size = α · v∞ = α · g / (1-β)

| β | Speedup Factor 1/(1-β) |
|---|---|
| 0.0 | 1× (vanilla SGD) |
| 0.5 | 2× |
| 0.9 | **10×** |
| 0.95 | 20× |
| 0.99 | 100× |

### Convergence Rate

For a strongly convex function with condition number κ:

| Method | Convergence Rate |
|---|---|
| Vanilla GD | O((κ-1)/(κ+1))ᵗ |
| Momentum GD | O((√κ-1)/(√κ+1))ᵗ |

Momentum improves dependence from κ to √κ — a dramatic improvement for ill-conditioned problems!

---

## 7. Worked Example

### Problem
Minimize f(θ₁, θ₂) = 10θ₁² + θ₂² using momentum with β=0.9, α=0.02, starting at θ=(1, 1).

### Solution

**Gradient:** ∇f = (20θ₁, 2θ₂)

**Step 0:** θ = (1, 1), v = (0, 0)
```
g₀ = (20, 2)
v₀ = 0.9·(0,0) + (20, 2) = (20, 2)          [PyTorch form]
θ₁ = (1, 1) - 0.02·(20, 2) = (0.60, 0.96)
```

**Step 1:** θ = (0.60, 0.96)
```
g₁ = (12.0, 1.92)
v₁ = 0.9·(20, 2) + (12.0, 1.92) = (30.0, 3.72)
θ₂ = (0.60, 0.96) - 0.02·(30.0, 3.72) = (0.00, 0.886)
```

**Step 2:** θ = (0.00, 0.886)
```
g₂ = (0.0, 1.772)
v₂ = 0.9·(30.0, 3.72) + (0.0, 1.772) = (27.0, 5.12)
θ₃ = (0.00, 0.886) - 0.02·(27.0, 5.12) = (-0.54, 0.784)
```

Notice: momentum in θ₁ causes overshoot (goes negative), but this eventually settles. The θ₂ direction converges more smoothly.

---

## 8. Python Implementation (PyTorch)

### Using PyTorch's Built-in Momentum

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(2000, 20)
y = (X @ torch.randn(20, 1) + 0.3 * torch.randn(2000, 1)).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Compare Vanilla SGD vs Momentum ───────────────────────────
def train(optimizer_fn, epochs=30):
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(20, 128), nn.ReLU(), nn.Linear(128, 1))
    criterion = nn.MSELoss()
    optimizer = optimizer_fn(model.parameters())
    
    history = []
    for epoch in range(epochs):
        total_loss = 0
        for xb, yb in loader:
            pred = model(xb).squeeze()
            loss = criterion(pred, yb)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            total_loss += loss.item() * len(xb)
        history.append(total_loss / len(X))
    return history

# Vanilla SGD
sgd_loss = train(lambda p: torch.optim.SGD(p, lr=0.01))

# SGD with Momentum β = 0.9
mom_loss = train(lambda p: torch.optim.SGD(p, lr=0.01, momentum=0.9))

# SGD with Momentum β = 0.95
mom95_loss = train(lambda p: torch.optim.SGD(p, lr=0.01, momentum=0.95))

print(f"Vanilla SGD  final loss: {sgd_loss[-1]:.4f}")
print(f"Momentum 0.9 final loss: {mom_loss[-1]:.4f}")
print(f"Momentum 0.95 final loss: {mom95_loss[-1]:.4f}")
```

### Manual Momentum Implementation

```python
class SGDMomentum:
    """Manual implementation of SGD with momentum (PyTorch convention)."""
    
    def __init__(self, params, lr=0.01, momentum=0.9):
        self.params = list(params)
        self.lr = lr
        self.beta = momentum
        # Initialize velocity buffers
        self.velocities = [torch.zeros_like(p) for p in self.params]
    
    def zero_grad(self):
        for p in self.params:
            if p.grad is not None:
                p.grad.zero_()
    
    def step(self):
        for i, p in enumerate(self.params):
            if p.grad is None:
                continue
            g = p.grad.data
            # Update velocity: v = β·v + g
            self.velocities[i] = self.beta * self.velocities[i] + g
            # Update parameters: θ = θ - α·v
            p.data -= self.lr * self.velocities[i]

# Usage
model = nn.Sequential(nn.Linear(20, 128), nn.ReLU(), nn.Linear(128, 1))
optimizer = SGDMomentum(model.parameters(), lr=0.01, momentum=0.9)
```

### Visualizing Momentum Effect on 2D Loss Surface

```python
import numpy as np
import matplotlib.pyplot as plt

def rosenbrock(x, y, a=1, b=5):
    """Elongated valley — good for showing momentum benefit."""
    return (a - x)**2 + b * (y - x**2)**2

def grad_rosenbrock(x, y, a=1, b=5):
    dx = -2*(a - x) + b * 2 * (y - x**2) * (-2*x)
    dy = b * 2 * (y - x**2)
    return np.array([dx, dy])

def optimize_2d(lr, beta, steps=200):
    """Run SGD+momentum on 2D Rosenbrock."""
    pos = np.array([-1.0, 2.0])
    vel = np.array([0.0, 0.0])
    path = [pos.copy()]
    
    for _ in range(steps):
        g = grad_rosenbrock(pos[0], pos[1])
        vel = beta * vel + g
        pos = pos - lr * vel
        path.append(pos.copy())
    
    return np.array(path)

# Compare
path_sgd = optimize_2d(lr=0.001, beta=0.0, steps=500)   # no momentum
path_mom = optimize_2d(lr=0.001, beta=0.9, steps=500)    # with momentum

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
for ax, path, title in [(axes[0], path_sgd, "Vanilla SGD"),
                         (axes[1], path_mom, "SGD + Momentum (β=0.9)")]:
    xx, yy = np.meshgrid(np.linspace(-2, 2, 100), np.linspace(-1, 3, 100))
    zz = rosenbrock(xx, yy)
    ax.contour(xx, yy, zz, levels=np.logspace(-1, 3, 20), cmap='viridis', alpha=0.6)
    ax.plot(path[:, 0], path[:, 1], 'r.-', markersize=2, linewidth=0.5)
    ax.plot(1, 1, 'g*', markersize=15)  # optimum
    ax.set_title(title)
    ax.set_xlabel("θ₁"); ax.set_ylabel("θ₂")

plt.tight_layout()
plt.savefig("momentum_comparison.png", dpi=150)
plt.show()
```

---

## 9. Comparison: Vanilla SGD vs Momentum SGD

```
  Training Loss over Epochs:
  
  Loss ↑
  2.0 │╲
      │ ╲ ·  ·                    · · · Vanilla SGD
  1.5 │  ╲·  ·  ·                ───── Momentum (β=0.9)
      │   ─╲ ·  ·  ·
  1.0 │    ─╲  ·  · · ·
      │      ──╲  ·  · · ·
  0.5 │        ──╲·  ·  · · · ·
      │           ───── · · · · · · ·
  0.0 │               ─────────────
      └──────────────────────────────→ Epoch
       0    5    10   15   20   25
  
  Momentum reaches low loss faster and with less oscillation.
```

| Aspect | Vanilla SGD | SGD + Momentum |
|---|---|---|
| Update formula | θ -= α·g | v = βv+g; θ -= α·v |
| Memory | No extra | +1 buffer per param |
| Oscillation | High | Dampened |
| Speed on flat regions | Slow | Fast (accumulated velocity) |
| Overshoot risk | Low | Moderate (tune β) |
| Typical β | N/A | 0.9 (standard), 0.95–0.99 (heavy) |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| EWMA | vₜ = β·vₜ₋₁ + (1-β)·gₜ |
| Momentum update (PyTorch) | vₜ = β·vₜ₋₁ + gₜ; θₜ₊₁ = θₜ - α·vₜ |
| Decay rate β | Controls memory length ≈ 1/(1-β) steps |
| Terminal velocity | v∞ = g/(1-β) → up to 1/(1-β) speedup |
| Oscillation dampening | Alternating gradients cancel in EWMA |
| Convergence improvement | κ → √κ (condition number dependence) |
| Physical analogy | Ball rolling downhill with friction |
| PyTorch | `torch.optim.SGD(params, lr=0.01, momentum=0.9)` |

---

## Revision Questions

1. **Explain why momentum dampens oscillations in the high-curvature direction.** Use the EWMA to show that alternating gradients cancel out.

2. **Derive the terminal velocity** for a constant gradient g with momentum parameter β. What is the effective speedup?

3. **What is the physical analogy for momentum in optimization?** Map each physical quantity (position, velocity, gravity, friction) to its optimization counterpart.

4. **Compare the convergence rates** of vanilla GD and momentum GD on a strongly convex function with condition number κ=100.

5. **Why is β=0.9 a common default?** What happens if β is too high (e.g., 0.999) or too low (e.g., 0.5)?

6. **Implement SGD with momentum from scratch in PyTorch.** Your implementation should match `torch.optim.SGD(momentum=0.9)` behavior.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Gradient Descent Variants](./01-gradient-descent-variants.md) | [Unit 5: Optimization](./README.md) | [Nesterov Accelerated Gradient →](./03-nesterov-accelerated-gradient.md) |
