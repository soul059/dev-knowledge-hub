# 3. Parameters: eps and min_samples

[← Previous: Core, Border & Noise Points](./02-core-border-noise-points.md) | [Next: Algorithm Steps →](./04-algorithm-steps.md)

---

## Chapter Overview

DBSCAN's behavior is controlled by just two parameters: **eps (ε)** — the neighborhood radius — and **min_samples** — the minimum number of points required to form a dense region. Unlike K-Means, DBSCAN does not require specifying the number of clusters. However, choosing appropriate values for eps and min_samples is critical: poor choices lead to either everything in one cluster, everything as noise, or fragmented results. This chapter covers the meaning of each parameter, systematic methods for selecting them, and sensitivity analysis.

---

## 3.1 Parameter: eps (ε) — Neighborhood Radius

### Definition

`eps` defines the radius of the neighborhood around each point. Two points are considered neighbors if the distance between them is ≤ eps.

```
N_eps(p) = { q ∈ D : dist(p, q) ≤ eps }
```

### Effect of eps on Clustering

```
eps too SMALL                  eps OPTIMAL                 eps too LARGE
─────────────                  ───────────                 ─────────────
   ·  ·  ·                     ┌─────────┐                ┌───────────────────┐
   ·  ·  ·                     │ · · · · │                │  · · · · · · · ·  │
   ·  ·  ·                     │ · · · · │                │  · · · · · · · ·  │
                                └─────────┘                │                   │
   ·  ·  ·                     ┌─────────┐                │  · · · · · · · ·  │
   ·  ·  ·                     │ · · · · │                │  · · · · · · · ·  │
   ·  ·  ·                     │ · · · · │                └───────────────────┘
                                └─────────┘

Tiny neighborhoods           Right-sized neighborhoods     Huge neighborhood
→ No core points              → Natural clusters found      → Everything in one cluster
→ Everything is NOISE         → Noise properly detected     → No noise detected
→ Too many small clusters     → Correct # of clusters       → 1 giant cluster
```

### Mathematical Perspective

The eps parameter implicitly defines the **density threshold**:

```
Density threshold ρ = min_samples / Volume(eps-ball)

For 2D:  ρ = min_samples / (π · eps²)
For dD:  ρ = min_samples / (V_d · eps^d)

where V_d = volume of unit d-dimensional ball
      V_d = π^(d/2) / Γ(d/2 + 1)
```

---

## 3.2 Parameter: min_samples — Density Threshold

### Definition

`min_samples` is the minimum number of points (including the point itself) that must be within the eps-neighborhood for a point to be classified as a **core point**.

### Effect of min_samples on Clustering

```
min_samples too SMALL (e.g., 2)     min_samples OPTIMAL        min_samples too LARGE
───────────────────────────          ──────────────────         ────────────────────────
  Many core points                   Balanced core/border       Few core points
  Noise absorbed into clusters       Natural cluster structure  Real clusters become noise
  Spurious small clusters            Noise properly identified  Over-fragmented results

  ★★★★★★★★★★★★★                    ★★★★★★◆                     ★★◆
  ★★★★★★★★★★★★★          →         ★★★★★★◆           →         ◆✕✕✕
  ★★★★★★★★★★★★★                    ★★★★★★                      ✕✕✕✕
                                     
  ★★★★★★★★                          ★★★★◆                       ✕✕✕
  ★★★★★★★★                          ★★★★◆                       ✕✕✕
```

### Rule of Thumb for min_samples

The most common heuristic:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   min_samples ≥ D + 1                                         │
│                                                                │
│   where D = number of dimensions (features)                   │
│                                                                │
│   Common practice:                                             │
│     • min_samples = 2 × D  (for noisy data)                  │
│     • min_samples ≥ 3      (absolute minimum for any data)    │
│     • For 2D data: min_samples = 4 to 8                       │
│     • For high-D data: scale with dimensionality               │
│                                                                │
│   Larger min_samples:                                          │
│     → More conservative (more points classified as noise)      │
│     → Smoother cluster boundaries                              │
│     → Better for larger datasets                               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Key Relationships:**

| Dataset Property | Suggested min_samples |
|-----------------|----------------------|
| Small dataset (< 100 points) | 3–5 |
| Medium dataset (100–10K) | 5–10 |
| Large dataset (> 10K) | 10–20+ |
| High-dimensional | ≥ 2 × D |
| Very noisy data | Increase by 2–5× |
| Clean data | Can use smaller values |

---

## 3.3 The k-Distance Plot for Choosing eps

The **k-distance plot** (also called the "elbow method for DBSCAN") is the primary systematic method for choosing eps.

### Algorithm

```
1. Choose k = min_samples (or min_samples - 1)
2. For each point, compute the distance to its k-th nearest neighbor
3. Sort these distances in ascending order
4. Plot the sorted distances
5. Look for an "elbow" (sharp bend) in the plot
6. The y-value at the elbow = optimal eps
```

### Intuition

```
k-Distance Plot:

Distance to
k-th neighbor
     │
  5  │                                          ╱
     │                                        ╱
  4  │                                      ╱     ← noise points
     │                                    ╱        (far from neighbors)
  3  │                                 ╱╱
     │                              ╱╱
  2  │                          ╱╱╱
     │                    ╌╌╌╌╌      ← ELBOW → eps ≈ 1.5
  1.5│··················╱····················
     │           ╱╱╱╱╱╱
  1  │     ╱╱╱╱╱╱
     │  ╱╱╱                ← cluster points
  0.5│╱╱                     (close to neighbors)
     │
  0  └──────────┬──────────┬──────────┬──────
               100        200        300
                Points (sorted by k-distance)
```

**Why the Elbow Works:**
- Points **within clusters** have small k-distances (densely packed)
- **Noise points** have large k-distances (isolated)
- The **transition** between these two regimes creates a sharp bend (elbow)
- The eps at the elbow separates cluster density from noise density

### Python Implementation

```python
import numpy as np
from sklearn.neighbors import NearestNeighbors
from sklearn.datasets import make_moons

# Generate data
X, _ = make_moons(n_samples=300, noise=0.1, random_state=42)

# Choose k = min_samples
k = 5

# Compute k-distances
nn = NearestNeighbors(n_neighbors=k)
nn.fit(X)
distances, _ = nn.kneighbors(X)

# Sort the k-th nearest neighbor distances
k_distances = np.sort(distances[:, k-1])  # k-1 because 0-indexed

# Find the elbow programmatically (simple heuristic)
# Using maximum curvature
diffs = np.diff(k_distances)
diffs2 = np.diff(diffs)
elbow_idx = np.argmax(diffs2) + 2
optimal_eps = k_distances[elbow_idx]

print(f"k = {k}")
print(f"Suggested eps (elbow) ≈ {optimal_eps:.3f}")
print(f"\nk-distance statistics:")
print(f"  Min:    {k_distances[0]:.4f}")
print(f"  Median: {np.median(k_distances):.4f}")
print(f"  Max:    {k_distances[-1]:.4f}")

# Print ASCII k-distance plot
n_bins = 20
bin_size = len(k_distances) // n_bins
print(f"\nApproximate k-Distance Plot (k={k}):")
print(f"{'Point Index':>12} | k-Distance")
print(f"{'-'*12}-+-{'-'*40}")
max_dist = k_distances[-1]
for i in range(n_bins):
    idx = (i + 1) * bin_size - 1
    dist = k_distances[idx]
    bar_len = int(dist / max_dist * 35)
    marker = " ← elbow" if abs(idx - elbow_idx) < bin_size else ""
    print(f"{idx:>12} | {'█' * bar_len} {dist:.3f}{marker}")

# For matplotlib visualization:
# import matplotlib.pyplot as plt
# plt.plot(k_distances)
# plt.xlabel('Points (sorted)')
# plt.ylabel(f'{k}-distance')
# plt.axhline(y=optimal_eps, color='r', linestyle='--', label=f'eps={optimal_eps:.3f}')
# plt.title('k-Distance Plot')
# plt.legend()
# plt.show()
```

---

## 3.4 Sensitivity Analysis

### 3.4.1 Varying eps with Fixed min_samples

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.datasets import make_blobs

# Generate data
X, y_true = make_blobs(n_samples=300, centers=3, cluster_std=0.8, random_state=42)
rng = np.random.RandomState(42)
X = np.vstack([X, rng.uniform(-10, 10, size=(30, 2))])  # Add noise

min_samples = 5
eps_values = [0.3, 0.5, 0.8, 1.0, 1.5, 2.0, 3.0, 5.0]

print(f"Sensitivity Analysis: Varying eps (min_samples = {min_samples})")
print(f"{'eps':>6} | {'Clusters':>8} | {'Noise':>6} | {'Core':>6} | {'Border':>6} | Notes")
print(f"{'-'*6}-+-{'-'*8}-+-{'-'*6}-+-{'-'*6}-+-{'-'*6}-+-------")

for eps in eps_values:
    db = DBSCAN(eps=eps, min_samples=min_samples).fit(X)
    labels = db.labels_
    n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
    n_noise = (labels == -1).sum()
    n_core = len(db.core_sample_indices_)
    n_border = len(X) - n_core - n_noise
    
    if n_clusters == 0:
        note = "ALL NOISE"
    elif n_clusters == 1:
        note = "MERGED" if n_noise < 10 else ""
    elif n_clusters == 3:
        note = "← OPTIMAL" if 15 < n_noise < 50 else "← GOOD"
    else:
        note = "fragmented" if n_clusters > 5 else ""
    
    print(f"{eps:>6.1f} | {n_clusters:>8} | {n_noise:>6} | {n_core:>6} | {n_border:>6} | {note}")
```

**Typical Output Pattern:**

```
Sensitivity Analysis: Varying eps (min_samples = 5)
   eps | Clusters |  Noise |   Core | Border | Notes
------+----------+--------+--------+--------+-------
   0.3 |        0 |    330 |      0 |      0 | ALL NOISE
   0.5 |       12 |    210 |     80 |     40 | fragmented
   0.8 |        3 |     35 |    270 |     25 | ← OPTIMAL
   1.0 |        3 |     30 |    285 |     15 | ← GOOD
   1.5 |        2 |     20 |    300 |     10 | clusters merging
   2.0 |        1 |      5 |    320 |      5 | MERGED
   3.0 |        1 |      0 |    330 |      0 | MERGED
   5.0 |        1 |      0 |    330 |      0 | MERGED
```

### 3.4.2 Varying min_samples with Fixed eps

```python
eps = 0.8
min_samples_values = [2, 3, 5, 8, 10, 15, 20, 30]

print(f"\nSensitivity Analysis: Varying min_samples (eps = {eps})")
print(f"{'min_pts':>7} | {'Clusters':>8} | {'Noise':>6} | {'Core':>6} | {'Border':>6}")
print(f"{'-'*7}-+-{'-'*8}-+-{'-'*6}-+-{'-'*6}-+-{'-'*6}")

for ms in min_samples_values:
    db = DBSCAN(eps=eps, min_samples=ms).fit(X)
    labels = db.labels_
    n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
    n_noise = (labels == -1).sum()
    n_core = len(db.core_sample_indices_)
    n_border = len(X) - n_core - n_noise
    
    print(f"{ms:>7} | {n_clusters:>8} | {n_noise:>6} | {n_core:>6} | {n_border:>6}")
```

---

## 3.5 Parameter Interaction Heatmap (Conceptual)

```
                         min_samples
              2      5      10     15     20     30
         ┌──────┬──────┬──────┬──────┬──────┬──────┐
    0.3  │ many │ many │ all  │ all  │ all  │ all  │  ← eps too small
         │ tiny │noise │noise │noise │noise │noise │
         ├──────┼──────┼──────┼──────┼──────┼──────┤
    0.5  │ some │ some │ frag │ all  │ all  │ all  │
         │ good │ good │ ment │noise │noise │noise │
e   ├──────┼──────┼──────┼──────┼──────┼──────┤
p   1.0  │  3   │  3   │  3   │  3   │ frag │ all  │  ← sweet spot
s        │ ✓✓   │ ✓✓✓  │ ✓✓   │ ✓    │      │noise │     range
         ├──────┼──────┼──────┼──────┼──────┼──────┤
    2.0  │  2   │  2   │  3   │  3   │  3   │  3   │
         │merged│merged│ ✓    │ ✓    │ ✓    │ ✓    │
         ├──────┼──────┼──────┼──────┼──────┼──────┤
    5.0  │  1   │  1   │  1   │  1   │  1   │  2   │  ← eps too large
         │merged│merged│merged│merged│merged│      │
         └──────┴──────┴──────┴──────┴──────┴──────┘
```

---

## 3.6 Automated Parameter Selection

### 3.6.1 Grid Search with Silhouette Score

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.metrics import silhouette_score
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_blobs

# Generate and scale data
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.7, random_state=42)
X = StandardScaler().fit_transform(X)

# Grid search
eps_range = np.arange(0.2, 2.0, 0.1)
min_samples_range = range(3, 15)

best_score = -1
best_params = {}
results = []

for eps in eps_range:
    for ms in min_samples_range:
        db = DBSCAN(eps=eps, min_samples=ms).fit(X)
        labels = db.labels_
        
        n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
        n_noise = (labels == -1).sum()
        
        # Need at least 2 clusters and not all noise
        if n_clusters >= 2 and n_noise < len(X) * 0.5:
            # Silhouette score excludes noise points
            non_noise = labels != -1
            if len(set(labels[non_noise])) >= 2:
                score = silhouette_score(X[non_noise], labels[non_noise])
                results.append((eps, ms, n_clusters, n_noise, score))
                
                if score > best_score:
                    best_score = score
                    best_params = {'eps': eps, 'min_samples': ms,
                                   'n_clusters': n_clusters, 'n_noise': n_noise}

print(f"Best parameters found:")
print(f"  eps = {best_params['eps']:.2f}")
print(f"  min_samples = {best_params['min_samples']}")
print(f"  Clusters = {best_params['n_clusters']}")
print(f"  Noise points = {best_params['n_noise']}")
print(f"  Silhouette score = {best_score:.4f}")
```

### 3.6.2 Using the Knee/Elbow Detector Library

```python
# Install: pip install kneed
from kneed import KneeLocator
from sklearn.neighbors import NearestNeighbors
import numpy as np

def find_optimal_eps(X, min_samples=5):
    """Automatically find optimal eps using knee detection."""
    nn = NearestNeighbors(n_neighbors=min_samples)
    nn.fit(X)
    distances, _ = nn.kneighbors(X)
    k_distances = np.sort(distances[:, min_samples - 1])
    
    kneedle = KneeLocator(
        range(len(k_distances)), 
        k_distances,
        curve='convex', 
        direction='increasing',
        interp_method='polynomial'
    )
    
    optimal_eps = k_distances[kneedle.knee] if kneedle.knee else np.median(k_distances)
    return optimal_eps

# Usage:
# optimal_eps = find_optimal_eps(X, min_samples=5)
# db = DBSCAN(eps=optimal_eps, min_samples=5).fit(X)
```

---

## 3.7 Common Pitfalls and Solutions

### Pitfall 1: Forgetting to Scale Data

```python
# BAD: Features on different scales
# Feature 1: age (20-80)
# Feature 2: salary (30000-200000)
# eps = 5 means very different things for each feature!

from sklearn.preprocessing import StandardScaler

# GOOD: Always scale before DBSCAN
X_scaled = StandardScaler().fit_transform(X)
db = DBSCAN(eps=0.5, min_samples=5).fit(X_scaled)
```

### Pitfall 2: Using Euclidean Distance for High Dimensions

```
In high dimensions, distances concentrate (curse of dimensionality):

  Dimension  |  max_dist / min_dist ratio
  ──────────┼──────────────────────────
      2      |  varies widely
     10      |  distances start converging
     50      |  most points equidistant
    100      |  nearly all points same distance
    
  → eps becomes meaningless in high dimensions
  → Consider dimensionality reduction (PCA) before DBSCAN
```

### Pitfall 3: Varying Density Clusters

```
  Dense cluster          Sparse cluster
  eps = 0.3 works        eps = 0.3 → all noise
  eps = 1.0 → merged     eps = 1.0 works

  · · · · · · ·              ·       ·
  · · · · · · ·                  ·
  · · · · · · ·            ·       ·
  · · · · · · ·                ·
  · · · · · · ·              ·     ·

  No single eps works for both! → Use HDBSCAN instead
```

---

## 3.8 Distance Metrics

DBSCAN supports various distance metrics via sklearn:

```python
from sklearn.cluster import DBSCAN

# Euclidean (default)
db = DBSCAN(eps=0.5, min_samples=5, metric='euclidean')

# Manhattan (L1) — better for grid-like data
db = DBSCAN(eps=0.5, min_samples=5, metric='manhattan')

# Cosine — for text/direction data (use eps close to 0)
db = DBSCAN(eps=0.1, min_samples=5, metric='cosine')

# Precomputed distance matrix
from sklearn.metrics import pairwise_distances
dist_matrix = pairwise_distances(X, metric='euclidean')
db = DBSCAN(eps=0.5, min_samples=5, metric='precomputed')
db.fit(dist_matrix)

# Haversine — for geographic coordinates (lat/lon in radians)
# eps in radians! (e.g., 0.5 km ≈ 0.5/6371 radians)
db = DBSCAN(eps=0.5/6371, min_samples=5, metric='haversine')
```

### Distance Metric Selection Guide

| Data Type | Recommended Metric | eps Scale |
|-----------|-------------------|-----------|
| Continuous, scaled | Euclidean | Based on k-distance plot |
| Grid / city-block | Manhattan | Slightly larger than Euclidean |
| Text / TF-IDF | Cosine | 0 to 1 (small values) |
| Geographic (lat/lon) | Haversine | Radians (km/6371) |
| Binary features | Jaccard | 0 to 1 |
| Mixed types | Gower | 0 to 1 |

---

## 3.9 Summary Table

| Parameter | Symbol | Controls | Too Small | Too Large | Selection Method |
|-----------|--------|----------|-----------|-----------|-----------------|
| **eps** | ε | Neighborhood radius | All noise, fragmented | One big cluster | k-distance plot (elbow) |
| **min_samples** | MinPts | Core point threshold | Spurious clusters, noise included | Real clusters → noise | Rule: ≥ D+1 or 2×D |
| **metric** | — | Distance function | — | — | Domain knowledge |

### Quick Reference Decision Guide

```
┌─────────────────────────────────────────────────────────┐
│  DBSCAN Parameter Selection Checklist:                  │
│                                                         │
│  1. □ Scale your data (StandardScaler)                  │
│  2. □ Set min_samples = 2 × n_features (start point)   │
│  3. □ Plot k-distance graph (k = min_samples)           │
│  4. □ Pick eps at the elbow                             │
│  5. □ Run DBSCAN, check results                         │
│  6. □ Adjust: too many clusters? → increase eps         │
│  7. □ Adjust: too few clusters? → decrease eps          │
│  8. □ Adjust: too much noise? → decrease min_samples    │
│  9. □ Adjust: noise in clusters? → increase min_samples │
│ 10. □ If varying density → switch to HDBSCAN            │
└─────────────────────────────────────────────────────────┘
```

---

## 3.10 Revision Questions

1. **Explain what the eps parameter controls** in DBSCAN. What happens to the clustering result if eps is set too small? Too large?

2. **Describe the k-distance plot method** for selecting eps. Why does the elbow in this plot correspond to a good eps value? What value of k should you use?

3. **What is the rule of thumb for min_samples?** How does it relate to the dimensionality of the data? Why should you increase min_samples for noisy datasets?

4. **You have a dataset with features on vastly different scales** (e.g., age 0–100, income 0–1,000,000). Why is it critical to scale the data before applying DBSCAN? What would happen if you didn't?

5. **Compare the effect of increasing eps vs. decreasing min_samples.** Both tend to produce larger clusters — how do they differ in their mechanism?

6. **Why does DBSCAN struggle with clusters of varying density?** Illustrate with an example where no single eps value works. What alternative would you suggest?

---

[← Previous: Core, Border & Noise Points](./02-core-border-noise-points.md) | [Next: Algorithm Steps →](./04-algorithm-steps.md)
