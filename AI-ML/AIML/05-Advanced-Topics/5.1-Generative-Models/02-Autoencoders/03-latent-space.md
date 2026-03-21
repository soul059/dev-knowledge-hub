# Latent Space

## Overview

The **latent space** is the compressed, lower-dimensional representation learned by an autoencoder's bottleneck layer. It captures the essential features and structure of the data. Understanding latent spaces is crucial because they enable interpolation between data points, clustering, visualization, and — in generative models — sampling new data by decoding random latent vectors.

---

## What Is the Latent Space?

```
Input space:  High-dimensional (e.g., 784 pixels for MNIST)
Latent space: Low-dimensional (e.g., 2-32 dimensions)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Input Space (784D)        Latent Space (2D)      │
  │                                                    │
  │  ████  ████  ████          ● 0                    │
  │  ████  ████  ████         ●● 1                    │
  │  (many pixels)      →    ● 2   ●● 7              │
  │                           ● 3  ● 8                │
  │  Each image is a            ●● 4                  │
  │  point in 784D            ●● 5  ● 9              │
  │                            ● 6                    │
  │                                                    │
  │  Encoder compresses to 2D where similar           │
  │  digits cluster together                          │
  └──────────────────────────────────────────────────┘

  Each dimension in latent space captures a "feature":
    z₁ might encode digit identity (0-9)
    z₂ might encode writing style (slant, thickness)
    
  The autoencoder discovers these features automatically!
```

---

## Properties of Good Latent Spaces

```
1. CONTINUITY:
   Nearby points in latent space → similar data
   Small change in z → small change in decoded output
   
2. COMPLETENESS:
   Every point in latent space decodes to meaningful data
   No "dead zones" that produce garbage

3. DISENTANGLEMENT:
   Each latent dimension controls one factor of variation
   z₁ = rotation, z₂ = size, z₃ = color (independent)

4. SMOOTHNESS:
   Moving along any direction produces gradual changes
   Enables meaningful interpolation

  ┌──────────────────────────────────────────────────┐
  │ Good latent space:        Bad latent space:       │
  │                                                    │
  │  ●●●●●●●●●●              ●  ●      ●             │
  │  ●●●●●●●●●●                ●    ●                │
  │  ●●●●●●●●●●              ●        ●  ●           │
  │  (smooth, filled)         (gaps, holes → garbage) │
  │                                                    │
  │  Regular AE: may have gaps (not a generative model)│
  │  VAE: smooth, complete (proper generative model)  │
  └──────────────────────────────────────────────────┘
```

---

## Latent Space Interpolation

```
Linear interpolation between two data points:

  z_interp = α × z₁ + (1 - α) × z₂,    α ∈ [0, 1]

  Then decode: x_interp = decoder(z_interp)

  Example (faces):
    z₁ = encode(face_A)      z₂ = encode(face_B)
    
    α=0.0  α=0.25  α=0.5  α=0.75  α=1.0
    Face_A  →  →  blend  →  →  Face_B
    
    Smooth transition between two faces!
    Works if latent space is continuous and smooth.

  Spherical interpolation (slerp) is often better:
    z = sin((1-α)θ)/sin(θ) × z₁ + sin(αθ)/sin(θ) × z₂
    
    where θ = arccos(z₁·z₂ / ||z₁|| ||z₂||)
    
    Better for high-dimensional spaces where points
    lie on the surface of a hypersphere.
```

---

## Latent Space Arithmetic

```
Discovered in Word2Vec, works in image latent spaces too:

  z_smiling_woman - z_neutral_woman + z_neutral_man = z_smiling_man
  
  "smiling" = z_smiling - z_neutral  (direction in latent space)
  
  Add this direction to any face → makes it smile!

  ┌──────────────────────────────────────────────┐
  │                                                │
  │  Latent directions:                           │
  │    → smile direction                          │
  │    ↑ age direction                            │
  │    ↗ glasses direction                        │
  │                                                │
  │  z_person + direction = z_modified_person     │
  │                                                │
  │  These directions are found by averaging      │
  │  latent codes of images with/without          │
  │  the attribute and subtracting.               │
  └──────────────────────────────────────────────┘
```

---

## Dimensionality of Latent Space

```
Choosing latent dimension is a critical hyperparameter:

  Too small (e.g., 2):
    Can't capture all data variation
    Poor reconstruction quality
    But: easy to visualize!

  Too large (e.g., 512):
    Easy to reconstruct perfectly
    But: may memorize instead of generalize
    Latent space may have lots of empty regions

  Sweet spot depends on data complexity:
    MNIST:   8-32 dimensions
    CIFAR:   64-256 dimensions
    Faces:   128-512 dimensions
    
  Rule of thumb:
    Start with dim(z) ≈ sqrt(dim(x))
    Tune based on reconstruction quality vs generalization
```

---

## Python: Visualizing 2D Latent Space

```python
import torch
import numpy as np
import matplotlib.pyplot as plt

def visualize_latent_space(model, test_loader, device='cpu'):
    """Plot data colored by label in 2D latent space."""
    model.eval()
    latents, labels = [], []
    
    with torch.no_grad():
        for x, y in test_loader:
            x = x.view(x.size(0), -1).to(device)
            z = model.encoder(x)
            latents.append(z.cpu().numpy())
            labels.append(y.numpy())
    
    latents = np.concatenate(latents)
    labels = np.concatenate(labels)
    
    plt.figure(figsize=(10, 8))
    scatter = plt.scatter(latents[:, 0], latents[:, 1],
                          c=labels, cmap='tab10', s=2, alpha=0.7)
    plt.colorbar(scatter)
    plt.title('Latent Space Visualization')
    plt.xlabel('z₁')
    plt.ylabel('z₂')
    plt.show()

def generate_from_grid(model, n=20, latent_range=(-3, 3)):
    """Generate images from a grid over 2D latent space."""
    model.eval()
    grid_x = np.linspace(latent_range[0], latent_range[1], n)
    grid_y = np.linspace(latent_range[0], latent_range[1], n)
    
    figure = np.zeros((28 * n, 28 * n))
    
    for i, yi in enumerate(grid_y):
        for j, xi in enumerate(grid_x):
            z = torch.FloatTensor([[xi, yi]])
            with torch.no_grad():
                x_decoded = model.decoder(z).view(28, 28)
            figure[i*28:(i+1)*28, j*28:(j+1)*28] = x_decoded.numpy()
    
    plt.figure(figsize=(12, 12))
    plt.imshow(figure, cmap='gray')
    plt.title('Decoded Latent Space Grid')
    plt.show()
```

---

## Revision Questions

1. **What is the latent space and what does it represent?**
2. **What makes a latent space "good" for generation?**
3. **How does interpolation work in latent space?**
4. **What is latent space arithmetic and how is it used?**
5. **How do you choose the right dimensionality for the latent space?**

---

[Previous: 02-encoder-and-decoder.md](02-encoder-and-decoder.md) | [Next: 04-undercomplete-autoencoders.md](04-undercomplete-autoencoders.md)
