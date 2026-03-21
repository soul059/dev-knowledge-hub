# 🔧 Soft Margin SVM

> **Unit 9, Chapter 3 — Slack Variables, Hinge Loss & the C Parameter**

---

## 📋 Table of Contents

1. [When Data Isn't Perfectly Separable](#1-when-data-isnt-perfectly-separable)
2. [Slack Variables ξᵢ](#2-slack-variables-ξᵢ)
3. [Modified Optimization Problem](#3-modified-optimization-problem)
4. [The C Parameter — Regularization-Margin Trade-off](#4-the-c-parameter--regularization-margin-trade-off)
5. [Hinge Loss Interpretation](#5-hinge-loss-interpretation)
6. [C as Regularization](#6-c-as-regularization)
7. [Choosing C via Cross-Validation](#7-choosing-c-via-cross-validation)
8. [ASCII Visualization of Soft Margin](#8-ascii-visualization-of-soft-margin)
9. [Dual Problem for Soft Margin](#9-dual-problem-for-soft-margin)
10. [Python Implementation — sklearn SVC C Parameter](#10-python-implementation--sklearn-svc-c-parameter)
11. [Applications](#11-applications)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. When Data Isn't Perfectly Separable

### The Problem with Hard Margin

The hard-margin SVM requires **all** points to satisfy yᵢ(w·xᵢ+b) ≥ 1. This fails when:

```
Scenario 1: OVERLAPPING CLASSES       Scenario 2: OUTLIER
                                      
    + + + - + +                           + + +
    + + - + + +                         + + + + +
    - + + + + -                         + + + + +        - (outlier!)
    - - + - - -                         + + + + +
    - - - - - +                           + + +
    - - - - - -                         
                                        No hyperplane can separate
No clean separation exists!             with large margin if this
                                        outlier must be correct.
```

### Why Hard Margin Fails

| Issue | Problem | Impact |
|-------|---------|--------|
| **Class overlap** | No separating hyperplane exists | Optimization is *infeasible* |
| **Outliers** | Force narrow margin or wrong boundary | Overfitting |
| **Noise** | Mislabeled points distort boundary | Poor generalization |
| **Real-world data** | Rarely perfectly separable | Need practical solution |

### The Solution: Allow Some Mistakes

The **soft margin SVM** (Cortes & Vapnik, 1995) relaxes the constraints by allowing some points to:
- Be inside the margin (but on correct side)
- Be on the wrong side of the hyperplane (misclassified)

But we **penalize** these violations proportionally.

---

## 2. Slack Variables ξᵢ

### Introduction

We introduce a **slack variable** ξᵢ ≥ 0 for each training point to measure how much each constraint is violated:

```
Original constraint:  yᵢ(w · xᵢ + b) ≥ 1         (hard margin)
Modified constraint:  yᵢ(w · xᵢ + b) ≥ 1 - ξᵢ    (soft margin)
```

### What ξᵢ Represents

The value of ξᵢ tells us exactly where point xᵢ falls:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ξᵢ = 0:    Point is on or beyond margin (correctly placed) │
│             yᵢ(w·xᵢ+b) ≥ 1                                │
│                                                             │
│  0 < ξᵢ < 1: Point is inside margin but on correct side    │
│              0 < yᵢ(w·xᵢ+b) < 1                           │
│              "Margin violation" but still correct class     │
│                                                             │
│  ξᵢ = 1:    Point is exactly on the decision boundary      │
│             yᵢ(w·xᵢ+b) = 0                                │
│                                                             │
│  ξᵢ > 1:    Point is on the wrong side (MISCLASSIFIED)     │
│             yᵢ(w·xᵢ+b) < 0                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Geometric Visualization

```
      w·x+b axis
         ↑
    ≥ +1 │ ++++++  ξ=0 (correctly beyond margin)      ← No penalty
         │═══════════════════ Positive margin boundary (w·x+b = +1)
   +0.5  │    +    ξ=0.5 (inside margin, correct side) ← Small penalty
         │
     0   │─── + ── ξ=1.0 (on decision boundary)       ← Medium penalty
         │         Decision boundary (w·x+b = 0)
   -0.5  │    +    ξ=1.5 (wrong side, MISCLASSIFIED)   ← Large penalty
         │
    ≤-1  │═══════════════════ Negative margin boundary (w·x+b = -1)
         │ ------  ξ=0 (correctly beyond margin)       ← No penalty
```

### Relationship: ξᵢ = max(0, 1 - yᵢ(w·xᵢ+b))

```
ξᵢ = max(0, 1 - yᵢ(w · xᵢ + b))

This is exactly the HINGE LOSS!
```

---

## 3. Modified Optimization Problem

### Soft Margin Primal

```
              1                    N
minimize    ─── ‖w‖²  +  C · Σ  ξᵢ
  w, b, ξ    2                  i=1

subject to:
    yᵢ(w · xᵢ + b) ≥ 1 - ξᵢ    for all i = 1, ..., N
    ξᵢ ≥ 0                       for all i = 1, ..., N
```

### Breaking Down the Objective

```
              1                    N
Objective = ─── ‖w‖²    +    C · Σ  ξᵢ
             2                   i=1
            ─────────         ──────────
          Regularization     Empirical Loss
          (maximize margin)  (minimize violations)
          
          "Keep model         "Keep mistakes
           simple"             small"
```

### The Trade-off

```
                    1
    minimize   ─── ‖w‖²  +  C · Σξᵢ
                2
               ↑               ↑
               │               │
          Want small ‖w‖²   Want small Σξᵢ
          (wide margin)     (few violations)
               │               │
               └───── TENSION ──┘
               
    These two goals CONFLICT:
    - Wide margin → more points may violate
    - Few violations → may need narrow margin
    - C controls the trade-off!
```

### L1 vs L2 Soft Margin

| Variant | Objective | Properties |
|---------|-----------|-----------|
| **L1 (standard)** | ½‖w‖² + CΣξᵢ | Sparse SVs, standard hinge loss |
| **L2** | ½‖w‖² + CΣξᵢ² | Differentiable, penalizes large violations more |

The L1 version is far more common and is the default in most implementations.

---

## 4. The C Parameter — Regularization-Margin Trade-off

### C Controls the Penalty for Violations

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  C → ∞ (very large):  HARD MARGIN behavior                      │
│  ├── Almost no violations allowed                                │
│  ├── Very narrow margin possible                                 │
│  ├── Overfitting risk (sensitive to outliers)                    │
│  └── All ξᵢ → 0                                                 │
│                                                                  │
│  C → 0 (very small):  IGNORE the data                           │
│  ├── Many violations tolerated                                   │
│  ├── Very wide margin (possibly useless)                         │
│  ├── Underfitting risk (ignores class structure)                 │
│  └── Large ξᵢ values accepted                                   │
│                                                                  │
│  C moderate:  BALANCED trade-off                                 │
│  ├── Some violations allowed for better margin                   │
│  ├── Good generalization                                         │
│  └── Sweet spot found via cross-validation                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
HIGH C (C=1000) - Narrow margin, few errors:

    + + +   + +
    + + + + ⊕ + |  - - - -         ⊕ = misclassified
    + + + + + + |  - - - -         | = narrow margin
    + + + + + + |  - - - -
    + + +   + + |  - - - -
                |
    (Tries to classify EVERY point correctly)
    (Boundary bends around outliers)


MODERATE C (C=1) - Medium margin, some errors:

    + + +   + +
    + + + +   + |   |  - - - -     | | = wider margin
    + + + + + ⊕ |   |  - - - -     ⊕ inside margin
    + + + + + + |   |  - - - -
    + + +   + + |   |  - - - -
                |   |
    (Allows a few margin violations for wider margin)


LOW C (C=0.01) - Very wide margin, many errors:

    + + +   + +
    + + ⊕ +   + |         |  - - - -
    + + + + + ⊕ |         |  ⊕ - - -
    ⊕ + + + + + |         |  - - - -     Many ⊕ allowed
    + + +   + + |         |  - - - -
                |         |
    (Very wide margin, tolerates many violations)
```

### C Parameter Analogy

```
Think of C as a "strictness dial":

         Low C ──────────────────────────── High C
    Lenient teacher                    Strict teacher
    "Errors are okay"                 "No errors allowed!"
    Wide margin                       Narrow margin
    Simple model                      Complex model
    Underfitting ←                    → Overfitting
    High bias                         High variance
```

---

## 5. Hinge Loss Interpretation

### Hinge Loss Function

The soft margin SVM objective can be rewritten using the **hinge loss**:

```
              N                        1
minimize    Σ  max(0, 1 - yᵢf(xᵢ)) + ─── ‖w‖²
            i=1                        2C

where f(xᵢ) = w · xᵢ + b
```

This is equivalent to the standard form (with a rescaling by C).

### Hinge Loss Definition

```
L_hinge(yᵢ, f(xᵢ)) = max(0, 1 - yᵢ · f(xᵢ))
```

### Hinge Loss Plot

```
Loss
  ↑
  │
3 │╲
  │ ╲
2 │  ╲
  │   ╲     Hinge Loss
1 │    ╲    max(0, 1 - y·f(x))
  │     ╲
0 │──────╲───────────────────────→ y·f(x)
  │   -1   0   1   2   3   4
  │        │   │
  │        │   └── After this point: ZERO loss
  │        │       (correctly classified with margin ≥ 1)
  │        │
  │        └── Hinge point (y·f(x) = 1)
  │            This is where the "hinge" bends
  │
  │  For y·f(x) < 1: Loss = 1 - y·f(x) (linear penalty)
  │  For y·f(x) ≥ 1: Loss = 0 (no penalty)
```

### Comparison of Loss Functions

```
Loss
  ↑
4 │╲
  │ ╲ ·
  │  ╲  ·            · 0-1 Loss (step function)
3 │   ╲   ·
  │    ╲    ╲
  │     ╲     ╲       ╲ Hinge Loss (SVM)
2 │      ╲      ╲
  │       ╲       ╲     ~ Log Loss (Logistic Regression)
  │        ╲        ╲
1 │·········╲·········╲
  │          ╲          ╲
  │           ╲           ╲
0 │────────────╲────────────────→ y·f(x)
  │     -2  -1  0   1   2   3
  │
  │  0-1 Loss:   L = 1 if y·f(x) < 0, else 0 (non-convex, hard to optimize)
  │  Hinge Loss: L = max(0, 1-y·f(x))  (convex upper bound on 0-1)
  │  Log Loss:   L = log(1 + e^(-y·f(x))) (smooth, always positive)
```

### Key Properties of Hinge Loss

| Property | Value |
|----------|-------|
| Convex? | ✅ Yes (piecewise linear, convex) |
| Differentiable? | ❌ Not at y·f(x) = 1 (use sub-gradients) |
| Sparse? | ✅ Zero for correctly classified points with margin ≥ 1 |
| Upper bound on 0-1 loss? | ✅ Yes |
| Leads to sparse models? | ✅ Only support vectors contribute |

---

## 6. C as Regularization

### The Bias-Variance Perspective

```
SVM Objective: ½‖w‖² + CΣξᵢ

Rewrite as:     Σ max(0, 1 - yᵢf(xᵢ))  +  λ‖w‖²
                ───────────────────────     ────────
                    Empirical Loss         Regularization
                                            (λ = 1/(2C))

When C is large  → λ is small → Less regularization → More complex model
When C is small  → λ is large → More regularization → Simpler model
```

### C and Regularization Parameter λ

| C value | λ = 1/(2C) | Regularization | Model Complexity | Risk |
|---------|-----------|----------------|------------------|------|
| C = 0.001 | λ = 500 | Very strong | Very simple | Underfitting |
| C = 0.1 | λ = 5 | Strong | Simple | Slight underfit |
| C = 1 | λ = 0.5 | Moderate | Balanced | Good tradeoff |
| C = 10 | λ = 0.05 | Weak | Complex | Slight overfit |
| C = 1000 | λ = 0.0005 | Very weak | Very complex | Overfitting |

### Relationship to Other Regularized Models

```
SVM (Hinge + L2):     Σ max(0, 1 - yf(x)) + λ‖w‖²
Ridge Regression:     Σ (yᵢ - w·xᵢ)²      + λ‖w‖²
Logistic Regression:  Σ log(1 + e^(-yf(x)))+ λ‖w‖²
                      ──────────────────     ────────
                      Different losses       Same regularization!
```

All these methods share L2 regularization (‖w‖²) but differ in their loss functions. SVM's hinge loss is unique in creating sparsity (support vectors).

---

## 7. Choosing C via Cross-Validation

### Standard Approach: Grid Search + CV

```
Typical C values to search:    C ∈ {0.001, 0.01, 0.1, 1, 10, 100, 1000}
                                    (logarithmic scale)

For each C:
├── Split data into k folds
├── Train on k-1 folds, validate on 1 fold
├── Repeat k times
├── Average validation accuracy
└── Select C with best average accuracy
```

### The C Selection Process

```
Accuracy
  ↑
  │
95│                ●────●
  │            ●──/      \──●
90│        ●──/              \──●
  │    ●──/                      \──●
85│●──/                              \──●
  │                                      
80│                                      
  │
  └──┬──────┬──────┬──────┬──────┬──────┬──→ log(C)
   0.001  0.01   0.1    1     10    100
     │                   │              │
     │            Best C = 1            │
  Underfitting                     Overfitting
```

### Python: Grid Search for C

```python
from sklearn.svm import SVC
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.datasets import make_classification
from sklearn.preprocessing import StandardScaler
import numpy as np

# Generate data
X, y = make_classification(n_samples=500, n_features=20,
                           n_informative=10, random_state=42)

# IMPORTANT: Scale features before SVM
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Grid search over C values
param_grid = {'C': [0.001, 0.01, 0.1, 1, 10, 100, 1000]}

grid_search = GridSearchCV(
    SVC(kernel='linear'),
    param_grid,
    cv=5,
    scoring='accuracy',
    return_train_score=True,
    n_jobs=-1
)

grid_search.fit(X_scaled, y)

# Results
print(f"Best C: {grid_search.best_params_['C']}")
print(f"Best CV Accuracy: {grid_search.best_score_:.4f}")

print("\nDetailed Results:")
print(f"{'C':>8} | {'Train Acc':>10} | {'Val Acc':>10} | {'Std':>6}")
print("-" * 45)
for i, C in enumerate(param_grid['C']):
    train_acc = grid_search.cv_results_['mean_train_score'][i]
    val_acc = grid_search.cv_results_['mean_test_score'][i]
    std = grid_search.cv_results_['std_test_score'][i]
    marker = " ← Best" if C == grid_search.best_params_['C'] else ""
    print(f"{C:>8} | {train_acc:>10.4f} | {val_acc:>10.4f} | {std:>6.4f}{marker}")
```

---

## 8. ASCII Visualization of Soft Margin

### Complete Soft Margin Diagram

```
                    SOFT MARGIN SVM VISUALIZATION
                    
     Feature x₂ ↑
                 │
             4   │   +          +          POSITIVE CLASS
                 │      +    +     +
             3   │         +    ⊕(ξ=0.3)
                 │      +     +    +
             2   │   +    +     +
                 │                                    Legend:
    ─────────1───┼═══════════════════════════  w·x+b = +1    + = Positive (y=+1)
                 │                             (positive      - = Negative (y=-1)
                 │            ⊕(ξ=0.6)          margin       ⊕ = Margin violator
    ─────────0───┼─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  w·x+b = 0      ⊗ = Misclassified
                 │            ⊗(ξ=1.3)         (boundary)
                 │                                           ═══ Margin boundary
   ────────-1────┼═══════════════════════════  w·x+b = -1    ─ ─ Decision boundary
                 │                             (negative
            -2   │   -    -     -               margin
                 │      -     -    -            boundary)
            -3   │         -    ⊕(ξ=0.4)
                 │      -    -     -
            -4   │   -          -          NEGATIVE CLASS
                 │
                 └──────────────────────────────→ Feature x₁

    Point Analysis:
    ┌───────────────────────────────────────────────────────┐
    │ Most + and - points: ξ=0 (correctly beyond margin)   │
    │ ⊕ at (x₂≈3):  ξ=0.3 (inside margin, correct side)  │
    │ ⊕ at (x₂≈0.4): ξ=0.6 (inside margin, correct side) │
    │ ⊗ at (x₂≈-0.3): ξ=1.3 (WRONG side, misclassified!) │
    │ ⊕ at (x₂≈-3): ξ=0.4 (inside margin, correct side)  │
    └───────────────────────────────────────────────────────┘
```

### Effect of C on the Decision Boundary

```
HIGH C = 100                    MODERATE C = 1                 LOW C = 0.01
(Almost no slack)               (Balanced)                     (Lots of slack)

  + +  + +                      + +  + +                      + +  + +
  + ⊕ + + +|  - - -             + ⊕ + +  |    |  - - -       + ⊕ + +   |          |  - - -
  + + + + +|  - - -             + + + +   |    |  - - -       + + + +   |          |  - - -
  + + + + +|  ⊗ - -             + + + + + |    |  ⊗ - -       + + + ⊕ + |          |  ⊗ - -
  + + + + +|  - - -             + + + + + |    |  - - -       + + + + + |          |  - - -
           |                              |    |                        |          |
  Narrow margin                 Medium margin                 Very wide margin
  ⊗ causes boundary             ⊗ inside margin               ⊗ tolerated easily
  to bend                       (penalized)                    (barely penalized)
  
  # SVs: Few                   # SVs: Moderate                # SVs: Many
  Risk: OVERFIT                 Risk: BALANCED                 Risk: UNDERFIT
```

---

## 9. Dual Problem for Soft Margin

### Soft Margin Lagrangian

```
              1                 N                                       N
L(w,b,ξ,α,μ) = ─── ‖w‖² + C Σ ξᵢ - Σ αᵢ[yᵢ(w·xᵢ+b) - 1 + ξᵢ] - Σ μᵢξᵢ
              2               i=1   i                                 i=1

where αᵢ ≥ 0 (for margin constraints) and μᵢ ≥ 0 (for ξᵢ ≥ 0 constraints)
```

### Stationarity Conditions

```
∂L/∂w = 0:    w = Σ αᵢyᵢxᵢ              (same as hard margin!)

∂L/∂b = 0:    Σ αᵢyᵢ = 0                (same as hard margin!)

∂L/∂ξᵢ = 0:   C - αᵢ - μᵢ = 0           (NEW! → αᵢ + μᵢ = C)
```

Since μᵢ ≥ 0, the third condition gives: **αᵢ ≤ C**

### The Soft Margin Dual

```
                      N           1   N   N
maximize  W(α) = Σ αᵢ  -  ─── Σ   Σ  αᵢ αⱼ yᵢ yⱼ (xᵢ · xⱼ)
   α              i=1       2  i=1 j=1

subject to:
    0 ≤ αᵢ ≤ C         for all i = 1, ..., N     ← BOX CONSTRAINT!
    Σᵢ αᵢ yᵢ = 0
```

### Key Difference from Hard Margin

| Aspect | Hard Margin | Soft Margin |
|--------|-------------|-------------|
| Dual constraint on α | αᵢ ≥ 0 | **0 ≤ αᵢ ≤ C** |
| Upper bound | None | C (from μᵢ ≥ 0) |
| Objective | Same | Same |
| Equality constraint | Same | Same |

The **only difference** is the upper bound C on αᵢ! This is called a **box constraint**.

### KKT Conditions for Soft Margin

```
┌──────────────────────────────────────────────────────────────────┐
│                SOFT MARGIN KKT CONDITIONS                        │
│                                                                  │
│  Complementary slackness for margin:                             │
│     αᵢ [yᵢ(w·xᵢ+b) - 1 + ξᵢ] = 0                             │
│                                                                  │
│  Complementary slackness for slack:                              │
│     μᵢ ξᵢ = 0   where μᵢ = C - αᵢ                              │
│     → (C - αᵢ) ξᵢ = 0                                          │
│                                                                  │
│  This gives THREE cases:                                         │
│                                                                  │
│  Case 1: αᵢ = 0                                                 │
│     → ξᵢ = 0 and yᵢ(w·xᵢ+b) ≥ 1                               │
│     → Point is correctly classified beyond margin                │
│     → NOT a support vector                                       │
│                                                                  │
│  Case 2: 0 < αᵢ < C                                             │
│     → ξᵢ = 0 (from (C-αᵢ)ξᵢ = 0 with C-αᵢ > 0 → ξᵢ = 0)     │
│     → yᵢ(w·xᵢ+b) = 1 (from αᵢ[...] = 0 with αᵢ > 0)         │
│     → Point is EXACTLY on the margin boundary                    │
│     → "Free" support vector (used to compute b)                 │
│                                                                  │
│  Case 3: αᵢ = C                                                 │
│     → ξᵢ ≥ 0 (can be positive)                                  │
│     → yᵢ(w·xᵢ+b) ≤ 1 (inside or beyond margin, wrong side)    │
│     → "Bounded" support vector (margin violator)                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Visual Summary of Three Cases

```
                                Decision Boundary (w·x+b=0)
                                        │
  CASE 1         CASE 2        │       CASE 2        CASE 1
  αᵢ = 0        0<αᵢ<C        │       0<αᵢ<C        αᵢ = 0
  ξᵢ = 0        ξᵢ = 0    CASE 3      ξᵢ = 0        ξᵢ = 0
                            αᵢ=C
  + + +           ◆    ⊗    │    ◆           - - -
  + + +           ◆    ξ>0  │    ◆           - - -
  + + +                     │                - - -

   Beyond         On        Inside/    On         Beyond
   margin       margin      Wrong    margin       margin
               boundary     side    boundary
```

---

## 10. Python Implementation — sklearn SVC C Parameter

### Exploring the Effect of C

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler

# Generate slightly overlapping data
np.random.seed(42)
X, y = make_classification(n_samples=300, n_features=2, n_redundant=0,
                           n_informative=2, n_clusters_per_class=1,
                           flip_y=0.1, class_sep=1.0, random_state=42)

# Scale features (IMPORTANT for SVM)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Train SVMs with different C values
C_values = [0.001, 0.01, 0.1, 1, 10, 100, 1000]

print(f"{'C':>8} | {'# SVs':>6} | {'% SVs':>6} | {'Train Acc':>10} | {'CV Acc':>8} | {'Margin':>8}")
print("-" * 65)

for C in C_values:
    svm = SVC(kernel='linear', C=C)
    svm.fit(X_scaled, y)
    
    train_acc = svm.score(X_scaled, y)
    cv_acc = cross_val_score(svm, X_scaled, y, cv=5).mean()
    n_svs = sum(svm.n_support_)
    pct_svs = 100 * n_svs / len(y)
    margin = 2 / np.linalg.norm(svm.coef_[0])
    
    print(f"{C:>8} | {n_svs:>6} | {pct_svs:>5.1f}% | {train_acc:>10.4f} | {cv_acc:>8.4f} | {margin:>8.4f}")
```

### Visualizing C Effect

```python
def plot_c_comparison(X, y, C_values=[0.01, 1, 100]):
    """Compare SVM decision boundaries for different C values."""
    fig, axes = plt.subplots(1, len(C_values), figsize=(6*len(C_values), 5))
    
    for ax, C in zip(axes, C_values):
        svm = SVC(kernel='linear', C=C)
        svm.fit(X, y)
        
        # Decision boundary mesh
        h = 0.02
        x_min, x_max = X[:, 0].min()-1, X[:, 0].max()+1
        y_min, y_max = X[:, 1].min()-1, X[:, 1].max()+1
        xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                             np.arange(y_min, y_max, h))
        
        Z = svm.decision_function(np.c_[xx.ravel(), yy.ravel()])
        Z = Z.reshape(xx.shape)
        
        # Plot
        ax.contourf(xx, yy, Z, levels=[-100, -1, 0, 1, 100],
                    colors=['#FFCCCC', '#FFE8E8', '#E8E8FF', '#CCCCFF'], alpha=0.4)
        ax.contour(xx, yy, Z, levels=[-1, 0, 1],
                   colors=['red', 'black', 'blue'],
                   linestyles=['--', '-', '--'], linewidths=[1, 2, 1])
        
        ax.scatter(X[y==0, 0], X[y==0, 1], c='red', marker='o', s=20, alpha=0.7)
        ax.scatter(X[y==1, 0], X[y==1, 1], c='blue', marker='s', s=20, alpha=0.7)
        
        # Support vectors
        ax.scatter(svm.support_vectors_[:, 0], svm.support_vectors_[:, 1],
                   s=100, facecolors='none', edgecolors='green', linewidths=1.5)
        
        margin = 2 / np.linalg.norm(svm.coef_[0])
        ax.set_title(f'C = {C}\n'
                     f'SVs: {sum(svm.n_support_)}, '
                     f'Margin: {margin:.3f}', fontsize=12)
        ax.set_xlabel('x₁')
        ax.set_ylabel('x₂')
    
    plt.suptitle('Effect of C on Soft Margin SVM', fontsize=14, y=1.02)
    plt.tight_layout()
    plt.savefig('c_parameter_comparison.png', dpi=150, bbox_inches='tight')
    plt.show()

plot_c_comparison(X_scaled, y)
```

### Complete Pipeline with C Selection

```python
from sklearn.svm import SVC
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.datasets import load_breast_cancer
from sklearn.metrics import classification_report

# Load real dataset
data = load_breast_cancer()
X, y = data.data, data.target

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Pipeline: Scale + SVM
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='linear'))
])

# Grid search for C
param_grid = {'svm__C': np.logspace(-3, 3, 7)}

grid_search = GridSearchCV(
    pipeline, param_grid, cv=5,
    scoring='accuracy', n_jobs=-1,
    return_train_score=True
)
grid_search.fit(X_train, y_train)

# Results
print(f"Best C: {grid_search.best_params_['svm__C']:.4f}")
print(f"Best CV Accuracy: {grid_search.best_score_:.4f}")
print(f"\nTest Accuracy: {grid_search.score(X_test, y_test):.4f}")
print(f"\nClassification Report:\n{classification_report(y_test, grid_search.predict(X_test), target_names=data.target_names)}")

# Number of support vectors
best_svm = grid_search.best_estimator_.named_steps['svm']
print(f"\nSupport vectors per class: {best_svm.n_support_}")
print(f"Total support vectors: {sum(best_svm.n_support_)} / {len(X_train)}")
```

---

## 11. Applications

### When to Use Soft Margin SVM

| Scenario | Why Soft Margin? |
|----------|-----------------|
| **Real-world data** | Classes almost always overlap to some degree |
| **Noisy labels** | Some training labels may be wrong → need tolerance |
| **Outliers present** | Hard margin would overfit to outliers |
| **Medical diagnosis** | Patient data has inherent variability |
| **Text classification** | Topic overlap between categories |
| **Image classification** | Visual similarity between classes |

### C Selection Guidelines

```
Starting point: C = 1.0 (sklearn default)

If model overfits (high train acc, low test acc):
└── Decrease C: try C = 0.1, 0.01, 0.001

If model underfits (low train acc):
└── Increase C: try C = 10, 100, 1000

For final model:
└── Use GridSearchCV with C ∈ [10⁻³, 10³] on log scale
```

---

## 12. Summary Table

| Concept | Key Point |
|---------|-----------|
| **Soft margin motivation** | Real data isn't perfectly separable |
| **Slack variable ξᵢ** | Measures constraint violation for point i |
| **ξᵢ = 0** | Point correctly beyond margin |
| **0 < ξᵢ < 1** | Point inside margin, correct side |
| **ξᵢ > 1** | Point misclassified |
| **Objective** | min ½‖w‖² + CΣξᵢ |
| **C large** | Less tolerance → narrow margin → overfit risk |
| **C small** | More tolerance → wide margin → underfit risk |
| **C = ∞** | Equivalent to hard margin |
| **Hinge loss** | max(0, 1 - y·f(x)) — zero for correct+margined points |
| **C ↔ λ** | C = 1/(2λ), inverse relationship to regularization strength |
| **Dual difference** | Only change: 0 ≤ αᵢ ≤ C (box constraint) |
| **Free SVs** | 0 < αᵢ < C, exactly on margin, ξᵢ = 0 |
| **Bounded SVs** | αᵢ = C, inside/beyond margin, ξᵢ > 0 |
| **Choosing C** | Grid search + cross-validation on log scale |

---

## 13. Revision Questions

### Q1: Slack Variable Interpretation
**A soft-margin SVM is trained and point xᵢ has ξᵢ = 2.5. What does this tell you about where the point lies relative to the decision boundary and margin?**

<details>
<summary>Answer</summary>

Since ξᵢ = 2.5 > 1, the point is **misclassified**. Specifically:
- yᵢ(w·xᵢ + b) = 1 - ξᵢ = 1 - 2.5 = -1.5
- The point is on the **wrong side** of the decision boundary (since yᵢ·f(xᵢ) < 0)
- It is 1.5 units of functional margin past the decision boundary on the wrong side
- Its Lagrange multiplier is αᵢ = C (bounded support vector)

</details>

### Q2: C Parameter Trade-off
**Explain the effect of increasing C from 0.01 to 1000 on: (a) margin width, (b) number of support vectors, (c) training accuracy, (d) test accuracy.**

<details>
<summary>Answer</summary>

**(a) Margin width**: **Decreases**. Higher C penalizes violations more, so the optimizer prefers a narrower margin with fewer violations over a wider margin with more violations.

**(b) Number of support vectors**: **Decreases** (generally). With high C, fewer points are allowed to violate the margin, so fewer points become support vectors. Only the points truly near the boundary remain as SVs.

**(c) Training accuracy**: **Increases** (or stays same). Higher C forces the model to classify more training points correctly, potentially at the cost of a complex boundary.

**(d) Test accuracy**: **Increases then decreases** (inverted U-shape). Initially, increasing C improves generalization by fitting the data better. But very high C causes overfitting to training noise, decreasing test accuracy.

</details>

### Q3: Hinge Loss
**Why does the hinge loss produce sparse models (few support vectors) while log loss (logistic regression) does not?**

<details>
<summary>Answer</summary>

The hinge loss max(0, 1 - y·f(x)) is **exactly zero** when y·f(x) ≥ 1. This means correctly classified points with sufficient margin contribute **zero loss and zero gradient**. The optimizer has no incentive to adjust the boundary based on these points, so their Lagrange multipliers αᵢ = 0. Only points with y·f(x) < 1 (support vectors) have non-zero αᵢ and influence the model.

In contrast, the log loss log(1 + e^{-y·f(x)}) is **never exactly zero** for any finite y·f(x). Every training point always contributes some (possibly tiny) gradient, so no point is ever completely ignored. This means all points influence the model parameters, preventing sparsity.

</details>

### Q4: Dual Formulation
**What is the only difference between the hard-margin and soft-margin dual problems? Why does this single change encode all the soft-margin behavior?**

<details>
<summary>Answer</summary>

The **only difference** is the upper bound on αᵢ:
- Hard margin: αᵢ ≥ 0 (no upper bound)
- Soft margin: **0 ≤ αᵢ ≤ C** (box constraint)

This single change encodes all soft-margin behavior because:
- αᵢ = C means the point is a "bounded" support vector that may violate the margin (ξᵢ > 0)
- Without the upper bound, the optimizer could increase αᵢ arbitrarily to penalize misclassified points, effectively requiring zero violations (hard margin)
- The cap at C limits how much the optimizer can "care" about any single point, naturally allowing some violations
- When C → ∞, the box constraint becomes αᵢ ≥ 0, recovering the hard-margin dual

</details>

### Q5: Practical Application
**You're building an SVM classifier for medical diagnosis where false negatives are very costly. You have noisy data with some mislabeled samples. How would you choose C, and what other sklearn parameters might help?**

<details>
<summary>Answer</summary>

**Choosing C**: Use a **moderate to slightly high C** (e.g., C=1 to C=10). Too high would overfit to noisy labels; too low would create too many false negatives by making the model too simple. Use cross-validation with a metric focused on recall (sensitivity) rather than accuracy.

**Other sklearn parameters**:
1. `class_weight='balanced'` or custom weights: Give higher weight to the positive class to reduce false negatives
2. `class_weight={0: 1, 1: 5}`: Make the cost of misclassifying positive cases 5x higher
3. Use `scoring='recall'` in GridSearchCV to optimize for minimizing false negatives
4. Consider `probability=True` with adjusted decision threshold instead of default 0.5

```python
svm = SVC(kernel='linear', C=best_C, class_weight={0: 1, 1: 10})
```

</details>

### Q6: Numerical Exercise
**For a soft-margin SVM with C=10, point xᵢ has αᵢ = 10 (= C). What can you conclude about ξᵢ and the classification of this point?**

<details>
<summary>Answer</summary>

Since αᵢ = C = 10 (bounded support vector):
1. **ξᵢ ≥ 0**: The slack variable can be positive (the KKT condition (C-αᵢ)ξᵢ = 0 is satisfied trivially since C-αᵢ = 0)
2. From the margin constraint: yᵢ(w·xᵢ+b) = 1 - ξᵢ
3. **If ξᵢ = 0**: Point is exactly on the margin boundary (yᵢ(w·xᵢ+b) = 1)
4. **If 0 < ξᵢ < 1**: Point is inside the margin but correctly classified
5. **If ξᵢ ≥ 1**: Point is **misclassified**

We know it's a bounded support vector, but we can't determine ξᵢ's exact value without more information. The point is either on the margin, inside the margin, or misclassified.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Maximum Margin Classifier](./02-maximum-margin-classifier.md) | [Unit 9: SVM](../) | [Kernel Trick →](./04-kernel-trick.md) |

---

*📝 Part of the AIML Study Notes — Unit 9: Support Vector Machines*
*🔖 Chapter 3 of 7 — Soft Margin SVM*
