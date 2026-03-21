# Multi-Object Tracking (MOT)

## Overview

**Multi-Object Tracking (MOT)** simultaneously tracks multiple objects across video frames. Each object must maintain a consistent identity over time. MOT is critical for autonomous driving (track all vehicles), surveillance (track all people), and sports analytics.

---

## Problem Formulation

```
Frame t:                    Frame t+1:
┌──────────────────┐        ┌──────────────────┐
│  ┌──┐    ┌──┐    │        │    ┌──┐  ┌──┐    │
│  │P1│    │P2│    │        │    │P1│  │P2│    │    Maintain IDs:
│  └──┘    └──┘    │  ────▶ │    └──┘  └──┘    │    P1 is still P1
│       ┌──┐       │        │  ┌──┐       ┌──┐ │    P2 is still P2
│       │P3│       │        │  │P3│       │P4│ │    P3 is still P3
│       └──┘       │        │  └──┘       └──┘ │    P4 is NEW
└──────────────────┘        └──────────────────┘

Challenges beyond SOT:
  - Variable number of targets (appear/disappear)
  - Association: which detection = which track?
  - Occlusion between tracked objects
  - ID switches (accidentally swapping identities)
  - Scale: track 100+ objects simultaneously
```

---

## Tracking-by-Detection Paradigm

```
The dominant MOT approach:

  Frame t                  Frame t+1
     │                        │
  ┌──▼──┐                 ┌──▼──┐
  │Detect│  (YOLO, etc.)  │Detect│
  └──┬──┘                 └──┬──┘
     │                        │
  Detections t            Detections t+1
  [d1, d2, d3]            [d4, d5, d6, d7]
     │                        │
     └────────┬───────────────┘
              │
     ┌────────▼────────┐
     │  Data Association│   Match detections across frames
     │  (Hungarian alg) │   using cost matrix
     └────────┬────────┘
              │
     Track updates:
       Track 1: d1 → d4
       Track 2: d2 → d5
       Track 3: d3 → d6
       New track: d7 (unmatched detection)

  Cost matrix for association:
              d4    d5    d6    d7
  Track 1  [ 0.2   3.5   4.1   5.2 ]   (distance/cost)
  Track 2  [ 3.8   0.3   2.9   4.7 ]
  Track 3  [ 4.5   3.1   0.4   6.0 ]

  Hungarian algorithm finds optimal assignment (min total cost)
```

---

## SORT (Simple Online and Realtime Tracking)

```
Bewley et al. (2016) — Beautifully simple baseline

Components:
  1. Detector: Any object detector (e.g., Faster R-CNN)
  2. State estimation: Kalman Filter per track
  3. Association: Hungarian algorithm with IoU cost

  Kalman Filter state: [x, y, s, r, ẋ, ẏ, ṡ]
    x, y = center position
    s = scale (area)
    r = aspect ratio (constant)
    ẋ, ẏ, ṡ = velocities

  Pipeline per frame:
  ┌─────────────────────────────────────────────┐
  │ 1. Predict: Kalman filter predicts tracks    │
  │ 2. Detect: Run detector on current frame     │
  │ 3. Associate: Match predictions to detections│
  │    - Cost = 1 - IoU(predicted, detected)     │
  │    - Hungarian algorithm                     │
  │ 4. Update:                                   │
  │    - Matched: Kalman update with detection    │
  │    - Unmatched detection: create new track    │
  │    - Unmatched track: increment age           │
  │      (delete if age > max_age)               │
  └─────────────────────────────────────────────┘

  Speed: 260 FPS (tracking only, excludes detection)
  Limitation: No appearance features → many ID switches
```

---

## DeepSORT

```
Wojke et al. (2017) — SORT + Deep Appearance Features

  Adds appearance descriptor to reduce ID switches:

  Cost = λ × motion_cost + (1-λ) × appearance_cost

  Motion cost: Mahalanobis distance from Kalman prediction
  Appearance cost: Cosine distance between deep features

  Deep feature extractor:
    ┌───────────┐
    │ Detection │ → Crop → CNN → 128-D feature vector
    │  (bbox)   │            (trained on re-ID dataset)
    └───────────┘

  Association strategy (cascade):
    1. First match: recently seen tracks (high priority)
    2. Then match: older tracks (lower priority)
    3. IoU-based fallback for remaining

  Gallery: Keep last 100 appearance vectors per track
    → Handles brief occlusions (re-identify after reappearing)

  Result: 45% fewer ID switches than SORT
```

---

## Modern MOT Methods

```
ByteTrack (2022):
  Key insight: Use LOW confidence detections too!
  
  Standard: Only match high-conf detections (> 0.5)
  ByteTrack: Two-stage matching:
    1. Match high-conf detections to tracks (IoU)
    2. Match LOW-conf detections to unmatched tracks
    → Recovers occluded/partial objects
    → State-of-the-art with simple IoU matching!

OC-SORT (2023):
  Observation-Centric SORT
  Better Kalman filter for non-linear motion
  Virtual trajectory during occlusion

BoT-SORT (2022):
  Camera motion compensation (CMC)
  + Better Kalman filter + better Re-ID
  
StrongSORT (2023):
  DeepSORT + better Re-ID (BoT features)
  + Gaussian-smoothed interpolation
  + ECC camera motion compensation
```

---

## MOT Metrics

```
CLEAR MOT Metrics:

  MOTA (Multi-Object Tracking Accuracy):
    MOTA = 1 - (FN + FP + IDSW) / GT
    
    FN = False Negatives (missed detections)
    FP = False Positives (false alarms)
    IDSW = Identity Switches
    GT = total GT objects across all frames
    
    Can be negative! (more errors than GT objects)

  MOTP (Multi-Object Tracking Precision):
    MOTP = Σ IoU(matched pairs) / #matches
    Average IoU of matched track-GT pairs

  IDF1 (ID F1 Score):
    F1 score of identity-correct detections
    IDF1 = 2 × IDTP / (2×IDTP + IDFP + IDFN)
    Better captures long-term identity consistency

  HOTA (Higher Order Tracking Accuracy):
    Geometric mean of detection and association accuracy
    HOTA = √(DetA × AssA)
    Modern replacement for MOTA

  MT (Mostly Tracked):  % of GT tracks covered > 80%
  ML (Mostly Lost):     % of GT tracks covered < 20%
  Frag:                 Number of track fragmentations
```

---

## Python: Simple MOT Pipeline

```python
from sort import Sort  # pip install sort
import numpy as np

tracker = Sort(max_age=30, min_hits=3, iou_threshold=0.3)

# Simulated detections per frame: [x1, y1, x2, y2, score]
detections_per_frame = [
    np.array([[100, 100, 200, 200, 0.95],
              [300, 150, 400, 250, 0.87]]),
    np.array([[110, 105, 210, 205, 0.93],
              [310, 155, 410, 255, 0.85],
              [500, 300, 600, 400, 0.72]]),
]

for frame_idx, dets in enumerate(detections_per_frame):
    # Returns: [x1, y1, x2, y2, track_id]
    tracks = tracker.update(dets)
    
    print(f"\nFrame {frame_idx}:")
    for track in tracks:
        x1, y1, x2, y2, track_id = track
        print(f"  Track {int(track_id)}: "
              f"[{x1:.0f}, {y1:.0f}, {x2:.0f}, {y2:.0f}]")
```

---

## Revision Questions

1. **How does tracking-by-detection work and why is it the dominant paradigm?**
2. **What role does the Kalman Filter play in SORT?**
3. **How does DeepSORT reduce identity switches compared to SORT?**
4. **What is ByteTrack's key insight about low-confidence detections?**
5. **What does MOTA measure and why can it be negative?**
6. **Why is IDF1 sometimes preferred over MOTA?**

---

[Previous: 01-single-object-tracking.md](01-single-object-tracking.md) | [Next: 03-tracking-algorithms.md](03-tracking-algorithms.md)
