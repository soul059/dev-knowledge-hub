# 2. PCA Intuition

[← Previous: Dimensionality Reduction Motivation](./01-dimensionality-reduction-motivation.md) | [Next: Mathematical Formulation →](./03-mathematical-formulation.md)

---

## Chapter Overview

PCA (Principal Component Analysis) finds the **directions of maximum variance** in the data and projects the data onto these directions. The first principal component captures the most variance, the second captures the most remaining variance (orthogonal to the first), and so on. This chapter builds geometric intuition for what PCA does, why variance matters, and how projection onto principal components preserves the most information.

---

## 2.1 PCA as Finding Directions of Maximum Variance

### The Core Idea

```
Given a cloud of data points in d dimensions, PCA asks:

  "What is the SINGLE DIRECTION along which the data
   varies the most?"

  → That direction is Principal Component 1 (PC1)

  "What is the next direction (perpendicular to PC1)
   along which the remaining data varies the most?"

  → That direction is Principal Component 2 (PC2)

  Continue until you have d orthogonal directions.
```

### ASCII Visualization: 2D Data

```
  y │                    ·
    │                 ·    ·
    │              ·    ·
    │           ·    ·        Data forms an elongated cloud
    │        ·    ·           tilted at ~45 degrees
    │     ·    ·
    │  ·    ·
    │    ·
    └────────────────────── x

  Now add the principal component directions:

  y │               ╱  ·
    │            ╱  ·    ·
    │         ╱  ·    ·            PC1: direction of MAX variance
    │      ╱  ·    ·               (along the elongated axis)
    │   ╱  ·    ·     PC1 ──→
    │╱  ·    ·            ╱
    ·    ·             ╱
    │  ·            ╱
    │            ╱
    │  ↑
    │  PC2: perpendicular to PC1
    │  (direction of MIN variance)
    └────────────────────── x

  PC1 captures the "spread" of the data
  PC2 captures the remaining "thickness"
```

---

## 2.2 Why Variance = Information

### The Key Insight

```
In PCA, "information" ≈ "variance"

  High variance direction → data points are spread out
                           → lots of distinguishing information
                           → KEEP this direction

  Low variance direction  → data points are bunched together
                           → little distinguishing information
                           → can DISCARD this direction

Example: Student data
  Feature 1: Exam scores [45, 67, 23, 89, 72, 55, 91, 34]  ← high variance
  Feature 2: Year of birth [2000, 2000, 2001, 2000, 2001, 2000, 2001, 2000] ← low variance

  Exam scores VARY a lot → useful for distinguishing students
  Year of birth barely varies → not useful for distinguishing
  → PCA would prioritize the exam score direction
```

### Variance and Reconstruction Error

```
The direction of maximum variance is also the direction that
MINIMIZES reconstruction error when projecting:

  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  Maximize variance along projection direction        │
  │        ⟺                                            │
  │  Minimize squared reconstruction error               │
  │                                                      │
  │  These two objectives give the SAME answer!          │
  │                                                      │
  └──────────────────────────────────────────────────────┘

  Projection onto PC1 (good):     Projection onto a bad direction:
  
  ·───→──x────────────→──·        ·
  ·──→───x────→──·                │↕ large error
  ·→─x──────────→─·               x
                                  │↕ large error
  ↕ small reconstruction errors   ·
    (points close to line)          x
                                  │↕
                                  ·
                                  (points far from line)
```

---

## 2.3 Projecting Data onto Principal Components

### What is a Projection?

```
Projecting a point x onto a direction w:

  The projection = (x · w / ||w||²) · w

  If w is a unit vector (||w|| = 1):
    projection coefficient = x · w = xᵀw
    
  This gives us the "coordinate" of x along direction w.

  Visual:
                    · x (original point)
                   ╱│
                  ╱ │
                 ╱  │ perpendicular
                ╱   │ distance
               ╱    │ (lost info)
  ────────────x'────┴──────────── w direction
              ↑
        projection of x onto w
        (preserved info)
```

### Projecting to Lower Dimensions

```
Original: d-dimensional data
Choose: k principal components (k < d)

New representation: k-dimensional

  x_original = [x₁, x₂, x₃, ..., x_d]  (d features)
       ↓
  PCA transform using k components
       ↓
  x_projected = [z₁, z₂, ..., z_k]      (k features)

  where:
    z₁ = xᵀw₁    (projection onto PC1)
    z₂ = xᵀw₂    (projection onto PC2)
    ...
    z_k = xᵀw_k  (projection onto PCk)

  Z = X · W_k    (matrix form)
  where W_k = [w₁ | w₂ | ... | w_k] is d×k matrix
```

### Step-by-Step: 2D → 1D Projection

```
Data (centered):
  x₁ = [2, 1]
  x₂ = [-1, -0.5]
  x₃ = [1, 0.5]
  x₄ = [-2, -1]

PC1 direction (found by PCA): w₁ = [0.894, 0.447]  (unit vector)

Projections onto PC1:
  z₁ = [2, 1] · [0.894, 0.447] = 2×0.894 + 1×0.447 = 2.235
  z₂ = [-1, -0.5] · [0.894, 0.447] = -1.118
  z₃ = [1, 0.5] · [0.894, 0.447] = 1.118
  z₄ = [-2, -1] · [0.894, 0.447] = -2.235

1D representation: z = [2.235, -1.118, 1.118, -2.235]

  Before (2D):                    After (1D):
  y │                             
    │  x₁(2,1)                    ──x₄──x₂────x₃──x₁──→ PC1
    │      x₃(1,0.5)              -2.24 -1.12  1.12  2.24
    │              
  ──┼──────────── x               4 points on a line!
    │  x₂(-1,-0.5)               
    │x₄(-2,-1)                   The relative ordering/spacing
                                   is preserved as much as possible.
```

---

## 2.4 Geometric Intuition: Rotation + Truncation

PCA can be understood as a two-step process:

```
Step 1: ROTATE the coordinate axes to align with the data

  Original axes:                  Rotated axes (PCA):
  
  y │     · ·                     PC2 │
    │   · · · ·                       │  ·  ·
    │ · · · · · ·         →           │· · · · · ·
    │   · · · ·                       │  ·  ·
    │     · ·                         │
    └──────────── x                   └──────────── PC1
    
  Data is tilted                  Data is axis-aligned
  relative to x,y                with PC1, PC2

Step 2: TRUNCATE by dropping the less important dimensions

  All PCs:                        Keep only PC1:
  
  PC2 │                           
      │  ·  ·                     ──·──·──·──·──·──·──→ PC1
      │· · · · · ·                
      │  ·  ·                     Dropped PC2 (small variance)
      │                           Kept PC1 (large variance)
      └──────────── PC1
```

### What PCA Preserves and Loses

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  PRESERVED (captured by kept PCs):                             │
│  • Directions of high variance → spread/structure              │
│  • Relative distances (approximately)                          │
│  • Cluster structure (usually)                                 │
│  • Overall shape of the data cloud                            │
│                                                                │
│  LOST (in discarded PCs):                                      │
│  • Directions of low variance → fine details, noise            │
│  • Some pairwise distances change                              │
│  • Original feature meaning (PCs are combinations)             │
│                                                                │
│  The amount preserved is measured by "explained variance ratio"│
│  keeping 95% of variance → 95% of information preserved       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 2.5 Principal Components Are Orthogonal

```
Key property: All principal components are MUTUALLY ORTHOGONAL

  w₁ · w₂ = 0
  w₁ · w₃ = 0
  w₂ · w₃ = 0
  ... etc.

  This means:
  1. No redundancy — each PC captures DIFFERENT information
  2. The projected coordinates are UNCORRELATED
  3. The PCs form an orthogonal basis for the subspace

  In 3D:
        PC3
        │  ╱ PC2
        │ ╱
        │╱
        └───── PC1

  Three perpendicular directions,
  ordered by variance captured:
    Var(PC1) ≥ Var(PC2) ≥ Var(PC3)
```

---

## 2.6 Variance Along Each PC

```
Eigenvalue λₖ = variance captured by PCk

Total variance = λ₁ + λ₂ + ... + λ_d = Σ λₖ

Explained variance ratio of PCk = λₖ / Σ λⱼ

Example (d=5):
  PC1: λ₁ = 8.5    →  8.5/12.0 = 70.8%    ████████████████
  PC2: λ₂ = 2.0    →  2.0/12.0 = 16.7%    ████
  PC3: λ₃ = 1.0    →  1.0/12.0 =  8.3%    ██
  PC4: λ₄ = 0.3    →  0.3/12.0 =  2.5%    █
  PC5: λ₅ = 0.2    →  0.2/12.0 =  1.7%    ▪
                       ──────────────────
  Total: 12.0          100.0%

  Keeping PC1 + PC2: 70.8% + 16.7% = 87.5% of variance
  Keeping PC1 + PC2 + PC3: 95.8% of variance
  
  → 3 components (out of 5) capture 95.8% of information!
  → Reduced from 5D to 3D with only 4.2% information loss
```

---

## 2.7 Python: PCA Intuition Demonstration

```python
import numpy as np
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

# Create 2D correlated data
np.random.seed(42)
n = 200
x1 = np.random.normal(0, 2, n)
x2 = 0.8 * x1 + np.random.normal(0, 0.5, n)  # Correlated with x1
X = np.column_stack([x1, x2])

# Center the data
X_centered = X - X.mean(axis=0)

# Fit PCA
pca = PCA()
X_pca = pca.fit_transform(X_centered)

print("=== PCA Results ===\n")
print(f"Original data shape: {X.shape}")
print(f"Mean (should be ≈0 after centering): {X_centered.mean(axis=0).round(6)}")

print(f"\nPrincipal Component Directions (unit vectors):")
for i, comp in enumerate(pca.components_):
    print(f"  PC{i+1}: [{comp[0]:.4f}, {comp[1]:.4f}]")

print(f"\nEigenvalues (variance per PC):")
for i, var in enumerate(pca.explained_variance_):
    print(f"  PC{i+1}: λ = {var:.4f}")

print(f"\nExplained Variance Ratio:")
for i, ratio in enumerate(pca.explained_variance_ratio_):
    bar = '█' * int(ratio * 40)
    print(f"  PC{i+1}: {ratio:.4f} ({ratio*100:.1f}%) {bar}")

print(f"\nCumulative variance: {np.cumsum(pca.explained_variance_ratio_)}")

# Verify orthogonality
dot_product = np.dot(pca.components_[0], pca.components_[1])
print(f"\nPC1 · PC2 = {dot_product:.10f} (should be ≈0)")

# Verify PCs are uncorrelated
corr = np.corrcoef(X_pca[:, 0], X_pca[:, 1])[0, 1]
print(f"Correlation of projected data: {corr:.10f} (should be ≈0)")

# Project onto PC1 only (2D → 1D)
X_1d = X_pca[:, :1]  # Keep only PC1
X_reconstructed = X_1d @ pca.components_[:1] + pca.mean_

# Reconstruction error
error = np.mean(np.sum((X_centered - (X_1d @ pca.components_[:1]))**2, axis=1))
total_var = np.sum(pca.explained_variance_)
print(f"\nReconstruction error (1 PC): {error:.4f}")
print(f"Total variance: {total_var:.4f}")
print(f"Ratio preserved: {1 - error/total_var:.4f} = {(1-error/total_var)*100:.1f}%")
```

---

## 2.8 Summary Table

| Concept | Description | Key Point |
|---------|-------------|-----------|
| **PC1** | Direction of maximum variance | Captures most information |
| **PC2** | Max variance ⊥ to PC1 | Captures next most info |
| **Orthogonal** | All PCs are perpendicular | No redundancy |
| **Eigenvalue λₖ** | Variance along PCk | Higher = more important |
| **Projection** | xᵀwₖ gives coordinate on PCk | Dimensionality reduction |
| **Variance = info** | High variance ↔ useful info | Low variance ↔ noise |
| **Rotation** | PCA rotates to align with data | Then truncates low-var directions |
| **Reconstruction** | Approximate x from k PCs | Error ↔ discarded variance |

---

## 2.9 Revision Questions

1. **In your own words, explain what PCA does** geometrically. Use the metaphor of rotation and truncation.

2. **Why does PCA equate variance with information?** Is this always a valid assumption? Give a counterexample where variance might not correspond to importance.

3. **Given 2D data centered at origin** with points [(3,1), (-3,-1), (2,0.5), (-2,-0.5)], what direction do you expect PC1 to point? Estimate the explained variance ratio for PC1.

4. **Why must principal components be orthogonal?** What would happen if we allowed non-orthogonal components?

5. **Explain the difference** between the projection of a point onto PC1 (a scalar) and the reconstruction of a point from PC1 (a vector in the original space).

6. **A dataset has 10 features** and PCA reveals that PC1 captures 90% of the variance. What does this suggest about the data structure? How many effective dimensions does the data have?

---

[← Previous: Dimensionality Reduction Motivation](./01-dimensionality-reduction-motivation.md) | [Next: Mathematical Formulation →](./03-mathematical-formulation.md)
