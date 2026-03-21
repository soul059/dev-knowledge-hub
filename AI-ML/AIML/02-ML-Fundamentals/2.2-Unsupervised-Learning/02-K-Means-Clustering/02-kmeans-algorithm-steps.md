# 2.2 K-Means Algorithm — Step-by-Step

> **Chapter Overview:** This chapter walks through the K-Means algorithm in complete detail — from initialization to convergence. We work through a full numerical example by hand, analyze time complexity, and implement K-Means both from scratch in pure Python/NumPy and using scikit-learn.

---

## Table of Contents

1. [The Algorithm — Four Steps](#1-the-algorithm--four-steps)
2. [Pseudocode](#2-pseudocode)
3. [Step-by-Step Numerical Example](#3-step-by-step-numerical-example)
4. [Convergence Criteria](#4-convergence-criteria)
5. [Time and Space Complexity](#5-time-and-space-complexity)
6. [K-Means from Scratch (Python)](#6-k-means-from-scratch-python)
7. [K-Means with scikit-learn](#7-k-means-with-scikit-learn)
8. [Worked Example — Iris Dataset](#8-worked-example--iris-dataset)
9. [Summary Table](#9-summary-table)
10. [Key Takeaways](#10-key-takeaways)
11. [Revision Questions](#11-revision-questions)

---

## 1. The Algorithm — Four Steps

```
    ┌───────────────────────────────────────────────────────────┐
    │                   K-MEANS ALGORITHM                       │
    │                                                           │
    │   ┌──────────────┐                                        │
    │   │ 1. INITIALIZE│  Choose K initial centroids             │
    │   └──────┬───────┘  (random, K-Means++, etc.)             │
    │          │                                                │
    │          ▼                                                │
    │   ┌──────────────┐                                        │
    │   │ 2. ASSIGN    │  Assign each point to nearest centroid  │
    │   │              │  cᵢ = argmin_k ‖xᵢ - μₖ‖²             │
    │   └──────┬───────┘                                        │
    │          │                                                │
    │          ▼                                                │
    │   ┌──────────────┐                                        │
    │   │ 3. UPDATE    │  Recompute centroids                    │
    │   │              │  μₖ = (1/|Cₖ|) Σ xᵢ                   │
    │   └──────┬───────┘       xᵢ∈Cₖ                           │
    │          │                                                │
    │          ▼                                                │
    │   ┌──────────────┐                                        │
    │   │ 4. CONVERGE? │  If assignments changed → go to step 2 │
    │   │              │  If no change → STOP                    │
    │   └──────────────┘                                        │
    └───────────────────────────────────────────────────────────┘
```

### Detailed Description

| Step | Operation | Mathematical Form |
|------|-----------|-------------------|
| **Initialize** | Pick K starting centroids | μ₁⁰, μ₂⁰, …, μₖ⁰ |
| **Assign** | Each point → nearest centroid | cᵢ = argmin_k ‖xᵢ − μₖ‖² |
| **Update** | Centroid = mean of assigned points | μₖ = (1/|Cₖ|) Σ_{xᵢ∈Cₖ} xᵢ |
| **Check** | Stop if assignments unchanged | labels⁽ᵗ⁾ == labels⁽ᵗ⁻¹⁾ |

---

## 2. Pseudocode

```
    ALGORITHM: K-Means Clustering
    ─────────────────────────────────────────
    Input:  X = {x₁, ..., xₙ} ∈ ℝⁿˣᵈ    (data matrix)
            K                              (number of clusters)
            max_iter                       (maximum iterations)
            tol                            (convergence tolerance)

    Output: labels ∈ {1, ..., K}ⁿ         (cluster assignments)
            μ₁, ..., μₖ ∈ ℝᵈ             (final centroids)

    1.  INITIALIZE centroids μ₁, ..., μₖ
        (e.g., randomly select K data points)

    2.  FOR t = 1, 2, ..., max_iter:
    
        3.  // ASSIGNMENT STEP
            FOR i = 1 to n:
                cᵢ = argmin   ‖xᵢ - μₖ‖²
                     k=1,...,K

        4.  // UPDATE STEP
            FOR k = 1 to K:
                Cₖ = {xᵢ : cᵢ = k}           // points in cluster k
                IF |Cₖ| > 0:
                    μₖ = (1/|Cₖ|) · Σ_{xᵢ∈Cₖ} xᵢ

        5.  // CONVERGENCE CHECK
            IF labels have not changed:
                BREAK
            IF max centroid shift < tol:
                BREAK

    6.  RETURN labels, centroids
```

---

## 3. Step-by-Step Numerical Example

### Setup

```
    Given 8 points in 2D, cluster into K=2:

    Point  x₁    x₂
    ─────  ────  ────
    A      1.0   1.0
    B      1.5   2.0
    C      3.0   4.0
    D      5.0   7.0
    E      3.5   5.0
    F      4.5   5.0
    G      3.5   4.5
    H      6.0   7.0
```

### STEP 1: Initialize (randomly pick 2 centroids)

```
    Select A and D as initial centroids:
    μ₁⁰ = (1.0, 1.0)    (point A)
    μ₂⁰ = (5.0, 7.0)    (point D)
```

### STEP 2a: Assign (Iteration 1)

Calculate squared Euclidean distance from each point to each centroid:

```
    d²(x, μ) = (x₁ - μ₁)² + (x₂ - μ₂)²

    Point │ d²(·, μ₁)           │ d²(·, μ₂)           │ Cluster
    ──────┼──────────────────────┼──────────────────────┼────────
    A     │ (0)²+(0)²     = 0.0 │ (4)²+(6)²    = 52.0 │  C₁
    B     │ (0.5)²+(1)²   = 1.25│ (3.5)²+(5)²  = 37.25│  C₁
    C     │ (2)²+(3)²     = 13.0│ (2)²+(3)²    = 13.0 │  C₁ *
    D     │ (4)²+(6)²     = 52.0│ (0)²+(0)²    = 0.0  │  C₂
    E     │ (2.5)²+(4)²   = 22.25│(1.5)²+(2)²  = 6.25 │  C₂
    F     │ (3.5)²+(4)²   = 28.25│(0.5)²+(2)²  = 4.25 │  C₂
    G     │ (2.5)²+(3.5)² = 18.5│ (1.5)²+(2.5)²= 8.5  │  C₂
    H     │ (5)²+(6)²     = 61.0│ (1)²+(0)²    = 1.0  │  C₂

    * For C: tie broken arbitrarily (assign to C₁)

    Cluster 1: {A, B, C}
    Cluster 2: {D, E, F, G, H}
```

### STEP 3a: Update (Iteration 1)

```
    μ₁¹ = mean of {A, B, C}
        = ((1.0+1.5+3.0)/3, (1.0+2.0+4.0)/3)
        = (5.5/3, 7.0/3)
        = (1.833, 2.333)

    μ₂¹ = mean of {D, E, F, G, H}
        = ((5.0+3.5+4.5+3.5+6.0)/5, (7.0+5.0+5.0+4.5+7.0)/5)
        = (22.5/5, 28.5/5)
        = (4.500, 5.700)
```

### STEP 2b: Assign (Iteration 2)

```
    New centroids: μ₁¹ = (1.833, 2.333),  μ₂¹ = (4.500, 5.700)

    Point │ d²(·, μ₁¹)          │ d²(·, μ₂¹)          │ Cluster
    ──────┼──────────────────────┼──────────────────────┼────────
    A     │ 0.694 + 1.778 = 2.47│ 12.25 + 22.09= 34.34│  C₁
    B     │ 0.111 + 0.111 = 0.22│ 9.00 + 13.69 = 22.69│  C₁
    C     │ 1.361 + 2.778 = 4.14│ 2.25 + 2.89  = 5.14 │  C₁
    D     │10.028 +21.778 =31.81│ 0.25 + 1.69  = 1.94 │  C₂
    E     │ 2.778 + 7.111 = 9.89│ 1.00 + 0.49  = 1.49 │  C₂
    F     │ 7.111 + 7.111 =14.22│ 0.00 + 0.49  = 0.49 │  C₂
    G     │ 2.778 + 4.694 = 7.47│ 1.00 + 1.44  = 2.44 │  C₂
    H     │17.361 +21.778 =39.14│ 2.25 + 1.69  = 3.94 │  C₂

    Cluster 1: {A, B, C}     — same as before!
    Cluster 2: {D, E, F, G, H} — same as before!

    → Assignments unchanged → CONVERGED!
```

### Final Result

```
    y
    8│
    7│                          D●    H●     Cluster 2
    6│                                       μ₂ = (4.50, 5.70)
    5│                    E●   F●
    4│              C○     G●
    3│                                       Cluster 1
    2│    B○                                 μ₁ = (1.83, 2.33)
    1│  A○
    0│
     └─────────────────────────────────── x
      0  1  2  3  4  5  6  7

    ○ = Cluster 1,  ● = Cluster 2
    
    Final Inertia (WCSS):
    J = d²(A,μ₁) + d²(B,μ₁) + d²(C,μ₁) + d²(D,μ₂) + d²(E,μ₂) + d²(F,μ₂) + d²(G,μ₂) + d²(H,μ₂)
      = 2.47 + 0.22 + 4.14 + 1.94 + 1.49 + 0.49 + 2.44 + 3.94
      = 17.13
```

---

## 4. Convergence Criteria

K-Means can stop based on several criteria:

```
    ┌─────────────────────────────────────────────────────────────┐
    │               CONVERGENCE CRITERIA                           │
    │                                                             │
    │  1. NO ASSIGNMENT CHANGES:                                   │
    │     labels⁽ᵗ⁾ == labels⁽ᵗ⁻¹⁾                               │
    │     Most common criterion. Exact convergence.               │
    │                                                             │
    │  2. CENTROID SHIFT BELOW TOLERANCE:                          │
    │     max_k ‖μₖ⁽ᵗ⁾ - μₖ⁽ᵗ⁻¹⁾‖ < tol                         │
    │     Centroids barely moved → effectively converged.         │
    │                                                             │
    │  3. INERTIA CHANGE BELOW TOLERANCE:                          │
    │     |J⁽ᵗ⁾ - J⁽ᵗ⁻¹⁾| / J⁽ᵗ⁻¹⁾ < tol                       │
    │     Objective function plateaued.                            │
    │                                                             │
    │  4. MAXIMUM ITERATIONS REACHED:                              │
    │     t >= max_iter                                            │
    │     Safety net to prevent infinite loops.                    │
    │                                                             │
    │  sklearn default: tol = 1e-4 (criterion 2), max_iter = 300  │
    └─────────────────────────────────────────────────────────────┘
```

### Typical Convergence Behavior

```
    Inertia (J)
    │
    │╲
    │ ╲
    │  ╲
    │   ╲
    │    ╲___
    │        ╲____
    │             ╲_________
    │                       ╲___________________
    └──────────────────────────────────────────── Iteration
     1   2   3   4   5   6   7   8   9  10  ...

    Key observations:
    • Biggest improvements in first few iterations
    • J is MONOTONICALLY NON-INCREASING
    • Typically converges in 10-20 iterations for most datasets
```

---

## 5. Time and Space Complexity

### Time Complexity

```
    Per iteration:
    ┌──────────────┬─────────────────────────────────────────┐
    │ Step         │ Complexity                               │
    ├──────────────┼─────────────────────────────────────────┤
    │ Assignment   │ O(n · K · d)                             │
    │              │ Each point: compute distance to K        │
    │              │ centroids, each in d dimensions          │
    ├──────────────┼─────────────────────────────────────────┤
    │ Update       │ O(n · d)                                 │
    │              │ Scan all points, accumulate into         │
    │              │ centroid sums                            │
    ├──────────────┼─────────────────────────────────────────┤
    │ Total/iter   │ O(n · K · d)                             │
    └──────────────┴─────────────────────────────────────────┘

    Total: O(n · K · d · I)

    Where:
    n = number of data points
    K = number of clusters
    d = number of dimensions
    I = number of iterations until convergence
```

### Space Complexity

```
    ┌──────────────────┬──────────────────┐
    │ Storage          │ Space            │
    ├──────────────────┼──────────────────┤
    │ Data matrix X    │ O(n · d)         │
    │ Centroids μ      │ O(K · d)         │
    │ Labels           │ O(n)             │
    │ Distances (temp) │ O(n · K) or O(n) │
    ├──────────────────┼──────────────────┤
    │ Total            │ O(n · d)         │
    └──────────────────┴──────────────────┘
```

### Practical Timing

```
    n=10,000, K=10, d=50:

    Assignment: 10000 × 10 × 50 = 5,000,000 operations/iter
    Update:     10000 × 50 = 500,000 operations/iter
    ~20 iterations → ~110,000,000 total ops → ~0.1 seconds

    n=1,000,000, K=100, d=200:

    Per iteration: 10⁶ × 100 × 200 = 2×10¹⁰ ops
    ~30 iterations → 6×10¹¹ total ops → ~10 minutes (single CPU)
```

---

## 6. K-Means from Scratch (Python)

```python
import numpy as np

class KMeansFromScratch:
    """K-Means clustering implemented from scratch using NumPy."""
    
    def __init__(self, n_clusters=3, max_iter=300, tol=1e-4, random_state=None):
        self.n_clusters = n_clusters
        self.max_iter = max_iter
        self.tol = tol
        self.random_state = random_state
    
    def fit(self, X):
        """Fit K-Means to data X."""
        rng = np.random.RandomState(self.random_state)
        n_samples, n_features = X.shape
        
        # Step 1: Initialize centroids (random selection from data)
        indices = rng.choice(n_samples, self.n_clusters, replace=False)
        self.centroids_ = X[indices].copy()
        
        for iteration in range(self.max_iter):
            # Step 2: Assign each point to nearest centroid
            distances = self._compute_distances(X, self.centroids_)
            labels = np.argmin(distances, axis=1)
            
            # Step 3: Update centroids
            new_centroids = np.zeros_like(self.centroids_)
            for k in range(self.n_clusters):
                mask = labels == k
                if mask.sum() > 0:
                    new_centroids[k] = X[mask].mean(axis=0)
                else:
                    # Handle empty cluster: reinitialize randomly
                    new_centroids[k] = X[rng.randint(n_samples)]
            
            # Step 4: Check convergence
            centroid_shift = np.max(np.linalg.norm(
                new_centroids - self.centroids_, axis=1))
            self.centroids_ = new_centroids
            
            if centroid_shift < self.tol:
                break
        
        self.labels_ = labels
        self.n_iter_ = iteration + 1
        self.inertia_ = self._compute_inertia(X, labels, self.centroids_)
        return self
    
    def predict(self, X):
        """Assign new points to nearest centroid."""
        distances = self._compute_distances(X, self.centroids_)
        return np.argmin(distances, axis=1)
    
    def fit_predict(self, X):
        """Fit and return cluster labels."""
        self.fit(X)
        return self.labels_
    
    def _compute_distances(self, X, centroids):
        """Compute squared Euclidean distances: each point to each centroid.
        
        Uses the expansion: ‖x - μ‖² = ‖x‖² - 2xᵀμ + ‖μ‖²
        """
        # Shape: (n_samples, n_clusters)
        X_sq = np.sum(X ** 2, axis=1, keepdims=True)       # (n, 1)
        C_sq = np.sum(centroids ** 2, axis=1, keepdims=True) # (K, 1)
        cross = X @ centroids.T                              # (n, K)
        distances = X_sq - 2 * cross + C_sq.T                # (n, K)
        return distances
    
    def _compute_inertia(self, X, labels, centroids):
        """Compute WCSS (within-cluster sum of squares)."""
        inertia = 0.0
        for k in range(self.n_clusters):
            mask = labels == k
            if mask.sum() > 0:
                inertia += np.sum((X[mask] - centroids[k]) ** 2)
        return inertia


# ---- DEMO ----
if __name__ == "__main__":
    from sklearn.datasets import make_blobs
    
    X, y_true = make_blobs(n_samples=300, centers=3, 
                             cluster_std=1.0, random_state=42)
    
    km = KMeansFromScratch(n_clusters=3, random_state=42)
    labels = km.fit_predict(X)
    
    print(f"Converged in {km.n_iter_} iterations")
    print(f"Inertia: {km.inertia_:.2f}")
    print(f"Cluster sizes: {np.bincount(labels)}")
    print(f"Centroids:\n{km.centroids_}")
```

---

## 7. K-Means with scikit-learn

### Basic Usage

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
import numpy as np

# Generate data
X, y_true = make_blobs(n_samples=500, centers=4, 
                         cluster_std=1.0, random_state=42)

# Create and fit K-Means
kmeans = KMeans(
    n_clusters=4,          # K: number of clusters
    init='k-means++',      # initialization method (default)
    n_init=10,             # number of runs with different seeds
    max_iter=300,          # maximum iterations per run
    tol=1e-4,              # convergence tolerance
    random_state=42,       # reproducibility
    algorithm='lloyd',     # 'lloyd' (classic) or 'elkan'
)

# Fit and predict
labels = kmeans.fit_predict(X)

# Access results
print(f"Inertia (WCSS): {kmeans.inertia_:.2f}")
print(f"Iterations: {kmeans.n_iter_}")
print(f"Cluster centers shape: {kmeans.cluster_centers_.shape}")
print(f"Labels shape: {kmeans.labels_.shape}")
print(f"Cluster sizes: {np.bincount(labels)}")
```

### Key Parameters Explained

```
    ┌──────────────┬────────────────────────────────────────────────────┐
    │ Parameter    │ Description                                        │
    ├──────────────┼────────────────────────────────────────────────────┤
    │ n_clusters   │ K — the number of clusters to find                │
    │ init         │ 'k-means++' (smart), 'random', or array           │
    │ n_init       │ Number of times to run with different init        │
    │              │ (returns best result by inertia)                   │
    │ max_iter     │ Max iterations per single run                     │
    │ tol          │ Convergence tolerance on centroid shift            │
    │ algorithm    │ 'lloyd' (classic O(nKd)), 'elkan' (faster w/      │
    │              │ triangle inequality, good for low-d)               │
    │ random_state │ Seed for reproducibility                          │
    └──────────────┴────────────────────────────────────────────────────┘
```

### Elkan vs Lloyd Algorithm

```
    Lloyd (classic):
    - Computes ALL n×K distances each iteration
    - Simple, works well in high dimensions
    - O(n·K·d) per iteration

    Elkan (1993):
    - Uses triangle inequality to skip distance calculations
    - ‖x - μⱼ‖ ≥ |‖x - μᵢ‖ - ‖μᵢ - μⱼ‖|
    - If lower bound > current best, skip!
    - Much faster for low/medium dimensions
    - Extra O(K²) overhead per iteration (centroid-centroid distances)
```

---

## 8. Worked Example — Iris Dataset

```python
from sklearn.cluster import KMeans
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import adjusted_rand_score, silhouette_score
import numpy as np

# Load Iris dataset
iris = load_iris()
X = iris.data          # 150 samples, 4 features
y_true = iris.target   # 3 species (we pretend we don't have these)

# Standardize features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Fit K-Means with K=3
kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
labels = kmeans.fit_predict(X_scaled)

# Evaluation
print("=== K-Means on Iris Dataset ===\n")
print(f"Inertia: {kmeans.inertia_:.2f}")
print(f"Silhouette Score: {silhouette_score(X_scaled, labels):.3f}")
print(f"Adjusted Rand Index: {adjusted_rand_score(y_true, labels):.3f}")
print(f"\nCluster sizes: {np.bincount(labels)}")

# Centroid analysis
feature_names = iris.feature_names
print(f"\nCentroid profiles (standardized):")
for k in range(3):
    print(f"  Cluster {k}: {dict(zip(feature_names, kmeans.cluster_centers_[k].round(2)))}")

# Cross-tabulation with true labels
print(f"\nCluster vs True Species:")
print(f"{'':>10} | Setosa | Versicolor | Virginica")
print("-" * 50)
for k in range(3):
    mask = labels == k
    counts = np.bincount(y_true[mask], minlength=3)
    print(f"Cluster {k} | {counts[0]:>6} | {counts[1]:>10} | {counts[2]:>9}")
```

### Expected Output Analysis

```
    Typical results:

    Inertia: ~140
    Silhouette Score: ~0.46
    Adjusted Rand Index: ~0.73

    Cross-tab:
                | Setosa | Versicolor | Virginica
    ──────────────────────────────────────────────
    Cluster 0   |     50 |          0 |         0     ← pure Setosa!
    Cluster 1   |      0 |         39 |        14     ← mostly Versicolor
    Cluster 2   |      0 |         11 |        36     ← mostly Virginica

    Key insight: K-Means perfectly separates Setosa but struggles
    to distinguish Versicolor from Virginica (they overlap in feature space).
```

---

## 9. Summary Table

```
┌───────────────────────┬──────────────────────────────────────────────────┐
│ Aspect                │ Detail                                           │
├───────────────────────┼──────────────────────────────────────────────────┤
│ Steps                 │ Initialize → Assign → Update → Check → Repeat   │
│ Assignment Rule       │ cᵢ = argmin_k ‖xᵢ − μₖ‖²                      │
│ Update Rule           │ μₖ = (1/|Cₖ|) Σ_{xᵢ∈Cₖ} xᵢ                   │
│ Convergence           │ Guaranteed (finite # partitions, J decreasing)  │
│ Time Complexity       │ O(n · K · d · I) per run                        │
│ Space Complexity      │ O(n · d)                                         │
│ Typical Iterations    │ 10-30 for most datasets                          │
│ sklearn Default Init  │ K-Means++ with n_init=10                        │
│ sklearn Algorithms    │ 'lloyd' (classic) or 'elkan' (faster, low-d)    │
│ Empty Cluster Fix     │ Reinitialize to random point or farthest point  │
└───────────────────────┴──────────────────────────────────────────────────┘
```

---

## 10. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | K-Means alternates between **assignment** (nearest centroid) and **update** (recompute mean) |
| 2 | Each iteration **decreases or maintains** the objective (WCSS/inertia) |
| 3 | Time complexity is **O(nKdI)** — linear in data size, making it very scalable |
| 4 | Convergence is **guaranteed** but typically takes only **10-30 iterations** |
| 5 | sklearn's `KMeans` uses **K-Means++** init and **n_init=10** runs by default |
| 6 | The **Elkan** algorithm can be significantly faster for low-dimensional data |
| 7 | Always **standardize features** before applying K-Means |

---

## 11. Revision Questions

1. **Algorithm Steps:** Write out the K-Means algorithm in pseudocode. For each step, state the mathematical operation and its time complexity.

2. **Numerical Example:** Given the points (1,1), (2,1), (4,3), (5,4), apply K-Means with K=2 and initial centroids μ₁=(1,1), μ₂=(5,4). Show all iterations until convergence.

3. **Convergence:** List three different convergence criteria for K-Means. Which does scikit-learn use by default?

4. **Complexity:** If you double the number of data points, by what factor does the runtime approximately increase? What about doubling K?

5. **Implementation:** In the from-scratch implementation, explain the vectorized distance computation using ‖x−μ‖² = ‖x‖² − 2xᵀμ + ‖μ‖². Why is this faster than a naive loop?

6. **Iris Example:** Why does K-Means perfectly separate Setosa but mix Versicolor and Virginica? What does this tell us about the feature space geometry?

---

<div align="center">

| [← Algorithm Intuition](./01-algorithm-intuition.md) | [Up: K-Means Clustering](./README.md) | [Next: Choosing K →](./03-choosing-k.md) |
|:----------------------------------------------------:|:--------------------------------------:|:----------------------------------------:|

</div>
