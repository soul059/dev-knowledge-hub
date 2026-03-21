# DDPM (Denoising Diffusion Probabilistic Models)

## Overview

DDPM (Ho et al., 2020) is the seminal paper that made diffusion models practical for high-quality image generation. It showed that a simplified training objective — predicting the noise added at each step — achieves results competitive with GANs while being much more stable to train. DDPM established the standard framework used by nearly all subsequent diffusion models.

---

## DDPM Algorithm

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  TRAINING:                                        │
  │  ─────────                                        │
  │  repeat:                                          │
  │    x₀ ~ data                                     │
  │    t ~ Uniform{1, ..., T}                        │
  │    ε ~ N(0, I)                                   │
  │    x_t = √ᾱ_t × x₀ + √(1-ᾱ_t) × ε             │
  │    loss = ||ε - ε_θ(x_t, t)||²                  │
  │    gradient step on θ                             │
  │                                                    │
  │  SAMPLING:                                        │
  │  ──────────                                       │
  │  x_T ~ N(0, I)                                   │
  │  for t = T, ..., 1:                              │
  │    z ~ N(0, I) if t > 1 else z = 0              │
  │    x_{t-1} = 1/√α_t (x_t - β_t/√(1-ᾱ_t) ε_θ)  │
  │            + σ_t z                               │
  │  return x₀                                       │
  │                                                    │
  └──────────────────────────────────────────────────┘

  That's the ENTIRE algorithm!
  Remarkably simple for state-of-the-art generation.
```

---

## Network Architecture

```
U-Net with modifications:

  Key components:
  1. Sinusoidal timestep embedding (like Transformer positional encoding)
  2. ResBlocks with timestep conditioning
  3. Self-attention at certain resolutions (16×16, 8×8)
  4. Group normalization (not BatchNorm)

  Timestep embedding:
    t → [sin(t/10000^(0/d)), cos(t/10000^(0/d)),
          sin(t/10000^(2/d)), cos(t/10000^(2/d)), ...]
    → MLP → embedding vector

  ResBlock with timestep:
    h = GroupNorm(x)
    h = SiLU(h)
    h = Conv(h)
    h = h + MLP(time_emb)     ← timestep injected here!
    h = GroupNorm(h)
    h = SiLU(h)
    h = Dropout(h)
    h = Conv(h)
    return x + h               ← residual connection

  Channels: 128 base, multiplied at each resolution
  Resolutions: 64→32→16→8→16→32→64 (for 64×64 images)
```

---

## Variance Schedule

```
DDPM uses linear schedule:

  β₁ = 10⁻⁴, β_T = 0.02, T = 1000

  Problems with linear:
  • Too much noise added too quickly
  • At t ≈ 500, image almost pure noise
  • Wasted capacity in early/late steps

  Improved DDPM (Nichol & Dhariwal, 2021):
  
  Cosine schedule:
    ᾱ_t = cos²((t/T + s)/(1+s) × π/2)
    s = 0.008 (prevents β near 0 or 1)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ᾱ_t comparison:                                  │
  │                                                    │
  │  1.0 ┤──╲╲                                       │
  │      │    ╲╲╲    cosine (better)                 │
  │  0.5 ┤      ╲╲╲                                  │
  │      │    ╲    ╲╲╲                               │
  │      │   linear╲  ╲╲                             │
  │  0.0 ┤──────────────╲─►                          │
  │      0          500   1000                        │
  │                                                    │
  │  Cosine: info preserved longer → better quality  │
  └──────────────────────────────────────────────────┘
```

---

## Improved DDPM

```
Nichol & Dhariwal (2021) improvements:

  1. LEARNED variance (instead of fixed):
     v_θ(x_t, t) → interpolates between β_t and β̃_t
     σ²_t = exp(v_θ log β_t + (1-v_θ) log β̃_t)
     
     Better log-likelihood with minimal quality change

  2. Cosine noise schedule (as above)

  3. Fewer sampling steps:
     Can use strided sampling: t = [1000, 900, 800, ...]
     Only 100 steps instead of 1000!

  4. Hybrid loss:
     L_hybrid = L_simple + λ × L_vlb
     L_vlb computed with stop-gradient on μ_θ
     
     Helps learn variance without hurting mean prediction
```

---

## Conditional Generation

```
Two approaches for conditioning:

  1. Classifier guidance (Dhariwal & Nichol, 2021):
     Use a separate classifier p(y|x_t) trained on noisy images
     Modify score: ε̃ = ε_θ - s × σ_t × ∇_x log p(y|x_t)
     
     s = guidance scale (higher = more adherence to condition)

  2. Classifier-free guidance (Ho & Salimans, 2022):
     Train ONE model with and without condition:
       ε_θ(x_t, t, y)    (conditional)
       ε_θ(x_t, t, ∅)    (unconditional, y = null)
     
     At sampling:
       ε̃ = ε_θ(x_t, t, ∅) + w × (ε_θ(x_t, t, y) - ε_θ(x_t, t, ∅))
       
       w = guidance weight
       w = 1: standard conditional
       w > 1: amplified conditioning (higher quality!)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Classifier-free guidance is DOMINANT:           │
  │                                                    │
  │  • No separate classifier needed                 │
  │  • Drop condition randomly during training (10%) │
  │  • At inference: interpolate between cond/uncond │
  │  • w = 7.5 typical for text-to-image            │
  │  • Higher w = more prompt adherence, less variety│
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Complete DDPM

```python
import torch
import torch.nn as nn
import math

class SinusoidalEmbedding(nn.Module):
    def __init__(self, dim):
        super().__init__()
        self.dim = dim
    
    def forward(self, t):
        half = self.dim // 2
        emb = math.log(10000) / (half - 1)
        emb = torch.exp(torch.arange(half, device=t.device) * -emb)
        emb = t.float().unsqueeze(1) * emb.unsqueeze(0)
        return torch.cat([torch.sin(emb), torch.cos(emb)], dim=1)

class ResBlock(nn.Module):
    def __init__(self, channels, time_dim):
        super().__init__()
        self.norm1 = nn.GroupNorm(8, channels)
        self.conv1 = nn.Conv2d(channels, channels, 3, 1, 1)
        self.time_mlp = nn.Linear(time_dim, channels)
        self.norm2 = nn.GroupNorm(8, channels)
        self.conv2 = nn.Conv2d(channels, channels, 3, 1, 1)
    
    def forward(self, x, t_emb):
        h = torch.silu(self.norm1(x))
        h = self.conv1(h)
        h = h + self.time_mlp(t_emb).unsqueeze(-1).unsqueeze(-1)
        h = torch.silu(self.norm2(h))
        h = self.conv2(h)
        return x + h

class SimpleUNet(nn.Module):
    def __init__(self, in_ch=3, base_ch=64, time_dim=256):
        super().__init__()
        self.time_embed = nn.Sequential(
            SinusoidalEmbedding(time_dim),
            nn.Linear(time_dim, time_dim),
            nn.SiLU(),
            nn.Linear(time_dim, time_dim),
        )
        # Encoder
        self.enc1 = ResBlock(base_ch, time_dim)
        self.enc2 = ResBlock(base_ch*2, time_dim)
        self.down1 = nn.Conv2d(in_ch, base_ch, 3, 1, 1)
        self.down2 = nn.Conv2d(base_ch, base_ch*2, 4, 2, 1)
        # Bottleneck
        self.mid = ResBlock(base_ch*2, time_dim)
        # Decoder
        self.up1 = nn.ConvTranspose2d(base_ch*2, base_ch, 4, 2, 1)
        self.dec1 = ResBlock(base_ch*2, time_dim)  # skip connection doubles channels
        self.out = nn.Conv2d(base_ch*2, in_ch, 1)
    
    def forward(self, x, t):
        t_emb = self.time_embed(t)
        # Encoder
        h1 = self.enc1(self.down1(x), t_emb)
        h2 = self.enc2(self.down2(h1), t_emb)
        # Bottleneck
        h = self.mid(h2, t_emb)
        # Decoder with skip
        h = self.up1(h)
        h = torch.cat([h, h1], dim=1)
        h = self.dec1(h, t_emb)
        h = torch.cat([h, self.down1(x)], dim=1)
        return self.out(h)

class DDPM:
    def __init__(self, model, T=1000, device='cpu'):
        self.model = model
        self.T = T
        self.device = device
        
        betas = torch.linspace(1e-4, 0.02, T).to(device)
        alphas = 1 - betas
        alpha_bars = torch.cumprod(alphas, dim=0)
        
        self.betas = betas
        self.alphas = alphas
        self.alpha_bars = alpha_bars
    
    def train_step(self, x0, optimizer):
        t = torch.randint(0, self.T, (x0.size(0),), device=self.device)
        noise = torch.randn_like(x0)
        
        ab = self.alpha_bars[t].view(-1, 1, 1, 1)
        x_t = torch.sqrt(ab) * x0 + torch.sqrt(1 - ab) * noise
        
        pred = self.model(x_t, t)
        loss = nn.functional.mse_loss(pred, noise)
        
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        return loss.item()
    
    @torch.no_grad()
    def sample(self, n, channels=3, size=64):
        x = torch.randn(n, channels, size, size, device=self.device)
        for t in reversed(range(self.T)):
            tb = torch.full((n,), t, device=self.device, dtype=torch.long)
            pred_noise = self.model(x, tb)
            
            a = self.alphas[t]; ab = self.alpha_bars[t]
            x = (1/a.sqrt()) * (x - (1-a)/((1-ab).sqrt()) * pred_noise)
            
            if t > 0:
                x += self.betas[t].sqrt() * torch.randn_like(x)
        return x
```

---

## Revision Questions

1. **Why does DDPM use a simplified loss instead of the full variational bound?**
2. **How does the cosine schedule improve over the linear schedule?**
3. **What is classifier-free guidance and why is it preferred?**
4. **How does Improved DDPM learn the variance?**
5. **Why does DDPM sampling require 1000 steps and how can this be reduced?**

---

[Previous: 02-forward-and-reverse-process.md](02-forward-and-reverse-process.md) | [Next: 04-score-matching.md](04-score-matching.md)
