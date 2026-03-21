# RealNVP

## Overview

RealNVP (Real-valued Non-Volume Preserving, Dinh et al., 2017) is a powerful and practical normalizing flow model that uses **affine coupling layers** to build expressive, efficiently invertible transformations. Building on NICE (which used additive coupling), RealNVP adds scaling to coupling layers, uses multi-scale architecture with squeeze operations, and achieves competitive image generation quality with exact likelihood computation.

---

## Architecture

```
RealNVP consists of:
  1. Affine coupling layers (with alternating masks)
  2. Squeeze operations (spatial → channel)
  3. Multi-scale architecture (early output of some dimensions)
  4. Batch normalization (special invertible version)

  Full pipeline:

  Input x (3, 32, 32)
    │
    ├─── Squeeze ───→ (12, 16, 16)
    │    │
    │    ├── Coupling × 3 (with checkerboard masks)
    │    │
    │    ├── Squeeze ───→ (48, 8, 8)
    │    │    │
    │    │    ├── Coupling × 3 (with channel masks)
    │    │    │
    │    │    ├── Split: z₂ → output (early!)
    │    │    │          remaining → continue
    │    │    │
    │    │    ├── Squeeze ───→ (96, 4, 4)
    │    │         │
    │    │         ├── Coupling × 3
    │    │         │
    │    │         └── z₃ → output
    │    │
    │    └── z₁ → output
    │
    └── z = [z₁, z₂, z₃]  (multi-scale latent)

  Each z_i models different scale of features!
```

---

## Affine Coupling Layer Details

```
  Checkerboard masking (spatial pattern):
  
    ┌─┬─┬─┬─┐
    │0│1│0│1│    0 = pass through
    │1│0│1│0│    1 = transform
    │0│1│0│1│
    │1│0│1│0│
    └─┴─┴─┴─┘
    
  Channel masking (after squeeze):
  
    Channels [1..C/2] = pass through
    Channels [C/2+1..C] = transform
    
  Alternate between:
    Mask type A: first half unchanged
    Mask type B: second half unchanged

  Neural networks s() and t():
    • ResNet blocks with Conv2d
    • BatchNorm + ReLU
    • Typically 4-8 residual blocks per coupling layer
    • Output same spatial size as input half
```

---

## Multi-Scale Architecture

```
Key idea: not all dimensions need the same processing depth

  Early dimensions (z₁): capture FINE details
    → factored out early, modeled by simple Gaussian
    → reduces computation for later layers
    
  Late dimensions (z₃): capture COARSE structure
    → processed by more coupling layers
    → more expressive modeling

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Benefits of multi-scale:                        │
  │                                                    │
  │  1. Computation: fewer dimensions in later layers│
  │     Total params ≈ 50% of full-depth model       │
  │                                                    │
  │  2. Modeling: different scales = different z's    │
  │     Fine texture = z₁  (simple distribution ok)  │
  │     Structure    = z₃  (needs more processing)   │
  │                                                    │
  │  3. Memory: can factor out dimensions early      │
  │     Less memory needed for deeper layers         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Training

```
Training objective: maximize log-likelihood

  log p(x) = log p_z(f⁻¹(x)) + Σ log|det J_i|

  For RealNVP with multi-scale:
  log p(x) = Σ_k [log p(z_k)] + Σ_i [log|det J_i|]

  Each z_k contributes independently!

  Dequantization:
    Images are discrete (0-255)
    Add uniform noise: x' = (x + U(0,1)) / 256
    Converts discrete → continuous for proper density

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Training details:                                │
  │                                                    │
  │  • Adam optimizer, lr = 1e-3                     │
  │  • Batch size 64                                 │
  │  • Weight decay 5e-5                             │
  │  • L2 regularization on s output                 │
  │  • Gradient clipping                             │
  │                                                    │
  │  No adversarial training needed!                 │
  │  Pure maximum likelihood → stable training       │
  └──────────────────────────────────────────────────┘
```

---

## Python: RealNVP for Images

```python
import torch
import torch.nn as nn

class ResNetBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(channels, channels, 3, 1, 1),
            nn.BatchNorm2d(channels),
            nn.ReLU(),
            nn.Conv2d(channels, channels, 3, 1, 1),
            nn.BatchNorm2d(channels),
        )
        self.relu = nn.ReLU()
    
    def forward(self, x):
        return self.relu(x + self.block(x))

class CouplingNetwork(nn.Module):
    """Neural network for s and t in coupling layer."""
    def __init__(self, in_channels, hidden_channels=64, n_blocks=4):
        super().__init__()
        layers = [nn.Conv2d(in_channels, hidden_channels, 3, 1, 1), nn.ReLU()]
        for _ in range(n_blocks):
            layers.append(ResNetBlock(hidden_channels))
        layers.append(nn.Conv2d(hidden_channels, in_channels * 2, 3, 1, 1))
        self.net = nn.Sequential(*layers)
    
    def forward(self, x):
        st = self.net(x)
        s, t = st.chunk(2, dim=1)
        s = torch.tanh(s) * 2  # bound scale
        return s, t

class RealNVPCouplingLayer(nn.Module):
    def __init__(self, channels, mask_type='channel'):
        super().__init__()
        half_ch = channels // 2
        self.net = CouplingNetwork(half_ch)
        self.mask_type = mask_type
    
    def forward(self, x):
        """Forward: x → z."""
        if self.mask_type == 'channel':
            x1, x2 = x.chunk(2, dim=1)
        else:
            x1, x2 = self._checkerboard_split(x)
        
        s, t = self.net(x1)
        z1 = x1
        z2 = x2 * torch.exp(s) + t
        log_det = s.sum(dim=[1, 2, 3])
        
        return torch.cat([z1, z2], dim=1), log_det
    
    def inverse(self, z):
        """Inverse: z → x."""
        z1, z2 = z.chunk(2, dim=1)
        s, t = self.net(z1)
        x1 = z1
        x2 = (z2 - t) * torch.exp(-s)
        return torch.cat([x1, x2], dim=1)

class Squeeze(nn.Module):
    """Squeeze: (C, H, W) → (4C, H/2, W/2)."""
    def forward(self, x):
        B, C, H, W = x.shape
        x = x.view(B, C, H//2, 2, W//2, 2)
        x = x.permute(0, 1, 3, 5, 2, 4).contiguous()
        return x.view(B, C*4, H//2, W//2)
    
    def inverse(self, x):
        B, C, H, W = x.shape
        x = x.view(B, C//4, 2, 2, H, W)
        x = x.permute(0, 1, 4, 2, 5, 3).contiguous()
        return x.view(B, C//4, H*2, W*2)

class RealNVP(nn.Module):
    def __init__(self, in_channels=3, n_coupling=4, n_scales=3):
        super().__init__()
        self.squeeze = Squeeze()
        self.scales = nn.ModuleList()
        channels = in_channels * 4  # after first squeeze
        
        for scale in range(n_scales):
            coupling_layers = nn.ModuleList()
            for i in range(n_coupling):
                coupling_layers.append(RealNVPCouplingLayer(channels))
            self.scales.append(coupling_layers)
            if scale < n_scales - 1:
                channels = channels * 4 // 2  # squeeze then split
    
    def forward(self, x):
        log_det_total = 0
        zs = []
        h = self.squeeze(x)
        
        for i, scale in enumerate(self.scales):
            for coupling in scale:
                h, log_det = coupling(h)
                log_det_total += log_det
            
            if i < len(self.scales) - 1:
                z_out, h = h.chunk(2, dim=1)
                zs.append(z_out)
                h = self.squeeze(h)
        
        zs.append(h)
        return zs, log_det_total
    
    def log_prob(self, x):
        zs, log_det = self.forward(x)
        log_pz = sum(-0.5 * (z ** 2).sum(dim=[1,2,3]) for z in zs)
        return log_pz + log_det
```

---

## Results

```
  Dataset      Bits/dim (lower = better)
  ────────     ────────
  CIFAR-10     3.49
  ImageNet 32  4.28
  ImageNet 64  3.98
  CelebA       3.02

  Compared to:
  • PixelCNN: 3.14 (better density, but SLOW sampling)
  • Glow:     3.35 (better than RealNVP)
  • VAE:      ~4.5 (worse density estimation)
```

---

## Revision Questions

1. **What is the difference between additive (NICE) and affine (RealNVP) coupling?**
2. **Why does RealNVP use a multi-scale architecture?**
3. **What is the purpose of checkerboard vs. channel masking?**
4. **Why is dequantization needed for image data?**
5. **How does the squeeze operation help the model?**

---

[Previous: 03-invertible-transformations.md](03-invertible-transformations.md) | [Next: 05-glow.md](05-glow.md)
