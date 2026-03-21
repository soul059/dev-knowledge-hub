# Object Detection — From R-CNN to YOLO

> **Unit 8 · CNN Applications · Topic 2/6**

---

## Table of Contents

1. [Problem Formulation](#1-problem-formulation)
2. [Core Concepts](#2-core-concepts)
3. [Two-Stage Detectors](#3-two-stage-detectors)
4. [Single-Stage Detectors](#4-single-stage-detectors)
5. [Non-Maximum Suppression (NMS)](#5-non-maximum-suppression-nms)
6. [Evaluation: mAP Metric](#6-evaluation-map-metric)
7. [Architecture Comparison](#7-architecture-comparison)
8. [Worked Examples](#8-worked-examples)
9. [PyTorch Implementation](#9-pytorch-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)
12. [Navigation](#12-navigation)

---

## 1. Problem Formulation

Object detection = **classification** (what?) + **localization** (where?) for multiple objects.

```
Input:  Image x ∈ ℝ^(H × W × 3)
Output: Set of detections, each containing:
  • Bounding box: (x, y, w, h) or (x₁, y₁, x₂, y₂)
  • Class label:  c ∈ {1, 2, ..., K}
  • Confidence:   p ∈ [0, 1]

  ┌──────────────────────────────────┐
  │                                  │
  │    ┌──────────┐                  │
  │    │   CAR    │  conf: 0.95     │
  │    │  (0.95)  │                  │
  │    └──────────┘                  │
  │         ┌──────────┐            │
  │         │ PERSON   │            │
  │         │ (0.88)   │            │
  │         └──────────┘            │
  │                                  │
  └──────────────────────────────────┘
```

---

## 2. Core Concepts

### Bounding Box Formats

```
Two common formats:

(x, y, w, h):  Center coordinates + width/height (YOLO, COCO)
  ┌──────────────────────┐
  │        w             │
  │   ┌────┼────┐        │
  │   │    ·(x,y)│  h     │
  │   │         │        │
  │   └─────────┘        │
  └──────────────────────┘

(x₁, y₁, x₂, y₂):  Top-left + bottom-right corners (Pascal VOC)
  ┌──────────────────────┐
  │                      │
  │   (x₁,y₁)·──────┐   │
  │   │              │   │
  │   │              │   │
  │   └──────·(x₂,y₂)   │
  └──────────────────────┘

Conversion:
  x₁ = x - w/2,  y₁ = y - h/2
  x₂ = x + w/2,  y₂ = y + h/2
```

### Intersection over Union (IoU)

```
IoU = Area of Overlap / Area of Union

  ┌──────────┐
  │    A     ┼──────┐
  │          │██████│     IoU = Area(█) / Area(A ∪ B)
  │          │██████│
  └──────────┼      │
             │   B  │
             └──────┘

IoU values:
  1.0 = perfect overlap
  0.5 = threshold for "correct detection" (typical)
  0.0 = no overlap

Formula:
  x_overlap = max(0, min(x₂ᴬ, x₂ᴮ) - max(x₁ᴬ, x₁ᴮ))
  y_overlap = max(0, min(y₂ᴬ, y₂ᴮ) - max(y₁ᴬ, y₁ᴮ))
  intersection = x_overlap × y_overlap
  union = area_A + area_B - intersection
  IoU = intersection / union
```

### Anchor Boxes

```
Predefined reference boxes at each spatial position:

Feature map (7×7):
  ┌──┬──┬──┬──┬──┬──┬──┐
  │ A│ A│ A│ A│ A│ A│ A│   A = set of anchor boxes
  ├──┼──┼──┼──┼──┼──┼──┤       at each position
  │ A│ A│ A│ A│ A│ A│ A│
  ├──┼──┼──┼──┼──┼──┼──┤   k anchors per position
  │ A│ A│ A│ A│ A│ A│ A│   Total: 7 × 7 × k boxes
  ...

At each position, k anchors with different:
  • Scales: {32², 64², 128²} pixels
  • Aspect ratios: {1:1, 1:2, 2:1}
  → k = 3 scales × 3 ratios = 9 anchors per position

Network predicts OFFSETS from anchors:
  bbox = anchor + Δ(tx, ty, tw, th)
  
  bx = anchor_x + tx × anchor_w
  by = anchor_y + ty × anchor_h
  bw = anchor_w × exp(tw)
  bh = anchor_h × exp(th)
```

---

## 3. Two-Stage Detectors

### R-CNN (Girshick et al., 2014)

```
R-CNN Pipeline:

Image → Selective Search (2000 regions) → CNN (each region) → SVM → BBox regression

  ┌──────────┐    ┌───────────────┐    ┌──────────┐    ┌──────────┐
  │  Input   │───►│ Selective     │───►│ Warp each│───►│ CNN      │
  │  Image   │    │ Search        │    │ to 227×227│   │ (AlexNet)│
  └──────────┘    │ (~2000 RoIs)  │    └──────────┘    └────┬─────┘
                  └───────────────┘                          │
                                                     ┌──────┴──────┐
                                                     │ SVM + BBox  │
                                                     │ Regression  │
                                                     └─────────────┘

Problem: ~2000 forward passes per image → ~47 seconds/image!
```

### Fast R-CNN (Girshick, 2015)

```
Key innovation: Share CNN computation across all regions

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  Input   │───►│ CNN      │───►│ RoI Pool │───►│ FC → cls │
  │  Image   │    │ (full    │    │ (crop    │    │ + bbox   │
  │          │    │  image)  │    │  features│    │ per RoI  │
  └──────────┘    └──────────┘    │  for each│    └──────────┘
                       ▲          │  region) │
                       │          └──────────┘
              Selective Search         ▲
              gives RoI coords ────────┘

RoI Pooling: crop feature map at each RoI → fixed-size output
  Feature map → region → divide into H×W grid → max pool each cell

Speed: ~2 seconds/image (20× faster than R-CNN)
Still bottleneck: Selective Search is slow and outside the network!
```

### Faster R-CNN (Ren et al., 2016)

```
Key innovation: Replace Selective Search with a LEARNED Region Proposal Network

  ┌──────────────────────────────────────────────────┐
  │              Faster R-CNN                         │
  │                                                   │
  │  Image → CNN Backbone (ResNet/VGG)                │
  │              │                                    │
  │              ▼                                    │
  │         Feature Map                               │
  │           ╱       ╲                               │
  │          ╱         ╲                              │
  │    ┌─────────┐  ┌───────────┐                    │
  │    │   RPN   │  │ RoI Pool  │                    │
  │    │(Region  │──►│+ FC Head  │                    │
  │    │Proposal │  │           │                    │
  │    │Network) │  │ Class +   │                    │
  │    │         │  │ BBox pred │                    │
  │    └─────────┘  └───────────┘                    │
  └──────────────────────────────────────────────────┘

RPN (Region Proposal Network):
  Input: feature map (H×W×C)
  At each position: k anchor boxes
  Output per anchor:
    • Objectness score: P(object) vs P(background)
    • BBox deltas: (Δx, Δy, Δw, Δh)
  
  Keep top-N proposals (e.g., 300) after NMS
  
  ┌────┐
  │3×3 │──► ┌─────────────┐
  │conv │    │ cls (2k)    │  object vs background
  └────┘    │ reg (4k)    │  box offsets
             └─────────────┘

Speed: ~200ms per image (real-time feasible!)
End-to-end trainable (everything is differentiable)
```

---

## 4. Single-Stage Detectors

### YOLO (You Only Look Once)

```
YOLO Philosophy: Treat detection as a SINGLE regression problem

  Image → Single CNN → Grid of predictions

  Input image divided into S×S grid (e.g., 7×7):
  ┌──┬──┬──┬──┬──┬──┬──┐
  │  │  │  │  │  │  │  │
  ├──┼──┼──┼──┼──┼──┼──┤
  │  │  │  │  │  │  │  │   Each cell predicts:
  ├──┼──┼──┼──┼──┼──┼──┤   • B bounding boxes (x,y,w,h,conf)
  │  │  │  │★ │  │  │  │   • C class probabilities
  ├──┼──┼──┼──┼──┼──┼──┤
  │  │  │  │  │  │  │  │   Output tensor:
  ├──┼──┼──┼──┼──┼──┼──┤   S × S × (B × 5 + C)
  │  │  │  │  │  │  │  │   7 × 7 × (2×5 + 20) = 7×7×30
  ├──┼──┼──┼──┼──┼──┼──┤
  │  │  │  │  │  │  │  │   ★ = cell responsible for object
  └──┴──┴──┴──┴──┴──┴──┘     (center falls in this cell)

YOLO versions evolution:
┌──────────┬──────────┬──────────┬──────────────────────┐
│ Version  │ mAP(%)   │ FPS      │ Key Innovation       │
├──────────┼──────────┼──────────┼──────────────────────┤
│ YOLOv1   │  63.4    │   45     │ Single-shot detection│
│ YOLOv2   │  78.6    │   40     │ Anchor boxes, BN     │
│ YOLOv3   │  57.9*   │   30     │ Multi-scale, FPN     │
│ YOLOv4   │  65.7*   │   62     │ CSP, Mish, mosaic    │
│ YOLOv5   │  68.9*   │   140    │ PyTorch, auto-anchor │
│ YOLOv8   │  53.9*   │   280    │ Anchor-free, C2f     │
└──────────┴──────────┴──────────┴──────────────────────┘
* mAP@0.5:0.95 on COCO (stricter metric)
```

### SSD (Single Shot MultiBox Detector)

```
SSD: Multi-scale detection from multiple feature map layers

  ┌──────────────────────────────────────────────────────┐
  │  Image → VGG-16/ResNet backbone                      │
  │                                                      │
  │  Feature maps at multiple scales:                    │
  │                                                      │
  │  38×38 ─── detect small objects                      │
  │   │                                                  │
  │  19×19 ─── detect medium objects                     │
  │   │                                                  │
  │  10×10 ─── detect medium-large objects               │
  │   │                                                  │
  │   5×5 ──── detect large objects                      │
  │   │                                                  │
  │   3×3 ──── detect very large objects                 │
  │   │                                                  │
  │   1×1 ──── detect full-image objects                 │
  │                                                      │
  │  At each scale: k anchors per position               │
  │  Predict: (class_scores, box_offsets) per anchor     │
  │                                                      │
  │  Total predictions: 38²×4 + 19²×6 + 10²×6 + ...    │
  │                    = 8732 boxes                       │
  │  After NMS: ~100-200 detections                      │
  └──────────────────────────────────────────────────────┘
```

---

## 5. Non-Maximum Suppression (NMS)

```
Problem: Multiple overlapping boxes for the same object

Before NMS:                   After NMS:
┌────────────────────┐       ┌────────────────────┐
│  ┌─────┐           │       │  ┌─────┐           │
│  │ ┌───┼──┐        │       │  │ CAR │           │
│  │ │CAR│  │        │  ──►  │  │ 0.95│           │
│  │ │0.95  │ 0.87│  │       │  └─────┘           │
│  └─┤     ─┘    │  │       │                    │
│    └───────────┘   │       │                    │
└────────────────────┘       └────────────────────┘

NMS Algorithm:
  1. Sort all boxes by confidence score (descending)
  2. Select highest-confidence box → add to final list
  3. Remove all remaining boxes with IoU > threshold (e.g., 0.5) 
     with the selected box
  4. Repeat from step 2 until no boxes remain

Step-by-step example (IoU threshold = 0.5):
  Boxes sorted: [A(0.95), B(0.87), C(0.82), D(0.70)]
  
  Step 1: Select A(0.95). Compute IoU with others:
    IoU(A,B)=0.72 > 0.5 → remove B
    IoU(A,C)=0.15 < 0.5 → keep C
    IoU(A,D)=0.60 > 0.5 → remove D
  
  Step 2: Select C(0.82). No remaining boxes overlap.
  
  Final: [A(0.95), C(0.82)]  — two distinct objects detected
```

---

## 6. Evaluation: mAP Metric

### Mean Average Precision

```
mAP Computation:

Step 1: For each class, rank detections by confidence
Step 2: Compute precision-recall curve

  Match detection to ground truth (IoU ≥ threshold):
  
  Det# │ Conf │ TP/FP │ Precision │ Recall
  ─────┼──────┼───────┼───────────┼────────
    1  │ 0.95 │  TP   │  1/1=1.00 │ 1/5=0.20
    2  │ 0.90 │  TP   │  2/2=1.00 │ 2/5=0.40
    3  │ 0.85 │  FP   │  2/3=0.67 │ 2/5=0.40
    4  │ 0.80 │  TP   │  3/4=0.75 │ 3/5=0.60
    5  │ 0.70 │  FP   │  3/5=0.60 │ 3/5=0.60
    6  │ 0.65 │  TP   │  4/6=0.67 │ 4/5=0.80
    7  │ 0.60 │  TP   │  5/7=0.71 │ 5/5=1.00

Step 3: Interpolate precision at 11 recall points [0, 0.1, ..., 1.0]
  AP = (1/11) × Σ max_precision_at_recall

Step 4: Average AP across all classes
  mAP = (1/K) × Σ AP_k

COCO metrics:
  mAP@0.5:      IoU threshold = 0.5
  mAP@0.75:     IoU threshold = 0.75 (stricter)
  mAP@0.5:0.95: Average over IoU from 0.5 to 0.95 (step 0.05) — primary metric
```

---

## 7. Architecture Comparison

```
┌────────────────┬──────────┬────────┬──────────┬──────────────────┐
│   Detector     │ mAP(COCO)│ Speed  │ Stages   │ Best For         │
├────────────────┼──────────┼────────┼──────────┼──────────────────┤
│ Faster R-CNN   │  42.0    │ 5 FPS  │ Two-stage│ High accuracy    │
│ Cascade R-CNN  │  44.3    │ 4 FPS  │ Two-stage│ Precise boxes    │
│ YOLOv8-L       │  52.9    │ 70 FPS │ One-stage│ Real-time        │
│ DETR           │  42.0    │ 28 FPS │ End2End  │ No NMS/anchors   │
│ RT-DETR        │  54.3    │ 74 FPS │ End2End  │ Real-time E2E    │
│ SSD300         │  41.2    │ 46 FPS │ One-stage│ Simple, fast     │
└────────────────┴──────────┴────────┴──────────┴──────────────────┘

Two-Stage vs Single-Stage:
┌──────────────────┬──────────────────────┬──────────────────────┐
│                  │ Two-Stage            │ Single-Stage         │
├──────────────────┼──────────────────────┼──────────────────────┤
│ Speed            │ Slower               │ Faster               │
│ Accuracy         │ Generally higher     │ Competitive now      │
│ Small objects    │ Better (RPN focus)   │ Harder               │
│ Architecture     │ Complex              │ Simpler              │
│ Training         │ More complex         │ End-to-end           │
│ Examples         │ Faster R-CNN, R-FCN  │ YOLO, SSD, RetinaNet│
└──────────────────┴──────────────────────┴──────────────────────┘
```

---

## 8. Worked Examples

### Example: IoU Computation

```
Box A: (x₁=100, y₁=100, x₂=300, y₂=250)
Box B: (x₁=150, y₁=120, x₂=350, y₂=280)

Intersection:
  x_left   = max(100, 150) = 150
  y_top    = max(100, 120) = 120
  x_right  = min(300, 350) = 300
  y_bottom = min(250, 280) = 250
  
  width  = 300 - 150 = 150
  height = 250 - 120 = 130
  intersection = 150 × 130 = 19,500

Areas:
  area_A = (300-100) × (250-100) = 200 × 150 = 30,000
  area_B = (350-150) × (280-120) = 200 × 160 = 32,000

Union:
  union = 30,000 + 32,000 - 19,500 = 42,500

IoU = 19,500 / 42,500 = 0.459

Since IoU < 0.5, these boxes do NOT match (at IoU@0.5 threshold)
```

---

## 9. PyTorch Implementation

### IoU Computation

```python
import torch

def compute_iou(boxes1, boxes2):
    """Compute IoU between two sets of boxes.
    boxes: (N, 4) in format (x1, y1, x2, y2)
    """
    x1 = torch.max(boxes1[:, 0], boxes2[:, 0])
    y1 = torch.max(boxes1[:, 1], boxes2[:, 1])
    x2 = torch.min(boxes1[:, 2], boxes2[:, 2])
    y2 = torch.min(boxes1[:, 3], boxes2[:, 3])
    
    intersection = torch.clamp(x2 - x1, min=0) * torch.clamp(y2 - y1, min=0)
    
    area1 = (boxes1[:, 2] - boxes1[:, 0]) * (boxes1[:, 3] - boxes1[:, 1])
    area2 = (boxes2[:, 2] - boxes2[:, 0]) * (boxes2[:, 3] - boxes2[:, 1])
    
    union = area1 + area2 - intersection
    return intersection / (union + 1e-6)
```

### Using Pretrained Detection Models

```python
import torchvision
from torchvision.models.detection import (
    fasterrcnn_resnet50_fpn_v2,
    FasterRCNN_ResNet50_FPN_V2_Weights,
)

# Faster R-CNN
model = fasterrcnn_resnet50_fpn_v2(
    weights=FasterRCNN_ResNet50_FPN_V2_Weights.COCO_V1
)
model.eval()

# Inference
image = torch.rand(3, 600, 800)
with torch.no_grad():
    predictions = model([image])

# predictions[0] contains:
#   'boxes':  (N, 4) — bounding boxes
#   'labels': (N,)   — class labels  
#   'scores': (N,)   — confidence scores

for box, label, score in zip(predictions[0]['boxes'],
                              predictions[0]['labels'],
                              predictions[0]['scores']):
    if score > 0.5:
        print(f"Class {label.item()}: {score:.2f} at {box.tolist()}")

# YOLOv8 with ultralytics
# pip install ultralytics
from ultralytics import YOLO
model = YOLO('yolov8n.pt')  # nano model
results = model('image.jpg')
```

---

## 10. Summary Table

| Concept | Key Idea |
|---|---|
| **Object detection** | Classification + localization of multiple objects |
| **IoU** | Overlap area / union area — measures box match quality |
| **Anchor boxes** | Predefined reference boxes; predict offsets from anchors |
| **R-CNN → Faster R-CNN** | Region proposals → shared features → RPN (learnable) |
| **YOLO** | Single-shot: divide image into grid, predict all at once |
| **SSD** | Multi-scale feature maps for detecting objects of different sizes |
| **NMS** | Remove duplicate detections by suppressing overlapping boxes |
| **mAP** | Mean of per-class average precision (area under P-R curve) |
| **Two-stage** | Higher accuracy, slower (Faster R-CNN) |
| **Single-stage** | Real-time speed, competitive accuracy (YOLO, SSD) |

---

## 11. Revision Questions

1. **Explain IoU** and compute it for two boxes: A=(50,50,200,200) and B=(100,100,250,250). Would this count as a match at IoU@0.5?

2. **Trace the evolution from R-CNN to Faster R-CNN.** What bottleneck did each version solve? What makes Faster R-CNN end-to-end trainable?

3. **How does YOLO differ fundamentally** from two-stage detectors? Draw the YOLO grid and explain what each cell predicts.

4. **Explain NMS step-by-step** with an example of 5 overlapping boxes. Why is NMS necessary?

5. **Compute mAP** given 5 detections and 3 ground truth boxes, with confidence scores and IoU values provided.

6. **Compare YOLO, SSD, and Faster R-CNN** for a real-time drone surveillance application. Which would you choose and why?

---

## 12. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Image Classification](./01-image-classification.md) | [Unit 8 Index](../08-CNN-Applications/) | [Semantic Segmentation →](./03-semantic-segmentation.md) |

---

> **References:**  
> - Girshick, R. et al. (2014). *Rich Feature Hierarchies for Accurate Object Detection.* CVPR (R-CNN).  
> - Ren, S. et al. (2016). *Faster R-CNN: Towards Real-Time Object Detection.* NeurIPS.  
> - Redmon, J. et al. (2016). *You Only Look Once: Unified, Real-Time Object Detection.* CVPR.  
> - Liu, W. et al. (2016). *SSD: Single Shot MultiBox Detector.* ECCV.
