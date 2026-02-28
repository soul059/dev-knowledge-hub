# Chapter 8.4: Point Estimation

[← Previous: Central Limit Theorem](03-central-limit-theorem.md) | [Back to README](../README.md) | [Next: Confidence Intervals →](05-confidence-intervals.md)

---

## Overview

**Point estimation** involves using sample data to compute a single "best guess" for an unknown population parameter. But not all estimators are equally good — we need criteria to evaluate and compare them.

---

## 1. Estimator vs Estimate

| Term | Definition | Example |
|------|-----------|---------|
| **Estimator** | A rule/formula (random variable) | $\hat{\mu} = \bar{X} = \frac{1}{n}\sum X_i$ |
| **Estimate** | A specific numerical value | $\hat{\mu} = 67.3$ (from observed data) |

```
  Parameter θ (unknown, fixed)
       ↑
       │ estimated by
       │
  Estimator θ̂ = g(X₁, ..., Xₙ)  ← random variable (before data)
       │
       │ plug in observed data
       ▼
  Estimate θ̂ = g(x₁, ..., xₙ)   ← number (after data)
```

---

## 2. Desirable Properties of Estimators

### 2.1 Unbiasedness

$$\boxed{E[\hat{\theta}] = \theta \quad \text{(for all } \theta \text{)}}$$

An estimator is **unbiased** if, on average, it hits the true value.

$$\text{Bias}(\hat{\theta}) = E[\hat{\theta}] - \theta$$

| Estimator | Unbiased? |
|-----------|-----------|
| $\bar{X}$ for $\mu$ | Yes: $E[\bar{X}] = \mu$ |
| $S^2$ for $\sigma^2$ | Yes: $E[S^2] = \sigma^2$ |
| $S$ for $\sigma$ | **No**: $E[S] < \sigma$ (Jensen) |
| $\hat{p}$ for $p$ | Yes: $E[\hat{p}] = p$ |

```
  Biased estimator:          Unbiased estimator:
  
  ────│────●────────         ────│────────────
      θ   E[θ̂]                   θ = E[θ̂]
      
  Bias = E[θ̂] - θ > 0       Bias = 0
  
  ●───→ Systematically       ● Centered on target
        off-target
```

### 2.2 Efficiency (Minimum Variance)

Among all unbiased estimators, prefer the one with **smallest variance**.

$$\text{Efficiency}(\hat{\theta}_1 \text{ vs } \hat{\theta}_2) = \frac{\text{Var}(\hat{\theta}_2)}{\text{Var}(\hat{\theta}_1)}$$

### 2.3 Consistency

$$\hat{\theta}_n \xrightarrow{P} \theta \quad \text{as } n \to \infty$$

The estimator converges to the true value as sample size grows.

**Example:** $\bar{X}$ is consistent for $\mu$ (by WLLN).

### 2.4 Sufficiency

$T$ is **sufficient** for $\theta$ if the conditional distribution of the sample given $T$ does not depend on $\theta$. Intuitively: $T$ captures all the information about $\theta$ in the sample.

---

## 3. Mean Squared Error (MSE)

$$\boxed{\text{MSE}(\hat{\theta}) = E[(\hat{\theta} - \theta)^2] = \text{Var}(\hat{\theta}) + [\text{Bias}(\hat{\theta})]^2}$$

```
  MSE Decomposition
  
  MSE = Variance + Bias²
  
  ┌──────────────────────────────────────────┐
  │ MSE (total error)                        │
  │ ┌──────────────┐ ┌────────────────────┐  │
  │ │   Bias²      │ │    Variance        │  │
  │ │ (accuracy)   │ │  (precision)       │  │
  │ └──────────────┘ └────────────────────┘  │
  └──────────────────────────────────────────┘
  
  Unbiased: MSE = Var          Biased: MSE = Var + Bias²
```

### Bias-Variance Tradeoff

Sometimes a slightly **biased** estimator with much **lower variance** has smaller MSE:

| Estimator | Bias | Variance | MSE |
|-----------|------|----------|-----|
| $\bar{X}$ (unbiased) | 0 | $\sigma^2/n$ | $\sigma^2/n$ |
| $\alpha\bar{X}$ (shrunken) | $(\alpha-1)\mu$ | $\alpha^2\sigma^2/n$ | Can be lower! |

---

## 4. Methods of Point Estimation

### 4.1 Method of Moments (MoM)

**Idea:** Set population moments equal to sample moments, solve for parameters.

$$\mu_k' = E[X^k] \quad \longleftrightarrow \quad m_k' = \frac{1}{n}\sum_{i=1}^n X_i^k$$

**Steps:**
1. Write population moments in terms of parameters
2. Set population moments = sample moments
3. Solve for parameter estimates

### Example: Estimating Normal Parameters

- $E[X] = \mu \Rightarrow \hat{\mu}_{\text{MoM}} = \bar{X}$
- $E[X^2] = \mu^2 + \sigma^2 \Rightarrow \hat{\sigma}^2_{\text{MoM}} = \frac{1}{n}\sum X_i^2 - \bar{X}^2 = \frac{1}{n}\sum(X_i - \bar{X})^2$

(Note: MoM gives biased $\hat{\sigma}^2$ with divisor $n$, not $n-1$.)

### Example: Exponential

$X \sim \text{Exp}(\lambda)$: $E[X] = 1/\lambda$

$$\bar{X} = 1/\hat{\lambda} \Rightarrow \hat{\lambda}_{\text{MoM}} = 1/\bar{X}$$

---

### 4.2 Maximum Likelihood Estimation (MLE)

**Idea:** Choose the parameter value that makes the observed data most probable.

$$\boxed{\hat{\theta}_{\text{MLE}} = \arg\max_\theta L(\theta) = \arg\max_\theta \ell(\theta)}$$

where:
- **Likelihood:** $L(\theta) = \prod_{i=1}^n f(x_i; \theta)$
- **Log-likelihood:** $\ell(\theta) = \sum_{i=1}^n \ln f(x_i; \theta)$

**Steps:**
1. Write the likelihood $L(\theta)$
2. Take log → log-likelihood $\ell(\theta)$
3. Differentiate: $\frac{d\ell}{d\theta} = 0$
4. Solve for $\hat{\theta}$
5. Verify it's a maximum (second derivative test)

```
  Likelihood Function
  
  L(θ)                        ℓ(θ) = ln L(θ)
  │     ╱╲                    │     ╱╲
  │    ╱  ╲                   │    ╱  ╲
  │   ╱    ╲                  │   ╱    ╲
  │  ╱      ╲                 │  ╱      ╲
  │ ╱        ╲                │ ╱        ╲
  └──────────────► θ          └──────────────► θ
        θ̂_MLE                       θ̂_MLE
        (max)                        (max)
```

### Example: Normal Mean (σ known)

$$\ell(\mu) = -\frac{n}{2}\ln(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum(x_i - \mu)^2$$

$$\frac{d\ell}{d\mu} = \frac{1}{\sigma^2}\sum(x_i - \mu) = 0 \Rightarrow \hat{\mu}_{\text{MLE}} = \bar{x}$$

### Example: Bernoulli

$$L(p) = p^k(1-p)^{n-k}, \quad \ell(p) = k\ln p + (n-k)\ln(1-p)$$

$$\frac{d\ell}{dp} = \frac{k}{p} - \frac{n-k}{1-p} = 0 \Rightarrow \hat{p}_{\text{MLE}} = k/n = \hat{p}$$

### Example: Exponential

$$L(\lambda) = \lambda^n e^{-\lambda\sum x_i}, \quad \ell(\lambda) = n\ln\lambda - \lambda\sum x_i$$

$$\frac{d\ell}{d\lambda} = \frac{n}{\lambda} - \sum x_i = 0 \Rightarrow \hat{\lambda}_{\text{MLE}} = \frac{n}{\sum x_i} = \frac{1}{\bar{x}}$$

---

## 5. Properties of MLE

| Property | Description |
|----------|-------------|
| Consistency | $\hat{\theta}_{\text{MLE}} \xrightarrow{P} \theta$ |
| Asymptotic normality | $\hat{\theta} \approx N(\theta, 1/(nI(\theta)))$ for large $n$ |
| Asymptotic efficiency | Achieves the Cramér-Rao lower bound |
| Invariance | If $\hat{\theta}$ is MLE of $\theta$, then $g(\hat{\theta})$ is MLE of $g(\theta)$ |
| May be biased | MLE not always unbiased (but bias → 0 as $n → ∞$) |

---

## 6. Cramér-Rao Lower Bound

For any unbiased estimator $\hat{\theta}$:

$$\boxed{\text{Var}(\hat{\theta}) \geq \frac{1}{nI(\theta)}}$$

where $I(\theta) = E\left[-\frac{\partial^2}{\partial\theta^2}\ln f(X;\theta)\right]$ is the **Fisher information**.

An estimator achieving this bound is called **efficient** (or MVUE — Minimum Variance Unbiased Estimator).

---

## 7. Comparison: MoM vs MLE

| Aspect | Method of Moments | Maximum Likelihood |
|--------|-------------------|-------------------|
| Ease | Usually simpler | Can be complex |
| Efficiency | Often less efficient | Asymptotically efficient |
| Consistency | Yes (usually) | Yes |
| Uniqueness | May not be unique | Usually unique |
| Invariance | No | Yes |

---

## Summary Table

| Concept | Key Formula/Idea |
|---------|-----------------|
| Bias | $\text{Bias} = E[\hat{\theta}] - \theta$ |
| MSE | $\text{MSE} = \text{Var} + \text{Bias}^2$ |
| Unbiased | $E[\hat{\theta}] = \theta$ |
| Consistent | $\hat{\theta}_n \to \theta$ as $n \to \infty$ |
| MoM | Set $E[X^k] = \frac{1}{n}\sum X_i^k$ |
| MLE | Maximize $L(\theta) = \prod f(x_i;\theta)$ |
| CRLB | $\text{Var}(\hat{\theta}) \geq 1/(nI(\theta))$ |

---

## Quick Revision Questions

1. Is $S$ (sample standard deviation) unbiased for $\sigma$? Why or why not?

2. Find the MLE of $\lambda$ for a Poisson sample $X_1, \ldots, X_n$.

3. Use Method of Moments to estimate $a$ and $b$ for $\text{Uniform}(a,b)$ from a sample.

4. Explain the bias-variance tradeoff. When might a biased estimator be preferred?

5. If $\hat{\theta}$ is the MLE of $\theta$, what is the MLE of $\theta^2$? (Use the invariance property.)

6. State the Cramér-Rao Lower Bound and explain what it means for an estimator to be "efficient."

---

[← Previous: Central Limit Theorem](03-central-limit-theorem.md) | [Back to README](../README.md) | [Next: Confidence Intervals →](05-confidence-intervals.md)
