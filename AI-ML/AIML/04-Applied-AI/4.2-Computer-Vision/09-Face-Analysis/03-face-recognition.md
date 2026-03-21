# Face Recognition

## Overview

**Face recognition** identifies WHO a person is from their face image. It maps face images to a compact embedding space where faces of the same person cluster together. Modern systems achieve superhuman accuracy on standard benchmarks.

---

## Pipeline

```
Full face recognition pipeline:

  Input Image
       │
  ┌────▼────┐
  │  Face   │  MTCNN / RetinaFace
  │Detection│  → bounding boxes
  └────┬────┘
       │
  ┌────▼────┐
  │  Face   │  Using 5 landmarks
  │Alignment│  → normalized 112×112 face
  └────┬────┘
       │
  ┌────▼────┐
  │ Feature │  Deep CNN → 512-D embedding
  │Extraction│  
  └────┬────┘
       │
  ┌────▼────┐
  │ Matching│  Compare embedding to database
  │         │  Cosine similarity or L2 distance
  └────┬────┘
       │
   Identity or "Unknown"
```

---

## Key Architectures

### DeepFace (Facebook, 2014)

```
First deep learning face recognition system

  Aligned face → 3D face alignment → CNN → 4096-D feature
  
  Training: Classification on 4.4M faces, 4K identities
  Verification: Compare features with χ² distance
  
  Result: 97.35% on LFW (near human performance: 97.53%)
  
  Significance: Showed deep learning could match humans
```

### FaceNet (Google, 2015)

```
Schroff et al. — Direct metric learning with triplet loss

  Face → GoogLeNet/Inception → 128-D embedding
  
  Training: Triplet loss on massive dataset
    L = Σ max(0, ||f(a)-f(p)||² - ||f(a)-f(n)||² + α)
    α = 0.2 margin
    
  Key innovations:
    1. Train on embeddings directly (not classification then transfer)
    2. Semi-hard negative mining
    3. L2-normalized embeddings (on hypersphere)
    
  Result: 99.63% on LFW
  
  Distance threshold for verification:
    d = ||f(face1) - f(face2)||²
    if d < threshold → same person
    if d ≥ threshold → different person
```

### ArcFace (2019, State-of-the-Art)

```
Deng et al. — Additive Angular Margin Loss

  Problem with softmax classification:
    Decision boundaries not discriminative enough
    
  ArcFace adds angular margin to softmax:
  
  Standard softmax:
    L = -log(exp(W_y^T × f) / Σ exp(W_j^T × f))
    
  ArcFace (with angular margin m):
    L = -log(exp(s×cos(θ_y + m)) / (exp(s×cos(θ_y + m)) + Σ_{j≠y} exp(s×cos(θ_j))))
    
    s = scale (typically 64)
    m = angular margin (typically 0.5 rad ≈ 28.6°)
    θ = angle between feature and class center

  Geometric interpretation:
  
       Without margin          With ArcFace margin
       ┌──────────┐            ┌──────────┐
       │  A ● ● B │            │  A●   ●B │
       │   ● ●    │            │  ●     ●  │    Wider gap
       │    │     │            │   │   │   │    between classes
       │   ● ●    │            │  ●     ●  │
       │  A ● ● B │            │  A●   ●B │
       └──────────┘            └──────────┘
       
  → Forces features to have larger angular separation
  → More discriminative embeddings
  
  Result: 99.83% on LFW, state-of-the-art on MegaFace
```

---

## Loss Function Comparison

```
Evolution of face recognition losses:

  Softmax (2014):        L = -log(softmax(W^T f))
    Simple classification, no explicit margin
    
  Center Loss (2016):    L = softmax + λ × ||f - c_y||²
    Pulls features toward class center
    Reduces intra-class variance
    
  SphereFace (2017):     L = -log(exp(||f||cos(mθ_y)) / ...)
    Multiplicative angular margin
    
  CosFace (2018):        L = -log(exp(s(cos(θ_y) - m)) / ...)
    Additive cosine margin
    
  ArcFace (2019):        L = -log(exp(s×cos(θ_y + m)) / ...)
    Additive angular margin (most popular)

  On unit hypersphere:
    SphereFace: multiply angle by m
    CosFace:   subtract m from cosine
    ArcFace:   add m to angle (most geometrically interpretable)
```

---

## Training Details

```
Dataset requirements:
  - Large-scale: MS1MV2 (5.8M images, 85K identities)
  - Cleaned: remove label noise, duplicates
  - Balanced: ensure diverse representation

Backbone: ResNet-50 or ResNet-100
  Modified with:
    - BN before last FC
    - No bias in FC
    - Feature dimension: 512

Training recipe:
  - Optimizer: SGD, momentum=0.9
  - LR: 0.1, divide by 10 at epochs [8, 12, 14]
  - Batch size: 512 (across 8 GPUs)
  - Weight decay: 5e-4
  - Scale s: 64
  - Margin m: 0.5
  
  Total training: ~24 hours on 8×V100 GPUs
```

---

## Python: Face Recognition

```python
# Method 1: face_recognition library (dlib-based)
import face_recognition

# Load known faces
known_img = face_recognition.load_image_file("known_person.jpg")
known_encoding = face_recognition.face_encodings(known_img)[0]

# Recognize in new image
test_img = face_recognition.load_image_file("test.jpg")
test_encodings = face_recognition.face_encodings(test_img)

for encoding in test_encodings:
    distance = face_recognition.face_distance(
        [known_encoding], encoding)[0]
    match = distance < 0.6  # threshold
    print(f"Distance: {distance:.3f}, Match: {match}")


# Method 2: InsightFace (ArcFace)
from insightface.app import FaceAnalysis

app = FaceAnalysis(providers=['CPUExecutionProvider'])
app.prepare(ctx_id=0)

img = cv2.imread("test.jpg")
faces = app.get(img)

for face in faces:
    embedding = face.embedding  # 512-D vector
    bbox = face.bbox
    landmarks = face.landmark_2d_106
    print(f"Face at {bbox}, embedding shape: {embedding.shape}")

# Compare two faces
import numpy as np
sim = np.dot(faces[0].embedding, faces[1].embedding) / \
      (np.linalg.norm(faces[0].embedding) * 
       np.linalg.norm(faces[1].embedding))
print(f"Similarity: {sim:.3f}")
```

---

## Benchmarks

| Benchmark | Size | Challenge | SOTA Accuracy |
|-----------|------|-----------|:---:|
| LFW | 13K images, 5749 people | Unconstrained | 99.85% |
| CFP-FP | 7000 pairs | Frontal-Profile | 99.0% |
| AgeDB-30 | 16,488 images | Age variation | 98.3% |
| MegaFace | 1M distractors | Large-scale | 98.5% |
| IJB-C | 31,334 images | Mixed media | 96.4% (TAR@FAR=1e-4) |

---

## Revision Questions

1. **How does the face recognition pipeline work end-to-end?**
2. **What is FaceNet's triplet loss and how does it learn embeddings?**
3. **How does ArcFace's angular margin improve upon softmax?**
4. **What is the geometric interpretation of angular margin losses?**
5. **How do you perform face verification using embeddings?**

---

[Previous: 02-facial-landmarks.md](02-facial-landmarks.md) | [Next: 04-verification-vs-identification.md](04-verification-vs-identification.md)
