# Chapter 2: Stochastic Gradient Descent (SGD)

> **Unit 6 · Optimization · Module 2 of 7**

---

## 2.1 Overview

Stochastic Gradient Descent (SGD) addresses the computational bottleneck of batch gradient descent by updating parameters using **one randomly selected training example** at each step. This dramatically reduces the per-iteration cost and introduces beneficial noise that can help escape local minima.

---

## 2.2 Motivation: Computational Efficiency

| Aspect | Batch GD | SGD |
|--------|----------|-----|
| Gradient computation | All m samples | 1 sample |
| Cost per update | O(m·n) | O(n) |
| Updates per epoch | 1 | m |
| Memory | High (full dataset) | Low (one sample) |

For a dataset with **m = 10 million** samples, batch GD computes 10M gradients before a single update. SGD updates after every single sample — giving **10 million updates per epoch**.

---

## 2.3 SGD Update Rule

At each step, randomly pick one sample `(xⁱ, yⁱ)`:

```
θ_(t+1) = θ_t − α · ∇θ L(f(xⁱ; θ_t), yⁱ)
```

### Algorithm

```
1. Initialize θ
2. For each epoch:
     a. Randomly shuffle the training data
     b. For each sample (xⁱ, yⁱ):
          i.   Compute gradient: g = ∇θ L(xⁱ, yⁱ; θ)
          ii.  Update: θ = θ − α · g
3. Repeat until convergence
4. Return θ
```

---

## 2.4 Noise in the Gradient Estimate

The SGD gradient is an **unbiased estimator** of the true gradient:

```
E[∇θ L(xⁱ, yⁱ; θ)] = ∇θ J(θ)     (expectation over random sample i)
```

But individual estimates have **high variance**:

```
Var[∇θ L(xⁱ)] = σ²    (can be large)
```

This variance introduces noise into the optimization path, which:
- ✅ Can help escape shallow local minima and saddle points
- ✅ Acts as implicit regularization
- ❌ Prevents exact convergence to the minimum (without LR decay)

---

## 2.5 Convergence Behavior

### Oscillations Around the Minimum

SGD does not converge smoothly — it oscillates:

```
  Loss
   |
   | \  /\    /\
   |  \/  \  /  \  /\   /\
   |       \/    \/  \ /  \  /\    Noisy but trending down
   |                  ·    \/  \_____
   |                               \____
   +———————————————————————————————————————→ Iterations
                SGD convergence path
```

Compare with batch GD:

```
  Loss
   |
   | \
   |   \
   |     \___
   |          \____
   |               \_________
   +———————————————————————————————————————→ Iterations
             Batch GD convergence path
```

---

## 2.6 Convergence Paths (ASCII Contour Comparison)

```
  Batch Gradient Descent          Stochastic Gradient Descent

  θ₂ ↑                            θ₂ ↑
     |    ╭─────────╮                |    ╭─────────╮
     |  ╭─┤  ╭───╮  ├─╮             |  ╭─┤  ╭───╮  ├─╮
     |  │ │  │ * │  │ │             |  │ │  │ * │  │ │
     |  ╰─┤  ╰───╯  ├─╯             |  ╰─┤  ╰───╯  ├─╯
     | x·→╰─────────╯               | x╰─────────╯
     |   ·→·→*                       |  ↘↗↘↗↘→↗↘*
     +——————————→ θ₁                 +——————————→ θ₁

     Smooth, direct path             Noisy, zigzag path
```

---

## 2.7 Learning Rate Decay Necessity

With a **fixed** learning rate, SGD oscillates around the minimum and never truly converges. The solution is **learning rate decay**:

```
α_t = α₀ / (1 + decay_rate · t)

or

α_t = α₀ · e^(−k·t)
```

### Robbins-Monro Conditions

For guaranteed convergence, the learning rate schedule must satisfy:

```
Σ_t  α_t  = ∞       (reach the minimum from any start)
Σ_t  α_t² < ∞       (variance shrinks to zero)
```

Example: `α_t = c / t` satisfies both conditions.

---

## 2.8 Step-by-Step Example

**Problem**: SGD for linear regression with data: (1,2), (2,4), (3,5), (4,4)

**Setup**: `y = θ₀ + θ₁x`, MSE loss, α = 0.01, init θ = [0, 0]

**Epoch 1** (shuffled order: sample 3, 1, 4, 2):

| Step | Sample (x,y) | Prediction | Error | ∂L/∂θ₀ | ∂L/∂θ₁ | θ₀ | θ₁ |
|------|-------------|------------|-------|---------|---------|------|------|
| 0 | — | — | — | — | — | 0.000 | 0.000 |
| 1 | (3, 5) | 0.000 | −5.00 | −5.00 | −15.00 | 0.050 | 0.150 |
| 2 | (1, 2) | 0.200 | −1.80 | −1.80 | −1.80 | 0.068 | 0.168 |
| 3 | (4, 4) | 0.740 | −3.26 | −3.26 | −13.04 | 0.101 | 0.298 |
| 4 | (2, 4) | 0.697 | −3.30 | −3.30 | −6.61 | 0.134 | 0.364 |

Note how each update uses only **one** sample — fast but noisy.

---

## 2.9 Python Implementation

```python
import numpy as np

def sgd(X, y, lr=0.01, n_epochs=50, tol=1e-6):
    """
    Stochastic Gradient Descent for linear regression.
    X: (m, n) feature matrix with bias column
    y: (m,) target vector
    """
    m, n = X.shape
    theta = np.zeros(n)
    history = []

    for epoch in range(n_epochs):
        # Shuffle data each epoch
        indices = np.random.permutation(m)

        for i in indices:
            xi = X[i]
            yi = y[i]

            # Prediction and error
            y_hat = xi @ theta
            error = y_hat - yi

            # Gradient (single sample)
            gradient = xi * error

            # Update
            theta = theta - lr * gradient

        # Track cost at end of each epoch
        cost = (1 / (2 * m)) * np.sum((X @ theta - y) ** 2)
        history.append(cost)

    return theta, history


# Example
X_raw = np.array([1, 2, 3, 4])
y = np.array([2, 4, 5, 4])
X = np.column_stack([np.ones(len(X_raw)), X_raw])

np.random.seed(42)
theta, history = sgd(X, y, lr=0.01, n_epochs=200)
print(f"θ₀ = {theta[0]:.4f}, θ₁ = {theta[1]:.4f}")
```

### SGD with Learning Rate Decay

```python
def sgd_with_decay(X, y, lr0=0.1, decay=0.001, n_epochs=100):
    m, n = X.shape
    theta = np.zeros(n)
    step = 0

    for epoch in range(n_epochs):
        indices = np.random.permutation(m)
        for i in indices:
            lr = lr0 / (1 + decay * step)  # Decaying LR
            gradient = X[i] * (X[i] @ theta - y[i])
            theta = theta - lr * gradient
            step += 1

    return theta
```

---

## 2.10 When to Use SGD

| ✅ Use SGD When | ❌ Avoid SGD When |
|----------------|------------------|
| Dataset is very large (> 100K samples) | Dataset is small (< 1K samples) |
| Need fast approximate solutions | Need exact, deterministic convergence |
| Want implicit regularization | Cost function is very noisy already |
| Online learning (streaming data) | High-precision solution required |
| Training deep neural networks | Problem is low-dimensional and convex |

---

## 2.11 Real-World ML Applications

| Application | Why SGD? |
|------------|----------|
| **Deep Learning** | Training on millions of images (ImageNet) — batch GD infeasible |
| **NLP (Transformers)** | Corpus sizes in billions of tokens |
| **Online Learning** | Recommendations, ad-click prediction — data arrives in streams |
| **Large-scale Logistic Regression** | Web-scale classification (spam, fraud) |
| **Reinforcement Learning** | Policy gradient methods use stochastic updates |

---

## 2.12 Key Theoretical Results

| Property | Detail |
|----------|--------|
| Convergence rate (convex) | O(1/√T) with proper LR decay |
| Convergence rate (strongly convex) | O(1/T) |
| Gradient estimate | Unbiased: E[gᵢ] = ∇J(θ) |
| Variance | Does not vanish without LR decay |
| Per-iteration cost | O(n) vs O(mn) for batch |

---

## 2.13 Summary Table

| Concept | Key Detail |
|---------|-----------|
| Update rule | `θ = θ − α · ∇L(xⁱ, yⁱ; θ)` (one sample) |
| Gradient | Noisy, unbiased estimate of true gradient |
| Convergence | Oscillatory; needs LR decay for true convergence |
| Cost per step | O(n) — independent of dataset size |
| Updates per epoch | m (one per sample) |
| Key benefit | Scales to massive datasets |
| Key drawback | Noisy updates, doesn't converge without LR decay |

---

## 2.14 Quick Revision Questions

1. **Why is the SGD gradient called an "unbiased estimator"?**
   → Because E[∇L(xⁱ; θ)] = ∇J(θ); on average it equals the true gradient.

2. **What are the Robbins-Monro conditions for SGD convergence?**
   → Σαₜ = ∞ (can reach minimum) and Σαₜ² < ∞ (variance vanishes).

3. **Why does SGD oscillate near the minimum?**
   → Because the gradient estimated from one sample has high variance.

4. **How does SGD help escape local minima?**
   → The gradient noise can push parameters out of shallow local minima.

5. **What is the per-iteration computational complexity of SGD?**
   → O(n) where n is the number of features.

6. **Why must we shuffle data each epoch in SGD?**
   → To ensure the gradient estimates are independent and unbiased; ordered data can introduce bias.

---

| [← Previous: Gradient Descent](01-gradient-descent.md) | [Next Chapter: Mini-Batch Gradient Descent →](03-mini-batch-gradient-descent.md) |
|:---|---:|
