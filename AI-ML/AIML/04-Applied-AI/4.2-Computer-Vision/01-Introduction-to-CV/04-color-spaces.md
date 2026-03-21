# Color Spaces

## Overview

A color space defines how colors are represented numerically. Different tasks benefit from different color spaces — RGB is natural for displays, HSV separates color from brightness (great for color-based detection), and LAB approximates human perception. Understanding color spaces is essential for preprocessing and feature engineering in CV.

---

## RGB Color Space

```
RGB: Red, Green, Blue — additive color model

  Each channel: 0-255
  Total colors: 256³ = 16.7 million

  Color cube:
          Green (0,255,0)
            │
            │    Yellow (255,255,0)
            │   /
            │  /
            │ /
  Black ────┼──────── Red (255,0,0)
  (0,0,0)  /│
          / │
         /  │
  Blue ─/   │
(0,0,255)   White (255,255,255)

  Common colors:
    Red:    (255,   0,   0)
    Green:  (  0, 255,   0)
    Blue:   (  0,   0, 255)
    White:  (255, 255, 255)
    Black:  (  0,   0,   0)
    Yellow: (255, 255,   0)
    Cyan:   (  0, 255, 255)

  Note: OpenCV uses BGR order, not RGB!
```

---

## HSV Color Space

```
HSV: Hue, Saturation, Value

  H (Hue):        Color type, 0-179 in OpenCV (0-360° mapped)
  S (Saturation):  Color intensity, 0-255
  V (Value):       Brightness, 0-255

  Hue wheel (OpenCV range 0-179):
    0-10:    Red
    10-25:   Orange
    25-35:   Yellow
    35-85:   Green
    85-130:  Blue
    130-170: Purple/Violet
    170-179: Red (wraps around)

  HSV cylinder:
          ┌─────────┐  V=255 (bright)
         /    H      \
        │  (0-179°)   │
        │   around    │
        │   circle    │
        │ S: center=0 │
        │   edge=255  │
         \           /
          └─────────┘  V=0 (dark)

  Why use HSV?
    - Separate color (H) from lighting (V)
    - Easy to detect specific colors regardless of brightness
    - Great for color-based object tracking
```

---

## LAB Color Space

```
CIELAB: Lightness, A (green-red), B (blue-yellow)

  L:  Lightness     0 (black) to 100 (white)
  a:  Green(-) to Red(+)     -128 to +127
  b:  Blue(-) to Yellow(+)   -128 to +127

  Key property: Perceptually uniform
    Equal numerical distance = equal perceived color difference
    (RGB is NOT perceptually uniform)

  Use cases:
    - Color difference measurement
    - Image enhancement
    - Color transfer between images
    - Skin detection
```

---

## Color Space Conversions

```python
import cv2
import numpy as np

img = cv2.imread("photo.jpg")  # BGR by default

# BGR → RGB (for matplotlib display)
rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

# BGR → HSV
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# BGR → Grayscale
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
# Formula: Gray = 0.299R + 0.587G + 0.114B

# BGR → LAB
lab = cv2.cvtColor(img, cv2.COLOR_BGR2LAB)

# --- Color-based detection using HSV ---
# Detect red objects
lower_red = np.array([0, 120, 70])
upper_red = np.array([10, 255, 255])
mask = cv2.inRange(hsv, lower_red, upper_red)

# Apply mask
result = cv2.bitwise_and(img, img, mask=mask)
cv2.imwrite("red_objects.jpg", result)
```

---

## Color Space Comparison

| Color Space | Channels | Best For |
|-------------|----------|----------|
| RGB/BGR | Red, Green, Blue | Display, storage |
| HSV | Hue, Saturation, Value | Color detection, tracking |
| LAB | Lightness, a, b | Color difference, enhancement |
| YCrCb | Luminance, Chrominance | Skin detection, JPEG compression |
| Grayscale | Intensity | Edge detection, many CV algorithms |

---

## Practical: Color-Based Object Tracking

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)  # webcam

while True:
    ret, frame = cap.read()
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    
    # Track blue objects
    lower_blue = np.array([100, 150, 50])
    upper_blue = np.array([130, 255, 255])
    mask = cv2.inRange(hsv, lower_blue, upper_blue)
    
    # Find contours of blue regions
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, 
                                     cv2.CHAIN_APPROX_SIMPLE)
    for cnt in contours:
        if cv2.contourArea(cnt) > 500:  # filter noise
            x, y, w, h = cv2.boundingRect(cnt)
            cv2.rectangle(frame, (x,y), (x+w,y+h), (0,255,0), 2)
    
    cv2.imshow("Tracking", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

---

## Revision Questions

1. **Why does OpenCV use BGR instead of RGB?**
2. **What advantage does HSV have over RGB for color detection?**
3. **What does "perceptually uniform" mean in the LAB color space?**
4. **How would you detect yellow objects using HSV thresholds?**
5. **What is the grayscale conversion formula and why are the weights different?**

---

[Previous: 03-image-fundamentals.md](03-image-fundamentals.md) | [Next: 05-images-as-matrices.md](05-images-as-matrices.md)
