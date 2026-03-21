# 6. HDBSCAN — Hierarchical Density-Based Clustering

[← Previous: Advantages over K-Means](./05-advantages-over-kmeans.md) | [Back to DBSCAN Unit →](./README.md)

---

## Chapter Overview

DBSCAN's biggest limitation is that a single `eps` value cannot handle clusters of **varying density**. **HDBSCAN** (Hierarchical DBSCAN) solves this by building a hierarchy of clusterings at all density levels and then extracting the most stable clusters from that hierarchy. It requires essentially **only one parameter** (`min_cluster_size`) and automatically determines the number of clusters — even when they have different densities.

---

## 6.1 The Varying Density Problem

### Why DBSCAN Fails

```
  Dense cluster (needs small eps)     Sparse cluster (needs large eps)
  ····················                  ·       ·
  ····················                      ·
  ····················                  ·       ·
  ····················                      ·
  ····················                  ·       ·

  eps = 0.2: Dense cluster found ✓    Sparse cluster = ALL NOISE ✗
  eps = 1.5: Sparse cluster found ✓   Dense cluster MERGED with others ✗

  No single eps works for both!
```

### HDBSCAN's Solution

```
HDBSCAN approach: "What if we ran DBSCAN at ALL possible eps values
and then picked the best clustering from each density level?"

  eps = 0.1   →  only densest cores found
  eps = 0.3   →  dense clusters emerge
  eps = 0.5   →  medium density clusters emerge
  eps = 1.0   →  sparse clusters emerge
  eps = 2.0   →  clusters start merging
  eps = 5.0   →  everything is one cluster

  HDBSCAN builds this full hierarchy, then selects
  the most "persistent" (stable) clusters across scales.
```

---

## 6.2 HDBSCAN Core Concepts

### 6.2.1 Core Distance

The **core distance** of a point p is the distance to its `min_samples`-th nearest neighbor.

```
core_dist(p) = dist(p, k-th nearest neighbor)

where k = min_samples

Intuition: core distance measures the LOCAL density around p.
  - Small core distance → dense neighborhood
  - Large core distance → sparse neighborhood

Example (min_samples = 4):
                                    4th nearest
  Point   1st   2nd   3rd   4th    = core_dist
  ─────────────────────────────────────────────
  p₁      0.1   0.2   0.3   0.4    0.4  (dense)
  p₂      0.5   0.8   1.2   1.5    1.5  (medium)
  p₃      2.0   3.0   4.0   5.0    5.0  (sparse)
```

### 6.2.2 Mutual Reachability Distance

The **mutual reachability distance** between two points p and q is:

```
d_mreach(p, q) = max( core_dist(p), core_dist(q), dist(p, q) )
```

```
  ┌───────────────────────────────────────────────────────────┐
  │                                                           │
  │  d_mreach(p, q) = max of three values:                   │
  │                                                           │
  │    1. core_dist(p)  — how sparse is p's neighborhood     │
  │    2. core_dist(q)  — how sparse is q's neighborhood     │
  │    3. dist(p, q)    — actual distance between p and q    │
  │                                                           │
  │  This "inflates" distances in sparse regions,             │
  │  making sparse points appear farther apart than they      │
  │  actually are. Dense regions are unaffected.              │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

**Why Mutual Reachability Distance Works:**

```
Dense region:                    Sparse region:
  core_dist(p) = 0.1              core_dist(p) = 2.0
  core_dist(q) = 0.1              core_dist(q) = 2.5
  dist(p,q)    = 0.3              dist(p,q)    = 0.5

  d_mreach = max(0.1, 0.1, 0.3)  d_mreach = max(2.0, 2.5, 0.5)
           = 0.3 (real distance)           = 2.5 (inflated!)

In dense regions: mutual reachability ≈ actual distance
In sparse regions: mutual reachability >> actual distance

This allows dense clusters to form at low thresholds
while sparse clusters need higher thresholds.
```

### Worked Example: Mutual Reachability

```
5 points, min_samples = 3:

Points: A(0,0), B(0.5,0), C(1,0), D(5,0), E(5.5,0)

Step 1: Compute core distances (distance to 3rd nearest neighbor)
  core_dist(A) = dist(A, C) = 1.0      (neighbors: B=0.5, C=1.0, D=5.0)
  core_dist(B) = dist(B, D) = 4.5      ... wait, let's sort properly
  
  Actually, including self as a point for kNN:
  A's neighbors sorted: B(0.5), C(1.0), D(5.0), E(5.5)
  core_dist(A) = 1.0  (3rd nearest = C, since min_samples=3 means 3-1=2nd index)
  
  Let's use min_samples = 3, so k = 3 (3rd nearest neighbor including self):
  A: self=0, B=0.5, C=1.0 → core_dist = 1.0
  B: self=0, A=0.5, C=0.5 → core_dist = 0.5
  C: self=0, B=0.5, A=1.0 → core_dist = 1.0
  D: self=0, E=0.5, C=4.0 → core_dist = 4.0
  E: self=0, D=0.5, C=4.5 → core_dist = 4.5

Step 2: Mutual reachability distance matrix
  d_mreach(A,B) = max(1.0, 0.5, 0.5) = 1.0
  d_mreach(A,C) = max(1.0, 1.0, 1.0) = 1.0
  d_mreach(B,C) = max(0.5, 1.0, 0.5) = 1.0
  d_mreach(A,D) = max(1.0, 4.0, 5.0) = 5.0
  d_mreach(B,D) = max(0.5, 4.0, 4.5) = 4.5
  d_mreach(C,D) = max(1.0, 4.0, 4.0) = 4.0
  d_mreach(D,E) = max(4.0, 4.5, 0.5) = 4.5
```

---

## 6.3 The HDBSCAN Algorithm

### Step-by-Step Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│  HDBSCAN Algorithm:                                                  │
│                                                                      │
│  Step 1: Compute core distances for all points                      │
│          (using min_samples parameter)                              │
│                                                                      │
│  Step 2: Build mutual reachability distance graph                   │
│          (complete graph with d_mreach as edge weights)             │
│                                                                      │
│  Step 3: Build Minimum Spanning Tree (MST)                          │
│          (Prim's or Kruskal's algorithm on d_mreach graph)          │
│                                                                      │
│  Step 4: Build cluster hierarchy (dendrogram)                       │
│          (sort MST edges by weight, merge connected components)      │
│                                                                      │
│  Step 5: Condense the cluster tree                                  │
│          (prune branches smaller than min_cluster_size)              │
│                                                                      │
│  Step 6: Extract stable clusters                                    │
│          (select clusters that persist longest in the hierarchy)     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Step 3: Minimum Spanning Tree

```
From the mutual reachability distances, build an MST:

  A ──1.0── B ──1.0── C ──4.0── D ──4.5── E
  
  (MST connects all points with minimum total weight)
```

### Step 4: Cluster Hierarchy (Dendrogram)

```
Distance (1/density)
  │
  5.0│         ┌──────────────┐
     │         │              │
  4.5│    ┌────┤         ┌────┤
     │    │    │         │    │
  4.0│    │    │    ┌────┤    │
     │    │    │    │    │    │
  1.0│ ┌──┤    │    │    │    │
     │ │  │    │    │    │    │
  0.0│ A  B    C    D    E    │
     └────┴────┴────┴────┴────┘

  Reading bottom-up:
  - At distance 1.0: A,B,C merge into cluster {A,B,C}
  - At distance 4.0: C connects to D
  - At distance 4.5: D connects to E, everything merges
```

### Step 5: Condensed Tree

The condensed tree prunes small clusters (< `min_cluster_size`).

```
Lambda (1/distance = density)
  │
  1.0│  ┌──────────────────┐
     │  │  Cluster {A,B,C} │     ← persists from λ=1.0 down to λ=0.22
     │  │                  │        Stability = area under curve
  0.5│  │                  │
     │  │                  │
  0.22│ └────────┬─────────┘     ← {A,B,C} merges with {D,E}
     │           │
  0.0│           ● root
     └───────────────────────

  Cluster {A,B,C}: stability = Σ (λ - λ_birth) = high
  Cluster {D,E}: stability = Σ (λ - λ_birth) = lower
  Merged {A,B,C,D,E}: stability = medium
  
  If Σ child stabilities > parent stability → keep children
  Otherwise → keep parent
```

### Step 6: Cluster Persistence and Stability

```
Stability of a cluster C:

  stability(C) = Σ   (λ_p - λ_birth(C))
                p∈C

  where:
    λ_p = the lambda value at which point p "falls out" of C
          (or the lambda when C merges with another cluster)
    λ_birth(C) = the lambda at which cluster C first appears
    λ = 1/distance (higher λ = higher density)

Selection rule:
  For each node in the condensed tree:
    IF sum(child stabilities) > parent stability:
       → select children as clusters (split)
    ELSE:
       → select parent as one cluster (merge)
```

---

## 6.4 HDBSCAN vs. DBSCAN

| Feature | DBSCAN | HDBSCAN |
|---------|--------|---------|
| **Parameters** | eps, min_samples | min_cluster_size (primary) |
| **Varying density** | ❌ Single density threshold | ✅ Handles varying density |
| **Number of clusters** | Auto (depends on eps) | Auto (from stability) |
| **Noise detection** | ✅ Yes | ✅ Yes (often better) |
| **Soft clustering** | ❌ Hard only | ✅ Probability scores |
| **Hierarchy** | No | Full dendrogram + condensed tree |
| **Time complexity** | O(n log n) to O(n²) | O(n log n) with optimizations |
| **Robustness to params** | Sensitive to eps | Much more robust |
| **Outlier scores** | Binary (noise or not) | Continuous outlier scores |

---

## 6.5 HDBSCAN Parameters

```
┌────────────────────────────────────────────────────────────────────┐
│  min_cluster_size (PRIMARY — the only one you usually need)       │
│  ─────────────────                                                │
│  Minimum number of points for a group to be considered a cluster. │
│  Smaller → more small clusters found                              │
│  Larger → only large, well-defined clusters                       │
│  Default: 5. Good starting point for most datasets.               │
│                                                                   │
│  min_samples (SECONDARY — controls density estimation)            │
│  ───────────                                                      │
│  Controls how conservative the clustering is.                     │
│  Defaults to min_cluster_size if not specified.                   │
│  Larger → more points classified as noise                         │
│  Smaller → less noise, more aggressive clustering                 │
│                                                                   │
│  cluster_selection_epsilon                                        │
│  ────────────────────────                                         │
│  Merge clusters closer than this distance. Helps prevent          │
│  over-splitting. Similar to DBSCAN's eps but optional.            │
│                                                                   │
│  cluster_selection_method                                         │
│  ────────────────────────                                         │
│  'eom' = Excess of Mass (default, best for varying density)       │
│  'leaf' = Leaf clusters (similar to flat DBSCAN cut)              │
│                                                                   │
└────────────────────────────────────────────────────────────────────┘
```

---

## 6.6 Python Implementation with hdbscan Library

### Installation

```bash
pip install hdbscan
```

### Basic Usage

```python
import numpy as np
import hdbscan
from sklearn.datasets import make_moons, make_blobs
from sklearn.preprocessing import StandardScaler

# Create varying density data
X1, _ = make_blobs(n_samples=300, centers=[(0,0)], cluster_std=0.3, random_state=42)
X2, _ = make_blobs(n_samples=100, centers=[(5,5)], cluster_std=1.5, random_state=42)
rng = np.random.RandomState(42)
noise = rng.uniform(-3, 10, size=(30, 2))
X = np.vstack([X1, X2, noise])
X = StandardScaler().fit_transform(X)

# Fit HDBSCAN
clusterer = hdbscan.HDBSCAN(
    min_cluster_size=15,
    min_samples=5,
    cluster_selection_method='eom',   # 'eom' or 'leaf'
    metric='euclidean',
    gen_min_span_tree=True            # Generate MST for visualization
)
labels = clusterer.fit_predict(X)

# Results
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = (labels == -1).sum()

print(f"Clusters found: {n_clusters}")
print(f"Noise points:   {n_noise}")
print(f"Cluster sizes:  {np.bincount(labels[labels >= 0])}")
```

### Probability and Outlier Scores

```python
# Soft clustering: probability of belonging to assigned cluster
probabilities = clusterer.probabilities_
print(f"\nCluster membership probabilities (first 10):")
for i in range(10):
    print(f"  Point {i}: cluster={labels[i]}, probability={probabilities[i]:.3f}")

# Outlier scores (0 = inlier, 1 = strong outlier)
outlier_scores = clusterer.outlier_scores_
print(f"\nOutlier scores (first 10):")
for i in range(10):
    print(f"  Point {i}: outlier_score={outlier_scores[i]:.3f}")

# Points with highest outlier scores
top_outliers = np.argsort(outlier_scores)[-5:]
print(f"\nTop 5 outlier points: {top_outliers}")
print(f"Their scores: {outlier_scores[top_outliers]}")
```

### Comparing DBSCAN vs HDBSCAN on Varying Density

```python
from sklearn.cluster import DBSCAN
from sklearn.metrics import adjusted_rand_score

# Create ground truth varying density data
X1, y1 = make_blobs(200, centers=[(0,0)], cluster_std=0.3, random_state=42)
X2, y2 = make_blobs(100, centers=[(4,4)], cluster_std=1.2, random_state=42)
X = np.vstack([X1, X2])
y_true = np.concatenate([np.zeros(200), np.ones(100)])
X_scaled = StandardScaler().fit_transform(X)

print("DBSCAN at different eps values:")
for eps in [0.2, 0.3, 0.5, 0.8, 1.0, 1.5]:
    db = DBSCAN(eps=eps, min_samples=5).fit(X_scaled)
    n_cl = len(set(db.labels_)) - (1 if -1 in db.labels_ else 0)
    n_noise = (db.labels_ == -1).sum()
    ari = adjusted_rand_score(y_true, db.labels_)
    print(f"  eps={eps:.1f}: {n_cl} clusters, {n_noise} noise, ARI={ari:.3f}")

print("\nHDBSCAN:")
hdb = hdbscan.HDBSCAN(min_cluster_size=15, min_samples=5)
hdb_labels = hdb.fit_predict(X_scaled)
n_cl = len(set(hdb_labels)) - (1 if -1 in hdb_labels else 0)
n_noise = (hdb_labels == -1).sum()
ari = adjusted_rand_score(y_true, hdb_labels)
print(f"  {n_cl} clusters, {n_noise} noise, ARI={ari:.3f}")
print(f"  (No eps needed!)")
```

**Expected Output Pattern:**
```
DBSCAN at different eps values:
  eps=0.2: 1 clusters, 180 noise, ARI=0.150   ← sparse cluster lost
  eps=0.3: 2 clusters, 95 noise, ARI=0.450    ← some improvement
  eps=0.5: 2 clusters, 20 noise, ARI=0.750    ← better but not great
  eps=0.8: 1 clusters, 5 noise, ARI=0.200     ← clusters merged!
  eps=1.0: 1 clusters, 0 noise, ARI=0.000     ← everything one cluster
  eps=1.5: 1 clusters, 0 noise, ARI=0.000

HDBSCAN:
  2 clusters, 8 noise, ARI=0.950
  (No eps needed!)
```

### Condensed Tree Visualization

```python
# Visualize the condensed tree (requires matplotlib)
# clusterer.condensed_tree_.plot()

# Access condensed tree data programmatically
tree = clusterer.condensed_tree_
print(f"\nCondensed tree structure:")
print(f"  Tree DataFrame shape: {tree.to_pandas().shape}")
print(f"  Columns: {list(tree.to_pandas().columns)}")
print(tree.to_pandas().head(10))
```

---

## 6.7 Advanced HDBSCAN Features

### Approximate Predict (for new points)

```python
# HDBSCAN can predict cluster labels for new, unseen points
# (approximate, based on the learned hierarchy)

from hdbscan import approximate_predict

new_points = np.array([[0, 0], [4, 4], [10, 10]])
new_points_scaled = StandardScaler().fit_transform(
    np.vstack([X, new_points])
)[-3:]  # Only take the new points after scaling

# Note: approximate_predict requires the clusterer to be fit
# labels_new, strengths_new = approximate_predict(clusterer, new_points_scaled)
# print(f"New point predictions: {labels_new}")
# print(f"Prediction strengths: {strengths_new}")
```

### HDBSCAN with sklearn API (scikit-learn >= 1.3)

```python
# Starting from scikit-learn 1.3, HDBSCAN is built-in!
from sklearn.cluster import HDBSCAN as SklearnHDBSCAN

hdb_sklearn = SklearnHDBSCAN(
    min_cluster_size=15,
    min_samples=5,
    cluster_selection_method='eom',
    store_centers='centroid'  # Also 'medoid' or None
)
labels = hdb_sklearn.fit_predict(X_scaled)

print(f"sklearn HDBSCAN clusters: {len(set(labels)) - (1 if -1 in labels else 0)}")
# Access centroids (if store_centers was set)
# print(f"Centroids: {hdb_sklearn.centroids_}")
```

---

## 6.8 When to Use HDBSCAN vs. DBSCAN

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Use DBSCAN when:                    Use HDBSCAN when:          │
│  ─────────────────                   ──────────────────         │
│  • Clusters have UNIFORM density     • Varying density clusters │
│  • You have a good eps value         • Don't want to tune eps   │
│  • Speed is critical (huge data)     • Need outlier scores      │
│  • Simple, well-understood needed    • Need soft assignments    │
│  • sklearn-only environment          • More robust results      │
│                                                                 │
│  In general: HDBSCAN is almost always equal or better          │
│  than DBSCAN. Use it as your default density-based method.     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6.9 Summary Table

| Concept | Description | Key Formula/Detail |
|---------|-------------|-------------------|
| **Core distance** | Distance to k-th nearest neighbor | Measures local density |
| **Mutual reachability** | max(core_dist(p), core_dist(q), dist(p,q)) | Inflates distances in sparse regions |
| **MST** | Minimum spanning tree on mutual reachability | Captures density hierarchy |
| **Condensed tree** | Hierarchy with small branches pruned | min_cluster_size controls pruning |
| **Stability** | Σ(λ_p - λ_birth) for points in cluster | Higher = more persistent cluster |
| **Cluster selection** | Keep children if their total stability > parent | EOM method (default) |
| **Outlier scores** | Continuous score from 0 (inlier) to 1 (outlier) | Better than binary noise labels |
| **Probabilities** | Soft membership probability per point | Uncertainty quantification |

---

## 6.10 Revision Questions

1. **Explain why DBSCAN fails on varying-density clusters.** Give a specific example with two clusters at different densities and show why no single eps value works.

2. **Define mutual reachability distance** and explain why it helps with varying density. How does it differ from regular Euclidean distance in dense vs. sparse regions?

3. **Describe the condensed tree** in HDBSCAN. How is it constructed from the full hierarchy, and how does the stability-based selection choose the final clusters?

4. **Compare HDBSCAN's parameters** (min_cluster_size, min_samples) with DBSCAN's parameters (eps, min_samples). Which algorithm is easier to tune and why?

5. **What are outlier scores and cluster probabilities** in HDBSCAN? How do they provide more information than DBSCAN's binary noise/cluster classification?

6. **When would you still prefer DBSCAN over HDBSCAN?** Give specific scenarios where DBSCAN's simplicity or behavior is advantageous.

---

[← Previous: Advantages over K-Means](./05-advantages-over-kmeans.md) | [Back to DBSCAN Unit →](./README.md)
