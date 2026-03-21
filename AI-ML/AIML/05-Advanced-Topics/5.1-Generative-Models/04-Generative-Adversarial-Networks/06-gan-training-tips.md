# GAN Training Tips and Best Practices

## Overview

Training GANs requires careful attention to architecture design, hyperparameter selection, and regularization techniques. This chapter compiles proven techniques — from spectral normalization and progressive growing to two-timescale updates and label smoothing — that have become standard practice for stable, high-quality GAN training.

---

## Architecture Guidelines

```
Generator:
  • Use ConvTranspose2d or Upsample+Conv2d (avoid checkerboard)
  • BatchNorm in all layers (except output)
  • ReLU activation (or LeakyReLU)
  • Tanh output (images normalized to [-1, 1])

Discriminator:
  • Use strided convolutions (no pooling)
  • Use LeakyReLU (slope 0.2), NOT ReLU
  • NO BatchNorm if using gradient penalty (use LayerNorm/InstanceNorm)
  • Spectral normalization highly recommended

  ┌─────────────────────────────────────┐
  │         DO                          │
  │  ✓ LeakyReLU in D                 │
  │  ✓ BatchNorm in G                 │
  │  ✓ Spectral norm in D             │
  │  ✓ Upsample + Conv in G           │
  │  ✓ Tanh output in G               │
  │                                     │
  │         DON'T                       │
  │  ✗ MaxPool in D                   │
  │  ✗ ReLU in D                      │
  │  ✗ Sigmoid output in D (for WGAN) │
  │  ✗ Large kernel sizes everywhere   │
  │  ✗ Deep bottleneck in early G      │
  └─────────────────────────────────────┘
```

---

## Hyperparameter Choices

```
Learning rate:
  • G: 1e-4 to 2e-4
  • D: 1e-4 to 4e-4 (can be higher than G)
  • TTUR: D learning rate > G learning rate (stabilizes!)

Optimizer:
  • Adam: β1 = 0.0 or 0.5, β2 = 0.9 or 0.999
  • β1 = 0.0 reduces momentum → less oscillation
  • WGAN often uses RMSProp

Batch size:
  • Larger = more stable (32, 64, 128)
  • But too large can reduce diversity

Latent dimension:
  • 100-512 typical
  • Too small → limited diversity
  • Too large → harder to train

D:G training ratio:
  • Standard GAN: 1:1
  • WGAN: 5:1 (train D more)
  • If D is weak, train D more
  • If D is too strong, train D less
```

---

## Spectral Normalization

```
Control Lipschitz constant of D by normalizing weight matrices:

  W_SN = W / σ(W)

  σ(W) = largest singular value of W

  Why? Constrains D to be 1-Lipschitz
  → Stable gradients for G
  → Prevents D from becoming too confident

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Without SN:  D weights grow → D too confident   │
  │               → G gradients vanish               │
  │                                                    │
  │  With SN:     D weights bounded → stable D       │
  │               → meaningful gradients for G        │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Much simpler than gradient penalty!
  Just wrap layers: nn.utils.spectral_norm(layer)
```

---

## Label Smoothing

```
Instead of:
  real label = 1.0    →    real label = 0.9
  fake label = 0.0    →    fake label = 0.0 (or 0.1)

Why? Prevents D from being 100% confident
  → Softer D outputs → better G gradients

One-sided smoothing (recommended):
  Only smooth real labels: 1.0 → [0.8, 1.0]
  Keep fake labels at 0.0

Two-sided smoothing:
  Real: 1.0 → [0.8, 1.0]
  Fake: 0.0 → [0.0, 0.2]
  (less common, can hurt)
```

---

## Instance Noise

```
Add Gaussian noise to D's inputs:

  D(x + ε)  where ε ~ N(0, σ²)

  Decreasing σ over training (annealing):
  σ_t = σ_0 × (1 - t/T)

  Why?
  • Smooths the data distribution
  • Prevents D from memorizing exact real samples
  • Overlapping supports → meaningful gradients
```

---

## Minibatch Discrimination

```
Help D detect mode collapse:

  D sees multiple samples at once
  If all generated samples are identical,
  D can detect this → penalize G

  How:
  1. Compute feature vectors for each sample
  2. Compute pairwise distances
  3. Use distance statistics as extra D input

  If all samples similar → distances small → D flags
```

---

## Two-Timescale Update Rule (TTUR)

```
Use different learning rates for G and D:

  lr_D > lr_G  (e.g., lr_D = 4e-4, lr_G = 1e-4)

  Theoretical guarantee of convergence!

  Why: D needs to "keep up" with G
  If D uses higher lr, it adapts faster
  → always provides useful gradients to G
```

---

## Exponential Moving Average (EMA)

```
Keep a running average of G's weights for generation:

  θ_EMA = α × θ_EMA + (1 - α) × θ_G

  Typical α = 0.999

  Use θ_EMA for inference, θ_G for training
  
  ┌──────────────────────────────────────────────────┐
  │ Training weights:  noisy, oscillating            │
  │ EMA weights:       smooth, stable, better quality│
  │                                                    │
  │ Nearly all modern GANs use EMA!                  │
  │ StyleGAN, BigGAN, ProjectedGAN, etc.             │
  └──────────────────────────────────────────────────┘
```

---

## Python: Complete Training with Best Practices

```python
import torch
import torch.nn as nn

class SpectralNormD(nn.Module):
    def __init__(self, channels=3, features=64):
        super().__init__()
        self.net = nn.Sequential(
            nn.utils.spectral_norm(nn.Conv2d(channels, features, 4, 2, 1)),
            nn.LeakyReLU(0.2),
            nn.utils.spectral_norm(nn.Conv2d(features, features*2, 4, 2, 1)),
            nn.LeakyReLU(0.2),
            nn.utils.spectral_norm(nn.Conv2d(features*2, features*4, 4, 2, 1)),
            nn.LeakyReLU(0.2),
            nn.utils.spectral_norm(nn.Conv2d(features*4, 1, 4, 1, 0)),
        )
    def forward(self, x):
        return self.net(x)

def ema_update(ema_model, model, decay=0.999):
    with torch.no_grad():
        for p_ema, p in zip(ema_model.parameters(), model.parameters()):
            p_ema.data.mul_(decay).add_(p.data, alpha=1 - decay)

def train_with_best_practices(G, D, G_ema, dataloader, epochs, 
                               latent_dim, device):
    opt_G = torch.optim.Adam(G.parameters(), lr=1e-4, betas=(0.0, 0.999))
    opt_D = torch.optim.Adam(D.parameters(), lr=4e-4, betas=(0.0, 0.999))
    
    for epoch in range(epochs):
        for real, _ in dataloader:
            real = real.to(device)
            bs = real.size(0)
            
            # Train D with hinge loss
            z = torch.randn(bs, latent_dim, device=device)
            fake = G(z).detach()
            d_real = D(real)
            d_fake = D(fake)
            d_loss = torch.relu(1 - d_real).mean() + torch.relu(1 + d_fake).mean()
            
            opt_D.zero_grad(); d_loss.backward(); opt_D.step()
            
            # Train G
            z = torch.randn(bs, latent_dim, device=device)
            fake = G(z)
            g_loss = -D(fake).mean()
            
            opt_G.zero_grad(); g_loss.backward(); opt_G.step()
            
            # EMA update
            ema_update(G_ema, G)
        
        print(f"Epoch {epoch}: D_loss={d_loss:.3f}, G_loss={g_loss:.3f}")
```

---

## Summary Table

| Technique | Purpose | Impact |
|-----------|---------|--------|
| Spectral Normalization | Constrain D | Stable gradients |
| Label Smoothing | Soften D confidence | Better G gradients |
| Instance Noise | Overlap distributions | Prevent memorization |
| TTUR | Faster D learning | Convergence guarantee |
| EMA | Smooth G weights | Higher quality output |
| Hinge Loss | Replace BCE | Stable training |
| Minibatch Discrimination | Detect mode collapse | Improved diversity |

---

## Revision Questions

1. **Why is spectral normalization preferred over gradient penalty in many architectures?**
2. **How does TTUR help with GAN convergence?**
3. **What is EMA and why does it improve generation quality?**
4. **Why should you use LeakyReLU instead of ReLU in the discriminator?**
5. **How does label smoothing prevent the discriminator from becoming too confident?**
6. **What causes checkerboard artifacts and how do you fix them?**

---

[Previous: 05-training-challenges.md](05-training-challenges.md) | [Next: ../05-GAN-Variants/01-dcgan.md](../05-GAN-Variants/01-dcgan.md)
