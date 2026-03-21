# 9. PCA Limitations and Alternatives

[← Previous: PCA for Visualization](./08-pca-for-visualization.md) | [Back to PCA Unit →](./README.md)

---

## Chapter Overview

PCA is powerful but has fundamental limitations. It is restricted to **linear** transformations, assumes **variance equals importance**, is **sensitive to feature scaling**, and produces components that may be **difficult to interpret**. This chapter examines each limitation in detail, demonstrates failure cases, and introduces alternatives — Kernel PCA, t-SNE, and UMAP — that address specific shortcomings.

---

## 9.1 Limitation 1: PCA is Linear Only

### The Problem

```
PCA can only find LINEAR directions. If the data lies on a
CURVED manifold, PCA will fail to capture the true structure.

  Linear structure (PCA works):      Non-linear structure (PCA FAILS):

  ·                                        · · ·
   ·                                     · · · · ·
    ·                                  · · · · · · ·
     ·           PC1 finds this →    ·               ·
      ·          line easily         ·       ○       ·
       ·                              ·             ·
        ·                               · · · · · ·
                                          · · · ·

  PCA projects onto a line:          PCA on the ring:
  ──·──·──·──·──·──·──→              ──·──·──·──·──·──·──→
  Perfect representation!            Both inner and outer ring
                                     project to same line!
                                     Structure completely lost!
```

### Example: Swiss Roll

```
The Swiss Roll is a classic example where PCA fails:

  Side view (3D):                  PCA projection (2D):
  
   ╭───╮                            ····················
  ╱ ╭───╮╲                          ····················
 │ ╱     ╲│                          ····················
 ││       ││     PCA →               ····················
 │╲       ╱│                         ····················
  ╲ ╰───╯╱                           ····················
   ╰───╯
                                    Everything overlaps!
  3D spiral structure               The spiral "unrolling" is lost
  lives on a 2D manifold            because PCA can only project
                                    linearly.
```

```python
from sklearn.datasets import make_swiss_roll
from sklearn.decomposition import PCA

# Generate Swiss roll
X, color = make_swiss_roll(n_samples=1000, noise=0.5, random_state=42)

# PCA fails on Swiss roll
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

print("Swiss Roll → PCA (2D):")
print(f"  Variance captured: {sum(pca.explained_variance_ratio_)*100:.1f}%")
print(f"  But the 2D projection OVERLAPS — structure is lost!")
print(f"  → Non-linear method needed (Kernel PCA, t-SNE, UMAP)")
```

---

## 9.2 Limitation 2: Variance ≠ Importance

### The Problem

```
PCA assumes: "The direction with most VARIANCE contains the most
INFORMATION." This is NOT always true!

Example: Noisy signal

  Feature 1 (signal):    [1, 2, 3, 4, 5]     Variance = 2.5
  Feature 2 (noise):     [50, -30, 80, -60, 40]  Variance = 2740.0

  PCA says: "Feature 2 has more variance → more important → PC1"
  Reality:  Feature 2 is PURE NOISE!
  
  PCA direction:
  ────────────────────────────→ Feature 2 (noise!)
  
  The important signal (Feature 1) is IGNORED because
  it has lower variance than the noise.
```

### When This Matters

```
  ✗ High-variance noise features dominate PCA
  ✗ Class-discriminative features may have LOW variance
  ✗ A feature that perfectly separates classes might have
    tiny variance compared to irrelevant features

  Example: Medical diagnosis
    Feature 1: age (range 0-100, variance=500)     ← not diagnostic
    Feature 2: biomarker (range 0-1, variance=0.01) ← CRITICAL for diagnosis
    
    PCA: "PC1 = age direction" (wrong for diagnosis!)
    Solution: Standardize first, or use supervised methods (LDA)
```

---

## 9.3 Limitation 3: Scaling Sensitivity

### The Problem

```
PCA results CHANGE DRAMATICALLY based on feature scaling:

  Unscaled:                          Scaled (standardized):
  Feature 1: height (cm) 150-200     Feature 1: z-scores -2 to +2
  Feature 2: weight (kg) 50-100      Feature 2: z-scores -2 to +2
  
  Var(height) = 250                   Var(height_scaled) = 1
  Var(weight) = 200                   Var(weight_scaled) = 1
  
  Unscaled PCA: PC1 ≈ height         Scaled PCA: PC1 considers both
  (height dominates due to units!)    features equally → more meaningful

  Same data, different results just because of UNITS!
```

### The Fix

```python
from sklearn.preprocessing import StandardScaler

# ALWAYS standardize before PCA (unless you have a reason not to)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
pca = PCA(n_components=k).fit_transform(X_scaled)

# Exceptions where you might NOT standardize:
# • All features have same units (e.g., image pixels 0-255)
# • Variance differences are meaningful
# • Working with a correlation matrix directly
```

---

## 9.4 Limitation 4: Interpretability Loss

### The Problem

```
PCA transforms features into LINEAR COMBINATIONS that are
often difficult to interpret:

  Original features (interpretable):
    x₁ = income ($)
    x₂ = age (years)
    x₃ = education (years)
    x₄ = experience (years)

  PCA components (hard to interpret):
    PC1 = 0.48·income + 0.23·age + 0.52·education + 0.45·experience
    PC2 = -0.31·income + 0.67·age - 0.15·education + 0.52·experience

  What IS PC1? "Some combination of income, education, and experience"
  Not very helpful for a business stakeholder!

  Contrast with feature selection:
    Selected: [income, education]  ← immediately interpretable
```

### Partial Solutions

```
1. Loading analysis: Examine which features contribute most to each PC
   → PC1 ≈ "overall career success" (income, education, experience)
   → PC2 ≈ "age vs. income" (age high, income low)

2. Sparse PCA: Forces some loadings to zero
   → PC1 = 0.6·income + 0·age + 0.5·education + 0·experience
   → Easier to interpret (only 2 non-zero features)

3. Factor rotation: Rotates PCs for simpler structure (Varimax)
   → Common in psychology and social sciences
```

---

## 9.5 Limitation 5: Assumes Linear Correlations

```
PCA captures LINEAR relationships (covariance/correlation).
It misses NON-LINEAR dependencies:

  Example: y = x²

  x:  [-3, -2, -1, 0, 1, 2, 3]
  y:  [ 9,  4,  1, 0, 1, 4, 9]

  Correlation(x, y) = 0    (no LINEAR relationship)
  But y is PERFECTLY determined by x!
  PCA: "x and y are independent" → WRONG
  PCA cannot capture this parabolic relationship.
```

---

## 9.6 Alternative: Kernel PCA

### Concept

```
Kernel PCA applies PCA in a HIGH-DIMENSIONAL feature space
via the kernel trick, capturing NON-LINEAR relationships.

  Standard PCA:
    X → covariance matrix → eigenvectors → project

  Kernel PCA:
    X → φ(X) → covariance in feature space → eigenvectors
    
  But φ(X) may be infinite-dimensional!
  → Use kernel trick: K(xᵢ,xⱼ) = φ(xᵢ)·φ(xⱼ)
  → Only need the kernel matrix K (N×N), not φ(X)
```

### Common Kernels

```
  Linear:      K(x,y) = xᵀy           (= standard PCA)
  Polynomial:  K(x,y) = (xᵀy + c)^d   (captures polynomial relations)
  RBF:         K(x,y) = exp(-γ||x-y||²) (most popular, captures any shape)
  Sigmoid:     K(x,y) = tanh(αxᵀy + c)  (neural network-like)
```

### Python

```python
from sklearn.decomposition import KernelPCA
from sklearn.datasets import make_circles

# Data that PCA can't separate
X, y = make_circles(n_samples=400, factor=0.3, noise=0.05, random_state=42)

# Standard PCA (fails)
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)
print("Standard PCA on circles:")
print(f"  The two circles overlap in PCA projection")

# Kernel PCA (succeeds!)
kpca = KernelPCA(n_components=2, kernel='rbf', gamma=15)
X_kpca = kpca.fit_transform(X)
print(f"\nKernel PCA (RBF, γ=15) on circles:")
print(f"  Inner and outer circles separated!")

# Compare separation
for method, X_proj, name in [(pca, X_pca, "PCA"), (kpca, X_kpca, "KPCA")]:
    c0 = X_proj[y == 0].mean(axis=0)
    c1 = X_proj[y == 1].mean(axis=0)
    sep = np.linalg.norm(c0 - c1)
    print(f"  {name}: class separation = {sep:.3f}")
```

---

## 9.7 Alternative: t-SNE

### Concept

```
t-SNE (t-distributed Stochastic Neighbor Embedding):
  Preserves LOCAL structure (nearby points stay nearby)
  
  Key idea:
    1. Compute pairwise similarities in HIGH-D (Gaussian kernel)
    2. Find LOW-D embedding that preserves these similarities
    3. Uses t-distribution in low-D (heavy tails → better separation)
  
  Properties:
    ✅ Excellent for visualization (2D/3D)
    ✅ Reveals cluster structure PCA misses
    ✅ Non-linear
    ❌ Non-parametric (can't transform new points)
    ❌ Non-deterministic (stochastic)
    ❌ Distances/sizes in t-SNE plot are NOT meaningful
    ❌ Slow for large datasets: O(N²)
    ❌ Only for visualization (not general dim reduction)
```

```python
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits
from sklearn.preprocessing import StandardScaler

digits = load_digits()
X = StandardScaler().fit_transform(digits.data)

# t-SNE (slow for large datasets)
tsne = TSNE(
    n_components=2,
    perplexity=30,       # Number of effective nearest neighbors
    learning_rate=200,   # Step size
    n_iter=1000,         # Iterations
    random_state=42
)
X_tsne = tsne.fit_transform(X)

print("t-SNE on digits (64D → 2D):")
print(f"  Creates beautiful cluster visualization")
print(f"  But distances between clusters are NOT meaningful!")
```

---

## 9.8 Alternative: UMAP

### Concept

```
UMAP (Uniform Manifold Approximation and Projection):
  Preserves BOTH local AND global structure (better than t-SNE)

  Properties:
    ✅ Excellent for visualization
    ✅ Preserves global structure better than t-SNE
    ✅ Much faster than t-SNE: O(N^1.14) vs O(N²)
    ✅ Can transform new points (has a transform method!)
    ✅ Works for general dimensionality reduction
    ❌ Non-linear
    ❌ Hyperparameters affect results significantly
    ❌ Less mathematically understood than PCA
```

```python
# pip install umap-learn
# import umap
# 
# reducer = umap.UMAP(
#     n_components=2,
#     n_neighbors=15,      # Local vs global structure
#     min_dist=0.1,        # How tightly points cluster
#     metric='euclidean',
#     random_state=42
# )
# X_umap = reducer.fit_transform(X)
# 
# # UMAP can transform new data!
# X_new_umap = reducer.transform(X_new)
```

---

## 9.9 Comparison: PCA vs. Alternatives

| Feature | PCA | Kernel PCA | t-SNE | UMAP |
|---------|-----|-----------|-------|------|
| **Type** | Linear | Non-linear | Non-linear | Non-linear |
| **Preserves** | Global variance | Non-linear variance | Local structure | Local + global |
| **Speed** | O(Nd²) | O(N²d) | O(N²) | O(N^1.14) |
| **Scalability** | Excellent | Moderate | Poor (>10K) | Good |
| **Transform new data** | ✅ | ✅ | ❌ | ✅ |
| **Interpretable** | Loading vectors | ❌ | ❌ | ❌ |
| **Deterministic** | ✅ | ✅ | ❌ | ❌ |
| **Distances meaningful** | ✅ | ✅ | ❌ | Partially |
| **Best for** | General reduction | Non-linear separation | Visualization | Visualization + reduction |
| **Parameters** | k | k, kernel, γ | perplexity, lr | n_neighbors, min_dist |

### When to Use What

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Use PCA when:                                                   │
│  • First exploratory step (always try PCA first!)               │
│  • Need interpretable components (loadings)                     │
│  • Preprocessing for downstream ML (noise reduction)            │
│  • Data has linear structure                                     │
│  • Need to transform new data quickly                           │
│  • Need deterministic, reproducible results                     │
│                                                                  │
│  Use Kernel PCA when:                                           │
│  • Non-linear structure + need transform for new data           │
│  • Know the appropriate kernel for your data                    │
│                                                                  │
│  Use t-SNE when:                                                │
│  • Visualization only (2D/3D plots for papers/presentations)    │
│  • Dataset size < 10,000                                        │
│  • Local cluster structure is most important                    │
│                                                                  │
│  Use UMAP when:                                                 │
│  • Visualization (generally better than t-SNE)                  │
│  • Larger datasets (faster than t-SNE)                          │
│  • Need to embed new points                                     │
│  • Want both local AND global structure preserved               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 9.10 Python: Full Comparison Pipeline

```python
import numpy as np
from sklearn.decomposition import PCA, KernelPCA
from sklearn.manifold import TSNE
from sklearn.datasets import make_moons
from sklearn.preprocessing import StandardScaler
import time

# Non-linear data
X, y = make_moons(n_samples=500, noise=0.1, random_state=42)
X = StandardScaler().fit_transform(X)

methods = {}

# PCA
t0 = time.time()
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)
methods['PCA'] = {'time': time.time()-t0, 'X': X_pca}

# Kernel PCA
t0 = time.time()
kpca = KernelPCA(n_components=2, kernel='rbf', gamma=5)
X_kpca = kpca.fit_transform(X)
methods['Kernel PCA'] = {'time': time.time()-t0, 'X': X_kpca}

# t-SNE
t0 = time.time()
tsne = TSNE(n_components=2, perplexity=30, random_state=42)
X_tsne = tsne.fit_transform(X)
methods['t-SNE'] = {'time': time.time()-t0, 'X': X_tsne}

print("=== Method Comparison on Moon Data ===\n")
print(f"{'Method':>15} | {'Time (ms)':>10} | {'Class Sep':>10} | Notes")
print(f"{'-'*15}-+-{'-'*10}-+-{'-'*10}-+-------")

for name, data in methods.items():
    X_proj = data['X']
    c0 = X_proj[y == 0].mean(axis=0)
    c1 = X_proj[y == 1].mean(axis=0)
    sep = np.linalg.norm(c0 - c1)
    print(f"{name:>15} | {data['time']*1000:>9.1f} | {sep:>10.3f} | "
          f"{'linear' if name == 'PCA' else 'non-linear'}")
```

---

## 9.11 Summary Table

| Limitation | Description | Solution |
|-----------|-------------|----------|
| **Linear only** | Can't capture curves, manifolds | Kernel PCA, t-SNE, UMAP |
| **Variance ≠ importance** | Noise with high variance dominates | Standardize; supervised methods (LDA) |
| **Scale sensitive** | Results depend on feature units | StandardScaler before PCA |
| **Interpretability** | PCs are abstract combinations | Loading analysis, sparse PCA |
| **Linear correlations only** | Misses non-linear dependencies | Kernel PCA |
| **Global structure only** | Misses local cluster structure | t-SNE, UMAP |

---

## 9.12 Revision Questions

1. **Why does PCA fail on the Swiss Roll dataset?** Explain what happens geometrically when you project a spiral onto a plane.

2. **Give an example where high variance does not mean high importance.** How would standardization help? When would even standardization not be enough?

3. **Compare PCA, Kernel PCA, t-SNE, and UMAP** along three dimensions: linearity, scalability, and ability to transform new data.

4. **What is the kernel trick** in Kernel PCA? Why is it computationally necessary? What happens if we tried to apply standard PCA in the high-dimensional feature space directly?

5. **Why are distances in a t-SNE plot not meaningful?** A colleague says "Cluster A is farther from Cluster B than from Cluster C in the t-SNE plot, so A is more different from B." Is this reasoning valid?

6. **You have a dataset with 1000 features** and suspect non-linear structure. Design a practical analysis plan that starts with PCA and progressively adds complexity. When would you switch to non-linear methods?

---

[← Previous: PCA for Visualization](./08-pca-for-visualization.md) | [Back to PCA Unit →](./README.md)
