# Video Understanding

## Overview

**Video understanding** extends image-based CV to the temporal domain — analyzing sequences of frames to understand actions, activities, events, and temporal relationships. Key tasks include action recognition, temporal action detection, video captioning, and video question answering.

---

## Video vs Image

```
Image: Single snapshot          Video: Temporal sequence
┌──────────┐                   ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│          │                   │ t=0  │ │ t=1  │ │ t=2  │ │ t=3  │
│  🏃     │                   │ 🧑  │ │ 🏃  │ │ 🏃  │ │ 🏃  │
│          │                   │ ↓    │ │  ↓   │ │   ↓  │ │    ↓ │
└──────────┘                   └──────┘ └──────┘ └──────┘ └──────┘

  Image: "Person"               Video: "Person is running"
  
  Temporal info adds:
    - Motion (what's moving and how)
    - Actions (running, jumping, waving)
    - Causality (because X happened, then Y)
    - Long-term context (cooking a meal, playing a game)
```

---

## Action Recognition

```
Classify a video clip into an action category:

  Video clip (T frames) → Model → Action label

  Approaches:

  1. Two-Stream Networks (Simonyan & Zisserman, 2014):
     ┌──────────┐
     │ RGB      │ → Spatial CNN  → ┐
     │ frames   │                  ├→ Fusion → Action
     │ Optical  │ → Temporal CNN → ┘
     │ flow     │
     └──────────┘
     
     Spatial stream: appearance (what objects)
     Temporal stream: motion (optical flow)
     Late fusion: average or FC combination

  2. 3D CNNs — Convolve in space AND time:
     
     2D Conv: H × W filter         3D Conv: T × H × W filter
     Processes one frame            Processes T frames together
     
     C3D (2015):  VGG-like with 3D convolutions (3×3×3)
     I3D (2017):  Inflate 2D Inception to 3D
                  Initialize 3D weights from 2D ImageNet weights!
     
     I3D inflation trick:
       2D weight W_2d [k×k]
       3D weight W_3d [t×k×k] = W_2d / t  (copy and normalize)
       → Preserves pre-trained knowledge!

  3. SlowFast Networks (2019):
     ┌────────────┐
     │ Slow path  │  Low frame rate (e.g., 4 fps)
     │ (spatial)  │  High channels, captures appearance
     ├────────────┤
     │ Fast path  │  High frame rate (e.g., 32 fps)
     │ (temporal) │  Low channels, captures motion
     └────────────┘
     
     Lateral connections fuse slow ↔ fast
     → Best of both worlds
```

---

## Video Transformers

```
TimeSformer (2021):
  Extend ViT to video with different attention patterns:
  
  Divided attention:
    1. Temporal attention: each patch attends across frames
    2. Spatial attention: patches attend within each frame
    
  ┌────────┐ ┌────────┐ ┌────────┐
  │ Frame 1│ │ Frame 2│ │ Frame 3│
  │ ●──────┼─┼──●─────┼─┼──●    │  Temporal: same position
  │        │ │        │ │        │           across frames
  │ ●──●   │ │ ●──●   │ │ ●──●  │  Spatial: within frame
  └────────┘ └────────┘ └────────┘

ViViT (2021):
  Video Vision Transformer
  Factorized: spatial encoder → temporal encoder
  
VideoMAE (2022):
  Masked autoencoder for video pre-training
  Mask 90% of video patches → reconstruct
  → Excellent video features with self-supervised learning

Video Swin Transformer (2022):
  Shifted window attention extended to 3D (space-time)
```

---

## Temporal Action Detection

```
Find WHEN actions happen in untrimmed videos:

  Input: Long untrimmed video
  Output: (start_time, end_time, action_class, confidence)

  ┌──────────────────────────────────────────────┐
  │  Background | Running | Background | Jumping │
  │  0s     10s   15s  25s   25s   35s  35s  42s │
  └──────────────────────────────────────────────┘
  
  Results:
    (15s, 25s, "Running", 0.95)
    (35s, 42s, "Jumping", 0.88)

  Approaches:
    Anchor-based:  Generate temporal proposals → classify
    ActionFormer (2022): Transformer-based, single-stage
    
  Datasets:
    ActivityNet: 200 classes, 20K videos
    THUMOS14: 20 classes, untrimmed
```

---

## Optical Flow

```
Per-pixel motion between consecutive frames:

  Frame t              Frame t+1            Optical Flow
  ┌──────────┐        ┌──────────┐         ┌──────────┐
  │    ●     │        │       ●  │         │    →→→   │
  │          │   →    │          │    =    │          │
  │  ●       │        │    ●     │         │  →→      │
  └──────────┘        └──────────┘         └──────────┘

  Flow field: (u, v) per pixel = horizontal, vertical displacement

  Classical: Lucas-Kanade, Horn-Schunck, Farneback
  Deep learning:
    FlowNet (2015): CNN for optical flow
    RAFT (2020):    Recurrent All-Pairs Field Transforms
                    State-of-the-art, iterative refinement
    FlowFormer (2022): Transformer-based
    
  Use in video understanding:
    - Input to temporal stream (two-stream networks)
    - Motion feature for action recognition
    - Video stabilization
    - Object tracking
```

---

## Video Captioning & QA

```
Video Captioning:
  Video → Encoder → Decoder → "A person is riding a bicycle in the park"
  
  Encoder: 3D CNN or Video Transformer → features
  Decoder: Language model (GPT/LSTM) → text
  
Video Question Answering (VideoQA):
  Video + "What is the person doing?" → "Cooking"
  
  Requires:
    - Visual understanding (what's happening)
    - Temporal reasoning (sequence of events)
    - Language understanding (what's being asked)
  
  Models:
    MERLOT (2021): Pre-trained video+language model
    VideoChat (2023): Video-based chatbot
    Video-LLaVA (2024): Multimodal LLM for video
```

---

## Python: Video Action Recognition

```python
import torch
from torchvision.models.video import r3d_18, R3D_18_Weights

# Load pre-trained 3D ResNet-18
weights = R3D_18_Weights.DEFAULT
model = r3d_18(weights=weights)
model.eval()

# Prepare video clip (T×C×H×W)
preprocess = weights.transforms()

import cv2
frames = []
cap = cv2.VideoCapture("action.mp4")
for _ in range(16):  # sample 16 frames
    ret, frame = cap.read()
    if ret:
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        frames.append(frame)
cap.release()

# Stack frames: T×H×W×C → T×C×H×W
import numpy as np
from torchvision import transforms

transform = transforms.Compose([
    transforms.ToPILImage(),
    transforms.Resize(128),
    transforms.CenterCrop(112),
    transforms.ToTensor(),
])

clips = torch.stack([transform(f) for f in frames])  # T×C×H×W
clips = clips.permute(1, 0, 2, 3).unsqueeze(0)  # 1×C×T×H×W

with torch.no_grad():
    output = model(clips)
    pred = output.argmax(1).item()
    
print(f"Predicted action: {weights.meta['categories'][pred]}")
```

---

## Key Datasets

| Dataset | Classes | Videos | Duration | Task |
|---------|---------|--------|----------|------|
| Kinetics-400 | 400 | 300K | 10s clips | Action recognition |
| UCF-101 | 101 | 13K | ~7s clips | Action recognition |
| ActivityNet | 200 | 20K | Untrimmed | Action detection |
| Something-Something | 174 | 220K | 2-6s | Fine-grained |
| Ego4D | 3600 hrs | 3K hrs | Hours | Egocentric |

---

## Revision Questions

1. **How do two-stream networks combine spatial and temporal information?**
2. **What is the I3D inflation trick and why is it effective?**
3. **How does SlowFast use different frame rates for video understanding?**
4. **What is optical flow and how is it used in action recognition?**
5. **How do video transformers extend image transformers to the temporal domain?**

---

[Previous: 04-image-generation.md](04-image-generation.md) | [Next: 06-self-supervised-cv.md](06-self-supervised-cv.md)
