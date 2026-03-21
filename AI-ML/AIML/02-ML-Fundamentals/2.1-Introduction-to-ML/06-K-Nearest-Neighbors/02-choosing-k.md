# 🎯 Unit 6, Chapter 2: Choosing the Optimal K

> **K-Nearest Neighbors — Part 2 of 6**
>
> The single most important hyperparameter in KNN is **K** — the number of neighbors consulted for each prediction. Too small and your model memorizes noise; too large and it washes out local structure. This chapter provides deep intuition, practical guidelines, and rigorous methods for selecting the right K.

---

## Table of Contents

1. [What K Means in KNN](#1-what-k-means-in-knn)
2. [Effect of K on Model Behavior](#2-effect-of-k-on-model-behavior)
3. [Bias-Variance Tradeoff with K](#3-bias-variance-tradeoff-with-k)
4. [Methods for Choosing K](#4-methods-for-choosing-k)
5. [Elbow Method](#5-elbow-method)
6. [Cross-Validation for K Selection](#6-cross-validation-for-k-selection)
7. [The K = √n Rule of Thumb](#7-the-k--n-rule-of-thumb)
8. [Odd K for Binary Classification](#8-odd-k-for-binary-classification)
9. [Error vs K Curve](#9-error-vs-k-curve)
10. [Practical Guidelines](#10-practical-guidelines)
11. [Python Example: Accuracy vs K](#11-python-example-accuracy-vs-k)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. What K Means in KNN

K determines how many nearest data points "vote" on the prediction for a query point.

```
  K = 1: Only the single closest point decides the class.
  K = 5: The 5 closest points vote; majority wins.
  K = n: Every point in the dataset votes (predicts the majority class everywhere).

  Example Dataset (2D, Binary Classification):

    y ↑
    4 │        ● (3,4)
    3 │  ○ (1,3)         ● (5,3)
    2 │        ○ (3,2)               ● = Class 1
    1 │  ● (1,1)   ○ (4,1)           ○ = Class 0
    0 └──────────────────→ x

  Query point Q = (3, 3):

  K=1:  Nearest = ● (3,4)           → Predict Class 1  ✓
  K=3:  Nearest = ● (3,4), ○ (3,2), ○ (1,3)  → 2○ vs 1●  → Predict Class 0
  K=5:  Nearest = all 5 nearest     → depends on count
```

---

## 2. Effect of K on Model Behavior

### Small K (e.g., K = 1 or 3)

```
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  ● ● ○     Complex, jagged decision boundary   │
  │  ○ ●       that wraps around individual points  │
  │    ○ ○ ●                                        │
  │  ● ○       Sensitive to noise / outliers        │
  │                                                 │
  │  ──────── Decision Boundary ─────────           │
  │  (wiggly, complex, many small regions)          │
  └─────────────────────────────────────────────────┘

  Properties:
  ✓ Low bias (captures local patterns)
  ✗ High variance (unstable, overfits noise)
  ✗ Sensitive to outliers
```

### Large K (e.g., K = 50 or n)

```
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  ● ● ○     Smooth, simple decision boundary    │
  │  ○ ●       that may miss local patterns         │
  │    ○ ○ ●                                        │
  │  ● ○       Robust to noise but loses detail     │
  │                                                 │
  │  ════════ Decision Boundary ═══════════         │
  │  (smooth, simple, possibly a straight line)     │
  └─────────────────────────────────────────────────┘

  Properties:
  ✗ High bias (too smooth, underfits)
  ✓ Low variance (stable predictions)
  ✓ Robust to individual outliers
```

### Visual Comparison

```
  K = 1                     K = 5                     K = 50
  (Overfitting)             (Good balance)            (Underfitting)

  ┌──────────┐              ┌──────────┐              ┌──────────┐
  │●○ ● ○    │              │●● ● ○    │              │●● ● ○    │
  │  ╲╱╲     │              │  ╲       │              │          │
  │● ╱╲ ○●   │              │●  ╲ ○●   │              │●   ○●    │
  │ ╱○  ╲    │              │    ╲     │              │──────────│
  │╱ ○ ● ╲●  │              │ ○ ●╲ ●  │              │ ○ ● ●    │
  └──────────┘              └──────────┘              └──────────┘

  Training Acc ≈ 100%       Training Acc ≈ 95%        Training Acc ≈ 70%
  Test Acc ≈ 75%            Test Acc ≈ 92%            Test Acc ≈ 72%
```

---

## 3. Bias-Variance Tradeoff with K

```
  Error
    ↑
    │
    │ ╲                                  ╱
    │  ╲  Total Error                  ╱
    │   ╲   (Test Error)             ╱
    │    ╲                         ╱
    │     ╲                      ╱
    │      ╲       ·─────·     ╱
    │       ╲    ╱         ╲ ╱
    │        ╲ ╱             ╳
    │         ╳            ╱  ╲
    │       ╱  ╲         ╱     ╲
    │     ╱    Optimal K          ╲───── Bias²
    │   ╱              ╲
    │  Variance          ╲
    │                      ╲──────────────────
    └──────────────────────────────────────→ K
    1    5    10   15   20   25   30   ...

  ┌───────────────────────────────────────────────────┐
  │  Small K → Low Bias,  High Variance  (Overfitting) │
  │  Large K → High Bias, Low Variance  (Underfitting) │
  │  Optimal K → Minimum Total Error                   │
  └───────────────────────────────────────────────────┘
```

### Mathematical Perspective

For KNN regression, the expected test error at a point x can be decomposed:

```
  E[Error] = Bias²(x) + Variance(x) + σ²(irreducible noise)

  As K increases:
    Bias²(x) ≈ increases   (averaging over a larger neighborhood blurs signal)
    Variance(x) ≈ σ²/K     (decreases — averaging reduces variance)

  The optimal K minimizes the sum of Bias² + Variance.
```

---

## 4. Methods for Choosing K

| Method | Description | Complexity |
|--------|-------------|:----------:|
| Elbow Method | Plot error vs K, find the "elbow" | Low |
| Cross-Validation | Systematically evaluate K values | Medium |
| √n Rule of Thumb | Quick heuristic: K ≈ √n | Very Low |
| Grid Search with CV | Exhaustive search over K range | Medium |
| Leave-One-Out CV | CV with fold size = 1 | High |

---

## 5. Elbow Method

### Concept

Plot the test error (or validation error) against K. Look for the "elbow" — the point where error stops decreasing significantly.

```
  Test Error
    ↑
  0.35│ ●
  0.30│   ●
  0.25│     ●
  0.20│       ●
  0.15│         ●
  0.12│           ●──●──●        ← Error plateaus
  0.13│                    ●──●──●──●──●   ← Starts increasing
    │
    └──────────────────────────────────→ K
       1   3   5   7   9  11  13  15  17  19  21

  The "elbow" is around K = 9 where error flattens out.
  Choose K = 9 (or K = 11 if you want odd K for binary).
```

### Interpreting the Curve

```
  Region 1 (K < elbow):   Error drops rapidly → model is overfit, adding
                           neighbors helps by reducing variance.

  Region 2 (at elbow):    Sweet spot → best balance of bias and variance.

  Region 3 (K > elbow):   Error slowly rises → model is underfit, too many
                           neighbors blur the decision boundary.
```

---

## 6. Cross-Validation for K Selection

### K-Fold Cross-Validation

```
  For each candidate value of K (the KNN hyperparameter):

  ┌─────┬─────┬─────┬─────┬─────┐
  │Fold1│Fold2│Fold3│Fold4│Fold5│  ← 5-fold CV
  └─────┴─────┴─────┴─────┴─────┘

  Iteration 1: Train on [2,3,4,5], Test on [1] → Error₁
  Iteration 2: Train on [1,3,4,5], Test on [2] → Error₂
  Iteration 3: Train on [1,2,4,5], Test on [3] → Error₃
  Iteration 4: Train on [1,2,3,5], Test on [4] → Error₄
  Iteration 5: Train on [1,2,3,4], Test on [5] → Error₅

  CV Error for this K = mean(Error₁, Error₂, ..., Error₅)

  Repeat for K = 1, 3, 5, 7, 9, 11, ...
  Choose K with lowest CV Error.
```

### Why Cross-Validation > Simple Train/Test Split

```
  Train/Test Split:
  ├── Uses only one split → results may be lucky/unlucky
  ├── Wastes data (test set not used for training)
  └── High variance in error estimate

  Cross-Validation:
  ├── Uses ALL data for both training and testing
  ├── Averages over multiple splits → more reliable
  └── Lower variance in error estimate → better K selection
```

### Leave-One-Out Cross-Validation (LOOCV)

```
  Special case: K-Fold CV where each fold has exactly 1 sample.

  For n data points:
    Iteration 1: Train on (n-1) points, test on point 1
    Iteration 2: Train on (n-1) points, test on point 2
    ...
    Iteration n: Train on (n-1) points, test on point n

  CV Error = (1/n) × Σ errors

  ✓ Nearly unbiased (almost all data used for training)
  ✗ Computationally expensive for large n
  ✓ For KNN, recomputation is cheap (no model training needed)
```

---

## 7. The K = √n Rule of Thumb

### The Heuristic

```
  K ≈ √n   (where n = number of training samples)

  Examples:
  ┌──────────────┬──────────┬──────────────────┐
  │  n (samples) │   √n     │  Suggested K     │
  ├──────────────┼──────────┼──────────────────┤
  │     25       │   5.0    │    5             │
  │    100       │  10.0    │    9 or 11 (odd) │
  │    400       │  20.0    │   19 or 21       │
  │   1000       │  31.6    │   31             │
  │  10000       │ 100.0    │   99 or 101      │
  └──────────────┴──────────┴──────────────────┘
```

### Limitations

- Just a starting point, not a guarantee
- Doesn't account for:
  - Number of classes
  - Dimensionality
  - Noise level
  - Class imbalance
- **Always validate with cross-validation**

```
  Recommendation:
    1. Start with K = √n (rounded to nearest odd number)
    2. Search a range around it: [√n - 10, √n + 10]
    3. Use cross-validation to pick the best K from that range
```

---

## 8. Odd K for Binary Classification

### The Tie Problem

```
  With even K, you can get ties in binary classification:

  K = 4:  Neighbors are [●, ●, ○, ○]  →  2 vs 2  →  TIE! 😟

  What happens with a tie?
    Option 1: Random choice (unreliable)
    Option 2: Use distance to break tie (better)
    Option 3: Just use odd K to avoid ties entirely (simplest)
```

### Why Odd K Solves It

```
  K = 5:  Neighbors are [●, ●, ●, ○, ○]  →  3 vs 2  →  Class ●  ✓
  K = 5:  Neighbors are [●, ●, ○, ○, ○]  →  2 vs 3  →  Class ○  ✓

  With odd K in binary classification, there's ALWAYS a clear winner.
```

### Multi-Class: Ties Are Still Possible

```
  With 3 classes and K = 9:
  [A, A, A, B, B, B, C, C, C]  →  3 vs 3 vs 3  →  TIE!

  For multi-class: Use distance-weighted voting or other tie-breaking.
  (See Chapter 3: Weighted KNN)
```

---

## 9. Error vs K Curve

### Typical Shape

```
  Error Rate (%)
    ↑
  40│●
    │
  35│  ●
    │
  30│    ●
    │
  25│      ●
    │
  20│        ●
    │          ●
  15│            ●
    │              ●     ●     ●           ●     ●
  12│                ●       ●   ●       ●   ●
  10│                  ● ● ●       ● ● ●
    │                    ↑
    │                 Optimal K
    │
    └──────────────────────────────────────────────→ K
       1  3  5  7  9 11 13 15 17 19 21 23 25 27 29

  Key observations:
  ├── K=1: High error (overfitting to noise)
  ├── K=3-5: Error drops rapidly
  ├── K=11-15: Error reaches minimum (optimal zone)
  ├── K=15+: Error slowly creeps up (underfitting)
  └── K=n: Error = majority class error rate
```

### Training Error vs Test Error

```
  Error
    ↑
    │  ╱─────────────────────────── Training Error
    │╱                               (increases with K)
    │
    │╲
    │  ╲
    │    ╲───────────────────────── Test Error
    │         ↑                      (U-shaped)
    │      Optimal K
    │
    └──────────────────────────────→ K
       1     5     10    15    20

  At K=1: Training Error = 0 (perfect memorization)
          Test Error = High (overfitting)

  At K=optimal: Both errors are balanced.

  At K=n: Both errors = majority class error rate.
```

---

## 10. Practical Guidelines

### Step-by-Step K Selection

```
  ┌──────────────────────────────────────────────────────────────┐
  │  STEP 1: Start with K = √n (nearest odd number)             │
  │                                                              │
  │  STEP 2: Define a search range:                              │
  │          K_values = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21] │
  │          (or up to √n ± 10)                                  │
  │                                                              │
  │  STEP 3: Run 5-fold or 10-fold Cross-Validation              │
  │          for each K in the range                             │
  │                                                              │
  │  STEP 4: Plot CV Error vs K                                  │
  │                                                              │
  │  STEP 5: Choose K with lowest CV Error                       │
  │          (prefer odd K for binary classification)            │
  │                                                              │
  │  STEP 6: If multiple K values have similar error,            │
  │          choose the LARGER K (simpler model, more stable)    │
  └──────────────────────────────────────────────────────────────┘
```

### Rules of Thumb

| Scenario | Recommendation |
|----------|---------------|
| Small dataset (n < 100) | K = 3 to 7 |
| Medium dataset (100 < n < 10000) | K = √n, validated with CV |
| Large dataset (n > 10000) | K = √n or larger; speed matters |
| Noisy data | Larger K to smooth noise |
| Clean data with sharp boundaries | Smaller K to capture detail |
| Binary classification | Always use odd K |
| Multi-class | Use distance-weighted voting |
| Imbalanced classes | Smaller K (large K → always predict majority) |

---

## 11. Python Example: Accuracy vs K

### Complete Implementation

```python
import numpy as np
import matplotlib
matplotlib.use('Agg')  # Non-interactive backend
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

# ───────────── Load and Prepare Data ─────────────
X, y = load_iris(return_X_y=True)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.3, random_state=42
)

# ───────────── Method 1: Simple Train/Test Accuracy vs K ─────────────
k_values = range(1, 31)
train_accuracies = []
test_accuracies = []

for k in k_values:
    knn = KNeighborsClassifier(n_neighbors=k)
    knn.fit(X_train, y_train)
    train_accuracies.append(knn.score(X_train, y_train))
    test_accuracies.append(knn.score(X_test, y_test))

print("=== Method 1: Train/Test Split ===")
print(f"{'K':>3}  {'Train Acc':>10}  {'Test Acc':>10}")
print("-" * 28)
for k, tr, te in zip(k_values, train_accuracies, test_accuracies):
    marker = " ←" if te == max(test_accuracies) else ""
    print(f"{k:3d}  {tr:10.4f}  {te:10.4f}{marker}")

best_k_simple = k_values[np.argmax(test_accuracies)]
print(f"\nBest K (test accuracy): {best_k_simple}")


# ───────────── Method 2: Cross-Validation ─────────────
print("\n=== Method 2: 10-Fold Cross-Validation ===")
cv_scores_mean = []
cv_scores_std = []

for k in k_values:
    knn = KNeighborsClassifier(n_neighbors=k)
    scores = cross_val_score(knn, X_scaled, y, cv=10, scoring='accuracy')
    cv_scores_mean.append(scores.mean())
    cv_scores_std.append(scores.std())

print(f"{'K':>3}  {'CV Mean Acc':>12}  {'CV Std':>8}")
print("-" * 28)
for k, mean, std in zip(k_values, cv_scores_mean, cv_scores_std):
    marker = " ←" if mean == max(cv_scores_mean) else ""
    print(f"{k:3d}  {mean:12.4f}  {std:8.4f}{marker}")

best_k_cv = k_values[np.argmax(cv_scores_mean)]
print(f"\nBest K (CV accuracy): {best_k_cv}")
print(f"CV Accuracy at best K: {max(cv_scores_mean):.4f} ± {cv_scores_std[np.argmax(cv_scores_mean)]:.4f}")


# ───────────── Method 3: √n Rule of Thumb ─────────────
n_train = len(X_train)
k_sqrt = int(np.sqrt(n_train))
if k_sqrt % 2 == 0:
    k_sqrt += 1  # Make it odd

print(f"\n=== Method 3: √n Rule ===")
print(f"n_train = {n_train}")
print(f"√n = {np.sqrt(n_train):.2f}")
print(f"Suggested K = {k_sqrt}")


# ───────────── Plot Results ─────────────
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Plot 1: Train vs Test Accuracy
axes[0].plot(k_values, train_accuracies, 'b-o', label='Train Accuracy', markersize=4)
axes[0].plot(k_values, test_accuracies, 'r-s', label='Test Accuracy', markersize=4)
axes[0].axvline(x=best_k_simple, color='green', linestyle='--', label=f'Best K={best_k_simple}')
axes[0].set_xlabel('K (Number of Neighbors)')
axes[0].set_ylabel('Accuracy')
axes[0].set_title('Train vs Test Accuracy')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Plot 2: Cross-Validation Accuracy with Error Bars
axes[1].errorbar(list(k_values), cv_scores_mean, yerr=cv_scores_std,
                 fmt='-o', capsize=3, markersize=4, color='purple')
axes[1].axvline(x=best_k_cv, color='green', linestyle='--', label=f'Best K={best_k_cv}')
axes[1].set_xlabel('K (Number of Neighbors)')
axes[1].set_ylabel('Cross-Validation Accuracy')
axes[1].set_title('10-Fold CV Accuracy vs K')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('knn_k_selection.png', dpi=150, bbox_inches='tight')
print("\nPlot saved as 'knn_k_selection.png'")
```

### Expected Output (Abbreviated)

```
=== Method 1: Train/Test Split ===
  K   Train Acc    Test Acc
----------------------------
  1      1.0000      0.9556
  3      0.9619      0.9778
  5      0.9714      0.9778 ←
  7      0.9714      0.9778 ←
  ...

Best K (test accuracy): 5

=== Method 2: 10-Fold Cross-Validation ===
  K  CV Mean Acc    CV Std
----------------------------
  1       0.9600    0.0533
  3       0.9667    0.0447
  5       0.9667    0.0447 ←
  7       0.9667    0.0447 ←
  ...

Best K (CV accuracy): 5
```

### Using GridSearchCV (Production Approach)

```python
from sklearn.model_selection import GridSearchCV

param_grid = {'n_neighbors': list(range(1, 31, 2))}  # Odd values only

grid_search = GridSearchCV(
    KNeighborsClassifier(),
    param_grid,
    cv=10,
    scoring='accuracy',
    return_train_score=True
)

grid_search.fit(X_scaled, y)

print(f"Best K: {grid_search.best_params_['n_neighbors']}")
print(f"Best CV Accuracy: {grid_search.best_score_:.4f}")

# Access all results
results = grid_search.cv_results_
for k, mean, std in zip(results['param_n_neighbors'],
                         results['mean_test_score'],
                         results['std_test_score']):
    print(f"  K={k:3d}: {mean:.4f} ± {std:.4f}")
```

---

## 12. Summary Table

| Aspect | Detail |
|--------|--------|
| **Small K (1–3)** | Low bias, high variance. Overfits noise. Jagged boundary. |
| **Large K (>20)** | High bias, low variance. Underfits. Smooth boundary. |
| **K = 1** | Voronoi tessellation. Training accuracy = 100%. |
| **K = n** | Always predicts majority class. |
| **Elbow method** | Plot error vs K; pick the knee/elbow point. |
| **Cross-validation** | Most reliable. 5-fold or 10-fold recommended. |
| **√n rule** | Quick starting point: K ≈ √(number of training samples). |
| **Odd K** | Prevents ties in binary classification. |
| **Imbalanced data** | Use smaller K; large K predicts majority class. |
| **Noisy data** | Use larger K to smooth out noise. |

---

## 13. Revision Questions

**Q1.** Explain with a diagram how increasing K from 1 to n transforms the KNN decision boundary from complex to simple. At K = n, what does KNN always predict?

**Q2.** You have a dataset with 400 training samples and 2 classes. Using the √n rule of thumb, what K would you start with? Why should you round to an odd number?

**Q3.** A KNN model with K = 1 achieves 100% training accuracy but only 72% test accuracy. Another model with K = 25 achieves 85% training accuracy and 84% test accuracy. Explain this phenomenon using the bias-variance tradeoff.

**Q4.** You perform 5-fold cross-validation for K = {1, 3, 5, 7, 9, 11, 13}. The CV accuracies are: {0.82, 0.87, 0.91, 0.93, 0.93, 0.92, 0.91}. Which K would you choose and why?

**Q5.** For a highly imbalanced dataset (95% Class A, 5% Class B), explain why a very large K value is problematic. What would happen at K = n?

**Q6.** Describe the difference between the elbow method and GridSearchCV for selecting K. When would you prefer one over the other?

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [6.1 Distance Metrics](./01-distance-metrics.md) | **6.2 Choosing K** | [6.3 Weighted KNN](./03-weighted-knn.md) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 2 of 6*
