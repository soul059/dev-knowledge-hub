# 3. Expectation-Maximization (EM) Algorithm

[← Previous: GMM Intuition](./02-gmm-intuition.md) | [Next: Soft Clustering →](./04-soft-clustering.md)

---

## Chapter Overview

The **Expectation-Maximization (EM)** algorithm is the standard method for fitting Gaussian Mixture Models. It's an iterative algorithm that alternates between two steps: the **E-step** (compute responsibilities — "who belongs where?") and the **M-step** (update parameters — "what's the best model given assignments?"). This chapter provides the complete mathematical derivation, a worked numerical example, convergence analysis, and the deep connection between K-Means and EM.

---

## 3.1 The EM Algorithm Overview

### Why Can't We Just Use Maximum Likelihood Directly?

```
The GMM log-likelihood:

  ln p(X|π,μ,Σ) = Σₙ ln [ Σₖ πₖ · N(xₙ|μₖ,Σₖ) ]
                           ↑
                     Sum INSIDE the log!

Problem: The sum inside the log makes the derivative analytically
intractable. We can't take the derivative, set to zero, and solve
in closed form (unlike simple linear regression or single Gaussian).

Solution: EM algorithm — iterate between two steps that ARE tractable.
```

### EM Algorithm Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  INITIALIZE: Random μₖ, Σₖ, πₖ (or use K-Means for initialization) │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────────────────────────────────┐                       │
│  │  E-STEP: Compute Responsibilities        │                       │
│  │                                          │                       │
│  │  For each point xₙ, for each component k:│                      │
│  │                                          │                       │
│  │  γ(zₙₖ) = ────πₖ · N(xₙ|μₖ,Σₖ)────     │                      │
│  │            Σⱼ πⱼ · N(xₙ|μⱼ,Σⱼ)          │                      │
│  │                                          │                       │
│  │  "How responsible is component k for     │                       │
│  │   generating point xₙ?"                  │                       │
│  └──────────────┬───────────────────────────┘                       │
│                 │                                                    │
│                 ▼                                                    │
│  ┌──────────────────────────────────────────┐                       │
│  │  M-STEP: Update Parameters               │                       │
│  │                                          │                       │
│  │  Nₖ = Σₙ γ(zₙₖ)           (effective #) │                       │
│  │                                          │                       │
│  │  μₖ = (1/Nₖ) Σₙ γ(zₙₖ)·xₙ  (new mean) │                       │
│  │                                          │                       │
│  │  Σₖ = (1/Nₖ) Σₙ γ(zₙₖ)·(xₙ-μₖ)(xₙ-μₖ)ᵀ│                     │
│  │                            (new covariance)│                     │
│  │                                          │                       │
│  │  πₖ = Nₖ / N              (new weight)  │                       │
│  │                                          │                       │
│  └──────────────┬───────────────────────────┘                       │
│                 │                                                    │
│                 ▼                                                    │
│  ┌──────────────────────────────────────────┐                       │
│  │  CONVERGED?                               │                       │
│  │  Check: |log-likelihood change| < ε       │                       │
│  │  or: max iterations reached               │                       │
│  └──────┬──────────────────┬────────────────┘                       │
│    NO   │                  │  YES                                    │
│    ↑    │                  ▼                                         │
│    └────┘            RETURN μₖ, Σₖ, πₖ                             │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 3.2 E-Step: Computing Responsibilities

### The Responsibility Formula (Bayes' Theorem)

```
γ(zₙₖ) = P(zₙ = k | xₙ, θ)

         πₖ · N(xₙ | μₖ, Σₖ)
       = ─────────────────────────
         Σⱼ₌₁ᴷ πⱼ · N(xₙ | μⱼ, Σⱼ)

Components:
  Numerator:   πₖ · N(xₙ|μₖ,Σₖ) = prior × likelihood
  Denominator: Σⱼ πⱼ · N(xₙ|μⱼ,Σⱼ) = evidence (normalizer)
```

### Properties of Responsibilities

```
For each data point n:
  1.  γ(zₙₖ) ≥ 0           for all k      (non-negative)
  2.  Σₖ γ(zₙₖ) = 1        for all n      (sum to 1 per point)
  3.  γ(zₙₖ) ∈ [0, 1]      soft assignment (probability)

The responsibility matrix Γ has shape (N × K):

       Component 1  Component 2  Component 3
Point 1 [  0.95        0.03         0.02   ]
Point 2 [  0.10        0.85         0.05   ]
Point 3 [  0.02        0.08         0.90   ]
  ...
Point N [  0.45        0.40         0.15   ]
```

---

## 3.3 M-Step: Updating Parameters

### Effective Number of Points

```
Nₖ = Σₙ₌₁ᴺ γ(zₙₖ)

Nₖ is the "effective number of points" assigned to component k.
It's a soft count: each point contributes its responsibility weight.

Example:
  If 100 points have γ(z₁) = [0.9, 0.8, 0.7, ..., 0.1]
  N₁ = 0.9 + 0.8 + 0.7 + ... ≈ 45.3 (not an integer!)

Note: Σₖ Nₖ = N  (total effective counts = total points)
```

### Update Formulas

```
NEW MEAN (weighted average of all points):

  μₖ(new) = ──1── Σₙ γ(zₙₖ) · xₙ
             Nₖ

  Compare with K-Means: μₖ = (1/|Cₖ|) Σ_{n∈Cₖ} xₙ
  Same idea, but with soft weights instead of hard assignment!


NEW COVARIANCE (weighted scatter matrix):

  Σₖ(new) = ──1── Σₙ γ(zₙₖ) · (xₙ - μₖ)(xₙ - μₖ)ᵀ
             Nₖ

  Each point's contribution is weighted by its responsibility.


NEW WEIGHT (fraction of effective points):

  πₖ(new) = Nₖ / N

  The new weight = effective proportion of data in component k.
```

---

## 3.4 Log-Likelihood and Convergence

### Log-Likelihood

```
ℓ(θ) = ln p(X|θ) = Σₙ₌₁ᴺ ln [ Σₖ₌₁ᴷ πₖ · N(xₙ|μₖ,Σₖ) ]

The EM algorithm guarantees:

  ℓ(θ(t+1)) ≥ ℓ(θ(t))

The log-likelihood NEVER decreases. It monotonically increases
(or stays the same) at each iteration.
```

### Convergence Criteria

```
Stop when ANY of these is met:

  1. |ℓ(θ(t+1)) - ℓ(θ(t))| < ε        (log-likelihood change)
     Typical: ε = 1e-6

  2. max_k ||μₖ(t+1) - μₖ(t)|| < δ     (parameter change)
     Typical: δ = 1e-4

  3. iteration count > max_iter          (maximum iterations)
     Typical: max_iter = 100-300
```

### Convergence Visualization

```
Log-likelihood
     │                        ╭──────── converged
     │                    ╭───╯
     │                ╭───╯
     │            ╭───╯
     │         ╭──╯
     │       ╭─╯
     │     ╭─╯
     │   ╭─╯
     │  ╱
     │ ╱
     │╱
     └───┬───┬───┬───┬───┬───┬───┬───── Iteration
         1   5   10  15  20  25  30

  Fast initial progress, then diminishing returns.
  Guaranteed to converge to a LOCAL maximum (not global!).
```

### Local Maxima Problem

```
⚠️ EM converges to a LOCAL maximum of the log-likelihood,
   not necessarily the GLOBAL maximum!

   ℓ(θ)
     │       ╱╲
     │      ╱  ╲         ╱╲
     │     ╱    ╲       ╱  ╲        ╱╲
     │    ╱      ╲     ╱    ╲      ╱  ╲
     │   ╱        ╲   ╱      ╲    ╱    ╲
     │  ╱          ╲ ╱        ╲  ╱      ╲
     │ ╱            ╳          ╲╱        ╲
     └──────────────────────────────────── θ
           local    global    local
           max      max       max

   Solution: Run EM multiple times with different initializations,
   keep the result with the highest log-likelihood.
   (sklearn does this with n_init parameter)
```

---

## 3.5 Worked Numerical Example

### Setup

```
Data: 6 points in 1D
  x = [1, 2, 4, 5, 7, 8]

Model: K = 2 Gaussian components

Initial parameters:
  π₁ = 0.5,  μ₁ = 2,  σ₁² = 1
  π₂ = 0.5,  μ₂ = 6,  σ₂² = 1
```

### Iteration 1: E-Step

```
Compute responsibilities γ(zₙₖ) for each point:

For x₁ = 1:
  N(1|μ₁=2,σ₁²=1) = (1/√(2π)) · exp(-(1-2)²/2) = 0.2420
  N(1|μ₂=6,σ₂²=1) = (1/√(2π)) · exp(-(1-6)²/2) = 0.000001 ≈ 0

  γ(z₁₁) = (0.5 × 0.2420) / (0.5 × 0.2420 + 0.5 × 0.000001)
          = 0.1210 / 0.1210 ≈ 1.000
  γ(z₁₂) = 1 - 1.000 ≈ 0.000

For x₂ = 2:
  N(2|2,1) = 0.3989,  N(2|6,1) = 0.0001
  γ(z₂₁) ≈ 1.000,  γ(z₂₂) ≈ 0.000

For x₃ = 4:
  N(4|2,1) = 0.0540,  N(4|6,1) = 0.0540
  γ(z₃₁) = (0.5×0.0540)/(0.5×0.0540 + 0.5×0.0540) = 0.500
  γ(z₃₂) = 0.500

For x₄ = 5:
  N(5|2,1) = 0.0044,  N(5|6,1) = 0.2420
  γ(z₄₁) = (0.5×0.0044)/(0.5×0.0044 + 0.5×0.2420) = 0.018
  γ(z₄₂) = 0.982

For x₅ = 7:
  N(7|2,1) ≈ 0.0000,  N(7|6,1) = 0.2420
  γ(z₅₁) ≈ 0.000,  γ(z₅₂) ≈ 1.000

For x₆ = 8:
  N(8|2,1) ≈ 0.0000,  N(8|6,1) = 0.0540
  γ(z₆₁) ≈ 0.000,  γ(z₆₂) ≈ 1.000

Responsibility Summary:
┌────────┬──────────┬──────────┐
│ Point  │  γ(z=1)  │  γ(z=2)  │
├────────┼──────────┼──────────┤
│ x₁=1  │  1.000   │  0.000   │
│ x₂=2  │  1.000   │  0.000   │
│ x₃=4  │  0.500   │  0.500   │
│ x₄=5  │  0.018   │  0.982   │
│ x₅=7  │  0.000   │  1.000   │
│ x₆=8  │  0.000   │  1.000   │
└────────┴──────────┴──────────┘
```

### Iteration 1: M-Step

```
Effective counts:
  N₁ = 1.000 + 1.000 + 0.500 + 0.018 + 0.000 + 0.000 = 2.518
  N₂ = 0.000 + 0.000 + 0.500 + 0.982 + 1.000 + 1.000 = 3.482

New means:
  μ₁ = (1×1.000 + 2×1.000 + 4×0.500 + 5×0.018 + 7×0.000 + 8×0.000) / 2.518
     = (1.000 + 2.000 + 2.000 + 0.090) / 2.518
     = 5.090 / 2.518
     = 2.022

  μ₂ = (1×0.000 + 2×0.000 + 4×0.500 + 5×0.982 + 7×1.000 + 8×1.000) / 3.482
     = (0 + 0 + 2.000 + 4.910 + 7.000 + 8.000) / 3.482
     = 21.910 / 3.482
     = 6.293

New variances:
  σ₁² = [1.000×(1-2.022)² + 1.000×(2-2.022)² + 0.500×(4-2.022)² 
         + 0.018×(5-2.022)² + ...] / 2.518
      = [1.044 + 0.000 + 1.958 + 0.160] / 2.518
      = 3.162 / 2.518
      = 1.256

  σ₂² = [0.500×(4-6.293)² + 0.982×(5-6.293)² + 1.000×(7-6.293)² 
         + 1.000×(8-6.293)²] / 3.482
      = [2.633 + 1.641 + 0.500 + 2.912] / 3.482
      = 7.686 / 3.482
      = 2.208

New weights:
  π₁ = 2.518 / 6 = 0.420
  π₂ = 3.482 / 6 = 0.580

After Iteration 1:
  π₁=0.420, μ₁=2.022, σ₁²=1.256
  π₂=0.580, μ₂=6.293, σ₂²=2.208
```

### Convergence Progress

```
┌───────────┬────────┬────────┬────────┬────────┬────────┬────────┐
│ Iteration │   π₁   │   μ₁   │  σ₁²   │   π₂   │   μ₂   │  σ₂²  │
├───────────┼────────┼────────┼────────┼────────┼────────┼────────┤
│ Initial   │ 0.500  │ 2.000  │ 1.000  │ 0.500  │ 6.000  │ 1.000 │
│ Iter 1    │ 0.420  │ 2.022  │ 1.256  │ 0.580  │ 6.293  │ 2.208 │
│ Iter 2    │ 0.385  │ 1.850  │ 0.820  │ 0.615  │ 6.440  │ 2.350 │
│ ...       │  ...   │  ...   │  ...   │  ...   │  ...   │  ...  │
│ Converged │ 0.333  │ 1.500  │ 0.250  │ 0.667  │ 6.667  │ 1.556 │
└───────────┴────────┴────────┴────────┴────────┴────────┴────────┘

Natural grouping: {1,2} and {4,5,7,8}
  Cluster 1 mean ≈ 1.5 ✓
  Cluster 2 mean ≈ 6.0 ✓
```

---

## 3.6 Connection to K-Means: Hard EM

K-Means is a **special case** of EM where:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  K-Means = EM with:                                            │
│                                                                 │
│  1. All covariances are identical: Σₖ = σ²I for all k         │
│  2. All weights are equal: πₖ = 1/K for all k                 │
│  3. σ² → 0 (hard assignments)                                 │
│                                                                 │
│  In this limit:                                                 │
│                                                                 │
│  γ(zₙₖ) →  { 1  if k = argmin ||xₙ - μₖ||²                  │
│              { 0  otherwise                                     │
│                                                                 │
│  The soft responsibilities become hard 0/1 assignments!         │
│  EM becomes: "Assign to nearest centroid, update centroids"    │
│  = K-Means!                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Side-by-Side: K-Means Steps vs. EM Steps

```
K-Means                              EM (GMM)
────────                              ────────

ASSIGN step:                          E-step:
  For each xₙ:                        For each xₙ:
    cₙ = argminₖ ||xₙ-μₖ||²            γ(zₙₖ) = πₖN(xₙ|μₖ,Σₖ)/Σⱼ...
    (HARD: 0 or 1)                      (SOFT: continuous [0,1])

UPDATE step:                          M-step:
  μₖ = (1/|Cₖ|) Σ_{n∈Cₖ} xₙ           μₖ = (1/Nₖ) Σₙ γ(zₙₖ)·xₙ
  (equal-weight average)                (responsibility-weighted avg)
                                        + update Σₖ and πₖ

OBJECTIVE:                            OBJECTIVE:
  Minimize within-cluster              Maximize log-likelihood
  sum of squares                       ℓ(θ) = Σₙ ln Σₖ πₖN(xₙ|μₖ,Σₖ)
  Σₖ Σ_{n∈Cₖ} ||xₙ-μₖ||²
```

---

## 3.7 Initialization Strategies

```
Good initialization is CRITICAL for EM (avoids poor local maxima).

┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│  Strategy 1: K-Means initialization (RECOMMENDED)                 │
│  ─────────────────────────────────────────                        │
│  1. Run K-Means on the data                                      │
│  2. Use K-Means centroids as initial μₖ                          │
│  3. Use K-Means cluster covariances as initial Σₖ                │
│  4. Use K-Means cluster proportions as initial πₖ                │
│                                                                    │
│  Strategy 2: Random initialization                                │
│  ─────────────────────────────────                                │
│  1. Pick K random points as initial μₖ                           │
│  2. Set Σₖ = I (identity) or data covariance                    │
│  3. Set πₖ = 1/K                                                 │
│  4. Run multiple times (n_init > 1), keep best                   │
│                                                                    │
│  Strategy 3: K-Means++ (sklearn default for init='kmeans')        │
│  ──────────────────────────────────────────────────                │
│  Smart random initialization that spreads initial centers          │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 3.8 Python Implementation

### Using sklearn

```python
import numpy as np
from sklearn.mixture import GaussianMixture
from sklearn.datasets import make_blobs

# Generate data
X, y_true = make_blobs(n_samples=300, centers=3, cluster_std=1.0, random_state=42)

# Fit GMM with EM
gmm = GaussianMixture(
    n_components=3,
    covariance_type='full',    # 'full', 'tied', 'diag', 'spherical'
    max_iter=100,              # Maximum EM iterations
    n_init=5,                  # Number of random restarts
    init_params='kmeans',      # 'kmeans', 'k-means++', 'random'
    tol=1e-3,                  # Convergence threshold
    random_state=42
)
gmm.fit(X)

print("=== EM Convergence ===")
print(f"  Converged: {gmm.converged_}")
print(f"  Iterations: {gmm.n_iter_}")
print(f"  Log-likelihood: {gmm.lower_bound_:.4f}")

print("\n=== Learned Parameters ===")
for k in range(3):
    print(f"\nComponent {k+1}:")
    print(f"  πₖ = {gmm.weights_[k]:.4f}")
    print(f"  μₖ = {gmm.means_[k].round(3)}")
    print(f"  Σₖ diagonal = {np.diag(gmm.covariances_[k]).round(3)}")

# Predictions
labels = gmm.predict(X)              # Hard labels
probs = gmm.predict_proba(X)         # Soft (responsibilities)
scores = gmm.score_samples(X)        # Log-likelihood per point
```

### From-Scratch Implementation

```python
import numpy as np
from scipy.stats import multivariate_normal

def em_gmm(X, K, max_iter=100, tol=1e-6, random_state=42):
    """
    EM algorithm for Gaussian Mixture Model from scratch.
    """
    rng = np.random.RandomState(random_state)
    N, D = X.shape
    
    # Initialize with random points
    indices = rng.choice(N, K, replace=False)
    means = X[indices].copy()
    covariances = [np.eye(D) for _ in range(K)]
    weights = np.ones(K) / K
    
    log_likelihoods = []
    
    for iteration in range(max_iter):
        # ===== E-STEP =====
        # Compute responsibilities
        resp = np.zeros((N, K))
        for k in range(K):
            resp[:, k] = weights[k] * multivariate_normal.pdf(
                X, mean=means[k], cov=covariances[k]
            )
        
        # Normalize (each row sums to 1)
        resp_sum = resp.sum(axis=1, keepdims=True)
        resp /= resp_sum
        
        # Compute log-likelihood
        log_likelihood = np.sum(np.log(resp_sum.flatten()))
        log_likelihoods.append(log_likelihood)
        
        # Check convergence
        if iteration > 0 and abs(log_likelihoods[-1] - log_likelihoods[-2]) < tol:
            print(f"  Converged at iteration {iteration}")
            break
        
        # ===== M-STEP =====
        for k in range(K):
            Nk = resp[:, k].sum()  # Effective count
            
            # Update mean
            means[k] = (resp[:, k] @ X) / Nk
            
            # Update covariance
            diff = X - means[k]
            covariances[k] = (resp[:, k].reshape(-1, 1) * diff).T @ diff / Nk
            # Add small regularization for numerical stability
            covariances[k] += 1e-6 * np.eye(D)
            
            # Update weight
            weights[k] = Nk / N
    
    return means, covariances, weights, resp, log_likelihoods

# Test
from sklearn.datasets import make_blobs
X, y_true = make_blobs(300, centers=3, cluster_std=0.8, random_state=42)

means, covs, weights, resp, ll = em_gmm(X, K=3)

print(f"\nFinal log-likelihood: {ll[-1]:.2f}")
print(f"Iterations: {len(ll)}")
for k in range(3):
    print(f"\nComponent {k+1}: weight={weights[k]:.3f}, mean={means[k].round(2)}")
```

---

## 3.9 Summary Table

| Step | What it does | Formula | K-Means Analogue |
|------|-------------|---------|-----------------|
| **E-Step** | Compute responsibilities | γ(zₙₖ) = πₖN(xₙ\|μₖ,Σₖ) / Σⱼ... | Assign to nearest centroid |
| **M-Step: mean** | Update component centers | μₖ = (1/Nₖ) Σ γ(zₙₖ)xₙ | Recompute centroid |
| **M-Step: covariance** | Update component shapes | Σₖ = (1/Nₖ) Σ γ(zₙₖ)(xₙ-μₖ)(xₙ-μₖ)ᵀ | — (implicit: identity) |
| **M-Step: weight** | Update component proportions | πₖ = Nₖ/N | — (implicit: equal) |
| **Log-likelihood** | Monitor convergence | ℓ = Σₙ ln Σₖ πₖN(xₙ\|μₖ,Σₖ) | Inertia (sum of sq dist) |
| **Convergence** | Guaranteed monotonic | ℓ(t+1) ≥ ℓ(t) | Inertia always decreases |
| **Optimality** | Local maximum only | Multiple restarts needed | Local minimum only |

---

## 3.10 Revision Questions

1. **Describe the E-step and M-step** of the EM algorithm for GMMs. What is computed in each step, and why can't we solve the optimization in one step?

2. **Derive the responsibility formula** using Bayes' theorem. Identify the prior, likelihood, and evidence terms.

3. **Work through one EM iteration** for the following 1D data with K=2: x = {0, 1, 5, 6}. Initial parameters: π₁=π₂=0.5, μ₁=0, μ₂=5, σ₁²=σ₂²=1. Compute the E-step responsibilities and M-step parameter updates.

4. **Explain why EM is guaranteed to converge** (log-likelihood monotonically increases). Why is this not sufficient to guarantee finding the global optimum?

5. **Show how K-Means is a special case of EM.** What constraints on the GMM parameters turn EM into K-Means? What happens to the responsibilities in this limit?

6. **Why is initialization important for EM?** What can go wrong with poor initialization? Describe two initialization strategies and their trade-offs.

---

[← Previous: GMM Intuition](./02-gmm-intuition.md) | [Next: Soft Clustering →](./04-soft-clustering.md)
