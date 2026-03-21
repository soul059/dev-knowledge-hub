# 3. Nesterov Accelerated Gradient (NAG)

> **Unit 5 · Optimization Algorithms** — Looking ahead before stepping to improve momentum

---

## Chapter Overview

Standard momentum computes the gradient at the **current position** and then adds it to the velocity. Nesterov Accelerated Gradient (NAG) flips the order: it first takes a **lookahead step** in the direction of the accumulated velocity, then computes the gradient at that anticipated position. This "look before you leap" strategy provides a **corrective mechanism** that slows down before overshooting, leading to faster and more stable convergence.

---

## 1. The Problem with Standard Momentum

### Overshooting

With standard momentum, the velocity can build up too much, causing the optimizer to **overshoot** the minimum and oscillate:

```
  Standard Momentum near a minimum:
  
  Loss ↑
       │            velocity carries us
       │            past the minimum!
       │    ╲      ╱
       │     ╲    ╱
       │      ╲  ╱
       │       ╲╱  ← overshoot
       │        ★  ← minimum
       └──────────────→ θ

  The gradient at the current position doesn't "see"
  the upcoming valley wall until it's too late.
```

### Why Lookahead Helps

If we could peek at where momentum is taking us, we could compute the gradient at that **future** position. If we're about to overshoot, the future gradient will point backward, naturally **braking** the velocity.

---

## 2. Nesterov Update Formula

### Standard Momentum (Review)

```
vₜ = β · vₜ₋₁ + ∇J(θₜ)              ← gradient at current position
θₜ₊₁ = θₜ - α · vₜ
```

### Nesterov Momentum

```
vₜ = β · vₜ₋₁ + ∇J(θₜ - α · β · vₜ₋₁)    ← gradient at LOOKAHEAD position
θₜ₊₁ = θₜ - α · vₜ
```

### Step-by-Step Process

```
Step 1: Lookahead — Compute anticipated position:
        θ_lookahead = θₜ - α · β · vₜ₋₁

Step 2: Compute gradient at the lookahead position:
        gₜ = ∇J(θ_lookahead)

Step 3: Update velocity:
        vₜ = β · vₜ₋₁ + gₜ

Step 4: Update parameters:
        θₜ₊₁ = θₜ - α · vₜ
```

### Visualization of the Difference

```
  Standard Momentum:              Nesterov Momentum:
  
       θₜ (current)                    θₜ (current)
        │                               │
        │ ① compute ∇J(θₜ)              │ ① lookahead: θ̃ = θₜ - αβvₜ₋₁
        │                               │
        ▼                               ▼
   gₜ = ∇J(θₜ)                    θ̃ (anticipated position)
        │                               │
        │ ② velocity: βv + g            │ ② compute ∇J(θ̃)
        │                               │
        ▼                               ▼
   vₜ = βvₜ₋₁ + gₜ                gₜ = ∇J(θ̃)
        │                               │
        │ ③ update: θ -= αv              │ ③ velocity: βv + g
        │                               │
        ▼                               ▼
       θₜ₊₁                        vₜ = βvₜ₋₁ + gₜ
                                        │
                                        │ ④ update: θ -= αv
                                        │
                                        ▼
                                       θₜ₊₁
```

---

## 3. Why Looking Ahead Helps

### Intuition: Corrective Braking

```
  Scenario: Approaching minimum from the left with high velocity
  
  Loss ↑
       │             Standard momentum:
       │    ╲        gradient at θₜ still points right (downhill)
       │     ╲       → velocity increases → OVERSHOOT
       │      ╲
       │       ╲                    Nesterov:
       │  θₜ ●  ╲★                 gradient at θ̃ points LEFT (uphill)
       │  ↑      │                  → velocity decreases → CONTROLLED
       │  │      │
       │  current│ θ̃ ● ← lookahead lands past minimum
       └──────────────→ θ
  
  At θ̃ (past the minimum), ∇J points backward → acts as a brake!
```

### Key Insight

| Situation | Standard Momentum | Nesterov |
|---|---|---|
| Approaching minimum | Gradient at θₜ still says "go forward" | Gradient at θ̃ says "slow down" |
| Overshooting | Detects overshoot only at next step | **Pre-corrects** before overshooting |
| Flat region | Same behavior (gradient ≈ 0) | Same behavior |

---

## 4. Equivalent Reformulation (For Implementation)

The lookahead formulation requires computing gradients at a non-standard point. A common **reparametrization** (used in PyTorch) reformulates NAG as:

### Substitution: φₜ = θₜ - α·β·vₜ

This leads to an equivalent update entirely in terms of the current parameters:

```
gₜ = ∇J(φₜ)                                   ← gradient at current (reparametrized) point
vₜ = β · vₜ₋₁ + gₜ                             ← velocity update (same as standard)
φₜ₊₁ = φₜ - α · (β · vₜ + gₜ)                  ← modified parameter update
```

This is what PyTorch implements when you set `nesterov=True`.

---

## 5. Convergence Rate Improvement

### Theoretical Rates for Smooth Convex Functions

| Method | Convergence Rate | Notes |
|---|---|---|
| Gradient Descent | O(1/t) | Standard rate |
| Momentum (Polyak) | O(1/t) | Same asymptotic rate, faster constant |
| **Nesterov (NAG)** | **O(1/t²)** | **Optimal for smooth convex** |

For strongly convex functions with condition number κ:

| Method | Linear Convergence Rate |
|---|---|
| Gradient Descent | 1 - 2/(κ+1) ≈ 1 - 2/κ |
| Momentum | 1 - 2/(√κ+1) ≈ 1 - 2/√κ |
| **Nesterov** | **1 - 2/(√κ+1)** (same rate, but achieved provably) |

Nesterov's method achieves the **optimal convergence rate** for first-order methods (Nesterov, 1983).

### Practical Improvement

In deep learning (non-convex), the theoretical guarantees don't directly apply, but NAG often:
- Converges slightly faster than standard momentum
- Oscillates less near the minimum
- Provides marginal but consistent improvement

---

## 6. Comparison with Standard Momentum

### Convergence Path Comparison

```
  Contour plot of f(x,y) = 10x² + y²:
  
  y ↑
    │    ╭──────────────╮
    │   ╱   ╭────────╮   ╲
    │  ╱   ╱  ╭────╮  ╲   ╲
    │ ╱   ╱  ╱  ★   ╲  ╲   ╲    ★ = optimum at (0,0)
    │ ╲   ╲  ╲      ╱  ╱   ╱
    │  ╲   ╲  ╰────╯  ╱   ╱
    │   ╲   ╰────────╯   ╱
    │    ╰──────────────╯
    └──────────────────────→ x
    
  Standard Momentum (β=0.9):
  S ──╲──╱──╲──╱──╲──★            (oscillates before settling)
  
  Nesterov (β=0.9):
  S ──╲─╱──╲─★                    (fewer oscillations, faster settling)
```

### Side-by-Side

| Aspect | Standard Momentum | Nesterov (NAG) |
|---|---|---|
| Gradient computation | At current θ | At lookahead θ̃ |
| Overshoot behavior | Detects after the fact | Pre-corrects |
| Convergence (convex) | O(1/t) | **O(1/t²)** |
| Computation cost | Same | Same (just different gradient point) |
| Implementation | Simple | Slightly complex (but PyTorch handles it) |
| Deep learning benefit | Good | Marginally better |

---

## 7. Worked Example

### Problem
Minimize f(θ) = θ² with Nesterov momentum, β=0.9, α=0.1, starting at θ₀=10.

### Solution (PyTorch Convention)

**Step 0:** θ = 10, v = 0

```
Lookahead: θ̃ = θ - α·β·v = 10 - 0.1·0.9·0 = 10
g = f'(θ̃) = 2·10 = 20
v₀ = β·v + g = 0.9·0 + 20 = 20
θ₁ = θ - α·v₀ = 10 - 0.1·20 = 8.0
```

**Step 1:** θ = 8.0, v = 20

```
Lookahead: θ̃ = 8.0 - 0.1·0.9·20 = 8.0 - 1.8 = 6.2
g = f'(θ̃) = 2·6.2 = 12.4
v₁ = 0.9·20 + 12.4 = 30.4
θ₂ = 8.0 - 0.1·30.4 = 4.96
```

**Step 2:** θ = 4.96, v = 30.4

```
Lookahead: θ̃ = 4.96 - 0.1·0.9·30.4 = 4.96 - 2.736 = 2.224
g = f'(θ̃) = 2·2.224 = 4.448
v₂ = 0.9·30.4 + 4.448 = 31.808
θ₃ = 4.96 - 0.1·31.808 = 1.779
```

Compare Standard Momentum:
```
Step 0: g=20, v=20, θ₁=8.0        (same)
Step 1: g=2·8=16, v=34, θ₂=4.6    (standard: gradient at current θ=8)
Step 2: g=2·4.6=9.2, v=39.8, θ₃=0.62 → then overshoots!
```

NAG: 10 → 8.0 → 4.96 → 1.779 → ... (smoother approach)
Standard: 10 → 8.0 → 4.6 → 0.62 → -3.36 → ... (overshoots!)

---

## 8. Python Implementation (PyTorch)

### Using PyTorch's Nesterov Flag

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset

# ── Data ────────────────────────────────────────────────────────
torch.manual_seed(42)
X = torch.randn(2000, 20)
y = (X @ torch.randn(20, 1) + torch.randn(2000, 1) * 0.3).squeeze()
loader = DataLoader(TensorDataset(X, y), batch_size=64, shuffle=True)

# ── Train function ──────────────────────────────────────────────
def train(optim_fn, epochs=40):
    torch.manual_seed(0)
    model = nn.Sequential(nn.Linear(20, 128), nn.ReLU(),
                          nn.Linear(128, 64), nn.ReLU(),
                          nn.Linear(64, 1))
    criterion = nn.MSELoss()
    optimizer = optim_fn(model.parameters())
    history = []
    for epoch in range(epochs):
        total = 0
        for xb, yb in loader:
            loss = criterion(model(xb).squeeze(), yb)
            optimizer.zero_grad(); loss.backward(); optimizer.step()
            total += loss.item() * len(xb)
        history.append(total / len(X))
    return history

# ── Compare Three Optimizers ────────────────────────────────────
vanilla = train(lambda p: torch.optim.SGD(p, lr=0.01))
momentum = train(lambda p: torch.optim.SGD(p, lr=0.01, momentum=0.9))
nesterov = train(lambda p: torch.optim.SGD(p, lr=0.01, momentum=0.9, nesterov=True))

print(f"Vanilla SGD:  {vanilla[-1]:.4f}")
print(f"Momentum:     {momentum[-1]:.4f}")
print(f"Nesterov:     {nesterov[-1]:.4f}")
```

### Manual Nesterov Implementation

```python
class NesterovSGD:
    """Manual implementation of SGD with Nesterov momentum."""
    
    def __init__(self, params, lr=0.01, momentum=0.9):
        self.params = list(params)
        self.lr = lr
        self.beta = momentum
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
            v = self.velocities[i]
            
            # Nesterov update (PyTorch-equivalent formulation):
            # v_new = β * v + g
            # θ_new = θ - α * (β * v_new + g)
            v_new = self.beta * v + g
            self.velocities[i] = v_new
            p.data -= self.lr * (self.beta * v_new + g)

# Verify equivalence with PyTorch
model1 = nn.Linear(5, 1)
model2 = nn.Linear(5, 1)
model2.load_state_dict(model1.state_dict())

opt1 = torch.optim.SGD(model1.parameters(), lr=0.01, momentum=0.9, nesterov=True)
opt2 = NesterovSGD(model2.parameters(), lr=0.01, momentum=0.9)

x = torch.randn(8, 5)
for _ in range(10):
    loss1 = model1(x).sum(); opt1.zero_grad(); loss1.backward(); opt1.step()
    loss2 = model2(x).sum(); opt2.zero_grad(); loss2.backward(); opt2.step()

print(f"Weight diff: {(model1.weight - model2.weight).abs().max():.1e}")  # ≈ 0
```

### Visualization: Momentum vs Nesterov on 2D Surface

```python
import numpy as np
import matplotlib.pyplot as plt

def quadratic(x, y):
    return 50*x**2 + y**2

def grad_quadratic(x, y):
    return np.array([100*x, 2*y])

def run_optimizer(use_nesterov, lr=0.002, beta=0.9, steps=100):
    pos = np.array([1.0, 10.0])
    vel = np.zeros(2)
    path = [pos.copy()]
    
    for _ in range(steps):
        if use_nesterov:
            lookahead = pos - lr * beta * vel
            g = grad_quadratic(lookahead[0], lookahead[1])
        else:
            g = grad_quadratic(pos[0], pos[1])
        
        vel = beta * vel + g
        pos = pos - lr * vel
        path.append(pos.copy())
    
    return np.array(path)

path_mom = run_optimizer(use_nesterov=False)
path_nag = run_optimizer(use_nesterov=True)

fig, ax = plt.subplots(figsize=(10, 6))
xx, yy = np.meshgrid(np.linspace(-1.5, 1.5, 100), np.linspace(-12, 12, 100))
zz = quadratic(xx, yy)
ax.contour(xx, yy, zz, levels=20, cmap='coolwarm', alpha=0.5)
ax.plot(path_mom[:, 0], path_mom[:, 1], 'b.-', label='Momentum', alpha=0.7, markersize=3)
ax.plot(path_nag[:, 0], path_nag[:, 1], 'r.-', label='Nesterov', alpha=0.7, markersize=3)
ax.plot(0, 0, 'g*', markersize=20)
ax.legend(fontsize=12)
ax.set_title("Momentum vs Nesterov on f(x,y) = 50x² + y²")
plt.tight_layout()
plt.savefig("nesterov_vs_momentum.png", dpi=150)
plt.show()
```

---

## 9. When to Use Nesterov

| Scenario | Recommendation |
|---|---|
| Convex optimization | **Use NAG** — provably faster |
| Deep learning with SGD | **Use NAG** — marginal but free improvement |
| Deep learning with Adam | Not applicable (Adam has its own mechanisms) |
| RNN training | Can help with gradient dynamics |
| Small learning rate | Less difference from standard momentum |
| Large learning rate | NAG's lookahead correction is more valuable |

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Core idea | Compute gradient at **lookahead** position, not current |
| Lookahead position | θ̃ = θₜ - α·β·vₜ₋₁ |
| Velocity update | vₜ = β·vₜ₋₁ + ∇J(θ̃) |
| Parameter update | θₜ₊₁ = θₜ - α·vₜ |
| Key benefit | Pre-corrective braking prevents overshoot |
| Convergence (convex) | O(1/t²) — optimal for first-order methods |
| PyTorch | `SGD(params, lr=α, momentum=β, nesterov=True)` |
| Computational cost | Same as standard momentum |

---

## Revision Questions

1. **Explain the lookahead concept in Nesterov momentum.** Why does computing the gradient at the anticipated position help prevent overshooting?

2. **Write the full Nesterov update equations** and compare them step-by-step with standard momentum. Where exactly does the difference lie?

3. **In the worked example, why does standard momentum overshoot while Nesterov doesn't?** Trace through the gradient values at each step to explain.

4. **What is the convergence rate of NAG vs standard GD for smooth convex functions?** Why is this significant?

5. **Derive the equivalent reformulation** that avoids computing the gradient at a separate point (the φₜ substitution). Why does PyTorch use this form?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Momentum](./02-momentum.md) | [Unit 5: Optimization](./README.md) | [AdaGrad →](./04-adagrad.md) |
