# Diffusion Process

## Overview

Diffusion models are a class of generative models inspired by **non-equilibrium thermodynamics**. The core idea is remarkably simple: gradually add noise to data until it becomes pure Gaussian noise (forward process), then learn to reverse this process (reverse/denoising process) to generate new data from noise. Diffusion models have become the dominant generative approach, powering Stable Diffusion, DALL-E 2, Midjourney, and Imagen.

---

## Intuition

```
Forward process: data → noise  (easy, fixed)
Reverse process: noise → data  (learned!)

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  Forward (add noise gradually):                           │
  │                                                            │
  │  x₀ → x₁ → x₂ → ... → x_T                              │
  │  🖼️ → 🌫️ → 🌫️🌫️ → ... → ⚪ (pure noise)               │
  │  data   slightly   more       Gaussian                    │
  │         noisy      noisy      N(0, I)                     │
  │                                                            │
  │  Reverse (denoise gradually):                             │
  │                                                            │
  │  x_T → x_{T-1} → ... → x₁ → x₀                          │
  │  ⚪ →  🌫️🌫️  → ... → 🌫️ → 🖼️                          │
  │  noise  less       less      clean                        │
  │         noisy      noisy     image!                       │
  │                                                            │
  │  We LEARN the reverse process!                            │
  │  Each step: predict & remove a small amount of noise     │
  └──────────────────────────────────────────────────────────┘

  Why does this work?
  • Each forward step adds tiny noise → easy to reverse!
  • A neural network can learn to remove small noise
  • Chain many small denoising steps → generate from scratch
```

---

## Forward Process (Diffusion)

```
Gradually add Gaussian noise over T timesteps:

  q(x_t | x_{t-1}) = N(x_t; √(1-β_t) × x_{t-1}, β_t × I)

  β_t = noise schedule (small values, e.g., 0.0001 to 0.02)
  T = total steps (typically 1000)

  At each step:
    x_t = √(1-β_t) × x_{t-1} + √(β_t) × ε    where ε ~ N(0, I)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  β_t schedule (linear):                          │
  │                                                    │
  │  β₁ = 0.0001                                     │
  │  β₂ = 0.0001 + small increment                  │
  │  ...                                               │
  │  β_T = 0.02                                      │
  │                                                    │
  │  Tiny noise at start → more noise later          │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Key property: can jump directly to any timestep!
  
  Define: α_t = 1 - β_t
         ᾱ_t = α₁ × α₂ × ... × α_t  (cumulative product)

  Then: q(x_t | x₀) = N(x_t; √ᾱ_t × x₀, (1-ᾱ_t) × I)

  In one step:
    x_t = √ᾱ_t × x₀ + √(1-ᾱ_t) × ε

  No need to iterate through all t steps!
  Crucial for efficient training.
```

---

## Properties of the Forward Process

```
As t → T:
  ᾱ_T → 0
  x_T ≈ ε ~ N(0, I)  (pure noise!)

As t → 0:
  ᾱ_0 → 1
  x_0 = original data (clean!)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Signal-to-Noise Ratio (SNR):                    │
  │                                                    │
  │  SNR(t) = ᾱ_t / (1 - ᾱ_t)                       │
  │                                                    │
  │  t=0:     SNR → ∞    (all signal)               │
  │  t=T/2:   SNR ≈ 1    (equal signal & noise)     │
  │  t=T:     SNR → 0    (all noise)                │
  │                                                    │
  │  SNR ▲                                            │
  │      │╲                                           │
  │      │  ╲                                         │
  │      │    ╲                                       │
  │  1   │─────╲─────                                │
  │      │       ╲                                    │
  │  0   │────────╲──►                                │
  │      0    T/2   T                                 │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Why Diffusion Works

```
Key insight: a small denoising step is EASY

  If x_t is slightly noisier than x_{t-1}:
    Predicting x_{t-1} from x_t ≈ simple denoising!
    Well-studied problem, neural networks can do this well.

  The trick: chain 1000 easy steps to do something hard:
    Generate a photorealistic image from pure noise!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Analogy: How to eat an elephant?                │
  │  One bite at a time!                              │
  │                                                    │
  │  How to generate an image from noise?            │
  │  One denoising step at a time!                   │
  │                                                    │
  │  Each step: trivially easy                       │
  │  All steps together: incredibly powerful          │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Compared to:
  • GAN: generate in ONE step (hard! → unstable)
  • VAE: decode in ONE step (hard! → blurry)
  • Diffusion: generate in 1000 easy steps (stable + sharp!)
```

---

## Noise Schedules

```
β_t schedule determines how fast noise is added:

  Linear:    β_t = β_min + t/T × (β_max - β_min)
  Cosine:    ᾱ_t = cos²(π/2 × (t/T + s)/(1+s))
  Sigmoid:   β_t = sigmoid(a + (b-a)×t/T)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Linear:  loses info too fast at early steps     │
  │           most of image destroyed before T/2     │
  │                                                    │
  │  Cosine (Nichol & Dhariwal, 2021):               │
  │           preserves info more uniformly            │
  │           better for smaller images               │
  │           ᾱ_t decreases smoothly                  │
  │                                                    │
  │  ᾱ_t  ▲                                          │
  │  1.0  │──╲                                       │
  │       │    ╲╲  cosine                            │
  │       │     ╲╲                                   │
  │       │    ╱  ╲╲                                 │
  │       │  ╱linear ╲                               │
  │  0.0  │──────────────►                            │
  │       0              T                            │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Forward Process

```python
import torch
import torch.nn.functional as F
import math

class DiffusionForwardProcess:
    def __init__(self, T=1000, beta_start=1e-4, beta_end=0.02, 
                 schedule='linear'):
        self.T = T
        
        if schedule == 'linear':
            self.betas = torch.linspace(beta_start, beta_end, T)
        elif schedule == 'cosine':
            steps = torch.arange(T + 1) / T
            alpha_bars = torch.cos((steps + 0.008) / 1.008 * math.pi / 2) ** 2
            alpha_bars = alpha_bars / alpha_bars[0]
            betas = 1 - alpha_bars[1:] / alpha_bars[:-1]
            self.betas = torch.clamp(betas, max=0.999)
        
        self.alphas = 1 - self.betas
        self.alpha_bars = torch.cumprod(self.alphas, dim=0)
    
    def add_noise(self, x0, t, noise=None):
        """
        q(x_t | x_0) - add noise to clean image.
        
        x_t = √ᾱ_t × x_0 + √(1-ᾱ_t) × ε
        """
        if noise is None:
            noise = torch.randn_like(x0)
        
        alpha_bar_t = self.alpha_bars[t].view(-1, 1, 1, 1)
        
        mean = torch.sqrt(alpha_bar_t) * x0
        std = torch.sqrt(1 - alpha_bar_t)
        
        x_t = mean + std * noise
        return x_t, noise
    
    def visualize_forward(self, x0, timesteps=[0, 250, 500, 750, 999]):
        """Show progressive noise addition."""
        results = {}
        for t in timesteps:
            t_tensor = torch.tensor([t])
            x_t, _ = self.add_noise(x0, t_tensor)
            results[t] = x_t
        return results

# Usage
diffusion = DiffusionForwardProcess(T=1000, schedule='cosine')
x0 = torch.randn(1, 3, 64, 64)  # clean image
t = torch.randint(0, 1000, (1,))
x_t, noise = diffusion.add_noise(x0, t)
```

---

## Revision Questions

1. **What is the forward diffusion process and why is it fixed (not learned)?**
2. **How can you jump directly to timestep t without iterating?**
3. **Why does breaking generation into many small steps help?**
4. **What is ᾱ_t and what does it represent?**
5. **Why is the cosine schedule preferred over linear?**

---

[Previous: ../06-Normalizing-Flows/06-applications.md](../06-Normalizing-Flows/06-applications.md) | [Next: 02-forward-and-reverse-process.md](02-forward-and-reverse-process.md)
