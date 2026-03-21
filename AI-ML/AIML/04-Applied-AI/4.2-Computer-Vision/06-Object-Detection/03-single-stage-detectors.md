# Single-Stage Detectors (YOLO, SSD)

## Overview

Single-stage detectors predict bounding boxes and class labels directly from the image in one pass — no separate region proposal step. This makes them significantly faster than two-stage detectors, enabling real-time detection. YOLO and SSD are the two foundational single-stage architectures.

---

## YOLO (You Only Look Once)

```
Key idea: Divide image into S×S grid, each cell predicts B boxes + classes

  Image → [CNN] → S×S×(B×5 + C) prediction tensor

  For each grid cell:
    - B bounding boxes: (x, y, w, h, confidence) × B
    - C class probabilities

  YOLOv1 (2016): S=7, B=2, C=20 → 7×7×30 output

  ┌───┬───┬───┬───┬───┬───┬───┐
  │   │   │   │   │   │   │   │
  ├───┼───┼───┼───┼───┼───┼───┤
  │   │   │ ★ │   │   │   │   │   ★ = cell responsible for object
  ├───┼───┼───┼───┼───┼───┼───┤       (center of object falls here)
  │   │   │   │   │   │   │   │
  ├───┼───┼───┼───┼───┼───┼───┤   That cell predicts:
  │   │   │   │   │   │   │   │   - 2 bounding boxes
  ├───┼───┼───┼───┼───┼───┼───┤   - class probabilities
  │   │   │   │   │   │   │   │
  └───┴───┴───┴───┴───┴───┴───┘

  Speed: 45 FPS (YOLOv1) → real-time!
```

---

## YOLO Evolution

```
Version  Year  Innovation                              mAP(COCO)  FPS
YOLOv1   2016  Grid-based detection                    63.4*      45
YOLOv2   2017  Anchor boxes, batch norm, multi-scale   48.1       67
YOLOv3   2018  FPN, 3 scale detection, Darknet-53      33.0       35
YOLOv4   2020  CSPDarknet, PANet, Mosaic augmentation  43.5       62
YOLOv5   2020  PyTorch, auto-anchor, smaller models    50.7       140
YOLOv7   2022  E-ELAN, re-parameterization             56.8       120
YOLOv8   2023  Anchor-free, decoupled head             53.9       280
YOLOv11  2024  Attention, lighter architecture          54.7       300+

* VOC mAP for v1, COCO mAP@0.5:0.95 for others
```

---

## SSD (Single Shot MultiBox Detector)

```
Key innovations:
  - Multi-scale feature maps (detect objects at different sizes)
  - Default boxes (anchors) at each feature map location

  ┌────────────────┐
  │    VGG-16      │
  │    backbone    │
  └───────┬────────┘
          │
  ┌───────┴────────┐
  │  38×38 feature  │ → detect small objects (4 anchors/cell)
  ├────────────────┤
  │  19×19 feature  │ → detect medium objects (6 anchors/cell)
  ├────────────────┤
  │  10×10 feature  │ → detect medium-large objects
  ├────────────────┤
  │   5×5 feature   │ → detect large objects
  ├────────────────┤
  │   3×3 feature   │ → detect larger objects
  ├────────────────┤
  │   1×1 feature   │ → detect largest objects
  └────────────────┘

  Each feature map cell → predict (class + box) for each anchor
  Total: ~8,732 predictions → NMS filters to final detections
```

---

## Using YOLOv8 (Ultralytics)

```python
from ultralytics import YOLO

# Load pre-trained model
model = YOLO("yolov8n.pt")  # nano (fastest), s, m, l, x available

# Inference
results = model("street.jpg")

# Process results
for result in results:
    boxes = result.boxes
    for box in boxes:
        cls = int(box.cls[0])
        conf = float(box.conf[0])
        xyxy = box.xyxy[0].tolist()
        print(f"Class: {model.names[cls]}, Conf: {conf:.2f}, Box: {xyxy}")

# Save annotated image
results[0].save("detected.jpg")

# Fine-tune on custom dataset
model.train(data="custom.yaml", epochs=50, imgsz=640)

# Export for deployment
model.export(format="onnx")
```

---

## Single-Stage vs Two-Stage

| Feature | Two-Stage (Faster R-CNN) | Single-Stage (YOLO) |
|---------|:---:|:---:|
| Speed | ~5 FPS | 30-300+ FPS |
| Accuracy | Higher (historically) | Comparable (modern YOLO) |
| Small objects | Better | Improving |
| Architecture | Complex (RPN + head) | Simpler |
| Training | More complex | Easier |
| Best for | Accuracy-critical | Real-time applications |

---

## Revision Questions

1. **How does YOLO's grid-based approach work?**
2. **What makes single-stage detectors faster than two-stage?**
3. **How does SSD use multi-scale feature maps for detection?**
4. **What are the key improvements from YOLOv1 to YOLOv8?**
5. **When would you still prefer Faster R-CNN over YOLO?**

---

[Previous: 02-two-stage-detectors.md](02-two-stage-detectors.md) | [Next: 04-anchor-methods.md](04-anchor-methods.md)
