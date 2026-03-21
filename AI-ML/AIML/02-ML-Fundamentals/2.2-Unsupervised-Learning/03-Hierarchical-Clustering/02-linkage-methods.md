# 3.2 Linkage Methods in Hierarchical Clustering

> **Chapter Overview:** The **linkage criterion** defines how the distance between two clusters is measured — this single choice fundamentally shapes the resulting dendrogram and cluster structure. This chapter covers the five major linkage methods (single, complete, average, Ward, and centroid), their mathematical formulas, their effect on cluster shape, and the chaining phenomenon, with a comprehensive comparison table.

---

## Table of Contents

1. [What is Linkage?](#1-what-is-linkage)
2. [Single Linkage (Minimum)](#2-single-linkage-minimum)
3. [Complete Linkage (Maximum)](#3-complete-linkage-maximum)
4. [Average Linkage (UPGMA)](#4-average-linkage-upgma)
5. [Ward's Linkage (Minimum Variance)](#5-wards-linkage-minimum-variance)
6. [Centroid Linkage (UPGMC)](#6-centroid-linkage-upgmc)
7. [The Chaining Effect](#7-the-chaining-effect)
8. [Lance-Williams Recurrence Formula](#8-lance-williams-recurrence-formula)
9. [Effect on Cluster Shape](#9-effect-on-cluster-shape)
10. [Comparison Table](#10-comparison-table)
11. [Python Comparison](#11-python-comparison)
12. [Key Takeaways](#12-key-takeaways)
13. [Revision Questions](#13-revision-questions)

---

## 1. What is Linkage?

When merging two clusters **A** and **B**, we need a way to define the **distance between clusters** (not just between individual points). The **linkage criterion** specifies this.

```
    Cluster A:  {a₁, a₂, a₃}          Cluster B: {b₁, b₂, b₃, b₄}
    
       a₁ ·                                    · b₁
       a₂ ·    ←── d(A, B) = ??? ───→    · b₂
       a₃ ·                                · b₃
                                             · b₄

    Different linkage methods give DIFFERENT answers for d(A, B):
    
    Single:    d(A,B) = min distance between any pair (aᵢ, bⱼ)
    Complete:  d(A,B) = max distance between any pair (aᵢ, bⱼ)
    Average:   d(A,B) = average of all pairwise distances
    Ward:      d(A,B) = increase in total WCSS upon merging
    Centroid:  d(A,B) = distance between cluster centroids
```

---

## 2. Single Linkage (Minimum)

### Definition

```
    d_single(A, B) = min    d(aᵢ, bⱼ)
                   aᵢ∈A, bⱼ∈B

    The distance between the two CLOSEST points in A and B.
    Also called: nearest-neighbor linkage.
```

### ASCII Illustration

```
    Cluster A          Cluster B
    
    a₁ ·              · b₁
    a₂ ·    ←──2──→   · b₂     d_single = 2 (closest pair)
    a₃ ·              · b₃
                      · b₄
    
    a₃ to b₂ = 2 (minimum)
    a₁ to b₁ = 7 (other pairs are larger)
```

### Properties

```
    ✓ Can detect NON-CONVEX clusters (elongated, irregular shapes)
    ✓ Computationally efficient: O(n²) using MST algorithm
    ✓ Single-link = minimum spanning tree cut
    
    ✗ CHAINING EFFECT: tends to create long, stringy clusters
    ✗ Sensitive to noise points between clusters
    ✗ Often produces unbalanced dendrograms
```

### Mathematical Connection to MST

```
    Theorem: Single linkage clustering is equivalent to 
    cutting the Minimum Spanning Tree (MST) of the data.

    MST: The tree connecting all points with minimum total edge weight.
    
    To get K clusters: remove the (K-1) longest edges from the MST.
    
    Points:  A ──2── B ──3── C ──8── D ──1── E
    
    MST = A-B-C-D-E
    K=2: Remove longest edge (C-D, weight=8)
    → Clusters: {A,B,C} and {D,E}
```

---

## 3. Complete Linkage (Maximum)

### Definition

```
    d_complete(A, B) = max    d(aᵢ, bⱼ)
                     aᵢ∈A, bⱼ∈B

    The distance between the two FARTHEST points in A and B.
    Also called: farthest-neighbor linkage, diameter linkage.
```

### ASCII Illustration

```
    Cluster A          Cluster B
    
    a₁ ·                          · b₁
    a₂ ·                      · b₂
    a₃ ·    ←────────10────────→  · b₃    d_complete = 10
                                  · b₄

    a₁ to b₄ = 10 (maximum pairwise distance)
```

### Properties

```
    ✓ Produces COMPACT, spherical clusters
    ✓ Avoids chaining effect
    ✓ Cluster diameter is bounded at each merge level
    
    ✗ Sensitive to outliers (single far-away point inflates distance)
    ✗ Tends to break large clusters
    ✗ Cannot detect elongated clusters
```

### Cluster Diameter Property

```
    After merging at distance h:
    
    Every pair of points in the merged cluster has distance ≤ h.
    (The "diameter" of the cluster is at most h.)
    
    This guarantees compact clusters, unlike single linkage.
```

---

## 4. Average Linkage (UPGMA)

### Definition

```
    d_average(A, B) = (1 / |A|·|B|) ·  Σ      Σ    d(aᵢ, bⱼ)
                                      aᵢ∈A   bⱼ∈B

    The AVERAGE of all pairwise distances between A and B.
    Also called: UPGMA (Unweighted Pair Group Method with Arithmetic Mean)
```

### ASCII Illustration

```
    Cluster A (3 pts)      Cluster B (2 pts)
    
    a₁ ·──3──┐      ┌──5──· b₁
    a₂ ·──4──┤──avg──┤──6──· b₂
    a₃ ·──7──┘      └──8──

    d_average = (3+5 + 4+6 + 7+8) / (3×2) = 33/6 = 5.5
```

### Properties

```
    ✓ Compromise between single and complete linkage
    ✓ Less sensitive to outliers than complete linkage
    ✓ Less prone to chaining than single linkage
    ✓ Robust and widely used
    
    ✗ Can be biased by cluster sizes
    ✗ Not as compact as complete linkage
    ✗ Not as flexible as single linkage for non-convex shapes
```

### Weighted vs Unweighted Average

```
    UPGMA (Unweighted): Each point counts equally
    d(A∪B, C) = (|A|·d(A,C) + |B|·d(B,C)) / (|A| + |B|)
    
    WPGMA (Weighted): Each cluster counts equally regardless of size
    d(A∪B, C) = (d(A,C) + d(B,C)) / 2
    
    UPGMA is more common and is the default "average" in scipy.
```

---

## 5. Ward's Linkage (Minimum Variance)

### Definition

```
    d_ward(A, B) = ΔWCSS = WCSS(A ∪ B) - WCSS(A) - WCSS(B)

    The INCREASE in total within-cluster sum of squares (WCSS)
    caused by merging A and B.

    Equivalently:

    d_ward(A, B) = (|A| · |B|) / (|A| + |B|) · ‖μ_A - μ_B‖²

    Where μ_A and μ_B are the centroids of A and B.
```

### Mathematical Derivation

```
    WCSS(C) = Σᵢ∈C ‖xᵢ - μ_C‖²

    For the merged cluster A ∪ B:
    
    WCSS(A ∪ B) = Σᵢ∈A∪B ‖xᵢ - μ_{A∪B}‖²
    
    The increase is:
    
    ΔWCSS = WCSS(A ∪ B) - WCSS(A) - WCSS(B)
    
           = (n_A · n_B) / (n_A + n_B) · ‖μ_A - μ_B‖²
    
    This is the "energy" cost of merging.
    Ward's method MINIMIZES this cost at each step.
```

### Properties

```
    ✓ Tends to produce EQUAL-SIZED, compact, spherical clusters
    ✓ Similar to K-Means objective (both minimize WCSS)
    ✓ Often produces the most "natural" looking dendrograms
    ✓ Most popular linkage method for general use
    
    ✗ Only works with Euclidean distance
    ✗ Biased toward equal-sized clusters
    ✗ Cannot handle non-Euclidean distance matrices
```

### Connection to K-Means

```
    Ward's linkage is the hierarchical analogue of K-Means:
    
    K-Means:  Minimize total WCSS by assigning points to centroids
    Ward:     Minimize total WCSS by choosing which clusters to merge
    
    Both optimize the SAME objective, but:
    - K-Means: flat, non-deterministic, requires K
    - Ward: hierarchical, deterministic, produces dendrogram
```

---

## 6. Centroid Linkage (UPGMC)

### Definition

```
    d_centroid(A, B) = ‖μ_A - μ_B‖²

    The squared Euclidean distance between cluster centroids.
    
    Where:
    μ_A = (1/|A|) Σᵢ∈A xᵢ
    μ_B = (1/|B|) Σⱼ∈B xⱼ
```

### Properties

```
    ✓ Intuitive — distance between "centers" of clusters
    ✓ Computationally efficient (just compare centroids)
    
    ✗ Can produce INVERSIONS in the dendrogram
      (merge distance can decrease as you go up the tree)
    ✗ Less popular due to inversion problem
    ✗ Only works with Euclidean distance
```

### The Inversion Problem

```
    Inversion: A merge at a HIGHER level in the dendrogram
    has a SMALLER distance than a merge at a lower level.

    Height
      │     ┌──────┐
    5 │     │      │          ← merge at height 5
      │  ┌──┤      │
    6 │  │  │      │          ← merge at height 6?! (INVERSION)
      │  │  │   ┌──┤
    3 │  │  │   │  │          ← merge at height 3
      │  A  B   C  D

    This makes the dendrogram hard to interpret!
    → Avoid centroid linkage unless you have a specific reason.
```

---

## 7. The Chaining Effect

### What is Chaining?

**Chaining** occurs primarily with **single linkage** when a sequence of points forms a "bridge" between two clusters, causing them to merge prematurely.

```
    TRUE CLUSTERS:                    SINGLE LINKAGE RESULT:
    
    ● ● ●     ○ ○ ○ ○ ○              ● ● ● · · · ○ ○ ○ ○ ○
    ● ● ●  ·  ○ ○ ○ ○ ○              ● ● ●       ○ ○ ○ ○ ○
    ● ● ●  ·  ○ ○ ○ ○ ○              ● ● ●       ○ ○ ○ ○ ○
    ● ● ●     ○ ○ ○ ○ ○              ● ● ●       ○ ○ ○ ○ ○
         ↑
    Noise points create a "chain"     Single linkage merges them
    between clusters                  into ONE cluster!
```

### Why It Happens

```
    Single linkage only looks at the CLOSEST pair.
    
    If noise points form a path:
    ● ──1.5── · ──1.5── · ──1.5── ○
    
    Each gap is only 1.5, so single linkage connects them
    even if the clusters are far apart (distance 6).
    
    The cluster grows point by point along the chain.
```

### Which Linkage Methods Suffer from Chaining?

```
    ┌──────────────────┬───────────────────┐
    │ Linkage          │ Chaining Risk     │
    ├──────────────────┼───────────────────┤
    │ Single           │ ★★★★★ (very high) │
    │ Average          │ ★★ (moderate)     │
    │ Centroid         │ ★★ (moderate)     │
    │ Complete         │ ★ (very low)      │
    │ Ward             │ ★ (very low)      │
    └──────────────────┴───────────────────┘
```

---

## 8. Lance-Williams Recurrence Formula

### The Universal Update Formula

When clusters A and B are merged into A∪B, the distance to any other cluster C can be computed using:

```
    d(A∪B, C) = αₐ · d(A,C) + α_b · d(B,C) + β · d(A,B) + γ · |d(A,C) - d(B,C)|

    Where αₐ, α_b, β, γ are constants that depend on the linkage method:
```

### Parameters for Each Linkage

```
┌──────────────┬────────────────────┬──────────────┬──────────┬────────┐
│ Linkage      │ αₐ                 │ α_b          │ β        │ γ      │
├──────────────┼────────────────────┼──────────────┼──────────┼────────┤
│ Single       │ 1/2                │ 1/2          │ 0        │ -1/2   │
│ Complete     │ 1/2                │ 1/2          │ 0        │ +1/2   │
│ Average      │ nₐ/(nₐ+n_b)       │ n_b/(nₐ+n_b)│ 0        │ 0      │
│ Ward         │ (nₐ+nc)/(n_tot)   │ (n_b+nc)/    │ -nc/     │ 0      │
│              │                    │ (n_tot)      │ (n_tot)  │        │
│ Centroid     │ nₐ/(nₐ+n_b)       │ n_b/(nₐ+n_b)│-nₐn_b/  │ 0      │
│              │                    │              │(nₐ+n_b)²│        │
└──────────────┴────────────────────┴──────────────┴──────────┴────────┘

Where n_tot = nₐ + n_b + nc
```

### Why This Matters

```
    The Lance-Williams formula allows EFFICIENT updating of the 
    distance matrix without recomputing all pairwise distances.
    
    Without it: Each merge requires O(n²) distance recomputation
    With it:    Each merge requires O(n) updates → total O(n² log n)
```

---

## 9. Effect on Cluster Shape

### Visual Comparison

```
    SINGLE LINKAGE:                    COMPLETE LINKAGE:
    (elongated, stringy)               (compact, spherical)
    
    ·····                              ○○○○    ●●●●
     ····                              ○○○○    ●●●●
      ····                             ○○○○    ●●●●
       ····                            
        ····                           
    One long cluster                   Two compact clusters
    
    
    AVERAGE LINKAGE:                   WARD LINKAGE:
    (moderate)                         (compact, equal-sized)
    
    ○○○○○     ●●●●                    ○○○○○    ●●●●●
    ○○○○      ●●●●                    ○○○○○    ●●●●●
    ○○        ●●                      ○○○○○    ●●●●●
    
    Balanced clusters                  Most balanced clusters
```

### How Linkage Shapes Clusters

```
    ┌──────────────────────────────────────────────────────────────┐
    │                                                              │
    │  Single:   Tends to produce ELONGATED clusters              │
    │            because it only needs one close pair to merge    │
    │            → good for non-convex shapes                    │
    │            → bad for noisy data (chaining)                 │
    │                                                              │
    │  Complete: Tends to produce COMPACT clusters                │
    │            because ALL points must be close                 │
    │            → good for spherical clusters                   │
    │            → bad for elongated clusters                    │
    │                                                              │
    │  Average:  COMPROMISE between single and complete           │
    │            → robust general-purpose choice                 │
    │                                                              │
    │  Ward:     Tends to produce EQUAL-SIZED, spherical clusters │
    │            → best when clusters are actually spherical      │
    │            → most similar to K-Means                       │
    │                                                              │
    └──────────────────────────────────────────────────────────────┘
```

---

## 10. Comparison Table

```
┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│              │ SINGLE       │ COMPLETE     │ AVERAGE      │ WARD         │ CENTROID     │
├──────────────┼──────────────┼──────────────┼──────────────┼──────────────┼──────────────┤
│ Formula      │ min d(a,b)   │ max d(a,b)   │ avg d(a,b)   │ ΔWCSS        │ ‖μ_A-μ_B‖²  │
│ Cluster shape│ Elongated    │ Compact      │ Moderate     │ Spherical    │ Spherical    │
│ Chaining     │ Very prone   │ Resistant    │ Moderate     │ Resistant    │ Moderate     │
│ Outlier sens.│ Low          │ High         │ Moderate     │ Moderate     │ Moderate     │
│ Distance req.│ Any metric   │ Any metric   │ Any metric   │ Euclidean    │ Euclidean    │
│ Inversions   │ No           │ No           │ No           │ No           │ YES          │
│ Cluster sizes│ Unbalanced   │ Moderate     │ Moderate     │ Balanced     │ Varies       │
│ Complexity   │ O(n²) MST    │ O(n² log n)  │ O(n² log n)  │ O(n² log n)  │ O(n² log n)  │
│ Best for     │ Non-convex   │ Compact      │ General      │ General      │ Simple       │
│              │ clusters     │ clusters     │ purpose      │ purpose      │ scenarios    │
│ Popularity   │ ★★★          │ ★★★          │ ★★★★         │ ★★★★★        │ ★★           │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘

RECOMMENDATION: Start with Ward (best for most cases).
                Use single only for non-convex clusters.
                Use average as a robust alternative.
```

---

## 11. Python Comparison

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import linkage, dendrogram, fcluster
from sklearn.datasets import make_moons, make_blobs

# Dataset 1: Spherical clusters (Ward/Complete should win)
X_blobs, y_blobs = make_blobs(n_samples=150, centers=3, 
                                cluster_std=1.0, random_state=42)

# Dataset 2: Non-convex clusters (Single should win)
X_moons, y_moons = make_moons(n_samples=200, noise=0.05, random_state=42)

linkage_methods = ['single', 'complete', 'average', 'ward']

fig, axes = plt.subplots(2, 4, figsize=(20, 10))

for col, method in enumerate(linkage_methods):
    # Row 1: Spherical clusters
    Z = linkage(X_blobs, method=method)
    labels = fcluster(Z, t=3, criterion='maxclust')
    axes[0, col].scatter(X_blobs[:, 0], X_blobs[:, 1], c=labels, 
                         cmap='viridis', s=30, alpha=0.7)
    axes[0, col].set_title(f'{method.capitalize()} Linkage\n(Spherical Data)')
    
    # Row 2: Non-convex clusters
    if method == 'ward':
        Z2 = linkage(X_moons, method='ward')
    else:
        Z2 = linkage(X_moons, method=method)
    labels2 = fcluster(Z2, t=2, criterion='maxclust')
    axes[1, col].scatter(X_moons[:, 0], X_moons[:, 1], c=labels2, 
                         cmap='viridis', s=30, alpha=0.7)
    axes[1, col].set_title(f'{method.capitalize()} Linkage\n(Two Moons)')

plt.suptitle("Effect of Linkage Method on Cluster Shape", fontsize=16)
plt.tight_layout()
plt.savefig("linkage_comparison.png", dpi=150, bbox_inches='tight')
plt.show()
```

### Dendrogram Comparison

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import linkage, dendrogram

np.random.seed(42)
X = np.vstack([
    np.random.randn(10, 2) + [0, 0],
    np.random.randn(10, 2) + [5, 5],
    np.random.randn(10, 2) + [10, 0]
])

fig, axes = plt.subplots(1, 4, figsize=(24, 6))
for ax, method in zip(axes, ['single', 'complete', 'average', 'ward']):
    Z = linkage(X, method=method)
    dendrogram(Z, ax=ax, leaf_rotation=90, leaf_font_size=8)
    ax.set_title(f'{method.capitalize()} Linkage', fontsize=14)
    ax.set_ylabel('Distance')

plt.suptitle("Dendrograms with Different Linkage Methods", fontsize=16)
plt.tight_layout()
plt.savefig("linkage_dendrograms.png", dpi=150, bbox_inches='tight')
plt.show()
```

---

## 12. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Linkage** defines how distance between clusters is measured — it's the most important choice |
| 2 | **Single** linkage (min distance) can find non-convex clusters but suffers from **chaining** |
| 3 | **Complete** linkage (max distance) produces compact clusters but is outlier-sensitive |
| 4 | **Average** linkage is a robust compromise between single and complete |
| 5 | **Ward's** linkage minimizes WCSS increase — most popular, produces balanced clusters |
| 6 | **Centroid** linkage can produce **inversions** in the dendrogram — generally avoided |
| 7 | The **Lance-Williams formula** enables efficient O(n) updates per merge step |

---

## 13. Revision Questions

1. **Definitions:** Write the mathematical formula for single, complete, average, and Ward's linkage. Explain what each measures in one sentence.

2. **Chaining Effect:** Explain the chaining effect with an example. Which linkage methods are most/least susceptible?

3. **Ward's Linkage:** Prove that Ward's distance between clusters A and B equals (nₐ·n_b)/(nₐ+n_b)·‖μ_A−μ_B‖². Why does this make Ward similar to K-Means?

4. **Shape Effect:** Given a dataset with two interleaving crescent shapes (two moons), which linkage method would you use? Which would fail? Explain why.

5. **Inversions:** What is a dendrogram inversion? Which linkage method can produce inversions, and why?

6. **Practical Choice:** You have a dataset of customer features and want to build a dendrogram. You don't know the cluster shapes. Which linkage method would you start with, and why?

---

<div align="center">

| [← Agglomerative vs Divisive](./01-agglomerative-vs-divisive.md) | [Up: Hierarchical Clustering](./README.md) | [Next: Dendrograms →](./03-dendrograms.md) |
|:----------------------------------------------------------------:|:------------------------------------------:|:------------------------------------------:|

</div>
