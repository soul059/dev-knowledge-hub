# Chapter 2: Gradient and Gradient Descent

> **Unit 2 — Calculus for ML** | [← Previous: Derivatives](./01-derivatives-and-partial-derivatives.md) | [Next → Chain Rule](./03-chain-rule.md)

---

## 1. Overview

The **gradient** generalizes the derivative to functions of multiple variables. It is the single most important concept for training machine learning models — gradient descent uses the gradient to iteratively minimize loss functions. This chapter covers the gradient vector, its geometric meaning, and the gradient descent algorithm in detail.

---

## 2. The Gradient Vector

For a function f(x₁, x₂, ..., xₙ), the gradient is a vector of all partial derivatives:

```
         ┌ ∂f/∂x₁ ┐
         │ ∂f/∂x₂ │
∇f(x) =  │   ⋮    │
         │ ∂f/∂xₙ │
         └         ┘
```

**Example:** f(x, y) = x² + 3xy + y²

```
∇f = [ ∂f/∂x ]   =   [ 2x + 3y ]
     [ ∂f/∂y ]       [ 3x + 2y ]
```

At point (1, 2): ∇f(1, 2) = [2(1)+3(2), 3(1)+2(2)] = [8, 7]

---

## 3. Geometric Interpretation

### 3.1 Direction of Steepest Ascent

The gradient vector **∇f** points in the direction of the **steepest increase** of f. Therefore:
- **−∇f** points in the direction of **steepest decrease**
- The **magnitude** ‖∇f‖ tells us **how steep** the slope is
- At a **minimum**, ∇f = 0 (zero vector)

### 3.2 Contour Plot Visualization

The gradient is always **perpendicular** to contour lines:

```
    y
    │    ╭──────╮
    │   ╱ f = 10 ╲        ∇f always points
    │  ╱  ╭────╮  ╲       perpendicular to
    │ ╱  ╱ f=5  ╲  ╲      these contours,
    │╱  ╱ ╭──╮   ╲  ╲     toward higher values
    │  ╱ ╱f=1 ╲   ╲  ╲
    │ ╱ ╱  ·→  ╲   ╲  ╲
    │╱ ╱  min   ╲   ╲  ╲
    └──────────────────── x
              ·→ = gradient direction (away from min)
```

---

## 4. Gradient Descent — Intuition

Imagine you are blindfolded on a hilly landscape and want to reach the lowest valley:

1. **Feel the slope** under your feet (compute the gradient)
2. **Step downhill** in the steepest direction (move opposite to gradient)
3. **Repeat** until the ground feels flat (gradient ≈ 0)

```
  Loss
   │ ·
   │  ·          Step 1: Start here
   │   ╲
   │    · ←───── Step 2: Move downhill
   │     ╲
   │      · ←─── Step 3: Continue
   │       ╲
   │        ·___ Step 4: Reached minimum!
   └──────────── weights
```

---

## 5. The Gradient Descent Algorithm

### 5.1 Update Rule

```
w(t+1) = w(t) − α · ∇L(w(t))
```

Where:
- **w(t)** = current weight vector
- **α** = learning rate (step size)
- **∇L(w(t))** = gradient of loss at current weights
- **w(t+1)** = updated weight vector

### 5.2 Pseudocode

```
ALGORITHM: Gradient Descent
─────────────────────────────────────
Input:  Loss function L(w)
        Initial weights w₀
        Learning rate α
        Convergence threshold ε
        Max iterations T

1.  w ← w₀
2.  FOR t = 1, 2, ..., T:
3.      g ← ∇L(w)              // Compute gradient
4.      IF ‖g‖ < ε:            // Check convergence
5.          RETURN w
6.      w ← w − α · g          // Update weights
7.  RETURN w
```

---

## 6. Learning Rate — The Critical Hyperparameter

The learning rate α controls step size and **dramatically affects** convergence:

```
 Too Small (α = 0.001)     Just Right (α = 0.1)      Too Large (α = 10)
                                    
   · · · · · · ·_              · · ·_                   ·       ·
  Very slow, may             Converges                    ·   ·
  never converge             efficiently                   · ·
  in practice                                            DIVERGES!
```

| Learning Rate   | Behavior                                    |
|-----------------|---------------------------------------------|
| Too small       | Extremely slow convergence, may get stuck    |
| Just right      | Smooth convergence to minimum                |
| Too large       | Oscillates or diverges, loss increases       |
| Adaptive (Adam) | Adjusts per-parameter, generally best        |

---

## 7. Variants of Gradient Descent

| Variant              | Batch Size          | Pros                          | Cons                        |
|----------------------|---------------------|-------------------------------|-----------------------------|
| Batch GD             | Entire dataset      | Stable, exact gradient         | Slow, memory-heavy          |
| Stochastic GD (SGD)  | 1 sample            | Fast, can escape local min     | Noisy, high variance        |
| Mini-batch GD        | 32–512 samples      | Balance of speed & stability   | Requires batch size tuning  |

```
  Loss trajectory:

  Batch GD:       ╲╲╲╲╲___         (smooth descent)
  SGD:            ╱╲╱╲╲╱╲__        (noisy but gets there)
  Mini-batch:     ╲╱╲╲╲╲___        (moderate noise)
```

---

## 8. Step-by-Step Example

**Problem:** Minimize f(x, y) = x² + 2y² using gradient descent.

**Step 1:** Compute gradient
```
∇f = [2x, 4y]
```

**Step 2:** Initialize: x₀ = 4, y₀ = 3, α = 0.1

**Step 3:** Iterate

| Iteration | x       | y       | ∇f          | f(x,y)   |
|-----------|---------|---------|-------------|-----------|
| 0         | 4.000   | 3.000   | [8.0, 12.0] | 34.000   |
| 1         | 3.200   | 1.800   | [6.4, 7.2]  | 16.720   |
| 2         | 2.560   | 1.080   | [5.12, 4.32]| 8.886    |
| 3         | 2.048   | 0.648   | [4.10, 2.59]| 5.034    |
| 4         | 1.638   | 0.389   | [3.28, 1.56]| 2.985    |
| ...       | →0      | →0      | →[0, 0]     | →0       |

The algorithm converges to the minimum at (0, 0) where f = 0.

---

## 9. Python Implementation

```python
import numpy as np
import matplotlib.pyplot as plt

def gradient_descent(gradient_fn, x0, lr=0.1, n_iters=50, tol=1e-8):
    """Generic gradient descent optimizer."""
    x = np.array(x0, dtype=float)
    history = [x.copy()]
    
    for i in range(n_iters):
        grad = np.array(gradient_fn(x))
        if np.linalg.norm(grad) < tol:
            print(f"Converged at iteration {i}")
            break
        x = x - lr * grad
        history.append(x.copy())
    
    return x, np.array(history)

# Minimize f(x, y) = x^2 + 2y^2
def grad_f(w):
    return [2 * w[0], 4 * w[1]]

minimum, path = gradient_descent(grad_f, x0=[4.0, 3.0], lr=0.1, n_iters=100)
print(f"Minimum found at: ({minimum[0]:.6f}, {minimum[1]:.6f})")
print(f"f(min) = {minimum[0]**2 + 2*minimum[1]**2:.8f}")

# Visualize the path on contour plot
fig, ax = plt.subplots(figsize=(8, 6))
xx, yy = np.meshgrid(np.linspace(-5, 5, 100), np.linspace(-4, 4, 100))
zz = xx**2 + 2*yy**2
ax.contour(xx, yy, zz, levels=20, cmap='viridis')
ax.plot(path[:, 0], path[:, 1], 'r.-', markersize=8, label='GD path')
ax.set_xlabel('x'); ax.set_ylabel('y')
ax.set_title('Gradient Descent on f(x,y) = x² + 2y²')
ax.legend()
plt.show()
```

---

## 10. Real-World ML Applications

| Application                | How Gradient Is Used                                      |
|----------------------------|-----------------------------------------------------------|
| Neural Network Training    | Backprop computes ∇L for all weights, then GD updates     |
| Linear Regression          | Closed-form exists, but GD scales to large data           |
| Deep Learning Optimizers   | Adam, RMSprop, Adagrad all build on basic gradient info    |
| Hyperparameter Tuning      | Gradient-based HPO tunes learning rate, regularization     |
| Generative Models (GANs)   | Two competing gradient descents (generator vs discriminator)|

---

## 11. Summary Table

| Concept              | Key Idea                                           | Formula / Rule                       |
|----------------------|----------------------------------------------------|--------------------------------------|
| Gradient             | Vector of all partial derivatives                   | ∇f = [∂f/∂x₁, ..., ∂f/∂xₙ]ᵀ       |
| Direction             | ∇f points toward steepest ascent                   | −∇f = steepest descent              |
| Magnitude             | How steep the slope is                             | ‖∇f‖                                |
| GD Update            | Move opposite to gradient                           | w ← w − α·∇L(w)                    |
| Learning Rate        | Controls step size                                  | Too small=slow, too large=diverge    |
| Batch GD             | Use all data per step                               | Stable but slow                      |
| SGD                  | Use 1 sample per step                               | Fast but noisy                       |
| Mini-batch GD        | Use subset per step                                 | Best trade-off in practice           |

---

## 12. Quick Revision Questions

1. **What is the gradient of f(x, y, z) = xy + yz + xz?** *(Answer: [y+z, x+z, y+x])*
2. **Why do we move in the negative gradient direction during gradient descent?**
3. **What happens if the learning rate is too large?**
4. **How does mini-batch gradient descent differ from stochastic gradient descent?**
5. **At a minimum, what is the value of the gradient vector?**
6. **Why might SGD sometimes outperform batch GD on non-convex loss surfaces?**

---

> [← Previous: Derivatives](./01-derivatives-and-partial-derivatives.md) | [Next → Chain Rule](./03-chain-rule.md)
