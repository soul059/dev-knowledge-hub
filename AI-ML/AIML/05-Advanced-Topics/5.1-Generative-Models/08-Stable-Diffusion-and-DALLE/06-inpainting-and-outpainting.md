# Inpainting and Outpainting

## Overview

**Inpainting** fills in missing or masked regions of an image, while **outpainting** extends an image beyond its original boundaries. Both leverage the iterative denoising of diffusion models — masked regions start as noise while unmasked regions are preserved, and the model generates coherent content that blends seamlessly. These techniques enable powerful image editing workflows: removing objects, replacing backgrounds, extending compositions, and creative editing.

---

## Inpainting

```
Fill in a masked region with contextually appropriate content:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Original        Mask            Result           │
  │  ┌────────┐     ┌────────┐     ┌────────┐       │
  │  │ 🏠 🚗  │     │ ░░ ██  │     │ 🏠 🌳  │       │
  │  │ 🌳 🚗  │  +  │ ░░ ██  │  =  │ 🌳 🌳  │       │
  │  │ ░░ ░░  │     │ ░░ ░░  │     │ ░░ ░░  │       │
  │  └────────┘     └────────┘     └────────┘       │
  │                                                    │
  │  ░░ = keep original pixels                       │
  │  ██ = replace (generate new content here)        │
  │                                                    │
  │  "Remove the car" + mask over car                │
  │  → diffusion fills with contextual content       │
  └──────────────────────────────────────────────────┘
```

---

## Diffusion-Based Inpainting

```
Method 1: Repaint (simple, no training needed)

  At each denoising step t:
    x_{t-1}^known  = √ᾱ_{t-1} × x₀ + √(1-ᾱ_{t-1}) × ε  (from original)
    x_{t-1}^unknown = denoise(x_t, t)                       (generated)
    
    x_{t-1} = mask × x_{t-1}^unknown + (1-mask) × x_{t-1}^known

  Problem: boundary between known/unknown can be visible!
  Solution: resample (jump back and denoise again) for coherence

Method 2: Fine-tuned inpainting model (better)

  Train diffusion model with additional input channels:
    Input: [noisy_latent, mask, masked_image]
    
    Standard SD: 4 input channels (latent)
    Inpainting SD: 9 input channels (latent + mask + masked_latent)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Inpainting U-Net input:                         │
  │                                                    │
  │  Channel 1-4:  Noisy latent z_t                  │
  │  Channel 5:    Binary mask (0 = keep, 1 = fill)  │
  │  Channel 6-9:  Masked image latent E(x ⊙ (1-m)) │
  │                                                    │
  │  Total: 9 channels → first conv adapted          │
  │                                                    │
  │  Much better boundary coherence!                 │
  │  Model learns to blend naturally                 │
  └──────────────────────────────────────────────────┘
```

---

## Outpainting

```
Extend image beyond original boundaries:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Original:           Outpainted:                 │
  │  ┌────────┐         ┌──────────────────┐        │
  │  │        │         │░░░░│        │░░░░│        │
  │  │ 🏔️    │    →    │░░░░│ 🏔️    │░░░░│        │
  │  │ 🌊    │         │░░░░│ 🌊    │░░░░│        │
  │  └────────┘         │░░░░│        │░░░░│        │
  │                      └──────────────────┘        │
  │                                                    │
  │  ░░ = generated extension                        │
  │                                                    │
  │  Process:                                         │
  │  1. Place original image in larger canvas        │
  │  2. Mask = everything outside original           │
  │  3. Run inpainting on the mask                   │
  │  4. Text prompt guides the extension             │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Challenges:
  • Perspective consistency
  • Color/lighting continuity
  • Semantic coherence (don't add random objects)
```

---

## Text-Guided Inpainting

```
Combine mask + text prompt for controlled editing:

  "Replace the car with a garden"
  + mask over car
  → Generates garden exactly in masked area!

  Process:
  1. User provides image + mask + text prompt
  2. Encode: z₀ = E(image), mask downscaled to latent size
  3. Add noise to masked region
  4. Denoise with text conditioning
  5. Decode: x̂ = D(z₀)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Prompt: "a golden retriever puppy"              │
  │  Mask: over existing pet                         │
  │                                                    │
  │  Result: original scene preserved                │
  │          masked region → golden retriever         │
  │          seamless blending at boundaries          │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Advanced Techniques

```
1. SDEdit (Meng et al., 2022):
   Add noise to ENTIRE image (not just mask)
   Then denoise → style transfer / editing
   
   Strength parameter controls edit amount:
     strength=0.3: minor changes
     strength=0.8: major restyling
     strength=1.0: generate from scratch (ignores input)

2. Prompt-to-Prompt editing:
   Modify cross-attention maps to edit specific parts
   "a cat" → "a dog" (swap attention for 'cat'→'dog')
   No mask needed!

3. Blended Diffusion:
   Use CLIP to guide generation in masked region
   Optimize for both text alignment and background coherence

4. Differential Diffusion:
   Variable-strength mask (gradient, not binary)
   Strong edit at center, subtle at edges
   Better blending
```

---

## Python: Inpainting with Stable Diffusion

```python
from diffusers import StableDiffusionInpaintPipeline
import torch
from PIL import Image
import numpy as np

# Load inpainting-specific model
pipe = StableDiffusionInpaintPipeline.from_pretrained(
    "stabilityai/stable-diffusion-2-inpainting",
    torch_dtype=torch.float16,
).to("cuda")

# Load image and create mask
image = Image.open("photo.png").resize((512, 512))
mask = Image.open("mask.png").resize((512, 512))  # white = inpaint

# Inpaint
result = pipe(
    prompt="a beautiful flower garden, detailed, photorealistic",
    negative_prompt="blurry, low quality",
    image=image,
    mask_image=mask,
    num_inference_steps=50,
    guidance_scale=7.5,
).images[0]
result.save("inpainted.png")

# Outpainting
def outpaint(pipe, image, direction="right", extend_pixels=256,
             prompt="continuation of the scene"):
    w, h = image.size
    
    if direction == "right":
        canvas = Image.new("RGB", (w + extend_pixels, h), (0, 0, 0))
        canvas.paste(image, (0, 0))
        mask = Image.new("L", (w + extend_pixels, h), 0)
        # White mask on extended area
        mask_arr = np.array(mask)
        mask_arr[:, w:] = 255
        mask = Image.fromarray(mask_arr)
    
    result = pipe(
        prompt=prompt,
        image=canvas.resize((512, 512)),
        mask_image=mask.resize((512, 512)),
        num_inference_steps=50,
    ).images[0]
    
    return result.resize((w + extend_pixels, h))

# SDEdit-style editing
from diffusers import StableDiffusionImg2ImgPipeline

pipe_edit = StableDiffusionImg2ImgPipeline.from_pretrained(
    "stabilityai/stable-diffusion-2-1",
    torch_dtype=torch.float16,
).to("cuda")

edited = pipe_edit(
    prompt="a watercolor painting of the scene",
    image=image,
    strength=0.6,  # 0=no change, 1=full generation
    num_inference_steps=50,
).images[0]
```

---

## Summary Table

| Technique | Input | Use Case | Quality |
|-----------|-------|----------|---------|
| Repaint | Any SD model + mask | Quick inpainting | Good |
| Inpainting model | Specialized model | Production inpainting | Excellent |
| SDEdit | Any SD model + strength | Style transfer, editing | Good |
| Outpainting | Inpainting model + canvas | Image extension | Good |
| Prompt-to-Prompt | Any SD model + edit map | Precise text edits | Very good |

---

## Revision Questions

1. **How does the Repaint method preserve unmasked regions during denoising?**
2. **Why is a specialized inpainting model better than basic masking?**
3. **How does outpainting work as a special case of inpainting?**
4. **What does the "strength" parameter control in SDEdit?**
5. **Why is boundary coherence a challenge in inpainting?**

---

[Previous: 05-controlnet.md](05-controlnet.md) | [Next: ../09-Audio-Generation/01-wavenet.md](../09-Audio-Generation/01-wavenet.md)
