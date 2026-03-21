# 📘 Chapter 7: Regularization in Logistic Regression

> **Unit 5: Logistic Regression** | **Section 7 of 7**
> Prevent overfitting with L1 (Lasso), L2 (Ridge), and ElasticNet regularization. Master the C parameter in sklearn, understand feature selection with L1, tune regularization strength, and build robust logistic regression models.

---

## 📑 Table of Contents

1. [Why Regularization?](#1-why-regularization)
2. [L2 Regularization (Ridge)](#2-l2-regularization-ridge)
3. [L1 Regularization (Lasso)](#3-l1-regularization-lasso)
4. [ElasticNet: Combining L1 and L2](#4-elasticnet-combining-l1-and-l2)
5. [The C Parameter in sklearn](#5-the-c-parameter-in-sklearn)
6. [Effect on Decision Boundary](#6-effect-on-decision-boundary)
7. [Feature Selection with L1](#7-feature-selection-with-l1)
8. [Practical Tuning Guide](#8-practical-tuning-guide)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Why Regularization?

### The Overfitting Problem

```
    Training Accuracy vs Model Complexity:
    
    Accuracy
    │
    │              ···················  ← Training accuracy
    │          ····        
    │        ··          ··········     ← Test accuracy
    │       ·         ···
    │      ·        ··
    │     ·       ··
    │    ·      ··
    │   ·     ··
    │  ·    ·                ← Test accuracy DROPS!
    │ ·   ·
    └──────────────────────────── Model Complexity
    
    │ Underfitting │  Good fit  │  Overfitting  │
    │ (high bias)  │  (balanced)│  (high var.)  │
```

### How Overfitting Manifests in Logistic Regression

```
Without Regularization (Overfitting):

    x₂                              
    │  ● ● ● ╲ ■ ■                  
    │  ● ●  ╱ ╲  ■ ■                
    │  ● ╱ ● ●╲╱■ ■                 Wiggly, complex boundary
    │  ╱ ● ●  ╱╲ ■ ■                that memorizes noise
    │╱ ● ● ╱●  ╲  ■                 
    └────────────────── x₁           

With Regularization (Good Fit):

    x₂                              
    │  ● ● ● │ ■ ■                  
    │  ● ● ● │ ■ ■                  Simple, smooth boundary
    │  ● ● ● │ ■ ■ ■                that generalizes well
    │  ● ● ● │ ■ ■                  
    │  ● ● ● │ ■ ■                  
    └────────────────── x₁           
```

### When Overfitting Occurs

```
High risk of overfitting when:

1. Many features (n) relative to samples (m)
   n >> m or n ≈ m

2. Polynomial features used
   Original: 5 features → Degree 3: 56 features!

3. No noise in training data interpretation
   Model fits EVERY point perfectly, including noise

4. Large weight magnitudes
   w = [105.3, -87.2, 42.9, ...]
   Large weights → extreme sensitivity to small input changes
```

### The Regularization Idea

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  Original cost:                                               │
│      J = -(1/m) Σ [y·log(ŷ) + (1-y)·log(1-ŷ)]              │
│          └───────────────┬──────────────────┘                │
│                    "fit the data"                             │
│                                                               │
│  Regularized cost:                                            │
│      J = -(1/m) Σ [y·log(ŷ) + (1-y)·log(1-ŷ)] + λ·R(w)    │
│          └───────────────┬──────────────────┘   └───┬───┘    │
│                    "fit the data"              "keep weights │
│                                                  small"      │
│                                                               │
│  λ (lambda) controls the TRADEOFF:                           │
│  • λ = 0:  no regularization (may overfit)                   │
│  • λ → ∞:  all weights → 0 (underfitting)                   │
│  • λ optimal: best generalization                            │
│                                                               │
│  ⚠️ Note: bias b is typically NOT regularized               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. L2 Regularization (Ridge)

### The L2 Penalty

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  R(w) = ‖w‖₂² = Σⱼ₌₁ⁿ wⱼ²                                 │
│                                                               │
│  Regularized cost:                                            │
│  J = -(1/m) Σ [y·log(ŷ) + (1-y)·log(1-ŷ)] + (λ/2m) Σ wⱼ² │
│                                                               │
│  The (1/2m) is a convenience factor for cleaner gradients    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Effect on Weights

```
L2 penalty SHRINKS all weights toward zero,
but never exactly to zero:

    Without L2:                With L2 (λ=1):
    w = [5.2, -3.8, 0.1,      w = [2.1, -1.5, 0.04,
         12.7, -0.3]                4.8, -0.12]
    
    Large weights become       ALL weights are SMALLER
    dominant                   but NONE are exactly zero

    ┌─────────┬─────────┐
    │ Feature │ Weight  │
    │─────────│─────────│
    │   w₁    │ ████▏   │  5.2  →  ██▏    2.1
    │   w₂    │ ███▏    │ -3.8  → -█▌    -1.5
    │   w₃    │ ▏       │  0.1  →  ▏      0.04
    │   w₄    │ ████████│ 12.7  →  ████▏  4.8
    │   w₅    │ ▏       │ -0.3  → -▏     -0.12
    └─────────┴─────────┘
    
    All weights shrink proportionally.
    No weight becomes exactly zero.
```

### Gradient with L2 Regularization

```
∂J/∂wⱼ = (1/m) Σ (ŷᵢ - yᵢ)·xᵢⱼ + (λ/m)·wⱼ
                  └──────┬──────┘    └──┬──┘
              original gradient    regularization
                                   "weight decay"

Update rule:
    wⱼ := wⱼ - α · [(1/m) Σ (ŷᵢ - yᵢ)·xᵢⱼ + (λ/m)·wⱼ]
    
    = wⱼ(1 - αλ/m) - α·(1/m) Σ (ŷᵢ - yᵢ)·xᵢⱼ
      └────┬────┘
    "weight decay" factor
    (shrinks w slightly each step)
```

### Geometric Interpretation

```
L2 constraint region is a CIRCLE (sphere in higher dimensions):

    w₂
    │     ╱──╲         ← L2 constraint: w₁² + w₂² ≤ r²
    │   ╱╱    ╲╲
    │  ╱╱   ●  ╲╲     ← optimal w (constrained)
    │ ╱╱  ╱    ╲╲╲
    │ ╱ ╱       ╲ ╲
    │╱╱     ○     ╲╲   ← unconstrained optimum (would overfit)
    │╱╲           ╱╲
    │ ╲ ╲       ╱ ╱
    │  ╲╲       ╱╱
    │   ╲╲    ╱╱
    │     ╲──╱
    └──────────────── w₁
    
    The regularized solution ● is where the cost function
    contours first TOUCH the constraint circle.
    
    Result: weights are pulled toward zero but stay on
    a smooth circle — no weights are exactly zero.
```

---

## 3. L1 Regularization (Lasso)

### The L1 Penalty

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  R(w) = ‖w‖₁ = Σⱼ₌₁ⁿ |wⱼ|                                 │
│                                                               │
│  Regularized cost:                                            │
│  J = -(1/m) Σ [y·log(ŷ) + (1-y)·log(1-ŷ)] + (λ/m) Σ|wⱼ|  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### The Key Difference: Sparsity

```
L1 DRIVES weights exactly to zero → FEATURE SELECTION!

    Without L1:                With L1 (λ=1):
    w = [5.2, -3.8, 0.1,      w = [3.1, -1.8, 0.0,
         12.7, -0.3]                8.2,  0.0]
    
    ┌─────────┬─────────┐
    │ Feature │ Weight  │
    │─────────│─────────│
    │   w₁    │ ████▏   │  5.2  →  ███    3.1
    │   w₂    │ ███▏    │ -3.8  → -█▊    -1.8
    │   w₃    │ ▏       │  0.1  →         0.0  ← ELIMINATED!
    │   w₄    │ ████████│ 12.7  →  ██████ 8.2
    │   w₅    │ ▏       │ -0.3  →         0.0  ← ELIMINATED!
    └─────────┴─────────┘
    
    Some weights become EXACTLY ZERO.
    → Automatic feature selection!
    → Sparser, more interpretable model.
```

### Why L1 Produces Zeros (Geometric Intuition)

```
L1 constraint region is a DIAMOND (corners touch axes):

    w₂
    │       ╱╲         ← L1 constraint: |w₁| + |w₂| ≤ r
    │     ╱    ╲
    │   ╱   ●    ╲     ← optimal w often at a CORNER!
    │ ╱  ╱         ╲
    ├╱╱             ╲──── w₁
    │╲               ╱
    │  ╲           ╱
    │    ╲       ╱
    │      ╲   ╱
    │        ╲╱
    
    Cost function contours (ellipses) tend to touch the
    diamond at CORNERS, where one or more wⱼ = 0.
    
    The diamond has sharp corners on the axes.
    The circle (L2) is smooth everywhere.
    
    This is why L1 produces sparse solutions!
```

### Comparison: L1 vs L2 Side by Side

```
                L2 (Ridge)                    L1 (Lasso)
    
    w₂         ╱──╲                  w₂       ╱╲
    │        ╱╱    ╲╲                │      ╱    ╲
    │  ●   ╱╱      ╲╲               │    ╱       ╲
    │     ╱╱ ○      ╲╲              │  ╱    ○     ╲ ←●(w₂=0!)
    │    ╱╱          ╲╲             ├╱              ╲
    ├───╱╱────────────╲╲──          │╲               ╱
    │    ╲╲          ╱╱             │  ╲            ╱
    │     ╲╲──────╱╱               │    ╲        ╱
    │                                │      ╲──╱
    
    ● = constrained optimum          ● = constrained optimum
    (on the circle, not at axis)     (at the corner! w₂=0)
    → No weights exactly zero        → Feature w₂ eliminated
```

---

## 4. ElasticNet: Combining L1 and L2

### The ElasticNet Penalty

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  R(w) = ρ · ‖w‖₁ + (1-ρ)/2 · ‖w‖₂²                        │
│                                                               │
│  where ρ ∈ [0, 1] is the L1 ratio:                           │
│    ρ = 1.0  → pure L1 (Lasso)                               │
│    ρ = 0.0  → pure L2 (Ridge)                               │
│    ρ = 0.5  → equal mix                                      │
│                                                               │
│  Full cost:                                                   │
│  J = log_loss + λ · [ρ·Σ|wⱼ| + (1-ρ)/2·Σwⱼ²]              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### When to Use ElasticNet

```
┌──────────────────────────────────────────────────────────────┐
│ Scenario                          │ Best Regularization      │
│───────────────────────────────────│──────────────────────────│
│ Few features, all important       │ L2 (Ridge)               │
│ Many features, few important      │ L1 (Lasso)               │
│ Correlated features               │ ElasticNet               │
│ Feature selection needed           │ L1 or ElasticNet         │
│ Interpretability important         │ L1 (sparse model)        │
│ Maximum accuracy                   │ L2 or ElasticNet + CV    │
│ Groups of correlated features      │ ElasticNet (selects      │
│                                   │   groups together)        │
└──────────────────────────────────────────────────────────────┘
```

### Constraint Region Shapes

```
L2 (Circle)        ElasticNet          L1 (Diamond)
    
    ╱──╲              ╱─╲               ╱╲
  ╱╱    ╲╲          ╱╱   ╲╲           ╱    ╲
 ╱╱      ╲╲       ╱╱      ╲╲       ╱        ╲
╱╱        ╲╲     ╱╱        ╲╲    ╱            ╲
╲╲        ╱╱     ╲╲        ╱╱    ╲            ╱
 ╲╲      ╱╱       ╲╲      ╱╱       ╲        ╱
  ╲╲    ╱╱          ╲╲   ╱╱           ╲    ╱
    ╲──╱              ╲─╱               ╲╱

Smooth,              Rounded diamond    Sharp corners,
no sparsity          Some sparsity      maximum sparsity

ρ = 0                ρ = 0.5            ρ = 1
```

---

## 5. The C Parameter in sklearn

### C vs λ: The Inverse Relationship

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│  sklearn uses C = 1/λ  (INVERSE of regularization strength)  │
│                                                               │
│  C = 1/λ:                                                    │
│  • Large C → small λ → LESS regularization → complex model   │
│  • Small C → large λ → MORE regularization → simple model    │
│                                                               │
│  sklearn cost:                                                │
│  J = C · log_loss + regularization_term                      │
│                                                               │
│  Equivalently:                                                │
│  J = log_loss + (1/C) · regularization_term                  │
│                                                               │
│  So (1/C) = λ                                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### C Value Guide

```
    C value:   0.001  0.01   0.1    1.0    10    100   1000
               ──────────────────────────────────────────────
    λ value:   1000   100    10     1.0    0.1   0.01  0.001
               ──────────────────────────────────────────────
    Effect:    Very    Strong  Moderate  Default  Weak   Very   Almost
               strong  reg.    reg.      reg.    reg.   weak   no reg.
               ──────────────────────────────────────────────
               Simple model ←──────────────→ Complex model
               Underfitting ←──────────────→ Overfitting

    
    Default: C = 1.0 (moderate regularization)
    
    ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐
    │C=0.01│     │C=0.1 │     │ C=1  │     │C=100 │
    │      │     │      │     │      │     │      │
    │  ──  │     │  ╱   │     │ ╱╲   │     │╱╲╱╲  │
    │      │     │ ╱    │     │╱  ╲  │     │    ╲╱│
    │      │     │      │     │    ╲ │     │      │
    └──────┘     └──────┘     └──────┘     └──────┘
    Too simple   Good         Good         Too complex
```

### sklearn LogisticRegression Parameters

```python
from sklearn.linear_model import LogisticRegression

# Full parameter specification
model = LogisticRegression(
    penalty='l2',        # 'l1', 'l2', 'elasticnet', 'none'
    C=1.0,               # Inverse regularization strength (1/λ)
    solver='lbfgs',      # Optimization algorithm
    max_iter=100,         # Maximum iterations
    l1_ratio=None,        # For elasticnet: ratio of L1 (0 to 1)
    random_state=42,
    multi_class='auto',   # 'ovr', 'multinomial', 'auto'
    fit_intercept=True,   # Whether to add bias term
    class_weight=None,    # 'balanced' for imbalanced datasets
)
```

### Penalty-Solver Compatibility (Quick Reference)

```
┌────────────┬──────────┬─────────┬──────────┬──────────┐
│            │ penalty= │ penalty=│ penalty= │ penalty= │
│ Solver     │  'l2'    │  'l1'   │'elasticnet│  'none' │
│────────────│──────────│─────────│──────────│──────────│
│ 'lbfgs'    │   ✓      │    ✗    │    ✗     │    ✓     │
│ 'liblinear'│   ✓      │    ✓    │    ✗     │    ✗     │
│ 'saga'     │   ✓      │    ✓    │    ✓     │    ✓     │
│ 'newton-cg'│   ✓      │    ✗    │    ✗     │    ✓     │
│ 'sag'      │   ✓      │    ✗    │    ✗     │    ✓     │
└────────────┴──────────┴─────────┴──────────┴──────────┘

Key: Use 'saga' for L1 or ElasticNet; 'lbfgs' for L2
```

---

## 6. Effect on Decision Boundary

### How Regularization Changes the Boundary

```
Weak Regularization (C=100):        Strong Regularization (C=0.01):

    x₂                                  x₂
    │  ● ● ●╲╱■ ■ ■                    │  ● ● ● │ ■ ■ ■
    │  ● ● ╱╲╱■ ■ ■                    │  ● ● ● │ ■ ■ ■
    │  ● ╱╲ ●  ╲ ■ ■                   │  ● ● ● │ ■ ■ ■
    │  ╱ ● ●╱╲  ■ ■                    │  ● ● ● │ ■ ■ ■
    │╱ ● ●╱  ╲■ ■ ■                    │  ● ● ● │ ■ ■ ■
    └────────────────── x₁              └────────────────── x₁
    
    Complex, wiggly boundary            Simple, straight boundary
    (fits training noise)               (ignores noise)
    ‖w‖ is LARGE                        ‖w‖ is SMALL

Weight comparison:
    C=100:  w = [12.5, -8.3, 15.7, -6.2, 9.1]
    C=1:    w = [3.2, -2.1, 4.0, -1.5, 2.3]
    C=0.01: w = [0.1, -0.07, 0.13, -0.05, 0.08]
```

### Regularization and Probability Confidence

```
Strong regularization:               Weak regularization:

    P(y=1|x)                            P(y=1|x)
    1.0 ┤        ···■■■■                1.0 ┤              ┌■■■
        │     ···                           │              │
    0.5 ┤─ ─◆─ ─ ─ ─ ─ ─              0.5 ┤─ ─ ─ ─ ─ ─ ─◆─ ─
        │  ···                              │              │
    0.0 ┤■■···                          0.0 ┤■■■────────── ┘
        └────────────── x                   └────────────── x
    
    Gradual transition                  Sharp transition
    (uncertain, regularized)            (over-confident)
    
    Small ‖w‖ → gentle sigmoid         Large ‖w‖ → steep sigmoid
```

---

## 7. Feature Selection with L1

### How L1 Creates Sparse Models

```
L1 regularization is a powerful tool for AUTOMATIC FEATURE SELECTION:

    Before L1 (50 features):
    w = [0.5, -0.3, 2.1, 0.0, -1.5, 0.8, 0.0, 0.0, 3.2, -0.1,
         0.4, 0.0, 0.0, 0.0, 1.7, -0.6, 0.0, 0.0, 0.0, -2.3,
         ... (50 non-zero weights)]
    
    After L1 (C=0.1):
    w = [0.0, 0.0, 1.8, 0.0, -1.2, 0.0, 0.0, 0.0, 2.9, 0.0,
         0.0, 0.0, 0.0, 0.0, 1.4, 0.0, 0.0, 0.0, 0.0, -1.9,
         0.0, 0.0, ... (only 5 non-zero weights!)]
    
    Result: Only features x₃, x₅, x₉, x₁₅, x₂₀ are used!
    The model automatically identified the 5 most important features.
```

### Feature Importance Ranking

```python
import numpy as np
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.preprocessing import StandardScaler

# Load dataset
data = load_breast_cancer()
X, y = data.data, data.target
feature_names = data.feature_names

# Scale features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# L1 model with moderate regularization
model_l1 = LogisticRegression(penalty='l1', C=0.1, solver='saga', max_iter=10000, random_state=42)
model_l1.fit(X_scaled, y)

# Feature importance analysis
weights = model_l1.coef_[0]
n_nonzero = np.sum(weights != 0)
n_total = len(weights)

print(f"L1 Regularization Feature Selection (C=0.1):")
print(f"  Non-zero features: {n_nonzero} / {n_total}")
print(f"  Sparsity: {(1 - n_nonzero/n_total)*100:.1f}%")
print(f"\nSelected Features (non-zero weights):")

# Sort by absolute weight
sorted_idx = np.argsort(np.abs(weights))[::-1]
for idx in sorted_idx:
    if weights[idx] != 0:
        bar = "█" * int(abs(weights[idx]) * 10)
        sign = "+" if weights[idx] > 0 else "-"
        print(f"  {feature_names[idx]:30s} {sign} {abs(weights[idx]):7.4f}  {bar}")
```

### Feature Selection at Different C Values

```
    C=0.001:  2 features selected    ████████████████████████ sparsity
    C=0.01:   5 features selected    ████████████████████ sparsity
    C=0.1:   12 features selected    ████████████ sparsity
    C=1.0:   25 features selected    █████ sparsity
    C=10:    29 features selected    █ sparsity
    C=100:   30 features selected    no sparsity
    
    Smaller C → stronger L1 → more features eliminated
    Larger C  → weaker L1  → more features kept
```

---

## 8. Practical Tuning Guide

### Step-by-Step Tuning Process

```
Step 1: Start with default (L2, C=1.0)
    → Establish baseline performance
    
Step 2: Try a range of C values
    → C ∈ [0.001, 0.01, 0.1, 1, 10, 100, 1000]
    → Use cross-validation
    
Step 3: Choose penalty type
    → L2 if all features are expected to be useful
    → L1 if you want feature selection
    → ElasticNet for correlated features
    
Step 4: Fine-tune C around the best value
    → If best C=1.0, try [0.3, 0.5, 0.7, 1.0, 1.5, 2.0, 3.0]
    
Step 5: Validate on held-out test set
    → Report final performance
```

### Cross-Validation for C Selection

```python
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import numpy as np

# Load and prepare data
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42, stratify=data.target
)

# Create pipeline (scaling + logistic regression)
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('lr', LogisticRegression(max_iter=10000, random_state=42))
])

# Grid search over C and penalty
param_grid = [
    {'lr__penalty': ['l2'], 'lr__C': [0.001, 0.01, 0.1, 1, 10, 100]},
    {'lr__penalty': ['l1'], 'lr__C': [0.001, 0.01, 0.1, 1, 10, 100],
     'lr__solver': ['saga']},
]

grid_search = GridSearchCV(
    pipeline, param_grid, cv=5, scoring='accuracy',
    n_jobs=-1, return_train_score=True
)
grid_search.fit(X_train, y_train)

# Results
print("Grid Search Results:")
print(f"  Best params: {grid_search.best_params_}")
print(f"  Best CV score: {grid_search.best_score_:.4f}")
print(f"  Test score: {grid_search.score(X_test, y_test):.4f}")

# Detailed results
results = grid_search.cv_results_
print(f"\n{'Penalty':>8} {'C':>8} {'Train Acc':>10} {'CV Acc':>10}")
print("─" * 40)
for i in range(len(results['params'])):
    penalty = results['params'][i].get('lr__penalty', 'l2')
    C = results['params'][i]['lr__C']
    train = results['mean_train_score'][i]
    cv = results['mean_test_score'][i]
    marker = " ←" if i == grid_search.best_index_ else ""
    print(f"{penalty:>8} {C:>8.3f} {train:>10.4f} {cv:>10.4f}{marker}")
```

### Quick Decision Flowchart

```
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  START: Choose regularization for logistic regression         │
│                                                               │
│  ┌─── Do you need feature selection? ───┐                    │
│  │                                       │                    │
│  │ YES                                NO │                    │
│  ▼                                       ▼                    │
│  ┌──── Correlated features? ────┐    Use L2 (Ridge)          │
│  │                               │    penalty='l2'            │
│  │ YES              NO           │    C: tune with CV         │
│  ▼                  ▼            │                            │
│  ElasticNet      L1 (Lasso)     │                            │
│  penalty=        penalty='l1'    │                            │
│  'elasticnet'    solver='saga'   │                            │
│  solver='saga'   C: tune w/ CV   │                            │
│  l1_ratio: tune                  │                            │
│  C: tune w/ CV                   │                            │
│                                                               │
│  ALWAYS: Use StandardScaler before fitting!                   │
│  ALWAYS: Use cross-validation to choose C!                    │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### Common Mistakes to Avoid

```
❌ Mistake 1: Not scaling features before regularization
   → Regularization penalizes large weights
   → Unscaled features have different magnitudes
   → Model unfairly penalizes features with small scales

❌ Mistake 2: Using default C=1.0 without tuning
   → C=1.0 is rarely the optimal value
   → Always search over at least [0.01, 0.1, 1, 10, 100]

❌ Mistake 3: Regularizing the bias term
   → The bias should NOT be penalized
   → sklearn handles this correctly by default (fit_intercept=True)

❌ Mistake 4: Using L1 with 'lbfgs' solver
   → Will raise an error! Use solver='saga' or 'liblinear'

❌ Mistake 5: Evaluating on training data
   → Regularization is about generalization
   → ALWAYS evaluate on validation/test data
```

---

## 9. Python Implementation

### Regularized Logistic Regression from Scratch

```python
import numpy as np

class RegularizedLogisticRegression:
    """
    Logistic Regression with L1, L2, or ElasticNet regularization.
    
    Parameters:
    -----------
    penalty : str, 'l1', 'l2', or 'elasticnet'
    C : float, inverse regularization strength (1/λ)
    l1_ratio : float, L1 ratio for ElasticNet (0=L2, 1=L1)
    learning_rate : float
    n_iterations : int
    """
    
    def __init__(self, penalty='l2', C=1.0, l1_ratio=0.5,
                 learning_rate=0.01, n_iterations=1000):
        self.penalty = penalty
        self.C = C
        self.l1_ratio = l1_ratio
        self.lr = learning_rate
        self.n_iter = n_iterations
        self.weights = None
        self.bias = None
        self.cost_history = []
    
    def _sigmoid(self, z):
        return np.where(z >= 0, 1/(1+np.exp(-z)), np.exp(z)/(1+np.exp(z)))
    
    def _compute_cost(self, y, y_pred, w):
        m = len(y)
        y_pred = np.clip(y_pred, 1e-15, 1-1e-15)
        log_loss = -(1/m) * np.sum(y * np.log(y_pred) + (1-y) * np.log(1-y_pred))
        
        # Regularization term
        lam = 1.0 / self.C  # λ = 1/C
        if self.penalty == 'l2':
            reg = (lam / (2*m)) * np.sum(w**2)
        elif self.penalty == 'l1':
            reg = (lam / m) * np.sum(np.abs(w))
        elif self.penalty == 'elasticnet':
            reg = (lam / m) * (
                self.l1_ratio * np.sum(np.abs(w)) +
                (1 - self.l1_ratio) / 2 * np.sum(w**2)
            )
        else:
            reg = 0
        
        return log_loss + reg
    
    def _compute_gradients(self, X, y, y_pred, w):
        m = len(y)
        error = y_pred - y
        dw = (1/m) * (X.T @ error)
        db = (1/m) * np.sum(error)
        
        # Add regularization gradient
        lam = 1.0 / self.C
        if self.penalty == 'l2':
            dw += (lam / m) * w
        elif self.penalty == 'l1':
            dw += (lam / m) * np.sign(w)
        elif self.penalty == 'elasticnet':
            dw += (lam / m) * (
                self.l1_ratio * np.sign(w) +
                (1 - self.l1_ratio) * w
            )
        
        return dw, db
    
    def fit(self, X, y):
        m, n = X.shape
        self.weights = np.zeros(n)
        self.bias = 0.0
        self.cost_history = []
        
        for i in range(self.n_iter):
            z = X @ self.weights + self.bias
            y_pred = self._sigmoid(z)
            
            cost = self._compute_cost(y, y_pred, self.weights)
            self.cost_history.append(cost)
            
            dw, db = self._compute_gradients(X, y, y_pred, self.weights)
            
            self.weights -= self.lr * dw
            self.bias -= self.lr * db
        
        return self
    
    def predict_proba(self, X):
        z = X @ self.weights + self.bias
        p1 = self._sigmoid(z)
        return np.column_stack([1-p1, p1])
    
    def predict(self, X):
        return (self.predict_proba(X)[:, 1] >= 0.5).astype(int)
    
    def score(self, X, y):
        return np.mean(self.predict(X) == y)
    
    def n_nonzero_features(self):
        """Count non-zero weights (for L1 sparsity analysis)."""
        return np.sum(np.abs(self.weights) > 1e-6)
```

### Comprehensive Comparison of Regularization Types

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
import numpy as np

# Load and prepare data
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42, stratify=data.target
)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# ─── Compare L1 vs L2 across different C values ───
print("=" * 70)
print("L1 vs L2 Regularization Comparison")
print("=" * 70)
print(f"\n{'C':>8} {'Penalty':>8} {'Train':>8} {'Test':>8} {'Non-zero':>10} {'‖w‖':>10}")
print("─" * 58)

for C in [0.001, 0.01, 0.1, 1.0, 10.0, 100.0]:
    for penalty, solver in [('l2', 'lbfgs'), ('l1', 'saga')]:
        model = LogisticRegression(
            penalty=penalty, C=C, solver=solver,
            max_iter=10000, random_state=42
        )
        model.fit(X_train_s, y_train)
        
        train_acc = model.score(X_train_s, y_train)
        test_acc = model.score(X_test_s, y_test)
        n_nonzero = np.sum(model.coef_[0] != 0)
        w_norm = np.linalg.norm(model.coef_[0])
        
        print(f"{C:>8.3f} {penalty:>8} {train_acc:>8.4f} {test_acc:>8.4f} "
              f"{n_nonzero:>10d}/30 {w_norm:>10.4f}")

# ─── ElasticNet comparison ───
print(f"\nElasticNet (C=0.1, varying l1_ratio):")
print(f"{'l1_ratio':>10} {'Train':>8} {'Test':>8} {'Non-zero':>10} {'‖w‖':>10}")
print("─" * 45)

for ratio in [0.0, 0.25, 0.5, 0.75, 1.0]:
    model = LogisticRegression(
        penalty='elasticnet', C=0.1, l1_ratio=ratio,
        solver='saga', max_iter=10000, random_state=42
    )
    model.fit(X_train_s, y_train)
    
    n_nonzero = np.sum(model.coef_[0] != 0)
    w_norm = np.linalg.norm(model.coef_[0])
    
    print(f"{ratio:>10.2f} {model.score(X_train_s, y_train):>8.4f} "
          f"{model.score(X_test_s, y_test):>8.4f} "
          f"{n_nonzero:>10d}/30 {w_norm:>10.4f}")
```

### Regularization Path Visualization

```python
# ─── Show how weights change with C (regularization path) ───

C_values = np.logspace(-3, 3, 50)
weights_l2 = []
weights_l1 = []

for C in C_values:
    # L2
    model = LogisticRegression(penalty='l2', C=C, max_iter=10000, random_state=42)
    model.fit(X_train_s, y_train)
    weights_l2.append(model.coef_[0].copy())
    
    # L1
    model = LogisticRegression(penalty='l1', C=C, solver='saga', max_iter=10000, random_state=42)
    model.fit(X_train_s, y_train)
    weights_l1.append(model.coef_[0].copy())

weights_l2 = np.array(weights_l2)
weights_l1 = np.array(weights_l1)

# Report key statistics
print("Regularization Path Summary:")
print(f"\n{'C':>8} {'L2 max|w|':>12} {'L2 nonzero':>12} {'L1 max|w|':>12} {'L1 nonzero':>12}")
print("─" * 60)
for i, C in enumerate([0.001, 0.01, 0.1, 1.0, 10.0, 100.0, 1000.0]):
    idx = np.argmin(np.abs(C_values - C))
    print(f"{C:>8.3f} {np.max(np.abs(weights_l2[idx])):>12.4f} "
          f"{np.sum(np.abs(weights_l2[idx]) > 1e-6):>12d}/30 "
          f"{np.max(np.abs(weights_l1[idx])):>12.4f} "
          f"{np.sum(np.abs(weights_l1[idx]) > 1e-6):>12d}/30")
```

### Complete Best Practice Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
import numpy as np

# ─── PRODUCTION-READY PIPELINE ───

# 1. Load data
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42, stratify=data.target
)

# 2. Create pipeline
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', LogisticRegression(max_iter=10000, random_state=42))
])

# 3. Define search space
param_grid = {
    'clf__penalty': ['l1', 'l2', 'elasticnet'],
    'clf__C': [0.01, 0.1, 1.0, 10.0],
    'clf__solver': ['saga'],  # supports all penalty types
    'clf__l1_ratio': [0.25, 0.5, 0.75],  # only used for elasticnet
}

# 4. Grid search with cross-validation
grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', 
                    n_jobs=-1, refit=True)
grid.fit(X_train, y_train)

# 5. Report results
print("Best Model Configuration:")
print(f"  Penalty:   {grid.best_params_['clf__penalty']}")
print(f"  C:         {grid.best_params_['clf__C']}")
print(f"  CV Score:  {grid.best_score_:.4f}")
print(f"  Test Score: {grid.score(X_test, y_test):.4f}")

# 6. Final evaluation
print(f"\nClassification Report (Test Set):")
print(classification_report(y_test, grid.predict(X_test), 
                            target_names=data.target_names))

# 7. Feature analysis
best_model = grid.best_estimator_.named_steps['clf']
weights = best_model.coef_[0]
n_nonzero = np.sum(np.abs(weights) > 1e-6)
print(f"Active features: {n_nonzero}/{len(weights)}")
```

---

## 10. Summary Table

| Concept | Formula / Detail | Key Insight |
|---------|-----------------|-------------|
| **L2 (Ridge)** | R = Σ wⱼ² | Shrinks all weights; no zeros |
| **L1 (Lasso)** | R = Σ \|wⱼ\| | Drives some weights to zero |
| **ElasticNet** | R = ρ·L1 + (1-ρ)·L2 | Best of both worlds |
| **C parameter** | C = 1/λ | Large C = less regularization |
| **Default C** | C = 1.0 | Always tune with CV! |
| **L2 geometry** | Circle constraint | Smooth, no corners → no sparsity |
| **L1 geometry** | Diamond constraint | Sharp corners → sparsity |
| **Feature scaling** | Required! | Regularization is scale-sensitive |
| **Solver for L1** | solver='saga' or 'liblinear' | 'lbfgs' doesn't support L1 |
| **l1_ratio** | 0=pure L2, 1=pure L1 | Only for ElasticNet |
| **Bias** | Not regularized | Would shift all predictions |
| **CV for C** | GridSearchCV | Gold standard for tuning |

---

## 11. Revision Questions

### Q1: L1 vs L2 Fundamental Difference
**Q:** Explain geometrically why L1 regularization produces sparse solutions (some weights exactly zero) while L2 does not. Draw the constraint regions and show where the cost function contours typically intersect them.

**A:** L2's constraint region is a **circle** (w₁² + w₂² ≤ r²) — smooth everywhere, so the cost function's elliptical contours touch it at arbitrary points along the curve, rarely exactly on an axis. L1's constraint region is a **diamond** (|w₁| + |w₂| ≤ r) with sharp **corners** on the axes (where one weight = 0). The elliptical contours are much more likely to first touch the diamond at a corner, setting one or more weights exactly to zero. This is why L1 naturally performs feature selection.

---

### Q2: C Parameter
**Q:** In sklearn, what does C=0.01 mean vs C=100? If your model is overfitting (high train accuracy, low test accuracy), should you increase or decrease C?

**A:**
- C=0.01: **strong regularization** (λ=100), weights heavily penalized → simple model
- C=100: **weak regularization** (λ=0.01), weights barely penalized → complex model
- If overfitting: **decrease C** to apply stronger regularization, which constrains the model's complexity and improves generalization. You're telling the model "don't fit the training data so aggressively."

---

### Q3: Regularization Gradient
**Q:** Write the gradient update rule for weight wⱼ with L2 regularization. Explain the "weight decay" interpretation.

**A:** 
Update: wⱼ := wⱼ - α · [(1/m)Σ(ŷᵢ - yᵢ)xᵢⱼ + (λ/m)·wⱼ]

Rearranging: wⱼ := wⱼ · **(1 - αλ/m)** - α · (1/m)Σ(ŷᵢ - yᵢ)xᵢⱼ

The factor (1 - αλ/m) multiplies wⱼ at each step, slightly **shrinking** it toward zero. This is "weight decay" — even without any gradient from the data, the weight naturally decays over time. The data-driven gradient then adjusts the weight, but it's always fighting against this decay force.

---

### Q4: ElasticNet Scenario
**Q:** You have a dataset with 100 features where groups of 5 features are highly correlated with each other. Should you use L1, L2, or ElasticNet? Explain why.

**A:** Use **ElasticNet**. Pure L1 would arbitrarily select one feature from each correlated group and eliminate the rest, which is unstable (different runs might pick different features). Pure L2 would keep all features but wouldn't identify which groups matter. ElasticNet combines the benefits: the L2 component ensures correlated features are kept/shrunk together (group stability), while the L1 component still performs feature selection across groups. Set l1_ratio ≈ 0.5 as a starting point.

---

### Q5: Feature Scaling Necessity
**Q:** Why is feature scaling ESSENTIAL when using regularization? Give a concrete numerical example where unscaled features lead to unfair penalization.

**A:** Consider two features: x₁ (age, range 0-100) and x₂ (income, range 0-100,000). To achieve equal influence, the model needs w₁ ≈ 1000 · w₂. Regularization penalizes large weights equally, so it penalizes w₁'s naturally larger magnitude, effectively under-weighting age relative to income. Example: without scaling, L2 might learn w = [0.1, 0.001] (age suppressed), while with scaling (both features 0-1), it would learn w = [0.5, 0.5] (equal importance). **StandardScaler** puts all features on the same scale, ensuring the penalty is applied fairly.

---

### Q6: Practical Pipeline
**Q:** Design a complete sklearn pipeline for a binary classification task with 500 features, including feature scaling, regularization selection, and hyperparameter tuning. Specify the exact code structure and parameter grid you would use.

**A:**
```python
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', LogisticRegression(solver='saga', max_iter=10000, random_state=42))
])

# With 500 features, L1/ElasticNet for feature selection is important
param_grid = [
    {'clf__penalty': ['l1'], 'clf__C': np.logspace(-3, 2, 10)},
    {'clf__penalty': ['l2'], 'clf__C': np.logspace(-3, 2, 10)},
    {'clf__penalty': ['elasticnet'], 
     'clf__C': np.logspace(-3, 2, 10),
     'clf__l1_ratio': [0.1, 0.5, 0.9]},
]

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='f1', n_jobs=-1)
grid.fit(X_train, y_train)
```
Use F1 score (not accuracy) if classes are imbalanced. The grid searches ~40 configurations × 5 folds = 200 fits. After finding the best model, check `coef_` for feature selection insights.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 6: Multi-Class Classification](06-multi-class-classification.md) | **Chapter 7: Regularization** | [Unit 6 →](../../06-Model-Evaluation/) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. [Sigmoid Function](02-sigmoid-function.md)
3. [Decision Boundary](03-decision-boundary.md)
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. [Multi-Class Classification](06-multi-class-classification.md)
7. **📍 Regularization** ← You are here

---

> *"Regularization teaches the model a valuable lesson: simplicity is not a limitation — it's a feature."*

© 2025 AI/ML Study Notes. All rights reserved.
