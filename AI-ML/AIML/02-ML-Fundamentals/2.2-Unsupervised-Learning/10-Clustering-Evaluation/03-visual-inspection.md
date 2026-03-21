# 👁️ Visual Inspection for Clustering Evaluation

> **Chapter 10.3 — Seeing Is Believing: Visualization Techniques for Clusters**

---

## 📌 Overview

**Visual inspection** complements quantitative metrics by providing intuitive understanding of cluster quality. Visualization reveals patterns, overlap, outliers, and structures that numbers alone cannot capture. For high-dimensional data, dimensionality reduction techniques (PCA, t-SNE, UMAP) enable visualization in 2D/3D.

---

## 📊 1. Scatter Plots (2D Cluster Visualization)

### Direct Feature Plots

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# Generate data
X, y_true = make_blobs(n_samples=500, centers=4, 
                        cluster_std=0.8, random_state=42)

# Cluster
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
labels = kmeans.fit_predict(X)
centers = kmeans.cluster_centers_

# Scatter plot
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Ground truth
axes[0].scatter(X[:, 0], X[:, 1], c=y_true, cmap='tab10', s=20, alpha=0.7)
axes[0].set_title('Ground Truth', fontsize=13)

# Predicted clusters
scatter = axes[1].scatter(X[:, 0], X[:, 1], c=labels, cmap='tab10', s=20, alpha=0.7)
axes[1].scatter(centers[:, 0], centers[:, 1], c='red', marker='X', 
                s=200, edgecolors='black', linewidths=2, label='Centroids')
axes[1].set_title('K-Means Clusters', fontsize=13)
axes[1].legend()

for ax in axes:
    ax.set_xlabel('Feature 1')
    ax.set_ylabel('Feature 2')
    ax.grid(True, alpha=0.3)

plt.suptitle('Cluster Visualization: Ground Truth vs Predicted', fontsize=14)
plt.tight_layout()
plt.show()
```

### Decision Boundary Visualization

```python
from matplotlib.colors import ListedColormap

fig, ax = plt.subplots(figsize=(10, 8))

# Create mesh
h = 0.2
x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                      np.arange(y_min, y_max, h))

# Predict cluster for each mesh point
Z = kmeans.predict(np.c_[xx.ravel(), yy.ravel()])
Z = Z.reshape(xx.shape)

# Plot
ax.contourf(xx, yy, Z, alpha=0.2, cmap='tab10')
ax.contour(xx, yy, Z, colors='gray', linewidths=0.5)
ax.scatter(X[:, 0], X[:, 1], c=labels, cmap='tab10', s=20, 
           edgecolors='k', linewidths=0.3)
ax.scatter(centers[:, 0], centers[:, 1], c='red', marker='X', 
           s=200, edgecolors='black', linewidths=2)
ax.set_title('K-Means Decision Boundaries')
plt.tight_layout()
plt.show()
```

---

## 📊 2. Silhouette Plot

The silhouette plot shows the silhouette coefficient for **each point**, grouped by cluster:

```
  Cluster 0  ████████████████████████  (avg = 0.78)
  Cluster 1  ███████████████████████   (avg = 0.72)
  Cluster 2  ██████████████████████████████  (avg = 0.85)
  Cluster 3  ████████                  (avg = 0.31) ← Problem!

  ──────────────────────────────────────────────────────
  -0.2  0.0   0.2   0.4   0.6   0.8   1.0
               ▲
          Mean silhouette = 0.67
```

```
┌──────────────────────────────────────────────────────────────┐
│ READING A SILHOUETTE PLOT:                                   │
│                                                              │
│ ✅ Good clusters: Wide bars, roughly equal size              │
│ ❌ Bad clusters:  Narrow bars, reaching into negative        │
│ ⚠️ Uneven:       Very different widths → imbalanced clusters │
│                                                              │
│ Each "knife" shape = one cluster                             │
│ Width = number of points                                     │
│ Length = silhouette coefficient                               │
│ Red dashed line = overall average                            │
│                                                              │
│ All bars should extend past the average line!                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```python
from sklearn.metrics import silhouette_samples, silhouette_score

def plot_silhouette(X, labels, title="Silhouette Plot"):
    """Create a detailed silhouette plot."""
    n_clusters = len(np.unique(labels))
    sample_silhouettes = silhouette_samples(X, labels)
    avg_score = silhouette_score(X, labels)
    
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 6))
    
    # Silhouette plot
    y_lower = 10
    for i in range(n_clusters):
        cluster_silhouettes = sample_silhouettes[labels == i]
        cluster_silhouettes.sort()
        
        size_cluster = len(cluster_silhouettes)
        y_upper = y_lower + size_cluster
        
        color = plt.cm.tab10(i / n_clusters)
        ax1.fill_betweenx(np.arange(y_lower, y_upper), 0, 
                         cluster_silhouettes, facecolor=color, alpha=0.7)
        ax1.text(-0.05, y_lower + 0.5 * size_cluster, f'C{i}')
        y_lower = y_upper + 10
    
    ax1.axvline(avg_score, color='red', linestyle='--', 
                label=f'Average: {avg_score:.3f}')
    ax1.set_xlabel('Silhouette Coefficient')
    ax1.set_ylabel('Cluster')
    ax1.set_title(title)
    ax1.legend()
    ax1.set_xlim([-0.2, 1.0])
    
    # Scatter plot
    scatter = ax2.scatter(X[:, 0], X[:, 1], c=labels, cmap='tab10', 
                          s=20, alpha=0.7)
    ax2.set_title('Cluster Assignments')
    ax2.set_xlabel('Feature 1')
    ax2.set_ylabel('Feature 2')
    
    plt.tight_layout()
    plt.show()

# Compare K values
for k in [2, 3, 4, 5]:
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labs = km.fit_predict(X)
    plot_silhouette(X, labs, title=f'Silhouette Plot (K={k})')
```

---

## 📊 3. PCA / t-SNE / UMAP for High-Dimensional Data

When data has more than 2-3 features, we must **reduce dimensions** before plotting:

```
┌──────────────────────────────────────────────────────────────┐
│           HIGH-D VISUALIZATION PIPELINE                      │
│                                                              │
│  High-D Data (d features)                                    │
│       │                                                      │
│       ├──► PCA → 2D        (fast, linear, global structure)  │
│       ├──► t-SNE → 2D      (slow, nonlinear, local clusters)│
│       └──► UMAP → 2D       (fast, nonlinear, both local/    │
│                              global)                         │
│       │                                                      │
│       ▼                                                      │
│  Color by cluster labels → Scatter plot                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```python
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits
from sklearn.cluster import KMeans

# High-dimensional data
digits = load_digits()
X_hd, y_true = digits.data, digits.target

# Cluster
kmeans = KMeans(n_clusters=10, random_state=42, n_init=10)
labels = kmeans.fit_predict(X_hd)

# Reduce for visualization
pca_2d = PCA(n_components=2).fit_transform(X_hd)
tsne_2d = TSNE(n_components=2, perplexity=30, random_state=42, 
               init='pca').fit_transform(X_hd)

# Compare
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

axes[0, 0].scatter(pca_2d[:, 0], pca_2d[:, 1], c=y_true, cmap='tab10', s=5)
axes[0, 0].set_title('PCA — True Labels')

axes[0, 1].scatter(pca_2d[:, 0], pca_2d[:, 1], c=labels, cmap='tab10', s=5)
axes[0, 1].set_title('PCA — K-Means Clusters')

axes[1, 0].scatter(tsne_2d[:, 0], tsne_2d[:, 1], c=y_true, cmap='tab10', s=5)
axes[1, 0].set_title('t-SNE — True Labels')

axes[1, 1].scatter(tsne_2d[:, 0], tsne_2d[:, 1], c=labels, cmap='tab10', s=5)
axes[1, 1].set_title('t-SNE — K-Means Clusters')

for ax in axes.flatten():
    ax.set_xticks([])
    ax.set_yticks([])

plt.suptitle('High-D Visualization: True Labels vs Clusters', fontsize=14)
plt.tight_layout()
plt.show()
```

---

## 📊 4. Elbow Plot

```python
def plot_elbow(X, k_range=range(1, 11)):
    """Elbow method for optimal K selection."""
    inertias = []
    for k in k_range:
        km = KMeans(n_clusters=k, random_state=42, n_init=10)
        km.fit(X)
        inertias.append(km.inertia_)
    
    plt.figure(figsize=(8, 5))
    plt.plot(list(k_range), inertias, 'bo-', linewidth=2, markersize=8)
    plt.xlabel('Number of Clusters (K)', fontsize=12)
    plt.ylabel('Inertia (WCSS)', fontsize=12)
    plt.title('Elbow Method for Optimal K', fontsize=14)
    plt.grid(True, alpha=0.3)
    
    # Annotate elbow (manual or use kneed library)
    plt.tight_layout()
    plt.show()

plot_elbow(X)
```

---

## 📊 5. Dendrogram Inspection (Hierarchical Clustering)

```python
from scipy.cluster.hierarchy import dendrogram, linkage

def plot_dendrogram(X, method='ward', max_d=None, truncate_mode='level', p=5):
    """Dendrogram for hierarchical clustering analysis."""
    Z = linkage(X, method=method)
    
    plt.figure(figsize=(14, 7))
    dendrogram(Z, truncate_mode=truncate_mode, p=p,
               leaf_rotation=90, leaf_font_size=10,
               show_leaf_counts=True)
    
    if max_d:
        plt.axhline(y=max_d, color='r', linestyle='--', 
                    label=f'Cut at d={max_d}')
        plt.legend()
    
    plt.xlabel('Sample Index or (Cluster Size)')
    plt.ylabel('Distance')
    plt.title(f'Dendrogram ({method} linkage)')
    plt.tight_layout()
    plt.show()

plot_dendrogram(X, method='ward', max_d=10)
```

```
READING A DENDROGRAM:
                    ┌────────────────────┐
                    │                    │  ← Large jump =
               ┌────┤               ┌───┤     natural split
          ┌────┤    │          ┌────┤   │
          │    │    │     ┌────┤    │   │
    ──────┤    ├────┘     │    │    ├───┘
          │    │          │    ├────┘
          └────┘          └────┘

  • Horizontal line height = distance at which clusters merge
  • Long vertical lines = well-separated clusters
  • Short vertical lines = close/overlapping clusters
  • Cut the dendrogram horizontally to get K clusters
```

---

## 📊 6. Practical Visualization Pipeline

```python
def comprehensive_cluster_evaluation(X, labels, y_true=None, 
                                      method_name="Clustering"):
    """
    Complete visual evaluation pipeline for clustering results.
    """
    from sklearn.metrics import silhouette_score, silhouette_samples
    from sklearn.decomposition import PCA
    
    n_clusters = len(np.unique(labels[labels >= 0]))
    d = X.shape[1]
    
    # Reduce to 2D if needed
    if d > 2:
        pca = PCA(n_components=2)
        X_2d = pca.fit_transform(X)
        var_explained = sum(pca.explained_variance_ratio_) * 100
    else:
        X_2d = X
        var_explained = 100
    
    # Setup figure
    n_cols = 3 if y_true is not None else 2
    fig, axes = plt.subplots(1, n_cols, figsize=(6*n_cols, 5))
    
    # Plot 1: Clusters in 2D
    axes[0].scatter(X_2d[:, 0], X_2d[:, 1], c=labels, cmap='tab10', 
                    s=15, alpha=0.7)
    axes[0].set_title(f'{method_name} ({n_clusters} clusters)\n'
                      f'({var_explained:.1f}% variance in 2D)')
    
    # Plot 2: Silhouette
    if n_clusters > 1:
        s_samples = silhouette_samples(X, labels)
        s_avg = silhouette_score(X, labels)
        
        y_lower = 10
        for i in range(n_clusters):
            sil = np.sort(s_samples[labels == i])
            y_upper = y_lower + len(sil)
            color = plt.cm.tab10(i / n_clusters)
            axes[1].fill_betweenx(np.arange(y_lower, y_upper), 0, sil,
                                 facecolor=color, alpha=0.7)
            y_lower = y_upper + 10
        
        axes[1].axvline(s_avg, color='red', linestyle='--')
        axes[1].set_title(f'Silhouette (avg={s_avg:.3f})')
        axes[1].set_xlabel('Silhouette Coefficient')
    
    # Plot 3: Ground truth comparison (if available)
    if y_true is not None:
        axes[2].scatter(X_2d[:, 0], X_2d[:, 1], c=y_true, cmap='tab10',
                        s=15, alpha=0.7)
        axes[2].set_title('Ground Truth')
    
    plt.suptitle(f'{method_name} Evaluation', fontsize=14)
    plt.tight_layout()
    plt.show()

# Usage
comprehensive_cluster_evaluation(X, labels, y_true, "K-Means (K=4)")
```

---

## 📋 Summary Table

| Visualization | What It Shows | When to Use |
|---------------|---------------|-------------|
| **Scatter Plot** | Cluster assignments in 2D | Low-D data (d ≤ 3) |
| **Decision Boundary** | Cluster regions | K-Means, GMM evaluation |
| **Silhouette Plot** | Per-point cluster quality | Choosing K, finding bad clusters |
| **PCA Plot** | Linear projection to 2D | Quick overview of high-D data |
| **t-SNE Plot** | Nonlinear 2D embedding | Detailed local structure |
| **UMAP Plot** | Nonlinear 2D embedding | Large datasets, global+local |
| **Elbow Plot** | WCSS vs K | Choosing K for K-Means |
| **Dendrogram** | Hierarchical merging | Choosing K, understanding hierarchy |

---

## ❓ Revision Questions

1. **Why is visual inspection important even when quantitative metrics are available?**
   Discuss what visualizations reveal that numbers cannot.

2. **How do you interpret a silhouette plot where one cluster has many negative values?**
   Explain what this means and what actions you would take.

3. **When visualizing 50-dimensional clustering results, which technique (PCA, t-SNE, UMAP) would you use and why?**
   Consider speed, structure preservation, and interpretability.

4. **Design a comprehensive visual evaluation pipeline for evaluating DBSCAN results on a 20-dimensional dataset with noise points.**
   Specify the plots, dimensionality reduction, and how to handle noise labels.

5. **A dendrogram shows two very long vertical lines at distances 5 and 15. All other merges occur at distances < 3. How many clusters would you choose?**
   Explain the cutting strategy and what the long lines represent.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← External Metrics](./02-external-metrics.md) | [📂 Unsupervised Learning](../README.md) | [Domain Knowledge →](./04-domain-knowledge.md) |

---

> **Key Takeaway:** Visual inspection is an indispensable complement to quantitative metrics. Scatter plots reveal spatial patterns, silhouette plots diagnose per-cluster quality, dimensionality reduction enables high-D visualization, and dendrograms guide hierarchical decisions. Always combine multiple visualizations with metrics for a complete picture of clustering quality.
