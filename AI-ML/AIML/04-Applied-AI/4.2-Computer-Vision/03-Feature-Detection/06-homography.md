# Homography

## Overview

A homography is a 3×3 transformation matrix that maps points from one plane to another. It describes the geometric relationship between two views of a planar surface and is essential for image stitching (panoramas), perspective correction, augmented reality, and visual SLAM.

---

## Homography Matrix

```
Maps point (x, y) in image 1 to (x', y') in image 2:

  [x']   [h₁₁  h₁₂  h₁₃] [x]
  [y'] = [h₂₁  h₂₂  h₂₃] [y]   (homogeneous coordinates)
  [w']   [h₃₁  h₃₂  h₃₃] [1]

  x'_actual = x'/w'
  y'_actual = y'/w'

  H has 8 degrees of freedom (9 elements, but scale is arbitrary)
  → Need at least 4 point correspondences to solve

  H encodes: rotation + translation + scaling + perspective
```

---

## Computing Homography

```
Given 4+ point correspondences between two images:

  Image 1 points    →    Image 2 points
  (x₁, y₁)              (x₁', y₁')
  (x₂, y₂)              (x₂', y₂')
  (x₃, y₃)              (x₃', y₃')
  (x₄, y₄)              (x₄', y₄')

  Solve for H using Direct Linear Transform (DLT):
    Set up system Ah = 0 (8 equations, 8 unknowns)
    Solve via SVD

  With outliers → use RANSAC:
    1. Randomly select 4 matches
    2. Compute H from these 4 points
    3. Count inliers (other points that agree with this H)
    4. Repeat many times
    5. Keep H with most inliers
    6. Recompute H using all inliers
```

---

## Panorama Stitching Pipeline

```
Image A                      Image B
   │                            │
   ▼                            ▼
[Detect SIFT/ORB]           [Detect SIFT/ORB]
   │                            │
   └──────── Match ─────────────┘
                │
                ▼
      [RANSAC → Homography H]
                │
                ▼
      [Warp Image A using H]
                │
                ▼
      [Blend A and B → Panorama]
```

```python
import cv2
import numpy as np

img1 = cv2.imread("left.jpg")
img2 = cv2.imread("right.jpg")

# 1. Detect and match features
sift = cv2.SIFT_create()
kp1, des1 = sift.detectAndCompute(cv2.cvtColor(img1, cv2.COLOR_BGR2GRAY), None)
kp2, des2 = sift.detectAndCompute(cv2.cvtColor(img2, cv2.COLOR_BGR2GRAY), None)

bf = cv2.BFMatcher()
matches = bf.knnMatch(des1, des2, k=2)

# Lowe's ratio test
good = [m for m, n in matches if m.distance < 0.75 * n.distance]

# 2. Compute homography with RANSAC
src_pts = np.float32([kp1[m.queryIdx].pt for m in good]).reshape(-1, 1, 2)
dst_pts = np.float32([kp2[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)

H, mask = cv2.findHomography(src_pts, dst_pts, cv2.RANSAC, 5.0)
print(f"Homography:\n{H}")
print(f"Inliers: {mask.sum()} / {len(good)}")

# 3. Warp and stitch
h, w = img1.shape[:2]
warped = cv2.warpPerspective(img1, H, (w * 2, h))
warped[0:h, 0:w] = img2  # simple overlay (production uses blending)

cv2.imwrite("panorama.jpg", warped)
```

---

## Perspective Correction

```python
import cv2
import numpy as np

img = cv2.imread("document_photo.jpg")

# Corners of the document in the photo (detected or manual)
src_points = np.float32([[56, 65], [368, 52], [28, 387], [389, 390]])

# Desired output corners (rectangle)
dst_points = np.float32([[0, 0], [400, 0], [0, 500], [400, 500]])

# Compute homography
H = cv2.getPerspectiveTransform(src_points, dst_points)

# Warp to get flat document
corrected = cv2.warpPerspective(img, H, (400, 500))
cv2.imwrite("document_flat.jpg", corrected)
```

---

## Applications

| Application | How Homography Is Used |
|-------------|----------------------|
| Panorama stitching | Warp images to common coordinate frame |
| Document scanning | Perspective correction to flat view |
| Augmented reality | Overlay virtual content on planar surfaces |
| Sports analysis | Map field markings to top-down view |
| Visual SLAM | Camera pose estimation from planar features |

---

## Revision Questions

1. **What is a homography and how many degrees of freedom does it have?**
2. **How many point correspondences are needed to compute a homography?**
3. **Why is RANSAC needed when computing homography from feature matches?**
4. **Describe the panorama stitching pipeline.**
5. **How is homography used for document scanning?**

---

[Previous: 05-feature-matching.md](05-feature-matching.md) | [Back to README](../README.md)
