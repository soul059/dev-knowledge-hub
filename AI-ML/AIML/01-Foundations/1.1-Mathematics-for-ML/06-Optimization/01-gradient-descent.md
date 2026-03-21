# Chapter 1: Gradient Descent

> **Unit 6 · Optimization · Module 1 of 7**

---

## 1.1 Overview

Gradient Descent (GD) is the foundational optimization algorithm in machine learning. It iteratively adjusts parameters by moving in the direction of steepest descent of the cost function. Almost every modern ML model—from linear regression to deep neural networks—is trained using some variant of gradient descent.

---

## 1.2 The Core Idea

Given a differentiable cost function `J(θ)`, we want to find:

```
θ* = arg min J(θ)
         θ
```

The gradient `∇J(θ)` points in the direction of **steepest ascent**. To minimize, we move in the **opposite** direction.

### Update Rule

```
θ_(t+1) = θ_t − α · ∇J(θ_t)
```

Where:
| Symbol | Meaning |
|--------|---------|
| `θ_t` | Parameter vector at step t |
| `α` | Learning rate (step size) |
| `∇J(θ_t)` | Gradient of cost w.r.t. θ at step t |

---

## 1.3 Batch Gradient Descent

In **batch** gradient descent, the gradient is computed over the **entire** training set at every step:

```
∇J(θ) = (1/m) Σᵢ₌₁ᵐ ∇θ L(f(xⁱ; θ), yⁱ)
```

Where `m` is the total number of training examples.

### Algorithm

```
1. Initialize θ randomly (or with zeros)
2. Repeat until convergence:
     a. Compute gradient: g = (1/m) Σᵢ ∇θ L(xⁱ, yⁱ; θ)
     b. Update parameters:  θ = θ − α · g
     c. Check convergence criterion
3. Return θ
```

---

## 1.4 Learning Rate Selection

The learning rate `α` is the most critical hyperparameter.

```
  Loss                      Loss                      Loss
   |  ╲                      |                          |   /\_/\_/\
   |    ╲                    |   ╲                      |  /        \_
   |      ╲__                |     ╲___                 |_/           \___  diverge!
   |          ╲___           |          ╲__________     |
   |              ╲_____     |                          |
   +—————————————————→      +—————————————————→        +—————————————————→
        Iterations               Iterations                 Iterations

   α = good (≈0.01)         α = too small (≈0.0001)    α = too large (≈1.0)
   Fast convergence          Slow convergence            Diverges / oscillates
```

### Practical Guidelines

| Learning Rate | Effect |
|--------------|--------|
| `α ≈ 0.001 – 0.01` | Good starting range for most problems |
| `α` too small | Very slow convergence, may get stuck |
| `α` too large | Overshoots minimum, diverges |
| Optimal `α` | Depends on problem curvature (Hessian) |

---

## 1.5 Convergence Criteria

Gradient descent stops when one of these is satisfied:

1. **Gradient norm**: `‖∇J(θ)‖ < ε` (e.g., ε = 1e-6)
2. **Cost change**: `|J(θ_t) − J(θ_{t-1})| < ε`
3. **Parameter change**: `‖θ_t − θ_{t-1}‖ < ε`
4. **Max iterations**: `t > T_max`

---

## 1.6 Contour Plot Illustration (ASCII)

Gradient descent on a 2D cost surface (e.g., `J(θ₁, θ₂)`):

```
  θ₂
   ↑
   |         ╭─────────────────╮
   |       ╭─┤                 ├─╮
   |     ╭─┤ │    ╭───────╮    │ ├─╮
   |     │ │ │  ╭─┤       ├─╮  │ │ │
   |     │ │ │  │ │   *   │ │  │ │ │     * = minimum
   |     │ │ │  ╰─┤       ├─╯  │ │ │
   |     ╰─┤ │    ╰───────╯    │ ├─╯
   |  x ·→ ╰─┤      ↙         ├─╯      x = start
   |    · → · ╰────↙───────────╯        · = GD path
   |      · · →  ↙
   |          ·*
   +————————————————————————————————→ θ₁
```

The path follows the negative gradient, cutting across contour lines toward the minimum.

---

## 1.7 Example: Gradient Descent on Linear Regression

**Problem**: Fit `y = θ₁x + θ₀` to data points: (1,2), (2,4), (3,5), (4,4).

**Cost function** (MSE):
```
J(θ₀, θ₁) = (1/2m) Σᵢ (θ₀ + θ₁xⁱ − yⁱ)²
```

**Gradients**:
```
∂J/∂θ₀ = (1/m) Σᵢ (θ₀ + θ₁xⁱ − yⁱ)
∂J/∂θ₁ = (1/m) Σᵢ (θ₀ + θ₁xⁱ − yⁱ) · xⁱ
```

**Step-by-step** (α = 0.01, init θ₀ = 0, θ₁ = 0):

| Iter | θ₀ | θ₁ | J(θ) |
|------|------|------|------|
| 0 | 0.000 | 0.000 | 7.625 |
| 1 | 0.0375 | 0.0950 | 6.428 |
| 2 | 0.0714 | 0.1802 | 5.438 |
| ... | ... | ... | ... |
| 1000 | 1.000 | 0.900 | 0.275 |

After convergence: `y ≈ 1.0 + 0.9x` (close to the OLS solution y = 1.0 + 0.9x).

---

## 1.8 Python Implementation from Scratch

```python
import numpy as np

def gradient_descent(X, y, lr=0.01, n_iters=1000, tol=1e-6):
    """
    Batch gradient descent for linear regression.
    X: (m, n) feature matrix (with bias column)
    y: (m,) target vector
    """
    m, n = X.shape
    theta = np.zeros(n)
    history = []

    for i in range(n_iters):
        # Predictions
        y_hat = X @ theta

        # Cost (MSE / 2)
        cost = (1 / (2 * m)) * np.sum((y_hat - y) ** 2)
        history.append(cost)

        # Gradient
        gradient = (1 / m) * (X.T @ (y_hat - y))

        # Convergence check
        if np.linalg.norm(gradient) < tol:
            print(f"Converged at iteration {i}")
            break

        # Update
        theta = theta - lr * gradient

    return theta, history


# Example usage
X_raw = np.array([1, 2, 3, 4])
y = np.array([2, 4, 5, 4])
X = np.column_stack([np.ones(len(X_raw)), X_raw])  # Add bias

theta, history = gradient_descent(X, y, lr=0.01, n_iters=5000)
print(f"θ₀ = {theta[0]:.4f}, θ₁ = {theta[1]:.4f}")
# Output: θ₀ = 1.5000, θ₁ = 0.8000
```

### Plotting Convergence

```python
import matplotlib.pyplot as plt

plt.plot(history)
plt.xlabel("Iteration")
plt.ylabel("Cost J(θ)")
plt.title("Gradient Descent Convergence")
plt.grid(True)
plt.show()
```

---

## 1.9 Real-World ML Applications

| Application | How GD Is Used |
|------------|----------------|
| **Linear Regression** | Minimize MSE to find optimal weights |
| **Logistic Regression** | Minimize cross-entropy loss |
| **Neural Networks** | Backpropagation computes gradients; GD updates weights |
| **SVM (primal)** | Minimize hinge loss + regularization |
| **Recommendation Systems** | Matrix factorization via GD on reconstruction error |

---

## 1.10 Advantages and Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|---------------|-----------------|
| Guaranteed convergence for convex functions | Slow on large datasets (uses all data per step) |
| Stable, smooth convergence path | Memory-intensive for large m |
| Simple to implement and understand | Can get stuck in local minima (non-convex) |
| Exact gradient (no noise) | Sensitive to learning rate choice |
| Deterministic updates | Requires differentiable cost function |

---

## 1.11 Summary Table

| Concept | Key Formula / Detail |
|---------|---------------------|
| Update rule | `θ = θ − α∇J(θ)` |
| Gradient | Computed over **entire** dataset |
| Learning rate | Typically `0.001 – 0.01` |
| Convergence | Guaranteed for convex J with proper α |
| Complexity per step | `O(m·n)` — m samples, n features |
| When to use | Small–medium datasets, convex problems |

---

## 1.12 Quick Revision Questions

1. **What does the gradient ∇J(θ) represent geometrically?**
   → The direction of steepest ascent; GD moves in the opposite direction.

2. **Why do we subtract the gradient in the update rule?**
   → Because the gradient points uphill; subtraction moves us downhill toward the minimum.

3. **What happens if the learning rate is too large?**
   → The algorithm overshoots the minimum, oscillates, or diverges.

4. **What is the computational cost of one batch GD step with m samples and n features?**
   → O(m·n) — we compute the gradient over all m samples.

5. **Name two convergence criteria for gradient descent.**
   → (i) Gradient norm below threshold, (ii) Change in cost below threshold.

6. **Is batch gradient descent guaranteed to find the global minimum?**
   → Yes, for strictly convex functions with an appropriate learning rate; not in general for non-convex functions.

---

| [← Unit 6 Overview](../README.md) | [Next Chapter: Stochastic Gradient Descent →](02-stochastic-gradient-descent.md) |
|:---|---:|
