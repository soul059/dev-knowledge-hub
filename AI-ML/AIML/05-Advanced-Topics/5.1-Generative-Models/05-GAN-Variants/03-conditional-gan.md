# Conditional GAN (cGAN)

## Overview

Conditional GAN (Mirza & Osindero, 2014) extends the standard GAN by conditioning both generator and discriminator on additional information — class labels, text descriptions, or images. This allows **controlled generation**: instead of random outputs, you can specify *what* to generate. cGAN is the foundation for text-to-image synthesis, image-to-image translation, and many practical applications.

---

## Concept

```
Standard GAN:
  G(z) → image       (no control over what is generated)
  D(x) → real/fake

Conditional GAN:
  G(z, y) → image    (y controls the output!)
  D(x, y) → real/fake (D checks if x matches condition y)

  y can be:
    • Class label (e.g., "digit 7", "cat")
    • Text embedding ("a red car on a road")
    • Another image (image-to-image translation)
    • Any auxiliary information

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Standard GAN:   z ──→ G ──→ random image        │
  │                                                    │
  │  Conditional GAN: z ─┐                            │
  │                      ├──→ G ──→ specific image    │
  │                  y ─┘                              │
  │                  (label)                           │
  │                                                    │
  │  y="dog" → generates dog                          │
  │  y="cat" → generates cat                          │
  │  y="car" → generates car                          │
  └──────────────────────────────────────────────────┘
```

---

## Architecture

```
Generator conditioning:

  Method 1: Concatenation (simple)
    z = [z_noise; y_embedding]  →  G → image
    
  Method 2: Projection (better for images)
    Map y to same dim as feature maps
    Multiply or add to intermediate features

  Method 3: Adaptive Instance Normalization
    Use y to predict γ, β for normalization layers


Discriminator conditioning:

  Method 1: Concatenation
    Input = [image; y_map]  →  D → real/fake
    y_map = y broadcast to image spatial dimensions

  Method 2: Projection (Miyato & Koyama, 2018)
    d = φ(x)ᵀ × ψ(y)  (inner product)
    Better gradient properties!

  ┌─────────────────────────────────────────────┐
  │  Projection Discriminator:                   │
  │                                               │
  │  image → CNN → features φ(x)                │
  │  label → Embedding → ψ(y)                   │
  │                                               │
  │  output = φ(x)ᵀψ(y) + linear(φ(x))         │
  │                                               │
  │  Much better than naive concatenation!        │
  └─────────────────────────────────────────────┘
```

---

## Training

```
Objective:

  L = E[log D(x, y)] + E[log(1 - D(G(z, y), y))]

  D must verify TWO things:
    1. Is the image realistic?
    2. Does the image match condition y?

  ┌──────────────────────────────────────────────────┐
  │  Training pairs:                                  │
  │                                                    │
  │  Real:  (image=dog_photo, y="dog") → D says REAL │
  │  Fake1: (image=G(z,"dog"), y="dog") → D says FAKE│
  │  Fake2: (image=dog_photo, y="cat") → D says FAKE!│
  │         (mismatched condition)                     │
  │                                                    │
  │  D learns: image must be realistic AND match y    │
  └──────────────────────────────────────────────────┘
```

---

## Conditional BatchNorm

```
Instead of standard BatchNorm:
  y = γ × (x - μ) / σ + β     (γ, β are learnable)

Conditional BatchNorm:
  γ = f_γ(y)    β = f_β(y)    (γ, β depend on condition!)
  output = γ(y) × (x - μ) / σ + β(y)

  f_γ and f_β are small networks (linear layers)

  Used in BigGAN, SPADE, and most modern conditional GANs

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Class "dog":  γ_dog, β_dog  →  dog features     │
  │  Class "cat":  γ_cat, β_cat  →  cat features     │
  │                                                    │
  │  Same network weights, different normalization!   │
  │  Very parameter-efficient conditioning            │
  └──────────────────────────────────────────────────┘
```

---

## Python: cGAN Implementation

```python
import torch
import torch.nn as nn

class ConditionalGenerator(nn.Module):
    def __init__(self, latent_dim=100, num_classes=10, img_channels=1):
        super().__init__()
        self.label_emb = nn.Embedding(num_classes, num_classes)
        
        self.net = nn.Sequential(
            nn.Linear(latent_dim + num_classes, 256),
            nn.BatchNorm1d(256),
            nn.ReLU(),
            nn.Linear(256, 512),
            nn.BatchNorm1d(512),
            nn.ReLU(),
            nn.Linear(512, 1024),
            nn.BatchNorm1d(1024),
            nn.ReLU(),
            nn.Linear(1024, img_channels * 28 * 28),
            nn.Tanh(),
        )
    
    def forward(self, z, labels):
        label_embed = self.label_emb(labels)
        x = torch.cat([z, label_embed], dim=1)
        return self.net(x).view(-1, 1, 28, 28)

class ConditionalDiscriminator(nn.Module):
    def __init__(self, num_classes=10, img_channels=1):
        super().__init__()
        self.label_emb = nn.Embedding(num_classes, 28 * 28)
        
        self.net = nn.Sequential(
            nn.Linear((img_channels + 1) * 28 * 28, 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 1),
            nn.Sigmoid(),
        )
    
    def forward(self, img, labels):
        label_map = self.label_emb(labels).view(-1, 1, 28, 28)
        x = torch.cat([img, label_map], dim=1).view(img.size(0), -1)
        return self.net(x)

def train_cgan(G, D, dataloader, latent_dim, device, epochs=50):
    criterion = nn.BCELoss()
    opt_G = torch.optim.Adam(G.parameters(), lr=2e-4, betas=(0.5, 0.999))
    opt_D = torch.optim.Adam(D.parameters(), lr=2e-4, betas=(0.5, 0.999))
    
    for epoch in range(epochs):
        for real_imgs, labels in dataloader:
            real_imgs, labels = real_imgs.to(device), labels.to(device)
            bs = real_imgs.size(0)
            ones = torch.ones(bs, 1, device=device)
            zeros = torch.zeros(bs, 1, device=device)
            
            # Train D
            z = torch.randn(bs, latent_dim, device=device)
            fake_imgs = G(z, labels).detach()
            d_loss = criterion(D(real_imgs, labels), ones) + \
                     criterion(D(fake_imgs, labels), zeros)
            opt_D.zero_grad(); d_loss.backward(); opt_D.step()
            
            # Train G
            z = torch.randn(bs, latent_dim, device=device)
            fake_imgs = G(z, labels)
            g_loss = criterion(D(fake_imgs, labels), ones)
            opt_G.zero_grad(); g_loss.backward(); opt_G.step()

# Generate specific digits
G.eval()
with torch.no_grad():
    z = torch.randn(10, 100, device=device)
    labels = torch.arange(10, device=device)  # digits 0-9
    generated = G(z, labels)  # one of each digit!
```

---

## Applications

```
  Application                 Condition y
  ───────────                 ───────────
  Class-conditional images    Class label (one-hot)
  Text-to-image              Text embedding (CLIP, BERT)
  Image-to-image             Source image (edges, masks)
  Super-resolution           Low-res image
  Colorization               Grayscale image
  Inpainting                 Masked image
  Style transfer             Style image/vector
  Face generation            Attributes (age, hair, etc.)
```

---

## Revision Questions

1. **How does conditioning change the GAN objective function?**
2. **Why must the discriminator also receive the condition y?**
3. **What is the difference between concatenation and projection conditioning?**
4. **How does Conditional BatchNorm work?**
5. **Give three real-world applications of conditional GANs.**

---

[Previous: 02-wgan-and-wgan-gp.md](02-wgan-and-wgan-gp.md) | [Next: 04-progressive-gan.md](04-progressive-gan.md)
