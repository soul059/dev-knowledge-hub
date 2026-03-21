# VAE Applications

## Overview

VAEs are used far beyond simple image generation. Their structured latent space enables **controlled generation**, **interpolation**, **anomaly detection**, **semi-supervised learning**, **drug discovery**, and **data augmentation**. This chapter surveys practical VAE applications across domains.

---

## Image Generation and Interpolation

```
Generation: sample z ~ N(0,1), decode to image

Interpolation: smoothly blend between two images
  z₁ = encode(image_A)
  z₂ = encode(image_B)
  z_mid = α × z₁ + (1-α) × z₂
  image_mid = decode(z_mid)

  α=0.0     α=0.25    α=0.5     α=0.75    α=1.0
  Image_A →  blend  → halfway →  blend  → Image_B

  Works because VAE latent space is smooth and continuous!
  (Standard AE interpolation may produce garbage at midpoints)

Attribute manipulation:
  z_smile = average_z(smiling faces) - average_z(neutral faces)
  new_face = decode(encode(face) + z_smile)  → adds a smile!
```

---

## Anomaly Detection

```
Train VAE on normal data only.
Anomalies have high reconstruction error OR low ELBO.

  Score(x) = Reconstruction_error(x) + KL(q(z|x) || P(z))
  
  Normal:  low reconstruction error + low KL
  Anomaly: high reconstruction error OR unusual KL

  Better than standard AE because:
  - KL term detects unusual latent representations
  - Probabilistic framework gives calibrated scores
  
  Applications:
    Manufacturing defect detection
    Medical image screening
    Fraud detection
```

---

## Semi-Supervised Learning

```
VAE can leverage unlabeled data for classification!

  M2 model (Kingma et al., 2014):
    Labeled:   P(x, y, z) = P(x|y,z) P(z) P(y)
    Unlabeled: P(x, z) = Σ_y P(x|y,z) P(z) P(y)

  For labeled data: train with known y
  For unlabeled data: marginalize over all possible y

  Result: much better classification with few labels
  
  Example: 100 labeled + 50000 unlabeled MNIST
    Supervised only: 90% accuracy
    VAE semi-supervised: 97% accuracy!
```

---

## Drug Discovery and Molecular Generation

```
VAE for molecules:

  1. Encode molecule (SMILES string or graph) → z
  2. Optimize z for desired properties (drug-likeness, binding)
  3. Decode z → new molecule
  
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Molecule → Encoder → z → [optimize z] → Decoder│
  │                             ↓                     │
  │                        z_optimized                │
  │                             ↓                     │
  │                        New molecule               │
  │                        (better properties!)       │
  └──────────────────────────────────────────────────┘

  Latent space captures chemical properties:
    Similar molecules → nearby in z
    Interpolation → valid molecules along the path
    Optimization → find molecules with target properties
```

---

## Text Generation

```
VAE for text (sentence VAE):

  Encoder: sentence → z (compress meaning)
  Decoder: z → sentence (generate text)
  
  Enables:
    • Interpolation between sentences
    • Controlled style transfer
    • Paraphrase generation
  
  "The cat sat on the mat" → z₁
  "The dog ran in the park" → z₂
  
  decode(0.5 × z₁ + 0.5 × z₂):
  "The cat ran on the path"  (semantic blend!)
  
  Challenge: KL collapse in text VAEs
  Solution: KL annealing, word dropout, cyclical annealing
```

---

## Data Augmentation

```
Generate synthetic training data:

  1. Train VAE on limited dataset
  2. Sample from latent space
  3. Decode to generate new training examples
  4. Combine with real data for training classifier

  Especially useful for:
    • Medical imaging (rare diseases, few examples)
    • Manufacturing (rare defect types)
    • Class-imbalanced datasets (oversample rare class)

  Conditional VAE (CVAE):
    Condition on class label → generate specific classes
    P(x|z, y) → decoder takes z AND label y
    → "Generate me a rare defect image"
```

---

## Python: Conditional VAE

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ConditionalVAE(nn.Module):
    def __init__(self, input_dim=784, latent_dim=20, n_classes=10):
        super().__init__()
        # Encoder: input + label → z
        self.enc = nn.Sequential(
            nn.Linear(input_dim + n_classes, 512), nn.ReLU(),
            nn.Linear(512, 256), nn.ReLU())
        self.fc_mu = nn.Linear(256, latent_dim)
        self.fc_logvar = nn.Linear(256, latent_dim)
        
        # Decoder: z + label → output
        self.dec = nn.Sequential(
            nn.Linear(latent_dim + n_classes, 256), nn.ReLU(),
            nn.Linear(256, 512), nn.ReLU(),
            nn.Linear(512, input_dim), nn.Sigmoid())
        
        self.n_classes = n_classes
    
    def encode(self, x, y_onehot):
        h = self.enc(torch.cat([x, y_onehot], dim=1))
        return self.fc_mu(h), self.fc_logvar(h)
    
    def decode(self, z, y_onehot):
        return self.dec(torch.cat([z, y_onehot], dim=1))
    
    def forward(self, x, y):
        y_onehot = F.one_hot(y, self.n_classes).float()
        mu, logvar = self.encode(x, y_onehot)
        z = mu + torch.exp(0.5 * logvar) * torch.randn_like(mu)
        return self.decode(z, y_onehot), mu, logvar
    
    def generate(self, y, n_samples=16, latent_dim=20):
        """Generate samples of a specific class."""
        y_onehot = F.one_hot(
            torch.full((n_samples,), y, dtype=torch.long),
            self.n_classes).float()
        z = torch.randn(n_samples, latent_dim)
        return self.decode(z, y_onehot)
```

---

## Applications Summary

```
  Application           Domain           Key Advantage
  ───────────           ──────           ─────────────
  Image generation      Vision           Smooth latent space
  Interpolation         Vision/Audio     Meaningful blending
  Anomaly detection     Industry/Med     Probabilistic scores
  Semi-supervised       Classification   Few labels needed
  Drug discovery        Pharma           Latent optimization
  Text generation       NLP              Controlled generation
  Data augmentation     Any              Synthetic training data
  Music generation      Audio            Style interpolation
  Recommender systems   E-commerce       User/item embeddings
  Compression           Storage          Learned codecs
```

---

## Revision Questions

1. **Why is VAE interpolation smoother than standard AE interpolation?**
2. **How can a VAE be used for anomaly detection?**
3. **How does a conditional VAE (CVAE) enable class-specific generation?**
4. **Why are VAEs valuable for drug discovery?**
5. **What is the KL collapse problem in text VAEs and how is it addressed?**

---

[Previous: 06-beta-vae.md](06-beta-vae.md) | [Back to README](../README.md)
