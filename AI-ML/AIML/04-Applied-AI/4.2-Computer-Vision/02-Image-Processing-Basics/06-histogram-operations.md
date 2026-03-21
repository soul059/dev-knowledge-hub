# Histogram Operations

## Overview

An image histogram shows the distribution of pixel intensities. It reveals whether an image is dark, bright, low-contrast, or high-contrast. Histogram operations — equalization, stretching, and matching — are fundamental preprocessing techniques that improve image quality and normalize inputs for CV algorithms.

---

## Understanding Histograms

```
Dark image histogram:       Bright image histogram:
  Count                       Count
  │ ██                        │              ██
  │ ████                      │           █████
  │ ██████                    │         ████████
  │ ████████                  │       ██████████
  └──────────────── Intensity └──────────────── Intensity
  0              255          0              255
  (pixels clustered left)     (pixels clustered right)

Low contrast:               High contrast:
  Count                       Count
  │       ████                │ ██        ██
  │      ██████               │ ████    ████
  │     ████████              │ ██████████████
  └──────────────── Intensity └──────────────── Intensity
  (narrow range)              (spread across full range)
```

---

## Computing Histograms

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt

img = cv2.imread("photo.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Grayscale histogram
hist = cv2.calcHist([gray], [0], None, [256], [0, 256])
plt.plot(hist)
plt.title("Grayscale Histogram")
plt.xlabel("Pixel Intensity")
plt.ylabel("Frequency")
plt.show()

# Color histogram (per channel)
colors = ('b', 'g', 'r')
for i, color in enumerate(colors):
    hist = cv2.calcHist([img], [i], None, [256], [0, 256])
    plt.plot(hist, color=color)
plt.title("Color Histogram")
plt.show()

# With NumPy
hist_np, bins = np.histogram(gray.ravel(), 256, [0, 256])
```

---

## Histogram Equalization

```
Spreads pixel values across the full range → improves contrast

Algorithm:
  1. Compute histogram h(i) for i = 0..255
  2. Compute CDF: cdf(i) = Σ_{j=0}^{i} h(j)
  3. Normalize CDF: cdf_norm(i) = cdf(i) / total_pixels
  4. Map: new_value(i) = round(cdf_norm(i) × 255)

Before equalization:          After equalization:
  Count                         Count
  │    ████                     │  ██  ██  ██  ██
  │   ██████                    │ ████████████████
  │  ████████                   │ ██████████████████
  └────────── Intensity         └────────── Intensity
  (clustered)                   (spread out)
```

```python
import cv2

gray = cv2.imread("dark_photo.jpg", 0)

# Global histogram equalization
equalized = cv2.equalizeHist(gray)

# CLAHE (Contrast Limited Adaptive Histogram Equalization)
# Applies equalization locally in tiles — much better results
clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
clahe_result = clahe.apply(gray)

# CLAHE on color image (apply to L channel in LAB)
img = cv2.imread("dark_photo.jpg")
lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)
l, a, b = cv2.split(lab)
l_clahe = clahe.apply(l)
lab_clahe = cv2.merge([l_clahe, a, b])
result = cv2.cvtColor(lab_clahe, cv2.COLOR_LAB2BGR)
```

---

## CLAHE vs Global Equalization

```
Global equalization:
  - Applies same transformation to entire image
  - Can over-amplify noise in uniform regions
  - May wash out some areas

CLAHE:
  - Divides image into tiles (e.g., 8×8)
  - Equalizes each tile independently
  - Clip limit prevents over-amplification
  - Interpolates tile boundaries for smooth result
  - Much better for real-world images
```

---

## Histogram Matching (Specification)

```python
from skimage import exposure

# Match histogram of source to reference image
source = cv2.imread("source.jpg", 0)
reference = cv2.imread("reference.jpg", 0)

matched = exposure.match_histograms(source, reference)

# Use case: Make a series of images have consistent brightness/contrast
# Common in satellite imagery, medical imaging, video processing
```

---

## Thresholding

```python
import cv2

gray = cv2.imread("document.jpg", 0)

# Simple thresholding
_, binary = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

# Otsu's method (automatic threshold selection)
_, otsu = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

# Adaptive thresholding (local thresholds — best for uneven lighting)
adaptive = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                  cv2.THRESH_BINARY, blockSize=11, C=2)
```

---

## Summary

| Operation | Purpose | When to Use |
|-----------|---------|-------------|
| Histogram | Analyze intensity distribution | Understand image properties |
| Equalization | Improve global contrast | Dark/low-contrast images |
| CLAHE | Improve local contrast | Real-world images, medical |
| Matching | Match distributions | Consistent image series |
| Otsu threshold | Auto binarization | Document scanning |
| Adaptive threshold | Local binarization | Uneven lighting |

---

## Revision Questions

1. **What does an image histogram tell you about the image?**
2. **How does histogram equalization improve contrast?**
3. **Why is CLAHE better than global equalization for most images?**
4. **How does Otsu's method automatically select a threshold?**
5. **When would you use adaptive thresholding over global thresholding?**

---

[Previous: 05-morphological-operations.md](05-morphological-operations.md) | [Back to README](../README.md)
