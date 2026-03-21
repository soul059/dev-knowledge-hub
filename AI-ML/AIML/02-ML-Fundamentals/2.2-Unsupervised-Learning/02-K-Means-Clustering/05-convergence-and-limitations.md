# 2.5 Convergence and Limitations of K-Means

> **Chapter Overview:** K-Means is fast and simple, but it has well-documented limitations. This chapter covers the convergence proof, the local optima problem, and the specific data geometries where K-Means fails — non-convex clusters, varying density, varying sizes, and outlier sensitivity — each illustrated with ASCII failure cases.

---

## Table of Contents

1. [Convergence Proof](#1-convergence-proof)
2. [Local Optima Problem](#2-local-optima-problem)
3. [Limitation 1: Non-Convex Clusters](#3-limitation-1-non-convex-clusters)
4. [Limitation 2: Clusters of Varying Size](#4-limitation-2-clusters-of-varying-size)
5. [Limitation 3: Clusters of Varying Density](#5-limitation-3-clusters-of-varying-density)
6. [Limitation 4: Outlier Sensitivity](#6-limitation-4-outlier-sensitivity)
7. [Limitation 5: Fixed K Requirement](#7-limitation-5-fixed-k-requirement)
8. [Limitation 6: High Dimensionality](#8-limitation-6-high-dimensionality)
9. [When K-Means Works Well vs Fails](#9-when-k-means-works-well-vs-fails)
10. [Alternatives for Each Failure Case](#10-alternatives-for-each-failure-case)
11. [Python Demonstrations](#11-python-demonstrations)
12. [Key Takeaways](#12-key-takeaways)
13. [Revision Questions](#13-revision-questions)

---

## 1. Convergence Proof

### Theorem: K-Means Always Converges

**Statement:** The K-Means algorithm terminates in a finite number of iterations.

### Proof

```
    Step 1: The objective function J is bounded.
    
    J = Σₖ Σᵢ∈Cₖ ‖xᵢ - μₖ‖²  ≥ 0
    
    J is a sum of squared distances → non-negative → bounded below by 0.
    
    
    Step 2: Each iteration does not increase J.
    
    a) ASSIGNMENT STEP (fix centroids, optimize assignments):
       Each point is assigned to its NEAREST centroid.
       If point xᵢ moves from cluster a to cluster b, then:
       ‖xᵢ - μ_b‖² < ‖xᵢ - μ_a‖²
       → The contribution of xᵢ to J DECREASES.
       → J cannot increase.
    
    b) UPDATE STEP (fix assignments, optimize centroids):
       The new centroid μₖ = mean of cluster Cₖ.
       The mean minimizes Σᵢ∈Cₖ ‖xᵢ - c‖² for any c.
       → J cannot increase.
    
    Therefore: J⁽ᵗ⁺¹⁾ ≤ J⁽ᵗ⁾ for all t.
    
    
    Step 3: The number of possible partitions is finite.
    
    There are at most Kⁿ possible assignments of n points to K clusters.
    (In practice, much fewer due to the Voronoi constraint.)
    Since J decreases strictly (or stays equal) and partitions are finite,
    the algorithm must eventually revisit a partition.
    But once it revisits, J hasn't changed → convergence.
    
    Therefore: K-Means converges in at most Kⁿ iterations.  ∎
    
    (In practice: typically 10-30 iterations, not Kⁿ!)
```

### Important Caveat

```
    ┌──────────────────────────────────────────────────────────┐
    │                     ⚠️  WARNING  ⚠️                      │
    │                                                          │
    │  K-Means converges to a LOCAL minimum, NOT necessarily   │
    │  the GLOBAL minimum of J.                                │
    │                                                          │
    │  Global optimum for K-Means is NP-hard in general        │
    │  (even for K=2 in general dimension d, or d=2 for        │
    │  general K).                                             │
    │                                                          │
    │  This is why K-Means++ and n_init > 1 are important!     │
    └──────────────────────────────────────────────────────────┘
```

---

## 2. Local Optima Problem

### Visualization

```
    WCSS landscape (conceptual):
    
    J(partition)
    │
    │  ╲                        ╱╲
    │   ╲   ╱╲           ╱╲   ╱  ╲
    │    ╲ ╱  ╲    ╱╲   ╱  ╲ ╱    ╲
    │     ╲    ╲  ╱  ╲ ╱    ╲      ╲
    │      ╲    ╲╱    ╲      ╲      ╲
    │       ╲         ╲       ╲
    │        ╲    B    ╲   C   ╲
    │         ●         ●       ●
    │     A = local     local   local
    │     GLOBAL min    min     min
    │
    └──────────────────────────────────── partition space

    Depending on initialization:
    • Init near A → converges to GLOBAL minimum ✓
    • Init near B → converges to local minimum B ✗
    • Init near C → converges to local minimum C ✗
```

### Example of Two Different Local Minima

```
    Same data, K=2, two different initializations:
    
    Run 1 (Good):                    Run 2 (Bad):
    
    ○ ○ ○ ○ ○    ● ● ● ● ●         ○ ○ ○ ○ ○ ○ ○ ○ ○ ○
    ○ ○ ○ ○ ○    ● ● ● ● ●         ● ● ● ● ● ● ● ● ● ●
    ○ ○ ○ ○ ○    ● ● ● ● ●         ○ ○ ○ ○ ○ ○ ○ ○ ○ ○
    
    J₁ = 100  (correct split)        J₂ = 350  (horizontal split!)
    GLOBAL minimum                   LOCAL minimum
```

---

## 3. Limitation 1: Non-Convex Clusters

### The Problem

K-Means creates **Voronoi cells** — which are always **convex**. It cannot identify clusters with non-convex (concave, ring, spiral) shapes.

### ASCII: Concentric Rings

```
    TRUE CLUSTERS:                    K-MEANS RESULT (K=2):
    (inner ring + outer ring)         (splits into LEFT and RIGHT!)
    
          ○ ○ ○ ○ ○                        ● ● ● ○ ○
        ○             ○                  ● ● ●     ○ ○
       ○    ● ● ●      ○               ● ● ● ● ●   ○ ○
      ○   ●       ●     ○              ● ● ●       ○ ○ ○
      ○   ●       ●     ○              ● ● ●       ○ ○ ○
       ○    ● ● ●      ○               ● ● ● ● ●   ○ ○
        ○             ○                  ● ● ●     ○ ○
          ○ ○ ○ ○ ○                        ● ● ● ○ ○
    
    K-Means CANNOT separate these!     It draws a vertical LINE
    The boundary must be circular,     through the center instead.
    but K-Means only makes LINEAR 
    boundaries.
```

### ASCII: Two Moons

```
    TRUE CLUSTERS:                    K-MEANS RESULT (K=2):

      ○ ○ ○ ○ ○                         ○ ○ ○ ○ ○
    ○ ○ ○ ○ ○ ○ ○                      ○ ○ ○ ○ ● ● ●
    ○ ○ ○ ○ ○ ○ ○                      ○ ○ ○ ○ ● ● ● ●
          ● ● ● ● ● ● ●              ○ ○ ● ● ● ● ● ● ●
          ● ● ● ● ● ● ●                  ● ● ● ● ● ● ●
            ● ● ● ● ●                      ● ● ● ● ●

    Two interleaving crescents         K-Means draws a horizontal
    (moons) — non-convex!              line — completely WRONG!
```

### ASCII: Spiral Clusters

```
    TRUE:                    K-MEANS:
         ○                        ○
       ○ ○ ●                    ● ○ ●
     ○ ○ ● ● ●               ● ● ● ● ○
    ○ ● ● ● ● ○             ● ● ● ● ○ ○
     ● ● ○ ○ ○               ● ○ ○ ○ ○
       ● ○ ○                    ● ○ ○
         ●                        ●
    
    Spiral = non-convex →    K-Means: random mess
    K-Means FAILS
```

### Why It Fails (Mathematical)

```
    K-Means boundary between clusters i and j:
    
    {x : ‖x - μᵢ‖² = ‖x - μⱼ‖²}
    
    This is always a HYPERPLANE (linear boundary).
    
    Non-convex clusters require NONLINEAR boundaries.
    
    Fix: Use DBSCAN, Spectral Clustering, or kernel K-Means.
```

---

## 4. Limitation 2: Clusters of Varying Size

### The Problem

K-Means tends to **equalize cluster sizes** because it minimizes total WCSS, which penalizes large spreads.

### ASCII: Large vs Small Cluster

```
    TRUE CLUSTERS:                    K-MEANS RESULT:
    
    ○ ○ ○ ○ ○ ○ ○ ○ ○  ● ● ●        ○ ○ ○ ○ ○│● ● ● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○  ● ● ●        ○ ○ ○ ○ ○│● ● ● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○  ● ● ●        ○ ○ ○ ○ ○│● ● ● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○  ● ● ●        ○ ○ ○ ○ ○│● ● ● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○  ● ● ●        ○ ○ ○ ○ ○│● ● ● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○                ○ ○ ○ ○ ○│● ● ●
    ○ ○ ○ ○ ○ ○ ○ ○ ○                ○ ○ ○ ○ ○│
    
    Large cluster (80%)  Small(20%)    K-Means "steals" points from
                                       the large cluster to balance!
```

### Why It Happens

```
    WCSS is minimized when each cluster has similar total variance.
    
    A large cluster contributes more to WCSS.
    Splitting it reduces WCSS more than keeping it whole.
    
    Result: K-Means splits big clusters and merges small ones.
```

---

## 5. Limitation 3: Clusters of Varying Density

### The Problem

K-Means is equally affected by cluster density. A **sparse cluster** is treated the same as a **dense cluster**, leading to incorrect boundaries.

### ASCII: Dense vs Sparse Clusters

```
    TRUE CLUSTERS:                    K-MEANS RESULT:

    ●●●●●●      ○   ○   ○   ○       ●●●●●●   │  ○   ○   ○   ○
    ●●●●●●●      ○   ○   ○   ○      ●●●●●●●  │   ○   ○   ○   ○
    ●●●●●●      ○   ○   ○   ○       ●●●●●●   │  ○   ○   ○   ○
    ●●●●●●●      ○   ○   ○   ○      ●●●●●●●○ │   ○   ○   ○   ○
    ●●●●●●      ○   ○   ○   ○       ●●●●●●  ○│  ○   ○   ○   ○
    
    Dense cluster   Sparse cluster    Boundary is NOT between clusters!
                                      K-Means draws boundary at the
                                      MIDPOINT of centroids, ignoring
                                      that the left cluster is much
                                      tighter.
```

### Mathematical Explanation

```
    K-Means boundary: equidistant hyperplane between centroids.
    
    This is correct ONLY IF both clusters have the SAME variance (spread).
    
    For Gaussian clusters:
    • Cluster 1: σ₁ = 0.5 (tight)
    • Cluster 2: σ₂ = 3.0 (spread)
    
    Optimal boundary should be CLOSER to the tight cluster,
    but K-Means places it at the MIDPOINT.
    
    Fix: GMM (Gaussian Mixture Models) can handle different Σₖ.
```

---

## 6. Limitation 4: Outlier Sensitivity

### The Problem

The mean (centroid) is **highly sensitive to outliers**. A single extreme point can shift a centroid significantly.

### Mathematical Impact

```
    Cluster: {(1,1), (2,2), (3,3), (2,1), (1,2)}
    
    Centroid without outlier:
    μ = (9/5, 9/5) = (1.8, 1.8)
    
    Add outlier: (100, 100)
    
    Centroid WITH outlier:
    μ = (109/6, 109/6) = (18.17, 18.17)
    
    The centroid shifted by (16.37, 16.37) due to ONE point!
```

### ASCII: Outlier Effect

```
    WITHOUT OUTLIER:                  WITH OUTLIER:
    
      ○ ○                              ○ ○
    ○ ★ ○    ← centroid = (2,2)      ○   ○         ★ ← centroid
      ○ ○                              ○ ○      shifted!
    
                                                        ◆ outlier
                                                        at (20, 20)
    
    The centroid is PULLED toward the outlier,
    misrepresenting the cluster center.
```

### Mitigation

```
    1. K-MEDOIDS (PAM): Use the MEDIAN data point as center
       → Robust to outliers (median is robust, mean is not)
    
    2. OUTLIER REMOVAL: Detect and remove outliers before clustering
       → Use IQR, Z-score, or Isolation Forest
    
    3. TRIMMED K-MEANS: Ignore top-α% most distant points when 
       computing centroids
    
    4. WINSORIZATION: Cap extreme values before clustering
```

---

## 7. Limitation 5: Fixed K Requirement

### The Problem

K-Means requires specifying K in advance. If you don't know how many clusters exist, you must rely on methods like elbow/silhouette, which are imperfect.

```
    "How many clusters should I use?"

    K-Means:  "You tell ME."
    DBSCAN:   "I'll figure it out."
    
    ┌────────────────────────────────────────────────────┐
    │  Algorithm        │ Requires K? │ Alternative      │
    ├────────────────────┼─────────────┼──────────────────┤
    │  K-Means          │ YES         │ Elbow, Sil., Gap │
    │  DBSCAN           │ NO          │ Finds K auto.    │
    │  Mean Shift        │ NO          │ Bandwidth param  │
    │  Hierarchical     │ NO *        │ Cut dendrogram   │
    │  GMM + BIC        │ Sort of     │ Select by BIC    │
    │  X-Means          │ NO          │ Splits by BIC    │
    └────────────────────┴─────────────┴──────────────────┘
    * Hierarchical produces all K values; you choose where to cut.
```

---

## 8. Limitation 6: High Dimensionality

### The Problem

In high dimensions, Euclidean distance loses discriminative power (curse of dimensionality), and K-Means struggles.

```
    d=2:    Clusters are well-separated    → K-Means works great
    d=100:  All points ~equidistant        → K-Means finds random partitions
    d=1000: Distance ratios → 1            → K-Means is essentially random

    Mitigation:
    1. Apply PCA/UMAP BEFORE K-Means
    2. Use subspace clustering
    3. Use cosine distance (not Euclidean) — requires kernel K-Means
```

---

## 9. When K-Means Works Well vs Fails

```
┌──────────────────────────────┬────────────────────────────────────┐
│ K-MEANS WORKS WELL           │ K-MEANS FAILS                     │
├──────────────────────────────┼────────────────────────────────────┤
│ Spherical/globular clusters  │ Non-convex shapes (rings, moons)  │
│ Similar cluster sizes        │ Very different cluster sizes       │
│ Similar cluster densities    │ Very different cluster densities   │
│ Well-separated clusters      │ Overlapping clusters               │
│ No outliers                  │ Outliers present                   │
│ Low-moderate dimensions      │ Very high dimensions               │
│ K is known or estimable      │ K is unknown and hard to estimate  │
│ Large datasets (scalable)    │ (Still works — it's fast!)         │
│ Continuous features          │ Categorical features               │
│ Pre-scaled features          │ Features on different scales       │
└──────────────────────────────┴────────────────────────────────────┘
```

### Visual Summary of Failure Cases

```
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ ╭──────╮ │  │          │  │ ●●●●     │  │  ○○○     │
    │ │ ╭──╮ │ │  │ ●●●  ○○ │  │ ●●●●  ○  │  │ ○○○  ◆  │
    │ │ │  │ │ │  │ ●●●  ○○ │  │ ●●●●  ○  │  │ ○○○     │
    │ │ ╰──╯ │ │  │ ●●●  ○  │  │ ●●●●  ○  │  │          │
    │ ╰──────╯ │  │ ●●●     │  │ ●●●●     │  │          │
    │  Rings   │  │ Unequal │  │ Unequal  │  │ Outlier  │
    │  FAIL    │  │ Size    │  │ Density  │  │ FAIL     │
    │          │  │ FAIL    │  │ FAIL     │  │          │
    └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

---

## 10. Alternatives for Each Failure Case

```
┌───────────────────────┬─────────────────────┬──────────────────────────────┐
│ Failure Case          │ Better Algorithm    │ Why It Works                  │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ Non-convex shapes     │ DBSCAN              │ Density-based, arbitrary     │
│                       │ Spectral Clustering │ shape via graph Laplacian    │
│                       │ Kernel K-Means      │ Nonlinear feature map        │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ Varying sizes         │ GMM                 │ Different covariances per    │
│                       │                     │ cluster                      │
│                       │ DBSCAN              │ No centroid assumption       │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ Varying density       │ HDBSCAN             │ Hierarchical density-based   │
│                       │ OPTICS              │ Handles varying density      │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ Outliers              │ K-Medoids (PAM)     │ Median instead of mean       │
│                       │ DBSCAN              │ Labels outliers as noise     │
│                       │ Trimmed K-Means     │ Ignores extreme points       │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ Unknown K             │ DBSCAN              │ No K needed                  │
│                       │ Mean Shift          │ Bandwidth instead of K       │
│                       │ X-Means             │ Automatically selects K      │
├───────────────────────┼─────────────────────┼──────────────────────────────┤
│ High dimensionality   │ PCA + K-Means       │ Reduce dims first            │
│                       │ Spectral Clustering │ Works in graph space         │
│                       │ Subspace clustering │ Finds clusters in subspaces  │
└───────────────────────┴─────────────────────┴──────────────────────────────┘
```

---

## 11. Python Demonstrations

### Non-Convex Failure (Two Moons)

```python
from sklearn.cluster import KMeans, DBSCAN
from sklearn.datasets import make_moons
import numpy as np
import matplotlib.pyplot as plt

X, y_true = make_moons(n_samples=300, noise=0.08, random_state=42)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Ground truth
axes[0].scatter(X[:, 0], X[:, 1], c=y_true, cmap='viridis', s=30)
axes[0].set_title("True Clusters (Two Moons)")

# K-Means
km = KMeans(n_clusters=2, random_state=42, n_init=10)
axes[1].scatter(X[:, 0], X[:, 1], c=km.fit_predict(X), cmap='viridis', s=30)
axes[1].set_title(f"K-Means (FAILS)\nInertia: {km.inertia_:.2f}")

# DBSCAN
db = DBSCAN(eps=0.2, min_samples=5)
axes[2].scatter(X[:, 0], X[:, 1], c=db.fit_predict(X), cmap='viridis', s=30)
axes[2].set_title("DBSCAN (SUCCEEDS)")

plt.suptitle("K-Means Fails on Non-Convex Shapes", fontsize=14)
plt.tight_layout()
plt.savefig("kmeans_nonconvex_failure.png", dpi=150)
plt.show()
```

### Outlier Sensitivity

```python
import numpy as np
from sklearn.cluster import KMeans

np.random.seed(42)
# Normal cluster
X_normal = np.random.randn(100, 2) + [5, 5]
# Add outlier
X_outlier = np.array([[50, 50]])
X = np.vstack([X_normal, X_outlier])

# Without outlier
km_clean = KMeans(n_clusters=1, n_init=10, random_state=42)
km_clean.fit(X_normal)
print(f"Centroid without outlier: {km_clean.cluster_centers_[0].round(3)}")

# With outlier  
km_dirty = KMeans(n_clusters=1, n_init=10, random_state=42)
km_dirty.fit(X)
print(f"Centroid with outlier:    {km_dirty.cluster_centers_[0].round(3)}")
print(f"Centroid shift:           {np.linalg.norm(km_clean.cluster_centers_[0] - km_dirty.cluster_centers_[0]):.3f}")
```

### Varying Cluster Size Problem

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
from sklearn.metrics import adjusted_rand_score
import numpy as np

np.random.seed(42)
# Large cluster (1000 points) and small cluster (50 points)
X_large = np.random.randn(1000, 2) * 2 + [0, 0]
X_small = np.random.randn(50, 2) * 0.5 + [8, 8]
X = np.vstack([X_large, X_small])
y_true = np.array([0]*1000 + [1]*50)

km = KMeans(n_clusters=2, random_state=42, n_init=10)
labels = km.fit_predict(X)
ari = adjusted_rand_score(y_true, labels)
print(f"ARI: {ari:.3f}")
print(f"Cluster sizes: {np.bincount(labels)}")
print(f"True sizes: [1000, 50]")
# K-Means may split the large cluster instead of finding the small one
```

---

## 12. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | K-Means **always converges** — WCSS is non-increasing and partitions are finite |
| 2 | Convergence is to a **local optimum**, not necessarily the global optimum |
| 3 | K-Means **fails on non-convex** clusters because it creates linear (hyperplane) boundaries |
| 4 | **Varying cluster sizes** cause K-Means to split large clusters and merge small ones |
| 5 | **Varying densities** cause incorrect boundary placement (equidistant vs density-aware) |
| 6 | **Outliers** pull centroids away from the true cluster center |
| 7 | For each failure case, there exists a **better alternative** (DBSCAN, GMM, K-Medoids, etc.) |

---

## 13. Revision Questions

1. **Convergence Proof:** Provide a complete proof that K-Means converges. What are the two key facts used in the proof?

2. **Local Optima:** Explain why K-Means can converge to different solutions from different initializations. How do K-Means++ and n_init help mitigate this?

3. **Non-Convex Failure:** Draw an ASCII diagram of concentric rings. Explain mathematically why K-Means cannot separate them. What algorithm would you use instead?

4. **Outlier Sensitivity:** Show numerically how adding an outlier at (100, 100) to the points (1,1), (2,2), (3,3) changes the centroid. Why is the median more robust?

5. **Practical Decision:** Given a dataset with (a) arbitrary-shaped clusters, (b) outliers, and (c) unknown K, which algorithm would you recommend? Justify each aspect.

6. **Comparison:** Create a table comparing K-Means, DBSCAN, GMM, and Hierarchical Clustering on the six limitations discussed. Which algorithm handles the most failure cases?

---

<div align="center">

| [← K-Means++ Initialization](./04-initialization-kmeans-plus.md) | [Up: K-Means Clustering](./README.md) | [Next: Mini-Batch K-Means →](./06-mini-batch-kmeans.md) |
|:----------------------------------------------------------------:|:--------------------------------------:|:--------------------------------------------------------:|

</div>
