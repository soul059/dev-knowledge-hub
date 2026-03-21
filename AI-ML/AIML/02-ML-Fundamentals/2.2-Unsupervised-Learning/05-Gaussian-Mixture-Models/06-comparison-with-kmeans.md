# 6. GMM vs. K-Means: A Complete Comparison

[← Previous: Model Selection (BIC/AIC)](./05-model-selection-bic-aic.md) | [Back to GMM Unit →](./README.md)

---

## Chapter Overview

Gaussian Mixture Models can be viewed as a **generalization of K-Means**. K-Means is a special case of GMM where covariances are spherical, equal, and approach zero (hard assignments). This chapter establishes the precise mathematical relationship, compares flexibility, computational cost, and provides a practical decision guide.

---

## 6.1 GMM as a Generalization of K-Means

### The Mathematical Connection

```
K-Means is GMM with three constraints:

  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  1. Σₖ = σ²I  for all k     (spherical, identical)         │
  │  2. πₖ = 1/K  for all k     (equal weights)                │
  │  3. σ² → 0                   (hard assignments)             │
  │                                                              │
  │  As σ² → 0, the responsibilities become:                    │
  │                                                              │
  │  γ(zₙₖ) → { 1  if k = argmin_j ||xₙ - μⱼ||²              │
  │            { 0  otherwise                                    │
  │                                                              │
  │  The soft GMM becomes hard K-Means!                         │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

### Generalization Hierarchy

```
                        GMM (full covariance)
                       ╱                      ╲
                      ╱                        ╲
           GMM (diagonal)              GMM (tied covariance)
                    ╱                              ╲
                   ╱                                ╲
        GMM (spherical)                              ╲
                 │                                    │
                 │ (σ² → 0, equal weights)            │
                 ▼                                    │
             K-Means                                  │
                                                      │
  Each arrow down = adding constraints = losing flexibility
  Each arrow up   = removing constraints = gaining flexibility
```

---

## 6.2 Feature-by-Feature Comparison

| Feature | K-Means | GMM |
|---------|---------|-----|
| **Assignment type** | Hard (0 or 1) | Soft (probabilities [0,1]) |
| **Cluster shape** | Spherical only | Ellipsoidal (any orientation) |
| **Cluster size** | Tends toward equal | Any size (controlled by πₖ) |
| **Cluster density** | Uniform (no density model) | Modeled by Σₖ |
| **Number of clusters** | Must specify K | Must specify K (but BIC/AIC help) |
| **Noise handling** | None (all assigned) | Low-probability = potential noise |
| **Probabilistic model** | No | Yes (full generative model) |
| **Objective** | Minimize inertia | Maximize log-likelihood |
| **Algorithm** | Lloyd's (hard EM) | EM (soft EM) |
| **Convergence** | Local minimum | Local maximum |
| **Deterministic** | No (random init) | No (random init) |
| **Parameters per cluster** | d (centroid) | d + d(d+1)/2 + 1 |
| **Time per iteration** | O(NKd) | O(NKd²) for full covariance |
| **Memory** | O(Nd + Kd) | O(Nd + Kd²) |
| **Scalability** | Excellent (MiniBatch) | Moderate (no MiniBatch) |
| **New point prediction** | Yes (distance to centroid) | Yes (posterior probability) |
| **Density estimation** | No | Yes (p(x) = Σ πₖN(x\|μₖ,Σₖ)) |

---

## 6.3 Cluster Shape Flexibility

```
K-Means (spherical Voronoi):

  Data with elongated clusters:     K-Means result:
     · · ·                           A A A
    · · · · ·                       A A B B B
   · · · · · · ·                   A A B B B B B
    · · · · ·                       A A B B B
     · · ·                           A A A
         · · · · ·                       B B B B B
        · · · · · · ·                   B B B B B B B
         · · · · ·                       B B B B B
                                    
  Vertical boundary cuts through    K-Means can't capture
  both elongated clusters!          the elongated shape.


GMM (ellipsoidal):

  Same data:                        GMM result:
     · · ·                           1 1 1
    · · · · ·                       1 1 1 1 1
   · · · · · · ·                   1 1 1 1 1 1 1
    · · · · ·                       1 1 1 1 1
     · · ·                           1 1 1
         · · · · ·                       2 2 2 2 2
        · · · · · · ·                   2 2 2 2 2 2 2
         · · · · ·                       2 2 2 2 2

  GMM fits ellipses that match     Correctly separates the
  the natural cluster shape!        elongated clusters.
```

---

## 6.4 Soft vs. Hard Assignment — When It Matters

```
Point in the overlap zone between two clusters:

  K-Means:
    distance to μ₁ = 3.01
    distance to μ₂ = 2.99
    → Label: Cluster 2 (100% confident!)
    
    But this is practically a coin flip!
    We lose ALL uncertainty information.

  GMM:
    P(k=1|x) = 0.48
    P(k=2|x) = 0.52
    → Soft: [0.48, 0.52]
    → Hard label: Cluster 2, BUT with only 52% confidence
    
    We KEEP the uncertainty information.
    We can flag this point for further review.
```

---

## 6.5 Computational Cost Comparison

```
Per iteration cost:

  K-Means:
    Distance computation: O(N × K × d)
    Centroid update:       O(N × d)
    Total per iteration:   O(NKd)

  GMM (full covariance):
    E-step (responsibilities):  O(N × K × d²)  [matrix operations]
    M-step (mean update):       O(N × K × d)
    M-step (covariance update): O(N × K × d²)
    Total per iteration:        O(NKd²)

  Ratio: GMM / K-Means ≈ d (scales with dimensionality!)

Practical comparison (N=10000, K=5):
  ┌──────────┬─────────────┬─────────────┬──────────────┐
  │ d        │ K-Means     │ GMM (full)  │ GMM/K-Means  │
  ├──────────┼─────────────┼─────────────┼──────────────┤
  │ 2        │ 100K ops    │ 200K ops    │ 2×           │
  │ 10       │ 500K ops    │ 5M ops      │ 10×          │
  │ 100      │ 5M ops      │ 500M ops    │ 100×         │
  │ 1000     │ 50M ops     │ 50B ops     │ 1000×        │
  └──────────┴─────────────┴─────────────┴──────────────┘

  For high dimensions, GMM is MUCH slower!
  → Use diagonal or spherical covariance
  → Or reduce dimensions first (PCA)
```

---

## 6.6 Python: Side-by-Side Comparison

```python
import numpy as np
import time
from sklearn.cluster import KMeans
from sklearn.mixture import GaussianMixture
from sklearn.datasets import make_blobs, make_moons
from sklearn.metrics import adjusted_rand_score, silhouette_score
from sklearn.preprocessing import StandardScaler

# ═══════════════════════════════════════════════
# Test 1: Well-separated spherical blobs
# ═══════════════════════════════════════════════
print("=" * 60)
print("TEST 1: Well-separated spherical blobs")
print("=" * 60)

X1, y1 = make_blobs(500, centers=3, cluster_std=0.8, random_state=42)

t0 = time.time()
km1 = KMeans(3, random_state=42, n_init=10).fit(X1)
t_km = time.time() - t0

t0 = time.time()
gmm1 = GaussianMixture(3, random_state=42, n_init=3).fit(X1)
t_gmm = time.time() - t0

print(f"  K-Means: ARI={adjusted_rand_score(y1, km1.labels_):.3f}, "
      f"time={t_km*1000:.1f}ms")
print(f"  GMM:     ARI={adjusted_rand_score(y1, gmm1.predict(X1)):.3f}, "
      f"time={t_gmm*1000:.1f}ms")
print(f"  → Both work well on spherical clusters!")

# ═══════════════════════════════════════════════
# Test 2: Elongated/anisotropic clusters
# ═══════════════════════════════════════════════
print(f"\n{'=' * 60}")
print("TEST 2: Elongated clusters")
print("=" * 60)

np.random.seed(42)
X2a = np.random.multivariate_normal([0,0], [[5,4],[4,5]], 200)
X2b = np.random.multivariate_normal([8,0], [[5,-4],[-4,5]], 200)
X2 = np.vstack([X2a, X2b])
y2 = np.array([0]*200 + [1]*200)

km2 = KMeans(2, random_state=42, n_init=10).fit(X2)
gmm2 = GaussianMixture(2, covariance_type='full', random_state=42).fit(X2)

print(f"  K-Means: ARI={adjusted_rand_score(y2, km2.labels_):.3f}")
print(f"  GMM:     ARI={adjusted_rand_score(y2, gmm2.predict(X2)):.3f}")
print(f"  → GMM handles elongated clusters better!")

# ═══════════════════════════════════════════════
# Test 3: Unequal cluster sizes
# ═══════════════════════════════════════════════
print(f"\n{'=' * 60}")
print("TEST 3: Unequal cluster sizes (300 vs 50 points)")
print("=" * 60)

np.random.seed(42)
X3a = np.random.normal(0, 1, (300, 2))
X3b = np.random.normal(5, 0.5, (50, 2))
X3 = np.vstack([X3a, X3b])
y3 = np.array([0]*300 + [1]*50)

km3 = KMeans(2, random_state=42, n_init=10).fit(X3)
gmm3 = GaussianMixture(2, random_state=42).fit(X3)

print(f"  K-Means: ARI={adjusted_rand_score(y3, km3.labels_):.3f}")
print(f"  GMM:     ARI={adjusted_rand_score(y3, gmm3.predict(X3)):.3f}")
print(f"  GMM weights: {gmm3.weights_.round(3)}")
print(f"  → GMM naturally handles unequal cluster sizes via πₖ!")

# ═══════════════════════════════════════════════
# Uncertainty analysis (GMM advantage)
# ═══════════════════════════════════════════════
print(f"\n{'=' * 60}")
print("UNCERTAINTY ANALYSIS (GMM exclusive)")
print("=" * 60)

probs = gmm1.predict_proba(X1)
confidence = probs.max(axis=1)

print(f"  Mean confidence:  {confidence.mean():.3f}")
print(f"  Min confidence:   {confidence.min():.3f}")
print(f"  Points < 0.7:     {(confidence < 0.7).sum()} ({(confidence < 0.7).mean()*100:.1f}%)")
print(f"  Points > 0.95:    {(confidence > 0.95).sum()} ({(confidence > 0.95).mean()*100:.1f}%)")
print(f"  → K-Means cannot provide this uncertainty information!")
```

---

## 6.7 Decision Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  CHOOSE K-MEANS when:                                              │
│  ─────────────────────                                              │
│  • Clusters are roughly spherical and similar size                  │
│  • Speed is paramount (very large datasets, >100K points)           │
│  • You need interpretable centroids                                 │
│  • High-dimensional data (d > 50)                                   │
│  • Simple baseline is sufficient                                    │
│  • MiniBatch/online clustering needed                               │
│                                                                     │
│  CHOOSE GMM when:                                                   │
│  ───────────────                                                    │
│  • Clusters have different shapes, sizes, or orientations           │
│  • You need soft/probabilistic assignments                          │
│  • Uncertainty quantification is important                          │
│  • You need a density model (p(x))                                  │
│  • Clusters have unequal sizes (different πₖ)                      │
│  • Anomaly detection via low-probability points                     │
│  • Data is low-to-moderate dimensional (d < 50)                     │
│                                                                     │
│  CHOOSE NEITHER when:                                               │
│  ────────────────────                                               │
│  • Clusters have arbitrary non-ellipsoidal shapes → DBSCAN          │
│  • Don't know K and can't use BIC → HDBSCAN                        │
│  • Need hierarchical structure → Agglomerative clustering           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Quick Decision Flowchart

```
        Do you know the number of clusters K?
                    │
             YES    │     NO
          ┌─────────┴─────────┐
          │                   │
  Are clusters              Use DBSCAN/HDBSCAN
  spherical?                or BIC with GMM
     │
  YES│    NO
     │     │
     ▼     ▼
  ┌─────┐ ┌──────┐
  │Need │ │ GMM  │
  │soft?│ │      │
  └──┬──┘ └──────┘
  YES│ NO
     │  │
     ▼  ▼
  GMM  K-Means
```

---

## 6.8 Summary Table

| Aspect | K-Means | GMM |
|--------|---------|-----|
| **Model type** | Distance-based | Probabilistic (generative) |
| **Assignment** | Hard: argmin distance | Soft: P(k\|x) ∈ [0,1] |
| **Cluster shape** | Spherical (Voronoi cells) | Ellipsoidal (full Σₖ) |
| **Cluster weight** | Implicit (equal) | Explicit (πₖ) |
| **Algorithm** | Hard EM (assign + update) | Soft EM (E-step + M-step) |
| **Objective** | Minimize Σ\|\|x-μ\|\|² | Maximize Σ ln p(x) |
| **K selection** | Elbow, silhouette | BIC, AIC |
| **Speed** | Fast: O(NKd) | Slower: O(NKd²) |
| **Density estimation** | ❌ | ✅ |
| **Uncertainty** | ❌ | ✅ (entropy, confidence) |
| **Relationship** | Special case of GMM | Generalization of K-Means |

---

## 6.9 Revision Questions

1. **Prove that K-Means is a special case of GMM.** What three constraints turn GMM into K-Means? Show what happens to the responsibilities under these constraints.

2. **Why is GMM slower than K-Means?** Derive the per-iteration complexity of both algorithms and explain when the difference matters most.

3. **Give a dataset example** where K-Means performs comparably to GMM, and another where GMM significantly outperforms K-Means. What properties of the data determine the outcome?

4. **You have 10,000 data points in 100 dimensions** with 5 clusters. Would you choose K-Means or GMM? Justify with computational and statistical arguments.

5. **How does the 'covariance_type' parameter** in sklearn's GMM relate to the K-Means ↔ GMM spectrum? Where does 'spherical' GMM sit relative to K-Means?

6. **A colleague says "GMM is always better than K-Means because it's more general."** Do you agree? Discuss the bias-variance trade-off and practical considerations.

---

[← Previous: Model Selection (BIC/AIC)](./05-model-selection-bic-aic.md) | [Back to GMM Unit →](./README.md)
