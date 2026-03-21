# 🌀 Unit 6, Chapter 4: The Curse of Dimensionality

> **K-Nearest Neighbors — Part 4 of 6**
>
> KNN seems simple and elegant — until you throw high-dimensional data at it. In high dimensions, the very concept of "nearest neighbor" breaks down. Distances become meaningless, data becomes impossibly sparse, and KNN's performance collapses. This chapter explains why, provides mathematical proof, and shows practical solutions.

---

## Table of Contents

1. [What Is the Curse of Dimensionality?](#1-what-is-the-curse-of-dimensionality)
2. [Distance Concentration](#2-distance-concentration)
3. [Volume of a Hypersphere](#3-volume-of-a-hypersphere)
4. [Sparsity in High Dimensions](#4-sparsity-in-high-dimensions)
5. [Visual Intuition](#5-visual-intuition)
6. [Practical Impact on KNN](#6-practical-impact-on-knn)
7. [Solutions: Feature Selection](#7-solutions-feature-selection)
8. [Solutions: PCA and Dimensionality Reduction](#8-solutions-pca-and-dimensionality-reduction)
9. [Dimensionality Reduction Before KNN Pipeline](#9-dimensionality-reduction-before-knn-pipeline)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. What Is the Curse of Dimensionality?

The **curse of dimensionality** (coined by Richard Bellman, 1961) refers to the collection of phenomena that arise when working in high-dimensional spaces — phenomena that are absent or mild in low dimensions.

### The Core Problem for KNN

```
  ┌────────────────────────────────────────────────────────────────┐
  │                                                                │
  │  In HIGH dimensions:                                           │
  │                                                                │
  │  1. All points become approximately EQUIDISTANT                │
  │     → "nearest" neighbor is barely closer than "farthest"      │
  │     → KNN can't distinguish meaningful neighbors               │
  │                                                                │
  │  2. Data becomes SPARSE                                        │
  │     → need exponentially more data to fill the space           │
  │     → neighbors are very far away                              │
  │                                                                │
  │  3. Volume concentrates near the SURFACE                       │
  │     → most of a hypersphere's volume is in a thin outer shell  │
  │     → data near the center is extremely rare                   │
  │                                                                │
  └────────────────────────────────────────────────────────────────┘
```

---

## 2. Distance Concentration

### The Phenomenon

As dimensionality increases, the ratio of the maximum distance to the minimum distance approaches 1.

```
  Mathematical Statement:

  For n random points in d dimensions:

       d_max - d_min
  lim  ─────────────  →  0      (as d → ∞)
  d→∞      d_min

  In words: The farthest point is barely farther than the nearest point.
```

### Numerical Demonstration

```
  Consider 100 random points uniformly distributed in [0,1]^d:

  ┌──────────┬───────────┬───────────┬──────────────┐
  │ Dims (d) │  d_min    │  d_max    │ d_max/d_min  │
  ├──────────┼───────────┼───────────┼──────────────┤
  │    2     │   0.03    │   1.28    │    42.7      │
  │    5     │   0.22    │   1.78    │     8.1      │
  │   10     │   0.73    │   2.31    │     3.2      │
  │   50     │   2.85    │   3.91    │     1.37     │
  │  100     │   4.32    │   5.21    │     1.21     │
  │  500     │  10.15    │  11.02    │     1.09     │
  │ 1000     │  14.51    │  15.34    │     1.06     │
  └──────────┴───────────┴───────────┴──────────────┘

  At d=2:   Nearest is 42× closer than farthest  → KNN works great!
  At d=1000: Nearest is only 1.06× closer         → KNN is guessing!
```

### Why This Happens (Intuition)

```
  In low dimensions:
    Some points cluster together, others are far apart.
    Clear distinction between "near" and "far."

  In high dimensions:
    Each dimension adds independent variation to the distance.
    By the Law of Large Numbers, the sum of squared differences
    converges to a similar value for ALL pairs of points.

    d(p,q)² = Σᵢ₌₁ᵈ (pᵢ - qᵢ)²

    As d → ∞, this sum averages out, making all distances similar.
```

### Python Demonstration

```python
import numpy as np

np.random.seed(42)
n_points = 200

print(f"{'Dims':>6}  {'Min Dist':>10}  {'Max Dist':>10}  {'Ratio':>8}  {'Spread%':>9}")
print("-" * 50)

for d in [2, 5, 10, 50, 100, 500, 1000]:
    points = np.random.uniform(0, 1, size=(n_points, d))

    # Compute all pairwise distances
    from scipy.spatial.distance import pdist
    dists = pdist(points, metric='euclidean')

    d_min = dists.min()
    d_max = dists.max()
    ratio = d_max / d_min
    spread = (d_max - d_min) / d_min * 100

    print(f"{d:6d}  {d_min:10.4f}  {d_max:10.4f}  {ratio:8.2f}  {spread:8.1f}%")
```

**Expected Output:**
```
  Dims    Min Dist    Max Dist     Ratio   Spread%
--------------------------------------------------
     2      0.0074      1.3742     185.7    18473%
     5      0.1258      1.8926      15.0     1404%
    10      0.4682      2.4519       5.2      424%
    50      2.3842      4.0213       1.7       69%
   100      3.7015      5.3201       1.4       44%
   500      9.5823     11.3129       1.2       18%
  1000     13.8942     15.6521       1.1       13%
```

---

## 3. Volume of a Hypersphere

### The Disappearing Hypersphere

The volume of a unit hypersphere (radius r = 1) relative to the enclosing unit hypercube decreases exponentially with dimension.

```
  Volume of d-dimensional unit ball:

              π^(d/2)
  V_d = ─────────────────
          Γ(d/2 + 1)

  Where Γ is the gamma function.
```

### Ratio: Hypersphere Volume / Hypercube Volume

```
  ┌──────────┬────────────────┬────────────────────┬──────────────┐
  │ Dims (d) │ V_sphere       │ V_cube = (2r)^d    │ Ratio        │
  ├──────────┼────────────────┼────────────────────┼──────────────┤
  │    1     │ 2.000          │ 2                  │ 1.000        │
  │    2     │ π = 3.142      │ 4                  │ 0.785        │
  │    3     │ 4π/3 = 4.189   │ 8                  │ 0.524        │
  │    5     │ 5.264          │ 32                 │ 0.164        │
  │   10     │ 2.550          │ 1024               │ 0.00249      │
  │   20     │ 0.0258         │ 1,048,576          │ 2.46×10⁻⁸   │
  │   50     │ ~0             │ ~10¹⁵              │ ~0           │
  │  100     │ ~0             │ ~10³⁰              │ ~0           │
  └──────────┴────────────────┴────────────────────┴──────────────┘

  The hypersphere vanishes inside the hypercube!
```

### What This Means for KNN

```
  Imagine your K neighbors must fit inside a hypersphere.
  
  In 2D: A circle of radius r covers πr² of the [0,1]² space.
         To cover 10% of area: r = √(0.1/π) ≈ 0.18

  In 10D: A hypersphere of radius r covers V₁₀·r¹⁰ of [0,1]¹⁰ space.
          To cover 10% of volume: r ≈ 0.74

  In 100D: Need r ≈ 0.955

  To capture the same FRACTION of data, you need a neighborhood
  that spans almost the ENTIRE feature range in each dimension!

  ┌──────────────────────────────────────────────┐
  │  "Local" in high dimensions means "global."  │
  │  KNN loses its locality advantage.           │
  └──────────────────────────────────────────────┘
```

### Volume Concentrates at the Surface

```
  Fraction of hypersphere volume in outer shell of thickness ε:

  f = 1 - (1 - ε)^d

  For ε = 0.01 (1% shell thickness):

  d = 2:    f = 1 - 0.99²   = 0.020   →  2% of volume in shell
  d = 10:   f = 1 - 0.99¹⁰  = 0.096   →  10% in shell
  d = 100:  f = 1 - 0.99¹⁰⁰ = 0.634   →  63% in shell
  d = 500:  f = 1 - 0.99⁵⁰⁰ = 0.993   →  99.3% in shell
  d = 1000: f = 1 - 0.99¹⁰⁰⁰ ≈ 1.0    →  ~100% in shell

  In high dimensions, virtually ALL volume is near the surface.
  Data points cluster near the edges of the space.
```

---

## 4. Sparsity in High Dimensions

### How Much Data Do You Need?

To maintain a fixed density of data points as dimensions increase:

```
  If you need 10 points per unit length in each dimension:

  d = 1:  10¹ = 10 points
  d = 2:  10² = 100 points
  d = 3:  10³ = 1,000 points
  d = 5:  10⁵ = 100,000 points
  d = 10: 10¹⁰ = 10 billion points!

  Data requirements grow EXPONENTIALLY with dimensions.
```

### ASCII Visualization

```
  1D: Dense coverage is easy
  ●●●●●●●●●●●●●●●●●●●●  (20 points fill the line)

  2D: Still manageable
  ● ● ● ● ●
  ● ● ● ● ●
  ● ● ● ● ●              (25 points: some coverage)
  ● ● ● ● ●
  ● ● ● ● ●

  3D: Getting sparse
  Each point occupies a cube
  25 points now cover very little of the 3D space

  10D: Astronomically sparse
  25 points in 10D space are like
  25 stars scattered across a galaxy
  — each point's nearest neighbor is very far away
```

### The Empty Space Problem

```
  In d dimensions with n uniformly distributed points in [0,1]^d:

  Expected distance to nearest neighbor:

                  Γ(1 + 1/d)
  E[d_nn] ≈  ─────────────────
               (n · V_d)^(1/d)

  This distance grows as d increases (for fixed n).

  For n = 1000 points:
  ┌──────┬─────────────────────────┐
  │  d   │  E[d_nn] (approx)      │
  ├──────┼─────────────────────────┤
  │  2   │  0.018                  │
  │  5   │  0.20                   │
  │ 10   │  0.53                   │
  │ 20   │  0.79                   │
  │ 50   │  0.93                   │
  │100   │  0.97                   │
  └──────┴─────────────────────────┘

  At d=100, even the NEAREST neighbor is at distance ~0.97
  (out of a max possible ~√100 ≈ 10). The space is empty!
```

---

## 5. Visual Intuition

### 2D vs High-D Neighborhood

```
  In 2D: A small circle around Q captures nearby, relevant points.

    ┌────────────────────┐
    │  ●     ○           │
    │    ● ╭───╮ ○       │
    │   ● │ Q  │  ○      │    Small neighborhood works.
    │    ● ╰───╯  ○      │    Neighbors are truly "local."
    │       ●  ○         │
    └────────────────────┘

  In 100D: To capture ANY neighbors, the "ball" must be huge.

    ┌────────────────────┐
    │╔════════════════╗  │
    │║  ●     ○       ║  │
    │║    ●       ○   ║  │
    │║   ●   Q    ○   ║  │    The neighborhood spans almost
    │║    ●       ○   ║  │    the entire space.
    │║       ●  ○     ║  │    "Neighbors" aren't local anymore.
    │╚════════════════╝  │
    └────────────────────┘
```

### Corner Concentration

```
  In high dimensions, most volume is in the CORNERS of a hypercube.

  2D: Corners hold ~21.5% of area (circle inscribed in square)

  ┌──────────┐
  │ ╲      ╱ │    Corners = outside circle
  │  ╲    ╱  │    = 1 - π/4 ≈ 21.5%
  │   (  )   │
  │  ╱    ╲  │
  │ ╱      ╲ │
  └──────────┘

  10D:  Corners hold 99.75% of volume
  100D: Corners hold 99.9999...% of volume

  Data points are almost ALL in corners far from the center.
  The center of the space is essentially empty.
```

---

## 6. Practical Impact on KNN

### Performance Degradation

```
  KNN Accuracy vs Dimensions (fixed n = 1000, K = 5)

  Accuracy
    ↑
  95│●●●
    │    ●●
  90│      ●●
    │        ●●
  85│          ●●
    │            ●●
  80│              ●●
    │                ●●
  75│                  ●●
    │                    ●●●●●●●●●●
  70│                              ●●●●●
    │
    └──────────────────────────────────────→ Dimensions
       2  5  10  20  30  50  100  200  500

  Performance plateaus around random-chance level
  when dimensions are much larger than √n.
```

### When Does the Curse Bite?

```
  Rule of thumb (very approximate):

  KNN works reasonably when:  n >> 2^d  or at least  n > 10^d

  For d = 10:   Need n >> 1024       (manageable)
  For d = 20:   Need n >> 1,048,576  (hard)
  For d = 50:   Need n >> 10¹⁵       (impossible)

  In practice:
  ┌──────────────────────────────────────────────┐
  │  d ≤ 10-20:  KNN usually works fine          │
  │  d = 20-50:  Performance degrades noticeably  │
  │  d > 50:     KNN often fails without          │
  │              dimensionality reduction          │
  └──────────────────────────────────────────────┘
```

### The Irrelevant Features Problem

```
  Suppose only 3 out of 100 features are relevant:

  True distance (relevant features only):
    d_true(A,Q) = 0.5,  d_true(B,Q) = 2.0

  Observed distance (all 100 features):
    d_obs(A,Q) = 7.83,  d_obs(B,Q) = 7.95

  The 97 irrelevant features add NOISE to the distance.
  The signal from the 3 relevant features is drowned out.

  Result: KNN selects the wrong neighbors!
```

---

## 7. Solutions: Feature Selection

### Why Feature Selection Helps

```
  Remove irrelevant / redundant features before applying KNN.
  Fewer dimensions → distances become meaningful again.

  100 features (97 irrelevant):        3 relevant features:
  ┌──────────────────────────┐        ┌──────────────────────┐
  │ All distances ≈ 7.8-8.0 │   →    │ Distances vary:      │
  │ Can't tell apart         │        │ 0.5, 2.0, 3.1, ...  │
  │ No useful neighbors      │        │ Clear nearest        │
  └──────────────────────────┘        │ neighbors!           │
                                      └──────────────────────┘
```

### Feature Selection Methods

```
  ┌─────────────────────┬─────────────────────────────────────────┐
  │  Method             │  Description                            │
  ├─────────────────────┼─────────────────────────────────────────┤
  │  Filter Methods     │  Statistical tests (correlation,        │
  │  (pre-processing)   │  mutual information, chi-squared)       │
  │                     │  to rank/select features independently  │
  │                     │  of the model.                          │
  ├─────────────────────┼─────────────────────────────────────────┤
  │  Wrapper Methods    │  Use KNN accuracy to evaluate subsets.  │
  │  (model-dependent)  │  Forward/backward selection,            │
  │                     │  recursive feature elimination.         │
  ├─────────────────────┼─────────────────────────────────────────┤
  │  Embedded Methods   │  Feature importance from another model  │
  │  (hybrid)           │  (e.g., Random Forest feature           │
  │                     │  importance) to select features for KNN │
  └─────────────────────┴─────────────────────────────────────────┘
```

### Python: Feature Selection for KNN

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.feature_selection import SelectKBest, mutual_info_classif

# Create dataset: 10 informative features + 90 noise features
X, y = make_classification(
    n_samples=500, n_features=100, n_informative=10,
    n_redundant=5, n_clusters_per_class=2, random_state=42
)

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# KNN on ALL 100 features
knn = KNeighborsClassifier(n_neighbors=5)
score_all = cross_val_score(knn, X_scaled, y, cv=5).mean()
print(f"KNN with all 100 features:  Accuracy = {score_all:.4f}")

# Feature selection: keep top 10 features
selector = SelectKBest(mutual_info_classif, k=10)
X_selected = selector.fit_transform(X_scaled, y)

score_selected = cross_val_score(knn, X_selected, y, cv=5).mean()
print(f"KNN with top 10 features:   Accuracy = {score_selected:.4f}")
print(f"Improvement: {(score_selected - score_all)*100:.1f}%")

# Try different numbers of features
print(f"\n{'# Features':>12}  {'Accuracy':>10}")
print("-" * 25)
for k_features in [5, 10, 15, 20, 30, 50, 100]:
    selector = SelectKBest(mutual_info_classif, k=min(k_features, 100))
    X_sel = selector.fit_transform(X_scaled, y)
    score = cross_val_score(knn, X_sel, y, cv=5).mean()
    print(f"{k_features:12d}  {score:10.4f}")
```

---

## 8. Solutions: PCA and Dimensionality Reduction

### Principal Component Analysis (PCA)

```
  PCA finds new orthogonal axes (principal components) that
  capture the maximum variance in the data.

  Original 100D data                PCA-reduced 10D data
  ┌────────────────────┐           ┌──────────────┐
  │  d₁ d₂ ... d₁₀₀   │   PCA    │ PC₁ PC₂...PC₁₀│
  │  x  x  ...  x     │  ────→   │  x   x  ... x │
  │  x  x  ...  x     │          │  x   x  ... x │
  │  ...               │          │  ...           │
  └────────────────────┘          └──────────────┘

  PCA preserves the most variance in fewer dimensions.
  KNN works better in the reduced space.
```

### How PCA Helps KNN

```
  1. Removes dimensions that are mostly noise
  2. Decorrelates features (each PC is orthogonal)
  3. Orders components by variance explained
  4. You choose how many PCs to keep

  Variance Explained by Principal Components:
  
  Var %
  40│ ████
  30│ ████ ████
  20│ ████ ████ ████
  10│ ████ ████ ████ ████ ████
   0│ ████ ████ ████ ████ ████ ████ ████ ··· (diminishing)
    └─────────────────────────────────────→
     PC1  PC2  PC3  PC4  PC5  PC6  PC7  ...

  Typically keep enough PCs to explain 90-95% of variance.
```

### Other Dimensionality Reduction Methods

```
  ┌──────────────────────┬──────────────────────────────────────┐
  │  Method              │  Notes                               │
  ├──────────────────────┼──────────────────────────────────────┤
  │  PCA                 │  Linear, unsupervised. Most common.  │
  │                      │  Fast, well-understood.              │
  ├──────────────────────┼──────────────────────────────────────┤
  │  LDA                 │  Linear, supervised. Maximizes class  │
  │  (Linear Discriminant│  separation. Ideal before KNN for    │
  │   Analysis)          │  classification.                     │
  ├──────────────────────┼──────────────────────────────────────┤
  │  t-SNE               │  Nonlinear. For visualization (2D/3D)│
  │                      │  NOT for preprocessing before KNN.   │
  ├──────────────────────┼──────────────────────────────────────┤
  │  UMAP                │  Nonlinear. Faster than t-SNE.       │
  │                      │  Can be used for preprocessing.      │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Autoencoders        │  Neural network-based. Nonlinear.    │
  │                      │  Powerful but complex.               │
  └──────────────────────┴──────────────────────────────────────┘
```

---

## 9. Dimensionality Reduction Before KNN Pipeline

### Complete Pipeline with scikit-learn

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.neighbors import KNeighborsClassifier
from sklearn.decomposition import PCA
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis as LDA
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score

# Create high-dimensional dataset
X, y = make_classification(
    n_samples=1000, n_features=200, n_informative=15,
    n_redundant=10, n_classes=3, n_clusters_per_class=2,
    random_state=42
)

# ───────────── Pipeline 1: Raw KNN (no reduction) ─────────────
pipe_raw = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier(n_neighbors=5))
])

# ───────────── Pipeline 2: PCA + KNN ─────────────
pipe_pca = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=20)),
    ('knn', KNeighborsClassifier(n_neighbors=5))
])

# ───────────── Pipeline 3: LDA + KNN ─────────────
pipe_lda = Pipeline([
    ('scaler', StandardScaler()),
    ('lda', LDA(n_components=2)),  # max = n_classes - 1 = 2
    ('knn', KNeighborsClassifier(n_neighbors=5))
])

# ───────────── Compare ─────────────
pipelines = {
    'Raw KNN (200 dims)': pipe_raw,
    'PCA(20) + KNN': pipe_pca,
    'LDA(2) + KNN': pipe_lda,
}

print(f"{'Method':>25}  {'CV Accuracy':>12}  {'CV Std':>8}")
print("-" * 50)
for name, pipe in pipelines.items():
    scores = cross_val_score(pipe, X, y, cv=5, scoring='accuracy')
    print(f"{name:>25}  {scores.mean():12.4f}  {scores.std():8.4f}")

# ───────────── Find Optimal Number of PCA Components ─────────────
print(f"\n{'PCA Components':>15}  {'CV Accuracy':>12}")
print("-" * 30)
for n_comp in [2, 5, 10, 15, 20, 30, 50, 100, 200]:
    pipe = Pipeline([
        ('scaler', StandardScaler()),
        ('pca', PCA(n_components=n_comp)),
        ('knn', KNeighborsClassifier(n_neighbors=5))
    ])
    score = cross_val_score(pipe, X, y, cv=5).mean()
    print(f"{n_comp:15d}  {score:12.4f}")
```

**Expected Output (approximate):**
```
                   Method   CV Accuracy    CV Std
--------------------------------------------------
      Raw KNN (200 dims)        0.7520    0.0234
            PCA(20) + KNN        0.8640    0.0187
             LDA(2) + KNN        0.8780    0.0156

  PCA Components   CV Accuracy
------------------------------
              2        0.6120
              5        0.7840
             10        0.8420
             15        0.8600
             20        0.8640   ← Best
             30        0.8520
             50        0.8200
            100        0.7800
            200        0.7520
```

### Visualization: Accuracy vs Components

```
  Accuracy
    ↑
  90│                ●──●
    │             ●       ●
  85│          ●             ●
    │       ●                   ●
  80│    ●                         ●
    │                                 ●──●
  75│ ●                                    ●
    │
  70│
    │
  65│●
    │
  60│
    └─────────────────────────────────────→ # PCA Components
       2   5  10  15  20  30  50 100  200

  Sweet spot: 15-20 components (captures signal, removes noise)
  Too few: underfitting (lost important variance)
  Too many: overfitting (kept noise dimensions)
```

### Comprehensive Pipeline with GridSearch

```python
from sklearn.model_selection import GridSearchCV

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA()),
    ('knn', KNeighborsClassifier())
])

param_grid = {
    'pca__n_components': [5, 10, 15, 20, 30],
    'knn__n_neighbors': [3, 5, 7, 9, 11],
    'knn__weights': ['uniform', 'distance'],
}

grid = GridSearchCV(pipe, param_grid, cv=5, scoring='accuracy', n_jobs=-1)
grid.fit(X, y)

print(f"Best Parameters: {grid.best_params_}")
print(f"Best CV Accuracy: {grid.best_score_:.4f}")
```

---

## 10. Summary Table

| Concept | Description | Impact on KNN |
|---------|-------------|---------------|
| **Distance concentration** | All distances converge to similar values | Can't identify meaningful neighbors |
| **Volume of hypersphere** | Volume → 0 relative to hypercube | Neighborhoods must be huge to capture data |
| **Surface concentration** | Volume concentrates in thin outer shell | Data clusters at edges, center is empty |
| **Sparsity** | Need exponentially more data | Neighbors are far away; not truly "local" |
| **Irrelevant features** | Noise in distance calculations | Wrong neighbors selected |
| **Feature selection** | Remove irrelevant/redundant features | Restores distance meaningfulness |
| **PCA** | Linear projection to lower dimensions | Removes noise; decorrelates features |
| **LDA** | Supervised linear projection | Maximizes class separation; ideal for KNN |

### Quick Reference: When to Worry

```
  ┌─────────────────────────────────────────────────────┐
  │  Dimensions vs Samples:                             │
  │                                                     │
  │  d < 10, n > 100:      ✅ KNN works fine            │
  │  d = 10-30, n > 1000:  ⚠️ Consider reduction        │
  │  d > 30, n < 10000:    ❌ Reduction strongly advised │
  │  d > 100:              ❌ Always reduce first         │
  └─────────────────────────────────────────────────────┘
```

---

## 11. Revision Questions

**Q1.** Explain the concept of distance concentration. If you have 500 points in 200 dimensions, approximately how much closer is the nearest neighbor compared to the farthest? Why is this a problem for KNN?

**Q2.** Calculate the fraction of a hypersphere's volume that lies in the outer 5% shell (ε = 0.05) for d = 2, d = 10, d = 50, and d = 100. What trend do you observe?

**Q3.** You have a dataset with 1000 samples and 100 features, but only about 8 features are truly informative. Describe two different strategies to improve KNN performance, and explain which you'd try first and why.

**Q4.** After applying PCA to reduce from 100 dimensions to 20, your KNN accuracy improves from 72% to 89%. Explain why reducing information (dropping 80 components) actually improved performance.

**Q5.** Compare PCA and LDA as dimensionality reduction methods before KNN. When would you prefer LDA over PCA, and what is LDA's main limitation?

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [6.3 Weighted KNN](./03-weighted-knn.md) | **6.4 Curse of Dimensionality** | [6.5 KNN Classification & Regression](./05-knn-classification-regression.md) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 4 of 6*
