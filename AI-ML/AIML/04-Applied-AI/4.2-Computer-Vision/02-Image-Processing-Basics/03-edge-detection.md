# Edge Detection

## Overview

Edge detection identifies boundaries in images where pixel intensity changes sharply. Edges correspond to object boundaries, texture changes, and depth discontinuities. The two most important edge detectors are the **Sobel operator** (gradient-based) and the **Canny edge detector** (multi-stage, gold standard).

---

## What Is an Edge?

```
Intensity profile across an edge:

  Intensity
  255 │            ┌──────────
      │            │
      │            │
      │            │  ← Edge = sharp intensity transition
      │            │
    0 │────────────┘
      └──────────────────────── Position

  Edge = location where the first derivative (gradient) is maximum
         or where the second derivative crosses zero
```

---

## Sobel Operator

```
Detects edges using first-order derivatives (gradients):

Horizontal edges (Sobel Y):     Vertical edges (Sobel X):
  ┌────┬────┬────┐               ┌────┬────┬────┐
  │ -1 │ -2 │ -1 │               │ -1 │  0 │  1 │
  ├────┼────┼────┤               ├────┼────┼────┤
  │  0 │  0 │  0 │               │ -2 │  0 │  2 │
  ├────┼────┼────┤               ├────┼────┼────┤
  │  1 │  2 │  1 │               │ -1 │  0 │  1 │
  └────┴────┴────┘               └────┴────┴────┘

Gradient magnitude:
  G = √(Gx² + Gy²)

Gradient direction:
  θ = arctan(Gy / Gx)
```

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg", 0)

# Sobel edges
sobel_x = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)  # vertical edges
sobel_y = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)  # horizontal edges

# Gradient magnitude
magnitude = np.sqrt(sobel_x**2 + sobel_y**2)
magnitude = np.uint8(np.clip(magnitude, 0, 255))

# Gradient direction
direction = np.arctan2(sobel_y, sobel_x) * 180 / np.pi
```

---

## Canny Edge Detector

```
The gold standard — multi-stage pipeline:

Step 1: Gaussian blur (noise reduction)
  Apply Gaussian filter to smooth the image

Step 2: Gradient computation (Sobel)
  Compute Gx, Gy, magnitude, direction

Step 3: Non-Maximum Suppression (NMS)
  Thin edges to 1-pixel width
  For each pixel, check if it's the maximum along gradient direction
  If not → suppress (set to 0)

  Example along gradient direction →
  ┌────┬────┬────┐
  │ 50 │ 80 │ 60 │   80 is max along → direction
  └────┴────┴────┘   Keep 80, suppress 50 and 60

Step 4: Double thresholding
  High threshold: strong edge pixels
  Low threshold:  weak edge pixels (candidates)
  Below low:      suppress

Step 5: Hysteresis edge tracking
  Weak pixels connected to strong pixels → keep
  Weak pixels NOT connected to strong → discard

  Strong ──── Weak ──── Weak ──── nothing
    ✓           ✓         ✓          ✗
  (connected weak pixels are kept)
```

```python
import cv2

img = cv2.imread("photo.jpg", 0)

# Canny edge detection
edges = cv2.Canny(img, threshold1=50, threshold2=150)

# With Gaussian blur first (recommended)
blurred = cv2.GaussianBlur(img, (5, 5), 1.4)
edges_clean = cv2.Canny(blurred, 50, 150)

# Auto-threshold using median
median = np.median(img)
low = int(max(0, 0.67 * median))
high = int(min(255, 1.33 * median))
edges_auto = cv2.Canny(blurred, low, high)

cv2.imwrite("edges.jpg", edges_clean)
```

---

## Other Edge Detectors

```
Laplacian (second derivative):
  ┌────┬────┬────┐
  │  0 │  1 │  0 │     Detects edges in all directions
  ├────┼────┼────┤     Finds zero-crossings of 2nd derivative
  │  1 │ -4 │  1 │     Sensitive to noise
  ├────┼────┼────┤
  │  0 │  1 │  0 │
  └────┴────┴────┘

Prewitt (simpler than Sobel):
  ┌────┬────┬────┐     ┌────┬────┬────┐
  │ -1 │  0 │  1 │     │ -1 │ -1 │ -1 │
  │ -1 │  0 │  1 │     │  0 │  0 │  0 │
  │ -1 │  0 │  1 │     │  1 │  1 │  1 │
  └────┴────┴────┘     └────┴────┴────┘
     Gx (vertical)        Gy (horizontal)
```

---

## Comparison

| Detector | Noise Tolerance | Edge Quality | Speed | Multi-direction |
|----------|:---:|:---:|:---:|:---:|
| Sobel | Medium | Good | Fast | Separate X/Y |
| Canny | High | Excellent | Medium | All directions |
| Laplacian | Low | Good | Fast | All directions |
| Prewitt | Low | Moderate | Fast | Separate X/Y |

---

## Revision Questions

1. **What is an edge in terms of pixel intensity?**
2. **What are the Sobel X and Sobel Y kernels and what does each detect?**
3. **Explain the five steps of the Canny edge detector.**
4. **What is non-maximum suppression in edge detection?**
5. **Why is hysteresis thresholding better than a single threshold?**

---

[Previous: 02-convolution-operations.md](02-convolution-operations.md) | [Next: 04-image-gradients.md](04-image-gradients.md)
