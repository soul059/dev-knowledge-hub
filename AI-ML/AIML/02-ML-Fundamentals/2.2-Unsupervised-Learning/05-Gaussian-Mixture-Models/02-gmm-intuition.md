# 2. GMM Intuition

[← Previous: Mixture Models Concept](./01-mixture-models-concept.md) | [Next: Expectation-Maximization →](./03-expectation-maximization.md)

---

## Chapter Overview

This chapter builds deep geometric and probabilistic intuition for Gaussian Mixture Models. We'll see how each cluster is modeled as a Gaussian "cloud" with its own center (μ), shape (Σ), and size (π), how the overall probability density is a superposition of these clouds, and how GMM provides **soft (probabilistic) assignments** rather than hard cluster labels. We contrast this with K-Means' rigid approach.

---

## 2.1 Each Cluster as a Gaussian Distribution

In a GMM, each cluster is not defined by a centroid and a radius — it's defined by a full **probability distribution**:

```
Cluster k is described by:
  μₖ = mean (center of the Gaussian)
  Σₖ = covariance matrix (shape and orientation)
  πₖ = weight (how prevalent this cluster is)
```

### ASCII Visualization: 2D GMM with 3 Components

```
  y
  │
  9│                    ╭──────────╮
   │                   ╱ · · · · · ╲   Component 2
  8│                  │ · · ★₂ · · │   μ₂=[6,8], π₂=0.3
   │                   ╲ · · · · · ╱   Σ₂: circular
  7│                    ╰──────────╯
   │
  6│
   │
  5│
   │         ╭────────────────╮
  4│        ╱  · · · · · · ·  ╲
   │       │ · · · · · · · · · │    Component 3
  3│       │ · · · ★₃ · · · · │    μ₃=[5,3], π₃=0.25
   │       │ · · · · · · · · · │    Σ₃: elongated horizontally
  2│        ╲  · · · · · · ·  ╱
   │         ╰────────────────╯
  1│  ╭───────╮
   │ ╱ · · · · ╲    Component 1
  0││ · ★₁ · · │    μ₁=[2,0], π₁=0.45
   │ ╲ · · · · ╱    Σ₁: tilted ellipse
  -1│ ╰───────╯
   │
   └──┬──┬──┬──┬──┬──┬──┬──┬──┬── x
      0  1  2  3  4  5  6  7  8
```

Each ellipse represents a **contour of constant probability density** (like a topographic map of a mountain of probability).

---

## 2.2 The Mean μₖ — Center of Each Component

```
The mean μₖ is the "center of gravity" of component k.

For 2D data:
  μₖ = [μₖ₁, μₖ₂]ᵀ

In a fitted GMM with 3 components:
  μ₁ = [2.1, 0.3]ᵀ   ← center of cluster 1
  μ₂ = [6.0, 7.8]ᵀ   ← center of cluster 2
  μ₃ = [5.2, 2.9]ᵀ   ← center of cluster 3

Compare with K-Means:
  K-Means centroids ≈ GMM means
  But GMM also captures shape and size!
```

---

## 2.3 The Covariance Matrix Σₖ — Shape of Each Component

The covariance matrix controls the **shape**, **size**, and **orientation** of each Gaussian ellipsoid.

```
2D Covariance Matrix:

  Σₖ = [ σ₁²    ρσ₁σ₂ ]
       [ ρσ₁σ₂   σ₂²  ]

  σ₁² = variance in x-direction (width²)
  σ₂² = variance in y-direction (height²)
  ρ   = correlation coefficient (-1 to 1)
  ρσ₁σ₂ = covariance (controls tilt)
```

### Different Covariance Shapes

```
  Spherical: Σ = σ²I          Diagonal: Σ = diag(σ₁²,σ₂²)
  ┌─────────────────┐         ┌─────────────────┐
  │      · · ·      │         │    · · · · ·    │
  │    · · · · ·    │         │   · · · · · ·   │
  │   · · · · · ·   │         │   · · ★ · · ·   │
  │    · · · · ·    │         │   · · · · · ·   │
  │      · · ·      │         │    · · · · ·    │
  └─────────────────┘         └─────────────────┘
    Circle: equal spread       Axis-aligned ellipse
    [σ² 0 ]                    [σ₁² 0  ]
    [0  σ²]                    [0   σ₂²]

  Full (positive ρ):           Full (negative ρ):
  ┌─────────────────┐         ┌─────────────────┐
  │            · ·  │         │  · ·            │
  │         · · · · │         │ · · · ·         │
  │      · · ★ · ·  │         │  · · ★ · ·      │
  │   · · · ·       │         │      · · · ·    │
  │  · ·            │         │         · ·     │
  └─────────────────┘         └─────────────────┘
    Tilted ╱ (positive corr)   Tilted ╲ (negative corr)
    [σ₁²    ρσ₁σ₂]            [σ₁²   -ρσ₁σ₂]
    [ρσ₁σ₂   σ₂² ]            [-ρσ₁σ₂   σ₂² ]
```

---

## 2.4 The Mixture Weight πₖ — Size/Prevalence

```
πₖ controls the "volume" or "weight" of component k.

Visual: Different weights change relative sizes

  π₁ = 0.6                    π₁ = 0.2
  ┌──────────────┐            ┌──────────────┐
  │              │            │              │
  │   ████████   │            │   ████       │
  │  ██████████  │            │              │
  │   ████████   │            │              │
  │              │            │              │
  │   ████       │            │   ████████   │
  │              │            │  ██████████  │
  │              │            │   ████████   │
  └──────────────┘            └──────────────┘
  π₂ = 0.4                    π₂ = 0.8

  Higher πₖ → more points expected from component k
  → higher probability of assignment to component k
```

---

## 2.5 The GMM Probability Density

The overall probability density is a weighted sum of Gaussians:

```
p(x) = Σ_{k=1}^{K} πₖ · N(x | μₖ, Σₖ)

     = π₁ · N(x|μ₁,Σ₁) + π₂ · N(x|μ₂,Σ₂) + ... + πₖ · N(x|μₖ,Σₖ)
```

### Worked Example (1D, 2 components)

```
Parameters:
  π₁ = 0.6,  μ₁ = 0,  σ₁ = 1
  π₂ = 0.4,  μ₂ = 4,  σ₂ = 1.5

GMM density at x = 2:

  N(2|0,1) = (1/√(2π)) · exp(-(2-0)²/(2·1²))
            = 0.3989 · exp(-2)
            = 0.3989 · 0.1353
            = 0.0540

  N(2|4,1.5²) = (1/√(2π·2.25)) · exp(-(2-4)²/(2·2.25))
              = (1/√(14.14)) · exp(-0.889)
              = 0.2660 · 0.4111
              = 0.1093

  p(x=2) = 0.6 · 0.0540 + 0.4 · 0.1093
          = 0.0324 + 0.0437
          = 0.0761

  ┌────────────────────────────────────────────────────────┐
  │  p(2) = 0.0761                                       │
  │                                                       │
  │  Contribution from component 1: 0.0324 (42.6%)       │
  │  Contribution from component 2: 0.0437 (57.4%)       │
  │                                                       │
  │  → Point x=2 is more likely from component 2!        │
  │     (even though component 1 has higher weight π)     │
  │     (because x=2 is closer to μ₂=4 than to μ₁=0)   │
  └────────────────────────────────────────────────────────┘
```

### Superposition Visualization

```
  p(x)
   │
   │  π₁·N(x|μ₁,σ₁)
   │     ╱╲
   │    ╱  ╲                π₂·N(x|μ₂,σ₂)
   │   ╱    ╲                  ╱╲
   │  ╱      ╲               ╱  ╲
   │ ╱        ╲             ╱    ╲
   │╱          ╲           ╱      ╲
   │            ╲         ╱        ╲
   │             ╲       ╱          ╲
   │              ╲     ╱            ╲
   │               ╲   ╱              ╲
   │                ╲ ╱                ╲
   │                 ╳                  ╲
   └────────┬────────┬────────┬────────┬───── x
           μ₁=0     2       μ₂=4      6

  The GMM density p(x) is the SUM of these two curves.
  At x=2, both components contribute, creating soft assignment.
```

---

## 2.6 Fitting Ellipsoidal Clusters

Unlike K-Means (which fits circles/spheres), GMM fits **ellipsoids** that can:

```
1. Have different sizes (controlled by eigenvalues of Σ)
2. Have different orientations (controlled by eigenvectors of Σ)
3. Have different shapes (elongated vs. circular)

K-Means:                    GMM:
  ┌───────────────┐          ┌───────────────┐
  │    ○          │          │    ◎          │
  │       ○       │          │       ⬮       │
  │  ○            │          │  ⬮            │
  │               │          │               │
  └───────────────┘          └───────────────┘
  All circles                Different ellipses
  (equal variance,           (different Σₖ for
   all directions)            each component)
```

### Contour Plot Interpretation

```
GMM contours = constant density curves

  Outer contour: low density (far from mean)
  Inner contour: high density (near mean)

       ╭─────────────────────╮
      ╱  ╭──────────────╮    ╲
     ╱  ╱  ╭─────────╮  ╲    ╲
    │  │  │     ★     │  │    │     Three nested ellipses
    │  │  │  (1σ)     │  │    │     at 1σ, 2σ, 3σ distances
     ╲  ╲  ╰─────────╯  ╱    ╱     from the mean
      ╲  ╰──────────────╯    ╱
       ╰─────(2σ)───────────╯
              (3σ)

  68% of data within 1σ ellipse
  95% of data within 2σ ellipse
  99.7% of data within 3σ ellipse
```

---

## 2.7 Hard vs. Soft Clustering

### K-Means: Hard Assignment

```
Each point gets ONE label. No uncertainty.

  Point x₁ → Cluster 2     (100% certain)
  Point x₂ → Cluster 1     (100% certain)
  Point x₃ → Cluster 3     (100% certain)

  What about a point equidistant from two centroids?
  → Still gets ONE label (arbitrary choice)
  → No indication of uncertainty!
```

### GMM: Soft (Probabilistic) Assignment

```
Each point gets a PROBABILITY for each cluster.

  Point x₁ → P(k=1|x₁) = 0.05,  P(k=2|x₁) = 0.90,  P(k=3|x₁) = 0.05
             "Strongly belongs to cluster 2"

  Point x₂ → P(k=1|x₂) = 0.85,  P(k=2|x₂) = 0.10,  P(k=3|x₂) = 0.05
             "Strongly belongs to cluster 1"

  Point x₃ → P(k=1|x₃) = 0.45,  P(k=2|x₃) = 0.40,  P(k=3|x₃) = 0.15
             "Uncertain! Almost equally between clusters 1 and 2"
```

### Visual Comparison

```
  HARD (K-Means):              SOFT (GMM):

    A A A│B B B               0.9A  0.6A  0.5A│0.5B  0.7B  0.9B
    A A A│B B B               0.9A  0.7A  0.6A│0.6B  0.8B  0.9B
    A A A│B B B               0.9A  0.8A  0.5A│0.5B  0.7B  0.9B
    ─────┼─────               ─────────────────┼─────────────────
    C C C│D D D               0.8C  0.6C  0.4C│0.4D  0.6D  0.9D
    C C C│D D D               0.9C  0.7C  0.5C│0.5D  0.7D  0.9D

  Sharp boundaries             Gradual transitions
  No uncertainty               Uncertainty quantified
  Deterministic (given K)      Probabilistic
```

### The Responsibility (Posterior Probability)

```
The "responsibility" of component k for point x:

  γ(zₖ) = P(z=k | x) = ───────πₖ · N(x|μₖ,Σₖ)───────
                          Σⱼ πⱼ · N(x|μⱼ,Σⱼ)

  This is Bayes' theorem applied to mixture models!

  γ(zₖ) tells us: "Given that we observed x, what's the
  probability that it came from component k?"
```

---

## 2.8 Python: Visualizing GMM Intuition

```python
import numpy as np
from sklearn.mixture import GaussianMixture
from sklearn.datasets import make_blobs

# Generate data with different cluster shapes
np.random.seed(42)
n1, n2, n3 = 200, 150, 100

# Component 1: compact circular
X1 = np.random.multivariate_normal([2, 2], [[0.5, 0], [0, 0.5]], n1)
# Component 2: elongated diagonal
X2 = np.random.multivariate_normal([7, 7], [[2, 1.5], [1.5, 2]], n2)
# Component 3: wide horizontal
X3 = np.random.multivariate_normal([5, 0], [[3, 0], [0, 0.3]], n3)

X = np.vstack([X1, X2, X3])
y_true = np.array([0]*n1 + [1]*n2 + [2]*n3)

# Fit GMM
gmm = GaussianMixture(n_components=3, covariance_type='full', random_state=42)
gmm.fit(X)

# Examine learned parameters
print("=== Learned GMM Parameters ===\n")
for k in range(3):
    print(f"Component {k+1}:")
    print(f"  Weight πₖ = {gmm.weights_[k]:.4f}")
    print(f"  Mean μₖ = {gmm.means_[k].round(3)}")
    print(f"  Covariance Σₖ =")
    print(f"    {gmm.covariances_[k].round(3)}")
    print()

# Soft assignments (responsibilities)
responsibilities = gmm.predict_proba(X)
print("=== Soft Assignments (first 5 points from each component) ===\n")
print(f"{'Point':>6} | {'P(k=1)':>8} | {'P(k=2)':>8} | {'P(k=3)':>8} | {'Hard':>4}")
print(f"{'-'*6}-+-{'-'*8}-+-{'-'*8}-+-{'-'*8}-+-{'-'*4}")

for idx in [0, 1, 2, 200, 201, 350, 351]:
    r = responsibilities[idx]
    hard = gmm.predict(X[idx:idx+1])[0]
    print(f"{idx:>6} | {r[0]:>8.4f} | {r[1]:>8.4f} | {r[2]:>8.4f} | {hard:>4}")

# Find uncertain points (high entropy in responsibilities)
from scipy.stats import entropy
entropies = np.array([entropy(r) for r in responsibilities])
uncertain_idx = np.argsort(entropies)[-5:]
print(f"\n=== Most Uncertain Points (highest entropy) ===")
for idx in uncertain_idx:
    r = responsibilities[idx]
    print(f"  Point {idx}: P = [{r[0]:.3f}, {r[1]:.3f}, {r[2]:.3f}], "
          f"entropy = {entropy(r):.3f}")
```

---

## 2.9 Summary Table

| Concept | Symbol | K-Means Equivalent | GMM Role |
|---------|--------|-------------------|----------|
| **Component mean** | μₖ | Centroid | Center of Gaussian |
| **Covariance** | Σₖ | — (implicit: spherical) | Shape, size, orientation |
| **Weight** | πₖ | — (implicit: equal) | Prior probability of component |
| **Assignment** | γ(zₖ) = P(z=k\|x) | argmin distance | Soft probability |
| **Density** | p(x) = Σ πₖ N(x\|μₖ,Σₖ) | — | Full probability model |
| **Cluster shape** | Determined by Σₖ | Always spherical | Ellipsoidal (any shape) |
| **Uncertainty** | Entropy of γ | None | Quantified per point |

---

## 2.10 Revision Questions

1. **Explain the three parameters** (μₖ, Σₖ, πₖ) of each GMM component. What aspect of the cluster does each control?

2. **Given a 1D GMM** with π₁=0.7, μ₁=2, σ₁=1 and π₂=0.3, μ₂=6, σ₂=2, compute the GMM density p(x=4). What is the responsibility of each component for this point?

3. **Draw (or describe in ASCII)** three different 2D Gaussian components: one spherical, one axis-aligned ellipse, and one tilted ellipse. What does the covariance matrix look like for each?

4. **What is the key difference** between hard clustering (K-Means) and soft clustering (GMM)? Give an example where soft clustering provides valuable information that hard clustering cannot.

5. **Why does a GMM fit ellipsoidal clusters** while K-Means fits only spherical ones? Relate this to the covariance matrix Σₖ.

6. **Explain the "responsibility" γ(zₖ)** using Bayes' theorem. What does it mean when γ(zₖ) ≈ 0.5 for two components?

---

[← Previous: Mixture Models Concept](./01-mixture-models-concept.md) | [Next: Expectation-Maximization →](./03-expectation-maximization.md)
