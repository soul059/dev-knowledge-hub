# Evaluation of Generative Models

## Overview

Evaluating generative models is uniquely challenging — unlike classification where accuracy is clear, "how good is a generated image?" is subjective and multi-dimensional. We need metrics for **quality** (are samples realistic?), **diversity** (does the model capture all modes?), and **novelty** (is it creating new content, not memorizing?). No single metric captures everything, so multiple evaluation approaches are used together.

---

## Key Evaluation Dimensions

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  1. QUALITY:  Are individual samples realistic?   │
  │  2. DIVERSITY: Does output cover all data modes?  │
  │  3. FIDELITY: Does output match conditions/prompts│
  │  4. NOVELTY:  Is it generating new content?       │
  │  5. COVERAGE: Does it represent the full dist.?   │
  │                                                    │
  │  Common failure modes:                            │
  │    High quality, low diversity → mode collapse     │
  │    High diversity, low quality → noisy outputs    │
  │    Perfect copies → memorization (no novelty)     │
  └──────────────────────────────────────────────────┘
```

---

## Inception Score (IS)

```
Measures quality AND diversity using Inception network:

  IS = exp(E_x [ KL(P(y|x) || P(y)) ])

  P(y|x): Inception classifier's prediction for generated image x
  P(y):   Marginal label distribution over all generated images

  High quality → P(y|x) is peaked (confident classification)
  High diversity → P(y) is uniform (many different classes)

  IS = exp(E[H(P(y)) - H(P(y|x))])
     = exp(high marginal entropy - low conditional entropy)
     = HIGH when both quality and diversity are good

  ┌────────────────────────────────────────────────┐
  │ IS interpretation:                              │
  │   IS ≈ 1:    Bad (random noise)                │
  │   IS ≈ 10:   Decent                            │
  │   IS ≈ 200+: ImageNet-level (real data ≈ 250)  │
  │                                                  │
  │ Limitations:                                    │
  │   • Only measures ImageNet classes              │
  │   • Ignores whether samples match REAL data     │
  │   • Can be fooled by adversarial examples       │
  └────────────────────────────────────────────────┘
```

---

## Fréchet Inception Distance (FID)

```
THE most commonly used metric. Compares statistics of generated 
vs real images in Inception feature space:

  FID = ||μ_r - μ_g||² + Tr(Σ_r + Σ_g - 2(Σ_r Σ_g)^{1/2})

  μ_r, Σ_r: mean and covariance of real image features
  μ_g, Σ_g: mean and covariance of generated image features

  Features extracted from Inception-v3 pool3 layer (2048-dim)

  FID = 0:    Generated = Real (perfect)
  FID < 10:   Excellent
  FID < 50:   Good
  FID > 100:  Poor

  ┌──────────────────────────────────────────────────┐
  │ FID captures both quality AND diversity:          │
  │                                                    │
  │ Real features:  ○○○○○  (cluster in feature space) │
  │ Good generator: ●●●●●  (overlaps with real)       │
  │ Bad quality:    ●  ●  ●  (scattered, noisy)       │
  │ Mode collapse:  ●●●      (tight cluster, partial) │
  │                                                    │
  │ FID measures distance between the two clusters    │
  └──────────────────────────────────────────────────┘

  Advantages over IS:
    ✓ Compares to real data (not just Inception classes)
    ✓ Captures diversity through covariance
    ✓ More stable and reliable
```

---

## Other Metrics

```
1. KERNEL INCEPTION DISTANCE (KID):
   Like FID but uses kernel (MMD) instead of Gaussian fit
   Unbiased estimator → works with fewer samples
   KID = MMD²(features_real, features_gen)

2. PRECISION AND RECALL:
   Precision: fraction of generated samples in real data support
              "How many generated samples are realistic?"
   Recall:    fraction of real data covered by generated samples
              "How much of the real data distribution is captured?"
   
   High precision, low recall → mode collapse (quality but no diversity)
   Low precision, high recall → noisy but diverse

3. LPIPS (Learned Perceptual Image Patch Similarity):
   Perceptual similarity between two images
   Uses deep features (VGG/AlexNet) not pixels
   Lower = more similar = better reconstruction

4. CLIP SCORE (text-to-image):
   cos_sim(CLIP(text), CLIP(image))
   Measures text-image alignment
   Higher = better prompt following

5. HUMAN EVALUATION:
   Gold standard but expensive and slow
   Turing test, preference ranking, MOS scores
```

---

## Likelihood-Based Evaluation

```
For models with tractable likelihood (VAE, flows, autoregressive):

  Log-likelihood: log P(x_test)
  
  Higher = model assigns higher probability to real data
  
  Bits per dimension (BPD):
    BPD = -log₂ P(x) / D
    D = data dimensionality
    Lower = better compression = better model
    
  ┌────────────────────────────────────────────────┐
  │ Caveat: High likelihood ≠ good samples!        │
  │                                                  │
  │ A model can assign high likelihood to real data│
  │ but generate poor samples (and vice versa).     │
  │                                                  │
  │ Flows often have high likelihood but lower     │
  │ sample quality than GANs or diffusion models.  │
  └────────────────────────────────────────────────┘
```

---

## Python: Computing FID

```python
import torch
import numpy as np
from scipy.linalg import sqrtm
from torchvision.models import inception_v3
from torchvision import transforms

def get_inception_features(images, model, batch_size=64):
    """Extract Inception-v3 features from images."""
    features = []
    for i in range(0, len(images), batch_size):
        batch = images[i:i+batch_size]
        with torch.no_grad():
            feat = model(batch)
        features.append(feat.cpu().numpy())
    return np.concatenate(features, axis=0)

def compute_fid(real_features, gen_features):
    """Compute Fréchet Inception Distance."""
    mu_r = np.mean(real_features, axis=0)
    mu_g = np.mean(gen_features, axis=0)
    sigma_r = np.cov(real_features, rowvar=False)
    sigma_g = np.cov(gen_features, rowvar=False)
    
    diff = mu_r - mu_g
    covmean = sqrtm(sigma_r @ sigma_g)
    
    # Handle numerical instability
    if np.iscomplexobj(covmean):
        covmean = covmean.real
    
    fid = diff @ diff + np.trace(sigma_r + sigma_g - 2 * covmean)
    return fid

# Example usage
# fid_score = compute_fid(real_feats, gen_feats)
# print(f"FID: {fid_score:.2f}")  # Lower is better
```

---

## Evaluation Summary

```
  Metric      Measures          Range       Better
  ──────      ────────          ─────       ──────
  IS          Quality+Diversity  1-∞        Higher
  FID         Quality+Diversity  0-∞        Lower
  KID         Quality+Diversity  0-∞        Lower
  Precision   Quality            0-1        Higher
  Recall      Diversity          0-1        Higher
  LPIPS       Perceptual sim.    0-1        Lower
  CLIP Score  Text alignment     0-1        Higher
  Log-lik.    Density fit        -∞ to 0    Higher
  BPD         Compression        0-∞        Lower
  Human eval  Overall quality    subjective  N/A
```

---

## Revision Questions

1. **Why is evaluating generative models harder than classification models?**
2. **How does FID capture both quality and diversity?**
3. **What is the difference between Precision and Recall in generative evaluation?**
4. **Why doesn't high log-likelihood guarantee good sample quality?**
5. **When would you use CLIP Score vs FID?**

---

[Previous: 03-applications-of-generative-ai.md](03-applications-of-generative-ai.md) | [Back to README](../README.md)
