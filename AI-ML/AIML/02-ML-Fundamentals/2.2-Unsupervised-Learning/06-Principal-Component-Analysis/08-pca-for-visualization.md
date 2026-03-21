# 8. PCA for Visualization

[← Previous: Explained Variance Ratio](./07-explained-variance-ratio.md) | [Next: Limitations →](./09-limitations.md)

---

## Chapter Overview

One of the most common applications of PCA is **visualizing high-dimensional data** in 2D or 3D. By projecting onto the first 2-3 principal components, we can create scatter plots that reveal cluster structure, outliers, and relationships invisible in the original high-dimensional space. This chapter covers 2D/3D PCA plots, interpreting PC axes through loading vectors, biplots, and complete sklearn visualization pipelines.

---

## 8.1 2D PCA Plots

### The Basic Idea

```
High-dimensional data → PCA → Project to 2D → Scatter plot

  Original (d dimensions):           PCA projection (2D):
  
  [x₁, x₂, x₃, ..., x₆₄]           PC2
  Cannot visualize!                    │    ▲ ▲
                                       │   ▲ ▲ ▲   ● ●
  ┌─────────────────────┐              │  ▲ ▲ ▲   ● ● ●
  │ ? ? ? ? ? ? ? ? ?   │              │   ▲ ▲     ● ● ●
  │ ? ? ? ? ? ? ? ? ?   │  ─── PCA ──→ │            ● ●
  │ ? ? ? ? ? ? ? ? ?   │              │  ■ ■ ■
  │ ? ? ? ? ? ? ? ? ?   │              │ ■ ■ ■ ■
  └─────────────────────┘              │  ■ ■ ■
                                       └──────────────── PC1
  "What does this data               "Three clusters!"
   look like?"
```

### Python: Complete 2D Visualization Pipeline

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris

# Step 1: Load and prepare data
iris = load_iris()
X = iris.data
y = iris.target
feature_names = iris.feature_names
target_names = iris.target_names

# Step 2: Standardize
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Step 3: Apply PCA (2 components for 2D plot)
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

# Step 4: Print results
print("=== 2D PCA Projection ===\n")
print(f"Original dimensions: {X.shape[1]}")
print(f"Reduced dimensions:  2")
print(f"Variance captured:   {sum(pca.explained_variance_ratio_)*100:.1f}%")
print(f"  PC1: {pca.explained_variance_ratio_[0]*100:.1f}%")
print(f"  PC2: {pca.explained_variance_ratio_[1]*100:.1f}%")

# Step 5: Display ASCII scatter (approximate)
print(f"\n=== ASCII Scatter Plot ===")
print(f"(PC1 on x-axis, PC2 on y-axis)\n")

# Normalize to grid
grid_w, grid_h = 60, 25
x_min, x_max = X_pca[:, 0].min(), X_pca[:, 0].max()
y_min, y_max = X_pca[:, 1].min(), X_pca[:, 1].max()

# Create grid
grid = [[' ' for _ in range(grid_w)] for _ in range(grid_h)]
symbols = ['o', '+', 'x']

for i, (px, py) in enumerate(X_pca):
    gx = int((px - x_min) / (x_max - x_min + 1e-10) * (grid_w - 1))
    gy = grid_h - 1 - int((py - y_min) / (y_max - y_min + 1e-10) * (grid_h - 1))
    gx = max(0, min(grid_w - 1, gx))
    gy = max(0, min(grid_h - 1, gy))
    grid[gy][gx] = symbols[y[i]]

for row in grid:
    print('  │' + ''.join(row) + '│')

print(f"  └{'─' * grid_w}┘")
print(f"  Legend: o={target_names[0]}, +={target_names[1]}, x={target_names[2]}")

# Step 6: Per-class statistics
print(f"\n=== Class Centroids in PC Space ===")
for cls in range(3):
    mask = y == cls
    centroid = X_pca[mask].mean(axis=0)
    spread = X_pca[mask].std(axis=0)
    print(f"  {target_names[cls]:>12}: center=({centroid[0]:.2f}, {centroid[1]:.2f}), "
          f"spread=({spread[0]:.2f}, {spread[1]:.2f})")

# For matplotlib:
# import matplotlib.pyplot as plt
# fig, ax = plt.subplots(figsize=(10, 7))
# for cls, name, color in zip([0,1,2], target_names, ['red','blue','green']):
#     mask = y == cls
#     ax.scatter(X_pca[mask, 0], X_pca[mask, 1], c=color, label=name, alpha=0.7, s=50)
# ax.set_xlabel(f'PC1 ({pca.explained_variance_ratio_[0]*100:.1f}% variance)')
# ax.set_ylabel(f'PC2 ({pca.explained_variance_ratio_[1]*100:.1f}% variance)')
# ax.set_title('PCA: Iris Dataset')
# ax.legend()
# plt.show()
```

---

## 8.2 3D PCA Plots

```python
# 3D projection
pca_3d = PCA(n_components=3)
X_3d = pca_3d.fit_transform(X_scaled)

print("=== 3D PCA Projection ===\n")
print(f"Variance: PC1={pca_3d.explained_variance_ratio_[0]*100:.1f}%, "
      f"PC2={pca_3d.explained_variance_ratio_[1]*100:.1f}%, "
      f"PC3={pca_3d.explained_variance_ratio_[2]*100:.1f}%")
print(f"Total: {sum(pca_3d.explained_variance_ratio_)*100:.1f}%")

# For matplotlib 3D:
# from mpl_toolkits.mplot3d import Axes3D
# fig = plt.figure(figsize=(10, 7))
# ax = fig.add_subplot(111, projection='3d')
# for cls, name, color in zip([0,1,2], target_names, ['red','blue','green']):
#     mask = y == cls
#     ax.scatter(X_3d[mask, 0], X_3d[mask, 1], X_3d[mask, 2], 
#                c=color, label=name, s=50, alpha=0.7)
# ax.set_xlabel(f'PC1 ({pca_3d.explained_variance_ratio_[0]*100:.1f}%)')
# ax.set_ylabel(f'PC2 ({pca_3d.explained_variance_ratio_[1]*100:.1f}%)')
# ax.set_zlabel(f'PC3 ({pca_3d.explained_variance_ratio_[2]*100:.1f}%)')
# ax.legend()
# plt.show()
```

---

## 8.3 Interpreting PC Axes — Loading Vectors

### What Are Loadings?

```
Each PC is a LINEAR COMBINATION of original features:

  PC1 = w₁₁·x₁ + w₁₂·x₂ + w₁₃·x₃ + ... + w₁d·xd

The coefficients w₁ⱼ are called "loadings."
They tell us WHICH ORIGINAL FEATURES contribute most to each PC.

High |loading| → feature strongly influences this PC
Low |loading|  → feature has little influence on this PC
Sign of loading → direction of contribution
```

### Python: Loading Analysis

```python
print("=== Loading Vectors ===\n")
print(f"{'Feature':>20} | {'PC1 Loading':>11} | {'PC2 Loading':>11} | Dominant PC")
print(f"{'-'*20}-+-{'-'*11}-+-{'-'*11}-+-{'-'*10}")

for i, fname in enumerate(feature_names):
    l1 = pca.components_[0][i]
    l2 = pca.components_[1][i]
    dominant = "PC1" if abs(l1) > abs(l2) else "PC2"
    bar1 = '█' * int(abs(l1) * 15)
    print(f"{fname:>20} | {l1:>+10.4f} | {l2:>+10.4f} | {dominant}")

print(f"\nInterpretation:")
print(f"  PC1 ≈ overall size (all loadings have same sign)")
print(f"  PC2 ≈ shape contrast (sepal vs petal width)")
```

### Loading Magnitude Visualization

```
Loading magnitudes for Iris PC1 and PC2:

              PC1 Loading              PC2 Loading
              (72.96% var)             (22.85% var)
  
sepal_length  ████████████  +0.52      █████  +0.38
sepal_width   ██████████── -0.27       ██████████████ +0.92
petal_length  ████████████████ +0.58   ██── -0.02
petal_width   ████████████████ +0.56   ███── -0.07

PC1: High positive loadings on petal length/width
     → "Overall plant size" (all features increase together)

PC2: Dominated by sepal_width (+0.92)
     → "Sepal width vs. rest" factor
```

---

## 8.4 Biplots

A **biplot** superimposes both the projected data points AND the loading vectors on the same plot.

```
Biplot = data scatter + loading arrows

  PC2
   │
   │  ▲ ▲ ▲              sepal_width ──→
   │ ▲ ▲ ▲ ▲           ╱
   │  ▲ ▲ ▲           ╱  (arrow shows how
   │                  ╱    this feature maps
   │  ● ● ● ●       ╱     to PC space)
   │ ● ● ● ● ●     ╱
───┼────────────────╱──────────────── PC1
   │  x x x x     ╲
   │ x x x x x     ╲
   │  x x x x       petal_length ──→
   │                  (petal_width similar direction)
   │

Reading a biplot:
  • Points show the data (color by class)
  • Arrows show how original features map to PC space
  • Arrow LENGTH ∝ feature importance
  • Arrow DIRECTION shows the feature's PC space direction
  • Arrows pointing SAME way → features are CORRELATED
  • Arrows pointing OPPOSITE → features are ANTI-CORRELATED
  • Arrows at 90° → features are UNCORRELATED
```

### Python: Biplot

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris

iris = load_iris()
X = StandardScaler().fit_transform(iris.data)
pca = PCA(n_components=2).fit(X)
X_pca = pca.transform(X)

# Loading vectors (scaled for visibility)
loadings = pca.components_.T
n_features = loadings.shape[0]

print("=== Biplot Data ===\n")
print("Loading vectors (arrows):")
for i, fname in enumerate(iris.feature_names):
    print(f"  {fname:>20}: PC1={loadings[i,0]:+.3f}, PC2={loadings[i,1]:+.3f}")
    
print("\nCorrelation between features (from loadings):")
# Features with similar loading directions are correlated
for i in range(n_features):
    for j in range(i+1, n_features):
        cos_angle = np.dot(loadings[i], loadings[j]) / (
            np.linalg.norm(loadings[i]) * np.linalg.norm(loadings[j]))
        relation = "correlated" if cos_angle > 0.5 else (
            "anti-correlated" if cos_angle < -0.5 else "independent")
        print(f"  {iris.feature_names[i]:>20} ↔ {iris.feature_names[j]:<20}: "
              f"cos={cos_angle:+.3f} ({relation})")

# For matplotlib biplot:
# import matplotlib.pyplot as plt
# fig, ax = plt.subplots(figsize=(10, 8))
# for cls, name, color in zip([0,1,2], iris.target_names, ['r','b','g']):
#     mask = iris.target == cls
#     ax.scatter(X_pca[mask, 0], X_pca[mask, 1], c=color, label=name, alpha=0.5)
# scale = 3
# for i, fname in enumerate(iris.feature_names):
#     ax.arrow(0, 0, loadings[i,0]*scale, loadings[i,1]*scale,
#              head_width=0.1, head_length=0.05, fc='black', ec='black')
#     ax.text(loadings[i,0]*scale*1.15, loadings[i,1]*scale*1.15, fname, fontsize=10)
# ax.set_xlabel(f'PC1 ({pca.explained_variance_ratio_[0]*100:.1f}%)')
# ax.set_ylabel(f'PC2 ({pca.explained_variance_ratio_[1]*100:.1f}%)')
# ax.legend()
# plt.title('Biplot: Iris Dataset')
# plt.show()
```

---

## 8.5 Practical Visualization Pipeline

### Complete Pipeline for Any Dataset

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits

def pca_visualization_report(X, y=None, feature_names=None, target_names=None):
    """Complete PCA visualization pipeline with report."""
    
    # Standardize
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)
    
    # Full PCA for analysis
    pca_full = PCA()
    pca_full.fit(X_scaled)
    cumulative = np.cumsum(pca_full.explained_variance_ratio_)
    
    # 2D PCA for visualization
    pca_2d = PCA(n_components=2)
    X_2d = pca_2d.fit_transform(X_scaled)
    
    print("=" * 60)
    print("PCA VISUALIZATION REPORT")
    print("=" * 60)
    
    print(f"\nDataset: {X.shape[0]} samples × {X.shape[1]} features")
    
    # Variance analysis
    print(f"\n--- Variance Analysis ---")
    for thresh in [0.80, 0.90, 0.95, 0.99]:
        k = np.argmax(cumulative >= thresh) + 1
        print(f"  {thresh*100:.0f}% variance: {k} components "
              f"({k/X.shape[1]*100:.0f}% of original)")
    
    # Top PCs
    print(f"\n--- Top 5 PCs ---")
    for i in range(min(5, X.shape[1])):
        bar = '█' * int(pca_full.explained_variance_ratio_[i] * 40)
        print(f"  PC{i+1}: {pca_full.explained_variance_ratio_[i]*100:>5.1f}% {bar}")
    
    # 2D projection stats
    print(f"\n--- 2D Projection ---")
    print(f"  Variance captured: {sum(pca_2d.explained_variance_ratio_)*100:.1f}%")
    
    if y is not None:
        classes = np.unique(y)
        print(f"\n--- Class Separation in 2D ---")
        for cls in classes:
            mask = y == cls
            centroid = X_2d[mask].mean(axis=0)
            name = target_names[cls] if target_names is not None else f"Class {cls}"
            print(f"  {name:>15}: centroid=({centroid[0]:+.2f}, {centroid[1]:+.2f}), "
                  f"n={mask.sum()}")
    
    # Feature contributions
    if feature_names is not None:
        print(f"\n--- Top Feature Contributions ---")
        for pc_idx in range(2):
            loadings = np.abs(pca_2d.components_[pc_idx])
            top_idx = np.argsort(loadings)[::-1][:3]
            features = [feature_names[i] for i in top_idx]
            print(f"  PC{pc_idx+1}: {', '.join(features)}")
    
    return X_2d, pca_2d

# Usage example
digits = load_digits()
X_2d, pca = pca_visualization_report(
    digits.data, digits.target,
    target_names=[str(i) for i in range(10)]
)
```

---

## 8.6 When PCA Visualization Fails

```
PCA visualization has limitations:

  ✅ Works well when:
    • Classes are linearly separable
    • First 2-3 PCs capture >60% variance
    • Clusters are roughly ellipsoidal

  ❌ Fails when:
    • Classes overlap heavily in PC space
    • Important structure is in lower PCs
    • Clusters are non-linear (use t-SNE/UMAP instead)
    • 2 PCs capture <30% variance (too much info lost)

  PCA finds GLOBAL linear structure:
    → Good for overall data shape
    → Bad for local/non-linear structure
    
  For non-linear visualization:
    → t-SNE: preserves local structure
    → UMAP: preserves both local and global
```

---

## 8.7 Summary Table

| Visualization Tool | What It Shows | When to Use |
|-------------------|---------------|-------------|
| **2D scatter (PC1 vs PC2)** | Cluster structure, outliers | First exploratory look |
| **3D scatter** | Additional separation | When 2D is insufficient |
| **Loading vectors** | Feature contributions to PCs | Interpreting what PCs mean |
| **Biplot** | Data + loadings combined | Understanding features AND clusters |
| **Scree/variance plot** | Information per component | Choosing number of components |
| **Pair plot of PCs** | All PC pairs | Checking if structure is in PC3+ |

---

## 8.8 Revision Questions

1. **Explain the PCA visualization pipeline.** What are the steps from raw data to a 2D scatter plot? Why is standardization important?

2. **What do the axes in a PCA scatter plot represent?** A student asks "what does PC1 = 2.5 mean for a data point?" How would you answer?

3. **What are loading vectors** and how do you interpret them in a biplot? Two feature arrows point in nearly the same direction — what does this mean?

4. **A 2D PCA plot captures only 25% of variance.** Should you trust the visual structure in this plot? What alternatives would you suggest?

5. **Compare PCA visualization** with t-SNE and UMAP. When is PCA preferred? When should you use the alternatives?

6. **Given the loading vectors** PC1 = [0.5, 0.5, -0.5, -0.5] for features [height, weight, reading_score, math_score], interpret what PC1 represents. What groups of points would score high vs. low on PC1?

---

[← Previous: Explained Variance Ratio](./07-explained-variance-ratio.md) | [Next: Limitations →](./09-limitations.md)
