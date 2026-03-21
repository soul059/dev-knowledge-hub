# Image Generation (GANs Preview)

## Overview

**Image generation** creates new, realistic images that don't exist in reality. **Generative Adversarial Networks (GANs)** revolutionized this field by training two networks in competition — a generator that creates images and a discriminator that judges them. This chapter provides a preview; the full Generative Models subject covers GANs, VAEs, and diffusion models in depth.

---

## GAN Framework

```
  ┌──────────────────────────────────────────────────┐
  │                 GAN Training                      │
  │                                                    │
  │  Random noise z           Real images x            │
  │  (latent vector)          (from dataset)           │
  │       │                        │                   │
  │  ┌────▼────┐              ┌────▼────┐              │
  │  │Generator│              │         │              │
  │  │   G(z)  │              │         │              │
  │  └────┬────┘              │         │              │
  │       │                   │         │              │
  │  Fake image G(z)          │ Real image x           │
  │       │                   │         │              │
  │       └───────┬───────────┘         │              │
  │               │                                    │
  │         ┌─────▼─────┐                              │
  │         │Discriminator│                             │
  │         │   D(·)     │                              │
  │         └─────┬─────┘                              │
  │               │                                    │
  │          Real or Fake?                             │
  │                                                    │
  │  Minimax game:                                     │
  │  min_G max_D  E[log D(x)] + E[log(1-D(G(z)))]    │
  │                                                    │
  │  D tries to distinguish real from fake             │
  │  G tries to fool D into accepting fakes            │
  └──────────────────────────────────────────────────┘

  Training alternates:
    1. Train D: maximize correct classification
    2. Train G: minimize D's ability to detect fakes
    
  At convergence: G generates images indistinguishable from real
                  D outputs 0.5 for everything (can't tell)
```

---

## GAN Evolution

```
  GAN (2014):        Basic MLP, low-res, unstable training
  DCGAN (2015):      Convolutional, 64×64, stable training guidelines
  WGAN (2017):       Wasserstein distance, better gradients
  ProGAN (2018):     Progressive growing, 1024×1024
  StyleGAN (2019):   Style-based generator, photorealistic faces
  StyleGAN2 (2020):  Improved, no artifacts, state-of-the-art
  StyleGAN3 (2021):  Alias-free, better video generation

  Quality progression:
  
  2014     2016        2018          2020
  ┌──┐    ┌────┐     ┌──────┐    ┌────────┐
  │??│    │ 😊 │     │  😊  │    │   😊   │
  │??│    │    │     │      │    │        │ Photo-
  └──┘    └────┘     └──────┘    └────────┘ realistic!
  4×4    64×64      512×512    1024×1024
```

---

## StyleGAN Architecture

```
  Mapping network: z → w (style space)
    z ∈ R^512 → 8-layer MLP → w ∈ R^512
    
  Synthesis network: w → image
    
    Constant input (4×4)
         │
    ┌────▼────┐
    │ AdaIN   │ ← w (style modulation)  + noise
    │ Conv    │
    │ Upsample│
    ├─────────┤
    │ AdaIN   │ ← w                     + noise
    │ Conv    │
    │ Upsample│   
    ├─────────┤   Each block doubles resolution:
    │   ...   │   4→8→16→32→64→128→256→512→1024
    ├─────────┤
    │ AdaIN   │ ← w                     + noise
    │ Conv    │
    └────┬────┘
         │
    Output (1024×1024×3)

  Style mixing:
    Use different w at different layers
    → Control coarse features (pose, shape) separately from
       fine features (color, texture)
    
  Truncation trick:
    Move w toward average w̄ for higher quality (less diversity)
    w' = w̄ + ψ(w - w̄)     ψ < 1 for quality, ψ > 1 for diversity
```

---

## Conditional GANs

```
Generate images conditioned on input:

  Pix2Pix (2017): Paired image translation
    Input → Output (paired training data)
    Edges → Photo, Segmentation → Photo, Day → Night
    
    ┌──────────┐     ┌──────────┐
    │ Edges    │ →  │  Photo   │
    │  ╱╲     │     │  🏠     │
    │ ╱  ╲    │     │         │
    └──────────┘     └──────────┘

  CycleGAN (2017): Unpaired image translation
    No paired data needed! Cycle consistency loss:
    
    Horse → G_AB → Zebra → G_BA → Horse ≈ original
    
    L_cycle = ||G_BA(G_AB(x)) - x||
    
    ┌──────┐   ┌──────┐   ┌──────┐
    │ 🐴  │ → │ 🦓  │ → │ 🐴  │  Should match!
    └──────┘   └──────┘   └──────┘

  SPADE (2019): Semantic layout → photorealistic image
    Control image generation with semantic segmentation map
```

---

## GANs for CV Tasks

```
Image generation is just the beginning:

  Super-resolution:  SRGAN, ESRGAN
    Low-res → GAN → High-res (4× upscaling)
    
  Inpainting:  DeepFill, LaMa
    Remove objects, fill missing regions
    
  Style transfer:  Neural style transfer
    Content image + style image → stylized image
    
  Data augmentation:  Generate synthetic training data
    Few real images → GAN → thousands of synthetic images
    
  Face editing:  InterFaceGAN
    Control attributes: age, smile, glasses, hair
    Move in latent space along attribute directions
    
  Deepfakes:  Face swapping
    Replace one person's face with another's
    Ethical concerns → detection methods
```

---

## Diffusion Models (The New Standard)

```
GANs are being replaced by diffusion models for many tasks:

  Diffusion process:
    Forward: Gradually add noise to image (T steps)
    Reverse: Learn to denoise step by step
    
    x_0 → x_1 → x_2 → ... → x_T (pure noise)
    x_T → x_{T-1} → ... → x_1 → x_0 (generated image)
    
  Key models:
    DALL-E 2 (2022):     Text → image (OpenAI)
    Stable Diffusion (2022): Open-source text → image
    Midjourney (2022):    Commercial text → image
    SDXL (2023):         Higher quality Stable Diffusion
    FLUX (2024):         Latest open-source
    
  Advantages over GANs:
    - More stable training (no mode collapse)
    - Better diversity
    - Text-conditioned generation
    - Better at complex compositions
    
  Disadvantages:
    - Slower generation (many denoising steps)
    - More memory for training
```

---

## Python: Image Generation

```python
# StyleGAN2 with PyTorch
import torch

# Generate random face
G = torch.hub.load('nvidia/stylegan2-ada-pytorch', 'generator',
                     pretrained=True)
z = torch.randn(1, 512)  # random latent
with torch.no_grad():
    img = G(z)  # 1×3×1024×1024
    
# Save generated face
from torchvision.utils import save_image
save_image(img, "generated_face.png", normalize=True)


# Stable Diffusion (text to image)
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained(
    "stabilityai/stable-diffusion-2-1",
    torch_dtype=torch.float16
).to("cuda")

image = pipe("A cat wearing sunglasses on a beach, 
              photorealistic, 4k").images[0]
image.save("cat_beach.png")
```

---

## Revision Questions

1. **How does the GAN training process work (generator vs discriminator)?**
2. **What makes StyleGAN's architecture different from basic GANs?**
3. **How does CycleGAN achieve unpaired image translation?**
4. **What CV tasks beyond generation can GANs be used for?**
5. **How do diffusion models differ from GANs and why are they gaining popularity?**

---

[Previous: 03-pose-estimation.md](03-pose-estimation.md) | [Next: 05-video-understanding.md](05-video-understanding.md)
