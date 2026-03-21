# 🎯 Common Kernel Functions

> **Unit 9, Chapter 5 — Linear, Polynomial, RBF & Sigmoid Kernels with Parameter Tuning**

---

## 📋 Table of Contents

1. [Chapter Overview](#1-chapter-overview)
2. [Linear Kernel](#2-linear-kernel)
3. [Polynomial Kernel](#3-polynomial-kernel)
4. [RBF / Gaussian Kernel](#4-rbf--gaussian-kernel)
5. [Sigmoid Kernel](#5-sigmoid-kernel)
6. [The γ (Gamma) Parameter Effect](#6-the-γ-gamma-parameter-effect)
7. [Choosing the Right Kernel — Flowchart](#7-choosing-the-right-kernel--flowchart)
8. [Kernel Comparison on Different Datasets](#8-kernel-comparison-on-different-datasets)
9. [sklearn Kernel Parameters](#9-sklearn-kernel-parameters)
10. [Python Implementation](#10-python-implementation)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Chapter Overview

Now that we understand the kernel trick (Chapter 4), let's study the most common kernel functions, their mathematical formulations, behaviors, and when to use each one.

```
Common Kernels:

1. LINEAR      K = x · y                          → Simplest, fastest
2. POLYNOMIAL  K = (γ·x·y + r)^d                  → Captures interactions
3. RBF         K = exp(-γ‖x-y‖²)                  → Most popular, flexible
4. SIGMOID     K = tanh(γ·x·y + r)                 → Neural network connection
```

---

## 2. Linear Kernel

### Formula

```
K(x, y) = x · y = Σⱼ xⱼyⱼ
```

No transformation — the kernel is just the dot product in the original space.

### Properties

| Property | Value |
|----------|-------|
| Feature space | Same as input space (ℝⁿ) |
| Parameters | None |
| Decision boundary | Linear (hyperplane) |
| Computational cost | O(n) per evaluation |
| Best for | High-dimensional, linearly separable data |

### When to Use Linear Kernel

```
Use LINEAR kernel when:
├── Number of features >> Number of samples (n >> N)
│   (e.g., text classification with bag-of-words)
├── Data is already high-dimensional
│   (likely linearly separable in high-D)
├── Want fast training and prediction
├── Need interpretable model (w gives feature importance)
└── As a baseline before trying nonlinear kernels

Examples:
├── Text classification (10,000+ features)
├── Gene expression data (20,000+ genes, 100 samples)
└── Sparse high-dimensional data
```

### Equivalent to LinearSVC

```python
# These are equivalent:
SVC(kernel='linear', C=1.0)           # Uses dual (slower for large N)
LinearSVC(C=1.0, loss='hinge')         # Uses primal (faster for large N)
SGDClassifier(loss='hinge', alpha=1/C)  # Stochastic (fastest for huge N)
```

---

## 3. Polynomial Kernel

### Formula

```
K(x, y) = (γ · x · y + r)^d

Parameters:
  γ (gamma):  Scaling factor for dot product (default: 1/n_features)
  r (coef0):  Independent term, shifts the kernel (default: 0)
  d (degree): Polynomial degree (default: 3)
```

### What It Captures

The polynomial kernel captures **feature interactions up to degree d**:

```
Degree 1: Individual features                    (like linear)
Degree 2: Pairwise feature interactions           x₁x₂, x₁x₃, ...
Degree 3: Triple feature interactions             x₁x₂x₃, ...
Degree d: All interactions up to order d
```

### Example: Degree 2 with r=1

For x = (x₁, x₂), K(x,y) = (x·y + 1)²:

```
Feature map:
φ(x) = (x₁², √2·x₁x₂, x₂², √2·x₁, √2·x₂, 1)
        ─────  ─────────  ─────  ───────  ───────  ─
        quadratic  interaction  quadratic  linear    linear   constant
        terms      terms       terms      terms     terms    term
        
Dimension: from 2 → 6
General: from n → C(n+d, d)
```

### The Role of r (coef0)

```
r = 0 (homogeneous):  K = (γ·x·y)^d
├── Only includes degree-d terms
├── φ includes ONLY degree-d monomials
└── No lower-order terms

r > 0 (inhomogeneous): K = (γ·x·y + r)^d
├── Includes ALL degrees from 0 to d
├── φ includes monomials of degree 0, 1, 2, ..., d
└── More flexible (usually better)
```

### Degree Effect

```
Degree 2:  Captures quadratic patterns (parabolas, ellipses)
Degree 3:  Captures cubic patterns (S-curves)
Degree 4+: Very flexible but risk of overfitting

    d=1 (linear)       d=2 (quadratic)       d=3 (cubic)
    
    + + |  - -          + +   ╲  - -          + +  ∽∽  - -
    + + |  - -          + + +   ╲ - -         + + + ∽∽ - -
    + + |  - -          + + + +  ╲- -         + + + + ∽- -
    + + |  - -          + + + + + ╲ -         + + + + ∽ - -
    + + |  - -          + + + + +  ╲-         + + + +∽ - -
```

---

## 4. RBF / Gaussian Kernel

### Formula

```
K(x, y) = exp(-γ ‖x - y‖²)

         = exp(-γ Σⱼ (xⱼ - yⱼ)²)

Parameters:
  γ (gamma): Controls the "width" of the Gaussian
              γ = 1/(2σ²) where σ is the standard deviation
```

### Key Properties

```
K(x, y) = exp(-γ ‖x - y‖²)

Properties:
├── K(x, x) = exp(0) = 1         (self-similarity is always 1)
├── 0 < K(x, y) ≤ 1              (similarity is always positive)
├── K → 1 as x → y               (identical points = max similarity)
├── K → 0 as ‖x-y‖ → ∞           (distant points = zero similarity)
├── Feature space: ∞-dimensional   (can represent ANY boundary)
└── Mercer's condition: Always PSD ✓
```

### Interpretation as Similarity

The RBF kernel measures **similarity** between points:

```
K(x, y) ≈ 1:   Points are very similar (close together)
K(x, y) ≈ 0:   Points are very different (far apart)

Think of each support vector as a "beacon":
├── Each SV creates a "bump" of influence around it
├── The height of the bump depends on αᵢ
├── γ controls how quickly the influence drops off
└── f(x) = Σᵢ αᵢyᵢ exp(-γ‖xᵢ-x‖²) + b
```

### Why RBF is the Most Popular Kernel

```
1. UNIVERSAL APPROXIMATOR
   └── Can approximate any continuous function (given enough SVs)

2. SINGLE PARAMETER
   └── Only γ to tune (vs polynomial with γ, r, d)

3. BOUNDED OUTPUT
   └── K ∈ (0, 1] — well-behaved numerically

4. LOCALITY
   └── Each SV has local influence — robust to distant points

5. NO ASSUMPTION
   └── Works well even when you don't know the data structure
```

### RBF Feature Space

The RBF kernel corresponds to an **infinite-dimensional** feature map:

```
K(x, y) = exp(-γ‖x-y‖²)

Taylor expansion:
= exp(-γ‖x‖²) · exp(-γ‖y‖²) · exp(2γ x·y)

= exp(-γ‖x‖²) · exp(-γ‖y‖²) · Σ (2γ)ᵏ(x·y)ᵏ / k!
                                 k=0

Each term (x·y)ᵏ generates a polynomial feature of degree k.
The sum goes to ∞ → feature space is ∞-dimensional!
```

---

## 5. Sigmoid Kernel

### Formula

```
K(x, y) = tanh(γ · x · y + r)

Parameters:
  γ (gamma): Scaling factor
  r (coef0): Shift parameter
```

### Connection to Neural Networks

```
The sigmoid kernel makes SVM behave similarly to a
two-layer neural network (perceptron):

Input ──→ [Σ weights · inputs + bias] ──→ tanh(·) ──→ Output
          ──────────────────────────       ────────
          This is γ·x·y + r              This is the sigmoid kernel!

Each support vector acts like a hidden neuron.
```

### Important Caveat

```
⚠️  WARNING: The sigmoid kernel is NOT always a valid Mercer kernel!

K(x,y) = tanh(γ·x·y + r) is NOT PSD for all γ and r combinations.

It only satisfies Mercer's condition for certain parameter ranges.
Despite this, it often works well in practice.

Valid when: γ > 0 and r < 0 (approximately)

Recommendation: Use RBF instead (always valid, usually better performance)
```

### When to Use Sigmoid Kernel

```
Rarely used in practice because:
├── Not always a valid kernel (Mercer's condition may fail)
├── RBF usually performs equally well or better
├── If you want neural network behavior → just use a neural network
└── Mainly of theoretical interest (SVM-NN connection)
```

---

## 6. The γ (Gamma) Parameter Effect

### What γ Controls

γ determines the **reach** or **influence** of each training point:

```
Large γ:                           Small γ:
├── Each point has SMALL influence  ├── Each point has LARGE influence
├── Decision boundary is COMPLEX    ├── Decision boundary is SMOOTH
├── Model fits training data well   ├── Model is more generalized
├── Risk of OVERFITTING             ├── Risk of UNDERFITTING
└── Like looking at data too close  └── Like looking at data from far away
```

### Visual Effect of γ on RBF

```
γ = 0.01 (Very small)          γ = 1 (Moderate)           γ = 100 (Very large)

  + + + + + + + + +            + + + + + + + + +           + + + + + + + + +
  + + + + + + + + +            + + + + + ╱+ + +            + + +⊕+ + + + +
  + + + + + + + + +            + + + + ╱ + + + +           + + +│⊕+ + + +
  + + + + + + + + +            + + + ╱+ + + + +            + +⊕+│+ + + +
  ─────────────────            + + ╱ + + + + + +           + +│ ⊕│+ + + +
  - - - - - - - - -            - ╱ - - - - - - -           - -│⊕ │- - - -
  - - - - - - - - -            - - - - - - - - -           - -⊕- │- - - -
  - - - - - - - - -            - - - - - - - - -           - - - ⊕- - - -
  
  UNDERFITTING                 GOOD FIT                    OVERFITTING
  Nearly linear               Smooth nonlinear            Wraps around each SV
  boundary                    boundary                    individually
  
  Train acc: 85%              Train acc: 95%              Train acc: 100%
  Test acc:  84%              Test acc:  93%              Test acc:  70%
```

### γ in Different Kernels

| Kernel | γ role | Effect of increasing γ |
|--------|--------|----------------------|
| **Polynomial** | K = (**γ**·x·y + r)^d | Scales dot product → changes curvature |
| **RBF** | K = exp(-**γ**‖x-y‖²) | Narrows Gaussian → each SV has smaller influence |
| **Sigmoid** | K = tanh(**γ**·x·y + r) | Steepens transition → sharper boundary |

### sklearn Default γ Values

```python
# sklearn gamma options:
gamma='scale'   # γ = 1 / (n_features × X.var())  ← DEFAULT (since 0.22)
gamma='auto'    # γ = 1 / n_features
gamma=0.1       # Custom fixed value
```

### Bias-Variance Tradeoff with γ

```
                    ┌──── Optimal γ ────┐
                    │                   │
    Error ↑         │                   │
          │  ╲      │      ╱            │
          │   ╲     │     ╱             │
          │    ╲    │    ╱              │
          │     ╲ Test Error            │
          │      ╲──│──╱               │
          │         │                   │
          │         │  ╱ Train Error    │
          │         │ ╱                 │
          │─────────│╱──────────────→ γ │
          │         │                   │
          │  Underfit│  Overfit         │
          │  (low γ) │  (high γ)       │
```

---

## 7. Choosing the Right Kernel — Flowchart

```
                    START
                      │
                      ▼
              ┌───────────────┐
              │ How many       │
              │ features?      │
              └───────┬───────┘
                      │
              ┌───────┴───────┐
              │               │
              ▼               ▼
        n >> N             n ≈ N or n << N
    (many features,      (moderate features,
     few samples)         adequate samples)
              │               │
              ▼               ▼
        ┌──────────┐   ┌───────────────┐
        │ Try       │   │ Is data       │
        │ LINEAR    │   │ linearly      │
        │ kernel    │   │ separable?    │
        │ first!    │   └───────┬───────┘
        └──────────┘           │
                         ┌─────┴─────┐
                         │           │
                         ▼           ▼
                        YES          NO
                         │           │
                         ▼           ▼
                   ┌──────────┐ ┌──────────┐
                   │ LINEAR    │ │ Try RBF   │
                   │ kernel    │ │ first!    │
                   └──────────┘ └──────┬───┘
                                       │
                                       ▼
                                ┌───────────────┐
                                │ RBF works     │
                                │ well enough?  │
                                └───────┬───────┘
                                        │
                                 ┌──────┴──────┐
                                 │             │
                                 ▼             ▼
                                YES            NO
                                 │             │
                                 ▼             ▼
                           ┌──────────┐  ┌──────────┐
                           │ Use RBF!  │  │ Try POLY  │
                           └──────────┘  │ (degree   │
                                         │  2 or 3)  │
                                         └──────────┘


    RULE OF THUMB:
    ┌────────────────────────────────────────────────┐
    │  1. Start with LINEAR (fast, interpretable)    │
    │  2. If not good → try RBF (most flexible)     │
    │  3. If still not good → try POLYNOMIAL        │
    │  4. Use cross-validation to compare!          │
    └────────────────────────────────────────────────┘
```

### Quick Decision Guide

| Scenario | Recommended Kernel | Reason |
|----------|-------------------|--------|
| Text classification (n=10000+) | **Linear** | Already high-dimensional |
| Image data (moderate n) | **RBF** | Complex nonlinear patterns |
| Tabular data, unknown structure | **RBF** | Universal approximator |
| Known polynomial relationship | **Polynomial** | Matches data generating process |
| Gene expression (n>>N) | **Linear** | More features than samples |
| Small dataset, nonlinear | **RBF** or **Poly** | Need nonlinear boundary |

---

## 8. Kernel Comparison on Different Datasets

### Performance on Classic Datasets

```
Dataset: MOONS (half-circle shaped)
┌──────────┬──────────┬───────────┬──────────┐
│ Kernel   │ Accuracy │ # SVs     │ Margin   │
├──────────┼──────────┼───────────┼──────────┤
│ Linear   │ 88.0%    │ 45        │ 0.632    │
│ Poly d=2 │ 93.0%    │ 32        │ N/A      │
│ Poly d=3 │ 96.5%    │ 28        │ N/A      │
│ RBF      │ 97.5%    │ 25        │ N/A      │
│ Sigmoid  │ 86.0%    │ 50        │ N/A      │
└──────────┴──────────┴───────────┴──────────┘

Dataset: CIRCLES (concentric)
┌──────────┬──────────┬───────────┬──────────┐
│ Kernel   │ Accuracy │ # SVs     │ Margin   │
├──────────┼──────────┼───────────┼──────────┤
│ Linear   │ 50.0%    │ 150       │ 0.012    │
│ Poly d=2 │ 99.0%    │ 15        │ N/A      │
│ Poly d=3 │ 99.0%    │ 18        │ N/A      │
│ RBF      │ 99.5%    │ 12        │ N/A      │
│ Sigmoid  │ 50.0%    │ 148       │ N/A      │
└──────────┴──────────┴───────────┴──────────┘

Dataset: LINEARLY SEPARABLE (blobs)
┌──────────┬──────────┬───────────┬──────────┐
│ Kernel   │ Accuracy │ # SVs     │ Margin   │
├──────────┼──────────┼───────────┼──────────┤
│ Linear   │ 100.0%   │ 3         │ 2.134    │
│ Poly d=2 │ 100.0%   │ 5         │ N/A      │
│ Poly d=3 │ 100.0%   │ 7         │ N/A      │
│ RBF      │ 100.0%   │ 6         │ N/A      │
│ Sigmoid  │ 99.0%    │ 8         │ N/A      │
└──────────┴──────────┴───────────┴──────────┘

Key Insight: RBF is the most consistently good performer,
but Linear is best when data IS linearly separable (fewer SVs).
```

---

## 9. sklearn Kernel Parameters

### Complete Parameter Reference

```python
from sklearn.svm import SVC

svm = SVC(
    # ── Core Parameters ──
    kernel='rbf',        # 'linear', 'poly', 'rbf', 'sigmoid', 'precomputed'
    C=1.0,               # Regularization (soft margin penalty)
    
    # ── Kernel-specific Parameters ──
    gamma='scale',       # Kernel coefficient for 'rbf', 'poly', 'sigmoid'
                         # 'scale': 1/(n_features * X.var())
                         # 'auto':  1/n_features
                         # float:   custom value
    
    degree=3,            # Polynomial degree (only for kernel='poly')
    coef0=0.0,           # Independent term r (for 'poly' and 'sigmoid')
    
    # ── Other Parameters ──
    shrinking=True,      # Use shrinking heuristic (speeds up training)
    probability=False,   # Enable probability estimates (Platt scaling)
    tol=1e-3,            # Tolerance for stopping criterion
    cache_size=200,      # Kernel cache size in MB
    class_weight=None,   # 'balanced' or dict for imbalanced classes
    max_iter=-1,         # Max iterations (-1 = no limit)
    random_state=None,   # For probability estimation reproducibility
)
```

### Parameter Applicability by Kernel

```
┌─────────────┬────────┬──────┬─────┬─────────┐
│ Parameter   │ Linear │ Poly │ RBF │ Sigmoid │
├─────────────┼────────┼──────┼─────┼─────────┤
│ C           │   ✅   │  ✅  │ ✅  │   ✅    │
│ gamma       │   ❌   │  ✅  │ ✅  │   ✅    │
│ degree      │   ❌   │  ✅  │ ❌  │   ❌    │
│ coef0       │   ❌   │  ✅  │ ❌  │   ✅    │
└─────────────┴────────┴──────┴─────┴─────────┘

Legend: ✅ = Used by this kernel, ❌ = Ignored
```

### gamma, degree, and coef0 Details

```
┌──────────────────────────────────────────────────────────┐
│ GAMMA (γ)                                                │
│                                                          │
│ Controls influence of individual training examples:      │
│                                                          │
│ gamma='scale' → γ = 1/(n_features × X.var())            │
│   - Automatically adapts to data scale                   │
│   - sklearn DEFAULT since v0.22                          │
│   - Good starting point                                  │
│                                                          │
│ gamma='auto'  → γ = 1/n_features                        │
│   - Older default, simpler                               │
│   - Doesn't account for data variance                    │
│                                                          │
│ gamma=float   → Custom γ value                           │
│   - Use with GridSearchCV for tuning                     │
│   - Typical range: [10⁻⁴, 10²] on log scale            │
├──────────────────────────────────────────────────────────┤
│ DEGREE (d)  — Polynomial kernel only                     │
│                                                          │
│ degree=2: Quadratic boundary (parabolas, ellipses)       │
│ degree=3: Cubic boundary (S-curves) [default]            │
│ degree=4+: Very flexible (risk of overfitting)           │
│                                                          │
│ Higher degree → more parameters → more complex model     │
├──────────────────────────────────────────────────────────┤
│ COEF0 (r)  — Polynomial and Sigmoid kernels              │
│                                                          │
│ coef0=0: Homogeneous kernel (only highest-degree terms)  │
│ coef0>0: Inhomogeneous kernel (includes lower-degree)    │
│                                                          │
│ For polynomial: K = (γ·x·y + r)^d                       │
│ For sigmoid:    K = tanh(γ·x·y + r)                     │
└──────────────────────────────────────────────────────────┘
```

---

## 10. Python Implementation

### Comprehensive Kernel Comparison

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_moons, make_circles, make_blobs
from sklearn.model_selection import cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# ── Generate datasets ──
datasets = {
    'Moons': make_moons(n_samples=300, noise=0.2, random_state=42),
    'Circles': make_circles(n_samples=300, noise=0.1, factor=0.3, random_state=42),
    'Blobs': make_blobs(n_samples=300, centers=2, cluster_std=1.5, random_state=42),
}

# ── Define kernels ──
kernels = {
    'Linear': SVC(kernel='linear', C=1.0),
    'Poly (d=2)': SVC(kernel='poly', degree=2, C=1.0, coef0=1),
    'Poly (d=3)': SVC(kernel='poly', degree=3, C=1.0, coef0=1),
    'RBF': SVC(kernel='rbf', C=1.0, gamma='scale'),
    'Sigmoid': SVC(kernel='sigmoid', C=1.0, gamma='scale', coef0=0),
}

# ── Compare ──
print(f"{'Dataset':<12} | {'Kernel':<12} | {'CV Accuracy':>12} | {'# SVs':>6}")
print("-" * 55)

for data_name, (X, y) in datasets.items():
    X_scaled = StandardScaler().fit_transform(X)
    for kern_name, model in kernels.items():
        cv_scores = cross_val_score(model, X_scaled, y, cv=5, scoring='accuracy')
        model.fit(X_scaled, y)
        n_svs = sum(model.n_support_)
        print(f"{data_name:<12} | {kern_name:<12} | {cv_scores.mean():>11.4f} | {n_svs:>6}")
    print("-" * 55)
```

### Visualizing Kernel Decision Boundaries

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_moons
from sklearn.preprocessing import StandardScaler

X, y = make_moons(n_samples=200, noise=0.2, random_state=42)
X = StandardScaler().fit_transform(X)

kernels = [
    ('Linear', {'kernel': 'linear'}),
    ('Poly d=2', {'kernel': 'poly', 'degree': 2, 'coef0': 1}),
    ('Poly d=3', {'kernel': 'poly', 'degree': 3, 'coef0': 1}),
    ('RBF γ=0.5', {'kernel': 'rbf', 'gamma': 0.5}),
    ('RBF γ=5', {'kernel': 'rbf', 'gamma': 5}),
    ('Sigmoid', {'kernel': 'sigmoid', 'gamma': 0.1, 'coef0': 0}),
]

fig, axes = plt.subplots(2, 3, figsize=(18, 10))

for ax, (name, params) in zip(axes.ravel(), kernels):
    svm = SVC(C=1.0, **params)
    svm.fit(X, y)
    
    h = 0.02
    x_min, x_max = X[:, 0].min()-1, X[:, 0].max()+1
    y_min, y_max = X[:, 1].min()-1, X[:, 1].max()+1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                         np.arange(y_min, y_max, h))
    Z = svm.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
    
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
    ax.scatter(X[y==0, 0], X[y==0, 1], c='red', s=30, edgecolors='k', alpha=0.7)
    ax.scatter(X[y==1, 0], X[y==1, 1], c='blue', s=30, edgecolors='k', alpha=0.7)
    ax.scatter(svm.support_vectors_[:, 0], svm.support_vectors_[:, 1],
               s=100, facecolors='none', edgecolors='lime', linewidths=1.5)
    
    acc = svm.score(X, y)
    ax.set_title(f'{name}\nAcc: {acc:.2%}, SVs: {sum(svm.n_support_)}')
    ax.set_xlim(x_min, x_max)
    ax.set_ylim(y_min, y_max)

plt.suptitle('Kernel Comparison on Moons Dataset', fontsize=16, y=1.02)
plt.tight_layout()
plt.savefig('kernel_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

### Grid Search for Best Kernel + Parameters

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV
from sklearn.datasets import load_breast_cancer
import numpy as np

# Load data
X, y = load_breast_cancer(return_X_y=True)

# Pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC())
])

# Comprehensive parameter grid
param_grid = [
    # Linear kernel: only tune C
    {
        'svm__kernel': ['linear'],
        'svm__C': [0.01, 0.1, 1, 10, 100],
    },
    # RBF kernel: tune C and gamma
    {
        'svm__kernel': ['rbf'],
        'svm__C': [0.1, 1, 10, 100],
        'svm__gamma': [0.001, 0.01, 0.1, 1, 'scale'],
    },
    # Polynomial kernel: tune C, degree, coef0
    {
        'svm__kernel': ['poly'],
        'svm__C': [0.1, 1, 10],
        'svm__degree': [2, 3, 4],
        'svm__coef0': [0, 1],
        'svm__gamma': ['scale'],
    },
]

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy',
                    n_jobs=-1, return_train_score=True, verbose=0)
grid.fit(X, y)

print(f"Best parameters: {grid.best_params_}")
print(f"Best CV accuracy: {grid.best_score_:.4f}")

# Top 5 configurations
results = grid.cv_results_
top_5 = np.argsort(results['mean_test_score'])[-5:][::-1]
print("\nTop 5 configurations:")
for rank, idx in enumerate(top_5, 1):
    print(f"  {rank}. Acc={results['mean_test_score'][idx]:.4f} "
          f"± {results['std_test_score'][idx]:.4f} | "
          f"{results['params'][idx]}")
```

### RBF Gamma Effect Visualization

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_moons
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score

X, y = make_moons(n_samples=200, noise=0.2, random_state=42)
X = StandardScaler().fit_transform(X)

gammas = [0.01, 0.1, 0.5, 1, 5, 50]

fig, axes = plt.subplots(2, 3, figsize=(18, 10))

for ax, gamma in zip(axes.ravel(), gammas):
    svm = SVC(kernel='rbf', C=1.0, gamma=gamma)
    svm.fit(X, y)
    
    h = 0.02
    xx, yy = np.meshgrid(np.arange(X[:,0].min()-1, X[:,0].max()+1, h),
                         np.arange(X[:,1].min()-1, X[:,1].max()+1, h))
    Z = svm.decision_function(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
    
    ax.contourf(xx, yy, Z, levels=20, cmap='RdBu', alpha=0.5)
    ax.contour(xx, yy, Z, levels=[0], colors='black', linewidths=2)
    ax.scatter(X[y==0, 0], X[y==0, 1], c='red', s=20, edgecolors='k')
    ax.scatter(X[y==1, 0], X[y==1, 1], c='blue', s=20, edgecolors='k')
    
    cv_acc = cross_val_score(svm, X, y, cv=5).mean()
    ax.set_title(f'γ = {gamma}\nTrain: {svm.score(X,y):.2%}, '
                 f'CV: {cv_acc:.2%}, SVs: {sum(svm.n_support_)}')

plt.suptitle('Effect of γ on RBF Kernel SVM', fontsize=16, y=1.02)
plt.tight_layout()
plt.savefig('gamma_effect.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 11. Applications

### Kernel Selection in Practice

| Domain | Typical Kernel | Reasoning |
|--------|---------------|-----------|
| **Text / NLP** | Linear | Very high-dimensional sparse vectors |
| **Computer Vision** | RBF, Histogram Intersection | Nonlinear pixel relationships |
| **Bioinformatics** | Linear, String kernels | High-dim gene data, sequence data |
| **Finance** | RBF | Complex nonlinear market patterns |
| **Speech Recognition** | RBF | Temporal nonlinear features |
| **Chemical Analysis** | RBF, Tanimoto kernel | Molecular similarity |

### Feature Scaling Importance

```
⚠️  CRITICAL: Always scale features before using kernel SVM!

Why? Kernels depend on distances/dot products.
Unscaled features can dominate the kernel computation.

Example:
Feature 1: Age (range 0-100)
Feature 2: Income (range 0-1,000,000)

Without scaling: ‖x-y‖² ≈ (income_diff)²  (age is negligible)
With scaling:    ‖x-y‖² balances both features

Always use StandardScaler or MinMaxScaler BEFORE SVM!
```

---

## 12. Summary Table

| Kernel | Formula | Parameters | Feature Space | Best For |
|--------|---------|-----------|---------------|----------|
| **Linear** | x · y | None | ℝⁿ (same) | High-dim, separable data |
| **Polynomial** | (γ·x·y + r)^d | γ, r, d | ℝ^C(n+d,d) | Known polynomial patterns |
| **RBF** | exp(-γ‖x-y‖²) | γ | ℝ^∞ | General nonlinear data |
| **Sigmoid** | tanh(γ·x·y + r) | γ, r | Not always valid | Rare (NN connection) |

| Parameter | Controls | Low Value | High Value |
|-----------|----------|-----------|------------|
| **C** | Error penalty | Wide margin (underfit) | Narrow margin (overfit) |
| **γ** | SV influence radius | Smooth boundary (underfit) | Complex boundary (overfit) |
| **degree** | Polynomial complexity | Simple curves | Complex curves |
| **coef0** | Lower-degree inclusion | Only highest degree | All degrees included |

---

## 13. Revision Questions

### Q1: Kernel Selection
**You have a dataset with 50,000 features and 500 samples (e.g., gene expression data). Which kernel would you choose first and why?**

<details>
<summary>Answer</summary>

**Linear kernel** is the best starting choice because:
1. With n=50,000 >> N=500, the data likely lives in a low-dimensional subspace and is probably linearly separable in the high-dimensional space (Cover's theorem)
2. Linear SVM is much faster to train: O(N²) vs O(N³) for kernel SVM
3. The resulting model is interpretable — weight vector w shows feature importance
4. Using a nonlinear kernel with so few samples risks severe overfitting
5. `LinearSVC` can solve the primal directly, which is efficient when n >> N

If linear SVM doesn't perform well, try RBF with very small γ (strong regularization).

</details>

### Q2: γ Parameter
**Explain what happens when you set γ = 1000 for an RBF kernel SVM. How would the training accuracy, test accuracy, and number of support vectors likely change compared to γ = 1?**

<details>
<summary>Answer</summary>

With γ = 1000 (very large):
- **Training accuracy**: Likely **100%** or very close. Each support vector creates a tiny "island" of influence, allowing the model to wrap around every training point individually.
- **Test accuracy**: Likely **much lower** (severe overfitting). The model memorizes training data instead of learning generalizable patterns.
- **Number of support vectors**: **Increases significantly** — almost every training point becomes a support vector because the influence of each point is so localized.
- **Decision boundary**: Extremely complex, jagged, with many isolated regions around individual points.

Compared to γ = 1:
- γ=1 would give a smoother boundary, fewer SVs, lower training acc but much better test acc.

</details>

### Q3: Mathematical
**Show that the polynomial kernel K(x,y) = (x·y)³ for x,y ∈ ℝ² corresponds to a feature map in ℝ⁴.**

<details>
<summary>Answer</summary>

For x = (x₁,x₂) and y = (y₁,y₂):

K(x,y) = (x₁y₁ + x₂y₂)³

Expanding using binomial theorem:
= x₁³y₁³ + 3x₁²x₂·y₁²y₂ + 3x₁x₂²·y₁y₂² + x₂³y₂³

This is the dot product of:
φ(x) = (x₁³, √3·x₁²x₂, √3·x₁x₂², x₂³)
φ(y) = (y₁³, √3·y₁²y₂, √3·y₁y₂², y₂³)

Verification:
φ(x)·φ(y) = x₁³y₁³ + 3x₁²x₂y₁²y₂ + 3x₁x₂²y₁y₂² + x₂³y₂³ = (x·y)³ ✓

So K maps from ℝ² to ℝ⁴.

In general, a homogeneous polynomial kernel of degree d in ℝⁿ maps to ℝ^C(n+d-1,d).

</details>

### Q4: Practical Comparison
**Compare the decision boundaries produced by linear, polynomial (d=2), and RBF kernels on an XOR-like dataset. Which would work best and why?**

<details>
<summary>Answer</summary>

**XOR dataset**: Points at (0,0) and (1,1) are class +1; points at (0,1) and (1,0) are class -1.

- **Linear**: **Fails completely** (~50% accuracy). No line can separate XOR data. The classes are interleaved in 2D.

- **Polynomial d=2**: **Works well**. The quadratic kernel creates features like x₁², x₂², x₁x₂. The interaction term x₁x₂ is key: it's high (positive) for (1,1) and low/zero for (0,1) and (1,0), enabling separation.

- **RBF**: **Works well** with appropriate γ. Creates localized "bumps" around each support vector, naturally handling the XOR pattern.

**Best choice**: Both Poly d=2 and RBF work. Poly d=2 is more efficient since it directly captures the interaction term needed for XOR. RBF is more general but may need more support vectors.

</details>

### Q5: sklearn Parameter Tuning
**Write Python code to find the optimal kernel and parameters for an SVM on the Iris dataset using nested cross-validation.**

<details>
<summary>Answer</summary>

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.datasets import load_iris
import numpy as np

X, y = load_iris(return_X_y=True)

pipe = Pipeline([('scaler', StandardScaler()), ('svm', SVC())])

param_grid = [
    {'svm__kernel': ['linear'], 'svm__C': [0.1, 1, 10]},
    {'svm__kernel': ['rbf'], 'svm__C': [0.1, 1, 10], 
     'svm__gamma': [0.01, 0.1, 1]},
    {'svm__kernel': ['poly'], 'svm__C': [0.1, 1, 10], 
     'svm__degree': [2, 3], 'svm__coef0': [0, 1]},
]

# Inner CV for hyperparameter selection
inner_cv = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', n_jobs=-1)

# Outer CV for unbiased performance estimate
outer_scores = cross_val_score(inner_cv, X, y, cv=5, scoring='accuracy')

print(f"Nested CV accuracy: {outer_scores.mean():.4f} ± {outer_scores.std():.4f}")

# Final model selection on full data
inner_cv.fit(X, y)
print(f"Best params: {inner_cv.best_params_}")
print(f"Best inner CV score: {inner_cv.best_score_:.4f}")
```

</details>

### Q6: Conceptual
**Why must you scale features before using an RBF kernel but might not need to for a linear kernel in text classification?**

<details>
<summary>Answer</summary>

**RBF kernel** (K = exp(-γ‖x-y‖²)):
- Depends on **Euclidean distances** between points
- If Feature A ranges [0, 1000] and Feature B ranges [0, 1], then ‖x-y‖² ≈ (diff_A)²
- Feature B becomes almost invisible to the kernel
- **Scaling is essential** to ensure all features contribute equally

**Linear kernel in text classification** (K = x·y):
- Text features (e.g., TF-IDF) are already on similar scales (typically [0, 1])
- The dot product naturally handles sparse high-dimensional vectors
- Scaling can still help but is less critical because:
  - TF-IDF normalization already accounts for feature importance
  - L2 normalization of documents equalizes their magnitudes
  
However, scaling is still **recommended as a best practice** even for linear kernels, especially with non-text features.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Kernel Trick](./04-kernel-trick.md) | [Unit 9: SVM](../) | [SVM Regression →](./06-svm-regression.md) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 5 of 7 — Common Kernels*
