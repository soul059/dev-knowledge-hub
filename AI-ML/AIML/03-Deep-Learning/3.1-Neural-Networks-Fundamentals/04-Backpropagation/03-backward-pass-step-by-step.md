# ⬅️ Backward Pass Step-by-Step

> **Chapter 4.3 — Complete Backpropagation Through a 2-Layer Network**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Deriving All Gradients |
| **Prerequisites** | Chapters 4.1-4.2 (Chain Rule, Computational Graphs) |
| **Key Insight** | Backprop computes ALL gradients in a single backward pass, reusing computations (dynamic programming) |

---

## 📌 Overview

This chapter walks through **complete backpropagation** in a 2-layer neural network with actual numbers. We derive every gradient — ∂L/∂W², ∂L/∂b², ∂L/∂W¹, ∂L/∂b¹ — step by step, using the cached values from the forward pass. By the end, you'll understand exactly how every parameter in a network receives its gradient update signal.

---

## 1. Network Setup

### 1.1 Architecture

```
    2-Layer Network: Input(2) → Hidden(3, ReLU) → Output(1, Sigmoid)
    Loss: MSE = (y - ŷ)²

    ┌────┐     W¹(3×2), b¹(3×1)     ┌────┐     W²(1×3), b²(1×1)     ┌────┐
    │ x₁ ├────────────────────────►│    ├────────────────────────►│    │
    │    │     Layer 1 (ReLU)       │ h₁ │     Layer 2 (Sigmoid)   │ ŷ  │
    │ x₂ ├────────────────────────►│ h₂ ├────────────────────────►│    │
    └────┘                          │ h₃ │                          └────┘
                                    └────┘
```

### 1.2 Concrete Values

```
    Input:  x = [1.0, -0.5]ᵀ
    Target: y = 1.0

    W¹ = ┌              ┐       b¹ = ┌     ┐
         │  0.3  -0.2   │            │ 0.1 │
         │  0.5   0.4   │            │-0.1 │
         │ -0.1   0.3   │            │ 0.0 │
         └              ┘            └     ┘

    W² = ┌                    ┐     b² = ┌    ┐
         │  0.4  -0.3   0.2  │          │0.1 │
         └                    ┘          └    ┘
```

---

## 2. Forward Pass (with Caching)

### 2.1 Layer 1: Linear + ReLU

```
    z¹ = W¹ · x + b¹

    ┌              ┐   ┌     ┐       ┌     ┐
    │  0.3  -0.2   │   │ 1.0 │       │ 0.1 │
    │  0.5   0.4   │ · │-0.5 │   +   │-0.1 │
    │ -0.1   0.3   │   └     ┘       │ 0.0 │
    └              ┘                  └     ┘

    z₁¹ = 0.3(1.0) + (-0.2)(-0.5) + 0.1 = 0.3 + 0.1 + 0.1 = 0.5
    z₂¹ = 0.5(1.0) + 0.4(-0.5) + (-0.1) = 0.5 - 0.2 - 0.1 = 0.2
    z₃¹ = -0.1(1.0) + 0.3(-0.5) + 0.0 = -0.1 - 0.15 + 0 = -0.25

    z¹ = [0.5, 0.2, -0.25]ᵀ     ← CACHE THIS

    a¹ = ReLU(z¹):
    a₁¹ = max(0, 0.5)  = 0.5
    a₂¹ = max(0, 0.2)  = 0.2
    a₃¹ = max(0, -0.25) = 0.0    ← Dead neuron!

    a¹ = [0.5, 0.2, 0.0]ᵀ       ← CACHE THIS
```

### 2.2 Layer 2: Linear + Sigmoid

```
    z² = W² · a¹ + b²

    z² = [0.4, -0.3, 0.2] · [0.5, 0.2, 0.0]ᵀ + 0.1
       = 0.4(0.5) + (-0.3)(0.2) + 0.2(0.0) + 0.1
       = 0.2 - 0.06 + 0 + 0.1
       = 0.24

    z² = 0.24                    ← CACHE THIS

    a² = σ(z²) = σ(0.24) = 1/(1 + e⁻⁰·²⁴)
       = 1/(1 + 0.7866)
       = 1/1.7866
       = 0.5597

    ŷ = a² = 0.5597              ← CACHE THIS
```

### 2.3 Loss Computation

```
    L = (y - ŷ)² = (1.0 - 0.5597)² = (0.4403)² = 0.1939
```

### 2.4 Cache Summary

```
    ┌───────────────────────────────────────┐
    │           CACHED VALUES               │
    ├───────────────────────────────────────┤
    │  a⁰ = x  = [1.0, -0.5]ᵀ             │
    │  z¹      = [0.5, 0.2, -0.25]ᵀ        │
    │  a¹      = [0.5, 0.2, 0.0]ᵀ          │
    │  z²      = 0.24                       │
    │  a² = ŷ  = 0.5597                     │
    │  y       = 1.0                        │
    │  L       = 0.1939                     │
    └───────────────────────────────────────┘
```

---

## 3. Backward Pass — Step by Step

### 3.1 Overview of What We're Computing

```
    We need four gradients (one for each learnable parameter):
    
    1. ∂L/∂W²  — how to adjust output layer weights
    2. ∂L/∂b²  — how to adjust output layer bias
    3. ∂L/∂W¹  — how to adjust hidden layer weights
    4. ∂L/∂b¹  — how to adjust hidden layer bias

    Strategy: Work backward from the loss!
    
    L ← a² ← z² ← (W², b², a¹) ← a¹ ← z¹ ← (W¹, b¹, x)
    
    Compute in order: ∂L/∂a² → ∂L/∂z² → ∂L/∂W², ∂L/∂b² → ∂L/∂a¹ → ∂L/∂z¹ → ∂L/∂W¹, ∂L/∂b¹
```

### 3.2 Step 1: ∂L/∂a² (gradient of loss w.r.t. output)

```
    L = (y - a²)²

    ∂L/∂a² = -2(y - a²) = -2(1.0 - 0.5597) = -2(0.4403) = -0.8806
```

### 3.3 Step 2: ∂L/∂z² (through sigmoid)

```
    a² = σ(z²)   →   ∂a²/∂z² = σ(z²)(1 - σ(z²)) = a²(1 - a²)
                                = 0.5597(1 - 0.5597) = 0.5597(0.4403) = 0.2464

    By chain rule:
    ∂L/∂z² = ∂L/∂a² · ∂a²/∂z²
            = (-0.8806)(0.2464)
            = -0.2170

    We define: δ² = ∂L/∂z² = -0.2170   (the "error signal" for layer 2)
```

### 3.4 Step 3: ∂L/∂W² and ∂L/∂b² (output layer gradients)

```
    z² = W² · a¹ + b²

    ∂z²/∂W² = (a¹)ᵀ     (the activation from the previous layer)
    ∂z²/∂b² = 1

    Therefore:
    ∂L/∂W² = ∂L/∂z² · (a¹)ᵀ = δ² · (a¹)ᵀ

    ∂L/∂W² = (-0.2170) · [0.5, 0.2, 0.0]
            = [-0.1085, -0.0434, 0.0000]

    ┌─────────────────────────────────────────────────┐
    │  ∂L/∂W² = [-0.1085, -0.0434, 0.0000]           │
    │  ∂L/∂b² = δ² = -0.2170                         │
    │                                                  │
    │  Interpretation:                                 │
    │  • w₁² should increase (gradient negative)       │
    │  • w₂² should increase slightly                  │
    │  • w₃² has zero gradient (dead ReLU neuron!)     │
    └─────────────────────────────────────────────────┘
```

### 3.5 Step 4: ∂L/∂a¹ (propagate error backward to hidden layer)

```
    z² = W² · a¹ + b²

    ∂z²/∂a¹ = (W²)ᵀ

    ∂L/∂a¹ = (W²)ᵀ · ∂L/∂z²

    ┌     ┐         ┌     ┐
    │ 0.4 │         │ 0.4 │
    │-0.3 │ · (-0.2170) = │-0.3 │ · (-0.2170)
    │ 0.2 │         │ 0.2 │
    └     ┘         └     ┘

    ∂L/∂a₁¹ = 0.4 × (-0.2170) = -0.0868
    ∂L/∂a₂¹ = -0.3 × (-0.2170) = 0.0651
    ∂L/∂a₃¹ = 0.2 × (-0.2170) = -0.0434

    ∂L/∂a¹ = [-0.0868, 0.0651, -0.0434]ᵀ
```

### 3.6 Step 5: ∂L/∂z¹ (through ReLU)

```
    a¹ = ReLU(z¹)

    ReLU'(z) = { 1 if z > 0
               { 0 if z ≤ 0

    ∂a¹/∂z¹ = diag(ReLU'(z¹)) = diag([1, 1, 0])

    (z₁¹ = 0.5 > 0 → derivative = 1)
    (z₂¹ = 0.2 > 0 → derivative = 1)
    (z₃¹ = -0.25 ≤ 0 → derivative = 0)   ← Dead neuron: NO gradient passes!

    ∂L/∂z¹ = ∂L/∂a¹ ⊙ ReLU'(z¹)    (element-wise multiplication)

    ∂L/∂z₁¹ = (-0.0868) × 1 = -0.0868
    ∂L/∂z₂¹ = (0.0651) × 1 = 0.0651
    ∂L/∂z₃¹ = (-0.0434) × 0 = 0.0000    ← Blocked by dead ReLU!

    δ¹ = ∂L/∂z¹ = [-0.0868, 0.0651, 0.0000]ᵀ
```

### 3.7 Step 6: ∂L/∂W¹ and ∂L/∂b¹ (hidden layer gradients)

```
    z¹ = W¹ · x + b¹

    ∂L/∂W¹ = δ¹ · (a⁰)ᵀ = δ¹ · xᵀ

    ┌         ┐   ┌           ┐
    │ -0.0868 │   │           │
    │  0.0651 │ · │ 1.0  -0.5│
    │  0.0000 │   │           │
    └         ┘   └           ┘
    (3×1)           (1×2)

    ∂L/∂W¹ = ┌                       ┐
             │ -0.0868   0.0434      │
             │  0.0651  -0.0326      │
             │  0.0000   0.0000      │
             └                       ┘

    ∂L/∂b¹ = δ¹ = [-0.0868, 0.0651, 0.0000]ᵀ
```

---

## 4. Complete Results Summary

```
    ┌────────────────────────────────────────────────────────┐
    │              BACKWARD PASS — ALL GRADIENTS              │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │  Layer 2 (Output):                                     │
    │  ─────────────────                                     │
    │  ∂L/∂W² = [-0.1085, -0.0434, 0.0000]                 │
    │  ∂L/∂b² = -0.2170                                     │
    │                                                        │
    │  Layer 1 (Hidden):                                     │
    │  ─────────────────                                     │
    │  ∂L/∂W¹ = ┌                       ┐                   │
    │           │ -0.0868   0.0434      │                   │
    │           │  0.0651  -0.0326      │                   │
    │           │  0.0000   0.0000      │                   │
    │           └                       ┘                   │
    │  ∂L/∂b¹ = [-0.0868, 0.0651, 0.0000]ᵀ                 │
    │                                                        │
    │  Note: Row 3 of ∂L/∂W¹ and ∂L/∂b¹ is all zeros        │
    │  because neuron 3 was killed by ReLU (z₃¹ = -0.25 < 0) │
    └────────────────────────────────────────────────────────┘
```

### 4.1 Parameter Update (SGD with lr=0.1)

```
    W² ← W² - 0.1 × ∂L/∂W²
    = [0.4, -0.3, 0.2] - 0.1 × [-0.1085, -0.0434, 0.0000]
    = [0.4109, -0.2957, 0.2000]

    b² ← b² - 0.1 × ∂L/∂b²
    = 0.1 - 0.1 × (-0.2170) = 0.1217

    W¹ ← W¹ - 0.1 × ∂L/∂W¹
    = ┌              ┐   ┌                       ┐
      │  0.3  -0.2   │ - 0.1 × │ -0.0868   0.0434      │
      │  0.5   0.4   │         │  0.0651  -0.0326      │
      │ -0.1   0.3   │         │  0.0000   0.0000      │
      └              ┘         └                       ┘
    = ┌                  ┐
      │  0.3087  -0.2043 │
      │  0.4935   0.4033 │
      │ -0.1000   0.3000 │
      └                  ┘

    After update, re-run forward pass → loss should decrease!
```

---

## 5. Python Verification

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

# Setup
x = np.array([[1.0], [-0.5]])
y = 1.0
W1 = np.array([[0.3, -0.2], [0.5, 0.4], [-0.1, 0.3]])
b1 = np.array([[0.1], [-0.1], [0.0]])
W2 = np.array([[0.4, -0.3, 0.2]])
b2 = np.array([[0.1]])

# ──── Forward Pass ────
z1 = W1 @ x + b1
a1 = np.maximum(0, z1)  # ReLU
z2 = W2 @ a1 + b2
a2 = sigmoid(z2)
y_hat = a2[0, 0]
L = (y - y_hat) ** 2

print("FORWARD PASS:")
print(f"  z1 = {z1.flatten()}")
print(f"  a1 = {a1.flatten()}")
print(f"  z2 = {z2[0,0]:.4f}")
print(f"  ŷ  = {y_hat:.4f}")
print(f"  L  = {L:.4f}")

# ──── Backward Pass ────
# Step 1: ∂L/∂a2
dL_da2 = -2 * (y - y_hat)

# Step 2: ∂L/∂z2
da2_dz2 = y_hat * (1 - y_hat)
dL_dz2 = dL_da2 * da2_dz2

# Step 3: ∂L/∂W2, ∂L/∂b2
dL_dW2 = dL_dz2 * a1.T
dL_db2 = dL_dz2

# Step 4: ∂L/∂a1
dL_da1 = W2.T * dL_dz2

# Step 5: ∂L/∂z1 (through ReLU)
relu_mask = (z1 > 0).astype(float)
dL_dz1 = dL_da1 * relu_mask

# Step 6: ∂L/∂W1, ∂L/∂b1
dL_dW1 = dL_dz1 @ x.T
dL_db1 = dL_dz1

print(f"\nBACKWARD PASS:")
print(f"  ∂L/∂a² = {dL_da2:.4f}")
print(f"  ∂L/∂z² = {dL_dz2:.4f}")
print(f"  ∂L/∂W² = {dL_dW2.flatten()}")
print(f"  ∂L/∂b² = {dL_db2[0,0]:.4f}")
print(f"  ∂L/∂a¹ = {dL_da1.flatten()}")
print(f"  ReLU mask = {relu_mask.flatten()}")
print(f"  ∂L/∂z¹ = {dL_dz1.flatten()}")
print(f"  ∂L/∂W¹ =\n{dL_dW1}")
print(f"  ∂L/∂b¹ = {dL_db1.flatten()}")

# ──── Update and verify loss decreases ────
lr = 0.1
W2_new = W2 - lr * dL_dW2
b2_new = b2 - lr * dL_db2
W1_new = W1 - lr * dL_dW1
b1_new = b1 - lr * dL_db1

# Recompute loss with updated parameters
z1_new = W1_new @ x + b1_new
a1_new = np.maximum(0, z1_new)
z2_new = W2_new @ a1_new + b2_new
y_hat_new = sigmoid(z2_new)[0, 0]
L_new = (y - y_hat_new) ** 2

print(f"\nAfter 1 gradient step (lr={lr}):")
print(f"  Old loss: {L:.6f}")
print(f"  New loss: {L_new:.6f}")
print(f"  Decreased: {'✅ Yes!' if L_new < L else '❌ No'}")
print(f"  Improvement: {(L - L_new)/L*100:.2f}%")
```

---

## 6. General Backprop Pattern

```
    For ANY L-layer network, the backward pass follows this pattern:

    Initialize: dA^[L] = ∂L/∂a^[L]  (depends on loss function)

    For l = L, L-1, ..., 1:
    ┌─────────────────────────────────────────────────────────┐
    │  dZ^[l] = dA^[l] ⊙ g'^[l](Z^[l])   (through activation)│
    │  dW^[l] = dZ^[l] · (A^[l-1])ᵀ       (weight gradient)   │
    │  db^[l] = dZ^[l]                      (bias gradient)    │
    │  dA^[l-1] = (W^[l])ᵀ · dZ^[l]       (propagate backward)│
    └─────────────────────────────────────────────────────────┘

    Where ⊙ denotes element-wise multiplication.
```

---

## 📊 Summary Table

| Step | Computation | Formula | Uses Cached |
|---|---|---|---|
| 1 | Loss gradient | dA^[L] = ∂L/∂a^[L] | a^[L], y |
| 2 | Through activation | dZ^[l] = dA^[l] ⊙ g'(Z^[l]) | Z^[l] |
| 3 | Weight gradient | dW^[l] = dZ^[l] · (A^[l-1])ᵀ | A^[l-1] |
| 4 | Bias gradient | db^[l] = dZ^[l] | — |
| 5 | Propagate back | dA^[l-1] = (W^[l])ᵀ · dZ^[l] | W^[l] |

---

## ❓ Revision Questions

1. **Walk through the complete backward pass for a 2-layer network with x=[2,-1]ᵀ, W¹=[[1,0],[0,1],[1,1]], b¹=[0,0,0]ᵀ, W²=[[1,-1,1]], b²=[0], using ReLU hidden, sigmoid output, MSE loss, and y=0. Compute all gradients.**

2. **Why does the dead ReLU neuron (z₃¹ = -0.25) have zero gradient for all its incoming weights? Will this neuron ever learn?**

3. **In step 4 (∂L/∂a¹), why do we multiply by (W²)ᵀ instead of W²? What is the geometric interpretation?**

4. **If we didn't cache z¹ and a¹ during the forward pass, could we still compute the backward pass? At what cost?**

5. **Write the general backpropagation equations for an L-layer network. Why is the backward pass O(same complexity) as the forward pass?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Computational Graphs](02-computational-graphs.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Gradient Computation →](04-gradient-computation.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
