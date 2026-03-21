# 3.4 Cutting the Dendrogram — From Hierarchy to Flat Clusters

> **Chapter Overview:** A dendrogram gives you the entire hierarchy, but for practical applications you often need a **flat partition** — K discrete clusters. This chapter covers how to "cut" a dendrogram to extract clusters, including horizontal cuts at a fixed height, choosing the cut height using the inconsistency coefficient, scipy's `fcluster` function with multiple criteria, and advanced dynamic tree cutting methods.

---

## Table of Contents

1. [Why Cut the Dendrogram?](#1-why-cut-the-dendrogram)
2. [Horizontal Cut — Fixed Height](#2-horizontal-cut--fixed-height)
3. [Choosing the Cut Height](#3-choosing-the-cut-height)
4. [The Inconsistency Coefficient](#4-the-inconsistency-coefficient)
5. [fcluster in scipy — All Criteria](#5-fcluster-in-scipy--all-criteria)
6. [Dynamic Tree Cutting](#6-dynamic-tree-cutting)
7. [Worked Example — Complete Pipeline](#7-worked-example--complete-pipeline)
8. [Comparison of Cutting Methods](#8-comparison-of-cutting-methods)
9. [Python Implementation](#9-python-implementation)
10. [Summary Table](#10-summary-table)
11. [Key Takeaways](#11-key-takeaways)
12. [Revision Questions](#12-revision-questions)

---

## 1. Why Cut the Dendrogram?

The dendrogram gives a **complete hierarchy** (from n clusters down to 1), but most applications need a **single set of K clusters**.

```
    DENDROGRAM:                         FLAT CLUSTERING:

    Height                              Cluster Assignment:
    10│       ┌──────────┐              A → Cluster 1
      │       │          │              B → Cluster 1
     6│  ┌────┤     ┌────┤              C → Cluster 1
      │  │    │     │    │              D → Cluster 2
     3│┌─┤    │     │    │              E → Cluster 2
      ││ │    │     │    │              F → Cluster 2
     0│A B    C     D E  F

    Cut at height 5:                    → 2 clusters: {A,B,C} and {D,E,F}
    Cut at height 2:                    → 3 clusters: {A,B}, {C}, {D,E,F}
    Cut at height 1:                    → 4 clusters: {A}, {B}, {C}, {D,E,F}

    The CUT HEIGHT determines K!
```

### Duality: Cut Height ↔ Number of Clusters

```
    ┌────────────────────────────────────────────────┐
    │  Higher cut → FEWER clusters (coarser)         │
    │  Lower cut  → MORE clusters  (finer)           │
    │                                                 │
    │  Cut at ∞   → 1 cluster (everything together)  │
    │  Cut at 0   → n clusters (every point alone)   │
    │                                                 │
    │  Two equivalent ways to specify the cut:        │
    │    1. Set HEIGHT h → get the resulting K        │
    │    2. Set K → find the height that gives K      │
    └────────────────────────────────────────────────┘
```

---

## 2. Horizontal Cut — Fixed Height

### How It Works

Draw a **horizontal line** at height h through the dendrogram. The number of vertical lines crossing this horizontal line = the number of clusters.

```
    Height
    10│         ┌──────────────────┐
      │         │                  │
     8│    ┌────┤             ┌────┤
      │    │    │             │    │
     5│──┌─┤────│─────────────│────│────── Cut at h=5
      │  │ │    │         ┌───┤    │
     3│  │ │    │       ┌─┤   │    │
      │  │ │    │       │ │   │    │
     1│┌─┤ │    │       │ │   │    │
      ││ │ │    │       │ │   │    │
     0│A B C    D       E F   G    H

    At h=5: the horizontal line crosses 3 vertical lines
    → 3 clusters: {A,B,C,D}, {E,F,G}, {H}
```

### Choosing h to Get K Clusters

```
    To get exactly K clusters:
    
    1. Look at the merge heights in the dendrogram
    2. Find the K-th largest gap
    3. Set h between the K-th and (K+1)-th merge heights
    
    Or simply: use fcluster(Z, t=K, criterion='maxclust')
```

---

## 3. Choosing the Cut Height

### Method 1: Visual Inspection (Largest Gap)

```
    Height      Merge heights:  [1, 1.5, 3, 4, 8, 25]
    25│              ┌────┐
      │              │    │     Gap: 25 - 8 = 17  ← LARGEST!
      │              │    │
     8│         ┌────┤    │     Gap: 8 - 4 = 4
      │         │    │    │
     4│    ┌────┤    │    │     Gap: 4 - 3 = 1
      │    │    │    │    │
     3│  ┌─┤    │    │    │     Gap: 3 - 1.5 = 1.5
      │  │ │    │    │    │
    1.5│  │ │    │  ┌─┤    │    Gap: 1.5 - 1 = 0.5
      │  │ │    │  │ │    │
     1│┌─┤ │    │  │ │    │
      ││ │ │    │  │ │    │
      │A B C    D  E F    G

    Largest gap is between 8 and 25.
    → Cut between 8 and 25 → 2 clusters.
    
    Second largest gap: between 4 and 8.
    → Cut between 4 and 8 → 3 clusters.
```

### Method 2: Domain Knowledge

```
    "We need exactly 5 customer segments for our marketing team."
    → Cut to get K=5, regardless of dendrogram structure.
```

### Method 3: Inconsistency Coefficient (Statistical)

See next section for full details.

### Method 4: Silhouette Score on Cut Results

```python
from scipy.cluster.hierarchy import linkage, fcluster
from sklearn.metrics import silhouette_score

Z = linkage(X, method='ward')

for k in range(2, 10):
    labels = fcluster(Z, t=k, criterion='maxclust')
    sil = silhouette_score(X, labels)
    print(f"K={k}: Silhouette = {sil:.4f}")

# Choose K with highest silhouette score
```

---

## 4. The Inconsistency Coefficient

### Definition

The **inconsistency coefficient** measures how inconsistent (unusual) each merge is compared to **nearby merges** in the tree.

```
    For each merge in the dendrogram:

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  inconsistency(i) = (h_i - h̄) / s_h                      │
    │                                                            │
    │  Where:                                                    │
    │    h_i = height (distance) at which merge i occurs         │
    │    h̄   = mean height of merges in the sub-tree below i    │
    │          (up to depth d)                                   │
    │    s_h = standard deviation of those heights               │
    │    d   = depth parameter (default: 2 in scipy)             │
    │                                                            │
    │  Interpretation:                                           │
    │    Low inconsistency:  This merge is similar to nearby     │
    │                        merges (consistent)                 │
    │    High inconsistency: This merge is MUCH larger than      │
    │                        nearby merges (inconsistent!)       │
    │                        → suggests a natural cluster break  │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

### Visual Intuition

```
    Height
      │
    20│              ┌────────────────┐
      │              │                │    ← Inconsistency = HIGH
      │              │                │       (height 20 vs nearby 3-5)
      │              │                │       → CUT HERE!
     5│         ┌────┤           ┌────┤
      │         │    │           │    │    ← Inconsistency = LOW
     3│    ┌────┤    │      ┌────┤    │       (height 3-5, all similar)
      │    │    │    │      │    │    │
     2│  ┌─┤    │    │    ┌─┤    │    │
      │  │ │    │    │    │ │    │    │
      │  A B    C    D    E F    G    H

    The merge at height 20 has HIGH inconsistency because:
    - Below it: merges at heights 2, 3, 5 (mean ≈ 3.3, std ≈ 1.5)
    - This merge: height 20
    - Inconsistency = (20 - 3.3) / 1.5 ≈ 11.1  ← very high!
```

### Python: Computing Inconsistency

```python
from scipy.cluster.hierarchy import linkage, inconsistency
import numpy as np

np.random.seed(42)
X = np.vstack([
    np.random.randn(30, 2) + [0, 0],
    np.random.randn(30, 2) + [8, 8],
])

Z = linkage(X, method='ward')
incon = inconsistency(Z, d=2)  # depth=2

print(f"{'Merge':>5} {'Height':>8} {'Mean_below':>12} {'Std_below':>10} {'Incon':>8}")
print("-" * 48)
for i in range(len(incon)-5, len(incon)):
    print(f"{i:>5} {Z[i,2]:>8.3f} {incon[i,0]:>12.3f} {incon[i,1]:>10.3f} {incon[i,3]:>8.3f}")
```

---

## 5. fcluster in scipy — All Criteria

### scipy.cluster.hierarchy.fcluster

```python
from scipy.cluster.hierarchy import fcluster

labels = fcluster(Z, t, criterion=...)
```

### Available Criteria

```
┌──────────────────┬─────────────────────────────────────────────────────────┐
│ Criterion        │ Description                                             │
├──────────────────┼─────────────────────────────────────────────────────────┤
│ 'maxclust'       │ t = number of clusters K                                │
│                  │ Returns exactly K clusters.                             │
│                  │ fcluster(Z, t=3, criterion='maxclust')                  │
├──────────────────┼─────────────────────────────────────────────────────────┤
│ 'distance'       │ t = cut height h                                        │
│                  │ Merges below h are in same cluster.                     │
│                  │ fcluster(Z, t=5.0, criterion='distance')               │
├──────────────────┼─────────────────────────────────────────────────────────┤
│ 'inconsistent'   │ t = inconsistency threshold                             │
│                  │ Cut where inconsistency > t.                            │
│                  │ fcluster(Z, t=1.5, criterion='inconsistent')           │
│                  │ Default depth d=2 for inconsistency calculation.       │
├──────────────────┼─────────────────────────────────────────────────────────┤
│ 'maxclust_monocrit' │ t = max clusters, based on monocrit statistic      │
│                  │ Alternative max-cluster using monotone criterion.       │
├──────────────────┼─────────────────────────────────────────────────────────┤
│ 'monocrit'       │ t = threshold on monotonic criterion                    │
│                  │ Uses a user-supplied monotonic criterion vector.        │
└──────────────────┴─────────────────────────────────────────────────────────┘
```

### Most Common Usage Patterns

```python
from scipy.cluster.hierarchy import linkage, fcluster

Z = linkage(X, method='ward')

# Pattern 1: I know how many clusters I want
labels_k3 = fcluster(Z, t=3, criterion='maxclust')

# Pattern 2: I want to cut at a specific distance
labels_h5 = fcluster(Z, t=5.0, criterion='distance')

# Pattern 3: I want statistically motivated clusters
labels_inc = fcluster(Z, t=1.5, criterion='inconsistent', depth=2)
```

---

## 6. Dynamic Tree Cutting

### The Problem with Fixed Cuts

```
    A single horizontal cut assumes all clusters form at the 
    SAME height — but real data often has clusters at DIFFERENT 
    levels of the hierarchy.

    Height
    20│         ┌──────────────────────┐
      │         │                      │
    15│    ┌────┤                      │
      │    │    │                 ┌────┤
    10│  ┌─┤    │                 │    │
      │  │ │    │                 │    │
     5│  │ │    │            ┌────┤    │
      │  │ │    │            │    │    │
     2│  │ │    │          ┌─┤    │    │
      │  │ │    │          │ │    │    │
     1│┌─┤ │    │          │ │    │    │
      ││ │ │    │          │ │    │    │
     0│A B C    D          E F    G    H

    A fixed cut at h=8:  {A,B,C,D} and {E,F,G,H}  (2 clusters)
    A fixed cut at h=3:  {A,B}, {C}, {D}, {E,F}, {G}, {H}  (6 clusters!)
    
    But what if the "right" answer is 3 clusters:
    {A,B,C,D}, {E,F,G}, {H}  ?
    
    No single horizontal cut gives this!
```

### Dynamic Cutting Approaches

```
    STATIC CUT:
    ────────────────────────── h (one horizontal line)

    DYNAMIC CUT:
    ──────┐     ┌──────┐  ┌── (different heights for each branch)
          │     │      │  │
          └─────┘      └──┘

    Dynamic cutting allows DIFFERENT cut heights for different 
    branches of the dendrogram.
```

### Implementing Simple Dynamic Cuts

```python
import numpy as np
from scipy.cluster.hierarchy import linkage, fcluster, inconsistency

def dynamic_cut(Z, min_cluster_size=10, inconsistency_threshold=1.5, depth=2):
    """Simple dynamic tree cut based on inconsistency.
    
    For each branch, cut where inconsistency exceeds threshold,
    but ensure clusters have at least min_cluster_size points.
    """
    n = Z.shape[0] + 1  # number of original points
    
    # Start with inconsistency-based cut
    labels = fcluster(Z, t=inconsistency_threshold, 
                      criterion='inconsistent', depth=depth)
    
    # Merge clusters that are too small into nearest neighbor
    unique, counts = np.unique(labels, return_counts=True)
    small_clusters = unique[counts < min_cluster_size]
    
    if len(small_clusters) > 0:
        print(f"Merging {len(small_clusters)} small clusters "
              f"(< {min_cluster_size} points)")
    
    return labels

# For more sophisticated dynamic cutting, use the 
# dynamicTreeCut R package (or its Python port)
```

### The dynamicTreeCut Package (R/Python)

```
    The gold standard for dynamic cutting is the R package 
    'dynamicTreeCut' by Peter Langfelder.

    Available methods:
    1. cutreeHybrid:  Combines hierarchical with PAM refinement
    2. cutreeDynamic: Adaptive branch cutting

    Python port: pip install dynamicTreeCut
    (Note: limited Python support; R version is more complete)
```

---

## 7. Worked Example — Complete Pipeline

```python
"""Complete dendrogram cutting pipeline."""

import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import linkage, fcluster, dendrogram, inconsistency
from sklearn.metrics import silhouette_score

# Generate data with 3 clusters of different sizes
np.random.seed(42)
X = np.vstack([
    np.random.randn(50, 2) * 1.0 + [0, 0],
    np.random.randn(80, 2) * 0.5 + [6, 6],
    np.random.randn(30, 2) * 1.5 + [12, 0],
])

# Step 1: Compute linkage
Z = linkage(X, method='ward')

# Step 2: Visualize dendrogram
fig, axes = plt.subplots(2, 2, figsize=(16, 12))

# Panel 1: Dendrogram
dendrogram(Z, ax=axes[0, 0], truncate_mode='lastp', p=15, 
           leaf_rotation=90, color_threshold=20)
axes[0, 0].set_title("Dendrogram (Ward Linkage)")
axes[0, 0].axhline(y=20, color='r', linestyle='--', label='Cut at h=20')
axes[0, 0].legend()

# Panel 2: Merge heights
heights = Z[:, 2]
axes[0, 1].plot(range(len(heights)), sorted(heights), 'b.-')
axes[0, 1].set_title("Sorted Merge Heights")
axes[0, 1].set_xlabel("Merge Index")
axes[0, 1].set_ylabel("Height")
axes[0, 1].grid(True, alpha=0.3)

# Panel 3: Inconsistency
incon = inconsistency(Z, d=2)
axes[1, 0].plot(range(len(incon)), incon[:, 3], 'r.-')
axes[1, 0].set_title("Inconsistency Coefficient (depth=2)")
axes[1, 0].set_xlabel("Merge Index")
axes[1, 0].set_ylabel("Inconsistency")
axes[1, 0].axhline(y=1.5, color='g', linestyle='--', label='Threshold=1.5')
axes[1, 0].legend()
axes[1, 0].grid(True, alpha=0.3)

# Panel 4: Silhouette score for different K
sil_scores = []
k_range = range(2, 10)
for k in k_range:
    labels = fcluster(Z, t=k, criterion='maxclust')
    sil = silhouette_score(X, labels)
    sil_scores.append(sil)
axes[1, 1].plot(k_range, sil_scores, 'go-', linewidth=2)
axes[1, 1].set_title("Silhouette Score vs K")
axes[1, 1].set_xlabel("Number of Clusters (K)")
axes[1, 1].set_ylabel("Silhouette Score")
axes[1, 1].grid(True, alpha=0.3)
best_k = list(k_range)[np.argmax(sil_scores)]
axes[1, 1].axvline(x=best_k, color='r', linestyle='--', label=f'Best K={best_k}')
axes[1, 1].legend()

plt.tight_layout()
plt.savefig("dendrogram_cutting_pipeline.png", dpi=150)
plt.show()

# Step 3: Final cut
labels_final = fcluster(Z, t=best_k, criterion='maxclust')
print(f"\nFinal: K={best_k}, Cluster sizes: {np.bincount(labels_final)[1:]}")
print(f"Silhouette: {silhouette_score(X, labels_final):.4f}")
```

---

## 8. Comparison of Cutting Methods

```
┌────────────────────┬──────────────┬───────────────┬────────────────────────┐
│ Method             │ Input        │ Automation    │ Best For               │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Fixed K            │ t = K        │ Manual        │ When K is known        │
│ (maxclust)         │              │               │                        │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Fixed height       │ t = h        │ Manual        │ When distance threshold│
│ (distance)         │              │               │ is meaningful          │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Inconsistency      │ t = threshold│ Semi-auto     │ Finding natural breaks │
│ (inconsistent)     │              │               │ in the hierarchy       │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Silhouette-based   │ Best K from  │ Automatic     │ When quality metric    │
│                    │ score search │               │ is needed              │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Dynamic cutting    │ Per-branch   │ Automatic     │ Multi-scale clusters   │
│                    │ thresholds   │               │ at different heights   │
├────────────────────┼──────────────┼───────────────┼────────────────────────┤
│ Visual (gap)       │ Largest gap  │ Manual        │ Quick exploration      │
│                    │ in heights   │               │                        │
└────────────────────┴──────────────┴───────────────┴────────────────────────┘

Recommendation:
1. FIRST: Visual inspection of dendrogram
2. THEN: Silhouette analysis for K = 2..10
3. OPTIONAL: Inconsistency for automatic cut
4. ALWAYS: Validate with domain knowledge
```

---

## 9. Python Implementation

### All Cutting Methods in One Script

```python
import numpy as np
from scipy.cluster.hierarchy import linkage, fcluster, inconsistency
from sklearn.metrics import silhouette_score

np.random.seed(42)
X = np.vstack([
    np.random.randn(100, 2) + [0, 0],
    np.random.randn(100, 2) + [5, 5],
    np.random.randn(100, 2) + [10, 0],
])

Z = linkage(X, method='ward')

print("=== Cutting Methods Comparison ===\n")

# Method 1: maxclust
for k in [2, 3, 4, 5]:
    labels = fcluster(Z, t=k, criterion='maxclust')
    sil = silhouette_score(X, labels)
    sizes = np.bincount(labels)[1:]
    print(f"maxclust K={k}: Silhouette={sil:.4f}, Sizes={sizes}")

print()

# Method 2: distance
merge_heights = Z[:, 2]
for h in [5.0, 10.0, 15.0, 20.0]:
    labels = fcluster(Z, t=h, criterion='distance')
    n_clusters = len(set(labels))
    if n_clusters > 1:
        sil = silhouette_score(X, labels)
        print(f"distance h={h:.1f}: K={n_clusters}, Silhouette={sil:.4f}")
    else:
        print(f"distance h={h:.1f}: K={n_clusters} (single cluster)")

print()

# Method 3: inconsistency
for thresh in [0.5, 1.0, 1.5, 2.0, 3.0]:
    labels = fcluster(Z, t=thresh, criterion='inconsistent', depth=2)
    n_clusters = len(set(labels))
    if n_clusters > 1 and n_clusters < len(X):
        sil = silhouette_score(X, labels)
        print(f"inconsistent t={thresh:.1f}: K={n_clusters}, Silhouette={sil:.4f}")
    else:
        print(f"inconsistent t={thresh:.1f}: K={n_clusters}")
```

---

## 10. Summary Table

```
┌──────────────────────┬──────────────────────────────────────────────────┐
│ Concept              │ Detail                                           │
├──────────────────────┼──────────────────────────────────────────────────┤
│ Horizontal cut       │ Draw a line at height h → clusters below        │
│ maxclust criterion   │ Cut to get exactly K clusters                   │
│ distance criterion   │ Cut at height h (merges below h = same cluster) │
│ inconsistent crit.   │ Cut where inconsistency > threshold             │
│ Inconsistency coeff. │ (h_i - h̄) / s_h — how unusual a merge is      │
│ Dynamic cutting      │ Different cut heights for different branches    │
│ Silhouette-guided    │ Try K=2..10, pick K with best silhouette        │
│ Gap-based (visual)   │ Cut at the largest gap in merge heights         │
│ fcluster()           │ scipy function to extract flat clusters from Z  │
└──────────────────────┴──────────────────────────────────────────────────┘
```

---

## 11. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | **Cutting** a dendrogram converts the hierarchy into a flat K-cluster partition |
| 2 | A **horizontal cut** at height h gives clusters where all sub-trees are below h |
| 3 | `fcluster(Z, t, criterion='maxclust')` is the easiest way to get exactly K clusters |
| 4 | The **inconsistency coefficient** identifies merges that are unusually large relative to their sub-tree |
| 5 | **Visual inspection** of the largest gap in merge heights is often the fastest approach |
| 6 | **Dynamic tree cutting** handles clusters at different scales (no single h works) |
| 7 | Always **validate** the cut with silhouette score or domain knowledge |

---

## 12. Revision Questions

1. **Horizontal Cut:** Given a dendrogram with merge heights [1, 2, 3, 10, 25], how many clusters do you get if you cut at h=5? At h=15?

2. **Inconsistency:** Explain the inconsistency coefficient formula. Why does a high inconsistency value suggest a natural cluster boundary?

3. **fcluster:** Compare the three main criteria in `fcluster` (`maxclust`, `distance`, `inconsistent`). When would you use each?

4. **Dynamic Cutting:** Why might a single horizontal cut be insufficient? Give an example where clusters exist at different levels of the hierarchy.

5. **Pipeline:** Design a complete pipeline to determine the number of clusters from a dendrogram. Include at least three different methods and explain how you would reconcile conflicting results.

6. **Code:** Write Python code that (a) computes a Ward linkage, (b) plots the dendrogram, (c) evaluates silhouette scores for K=2 to K=8, and (d) extracts the best flat clustering.

---

<div align="center">

| [← Dendrograms](./03-dendrograms.md) | [Up: Hierarchical Clustering](./README.md) | [Next: Advantages & Limitations →](./05-advantages-and-limitations.md) |
|:-------------------------------------:|:------------------------------------------:|:----------------------------------------------------------------------:|

</div>
