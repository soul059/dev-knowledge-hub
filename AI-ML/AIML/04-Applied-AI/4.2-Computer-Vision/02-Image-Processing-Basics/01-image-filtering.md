# Image Filtering

## Overview

Image filtering applies a kernel (small matrix) to every pixel in an image, replacing each pixel with a weighted combination of its neighbors. Filtering is used for blurring, sharpening, noise removal, and edge detection — it's the bread and butter of image processing.

---

## How Filtering Works

```
Kernel (3×3) slides over the image:

  Image patch:        Averaging kernel:      Output pixel:
  ┌────┬────┬────┐    ┌─────┬─────┬─────┐
  │ 10 │ 20 │ 30 │    │ 1/9 │ 1/9 │ 1/9 │    Output = Σ(patch × kernel)
  ├────┼────┼────┤    ├─────┼─────┼─────┤    = (10+20+30+40+50+60+70+80+90)/9
  │ 40 │ 50 │ 60 │  × │ 1/9 │ 1/9 │ 1/9 │    = 50
  ├────┼────┼────┤    ├─────┼─────┼─────┤
  │ 70 │ 80 │ 90 │    │ 1/9 │ 1/9 │ 1/9 │
  └────┴────┴────┘    └─────┴─────┴─────┘

  Slide kernel across entire image → filtered image
```

---

## Common Filters

```
Box (Average) filter:               Gaussian filter (σ=1):
  ┌─────┬─────┬─────┐               ┌──────┬──────┬──────┐
  │ 1/9 │ 1/9 │ 1/9 │               │ 1/16 │ 2/16 │ 1/16 │
  ├─────┼─────┼─────┤               ├──────┼──────┼──────┤
  │ 1/9 │ 1/9 │ 1/9 │               │ 2/16 │ 4/16 │ 2/16 │
  ├─────┼─────┼─────┤               ├──────┼──────┼──────┤
  │ 1/9 │ 1/9 │ 1/9 │               │ 1/16 │ 2/16 │ 1/16 │
  └─────┴─────┴─────┘               └──────┴──────┴──────┘
  Uniform blur                       Weighted blur (center heavier)

Sharpening kernel:
  ┌────┬────┬────┐
  │  0 │ -1 │  0 │      Enhances differences from neighbors
  ├────┼────┼────┤      Center pixel = 5× minus neighbors
  │ -1 │  5 │ -1 │
  ├────┼────┼────┤
  │  0 │ -1 │  0 │
  └────┴────┴────┘
```

---

## Python Implementation

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg")

# Box (average) blur
box_blur = cv2.blur(img, (5, 5))  # 5×5 kernel

# Gaussian blur
gauss_blur = cv2.GaussianBlur(img, (5, 5), sigmaX=1.0)

# Median filter (great for salt-and-pepper noise)
median_blur = cv2.medianBlur(img, 5)

# Bilateral filter (smooths while preserving edges)
bilateral = cv2.bilateralFilter(img, d=9, sigmaColor=75, sigmaSpace=75)

# Custom kernel (sharpening)
sharpen_kernel = np.array([[ 0, -1,  0],
                            [-1,  5, -1],
                            [ 0, -1,  0]])
sharpened = cv2.filter2D(img, -1, sharpen_kernel)

# Custom kernel (emboss)
emboss_kernel = np.array([[-2, -1, 0],
                           [-1,  1, 1],
                           [ 0,  1, 2]])
embossed = cv2.filter2D(img, -1, emboss_kernel)
```

---

## Filter Comparison

| Filter | Noise Removal | Edge Preservation | Speed | Best For |
|--------|:---:|:---:|:---:|---------|
| Box (average) | Good | Poor | Fastest | Quick smoothing |
| Gaussian | Good | Poor | Fast | General blurring |
| Median | Excellent (salt&pepper) | Good | Medium | Impulse noise |
| Bilateral | Good | Excellent | Slow | Photo enhancement |

---

## Border Handling

```
What happens at image edges where kernel extends beyond?

Options:
  BORDER_CONSTANT:   Pad with zeros (or specified value)
  BORDER_REPLICATE:  Repeat edge pixels
  BORDER_REFLECT:    Mirror pixels at border
  BORDER_WRAP:       Wrap around (circular)

  Original:    ... 5 | 8  6  3  1 | 2 ...
  
  CONSTANT:    0  0 | 8  6  3  1 | 0  0
  REPLICATE:   8  8 | 8  6  3  1 | 1  1
  REFLECT:     6  8 | 8  6  3  1 | 1  3
```

---

## Revision Questions

1. **What is a kernel and how does it process an image?**
2. **What is the difference between a box filter and a Gaussian filter?**
3. **Why is median filtering better for salt-and-pepper noise?**
4. **How does bilateral filtering preserve edges while smoothing?**
5. **What are the common border handling strategies?**

---

[Previous: ../01-Introduction-to-CV/05-images-as-matrices.md](../01-Introduction-to-CV/05-images-as-matrices.md) | [Next: 02-convolution-operations.md](02-convolution-operations.md)
