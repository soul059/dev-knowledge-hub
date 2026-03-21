# 2. Core Points, Border Points & Noise Points

[← Previous: Density-Based Concept](./01-density-based-concept.md) | [Next: Parameters eps & min_samples →](./03-parameters-eps-minsamples.md)

---

## Chapter Overview

DBSCAN classifies every point in the dataset into exactly one of three categories: **core point**, **border point**, or **noise point**. This classification is the engine that drives cluster formation. Understanding these three roles is essential to understanding how DBSCAN builds clusters of arbitrary shape while explicitly identifying outliers.

---

## 2.1 The Three Point Types

### 2.1.1 Core Point

A point **p** is a **core point** if it has at least `min_samples` points (including itself) within its ε-neighborhood.

```
Definition:
  p is a core point ⟺ |N_eps(p)| ≥ min_samples

  where N_eps(p) = { q ∈ D : dist(p, q) ≤ eps }
```

**Intuition:** Core points are in the "thick" of a cluster — surrounded by enough neighbors to constitute a dense region.

### 2.1.2 Border Point

A point **p** is a **border point** if:
1. It is **NOT** a core point (has fewer than `min_samples` neighbors)
2. It is within the ε-neighborhood of at least one core point

```
Definition:
  p is a border point ⟺ |N_eps(p)| < min_samples  AND
                          ∃ core point c : dist(p, c) ≤ eps
```

**Intuition:** Border points are on the "edge" of a cluster — they belong to a cluster but aren't dense enough to be core points themselves.

### 2.1.3 Noise Point

A point **p** is a **noise point** (outlier) if:
1. It is **NOT** a core point
2. It is **NOT** within the ε-neighborhood of any core point

```
Definition:
  p is a noise point ⟺ |N_eps(p)| < min_samples  AND
                         ∄ core point c : dist(p, c) ≤ eps
```

**Intuition:** Noise points are isolated — too far from any dense region to belong to any cluster.

---

## 2.2 ASCII Visualization

```
Parameters: eps = 1.0, min_samples = 4

Legend:  ★ = Core point   ◆ = Border point   ✕ = Noise point

     6 │                                    ✕ (isolated)
       │
     5 │         ★───★
       │        ╱ ╲ ╱ ╲
     4 │       ★───★───◆  (border: < 4 neighbors, but near core)
       │        ╲ ╱
     3 │         ★
       │                          ✕ (isolated)
     2 │
       │  ◆───★───★
     1 │      │ ╲ │
       │      ★───★───◆
     0 │
       └──┬──┬──┬──┬──┬──┬──┬──┬──┬──
          0  1  2  3  4  5  6  7  8

  Cluster 1 (bottom-left):  4 core + 2 border = 6 points
  Cluster 2 (upper-middle): 5 core + 1 border = 6 points
  Noise:                    2 noise points
```

### Detailed Neighborhood View

```
   eps = 1.0, min_samples = 4

   CORE POINT (★):                 BORDER POINT (◆):           NOISE POINT (✕):
   ┌─────────────────┐             ┌─────────────────┐         ┌─────────────────┐
   │    eps circle    │             │    eps circle    │         │    eps circle    │
   │   ╭─────────╮   │             │   ╭─────────╮   │         │   ╭─────────╮   │
   │   │  ·   ·  │   │             │   │         │   │         │   │         │   │
   │   │ · (★) · │   │             │   │ ·  (◆)  │   │         │   │   (✕)   │   │
   │   │  ·   ·  │   │             │   │    ·    │   │         │   │         │   │
   │   ╰─────────╯   │             │   ╰─────────╯   │         │   ╰─────────╯   │
   │                  │             │                  │         │                  │
   │ Neighbors ≥ 4 ✓  │             │ Neighbors < 4   │         │ Neighbors < 4   │
   │ → CORE           │             │ But near a ★    │         │ Not near any ★  │
   └─────────────────┘             │ → BORDER         │         │ → NOISE          │
                                    └─────────────────┘         └─────────────────┘
```

---

## 2.3 Classification Step-by-Step

Given a dataset D, parameters `eps` and `min_samples`, the classification proceeds:

### Algorithm: Classify All Points

```
PROCEDURE ClassifyPoints(D, eps, min_samples):

    Step 1: For each point p in D:
        ├── Compute N_eps(p) = {q ∈ D : dist(p, q) ≤ eps}
        └── Count |N_eps(p)|

    Step 2: Mark core points:
        └── IF |N_eps(p)| ≥ min_samples → mark p as CORE

    Step 3: Mark border points:
        └── For each non-core point p:
            └── IF ∃ core point c with dist(p, c) ≤ eps → mark p as BORDER

    Step 4: Mark noise points:
        └── All remaining unmarked points → mark as NOISE
```

### Worked Example

**Dataset:** 10 points in 2D, `eps = 2.0`, `min_samples = 3`

```
Points:
  P1(1,1)  P2(2,1)  P3(1.5,2)  P4(8,8)   P5(9,8)
  P6(8,9)  P7(9,9)  P8(5,5)    P9(2,2.5) P10(15,15)

  16│
  15│                                               P10
    │
    │
    │
  10│
   9│                    P6    P7
   8│                    P4    P5
    │
    │
   5│              P8
    │
    │
  2 │    P3 P9
  1 │  P1  P2
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──
       1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
```

**Step 1: Compute neighborhoods (eps = 2.0)**

| Point | Neighbors (within eps=2.0) | Count | 
|-------|---------------------------|-------|
| P1(1,1) | P2(1.0), P3(1.12), P9(1.80) | 4 |
| P2(2,1) | P1(1.0), P3(1.12), P9(1.50) | 4 |
| P3(1.5,2) | P1(1.12), P2(1.12), P9(0.71) | 4 |
| P4(8,8) | P5(1.0), P6(1.0), P7(1.41) | 4 |
| P5(9,8) | P4(1.0), P7(1.0), P6(1.41) | 4 |
| P6(8,9) | P4(1.0), P7(1.0), P5(1.41) | 4 |
| P7(9,9) | P5(1.0), P6(1.0), P4(1.41) | 4 |
| P8(5,5) | — | 1 |
| P9(2,2.5) | P1(1.80), P2(1.50), P3(0.71) | 4 |
| P10(15,15) | — | 1 |

**Step 2: Identify core points (count ≥ 3)**

```
Core Points: P1, P2, P3, P4, P5, P6, P7, P9    (count ≥ 3 ✓)
```

**Step 3: Identify border points**

```
P8: Not core (1 neighbor). Nearest core point? 
    dist(P8, nearest core) > 2.0 for all core points.
    → NOT border → will be NOISE

P10: Not core (1 neighbor). Nearest core point?
    dist(P10, nearest core) > 2.0 for all core points.
    → NOT border → will be NOISE
```

**Step 4: Final classification**

| Point | Type | Cluster |
|-------|------|---------|
| P1(1,1) | ★ Core | Cluster 1 |
| P2(2,1) | ★ Core | Cluster 1 |
| P3(1.5,2) | ★ Core | Cluster 1 |
| P9(2,2.5) | ★ Core | Cluster 1 |
| P4(8,8) | ★ Core | Cluster 2 |
| P5(9,8) | ★ Core | Cluster 2 |
| P6(8,9) | ★ Core | Cluster 2 |
| P7(9,9) | ★ Core | Cluster 2 |
| P8(5,5) | ✕ Noise | -1 |
| P10(15,15) | ✕ Noise | -1 |

---

## 2.4 The Role of Each Point Type in Clusters

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  ┌──────── CLUSTER ────────┐                             │
│  │                         │                             │
│  │  ★─────★─────★         │    ★ Core: forms cluster    │
│  │  │    ╱ │   ╱ │         │       backbone (connected   │
│  │  │   ╱  │  ╱  │         │       to other cores)       │
│  │  ★─────★    ◆─┤         │                             │
│  │  │    ╱      ├─┘         │    ◆ Border: attached to   │
│  │  │   ╱       │           │       cluster edge          │
│  │  ★─────◆     │           │                             │
│  │              │           │                             │
│  └──────────────┘           │    ✕ Noise: isolated        │
│                    ✕        │       from all clusters     │
│           ✕                 │                             │
└──────────────────────────────────────────────────────────┘
```

### Structural Roles

| Role | Core Point (★) | Border Point (◆) | Noise Point (✕) |
|------|----------------|-------------------|-----------------|
| **Cluster membership** | Always belongs to a cluster | Belongs to a cluster | No cluster (label = -1) |
| **Cluster expansion** | Can expand cluster to neighbors | Cannot expand cluster | N/A |
| **Multiple clusters** | Belongs to exactly one cluster | Could be near multiple cores (assigned to first found) | N/A |
| **Structural role** | Backbone / interior | Periphery / edge | Outlier |
| **Removal effect** | May split cluster | No structural impact | No impact |

---

## 2.5 Edge Cases and Important Details

### 2.5.1 Border Point Assignment Ambiguity

A border point may be within eps of core points from **different clusters**. DBSCAN assigns it to the **first cluster encountered** during traversal (order-dependent).

```
      Cluster A          Cluster B
      ★───★───★    ◆    ★───★───★
                   │
           This border point could belong
           to either cluster. Assignment
           depends on processing order.
```

> **Note:** This is the **only source of non-determinism** in DBSCAN (with ties in border point assignment). Core points and noise points are always deterministically classified.

### 2.5.2 The "Including Self" Convention

Some implementations count the point itself in its ε-neighborhood. sklearn's DBSCAN **includes the point itself** in the neighbor count.

```
If min_samples = 5 and a point has 4 other neighbors within eps:
  - With self-counting: |N_eps(p)| = 5 ≥ 5 → CORE ✓
  - Without self-counting: |N_eps(p)| = 4 < 5 → NOT core ✗

sklearn uses self-counting, so min_samples = 5 means
"the point plus at least 4 others within eps"
```

### 2.5.3 Sensitivity of Classification to Parameters

```
Same dataset, different parameters:

eps = 0.5, min_samples = 3:        eps = 2.0, min_samples = 3:
  Many noise points                   Fewer noise points
  Small, tight clusters               Larger, merged clusters
  Some core→border                    Some border→core
  Some border→noise                   Some noise→border

  ★ ◆ ✕ ✕ ✕    ★ ★           ★ ★ ★ ★ ★    ★ ★
  ✕ ★ ◆ ✕ ★    ★ ◆    →     ★ ★ ★ ◆ ★    ★ ★
  ✕ ✕ ★ ★ ◆    ✕             ★ ★ ★ ★ ★    ◆
```

---

## 2.6 Python Implementation

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.datasets import make_blobs
from collections import Counter

# Generate sample data with noise
X, _ = make_blobs(n_samples=300, centers=3, cluster_std=0.5, random_state=42)
# Add some noise points
rng = np.random.RandomState(42)
noise = rng.uniform(low=-10, high=10, size=(20, 2))
X = np.vstack([X, noise])

# Fit DBSCAN
db = DBSCAN(eps=0.8, min_samples=5)
labels = db.fit_predict(X)

# Identify point types
core_mask = np.zeros(len(X), dtype=bool)
core_mask[db.core_sample_indices_] = True

# Core points: in core_sample_indices_
n_core = core_mask.sum()

# Noise points: label == -1
noise_mask = labels == -1
n_noise = noise_mask.sum()

# Border points: not core AND not noise
border_mask = ~core_mask & ~noise_mask
n_border = border_mask.sum()

print(f"Total points:  {len(X)}")
print(f"Core points:   {n_core}")
print(f"Border points: {n_border}")
print(f"Noise points:  {n_noise}")
print(f"Clusters found: {len(set(labels)) - (1 if -1 in labels else 0)}")
print(f"\nCore sample indices (first 10): {db.core_sample_indices_[:10]}")

# Examine a specific border point
border_indices = np.where(border_mask)[0]
if len(border_indices) > 0:
    bp = border_indices[0]
    bp_cluster = labels[bp]
    # Find which core point it's near
    from sklearn.metrics import pairwise_distances
    dists = pairwise_distances(X[bp:bp+1], X[core_mask])[0]
    nearest_core_dist = dists.min()
    print(f"\nBorder point {bp}: cluster {bp_cluster}, "
          f"distance to nearest core = {nearest_core_dist:.3f}")
```

**Expected Output:**
```
Total points:  320
Core points:   276
Border points: 18
Noise points:  26
Clusters found: 3

Core sample indices (first 10): [  0   1   2   3   4   5   6   7   8   9]

Border point 300: cluster 0, distance to nearest core = 0.723
```

### Manual Classification Function

```python
def classify_dbscan_points(X, eps, min_samples):
    """Manually classify points as core, border, or noise."""
    from sklearn.neighbors import NearestNeighbors
    
    nn = NearestNeighbors(radius=eps)
    nn.fit(X)
    neighborhoods = nn.radius_neighbors(X, return_distance=False)
    
    # Step 1: Identify core points
    n_neighbors = np.array([len(neighbors) for neighbors in neighborhoods])
    core_mask = n_neighbors >= min_samples
    core_indices = set(np.where(core_mask)[0])
    
    classification = []
    for i in range(len(X)):
        if i in core_indices:
            classification.append('CORE')
        elif any(j in core_indices for j in neighborhoods[i]):
            classification.append('BORDER')
        else:
            classification.append('NOISE')
    
    return classification

# Test on simple data
X_simple = np.array([
    [0, 0], [0.5, 0], [0, 0.5], [0.3, 0.3],   # Dense group
    [5, 5], [5.5, 5], [5, 5.5],                 # Smaller group
    [10, 10]                                      # Isolated
])

result = classify_dbscan_points(X_simple, eps=1.0, min_samples=3)
for point, label in zip(X_simple, result):
    print(f"  Point {point} → {label}")
```

**Expected Output:**
```
  Point [0.  0. ] → CORE
  Point [0.5 0. ] → CORE
  Point [0.  0.5] → CORE
  Point [0.3 0.3] → CORE
  Point [5.  5. ] → CORE
  Point [5.5 5. ] → CORE
  Point [5.  5.5] → CORE
  Point [10. 10.] → NOISE
```

---

## 2.7 Visualizing Point Types

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.datasets import make_moons

# Generate moon-shaped data
X, _ = make_moons(n_samples=200, noise=0.1, random_state=42)
# Add noise
rng = np.random.RandomState(42)
X = np.vstack([X, rng.uniform(-1.5, 2.5, size=(15, 2))])

db = DBSCAN(eps=0.3, min_samples=5).fit(X)

core_mask = np.zeros(len(X), dtype=bool)
core_mask[db.core_sample_indices_] = True
border_mask = ~core_mask & (db.labels_ != -1)
noise_mask = db.labels_ == -1

# Print summary
print("Point Type Distribution:")
print(f"  ★ Core:   {core_mask.sum():3d} points (cluster interior)")
print(f"  ◆ Border: {border_mask.sum():3d} points (cluster edge)")
print(f"  ✕ Noise:  {noise_mask.sum():3d} points (outliers)")

# For visualization (matplotlib):
# import matplotlib.pyplot as plt
# plt.scatter(X[core_mask, 0], X[core_mask, 1], c='blue', s=50, label='Core')
# plt.scatter(X[border_mask, 0], X[border_mask, 1], c='green', s=50, label='Border')
# plt.scatter(X[noise_mask, 0], X[noise_mask, 1], c='red', s=50, marker='x', label='Noise')
# plt.legend()
# plt.title('DBSCAN Point Classification')
# plt.show()
```

---

## 2.8 Summary Table

| Concept | Core Point (★) | Border Point (◆) | Noise Point (✕) |
|---------|----------------|-------------------|-----------------|
| **Condition** | ≥ min_samples neighbors within eps | < min_samples neighbors, but within eps of a core | Not near any core point |
| **Cluster role** | Interior / backbone | Periphery / edge | No cluster (label = -1) |
| **Can expand cluster?** | ✅ Yes | ❌ No | ❌ No |
| **Deterministic?** | ✅ Always same | ⚠️ Border assignment may vary | ✅ Always same |
| **sklearn access** | `db.core_sample_indices_` | `~core & label != -1` | `label == -1` |
| **Structural impact** | Removing may disconnect | Removing has no structural effect | No effect |
| **Density** | High (dense neighborhood) | Moderate (sparse but connected) | Low (isolated) |

---

## 2.9 Revision Questions

1. **Define core point, border point, and noise point** using formal mathematical notation. What are the two conditions that must be checked for each?

2. **Given 8 points** with eps=1.5 and min_samples=3: A(0,0), B(1,0), C(0,1), D(5,5), E(6,5), F(5,6), G(3,3), H(20,20). Classify each point and identify the clusters.

3. **Explain why border point assignment is non-deterministic** in DBSCAN. Under what circumstances does this matter? How does sklearn handle this?

4. **What happens to the classification** of points if you increase eps while keeping min_samples fixed? What about increasing min_samples while keeping eps fixed?

5. **A point has exactly min_samples - 1 neighbors** within its eps-neighborhood. Under what conditions is it a border point vs. a noise point?

6. **Why are core points critical** for cluster structure while border points are not? What happens to a cluster if you remove a core point from its interior?

---

[← Previous: Density-Based Concept](./01-density-based-concept.md) | [Next: Parameters eps & min_samples →](./03-parameters-eps-minsamples.md)
