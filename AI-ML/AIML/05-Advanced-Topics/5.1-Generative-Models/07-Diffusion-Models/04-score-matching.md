# Score Matching

## Overview

Score matching provides an alternative theoretical framework for diffusion models. Instead of thinking about denoising, we think about learning the **score function** — the gradient of the log probability density, ∇_x log p(x). Score-based models (Song & Ermon, 2019) and the connection to diffusion models (Song et al., 2021) unified these perspectives, showing that DDPM's noise prediction is equivalent to score estimation.

---

## What is the Score Function?

```
Score = gradient of log-density with respect to data:

  s(x) = ∇_x log p(x)

  The score points toward regions of HIGH density:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  p(x):  density function                         │
  │                                                    │
  │           ╱╲                                      │
  │          ╱  ╲                                     │
  │         ╱    ╲                                    │
  │  ──────╱──────╲──────                            │
  │  ← ←  ← → →  → → →                             │
  │        ↗ ↑ ↖                                     │
  │     score vectors point toward peak!             │
  │                                                    │
  │  At the peak: ∇ log p = 0  (no gradient)        │
  │  Far from peak: ∇ log p points toward peak      │
  │                                                    │
  │  Score tells you: "which direction to go         │
  │  to find more probable data"                     │
  └──────────────────────────────────────────────────┘

  Why score instead of density?
  • p(x) requires normalization constant Z (intractable!)
    p(x) = p̃(x) / Z,  Z = ∫ p̃(x) dx
  • Score doesn't need Z:
    ∇_x log p(x) = ∇_x log p̃(x) - ∇_x log Z
                  = ∇_x log p̃(x)     (Z is constant!)
```

---

## Score Matching Objective

```
We want s_θ(x) ≈ ∇_x log p(x)

  Naive approach:
    min E[||s_θ(x) - ∇_x log p(x)||²]
    
    But we don't KNOW ∇_x log p(x)!

  Score matching trick (Hyvärinen, 2005):
    Can rewrite the objective without knowing the true score:

    J(θ) = E_data[½||s_θ(x)||² + tr(∇_x s_θ(x))]

    Only needs: s_θ(x) and its Jacobian trace
    Does NOT need the true score!

  Denoising score matching (Vincent, 2011):
    Even simpler: add noise, then match score of noisy data

    q_σ(x̃|x) = N(x̃; x, σ²I)   (add Gaussian noise)
    
    ∇_x̃ log q_σ(x̃|x) = -(x̃ - x)/σ²  (score of noisy data = known!)

    Objective: E[||s_θ(x̃) - (-(x̃-x)/σ²)||²]
             = E[||s_θ(x̃) + (x̃-x)/σ²||²]

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Connection to diffusion:                        │
  │                                                    │
  │  s_θ(x_t, t) = -ε_θ(x_t, t) / √(1-ᾱ_t)        │
  │                                                    │
  │  Predicting noise = predicting negative score!   │
  │  DDPM loss ≡ denoising score matching!           │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Noise-Conditioned Score Networks (NCSN)

```
Song & Ermon (2019): learn score at MULTIPLE noise levels

  Problem: score is inaccurate in low-density regions
  (no training data there → can't estimate score)

  Solution: add noise! Noise "fills in" low-density regions

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Original data:      Noisy data (σ large):      │
  │                                                    │
  │  ●    ●                ● ●●● ● ●                │
  │  ●●●●●●              ● ●●●●●●● ●               │
  │  ●    ●              ● ●●● ● ●● ●              │
  │                       ● ● ●●● ● ●               │
  │  (sparse, gaps)      (filled in, easier to score)│
  │                                                    │
  └──────────────────────────────────────────────────┘

  Multi-scale noise: σ₁ > σ₂ > ... > σ_L
  
  Train: s_θ(x, σ) for all noise levels
  Sample: Langevin dynamics, annealing σ from large to small
```

---

## Langevin Dynamics Sampling

```
Given the score, generate samples using Langevin MCMC:

  x_{t+1} = x_t + (η/2) × ∇_x log p(x_t) + √η × z_t
           = x_t + (η/2) × s_θ(x_t) + √η × z_t

  where z_t ~ N(0, I), η = step size

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Start: x₀ ~ random (e.g., noise)               │
  │                                                    │
  │  Each step:                                       │
  │    1. Compute score s_θ(x_t)  (direction to data)│
  │    2. Move in score direction  (toward data)     │
  │    3. Add noise               (exploration)       │
  │                                                    │
  │  After many steps: x converges to data distrib.  │
  │                                                    │
  │  Annealed Langevin dynamics:                     │
  │    Start with large σ (explore broadly)          │
  │    Reduce σ gradually (refine details)           │
  │    → coarse to fine generation                   │
  └──────────────────────────────────────────────────┘
```

---

## Score SDE (Unified Framework)

```
Song et al. (2021) unified everything with SDEs:

  Forward SDE: dx = f(x,t)dt + g(t)dw
    f = drift coefficient
    g = diffusion coefficient
    w = Wiener process (Brownian motion)

  Reverse SDE: dx = [f(x,t) - g(t)² ∇_x log p_t(x)]dt + g(t)dw̄

  The score ∇_x log p_t(x) is all you need!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Different SDEs → different models:              │
  │                                                    │
  │  VP-SDE (Variance Preserving):                   │
  │    dx = -½ β(t) x dt + √β(t) dw                │
  │    → equivalent to DDPM!                          │
  │                                                    │
  │  VE-SDE (Variance Exploding):                    │
  │    dx = √(dσ²/dt) dw                             │
  │    → equivalent to NCSN/SMLD!                    │
  │                                                    │
  │  sub-VP SDE, etc.                                │
  │                                                    │
  │  Unified framework: same theory, different SDEs  │
  └──────────────────────────────────────────────────┘
```

---

## Python: Score-Based Model

```python
import torch
import torch.nn as nn

class ScoreNetwork(nn.Module):
    """Simple score network s_θ(x, σ)."""
    def __init__(self, dim=2, hidden=256):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(dim + 1, hidden),  # +1 for noise level
            nn.SiLU(),
            nn.Linear(hidden, hidden),
            nn.SiLU(),
            nn.Linear(hidden, hidden),
            nn.SiLU(),
            nn.Linear(hidden, dim),
        )
    
    def forward(self, x, sigma):
        sigma_input = sigma.view(-1, 1) if sigma.dim() == 1 else sigma
        return self.net(torch.cat([x, sigma_input], dim=-1))

def denoising_score_matching_loss(score_net, x, sigma):
    """Denoising score matching loss."""
    noise = torch.randn_like(x)
    x_noisy = x + sigma.view(-1, 1) * noise
    
    # True score of q(x̃|x) = -(x̃ - x) / σ²
    target_score = -noise / sigma.view(-1, 1)
    
    pred_score = score_net(x_noisy, sigma)
    loss = ((pred_score - target_score) ** 2).sum(dim=-1).mean()
    return loss

@torch.no_grad()
def annealed_langevin_dynamics(score_net, sigmas, n_samples, dim,
                                 n_steps=100, step_size=1e-5):
    """Sample using annealed Langevin dynamics."""
    x = torch.randn(n_samples, dim)
    
    for sigma in sigmas:  # from large to small
        alpha = step_size * (sigma / sigmas[-1]) ** 2
        sigma_t = torch.full((n_samples,), sigma)
        
        for _ in range(n_steps):
            score = score_net(x, sigma_t)
            noise = torch.randn_like(x)
            x = x + (alpha / 2) * score + torch.sqrt(alpha) * noise
    
    return x

# Training
sigmas = torch.logspace(start=1, end=-2, steps=10)  # [10, ..., 0.01]
score_net = ScoreNetwork(dim=2)
optimizer = torch.optim.Adam(score_net.parameters(), lr=1e-3)

for step in range(10000):
    x = sample_data(256)
    sigma_idx = torch.randint(0, len(sigmas), (256,))
    sigma = sigmas[sigma_idx]
    
    loss = denoising_score_matching_loss(score_net, x, sigma)
    optimizer.zero_grad(); loss.backward(); optimizer.step()

# Sampling
samples = annealed_langevin_dynamics(score_net, sigmas, n_samples=1000, dim=2)
```

---

## Revision Questions

1. **What is the score function and why doesn't it need a normalization constant?**
2. **How does denoising score matching avoid knowing the true score?**
3. **What is the connection between noise prediction and score estimation?**
4. **Why are multiple noise levels needed for score-based models?**
5. **How does the Score SDE framework unify DDPM and NCSN?**

---

[Previous: 03-ddpm.md](03-ddpm.md) | [Next: 05-noise-scheduling.md](05-noise-scheduling.md)
