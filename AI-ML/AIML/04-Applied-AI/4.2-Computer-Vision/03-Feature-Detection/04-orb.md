# ORB (Oriented FAST and Rotated BRIEF)

## Overview

ORB is a fast, free, and efficient alternative to SIFT and SURF. It combines the FAST keypoint detector with the BRIEF descriptor, adding orientation to achieve rotation invariance. ORB is the go-to feature detector for real-time applications where speed matters.

---

## ORB = FAST + BRIEF + Improvements

```
FAST detector:
  Tests 16 pixels on a circle of radius 3 around candidate point
  If N contiguous pixels are all brighter (or darker) than center → corner

        ○ ○ ○
      ○       ○
    ○           ○
    ○     X     ○    X = candidate pixel
    ○           ○    ○ = test pixels (radius 3)
      ○       ○
        ○ ○ ○

  Very fast (just intensity comparisons)
  But: NOT scale invariant, NOT rotation invariant

ORB improvements:
  1. Scale: Build image pyramid, detect FAST at each level
  2. Orientation: Intensity centroid method
     C = (m10/m00, m01/m00)  where mpq = Σ x^p × y^q × I(x,y)
     θ = atan2(m01, m10)
  3. Descriptor: Rotated BRIEF (rBRIEF)
```

---

## BRIEF Descriptor

```
Binary Robust Independent Elementary Features:

  1. Take 31×31 patch around keypoint
  2. Select 256 pixel pairs (predefined pattern)
  3. For each pair (p₁, p₂):
     bit = 1 if I(p₁) > I(p₂), else 0
  4. Result: 256-bit binary string

  Example:
    Pair 1: pixel(3,5) vs pixel(12,8)  → 180 > 120 → 1
    Pair 2: pixel(7,2) vs pixel(1,14)  → 95 < 200  → 0
    ...
    Descriptor: 10110010...  (256 bits = 32 bytes)

  Matching: Hamming distance (count different bits)
    → XOR + popcount = extremely fast!
    
  BRIEF Hamming distance: ~10× faster than SIFT L2 distance
```

---

## rBRIEF (Rotated BRIEF)

```
Problem: BRIEF is NOT rotation invariant
Solution: Rotate the sampling pattern by the keypoint orientation

  Standard BRIEF:           rBRIEF:
  ┌─────────────┐           ┌─────────────┐
  │  •    •     │           │    •        │
  │    •   •    │           │  •     •    │
  │  •      •   │   Rotate  │       •     │
  │     •  •    │   by θ →  │  •   •      │
  │  •       •  │           │      •  •   │
  └─────────────┘           └─────────────┘
  Fixed pattern             Rotated pattern

  ORB precomputes rotated versions for efficiency
```

---

## Implementation

```python
import cv2

img = cv2.imread("scene.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Create ORB detector
orb = cv2.ORB_create(
    nfeatures=1000,        # max keypoints
    scaleFactor=1.2,       # pyramid scale factor
    nlevels=8,             # pyramid levels
    edgeThreshold=31,
    patchSize=31,
    scoreType=cv2.ORB_HARRIS_SCORE  # or ORB_FAST_SCORE
)

# Detect and compute
keypoints, descriptors = orb.detectAndCompute(gray, None)

print(f"Keypoints: {len(keypoints)}")
print(f"Descriptor shape: {descriptors.shape}")    # (N, 32) uint8
print(f"Descriptor dtype: {descriptors.dtype}")      # uint8 (binary)

# Draw
img_kp = cv2.drawKeypoints(img, keypoints, None, color=(0, 255, 0))
cv2.imwrite("orb_keypoints.jpg", img_kp)

# Matching with Hamming distance
bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
# matches = bf.match(desc1, desc2)
```

---

## Feature Detector Comparison

| Feature | SIFT | SURF | ORB |
|---------|:---:|:---:|:---:|
| Descriptor size | 128 float (512 B) | 64 float (256 B) | 256 bit (32 B) |
| Distance metric | L2 (Euclidean) | L2 | Hamming |
| Speed (detect) | Slow | Medium | Very fast |
| Speed (match) | Slow | Medium | Very fast |
| Scale invariant | ✓ | ✓ | ✓ (pyramid) |
| Rotation invariant | ✓ | ✓ | ✓ (rBRIEF) |
| Free / Open | ✓ (patent expired) | ✗ (patented) | ✓ |
| Best for | Accuracy-critical | Balanced | Real-time |

---

## Revision Questions

1. **How does the FAST corner detector work?**
2. **What is BRIEF and how does it create a binary descriptor?**
3. **How does ORB achieve rotation invariance?**
4. **Why is Hamming distance faster than Euclidean distance?**
5. **When would you choose ORB over SIFT?**

---

[Previous: 03-surf.md](03-surf.md) | [Next: 05-feature-matching.md](05-feature-matching.md)
