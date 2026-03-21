# Progressive GAN (ProGAN)

## Overview

Progressive GAN (Karras et al., 2018) introduced the idea of **growing both G and D during training** — starting at low resolution (4×4) and progressively adding layers to generate higher resolutions (up to 1024×1024). This stabilizes training dramatically and was the first GAN to generate photorealistic high-resolution faces. ProGAN's progressive growing strategy influenced all subsequent high-resolution GANs, including StyleGAN.

---

## Progressive Growing

```
Training phases:

  Phase 1:  4×4    →  Train until stable
  Phase 2:  8×8    →  Add layers, train
  Phase 3:  16×16  →  Add layers, train
  Phase 4:  32×32  →  Add layers, train
  Phase 5:  64×64  →  Add layers, train
  Phase 6:  128×128 → Add layers, train
  Phase 7:  256×256 → Add layers, train
  Phase 8:  512×512 → Add layers, train
  Phase 9:  1024×1024 → Final training

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  Phase 1:   [G: z → 4×4]  ←→  [D: 4×4 → score]         │
  │                                                            │
  │  Phase 2:   [G: z → 4×4 → 8×8]  ←→  [D: 8×8 → ... ]   │
  │                                                            │
  │  Phase 3:   [G: z → 4×4 → 8×8 → 16×16]  ←→  [D: ...]  │
  │                                                            │
  │  ...continues doubling until 1024×1024...                 │
  │                                                            │
  │  Key insight: low-res = large-scale structure             │
  │               high-res = fine details                     │
  │  Learn coarse features first, then refine!                │
  └──────────────────────────────────────────────────────────┘
```

---

## Smooth Fade-In

```
When adding a new layer, don't add it abruptly:
Use alpha blending to smoothly transition

  α goes from 0 → 1 during transition phase

  Generator output (when growing to 2N×2N):
    output = (1 - α) × upsampled(old_output) + α × new_layer(features)

  Discriminator input (when growing to 2N×2N):
    input = (1 - α) × downsampled(new_input) + α × new_layer(new_input)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  α = 0:  Only old path (existing layers)          │
  │  α = 0.5: Half old + half new                    │
  │  α = 1:  Only new path (new layer integrated)    │
  │                                                    │
  │  Old path:  features → toRGB → upsample          │
  │                                   ↓               │
  │                              blend (1-α)          │
  │  New path:  features → newBlock → toRGB           │
  │                                   ↓               │
  │                              blend (α)            │
  │                                                    │
  │  Prevents training instability during transitions │
  └──────────────────────────────────────────────────┘
```

---

## Key Innovations

```
1. Minibatch Standard Deviation
   Add a feature channel showing batch diversity
   Helps detect mode collapse

   For each spatial location:
     σ = std(features across batch)
     Append mean(σ) as extra channel to D's input

2. Pixelwise Feature Normalization (in G)
   Normalize each pixel's feature vector:
   b(x,y) = a(x,y) / sqrt(mean(a(x,y)²) + ε)
   
   Prevents signal magnitudes from growing out of control

3. Equalized Learning Rate
   Instead of careful initialization:
     w_runtime = w × c
     c = sqrt(2 / fan_in)  (He init constant)
   
   Weights initialized N(0, 1), scaled at runtime
   Ensures all layers learn at the same speed

4. No BatchNorm
   Uses pixelwise normalization instead
   More stable at high resolutions
```

---

## Python: Progressive GAN Concepts

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class EqualizedConv2d(nn.Module):
    """Conv2d with equalized learning rate."""
    def __init__(self, in_ch, out_ch, kernel_size, stride=1, padding=0):
        super().__init__()
        self.conv = nn.Conv2d(in_ch, out_ch, kernel_size, stride, padding)
        nn.init.normal_(self.conv.weight, 0, 1)
        nn.init.zeros_(self.conv.bias)
        self.scale = (2 / (in_ch * kernel_size ** 2)) ** 0.5
    
    def forward(self, x):
        return self.conv(x) * self.scale

class PixelNorm(nn.Module):
    """Pixelwise feature normalization."""
    def forward(self, x):
        return x / torch.sqrt(torch.mean(x ** 2, dim=1, keepdim=True) + 1e-8)

class MinibatchStdDev(nn.Module):
    """Minibatch standard deviation layer."""
    def forward(self, x):
        std = torch.std(x, dim=0, keepdim=True)
        mean_std = std.mean().expand(x.size(0), 1, x.size(2), x.size(3))
        return torch.cat([x, mean_std], dim=1)

class ProGANGenerator(nn.Module):
    def __init__(self, latent_dim=512, max_resolution=256):
        super().__init__()
        self.latent_dim = latent_dim
        
        # Initial block: z → 4×4
        self.initial = nn.Sequential(
            nn.ConvTranspose2d(latent_dim, 512, 4, 1, 0),
            PixelNorm(),
            nn.LeakyReLU(0.2),
            EqualizedConv2d(512, 512, 3, 1, 1),
            PixelNorm(),
            nn.LeakyReLU(0.2),
        )
        
        # Progressive blocks (added during training)
        self.blocks = nn.ModuleList()
        self.to_rgb = nn.ModuleList()
        
        # toRGB for 4×4
        self.to_rgb.append(EqualizedConv2d(512, 3, 1))
        
        # Add blocks for each resolution doubling
        channels = [512, 512, 256, 256, 128, 64, 32]
        in_ch = 512
        for out_ch in channels:
            self.blocks.append(nn.Sequential(
                nn.Upsample(scale_factor=2),
                EqualizedConv2d(in_ch, out_ch, 3, 1, 1),
                PixelNorm(),
                nn.LeakyReLU(0.2),
                EqualizedConv2d(out_ch, out_ch, 3, 1, 1),
                PixelNorm(),
                nn.LeakyReLU(0.2),
            ))
            self.to_rgb.append(EqualizedConv2d(out_ch, 3, 1))
            in_ch = out_ch
    
    def forward(self, z, current_depth, alpha):
        x = self.initial(z.view(-1, self.latent_dim, 1, 1))
        
        if current_depth == 0:
            return self.to_rgb[0](x)
        
        for i in range(current_depth - 1):
            x = self.blocks[i](x)
        
        # Fade-in: blend old and new path
        old = F.interpolate(self.to_rgb[current_depth - 1](x), scale_factor=2)
        x = self.blocks[current_depth - 1](x)
        new = self.to_rgb[current_depth](x)
        
        return (1 - alpha) * old + alpha * new

# Progressive training loop (conceptual)
def progressive_train(G, D, dataloader_fn, latent_dim, device, 
                       num_depths=7):
    for depth in range(num_depths):
        resolution = 4 * (2 ** depth)
        print(f"Training at {resolution}x{resolution}")
        dataloader = dataloader_fn(resolution)
        
        # Fade-in phase
        for epoch in range(fade_in_epochs):
            alpha = epoch / fade_in_epochs
            for real, _ in dataloader:
                # Train at this depth with fading alpha
                train_step(G, D, real, depth, alpha, latent_dim, device)
        
        # Stabilization phase (alpha = 1)
        for epoch in range(stabilize_epochs):
            for real, _ in dataloader:
                train_step(G, D, real, depth, 1.0, latent_dim, device)
```

---

## Results and Impact

```
ProGAN achievements:
  • First GAN to generate 1024×1024 face images
  • CelebA-HQ dataset (30k high-res faces)
  • Training time: ~4 days on 8 GPUs
  • Unprecedented quality for 2018

Impact on future work:
  ProGAN → StyleGAN → StyleGAN2 → StyleGAN3
  Progressive growing idea used widely:
    • MSG-GAN (multi-scale gradient)
    • StyleGAN (style-based progressive)
```

---

## Revision Questions

1. **Why does progressive growing help GAN training stability?**
2. **How does the smooth fade-in (alpha blending) work?**
3. **What is equalized learning rate and why is it used?**
4. **How does minibatch standard deviation help prevent mode collapse?**
5. **Why does ProGAN use pixel normalization instead of BatchNorm?**

---

[Previous: 03-conditional-gan.md](03-conditional-gan.md) | [Next: 05-stylegan.md](05-stylegan.md)
