# Morphological Operations

## Overview

Morphological operations process images based on shape. They apply a **structuring element** (small shape template) to probe the image, useful for noise removal, object separation, boundary extraction, and filling holes in binary and grayscale images.

---

## Structuring Elements

```
Common structuring elements:

  Rectangle (3×3):    Ellipse (5×5):      Cross (3×3):
  ┌───┬───┬───┐       ┌───┬───┬───┬───┬───┐   ┌───┬───┬───┐
  │ 1 │ 1 │ 1 │       │ 0 │ 0 │ 1 │ 0 │ 0 │   │ 0 │ 1 │ 0 │
  ├───┼───┼───┤       ├───┼───┼───┼───┼───┤   ├───┼───┼───┤
  │ 1 │ 1 │ 1 │       │ 1 │ 1 │ 1 │ 1 │ 1 │   │ 1 │ 1 │ 1 │
  ├───┼───┼───┤       ├───┼───┼───┼───┼───┤   ├───┼───┼───┤
  │ 1 │ 1 │ 1 │       │ 1 │ 1 │ 1 │ 1 │ 1 │   │ 0 │ 1 │ 0 │
  └───┴───┴───┘       ├───┼───┼───┼───┼───┤   └───┴───┴───┘
                       │ 1 │ 1 │ 1 │ 1 │ 1 │
                       ├───┼───┼───┼───┼───┤
                       │ 0 │ 0 │ 1 │ 0 │ 0 │
                       └───┴───┴───┴───┴───┘
```

---

## Core Operations

```
EROSION: Shrink white regions, remove small noise
  Pixel = 1 ONLY IF all structuring element pixels fit inside object
  
  Input:               After erosion:
  ┌─┬─┬─┬─┬─┐         ┌─┬─┬─┬─┬─┐
  │ │ │█│ │ │         │ │ │ │ │ │
  │ │█│█│█│ │         │ │ │█│ │ │
  │█│█│█│█│█│   →     │ │█│█│█│ │
  │ │█│█│█│ │         │ │ │█│ │ │
  │ │ │█│ │ │         │ │ │ │ │ │
  └─┴─┴─┴─┴─┘         └─┴─┴─┴─┴─┘
  Diamond shrinks       Smaller diamond

DILATION: Grow white regions, fill small holes
  Pixel = 1 IF any structuring element pixel overlaps object
  
  Input:               After dilation:
  ┌─┬─┬─┬─┬─┐         ┌─┬─┬─┬─┬─┐
  │ │ │ │ │ │         │ │ │█│ │ │
  │ │ │█│ │ │         │ │█│█│█│ │
  │ │ │ │ │ │   →     │█│█│█│█│█│
  │ │ │ │ │ │         │ │█│█│█│ │
  │ │ │ │ │ │         │ │ │█│ │ │
  └─┴─┴─┴─┴─┘         └─┴─┴─┴─┴─┘
  Single pixel          Grows outward
```

---

## Compound Operations

```
OPENING = Erosion → Dilation
  Removes small noise while preserving shape
  "Open" gaps between touching objects

CLOSING = Dilation → Erosion
  Fills small holes while preserving shape
  "Close" small gaps in objects

GRADIENT = Dilation - Erosion
  Extracts object boundary/outline

TOP HAT = Original - Opening
  Extracts bright spots on dark background

BLACK HAT = Closing - Original
  Extracts dark spots on bright background
```

---

## Python Implementation

```python
import cv2
import numpy as np

img = cv2.imread("binary.jpg", 0)

# Create structuring element
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
# Other shapes: cv2.MORPH_ELLIPSE, cv2.MORPH_CROSS

# Basic operations
eroded = cv2.erode(img, kernel, iterations=1)
dilated = cv2.dilate(img, kernel, iterations=1)

# Compound operations
opening = cv2.morphologyEx(img, cv2.MORPH_OPEN, kernel)
closing = cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel)
gradient = cv2.morphologyEx(img, cv2.MORPH_GRADIENT, kernel)
tophat = cv2.morphologyEx(img, cv2.MORPH_TOPHAT, kernel)
blackhat = cv2.morphologyEx(img, cv2.MORPH_BLACKHAT, kernel)

# Practical: Clean noisy binary image
noisy = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)[1]
clean = cv2.morphologyEx(noisy, cv2.MORPH_OPEN, kernel)   # remove noise
clean = cv2.morphologyEx(clean, cv2.MORPH_CLOSE, kernel)  # fill holes
```

---

## Operations Summary

| Operation | Formula | Effect |
|-----------|---------|--------|
| Erosion | min(neighbors) | Shrinks objects, removes noise |
| Dilation | max(neighbors) | Grows objects, fills holes |
| Opening | erode → dilate | Remove small noise |
| Closing | dilate → erode | Fill small holes |
| Gradient | dilate - erode | Extract boundaries |
| Top hat | original - opening | Bright details on dark BG |
| Black hat | closing - original | Dark details on bright BG |

---

## Revision Questions

1. **What is a structuring element and how does it affect morphological operations?**
2. **How do erosion and dilation differ in their effect on a binary image?**
3. **Why is opening (erosion then dilation) effective for noise removal?**
4. **How would you extract object boundaries using morphological operations?**
5. **When would you use top hat vs black hat transforms?**

---

[Previous: 04-image-gradients.md](04-image-gradients.md) | [Next: 06-histogram-operations.md](06-histogram-operations.md)
