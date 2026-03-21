# 📊 Internal Clustering Evaluation Metrics

> **Chapter 10.1 — Evaluating Clusters Without Ground Truth**

---

## 📌 Overview

**Internal evaluation metrics** assess clustering quality using **only the data and the cluster assignments** — no ground truth labels required. These metrics measure properties like **compactness** (how tight clusters are), **separation** (how far apart clusters are), and the balance between them.

### Why Internal Metrics?

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  In most real-world clustering tasks, we DON'T have ground truth!    │
│                                                                      │
│  We need metrics that answer:                                        │
│  • "Are the clusters well-separated?"                                │
│  • "Are points within a cluster similar to each other?"              │
│  • "Is this the right number of clusters (K)?"                       │
│                                                                      │
│  Internal metrics use ONLY:                                          │
│  ✅ The data points X                                                │
│  ✅ The cluster assignments y                                        │
│  ❌ No ground truth labels needed                                    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📐 1. Silhouette Score / Coefficient

### Definition

For each point i, the **silhouette coefficient** measures how similar a point is to its own cluster compared to other clusters:

```
          b(i) - a(i)
s(i) = ─────────────────
        max(a(i), b(i))

where:
  a(i) = average distance from point i to all OTHER points 
         in the SAME cluster (cohesion/compactness)
  
  b(i) = minimum average distance from point i to all points 
         in ANY OTHER cluster (separation)
         (i.e., average distance to nearest neighboring cluster)
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│          SILHOUETTE COEFFICIENT INTERPRETATION                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  s(i) ≈  1  → Point is WELL inside its cluster              │
│              (far from other clusters, close to own)         │
│                                                              │
│  s(i) ≈  0  → Point is ON THE BOUNDARY between clusters     │
│              (could belong to either cluster)                │
│                                                              │
│  s(i) ≈ -1  → Point is MISCLASSIFIED                        │
│              (closer to another cluster than its own)        │
│                                                              │
│  Scale:                                                      │
│  -1.0    -0.5     0.0     0.5     1.0                       │
│   │────────│────────│────────│────────│                       │
│  Bad  Poor    Boundary  Good  Excellent                      │
│                                                              │
│  OVERALL SILHOUETTE SCORE = mean of all s(i)                 │
│  • > 0.7  → Strong clustering structure                      │
│  • 0.5-0.7 → Reasonable structure                            │
│  • 0.25-0.5 → Weak structure (may be artificial)             │
│  • < 0.25 → No substantial structure                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Visual Explanation

```
              b(i) = min avg distance to another cluster
          ┌────────────────────────────────────┐
          │                                    │
          ▼                                    │
  ┌──────────────┐        ┌──────────────┐     │
  │   ○ ○ ○      │        │   ● ● ●      │     │
  │  ○ ○ i ○     │a(i)    │  ● ● ● ●     │     │
  │   ○ ○ ○      │  ←──── │   ● ● ●      │     │
  │  ○ ○ ○       │avg dist│              │     │
  └──────────────┘within   └──────────────┘     │
   Cluster of i                                  │
                           ┌──────────────┐     │
                           │   ▲ ▲ ▲      │ ◄───┘
                           │  ▲ ▲ ▲ ▲     │ avg dist to
                           │   ▲ ▲ ▲      │ nearest other
                           └──────────────┘ cluster
```

### Python Implementation

```python
from sklearn.metrics import silhouette_score, silhouette_samples
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
import numpy as np
import matplotlib.pyplot as plt

# Generate sample data
X, y_true = make_blobs(n_samples=500, centers=4, 
                        cluster_std=0.8, random_state=42)

# Try different K values
k_range = range(2, 8)
silhouette_scores = []

for k in k_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(X)
    score = silhouette_score(X, labels)
    silhouette_scores.append(score)
    print(f"K={k}: Silhouette Score = {score:.4f}")

# Plot
plt.figure(figsize=(8, 5))
plt.plot(list(k_range), silhouette_scores, 'bo-')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Silhouette Score')
plt.title('Silhouette Score vs K')
plt.grid(True, alpha=0.3)
best_k = list(k_range)[np.argmax(silhouette_scores)]
plt.axvline(best_k, color='red', linestyle='--', label=f'Best K={best_k}')
plt.legend()
plt.tight_layout()
plt.show()
```

### Silhouette Plot (Per-Sample)

```python
from sklearn.metrics import silhouette_samples

# Best K
k = 4
kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
labels = kmeans.fit_predict(X)
sample_silhouettes = silhouette_samples(X, labels)
avg_score = silhouette_score(X, labels)

fig, ax = plt.subplots(figsize=(8, 6))
y_lower = 10

for i in range(k):
    cluster_silhouettes = sample_silhouettes[labels == i]
    cluster_silhouettes.sort()
    
    y_upper = y_lower + len(cluster_silhouettes)
    color = plt.cm.tab10(i / k)
    ax.fill_betweenx(np.arange(y_lower, y_upper), 0, 
                     cluster_silhouettes, facecolor=color, alpha=0.7)
    ax.text(-0.05, y_lower + 0.5 * len(cluster_silhouettes), str(i))
    y_lower = y_upper + 10

ax.axvline(avg_score, color='red', linestyle='--', 
           label=f'Average: {avg_score:.3f}')
ax.set_xlabel('Silhouette Coefficient')
ax.set_ylabel('Cluster')
ax.set_title('Silhouette Plot')
ax.legend()
plt.tight_layout()
plt.show()
```

---

## 📐 2. Calinski-Harabasz Index (Variance Ratio Criterion)

### Definition

The **Calinski-Harabasz (CH) Index** is the ratio of between-cluster dispersion to within-cluster dispersion:

```
              trace(Bₖ) / (K - 1)
CH(K) = ───────────────────────────────
              trace(Wₖ) / (N - K)

where:
  Bₖ = between-cluster dispersion matrix
     = Σₖ nₖ (cₖ - c)(cₖ - c)ᵀ
  
  Wₖ = within-cluster dispersion matrix
     = Σₖ Σ_{x∈Cₖ} (x - cₖ)(x - cₖ)ᵀ
  
  K  = number of clusters
  N  = total number of points
  nₖ = number of points in cluster k
  cₖ = centroid of cluster k
  c  = grand centroid (overall mean)
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│ CALINSKI-HARABASZ INDEX                                      │
│                                                              │
│ HIGHER is BETTER                                             │
│                                                              │
│ High CH → Clusters are well-separated AND compact            │
│          (large between / small within)                      │
│                                                              │
│ Low CH  → Clusters overlap or are dispersed                  │
│                                                              │
│ No fixed "good" value — use to COMPARE K values              │
│ Choose K that MAXIMIZES CH index                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python Implementation

```python
from sklearn.metrics import calinski_harabasz_score

ch_scores = []
for k in k_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(X)
    ch = calinski_harabasz_score(X, labels)
    ch_scores.append(ch)
    print(f"K={k}: Calinski-Harabasz = {ch:.2f}")
```

---

## 📐 3. Davies-Bouldin Index

### Definition

The **Davies-Bouldin (DB) Index** measures the average **similarity** between each cluster and its most similar cluster:

```
           1    K      ┌ σᵢ + σⱼ ┐
DB(K) = ─────  Σ  max  │ ─────── │
           K   i=1 j≠i │ d(cᵢ,cⱼ)│
                        └         ┘

where:
  σᵢ = average distance of points in cluster i to centroid cᵢ
  d(cᵢ, cⱼ) = distance between centroids i and j
  
  The ratio (σᵢ + σⱼ) / d(cᵢ,cⱼ) measures "similarity"
  of clusters i and j (large spread + close centroids = similar)
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│ DAVIES-BOULDIN INDEX                                         │
│                                                              │
│ LOWER is BETTER (opposite of CH!)                            │
│                                                              │
│ Low DB  → Clusters are compact and well-separated            │
│ High DB → Clusters overlap or are spread out                 │
│                                                              │
│ DB = 0 → Perfect clustering (impossible in practice)         │
│                                                              │
│ Choose K that MINIMIZES DB index                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python Implementation

```python
from sklearn.metrics import davies_bouldin_score

db_scores = []
for k in k_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(X)
    db = davies_bouldin_score(X, labels)
    db_scores.append(db)
    print(f"K={k}: Davies-Bouldin = {db:.4f}")
```

---

## 📐 4. Inertia / WCSS (Within-Cluster Sum of Squares)

### Definition

```
              K
WCSS = Σ    Σ      ‖xᵢ - cₖ‖²
       k=1  xᵢ∈Cₖ

where cₖ = centroid of cluster k
```

This is the **objective function** minimized by K-Means. Used for the **Elbow Method**.

### The Elbow Method

```
  WCSS
  ▲
  │╲
  │ ╲
  │  ╲
  │   ╲
  │    ╲── "Elbow" point (K=4)
  │     ╲
  │      ────────────────────
  │
  └──────────────────────────► K
  1  2  3  4  5  6  7  8

  Choose K at the "elbow" — where adding more clusters
  gives diminishing returns.
```

### Python: Elbow Method

```python
inertias = []
for k in k_range:
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    kmeans.fit(X)
    inertias.append(kmeans.inertia_)

plt.figure(figsize=(8, 5))
plt.plot(list(k_range), inertias, 'bo-')
plt.xlabel('Number of Clusters (K)')
plt.ylabel('Inertia (WCSS)')
plt.title('Elbow Method')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## 📊 Comprehensive Comparison Dashboard

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Silhouette
axes[0, 0].plot(list(k_range), silhouette_scores, 'bo-')
axes[0, 0].set_title('Silhouette Score (↑ higher = better)')
axes[0, 0].set_xlabel('K')
axes[0, 0].set_ylabel('Score')
axes[0, 0].axvline(list(k_range)[np.argmax(silhouette_scores)], 
                    color='red', linestyle='--')

# Calinski-Harabasz
axes[0, 1].plot(list(k_range), ch_scores, 'go-')
axes[0, 1].set_title('Calinski-Harabasz Index (↑ higher = better)')
axes[0, 1].set_xlabel('K')
axes[0, 1].set_ylabel('CH Index')
axes[0, 1].axvline(list(k_range)[np.argmax(ch_scores)], 
                    color='red', linestyle='--')

# Davies-Bouldin
axes[1, 0].plot(list(k_range), db_scores, 'ro-')
axes[1, 0].set_title('Davies-Bouldin Index (↓ lower = better)')
axes[1, 0].set_xlabel('K')
axes[1, 0].set_ylabel('DB Index')
axes[1, 0].axvline(list(k_range)[np.argmin(db_scores)], 
                    color='red', linestyle='--')

# Inertia / Elbow
axes[1, 1].plot(list(k_range), inertias, 'mo-')
axes[1, 1].set_title('Inertia/WCSS (Elbow Method)')
axes[1, 1].set_xlabel('K')
axes[1, 1].set_ylabel('WCSS')

for ax in axes.flatten():
    ax.grid(True, alpha=0.3)

plt.suptitle('Internal Clustering Metrics Comparison', fontsize=14)
plt.tight_layout()
plt.show()
```

---

## 📋 Metrics Comparison Table

```
┌──────────────────────┬──────────┬────────────────┬──────────────────┐
│ Metric               │ Optimal  │ Range          │ Key Property     │
├──────────────────────┼──────────┼────────────────┼──────────────────┤
│ Silhouette Score     │ Higher   │ [-1, 1]        │ Cohesion +       │
│                      │          │                │ Separation       │
├──────────────────────┼──────────┼────────────────┼──────────────────┤
│ Calinski-Harabasz    │ Higher   │ [0, ∞)         │ Between/Within   │
│                      │          │                │ variance ratio   │
├──────────────────────┼──────────┼────────────────┼──────────────────┤
│ Davies-Bouldin       │ Lower    │ [0, ∞)         │ Avg cluster      │
│                      │          │                │ similarity       │
├──────────────────────┼──────────┼────────────────┼──────────────────┤
│ Inertia (WCSS)       │ Lower    │ [0, ∞)         │ Within-cluster   │
│                      │ (elbow)  │                │ compactness      │
├──────────────────────┼──────────┼────────────────┼──────────────────┤
│ Assumptions          │          │                │                  │
│ Silhouette           │          │                │ Works for any    │
│                      │          │                │ cluster shape    │
│ CH & DB              │          │                │ Assumes convex   │
│                      │          │                │ clusters         │
│ WCSS                 │          │                │ Assumes spherical│
│                      │          │                │ clusters         │
└──────────────────────┴──────────┴────────────────┴──────────────────┘
```

---

## ❓ Revision Questions

1. **Compute the silhouette coefficient for a point with a(i) = 2.0 and b(i) = 5.0. What if a(i) = 4.0 and b(i) = 3.0? Interpret both results.**
   Show the formula and explain what positive vs. negative values mean.

2. **Compare the Silhouette Score, Calinski-Harabasz Index, and Davies-Bouldin Index. Which metric(s) would you use for non-convex clusters?**
   Discuss the assumptions each metric makes about cluster shapes.

3. **The elbow method for choosing K is sometimes ambiguous. Propose two alternative approaches and their advantages.**
   Consider silhouette analysis, gap statistic, or information-theoretic methods.

4. **A K-Means clustering gives Silhouette Score = 0.72 for K=3 and 0.68 for K=4. Which K would you choose and why? What other factors should you consider?**
   Discuss statistical significance, domain knowledge, and practical utility.

5. **Why does WCSS (inertia) always decrease as K increases? Why can't we just choose the largest K?**
   Explain the bias-complexity tradeoff and the concept of overfitting in clustering.

6. **Given a dataset clustered with K=5, the per-cluster silhouette scores are: [0.81, 0.73, 0.12, 0.78, 0.69]. What does the low score for cluster 3 suggest?**
   Discuss potential causes and remedies.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← AE Anomaly Detection](../09-Anomaly-Detection/06-autoencoders-anomaly-detection.md) | [📂 Unsupervised Learning](../README.md) | [External Metrics →](./02-external-metrics.md) |

---

> **Key Takeaway:** Internal metrics evaluate clustering without ground truth. Use Silhouette Score as the most versatile metric, Calinski-Harabasz for quick variance-ratio assessment, Davies-Bouldin for cluster similarity analysis, and the Elbow Method (WCSS) as a visual guide. Always use multiple metrics together — they can disagree, and consensus across metrics gives stronger evidence for the optimal K.
