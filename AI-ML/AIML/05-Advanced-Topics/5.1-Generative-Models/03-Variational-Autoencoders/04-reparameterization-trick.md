# Reparameterization Trick

## Overview

The **reparameterization trick** is the key technical innovation that makes VAE training possible. Since we need to sample z from q(z|x) = N(μ, σ²), and sampling is a non-differentiable operation, gradients cannot flow through the sampling step. The trick rewrites the sampling as a deterministic transformation of a noise variable, making the entire pipeline differentiable and trainable with backpropagation.

---

## The Problem

```
VAE forward pass:
  x → encoder → (μ, σ) → SAMPLE z ~ N(μ, σ) → decoder → x̂

The sampling step is NON-DIFFERENTIABLE:
  z = sample(N(μ, σ))
  
  ∂z/∂μ = ???    Can't compute!
  ∂z/∂σ = ???    Can't compute!
  
  Sampling is a random operation — no gradient!
  → Can't backpropagate through the encoder!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  x → [Encoder] → μ,σ → ⊗ SAMPLE ⊗ → z → [Dec]  │
  │       θ_enc                ↑                       │
  │                       BLOCKED!                     │
  │                    No gradient here                │
  │                                                    │
  │  Backprop: ← ← ← ← ✗ ← ← ← ←                   │
  │                    Can't pass                      │
  └──────────────────────────────────────────────────┘
```

---

## The Solution

```
Instead of sampling z from N(μ, σ):

  z = μ + σ × ε,    where ε ~ N(0, 1)

  ε is sampled from standard normal (fixed, no parameters)
  μ and σ are deterministic outputs of the encoder
  
  The randomness is externalized to ε
  z is now a DETERMINISTIC FUNCTION of μ, σ, and ε

  ┌──────────────────────────────────────────────────┐
  │  Before (can't backprop):                         │
  │    μ, σ → [sample z ~ N(μ,σ)] → z                │
  │                                                    │
  │  After (CAN backprop!):                           │
  │    ε ~ N(0,1)  (external noise)                   │
  │    z = μ + σ × ε  (deterministic given ε)         │
  │                                                    │
  │  Gradients:                                       │
  │    ∂z/∂μ = 1       ✓                              │
  │    ∂z/∂σ = ε       ✓                              │
  │    ∂z/∂ε = σ       (but we don't need this)       │
  └──────────────────────────────────────────────────┘

  The computation graph becomes:

  x → [Encoder] → μ, log_var
                        ↓
        ε ~ N(0,1) → z = μ + exp(0.5 × log_var) × ε
                        ↓
                   [Decoder] → x̂
                   
  Now gradients flow through μ and σ!
```

---

## Why It Works

```
Mathematically equivalent:

  z ~ N(μ, σ²)
  
  is the SAME distribution as:
  
  ε ~ N(0, 1)
  z = μ + σ × ε

  Proof:
    E[z] = E[μ + σε] = μ + σ × E[ε] = μ + σ × 0 = μ  ✓
    Var[z] = Var[μ + σε] = σ² × Var[ε] = σ² × 1 = σ²  ✓

  Same distribution, but now z is differentiable w.r.t. μ and σ!

  Key insight: separate the RANDOMNESS (ε) from the PARAMETERS (μ, σ)
```

---

## In Practice: log_var Instead of σ

```
Encode log(σ²) instead of σ directly:

  Encoder outputs: μ, log_var = log(σ²)
  
  σ = exp(0.5 × log_var)
  z = μ + exp(0.5 × log_var) × ε

  Why log_var?
  1. σ must be positive → exp ensures this
  2. log_var can be any real number → no constraints
  3. Numerically more stable for very small/large σ
  4. KL formula simplifies:
     KL = -0.5 × Σ(1 + log_var - μ² - exp(log_var))
```

---

## Python Implementation

```python
import torch
import torch.nn as nn

class VAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=20):
        super().__init__()
        # Encoder
        self.enc = nn.Sequential(
            nn.Linear(input_dim, 512), nn.ReLU(),
            nn.Linear(512, 256), nn.ReLU())
        self.fc_mu = nn.Linear(256, latent_dim)
        self.fc_logvar = nn.Linear(256, latent_dim)
        
        # Decoder
        self.dec = nn.Sequential(
            nn.Linear(latent_dim, 256), nn.ReLU(),
            nn.Linear(256, 512), nn.ReLU(),
            nn.Linear(512, input_dim), nn.Sigmoid())
    
    def encode(self, x):
        h = self.enc(x)
        return self.fc_mu(h), self.fc_logvar(h)
    
    def reparameterize(self, mu, log_var):
        """The reparameterization trick."""
        std = torch.exp(0.5 * log_var)   # σ = exp(log_var / 2)
        eps = torch.randn_like(std)       # ε ~ N(0, 1)
        z = mu + std * eps                # z = μ + σ × ε
        return z
    
    def decode(self, z):
        return self.dec(z)
    
    def forward(self, x):
        mu, log_var = self.encode(x)
        z = self.reparameterize(mu, log_var)
        x_recon = self.decode(z)
        return x_recon, mu, log_var

# At test time (generation):
# z = torch.randn(batch_size, latent_dim)  # sample from N(0,1)
# x_new = model.decode(z)                  # no reparameterization needed
```

---

## Computation Graph

```
  ┌──────────────────────────────────────────────────────┐
  │                                                        │
  │  Forward pass:                                        │
  │                                                        │
  │  x ──→ encoder ──→ μ ──────────────┐                  │
  │                      log_var ──┐    │                  │
  │                                │    │                  │
  │                    exp(0.5×lv) = σ  │                  │
  │                                │    │                  │
  │  ε ~ N(0,1) ──────→ σ × ε ──→ + ──→ z ──→ decoder → x̂│
  │  (no gradient)                                        │
  │                                                        │
  │  Backward pass (gradients):                           │
  │                                                        │
  │  ∂L/∂μ = ∂L/∂z × ∂z/∂μ = ∂L/∂z × 1                 │
  │  ∂L/∂σ = ∂L/∂z × ∂z/∂σ = ∂L/∂z × ε                 │
  │  ∂L/∂ε = not needed (ε has no parameters)            │
  │                                                        │
  └──────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **Why can't we backpropagate through a sampling operation?**
2. **How does the reparameterization trick make sampling differentiable?**
3. **Show that z = μ + σε with ε~N(0,1) has the same distribution as z~N(μ,σ²).**
4. **Why do we encode log(σ²) instead of σ directly?**
5. **Is the reparameterization trick needed at generation time? Why or why not?**

---

[Previous: 03-elbo.md](03-elbo.md) | [Next: 05-vae-architecture.md](05-vae-architecture.md)
