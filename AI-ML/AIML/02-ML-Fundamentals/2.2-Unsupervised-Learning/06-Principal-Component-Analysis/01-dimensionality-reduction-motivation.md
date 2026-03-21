# 1. Dimensionality Reduction: Motivation

[← Back to PCA Unit](./README.md) | [Next: PCA Intuition →](./02-pca-intuition.md)

---

## Chapter Overview

Real-world datasets often have dozens, hundreds, or even thousands of features. **Dimensionality reduction** transforms high-dimensional data into a lower-dimensional representation while preserving the most important information. This chapter explores why dimensionality reduction is necessary — from the curse of dimensionality and computational efficiency to visualization, noise reduction, and multicollinearity — and distinguishes between **feature extraction** and **feature selection**.

---

## 1.1 The Curse of Dimensionality

### What Is It?

As the number of dimensions (features) increases, several counterintuitive and problematic phenomena emerge:

```
The Curse of Dimensionality:

  Dimension d   │  Volume of unit   │  % of data in    │  Avg distance
  (features)    │  hypercube needed │  outer 10% shell  │  between points
  ──────────────┼───────────────────┼───────────────────┼───────────────
       1        │     1.0           │     20%           │  moderate
       2        │     1.0           │     36%           │  moderate
       5        │     1.0           │     67%           │  increasing
      10        │     1.0           │     88%           │  large
      50        │     1.0           │     99.5%         │  very large
     100        │     1.0           │     99.997%       │  nearly equal
     500        │     1.0           │     ≈100%         │  all the same!
```

### Key Phenomena

**1. Distance Concentration**

```
In high dimensions, all pairwise distances become nearly equal:

  d=2:   max_dist / min_dist ≈ 5-10    (distances vary a lot)
  d=100: max_dist / min_dist ≈ 1.01    (all points equidistant!)

  Consequence: Distance-based algorithms (K-Means, KNN, DBSCAN)
  become meaningless because "nearest" and "farthest" are almost the same.

  ┌──────────────────────────────────────────────────┐
  │  Low-d: Distances are meaningful                 │
  │         · · · · · · · · · · · ·                  │
  │  min────────────────────────max                  │
  │         ↕ large range ↕                          │
  │                                                  │
  │  High-d: Distances concentrate                   │
  │                  · · · · · · ·                   │
  │              min──────────max                    │
  │              ↕ tiny range ↕                      │
  └──────────────────────────────────────────────────┘
```

**2. Data Sparsity**

```
To maintain the same density of points:

  d=1:   10 points is "dense"        → need 10¹ = 10 points
  d=2:   need 10² = 100 points       for same density
  d=3:   need 10³ = 1,000 points
  d=10:  need 10¹⁰ = 10 billion points!
  d=100: need 10¹⁰⁰ points ← more than atoms in the universe

  With a fixed dataset size, increasing dimensions
  makes the data exponentially sparser.
```

**3. Overfitting Risk**

```
More features = more parameters to estimate
  → Model has more capacity to memorize training data
  → Generalization to new data degrades

  Error
   │
   │ ╲       training error
   │  ╲────────────────────────
   │   ╲
   │    ╲         ╱ test error
   │     ╲───────╱
   │      ╲     ╱
   │       ╲   ╱
   │        ╲ ╱ ← sweet spot
   │         ╳
   └──────────┬──────────────── # Features
         optimal d
```

---

## 1.2 Computational Efficiency

### Why Fewer Dimensions = Faster Everything

```
Algorithm complexities that depend on dimension d:

  ┌────────────────────┬────────────────────┬──────────────┐
  │ Algorithm          │ Complexity         │ d impact     │
  ├────────────────────┼────────────────────┼──────────────┤
  │ K-Means            │ O(NKtd)            │ Linear in d  │
  │ KNN                │ O(NMd)             │ Linear in d  │
  │ SVM (linear)       │ O(N²d)             │ Linear in d  │
  │ Neural Network     │ O(N·d·h)           │ d = input    │
  │ PCA (compute cov)  │ O(Nd²)             │ Quadratic!   │
  │ Distance matrix    │ O(N²d)             │ Linear in d  │
  │ Gaussian density   │ O(d³) per eval     │ Cubic!       │
  └────────────────────┴────────────────────┴──────────────┘

  Reducing d=1000 to d=50:
  • K-Means: 20× faster
  • KNN: 20× faster
  • Covariance: 400× faster
  • Storage: 20× less memory
```

### Practical Impact

```
Example: Image classification
  Original: 256×256 pixels = 65,536 features
  After PCA (95% variance): ~200 features
  
  Speedup: 300× for distance computations
  Storage: 300× less memory
  Often: BETTER accuracy (noise removed!)
```

---

## 1.3 Visualization

### The Human Vision Limit

```
Humans can visualize:
  • 1D: Histograms, number lines
  • 2D: Scatter plots, heatmaps
  • 3D: 3D scatter plots (with rotation)
  • 4D+: ???  (color, size, shape encoding gets confusing)

Dimensionality reduction for visualization:
  d = 100 features → 2 or 3 principal components
  → Plot on a scatter plot
  → See cluster structure, outliers, patterns!
```

```
High-dimensional data:                    After PCA → 2D:
  [x₁, x₂, ..., x₁₀₀]                   
  Cannot visualize!                        PC2
                                            │    · ·
  ┌─────────────────────┐                   │   · · ·    · ·
  │ ? ? ? ? ? ? ? ? ?   │                   │  · · · ·  · · ·
  │ ? ? ? ? ? ? ? ? ?   │     ───PCA──→     │   · · ·    · · ·
  │ ? ? ? ? ? ? ? ? ?   │                   │    · ·      · ·
  │ ? ? ? ? ? ? ? ? ?   │                   │
  └─────────────────────┘                   └──────────────────── PC1
                                            
  "What does this data look like?"         "Aha! Two clusters!"
```

---

## 1.4 Noise Reduction

### Dimensionality Reduction as Denoising

```
Key insight: In many datasets, the important structure lives in a
LOW-DIMENSIONAL SUBSPACE, while noise spreads across ALL dimensions.

  Signal:  Lives in the first few principal components
  Noise:   Distributed uniformly across ALL components

  Variance per component:
  │ ████                        Signal
  │ ████                        dominates
  │ ████                        first few
  │ ██                          components
  │ █
  │ ▪ ▪ ▪ ▪ ▪ ▪ ▪ ▪ ▪ ▪ ▪    ← Noise floor
  └──┬──┬──┬──┬──┬──┬──┬──── Component
     1  2  3  4  5  ...  d

  By keeping only the top k components (above the noise floor),
  we effectively REMOVE the noise in the discarded components.
```

### Example: Signal Recovery

```
Original clean signal:  [3, 5, 7, 5, 3]  (lives in 2D subspace)
Noise added:           [0.1, -0.2, 0.3, -0.1, 0.2]  (random in all 5D)
Observed:              [3.1, 4.8, 7.3, 4.9, 3.2]

PCA with k=2:
  → Projects onto 2D subspace
  → Reconstructs: [3.02, 4.98, 7.01, 5.01, 2.98]
  → Noise largely removed!
```

---

## 1.5 Multicollinearity

### The Problem

```
Multicollinearity: features that are highly correlated with each other.

  Example: Predicting house price
    Feature 1: area_sqft
    Feature 2: area_sqm = area_sqft × 0.093    (perfectly correlated!)
    Feature 3: num_rooms                        (correlated with area)
    Feature 4: num_bathrooms                    (correlated with rooms)

  Problems:
    1. Redundant information → wasted computation
    2. Unstable regression coefficients
    3. Inflated standard errors
    4. Difficult to interpret feature importance
```

### How PCA Fixes It

```
PCA transforms correlated features into UNCORRELATED principal components:

  Before PCA:                   After PCA:
  ┌──────────────────────┐      ┌──────────────────────┐
  │  Correlation Matrix  │      │  Correlation Matrix  │
  │  1.0  0.95 0.8  0.7  │      │  1.0  0.0  0.0  0.0  │
  │  0.95 1.0  0.85 0.75 │      │  0.0  1.0  0.0  0.0  │
  │  0.8  0.85 1.0  0.9  │      │  0.0  0.0  1.0  0.0  │
  │  0.7  0.75 0.9  1.0  │      │  0.0  0.0  0.0  1.0  │
  └──────────────────────┘      └──────────────────────┘
  
  Highly correlated              Perfectly uncorrelated!
  (redundant info)               (independent components)
```

---

## 1.6 Feature Extraction vs. Feature Selection

### Feature Selection

```
Feature Selection: CHOOSE a subset of original features, discard the rest.

  Original: [height, weight, age, income, zip_code, shoe_size]
  Selected: [height, weight, income]                              
  
  Methods:
    • Filter: correlation, mutual information, χ² test
    • Wrapper: forward/backward selection, recursive elimination
    • Embedded: L1 regularization (Lasso), tree importance
  
  ✅ Keeps original feature meaning (interpretable)
  ✅ No transformation needed for new data
  ❌ Can only use existing features
  ❌ May miss combinations of features
```

### Feature Extraction

```
Feature Extraction: CREATE new features as transformations of originals.

  Original: [height, weight, age, income, zip_code, shoe_size]
  Extracted: [PC1, PC2, PC3]  where:
    PC1 = 0.5·height + 0.3·weight + 0.2·age + ...
    PC2 = -0.1·height + 0.4·weight - 0.3·age + ...
    PC3 = 0.2·height - 0.1·weight + 0.5·age + ...
  
  Methods:
    • PCA (linear)
    • Kernel PCA (nonlinear)
    • t-SNE, UMAP (nonlinear, visualization)
    • Autoencoders (neural network)
  
  ✅ Can capture combinations and interactions
  ✅ Often better dimension reduction ratio
  ✅ Noise reduction
  ❌ New features may not be interpretable
  ❌ Requires transformation pipeline for new data
```

### Comparison

```
                Feature Selection          Feature Extraction
                ─────────────────          ──────────────────
  Output:       Subset of originals        New derived features
  Interpretable: ✅ Yes                     ⚠️ Sometimes
  Reversible:   ❌ Lost features gone       ✅ Can reconstruct
  Info loss:    Can lose interactions       Minimal (keeps variance)
  New data:     Just drop columns          Apply transform
  PCA type:     ❌                          ✅
```

---

## 1.7 When to Reduce Dimensions

```
┌──────────────────────────────────────────────────────────────────┐
│  Decision: Do I need dimensionality reduction?                   │
│                                                                  │
│  ✅ YES if:                                                     │
│    • d > 50 and downstream model is distance-based              │
│    • Visualization needed (reduce to 2-3 components)            │
│    • Significant multicollinearity (corr > 0.9)                 │
│    • Training is too slow                                       │
│    • Overfitting suspected (d >> N)                             │
│    • Storage/memory constraints                                  │
│                                                                  │
│  ❌ NO if:                                                      │
│    • d is already small (< 10-20)                               │
│    • Feature interpretability is critical                        │
│    • Using tree-based models (handle high-d well)               │
│    • Each feature has known domain importance                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1.8 Python: Demonstrating the Curse of Dimensionality

```python
import numpy as np
from sklearn.neighbors import NearestNeighbors

np.random.seed(42)
N = 1000

print("=== Distance Concentration in High Dimensions ===\n")
print(f"{'Dimension':>10} | {'Min Dist':>10} | {'Max Dist':>10} | {'Max/Min':>10} | {'Mean Dist':>10}")
print(f"{'-'*10}-+-{'-'*10}-+-{'-'*10}-+-{'-'*10}-+-{'-'*10}")

for d in [2, 5, 10, 50, 100, 500]:
    X = np.random.uniform(0, 1, size=(N, d))
    
    # Compute pairwise distances for a sample
    from sklearn.metrics import pairwise_distances
    dists = pairwise_distances(X[:100])
    dists_nonzero = dists[dists > 0]
    
    min_d = dists_nonzero.min()
    max_d = dists_nonzero.max()
    mean_d = dists_nonzero.mean()
    ratio = max_d / min_d
    
    print(f"{d:>10} | {min_d:>10.4f} | {max_d:>10.4f} | {ratio:>10.4f} | {mean_d:>10.4f}")

print("\n→ As d increases, max/min ratio → 1 (all distances become similar)")
print("→ This makes distance-based algorithms ineffective!")
```

---

## 1.9 Summary Table

| Motivation | Problem | How Dim Reduction Helps |
|-----------|---------|------------------------|
| **Curse of dimensionality** | Distances concentrate, data becomes sparse | Reduce to meaningful dimensions |
| **Computational efficiency** | Algorithms scale with d | Fewer features = faster |
| **Visualization** | Can't plot > 3D | Project to 2-3D |
| **Noise reduction** | Noise in many dimensions | Keep signal, discard noise |
| **Multicollinearity** | Correlated features | PCA gives uncorrelated components |
| **Overfitting** | Too many features vs. samples | Reduce model complexity |
| **Storage** | High-d data uses lots of memory | Compressed representation |

---

## 1.10 Revision Questions

1. **Explain the curse of dimensionality** using the concept of distance concentration. Why do distances become meaningless in high dimensions?

2. **A dataset has 100 features but only 50 samples.** What problems might arise when building a machine learning model? How would dimensionality reduction help?

3. **Compare feature extraction and feature selection.** Give one advantage and one disadvantage of each approach. When would you prefer one over the other?

4. **Why does PCA act as a noise reduction technique?** Explain using the concepts of signal subspace and noise floor.

5. **You have a dataset with features [temperature_C, temperature_F, humidity, pressure].** Why is this problematic? How would PCA handle this differently from feature selection?

6. **List three downstream ML algorithms** that benefit significantly from dimensionality reduction and explain why for each.

---

[← Back to PCA Unit](./README.md) | [Next: PCA Intuition →](./02-pca-intuition.md)
