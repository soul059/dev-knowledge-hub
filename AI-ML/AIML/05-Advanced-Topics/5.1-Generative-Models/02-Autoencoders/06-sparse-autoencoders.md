# Sparse Autoencoders

## Overview

A **sparse autoencoder** adds a sparsity constraint to the latent representation, forcing most hidden units to be inactive (near zero) for any given input. Unlike undercomplete autoencoders that limit capacity by reducing dimensionality, sparse autoencoders can have an **overcomplete** latent space (more latent dims than input dims) while still learning useful features, because sparsity prevents trivial solutions.

---

## Concept

```
Standard AE: all latent units may be active
Sparse AE:   only a FEW latent units active per input

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Standard AE latent (32D):                        │
  │  [0.8, 0.5, 0.7, 0.3, 0.9, 0.4, 0.6, ...]      │
  │   Many units active                               │
  │                                                    │
  │  Sparse AE latent (128D, but sparse):             │
  │  [0.0, 0.0, 0.9, 0.0, 0.0, 0.0, 0.8, 0.0, ...]  │
  │   Most units are ~0, only a few active            │
  │                                                    │
  │  Each input activates a DIFFERENT subset!         │
  │  Input A: units {3, 7, 42} active                 │
  │  Input B: units {7, 15, 89} active                │
  │  → Each unit specializes for certain features     │
  └──────────────────────────────────────────────────┘

  Key insight:
    Overcomplete (dim(z) > dim(x)) + Sparsity
    = Large dictionary of features, used sparsely
    = Similar to how brains represent information!
```

---

## Sparsity Penalties

```
Add penalty to loss that discourages activation:

  L = Reconstruction loss + λ × Sparsity penalty

1. L1 PENALTY (simplest):
   L_sparse = λ × Σ|z_i|
   
   Pushes latent values toward exactly zero
   λ controls sparsity level

2. KL DIVERGENCE PENALTY:
   For each hidden unit j, compute average activation:
     ρ̂_j = (1/N) Σ z_j(x_i)    (average over dataset)
   
   Target: ρ (desired average activation, e.g., 0.05)
   
   L_sparse = Σ_j KL(ρ || ρ̂_j)
            = Σ_j [ρ log(ρ/ρ̂_j) + (1-ρ) log((1-ρ)/(1-ρ̂_j))]
   
   Penalizes when average activation differs from target ρ
   
   ρ = 0.05 → each unit active only 5% of the time

3. WINNER-TAKE-ALL:
   Only keep top-k activations, zero out rest
   Hard sparsity (exact k active units)
```

---

## Why Sparsity Helps

```
1. FEATURE LEARNING:
   Each unit learns a specific, interpretable feature
   Active for specific patterns, inactive otherwise
   Similar to V1 neurons in visual cortex

2. OVERCOMPLETE REPRESENTATIONS:
   Can have MORE latent dims than input dims
   Large feature dictionary → rich representation
   Sparsity prevents trivial copying

3. DISENTANGLEMENT:
   Different features activate for different inputs
   Easier to isolate individual factors of variation

4. ROBUSTNESS:
   Sparse codes are more robust to noise
   Small perturbation doesn't change which units are active

  ┌──────────────────────────────────────────────┐
  │ Sparse coding analogy:                        │
  │                                                │
  │ Dense: "This image is 30% feature A,          │
  │         25% feature B, 20% feature C, ..."    │
  │         (hard to interpret)                    │
  │                                                │
  │ Sparse: "This image IS feature 7 and 42"     │
  │          (clear, interpretable)                │
  └──────────────────────────────────────────────┘
```

---

## Python: Sparse Autoencoder

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SparseAutoencoder(nn.Module):
    def __init__(self, input_dim=784, latent_dim=256):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 512), nn.ReLU(),
            nn.Linear(512, latent_dim), nn.Sigmoid())
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 512), nn.ReLU(),
            nn.Linear(512, input_dim), nn.Sigmoid())
    
    def forward(self, x):
        z = self.encoder(x)
        x_hat = self.decoder(z)
        return x_hat, z

def kl_divergence_sparsity(rho, rho_hat):
    """KL divergence sparsity penalty."""
    return (rho * torch.log(rho / rho_hat) +
            (1 - rho) * torch.log((1 - rho) / (1 - rho_hat))).sum()

# Training with L1 sparsity
model = SparseAutoencoder(784, 256)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
lambda_sparse = 1e-3

for epoch in range(20):
    for x, _ in train_loader:
        x = x.view(x.size(0), -1)
        x_hat, z = model(x)
        
        recon_loss = F.mse_loss(x_hat, x)
        l1_penalty = lambda_sparse * z.abs().mean()
        
        loss = recon_loss + l1_penalty
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

# Training with KL divergence sparsity
rho = 0.05  # target sparsity
beta = 3.0  # sparsity weight

for epoch in range(20):
    for x, _ in train_loader:
        x = x.view(x.size(0), -1)
        x_hat, z = model(x)
        
        recon_loss = F.mse_loss(x_hat, x)
        
        rho_hat = z.mean(dim=0)  # average activation per unit
        kl_loss = beta * kl_divergence_sparsity(
            torch.full_like(rho_hat, rho), rho_hat)
        
        loss = recon_loss + kl_loss
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

---

## Sparse AE in Modern Context

```
Sparse autoencoders have seen a resurgence in:

1. MECHANISTIC INTERPRETABILITY:
   Train sparse AE on LLM hidden states
   Each sparse feature = an interpretable concept
   Used by Anthropic to understand Claude's internals
   
2. DICTIONARY LEARNING:
   Learn overcomplete dictionary of features
   Each data point = sparse combination of atoms
   
3. FEATURE VISUALIZATION:
   Sparse features correspond to interpretable patterns
   Easier to understand what the network has learned
```

---

## Revision Questions

1. **How does a sparse autoencoder differ from an undercomplete autoencoder?**
2. **Why can sparse autoencoders use overcomplete latent spaces?**
3. **What are the main sparsity penalties (L1 vs KL)?**
4. **How does sparsity lead to more interpretable features?**
5. **Why are sparse autoencoders important for mechanistic interpretability?**

---

[Previous: 05-denoising-autoencoders.md](05-denoising-autoencoders.md) | [Next: 07-applications.md](07-applications.md)
