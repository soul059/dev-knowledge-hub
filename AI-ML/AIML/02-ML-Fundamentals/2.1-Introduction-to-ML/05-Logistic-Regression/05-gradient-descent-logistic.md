# 📘 Chapter 5: Gradient Descent for Logistic Regression

> **Unit 5: Logistic Regression** | **Section 5 of 7**
> Implement gradient descent for logistic regression from scratch. Understand the update rules, vectorized computation, convergence behavior, and build a complete working classifier that matches sklearn's results.

---

## 📑 Table of Contents

1. [Gradient Computation Recap](#1-gradient-computation-recap)
2. [The Update Rules](#2-the-update-rules)
3. [Algorithm: Batch Gradient Descent](#3-algorithm-batch-gradient-descent)
4. [Vectorized Implementation](#4-vectorized-implementation)
5. [Convergence Analysis](#5-convergence-analysis)
6. [Complete Python Implementation from Scratch](#6-complete-python-implementation-from-scratch)
7. [Comparison with sklearn Results](#7-comparison-with-sklearn-results)
8. [Mini-Batch and Stochastic Variants](#8-mini-batch-and-stochastic-variants)
9. [Practical Tips and Common Pitfalls](#9-practical-tips-and-common-pitfalls)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Gradient Computation Recap

From [Chapter 4](04-cost-function-log-loss.md), we derived:

```
Cost Function:
    J(w, b) = -(1/m) Σᵢ₌₁ᵐ [yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]
    
    where ŷᵢ = σ(wᵀxᵢ + b)

Gradients:
    ∂J/∂wⱼ = (1/m) Σᵢ₌₁ᵐ (ŷᵢ - yᵢ) · xᵢⱼ      for each weight j
    ∂J/∂b  = (1/m) Σᵢ₌₁ᵐ (ŷᵢ - yᵢ)              for the bias
```

### A Surprising Similarity

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LINEAR REGRESSION gradient:                                     │
│      ∂J/∂wⱼ = (1/m) Σ (ŷᵢ - yᵢ) · xᵢⱼ   where ŷ = wᵀx + b  │
│                                                                  │
│  LOGISTIC REGRESSION gradient:                                   │
│      ∂J/∂wⱼ = (1/m) Σ (ŷᵢ - yᵢ) · xᵢⱼ   where ŷ = σ(wᵀx+b) │
│                                                                  │
│  THE FORMULA LOOKS IDENTICAL!                                    │
│  The difference is hidden inside ŷ:                              │
│  • Linear: ŷ is linear in w                                     │
│  • Logistic: ŷ is nonlinear in w (via sigmoid)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. The Update Rules

### Scalar Form (Single Feature Update)

```
Repeat until convergence:
{
    For j = 1, 2, ..., n:
        wⱼ := wⱼ - α · ∂J/∂wⱼ
        wⱼ := wⱼ - α · (1/m) Σᵢ₌₁ᵐ (σ(wᵀxᵢ + b) - yᵢ) · xᵢⱼ
    
    b := b - α · ∂J/∂b
    b := b - α · (1/m) Σᵢ₌₁ᵐ (σ(wᵀxᵢ + b) - yᵢ)
}
```

Where α is the **learning rate** (step size).

### Update Visualization

```
    J(w)
    │
    │ ╲
    │  ╲
    │   ╲  ← gradient points "uphill"
    │    ╲   so we go OPPOSITE direction
    │     ╲
    │      ●  w_old
    │       ╲
    │        ╲
    │         ╲
    │          ●  w_new = w_old - α·gradient
    │           ╲
    │            ╲
    │             ●  minimum (gradient ≈ 0)
    └──────────────────── w
    
    Each step: w_new = w_old - α · ∂J/∂w
    
    • gradient > 0 → move w LEFT (decrease)
    • gradient < 0 → move w RIGHT (increase)
    • gradient ≈ 0 → near minimum (stop)
```

---

## 3. Algorithm: Batch Gradient Descent

### Pseudocode

```
Algorithm: Batch Gradient Descent for Logistic Regression
═══════════════════════════════════════════════════════════

Input: X (m×n), y (m×1), α (learning rate), num_iterations
Output: w (n×1), b (scalar)

1.  Initialize w = zeros(n), b = 0
2.  
3.  For iteration = 1 to num_iterations:
4.      
5.      # ─── Forward pass ───
6.      z = X · w + b                    # (m×1) linear combination
7.      ŷ = σ(z)                          # (m×1) predictions
8.      
9.      # ─── Compute cost (optional, for monitoring) ───
10.     J = -(1/m) Σ [y·log(ŷ) + (1-y)·log(1-ŷ)]
11.     
12.     # ─── Compute gradients ───
13.     error = ŷ - y                     # (m×1) prediction errors
14.     dw = (1/m) · Xᵀ · error          # (n×1) weight gradients
15.     db = (1/m) · Σ(error)             # scalar bias gradient
16.     
17.     # ─── Update parameters ───
18.     w = w - α · dw
19.     b = b - α · db
20.     
21. Return w, b
```

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌─────────┐     ┌──────────┐     ┌─────────┐              │
│  │ Features │────→│  z = Xw+b │────→│ ŷ = σ(z) │             │
│  │   X      │     │  (linear) │     │(sigmoid) │             │
│  └─────────┘     └──────────┘     └────┬────┘              │
│                                         │                    │
│                                    ┌────▼────┐              │
│                                    │  Cost J  │←── y (true) │
│                                    │ (log     │              │
│                                    │  loss)   │              │
│                                    └────┬────┘              │
│                                         │                    │
│                                    ┌────▼────┐              │
│                                    │Gradients │              │
│                                    │ dw, db   │              │
│                                    └────┬────┘              │
│                                         │                    │
│  ┌──────────────────────────────────────▼──────────┐        │
│  │  Update: w = w - α·dw,  b = b - α·db           │        │
│  └─────────────────────────┬───────────────────────┘        │
│                            │                                 │
│                            ▼ (repeat)                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Vectorized Implementation

### Why Vectorize?

```
Loop-based (SLOW):                  Vectorized (FAST):
─────────────────                   ─────────────────
for i in range(m):                  z = X @ w + b
    z[i] = np.dot(w, X[i]) + b     y_hat = sigmoid(z)
    y_hat[i] = sigmoid(z[i])       error = y_hat - y
                                    dw = (1/m) * X.T @ error
for i in range(m):                  db = (1/m) * np.sum(error)
    for j in range(n):
        dw[j] += (y_hat[i]-y[i])*X[i,j]

dw /= m                            

Time: O(m·n) with Python overhead   Time: O(m·n) with C/BLAS speed
~100x slower!                        ~100x faster!
```

### Matrix Dimensions

```
    X:      (m × n)    m samples, n features
    w:      (n × 1)    n weights
    b:      scalar     bias
    
    z = X @ w + b:     (m × 1)    linear combinations
    ŷ = σ(z):          (m × 1)    predictions
    error = ŷ - y:     (m × 1)    prediction errors
    
    dw = (1/m) Xᵀ @ error:   (n × 1)    weight gradients
    db = (1/m) Σ(error):     scalar      bias gradient
    
    Example: m=1000 samples, n=10 features
    ┌──────────┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
    │ 1000×10  │ × │10×1│ = │1000│ → │1000│ = │1000│
    │   X      │   │ w  │   │ z  │   │ ŷ  │   │err │
    └──────────┘   └───┘   └───┘   └───┘   └───┘
    
    ┌──────────┐ᵀ  ┌───┐   ┌───┐
    │ 1000×10  │ × │1000│ = │10×1│
    │   Xᵀ     │   │err │   │ dw │
    └──────────┘   └───┘   └───┘
```

### Vectorized Gradient in NumPy

```python
# The ENTIRE gradient computation in 4 lines:
z = X @ w + b                    # Forward pass
y_hat = sigmoid(z)               # Predictions
dw = (1/m) * X.T @ (y_hat - y)  # Weight gradient
db = (1/m) * np.sum(y_hat - y)  # Bias gradient
```

---

## 5. Convergence Analysis

### Learning Rate Selection

```
    Too small α:                    Good α:                    Too large α:
    
    J                               J                          J
    │╲                              │╲                         │╲         ╱
    │ ╲                             │ ╲                        │ ╲      ╱
    │  ╲                            │  ╲                       │  ╲   ╱
    │   ╲                           │   ╲                      │   ╲╱  ← diverges!
    │    ╲                          │    ╲                      │    ╲
    │     ╲                         │     ╲──────              │
    │      ╲                        │     converged             │
    │       ╲╲╲╲╲╲╲╲╲╲╲╲╲          │                          │
    └───────────────────── iter     └─────────── iter          └─────── iter
    
    Takes forever!                  Converges quickly           Never converges!
```

### Convergence Criteria

```
Common stopping conditions:

1. Maximum iterations:
   if iteration >= max_iter: stop

2. Cost change threshold:
   if |J_new - J_old| < tolerance: stop

3. Gradient magnitude:
   if ‖∇J‖ < tolerance: stop

4. Parameter change:
   if ‖w_new - w_old‖ < tolerance: stop
```

### Convergence Rate

```
For convex functions (like log loss), gradient descent converges
at a rate of O(1/t) where t is the iteration number:

    J(wₜ) - J(w*) ≤ ‖w₀ - w*‖² / (2αt)
    
    where w* is the optimal solution.
    
    With careful learning rate scheduling, convergence 
    can be improved to O(1/t²) or even linear convergence
    for strongly convex functions.
    
    Iteration:  1    10    100    1000    10000
    Gap:       1.0   0.1   0.01   0.001   0.0001
    
    Each 10× more iterations → 10× smaller gap
```

---

## 6. Complete Python Implementation from Scratch

### The LogisticRegressionScratch Class

```python
import numpy as np

class LogisticRegressionScratch:
    """
    Logistic Regression implemented from scratch using gradient descent.
    
    Parameters:
    -----------
    learning_rate : float, default=0.01
        Step size for gradient descent.
    n_iterations : int, default=1000
        Number of gradient descent iterations.
    tolerance : float, default=1e-6
        Convergence threshold for cost change.
    verbose : bool, default=False
        Print cost during training.
    """
    
    def __init__(self, learning_rate=0.01, n_iterations=1000, 
                 tolerance=1e-6, verbose=False):
        self.learning_rate = learning_rate
        self.n_iterations = n_iterations
        self.tolerance = tolerance
        self.verbose = verbose
        self.weights = None
        self.bias = None
        self.cost_history = []
    
    def _sigmoid(self, z):
        """Numerically stable sigmoid function."""
        return np.where(
            z >= 0,
            1 / (1 + np.exp(-z)),
            np.exp(z) / (1 + np.exp(z))
        )
    
    def _compute_cost(self, y, y_pred):
        """Compute binary cross-entropy (log loss)."""
        m = len(y)
        epsilon = 1e-15  # prevent log(0)
        y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
        cost = -(1/m) * np.sum(
            y * np.log(y_pred) + (1 - y) * np.log(1 - y_pred)
        )
        return cost
    
    def _compute_gradients(self, X, y, y_pred):
        """Compute gradients of cost w.r.t. weights and bias."""
        m = len(y)
        error = y_pred - y
        dw = (1/m) * (X.T @ error)
        db = (1/m) * np.sum(error)
        return dw, db
    
    def fit(self, X, y):
        """
        Train the logistic regression model.
        
        Parameters:
        -----------
        X : array of shape (m, n) — training features
        y : array of shape (m,)   — training labels (0 or 1)
        
        Returns:
        --------
        self : fitted model
        """
        m, n = X.shape
        
        # Initialize parameters
        self.weights = np.zeros(n)
        self.bias = 0.0
        self.cost_history = []
        
        for i in range(self.n_iterations):
            # ─── Forward pass ───
            z = X @ self.weights + self.bias
            y_pred = self._sigmoid(z)
            
            # ─── Compute cost ───
            cost = self._compute_cost(y, y_pred)
            self.cost_history.append(cost)
            
            # ─── Check convergence ───
            if i > 0 and abs(self.cost_history[-2] - cost) < self.tolerance:
                if self.verbose:
                    print(f"Converged at iteration {i}, cost = {cost:.6f}")
                break
            
            # ─── Compute gradients ───
            dw, db = self._compute_gradients(X, y, y_pred)
            
            # ─── Update parameters ───
            self.weights -= self.learning_rate * dw
            self.bias -= self.learning_rate * db
            
            # ─── Logging ───
            if self.verbose and (i % 100 == 0 or i == self.n_iterations - 1):
                print(f"Iteration {i:5d}: cost = {cost:.6f}")
        
        return self
    
    def predict_proba(self, X):
        """
        Predict probability of class 1.
        
        Returns:
        --------
        proba : array of shape (m, 2) — probabilities for [class 0, class 1]
        """
        z = X @ self.weights + self.bias
        prob_1 = self._sigmoid(z)
        prob_0 = 1 - prob_1
        return np.column_stack([prob_0, prob_1])
    
    def predict(self, X, threshold=0.5):
        """Predict class labels (0 or 1)."""
        proba = self.predict_proba(X)[:, 1]
        return (proba >= threshold).astype(int)
    
    def score(self, X, y):
        """Compute accuracy."""
        predictions = self.predict(X)
        return np.mean(predictions == y)


# ═══════════════════════════════════════════════════════
# USAGE EXAMPLE
# ═══════════════════════════════════════════════════════

from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# Generate data
X, y = make_classification(
    n_samples=1000, n_features=5, n_informative=4,
    n_redundant=1, random_state=42
)

# Split and scale
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Train from scratch
model = LogisticRegressionScratch(
    learning_rate=0.1,
    n_iterations=2000,
    verbose=True
)
model.fit(X_train_s, y_train)

# Evaluate
train_acc = model.score(X_train_s, y_train)
test_acc = model.score(X_test_s, y_test)

print(f"\nResults (From Scratch):")
print(f"  Train Accuracy: {train_acc:.4f}")
print(f"  Test Accuracy:  {test_acc:.4f}")
print(f"  Final Cost:     {model.cost_history[-1]:.6f}")
print(f"  Iterations:     {len(model.cost_history)}")
print(f"  Weights:        {model.weights}")
print(f"  Bias:           {model.bias:.4f}")
```

### Training Progress Visualization (ASCII)

```python
def plot_cost_history_ascii(cost_history, width=60, height=15):
    """Print an ASCII plot of training cost over iterations."""
    costs = np.array(cost_history)
    n = len(costs)
    
    # Sample costs at regular intervals for display
    if n > width:
        indices = np.linspace(0, n-1, width).astype(int)
        sampled = costs[indices]
    else:
        sampled = costs
        indices = np.arange(n)
    
    c_min, c_max = sampled.min(), sampled.max()
    c_range = c_max - c_min if c_max > c_min else 1
    
    print(f"\n  Training Cost Over Iterations")
    print(f"  {'─' * (len(sampled) + 8)}")
    
    for row in range(height, -1, -1):
        y_val = c_min + (row / height) * c_range
        line = f"  {y_val:6.4f} │"
        for col in range(len(sampled)):
            normalized = (sampled[col] - c_min) / c_range * height
            if abs(normalized - row) < 0.5:
                line += "●"
            else:
                line += " "
        print(line)
    
    print(f"         └{'─' * len(sampled)}")
    print(f"          0" + " " * (len(sampled) - 8) + f"{n}")
    print(f"          {'iterations':^{len(sampled)}}")

# Example usage:
# plot_cost_history_ascii(model.cost_history)
```

---

## 7. Comparison with sklearn Results

### Head-to-Head Comparison

```python
from sklearn.linear_model import LogisticRegression

# ─── Train sklearn model ───
sklearn_model = LogisticRegression(
    max_iter=2000,
    solver='lbfgs',    # quasi-Newton method (more advanced)
    random_state=42,
    C=1e10             # very large C = no regularization (to match our model)
)
sklearn_model.fit(X_train_s, y_train)

# ─── Compare parameters ───
print("Parameter Comparison:")
print(f"{'':>20} {'From Scratch':>15} {'sklearn':>15}")
print(f"{'─'*55}")

for j in range(len(model.weights)):
    print(f"  w[{j}]              {model.weights[j]:>15.6f} {sklearn_model.coef_[0][j]:>15.6f}")

print(f"  bias              {model.bias:>15.6f} {sklearn_model.intercept_[0]:>15.6f}")

# ─── Compare performance ───
scratch_acc = model.score(X_test_s, y_test)
sklearn_acc = sklearn_model.score(X_test_s, y_test)

print(f"\nAccuracy Comparison:")
print(f"  From Scratch: {scratch_acc:.4f}")
print(f"  sklearn:      {sklearn_acc:.4f}")

# ─── Compare predictions ───
scratch_proba = model.predict_proba(X_test_s)[:, 1]
sklearn_proba = sklearn_model.predict_proba(X_test_s)[:, 1]

max_diff = np.max(np.abs(scratch_proba - sklearn_proba))
mean_diff = np.mean(np.abs(scratch_proba - sklearn_proba))

print(f"\nPrediction Comparison:")
print(f"  Max probability difference:  {max_diff:.6f}")
print(f"  Mean probability difference: {mean_diff:.6f}")
print(f"  Predictions agree: {np.mean(model.predict(X_test_s) == sklearn_model.predict(X_test_s))*100:.1f}%")

# ─── Show sample predictions ───
print(f"\nSample Predictions (first 5 test examples):")
print(f"{'True':>6} {'Scratch P':>12} {'sklearn P':>12} {'Scratch ŷ':>10} {'sklearn ŷ':>10}")
for i in range(5):
    print(f"{y_test[i]:>6d} {scratch_proba[i]:>12.6f} {sklearn_proba[i]:>12.6f} "
          f"{model.predict(X_test_s)[i]:>10d} {sklearn_model.predict(X_test_s)[i]:>10d}")
```

### Why Results Might Differ Slightly

```
┌──────────────────────────────────────────────────────────────────┐
│  Differences between our implementation and sklearn:             │
│                                                                  │
│  1. Optimizer: We use batch GD; sklearn uses L-BFGS              │
│     (quasi-Newton with better convergence)                       │
│                                                                  │
│  2. Regularization: sklearn adds L2 regularization by default    │
│     (C=1.0); we have none. Set C=very large to match.            │
│                                                                  │
│  3. Convergence: L-BFGS converges faster and more precisely      │
│                                                                  │
│  4. Feature scaling: Both need it, but sensitivity differs       │
│                                                                  │
│  With enough iterations and no regularization,                   │
│  results should be VERY close!                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 8. Mini-Batch and Stochastic Variants

### Three Flavors of Gradient Descent

```
┌──────────────────────────────────────────────────────────────┐
│ Variant        │ Batch Size │ Per Iteration              │
│────────────────│────────────│────────────────────────────│
│ Batch GD       │ m (all)    │ Uses ALL samples           │
│ Mini-Batch GD  │ B (e.g.32) │ Uses B random samples      │
│ Stochastic GD  │ 1          │ Uses ONE random sample     │
└──────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
Batch GD:                    Mini-Batch GD:              Stochastic GD:
(smooth but slow)            (good balance)              (noisy but fast)

    J                            J                           J
    │╲                           │╲                          │╲ ╱╲
    │ ╲                          │ ╲╱╲                       │ ╲╱  ╲╱╲
    │  ╲                         │    ╲╱╲                    │       ╲╱╲╱╲
    │   ╲                        │       ╲╱╲                 │           ╲╱╲╱
    │    ╲                       │          ╲──              │              ╲──
    │     ╲──────                │                           │
    └─────────── iter            └─────────── iter           └─────────── iter
    
    Smooth convergence           Slightly noisy              Very noisy
    Slow per iteration           Fast per iteration          Fastest per iteration
    Exact gradient               Approximate gradient        Very noisy gradient
```

### Mini-Batch Implementation

```python
class LogisticRegressionMiniBatch:
    """Logistic Regression with mini-batch gradient descent."""
    
    def __init__(self, learning_rate=0.01, n_epochs=100, 
                 batch_size=32, random_state=42):
        self.learning_rate = learning_rate
        self.n_epochs = n_epochs
        self.batch_size = batch_size
        self.random_state = random_state
        self.weights = None
        self.bias = None
        self.cost_history = []
    
    def _sigmoid(self, z):
        return np.where(z >= 0, 1/(1+np.exp(-z)), np.exp(z)/(1+np.exp(z)))
    
    def fit(self, X, y):
        m, n = X.shape
        self.weights = np.zeros(n)
        self.bias = 0.0
        self.cost_history = []
        
        rng = np.random.RandomState(self.random_state)
        
        for epoch in range(self.n_epochs):
            # Shuffle data at the start of each epoch
            indices = rng.permutation(m)
            X_shuffled = X[indices]
            y_shuffled = y[indices]
            
            epoch_cost = 0
            n_batches = 0
            
            # Process mini-batches
            for start in range(0, m, self.batch_size):
                end = min(start + self.batch_size, m)
                X_batch = X_shuffled[start:end]
                y_batch = y_shuffled[start:end]
                batch_m = len(y_batch)
                
                # Forward pass
                z = X_batch @ self.weights + self.bias
                y_pred = self._sigmoid(z)
                
                # Cost
                y_pred_clipped = np.clip(y_pred, 1e-15, 1-1e-15)
                batch_cost = -(1/batch_m) * np.sum(
                    y_batch * np.log(y_pred_clipped) + 
                    (1-y_batch) * np.log(1-y_pred_clipped)
                )
                epoch_cost += batch_cost
                n_batches += 1
                
                # Gradients
                error = y_pred - y_batch
                dw = (1/batch_m) * (X_batch.T @ error)
                db = (1/batch_m) * np.sum(error)
                
                # Update
                self.weights -= self.learning_rate * dw
                self.bias -= self.learning_rate * db
            
            self.cost_history.append(epoch_cost / n_batches)
        
        return self
    
    def predict_proba(self, X):
        z = X @ self.weights + self.bias
        p1 = self._sigmoid(z)
        return np.column_stack([1-p1, p1])
    
    def predict(self, X):
        return (self.predict_proba(X)[:, 1] >= 0.5).astype(int)
    
    def score(self, X, y):
        return np.mean(self.predict(X) == y)


# ─── Compare all three variants ───
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X, y = make_classification(n_samples=2000, n_features=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Batch GD
batch_model = LogisticRegressionScratch(learning_rate=0.1, n_iterations=500)
batch_model.fit(X_train_s, y_train)

# Mini-batch GD (batch_size=32)
mini_model = LogisticRegressionMiniBatch(learning_rate=0.05, n_epochs=50, batch_size=32)
mini_model.fit(X_train_s, y_train)

# Stochastic GD (batch_size=1)
sgd_model = LogisticRegressionMiniBatch(learning_rate=0.01, n_epochs=50, batch_size=1)
sgd_model.fit(X_train_s, y_train)

print("Comparison of GD Variants:")
print(f"{'Variant':<20} {'Train Acc':>10} {'Test Acc':>10} {'Iterations':>12}")
print("─" * 55)
print(f"{'Batch GD':<20} {batch_model.score(X_train_s, y_train):>10.4f} "
      f"{batch_model.score(X_test_s, y_test):>10.4f} {len(batch_model.cost_history):>12}")
print(f"{'Mini-Batch (B=32)':<20} {mini_model.score(X_train_s, y_train):>10.4f} "
      f"{mini_model.score(X_test_s, y_test):>10.4f} {len(mini_model.cost_history):>12}")
print(f"{'SGD (B=1)':<20} {sgd_model.score(X_train_s, y_train):>10.4f} "
      f"{sgd_model.score(X_test_s, y_test):>10.4f} {len(sgd_model.cost_history):>12}")
```

### Batch Size Recommendations

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Dataset Size      Recommended Batch Size                    │
│  ──────────────    ──────────────────────                    │
│  < 2,000           Use full batch (Batch GD)                │
│  2,000 - 20,000    32 - 128                                 │
│  20,000 - 200,000  128 - 512                                │
│  > 200,000         256 - 1024                               │
│                                                              │
│  Common sizes: 32, 64, 128, 256 (powers of 2 for GPU)      │
│                                                              │
│  Rule of thumb: Start with 32, increase if training         │
│  is too noisy, decrease if too slow.                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 9. Practical Tips and Common Pitfalls

### Tip 1: Feature Scaling is Critical

```python
# ─── Without scaling ───
model_unscaled = LogisticRegressionScratch(learning_rate=0.001, n_iterations=5000)
model_unscaled.fit(X_train, y_train)  # raw features

# ─── With scaling ───
model_scaled = LogisticRegressionScratch(learning_rate=0.1, n_iterations=500)
model_scaled.fit(X_train_s, y_train)  # standardized features

print(f"Without scaling: acc = {model_unscaled.score(X_test, y_test):.4f}, "
      f"iterations = {len(model_unscaled.cost_history)}")
print(f"With scaling:    acc = {model_scaled.score(X_test_s, y_test):.4f}, "
      f"iterations = {len(model_scaled.cost_history)}")
```

```
Why scaling matters:

    Unscaled features:               Scaled features:
    (very elongated contours)        (circular contours)
    
    w₂                               w₂
    │  ╲╲╲╲╲╲╲╲╲╲╲                   │    ╱──╲
    │   ╲╲╲╲╲╲╲╲╲╲                   │  ╱  ●  ╲
    │    ╲╲●╲╲╲╲╲╲                   │  ╲     ╱
    │     ╲╲╲╲╲╲╲╲                   │    ╲──╱
    └────────────── w₁               └────────── w₁
    
    GD zigzags!                      GD goes straight!
    Needs tiny learning rate         Can use larger learning rate
```

### Tip 2: Learning Rate Selection

```python
# ─── Learning rate finder ───
def find_learning_rate(X, y, lr_range=np.logspace(-5, 1, 20)):
    """Test different learning rates and report convergence."""
    print(f"{'Learning Rate':>15} {'Final Cost':>12} {'Converged':>10}")
    print("─" * 40)
    
    for lr in lr_range:
        model = LogisticRegressionScratch(learning_rate=lr, n_iterations=500)
        model.fit(X, y)
        final_cost = model.cost_history[-1]
        converged = len(model.cost_history) < 500
        
        status = "✓" if final_cost < 1.0 and not np.isnan(final_cost) else "✗"
        print(f"{lr:>15.6f} {final_cost:>12.6f} {status:>10}")

# find_learning_rate(X_train_s, y_train)
```

### Tip 3: Monitor Training

```python
# ─── Check for common issues ───
def diagnose_training(cost_history):
    """Diagnose potential training issues."""
    costs = np.array(cost_history)
    
    print("Training Diagnostics:")
    print(f"  Initial cost:    {costs[0]:.6f}")
    print(f"  Final cost:      {costs[-1]:.6f}")
    print(f"  Cost reduction:  {(1 - costs[-1]/costs[0])*100:.1f}%")
    
    # Check for divergence
    if costs[-1] > costs[0]:
        print("  ⚠️  DIVERGING! Learning rate is too high.")
    
    # Check for NaN
    if np.any(np.isnan(costs)):
        print("  ⚠️  NaN detected! Numerical instability.")
    
    # Check for slow convergence
    if len(costs) >= 1000 and costs[-1] > 0.5:
        print("  ⚠️  SLOW CONVERGENCE. Try increasing learning rate or scaling features.")
    
    # Check for oscillation
    diffs = np.diff(costs)
    sign_changes = np.sum(np.diff(np.sign(diffs)) != 0)
    if sign_changes > len(costs) * 0.3:
        print(f"  ⚠️  OSCILLATING ({sign_changes} direction changes). Reduce learning rate.")
    
    print(f"  ✓ Training completed in {len(costs)} iterations.")

# diagnose_training(model.cost_history)
```

### Common Pitfalls Summary

```
┌─────────────────────────────────────────────────────────────────┐
│  Pitfall                      │ Solution                        │
│───────────────────────────────│─────────────────────────────────│
│  Cost increasing (diverging)  │ Reduce learning rate            │
│  Cost barely changing         │ Increase learning rate          │
│  NaN in cost                  │ Clip sigmoid inputs, use stable │
│                               │ implementation                  │
│  Very slow convergence        │ Scale features, increase α      │
│  Training acc high, test low  │ Regularization (see Ch. 7)      │
│  Different from sklearn       │ Remove regularization (C=1e10)  │
│                               │ or increase iterations          │
│  Oscillating cost             │ Reduce learning rate or use     │
│                               │ learning rate decay             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **Weight update** | wⱼ := wⱼ - α·(1/m)Σ(ŷᵢ-yᵢ)xᵢⱼ | Same form as linear regression |
| **Bias update** | b := b - α·(1/m)Σ(ŷᵢ-yᵢ) | Average of prediction errors |
| **Vectorized gradient** | dw = (1/m)Xᵀ(ŷ-y) | One matrix multiply! |
| **Batch GD** | Uses all m samples per step | Smooth, but slow for large m |
| **Mini-batch GD** | Uses B samples per step | Good balance speed/stability |
| **Stochastic GD** | Uses 1 sample per step | Fast, noisy convergence |
| **Learning rate** | α ∈ [0.001, 1.0] typically | Too small=slow, too large=diverge |
| **Feature scaling** | StandardScaler (μ=0, σ=1) | Critical for convergence speed |
| **Convergence** | O(1/t) for convex functions | Cost decreases monotonically |
| **vs sklearn** | Same results with enough iterations | sklearn uses L-BFGS (faster) |

---

## 11. Revision Questions

### Q1: Gradient Similarity
**Q:** The gradient formula for logistic regression looks identical to linear regression: ∂J/∂wⱼ = (1/m)Σ(ŷᵢ-yᵢ)xᵢⱼ. If the formula is the same, why is logistic regression fundamentally different from linear regression in how it learns?

**A:** The formula appears identical, but **ŷ is computed differently**: in linear regression ŷ = wᵀx + b (linear), while in logistic regression ŷ = σ(wᵀx + b) (nonlinear via sigmoid). This means the gradient is a **nonlinear function** of w in logistic regression. Consequently: (1) the cost surface shape is different, (2) the gradients change nonlinearly as w updates, and (3) the optimization landscape is more complex despite being convex.

---

### Q2: Vectorized Computation
**Q:** Write the complete vectorized gradient descent update for logistic regression in exactly 4 lines of NumPy code. State the shapes of all intermediate variables for m=500 samples and n=10 features.

**A:**
```python
z = X @ w + b              # (500,) = (500,10) @ (10,) + scalar
y_hat = sigmoid(z)          # (500,) — element-wise sigmoid
dw = (1/m) * X.T @ (y_hat - y)  # (10,) = (10,500) @ (500,)
db = (1/m) * np.sum(y_hat - y)   # scalar
```

---

### Q3: Learning Rate Analysis
**Q:** You observe that your logistic regression cost oscillates wildly and even increases over iterations. What's wrong, and how do you fix it? What if the cost decreases but extremely slowly (barely changes after 10,000 iterations)?

**A:**
- **Oscillating/increasing cost:** Learning rate is **too large** — the gradient step overshoots the minimum and bounces around. Fix: reduce α by a factor of 2-10 (e.g., 0.1 → 0.01).
- **Very slow decrease:** Learning rate is **too small** — tiny steps toward the minimum. Fix: increase α. Also check if features are **unscaled** — different feature magnitudes cause elongated contours that make GD zigzag. Apply StandardScaler first.

---

### Q4: Batch Size Tradeoffs
**Q:** Compare batch gradient descent (m=all), mini-batch (m=32), and stochastic (m=1) in terms of: (a) compute per iteration, (b) convergence smoothness, (c) total time to reach a good solution, (d) memory requirements.

**A:**
| Aspect | Batch | Mini-batch (32) | Stochastic |
|--------|-------|----------------|------------|
| Compute/iter | O(m·n) — expensive | O(32·n) — cheap | O(n) — cheapest |
| Smoothness | Smooth descent | Slightly noisy | Very noisy |
| Total time | Slow (fewer but expensive iters) | **Fastest** overall | Fast but noisy |
| Memory | Load all m samples | Load 32 samples | Load 1 sample |
| Gradient quality | Exact | Good approximation | Noisy approximation |

Mini-batch is the **best overall** — it balances computation cost, gradient quality, and convergence speed.

---

### Q5: Implementation Debugging
**Q:** Your from-scratch logistic regression gets 50% accuracy (random guessing) on a dataset where sklearn achieves 95%. List 4 possible bugs and how to check for each.

**A:**
1. **Missing feature scaling:** Check if features have wildly different ranges. Fix: apply StandardScaler before training.
2. **Learning rate too small:** Check if cost barely decreases over iterations. Fix: try α = 0.01, 0.1, 1.0.
3. **Too few iterations:** Check if cost is still decreasing at the end. Fix: increase n_iterations to 5000+.
4. **Gradient sign error:** Check if cost increases every iteration (gradient ascent instead of descent). Fix: ensure update is `w -= α·dw` not `w += α·dw`. Verify with numerical gradient checking.

---

### Q6: Convergence Proof Intuition
**Q:** Why is gradient descent guaranteed to converge to the global minimum for logistic regression's log loss, but NOT for neural networks? What property of the cost function makes the difference?

**A:** Log loss for logistic regression is **convex** — it has a single global minimum and no local minima. Any direction "downhill" leads to the global optimum. The Hessian matrix H = (1/m)Xᵀ·diag(σ(z)·(1-σ(z)))·X is positive semi-definite, proving convexity. Neural networks have non-convex loss surfaces (due to multiple layers of nonlinear activations) with many local minima and saddle points, so gradient descent can get stuck in suboptimal solutions.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 4: Cost Function & Log Loss](04-cost-function-log-loss.md) | **Chapter 5: Gradient Descent** | [Chapter 6: Multi-Class Classification →](06-multi-class-classification.md) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. [Sigmoid Function](02-sigmoid-function.md)
3. [Decision Boundary](03-decision-boundary.md)
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. **📍 Gradient Descent for Logistic Regression** ← You are here
6. [Multi-Class Classification](06-multi-class-classification.md)
7. [Regularization](07-regularization.md)

---

> *"Gradient descent is the workhorse of machine learning — simple in concept, powerful in practice, and endlessly tunable."*

© 2025 AI/ML Study Notes. All rights reserved.
