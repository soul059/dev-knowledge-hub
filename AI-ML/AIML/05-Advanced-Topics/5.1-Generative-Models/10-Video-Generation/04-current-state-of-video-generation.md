# Current State of Video Generation

## Overview

Video generation has undergone explosive progress from 2022 to 2026. What began as short, low-resolution clips with visible artifacts has evolved into minute-long, high-definition videos with remarkable visual quality. This chapter surveys the current landscape — the leading models, remaining challenges, emerging capabilities, industry applications, and the open problems that define the frontier.

---

## Timeline of Progress

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  2016  VGAN               First GAN-based video (tiny clips) │
  │  2020  DVD-GAN            Higher resolution GAN videos       │
  │  2022  Make-A-Video       Text-to-video with image priors    │
  │  2022  Imagen Video       Cascade diffusion (HD quality)     │
  │  2023  Runway Gen-2       Commercial text-to-video           │
  │  2023  Stable Video Diff. Open-source image-to-video         │
  │  2023  Pika Labs          Consumer video generation          │
  │  2024  Sora (OpenAI)      1-minute coherent video            │
  │  2024  Kling (Kuaishou)   High-quality Chinese model         │
  │  2024  Veo (Google)       1080p, 60+ seconds                 │
  │  2024  Movie Gen (Meta)   Audio + video generation           │
  │  2025  Sora (public)      Widely available text-to-video     │
  │  2025  Wan (Alibaba)      Open-source competitive model      │
  │  2025  Veo 2 (Google)     4K, photorealistic quality         │
  │  2025+ Emerging:          Real-time, interactive, 3D-aware   │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Leading Models Comparison

```
  ┌──────────────┬────────────┬──────────┬──────────┬────────────────┐
  │ Model        │ Resolution │ Duration │ Arch.    │ Key Feature    │
  ├──────────────┼────────────┼──────────┼──────────┼────────────────┤
  │ Sora         │ up to 1080p│ ~60 sec  │ DiT      │ World model    │
  │ Veo 2        │ up to 4K   │ ~60 sec  │ Diffusion│ Photorealism   │
  │ Kling        │ 1080p      │ ~10 sec  │ DiT      │ Motion quality │
  │ Movie Gen    │ 1080p      │ ~16 sec  │ Diffusion│ Audio + video  │
  │ Runway Gen-3 │ 1080p      │ ~10 sec  │ Diffusion│ Camera control │
  │ SVD          │ 1024×576   │ ~4 sec   │ U-Net    │ Open-source    │
  │ Wan 2.1      │ 720p       │ ~5 sec   │ DiT      │ Open-weights   │
  │ CogVideoX    │ 720p       │ ~6 sec   │ DiT      │ Open-source    │
  │ AnimateDiff  │ 512×512    │ ~2 sec   │ U-Net    │ Plug-and-play  │
  └──────────────┴────────────┴──────────┴──────────┴────────────────┘
```

---

## Architectural Trends

```
  1. U-Net → DiT (Diffusion Transformer)
     • Sora, Kling, CogVideoX all use DiT
     • Better scaling properties
     • Native variable resolution/duration
     • Transformer scaling laws apply

  2. Latent space generation
     • All modern models work in latent space (not pixel space)
     • 3D VAE: compress spatial AND temporal dimensions
     • Typical compression: 8× spatial, 4× temporal
     
     Video (T×H×W×3) → 3D VAE → Latent (T/4 × H/8 × W/8 × C)
     → generate in this compact space → decode back

  3. Flow matching replacing DDPM
     • Simpler training objective
     • Straighter sampling trajectories
     • Fewer sampling steps needed
     • Used by SD3, Flux, and newer video models

  4. Rectified flows
     • Even straighter paths than flow matching
     • 1-step or few-step generation possible
     • Key for real-time video generation
```

---

## Emerging Capabilities

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. Camera Control                                           │
  │     Specify camera motion: pan, tilt, zoom, orbit            │
  │     → Cinematographic video generation                       │
  │                                                                │
  │  2. Character Consistency                                    │
  │     Same character across multiple generated videos          │
  │     → IP-Adapter, reference attention, identity tokens       │
  │                                                                │
  │  3. Physics Understanding                                   │
  │     Objects fall, liquids flow, cloth drapes realistically   │
  │     → Learned from large-scale video data (not physics sim)  │
  │                                                                │
  │  4. Image-to-Video                                           │
  │     Single image → animated video                            │
  │     → Product photography, illustration animation            │
  │                                                                │
  │  5. Video-to-Video                                           │
  │     Restyle or edit existing videos                          │
  │     → Change style, swap backgrounds, modify objects         │
  │                                                                │
  │  6. Audio-Synchronized Video                                 │
  │     Generate video synced with music or speech               │
  │     → Movie Gen generates matching audio track               │
  │                                                                │
  │  7. Text-Guided Video Editing                                │
  │     Edit specific aspects of existing videos with text       │
  │     → "Make it raining", "Change the car to blue"           │
  │                                                                │
  │  8. Multi-Shot / Story Generation                            │
  │     Generate multiple clips that form a coherent narrative   │
  │     → Early research, not yet production quality             │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Remaining Challenges

```
  Despite rapid progress, major challenges remain:

  1. Fine-grained motion control
     • Specifying exact movements is still difficult
     • "Person raises left hand while turning right" — often wrong
     • Trajectory-based conditioning helps but isn't solved

  2. Long-form consistency (>1 minute)
     • Character appearance drifts over time
     • Scene geometry can change subtly
     • Narrative coherence across many shots

  3. Text fidelity
     • Generating readable text in video is very hard
     • Signs, labels, subtitles often garbled
     • Related to the text rendering problem in image generation

  4. Hand and finger generation
     • Still produces extra/missing fingers frequently
     • Complex articulations remain challenging

  5. Physical accuracy
     • Objects sometimes pass through each other
     • Gravity, momentum, collisions not always correct
     • Learned physics ≠ simulated physics

  6. Counting and spatial relations
     • "Three birds flying" → may generate 2 or 5
     • Spatial relationships (left/right/behind) unreliable

  7. Generation speed
     • Minutes to hours for a single clip
     • Real-time video generation not yet practical
     • Distillation and caching methods improving this

  8. Controllability
     • Hard to specify exact framing, timing, composition
     • Gap between user intent and generated output
```

---

## Open-Source vs. Closed-Source Ecosystem

```
  Open-Source:
  ┌──────────────────────────────────────────────────┐
  │ • Stable Video Diffusion (Stability AI)          │
  │ • CogVideoX (Tsinghua / ZhipuAI)               │
  │ • Wan 2.1 (Alibaba)                              │
  │ • AnimateDiff (community)                        │
  │ • Open-Sora (HPC-AI Tech)                       │
  │ • Mochi (Genmo)                                   │
  │                                                    │
  │ Advantages:                                      │
  │ • Free to use, modify, fine-tune                 │
  │ • Community innovation and extensions            │
  │ • Research reproducibility                       │
  │ • Custom training on domain data                 │
  └──────────────────────────────────────────────────┘

  Closed-Source:
  ┌──────────────────────────────────────────────────┐
  │ • Sora (OpenAI)                                   │
  │ • Veo 2 (Google DeepMind)                        │
  │ • Kling (Kuaishou)                                │
  │ • Movie Gen (Meta — research only)               │
  │ • Runway Gen-3 Alpha                              │
  │ • Pika 2.0                                        │
  │                                                    │
  │ Advantages:                                      │
  │ • Higher quality (more compute, data)            │
  │ • Safety guardrails built-in                     │
  │ • API access, easy to use                        │
  │ • Regular improvements                           │
  └──────────────────────────────────────────────────┘

  Gap is narrowing: open models catching up rapidly
```

---

## Industry Applications

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Industry         │ Application                          │
  ├──────────────────┼──────────────────────────────────────┤
  │ Marketing        │ Product videos, social media ads,    │
  │                  │ A/B testing visual concepts          │
  ├──────────────────┼──────────────────────────────────────┤
  │ Film / TV        │ Previsualization, VFX prototyping,   │
  │                  │ background generation                │
  ├──────────────────┼──────────────────────────────────────┤
  │ Gaming           │ Cutscene prototyping, NPC animation, │
  │                  │ environment generation               │
  ├──────────────────┼──────────────────────────────────────┤
  │ Education        │ Explainer videos, historical         │
  │                  │ visualization, training content      │
  ├──────────────────┼──────────────────────────────────────┤
  │ E-commerce       │ Product animation, virtual try-on,   │
  │                  │ lifestyle imagery from photos        │
  ├──────────────────┼──────────────────────────────────────┤
  │ Architecture     │ Walkthrough videos from 3D plans,   │
  │                  │ design visualization                 │
  ├──────────────────┼──────────────────────────────────────┤
  │ Healthcare       │ Surgical training simulations,       │
  │                  │ patient education                    │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Ethical and Safety Considerations

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                                │
  │  1. Deepfakes                                                │
  │     • Realistic fake videos of real people                   │
  │     • Political manipulation, fraud, harassment              │
  │     • Detection tools exist but are in an arms race          │
  │                                                                │
  │  2. Misinformation                                           │
  │     • Fabricated "evidence" of events that never happened    │
  │     • Harder to verify video authenticity                    │
  │                                                                │
  │  3. Copyright and Consent                                    │
  │     • Training on copyrighted video content                  │
  │     • Generating content mimicking specific creators         │
  │     • Likeness rights and consent issues                     │
  │                                                                │
  │  4. Job Displacement                                         │
  │     • Stock footage, simple animation, B-roll production     │
  │     • Shifts skill requirements for video professionals      │
  │                                                                │
  │  Mitigations:                                                │
  │  • C2PA content credentials (provenance metadata)           │
  │  • Invisible watermarking (SynthID by Google)               │
  │  • AI-generated content labeling requirements               │
  │  • Responsible use policies and safety filters              │
  │  • Legislation (EU AI Act, proposed US regulations)         │
  │                                                                │
  └──────────────────────────────────────────────────────────────┘
```

---

## Future Directions

```
  Where video generation is heading:

  1. Real-Time Generation
     → Few-step or single-step diffusion for interactive use
     → Streaming video generation from live inputs

  2. World Models
     → Video generators as simulators of physical worlds
     → Sora framed as "world simulator" — learn physics from video
     → Robotics: plan actions by "imagining" outcomes

  3. 3D-Aware Video
     → Generate multi-view consistent video
     → Bridge between video generation and 3D/NeRF

  4. Interactive Video
     → User controls what happens next in real-time
     → Game-like experiences from generative models
     → GameNGen, Genie (Google): playable environments

  5. Personalized Video
     → Generate video starring specific people/objects
     → Consistent characters across scenes
     → DreamBooth/LoRA for video models

  6. Unified Multimodal Generation
     → Single model generates text + image + video + audio + 3D
     → "Any-to-any" generation
```

---

## Python: Working with Current Video Models

```python
import torch
from diffusers import CogVideoXPipeline
from diffusers.utils import export_to_video

# CogVideoX — open-source DiT-based video generation
pipe = CogVideoXPipeline.from_pretrained(
    "THUDM/CogVideoX-5b",
    torch_dtype=torch.bfloat16,
)
pipe.enable_model_cpu_offload()
pipe.vae.enable_tiling()

prompt = (
    "A drone shot flying over a beautiful coastal city at sunset, "
    "golden light reflecting off the ocean, cinematic quality"
)

video = pipe(
    prompt=prompt,
    num_videos_per_prompt=1,
    num_inference_steps=50,
    num_frames=49,
    guidance_scale=6.0,
).frames[0]

export_to_video(video, "cogvideox_output.mp4", fps=8)


# Wan 2.1 — Alibaba's open-source model
from diffusers import WanPipeline

pipe = WanPipeline.from_pretrained(
    "Wan-AI/Wan2.1-T2V-14B",
    torch_dtype=torch.bfloat16,
)
pipe.enable_model_cpu_offload()

video = pipe(
    prompt="A cat playing piano, studio lighting, 4K",
    num_frames=81,
    guidance_scale=5.0,
).frames[0]

export_to_video(video, "wan_output.mp4", fps=16)


# Evaluate generated video quality
from torchmetrics.image import FrechetInceptionDistance
from torchmetrics.image.lpip import LearnedPerceptualImagePatchSimilarity

def evaluate_temporal_consistency(frames_tensor):
    """
    frames_tensor: (T, C, H, W) — generated video frames
    Returns average LPIPS between consecutive frames.
    """
    lpips = LearnedPerceptualImagePatchSimilarity(net_type='alex')
    scores = []
    for i in range(len(frames_tensor) - 1):
        score = lpips(
            frames_tensor[i:i+1],
            frames_tensor[i+1:i+2]
        )
        scores.append(score.item())
    avg = sum(scores) / len(scores)
    print(f"Average LPIPS (lower=more consistent): {avg:.4f}")
    return avg
```

---

## Summary

```
  ┌────────────────────┬──────────────────────────────────────┐
  │ Aspect             │ Current State                        │
  ├────────────────────┼──────────────────────────────────────┤
  │ Quality            │ Near-photorealistic for short clips  │
  │ Duration           │ Up to ~60 seconds (best models)      │
  │ Resolution         │ Up to 4K (Veo 2)                     │
  │ Architecture       │ DiT (Diffusion Transformer) dominant │
  │ Training paradigm  │ Flow matching replacing DDPM         │
  │ Open-source        │ Competitive (CogVideoX, Wan, SVD)    │
  │ Speed              │ Minutes per clip (improving fast)     │
  │ Control            │ Text, image, camera, pose, depth     │
  │ Main challenges    │ Motion control, consistency, speed   │
  │ Next frontier      │ Real-time, interactive, world models │
  └────────────────────┴──────────────────────────────────────┘
```

---

## Revision Questions

1. **What architectural shift has defined video generation in 2024-2025?**
2. **What are the main advantages of latent space video generation?**
3. **Name three remaining challenges in video generation.**
4. **How do open-source video models compare to closed-source ones?**
5. **What is the "world model" perspective on video generation?**
6. **What safety measures exist for AI-generated video content?**

---

[Previous: 03-temporal-consistency.md](03-temporal-consistency.md) | [Back to README](../README.md)
