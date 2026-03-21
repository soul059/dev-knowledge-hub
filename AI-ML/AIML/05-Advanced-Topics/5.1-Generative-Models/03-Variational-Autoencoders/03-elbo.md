# Evidence Lower Bound (ELBO)

## Overview

The **ELBO** (Evidence Lower Bound) is the objective function that VAEs maximize during training. It provides a tractable lower bound on the intractable log-likelihood log P(x). The ELBO decomposes into two interpretable terms: a reconstruction term (how well the model regenerates data) and a regularization term (how close the approximate posterior is to the prior).

---

## Derivation

```
Start with log-evidence:
  log P(x) = log ∫ P(x,z) dz = log ∫ P(x|z)P(z) dz

Introduce approximate posterior q(z|x):
  log P(x) = log ∫ P(x|z)P(z) × q(z|x)/q(z|x) dz
            = log E_q[P(x|z)P(z)/q(z|x)]

Apply Jensen's inequality (log E ≥ E log):
  log P(x) ≥ E_q[log P(x|z)P(z)/q(z|x)]
            = E_q[log P(x|z)] + E_q[log P(z)/q(z|x)]
            = E_q[log P(x|z)] - KL(q(z|x) || P(z))

  ┌─────────────────────────────────────────────────────┐
  │                                                       │
  │  ELBO = E_q[log P(x|z)]  -  KL(q(z|x) || P(z))     │
  │         ────────────────     ────────────────────     │
  │         Reconstruction       Regularization           │
  │         (decode well)        (stay close to prior)    │
  │                                                       │
  │  log P(x) = ELBO + KL(q(z|x) || P(z|x))            │
  │                    ──────────────────────             │
  │                    ≥ 0 (always non-negative)          │
  │                                                       │
  │  Therefore: log P(x) ≥ ELBO                          │
  │  Maximizing ELBO ≈ maximizing log P(x)               │
  └─────────────────────────────────────────────────────┘
```

---

## The Two Terms

```
Term 1: RECONSTRUCTION  E_q[log P(x|z)]

  "How well can the decoder reconstruct x from z?"
  
  For Bernoulli decoder (binary data):
    E_q[log P(x|z)] ≈ Σ [x_i log x̂_i + (1-x_i) log(1-x̂_i)]
    = Binary Cross-Entropy (BCE)
  
  For Gaussian decoder (continuous data):
    E_q[log P(x|z)] ≈ -||x - x̂||²
    = negative MSE

  Estimated by sampling z ~ q(z|x) and decoding.

Term 2: REGULARIZATION  -KL(q(z|x) || P(z))

  "How close is the learned posterior to the prior?"
  
  q(z|x) = N(μ_φ(x), σ_φ(x)²)    (encoder output)
  P(z) = N(0, I)                    (standard normal prior)
  
  Closed-form for Gaussian-Gaussian KL:
    KL = -½ Σ_j [1 + log(σ_j²) - μ_j² - σ_j²]

  ┌──────────────────────────────────────────────────┐
  │  KL = 0 when q(z|x) = N(0,1) perfectly           │
  │  KL > 0 when q(z|x) deviates from N(0,1)        │
  │                                                    │
  │  This term:                                       │
  │  • Prevents latent space from collapsing          │
  │  • Ensures smooth, continuous latent space        │
  │  • Makes generation possible (sample from N(0,1))│
  └──────────────────────────────────────────────────┘
```

---

## The Tension

```
Reconstruction wants: q(z|x) very peaked, different per x
  → each input maps to a distinct point → easy to reconstruct
  → but latent space has gaps → bad for generation

KL wants: q(z|x) ≈ N(0,1) for all x
  → everything maps to the same distribution → smooth
  → but all inputs look the same → poor reconstruction

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Recon only:    KL only:     Balanced:            │
  │                                                    │
  │  ● ●    ● ●   ●●●●●●●●●    ●●●●●●●●             │
  │  ● ●    ● ●   ●●●●●●●●●    ●●●●●●●●             │
  │     ● ●        ●●●●●●●●●    ●●●●●●●●             │
  │  (sharp but    (all same,   (smooth clusters,     │
  │   gaps)        no info)      generation works!)    │
  └──────────────────────────────────────────────────┘

  Balance is controlled by relative weighting
  (or β in β-VAE)
```

---

## ELBO as Loss Function

```
VAE Loss = -ELBO = Reconstruction Loss + KL Loss

  L(θ, φ; x) = -E_q[log P_θ(x|z)] + KL(q_φ(z|x) || P(z))

  In practice:
    L = BCE(x, x̂)  +  ½ Σ(μ² + σ² - log(σ²) - 1)

  Or equivalently:
    L = MSE(x, x̂)  +  KL_term

  Both terms are differentiable → train with backprop!
  (thanks to the reparameterization trick — next chapter)
```

---

## Python: ELBO Loss

```python
import torch
import torch.nn.functional as F

def vae_loss(x, x_recon, mu, log_var):
    """
    Compute VAE ELBO loss.
    
    Args:
        x:       original input [batch, features]
        x_recon: reconstructed output [batch, features]
        mu:      encoder mean [batch, latent_dim]
        log_var: encoder log-variance [batch, latent_dim]
    
    Returns:
        total_loss, recon_loss, kl_loss
    """
    # Reconstruction loss (BCE for binary data)
    recon_loss = F.binary_cross_entropy(x_recon, x, reduction='sum')
    
    # KL divergence: KL(N(mu, sigma) || N(0, 1))
    # = -0.5 * sum(1 + log(sigma^2) - mu^2 - sigma^2)
    kl_loss = -0.5 * torch.sum(1 + log_var - mu.pow(2) - log_var.exp())
    
    total_loss = recon_loss + kl_loss
    return total_loss, recon_loss, kl_loss

# For continuous data, use MSE reconstruction
def vae_loss_mse(x, x_recon, mu, log_var):
    recon_loss = F.mse_loss(x_recon, x, reduction='sum')
    kl_loss = -0.5 * torch.sum(1 + log_var - mu.pow(2) - log_var.exp())
    return recon_loss + kl_loss, recon_loss, kl_loss
```

---

## Revision Questions

1. **What does ELBO stand for and why is it a "lower bound"?**
2. **What are the two terms of the ELBO and what does each encourage?**
3. **Why is there tension between reconstruction and KL divergence?**
4. **Write the closed-form KL divergence between N(μ,σ²) and N(0,1).**
5. **How does maximizing the ELBO relate to maximizing log P(x)?**

---

[Previous: 02-latent-variable-models.md](02-latent-variable-models.md) | [Next: 04-reparameterization-trick.md](04-reparameterization-trick.md)
