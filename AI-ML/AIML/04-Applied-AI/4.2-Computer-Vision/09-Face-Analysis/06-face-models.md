# Face Models: DeepFace, FaceNet, ArcFace

## Overview

This chapter provides a comparative deep dive into three landmark face recognition models that defined the field. Each introduced fundamental innovations: **DeepFace** proved deep learning works for faces, **FaceNet** pioneered direct metric learning, and **ArcFace** achieved state-of-the-art with angular margin losses.

---

## DeepFace (Facebook, 2014)

```
Taigman et al. — "Closing the Gap to Human-Level Performance"

Architecture:
  Input: 152×152 aligned face (3D alignment!)
  
  ┌──────────────────────────────────────────────┐
  │ Conv 11×11×32 → Pool → Conv 9×9×16 →        │
  │ Local Conv 9×9×16 → Local Conv 7×7×16 →      │
  │ Local Conv 5×5×16 → FC 4096 → FC 4096        │
  └──────────────────────────────────────────────┘

  Unique features:
  1. 3D Face Alignment:
     - Detect 6 fiducial points
     - Fit 3D face model
     - Frontalize face (remove pose variation)
     
     Profile view    →    Frontalized
     ┌──────────┐        ┌──────────┐
     │     ●    │   →    │  ●   ●   │
     │     ●    │        │    ●     │
     │    ●●    │        │  ●   ●   │
     └──────────┘        └──────────┘

  2. Locally connected layers (not shared weights)
     Different facial regions need different features
     
  3. Training: Softmax on 4,030 identities (4.4M images)
     → Facebook's internal dataset

  Results:
    LFW: 97.35% (vs human 97.53%)
    First to approach human performance!
    
  Limitations:
    - 3D alignment is complex and slow
    - Locally connected layers = many parameters
    - Not open-source initially
```

---

## FaceNet (Google, 2015)

```
Schroff et al. — "A Unified Embedding for Face Recognition and Clustering"

Architecture:
  Input: 96×96 or 224×224 face
  
  ┌──────────────────────────────────────────────┐
  │ GoogLeNet / Inception backbone                │
  │      ↓                                        │
  │ FC → L2 normalize → 128-D embedding           │
  └──────────────────────────────────────────────┘
  
  Key innovation: Direct metric learning (no classification!)
  
  Triplet Loss:
    Select (anchor, positive, negative) triplets
    
    L = Σ max(0, ||f(a)-f(p)||² - ||f(a)-f(n)||² + α)
    
    α = 0.2 (margin)
    
  Triplet selection strategy:
    ┌─────────────────────────────────────────┐
    │ Semi-hard negatives:                     │
    │                                          │
    │  ●anchor    ●positive                    │
    │        ↕ d(a,p)                          │
    │                                          │
    │                    ■ semi-hard negative   │
    │                    d(a,p) < d(a,n) < d(a,p)+α
    │                                          │
    │  × easy negative (too far, no gradient)  │
    │  ✦ hard negative (too close, noisy)      │
    └─────────────────────────────────────────┘
    
    Online mining: Select from mini-batch
    → Much more efficient than offline mining

  Training:
    200M face images, 8M identities (Google internal)
    Trained with SGD, 1000-2000 hours on CPU cluster
    
  Results:
    LFW: 99.63%
    YouTube Faces: 95.12%

  Key properties of learned embeddings:
    - L2-normalized (lie on unit hypersphere)
    - Euclidean distance = dissimilarity
    - Same person: d < 1.0 (typically)
    - Different person: d > 1.2 (typically)
    - Threshold: ~1.1 for verification
    
  Applications beyond recognition:
    - Face clustering (group unknown faces by identity)
    - Face retrieval (find similar faces in database)
```

---

## ArcFace (InsightFace, 2019)

```
Deng et al. — "Additive Angular Margin Loss"

Architecture:
  Input: 112×112 aligned face
  
  ┌──────────────────────────────────────────────┐
  │ ResNet-50 / ResNet-100 backbone               │
  │      ↓                                        │
  │ BN → FC → BN → 512-D embedding                │
  │      ↓                                        │
  │ ArcFace Head (during training only)            │
  └──────────────────────────────────────────────┘

  ArcFace Loss — step by step:
  
  1. L2-normalize features f and weights W:
     f̃ = f / ||f||
     W̃ = W / ||W||
  
  2. Compute cosine similarities:
     cos(θ_j) = W̃_j^T × f̃
  
  3. For the target class y, add angular margin:
     cos(θ_y + m)    where m = 0.5 radians
  
  4. Scale and apply softmax:
     L = -log(exp(s × cos(θ_y + m)) / 
              (exp(s × cos(θ_y + m)) + Σ_{j≠y} exp(s × cos(θ_j))))
     
     s = 64 (scale factor)

  Why angular margin works:
  
  Without margin:              With ArcFace (m=0.5):
  
  Class boundary is just       Must push features further
  where cosines are equal      from boundary by angle m
  
     W₁                           W₁
     ↑    ● class 1               ↑  ● class 1
     │   ●                        │ ●
     │  ●                         │●
     │────── boundary ──          │──┐ margin m
     │  ●                         │  └──── boundary
     │   ●                        │    ●
     ↓    ● class 2               │     ● class 2
     W₂                           W₂
  
  → Larger inter-class gap → more discriminative embeddings

  Training data: MS1MV2 (5.8M images, 85K identities)
  Cleaned version of MS-Celeb-1M
  
  Results:
    LFW:     99.83%
    CFP-FP:  98.27%
    AgeDB:   98.28%
    MegaFace (Rank-1): 98.35%
```

---

## Head-to-Head Comparison

| Feature | DeepFace | FaceNet | ArcFace |
|---------|:---:|:---:|:---:|
| Year | 2014 | 2015 | 2019 |
| Organization | Facebook | Google | Imperial College |
| Backbone | Custom CNN | GoogLeNet | ResNet-50/100 |
| Embedding dim | 4096 | 128 | 512 |
| Loss | Softmax | Triplet | ArcFace (angular margin) |
| Training data | 4.4M / 4K ids | 200M / 8M ids | 5.8M / 85K ids |
| LFW accuracy | 97.35% | 99.63% | 99.83% |
| Alignment | 3D frontalization | 2D affine | 2D similarity |
| Open source | No (initially) | Partial | Yes (InsightFace) |
| Key contribution | First deep face rec | Metric learning | Angular margin loss |
| Speed (inference) | Slow | Medium | Fast |

---

## Modern Landscape

```
Beyond these three, the field has evolved:

  AdaFace (2022):   Adaptive margin based on image quality
                    Easy images → larger margin
                    Hard images → smaller margin
                    Better on low-quality/surveillance data

  MagFace (2021):   Use magnitude of feature vector as quality indicator
                    Pull high-quality faces to class center
                    Allow low-quality faces more freedom

  ElasticFace (2022): Random margin for regularization
                      Prevents overfitting to specific margin value

  PartialFC (2022): Efficient training with partial class centers
                    Enables training with 10M+ identities
                    Only sample subset of class weights per batch

  Best current recipe:
    Backbone: ResNet-100 or ViT
    Loss: ArcFace or AdaFace
    Data: WebFace42M (42M images, 2M identities)
    Embedding: 512-D
    Result: 99.8%+ on all standard benchmarks
```

---

## Python: Using All Three

```python
from deepface import DeepFace

# Compare using different models
models = ["VGG-Face", "Facenet", "ArcFace"]

for model_name in models:
    result = DeepFace.verify(
        img1_path="person1.jpg",
        img2_path="person2.jpg",
        model_name=model_name
    )
    print(f"\n{model_name}:")
    print(f"  Verified: {result['verified']}")
    print(f"  Distance: {result['distance']:.4f}")
    print(f"  Threshold: {result['threshold']:.4f}")


# Using InsightFace (ArcFace) directly
from insightface.app import FaceAnalysis

app = FaceAnalysis(name='buffalo_l',
                    providers=['CPUExecutionProvider'])
app.prepare(ctx_id=0, det_size=(640, 640))

import cv2
img = cv2.imread("face.jpg")
faces = app.get(img)

for face in faces:
    print(f"Embedding shape: {face.embedding.shape}")
    print(f"Detection score: {face.det_score:.3f}")
    print(f"Age: {face.age}")
    print(f"Gender: {'Male' if face.gender == 1 else 'Female'}")
```

---

## Revision Questions

1. **What was DeepFace's key innovation and why was 3D alignment important?**
2. **How does FaceNet's triplet loss differ from classification-based training?**
3. **What is the geometric intuition behind ArcFace's angular margin?**
4. **Why does ArcFace outperform FaceNet despite using less training data?**
5. **How has AdaFace improved upon ArcFace for low-quality images?**
6. **Compare the three models in terms of embedding dimensions, training data, and accuracy.**

---

[Previous: 05-emotion-recognition.md](05-emotion-recognition.md) | [Back to README](../README.md)
