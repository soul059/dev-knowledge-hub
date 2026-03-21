# ControlNet

## Overview

ControlNet (Zhang & Agrawala, 2023) adds **spatial conditioning** to pretrained diffusion models, enabling precise control over the generated image structure. While text prompts control *what* to generate, ControlNet controls *how* — using edge maps, depth maps, poses, segmentation masks, and more. ControlNet preserves the pretrained model's quality while adding a trainable copy that learns the conditioning.

---

## Motivation

```
Text prompts are ambiguous about spatial layout:

  "a person standing in a park"
  → Where exactly? What pose? What angle?
  
  ControlNet provides spatial precision:
  
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Text only:       Text + ControlNet:             │
  │  "cat on table"   "cat on table" + edge map      │
  │                                                    │
  │  🐱? 🐱?           Uses your edge map to         │
  │  (random pose)     place cat EXACTLY where        │
  │                    you specified!                  │
  │                                                    │
  │  Condition types:                                 │
  │  • Canny edges        • Depth maps               │
  │  • Pose (OpenPose)    • Segmentation masks       │
  │  • Scribbles          • Normal maps               │
  │  • M-LSD lines        • Reference images         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Architecture: Trainable Copy

```
ControlNet creates a TRAINABLE COPY of the U-Net encoder:

  ┌──────────────────────────────────────────────────────┐
  │                                                        │
  │  Locked U-Net (pretrained Stable Diffusion):          │
  │  ┌─────────────────────────────────────────┐          │
  │  │ Encoder blocks → Middle → Decoder blocks │          │
  │  └─────────────────────────────────────────┘          │
  │       ↑ added features from ControlNet                │
  │                                                        │
  │  Trainable Copy (ControlNet):                         │
  │  ┌─────────────────────────────┐                      │
  │  │ Copied Encoder + Middle     │                      │
  │  │ + zero convolution outputs  │                      │
  │  └─────────────────────────────┘                      │
  │       ↑                                                │
  │  Condition input (edge map, depth, etc.)              │
  │                                                        │
  └──────────────────────────────────────────────────────┘

  Key design choices:
  1. Copy encoder blocks (not decoder)
  2. Connect via "zero convolution" layers
  3. Lock original model weights entirely
  4. Only train the copy + zero convs
```

---

## Zero Convolution

```
Zero convolution: 1×1 conv initialized to ALL ZEROS

  At training start:
    zero_conv(features) = 0   (for any input!)
    
    ControlNet output = 0
    → Original model behavior PRESERVED exactly
    → No harmful noise at initialization

  During training:
    Zero conv weights learn non-zero values gradually
    ControlNet output increases from 0 → meaningful

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Why zero init?                                   │
  │                                                    │
  │  Random init → random features added to U-Net   │
  │  → destroys pretrained model quality instantly!   │
  │                                                    │
  │  Zero init → starts as identity (no change)      │
  │  → gradually learns to add conditioning          │
  │  → pretrained quality ALWAYS preserved           │
  │                                                    │
  │  Much more stable than fine-tuning!              │
  └──────────────────────────────────────────────────┘
```

---

## Conditioning Types

```
  ┌────────────────┬──────────────────────────────────┐
  │  Condition      │  Description                     │
  ├────────────────┼──────────────────────────────────┤
  │  Canny Edge    │  Edge detection → precise outlines│
  │  Depth Map     │  MiDaS/ZoeDepth → 3D structure   │
  │  OpenPose      │  Body pose keypoints              │
  │  Segmentation  │  Semantic regions (sky, ground)   │
  │  Scribble      │  Hand-drawn rough sketch          │
  │  Normal Map    │  Surface orientation (3D lighting)│
  │  M-LSD Lines   │  Straight line segments           │
  │  SoftEdge      │  Soft/thick edges (more freedom)  │
  │  Shuffle       │  Rearranged color patches         │
  │  IP-Adapter    │  Reference image style            │
  └────────────────┴──────────────────────────────────┘

  Each ControlNet model is trained for ONE condition type
  Multiple ControlNets can be combined!
  
  Canny + Depth → shape AND 3D structure
  Pose + Depth  → person pose AND scene layout
```

---

## Multi-ControlNet

```
Combine multiple conditions simultaneously:

  ControlNet 1 (pose):    w₁ × features₁
  ControlNet 2 (depth):   w₂ × features₂
  ControlNet 3 (edges):   w₃ × features₃

  Combined = w₁×CN₁ + w₂×CN₂ + w₃×CN₃

  Add to locked U-Net decoder

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Locked U-Net                                     │
  │       ↑ w₁ × CN₁(pose)                          │
  │       ↑ w₂ × CN₂(depth)                         │
  │       ↑ w₃ × CN₃(canny)                         │
  │                                                    │
  │  Weights w control influence of each condition   │
  │  w=0: ignore, w=1: standard, w=2: double effect │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Training

```
Training ControlNet:
  
  Data: (image, condition, text_prompt) triplets
  
  Example for Canny ControlNet:
    image → Canny edge detection → condition
    (image, edges, "description of image")
  
  Loss: standard diffusion loss
    L = ||ε - ε_θ(z_t, t, c_text, c_control)||²
    
  Only ControlNet weights + zero convs are updated!
  Original SD weights stay frozen.

  Training cost: ~600 GPU hours (much less than SD itself)
  Can train on single GPU with gradient checkpointing

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Training data generation:                       │
  │                                                    │
  │  For edges:  Apply Canny to training images      │
  │  For depth:  Run MiDaS on training images        │
  │  For pose:   Run OpenPose on training images     │
  │                                                    │
  │  Automatically generated! No manual annotation!  │
  └──────────────────────────────────────────────────┘
```

---

## Python: Using ControlNet

```python
from diffusers import (StableDiffusionControlNetPipeline,
                        ControlNetModel)
import torch
import cv2
import numpy as np
from PIL import Image

# Load ControlNet for Canny edges
controlnet = ControlNetModel.from_pretrained(
    "lllyasviel/sd-controlnet-canny",
    torch_dtype=torch.float16
)

pipe = StableDiffusionControlNetPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    controlnet=controlnet,
    torch_dtype=torch.float16,
).to("cuda")

# Prepare Canny edge condition
def get_canny_edges(image_path, low=100, high=200):
    image = cv2.imread(image_path)
    edges = cv2.Canny(image, low, high)
    edges = np.stack([edges] * 3, axis=-1)  # 3 channels
    return Image.fromarray(edges)

canny_image = get_canny_edges("reference.jpg")

# Generate with ControlNet
image = pipe(
    prompt="a beautiful garden, photorealistic, detailed",
    image=canny_image,            # conditioning image
    num_inference_steps=30,
    guidance_scale=7.5,
    controlnet_conditioning_scale=1.0,  # ControlNet strength
).images[0]

# Multi-ControlNet
from diffusers import MultiControlNetModel

controlnet_canny = ControlNetModel.from_pretrained(
    "lllyasviel/sd-controlnet-canny", torch_dtype=torch.float16)
controlnet_depth = ControlNetModel.from_pretrained(
    "lllyasviel/sd-controlnet-depth", torch_dtype=torch.float16)

multi_controlnet = MultiControlNetModel([controlnet_canny, controlnet_depth])

pipe = StableDiffusionControlNetPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    controlnet=multi_controlnet,
    torch_dtype=torch.float16,
).to("cuda")

image = pipe(
    prompt="architectural rendering",
    image=[canny_image, depth_image],
    controlnet_conditioning_scale=[1.0, 0.8],
).images[0]
```

---

## Revision Questions

1. **Why does ControlNet copy the encoder instead of modifying the original?**
2. **What is zero convolution and why is it critical?**
3. **How can multiple ControlNets be combined?**
4. **What types of spatial conditions can ControlNet use?**
5. **Why is ControlNet training data easy to generate?**

---

[Previous: 04-classifier-free-guidance.md](04-classifier-free-guidance.md) | [Next: 06-inpainting-and-outpainting.md](06-inpainting-and-outpainting.md)
