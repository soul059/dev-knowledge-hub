# 🌀 t-SNE (t-Distributed Stochastic Neighbor Embedding)

> **Chapter 7.2 — Nonlinear Dimensionality Reduction for Visualization**

---

## 📌 Overview

**t-SNE** (t-Distributed Stochastic Neighbor Embedding) is a nonlinear dimensionality reduction technique specifically designed for **visualizing high-dimensional data** in 2D or 3D. Developed by **Laurens van der Maaten and Geoffrey Hinton (2008)**, it has become one of the most popular tools for exploratory data analysis and understanding cluster structures.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  HIGH-DIMENSIONAL SPACE              LOW-DIMENSIONAL SPACE (2D)     │
│  ┌───────────────────┐               ┌───────────────────┐          │
│  │ Points with        │    t-SNE     │ Points arranged    │          │
│  │ pairwise            │  ────────►  │ to preserve        │          │
│  │ similarities        │             │ neighborhood       │          │
│  │ (Gaussian)          │             │ structure          │          │
│  │                     │             │ (Student-t)        │          │
│  └───────────────────┘               └───────────────────┘          │
│                                                                      │
│  Key: Minimize the difference (KL divergence) between the           │
│       probability distributions in high-D and low-D spaces          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Why t-SNE? The Limitations of Linear Methods

```
  Linear Methods (PCA)          t-SNE
  ────────────────────          ──────
  
  Swiss Roll in 3D:            Swiss Roll "unrolled":
  
     ╱─────╲                      ●●●●●●●●●●●
    │ ╱───╲ │                     ●●●●●●●●●●●
    ││╱─╲ ││                      ●●●●●●●●●●●
    │││  │││                      ●●●●●●●●●●●
    ││╲─╱ ││                      ●●●●●●●●●●●
    │ ╲───╱ │   PCA flattens      ●●●●●●●●●●●
     ╲─────╱    → overlaps!       (structure preserved!)

  PCA projects to 2D:          t-SNE maps to 2D:
  ●●●●●●●●●                   Clean unrolled manifold
  (all overlapping!)           (neighbors preserved!)
```

---

## 🔢 Mathematical Foundation

### Step 1: Pairwise Similarities in High-Dimensional Space

For each pair of points xᵢ and xⱼ, compute the **conditional probability** that xᵢ would pick xⱼ as its neighbor under a Gaussian distribution centered at xᵢ:

```
                    exp(-‖xᵢ - xⱼ‖² / 2σᵢ²)
p(j|i) = ──────────────────────────────────────────
           Σ_{k≠i} exp(-‖xᵢ - xₖ‖² / 2σᵢ²)
```

Then **symmetrize** to get the joint probability:

```
         p(j|i) + p(i|j)
pᵢⱼ =  ─────────────────
              2N
```

```
┌───────────────────────────────────────────────────────┐
│ INTUITION:                                            │
│                                                       │
│ pᵢⱼ is HIGH when xᵢ and xⱼ are CLOSE (neighbors)    │
│ pᵢⱼ is LOW  when xᵢ and xⱼ are FAR   (dissimilar)   │
│                                                       │
│ σᵢ adapts per point (controlled by perplexity)        │
└───────────────────────────────────────────────────────┘
```

### Step 2: Pairwise Similarities in Low-Dimensional Space

For the low-dimensional map points yᵢ and yⱼ, use a **Student-t distribution** (1 degree of freedom = Cauchy distribution):

```
           (1 + ‖yᵢ - yⱼ‖²)⁻¹
qᵢⱼ = ──────────────────────────────
        Σ_{k≠l} (1 + ‖yₖ - yₗ‖²)⁻¹
```

### Step 3: Why Student-t Distribution? (The Crowding Problem)

```
┌──────────────────────────────────────────────────────────────┐
│                    THE CROWDING PROBLEM                       │
│                                                              │
│  In high-D space, many points can be equidistant from a      │
│  central point (surface of a hypersphere grows as r^(d-1)).  │
│                                                              │
│  In 2D, there's much less "room" — points get crowded!       │
│                                                              │
│  High-D (10D):           Low-D (2D):                         │
│       ○   ○                    ○                             │
│     ○   ●   ○            ○  ○● ○  ○                          │
│       ○   ○              (all squeezed together!)            │
│     ○       ○                                                │
│  (plenty of room)                                            │
│                                                              │
│  SOLUTION: Use Student-t distribution in low-D               │
│  • Heavier tails than Gaussian                               │
│  • Allows moderate distances in map to represent             │
│    large distances in high-D                                 │
│  • Nearby points stay close; far points get pushed farther   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```
    Probability
    ▲
    │  ╲ Gaussian
    │   ╲
    │    ╲      Student-t
    │     ╲   ╱────────────────
    │      ╲╱
    │       ╲──────────────────  ← Heavier tail
    │
    └───────────────────────────► Distance
    
    Student-t has heavier tails:
    • Moderate low-D distances still have reasonable probability
    • This "makes room" for points that were far in high-D
```

### Step 4: Objective Function — KL Divergence

Minimize the **Kullback-Leibler divergence** between P (high-D) and Q (low-D):

```
                         pᵢⱼ
C = KL(P‖Q) = Σᵢ Σⱼ pᵢⱼ log ────
                         qᵢⱼ
```

```
┌──────────────────────────────────────────────────────────┐
│ KL Divergence Properties:                                │
│                                                          │
│ • KL(P‖Q) ≥ 0  (always non-negative)                    │
│ • KL(P‖Q) = 0  iff P = Q  (perfect match)               │
│ • NOT symmetric: KL(P‖Q) ≠ KL(Q‖P)                      │
│                                                          │
│ • Penalizes HEAVILY when pᵢⱼ is large but qᵢⱼ is small  │
│   → Neighbors in high-D MUST be neighbors in low-D      │
│                                                          │
│ • Penalizes LIGHTLY when pᵢⱼ is small but qᵢⱼ is large  │
│   → Non-neighbors can end up close (acceptable)          │
│                                                          │
│ This asymmetry means t-SNE preserves LOCAL structure!    │
└──────────────────────────────────────────────────────────┘
```

### Step 5: Gradient Descent Optimization

The gradient with respect to yᵢ:

```
∂C         
──── = 4 Σⱼ (pᵢⱼ - qᵢⱼ)(yᵢ - yⱼ)(1 + ‖yᵢ - yⱼ‖²)⁻¹
∂yᵢ       
```

```
Attractive force (pᵢⱼ > qᵢⱼ): Points that should be close → PULL together
Repulsive force  (pᵢⱼ < qᵢⱼ): Points that shouldn't be close → PUSH apart

                     PULL ←──── yᵢ ────► PUSH
                   (neighbors)        (non-neighbors)
```

---

## 🎛️ The Perplexity Parameter

Perplexity controls the effective number of neighbors each point considers:

```
Perplexity = 2^H(Pᵢ)

where H(Pᵢ) = -Σⱼ p(j|i) log₂ p(j|i)  is the Shannon entropy
```

```
┌─────────────────────────────────────────────────────────────┐
│                  PERPLEXITY EFFECTS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Low perplexity (5-10):          High perplexity (50-100):  │
│  ┌──────────────────┐            ┌──────────────────┐       │
│  │ ○○  ○○  ●●  ●●   │            │    ○○○○   ●●●●    │       │
│  │ ○○  ○○  ●●  ●●   │            │    ○○○○   ●●●●    │       │
│  │ Very local        │            │  More global       │       │
│  │ Many small groups │            │  Fewer large groups│       │
│  └──────────────────┘            └──────────────────┘       │
│                                                             │
│  Rule of thumb: perplexity ≈ 5 to 50                        │
│  Should be smaller than N (number of points)                │
│                                                             │
│  Too low  → noise dominates, spurious clusters              │
│  Too high → structure washed out, everything merged         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 t-SNE Algorithm: Complete Pipeline

```
╔══════════════════════════════════════════════════════════════╗
║                    t-SNE ALGORITHM                          ║
╠══════════════════════════════════════════════════════════════╣
║                                                             ║
║  INPUT: X (N×d), perplexity, n_iter, learning_rate          ║
║                                                             ║
║  1. COMPUTE PAIRWISE AFFINITIES (High-D)                    ║
║     • For each point, find σᵢ using binary search           ║
║       to match target perplexity                            ║
║     • Compute conditional probabilities p(j|i)              ║
║     • Symmetrize: pᵢⱼ = (p(j|i) + p(i|j)) / 2N            ║
║                                                             ║
║  2. INITIALIZE LOW-D MAP                                    ║
║     • Random initialization (small Gaussian)                ║
║     • Or PCA initialization (often better)                  ║
║                                                             ║
║  3. EARLY EXAGGERATION (first ~250 iterations)              ║
║     • Multiply all pᵢⱼ by factor (e.g., 12)                ║
║     • Forces clusters to form tight groups early            ║
║                                                             ║
║  4. GRADIENT DESCENT (n_iter iterations)                    ║
║     • Compute qᵢⱼ using Student-t kernel                   ║
║     • Compute gradient of KL divergence                     ║
║     • Update: yᵢ ← yᵢ + η·∇ + α·(yᵢᵗ - yᵢᵗ⁻¹)            ║
║     • (momentum term for faster convergence)                ║
║                                                             ║
║  OUTPUT: Y (N×2 or N×3) — low-dimensional embedding        ║
║                                                             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🐍 Python Implementation

### Scikit-learn

```python
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits
import matplotlib.pyplot as plt
import numpy as np

# Load digits dataset (64 features, 10 classes)
digits = load_digits()
X, y = digits.data, digits.target

# Apply t-SNE
tsne = TSNE(
    n_components=2,          # Output dimensions (2 or 3)
    perplexity=30,           # Effective number of neighbors
    learning_rate='auto',    # Step size (default: 'auto' → N/12)
    n_iter=1000,             # Number of iterations
    init='pca',              # Initialization method
    random_state=42,         # Reproducibility
    metric='euclidean'       # Distance metric
)

X_tsne = tsne.fit_transform(X)

print(f"Original shape: {X.shape}")         # (1797, 64)
print(f"t-SNE shape:    {X_tsne.shape}")    # (1797, 2)
print(f"KL divergence:  {tsne.kl_divergence_:.4f}")
print(f"Iterations:     {tsne.n_iter_}")

# Visualization
plt.figure(figsize=(12, 8))
scatter = plt.scatter(X_tsne[:, 0], X_tsne[:, 1], 
                      c=y, cmap='tab10', s=15, alpha=0.7)
plt.colorbar(scatter, ticks=range(10), label='Digit')
plt.title('t-SNE of Handwritten Digits', fontsize=14)
plt.xlabel('t-SNE Component 1')
plt.ylabel('t-SNE Component 2')
plt.tight_layout()
plt.savefig('tsne_digits.png', dpi=150)
plt.show()
```

### Exploring Perplexity Effects

```python
fig, axes = plt.subplots(2, 3, figsize=(18, 12))
perplexities = [5, 10, 30, 50, 100, 200]

for ax, perp in zip(axes.flatten(), perplexities):
    tsne = TSNE(n_components=2, perplexity=perp, 
                random_state=42, n_iter=1000)
    X_embedded = tsne.fit_transform(X)
    
    ax.scatter(X_embedded[:, 0], X_embedded[:, 1], 
               c=y, cmap='tab10', s=5, alpha=0.6)
    ax.set_title(f'Perplexity = {perp}', fontsize=12)
    ax.set_xticks([])
    ax.set_yticks([])

plt.suptitle('t-SNE: Effect of Perplexity', fontsize=16)
plt.tight_layout()
plt.savefig('tsne_perplexity.png', dpi=150)
plt.show()
```

### 3D t-SNE

```python
from mpl_toolkits.mplot3d import Axes3D

tsne_3d = TSNE(n_components=3, perplexity=30, random_state=42)
X_3d = tsne_3d.fit_transform(X)

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')
scatter = ax.scatter(X_3d[:, 0], X_3d[:, 1], X_3d[:, 2],
                     c=y, cmap='tab10', s=10, alpha=0.6)
ax.set_title('3D t-SNE of Digits')
plt.colorbar(scatter)
plt.show()
```

---

## ⚠️ How to Interpret t-SNE Plots (Critical!)

```
╔══════════════════════════════════════════════════════════════════╗
║              t-SNE INTERPRETATION RULES                        ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                 ║
║  ✅ MEANINGFUL:                                                  ║
║  • Points in the same cluster are genuinely similar             ║
║  • Distinct clusters indicate genuine group structure            ║
║  • Relative position of points WITHIN a cluster                 ║
║                                                                 ║
║  ❌ NOT MEANINGFUL:                                              ║
║  • Distance BETWEEN clusters                                    ║
║  • Size of clusters                                             ║
║  • Density of clusters                                          ║
║  • Global position of clusters                                  ║
║                                                                 ║
║  WHY?                                                           ║
║  • t-SNE uses adaptive σᵢ per point → different scales          ║
║  • KL divergence asymmetry → local structure prioritized        ║
║  • Optimization is non-convex → different runs give             ║
║    different layouts                                            ║
║                                                                 ║
╚══════════════════════════════════════════════════════════════════╝
```

```
  COMMON MISINTERPRETATIONS:
  
  ❌ "Cluster A is far from Cluster B,             ✅ "Clusters A and B exist
     so they are very different"                       as distinct groups"
  
  ❌ "Cluster A is bigger than B,                   ✅ "Both A and B form
     so it has more variance"                          coherent clusters"
  
  ❌ "Cluster A is denser than B,                   ✅ "Run t-SNE multiple times
     so it's tighter in original space"                 and look for consistent
                                                        clusters"
```

---

## 📋 Practical Tips

```
┌─────────────────────────────────────────────────────────────┐
│                    t-SNE BEST PRACTICES                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. PREPROCESSING                                           │
│     • Always scale/normalize your data first                │
│     • Consider PCA to 50 dims before t-SNE (speeds up!)     │
│     • Reduces noise and computation time                    │
│                                                             │
│  2. PERPLEXITY                                              │
│     • Start with 30 (default), try 5-50 range               │
│     • Should be less than N                                 │
│     • Run with multiple values, compare results             │
│                                                             │
│  3. ITERATIONS                                              │
│     • At least 1000 (default), up to 5000                   │
│     • Check if KL divergence has converged                  │
│     • Early stopping can produce misleading results         │
│                                                             │
│  4. LEARNING RATE                                           │
│     • Use 'auto' (N/12) in modern sklearn                   │
│     • Too high → scattered, no structure                    │
│     • Too low  → points collapse together                   │
│                                                             │
│  5. REPRODUCIBILITY                                         │
│     • Always set random_state                               │
│     • Run multiple times and compare                        │
│     • Use PCA initialization for more stable results        │
│                                                             │
│  6. SCALABILITY                                             │
│     • O(N²) computation → slow for N > 10,000              │
│     • Use Barnes-Hut approximation (method='barnes_hut')    │
│     • Consider UMAP for larger datasets                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```python
# Recommended pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
from sklearn.pipeline import Pipeline

# Best practice: Scale → PCA → t-SNE
pipeline_steps = [
    StandardScaler(),                       # Normalize features
    PCA(n_components=50),                   # Reduce to 50 dims
    TSNE(n_components=2, perplexity=30,     # t-SNE to 2D
         init='pca', random_state=42)
]

# Manual pipeline (TSNE doesn't support Pipeline directly)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

pca = PCA(n_components=50)
X_pca = pca.fit_transform(X_scaled)

tsne = TSNE(n_components=2, perplexity=30, 
            init='pca', random_state=42, n_iter=2000)
X_tsne = tsne.fit_transform(X_pca)
```

---

## ⏱️ Computational Complexity

```
┌────────────────────────────────┬──────────────────────────┐
│         Method                 │     Complexity           │
├────────────────────────────────┼──────────────────────────┤
│ Exact t-SNE                   │ O(N²) per iteration      │
│ Barnes-Hut t-SNE              │ O(N log N) per iteration │
│ FIt-SNE (FFT-accelerated)     │ O(N) per iteration       │
├────────────────────────────────┼──────────────────────────┤
│ Memory (exact)                │ O(N²)                    │
│ Memory (Barnes-Hut)           │ O(N)                     │
├────────────────────────────────┼──────────────────────────┤
│ Practical limit (exact)       │ ~5,000 samples           │
│ Practical limit (Barnes-Hut)  │ ~100,000 samples         │
└────────────────────────────────┴──────────────────────────┘
```

---

## 🏭 Applications

| Domain | Application |
|--------|-------------|
| **Genomics** | Visualizing single-cell RNA-seq data (thousands of cells, thousands of genes) |
| **NLP** | Visualizing word embeddings (Word2Vec, GloVe) |
| **Computer Vision** | Visualizing CNN feature spaces |
| **Cybersecurity** | Visualizing network traffic patterns |
| **Drug Discovery** | Visualizing molecular fingerprint spaces |
| **Social Networks** | Visualizing community structures |

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Full Name** | t-Distributed Stochastic Neighbor Embedding |
| **Year** | 2008 (van der Maaten & Hinton) |
| **Type** | Nonlinear, unsupervised dimensionality reduction |
| **Primary Use** | Visualization (2D/3D) |
| **High-D Similarity** | Gaussian kernel → pᵢⱼ |
| **Low-D Similarity** | Student-t (1 df) → qᵢⱼ |
| **Objective** | Minimize KL(P ‖ Q) |
| **Key Parameter** | Perplexity (5-50 typical) |
| **Complexity** | O(N²) exact, O(N log N) Barnes-Hut |
| **Preserves** | Local structure ✅, Global structure ❌ |
| **New Points** | Cannot embed (no transform method) |
| **sklearn** | `TSNE(n_components=2, perplexity=30)` |

---

## ❓ Revision Questions

1. **Explain the crowding problem and why t-SNE uses a Student-t distribution instead of a Gaussian in the low-dimensional space.**
   Discuss how volume scales with dimensionality and how heavy tails provide relief.

2. **What does perplexity control in t-SNE, and how does changing it affect the resulting visualization?**
   Relate perplexity to the effective number of neighbors and entropy.

3. **Why is KL divergence asymmetric, and what consequences does this have for what t-SNE preserves (local vs. global structure)?**
   Explain the penalty structure: high pᵢⱼ / low qᵢⱼ vs. low pᵢⱼ / high qᵢⱼ.

4. **List three things you CANNOT reliably interpret from a t-SNE plot and explain why.**
   Discuss inter-cluster distances, cluster sizes, and cluster densities.

5. **A colleague runs t-SNE on 50,000 samples with 10,000 features and it takes hours. What preprocessing steps would you recommend to speed it up?**
   Consider PCA preprocessing, Barnes-Hut approximation, and UMAP as an alternative.

6. **Why can't t-SNE embed new (unseen) data points? How does this limit its use in production systems?**
   Discuss the lack of a parametric mapping and contrast with PCA/UMAP.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← LDA](./01-lda.md) | [📂 Unsupervised Learning](../README.md) | [UMAP →](./03-umap.md) |

---

> **Key Takeaway:** t-SNE is the gold standard for high-dimensional data visualization. It excels at revealing local cluster structure but remember: cluster distances, sizes, and densities in t-SNE plots are NOT meaningful. Always run with multiple perplexity values and random seeds.
