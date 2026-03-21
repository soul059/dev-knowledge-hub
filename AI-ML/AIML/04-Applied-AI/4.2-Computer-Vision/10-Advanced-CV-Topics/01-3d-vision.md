# 3D Computer Vision

## Overview

**3D Computer Vision** reconstructs and understands the three-dimensional structure of scenes from 2D images. It bridges the gap between flat images and the real 3D world — enabling robotics, AR/VR, autonomous vehicles, and digital twins.

---

## 3D Representations

```
1. Point Cloud
   Set of (x, y, z) points in 3D space
   ● ● ● ●
    ● ● ●
   ● ● ● ● ●
    ● ● ● ●
   Sparse but flexible, common from LiDAR/depth sensors
   
2. Mesh
   Vertices + edges + faces (triangles)
   ┌──────┐
   │\    /│
   │ \  / │     Compact, renderable
   │  \/  │     Standard for graphics
   │  /\  │
   │ /  \ │
   └──────┘
   
3. Voxel Grid
   3D pixels — regular 3D grid
   ┌─┬─┬─┐
   ├─┼─┼─┤      Memory-intensive: O(n³)
   ├─┼─┼─┤      Easy for 3D convolutions
   └─┴─┴─┘
   
4. Implicit (NeRF, SDF)
   Function: f(x, y, z) → occupancy/color
   Continuous, resolution-independent
   
5. Multi-view Images
   Multiple 2D views of same scene
   Camera 1 → ┐
   Camera 2 → ├→ 3D reconstruction
   Camera 3 → ┘
```

---

## Structure from Motion (SfM)

```
Reconstruct 3D scene from multiple unstructured photos:

  Photo 1        Photo 2        Photo 3
  ┌──────┐      ┌──────┐      ┌──────┐
  │ ● ●  │      │  ● ● │      │● ●   │
  │   ●  │      │ ●    │      │  ● ● │
  │  ● ● │      │  ●●  │      │ ●    │
  └──────┘      └──────┘      └──────┘
       \            │            /
        \           │           /
         →  3D Point Cloud  ←
            + Camera Poses

  Pipeline:
  1. Feature detection (SIFT/SuperPoint)
  2. Feature matching across views
  3. Estimate camera poses (fundamental/essential matrix)
  4. Triangulate 3D points
  5. Bundle adjustment (jointly optimize cameras + points)
  
  Tools: COLMAP, OpenMVG, VisualSFM
```

---

## Stereo Vision

```
Two cameras → depth from disparity:

  Left Camera              Right Camera
  ┌──────────┐            ┌──────────┐
  │    ●     │            │      ●   │   Same point appears
  │          │            │          │   at different positions
  └──────────┘            └──────────┘
  
  Disparity: d = x_L - x_R  (horizontal pixel difference)
  
  Depth: Z = f × B / d
    f = focal length
    B = baseline (distance between cameras)
    d = disparity
    
  Large disparity → close object
  Small disparity → far object
  
  Disparity map → Dense depth map:
  ┌──────────────┐      ┌──────────────┐
  │  Left image  │  →   │ Depth map    │
  │              │      │ ████ (near)  │
  │  🚗    🌳   │      │ ░░░░ (far)   │
  └──────────────┘      └──────────────┘

  Deep stereo: AANet, RAFT-Stereo, CREStereo
```

---

## NeRF (Neural Radiance Fields)

```
Mildenhall et al. (2020) — Novel view synthesis

  Input: Set of images with known camera poses
  Learn: Continuous 3D scene representation

  MLP: f(x, y, z, θ, φ) → (r, g, b, σ)
    (x,y,z) = 3D position
    (θ,φ) = viewing direction
    (r,g,b) = color
    σ = density (opacity)

  Rendering (volume rendering along ray):
    Cast ray from camera through each pixel
    Sample points along ray
    Query MLP at each point
    Accumulate color: C = Σ T_i × α_i × c_i
    
    T_i = transmittance (how much light passes through)
    α_i = 1 - exp(-σ_i × δ_i)  (opacity of segment)

  Training: Minimize MSE between rendered and real pixels
  
  Result: Photorealistic novel views from sparse photos!
  
  Variants:
    Instant-NGP (2022):  Hash encoding, trains in seconds
    3D Gaussian Splatting (2023):  Real-time rendering, explicit
    Zip-NeRF (2023):    Anti-aliased, high quality
```

---

## Point Cloud Processing

```
PointNet (Qi et al., 2017):
  First deep learning directly on point clouds

  Input: N × 3 point cloud
       │
  ┌────▼────┐
  │ Per-point│  MLP applied to each point independently
  │  MLP    │  N×3 → N×64 → N×1024
  └────┬────┘
       │
  ┌────▼────┐
  │ Max Pool│  Symmetric function → order invariance
  │ Global  │  N×1024 → 1×1024
  └────┬────┘
       │
  ┌────▼────┐
  │ MLP +   │  1024 → 512 → 256 → K classes
  │ Softmax │
  └─────────┘

  Key insight: Max pooling = symmetric → invariant to point order
  PointNet++: Adds hierarchical grouping for local features
  
  Modern: Point Transformer, PointNeXt
```

---

## Python: 3D Vision

```python
# Depth from stereo with OpenCV
import cv2
import numpy as np

left = cv2.imread("left.png", 0)
right = cv2.imread("right.png", 0)

stereo = cv2.StereoSGBM_create(
    minDisparity=0, numDisparities=64,
    blockSize=5, P1=8*3*5**2, P2=32*3*5**2
)
disparity = stereo.compute(left, right).astype(np.float32) / 16.0

# Convert disparity to depth
focal_length = 700  # pixels
baseline = 0.12     # meters
depth = focal_length * baseline / (disparity + 1e-6)


# Point cloud with Open3D
import open3d as o3d

pcd = o3d.io.read_point_cloud("scene.ply")
print(f"Points: {len(pcd.points)}")

# Visualize
o3d.visualization.draw_geometries([pcd])

# Downsample, estimate normals
pcd_down = pcd.voxel_down_sample(voxel_size=0.02)
pcd_down.estimate_normals()
```

---

## Revision Questions

1. **What are the main 3D representations and their trade-offs?**
2. **How does Structure from Motion reconstruct 3D from photos?**
3. **How does stereo vision compute depth from disparity?**
4. **What is NeRF and how does it render novel views?**
5. **How does PointNet handle the unordered nature of point clouds?**

---

[Next: 02-depth-estimation.md](02-depth-estimation.md)
