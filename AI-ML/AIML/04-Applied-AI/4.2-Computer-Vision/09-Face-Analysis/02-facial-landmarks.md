# Facial Landmarks Detection

## Overview

**Facial landmarks** (also called face alignment or face shape prediction) locates key points on a face — eyes, nose tip, mouth corners, jawline, eyebrows. These points enable face alignment for recognition, expression analysis, face swapping, AR filters, and more.

---

## Landmark Models

```
68-point model (dlib):                  5-point model (MTCNN):
                                        
         ●●●●●                              ●       ●
        ●       ●  Eyebrows                Left    Right
       ●●●   ●●●  Eyes                     Eye     Eye
         ●●●                                   ●
          ●     Nose                           Nose
        ●●●●●  Mouth                       ●       ●
       ●       ●                          Left    Right
        ●●●●●●●  Jawline                 Mouth   Mouth
       ●●●●●●●●●

68 points capture:                      5 points: minimal but
  17 jawline                            sufficient for face
  10 eyebrows (5 each)                  alignment
  12 eyes (6 each)
  9 nose
  20 mouth (inner + outer)

Other models:
  98 points (WFLW benchmark)
  194 points (300W benchmark)  
  468 points (MediaPipe Face Mesh)  — full 3D face mesh!
```

---

## Classical: Active Shape Models (ASM)

```
Cootes et al. (1995)

  1. Build statistical shape model from training faces:
     - Align all training faces
     - Compute mean shape + PCA of shape variations
     
     Mean face shape:
        ●───●───●
       / ●   ● \     Shape = mean + Σ(b_i × eigenvector_i)
      ●   ●   ●     b_i = shape parameters
       \ ●●●● /     (first few PCA components capture most variation)
        ●─●─●

  2. Fitting: Start from mean shape near face
     Iteratively:
       a. For each point, search along normal for best edge
       b. Update shape parameters (constrain to valid shapes)
       c. Repeat until convergence
```

---

## Deep Learning Approaches

### Regression-Based

```
Directly predict (x, y) coordinates for all landmarks:

  Face image → CNN → FC → [x1,y1, x2,y2, ..., x68,y68]
  
  136 output values for 68 landmarks
  
  Loss = MSE or Smooth L1:
    L = Σ_i ||predicted_i - GT_i||²
  
  Often normalized by inter-ocular distance:
    NME = (1/N) Σ ||p_i - g_i|| / d_IOD
    d_IOD = distance between eye centers
    
  Simple but can be imprecise for complex poses
```

### Heatmap-Based

```
Predict one heatmap per landmark:

  Face image → CNN → K heatmaps (one per landmark)
  
  Each heatmap: Gaussian blob centered at landmark location
  
  ┌──────────────┐     ┌──────────────┐
  │              │     │    ░░░       │
  │  Left eye    │  →  │   ░███░      │  Heatmap for left eye
  │  landmark    │     │    ░░░       │  Peak = landmark position
  │              │     │              │
  └──────────────┘     └──────────────┘
  
  Heatmap ground truth: Gaussian at GT location
    h(x,y) = exp(-((x-x_gt)² + (y-y_gt)²) / (2σ²))
  
  Loss: MSE between predicted and GT heatmaps
  
  Advantages:
    - Spatial structure preserved
    - More precise than regression
    - Can estimate uncertainty (spread of heatmap)
  
  Models: Hourglass Network, HRNet, PIPNet
```

### Hourglass Network

```
Stacked hourglass architecture (Newell et al., 2016):

  ┌─────────────────────────────────────────────┐
  │         Hourglass Module                     │
  │                                              │
  │  High-res  ─────────────skip──────────────  High-res │
  │     │                                     ↑  │
  │   Pool ─────────skip────────── Upsample    │
  │     │                        ↑             │
  │   Pool ────skip──── Upsample               │
  │     │             ↑                        │
  │   Bottleneck ─────┘                         │
  │                                              │
  └─────────────────────────────────────────────┘

  Stack 2-8 hourglass modules:
    Each module refines predictions from the previous
    Intermediate supervision at each stage
    
    → Captures multi-scale features
    → Iterative refinement improves accuracy
```

### HRNet (High-Resolution Network)

```
Sun et al. (2019) — maintains high resolution throughout:

  Standard:  High-res → downsample → ... → upsample (info lost!)
  
  HRNet: Parallel branches at multiple resolutions
  
  High   ████████████████████████████████
         ↓↑    ↓↑    ↓↑    ↓↑    ↓↑
  Med    ████████████████████████████
         ↓↑    ↓↑    ↓↑    ↓↑
  Low    ██████████████████████
         ↓↑    ↓↑    ↓↑
  VLow   ████████████████

  → High-resolution features maintained throughout
  → Repeated multi-scale fusion between branches
  → State-of-the-art for landmark detection and pose estimation
```

---

## Face Alignment

```
Purpose: Normalize face geometry before recognition

  Detected face          Aligned face
  ┌──────────────┐       ┌──────────────┐
  │    ●   ●     │       │   ●     ●    │
  │      ●    /  │  →    │      ●       │  Eyes horizontal
  │   ●     ●/   │       │   ●     ●    │  Face centered
  │       /      │       │              │  Standard size
  └──────────────┘       └──────────────┘

  Steps:
  1. Detect 5 landmarks (eyes, nose, mouth corners)
  2. Compute similarity transform (rotation + scale + translation)
     to map detected landmarks to canonical positions
  3. Apply transform to warp face to standard position

  Canonical positions (112×112 face):
    Left eye:   (38.29, 51.69)
    Right eye:  (73.53, 51.69)
    Nose tip:   (56.02, 71.73)
    Left mouth: (41.54, 92.37)
    Right mouth:(70.73, 92.37)
```

---

## Python: Facial Landmarks

```python
# Method 1: dlib (68 landmarks)
import dlib
import cv2

detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor(
    "shape_predictor_68_face_landmarks.dat")

img = cv2.imread("face.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

faces = detector(gray)
for face in faces:
    landmarks = predictor(gray, face)
    for i in range(68):
        x = landmarks.part(i).x
        y = landmarks.part(i).y
        cv2.circle(img, (x, y), 2, (0, 255, 0), -1)


# Method 2: MediaPipe Face Mesh (468 3D landmarks)
import mediapipe as mp

mp_mesh = mp.solutions.face_mesh
face_mesh = mp_mesh.FaceMesh(
    static_image_mode=True,
    max_num_faces=1,
    min_detection_confidence=0.5
)

img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
results = face_mesh.process(img_rgb)

if results.multi_face_landmarks:
    for lm in results.multi_face_landmarks[0].landmark:
        x = int(lm.x * img.shape[1])
        y = int(lm.y * img.shape[0])
        z = lm.z  # depth (relative)
        cv2.circle(img, (x, y), 1, (0, 255, 0), -1)
```

---

## Evaluation Metrics

| Metric | Description |
|--------|-------------|
| NME | Normalized Mean Error = mean distance / inter-ocular distance |
| AUC | Area under CED curve (cumulative error distribution) |
| FR@threshold | Failure rate: % of faces with NME > threshold |
| PCK | Percentage of Correct Keypoints (within threshold) |

---

## Revision Questions

1. **What are facial landmarks and why are they important?**
2. **How do heatmap-based methods differ from regression-based methods?**
3. **What makes HRNet effective for landmark detection?**
4. **How are landmarks used for face alignment before recognition?**
5. **What does NME measure and why is it normalized?**

---

[Previous: 01-face-detection.md](01-face-detection.md) | [Next: 03-face-recognition.md](03-face-recognition.md)
