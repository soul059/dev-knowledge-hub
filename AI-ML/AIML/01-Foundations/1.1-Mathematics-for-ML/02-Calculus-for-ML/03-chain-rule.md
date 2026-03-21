# Chapter 3: The Chain Rule and Backpropagation

> **Unit 2 — Calculus for ML** | [← Previous: Gradient Descent](./02-gradient-and-gradient-descent.md) | [Next → Jacobian & Hessian](./04-jacobian-and-hessian.md)

---

## 1. Overview

The chain rule is the **mathematical engine behind backpropagation** — the algorithm that trains every modern neural network. It tells us how to compute the derivative of **composed functions**, breaking a complex derivative into a product of simpler ones. Understanding the chain rule deeply means understanding how neural networks learn.

---

## 2. Single-Variable Chain Rule

If y = f(g(x)), i.e., y depends on u = g(x) which depends on x:

```
dy/dx = (dy/du) · (du/dx)
```

**Think of it as a chain:** x → u → y, and we multiply the derivatives along the chain.

### Example

Find dy/dx where y = (2x + 3)⁵

Let u = 2x + 3, so y = u⁵
```
dy/du = 5u⁴ = 5(2x + 3)⁴
du/dx = 2

dy/dx = 5(2x + 3)⁴ · 2 = 10(2x + 3)⁴
```

---

## 3. Multivariable Chain Rule

If z = f(x, y) where x = x(t) and y = y(t):

```
dz/dt = (∂z/∂x)·(dx/dt) + (∂z/∂y)·(dy/dt)
```

More generally, if f depends on variables that themselves depend on other variables, we **sum over all paths** from input to output.

### General Form

If L = L(u₁, u₂, ..., uₖ) and each uᵢ = uᵢ(w):

```
∂L/∂w = Σᵢ (∂L/∂uᵢ) · (∂uᵢ/∂w)
```

> **Key insight:** Sum over all paths from w to L in the computational graph.

---

## 4. Computational Graphs

A computational graph represents a function as a directed acyclic graph (DAG) where:
- **Nodes** = variables or operations
- **Edges** = data flow (which values feed into which operations)

### Example: f(x, y) = (x + y) · (x − 3)

```
    x ──┬──→ [+] ──→ a = x + y ──┐
        │                         ├──→ [×] ──→ f = a · b
    y ──┘                         │
                                  │
    x ──────→ [−3] ──→ b = x − 3 ┘
```

Each node computes a **local derivative**, and the chain rule chains them together.

---

## 5. Forward Pass vs Backward Pass

### 5.1 Forward Pass

Compute function value from **inputs to output**, layer by layer:

```
Input x  →  Hidden layer  →  Output  →  Loss L
   2     →    z = wx + b   →  ŷ = σ(z) → L = (y − ŷ)²
```

### 5.2 Backward Pass (Backpropagation)

Compute gradients from **output back to inputs**, using the chain rule at each step:

```
∂L/∂w = (∂L/∂ŷ) · (∂ŷ/∂z) · (∂z/∂w)
  ↑        ↑          ↑          ↑
  goal   from loss   sigmoid   from linear
                     derivative  layer
```

```
  FORWARD:   x ──→ z ──→ ŷ ──→ L
                                 │
  BACKWARD:  ∂L/∂w ←── ∂L/∂z ←── ∂L/∂ŷ ←─┘
             (chain rule multiplies along the path)
```

---

## 6. Step-by-Step: Backprop Through a Small Network

### Network Setup

```
  x₁ ──w₁──→ ╭──╮         ╭──╮
              │z₁│──σ──→a₁─│  │
  x₂ ──w₂──→ ╰──╯         │z₃│──σ──→ ŷ ──→ L
              ╭──╮         │  │
  x₁ ──w₃──→ │z₂│──σ──→a₂─╰──╯
  x₂ ──w₄──→ ╰──╯    w₅↗  w₆↗
```

**Forward pass equations:**
```
z₁ = w₁·x₁ + w₂·x₂           (linear combination)
a₁ = σ(z₁)                    (activation)
z₂ = w₃·x₁ + w₄·x₂
a₂ = σ(z₂)
z₃ = w₅·a₁ + w₆·a₂
ŷ  = σ(z₃)
L  = ½(y − ŷ)²                (loss)
```

### Backward Pass — Computing ∂L/∂w₁

**Step 1:** ∂L/∂ŷ
```
∂L/∂ŷ = −(y − ŷ)
```

**Step 2:** ∂ŷ/∂z₃
```
∂ŷ/∂z₃ = σ'(z₃) = ŷ(1 − ŷ)
```

**Step 3:** ∂z₃/∂a₁
```
∂z₃/∂a₁ = w₅
```

**Step 4:** ∂a₁/∂z₁
```
∂a₁/∂z₁ = σ'(z₁) = a₁(1 − a₁)
```

**Step 5:** ∂z₁/∂w₁
```
∂z₁/∂w₁ = x₁
```

**Chain them all together:**
```
∂L/∂w₁ = (∂L/∂ŷ) · (∂ŷ/∂z₃) · (∂z₃/∂a₁) · (∂a₁/∂z₁) · (∂z₁/∂w₁)
        = −(y − ŷ) · ŷ(1−ŷ) · w₅ · a₁(1−a₁) · x₁
```

> This is **exactly** what backpropagation computes — automatically and efficiently.

---

## 7. Numerical Example

Let x₁ = 1, x₂ = 0.5, y = 1 (target), and initial weights all = 0.5.

**Forward pass:**
```
z₁ = 0.5(1) + 0.5(0.5) = 0.75      a₁ = σ(0.75) = 0.6792
z₂ = 0.5(1) + 0.5(0.5) = 0.75      a₂ = σ(0.75) = 0.6792
z₃ = 0.5(0.6792) + 0.5(0.6792) = 0.6792    ŷ = σ(0.6792) = 0.6637
L  = 0.5(1 − 0.6637)² = 0.0565
```

**Backward pass for ∂L/∂w₁:**
```
∂L/∂ŷ   = −(1 − 0.6637) = −0.3363
∂ŷ/∂z₃  = 0.6637(1 − 0.6637) = 0.2232
∂z₃/∂a₁ = 0.5 (= w₅)
∂a₁/∂z₁ = 0.6792(1 − 0.6792) = 0.2179
∂z₁/∂w₁ = 1.0 (= x₁)

∂L/∂w₁ = (−0.3363)(0.2232)(0.5)(0.2179)(1.0) = −0.00818
```

**Weight update (α = 0.5):**
```
w₁_new = 0.5 − 0.5 × (−0.00818) = 0.5041
```

---

## 8. Python Implementation

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

def sigmoid_derivative(a):
    return a * (1 - a)

# Simple 2-layer network: 2 inputs → 2 hidden → 1 output
np.random.seed(42)
x = np.array([1.0, 0.5])
y_true = 1.0

# Weights
W1 = np.array([[0.5, 0.5],    # hidden neuron 1
                [0.5, 0.5]])   # hidden neuron 2
W2 = np.array([0.5, 0.5])     # output neuron
lr = 0.5

for epoch in range(5):
    # --- Forward Pass ---
    z1 = W1 @ x           # (2,)
    a1 = sigmoid(z1)       # (2,)
    z2 = W2 @ a1           # scalar
    y_hat = sigmoid(z2)    # scalar
    loss = 0.5 * (y_true - y_hat) ** 2

    # --- Backward Pass (Chain Rule) ---
    dL_dy    = -(y_true - y_hat)                   # ∂L/∂ŷ
    dy_dz2   = sigmoid_derivative(y_hat)            # ∂ŷ/∂z₂
    delta2   = dL_dy * dy_dz2                       # ∂L/∂z₂

    dL_dW2   = delta2 * a1                          # ∂L/∂W₂
    dL_da1   = delta2 * W2                          # ∂L/∂a₁
    da1_dz1  = sigmoid_derivative(a1)               # ∂a₁/∂z₁
    delta1   = dL_da1 * da1_dz1                     # ∂L/∂z₁
    dL_dW1   = np.outer(delta1, x)                  # ∂L/∂W₁

    # --- Update Weights ---
    W2 -= lr * dL_dW2
    W1 -= lr * dL_dW1

    print(f"Epoch {epoch+1}: loss = {loss:.6f}, ŷ = {y_hat:.6f}")
```

---

## 9. Why the Chain Rule Is Efficient

Without the chain rule, we'd need to compute **numerical derivatives** for every weight independently → O(n) forward passes for n weights.

With the chain rule (backpropagation):
- **One forward pass** to compute the output
- **One backward pass** to compute ALL gradients simultaneously
- Complexity: O(n) — same as a single forward pass

| Method                    | Forward Passes | Backward Passes | Total Cost  |
|---------------------------|----------------|-----------------|-------------|
| Numerical (per weight)    | 2n             | 0               | O(n²)       |
| Symbolic (manual)         | 1              | 0               | Intractable |
| **Backprop (chain rule)** | **1**          | **1**           | **O(n)**    |

---

## 10. Real-World ML Applications

| Application               | Chain Rule Role                                          |
|----------------------------|----------------------------------------------------------|
| Deep Neural Networks       | Backprop through 100+ layers via chain rule               |
| CNNs                       | Chain rule through convolution, pooling, activation       |
| RNNs / LSTMs              | Chain rule through time steps (backprop through time)     |
| Transformers               | Chain rule through attention, layer norm, feedforward     |
| Automatic Differentiation  | Libraries (PyTorch, TensorFlow) automate chain rule       |

---

## 11. Summary Table

| Concept                  | Key Idea                                    | Formula                                      |
|--------------------------|---------------------------------------------|----------------------------------------------|
| Single-var chain rule    | Derivative of composition                    | dy/dx = (dy/du)·(du/dx)                     |
| Multi-var chain rule     | Sum over all paths                           | ∂L/∂w = Σ (∂L/∂uᵢ)·(∂uᵢ/∂w)              |
| Computational graph      | DAG of operations                            | Nodes = ops, edges = data flow               |
| Forward pass             | Input → Output, compute values               | Layer by layer evaluation                    |
| Backward pass            | Output → Input, compute gradients            | Reverse-mode autodiff                        |
| Backpropagation          | Chain rule applied recursively               | Efficient gradient computation for all w     |

---

## 12. Quick Revision Questions

1. **State the single-variable chain rule** and give an example.
2. **Why does the multivariable chain rule involve a sum?** *(Hint: multiple paths)*
3. **In backpropagation, why do we traverse from output to input (not input to output)?**
4. **What is the computational complexity advantage of backprop over numerical differentiation?**
5. **If a network has 5 layers, how many local derivatives are multiplied to get ∂L/∂w₁ (first layer weight)?**
6. **What happens to gradients if many sigmoid derivatives (all < 0.25) are multiplied together?** *(Hint: vanishing gradient problem)*

---

> [← Previous: Gradient Descent](./02-gradient-and-gradient-descent.md) | [Next → Jacobian & Hessian](./04-jacobian-and-hessian.md)
