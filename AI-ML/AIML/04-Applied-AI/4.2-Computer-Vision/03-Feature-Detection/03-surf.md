# SURF (Speeded-Up Robust Features)

## Overview

SURF is a faster alternative to SIFT that achieves comparable matching performance. It uses box filters and integral images to approximate the Laplacian of Gaussian, making keypoint detection significantly faster while maintaining scale and rotation invariance.

---

## Key Innovations

```
SIFT bottlenecks:                SURF solutions:
  DoG pyramid (slow)        →     Box filter + integral image (fast)
  Gradient histograms       →     Haar wavelet responses
  128-dim descriptor        →     64-dim descriptor (or 128 extended)

Speed comparison:
  SIFT:  ~1 second for 1000 keypoints
  SURF:  ~0.3 seconds for 1000 keypoints
  → ~3× faster than SIFT
```

---

## Integral Images

```
Integral image allows computing sum of ANY rectangle in O(1):

  Original:           Integral image:
  ┌───┬───┬───┐       ┌────┬────┬────┐
  │ 1 │ 2 │ 3 │       │  1 │  3 │  6 │
  ├───┼───┼───┤       ├────┼────┼────┤
  │ 4 │ 5 │ 6 │       │  5 │ 12 │ 21 │
  ├───┼───┼───┤       ├────┼────┼────┤
  │ 7 │ 8 │ 9 │       │ 12 │ 27 │ 45 │
  └───┴───┴───┘       └────┴────┴────┘

  II(x,y) = Σ_{i≤x, j≤y} I(i,j)

  Sum of rectangle (r1,c1) to (r2,c2):
    S = II(r2,c2) - II(r1-1,c2) - II(r2,c1-1) + II(r1-1,c1-1)

  Any box filter = just 4 lookups regardless of box size!
```

---

## SURF Detector

```
Uses Hessian matrix approximated with box filters:

  H(x, σ) = [Dxx  Dxy]
             [Dxy  Dyy]

  Dxx, Dyy, Dxy = second-order derivatives approximated by box filters

  Box filter approximations:
  Dxx (9×9):         Dxy (9×9):
  ┌───┬───┬───┐      ┌───┬───┬───┐
  │ + │ - │ + │      │ + │   │ - │
  │ + │ - │ + │      │   │   │   │
  │ + │ - │ + │      │ - │   │ + │
  └───┴───┴───┘      └───┴───┴───┘

  Response: det(H) = Dxx·Dyy - (0.9·Dxy)²

  Scale-space: increase box filter size (9, 15, 21, 27...)
  instead of downsampling the image (faster!)
```

---

## SURF Descriptor

```
64-dimensional descriptor:

  1. Find dominant orientation:
     Compute Haar wavelet responses in x and y within 6σ radius
     Rotate sliding window (60°) to find dominant direction

  2. Build descriptor:
     20σ × 20σ region around keypoint
     Split into 4×4 = 16 sub-regions
     Each sub-region: [Σdx, Σdy, Σ|dx|, Σ|dy|] = 4 values
     Total: 16 × 4 = 64 dimensions

  Extended SURF (SURF-128):
     Split sums by sign of dy → 128 dimensions
     Better distinctiveness, slightly slower matching
```

---

## Implementation

```python
import cv2

img = cv2.imread("scene.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Create SURF detector (requires opencv-contrib)
surf = cv2.xfeatures2d.SURF_create(hessianThreshold=400)

# Detect and compute
keypoints, descriptors = surf.detectAndCompute(gray, None)

print(f"Keypoints: {len(keypoints)}")
print(f"Descriptor shape: {descriptors.shape}")  # (N, 64) or (N, 128)

# Draw
img_kp = cv2.drawKeypoints(img, keypoints, None,
                            flags=cv2.DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS)
cv2.imwrite("surf_keypoints.jpg", img_kp)

# U-SURF (upright SURF — no rotation invariance, faster)
surf_upright = cv2.xfeatures2d.SURF_create(400)
surf_upright.setUpright(True)  # skip orientation → faster
```

---

## SIFT vs SURF

| Feature | SIFT | SURF |
|---------|:---:|:---:|
| Descriptor dims | 128 | 64 (or 128) |
| Scale detection | DoG pyramid | Box filters + integral image |
| Speed | Slow | ~3× faster |
| Matching quality | Excellent | Very good |
| Rotation invariant | ✓ | ✓ |
| Scale invariant | ✓ | ✓ |
| Patent | Expired (free) | Patented (non-free) |

---

## Revision Questions

1. **How do integral images speed up box filter computation?**
2. **What is the Hessian matrix and how does SURF use it?**
3. **How does SURF build its scale-space differently from SIFT?**
4. **What are the 4 values computed per sub-region in the SURF descriptor?**
5. **When would you choose SURF over SIFT?**

---

[Previous: 02-sift.md](02-sift.md) | [Next: 04-orb.md](04-orb.md)
