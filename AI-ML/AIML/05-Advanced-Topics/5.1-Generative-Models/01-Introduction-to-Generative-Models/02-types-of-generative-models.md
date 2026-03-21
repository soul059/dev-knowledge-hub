# Types of Generative Models

## Overview

Generative models come in several families, each with distinct architectures and training paradigms. The major categories are **autoregressive models**, **autoencoders (AE/VAE)**, **GANs**, **normalizing flows**, and **diffusion models**. Each has unique strengths — some produce sharp images, others offer exact likelihoods, and some excel at text or audio.

---

## Taxonomy

```
  ┌────────────────────────────────────────────────────────┐
  │                  Generative Models                      │
  │                                                         │
  │  ┌──────────────┐  ┌──────────┐  ┌───────────────────┐ │
  │  │ Autoregressive│  │   VAE    │  │       GAN         │ │
  │  │               │  │          │  │                   │ │
  │  │ P(x)=Π P(xᵢ) │  │ Encode → │  │ Generator vs     │ │
  │  │ One token     │  │ Latent → │  │ Discriminator    │ │
  │  │ at a time     │  │ Decode   │  │ (adversarial)    │ │
  │  └──────────────┘  └──────────┘  └───────────────────┘ │
  │                                                         │
  │  ┌──────────────┐  ┌──────────────────────────────────┐│
  │  │  Normalizing │  │     Diffusion Models              ││
  │  │    Flows      │  │                                   ││
  │  │              │  │  Add noise → Learn to denoise    ││
  │  │ Invertible   │  │  Iterative refinement            ││
  │  │ transforms   │  │  (DDPM, Stable Diffusion)        ││
  │  └──────────────┘  └──────────────────────────────────┘│
  └────────────────────────────────────────────────────────┘
```

---

## 1. Autoregressive Models

```
Generate data one element at a time, conditioned on previous:

  P(x) = P(x₁) × P(x₂|x₁) × P(x₃|x₁,x₂) × ...
       = Π_i P(x_i | x_{<i})

  Examples:
    Text:  GPT, LLaMA → one token at a time
    Image: PixelCNN, PixelRNN → one pixel at a time
    Audio: WaveNet → one sample at a time

  Pros:
    ✓ Exact likelihood computation
    ✓ Flexible, high quality
    ✓ State-of-the-art for text (GPT)
  
  Cons:
    ✗ Sequential generation → SLOW
    ✗ Can't parallelize generation
    ✗ Must define ordering of elements
```

---

## 2. Variational Autoencoders (VAE)

```
Encode data to latent space, decode back:

  Input x → Encoder → z ~ q(z|x) → Decoder → x̂
  
  Latent z is a compressed representation
  Sample z → Decoder → new data!

  Training: maximize ELBO = E[log P(x|z)] - KL(q(z|x) || P(z))
  
  Pros:
    ✓ Principled probabilistic framework
    ✓ Smooth latent space → interpolation
    ✓ Fast generation (single forward pass)
  
  Cons:
    ✗ Blurry outputs (due to reconstruction loss)
    ✗ Limited expressiveness of posterior
```

---

## 3. Generative Adversarial Networks (GAN)

```
Two networks compete: Generator creates, Discriminator judges

  z ~ N(0,1) → Generator → fake image
                    ↕ compete
  real image → Discriminator → real or fake?

  Training: minimax game
    min_G max_D  E[log D(x)] + E[log(1-D(G(z)))]

  Pros:
    ✓ Sharp, high-quality images
    ✓ Fast generation (single forward pass)
    ✓ Implicit density (no likelihood needed)
  
  Cons:
    ✗ Training instability
    ✗ Mode collapse (generates limited variety)
    ✗ No density estimation
    ✗ Hard to evaluate
```

---

## 4. Normalizing Flows

```
Chain of invertible transformations from simple to complex:

  z ~ N(0,1) → f₁ → f₂ → ... → f_K → x
  
  Each f_i is invertible and has tractable Jacobian
  Can compute exact P(x) via change of variables!

  Pros:
    ✓ Exact likelihood
    ✓ Exact inference (invertible)
    ✓ Theoretically elegant
  
  Cons:
    ✗ Invertibility constraint limits architecture
    ✗ Expensive Jacobian computation
    ✗ Lower quality than GANs/diffusion
```

---

## 5. Diffusion Models

```
Gradually add noise (forward), learn to remove it (reverse):

  Forward:  x₀ → x₁ → x₂ → ... → x_T ≈ N(0,1)
            (add noise at each step)
  
  Reverse:  x_T → x_{T-1} → ... → x₀
            (learned denoising)

  Pros:
    ✓ State-of-the-art image quality
    ✓ Stable training (no adversarial)
    ✓ Mode coverage (no mode collapse)
    ✓ Flexible conditioning
  
  Cons:
    ✗ Slow generation (many denoising steps)
    ✗ High computational cost
```

---

## Comparison Table

```
  Model         Likelihood  Quality  Speed   Stability  Best For
  ─────         ──────────  ───────  ─────   ─────────  ────────
  Autoregressive Exact      High     Slow    Stable     Text
  VAE            Approx     Medium   Fast    Stable     Latent repr.
  GAN            None       High     Fast    Unstable   Sharp images
  Flow           Exact      Medium   Fast    Stable     Density est.
  Diffusion      Approx     Highest  Slow    Stable     Images/video

  Historical evolution:
  2013: VAE (Kingma)
  2014: GAN (Goodfellow)
  2015: Normalizing Flows (Rezende)
  2020: DDPM / Diffusion (Ho)
  2022: Stable Diffusion, DALL-E 2
  2023: Video diffusion, consistency models
```

---

## Revision Questions

1. **What are the five major families of generative models?**
2. **How do autoregressive models factorize the joint distribution P(x)?**
3. **Why do GANs produce sharper images than VAEs?**
4. **What unique advantage do normalizing flows have?**
5. **Why have diffusion models become dominant for image generation?**

---

[Previous: 01-discriminative-vs-generative.md](01-discriminative-vs-generative.md) | [Next: 03-applications-of-generative-ai.md](03-applications-of-generative-ai.md)
