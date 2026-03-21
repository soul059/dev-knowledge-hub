# GAN Concept

## Overview

**Generative Adversarial Networks (GANs)** use a game-theoretic framework where two neural networks compete: a **Generator** that creates fake data and a **Discriminator** that tries to distinguish real from fake. Through this adversarial training, the generator learns to produce increasingly realistic outputs until the discriminator can no longer tell the difference. Introduced by Ian Goodfellow in 2014, GANs revolutionized generative modeling.

---

## The Adversarial Game

```
  ┌──────────────────────────────────────────────────────┐
  │                                                        │
  │  Counterfeiter (Generator) vs Detective (Discriminator)│
  │                                                        │
  │  Generator G:                                         │
  │    Takes random noise z ~ N(0,1)                      │
  │    Produces fake data: x_fake = G(z)                  │
  │    Goal: fool the discriminator                       │
  │                                                        │
  │  Discriminator D:                                     │
  │    Takes data (real OR fake)                          │
  │    Outputs probability: D(x) → [0, 1]                │
  │    Goal: correctly classify real vs fake              │
  │                                                        │
  │  z ~ N(0,1)  →  [G] → x_fake ──┐                    │
  │                                   ├→ [D] → real/fake │
  │  x_real (from data) ────────────┘                    │
  │                                                        │
  │  Training alternates:                                 │
  │    1. Train D to better distinguish                   │
  │    2. Train G to better fool D                        │
  │    3. Repeat until equilibrium                        │
  └──────────────────────────────────────────────────────┘
```

---

## Minimax Objective

```
GAN training is a two-player minimax game:

  min_G max_D  V(D, G) = E_x~pdata[log D(x)] + E_z~pz[log(1 - D(G(z)))]

  Discriminator maximizes V:
    max_D  E[log D(x_real)] + E[log(1 - D(x_fake))]
    → D(real) → 1  (correctly identify real)
    → D(fake) → 0  (correctly identify fake)

  Generator minimizes V:
    min_G  E[log(1 - D(G(z)))]
    → D(G(z)) → 1  (fool discriminator)

  At equilibrium (Nash equilibrium):
    G generates data indistinguishable from real
    D(x) = 0.5 for all x (can't tell the difference)
    
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Optimal D*(x) = P_data(x) / (P_data(x) + P_G(x))│
  │                                                    │
  │  When P_G = P_data:  D*(x) = 0.5 everywhere      │
  │                                                    │
  │  The generator has perfectly learned the           │
  │  data distribution!                                │
  └──────────────────────────────────────────────────┘
```

---

## Training Process

```
For each training iteration:

  Step 1: TRAIN DISCRIMINATOR (k steps, usually k=1)
    Sample minibatch of real data: {x₁, ..., xₘ}
    Sample noise: {z₁, ..., zₘ}
    Generate fakes: {G(z₁), ..., G(zₘ)}
    
    Update D to maximize:
      (1/m) Σ [log D(xᵢ) + log(1 - D(G(zᵢ)))]

  Step 2: TRAIN GENERATOR
    Sample noise: {z₁, ..., zₘ}
    
    Update G to minimize:
      (1/m) Σ log(1 - D(G(zᵢ)))
    
    In practice, maximize log D(G(z)) instead (stronger gradient):
      (1/m) Σ log D(G(zᵢ))

  ┌──────────────────────────────────────────────────┐
  │ Why maximize log D(G(z)) instead of              │
  │ minimize log(1 - D(G(z)))?                       │
  │                                                    │
  │ Early training: G is bad → D(G(z)) ≈ 0           │
  │   log(1 - 0) = log(1) = 0  → gradient ≈ 0 !!!  │
  │   log(0) = -∞ → saturates                        │
  │                                                    │
  │ Non-saturating loss: -log D(G(z))                │
  │   -log(0) = +∞ → strong gradient early!          │
  │   Same fixed point, better gradients              │
  └──────────────────────────────────────────────────┘
```

---

## GAN vs Other Models

```
  Aspect          VAE              GAN
  ──────          ───              ───
  Training        ELBO (stable)    Adversarial (tricky)
  Output quality  Blurry           Sharp
  Likelihood      Approximate      None
  Mode coverage   Good             Can miss modes
  Latent space    Structured       Less structured
  Sampling speed  Fast             Fast
  Theory          Well-grounded    Game theory

  GAN advantage: SHARP, high-quality outputs
  GAN disadvantage: training instability, mode collapse
```

---

## Python: Minimal GAN

```python
import torch
import torch.nn as nn

class Generator(nn.Module):
    def __init__(self, latent_dim=100, output_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(latent_dim, 256), nn.LeakyReLU(0.2),
            nn.Linear(256, 512), nn.LeakyReLU(0.2),
            nn.Linear(512, output_dim), nn.Tanh())
    
    def forward(self, z):
        return self.net(z)

class Discriminator(nn.Module):
    def __init__(self, input_dim=784):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 512), nn.LeakyReLU(0.2),
            nn.Linear(512, 256), nn.LeakyReLU(0.2),
            nn.Linear(256, 1), nn.Sigmoid())
    
    def forward(self, x):
        return self.net(x)

# Training sketch
G = Generator()
D = Discriminator()
opt_G = torch.optim.Adam(G.parameters(), lr=2e-4, betas=(0.5, 0.999))
opt_D = torch.optim.Adam(D.parameters(), lr=2e-4, betas=(0.5, 0.999))
bce = nn.BCELoss()

for epoch in range(100):
    for real_data, _ in dataloader:
        real = real_data.view(-1, 784)
        batch = real.size(0)
        ones = torch.ones(batch, 1)
        zeros = torch.zeros(batch, 1)
        
        # Train Discriminator
        z = torch.randn(batch, 100)
        fake = G(z).detach()
        d_loss = bce(D(real), ones) + bce(D(fake), zeros)
        opt_D.zero_grad(); d_loss.backward(); opt_D.step()
        
        # Train Generator
        z = torch.randn(batch, 100)
        fake = G(z)
        g_loss = bce(D(fake), ones)  # fool D
        opt_G.zero_grad(); g_loss.backward(); opt_G.step()
```

---

## Revision Questions

1. **What are the roles of the generator and discriminator in a GAN?**
2. **What is the minimax objective and what happens at Nash equilibrium?**
3. **Why is the non-saturating generator loss preferred in practice?**
4. **How does GAN image quality compare to VAE?**
5. **What is the core training loop (alternating D and G updates)?**

---

[Next: 02-generator-and-discriminator.md](02-generator-and-discriminator.md) | [Back to README](../README.md)
