# ⚖️ Unit 6, Chapter 3: Weighted K-Nearest Neighbors

> **K-Nearest Neighbors — Part 3 of 6**
>
> In standard KNN, every neighbor gets an equal vote regardless of how close or far it is. Weighted KNN fixes this — closer neighbors get a louder voice. This chapter covers uniform vs distance-weighted voting, kernel-based weighting, custom weight functions, and practical implementation with scikit-learn.

---

## Table of Contents

1. [The Problem with Uniform Voting](#1-the-problem-with-uniform-voting)
2. [Distance-Weighted KNN](#2-distance-weighted-knn)
3. [Inverse Distance Weighting](#3-inverse-distance-weighting)
4. [Gaussian Kernel Weighting](#4-gaussian-kernel-weighting)
5. [Other Weighting Schemes](#5-other-weighting-schemes)
6. [Effect on Decision Boundary](#6-effect-on-decision-boundary)
7. [When Weighted KNN Helps](#7-when-weighted-knn-helps)
8. [Comparison: Uniform vs Weighted](#8-comparison-uniform-vs-weighted)
9. [Implementation with scikit-learn](#9-implementation-with-scikit-learn)
10. [Custom Weight Functions](#10-custom-weight-functions)
11. [Complete Worked Example](#11-complete-worked-example)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. The Problem with Uniform Voting

### Standard KNN: Every Neighbor Votes Equally

```
  Query point Q = ?

  K = 5 nearest neighbors:

  ┌────────────┬────────┬───────┐
  │  Neighbor  │ Class  │ Dist  │
  ├────────────┼────────┼───────┤
  │  N₁        │   A    │  0.5  │    ← Very close!
  │  N₂        │   A    │  0.8  │    ← Close
  │  N₃        │   B    │  3.1  │    ← Far
  │  N₄        │   B    │  3.5  │    ← Far
  │  N₅        │   B    │  3.9  │    ← Very far
  └────────────┴────────┴───────┘

  Uniform voting:  A = 2 votes,  B = 3 votes  →  Predict B ✗

  But wait — the 2 A-neighbors are RIGHT NEXT to Q,
  while the 3 B-neighbors are far away!

  Intuitively, Q should be class A. Uniform voting gets it wrong.
```

### The Core Insight

```
  ┌──────────────────────────────────────────────────┐
  │  Not all neighbors are created equal.            │
  │                                                  │
  │  A neighbor at distance 0.5 should count MORE    │
  │  than a neighbor at distance 3.9.                │
  │                                                  │
  │  Solution: Weight votes by proximity.            │
  └──────────────────────────────────────────────────┘
```

---

## 2. Distance-Weighted KNN

### Concept

Instead of each neighbor contributing 1 vote, each contributes a **weighted vote** that depends on its distance to the query point.

```
  Standard (Uniform):     Vote_i = 1
  Distance-Weighted:      Vote_i = w(d_i)

  Where d_i = distance from query point to neighbor i
  and w(d) is a decreasing function (closer → higher weight).
```

### Classification with Weights

```
  For each class c:
    Score(c) = Σ w(d_i)   for all neighbors i with class = c

  Predicted class = argmax_c Score(c)
```

### Regression with Weights

```
                Σᵢ w(d_i) · yᵢ
  ŷ = ──────────────────────────
              Σᵢ w(d_i)

  (Weighted average of neighbor values)
```

---

## 3. Inverse Distance Weighting

### Formula

The most common weighting scheme:

```
             1
  w(d) = ─────────
           d^p

  Most common: p = 1  →  w(d) = 1/d
  Alternative: p = 2  →  w(d) = 1/d²   (stronger decay)
```

### Worked Example — Classification

```
  Using the same example from Section 1:

  K = 5, weights = 1/d

  ┌────────┬───────┬───────┬────────────┐
  │Neighbor│ Class │ Dist  │ Weight=1/d │
  ├────────┼───────┼───────┼────────────┤
  │  N₁    │   A   │  0.5  │  2.000     │
  │  N₂    │   A   │  0.8  │  1.250     │
  │  N₃    │   B   │  3.1  │  0.323     │
  │  N₄    │   B   │  3.5  │  0.286     │
  │  N₅    │   B   │  3.9  │  0.256     │
  └────────┴───────┴───────┴────────────┘

  Score(A) = 2.000 + 1.250           = 3.250
  Score(B) = 0.323 + 0.286 + 0.256   = 0.865

  Predict: A (score 3.250 > 0.865)  ✓  Correct!

  With uniform: B (3 > 2)  ✗  Wrong!
```

### Worked Example — Regression

```
  Query: predict house price for Q

  K = 4 nearest neighbors:

  ┌────────┬──────────┬───────┬────────────┐
  │Neighbor│  Price   │ Dist  │ Weight=1/d │
  ├────────┼──────────┼───────┼────────────┤
  │  N₁    │ $300,000 │  1.0  │  1.000     │
  │  N₂    │ $320,000 │  2.0  │  0.500     │
  │  N₃    │ $280,000 │  3.0  │  0.333     │
  │  N₄    │ $500,000 │  8.0  │  0.125     │
  └────────┴──────────┴───────┴────────────┘

  Uniform average:
    ŷ = (300K + 320K + 280K + 500K) / 4 = $350,000

  Weighted average:
    Numerator   = 1.0×300K + 0.5×320K + 0.333×280K + 0.125×500K
                = 300K + 160K + 93.3K + 62.5K = 615,800

    Denominator = 1.0 + 0.5 + 0.333 + 0.125 = 1.958

    ŷ = 615,800 / 1.958 = $314,607

  The weighted prediction ($314,607) is much less influenced by the
  distant outlier ($500,000 at distance 8) than the uniform ($350,000). ✓
```

### Edge Case: Distance = 0

```
  If a neighbor has distance 0 (i.e., identical to query point):
    w = 1/0 = undefined!

  Solutions:
    1. If any neighbor has d = 0, use ONLY those neighbors (with equal weight)
    2. Add a small epsilon: w = 1/(d + ε)
    3. scikit-learn handles this automatically
```

---

## 4. Gaussian Kernel Weighting

### Formula

```
                    -d²
  w(d) = exp( ──────────── )
                 2σ²

  Where σ (sigma) controls the width of the kernel:
    Small σ → weights decay rapidly (only very close neighbors matter)
    Large σ → weights decay slowly (approaches uniform weighting)
```

### Weight Decay Visualization

```
  Weight
    ↑
  1.0│●
    │ ●
    │  ●
  0.8│   ●
    │    ●
  0.6│     ●            σ = 0.5 (sharp decay)
    │      ●
  0.4│       ●
    │        ●
  0.2│         ● ●
    │             ● ● ●
  0.0│                    ● ● ● ● ● ● ● ●
    └──────────────────────────────────────→ Distance
       0    1    2    3    4    5    6    7

  Weight
    ↑
  1.0│● ●
    │    ● ●
    │        ●
  0.8│         ●
    │          ●
  0.6│           ●       σ = 2.0 (gradual decay)
    │            ●
  0.4│             ●
    │              ●
  0.2│               ● ●
    │                   ● ●
  0.0│                       ● ● ● ●
    └──────────────────────────────────────→ Distance
       0    1    2    3    4    5    6    7
```

### Comparison of Weighting Schemes

```
  Weight
    ↑
  1.0│ U U U U U U U U U         U = Uniform (constant)
    │
  0.8│ I
    │  I                          I = Inverse Distance (1/d)
  0.6│   I
    │    I
  0.4│     I                      G = Gaussian Kernel
    │ G    I
  0.2│  G    I I
    │   G G    I I I I
  0.0│      G G G I I I I I I
    └──────────────────────────→ Distance
       0  1  2  3  4  5  6  7
```

---

## 5. Other Weighting Schemes

### Rank-Based Weighting

```
  Instead of using distance, weight by rank:

  w_i = (K - rank_i + 1) / K

  K = 5:
  ┌──────┬──────┬────────┐
  │ Rank │ Calc │ Weight │
  ├──────┼──────┼────────┤
  │  1   │ 5/5  │  1.00  │   (nearest)
  │  2   │ 4/5  │  0.80  │
  │  3   │ 3/5  │  0.60  │
  │  4   │ 2/5  │  0.40  │
  │  5   │ 1/5  │  0.20  │   (farthest)
  └──────┴──────┴────────┘

  ✓ Simple and robust
  ✓ Not affected by distance scale
  ✗ Ignores actual distance magnitudes
```

### Triangular Weighting

```
             d_max - d
  w(d) = ─────────────────    for d ≤ d_max
               d_max

  w(d) = 0                    for d > d_max

  Where d_max is the distance to the K-th nearest neighbor.
  Linear decay from 1 (at d=0) to 0 (at d=d_max).
```

### Epanechnikov Kernel

```
                3
  w(d) = ──── (1 - d²/d_max²)     for d ≤ d_max
                4

  w(d) = 0                          for d > d_max

  Optimal kernel in the sense of minimizing mean squared error.
```

---

## 6. Effect on Decision Boundary

### Uniform KNN Decision Boundary

```
  All K neighbors vote equally → boundary depends on
  which class has more neighbors in the neighborhood.

  ┌─────────────────────────────┐
  │ ●         ○ ○               │
  │   ● ●  ┊      ○            │     ┊ = Decision boundary
  │     ●  ┊  ○                 │       (relatively sharp)
  │   ●   ┊    ○ ○              │
  │  ●   ┊       ○              │
  └─────────────────────────────┘
```

### Distance-Weighted KNN Decision Boundary

```
  Closer neighbors have stronger influence →
  boundary shifts toward the class with closer neighbors.

  ┌─────────────────────────────┐
  │ ●        ┊○ ○               │
  │   ● ●   ┊     ○            │     ┊ = Decision boundary
  │     ●  ┊   ○               │       (shifted toward ○,
  │   ●    ┊   ○ ○             │        smoother)
  │  ●    ┊      ○             │
  └─────────────────────────────┘

  The boundary is pulled TOWARD the class whose points
  are farther away (since nearby points have stronger weight,
  the boundary extends the influence of the closer class).
```

### Key Effects

```
  ┌────────────────────┬─────────────────────────────────────┐
  │  Aspect            │  Effect of Distance Weighting       │
  ├────────────────────┼─────────────────────────────────────┤
  │  Boundary shape    │  Smoother, more refined             │
  │  Outlier handling  │  Less sensitive (far outliers get    │
  │                    │  low weight)                        │
  │  K sensitivity     │  Less sensitive to exact K value    │
  │  Overlap regions   │  Better at resolving ambiguous      │
  │                    │  areas between classes              │
  └────────────────────┴─────────────────────────────────────┘
```

---

## 7. When Weighted KNN Helps

### ✅ Use Weighted KNN When:

```
  1. OVERLAPPING CLASSES
     When classes are interleaved, distance provides crucial
     information about which class a point truly belongs to.

     ● ○ ● ○ ● ○     (high overlap → weighting helps)

  2. K IS LARGE
     With large K, distant points dilute the prediction.
     Weighting ensures only nearby points dominate.

  3. UNEVEN DENSITY
     If one class has denser clusters near the query point,
     distance weighting amplifies this advantage.

  4. REGRESSION
     Almost always better than uniform for regression
     (avoids discontinuous jumps in predictions).

  5. BREAKING TIES
     Naturally resolves ties in classification
     (the closer neighbor wins).
```

### ❌ When It May Not Help:

```
  1. WELL-SEPARATED CLASSES
     If classes don't overlap, uniform voting works fine.
     All K neighbors are likely the same class anyway.

     ● ● ●    |    ○ ○ ○     (no overlap → weighting doesn't matter)

  2. K IS SMALL (e.g., K = 1)
     With K = 1, there's only one neighbor. Weighting is irrelevant.

  3. UNIFORMLY DISTRIBUTED DATA
     If data is evenly spread, distance weighting provides
     little additional information.
```

---

## 8. Comparison: Uniform vs Weighted

### Head-to-Head Comparison

```
  ┌──────────────────┬────────────────────┬────────────────────┐
  │  Aspect          │  Uniform KNN       │  Weighted KNN      │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Voting          │  1 neighbor = 1    │  1 neighbor =      │
  │                  │  vote              │  w(distance) vote  │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Outlier         │  Equal influence   │  Low influence     │
  │  impact          │  if within K       │  (far → low wt)    │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  K sensitivity   │  High — small      │  Lower — K is less │
  │                  │  change in K can   │  critical since far │
  │                  │  flip prediction   │  neighbors matter   │
  │                  │                    │  less anyway        │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Boundary        │  Piecewise linear  │  Smoother          │
  │                  │  (blocky)          │                    │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Computation     │  Count votes       │  Compute weights   │
  │                  │  (faster)          │  + weighted sum    │
  │                  │                    │  (slightly slower) │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Ties            │  Can occur         │  Very rare         │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Regression      │  Step-like         │  Smooth            │
  │  surface         │  predictions       │  predictions       │
  ├──────────────────┼────────────────────┼────────────────────┤
  │  Complexity      │  Simpler           │  One extra         │
  │                  │                    │  hyperparameter    │
  └──────────────────┴────────────────────┴────────────────────┘
```

---

## 9. Implementation with scikit-learn

### The `weights` Parameter

```python
from sklearn.neighbors import KNeighborsClassifier, KNeighborsRegressor

# ───────────── Uniform (default) ─────────────
knn_uniform = KNeighborsClassifier(
    n_neighbors=5,
    weights='uniform'       # Every neighbor votes equally
)

# ───────────── Distance-Weighted ─────────────
knn_distance = KNeighborsClassifier(
    n_neighbors=5,
    weights='distance'      # Weight = 1/distance
)

# ───────────── Same for Regression ─────────────
knn_reg_uniform = KNeighborsRegressor(n_neighbors=5, weights='uniform')
knn_reg_distance = KNeighborsRegressor(n_neighbors=5, weights='distance')
```

### Full Comparison Example

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.model_selection import cross_val_score, train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import KNeighborsClassifier

# Load and prepare data
X, y = load_iris(return_X_y=True)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.3, random_state=42
)

# Compare uniform vs distance weighting across K values
print(f"{'K':>3}  {'Uniform Acc':>12}  {'Distance Acc':>13}  {'Better':>8}")
print("-" * 50)

for k in range(1, 21, 2):
    knn_u = KNeighborsClassifier(n_neighbors=k, weights='uniform')
    knn_d = KNeighborsClassifier(n_neighbors=k, weights='distance')

    score_u = cross_val_score(knn_u, X_scaled, y, cv=10).mean()
    score_d = cross_val_score(knn_d, X_scaled, y, cv=10).mean()

    better = "Distance" if score_d > score_u else "Uniform" if score_u > score_d else "Tie"
    print(f"{k:3d}  {score_u:12.4f}  {score_d:13.4f}  {better:>8}")
```

**Expected Output:**
```
  K   Uniform Acc  Distance Acc    Better
--------------------------------------------------
  1       0.9600        0.9600       Tie
  3       0.9667        0.9667       Tie
  5       0.9667        0.9733  Distance
  7       0.9667        0.9733  Distance
  9       0.9667        0.9733  Distance
 11       0.9733        0.9800  Distance
 13       0.9733        0.9800  Distance
 15       0.9667        0.9733  Distance
 17       0.9667        0.9733  Distance
 19       0.9600        0.9733  Distance
```

---

## 10. Custom Weight Functions

### scikit-learn Custom Callable

You can pass a **function** to the `weights` parameter:

```python
import numpy as np
from sklearn.neighbors import KNeighborsClassifier

# ───────────── Custom Weight: Inverse Square ─────────────
def inverse_square_weights(distances):
    """Weight = 1/d². Stronger decay than 1/d."""
    # distances is a 2D array: (n_queries, n_neighbors)
    # Handle zero distances
    weights = np.where(distances == 0, 1.0, 1.0 / (distances ** 2))
    return weights

knn_custom = KNeighborsClassifier(
    n_neighbors=5,
    weights=inverse_square_weights
)


# ───────────── Custom Weight: Gaussian Kernel ─────────────
def gaussian_weights(distances, sigma=1.0):
    """Gaussian kernel weighting."""
    return np.exp(-(distances ** 2) / (2 * sigma ** 2))

# Since sklearn expects a single-argument callable, use a closure:
def make_gaussian_weights(sigma):
    def weights_func(distances):
        return np.exp(-(distances ** 2) / (2 * sigma ** 2))
    return weights_func

knn_gauss = KNeighborsClassifier(
    n_neighbors=7,
    weights=make_gaussian_weights(sigma=2.0)
)


# ───────────── Custom Weight: Rank-Based ─────────────
def rank_weights(distances):
    """Weight based on rank, not distance."""
    n_neighbors = distances.shape[1]
    # All queries get same rank-based weights
    weights = np.zeros_like(distances)
    for i in range(distances.shape[0]):
        ranks = np.argsort(np.argsort(distances[i])) + 1  # 1-based ranks
        weights[i] = (n_neighbors - ranks + 1) / n_neighbors
    return weights

knn_rank = KNeighborsClassifier(
    n_neighbors=5,
    weights=rank_weights
)


# ───────────── Compare All Weight Functions ─────────────
from sklearn.datasets import load_iris
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler

X, y = load_iris(return_X_y=True)
X_scaled = StandardScaler().fit_transform(X)

weight_functions = {
    'uniform': 'uniform',
    'distance (1/d)': 'distance',
    'inverse_square (1/d²)': inverse_square_weights,
    'gaussian (σ=1.0)': make_gaussian_weights(1.0),
    'gaussian (σ=2.0)': make_gaussian_weights(2.0),
    'rank-based': rank_weights,
}

print(f"{'Weight Function':>25}  {'CV Accuracy':>12}")
print("-" * 42)

for name, wf in weight_functions.items():
    knn = KNeighborsClassifier(n_neighbors=7, weights=wf)
    score = cross_val_score(knn, X_scaled, y, cv=10).mean()
    print(f"{name:>25}  {score:12.4f}")
```

---

## 11. Complete Worked Example

### From Scratch: Weighted KNN Classifier

```python
import numpy as np
from collections import Counter

class WeightedKNN:
    """KNN classifier with configurable weight function."""

    def __init__(self, k=5, weights='uniform'):
        """
        Parameters:
            k: Number of neighbors.
            weights: 'uniform', 'distance', or a callable.
        """
        self.k = k
        self.weights = weights

    def fit(self, X, y):
        self.X_train = np.array(X)
        self.y_train = np.array(y)
        return self

    def _compute_distances(self, x):
        """Euclidean distances from x to all training points."""
        return np.sqrt(np.sum((self.X_train - x) ** 2, axis=1))

    def _get_weights(self, distances):
        """Compute weights for K nearest neighbors."""
        if self.weights == 'uniform':
            return np.ones(self.k)
        elif self.weights == 'distance':
            # Handle zero distances
            w = np.where(distances == 0, 1e10, 1.0 / distances)
            return w
        elif callable(self.weights):
            return self.weights(distances)
        else:
            raise ValueError(f"Unknown weights: {self.weights}")

    def predict_one(self, x):
        """Predict class for a single query point."""
        distances = self._compute_distances(x)
        k_indices = np.argsort(distances)[:self.k]
        k_distances = distances[k_indices]
        k_labels = self.y_train[k_indices]

        weights = self._get_weights(k_distances)

        # Weighted vote
        class_scores = {}
        for label, weight in zip(k_labels, weights):
            class_scores[label] = class_scores.get(label, 0) + weight

        return max(class_scores, key=class_scores.get)

    def predict(self, X):
        X = np.array(X)
        return np.array([self.predict_one(x) for x in X])

    def score(self, X, y):
        predictions = self.predict(X)
        return np.mean(predictions == np.array(y))


# ───────────── Demo ─────────────
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

print("=== Custom Weighted KNN ===")
for w in ['uniform', 'distance']:
    model = WeightedKNN(k=5, weights=w)
    model.fit(X_train_s, y_train)
    acc = model.score(X_test_s, y_test)
    print(f"  weights={w:10s}  →  Test Accuracy = {acc:.4f}")

# Custom Gaussian weights
def gauss_w(d, sigma=1.0):
    return np.exp(-d**2 / (2*sigma**2))

model_gauss = WeightedKNN(k=7, weights=gauss_w)
model_gauss.fit(X_train_s, y_train)
print(f"  weights={'gaussian':10s}  →  Test Accuracy = {model_gauss.score(X_test_s, y_test):.4f}")
```

### Detailed Step-by-Step Walkthrough

```
  Query Point Q = (5.0, 3.5, 1.4, 0.2)  [scaled]

  Step 1: Compute distances to ALL training points
          d₁ = 0.72, d₂ = 1.15, d₃ = 0.98, ... d₁₀₅ = 4.31

  Step 2: Find K=5 nearest neighbors
          Sorted: d₃₇=0.31, d₁₂=0.45, d₈₉=0.62, d₅₆=0.88, d₂₃=1.21

  Step 3: Get their labels
          y₃₇=0, y₁₂=0, y₈₉=0, y₅₆=1, y₂₃=0

  Step 4a: Uniform voting
          Class 0: 4 votes,  Class 1: 1 vote  →  Predict 0

  Step 4b: Distance-weighted voting (1/d)
          Class 0: 1/0.31 + 1/0.45 + 1/0.62 + 1/1.21
                 = 3.23 + 2.22 + 1.61 + 0.83 = 7.89
          Class 1: 1/0.88
                 = 1.14

          Predict: Class 0 (7.89 >> 1.14)

  Both agree here, but with different confidence levels.
  Weighted KNN is MORE confident (7.89 vs 1.14 ≫ 4 vs 1).
```

---

## 12. Summary Table

| Weighting Scheme | Formula | Pros | Cons |
|-----------------|---------|------|------|
| **Uniform** | w = 1 | Simple, fast | All neighbors equal; sensitive to K |
| **Inverse Distance** | w = 1/d | Natural; closer = more influence | Singular at d=0; scale-dependent |
| **Inverse Square** | w = 1/d² | Stronger locality; fast decay | Very sensitive to nearest neighbor |
| **Gaussian Kernel** | w = exp(-d²/2σ²) | Smooth, tunable bandwidth (σ) | Extra hyperparameter (σ) |
| **Rank-Based** | w = (K-rank+1)/K | Scale-independent | Ignores actual distance values |
| **Epanechnikov** | w = ¾(1 - d²/d²ₘₐₓ) | Theoretically optimal kernel | More complex to implement |

### Quick Decision Guide

```
  ┌──────────────────────────────────────────────────┐
  │  Start with weights='distance' (1/d)             │
  │                                                  │
  │  If results are good → done!                     │
  │                                                  │
  │  If you need more control:                       │
  │    → Try Gaussian with different σ               │
  │    → Use cross-validation to select              │
  │                                                  │
  │  For most problems:                              │
  │    weights='distance' ≥ weights='uniform'        │
  └──────────────────────────────────────────────────┘
```

---

## 13. Revision Questions

**Q1.** Given K=5 neighbors with classes [A, A, B, B, B] at distances [0.2, 0.5, 2.0, 2.5, 3.0], compute the predicted class using (a) uniform voting and (b) inverse distance weighting (1/d). Which gives a different (and arguably better) answer?

**Q2.** Explain why distance-weighted KNN makes the choice of K less critical than uniform KNN. What happens to far-away neighbors in distance-weighted voting?

**Q3.** You are using KNN for regression with K=3. The three nearest neighbors have values [100, 200, 150] at distances [1, 1, 5]. Compute the prediction using (a) uniform averaging and (b) distance-weighted averaging. Which is more reasonable and why?

**Q4.** Design a custom weight function where neighbors within distance 1.0 get full weight (1.0), neighbors between distance 1.0 and 3.0 get linearly decreasing weight, and neighbors beyond 3.0 get zero weight. Write the mathematical formula.

**Q5.** In scikit-learn, how do you specify (a) uniform weights, (b) inverse distance weights, and (c) a custom Gaussian kernel weight function?

**Q6.** A dataset has two overlapping classes with very different local densities. Explain why weighted KNN would outperform uniform KNN in this scenario, using a concrete example.

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [6.2 Choosing K](./02-choosing-k.md) | **6.3 Weighted KNN** | [6.4 Curse of Dimensionality](./04-curse-of-dimensionality.md) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 3 of 6*
