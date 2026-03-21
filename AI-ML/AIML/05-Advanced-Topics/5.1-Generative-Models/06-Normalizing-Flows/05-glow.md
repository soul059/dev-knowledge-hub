# Glow

## Overview

Glow (Kingma & Dhariwal, 2018) is the state-of-the-art normalizing flow model, extending RealNVP with three key innovations: **invertible 1×1 convolutions** (learned channel permutations), **ActNorm** (activation normalization replacing BatchNorm), and an improved multi-scale architecture. Glow generates high-quality 256×256 face images with exact log-likelihood, enables semantic manipulation in latent space, and demonstrates that flow models can compete with GANs.

---

## Architecture: One Step of Flow

```
Each "step of flow" in Glow has 3 components:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Input features                                   │
  │       │                                            │
  │       ▼                                            │
  │  ┌─────────────┐                                  │
  │  │   ActNorm    │  Data-dependent initialization  │
  │  └──────┬───────┘                                  │
  │         │                                          │
  │         ▼                                          │
  │  ┌─────────────┐                                  │
  │  │  1×1 Conv   │  Invertible, learned permutation │
  │  └──────┬───────┘                                  │
  │         │                                          │
  │         ▼                                          │
  │  ┌─────────────┐                                  │
  │  │  Affine      │  Coupling layer (like RealNVP) │
  │  │  Coupling    │                                  │
  │  └──────┬───────┘                                  │
  │         │                                          │
  │         ▼                                          │
  │  Output features                                  │
  │                                                    │
  └──────────────────────────────────────────────────┘

  This step is repeated K times per scale level
  Multi-scale: L levels with squeeze + split
```

---

## Invertible 1×1 Convolution

```
RealNVP: fixed channel shuffling (reverse order)
Glow:    LEARNED channel mixing via 1×1 convolution

  W: c × c weight matrix (c = channels)
  Forward:  y = W × x  (per pixel)
  Inverse:  x = W⁻¹ × y

  Log determinant:
    log|det J| = h × w × log|det W|
    (h, w = spatial dims; det W computed once)

  Optimization: LU decomposition
    W = P × L × U
    
    P: permutation matrix (fixed after init)
    L: lower triangular (learned)
    U: upper triangular (learned)
    
    log|det W| = Σ log|diag(U)|  → O(c) not O(c³)!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Channel mixing comparison:                      │
  │                                                    │
  │  NICE:     Reverse ordering     (fixed, limited) │
  │  RealNVP:  Fixed shuffle        (fixed, better)  │
  │  Glow:     Learned 1×1 conv     (optimal!)       │
  │                                                    │
  │  Learned mixing = more expressive coupling!      │
  └──────────────────────────────────────────────────┘
```

---

## ActNorm (Activation Normalization)

```
Problem: BatchNorm depends on batch statistics
  → Different behavior train/test
  → Not exactly invertible during inference

ActNorm: Per-channel affine transform
  y = s ⊙ x + b

  Initialization (data-dependent):
    Using first mini-batch:
    b = -mean(x, dims=[batch, height, width])
    s = 1 / std(x, dims=[batch, height, width])
    
    After init: mean(y) ≈ 0, std(y) ≈ 1
    
  After initialization: s, b are regular learnable parameters

  Log determinant:
    log|det| = h × w × Σ log|s_i|

  Advantages over BatchNorm:
    ✓ Exactly invertible
    ✓ Same behavior train/test
    ✓ Works with batch size 1
    ✓ Data-dependent init = good starting point
```

---

## Multi-Scale Architecture

```
  Input x (3, H, W)
    │
    ▼ Squeeze → (12, H/2, W/2)
    │
    ├── K steps of flow
    │
    ▼ Split → z₁ (6, H/2, W/2) + remaining (6, H/2, W/2)
    │
    ▼ Squeeze → (24, H/4, W/4)
    │
    ├── K steps of flow
    │
    ▼ Split → z₂ (12, H/4, W/4) + remaining
    │
    ▼ Squeeze → ...
    │
    ├── K steps of flow
    │
    └── z_L (final)

  z = [z₁, z₂, ..., z_L]  ← multi-scale latent code

  Split conditioning:
    p(z_i) is NOT just N(0,1)!
    p(z_i | remaining) = N(μ(remaining), σ(remaining))
    Learned conditional prior → much better likelihood!
```

---

## Latent Space Manipulation

```
Glow enables meaningful latent space operations:

  1. Interpolation:
     z₁ = encode(face_A)
     z₂ = encode(face_B)
     z_t = (1-t)×z₁ + t×z₂
     face_t = decode(z_t)  → smooth transition!

  2. Attribute manipulation:
     Compute average z for attribute (e.g., "smiling"):
       z_smile = mean(encode(smiling_faces))
       z_no_smile = mean(encode(neutral_faces))
       direction = z_smile - z_no_smile
     
     Edit any face:
       z_edited = encode(face) + α × direction
       face_edited = decode(z_edited)  → add smile!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Attribute vectors in Glow's latent space:       │
  │                                                    │
  │  +smile direction    → adds smile                │
  │  +glasses direction  → adds glasses              │
  │  +age direction      → makes older               │
  │  +blonde direction   → changes hair color        │
  │                                                    │
  │  Because encoding is EXACT (not approximate),    │
  │  manipulations are very precise!                 │
  └──────────────────────────────────────────────────┘
```

---

## Temperature Scaling

```
At sampling time, control diversity vs. quality:

  z ~ N(0, T²I)  where T = temperature

  T = 1.0: Full diversity (trained distribution)
  T = 0.7: Reduced diversity, higher quality
  T = 0.0: Mean sample (blurry average)

  Standard practice: T = 0.7 for best visual quality
  Similar to truncation trick in GANs
```

---

## Python: Glow Components

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ActNorm(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.log_scale = nn.Parameter(torch.zeros(1, channels, 1, 1))
        self.bias = nn.Parameter(torch.zeros(1, channels, 1, 1))
        self.initialized = False
    
    def initialize(self, x):
        with torch.no_grad():
            mean = x.mean(dim=[0, 2, 3], keepdim=True)
            std = x.std(dim=[0, 2, 3], keepdim=True)
            self.bias.data = -mean
            self.log_scale.data = -torch.log(std + 1e-6)
        self.initialized = True
    
    def forward(self, x):
        if not self.initialized:
            self.initialize(x)
        y = torch.exp(self.log_scale) * (x + self.bias)
        h, w = x.shape[2], x.shape[3]
        log_det = h * w * self.log_scale.sum()
        return y, log_det

class Invertible1x1Conv(nn.Module):
    def __init__(self, channels):
        super().__init__()
        W = torch.randn(channels, channels)
        Q, _ = torch.linalg.qr(W)
        self.W = nn.Parameter(Q)
    
    def forward(self, x):
        h, w = x.shape[2], x.shape[3]
        y = F.conv2d(x, self.W.unsqueeze(-1).unsqueeze(-1))
        log_det = h * w * torch.slogdet(self.W)[1]
        return y, log_det
    
    def inverse(self, y):
        W_inv = torch.inverse(self.W)
        return F.conv2d(y, W_inv.unsqueeze(-1).unsqueeze(-1))

class GlowStep(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.actnorm = ActNorm(channels)
        self.inv_conv = Invertible1x1Conv(channels)
        self.coupling = AffineCouplingLayer(channels)
    
    def forward(self, x):
        log_det_total = 0
        x, ld = self.actnorm(x);    log_det_total += ld
        x, ld = self.inv_conv(x);   log_det_total += ld
        x, ld = self.coupling(x);   log_det_total += ld
        return x, log_det_total

class GlowModel(nn.Module):
    def __init__(self, channels=3, K=32, L=3):
        super().__init__()
        self.L = L
        self.K = K
        self.flows = nn.ModuleList()
        c = channels * 4  # after squeeze
        
        for level in range(L):
            for step in range(K):
                self.flows.append(GlowStep(c))
            if level < L - 1:
                c = c * 2  # squeeze doubles channels after split
    
    def forward(self, x):
        # Squeeze + flow steps + split at each level
        log_det = 0
        zs = []
        h = squeeze(x)
        
        idx = 0
        for level in range(self.L):
            for _ in range(self.K):
                h, ld = self.flows[idx](h)
                log_det += ld
                idx += 1
            if level < self.L - 1:
                z_out, h = h.chunk(2, dim=1)
                zs.append(z_out)
                h = squeeze(h)
        zs.append(h)
        return zs, log_det
```

---

## Results

```
  Dataset         Resolution   Bits/dim
  ────────        ──────────   ────────
  CIFAR-10        32×32        3.35
  ImageNet 32     32×32        4.09
  ImageNet 64     64×64        3.81
  CelebA-HQ      256×256      1.03
  LSUN Bedroom    256×256      1.20

  Glow generates 256×256 faces competitive with GANs
  With EXACT likelihood (GANs have no likelihood!)
```

---

## Revision Questions

1. **What are the three components of each Glow step?**
2. **How does invertible 1×1 convolution improve over fixed permutation?**
3. **Why is ActNorm better than BatchNorm for flow models?**
4. **How does temperature scaling trade off diversity and quality?**
5. **What makes latent space manipulation in Glow more precise than in VAEs?**

---

[Previous: 04-realnvp.md](04-realnvp.md) | [Next: 06-applications.md](06-applications.md)
