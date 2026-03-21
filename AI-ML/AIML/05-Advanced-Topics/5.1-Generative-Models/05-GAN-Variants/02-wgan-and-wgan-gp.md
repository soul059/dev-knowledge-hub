# WGAN and WGAN-GP

## Overview

Wasserstein GAN (Arjovsky et al., 2017) revolutionized GAN training by replacing the Jensen-Shannon divergence with the **Wasserstein distance** (Earth Mover's Distance). This provides meaningful loss values that correlate with generation quality and eliminates mode collapse. WGAN-GP (Gulrajani et al., 2017) improved upon WGAN by replacing weight clipping with a **gradient penalty**, resulting in even more stable training.

---

## The Problem with Standard GANs

```
Standard GAN minimizes JS divergence: JSD(P_data || P_G)

  Problem: When P_data and P_G have non-overlapping supports
  (common in early training when G is poor):

  JSD = log(2) = constant!
  ∂JSD/∂θ = 0  →  no gradient!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  P_data        P_G         (no overlap)           │
  │   ┌──┐        ┌──┐                               │
  │   │  │        │  │                                │
  │   │  │        │  │   JSD = log(2) (constant)     │
  │   └──┘        └──┘   Gradient = 0!              │
  │   ─────────────────                               │
  │                                                    │
  │  Wasserstein distance is ALWAYS meaningful:       │
  │  W(P_data, P_G) = distance between distributions │
  │  Even with no overlap → finite, useful gradient  │
  └──────────────────────────────────────────────────┘
```

---

## Wasserstein Distance (Earth Mover's Distance)

```
Intuition: minimum "work" to transform P_G into P_data

  Imagine P_data and P_G as piles of dirt:
  W = minimum amount of dirt × distance to move

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  P_data:  ████         P_G:       ████           │
  │           ████                    ████            │
  │                                                    │
  │  W = amount of dirt × distance moved              │
  │                                                    │
  │  Unlike KL/JS: W is continuous and differentiable│
  │  even when distributions don't overlap!           │
  └──────────────────────────────────────────────────┘

  Mathematical form (Kantorovich-Rubinstein duality):

  W(P_data, P_G) = sup      E[f(x)] - E[f(G(z))]
                   ||f||_L ≤ 1

  f must be 1-Lipschitz: |f(x₁) - f(x₂)| ≤ |x₁ - x₂|
  
  f is the "critic" (replaces discriminator)
```

---

## WGAN Formulation

```
Key changes from standard GAN:

  1. D → Critic (no sigmoid, outputs any real number)
  2. Loss = E[f(G(z))] - E[f(x)]  (critic maximizes)
  3. G loss = -E[f(G(z))]
  4. Enforce Lipschitz via weight clipping

  Critic loss:  L_C = E[C(G(z))] - E[C(x)]    (minimize)
  G loss:       L_G = -E[C(G(z))]               (minimize)

  Weight clipping:
    After each update: clip C weights to [-c, c]
    Typical c = 0.01

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Standard GAN:           WGAN:                    │
  │  D outputs [0, 1]       C outputs ℝ             │
  │  BCE loss               Wasserstein loss          │
  │  D/G balanced           C trained more (5:1)     │
  │  Loss = meaningless     Loss = quality metric!   │
  │  Mode collapse          No mode collapse!        │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Problems with Weight Clipping

```
Weight clipping enforces Lipschitz but has issues:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Problem 1: Capacity underuse                    │
  │    Weights pushed to ±c (extremes)               │
  │    Critic becomes too simple                     │
  │    Can't model complex functions                 │
  │                                                    │
  │  Problem 2: Exploding/vanishing gradients        │
  │    c too large → exploding                       │
  │    c too small → vanishing                       │
  │    Sensitive to c choice                         │
  │                                                    │
  │  Problem 3: Slow convergence                     │
  │    Clipping limits expressive power              │
  │    Need many critic updates (5-10 per G step)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## WGAN-GP: Gradient Penalty

```
Replace weight clipping with gradient penalty:

  L_C = E[C(G(z))] - E[C(x)] + λ × GP

  GP = E[(||∇_x̂ C(x̂)||₂ - 1)²]

  x̂ = ε × x_real + (1 - ε) × x_fake,   ε ~ U(0,1)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  x_real ●────────────────● x_fake                │
  │              │                                     │
  │              ● x̂ (interpolated)                   │
  │              │                                     │
  │         ∇C(x̂) should have norm = 1               │
  │         If ||∇C(x̂)|| ≠ 1, penalty!              │
  │                                                    │
  │  This softly enforces 1-Lipschitz                │
  │  Much better than hard weight clipping!          │
  │                                                    │
  └──────────────────────────────────────────────────┘

  λ = 10 (standard, rarely needs tuning)

  IMPORTANT: Cannot use BatchNorm with GP!
  (BN couples samples in batch → invalid gradient)
  Use LayerNorm or InstanceNorm instead
```

---

## WGAN vs WGAN-GP Comparison

```
  Feature            WGAN            WGAN-GP
  ──────────         ────────        ────────
  Lipschitz method   Weight clip     Gradient penalty
  Optimizer          RMSProp         Adam
  BatchNorm in C     Allowed         NOT allowed
  Critic updates     5 per G         5 per G (or fewer)
  Stability          Good            Very good
  Compute cost       Low             Higher (grad compute)
  Quality            Good            Better
  Hyperparameter     c (sensitive)   λ=10 (robust)
```

---

## Python: WGAN-GP Implementation

```python
import torch
import torch.nn as nn

def compute_gradient_penalty(critic, real, fake, device):
    batch_size = real.size(0)
    epsilon = torch.rand(batch_size, 1, 1, 1, device=device)
    interpolated = (epsilon * real + (1 - epsilon) * fake).requires_grad_(True)
    
    critic_interp = critic(interpolated)
    
    gradients = torch.autograd.grad(
        outputs=critic_interp,
        inputs=interpolated,
        grad_outputs=torch.ones_like(critic_interp),
        create_graph=True,
        retain_graph=True
    )[0]
    
    gradients = gradients.view(batch_size, -1)
    gradient_norm = gradients.norm(2, dim=1)
    penalty = ((gradient_norm - 1) ** 2).mean()
    return penalty

class WGANCritic(nn.Module):
    """Critic with LayerNorm (no BatchNorm for GP compatibility)."""
    def __init__(self, channels=3, features=64):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(channels, features, 4, 2, 1),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features, features*2, 4, 2, 1),
            nn.InstanceNorm2d(features*2, affine=True),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features*2, features*4, 4, 2, 1),
            nn.InstanceNorm2d(features*4, affine=True),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features*4, 1, 4, 1, 0),
        )
    def forward(self, x):
        return self.net(x).view(-1)

def train_wgan_gp(G, C, dataloader, latent_dim, device,
                   epochs=100, n_critic=5, lambda_gp=10):
    opt_G = torch.optim.Adam(G.parameters(), lr=1e-4, betas=(0.0, 0.9))
    opt_C = torch.optim.Adam(C.parameters(), lr=1e-4, betas=(0.0, 0.9))
    
    for epoch in range(epochs):
        for i, (real, _) in enumerate(dataloader):
            real = real.to(device)
            bs = real.size(0)
            
            # Train Critic (n_critic times per G step)
            for _ in range(n_critic):
                z = torch.randn(bs, latent_dim, device=device)
                fake = G(z).detach()
                
                c_real = C(real).mean()
                c_fake = C(fake).mean()
                gp = compute_gradient_penalty(C, real, fake, device)
                
                c_loss = c_fake - c_real + lambda_gp * gp
                opt_C.zero_grad(); c_loss.backward(); opt_C.step()
            
            # Train Generator
            z = torch.randn(bs, latent_dim, device=device)
            fake = G(z)
            g_loss = -C(fake).mean()
            opt_G.zero_grad(); g_loss.backward(); opt_G.step()
        
        w_dist = c_real.item() - c_fake.item()
        print(f"Epoch {epoch}: W_dist={w_dist:.4f}, C_loss={c_loss:.4f}")
```

---

## Revision Questions

1. **Why does JS divergence fail when distributions don't overlap?**
2. **What is the Earth Mover's Distance and why is it better for GANs?**
3. **What is the Lipschitz constraint and why is it needed?**
4. **Why is gradient penalty better than weight clipping?**
5. **Why can't you use BatchNorm with gradient penalty?**
6. **Why does the WGAN loss correlate with generation quality?**

---

[Previous: 01-dcgan.md](01-dcgan.md) | [Next: 03-conditional-gan.md](03-conditional-gan.md)
