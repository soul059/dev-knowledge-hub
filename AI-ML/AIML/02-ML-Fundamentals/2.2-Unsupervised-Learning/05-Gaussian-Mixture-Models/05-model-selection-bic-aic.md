# 5. Model Selection: BIC and AIC

[← Previous: Soft Clustering](./04-soft-clustering.md) | [Next: Comparison with K-Means →](./06-comparison-with-kmeans.md)

---

## Chapter Overview

A critical question in GMM is: **how many components K should we use?** Too few components underfit (miss structure), too many overfit (model noise). This chapter covers the two primary information criteria — **BIC (Bayesian Information Criterion)** and **AIC (Akaike Information Criterion)** — for automated model selection, along with covariance type selection and practical sklearn workflows.

---

## 5.1 The Model Selection Problem

```
K=1: Underfitting            K=3: Just right            K=10: Overfitting
┌──────────────────┐         ┌──────────────────┐       ┌──────────────────┐
│                  │         │                  │       │                  │
│  ╭────────────╮  │         │ ╭───╮  ╭───╮     │       │ ╭ ╭╮ ╭╮╭╮      │
│ ╱              ╲ │         │╱   ╲╱   ╲╱  ╲    │       │╱╲╱ ╲╱ ╲╱ ╲╱╲╱╲ │
│╱                ╲│         │                  │       │                  │
│                  │         │                  │       │                  │
└──────────────────┘         └──────────────────┘       └──────────────────┘
Misses cluster structure     Captures true structure    Models noise as clusters
Low log-likelihood           Good log-likelihood        Highest log-likelihood
                                                         (but overfitting!)

Problem: Log-likelihood ALWAYS increases with more components.
         We need a penalty for model complexity!
```

---

## 5.2 Number of Parameters in a GMM

Before understanding BIC/AIC, we need to count the **free parameters** in a GMM:

```
For K components, d-dimensional data:

  Parameters per component:
    μₖ:  d parameters           (mean vector)
    Σₖ:  depends on type        (covariance matrix)
    πₖ:  1 parameter            (weight, minus 1 for constraint)

  Covariance parameters per component:
    ┌──────────────┬──────────────────┬───────────────────────┐
    │ Type         │ Parameters       │ Example (d=5)         │
    ├──────────────┼──────────────────┼───────────────────────┤
    │ spherical    │ 1                │ 1                     │
    │ diagonal     │ d                │ 5                     │
    │ full         │ d(d+1)/2         │ 15                    │
    │ tied         │ d(d+1)/2 (shared)│ 15 (total, not per K) │
    └──────────────┴──────────────────┴───────────────────────┘
```

### Total Parameter Count

```
Full covariance:
  p = K × [d + d(d+1)/2] + (K-1)
    = K × d + K × d(d+1)/2 + K - 1

  Example: K=3, d=2
  p = 3×2 + 3×2×3/2 + 3-1 = 6 + 9 + 2 = 17

Diagonal covariance:
  p = K × [d + d] + (K-1)
    = K × 2d + K - 1

  Example: K=3, d=2
  p = 3×4 + 2 = 14

Spherical covariance:
  p = K × [d + 1] + (K-1)
  
  Example: K=3, d=2
  p = 3×3 + 2 = 11

Tied (full, shared across components):
  p = K × d + d(d+1)/2 + (K-1)

  Example: K=3, d=2
  p = 6 + 3 + 2 = 11
```

---

## 5.3 BIC — Bayesian Information Criterion

### Formula

```
BIC = -2 · ln(L̂) + p · ln(N)

where:
  ln(L̂) = maximized log-likelihood = Σₙ ln[Σₖ πₖ N(xₙ|μₖ,Σₖ)]
  p      = number of free parameters in the model
  N      = number of data points
  
LOWER BIC = BETTER MODEL

  BIC = -2 · (fit quality) + (complexity penalty)
         ↑                    ↑
    want this HIGH        want this LOW
    (good fit)            (simple model)
```

### Intuition

```
BIC balances fit vs. complexity:

  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  -2·ln(L̂):   Decreases as K increases (better fit)     │
  │               → Favors MORE components                   │
  │                                                          │
  │  p·ln(N):     Increases as K increases (more params)     │
  │               → Penalizes MORE components                │
  │                                                          │
  │  BIC = sum of both → has a MINIMUM at optimal K          │
  │                                                          │
  └──────────────────────────────────────────────────────────┘

  BIC
   │╲
   │  ╲         ╱
   │   ╲       ╱
   │    ╲     ╱
   │     ╲   ╱
   │      ╰─╯     ← minimum = optimal K
   │
   └──┬──┬──┬──┬──┬──┬──── K
      1  2  3  4  5  6
```

### BIC Properties

- **Consistent**: As N→∞, BIC selects the true model
- **Penalizes complexity more** than AIC (penalty grows with ln(N))
- **Tends to select simpler models** than AIC
- **Preferred for model identification** (choosing K)

---

## 5.4 AIC — Akaike Information Criterion

### Formula

```
AIC = -2 · ln(L̂) + 2p

where:
  ln(L̂) = maximized log-likelihood
  p      = number of free parameters
  
LOWER AIC = BETTER MODEL

Compared to BIC:
  BIC penalty: p · ln(N)    → depends on sample size
  AIC penalty: 2p           → fixed, independent of N
  
  For N > 7: ln(N) > 2, so BIC penalizes more than AIC
  → BIC tends to select fewer components than AIC
```

### AIC vs. BIC Comparison

```
           AIC penalty    BIC penalty
  N=10:    2p             2.30·p         (BIC slightly more)
  N=100:   2p             4.61·p         (BIC 2× more)
  N=1000:  2p             6.91·p         (BIC 3× more)
  N=10000: 2p             9.21·p         (BIC 4.6× more)

  As N grows, BIC's penalty grows → selects simpler models.
```

---

## 5.5 Worked Example: Choosing K

```
Dataset: 300 points, d=2 dimensions, true K=3

Computing BIC and AIC for K=1 to K=6 (full covariance):

Parameters per K:
  K=1: p = 1×2 + 1×3 + 0 = 5
  K=2: p = 2×2 + 2×3 + 1 = 11
  K=3: p = 3×2 + 3×3 + 2 = 17
  K=4: p = 4×2 + 4×3 + 3 = 23
  K=5: p = 5×2 + 5×3 + 4 = 29
  K=6: p = 6×2 + 6×3 + 5 = 35

Results:
┌────┬──────────┬──────────┬──────────┬──────────┬─────────┐
│ K  │ ln(L̂)   │ -2ln(L̂) │ BIC      │ AIC      │ Best?   │
├────┼──────────┼──────────┼──────────┼──────────┼─────────┤
│  1 │  -1250   │  2500    │ 2528.5   │ 2510     │         │
│  2 │   -950   │  1900    │ 1962.7   │  1922    │         │
│  3 │   -820   │  1640    │  1736.9  │  1674    │ ✓ BIC   │
│  4 │   -810   │  1620    │  1751.2  │  1666    │ ✓ AIC   │
│  5 │   -805   │  1610    │  1775.3  │  1668    │         │
│  6 │   -802   │  1604    │  1803.5  │  1674    │         │
└────┴──────────┴──────────┴──────────┴──────────┴─────────┘

BIC minimum at K=3 ← correct!
AIC minimum at K=4 ← slight overfit

In this case, BIC correctly identifies K=3.
```

---

## 5.6 Covariance Type Selection

### The Four Covariance Types in sklearn

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│  1. 'spherical' — Each component has one variance (σ²ₖ)          │
│     Σₖ = σ²ₖ · I                                                 │
│     Shape: ○ circles (all axes equal)                             │
│     Params: 1 per component                                       │
│     Like K-Means                                                   │
│                                                                    │
│  2. 'diagonal' — Each component has d variances                   │
│     Σₖ = diag(σ²ₖ₁, σ²ₖ₂, ..., σ²ₖd)                          │
│     Shape: ▬ axis-aligned ellipses                                │
│     Params: d per component                                       │
│     No rotation                                                    │
│                                                                    │
│  3. 'full' — Each component has full d×d covariance              │
│     Σₖ = any positive definite matrix                             │
│     Shape: ⬮ arbitrary ellipses (rotated)                        │
│     Params: d(d+1)/2 per component                                │
│     Most flexible, most parameters                                │
│                                                                    │
│  4. 'tied' — All components share ONE covariance                  │
│     Σ₁ = Σ₂ = ... = Σₖ = Σ                                      │
│     Shape: Same shape, different locations                         │
│     Params: d(d+1)/2 total                                        │
│     Less flexible, fewer parameters                               │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
  spherical          diagonal           full               tied
  ┌─────────┐        ┌─────────┐       ┌─────────┐       ┌─────────┐
  │  ○   ○  │        │  ▭   ▯  │       │  ⬮  ◇  │       │  ⬮   ⬮  │
  │         │        │         │       │         │       │         │
  │    ○    │        │    ▭    │       │    ⬮    │       │    ⬮    │
  └─────────┘        └─────────┘       └─────────┘       └─────────┘
  Same size circles  Axis-aligned     Free rotation     Same shape,
  per component      ellipses          per component     all components
```

---

## 5.7 Complete Model Selection with sklearn

```python
import numpy as np
from sklearn.mixture import GaussianMixture
from sklearn.datasets import make_blobs
from sklearn.preprocessing import StandardScaler

# Generate data
X, y_true = make_blobs(n_samples=500, centers=4, cluster_std=0.8, random_state=42)
X = StandardScaler().fit_transform(X)

# Search over K and covariance types
K_range = range(1, 9)
cov_types = ['spherical', 'diagonal', 'full', 'tied']

results = []
best_bic = np.inf
best_model = None

print(f"{'K':>3} | {'Cov Type':>12} | {'BIC':>10} | {'AIC':>10} | {'Params':>6} | {'LL':>10}")
print(f"{'-'*3}-+-{'-'*12}-+-{'-'*10}-+-{'-'*10}-+-{'-'*6}-+-{'-'*10}")

for cov_type in cov_types:
    for K in K_range:
        gmm = GaussianMixture(
            n_components=K,
            covariance_type=cov_type,
            n_init=3,
            random_state=42
        )
        gmm.fit(X)
        
        bic = gmm.bic(X)
        aic = gmm.aic(X)
        ll = gmm.score(X) * len(X)  # Total log-likelihood
        
        # Count parameters
        d = X.shape[1]
        if cov_type == 'spherical':
            n_cov_params = K
        elif cov_type == 'diagonal':
            n_cov_params = K * d
        elif cov_type == 'full':
            n_cov_params = K * d * (d + 1) // 2
        elif cov_type == 'tied':
            n_cov_params = d * (d + 1) // 2
        n_params = K * d + n_cov_params + K - 1
        
        marker = ""
        if bic < best_bic:
            best_bic = bic
            best_model = (K, cov_type)
            marker = " ← best"
        
        results.append((K, cov_type, bic, aic, n_params, ll))
        
        if K in [1, 3, 4, 5, 8]:
            print(f"{K:>3} | {cov_type:>12} | {bic:>10.1f} | {aic:>10.1f} | {n_params:>6} | {ll:>10.1f}{marker}")

print(f"\n{'='*70}")
print(f"BEST MODEL: K={best_model[0]}, covariance='{best_model[1]}', BIC={best_bic:.1f}")

# Fit and use the best model
best_gmm = GaussianMixture(
    n_components=best_model[0],
    covariance_type=best_model[1],
    n_init=5,
    random_state=42
)
best_gmm.fit(X)
labels = best_gmm.predict(X)
print(f"Cluster sizes: {np.bincount(labels)}")
```

### Plotting BIC/AIC Curves

```python
# Collect results for plotting
K_range = range(1, 9)

bic_scores = []
aic_scores = []
for K in K_range:
    gmm = GaussianMixture(n_components=K, covariance_type='full',
                          n_init=3, random_state=42).fit(X)
    bic_scores.append(gmm.bic(X))
    aic_scores.append(gmm.aic(X))

print("\n=== BIC/AIC vs K (full covariance) ===")
print(f"{'K':>3} | {'BIC':>10} | {'AIC':>10} | BIC plot")
print(f"{'-'*3}-+-{'-'*10}-+-{'-'*10}-+-{'-'*30}")

min_bic = min(bic_scores)
max_bic = max(bic_scores)
for K, bic, aic in zip(K_range, bic_scores, aic_scores):
    bar_len = int((bic - min_bic) / (max_bic - min_bic + 1) * 25)
    marker = " ← min" if bic == min_bic else ""
    print(f"{K:>3} | {bic:>10.1f} | {aic:>10.1f} | {'█' * bar_len}{marker}")

# For matplotlib:
# import matplotlib.pyplot as plt
# fig, ax = plt.subplots(1, 1, figsize=(8, 5))
# ax.plot(K_range, bic_scores, 'bo-', label='BIC')
# ax.plot(K_range, aic_scores, 'rs-', label='AIC')
# ax.set_xlabel('Number of Components K')
# ax.set_ylabel('Information Criterion')
# ax.legend()
# ax.set_title('Model Selection: BIC and AIC')
# plt.show()
```

---

## 5.8 BIC vs. AIC: When to Use Which

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Use BIC when:                                               │
│  ─────────────                                               │
│  • Goal is MODEL IDENTIFICATION (finding true K)             │
│  • You want a parsimonious (simple) model                    │
│  • Dataset is large (N > 100)                                │
│  • You believe there is a "true" number of components        │
│                                                              │
│  Use AIC when:                                               │
│  ─────────────                                               │
│  • Goal is PREDICTION (best density estimate)                │
│  • You want the best approximation to the true distribution  │
│  • Dataset is small                                          │
│  • You don't believe in a "true" K (all models are wrong)    │
│                                                              │
│  In practice:                                                │
│  ─────────────                                               │
│  • BIC is more commonly used for GMM component selection     │
│  • If BIC and AIC disagree, prefer BIC for clustering        │
│  • Report both and note any disagreement                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.9 Summary Table

| Criterion | Formula | Penalty | Tends to Select | Best For |
|-----------|---------|---------|-----------------|----------|
| **BIC** | -2·ln(L̂) + p·ln(N) | p·ln(N) — heavier | Fewer components | Model identification |
| **AIC** | -2·ln(L̂) + 2p | 2p — lighter | More components | Prediction |
| **Log-likelihood** | Σₙ ln p(xₙ) | None | Most components (overfit!) | Not for selection |

### Covariance Type Summary

| Type | Shape | Params per K | Total Params | Flexibility |
|------|-------|-------------|-------------|-------------|
| **spherical** | Circles | 1 | K(d+1)+K-1 | Lowest |
| **diagonal** | Axis-aligned ellipses | d | K(2d)+K-1 | Medium |
| **tied** | Same ellipse everywhere | d(d+1)/2 shared | Kd+d(d+1)/2+K-1 | Medium |
| **full** | Any ellipse | d(d+1)/2 | K[d+d(d+1)/2]+K-1 | Highest |

---

## 5.10 Revision Questions

1. **Why can't we simply use log-likelihood** to choose the number of components K? What problem arises as K increases?

2. **Write the BIC and AIC formulas.** How do their penalties differ, and what effect does this have on model selection?

3. **For a GMM with K=4 components, d=3 dimensions, and full covariance**, how many free parameters does the model have? Show your calculation.

4. **Given BIC scores** for K=1,2,3,4,5 of [1500, 1200, 1050, 1080, 1150], what is the optimal K? What if AIC scores for the same K values are [1450, 1150, 980, 960, 970]?

5. **Describe the four covariance types** in sklearn's GaussianMixture. When would you choose 'diagonal' over 'full'? When would 'tied' be appropriate?

6. **How would you perform a complete model selection** searching over both K and covariance type? Write the procedure and explain what you would report.

---

[← Previous: Soft Clustering](./04-soft-clustering.md) | [Next: Comparison with K-Means →](./06-comparison-with-kmeans.md)
