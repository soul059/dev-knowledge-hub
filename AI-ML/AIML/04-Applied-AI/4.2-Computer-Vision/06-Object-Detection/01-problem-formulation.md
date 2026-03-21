# Object Detection: Problem Formulation

## Overview

Object detection finds **what** objects are in an image and **where** they are located. Unlike classification (one label per image), detection outputs multiple bounding boxes, each with a class label and confidence score. It's the foundation for autonomous driving, surveillance, robotics, and medical imaging.

---

## Task Definition

```
Input:  Image I (H × W × 3)
Output: List of detections, each containing:
  - Bounding box: (x, y, w, h) or (x1, y1, x2, y2)
  - Class label: c ∈ {car, person, dog, ...}
  - Confidence: p ∈ [0, 1]

Example output:
  ┌─────────────────────────────────────┐
  │                                     │
  │   ┌──────┐      ┌─────────┐        │
  │   │ cat  │      │ person  │        │
  │   │ 0.95 │      │  0.88   │        │
  │   └──────┘      └─────────┘        │
  │          ┌──────────┐               │
  │          │   dog    │               │
  │          │   0.92   │               │
  │          └──────────┘               │
  └─────────────────────────────────────┘

  Detections:
    [cat,    (50, 30, 120, 100), 0.95]
    [person, (200, 20, 300, 250), 0.88]
    [dog,    (150, 180, 280, 300), 0.92]
```

---

## Bounding Box Formats

```
Two common formats:

  (x, y, w, h) — COCO format:
    (x, y) = top-left corner
    w, h = width, height

  (x1, y1, x2, y2) — Pascal VOC format:
    (x1, y1) = top-left corner
    (x2, y2) = bottom-right corner

  Conversion:
    x1, y1 = x, y
    x2, y2 = x + w, y + h

  Normalized (YOLO format):
    (cx, cy, w, h) all in [0, 1] relative to image size
    cx, cy = center of box
```

---

## Detection Approaches

```
┌─────────────────────────────────────────────────┐
│              Object Detection Methods            │
├─────────────────────────────────────────────────┤
│                                                 │
│  Two-stage detectors:                           │
│    1. Propose regions → 2. Classify each        │
│    R-CNN → Fast R-CNN → Faster R-CNN            │
│    Accurate but slower                          │
│                                                 │
│  Single-stage detectors:                        │
│    Direct: image → detections                   │
│    YOLO, SSD, RetinaNet                         │
│    Faster but less accurate (historically)      │
│                                                 │
│  Transformer-based:                             │
│    DETR, DINO                                   │
│    End-to-end, no NMS needed                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## Loss Function Components

```
Detection loss has multiple parts:

  L_total = L_classification + λ₁ × L_localization + λ₂ × L_objectness

  Classification: Is this a cat, dog, or person?
    → CrossEntropy or Focal Loss

  Localization: Where exactly is the box?
    → Smooth L1 Loss or IoU Loss
    L_loc = Σ SmoothL1(predicted_box - ground_truth_box)

  Objectness: Is there an object here at all?
    → Binary CrossEntropy (object vs background)
```

---

## Key Datasets

| Dataset | Classes | Images | Boxes | Metric |
|---------|:-:|:-:|:-:|:-:|
| Pascal VOC | 20 | 11K | ~27K | mAP@0.5 |
| MS COCO | 80 | 330K | 1.5M | mAP@[0.5:0.95] |
| Open Images | 600 | 1.9M | 15M | mAP |
| Objects365 | 365 | 2M | 30M | mAP |

---

## Revision Questions

1. **How does object detection differ from image classification?**
2. **What are the three components of a detection output?**
3. **What are the main differences between two-stage and single-stage detectors?**
4. **What are the three parts of the detection loss function?**
5. **What is the difference between COCO and Pascal VOC bounding box formats?**

---

[Previous: ../05-Image-Classification/05-multi-label.md](../05-Image-Classification/05-multi-label.md) | [Next: 02-two-stage-detectors.md](02-two-stage-detectors.md)
