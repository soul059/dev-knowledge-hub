# Text-to-Image Generation

## Overview

Text-to-image generation transforms natural language descriptions into photorealistic or artistic images. This task combines the power of large language models (for understanding text) with diffusion models (for generating images). Key systems include **DALL-E** (OpenAI), **Imagen** (Google), **Stable Diffusion** (Stability AI), and **Midjourney**. The common paradigm uses a text encoder (CLIP or T5) to create embeddings that condition a diffusion model via cross-attention.

---

## Pipeline Architecture

```
  "A cat wearing sunglasses on a beach at sunset"
       │
       ▼
  ┌────────────────┐
  │  Text Encoder   │  CLIP / T5 / BERT
  │  (frozen)       │  
  └───────┬─────────┘
          │  text embeddings (77 × 768)
          ▼
  ┌────────────────┐
  │  Diffusion      │  U-Net with cross-attention
  │  Model          │  Iterative denoising
  │  (latent space) │  
  └───────┬─────────┘
          │  latent z₀ (64 × 64 × 4)
          ▼
  ┌────────────────┐
  │  Decoder        │  VAE decoder
  │  (frozen)       │  
  └───────┬─────────┘
          │
          ▼
  🖼️ Generated Image (512 × 512)
```

---

## Key Systems Comparison

```
  System          Text Encoder   Image Model        Resolution
  ──────────      ────────────   ───────────        ──────────
  DALL-E 1        GPT-style      VQ-VAE + AR        256×256
  DALL-E 2        CLIP           Diffusion + prior  1024×1024
  DALL-E 3        T5 + CLIP      Diffusion          1024×1024
  Imagen          T5-XXL         Pixel diffusion    1024×1024
  Stable Diff 1   CLIP ViT-L     Latent diffusion   512×512
  Stable Diff 2   OpenCLIP       Latent diffusion   768×768
  SD XL           CLIP + OpenCLIP Latent diffusion  1024×1024
  Midjourney      Proprietary    Proprietary        1024×1024
```

---

## DALL-E Evolution

```
DALL-E 1 (Jan 2021):
  Text → GPT tokens + image VQ-VAE tokens
  Autoregressive generation of image tokens
  256×256 resolution
  First viral text-to-image model

DALL-E 2 (Apr 2022):
  Two-stage approach:
  1. Prior: text embedding → CLIP image embedding
     (maps text space to image space)
  2. Decoder: CLIP image embedding → image
     (unCLIP: reversed CLIP)
  
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  text → CLIP text encoder → text embedding       │
  │                                  │                │
  │                                  ▼                │
  │                            ┌──────────┐          │
  │                            │  Prior    │          │
  │                            │(diffusion)│          │
  │                            └────┬─────┘          │
  │                                 │ CLIP image emb  │
  │                                 ▼                 │
  │                            ┌──────────┐          │
  │                            │ Decoder   │          │
  │                            │(diffusion)│          │
  │                            └────┬─────┘          │
  │                                 │                 │
  │                                 ▼                 │
  │                            🖼️ Image              │
  │                                                    │
  └──────────────────────────────────────────────────┘

DALL-E 3 (Sep 2023):
  Improved text understanding via recaptioning
  Synthetic captions for training data
  Better prompt following
  Native ChatGPT integration
```

---

## Imagen (Google)

```
Key insight: larger text encoder > larger image model

  Uses T5-XXL (4.6B params) as text encoder
  Much larger than CLIP (400M)
  
  Better text understanding → better prompt following

  Architecture:
  1. T5-XXL encodes text
  2. 64×64 base diffusion model (pixel space!)
  3. 64→256 super-resolution diffusion model
  4. 256→1024 super-resolution diffusion model

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  text → T5-XXL → embeddings                     │
  │                       │                           │
  │                       ▼                           │
  │              64×64 diffusion model               │
  │                       │                           │
  │                       ▼                           │
  │             256×256 super-resolution             │
  │                       │                           │
  │                       ▼                           │
  │            1024×1024 super-resolution             │
  │                       │                           │
  │                       ▼                           │
  │                  🖼️ Image                        │
  │                                                    │
  │  Cascaded pixel-space diffusion                  │
  │  (not latent — more expensive but higher quality)│
  └──────────────────────────────────────────────────┘

  Key finding: scaling T5 from 60M to 4.6B
  improved FID more than scaling the diffusion model
```

---

## Stable Diffusion Pipeline

```
  Stable Diffusion (Rombach et al. / Stability AI):

  1. Tokenizer: text → token IDs (max 77 tokens)
  2. CLIP text encoder: token IDs → embeddings (77×768)
  3. U-Net: iterative denoising in latent space
     - Cross-attention layers use text embeddings
     - Classifier-free guidance (w=7.5)
     - DDIM or DPM-Solver sampling (20-50 steps)
  4. VAE decoder: latent → image

  SDXL improvements:
  • Two text encoders: CLIP ViT-L + OpenCLIP ViT-bigG
  • Larger U-Net (2.6B params vs 860M)
  • Conditioning on original image size
  • Refiner model for details
  • 1024×1024 native resolution
```

---

## Training Data and Challenges

```
  Training datasets:
  • LAION-5B: 5 billion image-text pairs
  • Filtered for quality, resolution, aesthetics
  • Synthetic recaptioning (DALL-E 3)

  Challenges:
  1. Compositional understanding
     "a red cube on top of a blue sphere"
     → models may swap colors or positions
     
  2. Counting
     "five apples" → might generate 3 or 7
     
  3. Text rendering
     "a sign that says HELLO" → garbled text
     (improving with newer models)
     
  4. Spatial relationships
     "cat LEFT of dog" → sometimes reversed
     
  5. Rare concepts
     Unusual combinations harder to generate
```

---

## Python: Using Stable Diffusion

```python
# Using the diffusers library (Hugging Face)
from diffusers import StableDiffusionPipeline
import torch

# Load pretrained model
pipe = StableDiffusionPipeline.from_pretrained(
    "stabilityai/stable-diffusion-2-1",
    torch_dtype=torch.float16
).to("cuda")

# Basic generation
image = pipe(
    prompt="a serene mountain landscape at golden hour, photorealistic",
    negative_prompt="blurry, low quality, distorted",
    num_inference_steps=50,
    guidance_scale=7.5,
    width=768,
    height=768,
).images[0]

image.save("landscape.png")

# Batch generation with different seeds
images = []
for seed in range(4):
    generator = torch.Generator("cuda").manual_seed(seed)
    img = pipe(
        prompt="a cyberpunk city at night, neon lights, rain",
        generator=generator,
        num_inference_steps=30,
        guidance_scale=7.5,
    ).images[0]
    images.append(img)

# SDXL usage
from diffusers import StableDiffusionXLPipeline

pipe_xl = StableDiffusionXLPipeline.from_pretrained(
    "stabilityai/stable-diffusion-xl-base-1.0",
    torch_dtype=torch.float16,
).to("cuda")

image = pipe_xl(
    prompt="detailed portrait, studio lighting",
    num_inference_steps=40,
).images[0]
```

---

## Revision Questions

1. **How does the text-to-image pipeline work end-to-end?**
2. **What is DALL-E 2's prior and why is it needed?**
3. **Why did Imagen find that scaling the text encoder matters more than the image model?**
4. **What are the key differences between DALL-E, Imagen, and Stable Diffusion?**
5. **What compositional understanding challenges remain for text-to-image models?**

---

[Previous: 01-latent-diffusion-models.md](01-latent-diffusion-models.md) | [Next: 03-clip-text-image-alignment.md](03-clip-text-image-alignment.md)
