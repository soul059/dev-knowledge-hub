# GAN Loss Functions

## Overview

The choice of loss function critically affects GAN training stability and output quality. Beyond the original minimax loss, researchers have developed several alternatives — non-saturating loss, least squares loss, Wasserstein loss, and hinge loss — each addressing specific training pathologies. Understanding these losses explains why some GANs train much more stably than others.

---

## Original Minimax Loss

```
V(D, G) = E_x[log D(x)] + E_z[log(1 - D(G(z)))]

  D maximizes V (wants D(real)=1, D(fake)=0)
  G minimizes V (wants D(fake)=1)

  D loss: L_D = -E[log D(x)] - E[log(1 - D(G(z)))]
  G loss: L_G = E[log(1 - D(G(z)))]

  Problem: G loss saturates when D is confident
    D(G(z)) ≈ 0  →  log(1 - 0) = log(1) = 0
    Gradient ≈ 0 → G can't learn!
    
    ┌──────────────────────────────────────┐
    │ log(1 - D(G(z)))                      │
    │  0 ─────────╲                         │
    │              ╲                        │
    │               ╲                       │
    │ -∞             ╲                      │
    │    0          0.5          1           │
    │              D(G(z))                  │
    │                                        │
    │ Gradient near 0 is FLAT → no learning │
    └──────────────────────────────────────┘
```

---

## Non-Saturating Loss

```
Replace G's objective:

  Original:        L_G = E[log(1 - D(G(z)))]    (saturates!)
  Non-saturating:  L_G = -E[log D(G(z))]         (strong gradient!)

  Same fixed point, different gradient behavior:
    
    D(G(z)) ≈ 0:
      Original:       ∂/∂θ log(1-0) ≈ 0  (vanishing!)
      Non-saturating:  ∂/∂θ -log(0) → ∞  (strong signal!)

  ┌────────────────────────────────────────────────┐
  │ -log(D(G(z)))                                   │
  │  ∞ ╲                                            │
  │     ╲                                           │
  │      ╲                                          │
  │       ╲                                         │
  │  0     ─────────                                │
  │    0          0.5          1                     │
  │              D(G(z))                            │
  │                                                  │
  │  Strong gradient when D(G(z)) ≈ 0 (early train)│
  │  This is what most implementations use!         │
  └────────────────────────────────────────────────┘

  In PyTorch:
    g_loss = F.binary_cross_entropy(D(G(z)), ones)
    # Equivalent to -log(D(G(z)))
```

---

## Least Squares Loss (LSGAN)

```
Replace log with squared error:

  L_D = E[(D(x) - 1)²] + E[D(G(z))²]
  L_G = E[(D(G(z)) - 1)²]

  Advantages:
    • No log → no vanishing gradients
    • Penalizes samples far from decision boundary
    • More stable training
    • Higher quality images

  ┌──────────────────────────────────────────────────┐
  │ BCE loss:  Correct (D=1) → loss=0, done!        │
  │            No incentive to move further          │
  │                                                    │
  │ LSQ loss:  Even correct samples → penalty if     │
  │            they're far from decision boundary     │
  │            Pushes fakes TOWARD real, continuously │
  └──────────────────────────────────────────────────┘
```

---

## Wasserstein Loss (WGAN)

```
Uses Wasserstein distance (Earth Mover's Distance):

  L_D = E[D(G(z))] - E[D(x)]     (critic, not discriminator)
  L_G = -E[D(G(z))]

  D is now a "critic" (no sigmoid, unbounded output)
  Must satisfy Lipschitz constraint: ||∇D|| ≤ 1

  Enforcement:
    WGAN:    Weight clipping (clip D weights to [-c, c])
    WGAN-GP: Gradient penalty (preferred):
      L_D += λ × E[(||∇D(x̂)||₂ - 1)²]
      x̂ = random interpolation between real and fake

  Advantages:
    • Meaningful loss (correlates with sample quality!)
    • No mode collapse
    • No need to balance G and D carefully
    • Stable training

  ┌──────────────────────────────────────────────────┐
  │  Standard GAN: D loss oscillates, meaningless    │
  │  WGAN:         D loss correlates with quality!   │
  │                                                    │
  │  Wasserstein distance = "how much work to        │
  │  move one distribution to match another"          │
  │                                                    │
  │  W(P_data, P_G) → 0 as P_G → P_data             │
  └──────────────────────────────────────────────────┘
```

---

## Hinge Loss

```
  L_D = E[max(0, 1 - D(x))] + E[max(0, 1 + D(G(z)))]
  L_G = -E[D(G(z))]

  Used in SAGAN, BigGAN, StyleGAN
  
  D only penalized if:
    D(real) < 1 or D(fake) > -1
  Once correctly classified with margin → no more gradient
  
  Prevents D from becoming too confident
  Very stable in practice
```

---

## Loss Comparison

```
  Loss Type     D Output    G Gradient     Stability   Quality
  ─────────     ────────    ──────────     ─────────   ───────
  Minimax       Sigmoid     Saturates!     Poor        OK
  Non-saturating Sigmoid    Strong early   Better      Good
  Least Squares Linear      Always exists  Good        Good
  Wasserstein   Linear      Meaningful     Very good   Good
  WGAN-GP       Linear      Smooth         Very good   Very good
  Hinge         Linear      Clean          Excellent   Excellent
```

---

## Python: Multiple Loss Functions

```python
import torch
import torch.nn.functional as F

def vanilla_d_loss(d_real, d_fake):
    return F.binary_cross_entropy(d_real, torch.ones_like(d_real)) + \
           F.binary_cross_entropy(d_fake, torch.zeros_like(d_fake))

def vanilla_g_loss(d_fake):
    return F.binary_cross_entropy(d_fake, torch.ones_like(d_fake))

def lsgan_d_loss(d_real, d_fake):
    return ((d_real - 1) ** 2).mean() + (d_fake ** 2).mean()

def lsgan_g_loss(d_fake):
    return ((d_fake - 1) ** 2).mean()

def wgan_d_loss(d_real, d_fake):
    return d_fake.mean() - d_real.mean()

def wgan_g_loss(d_fake):
    return -d_fake.mean()

def hinge_d_loss(d_real, d_fake):
    return F.relu(1 - d_real).mean() + F.relu(1 + d_fake).mean()

def hinge_g_loss(d_fake):
    return -d_fake.mean()

def gradient_penalty(D, real, fake, device):
    """WGAN-GP gradient penalty."""
    alpha = torch.rand(real.size(0), 1, 1, 1, device=device)
    interpolated = (alpha * real + (1 - alpha) * fake).requires_grad_(True)
    d_out = D(interpolated)
    grads = torch.autograd.grad(d_out, interpolated,
        grad_outputs=torch.ones_like(d_out),
        create_graph=True)[0]
    gp = ((grads.norm(2, dim=1) - 1) ** 2).mean()
    return gp
```

---

## Revision Questions

1. **Why does the original minimax loss cause vanishing gradients for G?**
2. **How does the non-saturating loss fix the gradient problem?**
3. **What advantage does Wasserstein loss have over BCE loss?**
4. **What is the gradient penalty in WGAN-GP and why is it needed?**
5. **Why is hinge loss popular in modern GAN architectures?**

---

[Previous: 03-adversarial-training.md](03-adversarial-training.md) | [Next: 05-training-challenges.md](05-training-challenges.md)
