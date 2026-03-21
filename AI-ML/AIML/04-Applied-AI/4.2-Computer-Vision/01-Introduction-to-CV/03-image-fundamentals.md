# Image Fundamentals

## Overview

An image is a 2D function f(x, y) where x and y are spatial coordinates and f is the intensity value at that point. Understanding how images are represented, stored, and manipulated at the pixel level is the foundation of all computer vision.

---

## Digital Image Representation

```
Analog image (continuous)        Digital image (discrete)
  f(x, y) → continuous           f[i, j] → discrete pixels

  Digitization involves:
    1. Sampling:    Discretize spatial coordinates (x,y) → (i,j)
    2. Quantization: Discretize intensity values → 0 to 255

  Resolution = number of pixels
    640 × 480  = 307,200 pixels (VGA)
    1920 × 1080 = 2,073,600 pixels (Full HD)
    3840 × 2160 = 8,294,400 pixels (4K)
```

---

## Image Types

```
1. Binary Image (1-bit):
   ┌───┬───┬───┬───┐
   │ 0 │ 1 │ 1 │ 0 │    Values: 0 (black) or 1 (white)
   ├───┼───┼───┼───┤    Use: Document scanning, masks
   │ 1 │ 1 │ 1 │ 1 │
   ├───┼───┼───┼───┤
   │ 0 │ 1 │ 1 │ 0 │
   └───┴───┴───┴───┘

2. Grayscale Image (8-bit):
   ┌────┬────┬────┬────┐
   │  0 │ 64 │128 │255 │   Values: 0 (black) to 255 (white)
   ├────┼────┼────┼────┤   Channels: 1
   │ 32 │ 96 │192 │220 │   Shape: (H, W)
   └────┴────┴────┴────┘

3. Color Image (24-bit):
   ┌───────────┬───────────┬───────────┐
   │ R channel │ G channel │ B channel │   3 channels × 8 bits = 24 bits
   │ (H × W)   │ (H × W)   │ (H × W)   │   Shape: (H, W, 3)
   └───────────┴───────────┴───────────┘   16.7 million colors
```

---

## Pixel Coordinates

```
Origin (0,0) is at TOP-LEFT:

  (0,0) ────── x (columns) ──────→
    │
    │    ┌────┬────┬────┬────┐
    │    │0,0 │0,1 │0,2 │0,3 │
  y │    ├────┼────┼────┼────┤
(rows)   │1,0 │1,1 │1,2 │1,3 │
    │    ├────┼────┼────┼────┤
    │    │2,0 │2,1 │2,2 │2,3 │
    ↓    └────┴────┴────┴────┘

  Access: image[row, col] or image[y, x]
  
  Note: In NumPy/OpenCV, it's image[y, x] (row-first)
        In some libraries, it's image[x, y] (column-first)
```

---

## Working with Images in Python

```python
import cv2
import numpy as np

# Read image
img = cv2.imread("photo.jpg")           # BGR format (OpenCV default)
gray = cv2.imread("photo.jpg", 0)        # Grayscale

# Image properties
print(f"Shape: {img.shape}")             # (height, width, channels)
print(f"Dtype: {img.dtype}")             # uint8 (0-255)
print(f"Size:  {img.size}")              # total pixels × channels
print(f"Resolution: {img.shape[1]}×{img.shape[0]}")  # width × height

# Access a pixel
pixel = img[100, 200]                    # [B, G, R] values
print(f"Pixel at (100,200): B={pixel[0]}, G={pixel[1]}, R={pixel[2]}")

# Modify a pixel
img[100, 200] = [255, 0, 0]             # Set to blue

# Region of Interest (ROI)
roi = img[50:150, 100:300]               # Crop: rows 50-150, cols 100-300

# Create blank images
black = np.zeros((480, 640, 3), dtype=np.uint8)     # Black
white = np.ones((480, 640, 3), dtype=np.uint8) * 255 # White

# Resize
resized = cv2.resize(img, (320, 240))    # (width, height)

# Save
cv2.imwrite("output.jpg", img)
```

---

## Image File Formats

| Format | Compression | Lossless | Transparency | Use Case |
|--------|:-:|:-:|:-:|----------|
| JPEG | Yes | No | No | Photos, web |
| PNG | Yes | Yes | Yes | Screenshots, graphics |
| BMP | No | Yes | No | Raw storage |
| TIFF | Optional | Yes | Optional | Medical, satellite |
| WebP | Yes | Both | Yes | Web (modern) |

---

## Revision Questions

1. **What are sampling and quantization in image digitization?**
2. **What is the shape of a 1080p color image as a NumPy array?**
3. **Why is the origin at the top-left in image coordinates?**
4. **How do you access pixel (x=200, y=100) in OpenCV?**
5. **What is the difference between JPEG and PNG compression?**

---

[Previous: 02-applications.md](02-applications.md) | [Next: 04-color-spaces.md](04-color-spaces.md)
