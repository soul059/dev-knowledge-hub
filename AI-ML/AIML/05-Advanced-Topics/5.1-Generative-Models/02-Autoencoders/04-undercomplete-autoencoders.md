# Undercomplete Autoencoders

## Overview

An **undercomplete autoencoder** has a latent space with fewer dimensions than the input, forcing the network to learn a compressed representation. This is the most common autoencoder type and is analogous to PCA when using linear activations, but far more powerful with nonlinear networks. The bottleneck prevents trivial identity mapping and ensures meaningful feature learning.

---

## Concept

```
Undercomplete: dim(z) < dim(x)

  Input: x ∈ ℝ^784
  Latent: z ∈ ℝ^32     ← fewer dimensions!
  Output: x̂ ∈ ℝ^784

  The network CANNOT copy input directly through the bottleneck.
  It MUST learn to compress → learn important features.

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  dim(x) = 784                                     │
  │      ↓                                             │
  │  dim(h₁) = 512                                    │
  │      ↓                                             │
  │  dim(h₂) = 256                                    │
  │      ↓                                             │
  │  dim(z) = 32    ← BOTTLENECK (undercomplete)      │
  │      ↓                                             │
  │  dim(h₃) = 256                                    │
  │      ↓                                             │
  │  dim(h₄) = 512                                    │
  │      ↓                                             │
  │  dim(x̂) = 784                                    │
  │                                                    │
  │  Information is LOST at bottleneck                │
  │  → only essential info preserved                  │
  └──────────────────────────────────────────────────┘
```

---

## Relationship to PCA

```
Linear undercomplete autoencoder = PCA!

  If encoder and decoder are single linear layers:
    z = W_enc × x       (encode)
    x̂ = W_dec × z      (decode)
    
  Training with MSE loss:
    min ||x - W_dec × W_enc × x||²
    
  Solution: W_enc spans top-k eigenvectors of covariance matrix
           = same as PCA!

  Nonlinear autoencoder:
    z = f(W₂ × ReLU(W₁ × x))     (nonlinear encoder)
    x̂ = g(W₄ × ReLU(W₃ × z))    (nonlinear decoder)
    
  → Can capture NONLINEAR structure that PCA cannot!

  ┌──────────────────────────────────────────────┐
  │  PCA:              Autoencoder:               │
  │  Linear only       Nonlinear                  │
  │  Global structure  Local + global             │
  │  Optimal for       Can learn any              │
  │  Gaussian data     manifold structure          │
  │                                                │
  │  Data on a curve:                             │
  │    PCA: projects onto line (loses shape)      │
  │    AE:  follows the curve (preserves shape)   │
  └──────────────────────────────────────────────┘
```

---

## Choosing Bottleneck Size

```
Too large → trivial identity mapping (no compression)
Too small → too much information lost (poor reconstruction)

  Experiment: vary latent dim, measure reconstruction error

  latent_dim    Recon. Error    Features learned
  ──────────    ────────────    ────────────────
       2        High            Very abstract (visualizable)
       8        Medium-High     Main shape + some detail
      32        Medium          Good balance
     128        Low             Rich features, some redundancy
     512        Very low        Near-perfect but may overfit

  Practical approach:
    1. Start with dim(z) = dim(x) / 10
    2. Reduce until reconstruction quality drops noticeably
    3. Pick smallest dim(z) with acceptable quality
```

---

## Python: Undercomplete AE vs PCA

```python
import torch
import torch.nn as nn
import numpy as np
from sklearn.decomposition import PCA

class UndercompleteAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=32):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 256), nn.ReLU(),
            nn.Linear(256, latent_dim))
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 256), nn.ReLU(),
            nn.Linear(256, input_dim), nn.Sigmoid())
    
    def forward(self, x):
        return self.decoder(self.encoder(x))

# Compare reconstruction error: AE vs PCA
def compare_ae_pca(data, latent_dim=32):
    # PCA
    pca = PCA(n_components=latent_dim)
    z_pca = pca.fit_transform(data)
    recon_pca = pca.inverse_transform(z_pca)
    pca_error = np.mean((data - recon_pca) ** 2)
    
    print(f"PCA ({latent_dim}D) MSE: {pca_error:.4f}")
    print(f"AE  ({latent_dim}D) MSE: <train and measure>")
    print(f"AE will be lower (nonlinear > linear)")
```

---

## Limitations

```
1. No generation guarantee:
   Decoding random z may produce garbage
   Latent space is NOT necessarily smooth or complete
   → VAE solves this with probabilistic latent space

2. May memorize:
   With enough capacity, can memorize training data
   Bottleneck helps but doesn't prevent all overfitting
   → Regularization (denoising, sparse, dropout) helps

3. Blurry reconstructions:
   MSE loss averages over possibilities → blurry
   Especially for complex images
   → Perceptual losses or adversarial losses help
```

---

## Revision Questions

1. **What makes an autoencoder "undercomplete"?**
2. **How does a linear undercomplete autoencoder relate to PCA?**
3. **Why can a nonlinear autoencoder outperform PCA?**
4. **How do you choose the right bottleneck dimension?**
5. **What are the limitations of undercomplete autoencoders for generation?**

---

[Previous: 03-latent-space.md](03-latent-space.md) | [Next: 05-denoising-autoencoders.md](05-denoising-autoencoders.md)
