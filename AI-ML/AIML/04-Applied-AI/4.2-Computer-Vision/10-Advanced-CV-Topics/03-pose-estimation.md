# Human Pose Estimation

## Overview

**Pose estimation** detects the spatial location of body joints (keypoints) to understand body configuration. It outputs a skeleton representation of humans — enabling action recognition, fitness tracking, sign language understanding, motion capture, and human-computer interaction.

---

## Problem Formulation

```
Input: Image with person(s)
Output: Set of keypoints (joints) with (x, y) or (x, y, z) coordinates

  Common keypoint definitions:

  COCO (17 keypoints):           MPII (16 joints):
  
       0 nose                         0 head
      / \                            / \
     1   2  eyes                    1   2  shoulders
    /     \                        /     \
   3       4  ears                3       4  elbows
    \     /                        \     /
     5   6  shoulders               5   6  wrists
    /     \                        /     \
   7       8  elbows               7       8  hips
    \     /                        \     /
     9   10  wrists                 9   10  knees
    /     \                        \     /
   11  12  hips                     11  12  ankles
    \     /                        /     \
   13  14  knees                  13  14  additional
    \     /
   15  16  ankles

  Each keypoint: (x, y, visibility)
    visibility: 0 = not labeled, 1 = labeled but occluded, 2 = visible
```

---

## Top-Down vs Bottom-Up

```
Top-Down approach:
  1. Detect all persons (object detection)
  2. Crop each person
  3. Estimate pose for each crop independently
  
  Image → [Detector] → Person boxes → [Pose Net] → Skeletons
  
  ┌──────────────┐    ┌──────┐ ┌──────┐
  │ 🧑  🧑  🧑  │ → │ 🧑  │ │ 🧑  │ → Pose per person
  └──────────────┘    └──────┘ └──────┘
  
  Pros: High accuracy (one person per crop)
  Cons: Speed = O(N × pose_time), N = number of people

Bottom-Up approach:
  1. Detect ALL keypoints in entire image at once
  2. Group keypoints into individual persons
  
  Image → [Detect all joints] → [Group into persons] → Skeletons
  
  ┌──────────────┐    ●●●●●     ┌──── Person 1
  │ 🧑  🧑  🧑  │ → ●●●●● → ├──── Person 2
  └──────────────┘    ●●●●●     └──── Person 3
  
  Pros: Speed constant regardless of # people
  Cons: Grouping is challenging, slightly lower accuracy
```

---

## Key Architectures

### SimpleBaseline (Top-Down, 2018)

```
Xiao et al. — Surprisingly simple and effective

  ResNet backbone → 3 deconvolution layers → K heatmaps
  
  Input (256×192)
       │
  ┌────▼────┐
  │ResNet-50│  Pretrained on ImageNet
  │         │  Output: 8×6×2048
  └────┬────┘
       │
  ┌────▼────┐
  │ Deconv  │  3 layers of transposed convolution
  │ 256×3   │  Upsample: 8→16→32→64
  └────┬────┘
       │
  ┌────▼────┐
  │1×1 Conv │  K channels (K = 17 for COCO)
  └────┬────┘
       │
  K heatmaps (64×48 each)
  
  Each heatmap: Gaussian at keypoint location
  Loss: MSE between predicted and GT heatmaps
```

### HRNet (Top-Down, 2019)

```
High-Resolution Network — maintains resolution throughout

  ──── High (1/4)  ████████████████████████████████
       ↓↑  ↓↑  ↓↑  ↓↑  ↓↑  ↓↑
  ──── Med (1/8)   ████████████████████████████
       ↓↑  ↓↑  ↓↑  ↓↑  ↓↑
  ──── Low (1/16)  ██████████████████████
       ↓↑  ↓↑  ↓↑  ↓↑
  ──── VLow (1/32) ████████████████

  → High-res features never lost
  → Multi-scale fusion at every stage
  → State-of-the-art for pose estimation
  → AP = 75.5 on COCO test-dev
```

### OpenPose (Bottom-Up, 2017)

```
Cao et al. — Real-time multi-person pose estimation

  Two-branch architecture:
  
  ┌──────────────────────────────┐
  │ VGG backbone (shared)        │
  │        │                     │
  │   ┌────┼────┐                │
  │   │         │                │
  │ Part     Part Affinity       │
  │Confidence  Fields (PAFs)     │
  │ Maps       │                 │
  │   │         │                │
  │ Where are  Which joints      │
  │ joints?    connect?          │
  └──────────────────────────────┘

  Part Affinity Fields (PAFs):
    2D vector field pointing along limb direction
    Used to associate joints into persons
    
    ●shoulder ──→──→──→── ●elbow
    PAF vectors point from shoulder to elbow
    
  Greedy matching: Connect joints with highest PAF scores
  
  Speed: 22 FPS for multi-person (GPU)
```

### ViTPose (2022)

```
Transformer-based pose estimation:

  Input → ViT backbone → Simple decoder → Heatmaps
  
  Key insight: Plain ViT with minimal head works great!
  No fancy decoders or complex architectures needed
  
  ViTPose-H (ViT-Huge):
    AP = 79.1 on COCO test-dev (state-of-the-art)
```

---

## 3D Pose Estimation

```
Predict (x, y, z) for each joint:

  Approach 1: Lifting 2D → 3D
    2D pose → MLP → 3D pose
    Martinez et al. (2017): simple MLP achieves good results!
    
  Approach 2: Direct 3D from image
    Image → CNN → 3D joint coordinates
    Requires 3D ground truth (MoCap data)
    
  Approach 3: Volumetric
    Predict 3D heatmaps (x, y, z voxels)
    More accurate but memory-intensive

  Datasets:
    Human3.6M: 3.6M frames, MoCap GT, 11 subjects
    MPI-INF-3DHP: Outdoor + indoor
    
  SMPL body model:
    Parametric 3D body: shape β (10 params) + pose θ (72 params)
    → Full 3D mesh from a single image
    HMR, SPIN, PARE reconstruct SMPL from images
```

---

## Evaluation Metrics

```
  AP (Average Precision) — COCO primary:
    Use OKS (Object Keypoint Similarity) instead of IoU:
    
    OKS = Σ exp(-d²_i / (2 × s² × k²_i)) × δ(v_i > 0) / Σ δ(v_i > 0)
    
    d_i = Euclidean distance between predicted and GT keypoint i
    s = object scale (√(area))
    k_i = per-keypoint constant (larger for harder joints)
    
    OKS = 1.0: perfect prediction
    OKS = 0.0: completely wrong
    
    AP computed same as detection: at OKS thresholds [0.5:0.95]

  PCK (Percentage of Correct Keypoints):
    % of joints predicted within threshold distance
    PCK@0.5: within 50% of head segment length
    
  MPJPE (Mean Per Joint Position Error) — 3D:
    Average Euclidean distance per joint (in mm)
    After Procrustes alignment: PA-MPJPE
```

---

## Python: Pose Estimation

```python
# MediaPipe Pose (real-time, single person)
import mediapipe as mp
import cv2

mp_pose = mp.solutions.pose
pose = mp_pose.Pose(min_detection_confidence=0.5)
mp_draw = mp.solutions.drawing_utils

cap = cv2.VideoCapture(0)
while cap.isOpened():
    ret, frame = cap.read()
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = pose.process(rgb)
    
    if results.pose_landmarks:
        mp_draw.draw_landmarks(
            frame, results.pose_landmarks,
            mp_pose.POSE_CONNECTIONS)
        
        # Access specific landmarks
        nose = results.pose_landmarks.landmark[mp_pose.PoseLandmark.NOSE]
        print(f"Nose: ({nose.x:.2f}, {nose.y:.2f}, {nose.z:.2f})")
    
    cv2.imshow("Pose", frame)
    if cv2.waitKey(1) & 0xFF == 27:
        break

cap.release()
```

---

## Revision Questions

1. **What is the difference between top-down and bottom-up pose estimation?**
2. **How do heatmap-based methods predict keypoint locations?**
3. **What are Part Affinity Fields in OpenPose?**
4. **Why is HRNet effective for pose estimation?**
5. **What does OKS measure and how is it used for AP computation?**

---

[Previous: 02-depth-estimation.md](02-depth-estimation.md) | [Next: 04-image-generation.md](04-image-generation.md)
