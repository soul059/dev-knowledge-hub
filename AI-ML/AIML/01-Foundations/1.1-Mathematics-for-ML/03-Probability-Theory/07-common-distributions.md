# Chapter 7: Common Distributions — Deep Dive

> **Unit 3: Probability Theory** | **Topic**: Bernoulli, Binomial, Multinomial, Poisson, Gaussian, Exponential, Beta, Gamma

---

## 1. Overview

This chapter provides a **deep dive** into the most important probability distributions used in
machine learning. For each distribution, we cover: formula, parameters, mean/variance, shape
intuition, and ML applications. Mastering these distributions is essential for understanding
generative models, Bayesian inference, and statistical learning theory.

---

## 2. Bernoulli Distribution

Models a single binary trial.

```
X ~ Bernoulli(p)

PMF:      P(X=x) = pˣ(1-p)¹⁻ˣ,    x ∈ {0, 1}
Mean:     E[X] = p
Variance: Var(X) = p(1-p)
```

**Shape**: Two bars at x=0 and x=1.

```
p=0.7:    ███ (0.3)      ███████ (0.7)
             x=0              x=1
```

**ML Applications**:
- Binary classification labels (spam/not spam)
- Logistic regression models P(Y=1|X) as Bernoulli
- Dropout: each neuron is kept/dropped ~ Bernoulli(p)

---

## 3. Binomial Distribution

Sum of n independent Bernoulli trials.

```
X ~ Binomial(n, p)

PMF:      P(X=k) = C(n,k) · pᵏ(1-p)ⁿ⁻ᵏ,    k = 0,1,...,n
Mean:     E[X] = np
Variance: Var(X) = np(1-p)
```

**Relation**: If X₁,...,Xₙ ~ Bernoulli(p) i.i.d., then X₁+...+Xₙ ~ Binomial(n,p).

```
Binomial(20, 0.5):          Binomial(20, 0.2):
       ╱╲                          █
      ╱  ╲                        █ █
     ╱    ╲                      █ █ █
    ╱      ╲                    █ █ █ █
   ╱        ╲                  █ █ █ █ █
  ──────────────              ██ █ █ █ █ █─────
  0    10    20               0  4     10   20
  (symmetric)                 (right-skewed)
```

**ML Applications**: Number of correct predictions in n samples, A/B testing.

---

## 4. Multinomial Distribution

Generalization of Binomial to **k** categories.

```
(X₁,...,Xₖ) ~ Multinomial(n, p₁,...,pₖ)

PMF: P(X₁=x₁,...,Xₖ=xₖ) = n! / (x₁!···xₖ!) · p₁^x₁ · ... · pₖ^xₖ

Constraints: Σ xᵢ = n,  Σ pᵢ = 1

Mean:     E[Xᵢ] = npᵢ
Variance: Var(Xᵢ) = npᵢ(1-pᵢ)
Covariance: Cov(Xᵢ,Xⱼ) = -npᵢpⱼ   (i ≠ j)
```

**ML Applications**:
- Multi-class classification (softmax output is Multinomial parameter)
- Topic modeling (LDA)
- Word count distributions in NLP (bag of words)

---

## 5. Poisson Distribution

Counts of events in a fixed interval.

```
X ~ Poisson(λ)

PMF:      P(X=k) = λᵏe⁻λ / k!,    k = 0,1,2,...
Mean:     E[X] = λ
Variance: Var(X) = λ
```

**Key property**: Mean = Variance = λ.

**Useful approximation**: Binomial(n, p) ≈ Poisson(np) when n is large and p is small.

```
λ=1:   █                    λ=5:       █ █
       █ █                           █ █ █ █
       █ █ █                       █ █ █ █ █ █
       █ █ █ █                   █ █ █ █ █ █ █ █
       ─────────                 ──────────────────
       0 1 2 3 4                 0 2 4 6 8 10 12
```

**ML Applications**: Count regression, modeling rare events, Poisson loss for count data.

---

## 6. Gaussian (Normal) Distribution

### 6.1 Univariate Gaussian

```
X ~ N(μ, σ²)

PDF: f(x) = 1/(σ√(2π)) · exp(-(x-μ)²/(2σ²))

Mean:     E[X] = μ
Variance: Var(X) = σ²
```

```
Standard Normal N(0,1):

     │        ╱╲
     │       ╱  ╲
     │      ╱    ╲
     │     ╱ 68.3% ╲
     │    ╱──┊──────╲
     │   ╱  95.4%     ╲
     │──╱── 99.7% ──────╲──
     └──┴──┴──┴──┴──┴──┴──→
       -3σ -2σ -σ  μ  σ  2σ 3σ
```

**Properties**:
- Symmetric about μ
- Completely specified by μ and σ²
- Sum of independent Gaussians is Gaussian
- Maximum entropy distribution for given mean and variance

### 6.2 Multivariate Gaussian

```
X ~ N(μ, Σ)     where X ∈ ℝᵈ, μ ∈ ℝᵈ, Σ ∈ ℝᵈˣᵈ

PDF: f(x) = 1/((2π)^(d/2)|Σ|^(1/2)) · exp(-½(x-μ)ᵀΣ⁻¹(x-μ))

Mean:       E[X] = μ
Covariance: Cov(X) = Σ
```

**Σ determines the shape of the contours**:

```
Diagonal Σ (uncorrelated):     Full Σ (correlated):
         ┌─────┐                      ╱╲
        │  ●  │                     ╱  ╲
        │     │                    ╱ ●  ╲
         └─────┘                    ╲    ╱
     (axis-aligned ellipse)          ╲╱
                                  (rotated ellipse)
```

**ML Applications**:
- Gaussian Naive Bayes, Gaussian Mixture Models (GMM)
- Gaussian processes, weight initialization
- Variational autoencoders (latent space)
- Kalman filters, linear regression errors

---

## 7. Exponential Distribution

Time between events in a Poisson process.

```
X ~ Exponential(λ)

PDF:  f(x) = λe⁻λˣ,    x ≥ 0
CDF:  F(x) = 1 - e⁻λˣ

Mean:     E[X] = 1/λ
Variance: Var(X) = 1/λ²
```

**Memoryless property**: P(X > s+t | X > s) = P(X > t)

```
f(x)
  │╲
  │ ╲  λ=2
  │  ╲╲─── λ=1
  │   ╲ ╲───── λ=0.5
  │    ╲  ╲────────
  └──────────────────→ x
  0
```

**ML Applications**: Modeling inter-arrival times, survival analysis, learning rate decay.

---

## 8. Beta Distribution

Distribution over probabilities (values in [0, 1]).

```
X ~ Beta(α, β)

PDF: f(x) = x^(α-1)(1-x)^(β-1) / B(α,β),    x ∈ [0,1]

where B(α,β) = Γ(α)Γ(β)/Γ(α+β)

Mean:     E[X] = α/(α+β)
Variance: Var(X) = αβ / ((α+β)²(α+β+1))
Mode:     (α-1)/(α+β-2)   for α,β > 1
```

**Shape depends on α and β**:

```
α=1,β=1 (Uniform):    α=5,β=5 (Symmetric):    α=2,β=5 (Left-skewed):
  ────────────            ╱╲                      ╱╲
  │██████████│          ╱    ╲                   ╱  ╲───
  └──────────┘         ╱      ╲                 ╱      ╲
  0          1        0  0.5   1               0  0.3   1

α=0.5,β=0.5 (U-shaped):   α=5,β=2 (Right-skewed):
  ╲        ╱                        ╱╲
   ╲      ╱                    ───╱  ╲
    ╲    ╱                       ╱    ╲
     ╲──╱                      ╱      ╲
  0  0.5  1                   0   0.7  1
```

**ML Applications**:
- **Conjugate prior for Bernoulli/Binomial** — posterior is also Beta
- Thompson sampling in multi-armed bandits
- Bayesian A/B testing
- Modeling probabilities as random variables

### Conjugate Prior Example

```
Prior:      p ~ Beta(α, β)
Data:       k successes in n trials (Binomial likelihood)
Posterior:  p | data ~ Beta(α + k, β + n - k)

Example: Prior Beta(1,1) = Uniform, observe 7 heads in 10 flips
         Posterior: Beta(1+7, 1+3) = Beta(8, 4)
         Posterior mean: 8/12 = 0.667
```

---

## 9. Gamma Distribution

Generalization of Exponential; models waiting time for α events.

```
X ~ Gamma(α, β)     (shape α, rate β)

PDF:  f(x) = β^α · x^(α-1) · e^(-βx) / Γ(α),    x ≥ 0

Mean:     E[X] = α/β
Variance: Var(X) = α/β²
```

**Special cases**:
- Gamma(1, β) = Exponential(β)
- Gamma(n/2, 1/2) = Chi-squared(n)

```
α=1 (Exponential):    α=3:              α=10:
  │╲                    │   ╱╲             │      ╱╲
  │ ╲                   │  ╱  ╲            │    ╱    ╲
  │  ╲───               │ ╱    ╲──         │   ╱      ╲
  └──────→              └╱────────→        └──╱────────╲→
```

**ML Applications**:
- Conjugate prior for Poisson rate parameter
- Prior for precision (1/σ²) in Bayesian regression
- Modeling positive-valued quantities

---

## 10. Python Code: Distribution Exploration

```python
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

# --- Compare distributions ---
fig, axes = plt.subplots(2, 4, figsize=(16, 8))

# Bernoulli
x = [0, 1]; p = 0.7
axes[0,0].bar(x, [1-p, p]); axes[0,0].set_title('Bernoulli(0.7)')

# Binomial
k = np.arange(0, 21)
axes[0,1].bar(k, stats.binom.pmf(k, 20, 0.5)); axes[0,1].set_title('Binomial(20,0.5)')

# Poisson
k = np.arange(0, 20)
axes[0,2].bar(k, stats.poisson.pmf(k, 5)); axes[0,2].set_title('Poisson(5)')

# Gaussian
x = np.linspace(-4, 4, 200)
axes[0,3].plot(x, stats.norm.pdf(x)); axes[0,3].set_title('N(0,1)')

# Exponential
x = np.linspace(0, 5, 200)
for lam in [0.5, 1, 2]:
    axes[1,0].plot(x, stats.expon.pdf(x, scale=1/lam), label=f'λ={lam}')
axes[1,0].set_title('Exponential'); axes[1,0].legend()

# Beta
x = np.linspace(0.001, 0.999, 200)
for a, b in [(1,1),(2,5),(5,2),(5,5)]:
    axes[1,1].plot(x, stats.beta.pdf(x, a, b), label=f'({a},{b})')
axes[1,1].set_title('Beta'); axes[1,1].legend()

# Gamma
x = np.linspace(0.01, 15, 200)
for a in [1, 3, 5]:
    axes[1,2].plot(x, stats.gamma.pdf(x, a, scale=1), label=f'α={a}')
axes[1,2].set_title('Gamma'); axes[1,2].legend()

# Multinomial (visualize as bar chart of one sample)
sample = np.random.multinomial(100, [0.1, 0.2, 0.3, 0.15, 0.25])
axes[1,3].bar(range(5), sample); axes[1,3].set_title('Multinomial sample')

plt.tight_layout(); plt.show()
```

```python
# Beta as conjugate prior for Bernoulli
alpha_prior, beta_prior = 2, 2
data = [1, 1, 0, 1, 1, 1, 0, 1]
successes = sum(data)
failures = len(data) - successes

alpha_post = alpha_prior + successes
beta_post = beta_prior + failures

x = np.linspace(0, 1, 200)
plt.plot(x, stats.beta.pdf(x, alpha_prior, beta_prior), 'b--', label='Prior Beta(2,2)')
plt.plot(x, stats.beta.pdf(x, alpha_post, beta_post), 'r-', label=f'Posterior Beta({alpha_post},{beta_post})')
plt.axvline(alpha_post/(alpha_post+beta_post), color='r', ls=':', label=f'Post. Mean={alpha_post/(alpha_post+beta_post):.3f}')
plt.legend(); plt.title('Bayesian Updating with Beta-Bernoulli')
plt.show()
```

---

## 11. Comprehensive Comparison Table

| Distribution | Type | Parameters | PMF/PDF | Mean | Variance | Support | ML Use |
|---|---|---|---|---|---|---|---|
| Bernoulli | Disc. | p | pˣ(1-p)¹⁻ˣ | p | p(1-p) | {0,1} | Binary labels |
| Binomial | Disc. | n, p | C(n,k)pᵏ(1-p)ⁿ⁻ᵏ | np | np(1-p) | {0..n} | Batch accuracy |
| Multinomial | Disc. | n, p₁..pₖ | n!/(∏xᵢ!)·∏pᵢ^xᵢ | npᵢ | npᵢ(1-pᵢ) | Σxᵢ=n | Multi-class |
| Poisson | Disc. | λ | λᵏe⁻λ/k! | λ | λ | {0,1,2..} | Count data |
| Gaussian | Cont. | μ, σ² | (2πσ²)⁻½exp(−(x−μ)²/2σ²) | μ | σ² | ℝ | Everywhere |
| Exponential | Cont. | λ | λe⁻λˣ | 1/λ | 1/λ² | [0,∞) | Time models |
| Beta | Cont. | α, β | x^(α-1)(1-x)^(β-1)/B(α,β) | α/(α+β) | αβ/((α+β)²(α+β+1)) | [0,1] | Prior for p |
| Gamma | Cont. | α, β | β^α·x^(α-1)·e^(-βx)/Γ(α) | α/β | α/β² | [0,∞) | Prior for λ |

---

## 12. Quick Revision Questions

1. **What is the conjugate prior for the Bernoulli distribution?** Why is this useful?
2. **How does the multivariate Gaussian PDF differ from the univariate?** What role does Σ play?
3. **When should you use Poisson vs Binomial?** Give criteria.
4. **What are the special cases of the Gamma distribution?**
5. **Explain why the Beta distribution is ideal for modeling probabilities.**
6. **A softmax layer outputs [0.1, 0.3, 0.6].** Which distribution does this parameterize?

---

## 13. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 6: Bayes' Theorem](06-bayes-theorem.md) | [Chapter 8: Maximum Likelihood Estimation](08-maximum-likelihood-estimation.md) |

---

*Unit 3 · Probability Theory · Chapter 7 of 9*
