# 📐 Linear Support Vector Machines

> **Unit 9, Chapter 1 — SVM Intuition, Hyperplane Equation & Linear Classification**

---

## 📋 Table of Contents

1. [Introduction & Intuition](#1-introduction--intuition)
2. [The Hyperplane Equation](#2-the-hyperplane-equation)
3. [Classification Rule](#3-classification-rule)
4. [Support Vectors](#4-support-vectors)
5. [Why Maximize Margin?](#5-why-maximize-margin)
6. [Geometric Margin vs Functional Margin](#6-geometric-margin-vs-functional-margin)
7. [SVM for Linearly Separable Data](#7-svm-for-linearly-separable-data)
8. [ASCII Diagram — Separating Hyperplane with Margin](#8-ascii-diagram--separating-hyperplane-with-margin)
9. [Comparison with Logistic Regression](#9-comparison-with-logistic-regression)
10. [Python Implementation](#10-python-implementation)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Introduction & Intuition

**Support Vector Machines (SVMs)** are one of the most elegant and powerful supervised learning algorithms in machine learning. Introduced by Vladimir Vapnik and colleagues in the 1990s, SVMs find an **optimal decision boundary** (called a *hyperplane*) that separates data points of different classes with the **maximum possible margin**.

### Core Intuition

Imagine you have two groups of points on a table — red balls and blue balls. You want to place a ruler (a line) between them so that:

1. **All red balls are on one side** and **all blue balls are on the other side**
2. The ruler is placed so that it is **as far as possible from the nearest ball on each side**

This second criterion is what makes SVM special. While many algorithms can find *a* separating boundary, SVM finds the *best* one — the one with the **maximum margin**.

### Why "Support Vector" Machine?

The algorithm is named after the **support vectors** — the data points closest to the decision boundary. These points literally "support" the hyperplane:

- If you remove a support vector, the optimal hyperplane changes
- If you remove a non-support-vector point, the hyperplane stays the same
- The final model depends **only** on support vectors

### Historical Context

| Year | Milestone |
|------|-----------|
| 1963 | Vapnik & Chervonenkis propose linear classifier |
| 1992 | Boser, Guyon & Vapnik introduce kernel trick |
| 1995 | Cortes & Vapnik introduce soft margin SVM |
| 2000s | SVM dominates many ML competitions |

---

## 2. The Hyperplane Equation

### What is a Hyperplane?

A **hyperplane** is a flat affine subspace of dimension (n-1) in an n-dimensional space:

| Space Dimension | Hyperplane | Dimension |
|-----------------|------------|-----------|
| 2D (plane) | Line | 1D |
| 3D (space) | Plane | 2D |
| nD | Hyperplane | (n-1)D |

### Mathematical Formulation

The hyperplane is defined by:

```
w · x + b = 0
```

Where:
- **w** = weight vector (normal to the hyperplane) = [w₁, w₂, ..., wₙ]ᵀ
- **x** = input feature vector = [x₁, x₂, ..., xₙ]ᵀ
- **b** = bias term (also called intercept)
- **w · x** = dot product = w₁x₁ + w₂x₂ + ... + wₙxₙ

### Expanded Form (2D Example)

In 2 dimensions:

```
w₁x₁ + w₂x₂ + b = 0
```

This is simply a line! For example:
- `2x₁ + 3x₂ - 6 = 0` → a line with w = [2, 3]ᵀ and b = -6

### The Normal Vector w

The weight vector **w** is **perpendicular (normal)** to the hyperplane:

```
                    w (normal vector)
                    ↑
                    |
                    |
    ================|================  ← Hyperplane (w·x + b = 0)
                    |
```

**Key insight**: The direction of **w** determines which side is positive and which is negative.

### Distance from Origin to Hyperplane

The perpendicular distance from the origin to the hyperplane is:

```
            |b|
d = ─────────────
        ‖w‖

where ‖w‖ = √(w₁² + w₂² + ... + wₙ²)
```

---

## 3. Classification Rule

### Decision Function

For a binary classification problem with classes y ∈ {-1, +1}, the SVM decision function is:

```
f(x) = sign(w · x + b)
```

The classification rule:

```
        ┌ +1   if  w · x + b > 0    (positive class)
ŷ(x) = │
        └ -1   if  w · x + b < 0    (negative class)
```

### How It Works

The hyperplane `w · x + b = 0` divides the feature space into two half-spaces:

```
    Positive half-space          Negative half-space
    (w · x + b > 0)             (w · x + b < 0)
    Class = +1                   Class = -1

         + + +                     - - -
       + + + + +     |          - - - - -
     + + + + + + +   |        - - - - - - -
       + + + + +     |          - - - - -
         + + +       |            - - -
                     |
              w · x + b = 0
```

### Confidence of Prediction

The magnitude |w · x + b| indicates **confidence**:

- **Large** |w · x + b| → point is far from boundary → **high confidence**
- **Small** |w · x + b| → point is near boundary → **low confidence**
- |w · x + b| = 0 → point is **on** the boundary → **no confidence**

---

## 4. Support Vectors

### Definition

**Support vectors** are the training data points that lie **closest to the decision boundary** (hyperplane). They satisfy:

```
|w · xᵢ + b| = 1    (for the canonical hyperplane)
```

That is, they lie exactly on the **margin boundaries**:
- Positive support vectors: `w · xᵢ + b = +1`
- Negative support vectors: `w · xᵢ + b = -1`

### Properties of Support Vectors

| Property | Description |
|----------|-------------|
| **Define the margin** | The margin is determined entirely by support vectors |
| **Sparse representation** | Only a few training points are support vectors |
| **Model dependence** | The SVM model depends ONLY on support vectors |
| **Removal sensitivity** | Removing a support vector changes the model |
| **Non-SV removal** | Removing a non-support-vector does NOT change the model |

### Why Are Support Vectors Important?

```
Consider 1000 training points:
├── Maybe only 20 are support vectors
├── The other 980 points are irrelevant to the model
├── This makes SVM memory-efficient at prediction time
└── The model is fully defined by these 20 points + w, b
```

### Identifying Support Vectors

After training an SVM, support vectors have a Lagrange multiplier αᵢ > 0 (more on this in Chapter 2):

```
Point xᵢ is a support vector  ⟺  αᵢ > 0
Point xᵢ is NOT a support vector  ⟺  αᵢ = 0
```

---

## 5. Why Maximize Margin?

### The Margin

The **margin** is the distance between the two margin boundaries (the "street" between the classes):

```
Margin = 2 / ‖w‖
```

### Why Maximum Margin is Better

#### 1. Statistical Learning Theory (Vapnik's Argument)

From **VC theory**, the generalization error bound is:

```
                              h(log(2N/h) + 1) - log(η/4)
Error_test ≤ Error_train + √ ─────────────────────────────
                                          N

where h = VC dimension, N = number of samples, η = confidence
```

A **larger margin** corresponds to a **smaller VC dimension**, leading to better generalization.

#### 2. Geometric Intuition

```
NARROW MARGIN (Bad):           WIDE MARGIN (Good):
                               
  +  +                           +  +
  +  + |  - -                    +  +  |         |  - -
  + +  |  - -                    + +   |         |  - -
  +    |  - -                    +     |         |  - -
  +  + |  - -                    +  +  |         |  - -
                               
  ← Small gap →                 ← ── Large gap ── →
  Sensitive to noise            Robust to noise
  Likely overfits               Better generalization
```

#### 3. Robustness to Perturbations

With a wider margin:
- Small changes in data are less likely to cause misclassification
- New test points have more "room" before crossing the boundary
- The classifier is more **stable** and **robust**

#### 4. PAC Learning Connection

The maximum margin classifier minimizes the **structural risk**, not just the empirical risk, providing better bounds on the true error.

---

## 6. Geometric Margin vs Functional Margin

### Functional Margin

The **functional margin** of a point (xᵢ, yᵢ) with respect to hyperplane (w, b) is:

```
γ̂ᵢ = yᵢ(w · xᵢ + b)
```

Properties:
- γ̂ᵢ > 0 means the point is **correctly classified**
- γ̂ᵢ < 0 means the point is **misclassified**
- Larger γ̂ᵢ → more confident prediction

**Problem with functional margin**: It is **not scale-invariant**!

```
If we replace (w, b) with (2w, 2b):
- The hyperplane doesn't change (same set of points satisfying w·x + b = 0)
- But the functional margin doubles: yᵢ(2w·xᵢ + 2b) = 2·yᵢ(w·xᵢ + b)
- We can make the margin arbitrarily large just by scaling!
```

### Geometric Margin

The **geometric margin** normalizes by ‖w‖ to make it scale-invariant:

```
              yᵢ(w · xᵢ + b)       γ̂ᵢ
γᵢ = ───────────────────── = ─────
              ‖w‖                  ‖w‖
```

Properties:
- **Scale-invariant**: Doesn't change when we scale (w, b)
- Represents the **actual geometric distance** from point to hyperplane
- This is what we actually want to maximize

### Comparison

| Property | Functional Margin γ̂ | Geometric Margin γ |
|----------|---------------------|-------------------|
| Formula | yᵢ(w·xᵢ + b) | yᵢ(w·xᵢ + b) / ‖w‖ |
| Scale-invariant? | ❌ No | ✅ Yes |
| Physical meaning | Signed "raw score" | Actual perpendicular distance |
| Can be made large by scaling w? | ❌ Yes (problematic) | ✅ No (stable) |
| Used in optimization | Normalized to 1 | Maximized as 1/‖w‖ |

### The Canonical Hyperplane Trick

To handle the scaling ambiguity, we define the **canonical hyperplane** by requiring:

```
min |w · xᵢ + b| = 1   (over all training points)
 i
```

This means the closest points to the hyperplane have functional margin = 1.

Under this normalization:
```
Geometric margin = 1 / ‖w‖
Total margin (width of the "street") = 2 / ‖w‖
```

---

## 7. SVM for Linearly Separable Data

### Problem Setup

Given a training set of N points:

```
{(x₁, y₁), (x₂, y₂), ..., (xₙ, yₙ)}

where xᵢ ∈ ℝⁿ and yᵢ ∈ {-1, +1}
```

The data is **linearly separable** if there exists a hyperplane (w, b) such that:

```
yᵢ(w · xᵢ + b) > 0    for all i = 1, 2, ..., N
```

### The SVM Optimization Problem

Find the hyperplane that **maximizes the margin**:

```
           2
maximize  ─────    (maximize the margin)
          ‖w‖

subject to:  yᵢ(w · xᵢ + b) ≥ 1   for all i
```

Equivalently (since maximizing 2/‖w‖ is the same as minimizing ‖w‖²/2):

```
         1
minimize ─── ‖w‖²
          2

subject to:  yᵢ(w · xᵢ + b) ≥ 1   for all i
```

### Why This Formulation?

1. The constraint `yᵢ(w·xᵢ + b) ≥ 1` ensures all points are on the correct side with at least functional margin 1
2. Minimizing `½‖w‖²` maximizes the geometric margin `1/‖w‖`
3. The `½` factor is for mathematical convenience (cleaner derivatives)
4. This is a **convex quadratic programming** problem → guaranteed global optimum!

### Worked Example (2D)

**Given data:**

| Point | x₁ | x₂ | y (class) |
|-------|----|----|-----------|
| A | 1 | 1 | +1 |
| B | 2 | 2 | +1 |
| C | 2 | 0 | +1 |
| D | -1 | -1 | -1 |
| E | -1 | 0 | -1 |
| F | 0 | -1 | -1 |

**Step 1**: We need to find w = [w₁, w₂]ᵀ and b that minimize ½(w₁² + w₂²)

Subject to:
```
For A (1,1), y=+1:   w₁ + w₂ + b ≥ 1
For B (2,2), y=+1:   2w₁ + 2w₂ + b ≥ 1
For C (2,0), y=+1:   2w₁ + b ≥ 1
For D (-1,-1), y=-1:  w₁ + w₂ + b ≤ -1   →  -(w₁+w₂+b) ≥ 1   → -w₁-w₂-b ≥ 1
For E (-1,0), y=-1:   w₁ + b ≤ -1         →  -(w₁+b) ≥ 1       → -w₁-b ≥ 1
For F (0,-1), y=-1:   w₂ + b ≤ -1         →  -(w₂+b) ≥ 1       → -w₂-b ≥ 1
```

**Step 2**: Solving (via Lagrangian or QP solver), we get approximately:
```
w ≈ [1, 1]ᵀ,  b ≈ 0

Decision boundary: x₁ + x₂ = 0
Margin = 2/‖w‖ = 2/√2 ≈ 1.414
```

The support vectors are the points closest to this boundary.

---

## 8. ASCII Diagram — Separating Hyperplane with Margin

```
        x₂
        ↑
    4   |
        |                    + (3,3)
    3   |              + (2,3)
        |         + (2,2)
    2   |    + (1,2)          POSITIVE CLASS (y = +1)
        |                          w · x + b > 0
    1   |. . . . . . . . . . . . . . . . . . . .  ← w·x+b = +1 (positive margin boundary)
        |         /                                    ▲
    0.5 |        / ← MARGIN = 2/‖w‖                   │ Support
        |       /                                      │ Vectors
    0   ├──────/─────────────────────────────→ x₁      │ lie here
        |     /                                        ▼
   -0.5 |    /
        |   /
   -1   |. . . . . . . . . . . . . . . . . . . .  ← w·x+b = -1 (negative margin boundary)
        |                          NEGATIVE CLASS (y = -1)
   -2   |    - (-1,-2)                w · x + b < 0
        |         - (-2,-2)
   -3   |              - (-2,-3)
        |
   -4   |

                    w · x + b = 0   ← Decision Boundary (hyperplane)
                    (the center line of the margin)
```

### Detailed Margin Diagram

```
     ┌─────────────────────────────────────────────────────────┐
     │                                                         │
     │    + + +              POSITIVE REGION                   │
     │   + + + +                                               │
     │  + + + + +            w · x + b > +1                    │
     │   + + + +                                               │
     │    + ◆ +    ← Positive Support Vectors (w·x+b = +1)    │
     │═══════════════════════════════════════════ Margin ═══╗   │
     │             boundary                                ║   │
     │                                                     ║   │
     │ ─ ─ ─ ─ ─ DECISION BOUNDARY (w·x+b = 0) ─ ─ ─ ─   ║   │
     │                                                  2/‖w‖  │
     │                                                     ║   │
     │             boundary                                ║   │
     │═══════════════════════════════════════════ Margin ═══╝   │
     │    - ◆ -    ← Negative Support Vectors (w·x+b = -1)    │
     │   - - - -                                               │
     │  - - - - -            w · x + b < -1                    │
     │   - - - -                                               │
     │    - - -              NEGATIVE REGION                   │
     │                                                         │
     └─────────────────────────────────────────────────────────┘
     
     ◆ = Support Vectors (the critical points)
     + = Positive class points
     - = Negative class points
```

---

## 9. Comparison with Logistic Regression

### Key Differences

| Aspect | SVM | Logistic Regression |
|--------|-----|-------------------|
| **Objective** | Maximize margin (geometric) | Maximize likelihood (probabilistic) |
| **Loss function** | Hinge loss: max(0, 1 - yf(x)) | Log loss: log(1 + e⁻ʸᶠ⁽ˣ⁾) |
| **Output** | Class label (-1 or +1) | Probability P(y=1|x) |
| **Decision boundary** | Maximum margin hyperplane | Where P(y=1|x) = 0.5 |
| **Sparsity** | Only support vectors matter | All points contribute |
| **Sensitivity to outliers** | Less sensitive (hinge loss plateaus) | More sensitive (log loss never plateaus) |
| **Kernel trick** | Natural extension | Possible but less common |
| **Probabilistic output** | Not native (Platt scaling needed) | Native probability estimates |

### Loss Function Comparison

```
Loss
  ↑
4 │╲
  │ ╲  Hinge Loss: max(0, 1 - y·f(x))
3 │  ╲
  │   ╲
2 │    ╲         Log Loss: log(1 + e^(-y·f(x)))
  │     ╲       ╱
1 │──────╲─────╱────────────────────
  │       ╲   ╱
0 │────────╲─╱─────────────────────→ y·f(x)
  │    -1   0   1   2   3   4
  │
  │ Key:
  │  ╲ = Hinge loss (flat at 0 for y·f(x) ≥ 1)
  │  ╱ = Log loss (always > 0, never exactly 0)
```

### When to Use Each

```
Use SVM when:                       Use Logistic Regression when:
├── High-dimensional data           ├── Need probability outputs
├── Clear margin of separation      ├── Need model interpretability
├── Fewer training samples          ├── Online learning (streaming data)
├── Text classification             ├── Want feature importance (coefficients)
└── Kernel methods needed           └── Large-scale data (faster training)
```

---

## 10. Python Implementation

### Basic Linear SVM with sklearn

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_blobs
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# ── Generate linearly separable data ──
X, y = make_blobs(n_samples=200, centers=2, n_features=2,
                  cluster_std=1.2, random_state=42)

# ── Split data ──
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

# ── Train Linear SVM ──
svm_model = SVC(kernel='linear', C=1.0)
svm_model.fit(X_train, y_train)

# ── Predictions ──
y_pred = svm_model.predict(X_test)

# ── Evaluation ──
print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"\nClassification Report:\n{classification_report(y_test, y_pred)}")

# ── Model Details ──
print(f"Weight vector w: {svm_model.coef_[0]}")
print(f"Bias b: {svm_model.intercept_[0]:.4f}")
print(f"Number of support vectors: {svm_model.n_support_}")
print(f"Support vector indices: {svm_model.support_}")
```

### Visualizing the Decision Boundary and Margin

```python
def plot_svm_decision_boundary(model, X, y, title="Linear SVM"):
    """Visualize SVM decision boundary, margin, and support vectors."""
    plt.figure(figsize=(10, 8))
    
    # Create mesh grid
    h = 0.02
    x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                         np.arange(y_min, y_max, h))
    
    # Decision function values
    Z = model.decision_function(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    
    # Plot decision boundary and margins
    plt.contourf(xx, yy, Z, levels=[-100, -1, 0, 1, 100],
                 colors=['#FFCCCC', '#FFEEEE', '#EEEEFF', '#CCCCFF'], alpha=0.5)
    plt.contour(xx, yy, Z, levels=[-1, 0, 1],
                colors=['red', 'black', 'blue'],
                linestyles=['--', '-', '--'], linewidths=[1.5, 2, 1.5])
    
    # Plot data points
    plt.scatter(X[y == 0, 0], X[y == 0, 1], c='red', marker='o',
                edgecolors='k', s=50, label='Class 0')
    plt.scatter(X[y == 1, 0], X[y == 1, 1], c='blue', marker='s',
                edgecolors='k', s=50, label='Class 1')
    
    # Highlight support vectors
    plt.scatter(model.support_vectors_[:, 0], model.support_vectors_[:, 1],
                s=200, facecolors='none', edgecolors='green',
                linewidths=2, label='Support Vectors')
    
    # Calculate and display margin
    w = model.coef_[0]
    margin = 2 / np.linalg.norm(w)
    
    plt.title(f'{title}\nMargin = {margin:.4f}, '
              f'Support Vectors = {sum(model.n_support_)}')
    plt.xlabel('Feature 1')
    plt.ylabel('Feature 2')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig('linear_svm_boundary.png', dpi=150, bbox_inches='tight')
    plt.show()

plot_svm_decision_boundary(svm_model, X_train, y_train)
```

### Manual Implementation from Scratch

```python
import numpy as np
from scipy.optimize import minimize

class LinearSVMFromScratch:
    """Linear SVM using direct optimization of primal form."""
    
    def __init__(self, C=1.0, lr=0.001, n_iters=1000):
        self.C = C
        self.lr = lr
        self.n_iters = n_iters
        self.w = None
        self.b = None
        self.losses = []
    
    def _hinge_loss(self, X, y):
        """Compute hinge loss + regularization."""
        n = X.shape[0]
        margins = y * (X @ self.w + self.b)
        hinge = np.maximum(0, 1 - margins)
        loss = 0.5 * np.dot(self.w, self.w) + self.C * np.mean(hinge)
        return loss
    
    def fit(self, X, y):
        """Train using sub-gradient descent."""
        n_samples, n_features = X.shape
        
        # Convert labels to -1, +1
        y_ = np.where(y <= 0, -1, 1)
        
        self.w = np.zeros(n_features)
        self.b = 0
        
        for epoch in range(self.n_iters):
            margins = y_ * (X @ self.w + self.b)
            
            # Sub-gradient computation
            misclassified = margins < 1  # Points violating margin
            
            dw = self.w - self.C * np.mean(
                (y_ * misclassified).reshape(-1, 1) * X, axis=0
            )
            db = -self.C * np.mean(y_ * misclassified)
            
            self.w -= self.lr * dw
            self.b -= self.lr * db
            
            loss = self._hinge_loss(X, y_)
            self.losses.append(loss)
        
        return self
    
    def predict(self, X):
        return np.sign(X @ self.w + self.b)
    
    def decision_function(self, X):
        return X @ self.w + self.b


# ── Usage ──
svm_scratch = LinearSVMFromScratch(C=1.0, lr=0.001, n_iters=2000)
svm_scratch.fit(X_train, np.where(y_train == 0, -1, 1))

y_pred_scratch = svm_scratch.predict(X_test)
y_test_converted = np.where(y_test == 0, -1, 1)
accuracy = np.mean(y_pred_scratch == y_test_converted)
print(f"Scratch SVM Accuracy: {accuracy:.4f}")
```

---

## 11. Applications

### Real-World Applications of Linear SVM

| Domain | Application | Why SVM Works Well |
|--------|-------------|-------------------|
| **Text Classification** | Spam detection, sentiment analysis | High-dimensional sparse data |
| **Bioinformatics** | Gene expression classification | Few samples, many features |
| **Image Recognition** | Handwritten digit recognition | Clear margin between classes |
| **Finance** | Credit risk assessment | Robust to outliers |
| **Medical Diagnosis** | Cancer detection from microarray data | High accuracy needed |
| **Natural Language** | Document categorization | Bag-of-words is high-dimensional |

### Why SVM Excels in Text Classification

```
Text data characteristics:
├── Very high dimensional (10,000+ features)
├── Sparse (most features are 0 for any document)
├── Few irrelevant features
├── Linearly separable in high dimensions
└── SVM handles all of these naturally!
```

---

## 12. Summary Table

| Concept | Key Formula / Insight |
|---------|----------------------|
| **Hyperplane equation** | w · x + b = 0 |
| **Classification rule** | ŷ = sign(w · x + b) |
| **Support vectors** | Points with yᵢ(w · xᵢ + b) = 1 |
| **Margin width** | 2 / ‖w‖ |
| **Functional margin** | γ̂ᵢ = yᵢ(w · xᵢ + b) — NOT scale-invariant |
| **Geometric margin** | γᵢ = yᵢ(w · xᵢ + b) / ‖w‖ — scale-invariant |
| **Optimization** | min ½‖w‖² s.t. yᵢ(w·xᵢ+b) ≥ 1 |
| **SVM vs LR** | SVM maximizes margin; LR maximizes likelihood |
| **Key advantage** | Model depends only on support vectors |
| **Loss function** | Hinge loss: max(0, 1 - y·f(x)) |

---

## 13. Revision Questions

### Q1: Conceptual Understanding
**What is the geometric interpretation of the weight vector w in an SVM? How does it relate to the hyperplane?**

<details>
<summary>Answer</summary>

The weight vector **w** is the **normal vector** (perpendicular) to the decision hyperplane `w · x + b = 0`. It points in the direction of the positive class. Its magnitude ‖w‖ is inversely related to the margin width: margin = 2/‖w‖. A larger ‖w‖ means a narrower margin, and a smaller ‖w‖ means a wider margin.

</details>

### Q2: Mathematical
**Given a 2D SVM with w = [3, 4]ᵀ and b = -10, calculate: (a) the margin width, (b) the distance from the origin to the hyperplane, (c) classify the point x = [4, 3]ᵀ.**

<details>
<summary>Answer</summary>

**(a)** Margin = 2/‖w‖ = 2/√(9+16) = 2/5 = 0.4

**(b)** Distance from origin = |b|/‖w‖ = |-10|/5 = 2.0

**(c)** f(x) = w·x + b = 3(4) + 4(3) - 10 = 12 + 12 - 10 = 14 > 0 → **Class +1**

</details>

### Q3: Support Vectors
**Why does removing a non-support-vector data point not change the SVM decision boundary? What does this imply about the model's dependence on training data?**

<details>
<summary>Answer</summary>

Non-support-vector points have Lagrange multipliers αᵢ = 0 and satisfy yᵢ(w·xᵢ + b) > 1 (they are strictly beyond the margin). They don't participate in defining the optimal hyperplane. The SVM solution depends **only** on support vectors (points with αᵢ > 0), making it a **sparse model**. This means SVM generalizes well even when most training points are removed, as long as support vectors remain.

</details>

### Q4: Comparison
**Compare the loss functions of SVM (hinge loss) and logistic regression (log loss). Which is more robust to outliers and why?**

<details>
<summary>Answer</summary>

- **Hinge loss**: max(0, 1 - y·f(x)) — goes to zero for correctly classified points with y·f(x) ≥ 1. For misclassified points, loss grows linearly.
- **Log loss**: log(1 + e^{-y·f(x)}) — never exactly zero, always penalizes, but approaches zero exponentially for correct classifications.

SVM (hinge loss) is **more robust to outliers** because once a point is correctly classified with sufficient margin (y·f(x) ≥ 1), it incurs **zero** loss and zero gradient. Outliers far on the wrong side contribute only linear loss. Log loss always contributes some penalty, making logistic regression more sensitive to outliers on the correct side.

</details>

### Q5: Practical
**You trained a linear SVM and found 150 support vectors out of 10,000 training points. What does this tell you about your dataset and model?**

<details>
<summary>Answer</summary>

Having only 150/10,000 = 1.5% support vectors indicates:
1. **The classes are well-separated** — most points are far from the decision boundary
2. **The model is sparse** — prediction depends on only 150 points (memory-efficient)
3. **Good generalization expected** — few support vectors relative to data size suggests the model isn't overfitting
4. **The margin is relatively large** — since most points are beyond the margin boundaries

If the number were much larger (e.g., 5000/10,000), it would suggest the classes overlap significantly or the data isn't well-suited for a linear SVM.

</details>

### Q6: Coding Challenge
**Write a Python snippet that trains a linear SVM on the Iris dataset (using only 2 features for visualization), prints the support vectors, and calculates the margin width.**

<details>
<summary>Answer</summary>

```python
from sklearn.svm import SVC
from sklearn.datasets import load_iris
import numpy as np

# Load Iris (binary: classes 0 and 1, features 0 and 1)
iris = load_iris()
X = iris.data[:100, :2]  # First 2 features, first 2 classes
y = iris.target[:100]

# Train linear SVM
svm = SVC(kernel='linear', C=1.0)
svm.fit(X, y)

# Support vectors
print(f"Support vectors:\n{svm.support_vectors_}")
print(f"Number per class: {svm.n_support_}")

# Margin width
w = svm.coef_[0]
margin = 2 / np.linalg.norm(w)
print(f"Margin width: {margin:.4f}")
```

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [Unit 8: Ensemble Methods](../../08-Ensemble-Methods/) | [Unit 9: SVM](../) | [Maximum Margin Classifier →](./02-maximum-margin-classifier.md) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 1 of 7 — Linear SVM*
