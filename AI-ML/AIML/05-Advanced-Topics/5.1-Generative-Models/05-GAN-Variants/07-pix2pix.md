# Pix2Pix

## Overview

Pix2Pix (Isola et al., 2017) is the foundational **paired image-to-image translation** framework. Given aligned input-output pairs (e.g., edge maps → photos, semantic labels → street scenes), Pix2Pix learns a conditional GAN that maps from one domain to another. It introduced the **U-Net generator** with skip connections and the **PatchGAN discriminator**, both of which became standard components in image translation architectures.

---

## Problem Setup

```
Paired image-to-image translation:

  Input x          →    Output y
  ─────────               ─────────
  Edge map          →    Photo
  Semantic label    →    Street scene
  Sketch            →    Shoe photo
  BW image          →    Color image
  Day photo         →    Night photo

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Training data: paired examples (x_i, y_i)       │
  │                                                    │
  │  ┌─────────┐    ┌─────────┐                      │
  │  │ edges   │ ↔  │ photo   │  ← same scene!      │
  │  │ (input) │    │(target) │                       │
  │  └─────────┘    └─────────┘                      │
  │                                                    │
  │  Goal: learn G(x) ≈ y                            │
  │                                                    │
  │  Naive approach: G(x) = argmin ||G(x) - y||₁     │
  │  Problem: L1 loss alone → blurry outputs          │
  │  Solution: L1 loss + adversarial loss → sharp!    │
  └──────────────────────────────────────────────────┘
```

---

## Objective Function

```
L = L_cGAN(G, D) + λ × L_L1(G)

  L_cGAN = E[log D(x, y)] + E[log(1 - D(x, G(x, z)))]
  
  D sees BOTH input x AND output y (or G(x))
  D must check: is the output realistic AND does it match input?

  L_L1 = E[||y - G(x, z)||₁]
  
  L1 loss encourages output to be close to ground truth
  → provides low-frequency correctness (structure)
  
  Adversarial loss adds high-frequency sharpness (details)

  λ = 100 (L1 loss weighted heavily!)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  L1 only:        Blurry but correct structure    │
  │  GAN only:       Sharp but may hallucinate       │
  │  L1 + GAN:       Sharp AND correct! ✓            │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## U-Net Generator

```
Standard encoder-decoder loses fine details:
  encoder bottleneck compresses ALL information

U-Net adds SKIP CONNECTIONS:
  encoder features → directly → decoder features (concatenate)
  Low-level details bypass the bottleneck!

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  Input (256×256)                     Output (256×256)     │
  │       │                                   ↑               │
  │       ▼ ──────── skip connection ────────→│               │
  │  [Conv] 128×128                    [ConvT] 128×128       │
  │       │                                   ↑               │
  │       ▼ ──────── skip connection ────────→│               │
  │  [Conv] 64×64                      [ConvT] 64×64        │
  │       │                                   ↑               │
  │       ▼ ──────── skip connection ────────→│               │
  │  [Conv] 32×32                      [ConvT] 32×32        │
  │       │                                   ↑               │
  │       ▼ ──────── skip connection ────────→│               │
  │  [Conv] 16×16                      [ConvT] 16×16        │
  │       │                                   ↑               │
  │       ▼                                   │               │
  │    [Bottleneck 8×8 or 1×1]               │               │
  │                                                            │
  │  Skip = concatenate encoder + decoder features            │
  │  Decoder channel count doubles at each skip               │
  └──────────────────────────────────────────────────────────┘

  Why U-Net?
  • Edges in input should appear as edges in output
  • Skip connections pass this directly
  • Bottleneck captures high-level semantics
  • Decoder combines semantics + details
```

---

## PatchGAN Discriminator

```
Instead of one real/fake decision for whole image:
  Make real/fake decisions for each N×N patch

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Input: concat(x, y) or concat(x, G(x))         │
  │         6 channels total (3+3)                   │
  │         │                                         │
  │         ▼                                         │
  │  Conv layers (4×4, stride 2)                     │
  │         │                                         │
  │         ▼                                         │
  │  Output: 30×30×1                                 │
  │                                                    │
  │  Each output pixel = real/fake for a 70×70 patch │
  │  (receptive field of each output = 70×70 pixels) │
  │                                                    │
  │  Advantages:                                      │
  │  • Fewer parameters than full-image D            │
  │  • Works at any input resolution                 │
  │  • Focuses on local texture (high-frequency)     │
  │  • L1 loss handles global structure              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Pix2Pix Implementation

```python
import torch
import torch.nn as nn

class UNetBlock(nn.Module):
    """U-Net encoder block: Conv → BN → LeakyReLU."""
    def __init__(self, in_ch, out_ch, normalize=True):
        super().__init__()
        layers = [nn.Conv2d(in_ch, out_ch, 4, 2, 1, bias=False)]
        if normalize:
            layers.append(nn.BatchNorm2d(out_ch))
        layers.append(nn.LeakyReLU(0.2, True))
        self.model = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.model(x)

class UNetGenerator(nn.Module):
    def __init__(self, in_ch=3, out_ch=3, features=64):
        super().__init__()
        # Encoder
        self.e1 = UNetBlock(in_ch, features, normalize=False)    # 128
        self.e2 = UNetBlock(features, features*2)                 # 64
        self.e3 = UNetBlock(features*2, features*4)               # 32
        self.e4 = UNetBlock(features*4, features*8)               # 16
        self.e5 = UNetBlock(features*8, features*8)               # 8
        self.e6 = UNetBlock(features*8, features*8)               # 4
        self.e7 = UNetBlock(features*8, features*8)               # 2
        self.e8 = UNetBlock(features*8, features*8, normalize=False)  # 1
        
        # Decoder (with skip connections via concatenation)
        self.d1 = self._decoder_block(features*8, features*8, dropout=True)
        self.d2 = self._decoder_block(features*16, features*8, dropout=True)
        self.d3 = self._decoder_block(features*16, features*8, dropout=True)
        self.d4 = self._decoder_block(features*16, features*8)
        self.d5 = self._decoder_block(features*16, features*4)
        self.d6 = self._decoder_block(features*8, features*2)
        self.d7 = self._decoder_block(features*4, features)
        
        self.final = nn.Sequential(
            nn.ConvTranspose2d(features*2, out_ch, 4, 2, 1),
            nn.Tanh(),
        )
    
    def _decoder_block(self, in_ch, out_ch, dropout=False):
        layers = [
            nn.ConvTranspose2d(in_ch, out_ch, 4, 2, 1, bias=False),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(True),
        ]
        if dropout:
            layers.append(nn.Dropout(0.5))
        return nn.Sequential(*layers)
    
    def forward(self, x):
        # Encode
        e1 = self.e1(x);  e2 = self.e2(e1)
        e3 = self.e3(e2);  e4 = self.e4(e3)
        e5 = self.e5(e4);  e6 = self.e6(e5)
        e7 = self.e7(e6);  e8 = self.e8(e7)
        # Decode with skip connections
        d1 = self.d1(e8)
        d2 = self.d2(torch.cat([d1, e7], 1))
        d3 = self.d3(torch.cat([d2, e6], 1))
        d4 = self.d4(torch.cat([d3, e5], 1))
        d5 = self.d5(torch.cat([d4, e4], 1))
        d6 = self.d6(torch.cat([d5, e3], 1))
        d7 = self.d7(torch.cat([d6, e2], 1))
        return self.final(torch.cat([d7, e1], 1))

def pix2pix_loss(G, D, real_input, real_target, lambda_l1=100):
    criterion_GAN = nn.MSELoss()
    criterion_L1 = nn.L1Loss()
    
    fake_target = G(real_input)
    
    # D loss
    pred_real = D(torch.cat([real_input, real_target], 1))
    pred_fake = D(torch.cat([real_input, fake_target.detach()], 1))
    d_loss = (criterion_GAN(pred_real, torch.ones_like(pred_real)) +
              criterion_GAN(pred_fake, torch.zeros_like(pred_fake))) * 0.5
    
    # G loss
    pred_fake_for_g = D(torch.cat([real_input, fake_target], 1))
    g_gan = criterion_GAN(pred_fake_for_g, torch.ones_like(pred_fake_for_g))
    g_l1 = criterion_L1(fake_target, real_target)
    g_loss = g_gan + lambda_l1 * g_l1
    
    return d_loss, g_loss
```

---

## Applications

```
  Input               →      Output
  ─────────                   ─────────
  Edges               →      Photos (edges2shoes, edges2bags)
  Semantic labels     →      Street scenes (Cityscapes)
  Satellite images    →      Maps (and vice versa)
  BW photos           →      Colorized photos
  Day photos          →      Night photos
  Facades labels      →      Building photos
  Sketches            →      Realistic faces
  Pose skeleton       →      Person photo
```

---

## Revision Questions

1. **Why does L1 loss alone produce blurry outputs?**
2. **How do skip connections in U-Net help image translation?**
3. **Why does Pix2Pix concatenate input and output for the discriminator?**
4. **What is the 70×70 PatchGAN and why is it effective?**
5. **When would you use Pix2Pix vs. CycleGAN?**

---

[Previous: 06-cyclegan.md](06-cyclegan.md) | [Next: ../06-Normalizing-Flows/01-flow-based-models-concept.md](../06-Normalizing-Flows/01-flow-based-models-concept.md)
