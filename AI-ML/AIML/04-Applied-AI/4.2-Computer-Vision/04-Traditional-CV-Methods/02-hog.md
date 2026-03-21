# Histogram of Oriented Gradients (HOG)

## Overview

HOG is a feature descriptor that captures the distribution of gradient orientations in local image regions. It was the key innovation behind the Dalal-Triggs pedestrian detector (2005) and remained the state-of-the-art for object detection until deep learning. HOG features are still used for fast detection on resource-constrained devices.

---

## HOG Pipeline

```
Image → [Gradient Computation] → [Cell Histograms] → [Block Normalization] → Feature Vector

Step 1: Compute gradients (Gx, Gy, magnitude, direction)
Step 2: Divide image into cells (e.g., 8×8 pixels)
Step 3: For each cell, build histogram of gradient orientations (9 bins)
Step 4: Normalize across blocks of cells (e.g., 2×2 cells per block)
Step 5: Concatenate all block histograms → feature vector
```

---

## Step-by-Step

```
Step 1: Gradients
  Gx = [-1, 0, 1] convolution
  Gy = [-1, 0, 1]ᵀ convolution
  Magnitude = √(Gx² + Gy²)
  Direction = arctan(Gy/Gx)   → 0° to 180° (unsigned)

Step 2: Cell histograms (8×8 cell, 9 bins)
  Bins: 0°, 20°, 40°, 60°, 80°, 100°, 120°, 140°, 160°
  
  Each pixel votes into the histogram:
    Vote weight = gradient magnitude
    Vote bin = gradient direction (with interpolation)

  Example cell histogram:
    0°  [██████     ]  30
   20°  [████       ]  20
   40°  [██         ]  10
   60°  [███        ]  15
   80°  [█████████  ]  45  ← dominant vertical edges
  100°  [████       ]  20
  120°  [██         ]  10
  140°  [███        ]  15
  160°  [████       ]  20

Step 3: Block normalization (2×2 cells per block)
  Concatenate 4 cell histograms: 4 × 9 = 36 values
  L2 normalize: v → v / √(||v||² + ε²)
  → Invariance to local illumination changes

Step 4: Feature vector
  For 64×128 detection window:
    Cells: 8×16 = 128 cells
    Blocks: 7×15 = 105 blocks (overlapping)
    Features per block: 36
    Total: 105 × 36 = 3,780 features
```

---

## Implementation

```python
import cv2
import numpy as np
from skimage.feature import hog
from skimage import exposure

img = cv2.imread("person.jpg", 0)
img = cv2.resize(img, (64, 128))  # standard detection window

# Using scikit-image
features, hog_image = hog(
    img,
    orientations=9,        # number of bins
    pixels_per_cell=(8, 8),
    cells_per_block=(2, 2),
    visualize=True,
    block_norm='L2-Hys'
)

print(f"Feature vector length: {len(features)}")  # 3780

# Rescale HOG visualization for display
hog_image = exposure.rescale_intensity(hog_image, in_range=(0, 10))

# Using OpenCV HOGDescriptor
hog_cv = cv2.HOGDescriptor()
hog_cv.setSVMDetector(cv2.HOGDescriptor_getDefaultPeopleDetector())

# Pedestrian detection
img_full = cv2.imread("street.jpg")
boxes, weights = hog_cv.detectMultiScale(img_full, winStride=(8, 8),
                                          padding=(16, 16), scale=1.05)

for (x, y, w, h) in boxes:
    cv2.rectangle(img_full, (x, y), (x+w, y+h), (0, 255, 0), 2)
```

---

## HOG + SVM for Detection

```
Training pipeline:
  1. Collect positive samples (objects) and negative samples (background)
  2. Extract HOG features from all samples
  3. Train linear SVM classifier
  4. At test time: sliding window + multi-scale + NMS

  This was the dominant detection approach before CNNs (2005-2014)
```

---

## Summary

| Aspect | Detail |
|--------|--------|
| Feature type | Gradient orientation histograms |
| Cell size | Typically 8×8 pixels |
| Block size | 2×2 cells (with overlap) |
| Orientations | 9 bins (0°-180° unsigned) |
| Normalization | L2-Hys per block |
| Classifier | Linear SVM |
| Best for | Pedestrian detection, rigid objects |
| Limitations | Not invariant to large deformations |

---

## Revision Questions

1. **What does HOG capture about an image region?**
2. **Why is block normalization important in HOG?**
3. **How many features does a 64×128 HOG descriptor produce?**
4. **How was HOG used for pedestrian detection?**
5. **Why has HOG been largely replaced by CNNs?**

---

[Previous: 01-template-matching.md](01-template-matching.md) | [Next: 03-haar-cascades.md](03-haar-cascades.md)
