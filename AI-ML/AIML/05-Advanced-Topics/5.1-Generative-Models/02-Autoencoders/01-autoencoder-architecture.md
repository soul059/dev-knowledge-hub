# Autoencoder Architecture

## Overview

An **autoencoder** is a neural network that learns to compress data into a lower-dimensional representation and then reconstruct it. The network is forced through a bottleneck (latent space), which compels it to learn the most important features of the data. Autoencoders are the foundation of many generative models and are used for dimensionality reduction, denoising, anomaly detection, and feature learning.

---

## Architecture

```
  ┌───────────────────────────────────────────────────────┐
  │                    AUTOENCODER                         │
  │                                                        │
  │  Input x          Latent z           Output x̂         │
  │  [784]             [32]              [784]             │
  │                                                        │
  │  ████████         ████              ████████           │
  │  ████████    →    ████    →         ████████           │
  │  ████████         ████              ████████           │
  │  ████████         (bottleneck)      ████████           │
  │                                                        │
  │  ┌────────┐      ┌──────┐         ┌────────┐          │
  │  │Encoder │  →   │Latent│   →     │Decoder │          │
  │  │f_θ(x)  │      │Space │         │g_φ(z)  │          │
  │  └────────┘      └──────┘         └────────┘          │
  │                                                        │
  │  x ───→ z = f_θ(x) ───→ x̂ = g_φ(z)                  │
  │                                                        │
  │  Goal: x̂ ≈ x (reconstruct input)                     │
  │  Constraint: dim(z) << dim(x) (bottleneck)            │
  └───────────────────────────────────────────────────────┘
```

---

## How It Works

```
1. ENCODER: Maps input to latent representation
   z = f_θ(x)
   Compresses high-dimensional input to low-dimensional code
   
2. LATENT SPACE (Bottleneck): The compressed representation
   dim(z) << dim(x)
   Forces network to learn essential features
   
3. DECODER: Reconstructs input from latent code
   x̂ = g_φ(z)
   Maps latent code back to original dimension

Training objective:
  L = ||x - x̂||²  (MSE loss for reconstruction)
  or
  L = -Σ [x_i log(x̂_i) + (1-x_i) log(1-x̂_i)]  (BCE for binary)

  "Make the output as close to the input as possible"

  ┌────────────────────────────────────────────┐
  │ The autoencoder is NOT just memorizing!    │
  │                                             │
  │ The bottleneck forces it to learn a        │
  │ compressed representation:                  │
  │                                             │
  │ 784 pixels → 32 features → 784 pixels     │
  │                                             │
  │ Those 32 features must capture the         │
  │ essence of the image!                       │
  └────────────────────────────────────────────┘
```

---

## Types of Autoencoders

```
  Type              Bottleneck        Special Property
  ────              ──────────        ────────────────
  Undercomplete     dim(z) < dim(x)   Basic compression
  Overcomplete      dim(z) > dim(x)   Needs regularization
  Denoising         + noise to input  Robust features
  Sparse            + sparsity penalty Active feature selection
  Variational (VAE) + probabilistic z  Generative model
  Contractive       + Jacobian penalty Robust to perturbations
```

---

## Python: Basic Autoencoder

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchvision import datasets, transforms
from torch.utils.data import DataLoader

class Autoencoder(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32):
        super().__init__()
        # Encoder
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 256),
            nn.ReLU(),
            nn.Linear(256, 128),
            nn.ReLU(),
            nn.Linear(128, latent_dim))
        
        # Decoder
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 256),
            nn.ReLU(),
            nn.Linear(256, input_dim),
            nn.Sigmoid())
    
    def forward(self, x):
        z = self.encoder(x)          # Encode
        x_hat = self.decoder(z)      # Decode
        return x_hat
    
    def encode(self, x):
        return self.encoder(x)

# Training
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Lambda(lambda x: x.view(-1))])

train_data = datasets.MNIST('.', train=True, download=True, transform=transform)
loader = DataLoader(train_data, batch_size=128, shuffle=True)

model = Autoencoder(784, 32)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

for epoch in range(20):
    total_loss = 0
    for x, _ in loader:
        x_hat = model(x)
        loss = F.mse_loss(x_hat, x)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    
    print(f"Epoch {epoch+1}, Loss: {total_loss/len(loader):.4f}")
```

---

## Convolutional Autoencoder

```python
class ConvAutoencoder(nn.Module):
    def __init__(self):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Conv2d(1, 16, 3, stride=2, padding=1),  # 28→14
            nn.ReLU(),
            nn.Conv2d(16, 32, 3, stride=2, padding=1), # 14→7
            nn.ReLU(),
            nn.Conv2d(32, 64, 7))                       # 7→1
        
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(64, 32, 7),               # 1→7
            nn.ReLU(),
            nn.ConvTranspose2d(32, 16, 3, stride=2,
                               padding=1, output_padding=1), # 7→14
            nn.ReLU(),
            nn.ConvTranspose2d(16, 1, 3, stride=2,
                               padding=1, output_padding=1), # 14→28
            nn.Sigmoid())
    
    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z)
```

---

## Revision Questions

1. **What are the three main components of an autoencoder?**
2. **Why is the bottleneck (latent space) essential?**
3. **What loss function is used to train an autoencoder?**
4. **How is an autoencoder different from PCA for dimensionality reduction?**
5. **Why does an autoencoder use Sigmoid activation in the decoder output?**

---

[Next: 02-encoder-and-decoder.md](02-encoder-and-decoder.md) | [Back to README](../README.md)
