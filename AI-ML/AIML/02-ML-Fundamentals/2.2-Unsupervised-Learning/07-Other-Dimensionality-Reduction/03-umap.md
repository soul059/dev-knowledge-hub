# 🗺️ UMAP (Uniform Manifold Approximation and Projection)

> **Chapter 7.3 — Topologically-Grounded Nonlinear Dimensionality Reduction**

---

## 📌 Overview

**UMAP** (Uniform Manifold Approximation and Projection) is a modern nonlinear dimensionality reduction technique developed by **Leland McInnes, John Healy, and James Melville (2018)**. Grounded in **topological data analysis** and **Riemannian geometry**, UMAP provides theoretically justified embeddings that often outperform t-SNE in speed, scalability, and preservation of both local and global structure.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  1. Build a weighted graph (fuzzy simplicial set) in HIGH-D          │
│     based on local neighborhoods                                    │
│                                                                      │
│  2. Build a similar weighted graph in LOW-D                          │
│                                                                      │
│  3. Optimize the LOW-D graph to be as close to the HIGH-D            │
│     graph as possible (cross-entropy minimization)                   │
│                                                                      │
│  Result: A low-dimensional embedding that preserves both             │
│          LOCAL and GLOBAL topological structure                       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Mathematical Foundation

### Topological Data Analysis Intuition

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MANIFOLD ASSUMPTION:                                           │
│  Data lies on or near a low-dimensional manifold embedded       │
│  in high-dimensional space.                                     │
│                                                                 │
│  High-D Space:                    Underlying Manifold:          │
│  ┌──────────────────┐            ┌──────────────────┐           │
│  │      ●  ●        │            │                  │           │
│  │    ●  ●  ●       │  UMAP     │  ● ─ ● ─ ●      │           │
│  │  ●       ●  ●    │ ────────► │  │       │       │           │
│  │    ●  ●     ●    │           │  ● ─ ● ─ ●      │           │
│  │      ● ●  ●      │           │                  │           │
│  └──────────────────┘            └──────────────────┘           │
│                                                                 │
│  UMAP tries to learn and preserve this manifold structure       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: Construct High-Dimensional Graph (Fuzzy Simplicial Set)

#### 1a. Find k-Nearest Neighbors

For each point xᵢ, find its **k nearest neighbors** (parameter `n_neighbors`).

#### 1b. Compute Local Distances

Define a local distance metric using the distance to each point's nearest neighbor (ρᵢ) and a normalization factor (σᵢ):

```
                    ╱ max(0, d(xᵢ, xⱼ) - ρᵢ) ╲
wᵢⱼ = exp│─ ──────────────────────────── │
                    ╲           σᵢ              ╱
```

where:
- **d(xᵢ, xⱼ)** — distance between points i and j
- **ρᵢ** — distance from xᵢ to its nearest neighbor
- **σᵢ** — normalization factor (found via binary search to match `log₂(n_neighbors)`)

```
┌────────────────────────────────────────────────────────┐
│ INTUITION:                                             │
│                                                        │
│ • ρᵢ ensures every point has at least one connected    │
│   neighbor (connectivity constraint)                   │
│                                                        │
│ • σᵢ adapts the scale per point (like perplexity       │
│   in t-SNE, but theoretically motivated)               │
│                                                        │
│ • Each point "sees" the space at its own scale          │
│   (local metric assumption from Riemannian geometry)   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

#### 1c. Symmetrize the Graph

Combine the asymmetric weights using a **fuzzy union** (probabilistic OR):

```
wᵢⱼ_symmetric = wᵢⱼ + wⱼᵢ - wᵢⱼ · wⱼᵢ
```

This is the t-norm for fuzzy set union, equivalent to:

```
P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
```

### Step 2: Construct Low-Dimensional Graph

In the embedding space, define edge weights using a smooth approximation:

```
                     1
vᵢⱼ = ──────────────────────────────
       1 + a · ‖yᵢ - yⱼ‖²ᵇ
```

where **a** and **b** are parameters derived from `min_dist`:

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  min_dist controls how tightly points cluster:             │
│                                                            │
│  Small min_dist (0.001):     Large min_dist (0.99):        │
│  ┌──────────────────┐       ┌──────────────────┐           │
│  │  ●●●   ○○○       │       │  ● ● ●  ○ ○ ○    │           │
│  │  ●●●   ○○○       │       │  ● ● ●  ○ ○ ○    │           │
│  │  ●●●   ○○○       │       │  ● ● ●  ○ ○ ○    │           │
│  │ Very tight,       │       │ Spread out,       │           │
│  │ clumpy clusters   │       │ uniform clusters  │           │
│  └──────────────────┘       └──────────────────┘           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Step 3: Cross-Entropy Optimization

Minimize the **fuzzy set cross-entropy** between high-D graph (W) and low-D graph (V):

```
C = Σᵢⱼ [ wᵢⱼ · log(wᵢⱼ/vᵢⱼ) + (1-wᵢⱼ) · log((1-wᵢⱼ)/(1-vᵢⱼ)) ]
        ├────────────────────┤   ├──────────────────────────────────┤
           Attractive term            Repulsive term
```

```
┌──────────────────────────────────────────────────────────────┐
│ CROSS-ENTROPY vs KL DIVERGENCE (t-SNE):                      │
│                                                              │
│ t-SNE:  KL(P‖Q) = Σ pᵢⱼ log(pᵢⱼ/qᵢⱼ)                      │
│   → Only has attractive term                                 │
│   → Repulsion comes from normalization of qᵢⱼ                │
│   → Requires computing ALL pairwise distances                │
│                                                              │
│ UMAP: Cross-entropy has BOTH attractive AND repulsive terms  │
│   → Attractive: connected points should be close             │
│   → Repulsive: non-connected points should be far            │
│   → Can use negative sampling (very efficient!)              │
│   → Preserves more global structure                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 UMAP Algorithm: Complete Pipeline

```
╔══════════════════════════════════════════════════════════════════╗
║                     UMAP ALGORITHM                              ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  INPUT: X (N×d), n_neighbors, min_dist, n_components             ║
║                                                                  ║
║  PHASE 1: Build High-D Graph                                     ║
║  ┌──────────────────────────────────────────────────────┐        ║
║  │ 1. Compute k-NN graph (k = n_neighbors)             │        ║
║  │ 2. For each point, compute ρᵢ (nearest neighbor dist)│        ║
║  │ 3. Find σᵢ via binary search (match target entropy)  │        ║
║  │ 4. Compute edge weights wᵢⱼ                         │        ║
║  │ 5. Symmetrize: wᵢⱼ + wⱼᵢ - wᵢⱼ·wⱼᵢ                 │        ║
║  └──────────────────────────────────────────────────────┘        ║
║                                                                  ║
║  PHASE 2: Optimize Low-D Embedding                               ║
║  ┌──────────────────────────────────────────────────────┐        ║
║  │ 1. Initialize embedding (spectral or random)         │        ║
║  │ 2. Compute a, b from min_dist                        │        ║
║  │ 3. For n_epochs iterations:                          │        ║
║  │    a. Sample positive edge (i,j) with prob ∝ wᵢⱼ    │        ║
║  │    b. Apply attractive force between yᵢ and yⱼ      │        ║
║  │    c. Sample negative examples (random non-neighbors)│        ║
║  │    d. Apply repulsive force                          │        ║
║  │    e. Update positions with SGD                      │        ║
║  └──────────────────────────────────────────────────────┘        ║
║                                                                  ║
║  OUTPUT: Y (N × n_components)                                    ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 🎛️ Key Parameters

```
┌──────────────┬────────────────┬──────────────────────────────────────┐
│ Parameter    │ Default        │ Effect                               │
├──────────────┼────────────────┼──────────────────────────────────────┤
│ n_neighbors  │ 15             │ Size of local neighborhood           │
│              │                │ Small → local structure              │
│              │                │ Large → global structure             │
├──────────────┼────────────────┼──────────────────────────────────────┤
│ min_dist     │ 0.1            │ Minimum distance in embedding        │
│              │                │ Small → tight clusters               │
│              │                │ Large → spread out                   │
├──────────────┼────────────────┼──────────────────────────────────────┤
│ n_components │ 2              │ Embedding dimensionality             │
│              │                │ Can be > 3 (unlike t-SNE)            │
├──────────────┼────────────────┼──────────────────────────────────────┤
│ metric       │ 'euclidean'    │ Distance metric                      │
│              │                │ Also: cosine, manhattan,             │
│              │                │ correlation, etc.                    │
├──────────────┼────────────────┼──────────────────────────────────────┤
│ n_epochs     │ None (auto)    │ Training iterations                  │
│              │                │ More → better but slower             │
└──────────────┴────────────────┴──────────────────────────────────────┘
```

### Parameter Effects Visualization

```
  n_neighbors = 5              n_neighbors = 50
  ┌──────────────────┐        ┌──────────────────┐
  │ ●●  ○○  ▲▲  ■■   │        │  ●●●○○○▲▲▲■■■    │
  │ ●●  ○○  ▲▲  ■■   │        │  ●●●○○○▲▲▲■■■    │
  │ Fragmented,       │        │  Cohesive,        │
  │ many small groups │        │  fewer groups      │
  └──────────────────┘        └──────────────────┘

  min_dist = 0.0               min_dist = 0.5
  ┌──────────────────┐        ┌──────────────────┐
  │   ●●●    ○○○      │        │  ● ● ●  ○ ○ ○    │
  │   ●●●    ○○○      │        │  ● ● ●  ○ ○ ○    │
  │ Dense, packed      │        │  Even, spread     │
  │ clusters           │        │  distribution     │
  └──────────────────┘        └──────────────────┘
```

---

## 🐍 Python Implementation

### Installation

```bash
pip install umap-learn
```

### Basic Usage

```python
import umap
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.preprocessing import StandardScaler

# Load data
digits = load_digits()
X, y = digits.data, digits.target

# Scale data
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# UMAP embedding
reducer = umap.UMAP(
    n_neighbors=15,      # Local neighborhood size
    min_dist=0.1,        # Minimum embedding distance
    n_components=2,      # Output dimensions
    metric='euclidean',  # Distance metric
    random_state=42      # Reproducibility
)

X_umap = reducer.fit_transform(X_scaled)

print(f"Original shape: {X.shape}")        # (1797, 64)
print(f"UMAP shape:     {X_umap.shape}")   # (1797, 2)

# Visualization
plt.figure(figsize=(12, 8))
scatter = plt.scatter(X_umap[:, 0], X_umap[:, 1],
                      c=y, cmap='tab10', s=15, alpha=0.7)
plt.colorbar(scatter, ticks=range(10), label='Digit')
plt.title('UMAP of Handwritten Digits', fontsize=14)
plt.xlabel('UMAP 1')
plt.ylabel('UMAP 2')
plt.tight_layout()
plt.savefig('umap_digits.png', dpi=150)
plt.show()
```

### Embedding New Points (Transform)

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.2, random_state=42
)

# Fit on training data
reducer = umap.UMAP(n_neighbors=15, min_dist=0.1, random_state=42)
X_train_umap = reducer.fit_transform(X_train)

# Transform new points (t-SNE CANNOT do this!)
X_test_umap = reducer.transform(X_test)

# Visualize both
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

axes[0].scatter(X_train_umap[:, 0], X_train_umap[:, 1],
                c=y_train, cmap='tab10', s=10, alpha=0.5)
axes[0].set_title('Training Data (fit_transform)')

axes[1].scatter(X_test_umap[:, 0], X_test_umap[:, 1],
                c=y_test, cmap='tab10', s=10, alpha=0.5)
axes[1].set_title('Test Data (transform)')

plt.tight_layout()
plt.show()
```

### Parameter Exploration

```python
fig, axes = plt.subplots(3, 3, figsize=(18, 18))

params = [
    (5, 0.0), (15, 0.0), (50, 0.0),
    (5, 0.1), (15, 0.1), (50, 0.1),
    (5, 0.5), (15, 0.5), (50, 0.5),
]

for ax, (nn, md) in zip(axes.flatten(), params):
    reducer = umap.UMAP(n_neighbors=nn, min_dist=md, random_state=42)
    embedding = reducer.fit_transform(X_scaled)
    ax.scatter(embedding[:, 0], embedding[:, 1],
               c=y, cmap='tab10', s=3, alpha=0.5)
    ax.set_title(f'n_neighbors={nn}, min_dist={md}')
    ax.set_xticks([])
    ax.set_yticks([])

plt.suptitle('UMAP: Parameter Grid', fontsize=16)
plt.tight_layout()
plt.show()
```

### Supervised UMAP

```python
# UMAP can incorporate labels for semi-supervised embedding
reducer_supervised = umap.UMAP(
    n_neighbors=15, min_dist=0.1, random_state=42
)

# Pass labels to fit → incorporates label info
X_umap_supervised = reducer_supervised.fit_transform(X_scaled, y=y)

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

axes[0].scatter(X_umap[:, 0], X_umap[:, 1], c=y, cmap='tab10', s=5)
axes[0].set_title('Unsupervised UMAP')

axes[1].scatter(X_umap_supervised[:, 0], X_umap_supervised[:, 1],
                c=y, cmap='tab10', s=5)
axes[1].set_title('Supervised UMAP')

plt.tight_layout()
plt.show()
```

### UMAP for General Dimensionality Reduction (Not Just Visualization)

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

# Use UMAP to reduce to higher dims (e.g., 10)
reducer_10d = umap.UMAP(n_components=10, random_state=42)
X_umap_10d = reducer_10d.fit_transform(X_scaled)

# Compare classification accuracy
rf = RandomForestClassifier(n_estimators=100, random_state=42)

scores_original = cross_val_score(rf, X_scaled, y, cv=5)
scores_umap = cross_val_score(rf, X_umap_10d, y, cv=5)

print(f"Original (64D): {scores_original.mean():.4f} ± {scores_original.std():.4f}")
print(f"UMAP (10D):     {scores_umap.mean():.4f} ± {scores_umap.std():.4f}")
```

---

## ⚖️ UMAP vs t-SNE

```
┌──────────────────────┬─────────────────────┬─────────────────────┐
│     Aspect           │      t-SNE          │      UMAP           │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Speed                │ Slow (O(N²) or      │ Fast (O(N^1.14))    │
│                      │ O(N log N))         │                     │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Scalability          │ ~100K points        │ Millions of points  │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Global structure     │ Poorly preserved    │ Better preserved    │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Local structure      │ Excellent           │ Excellent           │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ New points           │ Cannot embed        │ .transform() works  │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ n_components         │ 2 or 3 only         │ Any dimension       │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Theory               │ Information theory  │ Riemannian geometry │
│                      │ (KL divergence)     │ & topology          │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Objective            │ KL divergence       │ Cross-entropy       │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Optimization         │ Full gradient       │ SGD + negative      │
│                      │ descent             │ sampling            │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Deterministic        │ No                  │ More stable         │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Supervised mode      │ No                  │ Yes                 │
├──────────────────────┼─────────────────────┼─────────────────────┤
│ Key parameter        │ perplexity          │ n_neighbors         │
└──────────────────────┴─────────────────────┴─────────────────────┘
```

---

## 🏭 Applications

| Domain | Application |
|--------|-------------|
| **Single-cell Biology** | Visualizing cell types in scRNA-seq (standard tool in bioinformatics) |
| **NLP** | Document/sentence embedding visualization |
| **Recommender Systems** | Visualizing user/item embedding spaces |
| **Anomaly Detection** | Visual identification of outliers |
| **Feature Engineering** | UMAP features as input to classifiers |
| **Image Retrieval** | Organizing image databases by similarity |

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Full Name** | Uniform Manifold Approximation and Projection |
| **Year** | 2018 (McInnes, Healy, Melville) |
| **Type** | Nonlinear dimensionality reduction |
| **Foundation** | Topological data analysis, Riemannian geometry |
| **High-D Graph** | Fuzzy simplicial set (k-NN with adaptive radii) |
| **Low-D Graph** | Smooth approximation with a, b parameters |
| **Objective** | Cross-entropy between fuzzy graphs |
| **Key Parameters** | `n_neighbors` (15), `min_dist` (0.1) |
| **Complexity** | O(N^1.14) empirically |
| **Preserves** | Local ✅ and Global ✅ structure |
| **New Points** | `.transform()` supported ✅ |
| **Library** | `umap-learn` (`import umap`) |

---

## ❓ Revision Questions

1. **How does UMAP's theoretical foundation differ from t-SNE's? What role does topology play?**
   Discuss manifold assumption, fuzzy simplicial sets, and Riemannian geometry.

2. **Explain how UMAP handles the local distance metric and why each point has its own scale (ρᵢ and σᵢ).**
   Relate to the assumption that the manifold is locally uniformly distributed.

3. **What is the fuzzy union operation used to symmetrize the graph, and why is it chosen over simple averaging?**
   Discuss probabilistic OR vs. arithmetic mean and the connectivity guarantee.

4. **Compare the optimization objectives of UMAP (cross-entropy) and t-SNE (KL divergence). Why does UMAP preserve global structure better?**
   Discuss the repulsive term and negative sampling.

5. **A bioinformatician needs to embed 500,000 single cells. Why would UMAP be preferred over t-SNE? What parameters would you recommend?**
   Consider computational complexity, scalability, and `.transform()` capability.

6. **How does `n_neighbors` affect the balance between local and global structure in UMAP embeddings?**
   Describe the tradeoff and recommend values for different use cases.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← t-SNE](./02-tsne.md) | [📂 Unsupervised Learning](../README.md) | [Autoencoders Preview →](./04-autoencoders-preview.md) |

---

> **Key Takeaway:** UMAP is often the best default choice for nonlinear dimensionality reduction — it's faster than t-SNE, preserves global structure better, can embed new points, and works in arbitrary dimensions. Use `n_neighbors` to balance local vs. global structure and `min_dist` to control cluster tightness.
