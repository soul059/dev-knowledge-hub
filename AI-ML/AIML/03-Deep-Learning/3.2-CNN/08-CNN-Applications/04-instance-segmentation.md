# Instance Segmentation — Detect + Segment Each Instance

> **Unit 8 · CNN Applications · Topic 4/6**

---

## Table of Contents

1. [Problem Formulation](#1-problem-formulation)
2. [Mask R-CNN](#2-mask-r-cnn)
3. [Panoptic Segmentation](#3-panoptic-segmentation)
4. [Evaluation Metrics](#4-evaluation-metrics)
5. [Worked Example](#5-worked-example)
6. [PyTorch Implementation](#6-pytorch-implementation)
7. [Applications](#7-applications)
8. [Summary Table](#8-summary-table)
9. [Revision Questions](#9-revision-questions)
10. [Navigation](#10-navigation)

---

## 1. Problem Formulation

```
Semantic Segmentation:        Instance Segmentation:
  Every pixel gets a class      Every pixel gets class + instance ID
  
  ┌────────────────────┐       ┌────────────────────┐
  │ ████ car ████ car █│       │ ████car1████car2███│
  │ ████████████████████│       │ ████████████████████│
  │ ████████████████████│       │ ████████████████████│
  │ ░░░░ road ░░░░░░░░│       │ ░░░░ road ░░░░░░░░│
  └────────────────────┘       └────────────────────┘
  
  Both cars = same label        Each car = separate instance
  Can't tell how many cars!     Knows there are 2 cars!

Output per detection:
  • Bounding box (x₁, y₁, x₂, y₂)
  • Class label c
  • Confidence score p
  • Binary mask M ∈ {0,1}^(H × W)   ← NEW: per-instance mask
```

---

## 2. Mask R-CNN

Mask R-CNN (He et al., 2017) = **Faster R-CNN + mask prediction branch**.

```
Mask R-CNN Architecture:

  Image → CNN Backbone (ResNet + FPN)
              │
              ▼
         Feature Pyramid
              │
              ▼
         RPN (Region Proposal Network)
              │
              ▼ (top-N proposals)
         RoI Align
           ╱     ╲
          ╱       ╲
    ┌─────────┐  ┌─────────┐
    │ FC layers│  │ FCN      │   ← NEW: Mask branch
    │ (head)   │  │ (small   │
    │          │  │  conv    │
    │ ┌─────┐ │  │  network)│
    │ │Class│ │  │          │
    │ │(K+1)│ │  │ ┌──────┐ │
    │ ├─────┤ │  │ │K masks│ │
    │ │BBox │ │  │ │28×28  │ │
    │ │(4)  │ │  │ │binary │ │
    │ └─────┘ │  │ └──────┘ │
    └─────────┘  └─────────┘

Key outputs per RoI:
  1. Class label (K+1 classes, including background)
  2. Bounding box (4 coordinates)
  3. Binary mask (28×28 per class) ← predicts mask for EACH class
```

### RoI Align (vs RoI Pool)

```
Problem with RoI Pooling:
  Quantization (rounding) causes spatial misalignment
  
  RoI boundaries: (10.3, 15.7, 25.8, 40.2)
  RoI Pool rounds to: (10, 16, 26, 40)  ← loses sub-pixel precision
  
  For classification: minor issue
  For masks: critical! Misalignment of even 1 pixel = wrong mask

RoI Align solution: Use BILINEAR INTERPOLATION instead of rounding

  ┌──────────────────────────┐
  │  Feature map grid:       │
  │  ·──·──·──·──·          │
  │  │  │  │  │  │          │
  │  ·──┼──·──·──·          │
  │  │  │ ★ │  │  │   ★ = sampling point
  │  ·──·──·──·──·        (bilinear interpolation
  │  │  │  │  │  │         from 4 nearest grid points)
  │  ·──·──·──·──·          │
  └──────────────────────────┘
  
  No quantization → perfectly aligned features → precise masks

RoI Align significantly improves mask quality (~10-50% relative)
```

### Mask Head Details

```
Mask Prediction Head (per RoI):

  RoI-Aligned features (14×14×256)
      │
      ▼
  [4× Conv3×3 (256)] → 14×14×256
      │
      ▼
  [ConvTranspose2d 2×2] → 28×28×256
      │
      ▼
  [Conv1×1 (K)] → 28×28×K   ← K binary masks (one per class)

Loss:
  Only the mask for the CORRECT class is used in the loss
  Binary cross-entropy per pixel:
  
  L_mask = -(1/m²) Σᵢ [yᵢ log(p̂ᵢ) + (1-yᵢ) log(1-p̂ᵢ)]
  
  Where m=28 (mask resolution), applied only to GT class mask

Total Mask R-CNN loss:
  L = L_cls + L_box + L_mask
```

---

## 3. Panoptic Segmentation

Panoptic = **semantic + instance** segmentation unified.

```
Panoptic Segmentation:
  • "Thing" classes (countable): car, person, dog → instance segmented
  • "Stuff" classes (uncountable): sky, road, grass → semantically segmented
  
  Every pixel assigned: (class_id, instance_id)
  
  ┌────────────────────────────────────────┐
  │  person_1   person_2     sky (stuff)   │
  │  ████████   ████████   ░░░░░░░░░░░░   │
  │  ████████   ████████   ░░░░░░░░░░░░   │
  │  ████████               ░░░░░░░░░░░░   │
  │         car_1                          │
  │  ▓▓▓▓▓▓▓▓▓▓▓▓          road (stuff)   │
  │  ▓▓▓▓▓▓▓▓▓▓▓▓  ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒  │
  └────────────────────────────────────────┘

Methods:
  1. Panoptic FPN: FPN + parallel semantic/instance heads
  2. MaskFormer/Mask2Former: Unified transformer-based approach
  3. DETR-based panoptic: End-to-end with transformer decoder
```

### Panoptic Quality (PQ) Metric

```
PQ = Σ_(tp) IoU(pᵢ, gᵢ) / (|TP| + ½|FP| + ½|FN|)

Can be decomposed:
  PQ = SQ × RQ

  SQ (Segmentation Quality) = average IoU of matched segments
  RQ (Recognition Quality)  = F1 score of segment matching

  SQ = Σ_(tp) IoU(pᵢ, gᵢ) / |TP|
  RQ = |TP| / (|TP| + ½|FP| + ½|FN|)
```

---

## 4. Evaluation Metrics

```
Instance Segmentation Metrics:

Mask AP (Average Precision):
  Same as detection mAP, but IoU computed on MASKS instead of boxes
  
  mAP@0.5:      IoU threshold 0.5 on masks
  mAP@0.75:     IoU threshold 0.75 on masks (stricter)
  mAP@0.5:0.95: Primary COCO metric

Mask IoU = pixel_intersection / pixel_union (not box IoU!)

Size-based metrics:
  AP_S:  AP for small objects (area < 32²)
  AP_M:  AP for medium objects (32² < area < 96²)
  AP_L:  AP for large objects (area > 96²)

State-of-the-art on COCO:
┌────────────────┬──────────┬──────────┐
│ Model          │ Box AP   │ Mask AP  │
├────────────────┼──────────┼──────────┤
│ Mask R-CNN R50 │  38.6    │  35.2    │
│ Mask R-CNN R101│  40.6    │  36.8    │
│ Mask2Former    │  47.7    │  44.0    │
│ ViTDet         │  51.6    │  46.7    │
└────────────────┴──────────┴──────────┘
```

---

## 5. Worked Example

```
Mask AP Computation (simplified, 1 class, 3 images):

Image 1: GT has 2 persons, model detects 2
  Detection 1: conf=0.95, mask IoU with GT_1 = 0.82 → TP
  Detection 2: conf=0.80, mask IoU with GT_2 = 0.55 → TP

Image 2: GT has 1 person, model detects 2
  Detection 3: conf=0.90, mask IoU with GT_1 = 0.71 → TP
  Detection 4: conf=0.60, no GT match              → FP

Image 3: GT has 1 person, model detects 0
  GT_1 is missed                                    → FN

Total: 4 GTs, 4 detections

Ranked by confidence: [0.95, 0.90, 0.80, 0.60]

At IoU@0.5:
  Det  │ Conf │ TP/FP │ Prec  │ Recall
  1    │ 0.95 │  TP   │ 1/1   │ 1/4
  3    │ 0.90 │  TP   │ 2/2   │ 2/4
  2    │ 0.80 │  TP   │ 3/3   │ 3/4
  4    │ 0.60 │  FP   │ 3/4   │ 3/4

AP ≈ area under P-R curve ≈ 0.75
```

---

## 6. PyTorch Implementation

```python
import torch
import torchvision
from torchvision.models.detection import (
    maskrcnn_resnet50_fpn_v2,
    MaskRCNN_ResNet50_FPN_V2_Weights,
)

# Load pretrained Mask R-CNN
model = maskrcnn_resnet50_fpn_v2(
    weights=MaskRCNN_ResNet50_FPN_V2_Weights.COCO_V1
)
model.eval()

# Inference
image = torch.rand(3, 600, 800)
with torch.no_grad():
    predictions = model([image])

# predictions[0] keys:
#   'boxes':  (N, 4)         bounding boxes
#   'labels': (N,)           class labels
#   'scores': (N,)           confidence scores
#   'masks':  (N, 1, H, W)  instance masks (soft, 0-1)

# Visualize
for i in range(len(predictions[0]['boxes'])):
    score = predictions[0]['scores'][i].item()
    if score > 0.5:
        label = predictions[0]['labels'][i].item()
        mask = predictions[0]['masks'][i, 0] > 0.5  # threshold
        box = predictions[0]['boxes'][i].tolist()
        print(f"Class {label}: {score:.2f}, box={box}")
        print(f"  Mask shape: {mask.shape}, pixels: {mask.sum()}")

# Custom Mask R-CNN for your dataset
from torchvision.models.detection.faster_rcnn import FastRCNNPredictor
from torchvision.models.detection.mask_rcnn import MaskRCNNPredictor

model = maskrcnn_resnet50_fpn_v2(weights="COCO_V1")

# Replace box predictor
in_features = model.roi_heads.box_predictor.cls_score.in_features
model.roi_heads.box_predictor = FastRCNNPredictor(in_features, num_classes=5)

# Replace mask predictor
in_features_mask = model.roi_heads.mask_predictor.conv5_mask.in_channels
model.roi_heads.mask_predictor = MaskRCNNPredictor(
    in_features_mask, 256, num_classes=5)
```

---

## 7. Applications

```
Instance Segmentation Applications:
┌──────────────────┬────────────────────────────────────────┐
│ Domain           │ Use Case                               │
├──────────────────┼────────────────────────────────────────┤
│ Autonomous driving│ Segment each car, pedestrian, cyclist │
│ Robotics         │ Pick-and-place: identify each object   │
│ Medical imaging  │ Count and segment individual cells     │
│ Agriculture      │ Count fruits, detect diseased plants   │
│ Retail           │ Product recognition on shelves         │
│ AR/VR            │ Real-time person segmentation          │
│ Video editing    │ Separate foreground objects             │
│ Sports analytics │ Track individual players               │
└──────────────────┴────────────────────────────────────────┘
```

---

## 8. Summary Table

| Concept | Key Idea |
|---|---|
| **Instance segmentation** | Detect + pixel-level mask for each object instance |
| **Mask R-CNN** | Faster R-CNN + parallel mask branch (FCN on each RoI) |
| **RoI Align** | Bilinear interpolation instead of quantized pooling |
| **Mask head** | Small FCN: 4×Conv3×3 → ConvTranspose → 28×28×K masks |
| **Mask loss** | Binary CE on correct class mask only |
| **Panoptic** | Things (instance seg) + stuff (semantic seg) unified |
| **PQ metric** | PQ = SQ × RQ; evaluates both segmentation and recognition |
| **Mask AP** | Like detection AP but IoU computed on pixel masks |
| **Mask2Former** | Transformer-based unified architecture for all segmentation tasks |

---

## 9. Revision Questions

1. **How does instance segmentation differ** from semantic segmentation? Give an example where semantic segmentation would fail but instance segmentation succeeds.

2. **Explain Mask R-CNN architecture.** How is the mask branch added to Faster R-CNN? Why is RoI Align critical for masks?

3. **What is RoI Align** and how does it differ from RoI Pooling? Why does the quantization error in RoI Pooling matter for segmentation masks?

4. **Explain panoptic segmentation.** What are "things" vs "stuff" classes? How does Panoptic Quality (PQ) evaluate both?

5. **The mask head predicts K masks (one per class).** Why not predict a single multi-class mask? What advantage does per-class prediction give?

---

## 10. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Semantic Segmentation](./03-semantic-segmentation.md) | [Unit 8 Index](../08-CNN-Applications/) | [Face Recognition →](./05-face-recognition.md) |

---

> **References:**  
> - He, K. et al. (2017). *Mask R-CNN.* ICCV.  
> - Kirillov, A. et al. (2019). *Panoptic Segmentation.* CVPR.  
> - Cheng, B. et al. (2022). *Masked-attention Mask Transformer for Universal Image Segmentation (Mask2Former).* CVPR.
