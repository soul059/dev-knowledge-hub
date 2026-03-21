# VAE Architecture

## Overview

A complete VAE combines the encoder (inference network), the reparameterization trick, and the decoder (generative network) into an end-to-end trainable model. This chapter covers the full architecture for different data types (images, sequences), training procedures, and practical implementation details.

---

## Full Architecture

```
  ┌──────────────────────────────────────────────────────┐
  │                  VAE Architecture                      │
  │                                                        │
  │  Input x [784]                                        │
  │      │                                                 │
  │  ┌───▼───────────┐                                    │
  │  │   Encoder      │                                    │
  │  │  (Inference    │                                    │
  │  │   Network)     │                                    │
  │  │   q_φ(z|x)    │                                    │
  │  └───┬───────┬───┘                                    │
  │      │       │                                         │
  │     μ [20] log_var [20]                               │
  │      │       │                                         │
  │      └───┬───┘                                         │
  │          │  Reparameterize                             │
  │    ε~N(0,1) │  z = μ + σ×ε                            │
  │          │                                             │
  │      z [20]                                            │
  │          │                                             │
  │  ┌───▼───────────┐                                    │
  │  │   Decoder      │                                    │
  │  │  (Generative   │                                    │
  │  │   Network)     │                                    │
  │  │   P_θ(x|z)    │                                    │
  │  └───┬───────────┘                                    │
  │      │                                                 │
  │  Output x̂ [784]                                       │
  │                                                        │
  │  Loss = BCE(x, x̂) + KL(q(z|x) || N(0,1))            │
  └──────────────────────────────────────────────────────┘
```

---

## Convolutional VAE (for Images)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ConvVAE(nn.Module):
    def __init__(self, latent_dim=128, img_channels=3):
        super().__init__()
        # Encoder: image → features → μ, log_var
        self.encoder = nn.Sequential(
            nn.Conv2d(img_channels, 32, 4, 2, 1), nn.ReLU(),  # 64→32
            nn.Conv2d(32, 64, 4, 2, 1), nn.ReLU(),             # 32→16
            nn.Conv2d(64, 128, 4, 2, 1), nn.ReLU(),            # 16→8
            nn.Conv2d(128, 256, 4, 2, 1), nn.ReLU(),           # 8→4
            nn.Flatten())                                        # 256*4*4
        
        self.fc_mu = nn.Linear(256 * 4 * 4, latent_dim)
        self.fc_logvar = nn.Linear(256 * 4 * 4, latent_dim)
        
        # Decoder: z → features → image
        self.fc_decode = nn.Linear(latent_dim, 256 * 4 * 4)
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(256, 128, 4, 2, 1), nn.ReLU(),  # 4→8
            nn.ConvTranspose2d(128, 64, 4, 2, 1), nn.ReLU(),   # 8→16
            nn.ConvTranspose2d(64, 32, 4, 2, 1), nn.ReLU(),    # 16→32
            nn.ConvTranspose2d(32, img_channels, 4, 2, 1),     # 32→64
            nn.Sigmoid())
    
    def encode(self, x):
        h = self.encoder(x)
        return self.fc_mu(h), self.fc_logvar(h)
    
    def reparameterize(self, mu, log_var):
        std = torch.exp(0.5 * log_var)
        eps = torch.randn_like(std)
        return mu + std * eps
    
    def decode(self, z):
        h = self.fc_decode(z).view(-1, 256, 4, 4)
        return self.decoder(h)
    
    def forward(self, x):
        mu, log_var = self.encode(x)
        z = self.reparameterize(mu, log_var)
        return self.decode(z), mu, log_var
```

---

## Training Loop

```python
def train_vae(model, train_loader, epochs=50, lr=1e-3):
    optimizer = torch.optim.Adam(model.parameters(), lr=lr)
    
    for epoch in range(epochs):
        model.train()
        total_loss, total_recon, total_kl = 0, 0, 0
        
        for batch_idx, (x, _) in enumerate(train_loader):
            x_recon, mu, log_var = model(x)
            
            # Reconstruction loss
            recon_loss = F.binary_cross_entropy(
                x_recon, x, reduction='sum') / x.size(0)
            
            # KL divergence
            kl_loss = -0.5 * torch.sum(
                1 + log_var - mu.pow(2) - log_var.exp()) / x.size(0)
            
            loss = recon_loss + kl_loss
            
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
            total_recon += recon_loss.item()
            total_kl += kl_loss.item()
        
        n = len(train_loader)
        print(f"Epoch {epoch+1}: Loss={total_loss/n:.1f}, "
              f"Recon={total_recon/n:.1f}, KL={total_kl/n:.1f}")

# Generation
def generate(model, n_samples=16, latent_dim=128):
    model.eval()
    with torch.no_grad():
        z = torch.randn(n_samples, latent_dim)
        samples = model.decode(z)
    return samples
```

---

## KL Annealing (Warm-up)

```
Problem: KL collapse — encoder ignores z, decoder does everything

  Early training: KL pushes all q(z|x) → N(0,1)
  Decoder learns to ignore z → z carries no information!
  
  Solution: KL annealing (warm-up)
    Gradually increase KL weight from 0 to 1:
    
    L = Recon + β(t) × KL
    
    β(0) = 0     (no KL, pure autoencoder first)
    β(t) → 1     (full KL after warm-up period)

  ┌────────────────────────────────────────────┐
  │  β(t) ▲                                    │
  │   1.0 │            ─────────────────       │
  │       │          ╱                         │
  │       │        ╱                           │
  │   0.0 │──────╱                             │
  │       └─────────────────────── epoch →     │
  │         warm-up   full training            │
  └────────────────────────────────────────────┘
```

---

## Common Pitfalls

```
1. POSTERIOR COLLAPSE (KL = 0):
   Encoder outputs μ=0, σ=1 for all inputs
   z carries no information about x
   Fix: KL annealing, free bits, δ-VAE

2. BLURRY OUTPUTS:
   MSE/BCE loss averages over modes → blurry
   Fix: use perceptual loss, adversarial loss (VAE-GAN)

3. POOR LATENT UTILIZATION:
   Only a few latent dimensions are used
   Fix: increase latent dim, reduce decoder capacity

4. TRAINING INSTABILITY:
   Large KL spikes early in training
   Fix: gradient clipping, KL warm-up, lower learning rate
```

---

## Revision Questions

1. **What are the key components of a VAE architecture?**
2. **How does the convolutional VAE differ from a fully-connected one?**
3. **What is KL collapse and how does KL annealing prevent it?**
4. **Why do VAEs tend to produce blurry outputs?**
5. **How do you generate new samples from a trained VAE?**

---

[Previous: 04-reparameterization-trick.md](04-reparameterization-trick.md) | [Next: 06-beta-vae.md](06-beta-vae.md)
