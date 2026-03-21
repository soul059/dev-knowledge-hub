# Chapter 9: Maximum A Posteriori (MAP) Estimation

> **Unit 3: Probability Theory** | **Topic**: MAP, Prior, Posterior, Regularization, Conjugate Priors, Bayesian Inference

---

## 1. Overview

Maximum A Posteriori (MAP) estimation extends MLE by incorporating **prior beliefs** about parameters.
While MLE asks "what parameters maximize the likelihood of the data?", MAP asks "what parameters
maximize the posterior probability given the data AND our prior knowledge?" This seemingly small
change has profound implications — MAP naturally leads to **regularization**, preventing overfitting.

---

## 2. From MLE to MAP

### 2.1 MLE Recap

```
θ̂_MLE = argmax_θ  P(D|θ)              (maximize likelihood only)
```

### 2.2 MAP Definition

Using Bayes' theorem:

```
P(θ|D) = P(D|θ) · P(θ) / P(D)

Since P(D) is constant w.r.t. θ:

θ̂_MAP = argmax_θ  P(θ|D)
       = argmax_θ  P(D|θ) · P(θ)
       = argmax_θ  [ln P(D|θ) + ln P(θ)]
       = argmax_θ  [log-likelihood + log-prior]
```

### 2.3 Comparison

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   MLE:  θ̂ = argmax  ln P(D|θ)                           │
│                                                          │
│   MAP:  θ̂ = argmax  ln P(D|θ)  +  ln P(θ)              │
│                     ──────────    ────────               │
│                     log-likelihood  log-prior             │
│                                    (regularization!)     │
│                                                          │
│   When P(θ) = uniform (flat prior):  MAP = MLE          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## 3. MAP as Regularized MLE

The log-prior acts as a **regularization term** that penalizes certain parameter values.

### 3.1 Gaussian Prior → L2 Regularization (Ridge)

```
If θ ~ N(0, σ²_prior):

ln P(θ) = -θ²/(2σ²_prior) + const

MAP objective:
  θ̂_MAP = argmax_θ  [Σ ln P(xᵢ|θ) - θ²/(2σ²_prior)]
        = argmin_θ  [NLL + λ||θ||²]

where λ = 1/(2σ²_prior)

This is exactly L2 regularization (Ridge regression / weight decay)!
```

```
Intuition:

Log-likelihood        +  Gaussian prior    =   MAP objective
                                               (= Ridge regression)
     ╱╲                     ╱╲                      ╱╲
    ╱  ╲                   ╱  ╲                    ╱  ╲
   ╱    ╲        +        ╱    ╲          =       ╱  ╲
  ╱  θ_MLE╲             ╱  0   ╲                ╱θ_MAP╲
  ──────────             ──────────              ──────────
  (data fit)          (prefer small θ)       (compromise)

θ_MAP is pulled toward 0 relative to θ_MLE
```

### 3.2 Laplace Prior → L1 Regularization (Lasso)

```
If θ ~ Laplace(0, b):

P(θ) = (1/2b) exp(-|θ|/b)
ln P(θ) = -|θ|/b + const

MAP objective:
  θ̂_MAP = argmin_θ  [NLL + λ||θ||₁]

where λ = 1/b

This is exactly L1 regularization (Lasso)!
```

### 3.3 Summary: Prior ↔ Regularization

```
┌───────────────────────────────────────────────────┐
│  Prior Distribution    Regularization    Effect   │
│  ──────────────────    ──────────────    ──────   │
│  Uniform (flat)        None             = MLE     │
│  Gaussian N(0, σ²)     L2 (Ridge)       Shrinks  │
│  Laplace(0, b)         L1 (Lasso)       Sparsity │
│  Spike-and-slab        L0 (subset)      Selection│
│  Horseshoe             Adaptive         Sparse   │
└───────────────────────────────────────────────────┘
```

---

## 4. Worked Example 1: MAP for Bernoulli with Beta Prior

**Setup**: Estimate coin bias p. Prior: p ~ Beta(α, β). Data: k heads in n flips.

```
Step 1: Log-posterior
  ln P(p|data) ∝ ln P(data|p) + ln P(p)

  Log-likelihood:  k·ln(p) + (n-k)·ln(1-p)
  Log-prior:       (α-1)·ln(p) + (β-1)·ln(1-p)

  Log-posterior:   (k+α-1)·ln(p) + (n-k+β-1)·ln(1-p)

Step 2: Differentiate and set to zero
  d/dp: (k+α-1)/p - (n-k+β-1)/(1-p) = 0

Step 3: Solve
  ┌───────────────────────────────────┐
  │  p̂_MAP = (k + α - 1)/(n + α + β - 2)  │
  └───────────────────────────────────┘

Compare with:
  p̂_MLE = k/n

With α=β=1 (uniform prior): p̂_MAP = k/n = p̂_MLE  ✓
```

### Numerical Example

```
Data: 7 heads in 10 flips. Prior: Beta(3, 3) — slight belief coin is fair.

MLE:  p̂ = 7/10 = 0.700

MAP:  p̂ = (7+3-1)/(10+3+3-2) = 9/14 = 0.643

The prior "pulled" the estimate toward 0.5 (center of Beta(3,3)).
With more data, the prior's influence diminishes.
```

---

## 5. Worked Example 2: MAP for Gaussian Mean with Gaussian Prior

**Setup**: Data xᵢ ~ N(μ, σ²) with σ² known. Prior: μ ~ N(μ₀, σ₀²).

```
Step 1: Log-posterior
  ln P(μ|data) ∝ -1/(2σ²) Σ(xᵢ-μ)² - (μ-μ₀)²/(2σ₀²)

Step 2: Differentiate w.r.t. μ
  1/σ² · Σ(xᵢ-μ) - (μ-μ₀)/σ₀² = 0
  n(x̄-μ)/σ² = (μ-μ₀)/σ₀²

Step 3: Solve
  ┌──────────────────────────────────────────────┐
  │  μ̂_MAP = (σ₀²·n·x̄ + σ²·μ₀) / (σ₀²·n + σ²) │
  │                                              │
  │  = w₁·x̄ + w₂·μ₀                              │
  │                                              │
  │  where w₁ = n/σ² / (n/σ² + 1/σ₀²)           │
  │        w₂ = 1/σ₀² / (n/σ² + 1/σ₀²)          │
  └──────────────────────────────────────────────┘

This is a WEIGHTED AVERAGE of the sample mean and prior mean,
weighted by their respective precisions!
```

### Key Insights

```
n → ∞:   μ̂_MAP → x̄       (data overwhelms prior → approaches MLE)
n → 0:   μ̂_MAP → μ₀      (no data → falls back to prior)
σ₀ → ∞:  μ̂_MAP → x̄       (uninformative prior → MLE)
σ₀ → 0:  μ̂_MAP → μ₀      (very confident prior → ignores data)
```

---

## 6. Conjugate Priors

A **conjugate prior** is a prior distribution that, when combined with a particular likelihood,
yields a posterior in the **same family** as the prior.

| Likelihood    | Conjugate Prior       | Posterior              |
|---------------|----------------------|------------------------|
| Bernoulli(p)  | Beta(α, β)          | Beta(α+k, β+n-k)      |
| Binomial(n,p) | Beta(α, β)          | Beta(α+k, β+n-k)      |
| Poisson(λ)    | Gamma(α, β)         | Gamma(α+Σxᵢ, β+n)     |
| Normal(μ,σ²)  | Normal(μ₀, σ₀²)     | Normal(μ_MAP, σ_MAP²)  |
| Normal(μ,σ²)  | Inv-Gamma (for σ²)   | Inv-Gamma(updated)    |
| Multinomial   | Dirichlet(α₁,...,αₖ) | Dirichlet(α₁+x₁,...) |
| Exponential(λ)| Gamma(α, β)         | Gamma(α+n, β+Σxᵢ)    |

**Why conjugate priors matter**:
- Posterior has a **closed-form** solution
- Easy sequential updating (online learning)
- Interpretable: prior parameters act as "pseudo-observations"

---

## 7. MAP vs MLE vs Full Bayesian

```
┌────────────────────────────────────────────────────────────────┐
│  Method          Estimate        Uses Prior?    Output         │
│  ──────          ────────        ──────────    ──────         │
│  MLE             argmax P(D|θ)   No            Point estimate │
│  MAP             argmax P(θ|D)   Yes           Point estimate │
│  Full Bayesian   P(θ|D)          Yes           Distribution   │
│                                                               │
│  Prediction:                                                  │
│  MLE/MAP:   P(y|x) = P(y|x, θ̂)              (single θ)     │
│  Bayesian:  P(y|x) = ∫ P(y|x,θ)P(θ|D)dθ    (average over θ)│
└────────────────────────────────────────────────────────────────┘
```

### When to Use Each

```
MLE:        - Large datasets (prior becomes irrelevant)
            - Simple models
            - When you want a quick baseline

MAP:        - Medium datasets
            - When you have informative priors
            - When you want regularization with probabilistic justification

Bayesian:   - Small datasets (uncertainty matters)
            - When you need confidence intervals
            - Sequential decision-making (bandits, active learning)
```

---

## 8. Bayesian Inference Overview

Full Bayesian inference keeps the **entire posterior distribution**, not just its mode:

```
1. Specify prior P(θ)
2. Compute posterior P(θ|D) = P(D|θ)P(θ)/P(D)
3. For prediction: P(y*|x*,D) = ∫ P(y*|x*,θ)P(θ|D)dθ

The integral in step 3 is usually intractable → approximation methods:
  - MCMC (Markov Chain Monte Carlo)
  - Variational Inference (VI)
  - Laplace Approximation
```

### Bayesian Linear Regression Example

```
Prior:      w ~ N(0, σ²_w · I)
Likelihood: y|X,w ~ N(Xw, σ²_n · I)

Posterior:  w|X,y ~ N(μ_w, Σ_w)
  where:
    Σ_w = (X^T X / σ²_n + I / σ²_w)⁻¹
    μ_w = Σ_w · X^T y / σ²_n

Note: The MAP estimate μ_w is equivalent to Ridge regression
with λ = σ²_n / σ²_w
```

---

## 9. Python Implementation

```python
import numpy as np
from scipy import stats

# --- MAP for Bernoulli with Beta prior ---
# Data: coin flips
data = np.array([1, 1, 0, 1, 1, 1, 0, 1, 0, 1])
k = data.sum()
n = len(data)

# MLE
p_mle = k / n

# MAP with Beta(3, 3) prior
alpha, beta = 3, 3
p_map = (k + alpha - 1) / (n + alpha + beta - 2)

print(f"Data: {k} heads in {n} flips")
print(f"MLE:  p̂ = {p_mle:.4f}")
print(f"MAP:  p̂ = {p_map:.4f} (with Beta({alpha},{beta}) prior)")

# Full Bayesian posterior
alpha_post = alpha + k
beta_post = beta + (n - k)
posterior = stats.beta(alpha_post, beta_post)
print(f"\nBayesian Posterior: Beta({alpha_post}, {beta_post})")
print(f"  Mean:     {posterior.mean():.4f}")
print(f"  Mode:     {(alpha_post-1)/(alpha_post+beta_post-2):.4f} (= MAP)")
print(f"  95% CI:   [{posterior.ppf(0.025):.4f}, {posterior.ppf(0.975):.4f}]")
```

```python
# --- MAP for Gaussian with Gaussian prior ---
np.random.seed(42)

# True parameters
mu_true = 5.0
sigma_known = 2.0

# Prior
mu_0 = 0.0
sigma_0 = 3.0

# Generate data
for n in [1, 5, 20, 100]:
    data = np.random.normal(mu_true, sigma_known, size=n)
    x_bar = data.mean()
    
    # Precision-weighted MAP
    precision_data = n / sigma_known**2
    precision_prior = 1 / sigma_0**2
    
    mu_map = (precision_data * x_bar + precision_prior * mu_0) / (precision_data + precision_prior)
    mu_mle = x_bar
    
    print(f"n={n:3d}: MLE={mu_mle:6.3f}, MAP={mu_map:6.3f} (true={mu_true})")
```

```python
# --- Ridge Regression = MAP with Gaussian prior ---
from sklearn.linear_model import Ridge, LinearRegression
from sklearn.datasets import make_regression

X, y = make_regression(n_samples=20, n_features=10, noise=5, random_state=42)

# MLE (no regularization)
lr = LinearRegression().fit(X, y)
print(f"\nOLS (MLE) weight norm:    ||w|| = {np.linalg.norm(lr.coef_):.4f}")

# MAP with Gaussian prior (Ridge = L2)
for alpha in [0.1, 1.0, 10.0, 100.0]:
    ridge = Ridge(alpha=alpha).fit(X, y)
    print(f"Ridge (α={alpha:5.1f}) weight norm: ||w|| = {np.linalg.norm(ridge.coef_):.4f}")
```

---

## 10. Real-World ML Applications

| Concept              | ML Application                                            |
|----------------------|-----------------------------------------------------------|
| MAP estimation       | Regularized training of any ML model                      |
| Gaussian prior       | L2 regularization / weight decay in neural networks       |
| Laplace prior        | L1 regularization / sparse models (Lasso)                 |
| Conjugate priors     | Online learning, streaming data updates                   |
| Full Bayesian        | Bayesian neural networks, Gaussian processes               |
| Prior selection      | Transfer learning (pre-trained weights as prior)          |
| Posterior inference   | Uncertainty estimation, active learning                   |

---

## 11. Summary Table

| Concept               | Formula / Key Point                                    |
|-----------------------|--------------------------------------------------------|
| MAP estimate          | θ̂ = argmax [ln P(D\|θ) + ln P(θ)]                    |
| MAP vs MLE            | MAP = MLE + log-prior (regularization)                 |
| Gaussian prior → L2   | ln P(θ) = -λ\|\|θ\|\|² → Ridge regression            |
| Laplace prior → L1    | ln P(θ) = -λ\|\|θ\|\|₁ → Lasso regression            |
| Uniform prior         | MAP = MLE (no regularization)                          |
| Conjugate prior       | Posterior stays in same family as prior                 |
| Beta-Bernoulli MAP    | p̂ = (k+α-1)/(n+α+β-2)                                |
| Bayesian prediction   | ∫ P(y\|x,θ)P(θ\|D)dθ (marginalize over θ)            |
| More data             | Prior influence diminishes → MAP → MLE                 |

---

## 12. Quick Revision Questions

1. **Write the MAP objective function.** How does it differ from MLE?
2. **Show that a Gaussian prior on weights leads to L2 regularization.** What is λ in terms of σ²?
3. **If you observe 3 heads in 3 flips with a Beta(2,2) prior,** what is p̂_MAP?
4. **Why does MAP reduce overfitting compared to MLE?**
5. **What is a conjugate prior?** Give an example for the Poisson distribution.
6. **Compare MLE, MAP, and full Bayesian inference** in terms of: (a) what they compute,
   (b) when to use each, (c) computational cost.

---

## 13. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 8: Maximum Likelihood Estimation](08-maximum-likelihood-estimation.md) | — |

---

*Unit 3 · Probability Theory · Chapter 9 of 9*
