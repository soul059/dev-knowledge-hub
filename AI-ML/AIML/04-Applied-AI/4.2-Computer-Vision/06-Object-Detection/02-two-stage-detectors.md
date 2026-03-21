# Two-Stage Detectors (R-CNN Family)

## Overview

Two-stage detectors first propose candidate regions that might contain objects, then classify and refine each proposal. The R-CNN family — R-CNN, Fast R-CNN, and Faster R-CNN — progressively improved speed while maintaining accuracy, culminating in Faster R-CNN which remains widely used.

---

## R-CNN (2014)

```
Pipeline:
  Image → [Selective Search] → ~2000 region proposals
                                    │
                    For each proposal:
                    ┌───────────────┴──────────────────┐
                    │  1. Crop & resize to 227×227     │
                    │  2. Forward through CNN (AlexNet) │
                    │  3. Extract features (4096-dim)   │
                    │  4. SVM classifier → class label  │
                    │  5. Bounding box regressor → refine│
                    └──────────────────────────────────┘

Problems:
  - 2000 forward passes per image → ~47 seconds/image!
  - Features not shared between proposals
  - Multi-stage training (CNN, SVM, regressor separately)
```

---

## Fast R-CNN (2015)

```
Key insight: Share the CNN computation!

  Image → [CNN backbone] → Feature map (shared!)
                              │
  Selective Search → Proposals ─┘
                              │
                    For each proposal:
                    ┌─────────┴──────────┐
                    │  RoI Pooling        │  Crop feature map region
                    │  → fixed size       │  → resize to 7×7
                    ├────────────────────┤
                    │  FC layers          │
                    ├────────┬───────────┤
                    │  Class │  BBox     │  Two heads: classification + regression
                    │  scores│  offsets  │
                    └────────┴───────────┘

Improvements over R-CNN:
  - Single CNN forward pass for entire image
  - End-to-end training (joint classification + regression)
  - ~10× faster than R-CNN (~2 seconds/image)
  
  Still uses Selective Search → bottleneck!
```

---

## Faster R-CNN (2016)

```
Replace Selective Search with a learned Region Proposal Network (RPN):

  Image → [CNN backbone] → Feature map
                              │
                    ┌─────────┴──────────┐
                    │  Region Proposal    │  Learned! (not hand-crafted)
                    │  Network (RPN)      │  Slides 3×3 window over feature map
                    │                    │  At each position: k anchor boxes
                    │  Outputs:          │  → objectness score + box refinement
                    │  - ~300 proposals   │
                    └─────────┬──────────┘
                              │
                    ┌─────────┴──────────┐
                    │  RoI Pooling        │
                    │  → FC layers        │
                    │  → Class + BBox     │
                    └────────────────────┘

  Speed: ~5 FPS (200ms/image) on GPU
  Accuracy: ~73% mAP on VOC 2007

Anchor boxes (at each feature map position):
  3 scales × 3 aspect ratios = 9 anchors
  
  ┌─────────┐  ┌───────────────┐  ┌───┐
  │         │  │               │  │   │
  │  1:1    │  │     2:1       │  │1:2│
  │         │  │               │  │   │
  └─────────┘  └───────────────┘  │   │
                                   │   │
                                   └───┘
```

---

## Implementation

```python
import torchvision
from torchvision.models.detection import fasterrcnn_resnet50_fpn_v2
from torchvision import transforms
from PIL import Image

# Load pre-trained Faster R-CNN
model = fasterrcnn_resnet50_fpn_v2(weights="DEFAULT")
model.eval()

# Prepare image
img = Image.open("street.jpg")
transform = transforms.ToTensor()
img_tensor = transform(img).unsqueeze(0)

# Detect
with torch.no_grad():
    predictions = model(img_tensor)

# Results
boxes = predictions[0]["boxes"]     # (N, 4) - x1, y1, x2, y2
labels = predictions[0]["labels"]   # (N,) - class indices
scores = predictions[0]["scores"]   # (N,) - confidence scores

# Filter by confidence
threshold = 0.5
keep = scores > threshold
for box, label, score in zip(boxes[keep], labels[keep], scores[keep]):
    print(f"Class {label.item()}: {score:.2f} at {box.tolist()}")
```

---

## R-CNN Family Comparison

| Model | Proposals | Speed | mAP (VOC) | Training |
|-------|-----------|:---:|:---:|---------|
| R-CNN | Selective Search | 47s | 58% | Multi-stage |
| Fast R-CNN | Selective Search | 2s | 66% | End-to-end (almost) |
| Faster R-CNN | RPN (learned) | 0.2s | 73% | Fully end-to-end |

---

## Revision Questions

1. **Why was R-CNN so slow and how did Fast R-CNN fix it?**
2. **What is RoI Pooling and why is it needed?**
3. **How does the Region Proposal Network (RPN) work in Faster R-CNN?**
4. **What are anchor boxes and how are they configured?**
5. **Why is Faster R-CNN still widely used despite being from 2016?**

---

[Previous: 01-problem-formulation.md](01-problem-formulation.md) | [Next: 03-single-stage-detectors.md](03-single-stage-detectors.md)
