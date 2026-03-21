# Corner Detection (Harris, Shi-Tomasi)

## Overview

Corners are points where two edges meet, creating a distinctive local pattern that is stable under transformations. Corner detection is fundamental for feature matching, image stitching, tracking, and 3D reconstruction. The two classic detectors are **Harris** and **Shi-Tomasi**.

---

## Why Corners Are Special

```
Three types of regions:

  Flat region:          Edge:               Corner:
  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
  │             │      │─────────────│      │      │      │
  │  No change  │      │  Change in  │      │──────┤      │
  │  in any     │      │  one        │      │  Change in  │
  │  direction  │      │  direction  │      │  ALL         │
  │             │      │             │      │  directions  │
  └─────────────┘      └─────────────┘      └─────────────┘
  
  Shifting a window:
    Flat:   No intensity change in any direction → NOT distinctive
    Edge:   Change along one direction only → somewhat distinctive
    Corner: Change in ALL directions → VERY distinctive ✓
```

---

## Harris Corner Detector

```
Algorithm:
  1. Compute gradients Ix, Iy at each pixel
  2. For each pixel, compute the structure tensor M in a window:
  
     M = Σ w(x,y) [Ix²    Ix·Iy]  = [A   C]
                    [Ix·Iy  Iy² ]    [C   B]
     
     w(x,y) = Gaussian window weight

  3. Compute Harris response:
     R = det(M) - k × trace(M)²
     R = (A·B - C²) - k × (A + B)²
     
     k = sensitivity parameter (typically 0.04-0.06)

  4. Classify:
     R > threshold  → Corner  ✓
     R ≈ 0          → Flat region
     R < 0          → Edge

  Eigenvalue interpretation:
     λ₁, λ₂ = eigenvalues of M
     
     Both large:  Corner (λ₁ ≈ λ₂ >> 0)
     One large:   Edge (λ₁ >> λ₂ ≈ 0)
     Both small:  Flat (λ₁ ≈ λ₂ ≈ 0)
```

---

## Harris Implementation

```python
import cv2
import numpy as np

img = cv2.imread("checkerboard.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY).astype(np.float32)

# Harris corner detection
harris = cv2.cornerHarris(gray, blockSize=2, ksize=3, k=0.04)

# Dilate for visualization
harris = cv2.dilate(harris, None)

# Mark corners on image (threshold at 1% of max response)
img[harris > 0.01 * harris.max()] = [0, 0, 255]  # Red dots

cv2.imwrite("harris_corners.jpg", img)
```

---

## Shi-Tomasi (Good Features to Track)

```
Improvement over Harris:

  Harris:     R = λ₁·λ₂ - k·(λ₁ + λ₂)²
  Shi-Tomasi: R = min(λ₁, λ₂)

  Corner if min(λ₁, λ₂) > threshold

  Why better?
    - Simpler, more intuitive criterion
    - Better corner selection in practice
    - Returns the N strongest corners
```

```python
import cv2
import numpy as np

img = cv2.imread("building.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Shi-Tomasi: Good Features to Track
corners = cv2.goodFeaturesToTrack(
    gray,
    maxCorners=100,      # max number of corners
    qualityLevel=0.01,   # minimum quality (fraction of best corner)
    minDistance=10,       # min distance between corners (pixels)
    blockSize=3
)

# Draw corners
for corner in corners:
    x, y = corner.ravel().astype(int)
    cv2.circle(img, (x, y), 5, (0, 255, 0), -1)

cv2.imwrite("shi_tomasi_corners.jpg", img)
print(f"Found {len(corners)} corners")
```

---

## Comparison

| Feature | Harris | Shi-Tomasi |
|---------|:---:|:---:|
| Response formula | det(M) - k·trace(M)² | min(λ₁, λ₂) |
| Returns N best | No (thresholding) | Yes |
| Rotation invariant | ✓ | ✓ |
| Scale invariant | ✗ | ✗ |
| Speed | Fast | Fast |
| Typical use | General corner detection | Feature tracking |

---

## Revision Questions

1. **Why are corners more distinctive than edges or flat regions?**
2. **What is the structure tensor M and what do its eigenvalues represent?**
3. **Write the Harris response formula and explain each term.**
4. **How does Shi-Tomasi improve upon Harris?**
5. **Why are neither Harris nor Shi-Tomasi scale invariant?**

---

[Previous: ../02-Image-Processing-Basics/06-histogram-operations.md](../02-Image-Processing-Basics/06-histogram-operations.md) | [Next: 02-sift.md](02-sift.md)
