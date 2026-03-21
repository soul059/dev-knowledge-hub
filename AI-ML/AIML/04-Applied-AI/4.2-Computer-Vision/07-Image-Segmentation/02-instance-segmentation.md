# Instance Segmentation

## Overview

**Instance segmentation** combines object detection and semantic segmentation — it identifies each individual object and produces a pixel-level mask for each one. Unlike semantic segmentation which labels all "person" pixels the same, instance segmentation distinguishes Person 1, Person 2, Person 3 with separate masks.

---

## Semantic vs Instance Segmentation

```
Original Scene: Two people and a car

  Semantic Segmentation          Instance Segmentation
  ┌──────────────────┐           ┌──────────────────┐
  │                  │           │                  │
  │  PPP    PPP      │           │  111    222      │  1 = Person 1
  │  PPP    PPP      │           │  111    222      │  2 = Person 2
  │  PPP    PPP      │           │  111    222      │  3 = Car 1
  │                  │           │                  │
  │    CCCCCCCC      │           │    33333333      │
  │    CCCCCCCC      │           │    33333333      │
  └──────────────────┘           └──────────────────┘
  
  P = Person (all same)          Each instance has unique ID
  C = Car (all same)             + bounding box + class + mask
```

---

## Mask R-CNN (Primary Architecture)

```
Mask R-CNN = Faster R-CNN + Mask Branch

  Image → Backbone (ResNet-FPN) → Feature Maps
                                      │
                          ┌───────────┼───────────┐
                          │           │           │
                     ┌────▼────┐ ┌────▼────┐ ┌────▼────┐
                     │   RPN    │ │  Box    │ │  Mask   │
                     │ Region  │ │  Head   │ │  Head   │
                     │Proposals│ │ cls+reg │ │ 28×28   │
                     └────┬────┘ └────┬────┘ └────┬────┘
                          │           │           │
                     Proposals   Class+Box    Binary Mask
                                              per instance

  Key Innovation: RoIAlign (not RoIPool)
  
  RoIPool (Faster R-CNN):  Quantizes to integer coordinates → misaligned
  RoIAlign (Mask R-CNN):   Bilinear interpolation → pixel-perfect alignment
  
  ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
  │ Q │ Q │   │   │       │ ● │ ● │   │   │   Q = quantized (snapped)
  ├───┼───┼───┼───┤       ├───┼───┼───┼───┤   ● = bilinear (precise)
  │ Q │ Q │   │   │       │ ● │ ● │   │   │
  ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
  │   │   │   │   │       │   │   │   │   │   RoIAlign preserves
  └───┴───┴───┴───┘       └───┴───┴───┴───┘   sub-pixel accuracy
      RoIPool                  RoIAlign        → better masks
```

---

## Mask Head Design

```
For each detected region (after RoIAlign):

  RoI Feature (14×14×256)
       │
  ┌────▼────┐
  │ 4× Conv │  3×3 convolutions
  │ 256 ch  │
  └────┬────┘
       │
  ┌────▼────┐
  │ Deconv  │  2× upsample (transposed conv)
  └────┬────┘
       │
  ┌────▼────┐
  │ 1×1 Conv│  C channels (one per class)
  └────┬────┘
       │
  Output: 28×28×C mask predictions
  
  Each class gets its own binary mask
  Final mask = mask for the predicted class
  Loss = binary cross-entropy per pixel (class-specific)
```

---

## Training Losses

```
Mask R-CNN has 3 losses:

  L_total = L_cls + L_box + L_mask

  L_cls:  Cross-entropy for classification (from Faster R-CNN)
  L_box:  Smooth L1 for box regression (from Faster R-CNN)  
  L_mask: Binary cross-entropy per pixel (NEW)

  Key insight: Mask prediction is decoupled from classification
    → Predict K binary masks (one per class)
    → Use mask for the predicted class only
    → No competition between classes in mask branch
    → Much better results than multi-class mask
```

---

## Python: Instance Segmentation

```python
import torch
import torchvision
from torchvision.models.detection import maskrcnn_resnet50_fpn
from PIL import Image
from torchvision import transforms
import numpy as np

# Load pre-trained Mask R-CNN
model = maskrcnn_resnet50_fpn(pretrained=True)
model.eval()

# Prepare image
img = Image.open("street.jpg")
transform = transforms.ToTensor()
input_tensor = transform(img).unsqueeze(0)

# Inference
with torch.no_grad():
    predictions = model(input_tensor)[0]

# Process results
for i in range(len(predictions["scores"])):
    if predictions["scores"][i] > 0.7:
        label = predictions["labels"][i].item()
        score = predictions["scores"][i].item()
        mask = predictions["masks"][i, 0].numpy()  # H×W binary mask
        box = predictions["boxes"][i].tolist()
        
        print(f"Object: {label}, Score: {score:.2f}")
        print(f"  Box: {box}")
        print(f"  Mask pixels: {(mask > 0.5).sum()}")

# Visualize masks overlaid on image
import matplotlib.pyplot as plt
import matplotlib.patches as patches

fig, ax = plt.subplots(1, figsize=(12, 8))
ax.imshow(img)

colors = plt.cm.tab10(np.linspace(0, 1, 10))
for i in range(min(5, len(predictions["scores"]))):
    if predictions["scores"][i] > 0.7:
        mask = predictions["masks"][i, 0].numpy() > 0.5
        colored_mask = np.zeros((*mask.shape, 4))
        colored_mask[mask] = [*colors[i % 10][:3], 0.5]
        ax.imshow(colored_mask)

plt.show()
```

---

## Modern Approaches

```
Beyond Mask R-CNN:

  YOLACT (2019):   Real-time instance segmentation
                   Generates prototype masks + coefficients per instance
                   30+ FPS (vs 5 FPS for Mask R-CNN)

  PointRend (2020): Treats mask as rendering problem
                    Iteratively refines uncertain boundary pixels
                    Much sharper mask boundaries

  SOLOv2 (2020):   Single-stage, no box detection needed
                    Predicts masks directly at grid locations

  Mask2Former (2022): Transformer-based, unified architecture
                      State-of-the-art for semantic, instance, panoptic
```

---

## Revision Questions

1. **How does instance segmentation differ from semantic segmentation?**
2. **What is RoIAlign and why is it better than RoIPool for masks?**
3. **How does Mask R-CNN's mask head predict per-instance masks?**
4. **Why does Mask R-CNN predict class-specific binary masks?**
5. **What are the advantages of YOLACT over Mask R-CNN?**

---

[Previous: 01-semantic-segmentation.md](01-semantic-segmentation.md) | [Next: 03-panoptic-segmentation.md](03-panoptic-segmentation.md)
