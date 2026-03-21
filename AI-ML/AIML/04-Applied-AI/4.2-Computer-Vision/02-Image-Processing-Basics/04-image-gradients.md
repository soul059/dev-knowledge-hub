# Image Gradients

## Overview

Image gradients measure the rate and direction of intensity change at each pixel. They are the foundation of edge detection, feature descriptors (HOG, SIFT), and optical flow. The gradient at each pixel has a **magnitude** (how strong the change) and a **direction** (which way the intensity increases).

---

## Gradient Definition

```
For a grayscale image f(x, y):

  Gradient vector:
    ∇f = [∂f/∂x, ∂f/∂y] = [Gx, Gy]

  Magnitude:
    |∇f| = √(Gx² + Gy²)

  Direction:
    θ = arctan(Gy / Gx)

  Discrete approximation:
    Gx ≈ f(x+1, y) - f(x-1, y)     (horizontal change)
    Gy ≈ f(x, y+1) - f(x, y-1)     (vertical change)

  Visual:
    Low gradient:  smooth region (sky, wall)
    High gradient: edge, texture boundary
    Direction:     points from dark → bright
```

---

## Gradient Visualization

```
Original image (5×5):         Gradient magnitude:
┌────┬────┬────┬────┬────┐    ┌─────┬─────┬─────┬─────┬─────┐
│ 50 │ 50 │ 50 │200 │200 │    │  0  │  0  │ 150 │ 150 │  0  │
├────┼────┼────┼────┼────┤    ├─────┼─────┼─────┼─────┼─────┤
│ 50 │ 50 │ 50 │200 │200 │    │  0  │  0  │ 150 │ 150 │  0  │
├────┼────┼────┼────┼────┤    ├─────┼─────┼─────┼─────┼─────┤
│ 50 │ 50 │ 50 │200 │200 │    │  0  │  0  │ 150 │ 150 │  0  │
├────┼────┼────┼────┼────┤    ├─────┼─────┼─────┼─────┼─────┤
│ 50 │ 50 │ 50 │200 │200 │    │  0  │  0  │ 150 │ 150 │  0  │
├────┼────┼────┼────┼────┤    ├─────┼─────┼─────┼─────┼─────┤
│ 50 │ 50 │ 50 │200 │200 │    │  0  │  0  │ 150 │ 150 │  0  │
└────┴────┴────┴────┴────┘    └─────┴─────┴─────┴─────┴─────┘
     Dark  │ Bright                Edge detected at boundary
           ↑ vertical edge
```

---

## Computing Gradients

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg", 0).astype(np.float64)

# Method 1: Simple finite differences
gx = np.zeros_like(img)
gy = np.zeros_like(img)
gx[:, 1:-1] = img[:, 2:] - img[:, :-2]   # horizontal gradient
gy[1:-1, :] = img[2:, :] - img[:-2, :]   # vertical gradient

magnitude = np.sqrt(gx**2 + gy**2)
direction = np.arctan2(gy, gx) * 180 / np.pi  # degrees

# Method 2: Sobel operator (better noise handling)
gx_sobel = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)
gy_sobel = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)

mag_sobel = cv2.magnitude(gx_sobel, gy_sobel)
dir_sobel = cv2.phase(gx_sobel, gy_sobel, angleInDegrees=True)

# Method 3: Scharr operator (more accurate than Sobel for 3×3)
gx_scharr = cv2.Scharr(img, cv2.CV_64F, 1, 0)
gy_scharr = cv2.Scharr(img, cv2.CV_64F, 0, 1)
```

---

## Gradient Operators Compared

```
Finite difference:          Sobel:                  Scharr:
  [-1  0  1]                [-1  0  1]              [-3   0   3]
                             [-2  0  2]              [-10  0  10]
                             [-1  0  1]              [-3   0   3]

  Simplest, most noise     Good balance            Most accurate 3×3
  sensitive                of accuracy & speed     rotation invariance
```

| Operator | Kernel Size | Accuracy | Noise Handling |
|----------|:-:|:-:|:-:|
| Finite difference | 1×3 | Low | Poor |
| Prewitt | 3×3 | Medium | Medium |
| Sobel | 3×3 | Good | Good |
| Scharr | 3×3 | Best (3×3) | Good |

---

## Applications of Gradients

| Application | How Gradients Are Used |
|-------------|----------------------|
| Edge detection | High gradient magnitude = edge |
| HOG descriptor | Histogram of gradient orientations per cell |
| SIFT | Gradient orientations around keypoints |
| Optical flow | Track gradient changes between frames |
| Image sharpening | Add scaled gradient back to image |
| Corner detection | Analyze gradient structure in windows |

---

## Revision Questions

1. **What does the gradient vector [Gx, Gy] represent at a pixel?**
2. **How is gradient magnitude computed and what does it indicate?**
3. **What is the difference between Sobel and Scharr operators?**
4. **Why do gradients point from dark to bright regions?**
5. **Name three CV applications that rely on image gradients.**

---

[Previous: 03-edge-detection.md](03-edge-detection.md) | [Next: 05-morphological-operations.md](05-morphological-operations.md)
