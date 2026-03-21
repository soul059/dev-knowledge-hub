# ⚡ Activation Functions

> **Chapter 1.3 — The Non-Linear Heart of Neural Networks**

| Topic | Details |
|---|---|
| **Subject** | Neural Networks — Activation Functions |
| **Prerequisites** | Calculus (derivatives), Chapter 1.2 |
| **Key Insight** | Without non-linear activation functions, any depth of network collapses to a single linear transformation |

---

## 📌 Overview

Activation functions are the gatekeepers of neural networks. They introduce **non-linearity** into the model, enabling it to learn complex patterns that go far beyond simple linear relationships. Choosing the right activation function can mean the difference between a network that trains beautifully and one that fails completely. This chapter provides a comprehensive, detailed treatment of every major activation function used in modern deep learning.

---

## 1. Why Do We Need Activation Functions?

### 1.1 The Linearity Problem

Without activation functions, a neural network is just a composition of linear transformations:

```
Layer 1:  z₁ = W₁x + b₁
Layer 2:  z₂ = W₂z₁ + b₂ = W₂(W₁x + b₁) + b₂ = (W₂W₁)x + (W₂b₁ + b₂)
                                                   = W'x + b'

No matter how many layers, the result is still a single linear transformation!
```

**Non-linear activations break this collapse**:
```
Layer 1:  a₁ = f(W₁x + b₁)        ← non-linear!
Layer 2:  a₂ = f(W₂a₁ + b₂)       ← non-linear again!

Now: a₂ = f(W₂ · f(W₁x + b₁) + b₂)  — this is NOT reducible to a linear function!
```

---

## 2. Activation Functions — Comprehensive Guide

---

### 2.1 Sigmoid (Logistic) Function

**Formula:**
```
                   1
    σ(z) = ─────────────
            1 + e⁻ᶻ

    Range: (0, 1)
    Centered at: (0, 0.5)
```

**Derivative:**
```
    σ'(z) = σ(z) · (1 - σ(z))

    Maximum derivative: σ'(0) = 0.25
    At extremes: σ'(z) → 0 as |z| → ∞
```

**ASCII Plot:**
```
    σ(z)
    │
  1 ┤ · · · · · · · · · · ·━━━━━━━━━
    │                   ╱·
    │                 ╱
    │               ╱
0.5 ┤ · · · · · ·●· · · · · · · · ·
    │           ╱
    │         ╱
    │       ╱·
  0 ┤━━━━━━· · · · · · · · · · · ·
    ┼───────────────●───────────── z
                    0

    S-shaped curve, squashes all values to (0, 1)
```

**Derivative Plot:**
```
    σ'(z)
    │
0.25┤ · · · · · · ·▲· · · · · · ·
    │            ╱   ╲
    │          ╱       ╲
    │        ╱           ╲
    │      ╱               ╲
  0 ┤━━━━╱· · · · · · · · · ╲━━━━
    ┼───────────────────────────── z
                    0
    Max at z=0, quickly vanishes → VANISHING GRADIENT!
```

**Pros:**
- Output in (0, 1) — interpretable as probability
- Smooth and differentiable everywhere
- Historically the most popular activation

**Cons:**
- ❌ **Vanishing gradient**: σ'(z) ≤ 0.25, multiplied across layers → gradients shrink exponentially
- ❌ **Not zero-centered**: Outputs always positive → zig-zag gradient updates
- ❌ **Computationally expensive**: Requires exp() calculation
- ❌ **Saturation**: For |z| > 5, gradient ≈ 0 → neuron "dies" during training

**When to use:** Output layer for binary classification (probability output). Rarely used in hidden layers of modern networks.

---

### 2.2 Tanh (Hyperbolic Tangent)

**Formula:**
```
                eᶻ - e⁻ᶻ
    tanh(z) = ──────────── = 2σ(2z) - 1
                eᶻ + e⁻ᶻ

    Range: (-1, 1)
    Centered at: (0, 0) — ZERO-CENTERED!
```

**Derivative:**
```
    tanh'(z) = 1 - tanh²(z)

    Maximum derivative: tanh'(0) = 1
    At extremes: tanh'(z) → 0 as |z| → ∞
```

**ASCII Plot:**
```
    tanh(z)
    │
  1 ┤ · · · · · · · · · · ·━━━━━━━
    │                   ╱·
    │                ╱·
    │             ╱·
  0 ┤ · · · · ●· · · · · · · · · ·
    │      ·╱
    │    ·╱
    │  ·╱
 -1 ┤━━· · · · · · · · · · · · · ·
    ┼───────────────●───────────── z
                    0

    Like sigmoid but zero-centered, range (-1, 1)
```

**Pros:**
- ✅ **Zero-centered**: Outputs can be negative → better gradient dynamics
- ✅ Stronger gradients than sigmoid (max derivative = 1.0 vs 0.25)
- ✅ Smooth and differentiable

**Cons:**
- ❌ **Still suffers from vanishing gradients** (tanh'(z) → 0 for large |z|)
- ❌ Still saturates at extremes
- ❌ Computationally expensive (exp calculations)

**When to use:** Hidden layers of RNNs, LSTMs (gate mechanisms). Generally preferred over sigmoid for hidden layers, but ReLU is usually better.

---

### 2.3 ReLU (Rectified Linear Unit)

**Formula:**
```
    ReLU(z) = max(0, z) = { z,  if z > 0
                          { 0,  if z ≤ 0

    Range: [0, ∞)
```

**Derivative:**
```
    ReLU'(z) = { 1,  if z > 0
               { 0,  if z < 0
               { undefined at z = 0 (use subgradient: 0 or 1)
```

**ASCII Plot:**
```
    ReLU(z)
    │                          ╱
    │                        ╱
    │                      ╱
    │                    ╱
    │                  ╱
  0 ┤━━━━━━━━━━━━━━━━●
    │               ╱│
    ┼─────────────●──┼──────────── z
                  0

    Dead simple: pass positive, kill negative
```

**Derivative Plot:**
```
    ReLU'(z)
    │
  1 ┤ · · · · · · · ·━━━━━━━━━━━━
    │               ┃
    │               ┃
  0 ┤━━━━━━━━━━━━━━━┃ · · · · · ·
    ┼───────────────●───────────── z
                    0

    Binary: either full gradient or zero gradient
```

**Pros:**
- ✅ **No vanishing gradient** for positive values (derivative = 1)
- ✅ **Computationally extremely fast** — just max(0, z), no exp()
- ✅ **Sparse activation** — many neurons output 0 → efficient representation
- ✅ **Converges faster** than sigmoid/tanh (6x faster in practice - Krizhevsky 2012)
- ✅ Biologically plausible — neurons either fire or don't

**Cons:**
- ❌ **Dying ReLU problem**: If z < 0 for all inputs → gradient = 0 → neuron permanently dead
- ❌ **Not zero-centered**: Outputs always ≥ 0
- ❌ **Not bounded**: Can produce very large activations
- ❌ Not differentiable at z = 0 (minor issue in practice)

**When to use:** **Default choice for hidden layers** in feedforward and convolutional networks. The workhorse of modern deep learning.

---

### 2.4 Leaky ReLU

**Formula:**
```
    LeakyReLU(z) = max(αz, z) = { z,   if z > 0
                                 { αz,  if z ≤ 0

    Where α is a small constant (typically α = 0.01)
    Range: (-∞, ∞)
```

**Derivative:**
```
    LeakyReLU'(z) = { 1,  if z > 0
                    { α,  if z ≤ 0
```

**ASCII Plot:**
```
    LeakyReLU(z)
    │                          ╱
    │                        ╱
    │                      ╱
    │                    ╱
    │                  ╱
  0 ┤ · · · · · · · ●
    │            ·╱  │
    │          ·╱    │      (slight negative slope = α)
    ┼─────────●──────┼──────────── z
              │      0
    slope = α │  slope = 1
```

**Pros:**
- ✅ **Solves dying ReLU** — small gradient (α) for negative inputs keeps neurons alive
- ✅ All benefits of ReLU (fast computation, no vanishing gradient for z > 0)
- ✅ Zero-centered on average

**Cons:**
- ❌ Extra hyperparameter α to tune
- ❌ Results inconsistent — sometimes no better than ReLU

**Variant — Parametric ReLU (PReLU):**
```
    PReLU(z) = max(αz, z)    where α is LEARNED during training
```

**When to use:** When you observe dying neurons with ReLU. Good default alternative.

---

### 2.5 ELU (Exponential Linear Unit)

**Formula:**
```
    ELU(z) = { z,              if z > 0
             { α(eᶻ - 1),     if z ≤ 0

    Where α > 0 (typically α = 1.0)
    Range: (-α, ∞)
```

**Derivative:**
```
    ELU'(z) = { 1,              if z > 0
              { α · eᶻ = ELU(z) + α,  if z ≤ 0
```

**ASCII Plot:**
```
    ELU(z)
    │                          ╱
    │                        ╱
    │                      ╱
    │                    ╱
    │                  ╱
  0 ┤ · · · · · · · ●
    │           ╱····│
    │       ╱········│
 -α ┤- - ━━━━━━━- - -│- - - - - - -
    ┼─────────────────┼──────────── z
                      0
    Smooth curve approaching -α for very negative z
```

**Pros:**
- ✅ **Smooth everywhere** (unlike ReLU/Leaky ReLU)
- ✅ Negative values → **zero-centered activations** → faster convergence
- ✅ Saturates for large negative values → **noise robustness**
- ✅ Better mean activations closer to zero → acts like batch normalization

**Cons:**
- ❌ **Slower computation** due to exp() for z ≤ 0
- ❌ Not as widely adopted as ReLU (less framework optimization)

**When to use:** When you need zero-centered activations and smooth gradients. Good for architectures without batch normalization.

---

### 2.6 GELU (Gaussian Error Linear Unit)

**Formula:**
```
    GELU(z) = z · Φ(z)

    Where Φ(z) is the CDF of the standard normal distribution:
    Φ(z) = ½[1 + erf(z/√2)]

    Approximation:
    GELU(z) ≈ 0.5z(1 + tanh[√(2/π)(z + 0.044715z³)])
```

**Derivative:**
```
    GELU'(z) = Φ(z) + z · φ(z)

    Where φ(z) is the standard normal PDF: φ(z) = (1/√(2π)) · e^(-z²/2)
```

**ASCII Plot:**
```
    GELU(z)
    │                          ╱
    │                        ╱
    │                      ╱
    │                    ╱
    │                 ╱·
  0 ┤ · · · · · · ●·
    │           ╱·   │
    │      ─── ·     │
-0.17┤- - - -╱- - - - │- - - - - - -
    │    ╱           │
    ┼─────────────────┼──────────── z
                      0
    Smooth, slight dip below zero, then linear for large z
```

**Pros:**
- ✅ **Smooth non-monotonic** — can output small negative values (provides regularization)
- ✅ **State-of-the-art results** in Transformers (BERT, GPT)
- ✅ Stochastic interpretation: randomly zeroing inputs weighted by magnitude
- ✅ Combines properties of ReLU and dropout

**Cons:**
- ❌ More computationally expensive than ReLU
- ❌ More complex to implement (CDF computation)

**When to use:** **Transformers, NLP models**. Default in BERT, GPT, and most modern language models.

---

### 2.7 Swish / SiLU (Sigmoid Linear Unit)

**Formula:**
```
    Swish(z) = z · σ(z) = z · ───────
                                1 + e⁻ᶻ

    Or with learnable β:
    Swish_β(z) = z · σ(βz)

    Range: [≈-0.278, ∞)
```

**Derivative:**
```
    Swish'(z) = σ(z) + z · σ(z) · (1 - σ(z))
              = σ(z) + z · σ'(z)
              = Swish(z) + σ(z) · (1 - Swish(z))
```

**ASCII Plot:**
```
    Swish(z)
    │                          ╱
    │                        ╱
    │                      ╱
    │                    ╱
    │                 ╱·
  0 ┤ · · · · · · ·●
    │          ╱··   │
    │     ─── ·      │     Very similar to GELU!
-0.28┤- - ╱- - - - - │- - - - - - -
    │  ╱             │
    ┼─────────────────┼──────────── z
                      0
```

**Pros:**
- ✅ **Self-gated**: Uses its own value to gate output (no extra parameters)
- ✅ Smooth, non-monotonic — similar benefits to GELU
- ✅ Discovered by **neural architecture search** (Google Brain, 2017)
- ✅ Outperforms ReLU on deeper models (40+ layers)

**Cons:**
- ❌ More expensive than ReLU (requires sigmoid computation)
- ❌ Similar performance to GELU — often interchangeable

**When to use:** Deep networks (EfficientNet, MobileNetV3), computer vision. Increasingly used as ReLU alternative.

---

### 2.8 Softmax (for Output Layer)

**Formula:**
```
    For output layer with K classes:

                    eᶻᵢ
    Softmax(z)ᵢ = ─────────     for i = 1, 2, ..., K
                   Σⱼ eᶻⱼ

    Properties:
    • All outputs in (0, 1)
    • Sum of all outputs = 1  →  valid probability distribution
    • Amplifies the largest input (winner-take-most)
```

**Example:**
```
    Input logits:    z = [2.0, 1.0, 0.1]

    Exponents:       e² = 7.389,  e¹ = 2.718,  e⁰·¹ = 1.105
    Sum:             7.389 + 2.718 + 1.105 = 11.212

    Softmax outputs: [7.389/11.212, 2.718/11.212, 1.105/11.212]
                   = [0.659, 0.242, 0.099]

    Sum = 0.659 + 0.242 + 0.099 = 1.000 ✓

    Interpretation: 65.9% class 1, 24.2% class 2, 9.9% class 3
```

**Numerical Stability:**
```
    Problem: e^z can overflow for large z (e.g., e^1000 = Inf)
    
    Solution: Subtract max before exponentiating:
    Softmax(z)ᵢ = e^(zᵢ - max(z)) / Σⱼ e^(zⱼ - max(z))
    
    This is mathematically equivalent but numerically stable.
```

**When to use:** **Always** for multi-class classification output layers where classes are mutually exclusive.

---

## 3. The Vanishing Gradient Problem — Explained

```
Deep Network with Sigmoid Activations:

Layer 1 → σ'(z₁) → Layer 2 → σ'(z₂) → ... → Layer n → σ'(zₙ) → Loss

Gradient at Layer 1 = ∂L/∂W₁ = ∂L/∂aₙ · σ'(zₙ) · ... · σ'(z₂) · σ'(z₁) · x

Since σ'(z) ≤ 0.25 for ALL z:

    |∂L/∂W₁| ≤ |∂L/∂aₙ| · (0.25)ⁿ

For n = 10 layers:  (0.25)¹⁰ ≈ 0.000001  →  gradient ≈ 0!
For n = 20 layers:  (0.25)²⁰ ≈ 10⁻¹²     →  gradient vanishes completely!
```

```
    Gradient Magnitude vs. Layer Depth (Sigmoid)
    
    |gradient|
    │
    │ █
    │ █ █
    │ █ █ █
    │ █ █ █ ░
    │ █ █ █ ░ ░
    │ █ █ █ ░ ░ ·
    │ █ █ █ ░ ░ · · · · ·
    ┼──────────────────────── Layer
     10  8  6  4  2  1
     (output)     (input)

    Earlier layers get exponentially smaller gradients
    → They barely learn at all!
```

**ReLU's solution:**
```
    ReLU'(z) = 1 for z > 0

    Gradient at Layer 1 = ∂L/∂aₙ · 1 · 1 · ... · 1 · 1 · x
                        = ∂L/∂aₙ · x    (if all activations positive)

    No multiplication by fractions → gradient flows freely!
```

---

## 4. Complete Comparison Table

| Activation | Formula | Range | Derivative | Zero-Centered | Vanishing Gradient | Computation | Best Use Case |
|---|---|---|---|---|---|---|---|
| **Sigmoid** | 1/(1+e⁻ᶻ) | (0,1) | σ(1-σ), max 0.25 | ❌ No | ❌ Yes | Slow (exp) | Binary output layer |
| **Tanh** | (eᶻ-e⁻ᶻ)/(eᶻ+e⁻ᶻ) | (-1,1) | 1-tanh², max 1.0 | ✅ Yes | ❌ Yes | Slow (exp) | RNN/LSTM hidden layers |
| **ReLU** | max(0,z) | [0,∞) | 0 or 1 | ❌ No | ✅ No (for z>0) | ⚡ Fast | **Default hidden layers** |
| **Leaky ReLU** | max(αz,z) | (-∞,∞) | α or 1 | ~Yes | ✅ No | ⚡ Fast | When ReLU neurons die |
| **ELU** | z or α(eᶻ-1) | (-α,∞) | 1 or ELU+α | ✅ Yes | ✅ No | Medium | Without batch norm |
| **GELU** | z·Φ(z) | [≈-0.17,∞) | Φ(z)+z·φ(z) | ~Yes | ✅ No | Medium | **Transformers, NLP** |
| **Swish/SiLU** | z·σ(z) | [≈-0.28,∞) | σ(z)+z·σ'(z) | ~Yes | ✅ No | Medium | Deep CNNs (40+ layers) |
| **Softmax** | eᶻⁱ/Σeᶻʲ | (0,1) | Jacobian matrix | N/A | N/A | Slow | **Multi-class output** |

---

## 5. Python Implementation — All Activation Functions

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.special import erf

class ActivationFunctions:
    """Complete collection of activation functions with derivatives."""
    
    # ──── Sigmoid ────
    @staticmethod
    def sigmoid(z):
        return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
    
    @staticmethod
    def sigmoid_derivative(z):
        s = ActivationFunctions.sigmoid(z)
        return s * (1 - s)
    
    # ──── Tanh ────
    @staticmethod
    def tanh(z):
        return np.tanh(z)
    
    @staticmethod
    def tanh_derivative(z):
        return 1 - np.tanh(z) ** 2
    
    # ──── ReLU ────
    @staticmethod
    def relu(z):
        return np.maximum(0, z)
    
    @staticmethod
    def relu_derivative(z):
        return (z > 0).astype(float)
    
    # ──── Leaky ReLU ────
    @staticmethod
    def leaky_relu(z, alpha=0.01):
        return np.where(z > 0, z, alpha * z)
    
    @staticmethod
    def leaky_relu_derivative(z, alpha=0.01):
        return np.where(z > 0, 1, alpha)
    
    # ──── ELU ────
    @staticmethod
    def elu(z, alpha=1.0):
        return np.where(z > 0, z, alpha * (np.exp(z) - 1))
    
    @staticmethod
    def elu_derivative(z, alpha=1.0):
        return np.where(z > 0, 1, alpha * np.exp(z))
    
    # ──── GELU ────
    @staticmethod
    def gelu(z):
        return 0.5 * z * (1 + erf(z / np.sqrt(2)))
    
    @staticmethod
    def gelu_approx(z):
        """Tanh approximation used in most frameworks"""
        return 0.5 * z * (1 + np.tanh(np.sqrt(2/np.pi) * (z + 0.044715 * z**3)))
    
    @staticmethod
    def gelu_derivative(z):
        phi = 0.5 * (1 + erf(z / np.sqrt(2)))
        pdf = np.exp(-z**2 / 2) / np.sqrt(2 * np.pi)
        return phi + z * pdf
    
    # ──── Swish / SiLU ────
    @staticmethod
    def swish(z):
        sig = ActivationFunctions.sigmoid(z)
        return z * sig
    
    @staticmethod
    def swish_derivative(z):
        sig = ActivationFunctions.sigmoid(z)
        return sig + z * sig * (1 - sig)
    
    # ──── Softmax ────
    @staticmethod
    def softmax(z):
        z_stable = z - np.max(z, axis=-1, keepdims=True)
        exp_z = np.exp(z_stable)
        return exp_z / np.sum(exp_z, axis=-1, keepdims=True)


# ──── Demo: Compute and Compare ────
af = ActivationFunctions()
z = np.linspace(-5, 5, 1000)

print("=" * 60)
print("Activation Function Values at Key Points")
print("=" * 60)
print(f"{'z':>6} | {'Sigmoid':>8} | {'Tanh':>8} | {'ReLU':>8} | {'GELU':>8} | {'Swish':>8}")
print("-" * 60)
for z_val in [-3, -1, 0, 1, 3]:
    z_arr = np.array([z_val], dtype=float)
    print(f"{z_val:>6.1f} | {af.sigmoid(z_arr)[0]:>8.4f} | {af.tanh(z_arr)[0]:>8.4f} | "
          f"{af.relu(z_arr)[0]:>8.4f} | {af.gelu(z_arr)[0]:>8.4f} | {af.swish(z_arr)[0]:>8.4f}")

print(f"\n{'Derivatives at Key Points':>40}")
print("-" * 60)
print(f"{'z':>6} | {'Sigmoid':>8} | {'Tanh':>8} | {'ReLU':>8} | {'GELU':>8} | {'Swish':>8}")
print("-" * 60)
for z_val in [-3, -1, 0, 1, 3]:
    z_arr = np.array([z_val], dtype=float)
    print(f"{z_val:>6.1f} | {af.sigmoid_derivative(z_arr)[0]:>8.4f} | "
          f"{af.tanh_derivative(z_arr)[0]:>8.4f} | {af.relu_derivative(z_arr)[0]:>8.4f} | "
          f"{af.gelu_derivative(z_arr)[0]:>8.4f} | {af.swish_derivative(z_arr)[0]:>8.4f}")

# Softmax example
print(f"\nSoftmax Example:")
logits = np.array([2.0, 1.0, 0.1])
probs = af.softmax(logits)
print(f"  Input logits:     {logits}")
print(f"  Softmax output:   {probs}")
print(f"  Sum of outputs:   {probs.sum():.6f}")
```

**Expected Output:**
```
============================================================
Activation Function Values at Key Points
============================================================
     z |  Sigmoid |     Tanh |     ReLU |     GELU |    Swish
------------------------------------------------------------
  -3.0 |   0.0474 |  -0.9951 |   0.0000 |  -0.0040 |  -0.1423
  -1.0 |   0.2689 |  -0.7616 |   0.0000 |  -0.1587 |  -0.2689
   0.0 |   0.5000 |   0.0000 |   0.0000 |   0.0000 |   0.0000
   1.0 |   0.7311 |   0.7616 |   1.0000 |   0.8413 |   0.7311
   3.0 |   0.9526 |   0.9951 |   3.0000 |   2.9960 |   2.8577

                      Derivatives at Key Points
------------------------------------------------------------
     z |  Sigmoid |     Tanh |     ReLU |     GELU |    Swish
------------------------------------------------------------
  -3.0 |   0.0452 |   0.0099 |   0.0000 |  -0.0119 |  -0.0883
  -1.0 |   0.1966 |   0.4200 |   0.0000 |  -0.0832 |   0.0724
   0.0 |   0.2500 |   1.0000 |   0.0000 |   0.5000 |   0.5000
   1.0 |   0.1966 |   0.4200 |   1.0000 |   1.0832 |   0.9276
   3.0 |   0.0452 |   0.0099 |   1.0000 |   1.0119 |   1.0883
```

---

## 6. PyTorch Activation Functions

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# All activations available as modules (for nn.Sequential)
model_with_activations = nn.Sequential(
    nn.Linear(784, 256),
    nn.ReLU(),           # Most common default
    nn.Linear(256, 128),
    nn.GELU(),           # Popular in Transformers
    nn.Linear(128, 10),
    nn.Softmax(dim=1)    # Multi-class output
)

# Or as functions (for use in forward())
z = torch.randn(32, 128)

# Functional API
a1 = F.relu(z)
a2 = F.leaky_relu(z, negative_slope=0.01)
a3 = F.elu(z, alpha=1.0)
a4 = F.gelu(z)
a5 = F.silu(z)           # Swish = SiLU in PyTorch
a6 = torch.sigmoid(z)
a7 = torch.tanh(z)

# Softmax on logits
logits = torch.randn(32, 10)
probs = F.softmax(logits, dim=1)  # dim=1 for class dimension
print(f"Probs sum per sample: {probs.sum(dim=1)[:3]}")  # Should be [1, 1, 1, ...]

# PReLU — learnable negative slope
prelu = nn.PReLU(num_parameters=128)  # One α per channel
a_prelu = prelu(z)
print(f"PReLU learnable α: {prelu.weight[:5]}")
```

---

## 7. Choosing the Right Activation Function — Decision Guide

```
                    ACTIVATION FUNCTION DECISION TREE
    ┌───────────────────────────────────────────────────────┐
    │                  What layer type?                      │
    └──────────────────────┬────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        Hidden Layer              Output Layer
              │                         │
              │                    ┌────┴────┐
              │                    ▼         ▼
              │              Regression  Classification
              │                 │              │
              │                 ▼         ┌────┴────┐
              │              Linear       ▼         ▼
              │             (no act.)   Binary   Multi-class
              │                          │         │
              │                       Sigmoid   Softmax
              │
         ┌────┴────────────────┐
         ▼                     ▼
    Standard CNN/MLP      Transformer/NLP
         │                     │
         ▼                     ▼
       ReLU                  GELU
    (or Leaky ReLU        (default for
     if dying neurons)    BERT, GPT, etc.)
```

---

## 📊 Quick Reference Card

| If you need... | Use... |
|---|---|
| Default hidden layer activation | **ReLU** |
| Transformer hidden layers | **GELU** |
| Binary classification output | **Sigmoid** |
| Multi-class classification output | **Softmax** |
| Regression output | **Linear (no activation)** |
| Fix dying ReLU neurons | **Leaky ReLU** or **ELU** |
| Deep CNN (40+ layers) | **Swish/SiLU** |
| RNN/LSTM gates | **Sigmoid** (gate) + **Tanh** (state) |

---

## ❓ Revision Questions

1. **Why are non-linear activation functions essential? What happens to a multi-layer network if all activations are linear (or identity)?**

2. **Explain the vanishing gradient problem mathematically. Why does sigmoid cause it, and how does ReLU solve it?**

3. **Compute sigmoid(2), sigmoid(-2), and their derivatives by hand. Verify that σ'(z) = σ(z)(1 - σ(z)).**

4. **What is the "dying ReLU" problem? How do Leaky ReLU and ELU address it? Give a scenario where this problem might occur.**

5. **Why is GELU preferred over ReLU in Transformer architectures? What is its stochastic interpretation?**

6. **Given logits z = [3.0, 1.0, -1.0], compute the Softmax output by hand. Verify that outputs sum to 1. Then show the numerically stable computation.**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Artificial Neurons & Perceptron](02-artificial-neurons-perceptron.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Multi-Layer Perceptron →](04-multi-layer-perceptron.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
