# Anchor-Based vs Anchor-Free Detection

## Overview

Object detectors differ in how they define candidate detection regions. **Anchor-based** methods use predefined reference boxes (anchors) at each spatial location, while **anchor-free** methods directly predict object locations without reference boxes. Modern detectors increasingly favor anchor-free approaches for their simplicity.

---

## Anchor-Based Detection

```
At each feature map position, define K predefined boxes (anchors):

  Feature map position (i, j):
  ┌─────────────────────────────────┐
  │   ┌──────┐                     │
  │   │anchor│  ← 128×128, ratio 1:1
  │   │  1   │                     │
  │   └──────┘                     │
  │  ┌──────────────┐              │
  │  │  anchor 2    │ ← 128×256, ratio 1:2
  │  └──────────────┘              │
  │  ┌────────────────────┐        │
  │  │   anchor 3         │ ← 256×128, ratio 2:1
  │  └────────────────────┘        │
  └─────────────────────────────────┘

  For each anchor, predict:
    - Offset (Δx, Δy, Δw, Δh) to refine box
    - Objectness score (is there an object?)
    - Class probabilities

  Refined box:
    x = anchor_x + Δx × anchor_w
    y = anchor_y + Δy × anchor_h
    w = anchor_w × exp(Δw)
    h = anchor_h × exp(Δh)

  Models: Faster R-CNN, SSD, YOLOv3-v5, RetinaNet
```

---

## Anchor-Free Detection

```
Approach 1: Keypoint-based (CornerNet, CenterNet)
  Detect object as keypoints:
  
  CornerNet: Detect top-left + bottom-right corners → pair them
  CenterNet: Detect object center → predict w, h from center
  
  ┌─────────────────┐
  │ ●               │   ● = top-left corner (CornerNet)
  │                 │   ○ = bottom-right corner
  │     ★           │   ★ = center point (CenterNet)
  │                 │
  │               ○ │
  └─────────────────┘

Approach 2: Dense prediction (FCOS)
  For each feature map pixel inside a GT box:
    Predict distances to box edges: (l, t, r, b)
    
    ┌──────────────────────┐
    │         t            │   l = distance to left edge
    │     ┌───┼───┐        │   t = distance to top edge
    │  l ─┤   ★   ├─ r    │   r = distance to right edge
    │     └───┼───┘        │   b = distance to bottom edge
    │         b            │
    └──────────────────────┘
    
  + centerness score (high at center, low at edges)
  Models: FCOS, YOLOv8
```

---

## Comparison

| Aspect | Anchor-Based | Anchor-Free |
|--------|:---:|:---:|
| Hyperparameters | Many (sizes, ratios, scales) | Few |
| Design complexity | Complex anchor configuration | Simpler |
| Speed | Slower (more computations) | Faster |
| Generalization | Sensitive to anchor design | Better cross-dataset |
| Small objects | Depends on anchor coverage | Can be better (FCOS) |
| Accuracy | Very good | Comparable or better |
| Examples | Faster R-CNN, RetinaNet, YOLOv5 | CenterNet, FCOS, YOLOv8 |

---

## RetinaNet: Focal Loss

```
Problem: In anchor-based detectors, 99% of anchors are background
         → Easy negatives dominate the loss → model learns nothing useful

Solution: Focal Loss (Lin et al., 2017)

  Standard CE:  L = -log(p)
  Focal Loss:   L = -α(1-p)^γ × log(p)

  γ = focusing parameter (typically 2)
  
  When p is high (easy example):  (1-p)^γ ≈ 0 → loss is tiny
  When p is low (hard example):   (1-p)^γ ≈ 1 → loss is large

  Effect: Down-weights easy examples by 100×+
  Result: RetinaNet matches Faster R-CNN accuracy at single-stage speed!
```

---

## Revision Questions

1. **What are anchor boxes and how are they used in detection?**
2. **What are the main drawbacks of anchor-based detection?**
3. **How does CenterNet detect objects as center points?**
4. **How does FCOS predict bounding boxes without anchors?**
5. **What problem does Focal Loss solve and how?**

---

[Previous: 03-single-stage-detectors.md](03-single-stage-detectors.md) | [Next: 05-nms.md](05-nms.md)
