# Feature Matching

## Overview

Feature matching finds corresponding keypoints between two images. After detecting features and computing descriptors, matching algorithms compare descriptors to establish point correspondences. This is the core of image stitching, object recognition, visual SLAM, and 3D reconstruction.

---

## Matching Pipeline

```
Image A              Image B
   │                    │
   ▼                    ▼
[Detect keypoints]   [Detect keypoints]
   │                    │
   ▼                    ▼
[Compute descriptors] [Compute descriptors]
   │                    │
   └──────┬─────────────┘
          ▼
   [Match descriptors]
          │
          ▼
   [Filter bad matches]
          │
          ▼
   Corresponding point pairs
```

---

## Brute-Force Matching

```
For each descriptor in Image A:
  Compare with ALL descriptors in Image B
  Find the closest match (minimum distance)

Distance metrics:
  L2 (Euclidean):  for SIFT, SURF (float descriptors)
    d = √(Σ(a_i - b_i)²)

  Hamming:  for ORB, BRIEF (binary descriptors)
    d = popcount(a XOR b)   (count differing bits)

Complexity: O(N × M) where N, M = number of keypoints
```

```python
import cv2

img1 = cv2.imread("image1.jpg", 0)
img2 = cv2.imread("image2.jpg", 0)

# Detect features (ORB)
orb = cv2.ORB_create(1000)
kp1, des1 = orb.detectAndCompute(img1, None)
kp2, des2 = orb.detectAndCompute(img2, None)

# Brute-force matching
bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
matches = bf.match(des1, des2)

# Sort by distance (best matches first)
matches = sorted(matches, key=lambda x: x.distance)

# Draw top 30 matches
img_matches = cv2.drawMatches(img1, kp1, img2, kp2, matches[:30], None,
                               flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS)
cv2.imwrite("matches.jpg", img_matches)
```

---

## FLANN Matching (Faster)

```python
import cv2

# SIFT features
sift = cv2.SIFT_create()
kp1, des1 = sift.detectAndCompute(img1, None)
kp2, des2 = sift.detectAndCompute(img2, None)

# FLANN (Fast Library for Approximate Nearest Neighbors)
FLANN_INDEX_KDTREE = 1
index_params = dict(algorithm=FLANN_INDEX_KDTREE, trees=5)
search_params = dict(checks=50)

flann = cv2.FlannBasedMatcher(index_params, search_params)
matches = flann.knnMatch(des1, des2, k=2)  # k=2 for ratio test

# Lowe's ratio test (filter ambiguous matches)
good_matches = []
for m, n in matches:
    if m.distance < 0.75 * n.distance:  # best < 75% of second best
        good_matches.append(m)

print(f"Good matches: {len(good_matches)} / {len(matches)}")
```

---

## Lowe's Ratio Test

```
For each descriptor, find 2 nearest neighbors:
  Best match:    distance d₁
  2nd best match: distance d₂

  If d₁ / d₂ < threshold (0.7-0.8):
    → Best match is much better than alternatives → KEEP ✓
  Else:
    → Ambiguous match → REJECT ✗

  Example:
    Feature A: d₁=50, d₂=200  → ratio=0.25 → KEEP (distinctive)
    Feature B: d₁=80, d₂=90   → ratio=0.89 → REJECT (ambiguous)
```

---

## BF vs FLANN

| Method | Speed | Accuracy | Best For |
|--------|:---:|:---:|---------|
| Brute-Force | Slow (O(N×M)) | Exact | Small datasets, ORB |
| FLANN | Fast (approximate) | ~99% | Large datasets, SIFT/SURF |

---

## Revision Questions

1. **What distance metric do you use for SIFT vs ORB matching?**
2. **How does Lowe's ratio test filter bad matches?**
3. **What is FLANN and why is it faster than brute-force?**
4. **What does crossCheck=True do in brute-force matching?**
5. **Why do we sort matches by distance?**

---

[Previous: 04-orb.md](04-orb.md) | [Next: 06-homography.md](06-homography.md)
