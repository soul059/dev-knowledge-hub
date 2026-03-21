# 3.3 Dendrograms — Reading and Creating

> **Chapter Overview:** The **dendrogram** is the defining output of hierarchical clustering — a tree diagram that shows the entire nested sequence of merges (or splits). This chapter teaches you how to read dendrograms, interpret merge heights and distances, create them with scipy, understand their structure, and draw them in ASCII.

---

## Table of Contents

1. [What is a Dendrogram?](#1-what-is-a-dendrogram)
2. [Anatomy of a Dendrogram](#2-anatomy-of-a-dendrogram)
3. [Reading a Dendrogram](#3-reading-a-dendrogram)
4. [Height = Merge Distance](#4-height--merge-distance)
5. [Interpreting Structure from Dendrograms](#5-interpreting-structure-from-dendrograms)
6. [ASCII Dendrogram Examples](#6-ascii-dendrogram-examples)
7. [Creating Dendrograms with scipy](#7-creating-dendrograms-with-scipy)
8. [The Linkage Matrix Z](#8-the-linkage-matrix-z)
9. [Advanced Dendrogram Options](#9-advanced-dendrogram-options)
10. [Cophenetic Distance & Correlation](#10-cophenetic-distance--correlation)
11. [Summary Table](#11-summary-table)
12. [Key Takeaways](#12-key-takeaways)
13. [Revision Questions](#13-revision-questions)

---

## 1. What is a Dendrogram?

A **dendrogram** (from Greek *dendron* = tree, *gramma* = drawing) is a tree diagram that illustrates the hierarchical arrangement of clusters produced by hierarchical clustering.

```
    ┌─────────────────────────────────────────────────────────────┐
    │                        DENDROGRAM                           │
    │                                                             │
    │  Height                                                     │
    │  (merge    10 │           ┌─────────────────┐               │
    │  distance)    │           │                 │               │
    │             8 │      ┌────┤            ┌────┤               │
    │               │      │    │            │    │               │
    │             5 │   ┌──┤    │         ┌──┤    │               │
    │               │   │  │    │         │  │    │               │
    │             2 │ ┌─┤  │    │       ┌─┤  │    │               │
    │               │ │ │  │    │       │ │  │    │               │
    │             0 │ A B  C    │       D E  F    │               │
    │               └───────────────────────────────              │
    │                                                             │
    │  • Leaves (bottom) = individual data points                │
    │  • Vertical lines = merge events                           │
    │  • Height of horizontal bars = distance at which merge     │
    │    occurred                                                │
    │  • Cutting horizontally at some height → K clusters        │
    └─────────────────────────────────────────────────────────────┘
```

---

## 2. Anatomy of a Dendrogram

### Components

```
    Height
      │
    12│              ┌────────────────────┐ ← Final merge (all data)
      │              │                    │
    10│         ┌────┤               ┌────┤ ← "U-shape" = merge event
      │         │    │               │    │
     8│    ┌────┤    │          ┌────┤    │
      │    │    │    │          │    │    │
     5│  ┌─┤    │    │        ┌─┤    │    │
      │  │ │    │    │        │ │    │    │
     2│┌─┤ │    │    │      ┌─┤ │    │    │
      ││ │ │    │    │      │ │ │    │    │
     0│A B C    D    │      E F G    H    │
      └──────────────────────────────────────
                     ↑              ↑
               Leaf labels    Internal nodes

    KEY:
    ─── Horizontal line:  Merge bar (height = merge distance)
    │   Vertical line:    Connects merged clusters to parent
    A,B,C...             Leaves (data points or original clusters)
```

### Reading Rules

```
    1. The Y-AXIS shows the DISTANCE (or dissimilarity) at which 
       clusters were merged.

    2. Two sub-clusters connected by a horizontal bar were MERGED 
       at the height of that bar.

    3. The LOWER a merge happens, the MORE SIMILAR those items are.

    4. TALL vertical lines between merges indicate WELL-SEPARATED 
       clusters.

    5. SHORT vertical lines suggest clusters that are not well-separated.
```

---

## 3. Reading a Dendrogram

### Example: 6 Data Points

```
    Given merge sequence:
    Step 1: Merge A and B at distance 1.0
    Step 2: Merge D and E at distance 1.5
    Step 3: Merge {A,B} and C at distance 3.0
    Step 4: Merge {D,E} and F at distance 4.0
    Step 5: Merge {A,B,C} and {D,E,F} at distance 8.0

    Dendrogram:

    Dist
     8 │          ┌──────────────────────┐
       │          │                      │
     4 │     ┌────┤                 ┌────┤
       │     │    │                 │    │
     3 │  ┌──┤    │                 │    │
       │  │  │    │                 │    │
   1.5│  │  │    │              ┌──┤    │
       │  │  │    │              │  │    │
     1 │┌─┤  │    │              │  │    │
       ││ │  │    │              │  │    │
     0 │A B  C    │              D  E    F
       └──────────────────────────────────

    Reading:
    • A and B are most similar (merged at dist 1.0)
    • D and E are next most similar (dist 1.5)
    • {A,B} merged with C at dist 3.0
    • {D,E} merged with F at dist 4.0
    • The two main groups merged at dist 8.0

    → Natural cut at 2 clusters: {A,B,C} and {D,E,F}
       (because the jump from 4.0 to 8.0 is the largest gap)
```

---

## 4. Height = Merge Distance

### What the Height Means for Each Linkage

```
    ┌──────────────────┬──────────────────────────────────────────┐
    │ Linkage          │ Height at merge = ...                    │
    ├──────────────────┼──────────────────────────────────────────┤
    │ Single           │ MIN distance between any two points     │
    │                  │ in the two clusters being merged        │
    │                  │                                          │
    │ Complete         │ MAX distance between any two points     │
    │                  │ in the two clusters being merged        │
    │                  │                                          │
    │ Average          │ AVERAGE pairwise distance between all   │
    │                  │ pairs across the two clusters           │
    │                  │                                          │
    │ Ward             │ INCREASE in WCSS caused by merging      │
    │                  │ (not a simple distance)                  │
    └──────────────────┴──────────────────────────────────────────┘
```

### Key Insight: Large Height Gaps

```
    Dist
     │
    25│              ┌─────────┐          ← Large gap (25 vs 8)
     │              │         │             means these two groups
    20│              │         │             are VERY different
     │              │         │
    15│              │         │
     │              │         │
    10│         ┌────┤    ┌────┤
     │         │    │    │    │
     8│    ┌────┤    │    │    │          ← Small gaps here
     │    │    │    │    │    │             = similar items
     5│  ┌─┤    │    │  ┌─┤    │
     │  │ │    │    │  │ │    │
     2│┌─┤ │    │    │┌─┤ │    │
     ││ │ │    │    ││ │ │    │
     0│A B C    D    │E F G    H
      └──────────────────────────

    The LARGEST GAP in heights suggests the natural number of clusters.
    Here: gap between 8 and 25 → cut between them → K=2 clusters.
```

---

## 5. Interpreting Structure from Dendrograms

### What Dendrograms Tell You

```
    1. NUMBER OF CLUSTERS:
       Look for the largest vertical gaps. Cut there.
       ─────────────────────────────────────────────

    2. CLUSTER MEMBERSHIP:
       Everything below a cut in the same subtree = same cluster.
       ──────────────────────────────────────────────────────────

    3. SIMILARITY BETWEEN ITEMS:
       Items merged at low height are very similar.
       Items merged at high height are very different.
       ─────────────────────────────────────────────

    4. HIERARCHICAL STRUCTURE:
       Some clusters are "sub-clusters" of larger clusters.
       The tree shows this nesting explicitly.
       ─────────────────────────────────────────────

    5. OUTLIERS:
       Items that merge very late (at large distance)
       are likely outliers.
       ─────────────────────────────────────────────
```

### Example: Identifying Outliers

```
    Dist
     │
    50│                                    ┌──── Outlier Z!
     │                                    │   (merges VERY late)
    25│              ┌────────────────────┤
     │              │                    │
    10│         ┌────┤               ┌────┤
     │         │    │               │    │
     5│    ┌────┤    │          ┌────┤    │
     │    │    │    │          │    │    │
     2│  ┌─┤    │    │        ┌─┤    │    │
     │  │ │    │    │        │ │    │    │
     0│  A B    C    D        E F    G    Z
      └──────────────────────────────────────

    Z joins at distance 50 — far from everything else = OUTLIER
```

---

## 6. ASCII Dendrogram Examples

### Example 1: Three Well-Separated Clusters

```
    Dist
    12│              ┌────────────────────────────┐
      │              │                            │
    10│         ┌────┤                       ┌────┤
      │         │    │                       │    │
     3│    ┌────┤    │                  ┌────┤    │
      │    │    │    │                  │    │    │
     2│  ┌─┤    │    │    ┌────┐     ┌──┤    │    │
      │  │ │    │    │    │    │     │  │    │    │
     1│┌─┤ │    │    │  ┌─┤    │   ┌─┤  │    │    │
      ││ │ │    │    │  │ │    │   │ │  │    │    │
     0│A B C    D    │  E F    G   H I  J    K    L
      └──────────────────────────────────────────────

    Three clear clusters:
    • {A, B, C, D} — merge internally at heights 1-3
    • {E, F, G}    — merge internally at heights 1-2
    • {H, I, J, K, L} — merge internally at heights 1-3
    • The three groups merge at heights 10 and 12

    Cut at height 5 → 3 clusters (clear separation!)
```

### Example 2: No Clear Structure

```
    Dist
    10│                    ┌───────────────┐
      │               ┌───┤               │
     9│          ┌────┤    │          ┌────┤
      │     ┌────┤    │    │     ┌────┤    │
     8│┌────┤    │    │    │┌────┤    │    │
      ││    │    │    │    ││    │    │    │
     7│A    B    C    D    │E    F    G    H
      └──────────────────────────────────────

    All merges happen at similar heights (7-10).
    → No clear cluster structure! 
    → Data may be uniformly distributed.
```

### Example 3: Hierarchical Sub-Structure

```
    Dist
    20│                    ┌──────────────────────────────┐
      │                    │                              │
    15│              ┌─────┤                        ┌─────┤
      │              │     │                        │     │
    10│         ┌────┤     │                   ┌────┤     │
      │         │    │     │                   │    │     │
     5│    ┌────┤    │     │              ┌────┤    │     │
      │    │    │    │     │              │    │    │     │
     2│  ┌─┤  ┌─┤    │     │            ┌─┤  ┌─┤    │     │
      │  │ │  │ │    │     │            │ │  │ │    │     │
     0│  A B  C D    E     │            F G  H I    J     K
      └────────────────────────────────────────────────────

    Hierarchical interpretation:
    K=2: {A,B,C,D,E} vs {F,G,H,I,J,K}
    K=4: {A,B} {C,D} {F,G} {H,I}  with E,J,K as singletons → K=6
    K=6: {A,B} {C,D} {E} {F,G} {H,I} {J,K}  → possible subgroups
```

---

## 7. Creating Dendrograms with scipy

### Basic Dendrogram

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import linkage, dendrogram

# Create sample data
np.random.seed(42)
X = np.vstack([
    np.random.randn(8, 2) + [0, 0],    # Cluster 1
    np.random.randn(8, 2) + [6, 6],    # Cluster 2
    np.random.randn(8, 2) + [12, 0],   # Cluster 3
])
labels = [f'C1_{i}' for i in range(8)] + \
         [f'C2_{i}' for i in range(8)] + \
         [f'C3_{i}' for i in range(8)]

# Compute linkage
Z = linkage(X, method='ward', metric='euclidean')

# Plot dendrogram
plt.figure(figsize=(14, 7))
dn = dendrogram(Z, labels=labels, leaf_rotation=90, leaf_font_size=10,
                color_threshold=15)
plt.title("Dendrogram (Ward Linkage)", fontsize=16)
plt.xlabel("Data Points", fontsize=12)
plt.ylabel("Distance (Ward)", fontsize=12)
plt.axhline(y=15, color='r', linestyle='--', label='Cut at K=3')
plt.legend(fontsize=12)
plt.tight_layout()
plt.savefig("dendrogram_basic.png", dpi=150)
plt.show()
```

### Customized Dendrogram

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import linkage, dendrogram, fcluster

np.random.seed(42)
X = np.vstack([
    np.random.randn(25, 2) + [0, 0],
    np.random.randn(25, 2) + [5, 5],
    np.random.randn(25, 2) + [10, 0],
])

Z = linkage(X, method='ward')

fig, axes = plt.subplots(1, 2, figsize=(18, 7))

# Full dendrogram
dendrogram(Z, ax=axes[0], leaf_rotation=90, leaf_font_size=6)
axes[0].set_title("Full Dendrogram (75 leaves)")
axes[0].set_ylabel("Distance")

# Truncated dendrogram (show only top levels)
dendrogram(Z, ax=axes[1], truncate_mode='lastp', p=10,
           show_leaf_counts=True, leaf_rotation=45, leaf_font_size=10)
axes[1].set_title("Truncated Dendrogram (top 10 merges)")
axes[1].set_ylabel("Distance")

plt.tight_layout()
plt.savefig("dendrogram_customized.png", dpi=150)
plt.show()
```

---

## 8. The Linkage Matrix Z

### Structure

```
    Z = linkage(X, method='ward')

    Z is an (n-1) × 4 matrix where each row represents one merge:

    Z[i] = [idx1, idx2, distance, count]

    • idx1, idx2:  Indices of the two clusters being merged
                   (0 to n-1 = original points; n+ = previously formed clusters)
    • distance:    The distance at which the merge occurred
    • count:       Number of original points in the new cluster

    Example for n=5 points (A=0, B=1, C=2, D=3, E=4):

    Z = [
        [0, 1, 1.0, 2],   # Merge 0: A(0) + B(1) → cluster 5, dist=1.0
        [3, 4, 1.5, 2],   # Merge 1: D(3) + E(4) → cluster 6, dist=1.5
        [2, 5, 3.0, 3],   # Merge 2: C(2) + cl5{A,B} → cluster 7, dist=3.0
        [6, 7, 8.0, 5],   # Merge 3: cl6{D,E} + cl7{A,B,C} → cluster 8, dist=8.0
    ]
```

### How to Interpret Z

```python
from scipy.cluster.hierarchy import linkage
import numpy as np

X = np.array([[0, 0], [1, 0], [5, 0], [6, 0], [3, 5]])
Z = linkage(X, method='single')

print(f"{'Merge':>5} {'Cl_i':>5} {'Cl_j':>5} {'Dist':>8} {'Size':>5}")
print("-" * 30)
for i, row in enumerate(Z):
    print(f"{i:>5} {int(row[0]):>5} {int(row[1]):>5} {row[2]:>8.3f} {int(row[3]):>5}")

# Merge    Cl_i  Cl_j     Dist  Size
# -------- ----- ----- -------- -----
#     0       0     1    1.000     2    ← points 0,1 merged
#     1       2     3    1.000     2    ← points 2,3 merged
#     2       5     6    3.000     4    ← clusters {0,1} and {2,3} merged
#     3       4     7    2.236     5    ← point 4 joins everything
```

---

## 9. Advanced Dendrogram Options

### Truncation

```python
# Show only the top p merges (for large datasets)
dendrogram(Z, truncate_mode='lastp', p=12)

# Show only merges above a certain level
dendrogram(Z, truncate_mode='level', p=3)
```

### Color Thresholding

```python
# Color clusters based on a distance threshold
dendrogram(Z, color_threshold=5.0)
# Merges below 5.0 = colored (same cluster)
# Merges above 5.0 = default color (different clusters)
```

### Orientation

```python
# Horizontal dendrogram
dendrogram(Z, orientation='left')

# Top-down
dendrogram(Z, orientation='top')  # default
```

### With Cluster IDs

```python
from scipy.cluster.hierarchy import fcluster

# Get cluster labels for a specific number of clusters
labels = fcluster(Z, t=3, criterion='maxclust')

# Or for a specific distance threshold
labels = fcluster(Z, t=5.0, criterion='distance')
```

---

## 10. Cophenetic Distance & Correlation

### Cophenetic Distance

The **cophenetic distance** between two points is the height in the dendrogram at which they first merge into the same cluster.

```
    Dendrogram:
    
    8 │         ┌──────────┐
      │         │          │
    5 │    ┌────┤     ┌────┤
      │    │    │     │    │
    2 │  ┌─┤    │     │    │
      │  │ │    │     │    │
    0 │  A B    C     D    E

    Cophenetic distances:
    d_coph(A, B) = 2   (merge at height 2)
    d_coph(A, C) = 5   (merge at height 5)
    d_coph(A, D) = 8   (merge at height 8)
    d_coph(D, E) = 5   (merge at height 5)
```

### Cophenetic Correlation Coefficient

```
    Measures how well the dendrogram preserves pairwise distances:

    c = corr(d_original(i,j), d_cophenetic(i,j))    for all pairs i,j

    Range: [0, 1]
    • c > 0.75:  Good representation
    • c > 0.85:  Very good representation
    • c < 0.50:  Poor — dendrogram doesn't reflect data well
```

### Python Implementation

```python
from scipy.cluster.hierarchy import linkage, cophenet
from scipy.spatial.distance import pdist
import numpy as np

np.random.seed(42)
X = np.vstack([
    np.random.randn(30, 2) + [0, 0],
    np.random.randn(30, 2) + [5, 5],
    np.random.randn(30, 2) + [10, 0],
])

# Compare linkage methods using cophenetic correlation
original_dists = pdist(X)

for method in ['single', 'complete', 'average', 'ward']:
    Z = linkage(X, method=method)
    c, coph_dists = cophenet(Z, original_dists)
    print(f"{method:>10s} linkage: cophenetic correlation = {c:.4f}")

# Typical output:
#     single linkage: cophenetic correlation = 0.85
#   complete linkage: cophenetic correlation = 0.78
#    average linkage: cophenetic correlation = 0.89  ← often best!
#       ward linkage: cophenetic correlation = 0.72
```

---

## 11. Summary Table

```
┌──────────────────────┬──────────────────────────────────────────────────┐
│ Concept              │ Detail                                           │
├──────────────────────┼──────────────────────────────────────────────────┤
│ Dendrogram           │ Tree diagram showing merge hierarchy            │
│ Height (Y-axis)      │ Distance at which merge occurred                │
│ Leaves               │ Individual data points (bottom)                 │
│ Internal nodes       │ Merged clusters                                 │
│ Horizontal bar       │ Merge event; height = merge distance            │
│ Large vertical gap   │ Suggests natural cluster separation             │
│ Cutting              │ Horizontal cut → K clusters                     │
│ Linkage matrix Z     │ (n-1)×4 matrix: [cl_i, cl_j, dist, size]       │
│ Cophenetic distance  │ Height at which two points first join           │
│ Cophenetic corr.     │ How well dendrogram preserves original dists    │
│ Truncation           │ Show only top merges for large datasets         │
│ Color threshold      │ Color clusters below a distance cutoff          │
└──────────────────────┴──────────────────────────────────────────────────┘
```

---

## 12. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | A dendrogram is a **tree diagram** showing the complete merge history |
| 2 | **Height** of horizontal bars = distance at which merge occurred |
| 3 | **Large gaps** between merge heights suggest natural cluster boundaries |
| 4 | The linkage matrix **Z** encodes the entire dendrogram in a (n-1)×4 matrix |
| 5 | **Cutting** the dendrogram horizontally at height h gives a flat clustering |
| 6 | **Cophenetic correlation** measures how faithfully the dendrogram represents original distances |
| 7 | For large datasets, use **truncated** dendrograms to show only the most informative merges |

---

## 13. Revision Questions

1. **Reading:** Given a dendrogram, how do you determine (a) the number of natural clusters, (b) which points are most similar, and (c) whether outliers exist?

2. **Height:** Explain what the height means for each of the four main linkage methods (single, complete, average, Ward).

3. **Linkage Matrix:** Given Z = [[0,1,2.0,2], [2,3,3.0,2], [4,5,7.0,4]], draw the corresponding dendrogram. How many original points are there?

4. **Cophenetic Correlation:** Define the cophenetic distance and cophenetic correlation coefficient. Which linkage method typically gives the highest cophenetic correlation?

5. **Practical:** You have 10,000 data points and create a dendrogram. Why would you use truncation? What parameter choices in `dendrogram()` would you use?

6. **ASCII:** Draw an ASCII dendrogram for 5 points where: A and B merge at height 1, C and D merge at height 2, {A,B} and {C,D} merge at height 5, and E joins at height 8.

---

<div align="center">

| [← Linkage Methods](./02-linkage-methods.md) | [Up: Hierarchical Clustering](./README.md) | [Next: Cutting the Dendrogram →](./04-cutting-the-dendrogram.md) |
|:---------------------------------------------:|:------------------------------------------:|:----------------------------------------------------------------:|

</div>
