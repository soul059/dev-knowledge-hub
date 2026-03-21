# Non-Maximum Suppression (NMS)

## Overview

Object detectors generate many overlapping predictions for each object. **Non-Maximum Suppression (NMS)** is a post-processing step that filters these redundant detections, keeping only the best bounding box per object. It is essential in virtually every detection pipeline.

---

## The Problem

```
Before NMS — detector outputs many overlapping boxes for one cat:

  ┌──────────────────────┐
  │  ┌─────────────────┐ │  Box A: cat, 0.95
  │  │ ┌──────────────┐│ │  Box B: cat, 0.88
  │  │ │ ┌───────────┐││ │  Box C: cat, 0.72
  │  │ │ │           │││ │  Box D: cat, 0.45
  │  │ │ │   🐱     │││ │
  │  │ │ │           │││ │  All correct, but we want just ONE box
  │  │ │ └───────────┘││ │
  │  │ └──────────────┘│ │
  │  └─────────────────┘ │
  └──────────────────────┘

After NMS — only the most confident box remains:

  ┌──────────────────────┐
  │  ┌─────────────────┐ │  Box A: cat, 0.95 ✓ (kept)
  │  │                 │ │  Box B: removed (IoU with A = 0.85 > 0.5)
  │  │     🐱         │ │  Box C: removed (IoU with A = 0.72 > 0.5)
  │  │                 │ │  Box D: removed (IoU with A = 0.60 > 0.5)
  │  └─────────────────┘ │
  └──────────────────────┘
```

---

## Standard NMS Algorithm

```
Algorithm: Greedy NMS
Input:  B = list of boxes, S = list of scores, threshold τ
Output: D = final detections

1. Sort B by scores S in descending order
2. D = []
3. While B is not empty:
   a. Pick box b with highest score → add to D
   b. Remove b from B
   c. For each remaining box b_i in B:
      - Compute IoU(b, b_i)
      - If IoU > τ: remove b_i from B   (suppressed)
4. Return D

Typical threshold: τ = 0.5

Step-by-step example:
  Boxes sorted by score: [A(0.95), B(0.88), C(0.72), D(0.45)]
  
  Step 1: Select A(0.95) → D = [A]
          IoU(A,B) = 0.85 > 0.5 → remove B
          IoU(A,C) = 0.72 > 0.5 → remove C
          IoU(A,D) = 0.60 > 0.5 → remove D
  
  Step 2: B is empty → done
  Result: D = [A]
```

---

## IoU Refresh

```
IoU (Intersection over Union):

  ┌─────────┐
  │    A     │
  │    ┌─────┼─────┐
  │    │/////│     │    IoU = Area of Intersection
  └────┼─────┘     │         ─────────────────────
       │     B     │         Area of Union
       └───────────┘

  IoU = |A ∩ B| / |A ∪ B|
  IoU = |A ∩ B| / (|A| + |B| - |A ∩ B|)

  IoU = 0 → no overlap
  IoU = 1 → perfect overlap
```

---

## NMS Variants

```
1. Soft-NMS
   Problem: Hard NMS removes boxes completely → misses nearby objects
   Solution: Reduce score instead of removing
   
   Standard: score_i = 0          if IoU > τ
   Soft-NMS: score_i *= exp(-IoU² / σ)   (Gaussian decay)
   
   → Nearby objects keep some score, re-ranked rather than deleted

2. DIoU-NMS
   Standard NMS uses IoU only
   DIoU-NMS considers center distance too:
   
   Suppress if: IoU - (d²/c²) > threshold
     d = distance between centers
     c = diagonal of smallest enclosing box
   
   → Better at keeping nearby distinct objects

3. Weighted NMS
   Instead of picking one box, merge overlapping boxes:
   
   final_box = Σ(score_i × box_i) / Σ(score_i)
   
   → Averaged box is more precise

4. Class-Specific NMS
   Run NMS independently per class
   → "cat" boxes don't suppress "dog" boxes
   (This is the standard practice in most detectors)
```

---

## Python Implementation

```python
import numpy as np

def nms(boxes, scores, iou_threshold=0.5):
    """
    boxes: Nx4 array of (x1, y1, x2, y2)
    scores: N array of confidence scores
    Returns: indices of kept boxes
    """
    x1, y1, x2, y2 = boxes[:, 0], boxes[:, 1], boxes[:, 2], boxes[:, 3]
    areas = (x2 - x1) * (y2 - y1)
    order = scores.argsort()[::-1]
    
    keep = []
    while order.size > 0:
        i = order[0]
        keep.append(i)
        
        # Compute IoU with remaining boxes
        xx1 = np.maximum(x1[i], x1[order[1:]])
        yy1 = np.maximum(y1[i], y1[order[1:]])
        xx2 = np.minimum(x2[i], x2[order[1:]])
        yy2 = np.minimum(y2[i], y2[order[1:]])
        
        w = np.maximum(0, xx2 - xx1)
        h = np.maximum(0, yy2 - yy1)
        intersection = w * h
        
        iou = intersection / (areas[i] + areas[order[1:]] - intersection)
        
        # Keep boxes with IoU below threshold
        inds = np.where(iou <= iou_threshold)[0]
        order = order[inds + 1]
    
    return np.array(keep)


# Using torchvision (GPU-accelerated)
import torchvision
keep = torchvision.ops.nms(boxes_tensor, scores_tensor, iou_threshold=0.5)
```

---

## Summary

| Method | Mechanism | Advantage | Drawback |
|--------|-----------|-----------|----------|
| Greedy NMS | Hard removal | Simple, fast | Misses crowded objects |
| Soft-NMS | Score decay | Handles crowds | Slower, needs σ tuning |
| DIoU-NMS | Center distance | Better for nearby objects | More computation |
| Weighted NMS | Score-weighted merge | More precise boxes | Slower |

---

## Revision Questions

1. **Why is NMS necessary in object detection pipelines?**
2. **Walk through the greedy NMS algorithm step by step.**
3. **What problem does Soft-NMS solve that standard NMS cannot?**
4. **How does DIoU-NMS improve upon IoU-based NMS?**
5. **Why is class-specific NMS standard practice?**

---

[Previous: 04-anchor-methods.md](04-anchor-methods.md) | [Next: 06-evaluation-metrics.md](06-evaluation-metrics.md)
