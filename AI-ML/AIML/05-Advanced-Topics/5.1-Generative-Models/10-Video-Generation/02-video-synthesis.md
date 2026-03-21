# Video Synthesis

## Overview

Video synthesis goes beyond prediction — it generates entirely new video content from various conditioning signals such as **text prompts**, **images**, **sketches**, or **semantic maps**. Unlike video prediction (which extends existing footage), video synthesis creates novel visual sequences. This chapter covers the architectures behind modern video generation systems like **Sora**, **Runway Gen-2**, **Stable Video Diffusion**, and **Make-A-Video**.

---

## From Image Generation to Video

```
Key insight: extend image generation models to the temporal dimension

  Image generation:        Video generation:
  Generate H × W × 3      Generate T × H × W × 3

  ┌──────────────┐         ┌──────────────────────────┐
  │              │         │ Frame 1 │ Frame 2 │ ... │
  │  Single      │   →→    │         │         │     │
  │  Image       │         │ Temporally consistent    │
  │              │         │ sequence of images       │
  └──────────────┘         └──────────────────────────┘

  Challenges in extending to video:
  1. Temporal consistency (no flickering)
  2. Motion coherence (smooth movement)
  3. Much higher compute (T× more data)
  4. Training data: high-quality video datasets
  5. Long-range narrative coherence
```

---

## Architecture Approaches

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. GAN-based (early era):                                   │
  │     VGAN, TGAN, MoCoGAN, DVD-GAN                            │
  │     3D convolutions or RNN + GAN                             │
  │     Limited quality and length                               │
  │                                                                │
  │  2. Autoregressive (token-based):                            │
  │     VideoGPT, CogVideo, NÜWA                                │
  │     VQ-VAE → discrete tokens → transformer                  │
  │     Good long-range coherence, slow generation               │
  │                                                                │
  │  3. Diffusion-based (current state-of-the-art):             │
  │     Video Diffusion Models, Make-A-Video, Imagen Video       │
  │     Stable Video Diffusion, Sora                             │
  │     Best quality, most flexible conditioning                 │
  │                                                                │
  │  Progression: GANs → Autoregressive → Diffusion             │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Video Diffusion Models

```
  Extend image diffusion to 3D (spatial + temporal):

  Image U-Net:
    2D convolutions + 2D attention (spatial)
    
  Video U-Net:
    2D convolutions + temporal convolutions
    2D attention + temporal attention

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Video U-Net Block:                               │
  │                                                    │
  │  Input: (B, T, C, H, W)                          │
  │       │                                            │
  │       ▼                                            │
  │  Spatial Conv2D (per frame)                       │
  │       │                                            │
  │       ▼                                            │
  │  Temporal Conv1D (across frames)                  │
  │       │                                            │
  │       ▼                                            │
  │  Spatial Self-Attention (per frame)               │
  │       │                                            │
  │       ▼                                            │
  │  Temporal Self-Attention (across frames)          │
  │       │                                            │
  │       ▼                                            │
  │  Cross-Attention (text conditioning)              │
  │       │                                            │
  │       ▼                                            │
  │  Output: (B, T, C, H, W)                         │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Factored attention: much cheaper than full 3D attention!
  Spatial: each frame attends to itself
  Temporal: each spatial position attends across time
```

---

## Make-A-Video (Meta)

```
  Make-A-Video (Singer et al., 2022):
  
  Key idea: leverage image generation knowledge!
    1. Train text-to-image model (on image-text pairs)
    2. Add temporal layers (trained on video-only data)
    3. No need for text-video pairs!

  Pipeline:
    Text → CLIP embedding
      │
      ▼
    Prior network → image embedding
      │
      ▼
    Spatiotemporal decoder (16×64×64)
      │
      ▼
    Frame interpolation network (increase framerate)
      │
      ▼
    Spatial super-resolution (16×256×256)
      │
      ▼
    Spatiotemporal super-resolution (16×768×768)

  Temporal layers:
    • 1D convolutions across time dimension
    • Temporal attention layers
    • Initialized from image model weights
    • Only temporal layers trained on video
```

---

## Imagen Video (Google)

```
  Imagen Video (Ho et al., 2022):
  
  Cascade of 7 diffusion models!
    
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "A teddy bear washing dishes"                   │
  │       │                                            │
  │       ▼                                            │
  │  1. Base model:    5 frames × 40×24              │
  │       │                                            │
  │       ▼                                            │
  │  2. Temporal SR:   5 → 17 frames                 │
  │       │                                            │
  │       ▼                                            │
  │  3. Spatial SR:    40×24 → 80×48                 │
  │       │                                            │
  │       ▼                                            │
  │  4. Temporal SR:   17 → 65 frames                │
  │       │                                            │
  │       ▼                                            │
  │  5. Spatial SR:    80×48 → 320×192               │
  │       │                                            │
  │       ▼                                            │
  │  6. Temporal SR:   65 → 129 frames               │
  │       │                                            │
  │       ▼                                            │
  │  7. Spatial SR:    320×192 → 1280×768            │
  │                                                    │
  │  Result: ~5 sec HD video at 24fps                │
  └──────────────────────────────────────────────────┘

  Uses v-prediction parameterization
  Progressive distillation for faster sampling
```

---

## Stable Video Diffusion (Stability AI)

```
  SVD (Blattmann et al., 2023):
  
  Based on Stable Diffusion image model:
    1. Start with pretrained Stable Diffusion
    2. Add temporal layers
    3. Three-stage training:
       a) Image pretraining (existing SD weights)
       b) Video pretraining (large video dataset)
       c) Video fine-tuning (high-quality subset)

  Image-to-video generation:
    Input: single image (first frame)
    Output: 14-25 frames of video

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Reference image ──→ VAE encoder ──→ latent     │
  │                                                    │
  │  Latent + noise ──→ 3D U-Net ──→ denoised      │
  │                       ↑                            │
  │  Text/CLIP conditioning                           │
  │  Temporal attention layers                        │
  │  Motion conditioning (fps, motion amount)         │
  │                                                    │
  │  Denoised latent ──→ VAE decoder ──→ video      │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Open-source and widely used as a foundation model
```

---

## Sora (OpenAI)

```
  Sora (2024): most capable video generation model

  Key innovations:
  1. Diffusion Transformer (DiT) architecture
     - Replaces U-Net with transformer
     - Patches of spacetime (3D patches)
     - Scales better than U-Net

  2. Native variable resolution
     - Trains on videos of different sizes/durations
     - No cropping to fixed aspect ratio

  3. Very long context
     - Up to 1 minute of HD video
     - Maintains coherence throughout

  Architecture:
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Video → VAE → spacetime latent patches         │
  │                                                    │
  │  ┌──────────────────────────────────────────┐    │
  │  │  Spacetime patches (like ViT tokens)    │    │
  │  │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐       │    │
  │  │  │p_1│ │p_2│ │p_3│ │...│ │p_N│       │    │
  │  │  └───┘ └───┘ └───┘ └───┘ └───┘       │    │
  │  │       ↓                                  │    │
  │  │  DiT transformer blocks                 │    │
  │  │  (self-attention + cross-attention)      │    │
  │  │       ↓                                  │    │
  │  │  Denoised patches                       │    │
  │  └──────────────────────────────────────────┘    │
  │                                                    │
  │  Denoised patches → VAE decoder → video          │
  │                                                    │
  │  Text conditioning via T5 + CLIP                 │
  └──────────────────────────────────────────────────┘

  Capabilities: physics understanding, 3D consistency,
  camera control, character consistency
```

---

## Conditioning Modalities

```
  Video synthesis can be conditioned on various inputs:

  ┌──────────────────┬────────────────────────────────┐
  │ Conditioning     │ Description                    │
  ├──────────────────┼────────────────────────────────┤
  │ Text             │ Natural language description   │
  │                  │ → text-to-video                │
  ├──────────────────┼────────────────────────────────┤
  │ Image            │ First frame or keyframes       │
  │                  │ → image-to-video (SVD, I2VGen) │
  ├──────────────────┼────────────────────────────────┤
  │ Pose sequence    │ Human skeleton keypoints       │
  │                  │ → dance/motion generation      │
  ├──────────────────┼────────────────────────────────┤
  │ Depth maps       │ Per-frame depth information    │
  │                  │ → controlled 3D camera motion  │
  ├──────────────────┼────────────────────────────────┤
  │ Edge maps        │ Canny/sketch per frame         │
  │                  │ → structure-guided generation  │
  ├──────────────────┼────────────────────────────────┤
  │ Audio            │ Music or speech signal         │
  │                  │ → audio-reactive video         │
  ├──────────────────┼────────────────────────────────┤
  │ Video            │ Style/motion from reference    │
  │                  │ → video-to-video translation   │
  └──────────────────┴────────────────────────────────┘
```

---

## Python: Text-to-Video with Diffusers

```python
import torch
from diffusers import DiffusionPipeline
from diffusers.utils import export_to_video

# Text-to-video with ModelScope
pipe = DiffusionPipeline.from_pretrained(
    "damo-vilab/text-to-video-ms-1.7b",
    torch_dtype=torch.float16,
    variant="fp16"
)
pipe.to("cuda")
pipe.enable_model_cpu_offload()

# Generate video
prompt = "A golden retriever playing in the snow, high quality"
video_frames = pipe(
    prompt,
    num_inference_steps=50,
    num_frames=16,
).frames[0]

export_to_video(video_frames, "output.mp4", fps=8)


# Image-to-video with Stable Video Diffusion
from diffusers import StableVideoDiffusionPipeline
from diffusers.utils import load_image

pipe = StableVideoDiffusionPipeline.from_pretrained(
    "stabilityai/stable-video-diffusion-img2vid-xt",
    torch_dtype=torch.float16,
)
pipe.to("cuda")

image = load_image("reference_frame.png")
image = image.resize((1024, 576))

# Generate 25 frames from single image
frames = pipe(
    image,
    decode_chunk_size=8,
    num_frames=25,
    motion_bucket_id=127,    # controls amount of motion
    fps=7,
).frames[0]

export_to_video(frames, "svd_output.mp4", fps=7)


# AnimateDiff: add motion to any SD checkpoint
from diffusers import AnimateDiffPipeline, MotionAdapter, DDIMScheduler

adapter = MotionAdapter.from_pretrained(
    "guoyww/animatediff-motion-adapter-v1-5-3")
pipe = AnimateDiffPipeline.from_pretrained(
    "runwayml/stable-diffusion-v1-5",
    motion_adapter=adapter,
    torch_dtype=torch.float16,
)
pipe.scheduler = DDIMScheduler.from_config(pipe.scheduler.config)
pipe.to("cuda")

output = pipe(
    prompt="a cat walking on the grass, best quality",
    negative_prompt="low quality, blurry",
    num_frames=16,
    guidance_scale=7.5,
    num_inference_steps=25,
)
export_to_video(output.frames[0], "animatediff.mp4")
```

---

## Revision Questions

1. **How do video diffusion models extend image diffusion architectures?**
2. **What is the key insight behind Make-A-Video's training approach?**
3. **How does Imagen Video achieve high-resolution long videos?**
4. **What makes Sora's DiT architecture different from U-Net approaches?**
5. **What conditioning modalities can be used for video synthesis?**
6. **How does Stable Video Diffusion perform image-to-video generation?**

---

[Previous: 01-video-prediction.md](01-video-prediction.md) | [Next: 03-temporal-consistency.md](03-temporal-consistency.md)
