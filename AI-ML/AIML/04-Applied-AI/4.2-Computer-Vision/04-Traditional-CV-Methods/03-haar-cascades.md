# HAAR Cascades

## Overview

Haar cascades (Viola-Jones, 2001) was the first real-time face detection framework. It uses simple rectangular features computed via integral images, a cascaded classifier trained with AdaBoost, and achieves detection at ~15 FPS — revolutionary for its time. OpenCV ships with pre-trained Haar cascade classifiers for faces, eyes, smiles, and more.

---

## Haar-like Features

```
Simple rectangular filters that capture intensity patterns:

  Edge features:        Line features:        Diagonal:
  ┌─────┬─────┐        ┌─────┬─────┬─────┐   ┌─────┬─────┐
  │█████│     │        │█████│     │█████│   │█████│     │
  │█████│     │        │█████│     │█████│   │     │█████│
  └─────┴─────┘        └─────┴─────┴─────┘   └─────┴─────┘

  Feature value = Σ(white pixels) - Σ(black pixels)

  Key insight: Faces have consistent patterns
    - Eyes darker than forehead → horizontal edge feature
    - Nose bridge lighter than eyes → vertical line feature
    - Eye region darker than cheeks → diagonal features

  Total possible features in 24×24 window: ~160,000!
  AdaBoost selects the best ~200 features
```

---

## Integral Image for Speed

```
Any Haar feature = sum of 2-4 rectangle sums
Each rectangle sum = 4 lookups in integral image
→ ANY feature computed in constant time O(1)!

  Rectangle sum using integral image:
    ┌───────────┐
    │ A ──── B  │
    │ │      │  │    Sum(rectangle) = II(D) - II(B) - II(C) + II(A)
    │ C ──── D  │
    └───────────┘
    Just 3 additions regardless of rectangle size
```

---

## Cascade Classifier

```
Key idea: Reject easy negatives early (most windows are NOT faces)

  Input window (24×24)
       │
       ▼
  ┌──────────┐
  │ Stage 1   │  2 features → rejects 50% of non-faces
  │ (simple)  │  passes 100% of faces
  └────┬──────┘
       │ pass?
       ▼
  ┌──────────┐
  │ Stage 2   │  10 features → rejects another 80%
  │ (medium)  │  passes 100% of faces
  └────┬──────┘
       │ pass?
       ▼
  ┌──────────┐
  │ Stage 3   │  25 features → rejects more
  └────┬──────┘
       ...
  ┌──────────┐
  │ Stage 25  │  ~200 total features used
  │ (complex) │
  └────┬──────┘
       │ pass all stages?
       ▼
    FACE DETECTED!

  Result: ~95% of windows rejected by stage 2
          → Only ~5% need full evaluation
          → Very fast!
```

---

## Implementation

```python
import cv2

img = cv2.imread("group_photo.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Load pre-trained cascade
face_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + "haarcascade_frontalface_default.xml"
)
eye_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + "haarcascade_eye.xml"
)

# Detect faces
faces = face_cascade.detectMultiScale(
    gray,
    scaleFactor=1.1,    # image pyramid scale factor
    minNeighbors=5,     # min detections to confirm face
    minSize=(30, 30)    # minimum face size
)

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
    
    # Detect eyes within face region
    roi_gray = gray[y:y+h, x:x+w]
    roi_color = img[y:y+h, x:x+w]
    eyes = eye_cascade.detectMultiScale(roi_gray)
    for (ex, ey, ew, eh) in eyes:
        cv2.circle(roi_color, (ex+ew//2, ey+eh//2), ew//2, (0, 255, 0), 2)

print(f"Found {len(faces)} faces")
cv2.imwrite("detected.jpg", img)
```

---

## Available Cascades in OpenCV

| Cascade | Detects |
|---------|---------|
| haarcascade_frontalface_default | Frontal faces |
| haarcascade_profileface | Side-view faces |
| haarcascade_eye | Eyes |
| haarcascade_smile | Smiles |
| haarcascade_fullbody | Full body |
| haarcascade_upperbody | Upper body |
| haarcascade_frontalcatface | Cat faces |

---

## Haar vs Modern Detectors

| Feature | Haar Cascade | Deep Learning (MTCNN, RetinaFace) |
|---------|:---:|:---:|
| Speed | Very fast (CPU) | Fast (GPU needed) |
| Accuracy | Moderate (~85%) | Excellent (~99%) |
| Pose handling | Frontal only | Any angle |
| Occlusion | Poor | Good |
| Training effort | Low (pre-trained) | High |
| Best for | Simple/embedded | Production systems |

---

## Revision Questions

1. **What are Haar-like features and how are they computed?**
2. **How does the integral image enable constant-time feature computation?**
3. **What is the cascade architecture and why does it make detection fast?**
4. **What role does AdaBoost play in Haar cascade training?**
5. **Why have deep learning detectors largely replaced Haar cascades?**

---

[Previous: 02-hog.md](02-hog.md) | [Next: 04-optical-flow.md](04-optical-flow.md)
