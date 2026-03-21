# DDIM (Denoising Diffusion Implicit Models)

## Overview

DDIM (Song et al., 2021) addresses DDPM's biggest limitation: the requirement for hundreds or thousands of denoising steps during sampling. DDIM reformulates the reverse process as a **non-Markovian** (deterministic) process that can skip steps, reducing sampling from 1000 steps to as few as 10-50 steps with minimal quality loss. The same trained DDPM model can be sampled with DDIM — no retraining needed!

---

## DDPM's Sampling Problem

```
DDPM sampling:
  x_T → x_{T-1} → x_{T-2} → ... → x₁ → x₀
  
  1000 sequential neural network evaluations!
  ~20-60 seconds per image (depending on GPU)

  Each step is Markovian: x_{t-1} depends ONLY on x_t
  Must go through ALL steps

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  DDPM: 1000 steps  (slow but high quality)       │
  │  x_T → x_999 → x_998 → ... → x₁ → x₀          │
  │                                                    │
  │  DDIM: 50 steps   (fast, same quality!)          │
  │  x_T → x_980 → x_960 → ... → x_20 → x₀        │
  │        (skip 20 steps at a time)                  │
  │                                                    │
  │  Same trained model! Different sampling process! │
  └──────────────────────────────────────────────────┘
```

---

## DDIM Formulation

```
Key idea: define a NON-MARKOVIAN forward process
that has the SAME marginals q(x_t|x₀) as DDPM

  DDPM: q(x_{t-1}|x_t)            (depends only on x_t)
  DDIM: q(x_{t-1}|x_t, x₀)       (depends on x_t AND x₀)

  DDIM update rule:

  x_{t-1} = √ᾱ_{t-1} × x̂₀ + √(1-ᾱ_{t-1}-σ²_t) × ε_θ + σ_t × z

  where:
    x̂₀ = (x_t - √(1-ᾱ_t) × ε_θ(x_t,t)) / √ᾱ_t    (predicted x₀)
    ε_θ = predicted noise from trained model
    σ_t = controls stochasticity

  When σ_t = √(β̃_t): equivalent to DDPM (stochastic)
  When σ_t = 0:        fully DETERMINISTIC (DDIM)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  σ_t = 0:  Deterministic sampling                │
  │            Same noise → same image (reproducible)│
  │            Can skip many steps                    │
  │                                                    │
  │  σ_t > 0:  Stochastic sampling                  │
  │            Different noise → different image     │
  │            Need more steps for quality           │
  │                                                    │
  │  η parameter: σ_t = η × √(β̃_t)                 │
  │  η = 0: fully deterministic (DDIM)              │
  │  η = 1: fully stochastic (DDPM)                 │
  └──────────────────────────────────────────────────┘
```

---

## Subsequence Sampling

```
Instead of t = [1000, 999, 998, ..., 1]:
Use a SUBSEQUENCE: τ = [1000, 950, 900, ..., 50]

  Only need to evaluate ε_θ at subsequence timesteps!

  Choosing subsequences:
    Uniform:     τ = [1000, 800, 600, 400, 200]  (5 steps)
    Quadratic:   τ = [1000, 750, 500, 250, 100]  
    Custom:      optimized for best quality

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Steps vs Quality:                                │
  │                                                    │
  │  Steps    FID (CIFAR-10)    Time                 │
  │  ─────    ──────────────    ────                 │
  │  1000     3.17 (DDPM)      60s                   │
  │  100      4.04 (DDIM)       6s                   │
  │  50       4.67              3s                    │
  │  20       6.84              1.2s                  │
  │  10       13.36             0.6s                  │
  │                                                    │
  │  50 steps ≈ 95% of DDPM quality at 5% cost!     │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Deterministic Encoding

```
Because DDIM is deterministic (σ=0):
  x₀ → x_T is also deterministic!

  Forward encoding: given x₀, compute x_T
  Reverse decoding: given x_T, compute x₀

  x₀ ↔ x_T  is a BIJECTION (like flows!)

  This enables:
    1. Image interpolation in latent space
    2. Image editing via latent manipulation
    3. Consistent reconstructions

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Image A → encode → x_T^A                       │
  │  Image B → encode → x_T^B                       │
  │                                                    │
  │  Interpolation:                                   │
  │  x_T^interp = slerp(x_T^A, x_T^B, α)           │
  │  decode(x_T^interp) → smooth transition!         │
  │                                                    │
  │  Spherical interpolation (slerp) works better    │
  │  than linear interpolation in high-dim noise     │
  └──────────────────────────────────────────────────┘
```

---

## DDIM Inversion

```
Reverse the sampling process to get the latent:

  Given x₀, find x_T such that DDIM(x_T) ≈ x₀

  Forward DDIM (encoding):
  x_{t+1} = √ᾱ_{t+1} × x̂₀ + √(1-ᾱ_{t+1}) × ε_θ(x_t, t)

  x₀ → x₁ → x₂ → ... → x_T

  Then modify x_T (or intermediate ε) → decode → edited image!

  Used extensively in image editing:
    • SDEdit
    • Prompt-to-Prompt
    • Null-text Inversion
```

---

## Python: DDIM Sampler

```python
import torch
import numpy as np

class DDIMSampler:
    def __init__(self, model, T=1000, device='cpu'):
        self.model = model
        self.T = T
        self.device = device
        
        betas = torch.linspace(1e-4, 0.02, T).to(device)
        self.alphas = 1 - betas
        self.alpha_bars = torch.cumprod(self.alphas, dim=0)
    
    def get_timestep_subsequence(self, num_steps):
        """Create evenly spaced subsequence."""
        step_size = self.T // num_steps
        return list(range(0, self.T, step_size))
    
    @torch.no_grad()
    def sample(self, shape, num_steps=50, eta=0.0):
        """
        DDIM sampling.
        eta=0: deterministic, eta=1: equivalent to DDPM
        """
        timesteps = self.get_timestep_subsequence(num_steps)
        timesteps = list(reversed(timesteps))
        
        x = torch.randn(shape, device=self.device)
        
        for i in range(len(timesteps)):
            t = timesteps[i]
            t_prev = timesteps[i + 1] if i + 1 < len(timesteps) else 0
            
            t_batch = torch.full((shape[0],), t, device=self.device, dtype=torch.long)
            
            # Predict noise
            eps = self.model(x, t_batch)
            
            # Predict x₀
            ab_t = self.alpha_bars[t]
            x0_pred = (x - torch.sqrt(1 - ab_t) * eps) / torch.sqrt(ab_t)
            x0_pred = torch.clamp(x0_pred, -1, 1)
            
            # DDIM update
            ab_prev = self.alpha_bars[t_prev] if t_prev > 0 else torch.tensor(1.0)
            
            # Compute sigma
            sigma = eta * torch.sqrt(
                (1 - ab_prev) / (1 - ab_t) * (1 - ab_t / ab_prev)
            )
            
            # Direction pointing to x_t
            dir_xt = torch.sqrt(1 - ab_prev - sigma**2) * eps
            
            # DDIM step
            x = torch.sqrt(ab_prev) * x0_pred + dir_xt
            
            if sigma > 0 and t_prev > 0:
                x += sigma * torch.randn_like(x)
        
        return x
    
    @torch.no_grad()
    def encode(self, x0, num_steps=50):
        """DDIM inversion: x₀ → x_T (deterministic encoding)."""
        timesteps = self.get_timestep_subsequence(num_steps)
        x = x0
        
        for i in range(len(timesteps) - 1):
            t = timesteps[i]
            t_next = timesteps[i + 1]
            
            t_batch = torch.full((x0.size(0),), t, device=self.device, dtype=torch.long)
            eps = self.model(x, t_batch)
            
            ab_t = self.alpha_bars[t]
            ab_next = self.alpha_bars[t_next]
            
            x0_pred = (x - torch.sqrt(1 - ab_t) * eps) / torch.sqrt(ab_t)
            x = torch.sqrt(ab_next) * x0_pred + torch.sqrt(1 - ab_next) * eps
        
        return x
    
    @torch.no_grad()
    def interpolate(self, x1, x2, num_interp=10, num_steps=50):
        """Interpolate between two images via latent space."""
        z1 = self.encode(x1, num_steps)
        z2 = self.encode(x2, num_steps)
        
        results = []
        for alpha in torch.linspace(0, 1, num_interp):
            # Spherical interpolation (slerp)
            omega = torch.acos(torch.clamp(
                (z1 * z2).sum() / (z1.norm() * z2.norm()), -1, 1
            ))
            z_interp = (torch.sin((1-alpha)*omega) * z1 + 
                       torch.sin(alpha*omega) * z2) / torch.sin(omega)
            
            img = self.sample_from_latent(z_interp, num_steps)
            results.append(img)
        
        return results
```

---

## Revision Questions

1. **Why can DDIM skip timesteps while DDPM cannot?**
2. **What does the η parameter control in DDIM?**
3. **Why is deterministic sampling useful for image editing?**
4. **How does DDIM inversion work and what is it used for?**
5. **What is the trade-off between number of sampling steps and quality?**

---

[Previous: 05-noise-scheduling.md](05-noise-scheduling.md) | [Next: ../08-Stable-Diffusion-and-DALL-E/01-latent-diffusion-models.md](../08-Stable-Diffusion-and-DALL-E/01-latent-diffusion-models.md)
