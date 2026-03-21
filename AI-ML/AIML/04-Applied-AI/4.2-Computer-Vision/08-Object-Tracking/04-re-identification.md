# Re-Identification (Re-ID)

## Overview

**Re-Identification (Re-ID)** recognizes the same object (typically a person) across different cameras, viewpoints, or time gaps. It answers: "Is this the same person seen earlier?" Re-ID is fundamental to multi-camera tracking, retrieval systems, and long-term identity maintenance.

---

## Problem Definition

```
Camera A (10:15 AM):         Camera B (10:17 AM):
┌──────────────────┐         ┌──────────────────┐
│                  │         │                  │
│   ┌────┐        │         │        ┌────┐    │
│   │    │        │    ?    │        │    │    │
│   │ 🧑 │  ──────┼────=────┼────── │ 🧑 │    │ Same person?
│   │    │        │         │        │    │    │
│   └────┘        │         │        └────┘    │
│                  │         │                  │
└──────────────────┘         └──────────────────┘

Challenges:
  - Different viewpoints (front vs back)
  - Different lighting / cameras
  - Pose changes (walking vs standing)
  - Partial occlusion
  - Clothing changes (long-term Re-ID)
  - Resolution differences
  - Background variation
```

---

## Approach: Metric Learning

```
Learn an embedding space where:
  - Same person → close embeddings
  - Different person → far embeddings

  ┌──────────┐     ┌──────────┐     ┌──────┐
  │ Person   │ →  │   CNN    │ →  │ 2048 │ → L2 normalize → 128-D
  │ crop     │     │ backbone │     │  →128│
  │ 256×128  │     │ (ResNet) │     │      │
  └──────────┘     └──────────┘     └──────┘

  Embedding space:
  
         ●A1  ●A2        Same person A: close together
                          
                          
    ■B1                   Person B: far from A
         ■B2
                          
              ▲C1  ▲C2    Person C: far from A and B
```

---

## Loss Functions

### Triplet Loss

```
Select triplets: (anchor, positive, negative)
  Anchor: person A, image 1
  Positive: person A, image 2 (same person)
  Negative: person B, image 1 (different person)

  L = max(0, d(a,p) - d(a,n) + margin)

  ●anchor ──d(a,p)── ●positive     d(a,p) should be small
       \                            d(a,n) should be large
        \──d(a,n)── ■negative       margin = 0.3 typically

  Hard mining: Select hardest triplets
    Hardest positive: same person, max distance
    Hardest negative: different person, min distance
    → Much faster convergence

  Semi-hard: d(a,p) < d(a,n) < d(a,p) + margin
    → In the margin zone, most informative
```

### Cross-Entropy (ID Loss)

```
Treat Re-ID as classification during training:
  Each identity = one class
  
  Feature → FC layer → softmax → class probabilities
  Loss = -log(p_correct_identity)

  At inference: discard FC layer, use features directly

  Surprisingly effective! Often combined with triplet loss.
```

### Combined Loss

```
L_total = L_ID + λ × L_triplet

  ID loss:      Learns separable features
  Triplet loss: Learns metric structure
  Together:     Best of both worlds

  Advanced losses:
    Circle Loss:   Unified pair-based learning
    ArcFace Loss:  Angular margin for better separation
    Center Loss:   Pull features toward class centers
```

---

## Architecture: Strong Baseline

```
Luo et al. (2019) "Bag of Tricks" — Simple but effective

  Input (256×128×3)
       │
  ┌────▼────┐
  │ ResNet-50│  Backbone (pretrained on ImageNet)
  │ (modified)│  Remove last spatial downsampling
  └────┬────┘
       │ 
  [8×4×2048]  Feature map
       │
  ┌────▼─────────────┐
  │ Global Average    │  → 2048-D global feature
  │ Pooling (GAP)     │
  └────┬─────────────┘
       │
  ┌────▼────┐
  │ BNNeck  │  Batch normalization (no bias)
  │         │  → separates ID loss and triplet loss
  └────┬────┘
       │
  ┌────┼────────────┐
  │    │            │
  ┌──▼───┐  ┌────▼────┐
  │Triplet│  │ ID Loss │
  │ Loss  │  │(softmax)│
  │(before│  │(after   │
  │ BN)   │  │ BN)     │
  └───────┘  └─────────┘

  BNNeck trick:
    Triplet loss on raw features (before BN)
    ID loss on BN-normalized features (after BN)
    → Each loss gets the representation it prefers
```

---

## Part-Based Models

```
Divide person into parts for finer matching:

  ┌────────┐     ┌────────┐     ┌────┐
  │  Head  │ →  │  CNN   │ →  │ f1 │  Head feature
  ├────────┤     │        │     ├────┤
  │ Torso  │ →  │        │ →  │ f2 │  Torso feature
  ├────────┤     │        │     ├────┤
  │  Legs  │ →  │        │ →  │ f3 │  Legs feature
  └────────┘     └────────┘     └────┘

  Final feature = concat(f1, f2, f3)
  
  Benefits:
    - Handles partial occlusion (visible parts still match)
    - Finer-grained matching
    
  PCB (Part-based Conv Baseline):
    Horizontally partition feature map into P strips
    Each strip → separate classifier
    
  MGN (Multiple Granularity Network):
    Global + 2-part + 3-part branches combined
```

---

## Evaluation

```
Metrics:

  Rank-k accuracy:
    Given query, sort gallery by distance
    Rank-1: Is the correct match the closest? (most important)
    Rank-5: Is correct match in top 5?
    Rank-10: Is correct match in top 10?

  mAP (mean Average Precision):
    Average precision across all queries
    Handles cases where multiple gallery images show same person

  CMC curve (Cumulative Match Characteristic):
    Plot Rank-k accuracy vs k
    
    Rank-k
    100% ┤                    ─────────
         │               ────┘
     80% ┤           ───┘
         │       ───┘
     60% ┤    ──┘
         │  ─┘
     40% ┤─┘
         │
     20% ┤
         │
      0% ┼───┬───┬───┬───┬───┬
         1   2   3   4   5  10   Rank

Benchmarks:
  Market-1501:  1501 identities, 6 cameras
  DukeMTMC:     1812 identities, 8 cameras
  MSMT17:       4101 identities, 15 cameras (hardest)
  CUHK03:       1467 identities, pairs of cameras
```

---

## Python: Feature Extraction for Re-ID

```python
import torch
import torchvision.transforms as T
from PIL import Image

# Simple Re-ID feature extractor
model = torch.hub.load('pytorch/vision', 'resnet50', pretrained=True)
model.fc = torch.nn.Identity()  # Remove classifier
model.eval()

transform = T.Compose([
    T.Resize((256, 128)),
    T.ToTensor(),
    T.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])
])

def extract_feature(img_path):
    img = Image.open(img_path).convert('RGB')
    x = transform(img).unsqueeze(0)
    with torch.no_grad():
        feat = model(x)
    return feat / feat.norm()  # L2 normalize

# Compare two person crops
f1 = extract_feature("person_cam1.jpg")
f2 = extract_feature("person_cam2.jpg")

similarity = torch.mm(f1, f2.t()).item()
print(f"Cosine similarity: {similarity:.3f}")
print(f"Same person: {similarity > 0.5}")
```

---

## Revision Questions

1. **What is the Re-ID problem and how does it differ from face recognition?**
2. **How does triplet loss learn a useful embedding space?**
3. **What is the BNNeck trick and why does it help?**
4. **How do part-based models handle partial occlusion?**
5. **What do Rank-1 accuracy and mAP measure in Re-ID evaluation?**

---

[Previous: 03-tracking-algorithms.md](03-tracking-algorithms.md) | [Back to README](../README.md)
