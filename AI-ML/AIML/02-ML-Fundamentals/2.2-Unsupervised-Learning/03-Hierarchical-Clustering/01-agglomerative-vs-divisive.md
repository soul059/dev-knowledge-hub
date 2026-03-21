# 3.1 Agglomerative vs Divisive Hierarchical Clustering

> **Chapter Overview:** Hierarchical clustering builds a tree-like structure (dendrogram) of nested clusters, either by starting from individual points and merging upward (**agglomerative / bottom-up**) or by starting from one big cluster and splitting downward (**divisive / top-down**). This chapter compares both approaches in depth, covering algorithms, computational complexity, and guidance on when to use each.

---

## Table of Contents

1. [What is Hierarchical Clustering?](#1-what-is-hierarchical-clustering)
2. [Agglomerative Clustering (Bottom-Up)](#2-agglomerative-clustering-bottom-up)
3. [Divisive Clustering (Top-Down)](#3-divisive-clustering-top-down)
4. [Side-by-Side Comparison](#4-side-by-side-comparison)
5. [Computational Complexity](#5-computational-complexity)
6. [The Distance/Proximity Matrix](#6-the-distanceproximity-matrix)
7. [Step-by-Step Numerical Example](#7-step-by-step-numerical-example)
8. [When to Use Each Approach](#8-when-to-use-each-approach)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Key Takeaways](#11-key-takeaways)
12. [Revision Questions](#12-revision-questions)

---

## 1. What is Hierarchical Clustering?

Unlike K-Means (which produces a **flat** partition), hierarchical clustering produces a **nested sequence** of partitions — a hierarchy from "all points separate" to "all points together."

### The Key Difference from K-Means

```
    K-MEANS (Flat):                    HIERARCHICAL (Nested):
    
    ┌──────────────┐                         ┌─┐
    │ ● ● ●  ▲ ▲  │                        ┌┴─┴┐
    │ ● ● ●  ▲ ▲  │                       ┌┴┐ ┌┴┐
    │        ■ ■ ■ │                      ┌┴┐│ │┌┴┐
    │        ■ ■ ■ │                      A B C D E F
    └──────────────┘
    ONE answer: K=3                    ALL levels of granularity!
    Must specify K                     Choose K AFTER seeing the tree
```

### Types of Hierarchical Clustering

```
    ┌───────────────────────────────────────────────────────────┐
    │         HIERARCHICAL CLUSTERING                           │
    │                                                           │
    │   ┌─────────────────┐       ┌─────────────────┐          │
    │   │  AGGLOMERATIVE  │       │    DIVISIVE      │          │
    │   │  (Bottom-Up)    │       │    (Top-Down)    │          │
    │   │                 │       │                  │          │
    │   │   Start: n      │       │  Start: 1 big    │          │
    │   │   clusters of   │       │  cluster with    │          │
    │   │   1 point each  │       │  all n points    │          │
    │   │                 │       │                  │          │
    │   │   Action: MERGE │       │  Action: SPLIT   │          │
    │   │   closest pair  │       │  most diverse    │          │
    │   │   at each step  │       │  cluster         │          │
    │   │                 │       │                  │          │
    │   │   End: 1 big    │       │  End: n clusters │          │
    │   │   cluster       │       │  of 1 point each │          │
    │   └─────────────────┘       └─────────────────┘          │
    └───────────────────────────────────────────────────────────┘
```

---

## 2. Agglomerative Clustering (Bottom-Up)

### Algorithm

```
    ┌──────────────────────────────────────────────────────────┐
    │           AGGLOMERATIVE CLUSTERING ALGORITHM              │
    │                                                          │
    │  Input: X = {x₁, ..., xₙ}, distance metric d(·,·)      │
    │                                                          │
    │  1. Initialize: each point is its own cluster            │
    │     C₁={x₁}, C₂={x₂}, ..., Cₙ={xₙ}                   │
    │                                                          │
    │  2. Compute distance matrix D[i,j] for all pairs        │
    │                                                          │
    │  3. REPEAT (n-1 times):                                  │
    │     a. Find the pair of clusters with MINIMUM distance:  │
    │        (Cᵢ, Cⱼ) = argmin D[i,j]                        │
    │                    i≠j                                   │
    │                                                          │
    │     b. MERGE Cᵢ and Cⱼ into a new cluster Cₘ = Cᵢ ∪ Cⱼ│
    │                                                          │
    │     c. Update distance matrix:                           │
    │        Compute D[m,k] for all remaining clusters k       │
    │        (using chosen LINKAGE method)                     │
    │                                                          │
    │     d. Record merge in dendrogram                        │
    │                                                          │
    │  4. Output: Dendrogram (tree of merges)                  │
    └──────────────────────────────────────────────────────────┘
```

### ASCII: Agglomerative Process

```
    Step 0:  {A} {B} {C} {D} {E} {F}     ← 6 clusters
    
    Step 1:  {A,B} {C} {D} {E} {F}       ← merge A,B (closest pair)
    
    Step 2:  {A,B} {C} {D,E} {F}         ← merge D,E
    
    Step 3:  {A,B} {C} {D,E,F}           ← merge {D,E},F
    
    Step 4:  {A,B,C} {D,E,F}             ← merge {A,B},C
    
    Step 5:  {A,B,C,D,E,F}               ← merge everything
    

    Corresponding dendrogram:
    
    Height
    (distance)
      5 │         ┌───────────────┐
        │         │               │
      3 │    ┌────┤          ┌────┤
        │    │    │          │    │
      2 │    │  ┌─┤        ┌─┤   │
        │    │  │ │        │ │   │
      1 │  ┌─┤  │ │        │ │   │
        │  │ │  │ │        │ │   │
      0 │  A B  C │        D E   F
        └────────────────────────────
```

---

## 3. Divisive Clustering (Top-Down)

### Algorithm

```
    ┌──────────────────────────────────────────────────────────┐
    │             DIVISIVE CLUSTERING ALGORITHM                 │
    │                                                          │
    │  Input: X = {x₁, ..., xₙ}                               │
    │                                                          │
    │  1. Initialize: all points in one cluster                │
    │     C₁ = {x₁, ..., xₙ}                                 │
    │                                                          │
    │  2. REPEAT (n-1 times):                                  │
    │     a. Select the cluster to SPLIT                       │
    │        (usually the one with largest diameter            │
    │         or highest within-cluster variance)              │
    │                                                          │
    │     b. SPLIT it into two sub-clusters                    │
    │        (using K-Means with K=2, or bisection)            │
    │                                                          │
    │     c. Record split in dendrogram                        │
    │                                                          │
    │  3. Output: Dendrogram (tree of splits)                  │
    └──────────────────────────────────────────────────────────┘
```

### ASCII: Divisive Process

```
    Step 0:  {A,B,C,D,E,F}               ← 1 cluster
    
    Step 1:  {A,B,C} {D,E,F}             ← split into 2
    
    Step 2:  {A,B,C} {D,E} {F}           ← split {D,E,F}
    
    Step 3:  {A,B} {C} {D,E} {F}         ← split {A,B,C}
    
    Step 4:  {A} {B} {C} {D,E} {F}       ← split {A,B}
    
    Step 5:  {A} {B} {C} {D} {E} {F}     ← split {D,E}

    Same dendrogram structure, but built TOP-DOWN!
```

### DIANA (DIvisive ANAlysis) Algorithm

```
    The classic divisive algorithm:
    
    1. Start with all objects in one cluster
    2. For the selected cluster:
       a. Find the object with highest average dissimilarity 
          to all other objects in the cluster → "splinter group"
       b. Repeatedly reassign objects closer to the splinter 
          group until no more reassignments occur
    3. Repeat step 2 for the most heterogeneous cluster
```

### Bisecting K-Means (Popular Divisive Method)

```
    A simpler and faster divisive approach:
    
    1. Start with all data in one cluster
    2. Pick the cluster with highest SSE (or largest size)
    3. Apply K-Means with K=2 to split it
    4. Repeat until desired number of clusters

    This is available in sklearn as BisectingKMeans (v1.1+)
```

---

## 4. Side-by-Side Comparison

```
┌──────────────────────┬───────────────────────────┬───────────────────────────┐
│ Aspect               │ AGGLOMERATIVE (Bottom-Up) │ DIVISIVE (Top-Down)       │
├──────────────────────┼───────────────────────────┼───────────────────────────┤
│ Direction            │ Merge small → big          │ Split big → small         │
│ Starting point       │ n singleton clusters       │ 1 cluster of all points   │
│ Each step            │ Merge 2 closest clusters   │ Split 1 cluster into 2    │
│ Number of steps      │ n - 1 merges               │ n - 1 splits              │
│ Time Complexity      │ O(n³) or O(n² log n)       │ O(2ⁿ) exact, O(n²) heur. │
│ Space Complexity     │ O(n²) (distance matrix)    │ O(n²) worst case          │
│ Linkage methods      │ Many (single, complete,    │ Usually K-Means bisection │
│                      │ average, Ward)              │                           │
│ Quality at top       │ Less reliable              │ More reliable (global view)│
│ Quality at bottom    │ More reliable (local view) │ Less reliable             │
│ Deterministic?       │ Yes                        │ Depends on split method   │
│ Software support     │ Excellent (scipy, sklearn) │ Limited (BisectingKMeans) │
│ Popularity           │ ★★★★★ Very popular         │ ★★ Less common            │
└──────────────────────┴───────────────────────────┴───────────────────────────┘
```

### Quality Trade-off Intuition

```
    AGGLOMERATIVE:
    ✓ Makes good decisions at the BOTTOM (small clusters are precise)
    ✗ Early merge mistakes propagate UP (cannot un-merge)
    
    DIVISIVE:
    ✓ Makes good decisions at the TOP (global structure captured first)
    ✗ Splitting is computationally harder (many ways to split a cluster)
    
    In practice: Agglomerative is preferred because it's simpler
    and has excellent software support.
```

---

## 5. Computational Complexity

### Agglomerative Clustering

```
    Naive algorithm:
    
    Step                          Cost per step    Steps    Total
    ─────────────────────────     ──────────────   ─────    ─────────
    Compute distance matrix       O(n²)            1        O(n²)
    Find minimum in matrix        O(n²)            n-1      O(n³)
    Update distances              O(n)             n-1      O(n²)
    ──────────────────────────────────────────────────────────────────
    Overall:                                                O(n³)

    With priority queue (heap):
    
    Find minimum:                 O(1) amortized
    Update heap:                  O(n log n)       n-1      O(n² log n)
    ──────────────────────────────────────────────────────────────────
    Overall:                                                O(n² log n)

    Special cases:
    • Single linkage: O(n²) using MST algorithm
    • Complete linkage: O(n² log n) with heap
    • Ward linkage: O(n² log n) with Lance-Williams
```

### Space Complexity

```
    The distance/proximity matrix requires O(n²) storage.
    
    n = 1,000:     ~4 MB  (1M entries × 4 bytes)
    n = 10,000:    ~400 MB
    n = 50,000:    ~10 GB   ← pushing limits!
    n = 100,000:   ~40 GB   ← INFEASIBLE for most machines
    n = 1,000,000: ~4 TB    ← completely impossible
    
    → Hierarchical clustering is LIMITED to ~50,000 points
      on a standard machine (16-32 GB RAM)
```

### Divisive Complexity

```
    Exact divisive (find optimal 2-way split): O(2ⁿ) — NP-hard!
    
    Heuristic (bisecting K-Means):
    Each split: O(n_cluster × d × K_iter)
    Total: O(n × d × K_iter × log K)  ← much more reasonable

    But doesn't produce the same dendrogram quality as agglomerative.
```

---

## 6. The Distance/Proximity Matrix

### Construction

```
    For n data points, compute the n × n distance matrix D:

    D[i,j] = d(xᵢ, xⱼ)     for all i, j

    Common distance metrics:

    Euclidean:    d(x,y) = √(Σⱼ (xⱼ - yⱼ)²)
    
    Manhattan:    d(x,y) = Σⱼ |xⱼ - yⱼ|
    
    Cosine:       d(x,y) = 1 - (x·y)/(‖x‖·‖y‖)
    
    Correlation:  d(x,y) = 1 - corr(x, y)
```

### Example Distance Matrix

```
    Points: A(0,0), B(1,1), C(4,4), D(5,5)

    Euclidean distances:
         A      B      C      D
    A  [ 0.00   1.41   5.66   7.07 ]
    B  [ 1.41   0.00   4.24   5.66 ]
    C  [ 5.66   4.24   0.00   1.41 ]
    D  [ 7.07   5.66   1.41   0.00 ]

    Minimum non-diagonal: d(A,B) = d(C,D) = 1.41
    → First merge: {A,B} or {C,D}
```

---

## 7. Step-by-Step Numerical Example

### Agglomerative Clustering (Single Linkage)

```
    Points: A(1,1), B(2,2), C(5,4), D(6,5), E(6,6)

    Step 0: Distance Matrix
              A      B      C      D      E
    A    [  0.00   1.41   5.00   5.83   6.40 ]
    B    [  1.41   0.00   3.61   5.00   5.66 ]
    C    [  5.00   3.61   0.00   1.41   2.24 ]
    D    [  5.83   5.00   1.41   0.00   1.00 ]
    E    [  6.40   5.66   2.24   1.00   0.00 ]

    Minimum: d(D,E) = 1.00

    Step 1: Merge D and E → {D,E}
    
    Update distances (single linkage = min):
    d({D,E}, A) = min(d(D,A), d(E,A)) = min(5.83, 6.40) = 5.83
    d({D,E}, B) = min(d(D,B), d(E,B)) = min(5.00, 5.66) = 5.00
    d({D,E}, C) = min(d(D,C), d(E,C)) = min(1.41, 2.24) = 1.41

    New Distance Matrix:
              A      B      C     {D,E}
    A    [  0.00   1.41   5.00   5.83 ]
    B    [  1.41   0.00   3.61   5.00 ]
    C    [  5.00   3.61   0.00   1.41 ]
    {D,E}[ 5.83   5.00   1.41   0.00 ]

    Minimum: d(A,B) = d(C,{D,E}) = 1.41

    Step 2: Merge A and B → {A,B} (tie-breaking: pick first)
    
    d({A,B}, C) = min(5.00, 3.61) = 3.61
    d({A,B}, {D,E}) = min(5.83, 5.00) = 5.00

    New Distance Matrix:
             {A,B}    C     {D,E}
    {A,B} [  0.00   3.61   5.00 ]
    C     [  3.61   0.00   1.41 ]
    {D,E} [  5.00   1.41   0.00 ]

    Minimum: d(C, {D,E}) = 1.41

    Step 3: Merge C and {D,E} → {C,D,E}
    
    d({A,B}, {C,D,E}) = min(3.61, 5.00) = 3.61

    Step 4: Merge {A,B} and {C,D,E} → {A,B,C,D,E}

    DENDROGRAM:
    
    Height
    3.61│              ┌───────────┐
        │              │           │
    1.41│         ┌────┤     ┌─────┤
        │         │    │     │     │
    1.00│         │    │   ┌─┤     │
        │         │    │   │ │     │
    0   │         A    B   D E     C
```

---

## 8. When to Use Each Approach

```
    USE AGGLOMERATIVE when:
    ┌──────────────────────────────────────────────────────────┐
    │ ✓ n < 50,000 (memory fits O(n²) distance matrix)       │
    │ ✓ You want a full dendrogram for exploration            │
    │ ✓ You want to choose K AFTER seeing the hierarchy       │
    │ ✓ Cluster shapes may be non-spherical                   │
    │ ✓ You want deterministic results                        │
    │ ✓ Different linkage methods give different perspectives │
    └──────────────────────────────────────────────────────────┘

    USE DIVISIVE (Bisecting K-Means) when:
    ┌──────────────────────────────────────────────────────────┐
    │ ✓ n is large (scales better than agglomerative)         │
    │ ✓ Global structure matters more than local detail       │
    │ ✓ You're comfortable with K-Means-style splits          │
    │ ✓ You want a hierarchical decomposition of K-Means      │
    └──────────────────────────────────────────────────────────┘

    IN MOST CASES → Use AGGLOMERATIVE
    (it's more popular, better supported, and more flexible)
```

---

## 9. Python Implementation

### Agglomerative Clustering with scipy

```python
import numpy as np
from scipy.cluster.hierarchy import linkage, fcluster, dendrogram
from scipy.spatial.distance import pdist
import matplotlib.pyplot as plt

# Generate sample data
np.random.seed(42)
cluster1 = np.random.randn(20, 2) + [0, 0]
cluster2 = np.random.randn(20, 2) + [5, 5]
cluster3 = np.random.randn(20, 2) + [10, 0]
X = np.vstack([cluster1, cluster2, cluster3])

# Compute linkage matrix
Z = linkage(X, method='ward', metric='euclidean')

# Z is an (n-1) × 4 matrix:
# [cluster_i, cluster_j, distance, size_of_new_cluster]
print("First 5 merges:")
print(f"{'Cluster i':>10} {'Cluster j':>10} {'Distance':>10} {'Size':>6}")
for row in Z[:5]:
    print(f"{int(row[0]):>10} {int(row[1]):>10} {row[2]:>10.4f} {int(row[3]):>6}")

# Plot dendrogram
plt.figure(figsize=(12, 6))
dendrogram(Z, truncate_mode='lastp', p=15, leaf_rotation=90)
plt.title("Agglomerative Clustering Dendrogram (Ward Linkage)")
plt.xlabel("Cluster Size")
plt.ylabel("Distance (Ward)")
plt.tight_layout()
plt.savefig("agglomerative_dendrogram.png", dpi=150)
plt.show()

# Cut at K=3 clusters
labels = fcluster(Z, t=3, criterion='maxclust')
print(f"\nCluster sizes: {np.bincount(labels)[1:]}")  # 1-indexed
```

### Agglomerative Clustering with sklearn

```python
from sklearn.cluster import AgglomerativeClustering
from sklearn.datasets import make_blobs
from sklearn.metrics import silhouette_score
import numpy as np

X, y_true = make_blobs(n_samples=200, centers=4, 
                         cluster_std=1.0, random_state=42)

# Fit agglomerative clustering
agg = AgglomerativeClustering(
    n_clusters=4,           # Number of clusters
    metric='euclidean',     # Distance metric
    linkage='ward',         # Linkage criterion
)
labels = agg.fit_predict(X)

print(f"Number of clusters: {agg.n_clusters_}")
print(f"Cluster sizes: {np.bincount(labels)}")
print(f"Silhouette score: {silhouette_score(X, labels):.4f}")
```

### Bisecting K-Means (Divisive, sklearn 1.1+)

```python
from sklearn.cluster import BisectingKMeans
from sklearn.datasets import make_blobs
import numpy as np

X, y_true = make_blobs(n_samples=500, centers=5, 
                         cluster_std=1.5, random_state=42)

bkm = BisectingKMeans(n_clusters=5, random_state=42, n_init=3)
labels = bkm.fit_predict(X)

print(f"Inertia: {bkm.inertia_:.2f}")
print(f"Cluster sizes: {np.bincount(labels)}")
```

---

## 10. Summary Table

```
┌────────────────────────┬────────────────────────┬────────────────────────┐
│ Property               │ Agglomerative          │ Divisive               │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ Direction              │ Bottom-up (merge)      │ Top-down (split)       │
│ Time Complexity        │ O(n³) or O(n² log n)   │ O(2ⁿ) exact / O(n²) h.│
│ Space Complexity       │ O(n²)                  │ O(n) to O(n²)          │
│ Max practical n        │ ~50,000                │ ~1,000,000 (bisecting) │
│ Dendrogram             │ Full, high quality     │ Partial, lower quality │
│ Linkage methods        │ Many (single, Ward...) │ Usually bisecting      │
│ Software               │ scipy, sklearn         │ sklearn (BisectingKM)  │
│ Deterministic          │ Yes                    │ Not always             │
│ Best for               │ Detailed hierarchy     │ Scalable hierarchy     │
│ Popularity             │ ★★★★★                  │ ★★                     │
└────────────────────────┴────────────────────────┴────────────────────────┘
```

---

## 11. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Hierarchical clustering produces a **nested hierarchy** of clusters, not a flat partition |
| 2 | **Agglomerative** (bottom-up) starts with singletons and merges; **divisive** (top-down) starts with all and splits |
| 3 | Agglomerative is **much more popular** due to simplicity and software support |
| 4 | Time complexity is **O(n³)** or **O(n² log n)** — limits it to ~50K points |
| 5 | The **distance matrix** requires O(n²) space — the main bottleneck |
| 6 | The **linkage method** (next chapter) determines HOW inter-cluster distance is measured |
| 7 | **Bisecting K-Means** is a practical divisive alternative that scales better |

---

## 12. Revision Questions

1. **Concept:** Explain the fundamental difference between agglomerative and divisive hierarchical clustering. Why is agglomerative more commonly used?

2. **Algorithm:** Write out the agglomerative clustering algorithm step by step. What data structure does it maintain, and how is it updated?

3. **Complexity:** Derive the O(n³) time complexity for naive agglomerative clustering. How can it be improved to O(n² log n)?

4. **Space:** If each distance value takes 8 bytes (double), how much memory does the distance matrix need for n = 10,000? n = 100,000?

5. **Numerical Example:** Given points A(0,0), B(1,0), C(3,0), D(4,0) and single linkage, trace through the complete agglomerative algorithm and draw the dendrogram.

6. **Divisive:** Describe the Bisecting K-Means algorithm. What advantage does it have over agglomerative clustering for large datasets?

---

<div align="center">

| [← Previous Unit: Mini-Batch K-Means](../02-K-Means-Clustering/06-mini-batch-kmeans.md) | [Up: Hierarchical Clustering](./README.md) | [Next: Linkage Methods →](./02-linkage-methods.md) |
|:----------------------------------------------------------------------------------------:|:------------------------------------------:|:--------------------------------------------------:|

</div>
