# 📐 Matrix Notation & Vectorized Forward Pass

> **Chapter 2.4 — Efficient Computation: From Loops to Linear Algebra**

| Topic | Details |
|---|---|
| **Subject** | Forward Propagation — Vectorized Computation |
| **Prerequisites** | Chapter 2.3, basic linear algebra |
| **Key Insight** | Vectorized operations replace explicit loops, enabling efficient GPU computation and cleaner code |

---

## 📌 Overview

In practice, neural networks process data in **batches**, not one sample at a time. Matrix notation and vectorized computation allow us to process an entire batch simultaneously using optimized linear algebra libraries. This chapter bridges the gap between the single-sample forward pass and the efficient batched computation used in real implementations.

---

## 1. From Loops to Matrices

### 1.1 The Naive Approach (Loops — Slow!)

Processing m samples one at a time:

```python
# SLOW: Loop over samples
for i in range(m):
    z = W @ x[:, i:i+1] + b    # Process sample i
    a = relu(z)                  # One at a time
    # ... very slow for large m
```

### 1.2 The Vectorized Approach (Matrices — Fast!)

Processing all m samples simultaneously:

```python
# FAST: All samples at once
Z = W @ X + b    # X has all m samples as columns
A = relu(Z)       # Element-wise, applied to entire matrix
```

### 1.3 Why Vectorization is Faster

```
    Loop (CPU, sequential):        Vectorized (GPU, parallel):
    
    Sample 1: W·x₁+b → a₁         ┌──────────────────────────┐
    Sample 2: W·x₁+b → a₂         │  W · [x₁ x₂ ... xₘ] + b │
    Sample 3: W·x₁+b → a₃         │       ↓↓↓↓↓↓↓↓↓↓↓       │
    ...                             │  = [z₁ z₂ ... zₘ]        │
    Sample m: W·xₘ+b → aₘ         │  All computed in parallel!│
                                    └──────────────────────────┘
    Time: O(m × n²)                Time: O(n²) with parallelism
    
    Speedup: 100-1000x on GPU!
```

---

## 2. Notation: Single Sample vs Batch

### 2.1 Convention

| Symbol | Meaning | Shape |
|---|---|---|
| **Lowercase** x, z, a | Single sample (column vector) | (n, 1) |
| **Uppercase** X, Z, A | Batch of m samples | (n, m) |
| x⁽ⁱ⁾ | The i-th training sample | (n, 1) |

### 2.2 Data Matrix Layout

```
    X = [x⁽¹⁾  x⁽²⁾  x⁽³⁾  ...  x⁽ᵐ⁾]

    Each COLUMN is one sample:
    
    X = ┌                                    ┐
        │ x₁⁽¹⁾  x₁⁽²⁾  x₁⁽³⁾  ...  x₁⁽ᵐ⁾ │   ← feature 1 across all samples
        │ x₂⁽¹⁾  x₂⁽²⁾  x₂⁽³⁾  ...  x₂⁽ᵐ⁾ │   ← feature 2 across all samples
        │ x₃⁽¹⁾  x₃⁽²⁾  x₃⁽³⁾  ...  x₃⁽ᵐ⁾ │   ← feature 3 across all samples
        │  ...    ...    ...    ...   ...    │
        │ xₙ⁽¹⁾  xₙ⁽²⁾  xₙ⁽³⁾  ...  xₙ⁽ᵐ⁾ │   ← feature n across all samples
        └                                    ┘
    
    Shape: (n_features, m_samples) = (n⁰, m)
    
    ┌────────────────────────────────────────────────────┐
    │  CONVENTION NOTE:                                   │
    │  • This course: samples as COLUMNS (common in      │
    │    math/theory notation — Andrew Ng's style)        │
    │  • PyTorch/sklearn: samples as ROWS (shape: m × n)  │
    │  • Both are valid — just be consistent!             │
    └────────────────────────────────────────────────────┘
```

---

## 3. Vectorized Forward Pass Equations

### 3.1 Complete Equations

For layer l, processing m samples simultaneously:

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │  Z^[l] = W^[l] · A^[l-1] + b^[l]                       │
    │                                                          │
    │  Shapes:                                                 │
    │  (n^[l], m) = (n^[l], n^[l-1]) · (n^[l-1], m) + (n^[l],1)
    │                                                          │
    │  A^[l] = g^[l](Z^[l])                                   │
    │  (n^[l], m) = element-wise activation                    │
    │                                                          │
    │  Note: b^[l] is (n^[l], 1) and gets BROADCAST to (n^[l], m)
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### 3.2 Broadcasting Explained

```
    Z = W · A + b

    W · A = ┌──────────┐         b = ┌───┐      Broadcasting:
            │ z₁₁ z₁₂ │             │ b₁│      ┌──────────┐
            │ z₂₁ z₂₂ │      +      │ b₂│  =   │ z₁₁+b₁ z₁₂+b₁│
            │ z₃₁ z₃₂ │             │ b₃│      │ z₂₁+b₂ z₂₂+b₂│
            └──────────┘             └───┘      │ z₃₁+b₃ z₃₂+b₃│
            (3, 2)                   (3, 1)     └──────────┘
                                                 (3, 2)

    The bias vector is automatically repeated for each sample.
    In NumPy, this happens automatically via broadcasting.
```

### 3.3 Dimension Flow Through the Network

```
    Architecture: n⁰ → n¹ → n² → n³  with batch size m

    A^[0]:  (n⁰, m)    ← input batch
    
    W^[1]:  (n¹, n⁰)   b^[1]: (n¹, 1)
    Z^[1]:  (n¹, m)     A^[1]: (n¹, m)   ← hidden layer 1 output
    
    W^[2]:  (n², n¹)   b^[2]: (n², 1)
    Z^[2]:  (n², m)     A^[2]: (n², m)   ← hidden layer 2 output
    
    W^[3]:  (n³, n²)   b^[3]: (n³, 1)
    Z^[3]:  (n³, m)     A^[3]: (n³, m)   ← output (predictions for all m samples)

    Specific example: 784 → 256 → 10, batch_size = 64

    A^[0]:  (784, 64)
    W^[1]:  (256, 784)  b^[1]: (256, 1)
    Z^[1]:  (256, 64)   A^[1]: (256, 64)
    W^[2]:  (10, 256)   b^[2]: (10, 1)
    Z^[2]:  (10, 64)    A^[2]: (10, 64)   ← 10 class probabilities for 64 samples
```

---

## 4. Numerical Example — Batched Forward Pass

### 4.1 Setup

```
    Architecture: 2 → 3 → 2
    Batch size: m = 3

    Input batch (2 features, 3 samples):
    X = A^[0] = ┌             ┐
                │ 1.0  0.5  2.0│   ← feature 1
                │-1.0  0.8 -0.5│   ← feature 2
                └             ┘
    Shape: (2, 3)

    W^[1] = ┌              ┐       b^[1] = ┌    ┐
            │ 0.3 -0.1      │               │ 0.1│
            │ 0.5  0.2      │               │ 0.0│
            │-0.2  0.4      │               │-0.1│
            └              ┘               └    ┘
    Shape: (3, 2)                   Shape: (3, 1)

    W^[2] = ┌                ┐     b^[2] = ┌    ┐
            │ 0.4 -0.3  0.2  │             │ 0.0│
            │-0.1  0.5  0.1  │             │ 0.1│
            └                ┘             └    ┘
    Shape: (2, 3)                   Shape: (2, 1)
```

### 4.2 Layer 1 Computation

```
    Z^[1] = W^[1] · A^[0] + b^[1]

    ┌              ┐   ┌             ┐       ┌    ┐
    │ 0.3 -0.1     │   │ 1.0  0.5  2.0│       │ 0.1│
    │ 0.5  0.2     │ · │-1.0  0.8 -0.5│   +   │ 0.0│   (broadcast)
    │-0.2  0.4     │   └             ┘       │-0.1│
    └              ┘                          └    ┘
      (3, 2)           (2, 3)                 (3, 1)

    Sample 1:                  Sample 2:              Sample 3:
    z₁ = 0.3(1)+(-0.1)(-1)+0.1  z₁ = 0.3(.5)+(-0.1)(.8)+0.1  z₁ = 0.3(2)+(-0.1)(-.5)+0.1
       = 0.3+0.1+0.1 = 0.5        = 0.15-0.08+0.1 = 0.17       = 0.6+0.05+0.1 = 0.75
    z₂ = 0.5(1)+0.2(-1)+0.0     z₂ = 0.5(.5)+0.2(.8)+0.0     z₂ = 0.5(2)+0.2(-.5)+0.0
       = 0.5-0.2 = 0.3            = 0.25+0.16 = 0.41            = 1.0-0.1 = 0.9
    z₃ = -0.2(1)+0.4(-1)-0.1    z₃ = -0.2(.5)+0.4(.8)-0.1    z₃ = -0.2(2)+0.4(-.5)-0.1
       = -0.2-0.4-0.1 = -0.7      = -0.1+0.32-0.1 = 0.12       = -0.4-0.2-0.1 = -0.7

    Z^[1] = ┌                    ┐
            │  0.50   0.17   0.75│
            │  0.30   0.41   0.90│
            │ -0.70   0.12  -0.70│
            └                    ┘
    Shape: (3, 3) ✓

    A^[1] = ReLU(Z^[1]) = ┌                    ┐
                           │  0.50   0.17   0.75│
                           │  0.30   0.41   0.90│
                           │  0.00   0.12   0.00│  ← negatives → 0
                           └                    ┘
```

### 4.3 Layer 2 Computation

```
    Z^[2] = W^[2] · A^[1] + b^[2]

    ┌                ┐   ┌                    ┐       ┌    ┐
    │ 0.4 -0.3  0.2  │   │  0.50   0.17   0.75│       │ 0.0│
    │-0.1  0.5  0.1  │ · │  0.30   0.41   0.90│   +   │ 0.1│
    └                ┘   │  0.00   0.12   0.00│       └    ┘
                         └                    ┘

    Z^[2] = ┌                        ┐
            │  0.110   -0.031   0.030│
            │  0.200    0.301   0.525│   (+ bias broadcast)
            └                        ┘

    A^[2] = Softmax(Z^[2], axis=0)  ← per column (per sample)
    
    Sample 1: softmax([0.110, 0.200]) = [0.478, 0.522]
    Sample 2: softmax([-0.031, 0.301]) = [0.418, 0.582]
    Sample 3: softmax([0.030, 0.525]) = [0.378, 0.622]

    A^[2] = ┌                    ┐
            │ 0.478  0.418  0.378│  ← P(class 0) for each sample
            │ 0.522  0.582  0.622│  ← P(class 1) for each sample
            └                    ┘

    Predictions: [1, 1, 1]  (all samples predicted as class 1)
```

---

## 5. NumPy Implementation — Vectorized

```python
import numpy as np

def vectorized_forward(X, parameters):
    """
    Vectorized forward pass — processes entire batch at once.
    
    Parameters:
        X: input matrix, shape (n_features, m_samples)
        parameters: dict with W1, b1, ..., WL, bL
    
    Returns:
        AL: output, shape (n_output, m_samples)
        cache: all intermediate values
    """
    cache = {'A0': X}
    A = X
    L = len(parameters) // 2
    
    for l in range(1, L + 1):
        W = parameters[f'W{l}']
        b = parameters[f'b{l}']
        
        # Vectorized linear transformation
        Z = W @ A + b  # Broadcasting handles bias!
        
        # Activation
        if l < L:
            A = np.maximum(0, Z)  # ReLU (vectorized)
        else:
            # Softmax (numerically stable, per-column)
            Z_stable = Z - np.max(Z, axis=0, keepdims=True)
            exp_Z = np.exp(Z_stable)
            A = exp_Z / np.sum(exp_Z, axis=0, keepdims=True)
        
        cache[f'Z{l}'] = Z
        cache[f'A{l}'] = A
    
    return A, cache


# ──── Timing comparison: Loop vs Vectorized ────
def loop_forward(X, W, b):
    """Non-vectorized (slow) forward pass for one layer"""
    m = X.shape[1]
    n = W.shape[0]
    Z = np.zeros((n, m))
    for i in range(m):
        Z[:, i:i+1] = W @ X[:, i:i+1] + b
    return Z

def vectorized(X, W, b):
    """Vectorized (fast) forward pass for one layer"""
    return W @ X + b

import time

# Create large batch for timing
np.random.seed(42)
X_large = np.random.randn(784, 10000)  # 10,000 samples of 784 features
W_large = np.random.randn(256, 784)
b_large = np.random.randn(256, 1)

# Time loop version
start = time.time()
Z_loop = loop_forward(X_large, W_large, b_large)
loop_time = time.time() - start

# Time vectorized version
start = time.time()
Z_vec = vectorized(X_large, W_large, b_large)
vec_time = time.time() - start

print(f"Loop version:       {loop_time:.4f} seconds")
print(f"Vectorized version: {vec_time:.4f} seconds")
print(f"Speedup:            {loop_time/vec_time:.1f}x")
print(f"Results match:      {np.allclose(Z_loop, Z_vec)}")
```

**Expected Output:**
```
Loop version:       0.8523 seconds
Vectorized version: 0.0089 seconds
Speedup:            95.8x
Results match:      True
```

---

## 6. Common Pitfalls and Tips

### 6.1 Dimension Mismatch Errors

```
    ERROR: shapes (256, 784) and (256, 64) not aligned

    This means you probably transposed something wrong.
    
    Check: W @ A  requires  W.shape[1] == A.shape[0]
           (n, k)   (k, m) → (n, m)  ✓
    
    Common fix: Use A, not A.T (or vice versa)
```

### 6.2 Broadcasting Gotchas

```python
# WRONG: bias shape (256,) — 1D array
b = np.zeros(256)           # shape: (256,)
Z = W @ X + b               # May work but can produce wrong results!

# RIGHT: bias shape (256, 1) — column vector
b = np.zeros((256, 1))      # shape: (256, 1)
Z = W @ X + b               # Correctly broadcasts to (256, m)
```

### 6.3 Avoiding Explicit Loops

```python
# ❌ NEVER do this in production:
for i in range(m):
    for j in range(n):
        Z[j, i] = sum(W[j, k] * X[k, i] for k in range(d)) + b[j]

# ✅ ALWAYS use matrix operations:
Z = W @ X + b
```

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Single sample** | z = Wx + b, a = g(z) — lowercase, vectors |
| **Batch of m samples** | Z = WA + b, A = g(Z) — uppercase, matrices |
| **X layout** | Each column is one sample: X shape (n_features, m) |
| **Broadcasting** | b shape (n, 1) automatically repeats across m columns |
| **Z^[l] shape** | (n^[l], m) — neurons × samples |
| **Speedup** | 50-1000x faster than explicit loops |
| **NumPy** | `Z = W @ X + b` — single line, fully vectorized |
| **PyTorch convention** | Samples as rows: X shape (m, n_features) — different from theory! |

---

## ❓ Revision Questions

1. **Why is vectorized computation so much faster than loops? What hardware feature makes this possible?**

2. **Given W shape (64, 128) and X shape (128, 32), what is the shape of Z = WX + b? What must be the shape of b?**

3. **Explain broadcasting in the context of Z = WA + b. Why does b need to be shape (n, 1) and not (n,)?**

4. **Write the vectorized forward pass equations for a 3-layer network. Specify all shapes for an architecture of 100→64→32→10 with batch size 256.**

5. **PyTorch uses rows as samples (batch, features) while theory notation uses columns. How would you convert between the two? When does this matter?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Forward Pass Computation](03-forward-pass-computation.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Batch Processing →](05-batch-processing.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
