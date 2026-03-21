# 4. DBSCAN Algorithm Steps

[← Previous: Parameters eps & min_samples](./03-parameters-eps-minsamples.md) | [Next: Advantages over K-Means →](./05-advantages-over-kmeans.md)

---

## Chapter Overview

This chapter presents the complete DBSCAN algorithm: formal pseudocode, a detailed step-by-step walkthrough with a numerical example, the cluster expansion mechanism, time and space complexity analysis, and a full sklearn implementation. By the end, you will be able to trace DBSCAN's execution by hand and implement it from scratch.

---

## 4.1 DBSCAN Pseudocode

### High-Level Algorithm

```
DBSCAN(D, eps, min_samples):
  ┌──────────────────────────────────────────────────────────────┐
  │  Input:  D = dataset of n points                            │
  │          eps = neighborhood radius                           │
  │          min_samples = minimum density threshold             │
  │  Output: Cluster labels for each point (-1 = noise)         │
  │                                                              │
  │  1. Initialize all points as UNVISITED                       │
  │  2. cluster_id = 0                                           │
  │  3. FOR each point p in D:                                   │
  │  │    IF p is VISITED: continue                              │
  │  │    Mark p as VISITED                                      │
  │  │    neighbors = RangeQuery(D, p, eps)                      │
  │  │    IF |neighbors| < min_samples:                          │
  │  │    │    Mark p as NOISE (may change later to border)      │
  │  │    ELSE:                                                  │
  │  │    │    cluster_id += 1                                   │
  │  │    │    ExpandCluster(p, neighbors, cluster_id, eps,      │
  │  │    │                  min_samples)                        │
  │  4. RETURN labels                                            │
  └──────────────────────────────────────────────────────────────┘
```

### ExpandCluster Subroutine

```
ExpandCluster(p, neighbors, cluster_id, eps, min_samples):
  ┌──────────────────────────────────────────────────────────────┐
  │  1. Assign p to cluster_id                                  │
  │  2. seed_set = neighbors (queue of points to process)       │
  │  3. FOR each point q in seed_set:                           │
  │  │    IF q was marked as NOISE:                              │
  │  │    │    Assign q to cluster_id (border point)            │
  │  │    IF q is UNVISITED:                                     │
  │  │    │    Mark q as VISITED                                 │
  │  │    │    q_neighbors = RangeQuery(D, q, eps)              │
  │  │    │    IF |q_neighbors| ≥ min_samples:                  │
  │  │    │    │    seed_set = seed_set ∪ q_neighbors           │
  │  │    │    │    (q is core → expand further)                │
  │  │    │    Assign q to cluster_id                           │
  └──────────────────────────────────────────────────────────────┘
```

### RangeQuery

```
RangeQuery(D, p, eps):
  RETURN { q ∈ D : dist(p, q) ≤ eps }
```

---

## 4.2 Algorithm Flow Diagram

```
          START
            │
            ▼
    ┌───────────────┐
    │ Pick unvisited │
    │    point p     │◄──────────────────────────────────┐
    └───────┬───────┘                                    │
            │                                            │
            ▼                                            │
    ┌───────────────┐                                    │
    │ Mark p VISITED │                                   │
    └───────┬───────┘                                    │
            │                                            │
            ▼                                            │
    ┌───────────────┐     YES    ┌───────────────┐       │
    │ |N_eps(p)| ≥  │──────────►│ New cluster_id │       │
    │ min_samples?  │           │ cluster_id++   │       │
    └───────┬───────┘           └───────┬───────┘       │
            │ NO                        │                │
            ▼                           ▼                │
    ┌───────────────┐          ┌─────────────────┐       │
    │ Mark p NOISE  │          │ ExpandCluster() │       │
    │ (tentative)   │          │ BFS/DFS from p  │       │
    └───────┬───────┘          └───────┬─────────┘       │
            │                          │                  │
            ▼                          ▼                  │
    ┌────────────────┐         ┌────────────────┐         │
    │ More unvisited │   YES   │ More unvisited │   YES   │
    │   points?     │────────►│   points?     │─────────┘
    └───────┬────────┘         └───────┬────────┘
            │ NO                       │ NO
            ▼                          ▼
          DONE                       DONE
```

---

## 4.3 Step-by-Step Numerical Walkthrough

### Dataset

```
10 points, eps = 1.5, min_samples = 3

Points:
  A(1, 1)    B(1.5, 1.2)   C(1, 2)     D(2, 1.5)    E(1.8, 2.2)
  F(6, 6)    G(6.5, 6.2)   H(6, 7)     I(4, 4)      J(10, 10)

Plot:
  10│                                         J
    │
   8│
   7│                    H
   6│                    F  G
    │
   4│              I
    │
   2│  C  E
  1 │  A  B  D
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──
       1  2  3  4  5  6  7  8  9  10
```

### Distance Matrix (relevant pairs)

| Pair | Distance | Within eps=1.5? |
|------|----------|-----------------|
| A-B | √(0.25+0.04) = 0.54 | ✅ |
| A-C | √(0+1) = 1.00 | ✅ |
| A-D | √(1+0.25) = 1.12 | ✅ |
| A-E | √(0.64+1.44) = 1.44 | ✅ |
| B-C | √(0.25+0.64) = 0.94 | ✅ |
| B-D | √(0.25+0.09) = 0.58 | ✅ |
| B-E | √(0.09+1.00) = 1.04 | ✅ |
| C-D | √(1+0.25) = 1.12 | ✅ |
| C-E | √(0.64+0.04) = 0.82 | ✅ |
| D-E | √(0.04+0.49) = 0.73 | ✅ |
| F-G | √(0.25+0.04) = 0.54 | ✅ |
| F-H | √(0+1) = 1.00 | ✅ |
| G-H | √(0.25+0.64) = 0.94 | ✅ |
| I-* | I is far from all clusters | ❌ |
| J-* | J is far from everything | ❌ |

### Execution Trace

```
Initialization: All points UNVISITED, cluster_id = 0

─── Iteration 1: Process A(1,1) ───────────────────────────
  Mark A as VISITED
  N_eps(A) = {A, B, C, D, E}  →  |N| = 5 ≥ 3  →  CORE POINT!
  cluster_id = 1
  ExpandCluster(A, {A,B,C,D,E}, cluster_id=1):
    Assign A → Cluster 1
    seed_set = {B, C, D, E}

    Process B: UNVISITED → VISITED
      N_eps(B) = {A, B, C, D, E}  →  |N| = 5 ≥ 3  →  CORE
      seed_set = seed_set ∪ {A,B,C,D,E} = {B,C,D,E} (no new)
      Assign B → Cluster 1

    Process C: UNVISITED → VISITED
      N_eps(C) = {A, B, C, D, E}  →  |N| = 5 ≥ 3  →  CORE
      seed_set unchanged (all already in set)
      Assign C → Cluster 1

    Process D: UNVISITED → VISITED
      N_eps(D) = {A, B, C, D, E}  →  |N| = 5 ≥ 3  →  CORE
      seed_set unchanged
      Assign D → Cluster 1

    Process E: UNVISITED → VISITED
      N_eps(E) = {A, B, C, D, E}  →  |N| = 5 ≥ 3  →  CORE
      seed_set unchanged
      Assign E → Cluster 1

  Cluster 1 complete: {A, B, C, D, E}

─── Iteration 2: Process F(6,6) ───────────────────────────
  (B, C, D, E already VISITED — skip)
  Mark F as VISITED
  N_eps(F) = {F, G, H}  →  |N| = 3 ≥ 3  →  CORE POINT!
  cluster_id = 2
  ExpandCluster(F, {F,G,H}, cluster_id=2):
    Assign F → Cluster 2
    seed_set = {G, H}

    Process G: UNVISITED → VISITED
      N_eps(G) = {F, G, H}  →  |N| = 3 ≥ 3  →  CORE
      seed_set unchanged
      Assign G → Cluster 2

    Process H: UNVISITED → VISITED
      N_eps(H) = {F, G, H}  →  |N| = 3 ≥ 3  →  CORE
      seed_set unchanged
      Assign H → Cluster 2

  Cluster 2 complete: {F, G, H}

─── Iteration 3: Process I(4,4) ───────────────────────────
  Mark I as VISITED
  N_eps(I) = {I}  →  |N| = 1 < 3  →  NOISE
  Mark I as NOISE (label = -1)

─── Iteration 4: Process J(10,10) ─────────────────────────
  Mark J as VISITED
  N_eps(J) = {J}  →  |N| = 1 < 3  →  NOISE
  Mark J as NOISE (label = -1)

═══ FINAL RESULT ═══════════════════════════════════════════

  Cluster 1: {A, B, C, D, E}    (5 core points)
  Cluster 2: {F, G, H}          (3 core points)
  Noise:     {I, J}             (2 noise points)
```

### Visualization of Result

```
  10│                                         J(✕)
    │
   8│
   7│                    H(★₂)
   6│                    F(★₂) G(★₂)
    │
   4│              I(✕)
    │
   2│  C(★₁) E(★₁)
  1 │  A(★₁) B(★₁) D(★₁)
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──
       1  2  3  4  5  6  7  8  9  10

  ★₁ = Cluster 1 (core)    ★₂ = Cluster 2 (core)    ✕ = Noise
```

---

## 4.4 Cluster Expansion Mechanism (BFS)

The ExpandCluster procedure works like **Breadth-First Search (BFS)** on a density graph:

```
Density Graph (edges connect points within eps that share core status):

  Cluster expansion from core point A:

  Step 1: Start at A (core)
           A ────→ {B, C, D, E}     (A's eps-neighborhood)

  Step 2: B is core → expand
           B ────→ {A, C, D, E}     (already in cluster, no new)

  Step 3: C is core → expand
           C ────→ {A, B, D, E}     (already in cluster, no new)

  Step 4: D is core → expand  
           D ────→ {A, B, C, E}     (already in cluster, no new)

  Step 5: E is core → expand
           E ────→ {A, B, C, D}     (already in cluster, no new)

  seed_set empty → Cluster 1 is complete!


  Cluster expansion reaching a border point:

  ★───★───★───◆         The chain stops at ◆ (border)
  │   │   │              because ◆ is not core and
  ★───★   ★              cannot expand the seed_set.
              ◆          ◆ gets the cluster label
                         but doesn't add new seeds.
```

### Key Insight: Why DBSCAN Finds Arbitrary Shapes

```
Chain of core points forming a crescent:

  ★₁──★₂                        Each core point expands
       ╲                         to its neighbors.
        ★₃                       The chain "walks" along
         ╲                       the dense region,
          ★₄                     regardless of shape.
           ╲                     
            ★₅──★₆              K-Means would place a
                  ╲              centroid in the middle
                   ★₇           of the crescent (sparse
                    │            area!) and split it.
                    ★₈
```

---

## 4.5 Time and Space Complexity

### Time Complexity

| Operation | Without Spatial Index | With Spatial Index (KD-tree/Ball-tree) |
|-----------|----------------------|----------------------------------------|
| **RangeQuery per point** | O(n) | O(log n) |
| **Total RangeQueries** | n queries | n queries |
| **Overall** | **O(n²)** | **O(n log n)** |

```
Complexity Breakdown:

  Without index:
    For each of n points:
      Compare against all n points → O(n)
    Total: O(n) × O(n) = O(n²)

  With KD-tree (low dimensions):
    Build tree: O(n log n)
    For each of n points:
      Range query: O(log n) average
    Total: O(n log n)

  With Ball-tree (higher dimensions):
    Build tree: O(n log n)
    For each of n points:
      Range query: O(log n) average
    Total: O(n log n)

  ⚠️ In very high dimensions, spatial indices degrade to O(n),
     making overall complexity O(n²) regardless.
```

### Space Complexity

```
  O(n) for storing:
    - Labels array: O(n)
    - Visited flags: O(n)
    - Seed set (queue): O(n) worst case
    - Spatial index: O(n)
    
  If using precomputed distance matrix:
    O(n²) — can be prohibitive for large datasets
```

### Comparison with Other Algorithms

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| **DBSCAN** | O(n log n) to O(n²) | O(n) | Depends on index/dimensions |
| **K-Means** | O(nKtd) | O(n) | t = iterations, d = dimensions |
| **Hierarchical** | O(n² log n) | O(n²) | Distance matrix required |
| **HDBSCAN** | O(n log n) | O(n) | Optimized variant |

---

## 4.6 sklearn Implementation

### Basic Usage

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_moons

# Step 1: Generate data
X, y_true = make_moons(n_samples=300, noise=0.08, random_state=42)

# Step 2: Scale the data
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Step 3: Fit DBSCAN
db = DBSCAN(
    eps=0.3,           # Neighborhood radius
    min_samples=5,      # Minimum points for core
    metric='euclidean', # Distance metric
    algorithm='auto',   # 'ball_tree', 'kd_tree', 'brute', or 'auto'
    leaf_size=30,       # Leaf size for tree algorithms
    n_jobs=-1           # Parallel computation (-1 = all cores)
)
db.fit(X_scaled)

# Step 4: Extract results
labels = db.labels_
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = list(labels).count(-1)
core_indices = db.core_sample_indices_
components = db.components_  # Core sample feature matrix

print(f"Clusters: {n_clusters}")
print(f"Noise points: {n_noise}")
print(f"Core points: {len(core_indices)}")
print(f"Labels: {np.unique(labels)}")
```

### Key Attributes After Fitting

```python
# db.labels_               — Cluster label for each point (-1 = noise)
# db.core_sample_indices_  — Indices of core points
# db.components_           — Feature matrix of core points only
# db.n_features_in_        — Number of features seen during fit
```

### Complete Pipeline with Evaluation

```python
import numpy as np
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import NearestNeighbors
from sklearn.metrics import silhouette_score, adjusted_rand_score
from sklearn.datasets import make_moons

# Generate data
X, y_true = make_moons(n_samples=500, noise=0.1, random_state=42)

# Scale
X_scaled = StandardScaler().fit_transform(X)

# Step 1: Find optimal eps via k-distance plot
min_samples = 5
nn = NearestNeighbors(n_neighbors=min_samples)
nn.fit(X_scaled)
distances, _ = nn.kneighbors(X_scaled)
k_distances = np.sort(distances[:, min_samples - 1])

# Simple elbow detection
diffs = np.diff(k_distances)
elbow_idx = np.argmax(np.diff(diffs)) + 2
optimal_eps = k_distances[elbow_idx]
print(f"Optimal eps from k-distance plot: {optimal_eps:.4f}")

# Step 2: Fit DBSCAN
db = DBSCAN(eps=optimal_eps, min_samples=min_samples, n_jobs=-1)
labels = db.fit_predict(X_scaled)

# Step 3: Evaluate
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = (labels == -1).sum()

print(f"\nResults:")
print(f"  Clusters found: {n_clusters}")
print(f"  Noise points:   {n_noise} ({100*n_noise/len(X):.1f}%)")
print(f"  Core points:    {len(db.core_sample_indices_)}")

# Silhouette (excluding noise)
if n_clusters >= 2:
    non_noise = labels != -1
    sil = silhouette_score(X_scaled[non_noise], labels[non_noise])
    print(f"  Silhouette:     {sil:.4f}")

# ARI (if true labels available)
ari = adjusted_rand_score(y_true, labels)
print(f"  Adjusted Rand:  {ari:.4f}")
```

### Using Precomputed Distance Matrix

```python
from sklearn.metrics import pairwise_distances

# Compute full distance matrix
dist_matrix = pairwise_distances(X_scaled, metric='euclidean')

# Fit with precomputed distances
db = DBSCAN(eps=0.5, min_samples=5, metric='precomputed')
labels = db.fit_predict(dist_matrix)
```

---

## 4.7 From-Scratch Implementation

```python
import numpy as np
from collections import deque

def dbscan_from_scratch(X, eps, min_samples):
    """
    DBSCAN implementation from scratch.
    
    Parameters:
        X: numpy array of shape (n_samples, n_features)
        eps: neighborhood radius
        min_samples: minimum points for core
    
    Returns:
        labels: array of cluster labels (-1 = noise)
    """
    n = len(X)
    labels = np.full(n, -1)  # Initialize all as noise (-1)
    visited = np.zeros(n, dtype=bool)
    cluster_id = 0
    
    def range_query(point_idx):
        """Find all points within eps of point_idx."""
        distances = np.sqrt(np.sum((X - X[point_idx]) ** 2, axis=1))
        return np.where(distances <= eps)[0]
    
    for i in range(n):
        if visited[i]:
            continue
        
        visited[i] = True
        neighbors = range_query(i)
        
        if len(neighbors) < min_samples:
            # Tentatively mark as noise (may become border later)
            continue
        
        # Core point found — start new cluster
        labels[i] = cluster_id
        
        # Expand cluster using BFS
        seed_queue = deque(neighbors[neighbors != i])
        
        while seed_queue:
            q = seed_queue.popleft()
            
            if labels[q] == -1:
                # Was noise → now border point
                labels[q] = cluster_id
            
            if visited[q]:
                continue
            
            visited[q] = True
            labels[q] = cluster_id
            
            q_neighbors = range_query(q)
            
            if len(q_neighbors) >= min_samples:
                # q is also core → add its neighbors to expansion
                for neighbor in q_neighbors:
                    if not visited[neighbor]:
                        seed_queue.append(neighbor)
        
        cluster_id += 1
    
    return labels

# Test
from sklearn.datasets import make_moons
X, y_true = make_moons(n_samples=200, noise=0.08, random_state=42)

labels = dbscan_from_scratch(X, eps=0.3, min_samples=5)
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = (labels == -1).sum()

print(f"From-scratch DBSCAN: {n_clusters} clusters, {n_noise} noise points")
```

---

## 4.8 Summary Table

| Step | Action | Details |
|------|--------|---------|
| **1. Initialize** | Mark all points unvisited | labels = undefined |
| **2. Pick point** | Select next unvisited point p | Sequential scan |
| **3. Range query** | Find N_eps(p) | All points within eps of p |
| **4. Check density** | \|N_eps(p)\| ≥ min_samples? | Core point test |
| **5a. If sparse** | Mark p as noise | May become border later |
| **5b. If dense** | Create new cluster, expand | BFS from core point |
| **6. Expand** | Process each neighbor q | If q is core → add q's neighbors |
| **7. Border rescue** | Noise → border if reached | Relabel from -1 to cluster_id |
| **8. Repeat** | Until all points visited | Deterministic (except border ties) |

---

## 4.9 Revision Questions

1. **Write the pseudocode** for DBSCAN's ExpandCluster procedure. Explain how it handles (a) noise points that should become border points, and (b) points that are already assigned to a cluster.

2. **Trace the DBSCAN algorithm** on the following 7 points with eps=2.0 and min_samples=3: P1(0,0), P2(1,0), P3(0,1), P4(5,5), P5(6,5), P6(5,6), P7(3,3). Show the state after each iteration.

3. **What is the time complexity** of DBSCAN with and without a spatial index? Under what conditions does a KD-tree degrade to O(n) per query?

4. **Explain why the ExpandCluster procedure** is similar to BFS. How does the "seed set" act like a BFS queue?

5. **In the sklearn implementation**, what are the roles of `algorithm='auto'`, `leaf_size`, and `n_jobs` parameters? When would you choose `metric='precomputed'`?

6. **Can the order in which points are processed** affect DBSCAN's output? If so, which aspects are affected and which are deterministic?

---

[← Previous: Parameters eps & min_samples](./03-parameters-eps-minsamples.md) | [Next: Advantages over K-Means →](./05-advantages-over-kmeans.md)
