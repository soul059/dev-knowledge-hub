# Semantic Segmentation — Pixel-Wise Classification

> **Unit 8 · CNN Applications · Topic 3/6**

---

## Table of Contents

1. [Problem Formulation](#1-problem-formulation)
2. [Fully Convolutional Networks (FCN)](#2-fully-convolutional-networks-fcn)
3. [Encoder-Decoder Architecture](#3-encoder-decoder-architecture)
4. [U-Net — Skip Connections for Segmentation](#4-u-net--skip-connections-for-segmentation)
5. [DeepLab — Atrous Convolution & ASPP](#5-deeplab--atrous-convolution--aspp)
6. [Evaluation: mIoU Metric](#6-evaluation-miou-metric)
7. [Loss Functions](#7-loss-functions)
8. [Worked Examples](#8-worked-examples)
9. [PyTorch Implementation](#9-pytorch-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)
12. [Navigation](#12-navigation)

---

## 1. Problem Formulation

Semantic segmentation assigns a class label to **every pixel** in the image.

```
Input:  Image x ∈ ℝ^(H × W × 3)
Output: Label map y ∈ {0,1,...,K-1}^(H × W)   — one class per pixel

Classification:  "There's a cat"        → single label
Detection:       "Cat at (x,y,w,h)"     → boxes
Segmentation:    "These exact pixels are cat" → pixel mask

  Input Image:              Segmentation Mask:
  ┌──────────────────┐     ┌──────────────────┐
  │     ╱──╲         │     │ ████sky████████  │
  │    │🐱  │        │     │ ████sky████████  │
  │    │    │  🌲    │     │ cat cat  tree   │
  │    └────┘  🌲    │     │ cat cat  tree   │
  │___ground_________│     │ ground ground   │
  └──────────────────┘     └──────────────────┘
                            Classes: sky, cat, tree, ground

Key challenge: Need BOTH:
  • Global context (what class?) — from deep/pooled features
  • Local precision (exact boundary) — from high-resolution features
```

---

## 2. Fully Convolutional Networks (FCN)

FCN (Long et al., 2015) was the first end-to-end CNN for segmentation:

```
Key insight: Replace FC layers with 1×1 convolutions

Classification CNN:          FCN:
Input → Conv → ... → FC → class    Input → Conv → ... → 1×1 Conv → H×W×K
(H×W×3)        flatten  (K,)       (H×W×3)         preserve  (h×w×K)
                                                     spatial    ↓
                                                               Upsample
                                                               to (H×W×K)

FCN Architecture:
  ┌──────────────────────────────────────────────────────┐
  │  Input (H×W×3)                                       │
  │    ↓                                                  │
  │  VGG backbone (conv + pool)                          │
  │    ↓  ↓  ↓  ↓  ↓                                    │
  │  Pool1 Pool2 Pool3 Pool4 Pool5                       │
  │  H/2   H/4   H/8  H/16  H/32                       │
  │    ↓                  ↓      ↓                       │
  │    │           1×1 conv  1×1 conv                    │
  │    │              ↓         ↓                        │
  │    │          upsample 2×   ↓                        │
  │    │              ↓    + ───┘                        │
  │    │          upsample 2×                            │
  │    │              ↓                                  │
  │    │          upsample 8×                            │
  │    │              ↓                                  │
  │    │           Prediction (H×W×K)                    │
  └──────────────────────────────────────────────────────┘

FCN variants:
  FCN-32s: Upsample pool5 by 32× (coarse output)
  FCN-16s: Combine pool5 + pool4, upsample 16×
  FCN-8s:  Combine pool5 + pool4 + pool3, upsample 8× (best)
```

---

## 3. Encoder-Decoder Architecture

```
General Encoder-Decoder:

Encoder (downsampling):         Decoder (upsampling):
Extracts features,              Recovers spatial resolution,
reduces spatial size             produces dense predictions

Input (H×W)                                         Output (H×W×K)
    │                                                     ▲
    ▼                                                     │
┌───────┐                                            ┌───────┐
│  H×W  │                                            │  H×W  │
│  Conv  │                                            │ Conv  │
└───┬───┘                                            └───▲───┘
    ▼                                                     │
┌───────┐                                            ┌───────┐
│ H/2×W/2│                                           │H/2×W/2│
│  Conv  │                                            │UpConv │
└───┬───┘                                            └───▲───┘
    ▼                                                     │
┌───────┐                                            ┌───────┐
│H/4×W/4│                                            │H/4×W/4│
│  Conv  │                                            │UpConv │
└───┬───┘                                            └───▲───┘
    ▼                                                     │
┌───────┐                                            ┌───────┐
│H/8×W/8│──────────── Bottleneck ────────────────────│H/8×W/8│
│  Conv  │                                            │UpConv │
└───────┘                                            └───────┘

Upsampling methods:
  1. Bilinear interpolation (no learnable params)
  2. Transposed convolution (learnable, may cause checkerboard)
  3. Nearest neighbor + Conv (simple, effective)
  4. PixelShuffle/SubPixelConv (rearrange channels to spatial)
```

---

## 4. U-Net — Skip Connections for Segmentation

U-Net (Ronneberger et al., 2015) adds **skip connections** from encoder to decoder.

```
U-Net Architecture:

  Encoder                                    Decoder
  (contracting path)                         (expanding path)

  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Input (572×572×1)                                       │
  │      │                                                   │
  │      ▼                                                   │
  │  [Conv3×3→Conv3×3] ───────────── Copy & Crop ──► [ConvConv]→ Output
  │      │ 568×568×64                                388×388×64   │
  │      ▼ MaxPool                            Upsample ▲          │
  │  [Conv3×3→Conv3×3] ───────────── Copy & Crop ──► [ConvConv]   │
  │      │ 280×280×128                          196×196×128       │
  │      ▼ MaxPool                            Upsample ▲          │
  │  [Conv3×3→Conv3×3] ───────────── Copy & Crop ──► [ConvConv]   │
  │      │ 136×136×256                          100×100×256       │
  │      ▼ MaxPool                            Upsample ▲          │
  │  [Conv3×3→Conv3×3] ───────────── Copy & Crop ──► [ConvConv]   │
  │      │ 64×64×512                            52×52×512         │
  │      ▼ MaxPool                            Upsample ▲          │
  │  [Conv3×3→Conv3×3]  ──── Bottleneck ─────► [ConvConv]         │
  │      28×28×1024                            28×28×1024         │
  │                                                               │
  └──────────────────────────────────────────────────────────────┘

Skip Connection Purpose:
  Encoder features: WHERE (precise spatial information, edges, textures)
  Decoder features: WHAT (semantic, high-level understanding)
  
  Concatenation combines BOTH → accurate boundaries + correct classes

  Without skips:  Decoder must "guess" exact boundaries → blurry
  With skips:     Decoder receives exact spatial info → sharp boundaries
```

### U-Net Building Blocks

```
Double Convolution Block:
  Input → Conv3×3 → BN → ReLU → Conv3×3 → BN → ReLU → Output

Encoder Step:
  Input → DoubleConv → (save for skip) → MaxPool2×2 → next level

Decoder Step:
  Input → Upsample2× → Concat(skip) → DoubleConv → next level

1×1 Output Convolution:
  Features (H×W×64) → Conv1×1(64, K) → Logits (H×W×K)
```

---

## 5. DeepLab — Atrous Convolution & ASPP

### Atrous (Dilated) Convolution

```
Standard 3×3 conv (rate=1):    Atrous 3×3 conv (rate=2):

  ███                           █ · █ · █
  ███    receptive field = 3    · · · · ·
  ███                           █ · █ · █   receptive field = 5
                                · · · · ·
                                █ · █ · █

  Same #parameters (9), but LARGER receptive field!
  No pooling needed → preserves resolution
```

### Atrous Spatial Pyramid Pooling (ASPP)

```
ASPP: Apply atrous convolutions at MULTIPLE rates in parallel

  Input Feature Map
      │
      ├──── 1×1 Conv ────────────────────────────┐
      │                                           │
      ├──── 3×3 Atrous Conv (rate=6)  ───────────┤
      │                                           │
      ├──── 3×3 Atrous Conv (rate=12) ───────────┤  Concat
      │                                           │  → 1×1
      ├──── 3×3 Atrous Conv (rate=18) ───────────┤  → Output
      │                                           │
      └──── Global Average Pool → 1×1 → Upsample ┘

  Each branch captures context at a different scale:
    rate=6:  medium context (nearby objects)
    rate=12: large context (scene understanding)
    rate=18: very large context (global layout)
    GAP:     image-level features (overall scene)
```

### DeepLab v3+ Architecture

```
DeepLabV3+:

  Input → Backbone (ResNet/Xception with atrous conv)
              │
              ├── ASPP (multi-scale features) ── 1×1 → Upsample 4×
              │                                           │
              │                                           ▼
              └── Low-level features (from early layer) → Concat
                   │                                      │
                   └──── 1×1 Conv (reduce channels) ──────┘
                                                          │
                                                     3×3 Conv
                                                          │
                                                     Upsample 4×
                                                          │
                                                     Prediction
                                                     (H×W×K)

Output stride: 16 during training (efficient)
               8 during inference (more accurate)
```

---

## 6. Evaluation: mIoU Metric

```
Mean Intersection over Union (mIoU):

For each class c:
  IoU_c = TP_c / (TP_c + FP_c + FN_c)

  TP_c = pixels correctly predicted as class c
  FP_c = pixels incorrectly predicted as class c
  FN_c = pixels of class c that were missed

mIoU = (1/K) × Σ_c IoU_c

Example (3 classes: sky, tree, ground):

  Ground Truth:          Prediction:
  ┌──────────────┐      ┌──────────────┐
  │ sky sky sky  │      │ sky sky sky  │
  │ sky tree sky │      │ sky tree gnd │ ← misclassified
  │ gnd gnd gnd  │      │ gnd gnd gnd  │
  └──────────────┘      └──────────────┘

  Sky:    TP=5, FP=0, FN=1   → IoU = 5/(5+0+1) = 0.833
  Tree:   TP=1, FP=0, FN=0   → IoU = 1/(1+0+0) = 1.000
  Ground: TP=3, FP=1, FN=0   → IoU = 3/(3+1+0) = 0.750

  mIoU = (0.833 + 1.000 + 0.750) / 3 = 0.861 = 86.1%

Other metrics:
  Pixel accuracy: correct_pixels / total_pixels
  Mean pixel accuracy: average per-class pixel accuracy
  Dice coefficient: 2×|A∩B| / (|A|+|B|)  (F1 for segmentation)
```

---

## 7. Loss Functions

### Cross-Entropy Loss (Standard)

```
Per-pixel cross-entropy:
  L_CE = -(1/N) Σᵢ Σ_c yᵢ,c · log(p̂ᵢ,c)

  Where:
    N = number of pixels
    yᵢ,c = 1 if pixel i belongs to class c (one-hot)
    p̂ᵢ,c = predicted probability for class c at pixel i

Problem: Doesn't handle class imbalance well
  (background is 70% of pixels → dominates the loss)
```

### Dice Loss

```
Dice coefficient: 2|A ∩ B| / (|A| + |B|)

Dice Loss = 1 - Dice

For soft predictions:
  Dice = 2 · Σᵢ pᵢ · yᵢ / (Σᵢ pᵢ + Σᵢ yᵢ)

Where pᵢ = predicted probability, yᵢ = ground truth (0 or 1)

Advantages:
  ✓ Handles class imbalance naturally
  ✓ Directly optimizes overlap metric
  ✓ Works well for small objects

Common: Combined loss = CE + Dice
  L = λ₁ · L_CE + λ₂ · L_Dice
```

### Focal Loss (for imbalanced classes)

```
Focal Loss = -α_t (1 - p_t)^γ · log(p_t)

  Standard CE:  -log(p_t)
  Focal:        -(1 - p_t)^γ · log(p_t)

  γ=0: same as CE
  γ=2: down-weights easy examples (well-classified pixels)
       focuses on hard/misclassified pixels

  When p_t=0.9 (easy): weight = (1-0.9)² = 0.01  (almost ignored)
  When p_t=0.1 (hard): weight = (1-0.1)² = 0.81  (full attention)
```

---

## 8. Worked Examples

### Example: Dice Coefficient Computation

```
Ground Truth (binary, class 'car'):    Prediction:
  ┌─────────────┐                     ┌─────────────┐
  │ 0 0 0 0 0 0 │                     │ 0 0 0 0 0 0 │
  │ 0 1 1 1 0 0 │                     │ 0 1 1 0 0 0 │
  │ 0 1 1 1 0 0 │                     │ 0 1 1 1 0 0 │
  │ 0 0 0 0 0 0 │                     │ 0 0 1 0 0 0 │
  └─────────────┘                     └─────────────┘

  Intersection (both 1): 4 pixels
  |A| (ground truth 1): 6 pixels
  |B| (prediction 1):   5 pixels
  
  Dice = 2 × 4 / (6 + 5) = 8/11 = 0.727
  IoU  = 4 / (6 + 5 - 4) = 4/7 = 0.571
  
  Note: Dice ≥ IoU always (for same sets)
  Relationship: Dice = 2·IoU / (1 + IoU)
```

---

## 9. PyTorch Implementation

### Simple U-Net

```python
import torch
import torch.nn as nn

class DoubleConv(nn.Module):
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.double_conv = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, padding=1, bias=False),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.Conv2d(out_ch, out_ch, 3, padding=1, bias=False),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )
    def forward(self, x):
        return self.double_conv(x)


class UNet(nn.Module):
    def __init__(self, in_channels=3, num_classes=21):
        super().__init__()
        # Encoder
        self.enc1 = DoubleConv(in_channels, 64)
        self.enc2 = DoubleConv(64, 128)
        self.enc3 = DoubleConv(128, 256)
        self.enc4 = DoubleConv(256, 512)
        self.pool = nn.MaxPool2d(2)

        # Bottleneck
        self.bottleneck = DoubleConv(512, 1024)

        # Decoder
        self.up4 = nn.ConvTranspose2d(1024, 512, 2, stride=2)
        self.dec4 = DoubleConv(1024, 512)  # 512+512 from skip
        self.up3 = nn.ConvTranspose2d(512, 256, 2, stride=2)
        self.dec3 = DoubleConv(512, 256)
        self.up2 = nn.ConvTranspose2d(256, 128, 2, stride=2)
        self.dec2 = DoubleConv(256, 128)
        self.up1 = nn.ConvTranspose2d(128, 64, 2, stride=2)
        self.dec1 = DoubleConv(128, 64)

        self.out_conv = nn.Conv2d(64, num_classes, 1)

    def forward(self, x):
        # Encoder
        e1 = self.enc1(x)
        e2 = self.enc2(self.pool(e1))
        e3 = self.enc3(self.pool(e2))
        e4 = self.enc4(self.pool(e3))
        
        # Bottleneck
        b = self.bottleneck(self.pool(e4))
        
        # Decoder with skip connections
        d4 = self.dec4(torch.cat([self.up4(b), e4], dim=1))
        d3 = self.dec3(torch.cat([self.up3(d4), e3], dim=1))
        d2 = self.dec2(torch.cat([self.up2(d3), e2], dim=1))
        d1 = self.dec1(torch.cat([self.up1(d2), e1], dim=1))
        
        return self.out_conv(d1)  # (B, K, H, W)

model = UNet(in_channels=3, num_classes=21)
x = torch.randn(2, 3, 256, 256)
print(model(x).shape)  # (2, 21, 256, 256)
```

### Dice Loss

```python
class DiceLoss(nn.Module):
    def __init__(self, smooth=1.0):
        super().__init__()
        self.smooth = smooth

    def forward(self, logits, targets):
        # logits: (B, K, H, W), targets: (B, H, W) with class indices
        probs = torch.softmax(logits, dim=1)
        targets_onehot = torch.zeros_like(probs).scatter_(
            1, targets.unsqueeze(1), 1)

        intersection = (probs * targets_onehot).sum(dim=(2, 3))
        union = probs.sum(dim=(2, 3)) + targets_onehot.sum(dim=(2, 3))

        dice = (2 * intersection + self.smooth) / (union + self.smooth)
        return 1 - dice.mean()

# Combined loss
criterion = lambda logits, targets: (
    nn.CrossEntropyLoss()(logits, targets) +
    DiceLoss()(logits, targets)
)
```

### Using Pretrained DeepLab

```python
from torchvision.models.segmentation import deeplabv3_resnet101
model = deeplabv3_resnet101(weights='COCO_WITH_VOC_LABELS_V1')
model.eval()

x = torch.randn(1, 3, 512, 512)
with torch.no_grad():
    output = model(x)['out']  # (1, 21, 512, 512)
    pred = output.argmax(1)    # (1, 512, 512)
```

---

## 10. Summary Table

| Concept | Key Idea |
|---|---|
| **Semantic segmentation** | Classify every pixel into a class |
| **FCN** | Replace FC with 1×1 conv; first end-to-end segmentation CNN |
| **Encoder-decoder** | Downsample to extract features, upsample to recover resolution |
| **U-Net** | Skip connections from encoder to decoder (concat) |
| **Skip connections** | Combine WHERE (spatial) with WHAT (semantic) |
| **DeepLab** | Atrous convolution for large receptive field without pooling |
| **ASPP** | Parallel atrous convolutions at multiple rates |
| **mIoU** | Mean IoU across all classes; primary metric |
| **Dice loss** | 1 - 2|A∩B|/(|A|+|B|); handles class imbalance |
| **CE + Dice** | Combined loss is the practical standard |

---

## 11. Revision Questions

1. **What is the key difference** between image classification, object detection, and semantic segmentation? What does each output?

2. **Explain U-Net's skip connections.** Why is concatenating encoder features with decoder features crucial for precise segmentation?

3. **How does atrous (dilated) convolution** increase receptive field without reducing resolution? Compare a 3×3 conv with rate=1 vs rate=3.

4. **Compute mIoU** for a 4×4 image with 2 classes (foreground/background) given a predicted and ground truth mask.

5. **Compare CE loss and Dice loss** for semantic segmentation. When does Dice loss outperform CE? Why is a combined loss often best?

6. **Design a segmentation pipeline** for autonomous driving (classes: road, car, pedestrian, sky, building). Choose architecture, loss function, and evaluation metric.

---

## 12. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Object Detection](./02-object-detection.md) | [Unit 8 Index](../08-CNN-Applications/) | [Instance Segmentation →](./04-instance-segmentation.md) |

---

> **References:**  
> - Long, J. et al. (2015). *Fully Convolutional Networks for Semantic Segmentation.* CVPR.  
> - Ronneberger, O. et al. (2015). *U-Net: Convolutional Networks for Biomedical Image Segmentation.* MICCAI.  
> - Chen, L.C. et al. (2018). *Encoder-Decoder with Atrous Separable Convolution for Semantic Image Segmentation (DeepLabV3+).* ECCV.
