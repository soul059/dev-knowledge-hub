# 2.1 K-Means Algorithm — Intuition

> **Chapter Overview:** K-Means is the most widely used clustering algorithm in machine learning. This chapter builds deep intuition for how it works — from the partitioning concept and centroid-based reasoning to the mathematical objective (WCSS/inertia) and the geometric interpretation through Voronoi diagrams.

---

## Table of Contents

1. [What is Partitional Clustering?](#1-what-is-partitional-clustering)
2. [The Centroid-Based Idea](#2-the-centroid-based-idea)
3. [The Objective Function — WCSS / Inertia](#3-the-objective-function--wcss--inertia)
4. [Voronoi Diagrams — Geometric View](#4-voronoi-diagrams--geometric-view)
5. [How K-Means "Thinks" — Step-by-Step Intuition](#5-how-k-means-thinks--step-by-step-intuition)
6. [ASCII Visualization of K-Means](#6-ascii-visualization-of-k-means)
7. [Why K-Means Works — Intuitive Proofs](#7-why-k-means-works--intuitive-proofs)
8. [Key Properties of K-Means](#8-key-properties-of-k-means)
9. [Python — Your First K-Means](#9-python--your-first-k-means)
10. [Summary Table](#10-summary-table)
11. [Key Takeaways](#11-key-takeaways)
12. [Revision Questions](#12-revision-questions)

---

## 1. What is Partitional Clustering?

**Partitional clustering** divides a dataset into **K non-overlapping subsets** (partitions/clusters), where every data point belongs to exactly one cluster.

### Formal Definition

```
    Given: Dataset X = {x₁, x₂, ..., xₙ} where xᵢ ∈ ℝᵈ
    Find:  Partition P = {C₁, C₂, ..., Cₖ} such that:

    1. Cₖ ≠ ∅         for all k          (no empty clusters)
    2. Cᵢ ∩ Cⱼ = ∅    for i ≠ j          (non-overlapping)
    3. ∪ₖ Cₖ = X       (covers all data)  (exhaustive)
```

### Partitional vs Hierarchical vs Density-Based

```
    ┌──────────────────┬──────────────────────┬───────────────────┐
    │   PARTITIONAL    │   HIERARCHICAL       │   DENSITY-BASED   │
    │                  │                      │                   │
    │  ┌───┐  ┌───┐   │       ┌─┐            │  ┌~~~~~~~~┐      │
    │  │ ● │  │ ▲ │   │      ┌┴─┴┐           │  │ ○○○○○○ │  ★   │
    │  │ ● │  │ ▲ │   │     ┌┴┐ ┌┴┐          │  │ ○○○○○  │      │
    │  │ ● │  │ ▲ │   │    ┌┴┐ ┌┴┐┌┴┐        │  └~~~~~~~~┘      │
    │  └───┘  └───┘   │    a b c d e f        │     noise: ★     │
    │                  │                      │                   │
    │  Flat partition   │  Nested hierarchy    │  Arbitrary shapes │
    │  Requires K      │  No K needed         │  No K needed      │
    │  K-Means, K-Med. │  Agglom., BIRCH      │  DBSCAN, OPTICS   │
    └──────────────────┴──────────────────────┴───────────────────┘
```

---

## 2. The Centroid-Based Idea

K-Means represents each cluster by its **centroid** (center of mass) — the mean of all points assigned to that cluster.

### What is a Centroid?

```
    Centroid of cluster Cₖ:

                 1
    μₖ = ────── · Σ   xᵢ
          |Cₖ|   xᵢ∈Cₖ

    It's simply the AVERAGE of all points in the cluster.
```

### Geometric Intuition

```
    Cluster with 5 points:

    y
    │
    5│    ·(2,5)
    4│         ·(4,4)
    3│  ·(1,3)       ★(3,3) ← centroid = mean of all points
    2│       ·(3,2)
    1│  ·(2,1)
    │
    └──────────────── x
     1  2  3  4  5

    μ = ((2+4+1+3+2)/5, (5+4+3+2+1)/5) = (2.4, 3.0)
    
    The centroid minimizes the sum of squared distances
    to all points in the cluster.
```

### Why Centroids?

The centroid has a special property: **it is the point that minimizes the sum of squared Euclidean distances to all cluster members.**

```
    Proof: Let c be any point. We want to minimize:

    J(c) = Σ     ‖xᵢ - c‖²
          xᵢ∈Cₖ

    Taking derivative and setting to 0:

    ∂J/∂c = -2 · Σ (xᵢ - c) = 0
    
    → Σ xᵢ = |Cₖ| · c

    → c = (1/|Cₖ|) · Σ xᵢ = μₖ    ← the mean!
```

---

## 3. The Objective Function — WCSS / Inertia

### Within-Cluster Sum of Squares (WCSS)

K-Means minimizes the **total within-cluster sum of squares**, also called **inertia**:

```
    ┌──────────────────────────────────────────────────────┐
    │                                                      │
    │            K                                         │
    │  J = Σ     Σ     ‖xᵢ - μₖ‖²                        │
    │     k=1  xᵢ∈Cₖ                                      │
    │                                                      │
    │  Where:                                              │
    │    K    = number of clusters                         │
    │    Cₖ   = set of points in cluster k                │
    │    μₖ   = centroid of cluster k                     │
    │    ‖·‖² = squared Euclidean distance                │
    │                                                      │
    └──────────────────────────────────────────────────────┘
```

### Expanded Form

```
    For d-dimensional data:

                K          d
    J = Σ     Σ     Σ   (xᵢⱼ - μₖⱼ)²
       k=1  xᵢ∈Cₖ  j=1

    This sums the squared difference in EACH dimension 
    for EACH point from its assigned centroid.
```

### What WCSS Measures

```
    LOW WCSS (tight clusters):          HIGH WCSS (loose clusters):

      ┌───────────┐                      ┌───────────────────┐
      │  ○○       │                      │   ○     ○         │
      │ ○★○       │  J = small           │ ○    ★        ○   │  J = large
      │  ○○       │                      │        ○    ○     │
      └───────────┘                      └───────────────────┘
    Points close to centroid ★          Points far from centroid ★

    K-Means tries to find the partition that makes J as small as possible.
```

### Decomposition: Total SS = WCSS + BCSS

```
    Total Sum of Squares (TSS):

    TSS = Σᵢ ‖xᵢ - μ_global‖²

    Between-Cluster Sum of Squares (BCSS):

    BCSS = Σₖ |Cₖ| · ‖μₖ - μ_global‖²

    Relationship:
    TSS = WCSS + BCSS
    (Total = Within + Between)

    → Minimizing WCSS is equivalent to maximizing BCSS
    → Good clustering: clusters are tight (low WCSS) and well-separated (high BCSS)
```

---

## 4. Voronoi Diagrams — Geometric View

### What is a Voronoi Diagram?

A **Voronoi diagram** partitions space into regions, one per centroid, where each region contains all points closest to that centroid.

```
    Voronoi region for centroid μₖ:

    Vₖ = {x ∈ ℝᵈ : ‖x - μₖ‖ ≤ ‖x - μⱼ‖  for all j ≠ k}

    In words: All points in space that are closer to μₖ
              than to any other centroid.
```

### ASCII Voronoi Diagram (3 Centroids)

```
    ┌──────────────────────────────────────────┐
    │                    ╱                     │
    │    Region 1       ╱     Region 2         │
    │                  ╱                       │
    │       ★₁        ╱         ★₂             │
    │                ╱                         │
    │               ╱                          │
    │              ╱                           │
    │─────────────╱────────────────────────────│
    │            ╱╲                            │
    │           ╱  ╲                           │
    │          ╱    ╲       Region 3            │
    │         ╱      ╲                         │
    │        ╱        ╲        ★₃              │
    │       ╱          ╲                       │
    └──────────────────────────────────────────┘

    ★ = centroid
    Lines = equidistant boundaries between centroids
    Each point in Region k is closest to ★ₖ
```

### K-Means and Voronoi Diagrams

**K-Means creates a Voronoi partition of the data space.** At convergence:
1. Each point is assigned to the nearest centroid → defines Voronoi regions
2. Each centroid is the mean of its Voronoi region's data points

```
    The assignment step of K-Means is equivalent to:
    "Assign each point to the Voronoi region it falls in"

    The update step:
    "Recompute the centroid of each Voronoi region"
```

### Voronoi Boundaries and Decision Boundaries

```
    The boundary between clusters i and j is the set of points
    equidistant from μᵢ and μⱼ:

    ‖x - μᵢ‖² = ‖x - μⱼ‖²

    Expanding:
    xᵀx - 2μᵢᵀx + μᵢᵀμᵢ = xᵀx - 2μⱼᵀx + μⱼᵀμⱼ

    Simplifying:
    2(μⱼ - μᵢ)ᵀx = μⱼᵀμⱼ - μᵢᵀμᵢ

    This is a LINEAR equation in x → the boundary is a HYPERPLANE!
    
    → K-Means can only create LINEAR decision boundaries
    → This is why K-Means fails on non-convex clusters
```

---

## 5. How K-Means "Thinks" — Step-by-Step Intuition

### The Two Alternating Steps

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │  STEP 1: ASSIGN                 STEP 2: UPDATE           │
    │  (fix centroids,                (fix assignments,        │
    │   assign points)                 update centroids)       │
    │                                                          │
    │  For each point xᵢ:             For each cluster k:     │
    │    Find nearest centroid         Recompute centroid       │
    │    cᵢ = argmin ‖xᵢ - μₖ‖²      μₖ = mean(points in k)  │
    │          k                                               │
    │                                                          │
    │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │
    │  │ Assign  │ →  │ Update  │ →  │ Assign  │ → ... → Done │
    │  │ Points  │    │Centroids│    │ Points  │              │
    │  └─────────┘    └─────────┘    └─────────┘              │
    │                                                          │
    │  Repeat until no points change cluster (convergence)     │
    └──────────────────────────────────────────────────────────┘
```

### Why It Works (Intuition)

Each step **can only decrease** (or keep equal) the objective J:

```
    Step 1 (Assign): Moving a point to a closer centroid 
                     decreases its contribution to J
                     → J decreases or stays same

    Step 2 (Update): The mean minimizes sum of squared distances
                     → J decreases or stays same

    Since J is bounded below by 0 and decreases each iteration:
    → J must converge!
```

---

## 6. ASCII Visualization of K-Means

### Full K-Means Run (K=3)

```
    ITERATION 0 — Random Initialization:
    
    y
    8│                    · · ·
    7│                  · · · · ·
    6│                · · · · · ·
    5│  · · ·                         ★₁ (random centroid)
    4│  · · · ·
    3│  · · · ·      ★₂ (random)
    2│  · · ·                    · · ·
    1│                           · · · · ★₃ (random)
    0│                           · · ·
     └─────────────────────────────────── x
      0  1  2  3  4  5  6  7  8  9  10

    ITERATION 1 — Assign to nearest centroid:

    y
    8│                    ▲ ▲ ▲
    7│                  ▲ ▲ ▲ ▲ ▲
    6│                ▲ ▲ ▲ ▲ ▲ ▲
    5│  ● ● ●                         ★₁
    4│  ● ● ● ●
    3│  ● ● ● ●      ★₂
    2│  ● ● ●                    ■ ■ ■
    1│                           ■ ■ ■ ■ ★₃
    0│                           ■ ■ ■
     └─────────────────────────────────── x

    ITERATION 1 — Update centroids:

    y
    8│                    ▲ ▲ ▲
    7│                  ▲ ▲ ▲ ▲ ▲
    6│                ▲ ▲ ▲★₁▲ ▲     ★₁ moved to cluster center!
    5│  ● ● ●                   
    4│  ● ● ● ●
    3│  ●★₂● ●                       ★₂ moved to cluster center!
    2│  ● ● ●                    ■ ■ ■
    1│                           ■★₃■ ■   ★₃ moved to cluster center!
    0│                           ■ ■ ■
     └─────────────────────────────────── x

    ITERATION 2 — Reassign and update → no changes → CONVERGED!
```

---

## 7. Why K-Means Works — Intuitive Proofs

### Coordinate Descent Interpretation

K-Means is a special case of **coordinate descent** on the objective J:

```
    J(C, μ) = Σ    Σ     ‖xᵢ - μₖ‖²
             k=1  xᵢ∈Cₖ

    Two sets of variables:
    1. Assignments C = {C₁, ..., Cₖ}  (discrete)
    2. Centroids   μ = {μ₁, ..., μₖ}  (continuous)

    Step 1: Fix μ, optimize C  →  assign each point to nearest μₖ
    Step 2: Fix C, optimize μ  →  set each μₖ = mean of Cₖ

    Each step optimally solves one variable set given the other.
```

### Connection to EM Algorithm

K-Means is a **hard-assignment special case** of the EM algorithm for Gaussian Mixture Models:

```
    GMM (soft):                      K-Means (hard):
    E-step: p(k|xᵢ) = soft prob.    Assign: argmin_k ‖xᵢ-μₖ‖² (hard)
    M-step: update μ, Σ, π          Update: μₖ = mean(Cₖ)

    K-Means = GMM with:
    • All covariances Σₖ = σ²I (spherical, equal)
    • σ² → 0 (hard assignment limit)
    • Equal mixing weights πₖ = 1/K
```

---

## 8. Key Properties of K-Means

```
┌─────────────────────────┬──────────────────────────────────────────┐
│ Property                │ Description                              │
├─────────────────────────┼──────────────────────────────────────────┤
│ Type                    │ Partitional, centroid-based              │
│ Objective               │ Minimize WCSS (inertia)                  │
│ Time Complexity         │ O(n · K · d · I)                         │
│ Space Complexity        │ O(n · d + K · d)                         │
│ Guarantees              │ Converges to LOCAL optimum               │
│ Cluster Shape           │ Spherical (Voronoi cells)                │
│ Requires                │ Number of clusters K                     │
│ Sensitive to            │ Initialization, outliers, scale          │
│ Distance Metric         │ Euclidean (L2) only                      │
│ Handles                 │ Continuous numerical features            │
│ Does NOT handle well    │ Categorical data, non-convex shapes      │
│ Deterministic?          │ No (depends on random initialization)    │
│ Scalability             │ Very good (linear in n)                  │
└─────────────────────────┴──────────────────────────────────────────┘
```

---

## 9. Python — Your First K-Means

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# Generate synthetic data with 4 natural clusters
X, y_true = make_blobs(n_samples=400, centers=4, 
                         cluster_std=0.8, random_state=42)

# Fit K-Means
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
labels = kmeans.fit_predict(X)
centroids = kmeans.cluster_centers_

# Print results
print(f"Inertia (WCSS): {kmeans.inertia_:.2f}")
print(f"Number of iterations: {kmeans.n_iter_}")
print(f"Cluster sizes: {np.bincount(labels)}")
print(f"\nCentroids:\n{centroids}")

# Visualization
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Before clustering
axes[0].scatter(X[:, 0], X[:, 1], c='gray', alpha=0.5, s=30)
axes[0].set_title("Before K-Means (Raw Data)")
axes[0].set_xlabel("Feature 1")
axes[0].set_ylabel("Feature 2")

# After clustering
scatter = axes[1].scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', 
                           alpha=0.5, s=30)
axes[1].scatter(centroids[:, 0], centroids[:, 1], c='red', marker='X', 
                s=200, edgecolors='black', linewidths=2, label='Centroids')
axes[1].set_title(f"After K-Means (K=4, Inertia={kmeans.inertia_:.0f})")
axes[1].set_xlabel("Feature 1")
axes[1].legend()

plt.tight_layout()
plt.savefig("kmeans_intuition.png", dpi=150)
plt.show()
```

### Understanding the Output

```python
# Access detailed information
print(f"\n--- K-Means Detailed Output ---")
print(f"kmeans.cluster_centers_.shape: {kmeans.cluster_centers_.shape}")  # (K, d)
print(f"kmeans.labels_.shape: {kmeans.labels_.shape}")                    # (n,)
print(f"kmeans.inertia_: {kmeans.inertia_:.2f}")                         # scalar (WCSS)
print(f"kmeans.n_iter_: {kmeans.n_iter_}")                               # iterations to converge

# Compute per-cluster inertia
for k in range(4):
    mask = labels == k
    cluster_inertia = np.sum((X[mask] - centroids[k]) ** 2)
    print(f"Cluster {k}: {mask.sum()} points, inertia = {cluster_inertia:.2f}")
```

---

## 10. Summary Table

```
┌─────────────────────┬───────────────────────────────────────────────────┐
│ Concept             │ Key Formula / Idea                               │
├─────────────────────┼───────────────────────────────────────────────────┤
│ Partitioning        │ Divide data into K non-overlapping subsets       │
│ Centroid            │ μₖ = (1/|Cₖ|) Σ xᵢ  (mean of cluster k)       │
│ WCSS / Inertia      │ J = Σₖ Σᵢ∈Cₖ ‖xᵢ - μₖ‖²                      │
│ Voronoi Diagram     │ Regions where each point is closest to one μₖ   │
│ Assignment Rule     │ cᵢ = argmin_k ‖xᵢ - μₖ‖²                      │
│ Convergence         │ J is non-increasing → guaranteed convergence    │
│ Cluster Shape       │ Convex, spherical (Voronoi cells)               │
│ Decision Boundary   │ Linear hyperplanes between centroids            │
└─────────────────────┴───────────────────────────────────────────────────┘
```

---

## 11. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | K-Means is a **partitional**, **centroid-based** clustering algorithm |
| 2 | It minimizes **WCSS (inertia)** — the total squared distance from each point to its centroid |
| 3 | The centroid is the **mean** of cluster members — it minimizes squared distances |
| 4 | K-Means partitions space into **Voronoi cells** with **linear** boundaries |
| 5 | It alternates between **Assign** (points to nearest centroid) and **Update** (recompute centroids) |
| 6 | Convergence is **guaranteed** but only to a **local optimum** |
| 7 | K-Means assumes **spherical, equally-sized** clusters and uses **Euclidean** distance |

---

## 12. Revision Questions

1. **Objective Function:** Write out the K-Means objective function (WCSS/inertia). Explain each term and why minimizing it produces good clusters.

2. **Centroid Property:** Prove that the mean of a set of points minimizes the sum of squared Euclidean distances to all points in the set.

3. **Voronoi Diagrams:** Explain how K-Means creates Voronoi partitions. Why does this imply that K-Means can only find convex clusters?

4. **Convergence:** Explain why K-Means is guaranteed to converge. Is it guaranteed to find the global optimum? Why or why not?

5. **Coordinate Descent:** K-Means can be viewed as coordinate descent on J(C, μ). Explain the two "coordinates" and how each step optimizes one.

6. **Limitations Preview:** Based on the Voronoi/spherical cluster assumption, describe a dataset geometry where K-Means would completely fail. Sketch the ASCII diagram.

---

<div align="center">

| [← Previous Unit: Challenges](../01-Introduction-to-Unsupervised-Learning/04-challenges.md) | [Up: K-Means Clustering](./README.md) | [Next: Algorithm Steps →](./02-kmeans-algorithm-steps.md) |
|:--------------------------------------------------------------------------------------------:|:--------------------------------------:|:---------------------------------------------------------:|

</div>
