# Single Object Tracking (SOT)

## Overview

**Single Object Tracking (SOT)** follows one target object across video frames. Given only the bounding box of the target in the first frame, the tracker must locate that same object in every subsequent frame — even through occlusion, scale changes, deformation, and appearance changes.

---

## Problem Formulation

```
Frame 1 (initialization):     Frame t (tracking):
┌──────────────────┐          ┌──────────────────┐
│                  │          │                  │
│   ┌────┐        │          │        ┌────┐    │
│   │ 🚗 │        │ ──────▶  │        │ 🚗 │    │  Track target
│   └────┘        │          │        └────┘    │  across frames
│                  │          │                  │
└──────────────────┘          └──────────────────┘
  GT box provided              Predict box here

Key constraints:
  - Only ONE target (defined in frame 1)
  - No class label needed
  - Must handle: scale change, rotation, occlusion,
    fast motion, appearance change, background clutter
  - Real-time: 30+ FPS for practical use
```

---

## Classical Approaches

### Correlation Filter Trackers

```
Key idea: Learn a filter that gives maximum response at target location

  Training: Target patch → FFT → learn filter W in frequency domain
  Tracking: Search region → FFT → correlate with W → response map

  ┌────────────────┐     ┌────────────────┐
  │  Search region │     │  Response map  │
  │                │     │                │
  │    ┌────┐      │  →  │       ★        │  ★ = peak = target location
  │    │tgt │      │     │                │
  │    └────┘      │     │                │
  └────────────────┘     └────────────────┘

  MOSSE (2010):    Basic correlation filter, 600+ FPS
  KCF (2015):      Kernelized CF, HOG features, 300 FPS
  DCF (2014):      Discriminative CF with regularization
  ECO (2017):      Efficient convolution operators, top accuracy

  Advantages: Extremely fast (FFT), simple
  Limitations: Fixed model, no deep features, drift over time
```

### Mean Shift / CamShift

```
Track based on color histogram matching:

  1. Compute color histogram of target region
  2. Back-project histogram onto new frame → probability map
  3. Find mode (peak) of probability map using mean shift iterations

  Iteration:
    ┌──────────────┐      ┌──────────────┐
    │   ┌───┐      │      │      ┌───┐   │
    │   │ ★ │ → →  │  →   │      │ ★ │   │  ★ converges to target
    │   └───┘      │      │      └───┘   │
    └──────────────┘      └──────────────┘

  CamShift: Adaptive window size (handles scale change)
  Simple but fragile — fails with similar colors in background
```

---

## Deep Learning Trackers

### SiamFC (Siamese Fully Convolutional, 2016)

```
Idea: Learn similarity between target template and search region

  Template z (127×127)     Search x (255×255)
       │                        │
  ┌────▼────┐              ┌────▼────┐
  │ Shared  │              │ Shared  │    Same CNN
  │  CNN    │              │  CNN    │    (weight-shared)
  └────┬────┘              └────┬────┘
       │                        │
  φ(z): 6×6×128          φ(x): 22×22×128
       │                        │
       └──────── ★ ────────────┘
               Cross-
            correlation
                │
        Response map: 17×17
        Peak = target location

  Training: Learn φ on image pairs from video datasets
  Speed: 86 FPS (GPU)
  No online update (template fixed from frame 1)
```

### SiamRPN (2018)

```
Adds Region Proposal Network to Siamese tracker:

  SiamFC only predicts location → no scale/aspect ratio change
  SiamRPN predicts box refinement too:

  Template → CNN → ┐
                   ├→ Cross-corr → Classification (fg/bg)
  Search  → CNN → ┘               + Regression (Δx,Δy,Δw,Δh)
  
  → Better bounding box estimation
  → Handles scale changes
```

### TransT / MixFormer (Transformer-based)

```
Modern trackers use transformers for feature interaction:

  Template tokens    Search tokens
       │                  │
  ┌────▼──────────────────▼────┐
  │   Transformer Encoder       │
  │   (cross-attention between  │
  │    template and search)     │
  └─────────────┬──────────────┘
                │
          ┌─────▼─────┐
          │  Predict   │
          │  box + score│
          └────────────┘

  MixFormer (2022): Mixed attention, 70+ FPS
  OSTrack (2022):   One-stream, joint feature extraction
  SeqTrack (2023):  Autoregressive coordinate prediction
```

---

## VOT Benchmark & Evaluation

```
Metrics for SOT:

  Success (AUC):
    Plot % of frames where IoU(pred, GT) > threshold
    Vary threshold from 0 to 1
    AUC = area under this curve

  Precision:
    % of frames where center distance < 20 pixels
    (less informative than Success)

  Normalized Precision:
    Precision normalized by GT box size
    Handles different object scales

  Speed: FPS (real-time = 30+ FPS)

  Popular benchmarks:
    OTB-100:  100 sequences, classic benchmark
    VOT:      Annual competition, restart-based evaluation
    LaSOT:    1400 long sequences (avg 2500 frames)
    GOT-10k:  10K training + 180 test sequences
    TrackingNet: 30K sequences from YouTube
```

---

## Python: OpenCV Tracking

```python
import cv2

# Initialize tracker
tracker = cv2.TrackerKCF_create()
# Other options: CSRT, MOSSE, MIL, MedianFlow

# Read video
cap = cv2.VideoCapture("video.mp4")
ret, frame = cap.read()

# Select target ROI (or provide programmatically)
bbox = cv2.selectROI("Select Target", frame)
tracker.init(frame, bbox)

# Track through video
while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    success, bbox = tracker.update(frame)
    
    if success:
        x, y, w, h = [int(v) for v in bbox]
        cv2.rectangle(frame, (x, y), (x+w, y+h), (0, 255, 0), 2)
        cv2.putText(frame, "Tracking", (x, y-10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
    else:
        cv2.putText(frame, "Lost", (50, 50),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
    
    cv2.imshow("Tracker", frame)
    if cv2.waitKey(30) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
```

---

## Challenges in SOT

| Challenge | Description | Solution Approach |
|-----------|-------------|-------------------|
| Occlusion | Target hidden behind objects | Re-detection, memory |
| Scale change | Target grows/shrinks | Multi-scale search, RPN |
| Fast motion | Large displacement between frames | Larger search region |
| Deformation | Non-rigid shape changes | Robust features |
| Background clutter | Similar objects nearby | Discriminative learning |
| Illumination change | Lighting varies | Invariant features |

---

## Revision Questions

1. **How do correlation filter trackers work and why are they fast?**
2. **How does SiamFC use Siamese networks for tracking?**
3. **What improvement does SiamRPN add over SiamFC?**
4. **What is the Success (AUC) metric for tracking evaluation?**
5. **What are the main challenges in single object tracking?**

---

[Next: 02-multi-object-tracking.md](02-multi-object-tracking.md)
