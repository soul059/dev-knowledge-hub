# What is Computer Vision?

## Overview

Computer Vision (CV) is a field of artificial intelligence that enables computers to interpret and understand visual information from the world — images, videos, and 3D scenes. It bridges the gap between raw pixel data and high-level semantic understanding, allowing machines to "see."

---

## Human Vision vs Computer Vision

```
Human Vision:
  Light → Eye (retina) → Optic nerve → Brain (visual cortex) → Understanding
  - Instant object recognition
  - 3D depth perception
  - Context and common sense
  - ~100 milliseconds for recognition

Computer Vision:
  Light → Camera (sensor) → Pixel array → Algorithm → Understanding
  - Processes numbers (0-255)
  - No inherent understanding
  - Must learn from millions of examples
  - Can process thousands of images per second

What humans see:          What computers see:
  ┌──────────────┐         ┌──────────────────────┐
  │              │         │ 142  145  148  150    │
  │   🐱 Cat     │         │ 139  143  147  151    │
  │              │         │ 135  140  145  149    │
  │              │         │ 130  136  142  148    │
  └──────────────┘         └──────────────────────┘
  Meaning                  Numbers (pixel intensities)
```

---

## The CV Pipeline

```
Raw Image → Preprocessing → Feature Extraction → Model → Prediction
   │            │                  │                │         │
   │         Resize,           Edges, corners,   CNN,      Class label,
   │         normalize,        textures,         SVM,      bounding box,
   │         augment           embeddings        etc.      segmentation mask
   │
   ├── Classical CV: Hand-crafted features + ML classifier
   │
   └── Deep Learning CV: End-to-end learning (pixels → prediction)
```

---

## Core Tasks

```
┌─────────────────────────────────────────────────────────────┐
│                    Computer Vision Tasks                     │
├──────────────────┬──────────────────────────────────────────┤
│ Classification   │ "What is in this image?" → cat           │
│ Detection        │ "Where are objects?" → bounding boxes    │
│ Segmentation     │ "Which pixels belong to what?" → masks   │
│ Tracking         │ "Where did it go?" → trajectory          │
│ Recognition      │ "Who is this?" → identity                │
│ Generation       │ "Create an image" → new image            │
│ Estimation       │ "What's the 3D structure?" → depth/pose  │
└──────────────────┴──────────────────────────────────────────┘
```

---

## Brief History

```
Timeline:
  1960s  │  Block world: edge detection, simple shapes
  1970s  │  Marr's computational theory of vision
  1980s  │  Stereo vision, optical flow
  1990s  │  Face detection (Viola-Jones), feature descriptors
  2000s  │  SIFT, HOG, deformable parts models
  2012   │  AlexNet wins ImageNet → deep learning revolution
  2014   │  GoogLeNet, VGG → deeper networks
  2015   │  ResNet (152 layers), Faster R-CNN
  2017   │  Mask R-CNN, Transformers enter vision
  2020   │  Vision Transformers (ViT)
  2022+  │  Foundation models (SAM, CLIP, DALL-E)
```

---

## Classical vs Deep Learning CV

| Aspect | Classical CV | Deep Learning CV |
|--------|:-:|:-:|
| Features | Hand-crafted (SIFT, HOG) | Learned automatically |
| Data needed | Small | Large (thousands+) |
| Interpretability | High | Lower |
| Compute | CPU sufficient | GPU required |
| Flexibility | Task-specific | General-purpose |
| Performance | Good for simple tasks | State-of-the-art |

---

## Revision Questions

1. **What is computer vision and how does it differ from human vision?**
2. **What are the main tasks in computer vision?**
3. **How did AlexNet (2012) change the field?**
4. **What is the difference between classical and deep learning approaches to CV?**
5. **Name three real-world applications of computer vision.**

---

[Next: 02-applications.md](02-applications.md)
