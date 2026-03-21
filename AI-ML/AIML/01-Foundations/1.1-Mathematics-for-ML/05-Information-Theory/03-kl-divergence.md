# Chapter 3: KL Divergence

> **Unit 5 — Information Theory** | Mathematics for Machine Learning

---

## 1. Overview

**Kullback-Leibler (KL) Divergence** measures how one probability distribution **diverges** from a reference distribution. It quantifies the extra bits needed when using distribution q to encode data that actually follows distribution p. KL divergence is central to variational inference, generative models (VAEs, GANs), and knowledge distillation.

---

## 2. KL Divergence — Definition

For discrete distributions p and q defined over the same support:

```
                   n           p(xᵢ)
D_KL(p ‖ q) =  Σ  p(xᵢ) log ───────
                  i=1          q(xᵢ)
```

Equivalently:

```
D_KL(p ‖ q) = Σ p(x) log p(x) - Σ p(x) log q(x)
            = -H(p) + H(p, q)
            = H(p, q) - H(p)
```

For continuous distributions:

```
D_KL(p ‖ q) = ∫ p(x) log [p(x) / q(x)] dx
```

---

## 3. Fundamental Properties

| Property               | Statement                                               |
|------------------------|---------------------------------------------------------|
| **Non-negativity**     | D_KL(p ‖ q) ≥ 0 (Gibbs' inequality)                   |
| **Zero condition**     | D_KL(p ‖ q) = 0 if and only if p = q (a.e.)            |
| **Asymmetric**         | D_KL(p ‖ q) ≠ D_KL(q ‖ p) in general                  |
| **Not a metric**       | Fails symmetry and triangle inequality                  |
| **Unbounded**          | D_KL can be infinite if q(x)=0 where p(x)>0            |
| **Additive**           | For independent variables: D_KL(p₁p₂ ‖ q₁q₂) = D_KL(p₁‖q₁) + D_KL(p₂‖q₂) |

### Why KL Divergence Is Not a Distance

```
D_KL(p ‖ q) ≠ D_KL(q ‖ p)     ← asymmetric!

Example:
  p = [0.5, 0.5],  q = [0.9, 0.1]

  D_KL(p ‖ q) = 0.5·log(0.5/0.9) + 0.5·log(0.5/0.1)
              = 0.5·(-0.848) + 0.5·(2.322)
              = -0.424 + 1.161 = 0.737 bits

  D_KL(q ‖ p) = 0.9·log(0.9/0.5) + 0.1·log(0.1/0.5)
              = 0.9·(0.848) + 0.1·(-2.322)
              = 0.763 - 0.232 = 0.531 bits

  0.737 ≠ 0.531  ✓ asymmetric confirmed
```

---

## 4. The Key Relationship

```
┌─────────────────────────────────────────────────┐
│  H(p, q)  =  H(p)  +  D_KL(p ‖ q)             │
│                                                  │
│  Cross-Entropy = Entropy + KL Divergence         │
│                                                  │
│  Since D_KL ≥ 0:  H(p, q) ≥ H(p)               │
└─────────────────────────────────────────────────┘
```

**Implication for ML:** Minimizing cross-entropy H(p,q) w.r.t. q is equivalent to minimizing D_KL(p‖q), since H(p) is constant w.r.t. model parameters.

---

## 5. Forward KL vs. Reverse KL

This distinction is critical in variational inference and generative modeling.

### Forward KL: D_KL(p ‖ q) — "Mean-seeking"

```
D_KL(p ‖ q) = Σ p(x) log [p(x) / q(x)]
```

- Expectation under **p** (true distribution)
- Penalizes q(x) = 0 where p(x) > 0 → **q must cover all modes of p**
- Result: q tends to be **broad**, covering all modes (mean-seeking)

### Reverse KL: D_KL(q ‖ p) — "Mode-seeking"

```
D_KL(q ‖ p) = Σ q(x) log [q(x) / p(x)]
```

- Expectation under **q** (approximate distribution)
- Penalizes q(x) > 0 where p(x) = 0 → **q avoids regions where p is zero**
- Result: q tends to **lock onto one mode** (mode-seeking)

### ASCII Visualization

```
  True distribution p (bimodal):

  p(x)
   │    ╱╲          ╱╲
   │   ╱  ╲        ╱  ╲
   │  ╱    ╲      ╱    ╲
   │ ╱      ╲    ╱      ╲
   │╱        ╲──╱        ╲
   └──────────────────────→ x

  Forward KL → q covers both modes (broad):
  q(x)
   │   ╱──────────────╲
   │  ╱                ╲
   │ ╱                  ╲
   │╱                    ╲
   └──────────────────────→ x

  Reverse KL → q locks onto one mode:
  q(x)
   │    ╱╲
   │   ╱  ╲
   │  ╱    ╲
   │ ╱      ╲
   │╱        ╲───────────
   └──────────────────────→ x
```

### Summary

| Property       | Forward KL D_KL(p‖q)       | Reverse KL D_KL(q‖p)        |
|----------------|---------------------------|------------------------------|
| Expectation    | Under p (data)             | Under q (model)              |
| Behavior       | Mean-seeking (broad)       | Mode-seeking (narrow)        |
| Zero-avoiding  | q avoids zero where p > 0  | q avoids areas where p ≈ 0   |
| Used in        | MLE, supervised learning   | Variational inference (ELBO) |

---

## 6. Step-by-Step Examples

### Example 1: Two Bernoulli Distributions

p = [0.7, 0.3], q = [0.5, 0.5]

```
D_KL(p ‖ q) = 0.7 · log₂(0.7/0.5) + 0.3 · log₂(0.3/0.5)
            = 0.7 · log₂(1.4) + 0.3 · log₂(0.6)
            = 0.7 · 0.485 + 0.3 · (-0.737)
            = 0.340 - 0.221
            = 0.119 bits
```

### Example 2: How Bad Is a Wrong Model?

True: p = [0.25, 0.25, 0.25, 0.25] (uniform, 4 classes)
Model: q = [0.1, 0.1, 0.1, 0.7]

```
D_KL(p ‖ q) = 0.25·log₂(0.25/0.1) + 0.25·log₂(0.25/0.1)
            + 0.25·log₂(0.25/0.1) + 0.25·log₂(0.25/0.7)

            = 3 × 0.25·log₂(2.5) + 0.25·log₂(0.357)
            = 3 × 0.25·1.322 + 0.25·(-1.485)
            = 0.992 - 0.371
            = 0.620 bits
```

### Example 3: KL Divergence Between Two Gaussians

For p = N(μ₁, σ₁²) and q = N(μ₂, σ₂²):

```
                      σ₂     σ₁² + (μ₁ - μ₂)²      1
D_KL(p ‖ q) = log ───── + ──────────────────── - ─────
                      σ₁         2σ₂²               2
```

Let p = N(0, 1), q = N(1, 2):
```
D_KL = log(√2/1) + (1 + 1)/(2·2) - 0.5
     = 0.347 + 0.5 - 0.5
     = 0.347 nats
```

---

## 7. Python Implementation

```python
import numpy as np
from scipy.stats import entropy as scipy_entropy

def kl_divergence(p, q, base=2):
    """Compute KL divergence D_KL(p || q)."""
    p = np.array(p, dtype=float)
    q = np.array(q, dtype=float)
    # Filter out zero probabilities in p
    mask = p > 0
    return np.sum(p[mask] * np.log(p[mask] / q[mask])) / np.log(base)

def kl_gaussian(mu1, sigma1, mu2, sigma2):
    """KL divergence between two univariate Gaussians (nats)."""
    return (np.log(sigma2 / sigma1)
            + (sigma1**2 + (mu1 - mu2)**2) / (2 * sigma2**2)
            - 0.5)

# --- Discrete examples ---
p = [0.7, 0.3]
q = [0.5, 0.5]
print(f"D_KL(p||q) = {kl_divergence(p, q):.4f} bits")
print(f"D_KL(q||p) = {kl_divergence(q, p):.4f} bits  ← asymmetric!")

# Verify with scipy
print(f"Scipy check: {scipy_entropy(p, q, base=2):.4f} bits")

# --- Gaussian example ---
print(f"D_KL(N(0,1) || N(1,2)) = {kl_gaussian(0, 1, 1, np.sqrt(2)):.4f} nats")

# --- Visualization ---
import matplotlib.pyplot as plt

ps = np.linspace(0.01, 0.99, 200)
q_fixed = 0.5
kl_vals = ps * np.log2(ps / q_fixed) + (1 - ps) * np.log2((1 - ps) / (1 - q_fixed))

plt.plot(ps, kl_vals)
plt.xlabel('p'); plt.ylabel('D_KL(p || q=0.5)')
plt.title('KL Divergence vs True Probability (q fixed at 0.5)')
plt.axhline(y=0, color='r', linestyle='--')
plt.grid(True); plt.show()
```

**Output:**
```
D_KL(p||q) = 0.1187 bits
D_KL(q||p) = 0.1280 bits  ← asymmetric!
Scipy check: 0.1187 bits
D_KL(N(0,1) || N(1,2)) = 0.3466 nats
```

---

## 8. Applications in Machine Learning

### 8.1 Variational Autoencoders (VAEs)

The VAE loss (ELBO) explicitly includes a KL term:

```
L_VAE = E_q[log p(x|z)] - D_KL(q(z|x) ‖ p(z))
        \_____________/   \___________________/
        Reconstruction     Regularization
```

The KL term pushes the learned latent distribution q(z|x) toward the prior p(z) = N(0, I).

### 8.2 Knowledge Distillation

```
L = α · CE(y, student) + (1-α) · T² · D_KL(teacher_soft ‖ student_soft)
```

The student network learns to match the teacher's soft probability distribution.

### 8.3 Policy Gradient / PPO (Reinforcement Learning)

PPO constrains policy updates using KL divergence:

```
D_KL(π_old ‖ π_new) ≤ δ
```

This prevents the policy from changing too drastically in a single update.

### 8.4 Other Applications

| Application                  | KL Divergence Role                                  |
|-----------------------------|-----------------------------------------------------|
| **Bayesian Inference**      | Variational inference minimizes reverse KL           |
| **GANs**                    | Related to JS divergence (symmetric KL variant)      |
| **Information Geometry**    | KL defines the Fisher information metric             |
| **Anomaly Detection**       | Detect distribution shift via KL monitoring          |
| **A/B Testing**             | Compare experimental distributions                   |

---

## 9. Related Divergence Measures

| Measure               | Formula                                    | Symmetric? |
|-----------------------|--------------------------------------------|------------|
| KL Divergence         | Σ p log(p/q)                               | No         |
| Reverse KL            | Σ q log(q/p)                               | No         |
| JS Divergence         | ½ D_KL(p‖m) + ½ D_KL(q‖m), m=(p+q)/2     | Yes        |
| Total Variation       | ½ Σ |p(x) - q(x)|                          | Yes        |
| Hellinger Distance    | √(½ Σ (√p - √q)²)                         | Yes        |
| Wasserstein Distance  | inf E[|X-Y|] over couplings               | Yes        |

---

## 10. Summary Table

| Concept                | Formula / Key Fact                                     |
|------------------------|-------------------------------------------------------|
| KL Divergence          | D_KL(p‖q) = Σ p(x) log[p(x)/q(x)]                   |
| Non-negativity         | D_KL(p‖q) ≥ 0, with equality iff p = q                |
| Asymmetry              | D_KL(p‖q) ≠ D_KL(q‖p)                                |
| Decomposition          | H(p,q) = H(p) + D_KL(p‖q)                            |
| Forward KL             | Mean-seeking; used in MLE                              |
| Reverse KL             | Mode-seeking; used in variational inference            |
| Gaussian KL            | log(σ₂/σ₁) + (σ₁²+(μ₁-μ₂)²)/(2σ₂²) - ½             |
| ML connection          | min_q H(p,q) ≡ min_q D_KL(p‖q)                       |

---

## 11. Quick Revision Questions

1. **Why is KL divergence not a true distance metric?**
   > It is asymmetric: D_KL(p‖q) ≠ D_KL(q‖p), and it doesn't satisfy the triangle inequality.

2. **What does D_KL(p‖q) = 0 tell us?**
   > The distributions p and q are identical (almost everywhere).

3. **Explain the difference between forward and reverse KL in one sentence each.**
   > Forward KL (D_KL(p‖q)) is mean-seeking — q spreads to cover all of p's modes. Reverse KL (D_KL(q‖p)) is mode-seeking — q concentrates on a single mode of p.

4. **Why does minimizing cross-entropy w.r.t. q equal minimizing KL divergence?**
   > Because H(p,q) = H(p) + D_KL(p‖q) and H(p) is constant w.r.t. q, so argmin_q H(p,q) = argmin_q D_KL(p‖q).

5. **What role does KL divergence play in VAEs?**
   > The KL term D_KL(q(z|x)‖p(z)) regularizes the encoder by pushing the approximate posterior toward the prior distribution.

6. **Can KL divergence be infinite? When?**
   > Yes, if q(x) = 0 for any x where p(x) > 0, then log(p(x)/q(x)) → ∞.

---

[← Previous Chapter: 02-Cross-Entropy](./02-cross-entropy.md) | [Next Chapter: 04-Mutual-Information →](./04-mutual-information.md)
