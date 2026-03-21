# Images as Matrices

## Overview

Every digital image is a matrix (or tensor) of numbers. Understanding images as matrices unlocks the entire toolbox of linear algebra for image manipulation — addition, subtraction, multiplication, and transformations all become matrix operations.

---

## Image-Matrix Correspondence

```
Grayscale image = 2D matrix (H × W):
  ┌─────┬─────┬─────┬─────┐
  │ 120 │ 130 │ 140 │ 150 │    shape: (4, 4)
  ├─────┼─────┼─────┼─────┤    dtype: uint8
  │ 125 │ 135 │ 145 │ 155 │    range: [0, 255]
  ├─────┼─────┼─────┼─────┤
  │ 130 │ 140 │ 150 │ 160 │    Each element = pixel intensity
  ├─────┼─────┼─────┼─────┤
  │ 135 │ 145 │ 155 │ 165 │
  └─────┴─────┴─────┴─────┘

Color image = 3D tensor (H × W × 3):
  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
  │ B channel   │  │ G channel   │  │ R channel   │
  │ (H × W)     │  │ (H × W)     │  │ (H × W)     │
  └─────────────┘  └─────────────┘  └─────────────┘
     Channel 0        Channel 1        Channel 2

  pixel[y, x] = [B, G, R] = 3 values per pixel
```

---

## Matrix Operations on Images

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg")

# --- Brightness (scalar addition) ---
brighter = cv2.add(img, np.ones_like(img) * 50)   # add 50 to all pixels
darker   = cv2.subtract(img, np.ones_like(img) * 50)

# --- Contrast (scalar multiplication) ---
high_contrast = cv2.convertScaleAbs(img, alpha=1.5, beta=0)   # 1.5× contrast
low_contrast  = cv2.convertScaleAbs(img, alpha=0.5, beta=0)

# --- Image blending (weighted addition) ---
img1 = cv2.imread("image1.jpg")
img2 = cv2.imread("image2.jpg")
blended = cv2.addWeighted(img1, 0.7, img2, 0.3, 0)  # 70% img1 + 30% img2

# --- Channel operations ---
b, g, r = cv2.split(img)     # Split into channels
merged = cv2.merge([b, g, r]) # Merge back

# Only keep red channel
red_only = img.copy()
red_only[:, :, 0] = 0  # zero out blue
red_only[:, :, 1] = 0  # zero out green
```

---

## Geometric Transformations

```
Transformations as matrix multiplication:

Translation:
  [x']   [1  0  tx] [x]
  [y'] = [0  1  ty] [y]    Move by (tx, ty)
  [1 ]   [0  0  1 ] [1]

Scaling:
  [x']   [sx  0  0] [x]
  [y'] = [0  sy  0] [y]    Scale by (sx, sy)
  [1 ]   [0   0  1] [1]

Rotation (by angle θ):
  [x']   [cosθ  -sinθ  0] [x]
  [y'] = [sinθ   cosθ  0] [y]
  [1 ]   [0      0     1] [1]

Affine transform:
  [x']   [a  b  tx] [x]
  [y'] = [c  d  ty] [y]    Combines rotation, scale, shear, translation
  [1 ]   [0  0  1 ] [1]
```

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg")
h, w = img.shape[:2]

# Translation
tx, ty = 100, 50
M_translate = np.float32([[1, 0, tx], [0, 1, ty]])
translated = cv2.warpAffine(img, M_translate, (w, h))

# Rotation (45 degrees around center)
center = (w // 2, h // 2)
M_rotate = cv2.getRotationMatrix2D(center, angle=45, scale=1.0)
rotated = cv2.warpAffine(img, M_rotate, (w, h))

# Scaling
scaled = cv2.resize(img, None, fx=0.5, fy=0.5)  # half size

# Flip
flipped_h = cv2.flip(img, 1)   # horizontal
flipped_v = cv2.flip(img, 0)   # vertical
flipped_both = cv2.flip(img, -1) # both
```

---

## Data Types and Normalization

```
Common data types:
  uint8:    0-255 (standard image format)
  float32:  0.0-1.0 (for neural networks)
  float64:  0.0-1.0 (high precision)

Conversions:
  uint8 → float32:  img_float = img.astype(np.float32) / 255.0
  float32 → uint8:  img_uint8 = (img_float * 255).astype(np.uint8)

Neural network input:
  1. Resize to expected size (e.g., 224×224)
  2. Convert to float32
  3. Normalize: (pixel - mean) / std
     ImageNet mean: [0.485, 0.456, 0.406]
     ImageNet std:  [0.229, 0.224, 0.225]
  4. Transpose: (H, W, C) → (C, H, W) for PyTorch
```

---

## Memory Calculation

```
Image size in memory:

  Grayscale 1080p:  1920 × 1080 × 1 byte  = 2.07 MB
  Color 1080p:      1920 × 1080 × 3 bytes  = 6.22 MB
  Color 4K:         3840 × 2160 × 3 bytes  = 24.88 MB
  Float32 4K:       3840 × 2160 × 3 × 4    = 99.53 MB

  Batch of 32 images (224×224×3, float32):
    32 × 224 × 224 × 3 × 4 bytes = 19.27 MB
```

---

## Revision Questions

1. **What is the shape of a 1080p color image in NumPy?**
2. **How do you increase image brightness using matrix operations?**
3. **Write the 3×3 rotation matrix for a 90° rotation.**
4. **Why do neural networks expect float32 normalized images instead of uint8?**
5. **How much memory does a batch of 64 RGB images of size 224×224 occupy?**

---

[Previous: 04-color-spaces.md](04-color-spaces.md) | [Back to README](../README.md)
