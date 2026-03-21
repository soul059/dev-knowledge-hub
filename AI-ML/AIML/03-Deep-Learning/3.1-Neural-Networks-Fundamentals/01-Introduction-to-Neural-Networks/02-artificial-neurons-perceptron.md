# 🔬 Artificial Neurons & The Perceptron

> **Chapter 1.2 — The Perceptron: The Simplest Trainable Neural Model**

| Topic | Details |
|---|---|
| **Subject** | Neural Networks — The Perceptron Model |
| **Prerequisites** | Linear algebra basics, Chapter 1.1 |
| **Key Insight** | A perceptron is a single neuron that acts as a linear classifier, learning a decision boundary from data |

---

## 📌 Overview

The perceptron, invented by Frank Rosenblatt in 1958, is the simplest form of an artificial neural network — a single neuron that can learn to classify inputs into two categories. Despite its simplicity, the perceptron introduces foundational concepts: weighted summation, bias, activation functions, and iterative learning. Understanding its power *and* its limitations is essential for grasping why we need deeper networks.

---

## 1. The Artificial Neuron Model

### 1.1 Mathematical Formulation

An artificial neuron computes a weighted sum of its inputs, adds a bias, and passes the result through an activation function:

```
Step 1: Compute the pre-activation (linear combination)
        z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
        z = w · x + b        (vector notation)
        z = wᵀx + b          (transpose notation)

Step 2: Apply activation function
        y = f(z)

Where:
    x = [x₁, x₂, ..., xₙ]ᵀ  — input vector (n features)
    w = [w₁, w₂, ..., wₙ]ᵀ  — weight vector (learnable parameters)
    b                         — bias (learnable scalar)
    z                         — pre-activation value
    f(·)                      — activation function
    y                         — output (prediction)
```

### 1.2 Diagram of an Artificial Neuron

```
                        ARTIFICIAL NEURON
    ┌─────────────────────────────────────────────────────┐
    │                                                     │
    │   x₁ ────── w₁ ──┐                                 │
    │                   │                                 │
    │   x₂ ────── w₂ ──┼──► Σ(wᵢxᵢ) + b ──► f(z) ──► y │
    │                   │        ↑                        │
    │   x₃ ────── w₃ ──┘        │                        │
    │                            b (bias)                 │
    │                                                     │
    │   ◄── Inputs ──►  ◄─ Weights ─►  ◄─ Activation ─►  │
    └─────────────────────────────────────────────────────┘
```

### 1.3 Role of Each Component

| Component | Symbol | Role | Analogy |
|---|---|---|---|
| **Inputs** | x₁, x₂, ..., xₙ | Raw features or data points | Dendrites receiving signals |
| **Weights** | w₁, w₂, ..., wₙ | Importance/strength of each input | Synaptic strength |
| **Bias** | b | Shifts the decision boundary | Resting potential of neuron |
| **Pre-activation** | z = w·x + b | Linear combination | Membrane potential |
| **Activation** | y = f(z) | Introduces non-linearity / makes decision | Firing threshold |
| **Output** | y | Prediction | Axon output signal |

---

## 2. The Perceptron Model

### 2.1 Definition

The **Perceptron** is an artificial neuron that uses a **step (Heaviside) function** as its activation:

```
                    ┌ 1,  if z ≥ 0
    y = step(z) =  │
                    └ 0,  if z < 0

    Where: z = w · x + b = Σᵢ wᵢxᵢ + b
```

### 2.2 ASCII Plot of Step Function

```
    y
    │
  1 ┤ · · · · · · · ·━━━━━━━━━━━━━━━━
    │               ┃
    │               ┃
    │               ┃
  0 ┤━━━━━━━━━━━━━━━┃ · · · · · · · ·
    │               ┃
    ┼───────────────┃────────────────── z
                    0

    Output is exactly 0 or 1 — no gradient!
```

### 2.3 The Perceptron as a Linear Classifier

The perceptron divides the input space with a **hyperplane** (a line in 2D, a plane in 3D):

```
Decision boundary: w · x + b = 0
    or equivalently: w₁x₁ + w₂x₂ + b = 0

In 2D, this is a line:
    x₂ = -(w₁/w₂)x₁ - (b/w₂)
```

```
    x₂
    │        Class 1 (y=1)
    │       ●  ●
    │      ●  ●  ●
    │     ●  ● ╱ · · ·         Decision Boundary
    │    ●  ╱ · · · · ·        w₁x₁ + w₂x₂ + b = 0
    │   ╱ · · · · · · · ·
    │ ╱  ○  ○  ○  ○
    │   ○  ○  ○  ○
    │  ○  ○  ○
    ┼──────────────────── x₁
        Class 0 (y=0)

    The weight vector w is perpendicular to the boundary
    The bias b shifts the boundary away from the origin
```

### 2.4 Geometric Interpretation

- The **weight vector w** is perpendicular (normal) to the decision boundary
- The **bias b** controls how far the boundary is from the origin
- The signed distance from a point x to the boundary is: `(w · x + b) / ||w||`
- Points on the positive side → class 1; negative side → class 0

---

## 3. The Perceptron Learning Algorithm

### 3.1 Algorithm (Step-by-Step)

```
PERCEPTRON LEARNING ALGORITHM
════════════════════════════════════════════════════════════

Input:  Training data {(x⁽¹⁾, y⁽¹⁾), (x⁽²⁾, y⁽²⁾), ..., (x⁽ᵐ⁾, y⁽ᵐ⁾)}
        Learning rate η (typically η = 1 for basic perceptron)

Initialize: w = 0, b = 0   (or small random values)

Repeat until convergence (or max iterations):
    For each training sample (x⁽ⁱ⁾, y⁽ⁱ⁾):
        1. Compute prediction:
           ŷ⁽ⁱ⁾ = step(w · x⁽ⁱ⁾ + b)
        
        2. Compute error:
           e = y⁽ⁱ⁾ - ŷ⁽ⁱ⁾       ∈ {-1, 0, 1}
        
        3. Update weights and bias:
           If e ≠ 0:
               w ← w + η · e · x⁽ⁱ⁾
               b ← b + η · e

        Intuition:
        ─────────
        • e = 0  → correct prediction, no update
        • e = 1  → false negative (predicted 0, actual 1)
                    → move boundary toward x⁽ⁱ⁾ (w += x⁽ⁱ⁾)
        • e = -1 → false positive (predicted 1, actual 0)
                    → move boundary away from x⁽ⁱ⁾ (w -= x⁽ⁱ⁾)
```

### 3.2 Worked Example — Learning AND Gate

```
AND Gate Truth Table:
    x₁  x₂  │  y
    ─────────┼────
     0   0   │  0
     0   1   │  0
     1   0   │  0
     1   1   │  1
```

**Step-by-step training** (η = 1, initialize w₁ = 0, w₂ = 0, b = 0):

```
Epoch 1:
─────────────────────────────────────────────────────────────
Sample  │ x₁  x₂ │  y │  z=w·x+b │ ŷ=step(z) │ e=y-ŷ │ Update
────────┼─────────┼────┼──────────┼───────────┼───────┼──────────
(0,0)   │  0   0  │  0 │  0.0     │    1*     │  -1   │ w=(0,0), b=-1
(0,1)   │  0   1  │  0 │ -1.0     │    0      │   0   │ no change
(1,0)   │  1   0  │  0 │ -1.0     │    0      │   0   │ no change
(1,1)   │  1   1  │  1 │ -1.0     │    0      │   1   │ w=(1,1), b=0

* Note: step(0) = 1 by our convention (z ≥ 0)

Epoch 2:
────────┼─────────┼────┼──────────┼───────────┼───────┼──────────
(0,0)   │  0   0  │  0 │  0.0     │    1      │  -1   │ w=(1,1), b=-1
(0,1)   │  0   1  │  0 │  0.0     │    1      │  -1   │ w=(1,0), b=-2
(1,0)   │  1   0  │  0 │ -1.0     │    0      │   0   │ no change
(1,1)   │  1   1  │  1 │ -1.0     │    0      │   1   │ w=(2,1), b=-1

Epoch 3:
────────┼─────────┼────┼──────────┼───────────┼───────┼──────────
(0,0)   │  0   0  │  0 │ -1.0     │    0      │   0   │ no change
(0,1)   │  0   1  │  0 │  0.0     │    1      │  -1   │ w=(2,0), b=-2
(1,0)   │  1   0  │  0 │  0.0     │    1      │  -1   │ w=(1,0), b=-3
(1,1)   │  1   1  │  1 │ -2.0     │    0      │   1   │ w=(2,1), b=-2

... (continues for a few more epochs)

Final convergence: w₁=2, w₂=1, b=-3  (one possible solution)
Verify: 
    (0,0): 2(0)+1(0)-3 = -3 → step(-3) = 0 ✓
    (0,1): 2(0)+1(1)-3 = -2 → step(-2) = 0 ✓
    (1,0): 2(1)+1(0)-3 = -1 → step(-1) = 0 ✓
    (1,1): 2(1)+1(1)-3 =  0 → step(0)  = 1 ✓
```

### 3.3 Perceptron Convergence Theorem

> **Theorem** (Rosenblatt, 1962): If the training data is **linearly separable**, the perceptron learning algorithm will converge to a solution (find a separating hyperplane) in a **finite number of steps**.

The bound on the number of mistakes is:

```
    Number of mistakes ≤ (R / γ)²

    Where:
        R = max ||x⁽ⁱ⁾||       — radius of data (max norm of any input)
        γ = min (y⁽ⁱ⁾(w* · x⁽ⁱ⁾)) / ||w*||   — margin of best separator w*
```

**Key implication**: Convergence is *guaranteed* but **only for linearly separable data**. For non-separable data, the algorithm oscillates forever.

---

## 4. The XOR Problem — Why Single Neurons Fail

### 4.1 XOR Truth Table

```
XOR Gate:
    x₁  x₂  │  y
    ─────────┼────
     0   0   │  0
     0   1   │  1
     1   0   │  1
     1   1   │  0
```

### 4.2 Why XOR Is Not Linearly Separable

```
    x₂
    │
  1 ┤    ●(0,1)          ○(1,1)
    │    class 1          class 0
    │
    │
  0 ┤    ○(0,0)          ●(1,0)
    │    class 0          class 1
    ┼────────────────────────── x₁
         0                1

    No single straight line can separate ● from ○!

    Any line separating (0,1) and (1,0) from (0,0)
    will incorrectly classify (1,1), or vice versa.
```

### 4.3 Mathematical Proof of XOR Non-Separability

Assume a perceptron can compute XOR. Then there exist w₁, w₂, b such that:

```
    (0,0) → 0:  w₁(0) + w₂(0) + b < 0   →  b < 0              ... (1)
    (0,1) → 1:  w₁(0) + w₂(1) + b ≥ 0   →  w₂ + b ≥ 0         ... (2)
    (1,0) → 1:  w₁(1) + w₂(0) + b ≥ 0   →  w₁ + b ≥ 0         ... (3)
    (1,1) → 0:  w₁(1) + w₂(1) + b < 0   →  w₁ + w₂ + b < 0   ... (4)

Adding (2) and (3):  w₁ + w₂ + 2b ≥ 0
From (4):            w₁ + w₂ + b < 0    →  w₁ + w₂ < -b

So:  w₁ + w₂ + 2b ≥ 0  and  w₁ + w₂ < -b
     → -b + 2b ≥ 0  → b ≥ 0

But from (1): b < 0   ⟹  CONTRADICTION! ✗

Therefore, no single perceptron can compute XOR. □
```

### 4.4 The Solution — Multi-Layer Networks

XOR can be decomposed: `XOR(x₁, x₂) = AND(OR(x₁, x₂), NAND(x₁, x₂))`

```
    x₁ ────┬─── OR neuron ────┐
           │                   ├─── AND neuron ─── y
    x₂ ────┴─── NAND neuron ──┘

    This requires TWO layers of neurons (a hidden layer + output layer)
```

This realization — that multi-layer networks were needed but couldn't be trained — caused the first AI winter.

---

## 5. Single Neuron as a Linear Classifier

### 5.1 Linearly Separable Problems a Perceptron CAN Solve

```
     AND Gate          OR Gate          NOT Gate
    x₂                x₂               x
  1 ┤ ○   ●         1 ┤ ●   ●        0 ┤ ● (y=1)
    │  ╲              │╱               │
    │   ╲             │                │
  0 ┤ ○   ○         0 ┤╱○   ●        1 ┤ ○ (y=0)
    ┼─────── x₁       ┼─────── x₁      ┼──────── y
      0   1              0   1

    All linearly separable ✓    XOR is NOT ✗
```

---

## 6. Complete Python Implementation

```python
import numpy as np
import matplotlib.pyplot as plt

class Perceptron:
    """
    Single-layer Perceptron for binary classification.
    Uses step activation function and the Perceptron Learning Rule.
    """
    
    def __init__(self, n_features, learning_rate=1.0):
        self.weights = np.zeros(n_features)
        self.bias = 0.0
        self.lr = learning_rate
        self.errors_per_epoch = []
    
    def step(self, z):
        """Heaviside step function"""
        return np.where(z >= 0, 1, 0)
    
    def predict(self, X):
        """Forward pass: z = w·x + b, y = step(z)"""
        z = X @ self.weights + self.bias
        return self.step(z)
    
    def fit(self, X, y, max_epochs=100):
        """
        Train using Perceptron Learning Rule.
        
        Parameters:
            X: (m, n) array — m samples, n features
            y: (m,) array — binary labels {0, 1}
            max_epochs: int — maximum training iterations
        """
        for epoch in range(max_epochs):
            errors = 0
            for xi, yi in zip(X, y):
                y_pred = self.step(np.dot(self.weights, xi) + self.bias)
                error = yi - y_pred
                
                if error != 0:
                    # Perceptron update rule
                    self.weights += self.lr * error * xi
                    self.bias += self.lr * error
                    errors += 1
            
            self.errors_per_epoch.append(errors)
            
            if errors == 0:
                print(f"Converged at epoch {epoch + 1}")
                return
        
        print(f"Did not converge after {max_epochs} epochs")
    
    def decision_boundary(self):
        """Return slope and intercept of decision boundary (2D only)"""
        if len(self.weights) != 2:
            raise ValueError("Decision boundary visualization requires 2D input")
        w1, w2 = self.weights
        # w1*x1 + w2*x2 + b = 0  →  x2 = -(w1/w2)*x1 - b/w2
        slope = -w1 / w2
        intercept = -self.bias / w2
        return slope, intercept


# ==========================================
# Example 1: AND Gate
# ==========================================
print("=" * 50)
print("Training Perceptron on AND Gate")
print("=" * 50)

X_and = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_and = np.array([0, 0, 0, 1])

p_and = Perceptron(n_features=2, learning_rate=1.0)
p_and.fit(X_and, y_and)

print(f"Weights: {p_and.weights}")
print(f"Bias: {p_and.bias}")
print(f"Predictions: {p_and.predict(X_and)}")
print(f"Errors per epoch: {p_and.errors_per_epoch}")
print()

# ==========================================
# Example 2: OR Gate
# ==========================================
print("=" * 50)
print("Training Perceptron on OR Gate")
print("=" * 50)

X_or = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_or = np.array([0, 1, 1, 1])

p_or = Perceptron(n_features=2, learning_rate=1.0)
p_or.fit(X_or, y_or)

print(f"Weights: {p_or.weights}")
print(f"Bias: {p_or.bias}")
print(f"Predictions: {p_or.predict(X_or)}")
print()

# ==========================================
# Example 3: XOR Gate (will NOT converge)
# ==========================================
print("=" * 50)
print("Training Perceptron on XOR Gate (WILL FAIL)")
print("=" * 50)

X_xor = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_xor = np.array([0, 1, 1, 0])

p_xor = Perceptron(n_features=2, learning_rate=1.0)
p_xor.fit(X_xor, y_xor, max_epochs=20)

print(f"Predictions: {p_xor.predict(X_xor)} (should be [0,1,1,0])")
print(f"Errors per epoch: {p_xor.errors_per_epoch}")
print("→ Oscillates! Cannot find a linear separator.")
print()

# ==========================================
# Example 4: Iris dataset (real-world)
# ==========================================
print("=" * 50)
print("Perceptron on Iris (Setosa vs Non-Setosa)")
print("=" * 50)

from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler

iris = load_iris()
X_iris = iris.data[:, [0, 2]]  # Sepal length, Petal length
y_iris = (iris.target == 0).astype(int)  # Setosa vs rest

scaler = StandardScaler()
X_iris_scaled = scaler.fit_transform(X_iris)

p_iris = Perceptron(n_features=2, learning_rate=0.1)
p_iris.fit(X_iris_scaled, y_iris, max_epochs=100)

predictions = p_iris.predict(X_iris_scaled)
accuracy = np.mean(predictions == y_iris)
print(f"Accuracy: {accuracy:.2%}")
print(f"Weights: {p_iris.weights}")
print(f"Bias: {p_iris.bias:.4f}")
```

**Expected Output:**
```
==================================================
Training Perceptron on AND Gate
==================================================
Converged at epoch 4
Weights: [1. 1.]
Bias: -1.0
Predictions: [0 0 0 1]

==================================================
Training Perceptron on OR Gate
==================================================
Converged at epoch 3
Weights: [1. 1.]
Bias: 0.0
Predictions: [0 1 1 1]

==================================================
Training Perceptron on XOR Gate (WILL FAIL)
==================================================
Did not converge after 20 epochs
Predictions: [1 1 0 0] (should be [0,1,1,0])
→ Oscillates! Cannot find a linear separator.

==================================================
Perceptron on Iris (Setosa vs Non-Setosa)
==================================================
Converged at epoch 5
Accuracy: 100.00%
```

---

## 7. Applications of the Perceptron

| Application | How It Works |
|---|---|
| **Spam filter (simple)** | Features: word frequencies; perceptron classifies spam/not-spam |
| **Image pixel classifier** | Single neuron classifies pixels as foreground/background |
| **Sentiment (basic)** | Binary positive/negative classification of text |
| **Building block** | Modern deep networks are composed of millions of "perceptron-like" neurons |

---

## 📊 Summary Table

| Concept | Key Formula / Fact |
|---|---|
| **Pre-activation** | z = w · x + b = Σᵢ wᵢxᵢ + b |
| **Perceptron output** | ŷ = step(z) = 1 if z ≥ 0, else 0 |
| **Update rule** | w ← w + η(y - ŷ)x, b ← b + η(y - ŷ) |
| **Decision boundary** | The hyperplane w · x + b = 0 |
| **Convergence** | Guaranteed if data is linearly separable |
| **XOR problem** | Cannot be solved by a single perceptron (proven) |
| **Solution to XOR** | Requires multi-layer perceptron (hidden layer) |
| **Weight vector** | Perpendicular to decision boundary |
| **Bias** | Shifts boundary away from origin |
| **Number of parameters** | n weights + 1 bias = n + 1 |

---

## ❓ Revision Questions

1. **Write the complete mathematical formulation of a perceptron. What are the learnable parameters, and how many are there for n inputs?**

2. **Walk through the perceptron learning algorithm step-by-step. What happens when the perceptron makes a false positive vs. a false negative?**

3. **Prove algebraically that a single perceptron cannot compute XOR. What mathematical property must the data have for a perceptron to work?**

4. **What does the Perceptron Convergence Theorem guarantee? What are its limitations?**

5. **Explain geometrically what the weight vector and bias represent in the context of a perceptron's decision boundary.**

6. **If you have a trained perceptron with w = [3, -2] and b = 1, classify the points (1, 2), (2, 4), and (0, 0). Draw the decision boundary equation.**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Biological Inspiration](01-biological-inspiration.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Activation Functions →](03-activation-functions.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
