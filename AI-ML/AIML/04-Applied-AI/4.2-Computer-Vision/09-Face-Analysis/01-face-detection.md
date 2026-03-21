# Face Detection

## Overview

**Face detection** locates all faces in an image, outputting bounding boxes (and sometimes landmarks) for each face. It is the first step in any face analysis pipeline — recognition, verification, emotion analysis, and more all depend on accurately detecting faces first.

---

## Classical: Viola-Jones (2001)

```
The breakthrough that made real-time face detection possible

Components:
  1. Haar-like features (simple rectangular filters)
  
     ┌────┬────┐   ┌──┬──┬──┐   ┌────────┐
     │ ██ │    │   │██│  │██│   │  ████  │
     │ ██ │    │   │██│  │██│   ├────────┤
     └────┴────┘   └──┴──┴──┘   │        │
     Edge feature  Line feature  └────────┘
                                 Four-rectangle

  Feature value = Σ(white pixels) - Σ(dark pixels)
  
  Faces have consistent patterns:
    Eyes darker than forehead: ┌──────┐
    Nose bridge lighter:       │██  ██│ eyes
                               │  ██  │ nose
                               │ ████ │ mouth
                               └──────┘

  2. Integral Image — compute any rectangular sum in O(1)
  
     I(x,y) = Σ_{x'≤x, y'≤y} pixel(x', y')
     
     Sum of rectangle = I(D) - I(B) - I(C) + I(A)
     ┌───┬───────┬───┐
     │ A │       │ B │
     ├───┼───────┼───┤
     │   │ RECT  │   │
     │ C │       │ D │
     └───┴───────┴───┘

  3. AdaBoost — select best features from 160,000+ candidates
     Each round picks the feature with lowest weighted error
     ~200 features sufficient for face detection
     
  4. Cascade classifier — early rejection
  
     ┌─────┐  pass  ┌─────┐  pass  ┌─────┐  pass
     │ 2   │──────▶│ 10  │──────▶│ 25  │──────▶ ... → FACE
     │feat │       │feat │       │feat │
     └──┬──┘       └──┬──┘       └──┬──┘
        │fail         │fail         │fail
        ▼             ▼             ▼
     NOT FACE      NOT FACE      NOT FACE
     
     99% of windows rejected in first 2 stages!
     → Real-time on 2001 hardware (15 FPS)
```

---

## Deep Learning Detectors

### MTCNN (Multi-Task Cascaded CNN, 2016)

```
Three-stage cascade:

  Image → Resize to multiple scales (image pyramid)
  
  Stage 1: P-Net (Proposal Network)
    12×12 shallow CNN → face/not-face + bbox
    Runs on all scales → generates candidates
    Fast, low accuracy
  
  Stage 2: R-Net (Refine Network)  
    24×24 CNN → refine candidates
    Removes most false positives
  
  Stage 3: O-Net (Output Network)
    48×48 CNN → final bbox + 5 landmarks
    (left eye, right eye, nose, left mouth, right mouth)
  
  ┌──────┐     ┌──────┐     ┌──────┐
  │P-Net │ → │R-Net │ → │O-Net │ → Faces + Landmarks
  │12×12 │     │24×24 │     │48×48 │
  │~1000 │     │~100  │     │~10   │
  │cands │     │cands │     │faces │
  └──────┘     └──────┘     └──────┘
  
  Speed: ~30 FPS, very reliable
  Still widely used as the de facto face detector
```

### RetinaFace (2020)

```
State-of-the-art face detector

  Based on RetinaNet with face-specific modifications:
  
  1. Single-stage, multi-scale detection (FPN)
  2. Multi-task learning:
     - Face classification (face/not-face)
     - Bounding box regression
     - 5-point landmark regression
     - 3D face mesh prediction (self-supervised)
  
  3. Dense anchors at multiple FPN levels
  
  Performance:
    WIDER FACE Easy:   96.9% AP
    WIDER FACE Medium: 96.1% AP
    WIDER FACE Hard:   91.4% AP
  
  Handles: tiny faces, profile views, occlusion, crowds
```

### BlazeFace (2019, Google/MediaPipe)

```
Designed for mobile/edge devices

  Key innovations:
  - Extremely lightweight backbone (< 100K params)
  - SSD-style detection with 6 anchor configurations
  - Predicts 6 keypoints per face (not just bbox)
  
  Speed: 200+ FPS on mobile GPU
  Accuracy: Slightly lower than MTCNN/RetinaFace
  
  Used in MediaPipe Face Detection pipeline
```

---

## Python: Face Detection

```python
# Method 1: MTCNN
from mtcnn import MTCNN
import cv2

detector = MTCNN()
img = cv2.imread("group_photo.jpg")
img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

faces = detector.detect_faces(img_rgb)
for face in faces:
    x, y, w, h = face['box']
    conf = face['confidence']
    landmarks = face['keypoints']
    
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)
    for name, (lx, ly) in landmarks.items():
        cv2.circle(img, (lx, ly), 3, (255, 0, 0), -1)
    print(f"Face at ({x},{y}), conf={conf:.3f}")


# Method 2: MediaPipe (BlazeFace)
import mediapipe as mp

mp_face = mp.solutions.face_detection
detector = mp_face.FaceDetection(
    model_selection=1,    # 0=short-range, 1=full-range
    min_detection_confidence=0.5
)

results = detector.process(img_rgb)
if results.detections:
    for det in results.detections:
        bbox = det.location_data.relative_bounding_box
        print(f"Face: x={bbox.xmin:.2f}, y={bbox.ymin:.2f}")


# Method 3: OpenCV Haar Cascade (classical)
face_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + 'haarcascade_frontalface_default.xml')
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
faces = face_cascade.detectMultiScale(gray, 1.3, 5)
```

---

## Face Detection Benchmarks

| Benchmark | # Images | Difficulty | Key Challenge |
|-----------|----------|------------|---------------|
| FDDB | 2,845 | Medium | Unconstrained |
| WIDER FACE | 32,203 | Hard | Scale, pose, occlusion |
| MAFA | 35,806 | Hard | Masked faces |
| DARK FACE | 10,000 | Hard | Low-light conditions |

---

## Revision Questions

1. **How does the Viola-Jones cascade achieve real-time detection?**
2. **What are Haar-like features and how does the integral image speed them up?**
3. **How does MTCNN's three-stage cascade refine face detections?**
4. **What makes RetinaFace state-of-the-art for face detection?**
5. **When would you choose BlazeFace over MTCNN?**

---

[Next: 02-facial-landmarks.md](02-facial-landmarks.md)
