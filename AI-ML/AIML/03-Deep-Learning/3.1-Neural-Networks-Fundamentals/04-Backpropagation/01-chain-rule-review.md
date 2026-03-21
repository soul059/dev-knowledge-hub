# 🔗 Chain Rule Review

> **Chapter 4.1 — The Mathematical Foundation of Backpropagation**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Chain Rule of Calculus |
| **Prerequisites** | Single-variable and multivariable calculus |
| **Key Insight** | The chain rule lets us compute how the loss depends on ANY parameter in the network, no matter how many layers deep |

---

## 📌 Overview

Backpropagation is nothing more than a clever and efficient application of the **chain rule of calculus**. Before diving into the full backprop algorithm, we must have a rock-solid understanding of the chain rule in both single-variable and multivariable forms. This chapter reviews the chain rule from first principles, builds intuition with examples, and shows exactly why it's the mathematical engine that powers all of deep learning.

---

## 1. Single-Variable Chain Rule

### 1.1 Statement

If y = f(u) and u = g(x), then:

```
    dy     dy   du
    ── = ── · ──
    dx     du   dx

    Or equivalently:
    d/dx [f(g(x))] = f'(g(x)) · g'(x)

    "The derivative of the outer function (evaluated at the inner)
     times the derivative of the inner function"
```

### 1.2 Visual Intuition

```
    x ──── g(x) = u ──── f(u) = y

    If x changes by Δx:
    • u changes by approximately g'(x) · Δx
    • y changes by approximately f'(u) · [g'(x) · Δx]
    • Therefore: Δy/Δx ≈ f'(u) · g'(x) = dy/du · du/dx

    The chain rule "chains" together local rates of change!
```

### 1.3 Worked Examples

**Example 1: Simple Composition**
```
    y = (3x + 2)⁵

    Let u = 3x + 2,  y = u⁵

    dy/du = 5u⁴ = 5(3x + 2)⁴
    du/dx = 3

    dy/dx = dy/du · du/dx = 5(3x + 2)⁴ · 3 = 15(3x + 2)⁴
```

**Example 2: Sigmoid Function (Critical for Neural Networks!)**
```
    σ(z) = 1/(1 + e^(-z))

    Let u = 1 + e^(-z),  σ = 1/u = u^(-1)

    dσ/du = -u^(-2) = -1/(1 + e^(-z))²
    du/dz = -e^(-z)

    dσ/dz = dσ/du · du/dz
           = -1/(1 + e^(-z))² · (-e^(-z))
           = e^(-z) / (1 + e^(-z))²

    Simplification:
    = [1/(1 + e^(-z))] · [e^(-z)/(1 + e^(-z))]
    = σ(z) · [1 - σ(z)]

    ┌────────────────────────────────────────┐
    │  σ'(z) = σ(z) · (1 - σ(z))           │
    │                                        │
    │  This beautiful result means we can    │
    │  compute the derivative from the       │
    │  output itself — no extra computation! │
    └────────────────────────────────────────┘
```

**Example 3: Longer Chain (3 functions)**
```
    y = sin(e^(x²))

    Let:  v = x²        (innermost)
          u = e^v = e^(x²)  (middle)
          y = sin(u)     (outermost)

    dy/dx = dy/du · du/dv · dv/dx
          = cos(u) · e^v · 2x
          = cos(e^(x²)) · e^(x²) · 2x
          = 2x · e^(x²) · cos(e^(x²))

    The chain rule extends to ANY number of composed functions!
```

### 1.4 The General Multi-Link Chain

```
    For y = f₁(f₂(f₃(...fₙ(x)...))):

    dy     dy    da_{n-1}   da_{n-2}        da₁
    ── = ──── · ──────── · ──────── · ... · ────
    dx   da_{n-1}  da_{n-2}   da_{n-3}       dx

    This is EXACTLY what backpropagation does!
    Each layer is one function in the chain.
```

---

## 2. Multivariable Chain Rule

### 2.1 Statement

If L depends on y₁, y₂, ..., yₙ, and each yᵢ depends on x, then:

```
    ∂L     n    ∂L   ∂yᵢ
    ── = Σ   ── · ──
    ∂x    i=1  ∂yᵢ   ∂x

    "Sum over ALL paths from L to x"
```

### 2.2 Why This Matters for Neural Networks

In a neural network, a single weight affects the loss through MULTIPLE neurons:

```
    Weight w feeds into neuron → affects MULTIPLE downstream paths:

                        ┌── z₁ → a₁ ──┐
                        │              │
    x ── w ── z ── a ──┼── z₂ → a₂ ──┼── L
                        │              │
                        └── z₃ → a₃ ──┘

    ∂L/∂w = ∂L/∂z₁ · ∂z₁/∂a · ∂a/∂z · ∂z/∂w
           + ∂L/∂z₂ · ∂z₂/∂a · ∂a/∂z · ∂z/∂w
           + ∂L/∂z₃ · ∂z₃/∂a · ∂a/∂z · ∂z/∂w

    = (∂L/∂z₁ · ∂z₁/∂a + ∂L/∂z₂ · ∂z₂/∂a + ∂L/∂z₃ · ∂z₃/∂a) · ∂a/∂z · ∂z/∂w

    We SUM contributions from all downstream paths.
```

### 2.3 Worked Example — Multivariable

```
    f(x, y) = x²y + sin(y)
    x = 2t + 1
    y = t²

    df/dt = ∂f/∂x · dx/dt + ∂f/∂y · dy/dt
          = (2xy) · (2) + (x² + cos(y)) · (2t)

    At t = 1:  x = 3, y = 1
    df/dt = (2·3·1)(2) + (9 + cos(1))(2)
          = 12 + (9 + 0.5403)(2)
          = 12 + 19.08
          = 31.08
```

---

## 3. Chain Rule in Neural Network Context

### 3.1 A Simple 2-Layer Network

```
    Forward pass:
    z⁽¹⁾ = w₁x + b₁
    a⁽¹⁾ = σ(z⁽¹⁾)
    z⁽²⁾ = w₂a⁽¹⁾ + b₂
    ŷ = σ(z⁽²⁾)
    L = (y - ŷ)²

    To find ∂L/∂w₁, apply the chain rule:

    ∂L/∂w₁ = ∂L/∂ŷ · ∂ŷ/∂z⁽²⁾ · ∂z⁽²⁾/∂a⁽¹⁾ · ∂a⁽¹⁾/∂z⁽¹⁾ · ∂z⁽¹⁾/∂w₁

    Each term:
    ∂L/∂ŷ      = -2(y - ŷ)                  ← loss derivative
    ∂ŷ/∂z⁽²⁾   = σ'(z⁽²⁾) = ŷ(1-ŷ)         ← output activation
    ∂z⁽²⁾/∂a⁽¹⁾ = w₂                         ← weight
    ∂a⁽¹⁾/∂z⁽¹⁾ = σ'(z⁽¹⁾) = a⁽¹⁾(1-a⁽¹⁾)   ← hidden activation
    ∂z⁽¹⁾/∂w₁  = x                          ← input

    ∂L/∂w₁ = -2(y-ŷ) · ŷ(1-ŷ) · w₂ · a⁽¹⁾(1-a⁽¹⁾) · x
```

### 3.2 Chain Rule = Signal Flowing Backward

```
    Forward pass (left to right):
    x ──►[W₁,b₁]──► z₁ ──►[σ]──► a₁ ──►[W₂,b₂]──► z₂ ──►[σ]──► ŷ ──►[L]──► Loss

    Backward pass (right to left — chain rule!):
    ∂L/∂w₁ ◄──[×x]◄── ∂L/∂z₁ ◄──[×σ']◄── ∂L/∂a₁ ◄──[×w₂]◄── ∂L/∂z₂ ◄──[×σ']◄── ∂L/∂ŷ ◄── -2(y-ŷ)

    Each "◄──[×?]◄──" is one application of the chain rule!
    The gradient "flows" backward through the network.
```

### 3.3 Numerical Example

```
    Given: x = 2, w₁ = 0.5, b₁ = 0.1, w₂ = -0.3, b₂ = 0.2, y = 1

    Forward:
    z₁ = 0.5(2) + 0.1 = 1.1
    a₁ = σ(1.1) = 1/(1+e^(-1.1)) = 0.7503
    z₂ = -0.3(0.7503) + 0.2 = -0.0251
    ŷ = σ(-0.0251) = 0.4937

    Loss: L = (1 - 0.4937)² = 0.2563

    Backward (chain rule):
    ∂L/∂ŷ = -2(1 - 0.4937) = -1.0126
    ∂ŷ/∂z₂ = 0.4937(1-0.4937) = 0.2500
    ∂z₂/∂w₂ = a₁ = 0.7503
    ∂z₂/∂a₁ = w₂ = -0.3
    ∂a₁/∂z₁ = 0.7503(1-0.7503) = 0.1875
    ∂z₁/∂w₁ = x = 2

    ∂L/∂w₂ = ∂L/∂ŷ · ∂ŷ/∂z₂ · ∂z₂/∂w₂
            = (-1.0126)(0.2500)(0.7503)
            = -0.1899

    ∂L/∂w₁ = ∂L/∂ŷ · ∂ŷ/∂z₂ · ∂z₂/∂a₁ · ∂a₁/∂z₁ · ∂z₁/∂w₁
            = (-1.0126)(0.2500)(-0.3)(0.1875)(2)
            = 0.0285

    Update (lr = 0.1):
    w₂ ← w₂ - 0.1 × (-0.1899) = -0.3 + 0.019 = -0.281
    w₁ ← w₁ - 0.1 × (0.0285) = 0.5 - 0.003 = 0.497
```

---

## 4. Python Verification

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

def sigmoid_derivative(z):
    s = sigmoid(z)
    return s * (1 - s)

# Forward pass
x, w1, b1, w2, b2, y = 2.0, 0.5, 0.1, -0.3, 0.2, 1.0

z1 = w1 * x + b1           # = 1.1
a1 = sigmoid(z1)            # = 0.7503
z2 = w2 * a1 + b2           # = -0.0251
y_hat = sigmoid(z2)         # = 0.4937
L = (y - y_hat) ** 2        # = 0.2563

print("Forward Pass:")
print(f"  z1 = {z1:.4f}, a1 = {a1:.4f}")
print(f"  z2 = {z2:.4f}, ŷ = {y_hat:.4f}")
print(f"  Loss = {L:.4f}")

# Backward pass (chain rule)
dL_dy_hat = -2 * (y - y_hat)                    # -1.0126
dy_hat_dz2 = sigmoid_derivative(z2)              # 0.2500
dz2_dw2 = a1                                     # 0.7503
dz2_da1 = w2                                     # -0.3
da1_dz1 = sigmoid_derivative(z1)                  # 0.1875
dz1_dw1 = x                                      # 2.0

dL_dw2 = dL_dy_hat * dy_hat_dz2 * dz2_dw2
dL_dw1 = dL_dy_hat * dy_hat_dz2 * dz2_da1 * da1_dz1 * dz1_dw1

print(f"\nBackward Pass (Chain Rule):")
print(f"  ∂L/∂ŷ = {dL_dy_hat:.4f}")
print(f"  ∂ŷ/∂z₂ = {dy_hat_dz2:.4f}")
print(f"  ∂L/∂w₂ = {dL_dw2:.4f}")
print(f"  ∂L/∂w₁ = {dL_dw1:.4f}")

# Verify with numerical gradient
eps = 1e-5
def compute_loss(w1, w2):
    z1 = w1 * x + b1
    a1 = sigmoid(z1)
    z2 = w2 * a1 + b2
    y_hat = sigmoid(z2)
    return (y - y_hat) ** 2

numerical_dw2 = (compute_loss(w1, w2+eps) - compute_loss(w1, w2-eps)) / (2*eps)
numerical_dw1 = (compute_loss(w1+eps, w2) - compute_loss(w1-eps, w2)) / (2*eps)

print(f"\nNumerical Verification:")
print(f"  ∂L/∂w₂: analytical={dL_dw2:.6f}, numerical={numerical_dw2:.6f}")
print(f"  ∂L/∂w₁: analytical={dL_dw1:.6f}, numerical={numerical_dw1:.6f}")
print(f"  Match: {'✅' if abs(dL_dw2 - numerical_dw2) < 1e-4 else '❌'}")
```

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Single-var chain rule** | dy/dx = dy/du · du/dx |
| **Multi-var chain rule** | ∂L/∂x = Σᵢ (∂L/∂yᵢ)(∂yᵢ/∂x) — sum over all paths |
| **Extended chain** | d/dx f₁(f₂(...fₙ(x))) = f₁'·f₂'·...·fₙ' (product of local derivatives) |
| **Neural network** | ∂L/∂wₗ = ∂L/∂aₗ · ... · ∂z/∂wₗ — gradient flows backward layer by layer |
| **Key insight** | Backpropagation IS the chain rule, applied efficiently |
| **Sigmoid derivative** | σ'(z) = σ(z)(1-σ(z)) — compute from output alone |
| **Sum over paths** | When a variable affects L through multiple paths, sum all contributions |

---

## ❓ Revision Questions

1. **Compute dy/dx for y = ln(cos(x²)). Show each application of the chain rule explicitly.**

2. **Derive the sigmoid derivative σ'(z) = σ(z)(1-σ(z)) step by step from σ(z) = 1/(1+e⁻ᶻ).**

3. **In a neural network, why do we need the multivariable chain rule (summing over paths) rather than just the single-variable version?**

4. **Given x=1, w₁=0.3, b₁=0, w₂=0.5, b₂=0, y=0, with sigmoid activations and MSE loss, compute ∂L/∂w₁ and ∂L/∂w₂ using the chain rule. Show all steps.**

5. **Why is the chain rule sometimes described as "local gradients multiplied together"? What is a "local gradient" in the context of a computational graph node?**

6. **If a neural network has L layers, how many terms are multiplied in the chain rule to compute ∂Loss/∂W¹? How does this relate to the vanishing gradient problem?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Custom Loss Functions](../03-Loss-Functions/04-custom-loss-functions.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Computational Graphs →](02-computational-graphs.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
