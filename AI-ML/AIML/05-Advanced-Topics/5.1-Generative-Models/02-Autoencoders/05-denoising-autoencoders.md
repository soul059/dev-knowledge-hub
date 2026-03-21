# Denoising Autoencoders

## Overview

A **denoising autoencoder (DAE)** is trained to reconstruct clean data from corrupted inputs. By intentionally adding noise to the input and training the network to remove it, the DAE learns more robust and meaningful features than a standard autoencoder. This technique also serves as a powerful regularizer, preventing the network from learning a trivial identity mapping.

---

## Concept

```
Standard AE:   x → encoder → z → decoder → x̂ ≈ x
Denoising AE:  x̃ → encoder → z → decoder → x̂ ≈ x  (original!)

  1. Take clean input x
  2. Corrupt it: x̃ = corrupt(x)
  3. Encode corrupted: z = encoder(x̃)
  4. Decode: x̂ = decoder(z)
  5. Loss: ||x̂ - x||²  ← compare with CLEAN x!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Clean x    Corrupt    Noisy x̃    Encode+Decode  │
  │  ████████ → ░█░█░█░░ → ░█░█░█░░ → ████████      │
  │  ████████   █░░█░░██   █░░█░░██   ████████      │
  │  ████████   ░░██░█░█   ░░██░█░█   ████████      │
  │                                                    │
  │  Network must learn STRUCTURE (not noise)         │
  │  to successfully denoise                          │
  └──────────────────────────────────────────────────┘

  Why this works:
    - Can't memorize: input is different every time
    - Must learn data manifold to "fill in" corrupted parts
    - Learns distribution of clean data implicitly
```

---

## Types of Corruption

```
1. GAUSSIAN NOISE:
   x̃ = x + ε,  ε ~ N(0, σ²)
   Most common, continuous corruption
   σ controls noise level

2. MASKING NOISE (dropout):
   Randomly set fraction of inputs to 0
   x̃_i = x_i × m_i,  m_i ~ Bernoulli(1-p)
   Similar to dropout, p = 0.3-0.5 typical
   Forces network to learn from partial info

3. SALT-AND-PEPPER NOISE:
   Randomly set pixels to 0 or 1
   Simulates dead/stuck pixels

4. STRUCTURED NOISE:
   Mask out rectangular patches
   Block out specific regions
   Similar to image inpainting

  ┌──────────────────────────────────────────────┐
  │ Corruption type     Effect                    │
  │ ───────────────     ──────                    │
  │ Gaussian noise      Blurred/fuzzy input       │
  │ Masking (dropout)   Missing features          │
  │ Salt-and-pepper     Random extreme pixels     │
  │ Patch masking       Missing regions           │
  └──────────────────────────────────────────────┘
```

---

## Mathematical Perspective

```
The DAE learns to move corrupted points back onto the data manifold:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Data manifold (true data lies here):             │
  │     ╱────────────╲                                │
  │    ╱              ╲                               │
  │   │  ● ● ● ● ●    │   ● = clean data             │
  │   │              │                                │
  │    ╲              ╱    ○ = corrupted data          │
  │     ╲────────────╱                                │
  │         ↑  ↑  ↑                                   │
  │      ○  ○  ○  ○         ← noise pushes off manifold│
  │                                                    │
  │  DAE learns: ○ → ● (project back onto manifold)  │
  │  This is equivalent to learning ∇ log P(x)        │
  │  (the score function / direction toward data)     │
  └──────────────────────────────────────────────────┘

  Connection to score matching:
    The DAE approximates:
      r(x̃) - x̃ ∝ ∇_x log P(x)|_{x=x̃}
    
    This is the "score" — the gradient of log probability.
    The DAE points from corrupted x̃ toward high-density regions.
    This insight led directly to diffusion models!
```

---

## Python: Denoising Autoencoder

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchvision import datasets, transforms
from torch.utils.data import DataLoader

class DenoisingAutoencoder(nn.Module):
    def __init__(self, input_dim=784, latent_dim=128):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 512), nn.ReLU(),
            nn.Linear(512, 256), nn.ReLU(),
            nn.Linear(256, latent_dim), nn.ReLU())
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 256), nn.ReLU(),
            nn.Linear(256, 512), nn.ReLU(),
            nn.Linear(512, input_dim), nn.Sigmoid())
    
    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z)

def add_noise(x, noise_type='gaussian', noise_level=0.3):
    if noise_type == 'gaussian':
        return x + noise_level * torch.randn_like(x)
    elif noise_type == 'masking':
        mask = torch.bernoulli(torch.full_like(x, 1 - noise_level))
        return x * mask
    elif noise_type == 'salt_pepper':
        noise = torch.rand_like(x)
        x_noisy = x.clone()
        x_noisy[noise < noise_level/2] = 0
        x_noisy[noise > 1 - noise_level/2] = 1
        return x_noisy
    return x

# Training
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Lambda(lambda x: x.view(-1))])

train_data = datasets.MNIST('.', train=True, download=True, transform=transform)
loader = DataLoader(train_data, batch_size=128, shuffle=True)

model = DenoisingAutoencoder()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

for epoch in range(20):
    total_loss = 0
    for x_clean, _ in loader:
        x_noisy = add_noise(x_clean, noise_type='gaussian', noise_level=0.4)
        x_noisy = x_noisy.clamp(0, 1)
        
        x_hat = model(x_noisy)
        loss = F.mse_loss(x_hat, x_clean)  # Compare with CLEAN input
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    
    print(f"Epoch {epoch+1}, Loss: {total_loss/len(loader):.4f}")
```

---

## DAE vs Standard AE

```
  Feature              Standard AE      Denoising AE
  ───────              ───────────      ────────────
  Input                Clean x          Corrupted x̃
  Target               x                x (clean)
  Risk of memorizing   Higher           Lower
  Feature quality      Good             Better (robust)
  Regularization       None built-in    Noise = regularizer
  Score learning       No               Yes (implicitly)
  Connection to        None             Foundation of
  diffusion models                      diffusion models!
```

---

## Revision Questions

1. **How does a denoising autoencoder differ from a standard autoencoder?**
2. **Why does training with corrupted inputs lead to better features?**
3. **What types of noise corruption are commonly used?**
4. **How is the denoising autoencoder connected to score matching?**
5. **Why is DAE considered a regularization technique?**

---

[Previous: 04-undercomplete-autoencoders.md](04-undercomplete-autoencoders.md) | [Next: 06-sparse-autoencoders.md](06-sparse-autoencoders.md)
