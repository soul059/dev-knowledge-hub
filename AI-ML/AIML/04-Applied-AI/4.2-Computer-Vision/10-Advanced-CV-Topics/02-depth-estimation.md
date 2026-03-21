# Depth Estimation

## Overview

**Depth estimation** predicts the distance from camera to each pixel in the scene. Unlike stereo vision (which needs two cameras), **monocular depth estimation** predicts depth from a single image using learned cues like perspective, texture gradients, object sizes, and occlusion patterns.

---

## Depth Cues

```
How humans (and models) infer depth from a single image:

  Monocular cues:
  
  1. Perspective convergence    2. Relative size
     ─────────╲                   ┌──────┐  ┌────┐
     ──────────╲                  │ NEAR │  │FAR │
     ───────────╲ vanishing       │      │  └────┘
     ──────────╱  point           └──────┘
     ─────────╱                   Larger = closer
     
  3. Texture gradient           4. Occlusion
     ███  ██  █  ▪ ▪             ┌────┐
     Dense texture = close       │ A  │┌────┐
     Sparse texture = far        │    ││ B  │  A is in front of B
                                 └────┘│    │
                                       └────┘
  5. Atmospheric perspective    6. Height in image
     Near: sharp, saturated      ┌──────────────┐
     Far: hazy, desaturated      │    sky       │ Far (top)
                                 │   buildings  │
                                 │ road  🚗    │ Near (bottom)
                                 └──────────────┘
```

---

## Monocular Depth Estimation

```
Supervised approach:

  Single image → CNN → Dense depth map

  ┌──────────────┐     ┌──────────────┐
  │  RGB image   │ →   │  Depth map   │
  │  🚗    🌳   │     │  ████ ░░░░  │   Dark = near
  │              │     │  ██░░ ░░░░  │   Light = far
  └──────────────┘     └──────────────┘

  Architecture: Encoder-Decoder
  
  Input (H×W×3) → Encoder (ResNet/ViT) → Features
                                           │
                                    Decoder (upsample)
                                           │
                                    Depth map (H×W×1)

  Loss functions:
    L1 loss:  L = |d_pred - d_gt|
    
    Scale-invariant loss (Eigen et al., 2014):
      L = (1/n)Σ(log d_pred - log d_gt)² - (λ/n²)(Σ(log d_pred - log d_gt))²
      → Handles depth ambiguity (scale)
    
    Gradient loss:
      L_grad = |∇d_pred - ∇d_gt|
      → Preserves sharp depth boundaries
```

---

## Key Models

### MiDaS (Intel, 2020)

```
"Towards Robust Monocular Depth Estimation"

  Key innovation: Mix multiple datasets for training
  
  Problem: Each depth dataset has different scale/shift
    NYU Depth: indoor, meters, absolute depth
    KITTI: outdoor, LiDAR, sparse depth
    Movies: relative depth from stereo
    
  Solution: Scale-and-shift-invariant loss
    Align predicted and GT depth via least squares
    Then compute loss on aligned depth
    
  Result: Generalizes to ANY image (zero-shot)
  
  MiDaS v3.1: Uses DPT (Dense Prediction Transformer)
    ViT encoder → reassemble → fuse → depth
```

### Depth Anything (2024)

```
Massive-scale pre-training for depth estimation

  1. Pre-train encoder on 62M unlabeled images (DINOv2)
  2. Train depth decoder on 1.5M labeled images
  3. Self-training: pseudo-label remaining unlabeled images
  
  Backbone: ViT-L/14 (DINOv2)
  
  Results: Best zero-shot depth estimation
  Variants: V2 adds metric depth (absolute scale)
  
  Very fast: real-time on GPU
```

---

## Self-Supervised Depth

```
Train without ground-truth depth!

  Key insight: Depth + ego-motion → can reconstruct adjacent frames
  
  Frame t-1    Frame t    Frame t+1
  ┌──────┐    ┌──────┐    ┌──────┐
  │      │    │      │    │      │
  │  ←   │    │      │    │  →   │
  └──────┘    └──────┘    └──────┘
       ↕           ↕           ↕
  ┌────▼───┐  ┌────▼───┐  ┌────▼───┐
  │ Depth  │  │ Depth  │  │ Depth  │
  │  Net   │  │  Net   │  │  Net   │
  └────┬───┘  └────┬───┘  └────┬───┘
       │           │           │
       └─────┬─────┘─────┬─────┘
             │           │
        ┌────▼────┐ ┌────▼────┐
        │ Pose    │ │ Pose    │
        │  Net    │ │  Net    │
        └────┬────┘ └────┬────┘
             │           │
        Reconstruct frame t from t-1 and t+1
        Loss = photometric error
  
  Monodepth2 (2019): Minimum reprojection loss + auto-masking
  
  Advantages: No expensive depth annotation needed!
  Disadvantage: Only relative (not metric) depth
```

---

## Metric vs Relative Depth

```
Relative depth:
  "Object A is closer than B"
  No absolute distances
  Output: arbitrary scale
  Use: photo effects, image editing

Metric depth:
  "Object A is 3.2 meters away"
  Absolute distances in meters
  Output: real-world units
  Use: robotics, autonomous driving, AR

  Converting relative → metric:
    Need at least one known distance (reference)
    Or camera intrinsics + training on metric datasets
    
  ZoeDepth (2023): Metric depth from single image
    MiDaS backbone + metric bins module
    Predicts absolute depth in meters
```

---

## Python: Depth Estimation

```python
# MiDaS depth estimation
import torch
import cv2
import numpy as np

# Load MiDaS model
model_type = "DPT_Large"  # or "MiDaS_small" for speed
midas = torch.hub.load("intel-isl/MiDaS", model_type)
midas.eval()

# Load transforms
midas_transforms = torch.hub.load("intel-isl/MiDaS", "transforms")
transform = midas_transforms.dpt_transform

# Predict depth
img = cv2.imread("scene.jpg")
img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
input_batch = transform(img_rgb)

with torch.no_grad():
    prediction = midas(input_batch)
    depth = prediction.squeeze().numpy()

# Normalize for visualization
depth_normalized = cv2.normalize(depth, None, 0, 255,
                                  cv2.NORM_MINMAX, cv2.CV_8U)
depth_colormap = cv2.applyColorMap(depth_normalized,
                                    cv2.COLORMAP_MAGMA)
cv2.imshow("Depth", depth_colormap)


# Depth Anything v2
from transformers import pipeline

pipe = pipeline("depth-estimation",
                model="depth-anything/Depth-Anything-V2-Large")
result = pipe("scene.jpg")
depth_map = np.array(result["depth"])
```

---

## Applications

| Application | Depth Type | Use Case |
|-------------|-----------|----------|
| Autonomous driving | Metric | Obstacle distance |
| AR placement | Metric | Place virtual objects |
| Portrait mode | Relative | Background blur |
| 3D reconstruction | Metric | Scene modeling |
| Robotics | Metric | Navigation, grasping |
| Video effects | Relative | Depth-based compositing |

---

## Revision Questions

1. **What monocular cues does a CNN learn to estimate depth?**
2. **How does MiDaS achieve zero-shot generalization across domains?**
3. **How does self-supervised depth estimation work without GT depth?**
4. **What is the difference between metric and relative depth?**
5. **Why is scale-invariant loss important for depth estimation?**

---

[Previous: 01-3d-vision.md](01-3d-vision.md) | [Next: 03-pose-estimation.md](03-pose-estimation.md)
