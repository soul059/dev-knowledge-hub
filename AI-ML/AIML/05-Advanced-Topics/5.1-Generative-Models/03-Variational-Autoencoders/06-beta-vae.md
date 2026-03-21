# β-VAE

## Overview

**β-VAE** modifies the standard VAE by weighting the KL divergence term with a hyperparameter β. When β > 1, the model is pressured to learn a more **disentangled** latent representation — where each latent dimension captures a single, independent factor of variation. This simple modification has profound effects on representation quality and controllability.

---

## Formulation

```
Standard VAE loss:
  L = Reconstruction + KL
  L = E_q[log P(x|z)] - KL(q(z|x) || P(z))

β-VAE loss:
  L = E_q[log P(x|z)] - β × KL(q(z|x) || P(z))

  β = 1:   Standard VAE
  β > 1:   Stronger regularization → more disentangled
  β < 1:   Weaker regularization → better reconstruction
  β = 0:   Standard autoencoder (no KL)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  β = 0.1:  Great reconstruction, entangled z      │
  │  β = 1.0:  Standard VAE                          │
  │  β = 4.0:  Good disentanglement, blurrier output │
  │  β = 10:   Very disentangled, poor reconstruction│
  │                                                    │
  │  Higher β = simpler latent structure              │
  │            = more pressure to compress             │
  │            = each dimension must be efficient      │
  └──────────────────────────────────────────────────┘
```

---

## What Is Disentanglement?

```
Entangled latent space:
  Changing z₁ affects size AND color AND rotation
  Factors are mixed together
  
  z₁ = 0.3×size + 0.5×color + 0.2×rotation
  z₂ = 0.1×size + 0.4×color + 0.5×rotation

Disentangled latent space:
  Each z_i controls ONE factor independently
  
  z₁ = size
  z₂ = color
  z₃ = rotation
  z₄ = position
  
  Change z₁ → ONLY size changes, nothing else!

  ┌──────────────────────────────────────────────────┐
  │  Entangled:            Disentangled:              │
  │                                                    │
  │  z₁→ size+color        z₁ → size                  │
  │  z₂→ color+rotation    z₂ → color                 │
  │  z₃→ size+rotation     z₃ → rotation              │
  │                                                    │
  │  Hard to control!      Easy to control!           │
  │  Change one → many     Change one → one           │
  │  things change          thing changes              │
  └──────────────────────────────────────────────────┘

  Benefits of disentanglement:
    • Interpretable latent space
    • Controllable generation
    • Better transfer learning
    • More robust representations
    • Fairness (can remove sensitive factors)
```

---

## Why β > 1 Encourages Disentanglement

```
Information bottleneck perspective:

  β > 1 → stronger constraint on latent capacity
  → model must encode EFFICIENTLY
  → best strategy: one factor per dimension
  → independent factors = disentangled!

  Think of it as a communication constraint:
    Low bandwidth (high β): "Use as few bits as possible"
    → encode each factor in its own efficient channel
    → natural disentanglement!

  Statistical pressure:
    KL(q(z|x) || N(0,1)) penalizes deviations from N(0,1)
    High β → z must be very close to N(0,1)
    → only the most important factors get encoded
    → each factor uses its own dimension (most efficient)
```

---

## Traversal Analysis

```
To check disentanglement, traverse each latent dimension:

  Fix all dimensions, vary one at a time:
  
  z = [0, 0, 0, ..., 0]
  
  Vary z₃ from -3 to +3:
  
  z₃ = -3:  😐 small face
  z₃ = -1:  😐 medium face
  z₃ =  0:  😐 normal face
  z₃ = +1:  😐 large face
  z₃ = +3:  😐 very large face
  
  If ONLY size changes → z₃ is disentangled for "size"!
  If multiple things change → still entangled
```

---

## Python: β-VAE

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BetaVAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=10, beta=4.0):
        super().__init__()
        self.beta = beta
        
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 256), nn.ReLU(),
            nn.Linear(256, 128), nn.ReLU())
        self.fc_mu = nn.Linear(128, latent_dim)
        self.fc_logvar = nn.Linear(128, latent_dim)
        
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 128), nn.ReLU(),
            nn.Linear(128, 256), nn.ReLU(),
            nn.Linear(256, input_dim), nn.Sigmoid())
    
    def encode(self, x):
        h = self.encoder(x)
        return self.fc_mu(h), self.fc_logvar(h)
    
    def reparameterize(self, mu, logvar):
        std = torch.exp(0.5 * logvar)
        return mu + std * torch.randn_like(std)
    
    def decode(self, z):
        return self.decoder(z)
    
    def forward(self, x):
        mu, logvar = self.encode(x)
        z = self.reparameterize(mu, logvar)
        return self.decode(z), mu, logvar
    
    def loss(self, x, x_recon, mu, logvar):
        recon = F.binary_cross_entropy(x_recon, x, reduction='sum')
        kl = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
        return (recon + self.beta * kl) / x.size(0)

def latent_traversal(model, x, dim, values=torch.linspace(-3, 3, 10)):
    """Generate images by traversing one latent dimension."""
    model.eval()
    with torch.no_grad():
        mu, _ = model.encode(x.unsqueeze(0))
        images = []
        for v in values:
            z = mu.clone()
            z[0, dim] = v
            img = model.decode(z)
            images.append(img.squeeze())
    return images
```

---

## Variants

```
  Variant        Modification                    Purpose
  ───────        ────────────                    ───────
  β-VAE          β × KL                         Disentanglement
  β-TCVAE        Decompose KL into TC + others  Better disentangle.
  FactorVAE      Density ratio trick for TC      Disentanglement
  DIP-VAE        Penalize covariance of q(z)    Decorrelation
  Annealed β     β increases during training    Gradual pressure
  
  TC = Total Correlation = KL(q(z) || Π q(z_j))
    Measures statistical dependence between latent dims
    Minimizing TC → independent (disentangled) factors
```

---

## Revision Questions

1. **What does the β hyperparameter control in β-VAE?**
2. **What is disentanglement and why is it desirable?**
3. **Why does increasing β encourage disentanglement?**
4. **What is the tradeoff when using β > 1?**
5. **How do you evaluate whether a latent space is disentangled?**

---

[Previous: 05-vae-architecture.md](05-vae-architecture.md) | [Next: 07-vae-applications.md](07-vae-applications.md)
