# Image Segmentation (Watershed, GrabCut)

## Overview

Classical image segmentation partitions an image into meaningful regions without deep learning. **Watershed** treats the image as a topographic surface and floods from markers, while **GrabCut** uses graph cuts with a user-provided bounding box to separate foreground from background.

---

## Watershed Algorithm

```
Treat the gradient image as a topographic surface:
  High gradient = mountains (edges)
  Low gradient  = valleys (homogeneous regions)

Algorithm:
  1. Compute gradient magnitude (edges become ridges)
  2. Place markers (seeds) in regions of interest
  3. "Flood" water from each marker
  4. Where different floods meet → watershed line (boundary)

  Visualization:
    Gradient surface:
         /\    /\    /\
        /  \  /  \  /  \
       /    \/    \/    \     ← ridges = boundaries
      /  A   |  B  |  C  \   ← valleys = regions
     /       |     |       \
    
  Water rises from A, B, C markers
  Boundaries form where floods meet
```

```python
import cv2
import numpy as np

img = cv2.imread("coins.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Threshold
_, thresh = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

# Remove noise
kernel = np.ones((3, 3), np.uint8)
opening = cv2.morphologyEx(thresh, cv2.MORPH_OPEN, kernel, iterations=2)

# Find sure background (dilate)
sure_bg = cv2.dilate(opening, kernel, iterations=3)

# Find sure foreground (distance transform + threshold)
dist_transform = cv2.distanceTransform(opening, cv2.DIST_L2, 5)
_, sure_fg = cv2.threshold(dist_transform, 0.7 * dist_transform.max(), 255, 0)
sure_fg = np.uint8(sure_fg)

# Unknown region
unknown = cv2.subtract(sure_bg, sure_fg)

# Label markers
_, markers = cv2.connectedComponents(sure_fg)
markers = markers + 1          # background = 1, not 0
markers[unknown == 255] = 0    # unknown = 0

# Apply watershed
markers = cv2.watershed(img, markers)
img[markers == -1] = [0, 0, 255]  # boundary pixels in red

cv2.imwrite("watershed_result.jpg", img)
```

---

## GrabCut Algorithm

```
Interactive foreground segmentation:

  1. User provides bounding box around object
  2. Algorithm models foreground and background as GMMs
  3. Graph cut minimizes energy function
  4. Iteratively refines the segmentation

  Energy function:
    E = Σ data_term(pixel) + Σ smoothness_term(neighbors)
    
    data_term:      How well pixel fits FG/BG color model
    smoothness_term: Penalty for labeling neighbors differently

  User refinement:
    Mark definite FG/BG pixels → rerun for better results
```

```python
import cv2
import numpy as np

img = cv2.imread("person.jpg")
mask = np.zeros(img.shape[:2], np.uint8)

# Background and foreground models (internal)
bgd_model = np.zeros((1, 65), np.float64)
fgd_model = np.zeros((1, 65), np.float64)

# Bounding box around the object (x, y, w, h)
rect = (50, 50, 400, 500)

# Run GrabCut
cv2.grabCut(img, mask, rect, bgd_model, fgd_model,
            iterCount=5, mode=cv2.GC_INIT_WITH_RECT)

# Create binary mask (0,2 = background, 1,3 = foreground)
fg_mask = np.where((mask == 2) | (mask == 0), 0, 1).astype('uint8')
result = img * fg_mask[:, :, np.newaxis]

cv2.imwrite("grabcut_result.jpg", result)
```

---

## Other Classical Methods

```
Contour-based:
  cv2.findContours() → extracts object boundaries
  cv2.drawContours() → visualizes contours
  Useful for shape analysis after thresholding

Flood fill:
  Starting from a seed pixel, fill connected similar-colored region
  Like the "paint bucket" tool

K-means color segmentation:
  Cluster pixels by color → each cluster = one segment
  Simple but effective for color-distinct regions
```

---

## Comparison

| Method | Input Required | Best For | Quality |
|--------|---------------|----------|:---:|
| Watershed | Markers/seeds | Touching objects (coins) | Good |
| GrabCut | Bounding box | FG/BG separation | Very good |
| Thresholding | Threshold value | Simple binary | Basic |
| K-means | Number of clusters | Color-based segments | Moderate |
| Contours | Preprocessed binary | Shape extraction | Good |

---

## Revision Questions

1. **How does the watershed algorithm use the gradient image?**
2. **What is the purpose of markers in watershed segmentation?**
3. **How does GrabCut use GMMs and graph cuts?**
4. **What user input does GrabCut require?**
5. **When would you use watershed vs GrabCut?**

---

[Previous: 04-optical-flow.md](04-optical-flow.md) | [Back to README](../README.md)
