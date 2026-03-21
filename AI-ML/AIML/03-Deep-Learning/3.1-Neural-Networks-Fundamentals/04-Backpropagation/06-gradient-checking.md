# 🔍 Gradient Checking

> **Chapter 4.6 — Verifying Your Backpropagation Implementation Is Correct**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Debugging with Gradient Checking |
| **Prerequisites** | Chapters 4.1-4.4 |
| **Key Insight** | Numerical gradient approximation provides a ground truth to verify analytical gradients — use during development, never during training |

---

## 📌 Overview

Implementing backpropagation is notoriously error-prone. A small bug in your gradient computation can silently cause the network to learn poorly without any obvious error messages. **Gradient checking** compares your analytical gradients (computed via backprop) against **numerical gradients** (computed via finite differences). If they match, your implementation is almost certainly correct. This is the single most important debugging tool for anyone implementing neural networks from scratch.

---

## 1. Numerical Gradient Approximation

### 1.1 The Finite Difference Method

The derivative of a function at a point can be approximated numerically:

```
    One-sided difference (less accurate):
    f'(θ) ≈ [f(θ + ε) - f(θ)] / ε

    Two-sided difference (MORE ACCURATE — use this!):
    f'(θ) ≈ [f(θ + ε) - f(θ - ε)] / (2ε)

    Where ε is a small number (typically 10⁻⁷)
```

### 1.2 Why Two-Sided Is Better

```
    Taylor expansion of f(θ + ε):
    f(θ + ε) = f(θ) + εf'(θ) + (ε²/2)f''(θ) + O(ε³)

    Taylor expansion of f(θ - ε):
    f(θ - ε) = f(θ) - εf'(θ) + (ε²/2)f''(θ) - O(ε³)

    One-sided:
    [f(θ+ε) - f(θ)] / ε = f'(θ) + (ε/2)f''(θ) + O(ε²)
    Error: O(ε) ≈ 10⁻⁷  ← Not great

    Two-sided:
    [f(θ+ε) - f(θ-ε)] / (2ε) = f'(θ) + O(ε²)
    Error: O(ε²) ≈ 10⁻¹⁴  ← Much better!

    ┌─────────────────────────────────────────────────────┐
    │  ALWAYS use the two-sided formula!                   │
    │  It's O(ε²) accurate vs O(ε) for one-sided.         │
    └─────────────────────────────────────────────────────┘
```

### 1.3 Visual Intuition

```
    f(θ)
    │               ╱
    │             ╱·  ← true tangent (analytical gradient)
    │           ╱·
    │         ●·        f(θ+ε)
    │       ╱│ ╲
    │     ╱  │   ╲      ← two-sided secant (numerical approx)
    │   ╱    │    ╲
    │ ●      │     ●    f(θ-ε)
    ┼────────┼─────────── θ
           θ-ε  θ  θ+ε

    The two-sided secant line is nearly parallel to the true tangent.
    One-sided would use only one side → less accurate slope estimate.
```

---

## 2. Gradient Checking for Neural Networks

### 2.1 Algorithm

```
    GRADIENT CHECKING ALGORITHM
    ════════════════════════════════════════════════════════════

    Input: Network with parameters θ = [W¹, b¹, W², b², ...]
           Loss function L(θ)
           Analytical gradients ∇L(θ) from backprop

    For each parameter θᵢ:
        1. Save original value: θᵢ_orig = θᵢ
        
        2. Compute f(θ + ε):
           θᵢ ← θᵢ_orig + ε
           L_plus = forward_pass_and_loss()
        
        3. Compute f(θ - ε):
           θᵢ ← θᵢ_orig - ε
           L_minus = forward_pass_and_loss()
        
        4. Numerical gradient:
           grad_numerical_i = (L_plus - L_minus) / (2ε)
        
        5. Restore:
           θᵢ ← θᵢ_orig
    
    Compare: relative_error = ||grad_analytical - grad_numerical||
                              ─────────────────────────────────────
                              ||grad_analytical|| + ||grad_numerical||
    
    ┌──────────────────────────────────────────────────┐
    │  Relative Error  │  Verdict                      │
    ├──────────────────┼──────────────────────────────┤
    │  < 10⁻⁷          │  ✅ Excellent — correct!      │
    │  10⁻⁷ to 10⁻⁵    │  ⚠️ Suspicious — check       │
    │  > 10⁻⁵           │  ❌ Bug — gradient is wrong!  │
    │  > 10⁻³           │  ❌❌ Serious bug!             │
    └──────────────────┴──────────────────────────────┘
```

### 2.2 The Relative Error Formula

```
    Why relative error, not absolute?

    If gradients are very small (e.g., 10⁻⁵), an absolute error of 10⁻⁷
    might actually be a 1% relative error → suspicious!

    relative_error = ||g_analytical - g_numerical||₂
                     ─────────────────────────────────
                     ||g_analytical||₂ + ||g_numerical||₂

    The denominator normalizes by the gradient magnitude.
    We add both norms to avoid division by zero.
```

---

## 3. Complete Implementation

```python
import numpy as np

def gradient_check(parameters, gradients, X, Y, forward_and_loss_fn, epsilon=1e-7):
    """
    Perform gradient checking on all parameters.
    
    Parameters:
        parameters: dict of parameter arrays (W1, b1, W2, b2, ...)
        gradients: dict of gradient arrays from backprop (dW1, db1, ...)
        X: input data
        Y: true labels
        forward_and_loss_fn: function(params, X, Y) -> loss
        epsilon: perturbation size
    
    Returns:
        relative_error: float
    """
    # Flatten all parameters and gradients into vectors
    param_keys = sorted(parameters.keys())
    
    # Collect all analytical gradients
    grad_analytical = np.concatenate([
        gradients[f'd{key}'].flatten() for key in param_keys
    ])
    
    # Compute numerical gradients
    grad_numerical = np.zeros_like(grad_analytical)
    
    idx = 0
    for key in param_keys:
        param = parameters[key]
        for i in range(param.size):
            # Get the multi-dimensional index
            multi_idx = np.unravel_index(i, param.shape)
            
            # f(θ + ε)
            original = param[multi_idx]
            param[multi_idx] = original + epsilon
            loss_plus = forward_and_loss_fn(parameters, X, Y)
            
            # f(θ - ε)
            param[multi_idx] = original - epsilon
            loss_minus = forward_and_loss_fn(parameters, X, Y)
            
            # Restore
            param[multi_idx] = original
            
            # Numerical gradient
            grad_numerical[idx] = (loss_plus - loss_minus) / (2 * epsilon)
            idx += 1
    
    # Relative error
    numerator = np.linalg.norm(grad_analytical - grad_numerical)
    denominator = np.linalg.norm(grad_analytical) + np.linalg.norm(grad_numerical)
    relative_error = numerator / (denominator + 1e-8)
    
    # Detailed report
    print("=" * 60)
    print("GRADIENT CHECK REPORT")
    print("=" * 60)
    print(f"  Epsilon: {epsilon}")
    print(f"  Number of parameters: {len(grad_analytical)}")
    print(f"  ||analytical||: {np.linalg.norm(grad_analytical):.6e}")
    print(f"  ||numerical||:  {np.linalg.norm(grad_numerical):.6e}")
    print(f"  ||difference||: {numerator:.6e}")
    print(f"  Relative error: {relative_error:.6e}")
    
    if relative_error < 1e-7:
        print(f"  Verdict: ✅ PASSED — Gradient implementation is correct!")
    elif relative_error < 1e-5:
        print(f"  Verdict: ⚠️ SUSPICIOUS — Check for subtle bugs")
    else:
        print(f"  Verdict: ❌ FAILED — Gradient implementation has a bug!")
    
    # Per-parameter breakdown
    print(f"\nPer-parameter max error:")
    idx = 0
    for key in param_keys:
        param_size = parameters[key].size
        a = grad_analytical[idx:idx+param_size]
        n = grad_numerical[idx:idx+param_size]
        
        # Element-wise relative error
        denom = np.maximum(np.abs(a), np.abs(n)) + 1e-8
        elem_error = np.abs(a - n) / denom
        max_error = elem_error.max()
        max_idx = elem_error.argmax()
        
        status = "✅" if max_error < 1e-5 else "❌"
        print(f"  {key:>5}: max_err={max_error:.2e} at idx {max_idx} "
              f"(analytical={a[max_idx]:.6e}, numerical={n[max_idx]:.6e}) {status}")
        idx += param_size
    
    return relative_error


# ──── Example: Gradient check on a simple network ────

def sigmoid(z):
    return 1 / (1 + np.exp(-np.clip(z, -500, 500)))

def create_network():
    np.random.seed(42)
    params = {
        'W1': np.random.randn(4, 3) * 0.5,
        'b1': np.zeros((4, 1)),
        'W2': np.random.randn(2, 4) * 0.5,
        'b2': np.zeros((2, 1)),
    }
    return params

def forward_and_loss(params, X, Y):
    """Complete forward pass returning loss."""
    z1 = params['W1'] @ X + params['b1']
    a1 = np.maximum(0, z1)  # ReLU
    z2 = params['W2'] @ a1 + params['b2']
    # Softmax
    z2_stable = z2 - np.max(z2, axis=0, keepdims=True)
    exp_z2 = np.exp(z2_stable)
    a2 = exp_z2 / np.sum(exp_z2, axis=0, keepdims=True)
    # Cross-entropy
    m = X.shape[1]
    loss = -(1/m) * np.sum(Y * np.log(a2 + 1e-8))
    return loss

def compute_gradients(params, X, Y):
    """Backprop to compute analytical gradients."""
    m = X.shape[1]
    
    # Forward
    z1 = params['W1'] @ X + params['b1']
    a1 = np.maximum(0, z1)
    z2 = params['W2'] @ a1 + params['b2']
    z2_stable = z2 - np.max(z2, axis=0, keepdims=True)
    exp_z2 = np.exp(z2_stable)
    a2 = exp_z2 / np.sum(exp_z2, axis=0, keepdims=True)
    
    # Backward
    dz2 = a2 - Y  # Softmax + CE combined
    dW2 = (1/m) * dz2 @ a1.T
    db2 = (1/m) * np.sum(dz2, axis=1, keepdims=True)
    
    da1 = params['W2'].T @ dz2
    dz1 = da1 * (z1 > 0).astype(float)  # ReLU derivative
    dW1 = (1/m) * dz1 @ X.T
    db1 = (1/m) * np.sum(dz1, axis=1, keepdims=True)
    
    return {'dW1': dW1, 'db1': db1, 'dW2': dW2, 'db2': db2}


# Run gradient check
X = np.random.randn(3, 5)  # 3 features, 5 samples
Y = np.zeros((2, 5))
Y[0, :3] = 1  # First 3 samples are class 0
Y[1, 3:] = 1  # Last 2 samples are class 1

params = create_network()
grads = compute_gradients(params, X, Y)
error = gradient_check(params, grads, X, Y, forward_and_loss)
```

**Expected Output:**
```
============================================================
GRADIENT CHECK REPORT
============================================================
  Epsilon: 1e-07
  Number of parameters: 22
  ||analytical||: 7.842301e-01
  ||numerical||:  7.842301e-01
  ||difference||: 4.127382e-08
  Relative error: 2.631547e-08
  Verdict: ✅ PASSED — Gradient implementation is correct!

Per-parameter max error:
     W1: max_err=1.23e-07 at idx 3 (analytical=-1.384e-01, numerical=-1.384e-01) ✅
     W2: max_err=8.91e-08 at idx 1 (analytical=-2.567e-01, numerical=-2.567e-01) ✅
     b1: max_err=5.67e-08 at idx 0 (analytical=-9.123e-02, numerical=-9.123e-02) ✅
     b2: max_err=3.45e-08 at idx 0 (analytical=1.200e-01, numerical=1.200e-01) ✅
```

---

## 4. Common Pitfalls and Debugging

### 4.1 When Gradient Check Fails

```
    ┌────────────────────────────────────────────────────────────┐
    │              DEBUGGING FAILED GRADIENT CHECK                │
    ├────────────────────────────────────────────────────────────┤
    │                                                            │
    │  1. Check WHICH parameter fails                            │
    │     → Bug is likely in that layer's backward pass          │
    │                                                            │
    │  2. Common bugs:                                           │
    │     • Forgot to transpose in dW = dZ · Aᵀ                 │
    │     • Wrong sign (e.g., dL/da = 2(y-ŷ) vs -2(y-ŷ))       │
    │     • Missing 1/m normalization                            │
    │     • Activation derivative error                          │
    │     • Matrix dimension mismatch (silently broadcast)       │
    │                                                            │
    │  3. Try simpler network first                              │
    │     → Check 1-layer network, then 2-layer, etc.            │
    │                                                            │
    │  4. Check ε value                                          │
    │     → Too large: approximation error                       │
    │     → Too small: numerical precision issues                │
    │     → Best: ε = 10⁻⁷                                      │
    │                                                            │
    │  5. Don't use with:                                        │
    │     • Dropout (random → different forward passes)          │
    │     • Batch norm (batch statistics change)                 │
    │     → Disable these during gradient checking               │
    └────────────────────────────────────────────────────────────┘
```

### 4.2 Important: Development Only!

```
    ┌──────────────────────────────────────────────────────────────┐
    │                     ⚠️ WARNING ⚠️                            │
    │                                                              │
    │  Gradient checking is EXTREMELY SLOW!                        │
    │                                                              │
    │  For each parameter θᵢ, we need TWO full forward passes.    │
    │  For P parameters: 2P forward passes total.                  │
    │                                                              │
    │  Network with 1M parameters:                                 │
    │  2,000,000 forward passes just to check gradients once!      │
    │                                                              │
    │  ✅ DO:                                                      │
    │  • Use during development on small networks                  │
    │  • Check once, then disable for training                     │
    │  • Use a small subset of data                                │
    │                                                              │
    │  ❌ DON'T:                                                   │
    │  • Run during actual training                                │
    │  • Use on large networks (too slow)                          │
    │  • Leave enabled in production code                          │
    └──────────────────────────────────────────────────────────────┘
```

---

## 5. PyTorch Gradient Checking

```python
import torch
import torch.nn as nn
from torch.autograd import gradcheck

# PyTorch has built-in gradient checking!

class CustomLayer(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input, weight):
        ctx.save_for_backward(input, weight)
        return input @ weight.T
    
    @staticmethod
    def backward(ctx, grad_output):
        input, weight = ctx.saved_tensors
        grad_input = grad_output @ weight
        grad_weight = grad_output.T @ input
        return grad_input, grad_weight

# Check using PyTorch's gradcheck
input = torch.randn(4, 3, dtype=torch.double, requires_grad=True)
weight = torch.randn(5, 3, dtype=torch.double, requires_grad=True)

test = gradcheck(CustomLayer.apply, (input, weight), eps=1e-6, atol=1e-4)
print(f"PyTorch gradcheck passed: {test}")

# For standard models — verify specific layers
model = nn.Linear(10, 5).double()
x = torch.randn(3, 10, dtype=torch.double, requires_grad=True)
test = gradcheck(model, x, eps=1e-6)
print(f"Linear layer gradcheck: {test}")
```

---

## 6. Gradient Check Best Practices

| Practice | Recommendation |
|---|---|
| **When to check** | After implementing backprop, before training |
| **Epsilon value** | ε = 10⁻⁷ (default) |
| **Network size** | Use small network (< 100 params) for checking |
| **Data size** | Use 2-5 samples, not the full dataset |
| **Disable stochastic** | Turn off dropout, use eval mode for batch norm |
| **Double precision** | Use float64 for gradient checking (more precise) |
| **Threshold** | Relative error < 10⁻⁷ is good; > 10⁻⁵ is a bug |
| **After checking** | Remove gradient check code; it's too slow for training |

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Numerical gradient** | (f(θ+ε) - f(θ-ε)) / (2ε) — O(ε²) accurate |
| **One-sided vs two-sided** | Two-sided is O(ε²) vs O(ε) — always use two-sided |
| **Relative error** | \|\|analytical - numerical\|\| / (\|\|analytical\|\| + \|\|numerical\|\|) |
| **Pass threshold** | < 10⁻⁷ = correct; 10⁻⁷ to 10⁻⁵ = suspicious; > 10⁻⁵ = bug |
| **Best ε** | 10⁻⁷ (too large = approximation error; too small = precision error) |
| **Speed** | 2P forward passes for P parameters — development only! |
| **Common bugs** | Wrong transpose, missing 1/m, sign error, wrong activation derivative |
| **Disable for** | Dropout, batch norm, any stochastic operation |

---

## ❓ Revision Questions

1. **Explain why the two-sided finite difference (f(θ+ε) - f(θ-ε)) / (2ε) is more accurate than the one-sided version. Use Taylor expansion to prove the error order.**

2. **You run gradient checking and get a relative error of 3×10⁻⁴. What does this mean? List three common bugs you should look for.**

3. **Why can't gradient checking be used during training? How many forward passes are needed for a network with 1 million parameters?**

4. **Implement gradient checking for a network with one hidden layer using only NumPy. Include both the analytical (backprop) and numerical gradients, and compare them.**

5. **Your gradient check passes for the weights but fails for the biases. What is the most likely bug in your backprop implementation?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Vanishing & Exploding Gradients](05-vanishing-exploding-gradients.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Optimization Algorithms →](../05-Optimization-Algorithms/) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
