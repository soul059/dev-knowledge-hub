# Latent Diffusion Models (LDM)

## Overview

Latent Diffusion Models (Rombach et al., 2022) are the breakthrough that made high-resolution diffusion models practical. Instead of running the diffusion process in pixel space (computationally prohibitive at 512×512+), LDM first compresses images into a lower-dimensional **latent space** using a pretrained autoencoder, then applies the diffusion process in that compact space. This reduces computation by 4-48× while maintaining image quality. LDM is the foundation of **Stable Diffusion**.

---

## Motivation

```
Problem: diffusion in pixel space is EXPENSIVE

  512×512×3 image = 786,432 dimensions
  U-Net must process this at EVERY denoising step
  1000 steps × huge tensor = extremely slow and memory-heavy

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Pixel-space diffusion (DDPM, Guided Diffusion): │
  │  • 256×256: feasible but slow (~hours training)  │
  │  • 512×512: very expensive                       │
  │  • 1024×1024: impractical for most               │
  │                                                    │
  │  Key insight: most of an image is perceptually   │
  │  redundant! Nearby pixels are highly correlated. │
  │                                                    │
  │  Solution: compress first, diffuse in latent!    │
  │                                                    │
  │  512×512×3 → encode → 64×64×4 → diffuse here!   │
  │  (786K dims)          (16K dims = 48× smaller!)  │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Two-Stage Architecture

```
Stage 1: Perceptual Compression (pretrained, frozen)
  Autoencoder: Encoder E, Decoder D
  
  E: x (512×512×3) → z (64×64×4)     [compress]
  D: z (64×64×4) → x̂ (512×512×3)    [reconstruct]
  
  Trained with:
    L = L_reconstruction + L_perceptual + L_adversarial + L_KL
  
  z captures perceptually important features
  Imperceptible details (exact pixel values) discarded

Stage 2: Latent Diffusion (trained)
  Standard diffusion but on z instead of x:
  
  Forward: z₀ → z₁ → ... → z_T  (add noise in latent space)
  Reverse: z_T → z_{T-1} → ... → z₀  (denoise with U-Net)
  
  Then decode: x̂ = D(z₀)

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  Training:                                                │
  │  image x → E(x) → z₀ → add noise → z_t                  │
  │  ε_θ(z_t, t, c) → predict noise                          │
  │  Loss = ||ε - ε_θ||²                                     │
  │                                                            │
  │  Sampling:                                                │
  │  z_T ~ N(0,I) → denoise → z₀ → D(z₀) → image           │
  │                                                            │
  │  Autoencoder trained ONCE, reused for all!               │
  └──────────────────────────────────────────────────────────┘
```

---

## Autoencoder (KL-VAE)

```
  KL-regularized autoencoder (Stable Diffusion):
    Latent z has KL regularization toward N(0,1)
    Continuous latent space
    Downsampling factor f = 4 or 8
    
    512×512×3 → 64×64×4  (f=8)

  Training losses:
  
  L_rec:    ||x - D(E(x))||²  (pixel-level)
  L_perc:   ||φ(x) - φ(D(E(x)))||²  (VGG features)
  L_adv:    PatchGAN discriminator (sharpness)
  L_KL:     small KL penalty on z (weight ~10⁻⁶)

  Together: high-quality compression
  D(E(x)) ≈ x perceptually
```

---

## Conditioning via Cross-Attention

```
LDM supports flexible conditioning through cross-attention:

  Condition c (text, class, image) → encoder τ_θ → embeddings

  U-Net modified:
    Standard layers: self-attention on spatial features
    Added layers:    CROSS-attention with condition embeddings

  Cross-attention:
    Q = W_Q × features       (from U-Net)
    K = W_K × τ_θ(c)        (from condition encoder)
    V = W_V × τ_θ(c)        (from condition encoder)
    
    Attention(Q, K, V) = softmax(QKᵀ/√d) × V

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  U-Net ResBlock with conditioning:               │
  │                                                    │
  │  features → Self-Attention → Cross-Attention → out│
  │                                    ↑               │
  │                              τ_θ(condition)        │
  │                                                    │
  │  Condition types:                                 │
  │  • Text → CLIP/BERT encoder → cross-attention    │
  │  • Class → embedding → cross-attention           │
  │  • Image → encoder → cross-attention             │
  │  • Anything encodable!                           │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Computational Advantages

```
  Comparison (512×512 generation):

  Method              Latent Size    GPU Hours
  ──────────          ───────────    ─────────
  Pixel Diffusion     512×512×3     thousands
  LDM (f=4)          128×128×3     hundreds
  LDM (f=8)           64×64×4      ~200
  LDM (f=16)          32×32×8      ~50

  f=8 is the sweet spot:
  • 48× fewer pixels to process
  • Minimal quality loss
  • Used in Stable Diffusion 1.x and 2.x

  Inference: ~2-5 seconds per image (vs. ~30-60 for pixel)
```

---

## Python: Latent Diffusion Concept

```python
import torch
import torch.nn as nn

class LatentDiffusion(nn.Module):
    def __init__(self, autoencoder, unet, text_encoder, T=1000):
        super().__init__()
        self.encoder = autoencoder.encoder
        self.decoder = autoencoder.decoder
        self.unet = unet
        self.text_encoder = text_encoder
        
        for p in self.encoder.parameters(): p.requires_grad = False
        for p in self.decoder.parameters(): p.requires_grad = False
        for p in self.text_encoder.parameters(): p.requires_grad = False
        
        betas = torch.linspace(1e-4, 0.02, T)
        self.register_buffer('alpha_bars', torch.cumprod(1 - betas, dim=0))
        self.T = T
    
    def training_step(self, images, text_prompts, optimizer):
        with torch.no_grad():
            z0 = self.encoder(images)
            text_emb = self.text_encoder(text_prompts)
        
        t = torch.randint(0, self.T, (z0.size(0),), device=z0.device)
        noise = torch.randn_like(z0)
        
        ab = self.alpha_bars[t].view(-1, 1, 1, 1)
        z_t = torch.sqrt(ab) * z0 + torch.sqrt(1 - ab) * noise
        
        pred_noise = self.unet(z_t, t, context=text_emb)
        loss = nn.functional.mse_loss(pred_noise, noise)
        
        optimizer.zero_grad(); loss.backward(); optimizer.step()
        return loss.item()
    
    @torch.no_grad()
    def generate(self, text_prompt, num_steps=50, guidance_scale=7.5):
        cond_emb = self.text_encoder(text_prompt)
        uncond_emb = self.text_encoder("")
        
        z = torch.randn(1, 4, 64, 64, device=self.alpha_bars.device)
        timesteps = torch.linspace(self.T-1, 0, num_steps).long()
        
        for t in timesteps:
            tb = t.unsqueeze(0)
            noise_cond = self.unet(z, tb, context=cond_emb)
            noise_uncond = self.unet(z, tb, context=uncond_emb)
            noise_pred = noise_uncond + guidance_scale * (noise_cond - noise_uncond)
            
            ab = self.alpha_bars[t]
            ab_prev = self.alpha_bars[max(t-self.T//num_steps, 0)]
            
            z0_pred = (z - torch.sqrt(1-ab) * noise_pred) / torch.sqrt(ab)
            z = torch.sqrt(ab_prev) * z0_pred + torch.sqrt(1-ab_prev) * noise_pred
        
        image = self.decoder(z)
        return image
```

---

## Revision Questions

1. **Why is diffusion in pixel space impractical for high-resolution images?**
2. **What does the autoencoder's latent space capture vs. discard?**
3. **How does cross-attention enable text conditioning?**
4. **Why is downsampling factor f=8 considered the sweet spot?**
5. **Why is the autoencoder frozen during diffusion training?**

---

[Previous: ../07-Diffusion-Models/06-ddim.md](../07-Diffusion-Models/06-ddim.md) | [Next: 02-text-to-image-generation.md](02-text-to-image-generation.md)
