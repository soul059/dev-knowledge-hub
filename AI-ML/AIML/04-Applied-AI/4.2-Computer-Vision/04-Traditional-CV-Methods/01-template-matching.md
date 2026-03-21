# Template Matching

## Overview

Template matching finds a small template image within a larger image by sliding the template across the image and computing a similarity score at each position. It's simple, fast, and effective for finding known objects that don't change in scale or rotation.

---

## How It Works

```
Slide template over the image, compute match score at each position:

  Image (H × W):                Template (h × w):
  ┌───────────────────────┐     ┌─────┐
  │                       │     │ ███ │
  │      ┌─────┐          │     │ █ █ │
  │      │ ███ │ ← match! │     │ ███ │
  │      │ █ █ │          │     └─────┘
  │      │ ███ │          │
  │      └─────┘          │     Score at each (x,y):
  │                       │     ┌───────────────────┐
  └───────────────────────┘     │    low low low     │
                                │    low HIGH low    │
  Result map: (H-h+1) × (W-w+1)│    low low low     │
                                └───────────────────┘
```

---

## Matching Methods

```
cv2.TM_SQDIFF:      Sum of squared differences (lower = better)
  R(x,y) = Σ (T(i,j) - I(x+i, y+j))²

cv2.TM_CCORR:       Cross-correlation (higher = better)
  R(x,y) = Σ T(i,j) × I(x+i, y+j)

cv2.TM_CCOEFF:      Correlation coefficient (higher = better)
  R(x,y) = Σ T'(i,j) × I'(x+i, y+j)
  where T' and I' are zero-mean versions

Normalized versions (_NORMED): Divide by energy → range [0,1] or [-1,1]
  TM_SQDIFF_NORMED:  0 = perfect match
  TM_CCORR_NORMED:   1 = perfect match
  TM_CCOEFF_NORMED:  1 = perfect match (best method)
```

---

## Implementation

```python
import cv2
import numpy as np

img = cv2.imread("screenshot.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
template = cv2.imread("icon.jpg", 0)
h, w = template.shape

# Template matching
result = cv2.matchTemplate(gray, template, cv2.TM_CCOEFF_NORMED)

# Find best match
min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)
top_left = max_loc  # for CCOEFF, max is best
bottom_right = (top_left[0] + w, top_left[1] + h)

cv2.rectangle(img, top_left, bottom_right, (0, 255, 0), 2)
print(f"Best match score: {max_val:.3f} at {top_left}")

# Find ALL matches above threshold
threshold = 0.8
locations = np.where(result >= threshold)
for pt in zip(*locations[::-1]):
    cv2.rectangle(img, pt, (pt[0]+w, pt[1]+h), (0, 0, 255), 2)

cv2.imwrite("matches.jpg", img)
```

---

## Limitations & Solutions

| Limitation | Description | Solution |
|------------|-------------|----------|
| Scale variance | Doesn't work if object is larger/smaller | Multi-scale matching (image pyramid) |
| Rotation variance | Doesn't work if object is rotated | Match at multiple angles |
| Illumination | Affected by lighting changes | Use normalized methods |
| Occlusion | Fails if object partially hidden | Feature-based matching |
| Speed | Slow for large images/templates | Pyramid + coarse-to-fine |

---

## Revision Questions

1. **How does template matching work conceptually?**
2. **What is the difference between TM_SQDIFF and TM_CCOEFF_NORMED?**
3. **How do you find multiple instances of a template?**
4. **Why is template matching not scale or rotation invariant?**
5. **When is template matching preferred over feature-based matching?**

---

[Previous: ../03-Feature-Detection/06-homography.md](../03-Feature-Detection/06-homography.md) | [Next: 02-hog.md](02-hog.md)
