# DCGAN (Deep Convolutional GAN)

## Overview

DCGAN (Radford et al., 2016) established the architectural blueprint for convolutional GANs. By replacing fully connected layers with strided convolutions, using batch normalization, and following specific activation function guidelines, DCGAN demonstrated that CNNs could generate high-quality images reliably. Nearly all subsequent image GANs build on DCGAN's design principles.

---

## Architecture Principles

```
DCGAN introduced 5 key architectural guidelines:

  1. Replace pooling with strided convolutions
     D: strided Conv2d (downsample)
     G: ConvTranspose2d (upsample)
  
  2. Use BatchNorm in both G and D
     Except: G output layer, D input layer
  
  3. Remove fully connected layers
     Use global average pooling or reshape
  
  4. G activations: ReLU everywhere, Tanh at output
  
  5. D activations: LeakyReLU everywhere

  ┌────────────────────────────────────────────────────────┐
  │                  DCGAN Architecture                     │
  │                                                          │
  │  Generator (z → image):                                 │
  │  z(100) → Reshape → ConvT → ConvT → ConvT → ConvT     │
  │  1×1     4×4        8×8     16×16    32×32   64×64      │
  │           BN+ReLU   BN+ReLU BN+ReLU BN+ReLU  Tanh      │
  │                                                          │
  │  Discriminator (image → real/fake):                     │
  │  64×64 → Conv → Conv → Conv → Conv → Sigmoid           │
  │  3ch     64     128    256    512    1                   │
  │          LReLU  BN+LR  BN+LR  BN+LR                    │
  └────────────────────────────────────────────────────────┘
```

---

## Generator Architecture

```
  Input: z ~ N(0, 1), shape: (batch, 100)

  z (100)
    │
    ▼
  Linear → Reshape to (512, 4, 4)
    │
    ▼  ConvTranspose2d(512, 256, 4, 2, 1)
  ┌─────────┐
  │ 256×8×8  │  BatchNorm + ReLU
  └────┬─────┘
       ▼  ConvTranspose2d(256, 128, 4, 2, 1)
  ┌──────────┐
  │ 128×16×16│  BatchNorm + ReLU
  └────┬──────┘
       ▼  ConvTranspose2d(128, 64, 4, 2, 1)
  ┌──────────┐
  │ 64×32×32 │  BatchNorm + ReLU
  └────┬──────┘
       ▼  ConvTranspose2d(64, 3, 4, 2, 1)
  ┌──────────┐
  │ 3×64×64  │  Tanh → output image [-1, 1]
  └──────────┘
```

---

## Discriminator Architecture

```
  Input: image (batch, 3, 64, 64)

  3×64×64
    │
    ▼  Conv2d(3, 64, 4, 2, 1)       No BatchNorm on first layer!
  ┌──────────┐
  │ 64×32×32 │  LeakyReLU(0.2)
  └────┬──────┘
       ▼  Conv2d(64, 128, 4, 2, 1)
  ┌──────────┐
  │ 128×16×16│  BatchNorm + LeakyReLU(0.2)
  └────┬──────┘
       ▼  Conv2d(128, 256, 4, 2, 1)
  ┌──────────┐
  │ 256×8×8  │  BatchNorm + LeakyReLU(0.2)
  └────┬──────┘
       ▼  Conv2d(256, 512, 4, 2, 1)
  ┌──────────┐
  │ 512×4×4  │  BatchNorm + LeakyReLU(0.2)
  └────┬──────┘
       ▼  Conv2d(512, 1, 4, 1, 0)
  ┌──────────┐
  │  1×1×1   │  Sigmoid → P(real)
  └──────────┘
```

---

## Latent Space Arithmetic

```
DCGAN showed that the latent space has semantic structure:

  z_smiling_woman - z_neutral_woman + z_neutral_man = z_smiling_man

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  G(z_smiling_woman) = 😊👩                       │
  │  G(z_neutral_woman) = 😐👩                       │
  │  G(z_neutral_man)   = 😐👨                       │
  │                                                    │
  │  z_result = z_smiling_woman                       │
  │           - z_neutral_woman                       │
  │           + z_neutral_man                         │
  │                                                    │
  │  G(z_result) ≈ 😊👨  (smiling man!)             │
  │                                                    │
  │  This means the latent space learned              │
  │  disentangled, meaningful representations!        │
  └──────────────────────────────────────────────────┘

  Also: smooth interpolation between z vectors
  creates smooth transitions in generated images
```

---

## Python: Full DCGAN Implementation

```python
import torch
import torch.nn as nn

class DCGenerator(nn.Module):
    def __init__(self, latent_dim=100, channels=3, features=64):
        super().__init__()
        self.net = nn.Sequential(
            # (latent_dim) → (features*8, 4, 4)
            nn.ConvTranspose2d(latent_dim, features*8, 4, 1, 0, bias=False),
            nn.BatchNorm2d(features*8),
            nn.ReLU(True),
            # → (features*4, 8, 8)
            nn.ConvTranspose2d(features*8, features*4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features*4),
            nn.ReLU(True),
            # → (features*2, 16, 16)
            nn.ConvTranspose2d(features*4, features*2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features*2),
            nn.ReLU(True),
            # → (features, 32, 32)
            nn.ConvTranspose2d(features*2, features, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features),
            nn.ReLU(True),
            # → (channels, 64, 64)
            nn.ConvTranspose2d(features, channels, 4, 2, 1, bias=False),
            nn.Tanh(),
        )
    
    def forward(self, z):
        return self.net(z.view(-1, z.size(1), 1, 1))

class DCDiscriminator(nn.Module):
    def __init__(self, channels=3, features=64):
        super().__init__()
        self.net = nn.Sequential(
            # (channels, 64, 64) → (features, 32, 32)
            nn.Conv2d(channels, features, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            # → (features*2, 16, 16)
            nn.Conv2d(features, features*2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features*2),
            nn.LeakyReLU(0.2, inplace=True),
            # → (features*4, 8, 8)
            nn.Conv2d(features*2, features*4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features*4),
            nn.LeakyReLU(0.2, inplace=True),
            # → (features*8, 4, 4)
            nn.Conv2d(features*4, features*8, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features*8),
            nn.LeakyReLU(0.2, inplace=True),
            # → (1, 1, 1)
            nn.Conv2d(features*8, 1, 4, 1, 0, bias=False),
            nn.Sigmoid(),
        )
    
    def forward(self, x):
        return self.net(x).view(-1, 1)

def weights_init(m):
    """DCGAN weight initialization."""
    classname = m.__class__.__name__
    if 'Conv' in classname:
        nn.init.normal_(m.weight.data, 0.0, 0.02)
    elif 'BatchNorm' in classname:
        nn.init.normal_(m.weight.data, 1.0, 0.02)
        nn.init.constant_(m.bias.data, 0)

# Usage
latent_dim = 100
G = DCGenerator(latent_dim).apply(weights_init)
D = DCDiscriminator().apply(weights_init)
z = torch.randn(16, latent_dim)
fake_images = G(z)  # (16, 3, 64, 64)
predictions = D(fake_images)  # (16, 1)
```

---

## Revision Questions

1. **What are the 5 architectural guidelines DCGAN introduced?**
2. **Why is BatchNorm omitted from D's first layer and G's output layer?**
3. **Why use LeakyReLU instead of ReLU in the discriminator?**
4. **What does latent space arithmetic demonstrate about DCGAN?**
5. **How does DCGAN's weight initialization differ from default PyTorch init?**

---

[Previous: ../04-Generative-Adversarial-Networks/06-gan-training-tips.md](../04-Generative-Adversarial-Networks/06-gan-training-tips.md) | [Next: 02-wgan-and-wgan-gp.md](02-wgan-and-wgan-gp.md)
