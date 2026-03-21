# SIFT (Scale-Invariant Feature Transform)

## Overview

SIFT detects and describes local features that are invariant to scale, rotation, and partially invariant to illumination and viewpoint changes. Introduced by David Lowe (2004), it was the gold standard for feature detection for over a decade and remains foundational to understanding modern feature descriptors.

---

## SIFT Pipeline

```
Image → [Scale-Space Extrema] → [Keypoint Localization] → 
        [Orientation Assignment] → [Descriptor Generation]

Step 1: Scale-space construction (DoG)
Step 2: Keypoint detection (extrema in DoG)
Step 3: Keypoint refinement (sub-pixel accuracy, reject low-contrast)
Step 4: Orientation assignment (dominant gradient direction)
Step 5: 128-dim descriptor (gradient histograms)
```

---

## Step 1: Scale-Space (Difference of Gaussians)

```
Build Gaussian pyramid at multiple scales:

  σ₁    σ₂    σ₃    σ₄    σ₅
  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐   Octave 1
  │G₁│  │G₂│  │G₃│  │G₄│  │G₅│   (original resolution)
  └──┘  └──┘  └──┘  └──┘  └──┘

  Difference of Gaussians (DoG):
  DoG₁ = G₂ - G₁
  DoG₂ = G₃ - G₂
  DoG₃ = G₄ - G₃
  DoG₄ = G₅ - G₄

  Then downsample and repeat for next octave

  DoG approximates the Laplacian of Gaussian (LoG)
  → detects blob-like structures at different scales
```

---

## Step 2-3: Keypoint Detection & Refinement

```
Find extrema in DoG scale-space:

  For each point, compare with 26 neighbors:
    8 in same scale + 9 above + 9 below

  ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
  │   │   │   │  │   │   │   │  │   │   │   │
  │   │   │   │  │   │ X │   │  │   │   │   │
  │   │   │   │  │   │   │   │  │   │   │   │
  └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
   Scale σ-1       Scale σ        Scale σ+1

  X is keypoint if it's max or min among all 26 neighbors

  Refinement:
    - Sub-pixel localization via Taylor expansion
    - Reject low-contrast points (|D(x)| < 0.03)
    - Reject edge responses (ratio of principal curvatures)
```

---

## Step 4-5: Orientation & Descriptor

```
Orientation assignment:
  For each keypoint:
    1. Take 16×16 window around keypoint
    2. Compute gradient magnitude and direction
    3. Build 36-bin histogram of gradient directions
    4. Peak = dominant orientation
    → Descriptor is computed relative to this → rotation invariance

Descriptor (128-dimensional):
  ┌────┬────┬────┬────┐
  │ 4×4│ 4×4│ 4×4│ 4×4│
  ├────┼────┼────┼────┤     16×16 window around keypoint
  │ 4×4│ 4×4│ 4×4│ 4×4│     Split into 4×4 = 16 sub-regions
  ├────┼────┼────┼────┤     Each sub-region: 8-bin gradient histogram
  │ 4×4│ 4×4│ 4×4│ 4×4│     Total: 16 × 8 = 128 dimensions
  ├────┼────┼────┼────┤
  │ 4×4│ 4×4│ 4×4│ 4×4│
  └────┴────┴────┴────┘
```

---

## Implementation

```python
import cv2

img = cv2.imread("building.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Create SIFT detector
sift = cv2.SIFT_create(nfeatures=500)

# Detect keypoints and compute descriptors
keypoints, descriptors = sift.detectAndCompute(gray, None)

print(f"Keypoints: {len(keypoints)}")
print(f"Descriptor shape: {descriptors.shape}")  # (N, 128)

# Draw keypoints
img_kp = cv2.drawKeypoints(img, keypoints, None,
                            flags=cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS)
cv2.imwrite("sift_keypoints.jpg", img_kp)

# Each keypoint has:
kp = keypoints[0]
print(f"Position: {kp.pt}")       # (x, y)
print(f"Scale:    {kp.size}")     # diameter
print(f"Angle:    {kp.angle}")    # orientation (degrees)
print(f"Response: {kp.response}") # strength
```

---

## SIFT Properties

| Property | Value |
|----------|-------|
| Descriptor size | 128 floats |
| Scale invariant | ✓ (via DoG pyramid) |
| Rotation invariant | ✓ (via orientation assignment) |
| Illumination invariant | Partial (gradient-based) |
| Speed | Slow (compared to ORB) |
| Patent | Expired 2020 (now free) |

---

## Revision Questions

1. **What is the Difference of Gaussians (DoG) and what does it approximate?**
2. **How are keypoints detected in scale-space?**
3. **How does SIFT achieve rotation invariance?**
4. **Describe how the 128-dimensional SIFT descriptor is constructed.**
5. **Why was SIFT considered the gold standard for feature detection?**

---

[Previous: 01-corner-detection.md](01-corner-detection.md) | [Next: 03-surf.md](03-surf.md)
