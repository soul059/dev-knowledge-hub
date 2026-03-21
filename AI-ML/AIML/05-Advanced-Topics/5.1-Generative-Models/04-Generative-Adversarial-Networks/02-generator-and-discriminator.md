# Generator and Discriminator

## Overview

The **generator** and **discriminator** are the two neural networks that form a GAN. The generator transforms random noise into data, while the discriminator classifies inputs as real or fake. Their architectures must be carefully balanced — if one is too powerful, training fails. This chapter covers design principles, architecture choices, and implementation details for both networks.

---

## Generator Architecture

```
Generator: maps noise z to data space x

  z ∈ ℝ^d_z  →  G_θ  →  x_fake ∈ ℝ^d_x

  z is sampled from a simple distribution (usually N(0,1))
  G learns to transform this into the data distribution

  FC Generator (MNIST):
    z [100] → 256 → 512 → 1024 → 784 → reshape [1,28,28]
    
  Conv Generator (images):
    z [100] → Dense → reshape [512,4,4]
      → ConvTranspose → [256,8,8]
      → ConvTranspose → [128,16,16]
      → ConvTranspose → [64,32,32]
      → ConvTranspose → [3,64,64]

  ┌──────────────────────────────────────────────────┐
  │  Generator design principles:                     │
  │                                                    │
  │  • Use BatchNorm (stabilizes training)            │
  │  • Use ReLU or LeakyReLU activations             │
  │  • Output: Tanh (images in [-1,1])               │
  │  • Use ConvTranspose2d for spatial upsampling     │
  │  • Progressively increase spatial resolution      │
  │  • Kernel size 4 or 5 with stride 2              │
  └──────────────────────────────────────────────────┘
```

---

## Discriminator Architecture

```
Discriminator: maps data x to real/fake probability

  x ∈ ℝ^d_x  →  D_φ  →  P(real) ∈ [0,1]

  Mirror of generator but in reverse:
    Spatial dims decrease, channels increase

  Conv Discriminator (images):
    Input [3,64,64]
      → Conv+LReLU → [64,32,32]
      → Conv+LReLU → [128,16,16]
      → Conv+LReLU → [256,8,8]
      → Conv+LReLU → [512,4,4]
      → Flatten → Dense → Sigmoid → P(real)

  ┌──────────────────────────────────────────────────┐
  │  Discriminator design principles:                 │
  │                                                    │
  │  • Use LeakyReLU (not ReLU — avoids dead neurons)│
  │  • NO BatchNorm in discriminator (or use         │
  │    LayerNorm/SpectralNorm instead)                │
  │  • Output: Sigmoid for vanilla GAN               │
  │  • Use strided convolutions (not pooling)         │
  │  • Dropout can help prevent overfitting           │
  │  • Don't make D too powerful relative to G        │
  └──────────────────────────────────────────────────┘
```

---

## Balance Between G and D

```
Critical: G and D must be roughly equally capable

  D too strong:
    D perfectly classifies everything → D loss = 0
    G gets no useful gradient → G can't improve!
    → Training stalls

  G too strong (rare):
    G perfectly fools D → D can't learn
    G might collapse to generating one "perfect" image
    → Mode collapse

  Balance strategies:
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  1. Same or similar architecture depth/width      │
  │  2. Train D for k steps per 1 G step (k=1-5)    │
  │  3. Use learning rate ratio (D_lr ≈ G_lr)        │
  │  4. Label smoothing (real=0.9 not 1.0)           │
  │  5. Spectral normalization on D                  │
  │  6. Monitor both losses during training           │
  │                                                    │
  │  Sign of good balance:                            │
  │    D loss ≈ 0.5-0.7 (uncertain)                  │
  │    G loss decreasing slowly                       │
  │    Sample quality improving                        │
  └──────────────────────────────────────────────────┘
```

---

## Python: DCGAN-Style Generator and Discriminator

```python
import torch
import torch.nn as nn

class DCGenerator(nn.Module):
    """DCGAN-style generator for 64x64 images."""
    def __init__(self, latent_dim=100, channels=3):
        super().__init__()
        self.net = nn.Sequential(
            # z → 512×4×4
            nn.ConvTranspose2d(latent_dim, 512, 4, 1, 0, bias=False),
            nn.BatchNorm2d(512), nn.ReLU(True),
            # → 256×8×8
            nn.ConvTranspose2d(512, 256, 4, 2, 1, bias=False),
            nn.BatchNorm2d(256), nn.ReLU(True),
            # → 128×16×16
            nn.ConvTranspose2d(256, 128, 4, 2, 1, bias=False),
            nn.BatchNorm2d(128), nn.ReLU(True),
            # → 64×32×32
            nn.ConvTranspose2d(128, 64, 4, 2, 1, bias=False),
            nn.BatchNorm2d(64), nn.ReLU(True),
            # → 3×64×64
            nn.ConvTranspose2d(64, channels, 4, 2, 1, bias=False),
            nn.Tanh())
    
    def forward(self, z):
        return self.net(z.view(-1, z.size(1), 1, 1))

class DCDiscriminator(nn.Module):
    """DCGAN-style discriminator for 64x64 images."""
    def __init__(self, channels=3):
        super().__init__()
        self.net = nn.Sequential(
            # 3×64×64 → 64×32×32
            nn.Conv2d(channels, 64, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            # → 128×16×16
            nn.Conv2d(64, 128, 4, 2, 1, bias=False),
            nn.BatchNorm2d(128), nn.LeakyReLU(0.2, inplace=True),
            # → 256×8×8
            nn.Conv2d(128, 256, 4, 2, 1, bias=False),
            nn.BatchNorm2d(256), nn.LeakyReLU(0.2, inplace=True),
            # → 512×4×4
            nn.Conv2d(256, 512, 4, 2, 1, bias=False),
            nn.BatchNorm2d(512), nn.LeakyReLU(0.2, inplace=True),
            # → 1×1×1
            nn.Conv2d(512, 1, 4, 1, 0, bias=False),
            nn.Sigmoid())
    
    def forward(self, x):
        return self.net(x).view(-1, 1)

# Weight initialization (important for GANs!)
def weights_init(m):
    if isinstance(m, (nn.Conv2d, nn.ConvTranspose2d)):
        nn.init.normal_(m.weight, 0.0, 0.02)
    elif isinstance(m, nn.BatchNorm2d):
        nn.init.normal_(m.weight, 1.0, 0.02)
        nn.init.constant_(m.bias, 0)
```

---

## Revision Questions

1. **What are the architectural differences between the generator and discriminator?**
2. **Why is LeakyReLU preferred over ReLU in the discriminator?**
3. **What happens when the discriminator is too powerful?**
4. **Why is weight initialization important for GAN training?**
5. **Why does the generator use Tanh output activation?**

---

[Previous: 01-gan-concept.md](01-gan-concept.md) | [Next: 03-adversarial-training.md](03-adversarial-training.md)
