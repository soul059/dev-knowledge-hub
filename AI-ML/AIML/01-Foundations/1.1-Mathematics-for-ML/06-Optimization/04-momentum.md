# Chapter 4: Momentum

> **Unit 6 · Optimization · Module 4 of 7**

---

## 4.1 Overview

Momentum accelerates gradient descent by accumulating a **velocity** from past gradients, much like a ball rolling downhill gains speed. It dampens oscillations in directions of high curvature and amplifies progress in consistent gradient directions. Momentum is a foundational component of nearly all modern optimizers (Adam, RMSProp, etc.).

---

## 4.2 Physics Analogy

Imagine a ball rolling down a hilly landscape:

```
  Surface               Without Momentum        With Momentum
                         (GD bounces)            (smooth path)
      ╱  ╲                  ↓↗↙↘↓                    ↓
     ╱    ╲                ↗  ↙                       ↓
    ╱  ╱╲  ╲              ↗  ↙                        ↓
   ╱  ╱  ╲  ╲           ↗  ↙                          ↓
  ╱  ╱ ·  ╲  ╲         ↗  ↙                         → · ←
  ──╱──·───╲──╲──    ··↗··↙··                      ·····*·····
      min                min                          min
                     Zigzags slowly              Rolls straight down
```

Key physical intuition:
- **Velocity** accumulates in the consistent downhill direction
- **Friction** (1−β) prevents infinite acceleration
- **Inertia** carries the ball through small bumps (noisy gradients)

---

## 4.3 Momentum Update Rule

### Classical Momentum (Polyak, 1964)

```
v_t = β · v_(t-1) + ∇J(θ_t)          ← velocity update
θ_(t+1) = θ_t − α · v_t              ← parameter update
```

Alternative (equivalent) formulation:

```
v_t = β · v_(t-1) + α · ∇J(θ_t)
θ_(t+1) = θ_t − v_t
```

| Symbol | Meaning | Typical Value |
|--------|---------|---------------|
| `v_t` | Velocity vector at step t | — |
| `β` | Momentum coefficient | 0.9 |
| `α` | Learning rate | 0.01 |
| `∇J(θ_t)` | Current gradient | — |

---

## 4.4 Exponentially Weighted Moving Average

The velocity `v_t` is an **exponentially weighted moving average** (EWMA) of past gradients:

```
v_t = β · v_(t-1) + (1−β) · g_t     (with rescaling, sometimes written this way)

Expanding:
v_t = g_t + β·g_(t-1) + β²·g_(t-2) + β³·g_(t-3) + ...
```

Recent gradients contribute more; older gradients decay exponentially.

```
  Weight
  1.0 |·
      | ·
      |  ·
  β   |   ·
      |    ·
  β²  |     ·
  β³  |      ·
      |        · · · · ·
  0   +—————————————————→  Steps ago
      0   1   2   3   4   5   6

  With β=0.9: effectively averages ~10 recent gradients
  With β=0.99: effectively averages ~100 recent gradients

  Effective window ≈ 1 / (1 − β)
```

---

## 4.5 How Momentum Dampens Oscillations

Consider optimizing a function with an elongated valley (different curvatures in θ₁ vs θ₂):

```
  Without Momentum (GD):        With Momentum:

  θ₂ ↑                          θ₂ ↑
     |     ╭─────────╮             |     ╭─────────╮
     |   ╭─┤  ╭───╮  ├─╮          |   ╭─┤  ╭───╮  ├─╮
     | x→╱ │  │ * │  │ ╲          | x──→──→──→ * │  │
     |   ╲ │  │   │  │ ╱          |   ╰─┤  ╰───╯  ├─╯
     |   ╰↗╲↙↗╲↙↗╲↙ ├─╯          |     ╰─────────╯
     |     ╰─────────╯             |
     +——————————————→ θ₁           +——————————————→ θ₁

     Oscillates across valley       Smooth, fast convergence
     (gradients in θ₂ cancel)       (θ₂ oscillations cancel in EWMA)
```

**Why it works**: Gradient components that oscillate (e.g., across the valley) cancel out in the moving average, while components that consistently point downhill (along the valley) accumulate.

---

## 4.6 Momentum Coefficient β

| β Value | Behavior |
|---------|----------|
| 0.0 | No momentum (vanilla GD) |
| 0.5 | Light momentum |
| **0.9** | **Standard choice** — averages ~10 gradients |
| 0.95 | Strong momentum |
| 0.99 | Very strong — averages ~100 gradients, can overshoot |

### Effect of β on Convergence

```
  Loss
   |
   | \                               β = 0 (no momentum)
   |   \___
   |       \___________
   |
   | \                               β = 0.9 (standard)
   |  \___
   |      \___
   |
   |  \_                             β = 0.99 (aggressive)
   |    \___↗↘___
   |              \_________          May overshoot then recover
   +————————————————————————→ Iterations
```

---

## 4.7 Nesterov Accelerated Gradient (NAG)

NAG computes the gradient at the **lookahead** position, not the current position:

```
Standard Momentum:  Look at current position → compute gradient → move
Nesterov Momentum:  Look ahead (using velocity) → compute gradient → correct

v_t = β · v_(t-1) + α · ∇J(θ_t − β · v_(t-1))     ← gradient at lookahead
θ_(t+1) = θ_t − v_t
```

### Nesterov Lookahead Illustration

```
  θ_current ──── β·v (momentum step) ────→ θ_lookahead
       │                                       │
       │                                  Compute ∇J here
       │                                       │
       └───────── actual update ←──────────────┘
```

### Why Nesterov Is Better

```
  Nesterov "looks ahead" to see if we're about to overshoot:

  ──→──→──→ lookahead  →  "gradient says SLOW DOWN"  →  smaller step
                           (corrective signal)

  Result: Faster convergence, less overshooting
```

**Convergence rate**: NAG achieves `O(1/t²)` for convex functions vs `O(1/t)` for vanilla GD.

---

## 4.8 Step-by-Step Example

**Minimize**: `f(x) = x⁴ − 3x³ + 2` starting at `x₀ = 3.0`

**Gradient**: `f'(x) = 4x³ − 9x²`

Settings: `α = 0.002, β = 0.9, v₀ = 0`

| Step | x | f'(x) | v_t = β·v + f'(x) | x_new = x − α·v |
|------|-------|---------|-----------|------------|
| 0 | 3.000 | 27.000 | 27.000 | 2.946 |
| 1 | 2.946 | 24.490 | 48.790 | 2.848 |
| 2 | 2.848 | 19.440 | 63.351 | 2.721 |
| 3 | 2.721 | 14.216 | 71.232 | 2.579 |
| 4 | 2.579 | 9.278 | 73.387 | 2.432 |
| ... | ... | ... | ... | ... |

Without momentum (vanilla GD at same α), this would take many more steps.

---

## 4.9 Python Implementation

```python
import numpy as np

def sgd_momentum(X, y, lr=0.01, beta=0.9, n_epochs=100, batch_size=32):
    """
    Mini-batch SGD with momentum for linear regression.
    """
    m, n = X.shape
    theta = np.zeros(n)
    velocity = np.zeros(n)
    history = []

    for epoch in range(n_epochs):
        indices = np.random.permutation(m)

        for start in range(0, m, batch_size):
            end = min(start + batch_size, m)
            idx = indices[start:end]
            X_b, y_b = X[idx], y[idx]
            B = len(y_b)

            # Gradient
            gradient = (1/B) * X_b.T @ (X_b @ theta - y_b)

            # Momentum update
            velocity = beta * velocity + gradient
            theta = theta - lr * velocity

        cost = (1/(2*m)) * np.sum((X @ theta - y)**2)
        history.append(cost)

    return theta, history


def sgd_nesterov(X, y, lr=0.01, beta=0.9, n_epochs=100, batch_size=32):
    """
    Mini-batch SGD with Nesterov momentum.
    """
    m, n = X.shape
    theta = np.zeros(n)
    velocity = np.zeros(n)

    for epoch in range(n_epochs):
        indices = np.random.permutation(m)

        for start in range(0, m, batch_size):
            end = min(start + batch_size, m)
            idx = indices[start:end]
            X_b, y_b = X[idx], y[idx]
            B = len(y_b)

            # Lookahead position
            theta_lookahead = theta - lr * beta * velocity

            # Gradient at lookahead
            gradient = (1/B) * X_b.T @ (X_b @ theta_lookahead - y_b)

            # Update velocity and parameters
            velocity = beta * velocity + gradient
            theta = theta - lr * velocity

    return theta
```

### PyTorch Usage

```python
import torch.optim as optim

# Standard momentum
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

# Nesterov momentum
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9, nesterov=True)
```

---

## 4.10 Real-World ML Applications

| Application | Role of Momentum |
|------------|-----------------|
| **CNN Training (ResNet, VGG)** | Standard SGD + momentum (β=0.9) is the default optimizer |
| **Adam Optimizer** | Uses first-moment estimate (momentum-like) + second-moment |
| **Super-Convergence** | High momentum (0.95) combined with learning rate warmup |
| **GAN Training** | Momentum stabilizes the adversarial dynamics |
| **Federated Learning** | Server-side momentum aggregates client updates |

---

## 4.11 Comparison: With vs Without Momentum

| Property | Vanilla GD | GD + Momentum | GD + Nesterov |
|----------|-----------|---------------|---------------|
| Oscillation dampening | ❌ | ✅ | ✅✅ |
| Speed through valleys | Slow | Fast | Faster |
| Overshooting risk | Low | Moderate | Low (corrective) |
| Convergence rate (convex) | O(1/t) | O(1/t) | O(1/t²) |
| Extra memory | None | +1 velocity vector | +1 velocity vector |
| Hyperparameters | α | α, β | α, β |

---

## 4.12 Summary Table

| Concept | Key Detail |
|---------|-----------|
| Velocity update | `v = β·v + ∇J(θ)` |
| Parameter update | `θ = θ − α·v` |
| β (momentum coeff.) | Typically 0.9 |
| Effective window | ≈ 1/(1−β) past gradients |
| Nesterov | Evaluate gradient at lookahead position |
| Key benefit | Dampens oscillations, accelerates convergence |
| Key cost | One extra vector (velocity) in memory |
| When to use | Almost always — default with SGD |

---

## 4.13 Quick Revision Questions

1. **What physical analogy explains momentum?**
   → A ball rolling downhill accumulates velocity; friction prevents infinite speed.

2. **What is the effective averaging window for β = 0.9?**
   → Approximately 1/(1−0.9) = 10 recent gradients.

3. **How does momentum help with elongated cost surfaces?**
   → Oscillating gradient components cancel in the moving average; consistent components accumulate.

4. **What is the key difference between classical and Nesterov momentum?**
   → Nesterov evaluates the gradient at the lookahead position (θ − β·v), providing a corrective signal.

5. **What extra memory does momentum require?**
   → One velocity vector of the same shape as θ.

6. **What is a typical value of β used in practice?**
   → β = 0.9 is the standard default; β = 0.99 for very smooth optimization.

---

| [← Previous: Mini-Batch GD](03-mini-batch-gradient-descent.md) | [Next Chapter: Learning Rate Scheduling →](05-learning-rate-scheduling.md) |
|:---|---:|
