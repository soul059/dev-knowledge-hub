# Optical Flow

## Overview

Optical flow estimates the motion of pixels between consecutive video frames. Each pixel gets a motion vector (dx, dy) indicating where it moved. It's used for video stabilization, action recognition, autonomous driving, and motion segmentation.

---

## Concept

```
Frame t:              Frame t+1:            Optical flow:
┌───────────┐         ┌───────────┐         ┌───────────┐
│           │         │           │         │           │
│    ●      │         │      ●    │         │    →→     │
│           │    →    │           │    =    │           │
│       ●   │         │         ●│         │       →→  │
│           │         │           │         │           │
└───────────┘         └───────────┘         └───────────┘
Object positions      Object moved right    Motion vectors

Each pixel (x, y) gets flow vector (u, v):
  u = horizontal displacement
  v = vertical displacement
```

---

## Lucas-Kanade (Sparse Optical Flow)

```
Assumptions:
  1. Brightness constancy: I(x,y,t) = I(x+u, y+v, t+1)
  2. Small motion: Taylor expand → Ix·u + Iy·v + It = 0
  3. Spatial coherence: all pixels in a window have same flow

  For a window of N pixels:
    [Ix₁ Iy₁] [u]   [-It₁]
    [Ix₂ Iy₂] [v] = [-It₂]    → Overdetermined system
    [... ...]         [...]
    [IxN IyN]         [-ItN]

  Solve via least squares:
    [u]   [ΣIx²   ΣIxIy]⁻¹  [-ΣIxIt]
    [v] = [ΣIxIy  ΣIy² ]     [-ΣIyIt]

  Works well for sparse feature tracking (corners)
```

```python
import cv2
import numpy as np

cap = cv2.VideoCapture("video.mp4")
ret, old_frame = cap.read()
old_gray = cv2.cvtColor(old_frame, cv2.COLOR_BGR2GRAY)

# Detect corners to track
p0 = cv2.goodFeaturesToTrack(old_gray, maxCorners=100, 
                               qualityLevel=0.3, minDistance=7)

# Lucas-Kanade parameters
lk_params = dict(winSize=(15, 15), maxLevel=2,
                  criteria=(cv2.TERM_CRITERIA_EPS | cv2.TERM_CRITERIA_COUNT, 10, 0.03))

while True:
    ret, frame = cap.read()
    if not ret:
        break
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
    # Calculate optical flow
    p1, status, err = cv2.calcOpticalFlowPyrLK(old_gray, gray, p0, None, **lk_params)
    
    # Select good points
    good_new = p1[status == 1]
    good_old = p0[status == 1]
    
    # Draw motion vectors
    for new, old in zip(good_new, good_old):
        a, b = new.ravel().astype(int)
        c, d = old.ravel().astype(int)
        cv2.arrowedLine(frame, (c, d), (a, b), (0, 255, 0), 2)
    
    old_gray = gray.copy()
    p0 = good_new.reshape(-1, 1, 2)

cap.release()
```

---

## Farneback (Dense Optical Flow)

```python
import cv2
import numpy as np

cap = cv2.VideoCapture("video.mp4")
ret, frame1 = cap.read()
prev = cv2.cvtColor(frame1, cv2.COLOR_BGR2GRAY)

while True:
    ret, frame2 = cap.read()
    if not ret:
        break
    next_gray = cv2.cvtColor(frame2, cv2.COLOR_BGR2GRAY)
    
    # Dense optical flow (every pixel gets a flow vector)
    flow = cv2.calcOpticalFlowFarneback(
        prev, next_gray, None,
        pyr_scale=0.5, levels=3, winsize=15,
        iterations=3, poly_n=5, poly_sigma=1.2, flags=0
    )
    
    # Visualize as HSV
    mag, ang = cv2.cartToPolar(flow[..., 0], flow[..., 1])
    hsv = np.zeros_like(frame2)
    hsv[..., 0] = ang * 180 / np.pi / 2   # hue = direction
    hsv[..., 1] = 255                       # saturation = max
    hsv[..., 2] = cv2.normalize(mag, None, 0, 255, cv2.NORM_MINMAX)  # value = magnitude
    flow_rgb = cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR)
    
    prev = next_gray

cap.release()
```

---

## Sparse vs Dense

| Feature | Lucas-Kanade (Sparse) | Farneback (Dense) |
|---------|:---:|:---:|
| Coverage | Selected points only | Every pixel |
| Speed | Fast | Slower |
| Accuracy | High at tracked points | Moderate everywhere |
| Use case | Feature tracking | Motion segmentation |
| Handles large motion | With pyramids | With pyramids |

---

## Revision Questions

1. **What is optical flow and what does each flow vector represent?**
2. **What are the three assumptions of Lucas-Kanade optical flow?**
3. **What is the difference between sparse and dense optical flow?**
4. **How is optical flow visualized using HSV color encoding?**
5. **Name three applications of optical flow in CV.**

---

[Previous: 03-haar-cascades.md](03-haar-cascades.md) | [Next: 05-image-segmentation.md](05-image-segmentation.md)
