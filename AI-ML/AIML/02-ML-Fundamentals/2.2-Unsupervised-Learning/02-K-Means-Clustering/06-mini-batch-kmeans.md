# 2.6 Mini-Batch K-Means — Scaling to Large Datasets

> **Chapter Overview:** Standard K-Means processes all n data points in every iteration, which can be slow for datasets with millions of rows. Mini-Batch K-Means solves this by using **small random subsets (mini-batches)** in each iteration, dramatically reducing computation time with only a slight loss in quality. This chapter covers the algorithm, the speed-quality tradeoff, sklearn implementation, and online clustering scenarios.

---

## Table of Contents

1. [Why Standard K-Means Struggles at Scale](#1-why-standard-k-means-struggles-at-scale)
2. [Mini-Batch K-Means Algorithm](#2-mini-batch-k-means-algorithm)
3. [Mathematical Foundation](#3-mathematical-foundation)
4. [Speed vs Accuracy Tradeoff](#4-speed-vs-accuracy-tradeoff)
5. [Key Hyperparameters](#5-key-hyperparameters)
6. [Python Implementation with sklearn](#6-python-implementation-with-sklearn)
7. [Online / Streaming Clustering](#7-online--streaming-clustering)
8. [Comparison: K-Means vs Mini-Batch K-Means](#8-comparison-k-means-vs-mini-batch-k-means)
9. [When to Use Mini-Batch K-Means](#9-when-to-use-mini-batch-k-means)
10. [Worked Example — Large Dataset](#10-worked-example--large-dataset)
11. [Summary Table](#11-summary-table)
12. [Key Takeaways](#12-key-takeaways)
13. [Revision Questions](#13-revision-questions)

---

## 1. Why Standard K-Means Struggles at Scale

### Cost per Iteration

```
    Standard K-Means: O(n · K · d) per iteration

    n = 10,000,000 (10M samples)
    K = 100 clusters
    d = 200 features

    Per iteration: 10⁷ × 100 × 200 = 2 × 10¹¹ operations
    ~30 iterations: 6 × 10¹² total operations
    
    Wall clock: ~30-60 minutes on a modern CPU
    
    If you need to experiment with different K values or 
    hyperparameters, this becomes prohibitive!
```

### The Key Insight

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │  Not every data point needs to participate in every      │
    │  iteration!                                              │
    │                                                          │
    │  A RANDOM SAMPLE of the data gives a good approximation  │
    │  of the full-data centroid update.                       │
    │                                                          │
    │  Standard K-Means:   Use ALL n points each iteration     │
    │  Mini-Batch K-Means: Use b ≪ n points each iteration    │
    │                                                          │
    │  Speedup: ~n/b per iteration                             │
    │  b = 1000, n = 10M → 10,000× faster per iteration!     │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

---

## 2. Mini-Batch K-Means Algorithm

### Algorithm (Sculley, 2010)

```
    ┌──────────────────────────────────────────────────────────┐
    │               MINI-BATCH K-MEANS ALGORITHM               │
    │                                                          │
    │  Input: X (data), K (clusters), b (batch size),          │
    │         max_iter, tol                                    │
    │                                                          │
    │  1. Initialize centroids μ₁,...,μₖ (K-Means++ on sample)│
    │  2. Initialize per-centroid counts: v₁=...=vₖ=0          │
    │                                                          │
    │  3. FOR t = 1, 2, ..., max_iter:                         │
    │                                                          │
    │     a. SAMPLE mini-batch M ⊂ X, |M| = b                 │
    │        (random sample WITHOUT replacement)               │
    │                                                          │
    │     b. ASSIGN: For each xᵢ ∈ M:                         │
    │        cᵢ = argmin_k ‖xᵢ - μₖ‖²                        │
    │                                                          │
    │     c. UPDATE: For each xᵢ ∈ M:                         │
    │        k = cᵢ  (assigned cluster)                       │
    │        vₖ += 1  (increment count)                       │
    │        η = 1/vₖ  (learning rate)                        │
    │        μₖ = (1 - η) · μₖ + η · xᵢ                      │
    │                                                          │
    │     d. CHECK convergence                                 │
    │                                                          │
    │  4. RETURN centroids, labels                             │
    └──────────────────────────────────────────────────────────┘
```

### The Update Rule Explained

```
    Standard K-Means update:
    μₖ = (1/|Cₖ|) Σᵢ∈Cₖ xᵢ         ← batch update (exact mean)

    Mini-Batch update:
    μₖ ← (1 - 1/vₖ) · μₖ + (1/vₖ) · xᵢ   ← online update (running mean)

    This is equivalent to a STREAMING AVERAGE:
    
    After seeing vₖ points assigned to cluster k:
    μₖ = (1/vₖ) Σ (all points ever assigned to k)

    As vₖ → ∞, this converges to the TRUE centroid!

    The learning rate η = 1/vₖ decreases over time:
    η₁ = 1/1 = 1.0  (first point: centroid jumps to it)
    η₂ = 1/2 = 0.5  (second: average of two)
    η₁₀ = 1/10 = 0.1
    η₁₀₀₀ = 1/1000 = 0.001  (barely moves: stable)
```

### Visual Comparison

```
    Standard K-Means (Iteration 1):        Mini-Batch K-Means (Iteration 1):
    
    Use ALL points:                        Use SAMPLE of b points:
    ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○            ○     ○   ○         ○
    ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○              ○       ○     ○
    ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○            ○   ○       ○       ○
    ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○                ○   ○       ○
    
    n = 52 distance calculations/centroid  b = 13 calculations/centroid
    EXACT centroid                         APPROXIMATE centroid
```

---

## 3. Mathematical Foundation

### Convergence Properties

```
    Standard K-Means:     Converges to local minimum of WCSS
    Mini-Batch K-Means:   Converges to VICINITY of local minimum

    Theorem (Sculley, 2010):
    
    Let J* be the WCSS at a local optimum of standard K-Means.
    Let J_mb be the WCSS achieved by Mini-Batch K-Means.
    
    Then:  E[J_mb] ≤ J* + ε

    Where ε → 0 as the number of iterations → ∞.
    
    The approximation error is bounded:
    
    ε ≤ O(√(d/b)) · max_k σ_k²
    
    Where:
    d = number of dimensions
    b = batch size
    σ_k² = variance of cluster k
    
    → Larger batch size b → smaller error → closer to standard K-Means
```

### Connection to Stochastic Gradient Descent

```
    K-Means can be viewed as an optimization problem:
    
    min  J(μ) = Σᵢ min_k ‖xᵢ - μₖ‖²
     μ

    Standard K-Means = full gradient descent (use all data)
    Mini-Batch K-Means = stochastic gradient descent (use sample)

    Just like SGD vs batch GD in neural networks:
    • SGD is noisier but MUCH faster per iteration
    • SGD converges to vicinity of optimum
    • Noise can help escape bad local minima!
```

---

## 4. Speed vs Accuracy Tradeoff

### Empirical Comparison

```
    Dataset: n = 1,000,000 points, d = 50, K = 20

    Algorithm          │ Time    │ Inertia  │ Relative Quality
    ───────────────────┼─────────┼──────────┼──────────────────
    K-Means            │ 45 min  │ 1000.0   │ 100% (baseline)
    MB K-Means (b=100) │ 15 sec  │ 1028.3   │ 97.2%
    MB K-Means (b=1K)  │ 45 sec  │ 1008.5   │ 99.2%
    MB K-Means (b=10K) │ 3 min   │ 1002.1   │ 99.8%
    MB K-Means (b=100K)│ 12 min  │ 1000.5   │ 99.95%
    
    → b = 1000-10000 gives excellent quality at 10-100× speedup!
```

### Batch Size Effect

```
    Quality                              Speed (iterations/sec)
    (1 - relative error)
    
    1.00 │            ___________        │╲
         │          ╱                    │ ╲
    0.99 │        ╱                      │  ╲
         │      ╱                        │   ╲
    0.98 │    ╱                          │    ╲___
         │  ╱                            │        ╲___
    0.95 │ ╱                             │            ╲________
         │╱                              │
    0.90 │                               │
         └──────────────── b             └──────────────────── b
          100 1K  10K 100K  n             100 1K  10K 100K  n

    Sweet spot: b ≈ 1000-10000
    → 99%+ quality at 10-100× speed improvement
```

---

## 5. Key Hyperparameters

```
┌─────────────────┬──────────────┬───────────────────────────────────────────┐
│ Parameter       │ Default      │ Effect                                     │
├─────────────────┼──────────────┼───────────────────────────────────────────┤
│ n_clusters      │ 8            │ K: number of clusters                     │
│ batch_size      │ 1024         │ b: points per mini-batch                  │
│                 │              │ Larger → better quality, slower            │
│ max_iter        │ 100          │ Maximum number of mini-batch iterations    │
│ max_no_improv.  │ 10           │ Stop after N iterations w/o improvement    │
│ init            │ 'k-means++'  │ Initialization method                     │
│ n_init          │ 3            │ Number of random restarts                 │
│ init_size       │ 3 × b        │ Number of samples for initialization      │
│ reassignment_f  │ 0.01         │ Fraction of cluster centers to reassign   │
│ tol             │ 0.0          │ Early stopping tolerance                  │
│ random_state    │ None         │ Reproducibility                           │
└─────────────────┴──────────────┴───────────────────────────────────────────┘
```

### Choosing Batch Size

```
    Rule of thumb:
    
    b ≈ max(256, min(n/10, 10000))
    
    • Minimum 256 for statistical stability
    • Maximum 10000 for speed benefit
    • Between: proportional to dataset size
    
    Very small b (< 100):  Noisy centroids, unstable results
    Medium b (1000-5000):  Good balance of speed and quality ← RECOMMENDED
    Large b (> n/2):       No speed benefit over standard K-Means
```

---

## 6. Python Implementation with sklearn

### Basic Usage

```python
from sklearn.cluster import MiniBatchKMeans
from sklearn.datasets import make_blobs
import numpy as np
import time

# Generate large dataset
X, y_true = make_blobs(n_samples=100000, centers=10, 
                         cluster_std=1.5, random_state=42)

# Standard K-Means
start = time.time()
from sklearn.cluster import KMeans
km = KMeans(n_clusters=10, random_state=42, n_init=3)
km.fit(X)
time_km = time.time() - start

# Mini-Batch K-Means
start = time.time()
mbkm = MiniBatchKMeans(n_clusters=10, batch_size=1024, 
                        random_state=42, n_init=3)
mbkm.fit(X)
time_mbkm = time.time() - start

print("=== Speed Comparison ===")
print(f"K-Means:       {time_km:.2f}s, Inertia: {km.inertia_:.0f}")
print(f"MiniBatch K-M: {time_mbkm:.2f}s, Inertia: {mbkm.inertia_:.0f}")
print(f"Speedup:       {time_km/time_mbkm:.1f}×")
print(f"Quality loss:  {(mbkm.inertia_ - km.inertia_)/km.inertia_*100:.2f}%")
```

### Batch Size Experiment

```python
from sklearn.cluster import MiniBatchKMeans, KMeans
from sklearn.datasets import make_blobs
import numpy as np
import time

X, _ = make_blobs(n_samples=200000, centers=15, 
                    cluster_std=1.5, random_state=42)

# Baseline: standard K-Means
start = time.time()
km = KMeans(n_clusters=15, random_state=42, n_init=3)
km.fit(X)
base_time = time.time() - start
base_inertia = km.inertia_

print(f"{'Method':<25} {'Time':>8} {'Inertia':>12} {'Quality':>8} {'Speedup':>8}")
print("-" * 65)
print(f"{'K-Means (baseline)':<25} {base_time:>7.2f}s {base_inertia:>12.0f} {'100.0%':>8} {'1.0×':>8}")

for batch_size in [100, 256, 512, 1024, 2048, 5000, 10000]:
    start = time.time()
    mbkm = MiniBatchKMeans(n_clusters=15, batch_size=batch_size, 
                            random_state=42, n_init=3)
    mbkm.fit(X)
    t = time.time() - start
    quality = (1 - (mbkm.inertia_ - base_inertia) / base_inertia) * 100
    speedup = base_time / t
    print(f"{'MB (b=' + str(batch_size) + ')':<25} {t:>7.2f}s {mbkm.inertia_:>12.0f} "
          f"{quality:>7.1f}% {speedup:>7.1f}×")
```

### Incremental / Partial Fit (Online Learning)

```python
from sklearn.cluster import MiniBatchKMeans
import numpy as np

# Simulate streaming data
np.random.seed(42)
mbkm = MiniBatchKMeans(n_clusters=5, random_state=42, batch_size=500)

total_points = 0
for batch_num in range(20):
    # Simulate receiving a new batch of data
    X_batch = np.random.randn(500, 10) + np.random.choice([-3, 0, 3, 6, 9], 
                                                           size=(500, 1))
    mbkm.partial_fit(X_batch)
    total_points += len(X_batch)
    
    if (batch_num + 1) % 5 == 0:
        print(f"After batch {batch_num+1}: "
              f"{total_points} total points processed, "
              f"Inertia: {mbkm.inertia_:.2f}")

print(f"\nFinal cluster centers shape: {mbkm.cluster_centers_.shape}")
print(f"Final cluster sizes: {np.bincount(mbkm.labels_)}")
```

---

## 7. Online / Streaming Clustering

### The Use Case

```
    ┌──────────────────────────────────────────────────────────┐
    │                  STREAMING DATA                          │
    │                                                          │
    │  Data arrives continuously — can't store it all!         │
    │                                                          │
    │  Examples:                                               │
    │  • IoT sensor readings (1M events/sec)                   │
    │  • Web clickstream data                                  │
    │  • Financial transaction monitoring                      │
    │  • Social media posts                                    │
    │                                                          │
    │  Requirements:                                           │
    │  ✓ Process each batch once                               │
    │  ✓ Don't store all historical data                       │
    │  ✓ Update model incrementally                            │
    │  ✓ Handle concept drift (clusters change over time)      │
    └──────────────────────────────────────────────────────────┘
```

### Online Clustering Pipeline

```
    Data Stream:  ──batch₁──batch₂──batch₃──batch₄──batch₅──→ time
                      │       │       │       │       │
                      ▼       ▼       ▼       ▼       ▼
                  ┌───────────────────────────────────────┐
                  │    MiniBatchKMeans.partial_fit()       │
                  │                                       │
                  │    Centroids updated incrementally     │
                  │    No need to re-process old data      │
                  └──────────────┬────────────────────────┘
                                 │
                                 ▼
                         Current cluster model
                         (always up to date)
```

### Concept Drift Handling

```python
from sklearn.cluster import MiniBatchKMeans
import numpy as np

# Simulate concept drift: cluster centers shift over time
np.random.seed(42)
mbkm = MiniBatchKMeans(n_clusters=3, random_state=42, batch_size=200)

for epoch in range(50):
    # Clusters slowly drift over time
    drift = epoch * 0.1
    centers = np.array([[0 + drift, 0], [5, 5 + drift], [10 - drift, 0]])
    
    X_batch = np.vstack([
        np.random.randn(100, 2) * 0.5 + centers[0],
        np.random.randn(100, 2) * 0.5 + centers[1],
        np.random.randn(100, 2) * 0.5 + centers[2],
    ])
    
    mbkm.partial_fit(X_batch)
    
    if (epoch + 1) % 10 == 0:
        print(f"Epoch {epoch+1}: Centers = {mbkm.cluster_centers_.round(2)}")
        print(f"  True drift centers: {centers.round(2)}")
```

---

## 8. Comparison: K-Means vs Mini-Batch K-Means

```
┌──────────────────────┬──────────────────────────┬──────────────────────────┐
│ Aspect               │ K-Means (Lloyd's)        │ Mini-Batch K-Means       │
├──────────────────────┼──────────────────────────┼──────────────────────────┤
│ Data per iteration   │ ALL n points             │ Random sample of b points│
│ Time per iteration   │ O(n · K · d)             │ O(b · K · d)             │
│ Total time           │ O(n · K · d · I)         │ O(b · K · d · I)         │
│ Convergence          │ Exact local minimum      │ Approximate local min    │
│ Quality              │ Best (for given init)    │ ~99% of standard         │
│ Memory               │ O(n · d)                 │ O(b · d)  ← much less   │
│ Online learning      │ NO (needs all data)      │ YES (partial_fit)        │
│ Streaming            │ NO                       │ YES                      │
│ sklearn class        │ KMeans                   │ MiniBatchKMeans          │
│ Default batch_size   │ n (all data)             │ 1024                     │
│ Default n_init       │ 10                       │ 3                        │
│ Typical speedup      │ 1× (baseline)            │ 3-300×                   │
│ When to use          │ n < 100K                 │ n > 100K                 │
└──────────────────────┴──────────────────────────┴──────────────────────────┘
```

---

## 9. When to Use Mini-Batch K-Means

```
    USE Mini-Batch K-Means when:
    ┌──────────────────────────────────────────────────────┐
    │ ✓ Dataset is LARGE (n > 100,000)                     │
    │ ✓ Speed is more important than marginal quality      │
    │ ✓ Data arrives in a STREAM                           │
    │ ✓ You need to experiment with many K values quickly  │
    │ ✓ Memory is limited (can't load all data)            │
    │ ✓ Real-time clustering is needed                     │
    └──────────────────────────────────────────────────────┘

    USE Standard K-Means when:
    ┌──────────────────────────────────────────────────────┐
    │ ✓ Dataset is SMALL to MEDIUM (n < 100,000)           │
    │ ✓ Quality is paramount (e.g., final production model)│
    │ ✓ All data is available at once (batch setting)      │
    │ ✓ You can afford the computation time                │
    └──────────────────────────────────────────────────────┘
```

---

## 10. Worked Example — Large Dataset

```python
"""Complete Mini-Batch K-Means workflow on a large dataset."""

from sklearn.cluster import MiniBatchKMeans, KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
from sklearn.datasets import make_blobs
import numpy as np
import time

# Step 1: Generate large dataset
print("Generating dataset...")
X, y_true = make_blobs(n_samples=500000, centers=8, 
                         n_features=20, cluster_std=2.0, random_state=42)

# Step 2: Standardize
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Step 3: Find optimal K using Mini-Batch (fast!)
print("\nFinding optimal K...")
for k in range(4, 12):
    start = time.time()
    mbkm = MiniBatchKMeans(n_clusters=k, batch_size=2048, 
                            random_state=42, n_init=3)
    labels = mbkm.fit_predict(X_scaled)
    t = time.time() - start
    
    # Silhouette on sample (computing on all 500K is slow)
    sample_idx = np.random.choice(len(X_scaled), 10000, replace=False)
    sil = silhouette_score(X_scaled[sample_idx], labels[sample_idx])
    
    print(f"  K={k:>2}: Inertia={mbkm.inertia_:>12.0f}, "
          f"Silhouette={sil:.4f}, Time={t:.2f}s")

# Step 4: Final model with best K
print("\nTraining final model (K=8)...")
start = time.time()
final_model = MiniBatchKMeans(n_clusters=8, batch_size=4096, 
                               random_state=42, n_init=5)
final_labels = final_model.fit_predict(X_scaled)
final_time = time.time() - start

print(f"Final model: Inertia={final_model.inertia_:.0f}, "
      f"Time={final_time:.2f}s")
print(f"Cluster sizes: {np.bincount(final_labels)}")
```

---

## 11. Summary Table

```
┌────────────────────┬───────────────────────────────────────────────────────┐
│ Concept            │ Detail                                                │
├────────────────────┼───────────────────────────────────────────────────────┤
│ Key Idea           │ Use random mini-batches instead of all data          │
│ Update Rule        │ μₖ ← (1-1/vₖ)·μₖ + (1/vₖ)·xᵢ (online mean)        │
│ Time Complexity    │ O(b·K·d·I) where b ≪ n                              │
│ Quality Loss       │ Typically < 1-3% compared to standard K-Means       │
│ Speedup            │ 3-300× depending on n/b ratio                       │
│ Batch Size Rule    │ b ≈ 1000-10000 is usually good                      │
│ Online Learning    │ partial_fit() for streaming data                     │
│ sklearn Class      │ MiniBatchKMeans                                      │
│ Use When           │ n > 100K or streaming/real-time needs               │
│ Connection to SGD  │ Mini-Batch K-Means ≈ SGD for clustering objective   │
└────────────────────┴───────────────────────────────────────────────────────┘
```

---

## 12. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Mini-Batch K-Means uses **random subsets** of size b per iteration instead of all n points |
| 2 | The update rule is a **running average** — equivalent to online mean computation |
| 3 | Speedup is approximately **n/b** per iteration (often 10-100×) |
| 4 | Quality loss is typically **< 1-3%** compared to standard K-Means |
| 5 | The `partial_fit()` method enables **online/streaming** clustering |
| 6 | Batch size b ≈ **1000-10000** is usually a good choice |
| 7 | Use Mini-Batch K-Means when **n > 100K** or for real-time applications |

---

## 13. Revision Questions

1. **Algorithm:** Describe the Mini-Batch K-Means update rule. How does it differ from the standard K-Means update? Show that the online update converges to the batch mean.

2. **Batch Size:** What happens if the batch size b is too small (e.g., b=10)? What if b=n? How would you choose b for a dataset with 5 million points?

3. **Speed-Quality Tradeoff:** On a dataset with n=1M, K=20, d=100, estimate the speedup of Mini-Batch K-Means with b=2000 compared to standard K-Means. What quality loss would you expect?

4. **Online Learning:** Explain how `partial_fit()` enables streaming clustering. What is concept drift, and how does Mini-Batch K-Means handle it?

5. **SGD Connection:** Explain the connection between Mini-Batch K-Means and Stochastic Gradient Descent. What is the "loss function" being optimized?

6. **Practical:** Design a complete pipeline for clustering 50 million web logs in real time. Which algorithm would you use? What batch size? How would you evaluate quality?

---

<div align="center">

| [← Convergence & Limitations](./05-convergence-and-limitations.md) | [Up: K-Means Clustering](./README.md) | [Next Unit: Hierarchical Clustering →](../03-Hierarchical-Clustering/01-agglomerative-vs-divisive.md) |
|:------------------------------------------------------------------:|:--------------------------------------:|:-----------------------------------------------------------------------------------------------------:|

</div>
