# 💀 Vanishing & Exploding Gradients

> **Chapter 4.5 — Why Deep Networks Are Hard to Train (and How to Fix It)**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Gradient Pathologies |
| **Prerequisites** | Chapters 4.1-4.4 |
| **Key Insight** | Gradients are products of many factors — if each factor is < 1, the product vanishes; if > 1, it explodes. This is the fundamental challenge of training deep networks |

---

## 📌 Overview

The vanishing and exploding gradient problems are the most significant obstacles to training deep neural networks. They explain why deep networks were impractical before 2012, and their solutions — ReLU, proper initialization, batch normalization, skip connections, gradient clipping — represent the key innovations that enabled the deep learning revolution. This chapter provides a thorough mathematical analysis of both problems and their solutions.

---

## 1. The Core Problem — Products of Factors

### 1.1 Mathematical Analysis

Consider the gradient of the loss with respect to parameters in layer 1 of an L-layer network:

```
    ∂L        ∂L      ∂a^[L]   ∂a^[L-1]         ∂a^[2]   ∂a^[1]
    ──── = ──── · ────── · ──────── · ... · ──── · ────
    ∂W¹     ∂a^[L]  ∂z^[L]   ∂z^[L-1]         ∂z^[2]   ∂z^[1]  · x

         = ∂L/∂a^[L] · g'(z^[L]) · W^[L] · g'(z^[L-1]) · W^[L-1] · ... · g'(z^[1]) · x

    The gradient is a PRODUCT of (L-1) activation derivatives and (L-1) weight matrices!
```

### 1.2 What Can Go Wrong

```
    ┌──────────────────────────────────────────────────────────────┐
    │           THE PRODUCT OF MANY FACTORS                        │
    │                                                              │
    │  If each factor ≈ 0.25 (sigmoid):                           │
    │    Product after 10 layers: 0.25¹⁰ ≈ 10⁻⁶   → VANISHES!   │
    │    Product after 20 layers: 0.25²⁰ ≈ 10⁻¹²  → ZERO!       │
    │    Product after 50 layers: 0.25⁵⁰ ≈ 10⁻³⁰  → GONE!       │
    │                                                              │
    │  If each factor ≈ 2.0 (large weights):                      │
    │    Product after 10 layers: 2.0¹⁰ ≈ 10³     → GROWS!       │
    │    Product after 20 layers: 2.0²⁰ ≈ 10⁶     → EXPLODES!    │
    │    Product after 50 layers: 2.0⁵⁰ ≈ 10¹⁵    → INF!         │
    │                                                              │
    │  Ideal: each factor ≈ 1.0 → Product stays O(1) ✓            │
    └──────────────────────────────────────────────────────────────┘
```

---

## 2. Vanishing Gradients

### 2.1 Why Sigmoid and Tanh Cause Vanishing

```
    Sigmoid derivative: σ'(z) = σ(z)(1-σ(z))
    
    Maximum value: σ'(0) = 0.25    ← Always ≤ 0.25!
    
    σ'(z)
    │
0.25┤ · · · · · ·▲· · · · · ·
    │          ╱    ╲
    │        ╱        ╲
    │      ╱            ╲
  0 ┤━━━━╱· · · · · · · ╲━━━━
    ┼───────────────────────── z
                0

    Every time gradient passes through sigmoid, it's multiplied by ≤ 0.25
    After L layers: gradient ≤ 0.25^L

    L=4:   0.25⁴ = 0.0039     (gradient is 0.4% of original)
    L=10:  0.25¹⁰ = 9.5×10⁻⁷  (one millionth!)
    L=20:  0.25²⁰ = 9.1×10⁻¹³ (essentially zero)
```

```
    Tanh derivative: tanh'(z) = 1 - tanh²(z)
    
    Maximum value: tanh'(0) = 1    ← Can be 1 at z=0!
    But in practice, most activations are NOT at z=0
    
    For |z| > 2: tanh'(z) < 0.07   ← Still vanishes!
    
    Better than sigmoid, but still problematic for deep nets.
```

### 2.2 Gradient Magnitude Across Layers

```
    10-Layer Network with Sigmoid

    Layer:    10    9    8    7    6    5    4    3    2    1
    |grad|:   1   0.25 0.06 0.02 0.004 10⁻³ 10⁻⁴ 10⁻⁴ 10⁻⁵ 10⁻⁶
    
    ████████ ██████ ████ ███ ██ █ · · · ·
    Layer 10 Layer 9                    Layer 1
    (output)                            (input)
    
    Early layers learn NOTHING — their gradients are ~0!
    Later layers learn fine — they get strong gradients.
    
    Result: A "deep" network effectively acts as a "shallow" one!
```

### 2.3 Symptoms

```
    How to detect vanishing gradients:
    
    ✗ Loss decreases very slowly or plateaus early
    ✗ Early layers' weights barely change during training
    ✗ Gradient norms: ||∂L/∂W¹|| << ||∂L/∂W^L||
    ✗ Adding more layers makes the network WORSE, not better
    ✗ Weights in early layers stay near their initialization
```

---

## 3. Exploding Gradients

### 3.1 When Weights Are Too Large

```
    If weight matrices have spectral norm > 1:
    
    ∂L/∂W¹ involves products like: ... · W^[L] · W^[L-1] · ... · W^[2]

    If ||W^[l]|| > 1 for most layers:
    The product grows exponentially with depth!

    Example: Simple scalar case, w = 1.5 at each layer

    Layer L:     gradient = 1
    Layer L-1:   gradient = 1.5
    Layer L-2:   gradient = 2.25
    Layer L-3:   gradient = 3.375
    ...
    Layer 1:     gradient = 1.5^(L-1)

    For L = 50: 1.5⁴⁹ ≈ 6.4 × 10⁸  → EXPLODES!
```

### 3.2 Symptoms

```
    How to detect exploding gradients:
    
    ✗ Loss becomes NaN or Inf during training
    ✗ Loss oscillates wildly or increases suddenly
    ✗ Gradient norms are very large (>1000)
    ✗ Weights grow to very large values
    ✗ Training is numerically unstable
```

---

## 4. Solutions

### 4.1 ReLU Activation (Solves Vanishing for z > 0)

```
    ReLU'(z) = { 1  if z > 0      ← Perfect! No shrinking!
               { 0  if z ≤ 0

    For positive activations, the gradient passes through UNCHANGED.
    
    Sigmoid chain: 0.25 × 0.25 × 0.25 × ... = 0.25^n → 0
    ReLU chain:    1.0  × 1.0  × 1.0  × ... = 1.0^n  → 1  ✓
    
    BUT: For negative activations, gradient = 0 → "dying ReLU"
    Solution: Leaky ReLU, ELU, GELU
```

### 4.2 Proper Weight Initialization

```
    The goal: Keep the variance of activations and gradients stable
    across layers.

    ┌────────────────────────────────────────────────────────────┐
    │  Xavier/Glorot Init (for sigmoid/tanh):                    │
    │  W ~ N(0, 2/(n_in + n_out))                               │
    │  or W ~ U(-√(6/(n_in+n_out)), +√(6/(n_in+n_out)))        │
    │                                                            │
    │  He/Kaiming Init (for ReLU):                               │
    │  W ~ N(0, 2/n_in)                                         │
    │  or W ~ U(-√(6/n_in), +√(6/n_in))                        │
    │                                                            │
    │  Why these specific values?                                │
    │  They ensure Var(a^[l]) ≈ Var(a^[l-1]) for all layers     │
    │  → activations neither grow nor shrink across layers       │
    └────────────────────────────────────────────────────────────┘

    Effect of initialization:
    
    Too small (σ = 0.001):          Correct (He init):          Too large (σ = 1.0):
    Layer 1: σ_act = 0.5            Layer 1: σ_act = 0.8       Layer 1: σ_act = 50
    Layer 2: σ_act = 0.01           Layer 2: σ_act = 0.7       Layer 2: σ_act = 2500
    Layer 3: σ_act = 0.0001         Layer 3: σ_act = 0.8       Layer 3: σ_act = Inf
    → Vanishes!                     → Stable!                  → Explodes!
```

### 4.3 Batch Normalization

```
    Normalize activations within each mini-batch:

    μ_B = (1/m) Σ zᵢ              (batch mean)
    σ²_B = (1/m) Σ (zᵢ - μ_B)²   (batch variance)
    ẑᵢ = (zᵢ - μ_B) / √(σ²_B + ε)  (normalize)
    yᵢ = γ · ẑᵢ + β               (scale and shift — learnable!)

    Effect:
    • Forces activations to have mean ≈ 0, variance ≈ 1
    • Prevents internal covariate shift
    • Acts as regularization (noise from batch statistics)
    • Allows higher learning rates
    • Dramatically reduces vanishing/exploding gradients
```

### 4.4 Skip/Residual Connections (ResNet)

```
    Standard layer:                 Residual connection:
    a^[l] = g(W·a^[l-1]+b)         a^[l] = g(W·a^[l-1]+b) + a^[l-1]
                                              ↑                    ↑
                                           learned              identity
                                           
    Gradient through residual block:
    ∂a^[l]/∂a^[l-1] = ∂g(Wz+b)/∂a^[l-1] + I
                                            ↑
                                     ALWAYS 1! (identity)

    Even if the learned gradient vanishes, the identity term preserves it!
    This is why ResNet can train 152+ layers!

    ┌───┐                     ┌───┐
    │ x ├──────┬──────────────► + ├──► a^[l]
    └───┘      │              └─┬─┘
               │   ┌─────┐     │
               └──►│W,b,g├─────┘
                   └─────┘
               "Skip connection"
```

### 4.5 Gradient Clipping

```
    If ||∇L|| > threshold:
        ∇L ← threshold × (∇L / ||∇L||)
    
    This caps the gradient norm, preventing explosion:
    
    Before clipping:          After clipping (threshold=5):
    
    gradient = [100, -200]    gradient = [100, -200]
    ||gradient|| = 223.6      ||gradient|| = 223.6 > 5
                               gradient ← 5 × [100, -200] / 223.6
                               gradient = [2.24, -4.47]
                               ||gradient|| = 5.0 ✓

    PyTorch:
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=5.0)
```

---

## 5. Solutions Comparison

| Solution | Addresses | Mechanism | When to Use |
|---|---|---|---|
| **ReLU** | Vanishing | Gradient = 1 for z > 0 | Default hidden activation |
| **He Init** | Both | Proper weight variance | Always with ReLU |
| **Xavier Init** | Both | Proper weight variance | Always with sigmoid/tanh |
| **Batch Norm** | Both | Normalize activations | Most architectures |
| **Skip Connections** | Vanishing | Identity shortcut | Deep networks (10+ layers) |
| **Gradient Clipping** | Exploding | Cap gradient norm | RNNs, Transformers |
| **LSTM/GRU** | Vanishing (RNN) | Gating mechanism | Sequence models |
| **Careful LR** | Exploding | Smaller step size | Always (tuning) |

---

## 6. Python Demonstration

```python
import numpy as np

def demonstrate_vanishing_gradient():
    """Show how gradients vanish with sigmoid vs survive with ReLU."""
    
    np.random.seed(42)
    depths = [5, 10, 20, 50]
    
    print("=" * 70)
    print("VANISHING GRADIENT DEMONSTRATION")
    print("=" * 70)
    print(f"\n{'Depth':>6} │ {'Sigmoid Gradient':>20} │ {'ReLU Gradient':>20}")
    print("─" * 55)
    
    for L in depths:
        # Sigmoid: product of derivatives (max 0.25 each)
        sigmoid_grad = 0.25 ** L  # Worst case
        
        # ReLU: product of derivatives (1 if active, assume 50% active)
        relu_grad = 0.5 ** L  # Average case (50% of neurons active)
        relu_grad_best = 1.0 ** L  # Best case (all active)
        
        print(f"{L:>6} │ {sigmoid_grad:>20.2e} │ {relu_grad_best:>20.2e} (best)")
    
    print("\nWith actual weight matrices:")
    print(f"\n{'Depth':>6} │ {'Sigmoid |∂L/∂W¹|':>20} │ {'ReLU |∂L/∂W¹|':>20}")
    print("─" * 55)
    
    for L in depths:
        n = 64  # neurons per layer
        
        # Simulate gradient flow with sigmoid
        grad_sigmoid = 1.0
        for _ in range(L):
            W = np.random.randn(n, n) * np.sqrt(1.0 / n)  # Xavier
            sigmoid_deriv = 0.2  # Average sigmoid derivative
            grad_sigmoid *= np.linalg.norm(W) * sigmoid_deriv
        
        # Simulate gradient flow with ReLU
        grad_relu = 1.0
        for _ in range(L):
            W = np.random.randn(n, n) * np.sqrt(2.0 / n)  # He
            relu_active_frac = 0.5
            grad_relu *= np.linalg.norm(W) * relu_active_frac
        
        print(f"{L:>6} │ {grad_sigmoid:>20.2e} │ {grad_relu:>20.2e}")


def demonstrate_exploding_gradient():
    """Show how large weights cause gradient explosion."""
    
    print(f"\n{'=' * 70}")
    print("EXPLODING GRADIENT DEMONSTRATION")
    print("=" * 70)
    
    np.random.seed(42)
    n = 10
    
    print(f"\n{'Init Scale':>12} │ {'L=5 |grad|':>15} │ {'L=10 |grad|':>15} │ {'L=20 |grad|':>15}")
    print("─" * 70)
    
    for scale in [0.01, 0.1, 0.5, 1.0, 1.5, 2.0]:
        results = []
        for L in [5, 10, 20]:
            grad = np.eye(n)
            for _ in range(L):
                W = np.random.randn(n, n) * scale
                grad = W @ grad
            results.append(np.linalg.norm(grad))
        
        status = ""
        if results[-1] > 1e10:
            status = "💥 EXPLODES"
        elif results[-1] < 1e-10:
            status = "💀 VANISHES"
        else:
            status = "✅ STABLE"
        
        print(f"{scale:>12.2f} │ {results[0]:>15.2e} │ {results[1]:>15.2e} │ "
              f"{results[2]:>15.2e} │ {status}")


demonstrate_vanishing_gradient()
demonstrate_exploding_gradient()
```

---

## 7. Practical Diagnostic Code

```python
import torch
import torch.nn as nn

def monitor_gradients(model, loss):
    """Monitor gradient health during training."""
    loss.backward()
    
    print(f"\n{'Layer':>20} │ {'|grad| mean':>12} │ {'|grad| max':>12} │ {'Status':>10}")
    print("─" * 65)
    
    for name, param in model.named_parameters():
        if param.grad is not None:
            grad_mean = param.grad.abs().mean().item()
            grad_max = param.grad.abs().max().item()
            
            if grad_mean < 1e-7:
                status = "💀 VANISH"
            elif grad_max > 1000:
                status = "💥 EXPLODE"
            else:
                status = "✅ OK"
            
            print(f"{name:>20} │ {grad_mean:>12.2e} │ {grad_max:>12.2e} │ {status}")

# Example: Deep sigmoid network (will show vanishing!)
model = nn.Sequential(*[
    layer for _ in range(10) 
    for layer in [nn.Linear(64, 64), nn.Sigmoid()]
] + [nn.Linear(64, 10)])

x = torch.randn(32, 64)
y = torch.randint(0, 10, (32,))
logits = model(x)
loss = nn.CrossEntropyLoss()(logits, y)
monitor_gradients(model, loss)
```

---

## 📊 Summary Table

| Problem | Cause | Effect | Solution |
|---|---|---|---|
| **Vanishing** | σ'(z) ≤ 0.25 (sigmoid); product shrinks | Early layers don't learn | ReLU, skip connections, batch norm |
| **Exploding** | \|\|W\|\| > 1; product grows | NaN loss, unstable training | Gradient clipping, proper init |
| **Dying ReLU** | z < 0 for all inputs; grad = 0 permanently | Neurons stop learning | Leaky ReLU, ELU, careful init |
| **Internal covariate shift** | Distribution of activations shifts per layer | Slows convergence | Batch normalization |

---

## ❓ Revision Questions

1. **Prove mathematically that using sigmoid activations in a 20-layer network causes the gradient at layer 1 to be at most 0.25²⁰ ≈ 10⁻¹² times the gradient at layer 20.**

2. **Why does ReLU largely solve the vanishing gradient problem? Under what conditions can ReLU still fail (dying ReLU)?**

3. **Explain the He initialization formula W ~ N(0, 2/n_in). Derive why the factor 2/n_in keeps activation variance stable across layers with ReLU.**

4. **How do skip connections solve the vanishing gradient problem? Write the gradient flow equation for a residual block and show that the identity term guarantees gradient ≥ 1.**

5. **You're training a 50-layer network and observe that the loss becomes NaN after 100 iterations. What is likely happening, and what three steps would you take to fix it?**

6. **Compare batch normalization and skip connections as solutions to gradient problems. Can they be used together? Why does ResNet use both?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Gradient Computation](04-gradient-computation.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Gradient Checking →](06-gradient-checking.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
