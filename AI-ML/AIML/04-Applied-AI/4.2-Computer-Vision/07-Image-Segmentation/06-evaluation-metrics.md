# Segmentation Evaluation Metrics

## Overview

Evaluating segmentation models requires pixel-level metrics that capture both classification accuracy and spatial agreement. Different segmentation tasks (semantic, instance, panoptic) use different primary metrics, but all are built on the concepts of pixel overlap and intersection.

---

## Mean IoU (mIoU) — Semantic Segmentation

```
IoU per class = True Positive / (TP + FP + FN)

For class "car":
  TP = pixels correctly predicted as car
  FP = pixels incorrectly predicted as car
  FN = car pixels missed (predicted as something else)

  IoU_car = |Pred_car ∩ GT_car| / |Pred_car ∪ GT_car|

  Predicted             Ground Truth          Intersection
  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
  │     CCCC     │      │      CCCC    │      │      CCC     │
  │    CCCCC     │      │     CCCCC    │      │     CCCC     │
  │    CCCCC     │      │     CCCCC    │      │     CCCC     │
  │     CCC      │      │      CCC     │      │      CC      │
  └──────────────┘      └──────────────┘      └──────────────┘
  
  IoU_car = |Intersection| / |Union| = 14 / 22 = 0.636

mIoU = (1/C) × Σ IoU_c    (average over all C classes)

Example:
  Class        IoU
  Road         0.95
  Car          0.72
  Person       0.65
  Sky          0.92
  Building     0.78
  Tree         0.80
  
  mIoU = (0.95+0.72+0.65+0.92+0.78+0.80) / 6 = 0.803
```

---

## Pixel Accuracy

```
Pixel Accuracy = Total correct pixels / Total pixels

  Simple but misleading with class imbalance!

  Example: 90% of image is "background"
    Model predicts everything as background
    → 90% pixel accuracy, but completely useless

  ┌──────────────────────────────────────┐
  │ BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB │  B = Background (90%)
  │ BBBBBBBB  CCCCC  BBBBB  PP  BBBBB  │  C = Car (7%)
  │ BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB │  P = Person (3%)
  └──────────────────────────────────────┘
  
  "All background" model: 90% accuracy, 0 mIoU for car/person
  → mIoU is the better metric

Frequency-weighted IoU:
  FWIoU = Σ (freq_c × IoU_c)
  Weights each class by its frequency
  Balances between pixel accuracy and mIoU
```

---

## Dice Coefficient (F1 Score)

```
Dice = 2 × |Pred ∩ GT| / (|Pred| + |GT|)

  Very common in medical image segmentation

  Relationship to IoU:
    Dice = 2 × IoU / (1 + IoU)
    IoU  = Dice / (2 - Dice)

  When Dice = 0.8 → IoU = 0.667
  When Dice = 0.9 → IoU = 0.818

  Dice is always ≥ IoU for the same prediction
  → Numbers look better but carry the same information

  Dice Loss = 1 - Dice  (differentiable, used for training)
  
  Dice loss handles class imbalance better than cross-entropy
  because it normalizes by the sizes of prediction and GT
```

---

## AP (Average Precision) — Instance Segmentation

```
Same concept as object detection AP, but using mask IoU:

  1. Compute mask IoU between predicted and GT masks
  2. Match predictions to GT (IoU ≥ threshold → TP)
  3. Sort predictions by confidence
  4. Compute precision-recall curve
  5. AP = area under PR curve

COCO Instance Segmentation Metrics:
  AP          mAP @ IoU [0.5:0.95]  (primary, mask IoU)
  AP_50       mAP @ IoU 0.5
  AP_75       mAP @ IoU 0.75
  AP_S        AP for small objects (area < 32²)
  AP_M        AP for medium objects (32² < area < 96²)
  AP_L        AP for large objects (area > 96²)

Note: Mask IoU is computed pixel-by-pixel:
  Mask IoU = |mask_pred ∩ mask_gt| / |mask_pred ∪ mask_gt|
```

---

## PQ (Panoptic Quality) — Panoptic Segmentation

```
PQ = SQ × RQ

  SQ (Segmentation Quality) = average IoU of matched segments
  RQ (Recognition Quality) = F1 of segment matching

  Computed per class, then averaged:

  PQ = (1/C) × Σ_c PQ_c

  PQ_c = (Σ_{(p,g)∈TP_c} IoU(p,g)) / (|TP_c| + ½|FP_c| + ½|FN_c|)

  PQ reported separately for:
    PQ_things: PQ over thing classes only
    PQ_stuff:  PQ over stuff classes only
    PQ_all:    PQ over all classes (primary)

  Example:
    Class     TP  FP  FN  Avg IoU   SQ     RQ     PQ
    Car       8   2   1   0.78     0.78   0.842  0.657
    Person    5   3   2   0.72     0.72   0.667  0.480
    Road      1   0   0   0.91     0.91   1.000  0.910
    Sky       1   0   0   0.95     0.95   1.000  0.950
    
    PQ_things = (0.657 + 0.480) / 2 = 0.569
    PQ_stuff  = (0.910 + 0.950) / 2 = 0.930
    PQ_all    = (0.657+0.480+0.910+0.950) / 4 = 0.749
```

---

## Boundary Metrics

```
Standard IoU treats all pixels equally
Boundary-focused metrics evaluate edge quality:

Boundary IoU (Boundary F1):
  Only evaluate pixels within d pixels of object boundary
  
  ┌──────────────────┐
  │                  │
  │   ┌──────────┐   │
  │   │ ████████ │   │  █ = boundary band (d=2 pixels)
  │   │ █      █ │   │
  │   │ █      █ │   │  Boundary IoU considers ONLY these pixels
  │   │ ████████ │   │
  │   └──────────┘   │
  │                  │
  └──────────────────┘

  Boundary F1 = 2 × BP × BR / (BP + BR)
  BP = Boundary Precision
  BR = Boundary Recall
  
  Useful for: autonomous driving (precise lane boundaries),
              medical (tumor boundary accuracy)
```

---

## Python: Computing Metrics

```python
import numpy as np

def compute_iou_per_class(pred, gt, num_classes):
    """Compute IoU for each class."""
    ious = []
    for c in range(num_classes):
        pred_c = (pred == c)
        gt_c = (gt == c)
        
        intersection = np.logical_and(pred_c, gt_c).sum()
        union = np.logical_or(pred_c, gt_c).sum()
        
        if union == 0:
            ious.append(float('nan'))  # class not present
        else:
            ious.append(intersection / union)
    return ious

def compute_miou(pred, gt, num_classes):
    ious = compute_iou_per_class(pred, gt, num_classes)
    valid = [x for x in ious if not np.isnan(x)]
    return np.mean(valid)

def dice_coefficient(pred, gt):
    intersection = np.logical_and(pred, gt).sum()
    return 2 * intersection / (pred.sum() + gt.sum() + 1e-8)

# Example
pred = np.array([[0, 0, 1, 1],
                 [0, 1, 1, 1],
                 [2, 2, 1, 1],
                 [2, 2, 2, 0]])

gt = np.array([[0, 0, 0, 1],
               [0, 1, 1, 1],
               [2, 2, 1, 1],
               [2, 2, 2, 2]])

ious = compute_iou_per_class(pred, gt, num_classes=3)
for c, iou in enumerate(ious):
    print(f"Class {c}: IoU = {iou:.3f}")
print(f"mIoU = {compute_miou(pred, gt, 3):.3f}")
```

---

## Summary

| Metric | Task | Formula | Range |
|--------|------|---------|-------|
| mIoU | Semantic seg | Mean of per-class IoU | 0-1 |
| Pixel Accuracy | Semantic seg | Correct / Total pixels | 0-1 |
| Dice / F1 | Medical seg | 2\|P∩G\| / (\|P\|+\|G\|) | 0-1 |
| AP (mask) | Instance seg | Area under mask PR curve | 0-1 |
| PQ | Panoptic seg | SQ × RQ | 0-1 |
| Boundary F1 | Any | Precision × Recall on boundaries | 0-1 |

---

## Revision Questions

1. **Why is mIoU preferred over pixel accuracy for semantic segmentation?**
2. **What is the relationship between Dice coefficient and IoU?**
3. **How does AP for instance segmentation differ from detection AP?**
4. **What do SQ and RQ measure in Panoptic Quality?**
5. **When would boundary metrics be more informative than mIoU?**

---

[Previous: 05-mask-rcnn.md](05-mask-rcnn.md) | [Back to README](../README.md)
