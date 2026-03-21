# ⚖️ Comparison of Dimensionality Reduction Methods

> **Chapter 7.5 — PCA vs LDA vs t-SNE vs UMAP vs Autoencoders: When to Use What**

---

## 📌 Overview

Choosing the right dimensionality reduction method depends on your data characteristics, objectives, and constraints. This chapter provides a comprehensive comparison of all major methods covered in this unit, with decision guidelines and practical recommendations.

---

## 📊 Master Comparison Table

```
┌──────────────────┬──────────┬──────────┬──────────┬──────────┬──────────────┐
│    Property      │   PCA    │   LDA    │  t-SNE   │  UMAP    │ Autoencoder  │
├──────────────────┼──────────┼──────────┼──────────┼──────────┼──────────────┤
│ Type             │ Linear   │ Linear   │Nonlinear │Nonlinear │ Nonlinear    │
│ Supervised?      │ No       │ Yes      │ No       │ No/Yes   │ No           │
│ Max components   │ min(N,d) │ C-1      │ 2-3      │ Any      │ Any          │
│ Preserves global │ ✅ Yes   │ ✅ Yes   │ ❌ No    │ ✅ Yes*  │ ✅ Yes       │
│ Preserves local  │ ❌ Poor  │ ❌ Poor  │ ✅ Yes   │ ✅ Yes   │ ✅ Yes       │
│ New points       │ ✅ Yes   │ ✅ Yes   │ ❌ No    │ ✅ Yes   │ ✅ Yes       │
│ Scalability      │ ★★★★★   │ ★★★★☆   │ ★★☆☆☆   │ ★★★★☆   │ ★★★☆☆       │
│ Speed            │ Very fast│ Fast     │ Slow     │ Fast     │ Slow(train)  │
│ Deterministic    │ Yes      │ Yes      │ No       │ No*      │ No           │
│ Interpretable    │ High     │ High     │ Low      │ Low      │ Very Low     │
│ Hyperparameters  │ Few      │ Few      │ Moderate │ Moderate │ Many         │
│ Handles nonlinear│ ❌ No    │ ❌ No    │ ✅ Yes   │ ✅ Yes   │ ✅ Yes       │
│ Theory           │ Variance │ Fisher   │ KL div   │ Topology │ Reconstruction│
│ Primary use      │ General  │ Classify │ Visualize│ General  │ General      │
│ Implementation   │ sklearn  │ sklearn  │ sklearn  │ umap-    │ TF/PyTorch   │
│                  │          │          │          │ learn    │              │
└──────────────────┴──────────┴──────────┴──────────┴──────────┴──────────────┘
```

---

## 🔍 Detailed Method Profiles

### PCA — Principal Component Analysis

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ STRENGTHS                    ❌ WEAKNESSES               │
├──────────────────────────────────────────────────────────────┤
│ • Fast, deterministic            • Linear only               │
│ • No hyperparameters to tune     • Misses curved manifolds   │
│ • Well-understood theory         • Sensitive to scale        │
│ • Explained variance ratio       • Outlier sensitive         │
│ • Invertible (reconstruct)       • Assumes orthogonal axes   │
│ • Works as preprocessing step    • May not separate classes  │
│ • Scales to millions of points                              │
│                                                              │
│  BEST FOR: Preprocessing, noise reduction, initial           │
│  exploration, feature decorrelation, compression             │
└──────────────────────────────────────────────────────────────┘
```

### LDA — Linear Discriminant Analysis

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ STRENGTHS                    ❌ WEAKNESSES               │
├──────────────────────────────────────────────────────────────┤
│ • Uses class labels (supervised) • Requires labels           │
│ • Optimal for classification     • Max C-1 components        │
│ • Produces class-separable axes  • Assumes Gaussian classes  │
│ • Fast computation               • Assumes equal covariance  │
│ • Can serve as classifier too    • Linear boundaries only    │
│ • Well-suited for few classes    • Singular Sw if N < d      │
│                                                              │
│  BEST FOR: Supervised dimensionality reduction before        │
│  classification, face recognition (Fisherfaces)              │
└──────────────────────────────────────────────────────────────┘
```

### t-SNE — t-Distributed Stochastic Neighbor Embedding

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ STRENGTHS                    ❌ WEAKNESSES               │
├──────────────────────────────────────────────────────────────┤
│ • Excellent local structure      • Slow (O(N²) or O(NlogN)) │
│ • Reveals clusters beautifully   • No transform for new pts │
│ • Well-established, many papers  • Limited to 2D/3D         │
│ • Handles complex manifolds      • Global structure lost     │
│                                  • Non-deterministic         │
│                                  • Distances NOT meaningful  │
│                                  • Sensitive to perplexity   │
│                                                              │
│  BEST FOR: Visualization of cluster structure in 2D,         │
│  exploring high-dimensional datasets                        │
└──────────────────────────────────────────────────────────────┘
```

### UMAP — Uniform Manifold Approximation and Projection

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ STRENGTHS                    ❌ WEAKNESSES               │
├──────────────────────────────────────────────────────────────┤
│ • Fast (much faster than t-SNE)  • Less established          │
│ • Preserves global AND local     • Requires parameter tuning │
│ • Can embed new points           • Non-deterministic         │
│ • Any output dimensionality      • Less theoretical backing  │
│ • Supports supervised mode       │   than PCA                │
│ • Scales to millions of pts     • Can create misleading      │
│ • Works for general reduction    │   visual artifacts         │
│                                                              │
│  BEST FOR: General-purpose nonlinear reduction,              │
│  visualization at scale, preprocessing for ML                │
└──────────────────────────────────────────────────────────────┘
```

### Autoencoders

```
┌──────────────────────────────────────────────────────────────┐
│  ✅ STRENGTHS                    ❌ WEAKNESSES               │
├──────────────────────────────────────────────────────────────┤
│ • Maximum flexibility            • Requires neural network   │
│ • Handles ANY nonlinearity       • Many hyperparameters      │
│ • Customizable architecture      • Slow training (needs GPU) │
│ • Can embed new points           • Overfitting risk          │
│ • Extends to generation (VAE)    • Non-deterministic         │
│ • Domain-specific designs (CNN)  • Hard to interpret         │
│ • Foundation for deep learning   • Needs large datasets      │
│                                                              │
│  BEST FOR: Deep learning pipelines, image/text/sequence      │
│  data, when other methods fail, generative modeling          │
└──────────────────────────────────────────────────────────────┘
```

---

## 🧮 Computational Complexity Comparison

```
┌──────────────┬───────────────────┬──────────────────┬──────────────┐
│   Method     │ Time Complexity   │ Space Complexity │ Practical N  │
├──────────────┼───────────────────┼──────────────────┼──────────────┤
│ PCA (full)   │ O(d²N + d³)      │ O(dN)            │ Millions     │
│ PCA (random) │ O(dNk)           │ O(dN)            │ Millions     │
│ LDA          │ O(d²N + d³)      │ O(d²)            │ Millions     │
│ t-SNE exact  │ O(N²) per iter   │ O(N²)            │ ~5,000       │
│ t-SNE BH     │ O(N log N)/iter  │ O(N)             │ ~100,000     │
│ UMAP         │ O(N^1.14)        │ O(N)             │ Millions     │
│ Autoencoder  │ O(epochs·N·params)│ O(params)       │ Millions(GPU)│
└──────────────┴───────────────────┴──────────────────┴──────────────┘
```

---

## 🧪 Visual Comparison on Same Dataset

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.manifold import TSNE

# Load data
digits = load_digits()
X, y = digits.data, digits.target
X_scaled = StandardScaler().fit_transform(X)

# ═══════════════════════════════════════
# Apply all methods
# ═══════════════════════════════════════

# PCA
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

# LDA
lda = LinearDiscriminantAnalysis(n_components=2)
X_lda = lda.fit_transform(X_scaled, y)

# t-SNE
tsne = TSNE(n_components=2, perplexity=30, random_state=42, init='pca')
X_tsne = tsne.fit_transform(X_scaled)

# UMAP
try:
    import umap
    reducer = umap.UMAP(n_neighbors=15, min_dist=0.1, random_state=42)
    X_umap = reducer.fit_transform(X_scaled)
    has_umap = True
except ImportError:
    has_umap = False
    print("Install umap-learn: pip install umap-learn")

# ═══════════════════════════════════════
# Visualize
# ═══════════════════════════════════════

n_plots = 4 if has_umap else 3
fig, axes = plt.subplots(1, n_plots, figsize=(5*n_plots, 5))

methods = [
    ('PCA', X_pca),
    ('LDA', X_lda),
    ('t-SNE', X_tsne),
]
if has_umap:
    methods.append(('UMAP', X_umap))

for ax, (name, X_emb) in zip(axes, methods):
    scatter = ax.scatter(X_emb[:, 0], X_emb[:, 1], 
                         c=y, cmap='tab10', s=5, alpha=0.6)
    ax.set_title(name, fontsize=14, fontweight='bold')
    ax.set_xticks([])
    ax.set_yticks([])

plt.suptitle('Dimensionality Reduction Methods Comparison (Digits)', 
             fontsize=16, y=1.02)
plt.tight_layout()
plt.savefig('methods_comparison.png', dpi=150, bbox_inches='tight')
plt.show()
```

### What You'll Typically See

```
  PCA                    LDA                   t-SNE                  UMAP
  ┌────────────┐        ┌────────────┐        ┌────────────┐        ┌────────────┐
  │  ◆◆◇◇      │        │  ◆◆   ◇◇   │        │  ◆◆  ◇◇    │        │  ◆◆   ◇◇   │
  │ ◆◆◇◇●●     │        │            │        │      ●●    │        │            │
  │  ●●  ▲▲    │        │  ●●   ▲▲   │        │  ●●        │        │  ●●   ▲▲   │
  │ ▲▲ overlaps │        │            │        │      ▲▲    │        │            │
  │             │        │ Clean sep.  │        │ Clear clust│        │ Clear clust│
  └────────────┘        └────────────┘        └────────────┘        └────────────┘
  • Some overlap        • Best class sep     • Best local          • Good local
  • Global struct ok    • Uses labels!       • No global info      • Good global
  • Very fast           • Max C-1 dims       • Slow                • Fast
```

---

## 🔀 Decision Flowchart

```
                              START
                                │
                                ▼
                    ┌─────────────────────┐
                    │ Do you have labels? │
                    └──────┬──────┬───────┘
                           │      │
                         Yes      No
                           │      │
                           ▼      ▼
                ┌──────────────┐  ┌───────────────────┐
                │ Goal is      │  │ What is your goal?│
                │ classification│  └───┬───────┬───────┘
                │ preprocessing?│      │       │
                └──────┬───────┘  Visualization  General
                       │          │           Reduction
                     Yes          │              │
                       │          │              │
                       ▼          │              ▼
                    ┌──────┐      │     ┌────────────────┐
                    │ LDA  │      │     │ Data size?     │
                    └──────┘      │     └──┬──────┬──────┘
                                  │        │      │
                                  ▼     Small   Large
                         ┌──────────────┐  │      │
                         │ Data size?   │  │      │
                         └──┬───────┬───┘  │      │
                            │       │      │      │
                         Small    Large    │      │
                         (<10K)  (>10K)    │      │
                            │       │      │      │
                            ▼       ▼      ▼      ▼
                        ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
                        │t-SNE │ │UMAP  │ │ PCA  │ │ UMAP │
                        └──────┘ └──────┘ └──────┘ └──────┘
                                                      │
                                          ┌───────────┘
                                          ▼
                                  ┌──────────────┐
                                  │ Linear data? │
                                  └──┬───────┬───┘
                                     │       │
                                   Yes       No
                                     │       │
                                     ▼       ▼
                                 ┌──────┐ ┌────────────┐
                                 │ PCA  │ │Autoencoder/│
                                 └──────┘ │   UMAP     │
                                          └────────────┘
```

---

## 🎯 When to Use Each Method

### Use PCA When:
- You need a **fast baseline** for dimensionality reduction
- You want **interpretable** components (loadings)
- You need **preprocessing** before another algorithm (t-SNE, clustering)
- Your data relationships are approximately **linear**
- You need **explained variance** for reporting
- You need to reduce dimensions before LDA (to avoid singular Sw)

### Use LDA When:
- You have **class labels** and want supervised reduction
- Your goal is **classification** (LDA as classifier or preprocessing)
- You need dimensions that **separate classes**
- Number of classes is small (C−1 limit is acceptable)
- Data approximately follows **Gaussian** distributions per class

### Use t-SNE When:
- You want beautiful **2D/3D visualizations** of high-D data
- You care about **local neighborhood** structure
- Dataset is **small to moderate** (< 10,000 samples)
- You don't need to embed **new points**
- You want to **explore** cluster structure without labels

### Use UMAP When:
- You need **fast, scalable** nonlinear reduction
- You want both **local and global** structure preserved
- You need to **embed new points** (`.transform()`)
- Dataset is **large** (> 10,000 samples)
- You need output dims **> 3** (general reduction, not just visualization)
- You want to use as **preprocessing for ML**

### Use Autoencoders When:
- Data has **complex nonlinear** structure that others can't capture
- You're working within a **deep learning** pipeline
- You need **domain-specific architectures** (CNN for images, RNN for sequences)
- You want **generation** capability (VAE)
- Other methods have **failed** to provide useful representations
- You have **sufficient data** and **compute resources**

---

## 📊 Performance Benchmarks (Typical)

```
Dataset: MNIST (60,000 samples, 784 features, 10 classes)

┌──────────────┬──────────┬───────────────┬────────────────────┐
│   Method     │  Time    │ KNN Accuracy  │ Visual Quality     │
│              │          │ (after reduce)│ (subjective)       │
├──────────────┼──────────┼───────────────┼────────────────────┤
│ PCA (50D)    │ ~1 sec   │ ~97.2%        │ ★★☆☆☆ (overlapping)│
│ LDA (9D)    │ ~2 sec   │ ~96.8%        │ ★★★☆☆ (separated) │
│ t-SNE (2D)  │ ~5 min   │ ~97.5%        │ ★★★★★ (beautiful) │
│ UMAP (2D)   │ ~30 sec  │ ~97.3%        │ ★★★★★ (beautiful) │
│ UMAP (50D)  │ ~45 sec  │ ~97.8%        │ N/A (>3D)          │
│ AE (32D)    │ ~3 min   │ ~97.6%        │ ★★★★☆ (good)      │
└──────────────┴──────────┴───────────────┴────────────────────┘

Note: Times are approximate and depend on hardware.
KNN accuracy is with k=5 nearest neighbors classifier.
```

---

## 🔄 Combining Methods (Common Pipelines)

### Pipeline 1: PCA → t-SNE (Standard for visualization)

```python
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE

# PCA reduces noise and speeds up t-SNE
X_pca_50 = PCA(n_components=50).fit_transform(X_scaled)
X_vis = TSNE(n_components=2, perplexity=30).fit_transform(X_pca_50)
```

### Pipeline 2: PCA → LDA (Handles high-D for classification)

```python
from sklearn.decomposition import PCA
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.pipeline import Pipeline

pipe = Pipeline([
    ('pca', PCA(n_components=50)),
    ('lda', LinearDiscriminantAnalysis(n_components=9))
])
X_lda = pipe.fit_transform(X_scaled, y)
```

### Pipeline 3: UMAP → Clustering

```python
import umap
from sklearn.cluster import HDBSCAN

X_umap = umap.UMAP(n_components=10, n_neighbors=30).fit_transform(X_scaled)
clusters = HDBSCAN(min_cluster_size=50).fit_predict(X_umap)
```

### Pipeline 4: Autoencoder → Classifier

```python
# Train autoencoder, extract encoder
# Use encoder output as features for any classifier
X_features = encoder.predict(X_scaled)  # From trained autoencoder
# Feed X_features into SVM, Random Forest, etc.
```

---

## 📋 Quick Reference Card

```
╔══════════════════════════════════════════════════════════════════════╗
║                 DIMENSIONALITY REDUCTION CHEAT SHEET               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  FAST + SIMPLE    → PCA         (always a good starting point)      ║
║  WITH LABELS      → LDA         (for classification tasks)          ║
║  VISUALIZATION    → t-SNE/UMAP  (2D/3D scatter plots)              ║
║  LARGE DATA       → PCA or UMAP (scalable options)                  ║
║  NONLINEAR        → UMAP or AE  (curved manifolds)                  ║
║  NEW POINTS       → PCA/LDA/UMAP/AE  (NOT t-SNE)                   ║
║  DEEP LEARNING    → Autoencoder (integrate with neural nets)        ║
║  GENERATION       → VAE         (sample new data points)            ║
║                                                                      ║
║  RULE OF THUMB:                                                      ║
║  1. Start with PCA → check if linear reduction suffices             ║
║  2. If visualization needed → try UMAP (faster) or t-SNE            ║
║  3. If classification → try LDA                                      ║
║  4. If nothing works → try Autoencoder                               ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## ❓ Revision Questions

1. **Compare PCA and LDA in terms of their objective functions, assumptions, and the types of information they preserve.**
   Discuss variance maximization vs. class separability, and when each is preferable.

2. **You have a dataset with 100,000 samples, 500 features, and no labels. You want to visualize it in 2D. Which method would you choose and why? What preprocessing would you apply?**
   Consider computational constraints, scalability, and the role of PCA preprocessing.

3. **Why can t-SNE not embed new data points, while PCA, UMAP, and autoencoders can?**
   Discuss parametric vs. non-parametric mappings and optimization strategies.

4. **A colleague claims "UMAP is strictly better than t-SNE so we should always use UMAP." Do you agree? In what scenarios might t-SNE be preferred?**
   Consider established literature, specific visualization tasks, and methodological rigor.

5. **Design a complete dimensionality reduction pipeline for a dataset of 50,000 medical images (256×256 pixels, 3 channels) with 5 disease categories. Justify each step.**
   Consider the data type, dataset size, available labels, and the final objective.

6. **Create a decision tree for choosing a dimensionality reduction method based on: (a) availability of labels, (b) dataset size, (c) linearity of data, (d) need for new point embedding.**
   Synthesize the information from this chapter into a practical decision framework.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Autoencoders Preview](./04-autoencoders-preview.md) | [📂 Unsupervised Learning](../README.md) | [Association Rules →](../08-Association-Rules/01-market-basket-analysis.md) |

---

> **Key Takeaway:** No single dimensionality reduction method dominates all scenarios. Start with PCA for a fast baseline, use LDA when labels are available, UMAP for scalable nonlinear reduction, t-SNE for publication-quality 2D visualizations, and autoencoders when maximum flexibility is needed. In practice, methods are often combined in pipelines (PCA → t-SNE, PCA → LDA, UMAP → clustering).
