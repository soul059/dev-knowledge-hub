# Detection Evaluation Metrics (mAP, IoU)

## Overview

Evaluating object detection requires metrics that measure both **localization accuracy** (where is the box?) and **classification accuracy** (what class is it?). Mean Average Precision (mAP) is the standard metric, built on top of Intersection over Union (IoU), precision, and recall.

---

## IoU (Intersection over Union)

```
IoU measures how well a predicted box matches the ground truth:

  Ground Truth        Prediction        Overlap
  ┌──────────┐       ┌──────────┐       
  │          │       │          │       IoU = Intersection / Union
  │   GT     │       │   Pred   │       
  │     ┌────┼───┐   │          │       IoU ≥ 0.5 → True Positive (TP)
  │     │////│   │   │          │       IoU < 0.5 → False Positive (FP)
  └─────┼────┘   │   └──────────┘
        └────────┘

  Common IoU thresholds:
    0.5   → lenient (PASCAL VOC standard)
    0.75  → strict
    0.5:0.95 → COCO standard (average over 0.5, 0.55, ..., 0.95)
```

---

## Precision and Recall in Detection

```
For each class:

  TP = prediction with IoU ≥ threshold matching a GT box
  FP = prediction with IoU < threshold OR duplicate match
  FN = GT box with no matching prediction

  Precision = TP / (TP + FP)     "Of all predictions, how many are correct?"
  Recall    = TP / (TP + FN)     "Of all GT objects, how many did we find?"

  Example: Image has 3 cars (GT). Detector predicts 5 boxes.
    Pred 1: IoU=0.82 with GT1 → TP     Precision = 3/5 = 0.60
    Pred 2: IoU=0.71 with GT2 → TP     Recall    = 3/3 = 1.00
    Pred 3: IoU=0.55 with GT3 → TP
    Pred 4: IoU=0.30 with GT1 → FP (low IoU)
    Pred 5: IoU=0.60 with GT1 → FP (duplicate, GT1 already matched)
```

---

## Precision-Recall Curve

```
1. Sort all predictions by confidence (descending)
2. Walk through predictions, updating TP/FP counts
3. Compute precision and recall at each step

  Predictions sorted by confidence:
  Rank  Conf   Match    TP  FP   Prec    Recall
  1     0.95   TP       1   0    1.00    0.33
  2     0.88   TP       2   0    1.00    0.67
  3     0.72   FP       2   1    0.67    0.67
  4     0.65   TP       3   1    0.75    1.00
  5     0.40   FP       3   2    0.60    1.00

  PR Curve:
  Precision
  1.0 ┤●─────●
      │       \
  0.8 ┤        \
      │         ●─────●
  0.6 ┤               │
      │               │
  0.4 ┤               │
      │               │
  0.2 ┤               │
      │               │
  0.0 ┼───┬───┬───┬───┤
      0  0.25 0.5 0.75 1.0  Recall
```

---

## Average Precision (AP)

```
AP = Area under the Precision-Recall curve

Method 1: 11-point interpolation (PASCAL VOC 2007)
  Sample at recall = {0, 0.1, 0.2, ..., 1.0}
  AP = (1/11) × Σ max_precision at each recall point

Method 2: All-point interpolation (PASCAL VOC 2010+, COCO)
  Interpolate precision at every recall change point
  AP = Σ (r_{n+1} - r_n) × p_interp(r_{n+1})
  where p_interp(r) = max(p(r') for r' ≥ r)

  From our example:
  Interpolated precision:
    Recall 0.0-0.33  → precision = 1.00
    Recall 0.33-0.67 → precision = 1.00
    Recall 0.67-1.00 → precision = 0.75

  AP = 0.33×1.0 + 0.34×1.0 + 0.33×0.75 = 0.917
```

---

## Mean Average Precision (mAP)

```
mAP = mean of AP across all classes

  Class       AP@0.5
  Car         0.85
  Person      0.78
  Dog         0.65
  Cat         0.72
  Bike        0.81

  mAP@0.5 = (0.85 + 0.78 + 0.65 + 0.72 + 0.81) / 5 = 0.762

COCO mAP (primary metric):
  mAP@[0.5:0.95] = average mAP at IoU thresholds 0.5, 0.55, ..., 0.95
  This is much stricter than mAP@0.5

  mAP@0.5   → how well does it find objects (lenient)
  mAP@0.75  → how precisely does it localize (strict)
  mAP@[0.5:0.95] → overall quality
```

---

## COCO Evaluation Suite

```
COCO provides detailed breakdown:

  Metric          Description
  ──────────────────────────────────────────
  AP              mAP@[0.5:0.95] (primary)
  AP_50           mAP@0.5
  AP_75           mAP@0.75
  AP_S            AP for small objects (< 32² pixels)
  AP_M            AP for medium objects (32²-96² pixels)
  AP_L            AP for large objects (> 96² pixels)
  AR_1            Avg recall with 1 detection/image
  AR_10           Avg recall with 10 detections/image
  AR_100          Avg recall with 100 detections/image
```

---

## Python: Computing mAP

```python
from pycocotools.coco import COCO
from pycocotools.cocoeval import COCOeval

# Load ground truth and detections
coco_gt = COCO("annotations.json")
coco_dt = coco_gt.loadRes("detections.json")

# Evaluate
coco_eval = COCOeval(coco_gt, coco_dt, "bbox")
coco_eval.evaluate()
coco_eval.accumulate()
coco_eval.summarize()

# Output:
# AP @[IoU=0.50:0.95] = 0.432
# AP @[IoU=0.50]      = 0.654
# AP @[IoU=0.75]      = 0.471
# AP (small)           = 0.213
# AP (medium)          = 0.478
# AP (large)           = 0.602


# Manual AP computation
from sklearn.metrics import average_precision_score

y_true = [1, 1, 0, 1, 0]    # TP/FP labels
y_scores = [0.95, 0.88, 0.72, 0.65, 0.40]  # confidence
ap = average_precision_score(y_true, y_scores)
print(f"AP = {ap:.3f}")
```

---

## Summary

| Metric | Formula | Use |
|--------|---------|-----|
| IoU | Intersection / Union | Match predictions to GT |
| Precision | TP / (TP + FP) | Quality of predictions |
| Recall | TP / (TP + FN) | Coverage of GT objects |
| AP | Area under PR curve | Per-class performance |
| mAP | Mean of AP over classes | Overall detector quality |
| mAP@[.5:.95] | Mean mAP at multiple IoU | COCO primary metric |

---

## Revision Questions

1. **How is IoU computed and what threshold defines a correct detection?**
2. **Why do we need both precision and recall for detection evaluation?**
3. **How is the precision-recall curve constructed?**
4. **What is the difference between mAP@0.5 and mAP@[0.5:0.95]?**
5. **Why does COCO report AP_S, AP_M, and AP_L separately?**

---

[Previous: 05-nms.md](05-nms.md) | [Back to README](../README.md)
