# Classifier-Free Guidance

## Overview

Classifier-free guidance (Ho & Salimans, 2022) is the technique that made conditional diffusion models produce high-quality, prompt-adherent images. By training a single model to handle both conditional and unconditional generation (randomly dropping the condition during training), we can amplify the effect of conditioning at inference time. This eliminates the need for a separate classifier and is now the standard approach in all major text-to-image systems.

---

## Classifier Guidance (Background)

```
Before classifier-free guidance, there was classifier guidance:

  Train a separate classifier p(y|x_t) on noisy images
  Use its gradients to steer the diffusion process

  Modified noise prediction:
    ε̃ = ε_θ(x_t, t) - s × σ_t × ∇_{x_t} log p(y|x_t)

  s = guidance scale (higher = stronger conditioning)

  Problems:
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  1. Need separate classifier                     │
  │     → Must train on noisy images at ALL noise    │
  │       levels (expensive!)                        │
  │                                                    │
  │  2. Classifier gradients can be adversarial      │
  │     → Generated images may "fool" classifier     │
  │       without looking realistic                  │
  │                                                    │
  │  3. Limited to labels classifier was trained on  │
  │     → Can't use with free-form text              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Classifier-Free Guidance Concept

```
Key idea: the model IS its own classifier!

  Train ONE model for both:
    ε_θ(x_t, t, c)     conditional   (with text/class c)
    ε_θ(x_t, t, ∅)     unconditional (c = null/empty)

  During training:
    Randomly drop condition c with probability p_uncond (e.g., 10%)
    When dropped, replace c with ∅ (null embedding)

  During inference:
    ε̃ = ε_θ(x_t, t, ∅) + w × (ε_θ(x_t, t, c) - ε_θ(x_t, t, ∅))

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  ε_uncond = ε_θ(x_t, t, ∅)     "what to generate│
  │                                   without prompt" │
  │                                                    │
  │  ε_cond = ε_θ(x_t, t, c)       "what to generate│
  │                                   WITH prompt"    │
  │                                                    │
  │  difference = ε_cond - ε_uncond  "effect of the  │
  │                                    prompt"        │
  │                                                    │
  │  ε_guided = ε_uncond + w × difference            │
  │                                                    │
  │  w > 1: AMPLIFY the prompt's effect!             │
  │  w = 1: standard conditional (no guidance)       │
  │  w = 0: unconditional                             │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Effect of Guidance Scale

```
  w = 1:   Standard conditional generation
           Diverse but may ignore parts of prompt
           
  w = 3:   Moderate guidance
           Better prompt following
           Still good diversity

  w = 7.5: Standard for Stable Diffusion
           Strong prompt following
           Good quality/diversity trade-off

  w = 15:  Very strong guidance
           Excellent prompt following
           Reduced diversity, may over-saturate

  w = 30+: Extreme guidance
           Artifacts, over-saturated colors
           Distorted, unrealistic

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Quality/Diversity Trade-off:                    │
  │                                                    │
  │  Diversity ▲                                     │
  │            │╲                                     │
  │            │  ╲                                   │
  │            │    ╲     sweet spot                  │
  │            │     ╲ ●  (w=7.5)                    │
  │            │       ╲                              │
  │            │         ╲                            │
  │            └──────────────► Fidelity              │
  │            w=1       w=7.5  w=20                  │
  │                                                    │
  │  Higher w = better alignment, less diversity     │
  └──────────────────────────────────────────────────┘
```

---

## Mathematical Derivation

```
Classifier guidance uses Bayes' rule:

  ∇ log p(x|c) = ∇ log p(x) + ∇ log p(c|x)
  (conditional)   (unconditional) (classifier gradient)

  With guidance scale s:
  ∇ log p(x|c) = ∇ log p(x) + s × ∇ log p(c|x)

Classifier-FREE guidance implicitly estimates ∇ log p(c|x):

  ∇ log p(c|x) ∝ ε_cond - ε_uncond

  So: ε̃ = ε_uncond + w × (ε_cond - ε_uncond)
      = (1 - w) × ε_uncond + w × ε_cond

  This is equivalent to:
  ∇ log p(x|c) with guidance scale (w-1)!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  The implicit classifier:                        │
  │                                                    │
  │  log p(c|x) ∝ ε_θ(x_t,t,c) - ε_θ(x_t,t,∅)    │
  │                                                    │
  │  No separate classifier needed!                  │
  │  The diffusion model implicitly learns one!      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Training with Condition Dropout

```
Simple modification to training:

  Standard training:
    Always use condition c

  Classifier-free training:
    With probability p_uncond (10-20%):
      Replace c with null ∅
    Otherwise:
      Use real condition c

  For text conditioning:
    ∅ = empty string "" → CLIP("") → null embedding

  For class conditioning:
    ∅ = special "null class" token

  Cost: ~10% of training steps are unconditional
  Benefit: huge quality improvement at inference!
```

---

## Computational Cost

```
  Classifier-free guidance requires TWO forward passes:

  1. ε_θ(x_t, t, c)    conditional pass
  2. ε_θ(x_t, t, ∅)    unconditional pass

  2× the inference cost per step!

  Optimization: batch both together
    Input: [x_t, x_t]  (duplicate)
    Cond:  [c, ∅]      (cond and uncond)
    → Single batched forward pass (more efficient)

  Further optimizations:
    • Distillation to single-pass model
    • Apply guidance only at certain timesteps
    • Negative prompting (replace ∅ with negative prompt)
```

---

## Negative Prompts

```
Replace unconditional ∅ with a NEGATIVE prompt:

  ε̃ = ε_θ(x_t, t, neg) + w × (ε_θ(x_t, t, pos) - ε_θ(x_t, t, neg))

  "Move AWAY from negative prompt, TOWARD positive prompt"

  Examples:
    Positive: "high quality photograph of a landscape"
    Negative: "blurry, low quality, distorted, watermark"

  The model actively avoids generating features
  described in the negative prompt!

  Common negative prompts:
    "blurry, bad anatomy, low quality, watermark,
     text, deformed, ugly, duplicate"
```

---

## Python: Classifier-Free Guidance

```python
import torch

class ClassifierFreeGuidanceSampler:
    def __init__(self, unet, scheduler, text_encoder):
        self.unet = unet
        self.scheduler = scheduler
        self.text_encoder = text_encoder
    
    def encode_prompt(self, prompt, negative_prompt=""):
        cond_emb = self.text_encoder(prompt)
        uncond_emb = self.text_encoder(negative_prompt)
        return cond_emb, uncond_emb
    
    @torch.no_grad()
    def sample(self, prompt, negative_prompt="",
               guidance_scale=7.5, num_steps=50, shape=(1,4,64,64)):
        cond_emb, uncond_emb = self.encode_prompt(prompt, negative_prompt)
        
        # Start from noise
        latent = torch.randn(shape)
        
        timesteps = self.scheduler.get_timesteps(num_steps)
        
        for t in timesteps:
            # Batch conditional and unconditional together
            latent_input = torch.cat([latent, latent])
            text_input = torch.cat([uncond_emb, cond_emb])
            
            # Single forward pass for both
            noise_pred = self.unet(latent_input, t, context=text_input)
            
            # Split predictions
            noise_uncond, noise_cond = noise_pred.chunk(2)
            
            # Apply guidance
            noise_guided = noise_uncond + guidance_scale * (noise_cond - noise_uncond)
            
            # Denoise step
            latent = self.scheduler.step(noise_guided, t, latent)
        
        return latent

    def sample_with_dynamic_guidance(self, prompt, num_steps=50,
                                      guidance_start=3.0, guidance_end=12.0):
        """Increase guidance scale during sampling."""
        cond_emb, uncond_emb = self.encode_prompt(prompt)
        latent = torch.randn((1, 4, 64, 64))
        
        timesteps = self.scheduler.get_timesteps(num_steps)
        
        for i, t in enumerate(timesteps):
            progress = i / len(timesteps)
            w = guidance_start + progress * (guidance_end - guidance_start)
            
            noise_uncond = self.unet(latent, t, context=uncond_emb)
            noise_cond = self.unet(latent, t, context=cond_emb)
            
            noise_guided = noise_uncond + w * (noise_cond - noise_uncond)
            latent = self.scheduler.step(noise_guided, t, latent)
        
        return latent
```

---

## Revision Questions

1. **Why is classifier-free guidance preferred over classifier guidance?**
2. **How does condition dropout during training enable guidance at inference?**
3. **What is the trade-off when increasing the guidance scale w?**
4. **How do negative prompts work mathematically?**
5. **Why does classifier-free guidance double the inference cost?**

---

[Previous: 03-clip-text-image-alignment.md](03-clip-text-image-alignment.md) | [Next: 05-controlnet.md](05-controlnet.md)
