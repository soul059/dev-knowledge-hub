# StyleGAN

## Overview

StyleGAN (Karras et al., 2019) is one of the most important GAN architectures ever developed. By introducing a **mapping network** and **style-based generator** with Adaptive Instance Normalization (AdaIN), StyleGAN enables unprecedented control over generated images. It separates high-level attributes (pose, identity) from fine details (freckles, hair) through automatic style hierarchy, enabling style mixing, interpolation, and fine-grained control over generation.

---

## Architecture Overview

```
Traditional Generator:
  z ──→ G ──→ image
  (latent directly fed to network)

StyleGAN Generator:
  z ──→ Mapping Network ──→ w ──→ Style Injection ──→ image
        (8 FC layers)         (intermediate latent)

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │   z ∈ Z                                                   │
  │   │                                                        │
  │   ▼                                                        │
  │  ┌──────────────────┐                                     │
  │  │ Mapping Network   │  8 fully connected layers          │
  │  │ f: Z → W          │  z → FC → FC → ... → FC → w       │
  │  └────────┬──────────┘                                     │
  │           │ w ∈ W                                          │
  │           │                                                │
  │   ┌───────┼───────┐  w is broadcast to ALL layers         │
  │   │       │       │                                        │
  │   ▼       ▼       ▼                                        │
  │  [A]     [A]     [A]   Learned affine transforms          │
  │   │       │       │    A(w) → (y_s, y_b) for AdaIN       │
  │   ▼       ▼       ▼                                        │
  │  ┌───┐  ┌───┐  ┌───┐                                     │
  │  │4×4│→ │8×8│→ │...│→  image                             │
  │  └───┘  └───┘  └───┘                                     │
  │   ↑       ↑       ↑                                        │
  │  noise   noise   noise   Per-pixel noise injection        │
  │                                                            │
  └──────────────────────────────────────────────────────────┘
```

---

## Mapping Network

```
Why not use z directly?

  z ~ N(0, I)  — uniform probability in all directions
  
  Problem: z space must match training data distribution
  If features are correlated (e.g., age → wrinkles),
  z space can't represent this well → "entanglement"

  Solution: Learn a mapping f: Z → W
  W space can be non-linear, match data distribution

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Z space:                    W space:             │
  │  ┌────────────────┐         ┌────────────────┐   │
  │  │ ● ● ● ● ● ● ● │         │   ●●●          │   │
  │  │ ● ● ● ● ● ● ● │   f→   │  ●●●●●         │   │
  │  │ ● ● ● ● ● ● ● │         │ ●●●●●●●        │   │
  │  │ ● ● ● ● ● ● ● │         │  ●●●●●●        │   │
  │  │ ● ● ● ● ● ● ● │         │    ●●●●        │   │
  │  └────────────────┘         └────────────────┘   │
  │  Uniform (entangled)       Shaped (disentangled) │
  │                                                    │
  │  W space separates features that Z space tangles │
  └──────────────────────────────────────────────────┘
```

---

## Adaptive Instance Normalization (AdaIN)

```
Standard Instance Norm:
  IN(x) = (x - μ(x)) / σ(x)

AdaIN: Replace learned γ, β with style-dependent ones:
  AdaIN(x, w) = y_s(w) × IN(x) + y_b(w)

  y_s, y_b come from affine transform A(w)
  Each layer gets its OWN y_s, y_b from w

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  features x → InstanceNorm → × y_s + y_b → out  │
  │                                ↑       ↑          │
  │                            from w via learned A   │
  │                                                    │
  │  Different w → different normalization stats      │
  │  → different "style" at each resolution level     │
  │                                                    │
  │  Coarse layers (4×4, 8×8):  pose, shape, face    │
  │  Medium layers (16-32):     hair, expression      │
  │  Fine layers (64-1024):     color, texture, pores│
  └──────────────────────────────────────────────────┘
```

---

## Style Mixing

```
Because w is injected independently at each layer,
you can use DIFFERENT w vectors at different layers!

  w₁ from z₁ → inject at coarse layers (4×4 to 8×8)
  w₂ from z₂ → inject at fine layers (16×16 to 1024)

  Result: coarse structure from w₁ + fine details from w₂

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Source A (w₁):     Source B (w₂):    Mixed:     │
  │  ┌────────┐        ┌────────┐       ┌────────┐  │
  │  │  👨    │        │  👩    │       │ A+B    │   │
  │  │ brown  │        │ blonde │       │ A pose │   │
  │  │ hair   │        │ hair   │       │ B hair │   │
  │  └────────┘        └────────┘       └────────┘  │
  │                                                    │
  │  Coarse styles from A (identity, pose)           │
  │  Fine styles from B (hair color, texture)        │
  └──────────────────────────────────────────────────┘

  Used as regularization during training!
  Prevents adjacent layers from becoming correlated
```

---

## Noise Injection

```
Per-pixel noise for stochastic details:

  Each conv layer receives random noise B:
    output = conv_output + B × noise

  B is a learned per-channel scaling factor
  noise is random N(0,1), different each forward pass

  What noise controls:
    • Exact hair strand placement
    • Pore locations
    • Background details
    • Freckle positions

  Noise does NOT affect:
    • Identity
    • Pose
    • Expression
    (those are controlled by w!)

  This separation = better disentanglement
```

---

## StyleGAN2 Improvements

```
StyleGAN had artifacts. StyleGAN2 fixed them:

  1. Weight Demodulation (replaces AdaIN)
     AdaIN → removes/normalizes mean → blob artifacts
     Fix: modulate conv weights directly, then demodulate
     
  2. No Progressive Growing
     Replaced with skip connections + residual
     Simpler training, no phase transitions
     
  3. Path Length Regularization
     Encourages smooth W space
     Small Δw → small Δimage (locally linear)
     
  4. Lazy Regularization
     Apply regularization every 16 steps (not every step)
     Much faster, same quality

  StyleGAN2 > StyleGAN in quality, diversity, and speed
```

---

## Python: StyleGAN Components

```python
import torch
import torch.nn as nn

class MappingNetwork(nn.Module):
    def __init__(self, z_dim=512, w_dim=512, num_layers=8):
        super().__init__()
        layers = []
        for i in range(num_layers):
            layers.append(nn.Linear(z_dim if i == 0 else w_dim, w_dim))
            layers.append(nn.LeakyReLU(0.2))
        self.net = nn.Sequential(*layers)
    
    def forward(self, z):
        return self.net(z)

class AdaIN(nn.Module):
    def __init__(self, channels, w_dim=512):
        super().__init__()
        self.norm = nn.InstanceNorm2d(channels)
        self.style = nn.Linear(w_dim, channels * 2)
    
    def forward(self, x, w):
        style = self.style(w).unsqueeze(-1).unsqueeze(-1)
        gamma, beta = style.chunk(2, dim=1)
        return gamma * self.norm(x) + beta

class NoiseInjection(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.weight = nn.Parameter(torch.zeros(1, channels, 1, 1))
    
    def forward(self, x):
        noise = torch.randn(x.size(0), 1, x.size(2), x.size(3), device=x.device)
        return x + self.weight * noise

class StyleBlock(nn.Module):
    def __init__(self, in_ch, out_ch, w_dim=512):
        super().__init__()
        self.conv = nn.Conv2d(in_ch, out_ch, 3, 1, 1)
        self.noise = NoiseInjection(out_ch)
        self.adain = AdaIN(out_ch, w_dim)
        self.act = nn.LeakyReLU(0.2)
    
    def forward(self, x, w):
        x = self.conv(x)
        x = self.noise(x)
        x = self.act(x)
        x = self.adain(x, w)
        return x

class StyleGenerator(nn.Module):
    def __init__(self, z_dim=512, w_dim=512):
        super().__init__()
        self.mapping = MappingNetwork(z_dim, w_dim)
        self.const = nn.Parameter(torch.randn(1, 512, 4, 4))
        
        # Style blocks at increasing resolutions
        self.blocks = nn.ModuleList([
            StyleBlock(512, 512, w_dim),  # 4×4
            StyleBlock(512, 512, w_dim),  # 8×8
            StyleBlock(512, 256, w_dim),  # 16×16
            StyleBlock(256, 128, w_dim),  # 32×32
            StyleBlock(128, 64, w_dim),   # 64×64
        ])
        self.to_rgb = nn.Conv2d(64, 3, 1)
        self.upsample = nn.Upsample(scale_factor=2, mode='bilinear',
                                      align_corners=False)
    
    def forward(self, z, style_mixing=False):
        w = self.mapping(z)
        x = self.const.expand(z.size(0), -1, -1, -1)
        
        for i, block in enumerate(self.blocks):
            if i > 0:
                x = self.upsample(x)
            x = block(x, w)
        
        return torch.tanh(self.to_rgb(x))
```

---

## Summary Table

| Component | Purpose | Impact |
|-----------|---------|--------|
| Mapping Network | z → w disentanglement | Better control |
| AdaIN | Style injection per layer | Hierarchical styles |
| Noise Injection | Stochastic variation | Fine details |
| Style Mixing | Different w per layer | Regularization |
| Truncation Trick | Trade diversity for quality | Better FID |

---

## Revision Questions

1. **Why does StyleGAN use a mapping network instead of feeding z directly?**
2. **How does AdaIN inject style information at each layer?**
3. **What's the difference between what noise controls vs. what w controls?**
4. **How does style mixing work and why is it useful?**
5. **What were the main improvements in StyleGAN2 over StyleGAN?**

---

[Previous: 04-progressive-gan.md](04-progressive-gan.md) | [Next: 06-cyclegan.md](06-cyclegan.md)
