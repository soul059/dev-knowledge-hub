# Flow-Based Models Concept

## Overview

Flow-based generative models learn an **invertible transformation** between a simple distribution (e.g., Gaussian) and the complex data distribution. Unlike VAEs (approximate posterior) and GANs (no explicit density), flow models provide **exact likelihood computation** and **exact inference**. The "flow" refers to a chain of invertible transformations that "flow" a simple distribution into a complex one.

---

## Core Idea

```
Goal: Model p(x) where x = real data (images, audio, etc.)

  Simple distribution z ~ N(0, I)
  Complex distribution x ~ p_data(x)

  Learn invertible function f such that:
    x = f(z)     (generation: sample z, compute x)
    z = f⁻¹(x)   (inference: given x, compute z)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  z ~ N(0, I)          x ~ p_data(x)              │
  │  ┌──────────┐         ┌──────────┐               │
  │  │  Simple   │  ──f──→ │  Complex │  Generation  │
  │  │  Gaussian │ ←f⁻¹── │  Data    │  Inference   │
  │  └──────────┘         └──────────┘               │
  │                                                    │
  │  Key: f must be INVERTIBLE                        │
  │  AND: Jacobian determinant must be computable     │
  └──────────────────────────────────────────────────┘
```

---

## Change of Variables Formula

```
If x = f(z) and z = f⁻¹(x), then:

  p(x) = p_z(f⁻¹(x)) × |det(∂f⁻¹/∂x)|

  In log form:
  log p(x) = log p_z(f⁻¹(x)) + log |det(∂f⁻¹/∂x)|

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  log p(x) = log p_z(z) + log |det J_{f⁻¹}|      │
  │              ───────────   ──────────────────     │
  │              Base density   Volume change          │
  │              (Gaussian)     (how f stretches space)│
  │                                                    │
  │  Training: maximize log p(x) over data            │
  │  = maximize log p_z(f⁻¹(x)) + log |det J|       │
  │                                                    │
  │  We need:                                         │
  │  1. f⁻¹ computable (to map x → z)               │
  │  2. det J computable (to evaluate likelihood)     │
  │  3. Both must be efficient!                       │
  └──────────────────────────────────────────────────┘
```

---

## Flow as Composition

```
Instead of one complex f, compose simple invertible functions:

  f = f_K ∘ f_{K-1} ∘ ... ∘ f_2 ∘ f_1

  Each f_i is a simple invertible transform (a "flow step")
  
  z₀ → f₁ → z₁ → f₂ → z₂ → ... → f_K → x

  Log-likelihood decomposes:
  log p(x) = log p_z(z₀) + Σ log |det J_{f_i}|

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  z₀ ──f₁──→ z₁ ──f₂──→ z₂ ──f₃──→ ... ──→ x   │
  │  N(0,I)                                    data   │
  │                                                    │
  │  Each step slightly reshapes the distribution:   │
  │                                                    │
  │  Step 0: ○  (Gaussian)                           │
  │  Step 1: ◇  (slightly deformed)                  │
  │  Step 2: ◈  (more complex)                       │
  │  Step K: ✦  (matches data!)                      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Comparison with Other Generative Models

```
  Feature          Flow        VAE         GAN
  ─────────        ────        ───         ───
  Exact likelihood ✓           ✗ (ELBO)    ✗
  Exact inference  ✓           ✗ (approx)  ✗
  Mode coverage    ✓           ✓           ✗ (collapse)
  Sample quality   Good        OK          Best
  Training         Stable      Stable      Unstable
  Latent space     Meaningful  Meaningful  Limited
  Architecture     Restricted  Flexible    Flexible
  Speed (train)    Moderate    Fast        Slow
  Speed (sample)   Fast        Fast        Fast

  Flow advantages:
    • Exact log-likelihood → proper density estimation
    • Exact inference → meaningful latent codes
    • Stable training → no mode collapse
    • Invertible → can go both directions

  Flow disadvantages:
    • Architecture constraints (must be invertible!)
    • Large models (bijective → same dimensionality)
    • Generally lower quality than GANs
```

---

## Types of Flows

```
  Autoregressive flows:
    Each dimension depends on previous ones
    Slow generation, fast density evaluation
    Examples: MAF, IAF

  Coupling flows:
    Split input, transform one part conditioned on other
    Fast in both directions!
    Examples: RealNVP, NICE, Glow

  Residual flows:
    f(x) = x + g(x) where g is contractive
    More flexible but need Jacobian approximation
    Examples: i-ResNet, FFJORD

  Continuous flows:
    Model transformation as ODE
    Very flexible
    Examples: Neural ODE, FFJORD
```

---

## Python: Simple Flow Concept

```python
import torch
import torch.nn as nn

class SimpleAffineFlow(nn.Module):
    """Most basic flow: affine transformation z = (x - b) / a."""
    def __init__(self, dim):
        super().__init__()
        self.log_scale = nn.Parameter(torch.zeros(dim))
        self.bias = nn.Parameter(torch.zeros(dim))
    
    def forward(self, x):
        """x → z (inference direction)."""
        z = (x - self.bias) * torch.exp(-self.log_scale)
        log_det = -self.log_scale.sum()
        return z, log_det
    
    def inverse(self, z):
        """z → x (generation direction)."""
        x = z * torch.exp(self.log_scale) + self.bias
        return x

class ComposedFlow(nn.Module):
    """Chain of flow transformations."""
    def __init__(self, flows):
        super().__init__()
        self.flows = nn.ModuleList(flows)
    
    def forward(self, x):
        log_det_total = 0
        z = x
        for flow in self.flows:
            z, log_det = flow(z)
            log_det_total += log_det
        return z, log_det_total
    
    def inverse(self, z):
        x = z
        for flow in reversed(self.flows):
            x = flow.inverse(x)
        return x
    
    def log_prob(self, x):
        z, log_det = self.forward(x)
        log_pz = -0.5 * (z ** 2 + torch.log(torch.tensor(2 * 3.14159)))
        log_pz = log_pz.sum(dim=-1)
        return log_pz + log_det

# Training
flow = ComposedFlow([SimpleAffineFlow(2) for _ in range(5)])
optimizer = torch.optim.Adam(flow.parameters(), lr=1e-3)

for epoch in range(1000):
    x = sample_data(batch_size=256)
    loss = -flow.log_prob(x).mean()  # Negative log-likelihood
    optimizer.zero_grad(); loss.backward(); optimizer.step()

# Generation
z = torch.randn(100, 2)
x_generated = flow.inverse(z)
```

---

## Revision Questions

1. **What makes flow models unique compared to VAEs and GANs?**
2. **What is the change of variables formula and why is it important?**
3. **Why must flow transformations be invertible?**
4. **Why is the Jacobian determinant needed for likelihood computation?**
5. **What are the main categories of normalizing flows?**

---

[Previous: ../05-GAN-Variants/07-pix2pix.md](../05-GAN-Variants/07-pix2pix.md) | [Next: 02-change-of-variables.md](02-change-of-variables.md)
