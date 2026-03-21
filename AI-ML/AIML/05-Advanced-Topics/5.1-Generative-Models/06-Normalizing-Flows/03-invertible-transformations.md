# Invertible Transformations

## Overview

The building blocks of normalizing flows are **invertible transformations** — functions that can be reversed exactly. The challenge is designing transformations that are (1) expressive enough to model complex distributions, (2) efficiently invertible, and (3) have tractable Jacobian determinants. This chapter covers the main families: coupling layers, autoregressive transforms, and 1×1 convolutions.

---

## Coupling Layers

```
The most popular building block (used in NICE, RealNVP, Glow):

  Split input into two halves: z = [z₁, z₂]

  Forward (z → x):
    x₁ = z₁                          (pass-through!)
    x₂ = z₂ ⊙ exp(s(z₁)) + t(z₁)   (affine transform)

  Inverse (x → z):
    z₁ = x₁                          (trivial!)
    z₂ = (x₂ - t(x₁)) ⊙ exp(-s(x₁)) (solve for z₂)

  s() and t() can be ANY neural network (no invertibility needed!)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Input z = [z₁ | z₂]                             │
  │             │     │                                │
  │             │     ▼                                │
  │             │   ┌─────────────────┐               │
  │             ├──→│  s(z₁), t(z₁)  │               │
  │             │   └──────┬──────────┘               │
  │             │          │ scale & shift             │
  │             │          ▼                            │
  │             │    x₂ = z₂ × exp(s) + t            │
  │             │                                      │
  │             ▼          ▼                            │
  │  Output x = [x₁     | x₂]                        │
  │                                                    │
  │  x₁ unchanged → easy to invert!                  │
  │  s, t = neural nets → powerful!                   │
  └──────────────────────────────────────────────────┘

  Jacobian:
  J = ┌  I           0          ┐
      └ ∂x₂/∂z₁    diag(exp(s))┘

  det(J) = Π exp(s_i) = exp(Σ s_i)
  log|det(J)| = Σ s_i  ← just sum the s values!
```

---

## Alternating Coupling

```
Problem: x₁ is never transformed (just copied)!

Solution: Alternate which half gets transformed:

  Layer 1: Transform z₂ using z₁  →  [z₁, f(z₂|z₁)]
  Layer 2: Transform z₁ using z₂  →  [g(z₁|z₂), z₂]
  Layer 3: Transform z₂ using z₁  →  [z₁, h(z₂|z₁)]
  ...

  After multiple layers, ALL dimensions get transformed!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Layer 1:   [A | B] → [A    | f(B,A)]           │
  │  Layer 2:   [A | B] → [g(A,B) | B   ]           │
  │  Layer 3:   [A | B] → [A    | h(B,A)]           │
  │  ...                                              │
  │                                                    │
  │  Both A and B get transformed over multiple layers│
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Autoregressive Transforms

```
Each output dimension depends on ALL previous inputs:

  x₁ = τ(z₁; θ₁)
  x₂ = τ(z₂; θ₂(x₁))
  x₃ = τ(z₃; θ₃(x₁, x₂))
  x_i = τ(z_i; θ_i(x₁, ..., x_{i-1}))

  Where τ is an invertible scalar function (e.g., affine)

  Jacobian is LOWER TRIANGULAR:
  J = ┌ ∂x₁/∂z₁    0         0      ┐
      │ ∂x₂/∂z₁    ∂x₂/∂z₂   0      │
      └ ∂x₃/∂z₁    ∂x₃/∂z₂   ∂x₃/∂z₃┘

  det(J) = Π ∂x_i/∂z_i  (product of diagonal)

  MAF (Masked Autoregressive Flow):
    Fast density evaluation (parallel)
    Slow generation (sequential — must compute x₁, then x₂, ...)

  IAF (Inverse Autoregressive Flow):
    Fast generation (parallel)
    Slow density evaluation (sequential)
```

---

## 1×1 Convolution (Invertible)

```
Used in Glow to mix channels:

  W: c × c weight matrix (c = number of channels)
  x_out = W × x_in   (per spatial location)

  Inverse: x_in = W⁻¹ × x_out

  Jacobian determinant:
    log|det(J)| = h × w × log|det(W)|
    (h, w = spatial dimensions)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Why 1×1 conv?                                   │
  │                                                    │
  │  Coupling layers only transform HALF the channels│
  │  1×1 conv MIXES all channels → better coupling   │
  │                                                    │
  │  Alternative to fixed permutation:                │
  │  • Fixed: shuffle channel order (NICE, RealNVP)  │
  │  • Learned: 1×1 conv learns optimal mixing (Glow)│
  │                                                    │
  │  LU decomposition for efficiency:                 │
  │  W = P × L × U                                  │
  │  det(W) = Π diag(U)  → O(c) instead of O(c³)   │
  └──────────────────────────────────────────────────┘
```

---

## ActNorm (Activation Normalization)

```
Replaces BatchNorm in flow models:
  (BatchNorm is not invertible during inference!)

  y = s ⊙ x + b

  s, b initialized using first batch:
    b = -mean(x)  →  zero mean
    s = 1/std(x)  →  unit variance

  After initialization: s, b are learned parameters
  
  Invertible! x = (y - b) / s
  log|det| = h × w × Σ log|s_i|
```

---

## Squeeze Operation

```
Trade spatial dimensions for channel dimensions:

  Input: (C, H, W) → Squeeze → (4C, H/2, W/2)

  ┌────────────────────────────────────────────┐
  │                                              │
  │  Before squeeze:  (3, 8, 8)                │
  │                                              │
  │  ┌──┬──┐       ┌──┬──┐                    │
  │  │a │b │   →   │a │b │c │d │              │
  │  ├──┼──┤       └──┴──┴──┴──┘              │
  │  │c │d │                                    │
  │  └──┴──┘       After: (12, 4, 4)          │
  │                                              │
  │  Each 2×2 spatial block → 4 channels       │
  │  Perfectly invertible!                      │
  │  Allows flow to operate on more channels   │
  │  at lower spatial resolution               │
  └────────────────────────────────────────────┘
```

---

## Python: Coupling Layer Implementation

```python
import torch
import torch.nn as nn

class AffineCouplingLayer(nn.Module):
    def __init__(self, dim, hidden_dim=256, mask_type='even'):
        super().__init__()
        self.dim = dim
        half = dim // 2
        
        # s and t networks (can be arbitrarily complex!)
        self.scale_net = nn.Sequential(
            nn.Linear(half, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, half),
            nn.Tanh(),  # bound scale for stability
        )
        self.translate_net = nn.Sequential(
            nn.Linear(half, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, half),
        )
    
    def forward(self, z):
        """z → x (generation direction)."""
        z1, z2 = z.chunk(2, dim=-1)
        s = self.scale_net(z1)
        t = self.translate_net(z1)
        x1 = z1
        x2 = z2 * torch.exp(s) + t
        log_det = s.sum(dim=-1)
        return torch.cat([x1, x2], dim=-1), log_det
    
    def inverse(self, x):
        """x → z (inference direction)."""
        x1, x2 = x.chunk(2, dim=-1)
        s = self.scale_net(x1)
        t = self.translate_net(x1)
        z1 = x1
        z2 = (x2 - t) * torch.exp(-s)
        log_det = -s.sum(dim=-1)
        return torch.cat([z1, z2], dim=-1), log_det

class RealNVPFlow(nn.Module):
    def __init__(self, dim, n_layers=6, hidden_dim=256):
        super().__init__()
        self.layers = nn.ModuleList()
        for i in range(n_layers):
            self.layers.append(AffineCouplingLayer(dim, hidden_dim))
    
    def forward(self, x):
        """x → z with log det."""
        log_det_total = 0
        z = x
        for i, layer in enumerate(self.layers):
            z, log_det = layer.inverse(z)
            log_det_total += log_det
            # Flip halves for alternating coupling
            z = torch.cat([z.chunk(2, dim=-1)[1],
                          z.chunk(2, dim=-1)[0]], dim=-1)
        return z, log_det_total
    
    def log_prob(self, x):
        z, log_det = self.forward(x)
        log_pz = -0.5 * (z ** 2).sum(dim=-1) - 0.5 * z.shape[-1] * 1.8379
        return log_pz + log_det
    
    def sample(self, n):
        z = torch.randn(n, self.layers[0].dim)
        x = z
        for layer in reversed(self.layers):
            x = torch.cat([x.chunk(2, dim=-1)[1],
                          x.chunk(2, dim=-1)[0]], dim=-1)
            x, _ = layer(x)
        return x
```

---

## Revision Questions

1. **How do coupling layers achieve efficient invertibility?**
2. **Why does half the input remain unchanged in a coupling layer?**
3. **How do autoregressive transforms differ from coupling layers?**
4. **What is the purpose of 1×1 convolutions in Glow?**
5. **Why can't we use BatchNorm in flow models?**
6. **What does the squeeze operation do and why is it useful?**

---

[Previous: 02-change-of-variables.md](02-change-of-variables.md) | [Next: 04-realnvp.md](04-realnvp.md)
