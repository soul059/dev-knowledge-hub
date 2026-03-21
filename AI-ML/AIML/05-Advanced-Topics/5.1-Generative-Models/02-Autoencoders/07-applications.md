# Autoencoder Applications

## Overview

Autoencoders are versatile beyond simple compression. They excel at **anomaly detection** (recognizing what's unusual), **data denoising** (cleaning corrupted data), **dimensionality reduction** (visualizing high-dimensional data), **feature learning** (extracting useful representations), and **data generation** (creating new samples). This chapter covers practical applications across domains.

---

## 1. Anomaly Detection

```
Train autoencoder on NORMAL data only.
Anomalies → high reconstruction error (model hasn't seen them).

  Normal data: x → encoder → z → decoder → x̂ ≈ x  (low error)
  Anomaly:     x → encoder → z → decoder → x̂ ≠ x  (HIGH error!)

  ┌──────────────────────────────────────────────────┐
  │ Reconstruction error distribution:                │
  │                                                    │
  │  Count ▲                                          │
  │        │  ██                                       │
  │        │ ████                                      │
  │        │██████          ██                         │
  │        │████████      ████                         │
  │        └──────────────────── error →               │
  │          Normal      Anomalies                    │
  │          (low error)  (high error)                │
  │                                                    │
  │  Set threshold → classify as normal/anomaly       │
  └──────────────────────────────────────────────────┘

  Applications:
    • Manufacturing defect detection
    • Network intrusion detection
    • Credit card fraud detection
    • Medical diagnosis (unusual scans)
```

---

## 2. Image Denoising

```
Denoising autoencoder in practice:

  Noisy photo → DAE → clean photo

  Use cases:
    • Low-light photography enhancement
    • Medical image denoising (CT, MRI)
    • Satellite image cleanup
    • Old photo restoration

  Training: add synthetic noise, train to reconstruct clean
  Deployment: feed real noisy images
```

---

## 3. Dimensionality Reduction & Visualization

```
Nonlinear alternative to PCA / t-SNE:

  High-dim data → encoder → 2D or 3D → visualize

  Advantages over PCA:
    • Captures nonlinear structure
    • Better preservation of local geometry
    
  Advantages over t-SNE:
    • Has an encoder (can embed new points!)
    • t-SNE must recompute for new data

  Application: visualize gene expression data, customer segments
```

---

## 4. Feature Learning / Pretraining

```
Use encoder as feature extractor for downstream tasks:

  Step 1: Train autoencoder on unlabeled data (self-supervised)
  Step 2: Remove decoder
  Step 3: Use encoder features for classification/regression
  Step 4: Fine-tune on small labeled dataset

  ┌────────────────────────────────────────────────┐
  │  Pretraining (unsupervised):                    │
  │    Large unlabeled data → AE → encoder learned │
  │                                                  │
  │  Fine-tuning (supervised):                      │
  │    encoder(x) → classifier head → labels       │
  │    Small labeled data → fine-tune               │
  └────────────────────────────────────────────────┘

  Useful when labeled data is scarce but unlabeled is abundant
```

---

## 5. Data Compression

```
Learned compression (vs JPEG/PNG):

  Image → encoder → compact code → decoder → reconstructed

  Advantages:
    • Can outperform traditional codecs at low bitrates
    • Learned for specific data type (faces, medical, etc.)
    
  Google's work: learned image compression
    Better than JPEG at same file size
    Especially for very low bitrates
```

---

## 6. Image Super-Resolution

```
Low resolution → autoencoder → high resolution

  Input:  64×64 image
  Output: 256×256 image (4× upscaling)

  Architecture: encoder processes low-res, decoder generates high-res
  Loss: MSE + perceptual loss (VGG features) + adversarial loss
```

---

## Python: Anomaly Detection

```python
import torch
import torch.nn as nn
import numpy as np

class AnomalyDetectorAE(nn.Module):
    def __init__(self, input_dim, latent_dim=32):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 128), nn.ReLU(),
            nn.Linear(128, 64), nn.ReLU(),
            nn.Linear(64, latent_dim))
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 64), nn.ReLU(),
            nn.Linear(64, 128), nn.ReLU(),
            nn.Linear(128, input_dim))
    
    def forward(self, x):
        return self.decoder(self.encoder(x))

def detect_anomalies(model, data, threshold=None, percentile=95):
    """Detect anomalies based on reconstruction error."""
    model.eval()
    with torch.no_grad():
        recon = model(data)
        errors = ((data - recon) ** 2).mean(dim=1)
    
    if threshold is None:
        threshold = np.percentile(errors.numpy(), percentile)
    
    anomalies = errors > threshold
    return anomalies, errors, threshold

# Usage
model = AnomalyDetectorAE(input_dim=30)
# Train on normal data only...

# Detect anomalies on new data
is_anomaly, scores, thresh = detect_anomalies(model, test_data)
print(f"Found {is_anomaly.sum()} anomalies (threshold: {thresh:.4f})")
```

---

## Application Summary

```
  Application            AE Type          Domain
  ───────────            ───────          ──────
  Anomaly detection      Standard/VAE     Security, finance, QA
  Denoising              Denoising AE     Medical, photography
  Dim. reduction         Standard AE      Visualization, EDA
  Feature learning       Sparse/DAE       NLP, vision pretraining
  Compression            Conv AE          Image/video storage
  Super-resolution       Conv AE + GAN    Photography, medical
  Inpainting             Conv AE          Photo editing
  Drug discovery         VAE              Pharmaceutical
  Recommender systems    AE               E-commerce
  Speech enhancement     Conv AE          Audio processing
```

---

## Revision Questions

1. **How does an autoencoder detect anomalies?**
2. **Why is the encoder useful as a feature extractor for downstream tasks?**
3. **How does autoencoder-based compression differ from traditional codecs?**
4. **What makes denoising autoencoders suitable for image restoration?**
5. **What are the advantages of AE-based dimensionality reduction over PCA and t-SNE?**

---

[Previous: 06-sparse-autoencoders.md](06-sparse-autoencoders.md) | [Back to README](../README.md)
