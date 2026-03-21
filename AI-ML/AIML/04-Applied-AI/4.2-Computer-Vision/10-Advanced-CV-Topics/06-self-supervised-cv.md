# Self-Supervised Learning for Computer Vision

## Overview

**Self-supervised learning (SSL)** trains visual models on unlabeled images by creating supervision signals from the data itself. Instead of requiring expensive manual labels, SSL uses pretext tasks (like predicting image rotations or matching augmented views) to learn powerful visual representations that transfer to downstream tasks.

---

## Why Self-Supervised?

```
Labeled data is expensive:
  ImageNet:   14M images, years of annotation
  COCO:       330K images, $millions in annotation
  
  But: BILLIONS of unlabeled images exist online!

  ┌────────────────────────────────────────┐
  │  Labeled data     █                    │   << tiny
  │  Unlabeled data   ████████████████████ │   >> massive
  └────────────────────────────────────────┘

SSL paradigm:
  1. Pre-train on massive unlabeled data (SSL)
  2. Fine-tune on small labeled dataset (supervised)
  
  Result: Often BETTER than purely supervised training!
  
  SSL pre-trained (1B images) + fine-tune (1K labels)
    > Supervised (1M labeled images)
```

---

## Pretext Tasks (Early SSL)

```
Create labels from the data itself:

1. Rotation Prediction (Gidaris et al., 2018):
   Rotate image → predict rotation angle
   
   ┌────┐  ┌────┐  ┌────┐  ┌────┐
   │ 🏠 │  │🏠  │  │ 🏠│  │  🏠│
   │    │  │    │  │    │  │    │
   └────┘  └────┘  └────┘  └────┘
    0°      90°     180°    270°
   
   Model must understand object structure to predict rotation!

2. Jigsaw Puzzle (Noroozi & Favaro, 2016):
   Split image into patches, shuffle → predict original arrangement
   
   ┌──┬──┬──┐    ┌──┬──┬──┐
   │1 │2 │3 │ →  │5 │1 │8 │ → Predict: permutation index
   ├──┼──┼──┤    ├──┼──┼──┤
   │4 │5 │6 │    │3 │9 │2 │
   ├──┼──┼──┤    ├──┼──┼──┤
   │7 │8 │9 │    │6 │4 │7 │
   └──┴──┴──┘    └──┴──┴──┘

3. Colorization:
   Grayscale → predict colors
   Must understand objects to color them correctly

4. Inpainting:
   Mask region → predict missing content
```

---

## Contrastive Learning

### SimCLR (Chen et al., 2020)

```
Simple framework for contrastive learning:

  1. Take image x
  2. Apply two random augmentations → x_i, x_j (positive pair)
  3. Other images in batch → negative pairs
  4. Learn: similar augmentations close, different images far

  Image x
    │
  ┌─┴─┐
  │   │
  Aug1  Aug2        Random crop + flip + color jitter + blur
  │   │
  ┌▼┐ ┌▼┐
  │f│ │f│          Same encoder f (ResNet-50)
  └┬┘ └┬┘
  │   │
  ┌▼┐ ┌▼┐
  │g│ │g│          Projection head g (MLP: 2048→256→128)
  └┬┘ └┬┘
  z_i  z_j

  Loss: NT-Xent (Normalized Temperature-scaled Cross-Entropy)
  
  l(i,j) = -log(exp(sim(z_i,z_j)/τ) / Σ_k exp(sim(z_i,z_k)/τ))
  
  τ = temperature (0.07 typically)
  sim = cosine similarity
  
  Numerator: pull positive pair together
  Denominator: push negatives apart
  
  Key ingredients for success:
    - Large batch size (4096+)
    - Strong augmentations
    - Projection head (discard after training)
    - Temperature scaling
```

### MoCo (Momentum Contrast, He et al., 2020)

```
Key innovation: Momentum-updated encoder + queue of negatives

  ┌──────────┐          ┌──────────┐
  │  Query   │          │   Key    │
  │ encoder  │          │ encoder  │  (momentum update)
  │  f_q     │          │  f_k     │  θ_k ← m×θ_k + (1-m)×θ_q
  └────┬─────┘          └────┬─────┘  m = 0.999
       │                     │
    q (query)             k (key)
       │                     │
       │            ┌────────┤
       │            │   Queue of keys │
       │            │   [k1, k2, ..., kN] │ (65536 negatives!)
       │            └──────────────────┘
       │                     │
  Contrastive loss: q should match its own k,
                    not match other keys in queue

  Advantage over SimCLR: Don't need huge batch size
  Queue provides abundant negatives efficiently
```

---

## Non-Contrastive Methods

```
Problem with contrastive: needs many negatives

Solution: Learn without negatives!

BYOL (Bootstrap Your Own Latent, 2020):
  Two networks: online and target (momentum-updated)
  
  Online:  image → aug → encoder → projector → predictor → q
  Target:  image → aug → encoder → projector              → k
  
  Loss: ||q - k||²  (just match the two views!)
  
  Why doesn't it collapse to constant output?
    → Asymmetry (predictor only on online side)
    → Momentum update prevents collapse
    → Still debated!

SimSiam (Simple Siamese, 2021):
  Even simpler — no momentum encoder!
  
  ┌──────┐     ┌──────┐
  │ Aug 1│     │ Aug 2│
  └──┬───┘     └──┬───┘
     │             │
  ┌──▼───┐     ┌──▼───┐
  │Encoder│    │Encoder│  (shared weights)
  └──┬───┘     └──┬───┘
     │             │
  ┌──▼───┐        │
  │Predict│       │
  └──┬───┘        │
     │             │
  minimize ||p₁ - stopgrad(z₂)||²
  
  stop_gradient on one branch prevents collapse!

VICReg (2022):
  Variance-Invariance-Covariance regularization
  Explicitly prevents collapse via:
    - Variance: keep features from collapsing (high variance)
    - Invariance: match augmented views
    - Covariance: decorrelate feature dimensions
```

---

## Masked Image Modeling

### MAE (Masked Autoencoder, He et al., 2022)

```
Mask random patches → reconstruct pixels

  ┌──┬──┬──┬──┐     ┌──┬──┬──┬──┐
  │  │██│  │██│     │  │  │  │  │
  ├──┼──┼──┼──┤     ├──┼──┼──┼──┤
  │██│  │██│  │ →   │  │  │  │  │
  ├──┼──┼──┼──┤ MAE ├──┼──┼──┼──┤
  │  │██│  │██│     │  │  │  │  │
  ├──┼──┼──┼──┤     ├──┼──┼──┼──┤
  │██│  │██│  │     │  │  │  │  │
  └──┴──┴──┴──┘     └──┴──┴──┴──┘
  75% masked!       Reconstructed

  Architecture:
    Encoder (ViT): Only processes VISIBLE patches (25%)
    → Very efficient! (processes only 1/4 of patches)
    
    Decoder: Small transformer, reconstructs masked patches
    Loss: MSE on masked patches only
    
  Result: ViT-H pre-trained with MAE on ImageNet
    → 87.8% top-1 accuracy (state-of-the-art)
    → Works with only images, no labels!
```

### DINOv2 (Meta, 2023)

```
State-of-the-art self-supervised features:

  Combines:
    - Self-distillation (DINO: student-teacher with centering)
    - Masked image modeling (iBOT)
    - Large-scale curated data (LVD-142M)
    
  Result: Universal visual features that work for EVERYTHING:
    - Classification: competitive with supervised
    - Segmentation: strong zero-shot
    - Depth estimation: used as backbone for Depth Anything
    - Retrieval: excellent feature quality
    
  Key: Features are so good they transfer without fine-tuning!
```

---

## Comparison

| Method | Type | Negatives? | Batch Size | Key Innovation |
|--------|------|:---:|:---:|----------------|
| SimCLR | Contrastive | Yes (in-batch) | 4096 | Strong augmentation |
| MoCo v2 | Contrastive | Yes (queue) | 256 | Momentum encoder |
| BYOL | Non-contrastive | No | 4096 | Predictor + momentum |
| SimSiam | Non-contrastive | No | 256 | Stop-gradient |
| MAE | Masked modeling | No | 4096 | 75% masking + ViT |
| DINOv2 | Distillation + MIM | No | Large | Universal features |

---

## Python: Using SSL Models

```python
import torch

# DINOv2 features (zero-shot, no fine-tuning)
dinov2 = torch.hub.load('facebookresearch/dinov2', 'dinov2_vitl14')
dinov2.eval()

from PIL import Image
from torchvision import transforms

transform = transforms.Compose([
    transforms.Resize(224),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225])
])

img = transform(Image.open("cat.jpg")).unsqueeze(0)

with torch.no_grad():
    features = dinov2(img)  # 1×1024 global features
    patch_features = dinov2.forward_features(img)
    # patch_features["x_norm_patchtokens"]: 1×256×1024

print(f"Feature shape: {features.shape}")
# Use features for any downstream task:
# classification, retrieval, segmentation, etc.
```

---

## Revision Questions

1. **What is self-supervised learning and why is it important?**
2. **How does SimCLR create positive and negative pairs?**
3. **How does MoCo solve the large batch size problem of SimCLR?**
4. **Why don't non-contrastive methods (BYOL, SimSiam) collapse?**
5. **How does MAE achieve efficient pre-training with 75% masking?**
6. **Why are DINOv2 features considered "universal"?**

---

[Previous: 05-video-understanding.md](05-video-understanding.md) | [Back to README](../README.md)
