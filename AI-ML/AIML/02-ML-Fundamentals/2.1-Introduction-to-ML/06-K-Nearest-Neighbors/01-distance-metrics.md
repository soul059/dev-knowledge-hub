# 📐 Unit 6, Chapter 1: Distance Metrics for KNN

> **K-Nearest Neighbors — Part 1 of 6**
>
> The heart of KNN lies in measuring "closeness." The choice of distance metric fundamentally shapes which neighbors are deemed nearest and, consequently, the predictions your model makes. This chapter provides an exhaustive treatment of every major distance metric, complete with formulas, geometric intuition, worked calculations, and Python implementations.

---

## Table of Contents

1. [Why Distance Metrics Matter](#1-why-distance-metrics-matter)
2. [Euclidean Distance (L2)](#2-euclidean-distance-l2)
3. [Manhattan Distance (L1)](#3-manhattan-distance-l1)
4. [Minkowski Distance (Generalized Lp)](#4-minkowski-distance-generalized-lp)
5. [Chebyshev Distance (L∞)](#5-chebyshev-distance-l∞)
6. [Hamming Distance](#6-hamming-distance)
7. [Cosine Similarity & Cosine Distance](#7-cosine-similarity--cosine-distance)
8. [Mahalanobis Distance](#8-mahalanobis-distance)
9. [Effect of Feature Scaling](#9-effect-of-feature-scaling)
10. [When to Use Each Metric](#10-when-to-use-each-metric)
11. [Python Implementations](#11-python-implementations)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)

---

## 1. Why Distance Metrics Matter

KNN classifies or predicts based on the K data points **closest** to a query point. But "closest" is defined entirely by the distance metric you choose.

```
Query Point: Q = (3, 4)

    Data Points:
    A = (1, 1)    B = (4, 5)    C = (6, 4)

    Using Euclidean:  d(Q,A)=3.61  d(Q,B)=1.41  d(Q,C)=3.00  → Nearest = B
    Using Manhattan:  d(Q,A)=5     d(Q,B)=2     d(Q,C)=3     → Nearest = B
    Using Chebyshev:  d(Q,A)=3     d(Q,B)=1     d(Q,C)=3     → Nearest = B

    But change the data and the rankings CAN differ across metrics!
```

**Key insight:** Different metrics define different shapes for "equidistant" neighborhoods.

```
    Euclidean (L2)         Manhattan (L1)         Chebyshev (L∞)
    neighborhood:          neighborhood:          neighborhood:

         ·····                  ·                    ·····
        ·······               ···                   ·······
       ·········             ·····                  ·······
        ·······               ···                   ·······
         ·····                  ·                    ·····

      (circle)             (diamond)               (square)
```

---

## 2. Euclidean Distance (L2)

### Formula

The **Euclidean distance** is the straight-line distance between two points in n-dimensional space.

```
For points  p = (p₁, p₂, ..., pₙ)  and  q = (q₁, q₂, ..., qₙ):

                    ┌─────────────────────────────────┐
  d(p, q) = √      │ Σᵢ₌₁ⁿ  (pᵢ - qᵢ)²              │
                    └─────────────────────────────────┘

  Expanded:
  d(p, q) = √[ (p₁-q₁)² + (p₂-q₂)² + ... + (pₙ-qₙ)² ]
```

### Geometric Intuition

In 2D, this is the Pythagorean theorem:

```
    q = (4, 5)
    |  ╱
    | ╱  d = √[(4-1)² + (5-1)²]
    |╱       = √[9 + 16]
    p = (1, 1)    = √25 = 5

       y
       5 ─ ─ ─ ─ q
       |        ╱ |
       |      ╱   |  Δy = 4
       |    ╱     |
       1  p ──────┘
       |    Δx = 3
       └──────────── x
          1       4
```

### Worked Example (3D)

```
  p = (2, 3, 1),  q = (5, 7, 4)

  d = √[ (2-5)² + (3-7)² + (1-4)² ]
    = √[ (-3)²  + (-4)²  + (-3)²  ]
    = √[ 9 + 16 + 9 ]
    = √34
    ≈ 5.831
```

### Properties

| Property | Holds? |
|----------|--------|
| Non-negativity: d(p,q) ≥ 0 | ✅ |
| Identity: d(p,q) = 0 iff p = q | ✅ |
| Symmetry: d(p,q) = d(q,p) | ✅ |
| Triangle inequality: d(p,r) ≤ d(p,q) + d(q,r) | ✅ |

**When to use:** Default choice. Works well when features are continuous, similarly scaled, and you want rotationally invariant distance.

---

## 3. Manhattan Distance (L1)

### Formula

Also called **taxicab distance** or **city block distance**. Measures distance along axis-aligned paths.

```
  d(p, q) = Σᵢ₌₁ⁿ  |pᵢ - qᵢ|

  Expanded:
  d(p, q) = |p₁-q₁| + |p₂-q₂| + ... + |pₙ-qₙ|
```

### Geometric Intuition

```
    Euclidean path (direct):          Manhattan path (grid):

    q ·                               q ·
      ╲                                 │
       ╲  d = 5                         │ 4 blocks
        ╲                               │
         · p                            └─────· p
                                        3 blocks

    Manhattan d = |3| + |4| = 7     (always ≥ Euclidean)
```

### Worked Example

```
  p = (1, 2, 3, 4),  q = (4, 0, 1, 7)

  d = |1-4| + |2-0| + |3-1| + |4-7|
    = |−3|  + |2|   + |2|   + |−3|
    = 3 + 2 + 2 + 3
    = 10
```

### When to Use

- High-dimensional data (less affected by curse of dimensionality than L2)
- Sparse data
- When features represent genuinely different quantities (e.g., different physical units)
- Grid-like structures (e.g., city navigation)

---

## 4. Minkowski Distance (Generalized Lp)

### Formula

The **Minkowski distance** is the generalization that unifies Euclidean and Manhattan distances.

```
                                    1/p
  d(p, q) = [ Σᵢ₌₁ⁿ |pᵢ - qᵢ|ᵖ ]

  Where p ≥ 1 is the order parameter.
```

### Special Cases

```
  ┌──────────┬────────────────────────────────────┐
  │  p value │  Distance Metric                   │
  ├──────────┼────────────────────────────────────┤
  │  p = 1   │  Manhattan Distance (L1)           │
  │  p = 2   │  Euclidean Distance (L2)           │
  │  p → ∞   │  Chebyshev Distance (L∞)           │
  └──────────┴────────────────────────────────────┘
```

### Visualizing the Unit Ball for Different p

```
  The set of all points at distance 1 from origin:

  p = 1 (Manhattan)       p = 2 (Euclidean)       p → ∞ (Chebyshev)

        ·                      ···                  ·········
       · ·                   ·     ·                ·       ·
      ·   ·                 ·       ·               ·       ·
       · ·                   ·     ·                ·       ·
        ·                      ···                  ·········

     (Diamond)              (Circle)               (Square)


  p = 0.5 (not a metric)   p = 3                   p = 10

      ·                        ···                  ·········
     ···                     ··   ··                ·       ·
      ·                     ·       ·               ·       ·
     ···                     ··   ··                ·       ·
      ·                        ···                  ·········

   (star/concave)         (rounded square)       (almost square)
```

### Worked Example (p = 3)

```
  a = (1, 3),  b = (4, 7)

  d = [ |1-4|³ + |3-7|³ ]^(1/3)
    = [ 3³ + 4³ ]^(1/3)
    = [ 27 + 64 ]^(1/3)
    = 91^(1/3)
    ≈ 4.498
```

---

## 5. Chebyshev Distance (L∞)

### Formula

Also called **chessboard distance**. It is the maximum absolute difference along any dimension.

```
  d(p, q) = maxᵢ |pᵢ - qᵢ|

  This is the limit of Minkowski distance as p → ∞.
```

### Geometric Intuition — The King's Move

```
  On a chessboard, a king can move one step in any direction
  (horizontal, vertical, diagonal). The Chebyshev distance
  is the number of king moves to reach from one square to another.

    8 ┌───┬───┬───┬───┬───┬───┬───┬───┐
    7 │   │   │   │   │   │   │   │   │
    6 │   │   │   │   │   │   │   │   │
    5 │   │   │   │ K₂│   │   │   │   │    K₁ = (2,2)
    4 │   │   │ ↗ │   │   │   │   │   │    K₂ = (5,5)
    3 │   │ ↗ │   │   │   │   │   │   │
    2 │ K₁│   │   │   │   │   │   │   │    Chebyshev d = max(|5-2|, |5-2|)
    1 │   │   │   │   │   │   │   │   │                = max(3, 3) = 3
      └───┴───┴───┴───┴───┴───┴───┴───┘
        1   2   3   4   5   6   7   8
```

### Worked Example

```
  p = (3, 5, 1, 8),  q = (7, 2, 4, 6)

  Differences: |3-7|=4, |5-2|=3, |1-4|=3, |8-6|=2

  d = max(4, 3, 3, 2) = 4
```

### When to Use

- Warehouse robotics (movements that can occur simultaneously along all axes)
- Game AI (chessboard-like grids)
- When you care about the **worst-case** deviation in any single feature

---

## 6. Hamming Distance

### Formula

Counts the number of positions at which the corresponding values are **different**. Used for categorical or binary data.

```
  d(p, q) = Σᵢ₌₁ⁿ  𝟙(pᵢ ≠ qᵢ)

  where 𝟙(·) is the indicator function (1 if true, 0 if false).
```

### Worked Example — Binary Strings

```
  p = [1, 0, 1, 1, 0, 1, 0]
  q = [1, 1, 0, 1, 0, 0, 0]
       =  ≠  ≠  =  =  ≠  =

  Positions that differ: 2, 3, 6  →  d = 3
```

### Worked Example — Categorical Data

```
  Patient A: [Male,   Young,  Flu,    Yes]
  Patient B: [Female, Young,  Cold,   Yes]
              ≠       =       ≠       =

  d = 2  (2 out of 4 features differ)
```

### When to Use

- Binary feature vectors (e.g., one-hot encoded data)
- Error detection/correction codes
- Comparing strings of equal length
- Categorical KNN

---

## 7. Cosine Similarity & Cosine Distance

### Cosine Similarity Formula

Measures the **angle** between two vectors, ignoring magnitude.

```
                        p · q              Σᵢ pᵢ · qᵢ
  cos(θ) = ──────────────────── = ─────────────────────────
             ‖p‖ · ‖q‖           √(Σᵢ pᵢ²) · √(Σᵢ qᵢ²)

  Range: [-1, 1]   (for non-negative vectors: [0, 1])

  Cosine Distance = 1 - cos(θ)
  Range: [0, 2]    (for non-negative vectors: [0, 1])
```

### Geometric Intuition

```
                    ↑ Feature 2
                    │
                    │    B = (1, 4)
                    │   ╱
                    │  ╱  θ = angle between A and B
                    │ ╱
                    │╱  A = (4, 2)
                    └──────────────→ Feature 1

  A and B may have very different magnitudes,
  but cosine similarity only cares about the angle θ.

  If θ = 0°   → cos(θ) = 1   (identical direction, most similar)
  If θ = 90°  → cos(θ) = 0   (perpendicular, unrelated)
  If θ = 180° → cos(θ) = -1  (opposite direction)
```

### Worked Example

```
  p = (3, 4),  q = (4, 3)

  Dot product:  p · q = 3×4 + 4×3 = 12 + 12 = 24

  Magnitudes:   ‖p‖ = √(9+16) = √25 = 5
                ‖q‖ = √(16+9) = √25 = 5

  cos(θ) = 24 / (5 × 5) = 24/25 = 0.96

  Cosine Distance = 1 - 0.96 = 0.04  (very similar!)
```

### Worked Example — Text Similarity

```
  Document 1: "the cat sat"     → TF vector: [1, 1, 1, 0]  (the, cat, sat, dog)
  Document 2: "the dog sat"     → TF vector: [1, 0, 1, 1]

  p · q = 1×1 + 1×0 + 1×1 + 0×1 = 2

  ‖p‖ = √(1+1+1+0) = √3
  ‖q‖ = √(1+0+1+1) = √3

  cos(θ) = 2 / (√3 × √3) = 2/3 ≈ 0.667

  Cosine Distance = 1 - 0.667 = 0.333
```

### When to Use

- **Text mining / NLP** (TF-IDF vectors, word embeddings)
- **Recommendation systems** (user-item preference vectors)
- When **magnitude** doesn't matter, only **direction/proportion**
- High-dimensional sparse data

---

## 8. Mahalanobis Distance

### Formula

The Mahalanobis distance accounts for **correlations** between features and **varying scales**, using the covariance matrix.

```
  d(p, q) = √[ (p - q)ᵀ  S⁻¹  (p - q) ]

  Where:
    S   = covariance matrix of the dataset
    S⁻¹ = inverse of the covariance matrix
    (p - q) is treated as a column vector
```

### Why It Matters

```
  Euclidean treats all features equally:

    Feature 2                    Feature 2
    ↑                            ↑
    │    · · ·                   │    ·
    │   · · · ·                  │   · · ·
    │  · · · · ·     vs.         │  · · · · · · · ·
    │   · · · ·                  │   · · ·
    │    · · ·                   │    ·
    └───────────→ Feature 1      └───────────→ Feature 1

    Spherical clusters:          Elongated/correlated clusters:
    Euclidean works fine.        Mahalanobis is needed!

  Mahalanobis "stretches" and "rotates" space so that the
  distribution becomes spherical, then measures Euclidean distance
  in that transformed space.
```

### Worked Example (2D)

```
  Given:
    p = (2, 5),  q = (6, 4)
    
    Covariance matrix S = │ 4   2 │
                          │ 2   3 │

  Step 1: Compute (p - q)
    Δ = (2-6, 5-4) = (-4, 1)

  Step 2: Compute S⁻¹
    det(S) = 4×3 - 2×2 = 12 - 4 = 8

    S⁻¹ = (1/8) │  3  -2 │  =  │ 0.375  -0.25 │
                 │ -2   4 │     │-0.25    0.50  │

  Step 3: Compute Δᵀ S⁻¹ Δ
    S⁻¹ Δ = │ 0.375×(-4) + (-0.25)×1 │ = │ -1.75 │
             │(-0.25)×(-4) + 0.50×1  │   │  1.50 │

    Δᵀ (S⁻¹ Δ) = (-4)×(-1.75) + (1)×(1.50) = 7.0 + 1.5 = 8.5

  Step 4: Take square root
    d = √8.5 ≈ 2.915

  Compare: Euclidean d = √(16+1) = √17 ≈ 4.123
  The Mahalanobis distance is smaller because the direction (-4,1) 
  aligns somewhat with the data's spread.
```

### When to Use

- Features are **correlated**
- Features have **different variances**
- Outlier/anomaly detection
- When you want a distance that is **scale-invariant** and **correlation-aware**

### Limitations

- Requires computing and inverting the covariance matrix (expensive for high dimensions)
- Covariance matrix must be non-singular (invertible)
- Sensitive to sample size: unreliable when n < number of features

---

## 9. Effect of Feature Scaling

### The Problem

```
  Consider two features:
    Age:    ranges from 0 to 100
    Salary: ranges from 20,000 to 200,000

  Point A: (25, 50000)
  Point B: (30, 52000)
  Point C: (65, 51000)

  Euclidean distances from A:
    d(A,B) = √[(30-25)² + (52000-50000)²]
           = √[25 + 4,000,000]
           = √4,000,025
           ≈ 2000.01

    d(A,C) = √[(65-25)² + (51000-50000)²]
           = √[1600 + 1,000,000]
           = √1,001,600
           ≈ 1000.80

  Result: C is "closer" to A than B!
  But B is clearly more similar (age 30 vs 65 matters!).

  The salary feature DOMINATES because of its large scale.
```

### Solution: Feature Normalization

**Min-Max Scaling (Normalization):**
```
              xᵢ - min(x)
  x'ᵢ = ─────────────────────
            max(x) - min(x)

  Scales features to [0, 1]
```

**Standardization (Z-score):**
```
            xᵢ - μ
  x'ᵢ = ──────────
              σ

  Transforms to mean=0, std=1
```

### After Scaling (Min-Max Example)

```
  Age normalized:    A'=(0.25), B'=(0.30), C'=(0.65)
  Salary normalized: A'=(0.17), B'=(0.18), C'=(0.17)

  d(A',B') = √[(0.30-0.25)² + (0.18-0.17)²] = √[0.0025 + 0.0001] ≈ 0.051
  d(A',C') = √[(0.65-0.25)² + (0.17-0.17)²] = √[0.16 + 0.0]       = 0.400

  Now B is correctly identified as closer to A! ✓
```

### Which Scaling to Use?

| Method | When to Use |
|--------|------------|
| Min-Max | When you need bounded range [0,1]; no strong outliers |
| Standardization | When features may have outliers; Gaussian-like distributions |
| Robust Scaling | When outliers are present (uses median and IQR) |

> ⚠️ **Rule:** Always scale features before applying KNN (unless all features are already on the same scale).

---

## 10. When to Use Each Metric

```
  ┌─────────────────────┬──────────────────────────────────────────────┐
  │  Metric             │  Best For                                    │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Euclidean (L2)     │  Default choice. Continuous, scaled data.    │
  │                     │  Low-to-moderate dimensions.                 │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Manhattan (L1)     │  High-dimensional data. Sparse features.     │
  │                     │  Features with different physical meanings.   │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Minkowski (Lp)     │  When you want to tune p as a                │
  │                     │  hyperparameter via cross-validation.        │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Chebyshev (L∞)     │  When the worst-case feature deviation       │
  │                     │  matters. Game AI, warehouse logistics.      │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Hamming            │  Binary / categorical features.              │
  │                     │  One-hot encoded data. Error detection.      │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Cosine             │  Text/NLP (TF-IDF, embeddings).              │
  │                     │  Recommendations. When magnitude ≠ important │
  ├─────────────────────┼──────────────────────────────────────────────┤
  │  Mahalanobis        │  Correlated features. Anomaly detection.     │
  │                     │  When covariance structure matters.          │
  └─────────────────────┴──────────────────────────────────────────────┘
```

---

## 11. Python Implementations

### From Scratch

```python
import numpy as np

def euclidean_distance(p, q):
    """L2 distance between two vectors."""
    p, q = np.array(p), np.array(q)
    return np.sqrt(np.sum((p - q) ** 2))

def manhattan_distance(p, q):
    """L1 distance between two vectors."""
    p, q = np.array(p), np.array(q)
    return np.sum(np.abs(p - q))

def minkowski_distance(p, q, order=2):
    """Generalized Lp distance."""
    p, q = np.array(p), np.array(q)
    return np.sum(np.abs(p - q) ** order) ** (1.0 / order)

def chebyshev_distance(p, q):
    """L-infinity distance (maximum absolute difference)."""
    p, q = np.array(p), np.array(q)
    return np.max(np.abs(p - q))

def hamming_distance(p, q):
    """Number of positions where values differ."""
    p, q = np.array(p), np.array(q)
    return np.sum(p != q)

def cosine_similarity(p, q):
    """Cosine of angle between two vectors."""
    p, q = np.array(p, dtype=float), np.array(q, dtype=float)
    dot_product = np.dot(p, q)
    magnitude = np.linalg.norm(p) * np.linalg.norm(q)
    if magnitude == 0:
        return 0.0
    return dot_product / magnitude

def cosine_distance(p, q):
    """1 - cosine similarity."""
    return 1.0 - cosine_similarity(p, q)

def mahalanobis_distance(p, q, cov_matrix):
    """Distance accounting for feature correlations."""
    p, q = np.array(p), np.array(q)
    delta = p - q
    cov_inv = np.linalg.inv(cov_matrix)
    return np.sqrt(delta @ cov_inv @ delta)


# ──────────────── DEMO ────────────────

p = [2, 3, 1]
q = [5, 7, 4]

print("=== Distance Metrics Demo ===")
print(f"Points: p = {p}, q = {q}")
print(f"  Euclidean  (L2):  {euclidean_distance(p, q):.4f}")
print(f"  Manhattan  (L1):  {manhattan_distance(p, q):.4f}")
print(f"  Minkowski (p=3):  {minkowski_distance(p, q, order=3):.4f}")
print(f"  Chebyshev  (L∞):  {chebyshev_distance(p, q):.4f}")

# Binary example
b1 = [1, 0, 1, 1, 0]
b2 = [1, 1, 0, 1, 0]
print(f"\nBinary vectors: {b1}, {b2}")
print(f"  Hamming:          {hamming_distance(b1, b2)}")

# Cosine example
v1 = [3, 4]
v2 = [4, 3]
print(f"\nVectors: {v1}, {v2}")
print(f"  Cosine Similarity: {cosine_similarity(v1, v2):.4f}")
print(f"  Cosine Distance:   {cosine_distance(v1, v2):.4f}")

# Mahalanobis example
p2 = [2, 5]
q2 = [6, 4]
S = np.array([[4, 2], [2, 3]])
print(f"\nPoints: p = {p2}, q = {q2}")
print(f"  Covariance Matrix: {S.tolist()}")
print(f"  Mahalanobis:       {mahalanobis_distance(p2, q2, S):.4f}")
print(f"  Euclidean:         {euclidean_distance(p2, q2):.4f}")
```

**Expected Output:**
```
=== Distance Metrics Demo ===
Points: p = [2, 3, 1], q = [5, 7, 4]
  Euclidean  (L2):  5.8310
  Manhattan  (L1):  10.0000
  Minkowski (p=3):  5.2536
  Chebyshev  (L∞):  4.0000

Binary vectors: [1, 0, 1, 1, 0], [1, 1, 0, 1, 0]
  Hamming:          2

Vectors: [3, 4], [4, 3]
  Cosine Similarity: 0.9600
  Cosine Distance:   0.0400

Points: p = [2, 5], q = [6, 4]
  Covariance Matrix: [[4, 2], [2, 3]]
  Mahalanobis:       2.9155
  Euclidean:         4.1231
```

### Using scikit-learn & SciPy

```python
from sklearn.neighbors import KNeighborsClassifier
from scipy.spatial.distance import (
    euclidean, cityblock, minkowski, chebyshev,
    hamming, cosine, mahalanobis
)
import numpy as np

p = np.array([2, 3, 1])
q = np.array([5, 7, 4])

# SciPy distance functions
print("SciPy distances:")
print(f"  Euclidean:   {euclidean(p, q):.4f}")
print(f"  Manhattan:   {cityblock(p, q):.4f}")         # cityblock = Manhattan
print(f"  Minkowski:   {minkowski(p, q, p=3):.4f}")
print(f"  Chebyshev:   {chebyshev(p, q):.4f}")

# Using different metrics in KNN
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

metrics = ['euclidean', 'manhattan', 'chebyshev', 'minkowski']

print("\n=== KNN with Different Metrics on Iris Dataset ===")
for metric in metrics:
    knn = KNeighborsClassifier(n_neighbors=5, metric=metric)
    knn.fit(X_train_scaled, y_train)
    accuracy = knn.score(X_test_scaled, y_test)
    print(f"  {metric:12s}: Accuracy = {accuracy:.4f}")
```

---

## 12. Summary Table

| Metric | Formula | Best For | Scale Sensitive? | Handles Categorical? |
|--------|---------|----------|:---:|:---:|
| Euclidean (L2) | √(Σ(pᵢ-qᵢ)²) | Default, continuous data | ✅ Yes | ❌ |
| Manhattan (L1) | Σ\|pᵢ-qᵢ\| | High dimensions, sparse | ✅ Yes | ❌ |
| Minkowski (Lp) | (Σ\|pᵢ-qᵢ\|ᵖ)^(1/p) | Tunable generalization | ✅ Yes | ❌ |
| Chebyshev (L∞) | max\|pᵢ-qᵢ\| | Worst-case deviation | ✅ Yes | ❌ |
| Hamming | Σ𝟙(pᵢ≠qᵢ) | Binary/categorical data | ❌ No | ✅ |
| Cosine | 1 - (p·q)/(‖p‖‖q‖) | Text, NLP, recommendations | ❌ No* | ❌ |
| Mahalanobis | √((p-q)ᵀS⁻¹(p-q)) | Correlated features | ❌ No** | ❌ |

\* Cosine ignores magnitude by design.
\** Mahalanobis incorporates variance through the covariance matrix.

---

## 13. Revision Questions

**Q1.** You have a dataset with two features: "Number of Rooms" (range: 1–10) and "House Price" (range: 100,000–1,000,000). You use Euclidean distance with KNN without scaling. What problem will occur, and how do you fix it?

**Q2.** Calculate the Euclidean and Manhattan distances between p = (1, 4, 2) and q = (3, 1, 5). Which metric gives a larger distance, and why?

**Q3.** For text classification using TF-IDF vectors, which distance metric would you choose — Euclidean or Cosine? Justify your answer with an example showing how document length can mislead Euclidean distance.

**Q4.** Two features in your dataset are highly correlated (e.g., height and arm span). You're using Euclidean distance for KNN. What issue might arise, and which metric would better handle this situation?

**Q5.** A dataset has 20 binary features (one-hot encoded categorical variables). Which distance metric is most natural for this data? Write the formula for the distance between vectors [1,0,1,0,1] and [1,1,0,0,1].

**Q6.** Explain with an example why the Minkowski distance with p = 1 is more robust to outlier dimensions than p = 2. Consider two points where one dimension has a very large difference and others have small differences.

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [Unit 5: Naive Bayes](../../05-Naive-Bayes/) | **6.1 Distance Metrics** | [6.2 Choosing K](./02-choosing-k.md) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 1 of 6*
