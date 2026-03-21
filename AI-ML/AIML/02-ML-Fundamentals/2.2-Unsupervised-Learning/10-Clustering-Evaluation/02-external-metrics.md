# 🏷️ External Clustering Evaluation Metrics

> **Chapter 10.2 — Evaluating Clusters When Ground Truth Is Available**

---

## 📌 Overview

**External evaluation metrics** compare cluster assignments against **known ground truth labels**. While true labels are rarely available in real clustering tasks, external metrics are invaluable for **benchmarking algorithms**, **validating methods** on labeled datasets, and **comparing** different clustering approaches objectively.

### When to Use External Metrics

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  You have GROUND TRUTH LABELS when:                                  │
│  • Benchmarking on a labeled dataset (Iris, MNIST, etc.)             │
│  • Evaluating clustering as a pre-step for classification            │
│  • Validating a clustering algorithm before deploying                │
│  • Comparing multiple algorithms on the same data                   │
│  • Research papers / academic evaluation                             │
│                                                                      │
│  External metrics answer:                                            │
│  "How well does my clustering match the TRUE grouping?"              │
│                                                                      │
│  ⚠️ Note: Cluster labels don't need to MATCH ground truth labels.   │
│  Metrics are invariant to label permutation.                         │
│  (Cluster "0" matching class "2" is perfectly fine)                  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📐 1. Adjusted Rand Index (ARI)

### Rand Index

The **Rand Index** measures the agreement between two partitions by counting pairs:

```
               a + d
RI = ─────────────────────
      a + b + c + d

where (for all pairs of points):
  a = pairs in SAME cluster in BOTH partitions (True Positive)
  b = pairs in SAME cluster in predicted, DIFFERENT in true (False Positive)
  c = pairs in DIFFERENT clusters in predicted, SAME in true (False Negative)
  d = pairs in DIFFERENT clusters in BOTH partitions (True Negative)
```

### Adjusted Rand Index

The ARI corrects for **chance** — random clustering has ARI ≈ 0:

```
              RI - E[RI]
ARI = ────────────────────────
           max(RI) - E[RI]
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│ ADJUSTED RAND INDEX                                          │
│                                                              │
│  ARI = 1.0   → Perfect agreement with ground truth           │
│  ARI = 0.0   → Random clustering (no better than chance)     │
│  ARI < 0.0   → Worse than random (disagreement)              │
│                                                              │
│  Range: [-0.5, 1.0] (approximately)                          │
│                                                              │
│  Guidelines:                                                 │
│  • > 0.9  → Excellent                                        │
│  • 0.7-0.9 → Good                                            │
│  • 0.5-0.7 → Moderate                                        │
│  • < 0.5  → Poor                                             │
│                                                              │
│  Properties:                                                 │
│  ✅ Adjusted for chance                                      │
│  ✅ Symmetric: ARI(U,V) = ARI(V,U)                          │
│  ✅ Doesn't require same number of clusters                  │
│  ❌ Assumes equal-sized clusters perform better               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python

```python
from sklearn.metrics import adjusted_rand_score
from sklearn.cluster import KMeans
from sklearn.datasets import load_iris

iris = load_iris()
X, y_true = iris.data, iris.target

kmeans = KMeans(n_clusters=3, random_state=42, n_init=10)
y_pred = kmeans.fit_predict(X)

ari = adjusted_rand_score(y_true, y_pred)
print(f"Adjusted Rand Index: {ari:.4f}")
```

---

## 📐 2. Normalized Mutual Information (NMI)

### Mutual Information

**Mutual Information** measures how much knowing the cluster assignment reduces uncertainty about the true label (and vice versa):

```
                    |Cᵢ ∩ Tⱼ|         |Cᵢ ∩ Tⱼ| · N
MI(C, T) = Σᵢ Σⱼ  ─────────── · log ────────────────
                        N              |Cᵢ| · |Tⱼ|

where:
  Cᵢ = set of points in cluster i
  Tⱼ = set of points in true class j
  N  = total number of points
```

### Normalized Mutual Information

NMI normalizes MI to [0, 1]:

```
                    MI(C, T)
NMI(C, T) = ─────────────────────
              √(H(C) · H(T))

where H(C) and H(T) are entropies of the cluster and true partitions

Alternative normalizations exist:
  • 'arithmetic': (H(C) + H(T)) / 2
  • 'geometric':  √(H(C) · H(T))    ← default in sklearn
  • 'min':        min(H(C), H(T))
  • 'max':        max(H(C), H(T))
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│ NORMALIZED MUTUAL INFORMATION                                │
│                                                              │
│  NMI = 1.0  → Perfect agreement (knowing clusters =         │
│               knowing true labels)                           │
│  NMI = 0.0  → No mutual information (independent)           │
│                                                              │
│  Range: [0, 1]                                               │
│                                                              │
│  Properties:                                                 │
│  ✅ Normalized to [0, 1] — easy to interpret                 │
│  ✅ Symmetric: NMI(C,T) = NMI(T,C)                          │
│  ⚠️ Not adjusted for chance (can be high for random          │
│     clustering with many clusters)                           │
│  → Use Adjusted Mutual Information (AMI) for chance-adjusted │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python

```python
from sklearn.metrics import normalized_mutual_info_score, adjusted_mutual_info_score

nmi = normalized_mutual_info_score(y_true, y_pred)
ami = adjusted_mutual_info_score(y_true, y_pred)

print(f"NMI: {nmi:.4f}")
print(f"AMI: {ami:.4f}")  # Adjusted for chance
```

---

## 📐 3. Fowlkes-Mallows Index (FMI)

### Definition

The **Fowlkes-Mallows Index** is the geometric mean of precision and recall for pair-counting:

```
FMI = √(TP/(TP+FP) · TP/(TP+FN)) = √(Precision · Recall)

where:
  TP = pairs in same cluster AND same true class
  FP = pairs in same cluster but different true class
  FN = pairs in different clusters but same true class
```

### Interpretation

```
┌──────────────────────────────────────────────────────────────┐
│ FOWLKES-MALLOWS INDEX                                        │
│                                                              │
│  FMI = 1.0  → Perfect agreement                             │
│  FMI = 0.0  → No agreement                                  │
│                                                              │
│  Range: [0, 1]                                               │
│                                                              │
│  Properties:                                                 │
│  ✅ Based on precision and recall (familiar concepts)        │
│  ✅ Works for different numbers of clusters                  │
│  ⚠️ Not adjusted for chance                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python

```python
from sklearn.metrics import fowlkes_mallows_score

fmi = fowlkes_mallows_score(y_true, y_pred)
print(f"Fowlkes-Mallows Index: {fmi:.4f}")
```

---

## 📐 4. Homogeneity, Completeness, and V-Measure

### Definitions

These three metrics form a coherent evaluation framework:

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  HOMOGENEITY:                                                │
│  Each cluster contains only members of a single class.       │
│  "Are clusters PURE?"                                        │
│                                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐                         │
│  │ ● ● ●  │  │ ○ ○ ○  │  │ ▲ ▲ ▲  │  Homogeneity = 1.0     │
│  │ ● ● ●  │  │ ○ ○ ○  │  │ ▲ ▲ ▲  │  (each cluster=1 class)│
│  └────────┘  └────────┘  └────────┘                         │
│                                                              │
│  ┌────────┐  ┌────────┐                                     │
│  │ ● ○ ●  │  │ ○ ▲ ○  │  Homogeneity < 1.0                  │
│  │ ○ ● ○  │  │ ▲ ○ ▲  │  (clusters are mixed)               │
│  └────────┘  └────────┘                                     │
│                                                              │
│  COMPLETENESS:                                               │
│  All members of a given class are assigned to the same       │
│  cluster. "Are classes COMPLETE in one cluster?"             │
│                                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐                         │
│  │ ● ● ●  │  │ ○ ○ ○  │  │ ▲ ▲ ▲  │  Completeness = 1.0    │
│  │ ● ● ●  │  │ ○ ○ ○  │  │ ▲ ▲ ▲  │  (all of each class    │
│  └────────┘  └────────┘  └────────┘   in one cluster)       │
│                                                              │
│  ┌────────┐  ┌────────┐  ┌────────┐                         │
│  │ ● ● ●  │  │ ● ● ●  │  │ ○ ○ ○  │  Completeness < 1.0    │
│  └────────┘  └────────┘  └────────┘  (●'s split across 2    │
│                                       clusters)              │
│                                                              │
│  V-MEASURE = Harmonic mean of Homogeneity and Completeness  │
│                                                              │
│  V = 2 · (h · c) / (h + c)                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Python

```python
from sklearn.metrics import homogeneity_score, completeness_score, v_measure_score
from sklearn.metrics import homogeneity_completeness_v_measure

h = homogeneity_score(y_true, y_pred)
c = completeness_score(y_true, y_pred)
v = v_measure_score(y_true, y_pred)

print(f"Homogeneity:  {h:.4f}")
print(f"Completeness: {c:.4f}")
print(f"V-measure:    {v:.4f}")

# Or all at once:
h, c, v = homogeneity_completeness_v_measure(y_true, y_pred)
```

---

## 📊 Comprehensive Comparison

```python
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.mixture import GaussianMixture
from sklearn.metrics import (adjusted_rand_score, normalized_mutual_info_score,
                             fowlkes_mallows_score, v_measure_score)

# Algorithms to compare
algorithms = {
    'K-Means (K=3)': KMeans(n_clusters=3, random_state=42, n_init=10),
    'Agglom. (K=3)': AgglomerativeClustering(n_clusters=3),
    'GMM (K=3)':     GaussianMixture(n_components=3, random_state=42),
    'K-Means (K=5)': KMeans(n_clusters=5, random_state=42, n_init=10),
}

results = []
for name, alg in algorithms.items():
    if hasattr(alg, 'fit_predict'):
        labels = alg.fit_predict(X)
    else:
        alg.fit(X)
        labels = alg.predict(X)
    
    results.append({
        'Algorithm': name,
        'ARI': adjusted_rand_score(y_true, labels),
        'NMI': normalized_mutual_info_score(y_true, labels),
        'FMI': fowlkes_mallows_score(y_true, labels),
        'V-measure': v_measure_score(y_true, labels),
    })

import pandas as pd
df_results = pd.DataFrame(results)
print(df_results.to_string(index=False))
```

---

## 📋 Metrics Comparison Table

```
┌──────────────────────┬──────────┬──────────────┬──────────────────┐
│ Metric               │ Range    │ Chance-      │ Key Property     │
│                      │          │ Adjusted?    │                  │
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Adjusted Rand Index  │ [-0.5,1] │ ✅ Yes       │ Pair-counting    │
│ (ARI)                │          │              │ agreement        │
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Normalized MI (NMI)  │ [0, 1]   │ ❌ No        │ Information      │
│                      │          │              │ shared           │
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Adjusted MI (AMI)    │ [-1, 1]  │ ✅ Yes       │ Info-theoretic   │
│                      │          │              │ (chance-adjusted)│
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Fowlkes-Mallows      │ [0, 1]   │ ❌ No        │ √(Precision ×    │
│ (FMI)                │          │              │ Recall) for pairs│
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Homogeneity          │ [0, 1]   │ ❌ No        │ Cluster purity   │
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ Completeness         │ [0, 1]   │ ❌ No        │ Class coverage   │
├──────────────────────┼──────────┼──────────────┼──────────────────┤
│ V-measure            │ [0, 1]   │ ❌ No        │ Harmonic mean    │
│                      │          │              │ of H & C         │
└──────────────────────┴──────────┴──────────────┴──────────────────┘
```

---

## ❓ Revision Questions

1. **Why is the Adjusted Rand Index preferred over the plain Rand Index? What does "adjusted for chance" mean?**
   Explain what expected value under random clustering means and why correction matters.

2. **Compare NMI and AMI. On a dataset with 1000 points and K=50 clusters, which metric might misleadingly show high agreement with ground truth?**
   Discuss the over-estimation problem of NMI with many clusters.

3. **A clustering result has Homogeneity = 1.0 but Completeness = 0.3. What does this tell you about the clustering?**
   Describe a concrete scenario and explain what the V-measure would be.

4. **Compute the Rand Index for predicted labels [0,0,1,1,2] and true labels [A,A,A,B,B]. Show the pair-counting table.**
   Enumerate all 10 pairs and classify as TP, FP, FN, TN.

5. **Why can't external metrics be used alone to choose the best clustering algorithm for a production system?**
   Discuss the requirement for ground truth and the gap between benchmark and real-world performance.

6. **For the Iris dataset, compare K-Means, DBSCAN, and Agglomerative Clustering using all external metrics covered. Which algorithm best recovers the true species labels?**
   Discuss why certain algorithms may perform better based on the data structure.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Internal Metrics](./01-internal-metrics.md) | [📂 Unsupervised Learning](../README.md) | [Visual Inspection →](./03-visual-inspection.md) |

---

> **Key Takeaway:** External metrics evaluate how well clustering matches ground truth. Use **ARI** (chance-adjusted, pair-based) and **AMI** (chance-adjusted, information-theoretic) as primary metrics. Homogeneity and Completeness provide additional diagnostic insight. Always use chance-adjusted metrics to avoid being fooled by random agreement.
