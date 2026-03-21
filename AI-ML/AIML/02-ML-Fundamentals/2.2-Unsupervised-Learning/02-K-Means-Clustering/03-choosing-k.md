# 2.3 Choosing K — The Right Number of Clusters

> **Chapter Overview:** The number of clusters K is the most critical hyperparameter in K-Means. This chapter provides a thorough treatment of the three main methods for choosing K — the Elbow method, Silhouette analysis, and the Gap statistic — along with the role of domain knowledge. Complete Python implementations and visualizations are included for each method.

---

## Table of Contents

1. [Why Choosing K is Hard](#1-why-choosing-k-is-hard)
2. [The Elbow Method](#2-the-elbow-method)
3. [The Silhouette Score](#3-the-silhouette-score)
4. [The Gap Statistic](#4-the-gap-statistic)
5. [Domain Knowledge](#5-domain-knowledge)
6. [Other Methods (Brief)](#6-other-methods-brief)
7. [Comparison of Methods](#7-comparison-of-methods)
8. [Complete Python Implementation](#8-complete-python-implementation)
9. [Summary Table](#9-summary-table)
10. [Key Takeaways](#10-key-takeaways)
11. [Revision Questions](#11-revision-questions)

---

## 1. Why Choosing K is Hard

### The Fundamental Dilemma

```
    K = 1:  Everything in one cluster     →  WCSS = total variance (maximum)
    K = n:  Each point is its own cluster  →  WCSS = 0 (minimum)

    Both extremes are useless!
    
    WCSS
    │╲
    │ ╲
    │  ╲
    │   ╲_
    │     ╲__
    │        ╲____
    │             ╲________
    │                      ╲_______________
    │                                      ╲______ → 0
    └──────────────────────────────────────────────── K
     1  2  3  4  5  6  7  8  9  10  ...  n

    WCSS ALWAYS decreases as K increases!
    → We can't simply pick K that minimizes WCSS.
```

### Why Not Just Minimize WCSS?

Adding more clusters **always** reduces WCSS because:

```
    Mathematical proof:
    
    If we split cluster Cₖ into C_a and C_b:
    
    WCSS(Cₖ) ≥ WCSS(C_a) + WCSS(C_b)

    Because the variance within each sub-cluster ≤ variance of the whole.
    
    Extreme: K = n → WCSS = 0 (each point is its own centroid)
    
    → We need methods that balance WCSS reduction with model simplicity
```

---

## 2. The Elbow Method

### Core Idea

Plot WCSS (inertia) against K and look for an **"elbow"** — a point where the rate of decrease sharply changes.

### Mathematical Basis

```
    For each K = 1, 2, ..., K_max:
        Run K-Means, record WCSS(K) = inertia

    Look for K where:
        ΔWCSS(K) = WCSS(K-1) - WCSS(K)

    transitions from "large" to "small"

    Formally, we're looking for the point of maximum curvature:
    
    κ(K) = |WCSS''(K)| / (1 + WCSS'(K)²)^(3/2)
    
    The elbow is where κ(K) is maximized.
```

### ASCII Elbow Diagram

```
    WCSS
    (Inertia)
    │
    │╲
    350│ ╲
    │  ╲
    300│   ╲
    │    ╲
    250│     ╲
    │      ╲_                    The "elbow" is at K=3
    200│        ╲__               ↙
    │           ╲____           
    150│               ╲__________
    │                          ╲___________
    100│                                    ╲__________
    │
    └──────────────────────────────────────────────── K
      1    2    3    4    5    6    7    8    9   10

    Before elbow (K<3): Large drops → adding clusters captures real structure
    At elbow (K=3):     Diminishing returns begin
    After elbow (K>3):  Small drops → adding clusters only splits noise
```

### Strengths and Weaknesses

```
    ✓ Simple and intuitive
    ✓ Visual method — easy to communicate
    ✓ Works well when clusters are well-separated
    
    ✗ Often AMBIGUOUS — no clear elbow
    ✗ Subjective (different people see different elbows)
    ✗ Doesn't work well for overlapping clusters
    ✗ No statistical test or p-value
```

### Python Implementation

```python
from sklearn.cluster import KMeans
import numpy as np
import matplotlib.pyplot as plt

def elbow_method(X, k_range=range(1, 11), random_state=42):
    """Compute and plot the Elbow method for choosing K."""
    inertias = []
    for k in k_range:
        km = KMeans(n_clusters=k, random_state=random_state, n_init=10)
        km.fit(X)
        inertias.append(km.inertia_)
    
    # Plot
    plt.figure(figsize=(10, 6))
    plt.plot(k_range, inertias, 'bo-', linewidth=2, markersize=8)
    plt.xlabel('Number of Clusters (K)', fontsize=12)
    plt.ylabel('Inertia (WCSS)', fontsize=12)
    plt.title('Elbow Method for Optimal K', fontsize=14)
    plt.grid(True, alpha=0.3)
    
    # Annotate rate of change
    for i in range(1, len(inertias)):
        drop = inertias[i-1] - inertias[i]
        pct_drop = drop / inertias[i-1] * 100
        plt.annotate(f'-{pct_drop:.0f}%', 
                    xy=(list(k_range)[i], inertias[i]),
                    xytext=(10, 10), textcoords='offset points',
                    fontsize=8, color='red')
    
    plt.tight_layout()
    plt.savefig("elbow_method.png", dpi=150)
    plt.show()
    
    return dict(zip(k_range, inertias))

# Usage
from sklearn.datasets import make_blobs
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=1.0, random_state=42)
results = elbow_method(X)
```

---

## 3. The Silhouette Score

### Core Idea

The silhouette score measures how **similar** a point is to its own cluster compared to **other** clusters. It provides a per-point, per-cluster, and overall quality measure.

### Mathematics

```
    For each data point i:

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  a(i) = (1 / |Cᵢ| - 1) ·  Σ    d(i, j)                   │
    │                           j∈Cᵢ,j≠i                        │
    │                                                            │
    │  Average distance from i to all OTHER points in            │
    │  its own cluster (measure of cohesion)                     │
    │                                                            │
    │  b(i) = min   { (1/|Cₖ|) ·  Σ   d(i, j) }                │
    │        k≠cᵢ              j∈Cₖ                              │
    │                                                            │
    │  Minimum average distance from i to points in the          │
    │  nearest OTHER cluster (measure of separation)             │
    │                                                            │
    │  s(i) = (b(i) - a(i)) / max(a(i), b(i))                   │
    │                                                            │
    │  Range: [-1, +1]                                           │
    │                                                            │
    │  s(i) → +1:  Well-clustered (a ≪ b)                       │
    │  s(i) →  0:  On boundary between clusters (a ≈ b)         │
    │  s(i) → -1:  Probably in wrong cluster (a ≫ b)            │
    │                                                            │
    └────────────────────────────────────────────────────────────┘

    Overall Silhouette Score = (1/n) Σᵢ s(i)
```

### Worked Example

```
    Points: A(1,1), B(2,1), C(1.5,2) in Cluster 1
            D(5,5), E(6,5), F(5.5,6) in Cluster 2

    For point A = (1, 1):

    a(A) = avg distance to {B, C}
         = (d(A,B) + d(A,C)) / 2
         = (1.0 + 1.12) / 2 = 1.06

    b(A) = avg distance to cluster 2 {D, E, F}
         = (d(A,D) + d(A,E) + d(A,F)) / 3
         = (5.66 + 6.40 + 6.73) / 3 = 6.26

    s(A) = (6.26 - 1.06) / max(1.06, 6.26)
         = 5.20 / 6.26
         = 0.83  ← well-clustered!
```

### Silhouette Plot

```
    For K=3:
    
    Silhouette Value
    ──────────────────────────────────────────────
    Cluster 0  ████████████████████████░░░░░░  avg=0.72
    (n=95)     █████████████████████████
               ████████████████████████████
               ███████████████████████████████
    ──────────────────────────────────────────────
    Cluster 1  ███████████████████░░░░░░░░░░  avg=0.55
    (n=110)    ████████████████████████
               ███████████████████
               ██████████████████████████
    ──────────────────────────────────────────────
    Cluster 2  ████████████████████████████░░  avg=0.68
    (n=95)     ██████████████████████████████
               ████████████████████████████
               ██████████████████████████
    ──────────────────────────────────────────────
               -0.2  0.0  0.2  0.4  0.6  0.8  1.0

    Overall avg silhouette = 0.65
    
    What to look for:
    • All clusters roughly same width (balanced sizes)
    • No values below 0 (no misclassified points)
    • High overall average (>0.5 is good, >0.7 is excellent)
```

### Interpretation Guide

```
    Overall Silhouette Score    Interpretation
    ────────────────────────    ──────────────────────────
    0.71 – 1.00                 Strong structure found
    0.51 – 0.70                 Reasonable structure found
    0.26 – 0.50                 Weak structure (may be artificial)
    ≤ 0.25                      No substantial structure
```

### Python Implementation

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score, silhouette_samples
import numpy as np
import matplotlib.pyplot as plt

def silhouette_analysis(X, k_range=range(2, 11), random_state=42):
    """Compute silhouette scores and create silhouette plots."""
    scores = []
    
    fig, axes = plt.subplots(1, 2, figsize=(16, 6))
    
    for k in k_range:
        km = KMeans(n_clusters=k, random_state=random_state, n_init=10)
        labels = km.fit_predict(X)
        score = silhouette_score(X, labels)
        scores.append(score)
        print(f"K={k}: Silhouette Score = {score:.4f}")
    
    # Plot 1: Silhouette score vs K
    axes[0].plot(k_range, scores, 'ro-', linewidth=2, markersize=8)
    axes[0].set_xlabel('Number of Clusters (K)', fontsize=12)
    axes[0].set_ylabel('Silhouette Score', fontsize=12)
    axes[0].set_title('Silhouette Score vs K', fontsize=14)
    axes[0].grid(True, alpha=0.3)
    
    best_k = list(k_range)[np.argmax(scores)]
    axes[0].axvline(x=best_k, color='green', linestyle='--', 
                    label=f'Best K = {best_k}')
    axes[0].legend()
    
    # Plot 2: Silhouette plot for best K
    km_best = KMeans(n_clusters=best_k, random_state=random_state, n_init=10)
    labels_best = km_best.fit_predict(X)
    sample_scores = silhouette_samples(X, labels_best)
    
    y_lower = 10
    for k in range(best_k):
        cluster_scores = sample_scores[labels_best == k]
        cluster_scores.sort()
        cluster_size = len(cluster_scores)
        y_upper = y_lower + cluster_size
        
        axes[1].fill_betweenx(np.arange(y_lower, y_upper),
                              0, cluster_scores, alpha=0.7)
        axes[1].text(-0.05, y_lower + 0.5 * cluster_size, str(k))
        y_lower = y_upper + 10
    
    avg_score = silhouette_score(X, labels_best)
    axes[1].axvline(x=avg_score, color='red', linestyle='--',
                    label=f'Avg = {avg_score:.3f}')
    axes[1].set_xlabel('Silhouette Coefficient', fontsize=12)
    axes[1].set_ylabel('Cluster', fontsize=12)
    axes[1].set_title(f'Silhouette Plot (K={best_k})', fontsize=14)
    axes[1].legend()
    
    plt.tight_layout()
    plt.savefig("silhouette_analysis.png", dpi=150)
    plt.show()
    
    return best_k, dict(zip(k_range, scores))

# Usage
from sklearn.datasets import make_blobs
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=1.0, random_state=42)
best_k, scores = silhouette_analysis(X)
print(f"\nBest K by silhouette: {best_k}")
```

---

## 4. The Gap Statistic

### Core Idea (Tibshirani, Walther & Hastie, 2001)

Compare the WCSS on the real data to the WCSS expected under a **null reference distribution** (uniform random data). The "gap" measures how far the real clustering is from random.

### Mathematics

```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │  Step 1: For each K = 1, ..., K_max:                          │
    │          Run K-Means on real data → compute log(W_K)          │
    │                                                                │
    │  Step 2: Generate B reference datasets from uniform            │
    │          distribution (same bounding box as data)              │
    │          Run K-Means on each → compute log(W_K^b) for b=1..B  │
    │                                                                │
    │  Step 3: Gap(K) = (1/B) Σᵇ log(W_K^b) - log(W_K)            │
    │                 = E*[log(W_K)] - log(W_K)                     │
    │                                                                │
    │  Step 4: Standard deviation:                                   │
    │          sd_K = √((1/B) Σᵇ (log(W_K^b) - mean)²)             │
    │          s_K  = sd_K × √(1 + 1/B)                             │
    │                                                                │
    │  Step 5: Choose smallest K such that:                         │
    │          Gap(K) ≥ Gap(K+1) - s_{K+1}                          │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Intuition

```
    log(W_K)
    │
    │  ╲  Reference (random data)
    │   ╲ ╲
    │    ╲  ╲
    │     ╲   ╲___________    ← gradual decrease (no structure)
    │      ╲              ╲_______________
    │       ╲
    │        ╲  Real data
    │         ╲
    │          ╲_                   ← sharp decrease (real structure!)
    │            ╲_________
    │                      ╲_________________
    │
    └───────────────────────────────────────── K
      1   2   3   4   5   6   7   8

    Gap(K) = vertical distance between reference and real curves
    
    Gap
    │
    │              ╱╲
    │             ╱  ╲
    │            ╱    ╲
    │           ╱      ╲____
    │          ╱             ╲___
    │         ╱
    │        ╱
    │       ╱
    └──────────────────────────── K
      1   2   3   4   5   6   7

    Maximum gap at K=4 → optimal K
```

### Python Implementation

```python
import numpy as np
from sklearn.cluster import KMeans

def gap_statistic(X, k_range=range(1, 11), n_refs=20, random_state=42):
    """Compute the Gap statistic for choosing K."""
    rng = np.random.RandomState(random_state)
    gaps = []
    s_values = []
    
    # Bounding box for reference distribution
    mins = X.min(axis=0)
    maxs = X.max(axis=0)
    
    for k in k_range:
        # WCSS on real data
        km = KMeans(n_clusters=k, random_state=random_state, n_init=10)
        km.fit(X)
        log_wk = np.log(km.inertia_)
        
        # WCSS on reference datasets
        ref_log_wks = []
        for _ in range(n_refs):
            # Generate uniform random data in same bounding box
            X_ref = rng.uniform(mins, maxs, size=X.shape)
            km_ref = KMeans(n_clusters=k, random_state=random_state, n_init=5)
            km_ref.fit(X_ref)
            ref_log_wks.append(np.log(km_ref.inertia_))
        
        ref_log_wks = np.array(ref_log_wks)
        gap = ref_log_wks.mean() - log_wk
        sd = ref_log_wks.std()
        s = sd * np.sqrt(1 + 1/n_refs)
        
        gaps.append(gap)
        s_values.append(s)
        print(f"K={k}: Gap = {gap:.4f}, s = {s:.4f}")
    
    # Find optimal K: smallest K such that Gap(K) >= Gap(K+1) - s(K+1)
    gaps = np.array(gaps)
    s_values = np.array(s_values)
    k_list = list(k_range)
    
    optimal_k = k_list[0]
    for i in range(len(gaps) - 1):
        if gaps[i] >= gaps[i+1] - s_values[i+1]:
            optimal_k = k_list[i]
            break
    
    print(f"\nOptimal K by Gap Statistic: {optimal_k}")
    return optimal_k, dict(zip(k_range, gaps))

# Usage
from sklearn.datasets import make_blobs
X, _ = make_blobs(n_samples=300, centers=4, cluster_std=1.0, random_state=42)
optimal_k, gaps = gap_statistic(X)
```

---

## 5. Domain Knowledge

### When Metrics Disagree, Domain Wins

```
    Scenario: Customer segmentation for a marketing team

    Method              Suggested K
    ────────────────    ───────────
    Elbow               K = 3 or 4 (ambiguous)
    Silhouette          K = 3
    Gap Statistic       K = 4
    Marketing team      K = 5 (they need 5 campaigns)

    → Use K = 5 if the clusters are interpretable and actionable!
```

### Domain Knowledge Examples

| Domain | Natural K | Reasoning |
|--------|-----------|-----------|
| **Retail** | 4-6 | RFM segments (Champions, Loyal, At-Risk, Lost, New) |
| **Image compression** | 8-256 | Number of colors to keep |
| **Genomics** | 10-50 | Known cell types in tissue |
| **Document clustering** | 5-20 | Number of topic categories |
| **Network analysis** | varies | Known community structure |

### Practical Guidelines

```
    1. START with domain knowledge → set initial K estimate
    2. USE metrics (elbow, silhouette) → refine around that estimate
    3. VALIDATE with domain experts → do clusters make sense?
    4. ITERATE if needed → adjust K based on interpretability

    Remember: There is NO mathematically "correct" K!
    The right K is the one that is USEFUL for your application.
```

---

## 6. Other Methods (Brief)

### Information Criteria (BIC/AIC for GMM)

```
    BIC = -2 · log(L) + p · log(n)
    AIC = -2 · log(L) + 2p

    Where:
    L = likelihood of the model
    p = number of parameters
    n = number of data points

    Choose K that minimizes BIC/AIC.
    Only applies to model-based clustering (GMM), not K-Means directly.
```

### Calinski-Harabasz Index

```
    CH(K) = [BCSS / (K-1)] / [WCSS / (n-K)]

    = (between-cluster variance) / (within-cluster variance)
    × adjusted for degrees of freedom

    Higher CH → better clustering. Choose K with max CH.
```

### Davies-Bouldin Index

```
    DB = (1/K) Σᵢ max  (σᵢ + σⱼ) / d(μᵢ, μⱼ)
                  j≠i

    Where σᵢ = avg distance of points in cluster i to centroid μᵢ

    Lower DB → better clustering. Choose K with min DB.
```

### X-Means

```
    Automatically finds K by:
    1. Start with K=2
    2. For each cluster, try splitting it in two
    3. Use BIC to decide if the split improves the model
    4. Continue until no more splits improve BIC
```

---

## 7. Comparison of Methods

```
┌──────────────────┬───────────────┬──────────────┬──────────────┬──────────────┐
│ Method           │ Statistical   │ Handles      │ Computation  │ Automation   │
│                  │ Rigor         │ Ambiguity    │ Cost         │ Level        │
├──────────────────┼───────────────┼──────────────┼──────────────┼──────────────┤
│ Elbow            │ Low           │ Poorly       │ K_max runs   │ Manual       │
│ Silhouette       │ Medium        │ Moderately   │ K_max runs   │ Automatic    │
│ Gap Statistic    │ High          │ Well         │ K_max × B    │ Automatic    │
│                  │               │              │ runs (slow)  │              │
│ Domain Knowledge │ N/A           │ Best         │ None         │ Manual       │
│ BIC/AIC (GMM)    │ High          │ Well         │ K_max runs   │ Automatic    │
│ Calinski-Harabasz│ Medium        │ Moderately   │ K_max runs   │ Automatic    │
│ X-Means          │ Medium        │ Well         │ Adaptive     │ Automatic    │
└──────────────────┴───────────────┴──────────────┴──────────────┴──────────────┘
```

### Recommended Approach

```
    ┌─────────────────────────────────────────────────────────────┐
    │  PRACTICAL WORKFLOW FOR CHOOSING K                          │
    │                                                             │
    │  1. Compute ELBOW PLOT         → visual estimate            │
    │  2. Compute SILHOUETTE SCORES  → quantitative ranking       │
    │  3. (Optional) GAP STATISTIC   → statistical validation     │
    │  4. Consult DOMAIN KNOWLEDGE   → practical constraints      │
    │  5. Choose K where methods AGREE (or domain override)       │
    │  6. VALIDATE chosen K:                                      │
    │     - Inspect cluster profiles                              │
    │     - Visualize with PCA/t-SNE                              │
    │     - Check stability (run multiple times)                  │
    └─────────────────────────────────────────────────────────────┘
```

---

## 8. Complete Python Implementation

```python
"""Complete workflow for choosing K using multiple methods."""

import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
from sklearn.metrics import (silhouette_score, 
                              calinski_harabasz_score, 
                              davies_bouldin_score)

# Generate synthetic data
X, y_true = make_blobs(n_samples=400, centers=4, 
                         cluster_std=1.0, random_state=42)

K_range = range(2, 11)
results = {'inertia': [], 'silhouette': [], 'calinski': [], 'davies_bouldin': []}

for k in K_range:
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = km.fit_predict(X)
    
    results['inertia'].append(km.inertia_)
    results['silhouette'].append(silhouette_score(X, labels))
    results['calinski'].append(calinski_harabasz_score(X, labels))
    results['davies_bouldin'].append(davies_bouldin_score(X, labels))

# Create 4-panel figure
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Panel 1: Elbow Method
axes[0,0].plot(K_range, results['inertia'], 'bo-', linewidth=2)
axes[0,0].set_title('Elbow Method (WCSS)', fontsize=13)
axes[0,0].set_xlabel('K')
axes[0,0].set_ylabel('Inertia')
axes[0,0].grid(True, alpha=0.3)

# Panel 2: Silhouette Score
axes[0,1].plot(K_range, results['silhouette'], 'ro-', linewidth=2)
axes[0,1].set_title('Silhouette Score (↑ better)', fontsize=13)
axes[0,1].set_xlabel('K')
axes[0,1].set_ylabel('Silhouette Score')
axes[0,1].grid(True, alpha=0.3)
best_sil_k = list(K_range)[np.argmax(results['silhouette'])]
axes[0,1].axvline(x=best_sil_k, color='green', linestyle='--', 
                  label=f'Best K={best_sil_k}')
axes[0,1].legend()

# Panel 3: Calinski-Harabasz
axes[1,0].plot(K_range, results['calinski'], 'go-', linewidth=2)
axes[1,0].set_title('Calinski-Harabasz Index (↑ better)', fontsize=13)
axes[1,0].set_xlabel('K')
axes[1,0].set_ylabel('CH Index')
axes[1,0].grid(True, alpha=0.3)

# Panel 4: Davies-Bouldin
axes[1,1].plot(K_range, results['davies_bouldin'], 'mo-', linewidth=2)
axes[1,1].set_title('Davies-Bouldin Index (↓ better)', fontsize=13)
axes[1,1].set_xlabel('K')
axes[1,1].set_ylabel('DB Index')
axes[1,1].grid(True, alpha=0.3)

plt.suptitle('Choosing K: Multiple Methods Comparison', fontsize=15, y=1.02)
plt.tight_layout()
plt.savefig("choosing_k_comparison.png", dpi=150, bbox_inches='tight')
plt.show()

# Print summary
print("\n=== Summary ===")
print(f"Best K by Silhouette:       {best_sil_k}")
print(f"Best K by Calinski-Harabasz:{list(K_range)[np.argmax(results['calinski'])]}")
print(f"Best K by Davies-Bouldin:   {list(K_range)[np.argmin(results['davies_bouldin'])]}")
print(f"True K:                     4")
```

---

## 9. Summary Table

```
┌───────────────────┬───────────────────────────┬────────────────────────────────┐
│ Method            │ What It Measures          │ Decision Rule                  │
├───────────────────┼───────────────────────────┼────────────────────────────────┤
│ Elbow             │ WCSS drop rate            │ K at the "bend" in the curve   │
│ Silhouette        │ Cohesion vs separation    │ K with highest score           │
│ Gap Statistic     │ Gap vs random reference   │ Smallest K with Gap(K)≥        │
│                   │                           │ Gap(K+1)-s_{K+1}              │
│ Calinski-Harabasz │ Between/Within ratio      │ K with highest CH              │
│ Davies-Bouldin    │ Avg cluster similarity    │ K with lowest DB               │
│ Domain Knowledge  │ Business/science needs    │ K that's actionable            │
└───────────────────┴───────────────────────────┴────────────────────────────────┘
```

---

## 10. Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | WCSS **always decreases** with K — we can't simply minimize it |
| 2 | The **Elbow method** is simple but often ambiguous |
| 3 | The **Silhouette score** is more objective — higher is better, range [-1, +1] |
| 4 | The **Gap statistic** provides statistical rigor but is computationally expensive |
| 5 | **No single method is perfect** — use multiple methods and look for agreement |
| 6 | **Domain knowledge** should always inform the final choice |
| 7 | The "right" K is the one that is **useful and interpretable** for your application |

---

## 11. Revision Questions

1. **Elbow Method:** Why does WCSS always decrease as K increases? Draw the typical shape of the elbow curve and explain what the "elbow" represents.

2. **Silhouette Score:** Derive the silhouette coefficient for a point. What does s(i) = 0.9 mean? What does s(i) = -0.3 mean?

3. **Gap Statistic:** Explain the null reference distribution used in the Gap statistic. Why do we compare against uniform random data?

4. **Comparison:** Run K-Means on a dataset where the true K=5. Compare the K suggested by the elbow method, silhouette score, and gap statistic. Do they agree? Why or why not?

5. **Domain Knowledge:** Give an example where the statistically "optimal" K is different from the practically useful K. How would you handle this conflict?

6. **Implementation:** Write Python code that computes elbow, silhouette, and Calinski-Harabasz for K=2 to K=10 and recommends the best K based on majority vote.

---

<div align="center">

| [← Algorithm Steps](./02-kmeans-algorithm-steps.md) | [Up: K-Means Clustering](./README.md) | [Next: K-Means++ Initialization →](./04-initialization-kmeans-plus.md) |
|:----------------------------------------------------:|:--------------------------------------:|:----------------------------------------------------------------------:|

</div>
