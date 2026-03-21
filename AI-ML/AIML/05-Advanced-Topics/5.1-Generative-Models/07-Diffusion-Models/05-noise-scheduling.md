# Noise Scheduling

## Overview

The **noise schedule** — how quickly noise is added during the forward process — is a critical design choice that significantly impacts sample quality, training stability, and generation speed. From the original linear schedule to the widely-adopted cosine schedule and more recent learned/optimized schedules, choosing the right schedule can improve FID scores by 10-20% without changing the model architecture.

---

## Why Scheduling Matters

```
The schedule defines β_t (or equivalently ᾱ_t):

  ᾱ_t = Π (1 - β_i) for i=1..t

  ᾱ_t determines signal-to-noise ratio at step t:
    SNR(t) = ᾱ_t / (1 - ᾱ_t)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Bad schedule:                                    │
  │    • Info lost too fast → wasted steps at end    │
  │    • Info preserved too long → wasted at start   │
  │    • Both → fewer "useful" denoising steps       │
  │                                                    │
  │  Good schedule:                                   │
  │    • Uniform information destruction rate         │
  │    • Each step contributes meaningful denoising   │
  │    • All timesteps used efficiently              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Linear Schedule

```
Original DDPM schedule:

  β_t = β_min + (t-1)/(T-1) × (β_max - β_min)
  β_min = 0.0001, β_max = 0.02, T = 1000

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ᾱ_t with linear schedule:                       │
  │                                                    │
  │  1.0 ┤╲                                          │
  │      │ ╲                                          │
  │      │  ╲                                         │
  │  0.5 ┤   ╲                                       │
  │      │    ╲╲                                      │
  │      │      ╲╲╲                                   │
  │  0.0 ┤─────────╲╲──►                             │
  │      0    500    1000                              │
  │                                                    │
  │  Problem: image nearly destroyed by t=500        │
  │  Steps 500-1000 mostly redundant                 │
  │  → wasted computation and model capacity         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Cosine Schedule

```
Nichol & Dhariwal (2021):

  ᾱ_t = cos²((t/T + s)/(1+s) × π/2)
  s = 0.008 (small offset to prevent β near 0 at t=0)

  β_t = 1 - ᾱ_t / ᾱ_{t-1}
  β_t = min(β_t, 0.999)  (clip for stability)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ᾱ_t comparison:                                  │
  │                                                    │
  │  1.0 ┤──╲                                        │
  │      │    ╲╲  cosine (smooth, gradual)           │
  │      │      ╲╲                                    │
  │  0.5 ┤        ╲╲                                 │
  │      │   ╲      ╲╲                               │
  │      │   linear    ╲╲                            │
  │  0.0 ┤─────────────────►                         │
  │      0          500   1000                        │
  │                                                    │
  │  Cosine: preserves info more uniformly           │
  │  No "wasted" steps                               │
  │  Better for 64×64 and smaller images             │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Log-SNR Perspective

```
Analyzing schedules via log-SNR:

  log_SNR(t) = log(ᾱ_t / (1 - ᾱ_t))

  Ideal: log_SNR should decrease roughly linearly with t
  → each step removes equal "bits" of information

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  log_SNR:                                        │
  │                                                    │
  │  +10 ┤╲                                          │
  │      │ ╲  linear (too steep early)               │
  │   0  ┤──╲─╲──────────                            │
  │      │    ╲  ╲  cosine (more uniform slope)      │
  │  -10 ┤──────╲──╲──►                              │
  │      0         T                                  │
  │                                                    │
  │  Linear: log_SNR drops steeply, then flattens    │
  │  Cosine: log_SNR drops more uniformly            │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Kingma et al. (2021) proposed:
    Choose schedule endpoints based on data:
    SNR_max = high enough to nearly reconstruct
    SNR_min = low enough to be near pure noise
    Interpolate log_SNR linearly between them
```

---

## Schedules for High Resolution

```
Higher resolution images need different schedules:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Observation (Hoogeboom et al., 2023):           │
  │                                                    │
  │  64×64:   cosine schedule works well             │
  │  256×256: need to shift schedule                 │
  │  512×512: need to shift MORE                     │
  │  1024×1024: need even more shift                 │
  │                                                    │
  │  Why? Larger images have more redundancy         │
  │  More pixels → slower entropy loss per step      │
  │  Need more total noise to destroy information    │
  │                                                    │
  │  Solution:                                        │
  │  Shift SNR endpoints:                            │
  │    SNR_max ∝ resolution²                         │
  │    (more signal to start from)                   │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Stable Diffusion works in LATENT space (64×64)
  → avoids this resolution-dependent problem!
```

---

## Discrete vs. Continuous Time

```
Discrete: t ∈ {1, 2, ..., T}  with specific T
  + Simple to implement
  + Fixed number of steps
  - Quality depends on T choice
  - Interpolation between steps is ad-hoc

Continuous: t ∈ [0, 1]  with learned schedule
  + Resolution-independent
  + Can sample arbitrary number of steps
  + More principled
  - Slightly more complex

  Continuous formulation:
    β(t) = continuous function
    ᾱ(t) = exp(-∫₀ᵗ β(s) ds)
```

---

## Python: Schedule Implementations

```python
import torch
import math

def linear_schedule(T, beta_start=1e-4, beta_end=0.02):
    return torch.linspace(beta_start, beta_end, T)

def cosine_schedule(T, s=0.008):
    steps = torch.arange(T + 1) / T
    alpha_bars = torch.cos((steps + s) / (1 + s) * math.pi / 2) ** 2
    alpha_bars = alpha_bars / alpha_bars[0]
    betas = 1 - alpha_bars[1:] / alpha_bars[:-1]
    return torch.clamp(betas, max=0.999)

def sigmoid_schedule(T, start=-3, end=3):
    steps = torch.linspace(start, end, T)
    sigm = torch.sigmoid(steps)
    betas = 1 - sigm / sigm[0]  # normalize
    return torch.clamp(betas, min=1e-5, max=0.999)

def log_snr_schedule(T, snr_max=10.0, snr_min=-10.0):
    """Linear interpolation in log-SNR space."""
    log_snrs = torch.linspace(snr_max, snr_min, T)
    alpha_bars = torch.sigmoid(log_snrs)  # sigmoid(log_snr) = alpha_bar
    betas = torch.zeros(T)
    betas[0] = 1 - alpha_bars[0]
    betas[1:] = 1 - alpha_bars[1:] / alpha_bars[:-1]
    return torch.clamp(betas, min=1e-5, max=0.999)

def compare_schedules(T=1000):
    """Compare different schedules."""
    schedules = {
        'linear': linear_schedule(T),
        'cosine': cosine_schedule(T),
        'sigmoid': sigmoid_schedule(T),
    }
    
    for name, betas in schedules.items():
        alphas = 1 - betas
        alpha_bars = torch.cumprod(alphas, dim=0)
        snr = alpha_bars / (1 - alpha_bars)
        
        print(f"\n{name} schedule:")
        for t in [0, 250, 500, 750, 999]:
            print(f"  t={t}: ᾱ_t={alpha_bars[t]:.4f}, "
                  f"SNR={snr[t]:.2f}, "
                  f"log_SNR={torch.log(snr[t]):.2f}")

compare_schedules()
```

---

## Summary Table

| Schedule | Formula | Best For | Weakness |
|----------|---------|----------|----------|
| Linear | β linearly increasing | DDPM original | Info lost too fast |
| Cosine | ᾱ = cos² curve | Small images (≤64) | Not ideal for hi-res |
| Sigmoid | β follows sigmoid | Moderate images | Requires tuning |
| Log-SNR linear | Linear in log-SNR | Theory-optimal | Complex to implement |
| Shifted cosine | Cosine with offset | High-resolution | Per-resolution tuning |

---

## Revision Questions

1. **Why does the linear schedule waste computation in later timesteps?**
2. **How does the cosine schedule fix the linear schedule's problems?**
3. **What is the log-SNR and why should it decrease linearly?**
4. **Why do higher resolution images need different schedules?**
5. **What is the advantage of continuous-time over discrete-time schedules?**

---

[Previous: 04-score-matching.md](04-score-matching.md) | [Next: 06-ddim.md](06-ddim.md)
