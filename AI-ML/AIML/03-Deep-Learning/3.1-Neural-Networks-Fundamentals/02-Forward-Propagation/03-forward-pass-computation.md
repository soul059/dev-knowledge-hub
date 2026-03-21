# 🔄 Forward Pass Computation

> **Chapter 2.3 — Step-by-Step: How Data Flows Through a Neural Network**

| Topic | Details |
|---|---|
| **Subject** | Forward Propagation — Computing Predictions |
| **Prerequisites** | Chapters 2.1-2.2 (Layers, Weights, Biases) |
| **Key Insight** | The forward pass is simply repeated application of z = Wa + b followed by a = g(z), layer by layer |

---

## 📌 Overview

The **forward pass** (or forward propagation) is the process of computing the output of a neural network given an input. Data flows from input to output through a sequence of linear transformations and non-linear activations. Understanding every step — and being able to trace actual numbers through a network — is essential for debugging, implementing from scratch, and understanding backpropagation.

---

## 1. The Forward Pass — General Algorithm

### 1.1 Core Equations

```
    For each layer l = 1, 2, ..., L:
    
    ┌────────────────────────────────────────────┐
    │  Step 1: Linear transformation              │
    │          z^[l] = W^[l] · a^[l-1] + b^[l]   │
    │                                             │
    │  Step 2: Non-linear activation              │
    │          a^[l] = g^[l](z^[l])               │
    └────────────────────────────────────────────┘
    
    Start:  a^[0] = x  (the input)
    End:    ŷ = a^[L]  (the prediction)
```

### 1.2 The Forward Pass Pipeline

```
     x        W^[1],b^[1]    g^[1]     W^[2],b^[2]    g^[2]     W^[L],b^[L]    g^[L]
     │            │           │            │           │            │           │
     ▼            ▼           ▼            ▼           ▼            ▼           ▼
    ┌──┐       ┌─────┐    ┌─────┐      ┌─────┐    ┌─────┐      ┌─────┐    ┌─────┐
    │a⁰├──────►│z^[1]├───►│a^[1]├─────►│z^[2]├───►│a^[2]├─···─►│z^[L]├───►│a^[L]│= ŷ
    └──┘       └─────┘    └─────┘      └─────┘    └─────┘      └─────┘    └─────┘
     input     linear     activate     linear     activate     linear     activate
               Wa+b       ReLU         Wa+b       ReLU         Wa+b       softmax
```

### 1.3 What Gets Cached (Important for Backprop!)

During the forward pass, we must **cache intermediate values** for use during backpropagation:

```
    Cache = {
        z^[1], a^[1],
        z^[2], a^[2],
        ...
        z^[L], a^[L]
    }
    
    Plus: a^[0] = x (the input)
    
    Why? Backprop needs these to compute gradients:
    • z^[l] needed for g'(z^[l])
    • a^[l-1] needed for ∂L/∂W^[l] = dz^[l] · a^[l-1]ᵀ
```

---

## 2. Complete Numerical Example — 2-Layer Network

### 2.1 Network Setup

```
    Architecture: 3 → 4 → 2
    
    Input:  x = [1.0, 0.5, -1.5]ᵀ    (3 features)
    Hidden: 4 neurons, ReLU activation
    Output: 2 neurons, Softmax activation (2-class classification)
    
    Layer 1 (Hidden):
    W^[1] = ┌                          ┐
            │  0.2   0.4  -0.5         │
            │ -0.3   0.1   0.2         │
            │  0.5  -0.2   0.3         │
            │  0.1   0.6  -0.1         │
            └                          ┘
    (4×3 matrix)
    
    b^[1] = [0.1, -0.1, 0.0, 0.2]ᵀ
    (4×1 vector)
    
    Layer 2 (Output):
    W^[2] = ┌                          ┐
            │  0.3  -0.2   0.4   0.1   │
            │ -0.1   0.5  -0.3   0.2   │
            └                          ┘
    (2×4 matrix)
    
    b^[2] = [0.1, -0.1]ᵀ
    (2×1 vector)
```

### 2.2 Step-by-Step Forward Pass

#### Step 1: Input Layer

```
    a^[0] = x = [1.0, 0.5, -1.5]ᵀ
```

#### Step 2: Layer 1 — Linear Transformation

```
    z^[1] = W^[1] · a^[0] + b^[1]

    ┌                  ┐   ┌      ┐       ┌     ┐
    │  0.2  0.4  -0.5  │   │  1.0 │       │ 0.1 │
    │ -0.3  0.1   0.2  │ · │  0.5 │   +   │-0.1 │
    │  0.5 -0.2   0.3  │   │ -1.5 │       │ 0.0 │
    │  0.1  0.6  -0.1  │   └      ┘       │ 0.2 │
    └                  ┘                   └     ┘

    Computing each element:
    z₁^[1] = (0.2)(1.0) + (0.4)(0.5) + (-0.5)(-1.5) + 0.1
           = 0.2 + 0.2 + 0.75 + 0.1
           = 1.25

    z₂^[1] = (-0.3)(1.0) + (0.1)(0.5) + (0.2)(-1.5) + (-0.1)
           = -0.3 + 0.05 + (-0.3) + (-0.1)
           = -0.65

    z₃^[1] = (0.5)(1.0) + (-0.2)(0.5) + (0.3)(-1.5) + 0.0
           = 0.5 + (-0.1) + (-0.45) + 0.0
           = -0.05

    z₄^[1] = (0.1)(1.0) + (0.6)(0.5) + (-0.1)(-1.5) + 0.2
           = 0.1 + 0.3 + 0.15 + 0.2
           = 0.75

    z^[1] = [1.25, -0.65, -0.05, 0.75]ᵀ
```

#### Step 3: Layer 1 — ReLU Activation

```
    a^[1] = ReLU(z^[1]) = max(0, z^[1])

    a₁^[1] = max(0, 1.25)  = 1.25   ← positive, passes through
    a₂^[1] = max(0, -0.65) = 0.00   ← negative, killed by ReLU!
    a₃^[1] = max(0, -0.05) = 0.00   ← negative, killed by ReLU!
    a₄^[1] = max(0, 0.75)  = 0.75   ← positive, passes through

    a^[1] = [1.25, 0.00, 0.00, 0.75]ᵀ

    Note: 2 out of 4 neurons are "dead" (output 0)
    This is ReLU's sparse activation in action!
```

#### Step 4: Layer 2 — Linear Transformation

```
    z^[2] = W^[2] · a^[1] + b^[2]

    ┌                          ┐   ┌      ┐       ┌     ┐
    │  0.3  -0.2   0.4   0.1  │   │ 1.25 │       │ 0.1 │
    │ -0.1   0.5  -0.3   0.2  │ · │ 0.00 │   +   │-0.1 │
    └                          ┘   │ 0.00 │       └     ┘
                                   │ 0.75 │
                                   └      ┘

    z₁^[2] = (0.3)(1.25) + (-0.2)(0.00) + (0.4)(0.00) + (0.1)(0.75) + 0.1
           = 0.375 + 0 + 0 + 0.075 + 0.1
           = 0.55

    z₂^[2] = (-0.1)(1.25) + (0.5)(0.00) + (-0.3)(0.00) + (0.2)(0.75) + (-0.1)
           = -0.125 + 0 + 0 + 0.15 + (-0.1)
           = -0.075

    z^[2] = [0.55, -0.075]ᵀ
```

#### Step 5: Layer 2 — Softmax Activation

```
    a^[2] = Softmax(z^[2])

    e^0.55  = 1.7333
    e^-0.075 = 0.9277

    Sum = 1.7333 + 0.9277 = 2.6610

    a₁^[2] = 1.7333 / 2.6610 = 0.6514
    a₂^[2] = 0.9277 / 2.6610 = 0.3486

    a^[2] = [0.6514, 0.3486]ᵀ

    Verification: 0.6514 + 0.3486 = 1.0000 ✓
```

#### Step 6: Prediction

```
    ŷ = a^[2] = [0.6514, 0.3486]ᵀ

    Interpretation:
    • Class 0 probability: 65.14%
    • Class 1 probability: 34.86%
    • Predicted class: 0 (argmax)
```

### 2.3 Complete Summary Diagram

```
    INPUT              HIDDEN (ReLU)        OUTPUT (Softmax)
    a^[0]              z^[1] → a^[1]        z^[2] → a^[2]
    
    ┌─────┐     W,b    ┌───────────────┐     W,b    ┌─────────────┐
    │ 1.00├────────────►│ 1.25 → 1.25  ├────────────►│ 0.55 → 0.65│ = P(class 0)
    │     │             │-0.65 → 0.00  │             │-0.08 → 0.35│ = P(class 1)
    │ 0.50├────────────►│-0.05 → 0.00  ├────────────►└─────────────┘
    │     │             │ 0.75 → 0.75  │
    │-1.50├────────────►└───────────────┘
    └─────┘
```

---

## 3. Python Implementation

```python
import numpy as np

def forward_pass(x, parameters):
    """
    Compute the forward pass through a multi-layer network.
    
    Parameters:
        x: input vector, shape (n_features, 1)
        parameters: dict with W1, b1, W2, b2, ..., WL, bL
    
    Returns:
        output: final activation a^[L]
        cache: dict of all intermediate z and a values
    """
    cache = {'a0': x}
    a = x
    L = len(parameters) // 2  # number of layers
    
    for l in range(1, L + 1):
        W = parameters[f'W{l}']
        b = parameters[f'b{l}']
        
        # Linear: z = Wa + b
        z = W @ a + b
        cache[f'z{l}'] = z
        
        # Activation
        if l < L:
            # Hidden layers: ReLU
            a = np.maximum(0, z)
        else:
            # Output layer: Softmax
            z_stable = z - np.max(z)
            exp_z = np.exp(z_stable)
            a = exp_z / np.sum(exp_z)
        
        cache[f'a{l}'] = a
    
    return a, cache


# ──── Set up the exact example from above ────
x = np.array([[1.0], [0.5], [-1.5]])

parameters = {
    'W1': np.array([[ 0.2,  0.4, -0.5],
                    [-0.3,  0.1,  0.2],
                    [ 0.5, -0.2,  0.3],
                    [ 0.1,  0.6, -0.1]]),
    'b1': np.array([[0.1], [-0.1], [0.0], [0.2]]),
    
    'W2': np.array([[ 0.3, -0.2,  0.4,  0.1],
                    [-0.1,  0.5, -0.3,  0.2]]),
    'b2': np.array([[0.1], [-0.1]])
}

# Run forward pass
output, cache = forward_pass(x, parameters)

# Print all intermediate values
print("=" * 50)
print("FORWARD PASS — Complete Trace")
print("=" * 50)
print(f"\na^[0] (input):     {cache['a0'].flatten()}")
print(f"\nz^[1] (pre-act):   {cache['z1'].flatten()}")
print(f"a^[1] (ReLU):      {cache['a1'].flatten()}")
print(f"\nz^[2] (pre-act):   {cache['z2'].flatten()}")
print(f"a^[2] (softmax):   {cache['a2'].flatten()}")
print(f"\nPredicted class: {np.argmax(output)}")
print(f"Confidence:      {np.max(output):.4f}")

# Dimension verification
print(f"\n{'Dimension Verification':^50}")
print("-" * 50)
for key, val in cache.items():
    print(f"  {key}: {val.shape}")
for key in ['W1', 'b1', 'W2', 'b2']:
    print(f"  {key}: {parameters[key].shape}")
```

**Expected Output:**
```
==================================================
FORWARD PASS — Complete Trace
==================================================

a^[0] (input):     [ 1.   0.5 -1.5]

z^[1] (pre-act):   [ 1.25 -0.65 -0.05  0.75]
a^[1] (ReLU):      [1.25 0.   0.   0.75]

z^[2] (pre-act):   [ 0.55  -0.075]
a^[2] (softmax):   [0.6514 0.3486]

Predicted class: 0
Confidence:      0.6514
```

---

## 4. Forward Pass for Different Tasks

### 4.1 Regression

```python
# For regression: output layer has NO activation (linear output)
# Final layer: a^[L] = z^[L]  (no softmax/sigmoid)

def forward_regression(x, parameters):
    cache = {'a0': x}
    a = x
    L = len(parameters) // 2
    
    for l in range(1, L + 1):
        z = parameters[f'W{l}'] @ a + parameters[f'b{l}']
        cache[f'z{l}'] = z
        
        if l < L:
            a = np.maximum(0, z)  # ReLU for hidden
        else:
            a = z                 # Linear for output (regression)
        
        cache[f'a{l}'] = a
    return a, cache

# Example: predict house price
x_house = np.array([[3.0], [2.0], [1500.0], [0.8]])  # bedrooms, baths, sqft, garage
# output: single number (predicted price)
```

### 4.2 Binary Classification

```python
# For binary: output layer has sigmoid activation
# Final layer: a^[L] = σ(z^[L])

def forward_binary(x, parameters):
    cache = {'a0': x}
    a = x
    L = len(parameters) // 2
    
    for l in range(1, L + 1):
        z = parameters[f'W{l}'] @ a + parameters[f'b{l}']
        cache[f'z{l}'] = z
        
        if l < L:
            a = np.maximum(0, z)  # ReLU for hidden
        else:
            a = 1 / (1 + np.exp(-z))  # Sigmoid for output
        
        cache[f'a{l}'] = a
    return a, cache

# Output: P(spam) = 0.87 → classify as spam
```

---

## 5. Dimension Verification Checklist

Always verify dimensions match at each step:

```
    ┌──────────────────────────────────────────────────────────────┐
    │              DIMENSION VERIFICATION CHECKLIST                 │
    ├──────────────────────────────────────────────────────────────┤
    │                                                              │
    │  For layer l:                                                │
    │                                                              │
    │  W^[l]:     (n^[l], n^[l-1])     ← rows=current, cols=prev  │
    │  a^[l-1]:   (n^[l-1], 1)         ← column vector            │
    │  b^[l]:     (n^[l], 1)           ← matches rows of W        │
    │                                                              │
    │  z^[l] = W^[l] · a^[l-1] + b^[l]                           │
    │          (n^[l], n^[l-1])·(n^[l-1], 1) + (n^[l], 1)        │
    │        = (n^[l], 1) + (n^[l], 1)                            │
    │        = (n^[l], 1)              ✓ dimensions match!         │
    │                                                              │
    │  a^[l] = g(z^[l])               ← same shape as z^[l]       │
    │        = (n^[l], 1)                                          │
    │                                                              │
    │  ✓ Output of layer l has shape (n^[l], 1)                    │
    │  ✓ This becomes input to layer l+1                           │
    └──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Forward pass** | Repeated application of z = Wa+b and a = g(z) from input to output |
| **Linear step** | z^[l] = W^[l] · a^[l-1] + b^[l] |
| **Activation step** | a^[l] = g^[l](z^[l]) |
| **Input** | a^[0] = x (raw features) |
| **Output** | ŷ = a^[L] (final prediction) |
| **Caching** | Store all z and a values — needed for backpropagation |
| **Dimension check** | z^[l] shape = (n^[l], 1) — must match at every step |
| **Hidden activation** | Typically ReLU (hidden layers) |
| **Output activation** | Softmax (multi-class), sigmoid (binary), linear (regression) |
| **ReLU sparsity** | Many hidden neurons output 0 — this is normal and desirable |

---

## ❓ Revision Questions

1. **Given a network with architecture 2→3→1 and the following parameters, compute the complete forward pass for input x = [1, -1]ᵀ:**
   - W¹ = [[0.5, -0.3], [0.2, 0.7], [-0.4, 0.1]], b¹ = [0.1, 0, -0.2]ᵀ
   - W² = [[0.6, -0.5, 0.3]], b² = [0.2]
   - Hidden: ReLU, Output: Sigmoid

2. **Why must we cache z^[l] and a^[l-1] during the forward pass? What specific backpropagation computations need them?**

3. **In the worked example, two out of four hidden neurons output zero. Is this a problem? Explain the concept of sparse activation.**

4. **Write a dimension verification for a 4-layer network: 100→64→32→16→5. Show the shape of every W, b, z, and a.**

5. **What is the difference in the forward pass between classification and regression networks? Be specific about the output layer.**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Weight Matrices & Bias Vectors](02-weight-matrices-bias-vectors.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Matrix Notation →](04-matrix-notation.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
