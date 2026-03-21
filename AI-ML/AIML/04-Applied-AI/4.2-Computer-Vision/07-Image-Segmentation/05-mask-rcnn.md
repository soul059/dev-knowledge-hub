# Mask R-CNN Deep Dive

## Overview

**Mask R-CNN** (He et al., 2017) is the foundational architecture for instance segmentation. It extends Faster R-CNN by adding a parallel mask prediction branch. Its clean design, strong performance, and extensibility have made it the baseline for nearly all instance-level tasks.

---

## Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │                    Mask R-CNN Pipeline                    │
  │                                                          │
  │  Image (800×1333)                                        │
  │       │                                                  │
  │  ┌────▼─────────────┐                                    │
  │  │  ResNet Backbone  │  Extract hierarchical features     │
  │  │  (ResNet-50/101)  │  C2, C3, C4, C5                   │
  │  └────┬─────────────┘                                    │
  │       │                                                  │
  │  ┌────▼─────────────┐                                    │
  │  │  FPN (Feature     │  Build multi-scale feature pyramid │
  │  │  Pyramid Network) │  P2, P3, P4, P5, P6               │
  │  └────┬─────────────┘                                    │
  │       │                                                  │
  │  ┌────▼─────────────┐                                    │
  │  │  RPN (Region      │  Generate ~1000 proposals          │
  │  │  Proposal Network)│  Anchor-based, class-agnostic      │
  │  └────┬─────────────┘                                    │
  │       │                                                  │
  │  ┌────▼─────────────┐                                    │
  │  │  RoIAlign         │  Extract 7×7 (box) or 14×14 (mask) │
  │  │                   │  features for each proposal        │
  │  └────┬─────────────┘                                    │
  │       │                                                  │
  │  ┌────┼────────────────────┐                              │
  │  │    │                    │                              │
  │  │ ┌──▼──────┐  ┌─────────▼────┐                         │
  │  │ │Box Head │  │  Mask Head   │                         │
  │  │ │ FC×2    │  │  Conv×4+     │                         │
  │  │ │cls + reg│  │  Deconv      │                         │
  │  │ └────┬────┘  └──────┬──────┘                         │
  │  │      │              │                                  │
  │  │  Class+Box      28×28×C mask                          │
  │  └────────────────────────────────────────────────────────┘
```

---

## RoIAlign vs RoIPool

```
Problem with RoIPool:
  RoI maps to feature map coordinates that aren't integers
  → Quantize (round) to nearest integer → spatial misalignment

  Example: RoI at (12.3, 7.8) with size (5.6, 4.2)
  RoIPool:  rounds to (12, 7), size (5, 4) → up to 1 pixel shift
  At 1/16 stride → 16 pixel error in original image!

RoIAlign solution:
  1. Don't quantize — keep floating-point coordinates
  2. Sample at regular grid points within each bin
  3. Use bilinear interpolation to compute values at sample points

  ┌─────┬─────┬─────┐    Each bin:
  │  ●  │  ●  │  ●  │    ● = sample point (4 per bin)
  │  ●  │  ●  │  ●  │    Value computed via bilinear
  ├─────┼─────┼─────┤    interpolation of surrounding
  │  ●  │  ●  │  ●  │    feature map values
  │  ●  │  ●  │  ●  │
  ├─────┼─────┼─────┤    Then average/max pool within bin
  │  ●  │  ●  │  ●  │
  │  ●  │  ●  │  ●  │    Result: pixel-perfect alignment
  └─────┴─────┴─────┘    → crucial for mask quality
```

---

## Multi-Task Loss

```
L = L_cls + L_box + L_mask

Classification Loss (L_cls):
  Cross-entropy over K+1 classes (K classes + background)
  L_cls = -log(p_k)  where k is the true class

Box Regression Loss (L_box):
  Smooth L1 loss over 4 offsets (dx, dy, dw, dh)
  L_box = Σ smooth_L1(t_i - t_i*)
  
  smooth_L1(x) = { 0.5x²      if |x| < 1
                  { |x| - 0.5   otherwise

Mask Loss (L_mask):
  Binary cross-entropy, per-pixel, class-specific
  
  For each pixel in 28×28 mask:
    L_mask = -(y × log(p) + (1-y) × log(1-p))
  
  Only computed on the mask for the GT class
  → Decouples mask from classification
  → Each class predicts independently → no inter-class competition
```

---

## Training Pipeline

```
1. Generate anchor proposals (RPN)
   Anchors per FPN level:
     P2: 32²  × {1:2, 1:1, 2:1}
     P3: 64²  × {1:2, 1:1, 2:1}
     P4: 128² × {1:2, 1:1, 2:1}
     P5: 256² × {1:2, 1:1, 2:1}
     P6: 512² × {1:2, 1:1, 2:1}
   
   Total: ~200K anchors per image

2. RPN training (first stage)
   Positive: IoU ≥ 0.7 with any GT box
   Negative: IoU < 0.3 with all GT boxes
   Mini-batch: 256 anchors (1:1 pos:neg)

3. RoI sampling (second stage)
   Positive: IoU ≥ 0.5 with GT box
   Negative: 0.1 ≤ IoU < 0.5
   Mini-batch: 512 RoIs per image (1:3 pos:neg)

4. FPN level assignment
   Assign RoI to FPN level based on size:
   k = floor(k₀ + log₂(√(wh)/224))
   k₀ = 4 (P4 for 224×224 RoIs)
   Small objects → P2, Large objects → P5
```

---

## Extensions of Mask R-CNN

```
Mask R-CNN is the base for many tasks:

  Task                    Modification
  ─────────────────────── ────────────────────────
  Keypoint detection      Replace mask head with
                          keypoint heatmap head (K × 56×56)
  
  Human pose estimation   17 keypoint channels
                          (nose, eyes, shoulders, etc.)
  
  Dense pose (DensePose)  Predict UV coordinates mapping
                          each pixel to 3D body surface
  
  Cascade Mask R-CNN      Multiple detection stages at
                          increasing IoU thresholds
                          (0.5 → 0.6 → 0.7)
  
  HTC (Hybrid Task        Interleave box and mask heads
  Cascade)                across cascade stages
```

---

## Python: Training Mask R-CNN

```python
import torch
import torchvision
from torchvision.models.detection import maskrcnn_resnet50_fpn
from torchvision.models.detection.faster_rcnn import FastRCNNPredictor
from torchvision.models.detection.mask_rcnn import MaskRCNNPredictor

def get_model(num_classes):
    model = maskrcnn_resnet50_fpn(pretrained=True)
    
    # Replace box predictor
    in_features = model.roi_heads.box_predictor.cls_score.in_features
    model.roi_heads.box_predictor = FastRCNNPredictor(
        in_features, num_classes)
    
    # Replace mask predictor
    in_features_mask = \
        model.roi_heads.mask_predictor.conv5_mask.in_channels
    model.roi_heads.mask_predictor = MaskRCNNPredictor(
        in_features_mask, 256, num_classes)
    
    return model

# Custom dataset must return:
# image: Tensor[C, H, W]
# target: dict with:
#   "boxes": Tensor[N, 4] (x1, y1, x2, y2)
#   "labels": Tensor[N]
#   "masks": Tensor[N, H, W] (binary masks)

model = get_model(num_classes=3)  # background + 2 classes
optimizer = torch.optim.SGD(model.parameters(),
                            lr=0.005, momentum=0.9,
                            weight_decay=0.0005)

# Training loop
model.train()
for images, targets in dataloader:
    loss_dict = model(images, targets)
    losses = sum(loss for loss in loss_dict.values())
    
    optimizer.zero_grad()
    losses.backward()
    optimizer.step()
    
    print(f"Loss: {losses.item():.4f}")
    print(f"  cls: {loss_dict['loss_classifier']:.4f}")
    print(f"  box: {loss_dict['loss_box_reg']:.4f}")
    print(f"  mask: {loss_dict['loss_mask']:.4f}")
```

---

## Revision Questions

1. **What are the three parallel heads in Mask R-CNN?**
2. **Why is RoIAlign critical for mask quality?**
3. **How does the multi-task loss combine classification, box, and mask?**
4. **How are RoIs assigned to different FPN levels?**
5. **Name three tasks that extend Mask R-CNN beyond instance segmentation.**

---

[Previous: 04-segmentation-architectures.md](04-segmentation-architectures.md) | [Next: 06-evaluation-metrics.md](06-evaluation-metrics.md)
