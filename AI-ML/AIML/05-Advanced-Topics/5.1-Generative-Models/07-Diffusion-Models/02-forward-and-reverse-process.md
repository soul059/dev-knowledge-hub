# Forward and Reverse Process

## Overview

The forward process (q) progressively corrupts data into noise, while the reverse process (p) is a learned neural network that denoises step by step. The mathematical framework shows that the reverse process is also Gaussian when steps are small, and the neural network needs to predict either the noise, the clean data, or the score at each step. Understanding both processes is essential for training and sampling.

---

## Forward Process (q) — Recap and Details

```
q(x₁, ..., x_T | x₀) = Π q(x_t | x_{t-1})

  q(x_t | x_{t-1}) = N(x_t; √(1-β_t) × x_{t-1}, β_t I)

  Marginal at any t:
  q(x_t | x₀) = N(x_t; √ᾱ_t × x₀, (1-ᾱ_t) I)

  Posterior (needed for training):
  q(x_{t-1} | x_t, x₀) = N(x_{t-1}; μ̃_t, β̃_t I)

  where:
    μ̃_t = (√ᾱ_{t-1} × β_t)/(1-ᾱ_t) × x₀ 
         + (√α_t × (1-ᾱ_{t-1}))/(1-ᾱ_t) × x_t

    β̃_t = (1-ᾱ_{t-1})/(1-ᾱ_t) × β_t

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Key insight: q(x_{t-1} | x_t, x₀) is tractable │
  │  We know what the TRUE reverse step looks like!  │
  │  But it requires x₀ (which we don't have during │
  │  generation). So we LEARN to approximate it.     │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Reverse Process (p) — Learned

```
p_θ(x₀, ..., x_{T-1} | x_T) = p(x_T) Π p_θ(x_{t-1} | x_t)

  p(x_T) = N(0, I)   (start from noise)

  p_θ(x_{t-1} | x_t) = N(x_{t-1}; μ_θ(x_t, t), σ²_t I)

  The neural network predicts μ_θ(x_t, t):
  the denoised mean at each step

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Forward:  x₀ ──q──→ x₁ ──q──→ ... ──q──→ x_T  │
  │           (data)                          (noise) │
  │                                                    │
  │  Reverse:  x_T ──p_θ→ x_{T-1} ──p_θ→ ... → x₀  │
  │           (noise)                         (data!) │
  │                                                    │
  │  Each reverse step:                               │
  │    x_{t-1} = μ_θ(x_t, t) + σ_t × z              │
  │              (predicted mean) + (random noise)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## What Does the Network Predict?

```
Three equivalent parameterizations:

  1. Predict the NOISE ε_θ(x_t, t) ← most common (DDPM)
     μ_θ = 1/√α_t × (x_t - β_t/√(1-ᾱ_t) × ε_θ(x_t, t))
     
     Network learns: "what noise was added?"
     Then we subtract it!

  2. Predict the CLEAN IMAGE x̂₀(x_t, t)
     μ_θ = derived from x̂₀ using posterior formula
     
     Network learns: "what does the clean image look like?"
     
  3. Predict the SCORE ∇_x log q(x_t)
     Score = gradient of log density
     Equivalent to noise prediction (up to scaling)
     s_θ(x_t, t) = -ε_θ(x_t, t) / √(1-ᾱ_t)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  All three are mathematically equivalent!        │
  │                                                    │
  │  Predict noise:      ε_θ(x_t, t)  ← DDPM        │
  │  Predict clean:      x̂₀(x_t, t)  ← some papers  │
  │  Predict score:      s_θ(x_t, t)  ← score match  │
  │                                                    │
  │  Noise prediction is most stable in practice     │
  │  (uniform scale across timesteps)                │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Training Objective

```
Variational lower bound (ELBO):

  L = E[-log p_θ(x₀)] ≤ L_T + Σ L_t + L₀

  L_T = KL(q(x_T|x₀) || p(x_T))     ← constant (no params)
  L_t = KL(q(x_{t-1}|x_t,x₀) || p_θ(x_{t-1}|x_t))   ← main loss
  L₀ = -log p_θ(x₀|x₁)              ← reconstruction

  Simplified loss (Ho et al., DDPM):
  
  L_simple = E_{t,x₀,ε} [||ε - ε_θ(x_t, t)||²]

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Training algorithm:                              │
  │                                                    │
  │  1. Sample x₀ from data                          │
  │  2. Sample t ~ Uniform(1, T)                     │
  │  3. Sample ε ~ N(0, I)                           │
  │  4. Compute x_t = √ᾱ_t × x₀ + √(1-ᾱ_t) × ε   │
  │  5. Predict ε̂ = ε_θ(x_t, t)                     │
  │  6. Loss = ||ε - ε̂||²                           │
  │  7. Backpropagate, update θ                      │
  │                                                    │
  │  That's it! Just predict the noise!              │
  │  No adversarial training, no ELBO tricks         │
  └──────────────────────────────────────────────────┘
```

---

## Sampling Algorithm

```
  Algorithm: DDPM Sampling

  1. Sample x_T ~ N(0, I)
  2. For t = T, T-1, ..., 1:
       z ~ N(0, I) if t > 1, else z = 0
       ε̂ = ε_θ(x_t, t)
       x_{t-1} = 1/√α_t × (x_t - β_t/√(1-ᾱ_t) × ε̂) + σ_t × z
  3. Return x₀

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  x_T (noise)                                     │
  │   │                                               │
  │   ├── predict ε → remove some noise              │
  │   │   + add small random noise                   │
  │   ▼                                               │
  │  x_{T-1}                                         │
  │   │                                               │
  │   ├── predict ε → remove some noise              │
  │   │   + add small random noise                   │
  │   ▼                                               │
  │  x_{T-2}                                         │
  │   │                                               │
  │   ... (1000 steps)                                │
  │   │                                               │
  │   ▼                                               │
  │  x₀ (generated image!)                           │
  │                                                    │
  │  Variance σ_t choices:                           │
  │    σ²_t = β_t           (DDPM default)           │
  │    σ²_t = β̃_t           (optimal for known x₀)  │
  │    σ²_t = 0             (deterministic = DDIM)   │
  └──────────────────────────────────────────────────┘
```

---

## U-Net Architecture for Noise Prediction

```
  The neural network ε_θ is typically a U-Net:

  Input: (x_t, t) → noisy image + timestep

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Timestep t → sinusoidal embedding → MLP         │
  │                    → added to each ResBlock       │
  │                                                    │
  │  x_t (C×H×W)                                     │
  │    │                                               │
  │    ▼  Encoder                                     │
  │  [ResBlock + Attention] → skip                   │
  │  [ResBlock + Attention] → skip                   │
  │  [Downsample]                                     │
  │  [ResBlock + Attention] → skip                   │
  │  [ResBlock + Attention] → skip                   │
  │  [Downsample]                                     │
  │    │                                               │
  │    ▼  Bottleneck                                  │
  │  [ResBlock + Self-Attention + ResBlock]           │
  │    │                                               │
  │    ▼  Decoder                                     │
  │  [Upsample]                                       │
  │  [ResBlock + Attention + skip]                   │
  │  [ResBlock + Attention + skip]                   │
  │  [Upsample]                                       │
  │  [ResBlock + Attention + skip]                   │
  │  [ResBlock + Attention + skip]                   │
  │    │                                               │
  │    ▼                                               │
  │  ε̂ (C×H×W)  ← predicted noise (same size as input)│
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Complete Training and Sampling

```python
import torch
import torch.nn as nn

class SimpleDiffusion:
    def __init__(self, T=1000, beta_start=1e-4, beta_end=0.02, device='cpu'):
        self.T = T
        self.device = device
        
        self.betas = torch.linspace(beta_start, beta_end, T).to(device)
        self.alphas = 1 - self.betas
        self.alpha_bars = torch.cumprod(self.alphas, dim=0)
        self.sqrt_alpha_bars = torch.sqrt(self.alpha_bars)
        self.sqrt_one_minus_alpha_bars = torch.sqrt(1 - self.alpha_bars)
    
    def training_step(self, model, x0, optimizer):
        """One training step."""
        bs = x0.size(0)
        t = torch.randint(0, self.T, (bs,), device=self.device)
        noise = torch.randn_like(x0)
        
        # Forward process: add noise
        sqrt_ab = self.sqrt_alpha_bars[t].view(-1, 1, 1, 1)
        sqrt_one_minus_ab = self.sqrt_one_minus_alpha_bars[t].view(-1, 1, 1, 1)
        x_t = sqrt_ab * x0 + sqrt_one_minus_ab * noise
        
        # Predict noise
        noise_pred = model(x_t, t)
        loss = nn.functional.mse_loss(noise_pred, noise)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        return loss.item()
    
    @torch.no_grad()
    def sample(self, model, shape):
        """Generate samples via reverse process."""
        x = torch.randn(shape, device=self.device)
        
        for t in reversed(range(self.T)):
            t_batch = torch.full((shape[0],), t, device=self.device, dtype=torch.long)
            noise_pred = model(x, t_batch)
            
            alpha = self.alphas[t]
            alpha_bar = self.alpha_bars[t]
            beta = self.betas[t]
            
            # Compute mean
            mean = (1 / torch.sqrt(alpha)) * (
                x - beta / torch.sqrt(1 - alpha_bar) * noise_pred
            )
            
            # Add noise (except at t=0)
            if t > 0:
                z = torch.randn_like(x)
                sigma = torch.sqrt(beta)
                x = mean + sigma * z
            else:
                x = mean
        
        return x
```

---

## Revision Questions

1. **What is the posterior q(x_{t-1} | x_t, x₀) and why is it important?**
2. **What are the three parameterizations for what the network predicts?**
3. **Why is the simplified loss (predict ε) effective despite ignoring loss weighting?**
4. **How does the U-Net incorporate timestep information?**
5. **Why is noise added during sampling for t > 0 but not at t = 0?**

---

[Previous: 01-diffusion-process.md](01-diffusion-process.md) | [Next: 03-ddpm.md](03-ddpm.md)
