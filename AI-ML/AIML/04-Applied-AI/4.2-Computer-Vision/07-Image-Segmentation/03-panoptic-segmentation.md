# Panoptic Segmentation

## Overview

**Panoptic segmentation** unifies semantic and instance segmentation into a single task. Every pixel gets a class label (semantic) AND countable objects get individual instance IDs. It handles both **"stuff"** (amorphous regions like sky, road, grass) and **"things"** (countable objects like cars, people).

---

## Stuff vs Things

```
  ┌───────────────────────────────────────┐
  │  SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS  │  S = Sky (stuff)
  │  SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS  │  
  │  TTTTTT              TTTTTTTTTTTTT  │  T = Tree (stuff)
  │  TTTTTT   1111       TTTTTTTTTTTTT  │  1 = Person 1 (thing)
  │           1111                      │  2 = Person 2 (thing)
  │        22  1111                     │  3 = Car 1 (thing)
  │        22                           │  4 = Car 2 (thing)
  │  RRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR  │  R = Road (stuff)
  │  RRR  33333333  RRRR  4444444  RRR  │
  │  RRR  33333333  RRRR  4444444  RRR  │
  └───────────────────────────────────────┘

  Stuff classes: Sky, Road, Grass, Water, Building
    → No instance IDs (just class label per pixel)
    → Semantic segmentation handles these

  Thing classes: Person, Car, Dog, Bicycle
    → Each instance gets unique ID + class + mask
    → Instance segmentation handles these

  Panoptic = Both in one unified output!
```

---

## Panoptic Quality (PQ) Metric

```
PQ combines detection quality and segmentation quality:

  PQ = (Σ IoU(p,g)) / (TP + ½FP + ½FN)

  Equivalently:  PQ = SQ × RQ

  SQ (Segmentation Quality) = average IoU of matched segments
     SQ = (Σ IoU(p,g)) / TP

  RQ (Recognition Quality) = F1 score of segment matching
     RQ = TP / (TP + ½FP + ½FN)

  Matching: predicted segment p matches GT segment g if:
    - Same class
    - IoU(p, g) > 0.5

  Example:
    3 GT segments, 3 predicted, 2 matched
    Matched IoUs: 0.85, 0.72
    
    TP = 2, FP = 1, FN = 1
    SQ = (0.85 + 0.72) / 2 = 0.785
    RQ = 2 / (2 + 0.5 + 0.5) = 0.667
    PQ = 0.785 × 0.667 = 0.524
```

---

## Architectures

### Panoptic FPN (2019)

```
  Image → ResNet + FPN backbone
               │
        ┌──────┼──────┐
        │             │
   ┌────▼────┐   ┌────▼────┐
   │ Instance │   │Semantic │
   │  Branch  │   │ Branch  │
   │(Mask     │   │(FPN →   │
   │ R-CNN)   │   │ dense   │
   │          │   │ pred)   │
   └────┬────┘   └────┬────┘
        │             │
   Instance masks  Stuff masks
        │             │
        └──────┬──────┘
               │
        ┌──────▼──────┐
        │   Merge     │  Resolve overlaps:
        │   Logic     │  Things > Stuff
        └──────┬──────┘  (things override stuff at overlap pixels)
               │
        Panoptic Output
```

### Mask2Former (2022, State-of-the-Art)

```
  Key: Treats ALL segmentation as mask classification
  (No separate branches for stuff vs things)

  Image → Backbone → Pixel Decoder → Multi-scale features
                                          │
                                    ┌─────▼──────┐
                                    │ Transformer │
                                    │  Decoder    │
                                    │             │
                                    │ N learnable │
                                    │  queries    │
                                    └─────┬──────┘
                                          │
                                    N mask predictions
                                    + N class predictions
                                          │
                                    ┌─────▼──────┐
                                    │  Hungarian  │
                                    │  Matching   │  Match predictions to GT
                                    └─────┬──────┘
                                          │
                                    Loss computation

  Works for semantic, instance, AND panoptic
  with the same architecture!
```

---

## Merging Strategy

```
How to combine things and stuff predictions:

  1. Sort thing instances by confidence (descending)
  2. Place each thing mask, resolving overlaps (higher conf wins)
  3. Fill remaining pixels with stuff predictions
  4. Remove small disconnected regions

  Pixel priority:
    Thing (high conf) > Thing (low conf) > Stuff > Void

  ┌────────────────────┐
  │  sky  sky  sky     │  Step 1: Place Person (conf=0.95)
  │  sky [PERSON] sky  │  Step 2: Place Car (conf=0.88)
  │  road road road    │  Step 3: Fill remaining with stuff
  │  road [CAR]  road  │  Step 4: No overlap conflicts here
  └────────────────────┘
```

---

## Python Implementation

```python
import torch
from detectron2 import model_zoo
from detectron2.engine import DefaultPredictor
from detectron2.config import get_cfg

# Setup Panoptic FPN
cfg = get_cfg()
cfg.merge_from_file(model_zoo.get_config_file(
    "COCO-PanopticSegmentation/panoptic_fpn_R_101_3x.yaml"))
cfg.MODEL.WEIGHTS = model_zoo.get_checkpoint_url(
    "COCO-PanopticSegmentation/panoptic_fpn_R_101_3x.yaml")

predictor = DefaultPredictor(cfg)

# Run inference
import cv2
img = cv2.imread("street.jpg")
panoptic_seg, segments_info = predictor(img)["panoptic_seg"]

# Each pixel in panoptic_seg has a segment ID
# segments_info maps ID → {category_id, isthing, score, area}
for seg in segments_info:
    cat = seg["category_id"]
    is_thing = seg["isthing"]
    area = seg["area"]
    print(f"Category: {cat}, Thing: {is_thing}, Area: {area} px")
```

---

## Comparison of Segmentation Tasks

| Aspect | Semantic | Instance | Panoptic |
|--------|:---:|:---:|:---:|
| Pixel-level labels | ✓ | ✓ | ✓ |
| Instance separation | ✗ | ✓ (things only) | ✓ (things only) |
| Stuff classes | ✓ | ✗ | ✓ |
| Every pixel labeled | ✓ | ✗ | ✓ |
| Output format | Class map | Masks + boxes | Class map + instance IDs |
| Metric | mIoU | AP | PQ |

---

## Revision Questions

1. **What is the difference between "stuff" and "things" in panoptic segmentation?**
2. **How does Panoptic Quality (PQ) decompose into SQ and RQ?**
3. **How does Panoptic FPN merge instance and semantic predictions?**
4. **Why is Mask2Former considered a unified architecture?**
5. **When would you choose panoptic over semantic or instance segmentation?**

---

[Previous: 02-instance-segmentation.md](02-instance-segmentation.md) | [Next: 04-segmentation-architectures.md](04-segmentation-architectures.md)
