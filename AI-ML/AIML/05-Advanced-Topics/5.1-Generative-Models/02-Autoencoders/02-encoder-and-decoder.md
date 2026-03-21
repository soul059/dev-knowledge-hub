# Encoder and Decoder

## Overview

The **encoder** and **decoder** are the two halves of an autoencoder. The encoder compresses input data into a compact latent representation, while the decoder reconstructs the original data from this representation. Their design — depth, width, activation functions, and architecture — critically determines what the autoencoder can learn.

---

## Encoder

```
The encoder maps high-dimensional input to low-dimensional latent code:

  z = f_θ(x)    where dim(z) << dim(x)

  Structure: progressively reduces dimensionality

  FC Encoder (MNIST):
    Input (784) → 512 → 256 → 128 → Latent (32)
    
  Conv Encoder (images):
    Input (3×64×64) → Conv+Pool → Conv+Pool → Flatten → Latent
    Spatial dims shrink: 64→32→16→8→4
    Channels grow: 3→32→64→128→256

  ┌──────────────────────────────────────────┐
  │  Encoder layers:                          │
  │                                           │
  │  x [784] ─────────────────── wide         │
  │      ↓                                    │
  │  h₁ [512] ──────────────── narrower       │
  │      ↓                                    │
  │  h₂ [256] ────────────── narrower         │
  │      ↓                                    │
  │  h₃ [128] ──────────── narrower           │
  │      ↓                                    │
  │  z  [32]  ────────── bottleneck           │
  │                                           │
  │  Each layer extracts higher-level features│
  └──────────────────────────────────────────┘

  Key design choices:
    - Number of layers (depth)
    - Hidden dimensions (width)
    - Activation functions (ReLU, LeakyReLU, Tanh)
    - No activation on final layer (latent can be any value)
```

---

## Decoder

```
The decoder maps latent code back to original dimension:

  x̂ = g_φ(z)    where dim(x̂) = dim(x)

  Structure: mirror of encoder (progressively increases dimensionality)

  FC Decoder (MNIST):
    Latent (32) → 128 → 256 → 512 → Output (784)
    
  Conv Decoder (images):
    Latent → Reshape → ConvTranspose → ConvTranspose → Output
    Spatial dims grow: 4→8→16→32→64
    Channels shrink: 256→128→64→32→3

  ┌──────────────────────────────────────────┐
  │  Decoder layers (mirror of encoder):     │
  │                                           │
  │  z  [32]  ────────── narrow               │
  │      ↓                                    │
  │  h₃ [128] ──────────── wider              │
  │      ↓                                    │
  │  h₂ [256] ────────────── wider            │
  │      ↓                                    │
  │  h₁ [512] ──────────────── wider          │
  │      ↓                                    │
  │  x̂ [784] ─────────────────── original dim│
  └──────────────────────────────────────────┘

  Output activation depends on data:
    Binary data (MNIST 0-1): Sigmoid
    Real-valued (normalized): Tanh or none
    Images (0-255): Sigmoid × 255
```

---

## Transposed Convolutions

```
ConvTranspose2d: "learnable upsampling" for decoders

  Regular Conv: reduces spatial size
    Input: 8×8, Kernel: 3×3, Stride: 2 → Output: 4×4

  Transposed Conv: increases spatial size
    Input: 4×4, Kernel: 3×3, Stride: 2 → Output: 8×8

  ┌──────────────────────────────────────────┐
  │ Conv (encoder):      4×4 → 2×2          │
  │ ConvTranspose (dec): 2×2 → 4×4          │
  │                                          │
  │ NOT the inverse of convolution!          │
  │ It's a different operation that          │
  │ happens to increase spatial dimensions.  │
  └──────────────────────────────────────────┘

  Alternatives to ConvTranspose:
    1. Upsample + Conv:
       nn.Upsample(scale_factor=2) + nn.Conv2d(...)
       Avoids checkerboard artifacts
    
    2. PixelShuffle:
       Rearranges channels into spatial dimensions
       nn.PixelShuffle(upscale_factor=2)
```

---

## Symmetric vs Asymmetric Design

```
Symmetric (most common):
  Encoder: [784, 512, 256, 128, 32]
  Decoder: [32, 128, 256, 512, 784]
  
  Mirror architecture → balanced capacity

Asymmetric:
  Encoder: [784, 512, 256, 32]      (fewer layers)
  Decoder: [32, 128, 256, 512, 784] (more layers)
  
  Sometimes decoder needs more capacity
  (reconstruction is harder than encoding)

Skip connections (U-Net style):
  Encoder features are concatenated with decoder features
  Helps preserve spatial details
  Used in image segmentation, not standard autoencoders
```

---

## Python: Flexible Encoder-Decoder

```python
import torch
import torch.nn as nn

class Encoder(nn.Module):
    def __init__(self, input_dim, hidden_dims, latent_dim):
        super().__init__()
        layers = []
        in_dim = input_dim
        for h_dim in hidden_dims:
            layers.extend([
                nn.Linear(in_dim, h_dim),
                nn.BatchNorm1d(h_dim),
                nn.ReLU()])
            in_dim = h_dim
        layers.append(nn.Linear(in_dim, latent_dim))
        self.net = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.net(x)

class Decoder(nn.Module):
    def __init__(self, latent_dim, hidden_dims, output_dim):
        super().__init__()
        layers = []
        in_dim = latent_dim
        for h_dim in hidden_dims:
            layers.extend([
                nn.Linear(in_dim, h_dim),
                nn.BatchNorm1d(h_dim),
                nn.ReLU()])
            in_dim = h_dim
        layers.extend([
            nn.Linear(in_dim, output_dim),
            nn.Sigmoid()])
        self.net = nn.Sequential(*layers)
    
    def forward(self, z):
        return self.net(z)

class FlexibleAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32,
                 hidden_dims=[512, 256, 128]):
        super().__init__()
        self.encoder = Encoder(input_dim, hidden_dims, latent_dim)
        self.decoder = Decoder(latent_dim, hidden_dims[::-1], input_dim)
    
    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z)

# Example
model = FlexibleAE(784, 32, [512, 256, 128])
x = torch.randn(16, 784)
x_hat = model(x)  # x_hat.shape = [16, 784]
```

---

## Revision Questions

1. **What is the role of the encoder vs the decoder?**
2. **Why is the decoder typically a mirror of the encoder?**
3. **What is a transposed convolution and why is it used in decoders?**
4. **How does the output activation function depend on the data type?**
5. **What are the alternatives to transposed convolutions for upsampling?**

---

[Previous: 01-autoencoder-architecture.md](01-autoencoder-architecture.md) | [Next: 03-latent-space.md](03-latent-space.md)
