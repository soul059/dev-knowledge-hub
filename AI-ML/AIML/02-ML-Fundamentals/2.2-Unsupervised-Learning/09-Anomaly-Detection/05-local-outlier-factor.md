# 🔍 Local Outlier Factor (LOF)

> **Chapter 9.5 — Density-Based Anomaly Detection**

---

## 📌 Overview

**Local Outlier Factor (LOF)**, introduced by **Breunig, Kriegel, Ng, and Sander (2000)**, detects anomalies by comparing the **local density** of a point to the local densities of its neighbors. A point with substantially **lower density** than its neighbors is considered an outlier. The key innovation is the word "local" — LOF can detect anomalies even in datasets with clusters of varying density.

### Core Intuition

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  GLOBAL methods fail here:        LOF handles it correctly:          │
│                                                                      │
│  Dense cluster     Sparse cluster Dense cluster    Sparse cluster    │
│  ┌──────────┐      ┌──────────┐   ┌──────────┐    ┌──────────┐     │
│  │●●●●●●●●●●│      │  ●   ●   │   │●●●●●●●●●●│    │  ●   ●   │     │
│  │●●●●●●●●●●│      │    ●     │   │●●●●●●●●●●│    │    ●     │     │
│  │●●●●●●●●●●│  ★   │  ●   ●   │   │●●●●●●●●●●│ ★  │  ●   ●   │     │
│  │●●●●●●●●●●│      │    ●     │   │●●●●●●●●●●│    │    ●     │     │
│  └──────────┘      └──────────┘   └──────────┘    └──────────┘     │
│                                                                      │
│  ★ is between clusters.           LOF compares ★'s density          │
│  Global density check might        to its NEIGHBORS' density.       │
│  say ★ is normal (similar to       ★ is less dense than its         │
│  sparse cluster density).          nearest neighbors → ANOMALY!     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔢 Mathematical Foundation

### Step 1: k-Distance

The **k-distance** of a point p is the distance to its **k-th nearest neighbor**:

```
k-distance(p) = d(p, o)

where o is the k-th nearest neighbor of p

  Example (k=3):
  
       ○₃ (3rd nearest)
      ╱
     ╱  d₃ = 5
    ╱
   p ── d₁ = 2 ── ○₁ (nearest)
    ╲
     ╲  d₂ = 3
      ╲
       ○₂ (2nd nearest)
  
  k-distance(p) = d₃ = 5
```

### Step 2: Reachability Distance

The **reachability distance** of point p with respect to point o:

```
reach-dist_k(p, o) = max(k-distance(o), d(p, o))

This "smooths" distances by enforcing a minimum:
  - If p is very close to o → use k-distance(o) instead
  - If p is far from o → use actual distance d(p, o)
```

```
  Why smoothing?
  
  Without smoothing:          With smoothing:
  ○ ○○○○○○ → fluctuating     ○ ○○○○○○ → stable density
     distances                   estimates for core points
```

### Step 3: Local Reachability Density (LRD)

The **local reachability density** of point p:

```
                              1
lrd_k(p) = ──────────────────────────────────────────
           (1/|N_k(p)|) Σ_{o ∈ N_k(p)} reach-dist_k(p, o)

         =        |N_k(p)|
           ─────────────────────────────────
           Σ_{o ∈ N_k(p)} reach-dist_k(p, o)

where N_k(p) = k-nearest neighbors of p
```

```
┌──────────────────────────────────────────────────────────────┐
│ INTUITION:                                                   │
│                                                              │
│ lrd(p) = 1 / (average reachability distance to neighbors)    │
│                                                              │
│ High lrd → Point is in a DENSE region                        │
│            (small distances to neighbors)                    │
│                                                              │
│ Low lrd  → Point is in a SPARSE region                       │
│            (large distances to neighbors)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Step 4: Local Outlier Factor (LOF)

The **LOF score** compares p's density to its neighbors' densities:

```
              1       Σ_{o ∈ N_k(p)} lrd_k(o)
LOF_k(p) = ──────── × ──────────────────────────
           |N_k(p)|           lrd_k(p)

         = Average density of neighbors
           ──────────────────────────────
                 Density of p
```

### LOF Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│                LOF SCORE INTERPRETATION                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  LOF ≈ 1.0  → NORMAL                                        │
│               Point has similar density to its neighbors     │
│                                                              │
│  LOF >> 1.0 → ANOMALY (OUTLIER)                              │
│               Point has MUCH LOWER density than neighbors    │
│               (it's in a "desert" compared to neighbors)     │
│                                                              │
│  LOF < 1.0  → INLIER                                        │
│               Point has HIGHER density than neighbors        │
│               (it's in the core of a dense cluster)          │
│                                                              │
│  Typical threshold: LOF > 1.5 or LOF > 2.0                  │
│                                                              │
│  Scale:                                                      │
│  ◄── Inlier ──── Normal ──── Outlier ──────────►            │
│  0.5    0.8    1.0    1.5    2.0    3.0    5.0+             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📝 Worked Example

### Dataset

```
Points: A(1,1), B(1,2), C(2,1), D(2,2), E(10,10)

  y ▲
 10 │                     E★
    │
  2 │ B●    D●
  1 │ A●    C●
    └──┼──┼──────────────┼──► x
       1  2              10

k = 2 (2 nearest neighbors)
```

### Step 1: Find 2-Nearest Neighbors

```
Distances (Euclidean):
  d(A,B) = 1.0    d(A,C) = 1.0    d(A,D) = 1.41   d(A,E) = 12.73
  d(B,C) = 1.41   d(B,D) = 1.0    d(B,E) = 12.04
  d(C,D) = 1.0    d(C,E) = 12.04
  d(D,E) = 11.31

2-Nearest Neighbors:
  N₂(A) = {B, C}     2-dist(A) = 1.0
  N₂(B) = {A, D}     2-dist(B) = 1.0
  N₂(C) = {A, D}     2-dist(C) = 1.0
  N₂(D) = {B, C}     2-dist(D) = 1.0
  N₂(E) = {C, D}     2-dist(E) = 12.04
```

### Step 2: Reachability Distances

```
For point A:
  reach-dist(A, B) = max(2-dist(B), d(A,B)) = max(1.0, 1.0) = 1.0
  reach-dist(A, C) = max(2-dist(C), d(A,C)) = max(1.0, 1.0) = 1.0

For point E:
  reach-dist(E, C) = max(2-dist(C), d(E,C)) = max(1.0, 12.04) = 12.04
  reach-dist(E, D) = max(2-dist(D), d(E,D)) = max(1.0, 11.31) = 11.31
```

### Step 3: Local Reachability Density

```
lrd(A) = 2 / (1.0 + 1.0) = 2/2 = 1.0
lrd(B) = 2 / (1.0 + 1.0) = 1.0
lrd(C) = 2 / (1.0 + 1.0) = 1.0
lrd(D) = 2 / (1.0 + 1.0) = 1.0
lrd(E) = 2 / (12.04 + 11.31) = 2/23.35 = 0.0856
```

### Step 4: LOF Scores

```
LOF(A) = (lrd(B) + lrd(C)) / (2 × lrd(A))
       = (1.0 + 1.0) / (2 × 1.0) = 1.0  ✅ NORMAL

LOF(B) = (lrd(A) + lrd(D)) / (2 × lrd(B))
       = (1.0 + 1.0) / (2 × 1.0) = 1.0  ✅ NORMAL

LOF(E) = (lrd(C) + lrd(D)) / (2 × lrd(E))
       = (1.0 + 1.0) / (2 × 0.0856) = 2.0/0.171 = 11.7  🚨 ANOMALY!
```

```
  Results:
  ┌───────┬──────────┬───────────┬────────────────┐
  │ Point │ lrd      │ LOF       │ Classification │
  ├───────┼──────────┼───────────┼────────────────┤
  │ A     │ 1.0      │ 1.0       │ Normal         │
  │ B     │ 1.0      │ 1.0       │ Normal         │
  │ C     │ 1.0      │ 1.0       │ Normal         │
  │ D     │ 1.0      │ 1.0       │ Normal         │
  │ E     │ 0.086    │ 11.7      │ ANOMALY! 🚨    │
  └───────┴──────────┴───────────┴────────────────┘

  E has LOF >> 1 because its density (0.086) is MUCH lower
  than its neighbors' density (1.0 each).
```

---

## 🐍 Python Implementation

### Scikit-learn

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.neighbors import LocalOutlierFactor

# Generate data
np.random.seed(42)
# Dense cluster
X_dense = 0.3 * np.random.randn(200, 2) + [0, 0]
# Sparse cluster
X_sparse = 1.0 * np.random.randn(50, 2) + [5, 5]
# Anomalies
X_outliers = np.array([[-3, 3], [3, -3], [7, -2], [-2, 7]])

X = np.vstack([X_dense, X_sparse, X_outliers])

# LOF for outlier detection (fits AND predicts in one step)
lof = LocalOutlierFactor(
    n_neighbors=20,          # Number of neighbors (k)
    contamination=0.05,      # Expected outlier fraction
    metric='euclidean',      # Distance metric
    algorithm='auto',        # NN algorithm
)

# Note: LOF only supports fit_predict (not separate fit/predict)
y_pred = lof.fit_predict(X)         # 1=normal, -1=anomaly
lof_scores = -lof.negative_outlier_factor_  # LOF scores (positive, higher=more outlier)

print(f"Anomalies detected: {(y_pred == -1).sum()}")
print(f"Max LOF score: {lof_scores.max():.2f}")
print(f"Anomaly indices: {np.where(y_pred == -1)[0]}")

# Visualization
plt.figure(figsize=(12, 8))

# Plot with size proportional to LOF score
sizes = 30 * np.ones(len(X))
sizes[y_pred == -1] = 100

plt.scatter(X[y_pred == 1, 0], X[y_pred == 1, 1], 
            c='blue', s=20, alpha=0.6, label='Normal')
plt.scatter(X[y_pred == -1, 0], X[y_pred == -1, 1], 
            c='red', s=100, marker='*', label='Anomaly')

# Show LOF contour
plt.scatter(X[:, 0], X[:, 1], s=lof_scores * 20, 
            facecolors='none', edgecolors='orange', alpha=0.5,
            label='LOF magnitude')

plt.title(f'Local Outlier Factor (k={lof.n_neighbors})')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### LOF for Novelty Detection

```python
# For novelty detection, use novelty=True
# This allows separate fit() and predict() calls

lof_novelty = LocalOutlierFactor(
    n_neighbors=20,
    contamination=0.05,
    novelty=True  # Enable novelty detection mode
)

# Fit on clean training data
lof_novelty.fit(X_dense)  # Only normal data!

# Predict on new data
X_new = np.array([[0, 0], [5, 5], [10, 10]])
y_new = lof_novelty.predict(X_new)
scores_new = lof_novelty.decision_function(X_new)

for point, pred, score in zip(X_new, y_new, scores_new):
    status = "Normal" if pred == 1 else "ANOMALY"
    print(f"Point {point}: {status} (score: {score:.3f})")
```

### Effect of k (n_neighbors)

```python
fig, axes = plt.subplots(1, 4, figsize=(20, 5))
k_values = [5, 10, 20, 50]

for ax, k in zip(axes, k_values):
    lof = LocalOutlierFactor(n_neighbors=k, contamination=0.05)
    y_pred = lof.fit_predict(X)
    lof_scores = -lof.negative_outlier_factor_
    
    ax.scatter(X[y_pred == 1, 0], X[y_pred == 1, 1], c='blue', s=10)
    ax.scatter(X[y_pred == -1, 0], X[y_pred == -1, 1], c='red', s=50, marker='*')
    
    n_outliers = (y_pred == -1).sum()
    ax.set_title(f'k={k} ({n_outliers} outliers)')

plt.suptitle('LOF: Effect of k (n_neighbors)', fontsize=14)
plt.tight_layout()
plt.show()
```

---

## ⚖️ LOF vs Other Methods

```
┌──────────────────────┬──────────────┬──────────────┬──────────────┐
│     Aspect           │     LOF      │  Isolation   │  One-Class   │
│                      │              │  Forest      │  SVM         │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Approach             │ Density      │ Isolation    │ Boundary     │
│ Handles varying      │ ✅ Yes!      │ ❌ No        │ ❌ No        │
│ density clusters     │ (LOCAL)      │ (global)     │ (global)     │
│ Scalability          │ O(N²) worst  │ O(N log N)   │ O(N²-N³)    │
│ High-D data          │ Poor         │ Good         │ Poor         │
│ Interpretability     │ LOF score    │ Path length  │ Decision fn  │
│ New point detection  │ Only with    │ ✅ Yes       │ ✅ Yes       │
│                      │ novelty=True │              │              │
│ Key parameter        │ k            │ contamination│ ν, γ         │
│ Local vs Global      │ Local        │ Global       │ Global       │
└──────────────────────┴──────────────┴──────────────┴──────────────┘
```

### LOF's Key Advantage: Varying Density

```
  Scenario: Two clusters with DIFFERENT densities

  Method that FAILS:                 LOF SUCCEEDS:
  
  Global threshold:                  Local comparison:
  ┌──────────────────┐              ┌──────────────────┐
  │ ●●●●  ★          │              │ ●●●●  ★  ←LOF>1 │
  │ ●●●●        ○    │              │ ●●●●       ○     │
  │ ●●●●    ○   ○    │              │ ●●●●   ○   ○    │
  │ ●●●●        ○    │              │ ●●●●       ○     │
  │                   │              │                  │
  │ ★ has same density│              │ ★ has LOW density│
  │ as sparse cluster │              │ relative to its  │
  │ → MISSED!         │              │ neighbors → ✅!   │
  └──────────────────┘              └──────────────────┘
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Algorithm** | Local Outlier Factor (Breunig et al., 2000) |
| **Approach** | Compare local density of point to neighbors' density |
| **k-distance** | Distance to k-th nearest neighbor |
| **Reachability Distance** | max(k-dist(o), d(p,o)) — smoothed distance |
| **LRD** | 1 / average reachability distance to neighbors |
| **LOF Score** | Avg neighbor density / point density |
| **LOF ≈ 1** | Normal (similar density to neighbors) |
| **LOF >> 1** | Anomaly (much lower density than neighbors) |
| **Key Advantage** | Handles clusters with varying densities |
| **Complexity** | O(N · k) for NN queries; O(N²) worst case |
| **sklearn** | `LocalOutlierFactor(n_neighbors=20, contamination=0.05)` |

---

## ❓ Revision Questions

1. **What does "local" mean in Local Outlier Factor? Why is this property crucial for datasets with clusters of different densities?**
   Compare with global methods and give a concrete example where global methods fail.

2. **Walk through the LOF computation for points A(0,0), B(1,0), C(0,1), D(10,10) with k=2.**
   Compute k-distances, reachability distances, LRD, and LOF scores for each point.

3. **Explain the reachability distance and why it uses max(k-dist(o), d(p,o)) instead of just d(p,o).**
   Discuss the smoothing effect and stability of density estimates.

4. **How does the choice of k (n_neighbors) affect LOF results? What happens with very small k vs. very large k?**
   Discuss noise sensitivity, computational cost, and loss of locality.

5. **Compare LOF with Isolation Forest. For a dataset with two clusters (one dense, one sparse) and anomalies near the sparse cluster, which method would perform better?**
   Analyze why local density comparison matters in this scenario.

6. **LOF's sklearn implementation uses fit_predict() by default. Why can't it use separate fit() and predict() calls? How does novelty=True change this?**
   Discuss the transductive vs. inductive settings.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← One-Class SVM](./04-one-class-svm.md) | [📂 Unsupervised Learning](../README.md) | [Autoencoders for AD →](./06-autoencoders-anomaly-detection.md) |

---

> **Key Takeaway:** LOF is the go-to method when your data has clusters of varying density. By comparing each point's density to its neighbors' densities (not a global threshold), LOF can detect anomalies that global methods miss. Points with LOF >> 1 are surrounded by denser neighborhoods — they are the "outsiders among insiders."
