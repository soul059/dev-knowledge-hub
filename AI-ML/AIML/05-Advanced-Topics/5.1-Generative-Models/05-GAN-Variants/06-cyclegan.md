# CycleGAN

## Overview

CycleGAN (Zhu et al., 2017) enables **unpaired image-to-image translation** — converting between two domains (e.g., horses↔zebras, summer↔winter, photos↔paintings) without requiring matched pairs of training images. The key innovation is the **cycle consistency loss**: if you translate an image from domain A to B and back to A, you should get the original image back. This elegantly constrains the problem without paired data.

---

## Paired vs. Unpaired Translation

```
Paired (pix2pix):
  Requires exact correspondences:
  Input photo → Target sketch (same scene!)
  Hard to collect: need aligned pairs

  ┌─────────┐    ┌─────────┐
  │ Photo A  │ ↔ │ Sketch A │  ← same scene!
  │ Photo B  │ ↔ │ Sketch B │
  │ Photo C  │ ↔ │ Sketch C │
  └─────────┘    └─────────┘

Unpaired (CycleGAN):
  Just need two COLLECTIONS from different domains:
  Set of horse photos + Set of zebra photos (unrelated!)
  
  ┌─────────┐    ┌─────────┐
  │ Horse 1  │    │ Zebra X │  ← NOT the same scene!
  │ Horse 2  │    │ Zebra Y │
  │ Horse 3  │    │ Zebra Z │
  └─────────┘    └─────────┘
  
  No correspondence needed! Much easier to collect!
```

---

## Architecture: Two Generator-Discriminator Pairs

```
  Domain A (horses)              Domain B (zebras)

       G_AB                           G_BA
  A ─────────→ B'               B ─────────→ A'
  (horse)    (fake zebra)       (zebra)    (fake horse)

  D_A: Is this a real horse?    D_B: Is this a real zebra?

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Four networks total:                             │
  │                                                    │
  │  G_AB:  Translates A → B  (horse → zebra)        │
  │  G_BA:  Translates B → A  (zebra → horse)        │
  │  D_A:   Distinguishes real A from fake A          │
  │  D_B:   Distinguishes real B from fake B          │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Cycle Consistency Loss

```
The key innovation:

  Forward cycle:  A → G_AB(A) → G_BA(G_AB(A)) ≈ A
  Backward cycle: B → G_BA(B) → G_AB(G_BA(B)) ≈ B

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  🐴 horse                                        │
  │    │                                               │
  │    ▼  G_AB (horse→zebra)                          │
  │  🦓 fake zebra                                   │
  │    │                                               │
  │    ▼  G_BA (zebra→horse)                          │
  │  🐴 reconstructed horse  ← should match original!│
  │                                                    │
  │  L_cycle = ||G_BA(G_AB(A)) - A||₁               │
  │          + ||G_AB(G_BA(B)) - B||₁               │
  │                                                    │
  │  Forces G to preserve content while changing style│
  └──────────────────────────────────────────────────┘

  Without cycle consistency:
    G_AB could map ALL horses to one zebra image!
    (mode collapse in translation)
  
  With cycle consistency:
    G_AB must be (approximately) invertible
    → content must be preserved
```

---

## Full Objective

```
Total loss = L_GAN(G_AB, D_B) + L_GAN(G_BA, D_A) + λ × L_cycle

  L_GAN(G_AB, D_B):  Standard adversarial loss
    D_B tries to distinguish real B from G_AB(A)
    G_AB tries to fool D_B

  L_GAN(G_BA, D_A):  Standard adversarial loss
    D_A tries to distinguish real A from G_BA(B)
    G_BA tries to fool D_A

  L_cycle:  Cycle consistency loss
    ||G_BA(G_AB(A)) - A||₁ + ||G_AB(G_BA(B)) - B||₁

  Optional: Identity loss (preserves color):
    L_identity = ||G_AB(B) - B||₁ + ||G_BA(A) - A||₁
    If input is already in target domain, don't change it!

  λ = 10 (typical for cycle loss)
```

---

## Generator: ResNet Architecture

```
CycleGAN uses a ResNet-based generator (not U-Net):

  Input (256×256×3)
    │
    ▼  Encoder (downsample)
  ┌──────────┐
  │ Conv 7×7  │ → InstanceNorm → ReLU
  │ Conv 3×3  │ → InstanceNorm → ReLU (stride 2)
  │ Conv 3×3  │ → InstanceNorm → ReLU (stride 2)
  └────┬──────┘
       │  64×64 feature maps
       ▼  Transformer (9 ResNet blocks)
  ┌──────────┐
  │ ResBlock  │ × 9  (conv → IN → ReLU → conv → IN + skip)
  └────┬──────┘
       │  64×64 feature maps
       ▼  Decoder (upsample)
  ┌──────────┐
  │ ConvT 3×3│ → InstanceNorm → ReLU (stride 2)
  │ ConvT 3×3│ → InstanceNorm → ReLU (stride 2)
  │ Conv 7×7  │ → Tanh
  └──────────┘
    Output (256×256×3)

  Uses InstanceNorm (not BatchNorm)
  → per-image normalization, better for style transfer
```

---

## PatchGAN Discriminator

```
  Instead of classifying whole image as real/fake,
  classify each N×N patch independently:

  Input (256×256×3)
    │
    ▼
  Conv → Conv → Conv → Conv → Conv
    │
    ▼
  Output (30×30×1)  ← each value = real/fake for that patch

  Advantages:
    • Fewer parameters
    • Works at any image size
    • Captures local texture/style effectively
    • Equivalent to a texture/style discriminator
```

---

## Python: CycleGAN Implementation

```python
import torch
import torch.nn as nn

class ResidualBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.block = nn.Sequential(
            nn.ReflectionPad2d(1),
            nn.Conv2d(channels, channels, 3),
            nn.InstanceNorm2d(channels),
            nn.ReLU(inplace=True),
            nn.ReflectionPad2d(1),
            nn.Conv2d(channels, channels, 3),
            nn.InstanceNorm2d(channels),
        )
    def forward(self, x):
        return x + self.block(x)

class CycleGenerator(nn.Module):
    def __init__(self, in_ch=3, out_ch=3, n_residual=9, features=64):
        super().__init__()
        # Encoder
        model = [
            nn.ReflectionPad2d(3),
            nn.Conv2d(in_ch, features, 7),
            nn.InstanceNorm2d(features),
            nn.ReLU(inplace=True),
        ]
        # Downsampling
        for i in range(2):
            mult = 2 ** i
            model += [
                nn.Conv2d(features*mult, features*mult*2, 3, 2, 1),
                nn.InstanceNorm2d(features*mult*2),
                nn.ReLU(inplace=True),
            ]
        # Residual blocks
        mult = 4
        for _ in range(n_residual):
            model.append(ResidualBlock(features * mult))
        # Upsampling
        for i in range(2):
            mult = 2 ** (2 - i)
            model += [
                nn.ConvTranspose2d(features*mult, features*mult//2, 3, 2, 1, 1),
                nn.InstanceNorm2d(features*mult//2),
                nn.ReLU(inplace=True),
            ]
        model += [nn.ReflectionPad2d(3), nn.Conv2d(features, out_ch, 7), nn.Tanh()]
        self.model = nn.Sequential(*model)
    
    def forward(self, x):
        return self.model(x)

class PatchDiscriminator(nn.Module):
    def __init__(self, in_ch=3, features=64):
        super().__init__()
        self.model = nn.Sequential(
            nn.Conv2d(in_ch, features, 4, 2, 1),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features, features*2, 4, 2, 1),
            nn.InstanceNorm2d(features*2),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features*2, features*4, 4, 2, 1),
            nn.InstanceNorm2d(features*4),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features*4, features*8, 4, 1, 1),
            nn.InstanceNorm2d(features*8),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features*8, 1, 4, 1, 1),
        )
    def forward(self, x):
        return self.model(x)

def cycle_gan_loss(G_AB, G_BA, D_A, D_B, real_A, real_B, lambda_cyc=10):
    criterion_GAN = nn.MSELoss()
    criterion_cycle = nn.L1Loss()
    
    # Generate fakes
    fake_B = G_AB(real_A)
    fake_A = G_BA(real_B)
    
    # Cycle consistency
    recon_A = G_BA(fake_B)
    recon_B = G_AB(fake_A)
    loss_cycle = criterion_cycle(recon_A, real_A) + criterion_cycle(recon_B, real_B)
    
    # GAN losses
    loss_G_AB = criterion_GAN(D_B(fake_B), torch.ones_like(D_B(fake_B)))
    loss_G_BA = criterion_GAN(D_A(fake_A), torch.ones_like(D_A(fake_A)))
    
    # Total generator loss
    loss_G = loss_G_AB + loss_G_BA + lambda_cyc * loss_cycle
    return loss_G, fake_A, fake_B
```

---

## Applications

```
  Domain A        ↔     Domain B
  ─────────             ─────────
  Horses          ↔     Zebras
  Summer          ↔     Winter
  Photos          ↔     Monet paintings
  Day             ↔     Night
  Apples          ↔     Oranges
  CT scans        ↔     MRI scans
  Satellite       ↔     Maps
  Young faces     ↔     Old faces
```

---

## Revision Questions

1. **Why is CycleGAN's unpaired approach harder than paired translation?**
2. **What does cycle consistency loss enforce and why is it necessary?**
3. **Why does CycleGAN use InstanceNorm instead of BatchNorm?**
4. **What is a PatchGAN discriminator and why is it effective?**
5. **When would you add identity loss to CycleGAN?**

---

[Previous: 05-stylegan.md](05-stylegan.md) | [Next: 07-pix2pix.md](07-pix2pix.md)
