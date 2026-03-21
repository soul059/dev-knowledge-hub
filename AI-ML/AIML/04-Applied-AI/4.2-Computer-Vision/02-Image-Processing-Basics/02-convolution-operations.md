# Convolution Operations

## Overview

Convolution is the core mathematical operation in image processing and CNNs. It slides a kernel over an image, computing element-wise multiplication and summation at each position. Understanding convolution deeply is essential — it's used in filtering, edge detection, feature extraction, and every convolutional neural network.

---

## 2D Convolution Formula

```
(f * k)[i, j] = ΣΣ f[i - m, j - n] × k[m, n]
                 m  n

Where:
  f = input image
  k = kernel (filter)
  * = convolution operator

In practice, most libraries implement CORRELATION (no kernel flip):
  (f ⊛ k)[i, j] = ΣΣ f[i + m, j + n] × k[m, n]

For symmetric kernels (Gaussian, box), correlation = convolution
```

---

## Step-by-Step Example

```
Input image (5×5):                  Kernel (3×3):
┌───┬───┬───┬───┬───┐              ┌───┬───┬───┐
│ 1 │ 2 │ 3 │ 0 │ 1 │              │ 1 │ 0 │ -1│
├───┼───┼───┼───┼───┤              ├───┼───┼───┤
│ 0 │ 1 │ 2 │ 3 │ 0 │              │ 1 │ 0 │ -1│
├───┼───┼───┼───┼───┤              ├───┼───┼───┤
│ 3 │ 0 │ 1 │ 2 │ 1 │              │ 1 │ 0 │ -1│
├───┼───┼───┼───┼───┤              └───┴───┴───┘
│ 1 │ 2 │ 0 │ 1 │ 3 │
├───┼───┼───┼───┼───┤
│ 2 │ 1 │ 3 │ 0 │ 2 │
└───┴───┴───┴───┴───┘

Position (1,1) — center of first valid window:
  ┌───┬───┬───┐     ┌───┬───┬───┐
  │ 1 │ 2 │ 3 │     │ 1 │ 0 │-1 │
  │ 0 │ 1 │ 2 │  ×  │ 1 │ 0 │-1 │
  │ 3 │ 0 │ 1 │     │ 1 │ 0 │-1 │
  └───┴───┴───┘     └───┴───┴───┘

  = 1×1 + 2×0 + 3×(-1) + 0×1 + 1×0 + 2×(-1) + 3×1 + 0×0 + 1×(-1)
  = 1 + 0 - 3 + 0 + 0 - 2 + 3 + 0 - 1
  = -2
```

---

## Output Size Calculation

```
Output size formula:
  O = (I - K + 2P) / S + 1

  I = input size
  K = kernel size
  P = padding
  S = stride

Examples:
  I=5, K=3, P=0, S=1:  O = (5-3+0)/1 + 1 = 3    → 3×3 output
  I=5, K=3, P=1, S=1:  O = (5-3+2)/1 + 1 = 5    → same size (valid)
  I=5, K=3, P=0, S=2:  O = (5-3+0)/2 + 1 = 2    → 2×2 output

Padding types:
  "valid":  P=0  → output shrinks
  "same":   P=K//2 → output same size as input
  "full":   P=K-1 → output grows
```

---

## Stride and Padding Visualization

```
Stride = 1 (default):              Stride = 2:
  ┌─┬─┬─┬─┬─┐                      ┌─┬─┬─┬─┬─┐
  │█│█│█│ │ │  → slide 1            │█│█│█│ │ │  → jump 2
  │█│█│█│ │ │                        │█│█│█│ │ │
  │█│█│█│ │ │                        │█│█│█│ │ │
  │ │ │ │ │ │                        │ │ │ │ │ │
  │ │ │ │ │ │                        │ │ │ │ │ │
  └─┴─┴─┴─┴─┘                      └─┴─┴─┴─┴─┘
  then:                              then:
  ┌─┬─┬─┬─┬─┐                      ┌─┬─┬─┬─┬─┐
  │ │█│█│█│ │                        │ │ │█│█│█│
  │ │█│█│█│ │                        │ │ │█│█│█│
  │ │█│█│█│ │                        │ │ │█│█│█│
  │ │ │ │ │ │                        │ │ │ │ │ │
  └─┴─┴─┴─┴─┘                      └─┴─┴─┴─┴─┘

Padding (P=1, adds zeros around border):
  ┌───┬───┬───┬───┬───┬───┬───┐
  │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
  ├───┼───┼───┼───┼───┼───┼───┤
  │ 0 │ 1 │ 2 │ 3 │ 0 │ 1 │ 0 │
  │ 0 │ 0 │ 1 │ 2 │ 3 │ 0 │ 0 │  5×5 → padded to 7×7
  │ 0 │ 3 │ 0 │ 1 │ 2 │ 1 │ 0 │  Output with 3×3 kernel = 5×5 (same!)
  │ 0 │ 1 │ 2 │ 0 │ 1 │ 3 │ 0 │
  │ 0 │ 2 │ 1 │ 3 │ 0 │ 2 │ 0 │
  │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
  └───┴───┴───┴───┴───┴───┴───┘
```

---

## Implementation

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg", 0)  # grayscale

# OpenCV filter2D (correlation)
kernel = np.array([[1, 0, -1],
                    [1, 0, -1],
                    [1, 0, -1]], dtype=np.float32)

result = cv2.filter2D(img, -1, kernel)

# Manual convolution with NumPy
def convolve2d(image, kernel):
    kh, kw = kernel.shape
    ph, pw = kh // 2, kw // 2
    padded = np.pad(image, ((ph, ph), (pw, pw)), mode='constant')
    
    output = np.zeros_like(image, dtype=np.float64)
    for i in range(image.shape[0]):
        for j in range(image.shape[1]):
            patch = padded[i:i+kh, j:j+kw]
            output[i, j] = np.sum(patch * kernel)
    
    return np.clip(output, 0, 255).astype(np.uint8)

result_manual = convolve2d(img, kernel)
```

---

## Convolution vs Correlation

| Operation | Kernel Flipped? | Used In |
|-----------|:---:|---------|
| Convolution | Yes (flip 180°) | Signal processing, math |
| Correlation | No | OpenCV `filter2D`, deep learning |

For symmetric kernels, they produce identical results.

---

## Revision Questions

1. **Write the formula for 2D convolution.**
2. **Calculate the output size for input=7×7, kernel=3×3, padding=1, stride=2.**
3. **What is the difference between convolution and correlation?**
4. **What does "same" padding achieve?**
5. **Why is stride > 1 used in CNNs?**

---

[Previous: 01-image-filtering.md](01-image-filtering.md) | [Next: 03-edge-detection.md](03-edge-detection.md)
