# 5.1 Generative Models

## Course Overview

This subject covers the theory and practice of **generative models** — algorithms that learn to create new data (images, text, audio, video) by modeling the underlying data distribution. From classical autoencoders to modern diffusion models, this course explores the architectures, training procedures, and applications that power today's generative AI revolution.

---

## Table of Contents

### Unit 1: Introduction to Generative Models
- [Discriminative vs Generative Models](01-Introduction-to-Generative-Models/01-discriminative-vs-generative.md)
- [Types of Generative Models](01-Introduction-to-Generative-Models/02-types-of-generative-models.md)
- [Applications of Generative AI](01-Introduction-to-Generative-Models/03-applications-of-generative-ai.md)
- [Evaluation of Generative Models](01-Introduction-to-Generative-Models/04-evaluation-of-generative-models.md)

### Unit 2: Autoencoders
- [Autoencoder Architecture](02-Autoencoders/01-autoencoder-architecture.md)
- [Encoder and Decoder](02-Autoencoders/02-encoder-and-decoder.md)
- [Latent Space](02-Autoencoders/03-latent-space.md)
- [Undercomplete Autoencoders](02-Autoencoders/04-undercomplete-autoencoders.md)
- [Denoising Autoencoders](02-Autoencoders/05-denoising-autoencoders.md)
- [Sparse Autoencoders](02-Autoencoders/06-sparse-autoencoders.md)
- [Applications](02-Autoencoders/07-applications.md)

### Unit 3: Variational Autoencoders (VAE)
- [VAE Motivation](03-Variational-Autoencoders/01-vae-motivation.md)
- [Latent Variable Models](03-Variational-Autoencoders/02-latent-variable-models.md)
- [Evidence Lower Bound (ELBO)](03-Variational-Autoencoders/03-elbo.md)
- [Reparameterization Trick](03-Variational-Autoencoders/04-reparameterization-trick.md)
- [VAE Architecture](03-Variational-Autoencoders/05-vae-architecture.md)
- [β-VAE](03-Variational-Autoencoders/06-beta-vae.md)
- [VAE Applications](03-Variational-Autoencoders/07-vae-applications.md)

### Unit 4: Generative Adversarial Networks (GAN)
- [GAN Concept](04-Generative-Adversarial-Networks/01-gan-concept.md)
- [Generator and Discriminator](04-Generative-Adversarial-Networks/02-generator-and-discriminator.md)
- [Adversarial Training](04-Generative-Adversarial-Networks/03-adversarial-training.md)
- [GAN Loss Functions](04-Generative-Adversarial-Networks/04-gan-loss-functions.md)
- [Training Challenges](04-Generative-Adversarial-Networks/05-training-challenges.md)
- [GAN Training Tips](04-Generative-Adversarial-Networks/06-gan-training-tips.md)

### Unit 5: GAN Variants
- [DCGAN](05-GAN-Variants/01-dcgan.md)
- [WGAN and WGAN-GP](05-GAN-Variants/02-wgan-and-wgan-gp.md)
- [Conditional GAN (cGAN)](05-GAN-Variants/03-conditional-gan.md)
- [Progressive GAN](05-GAN-Variants/04-progressive-gan.md)
- [StyleGAN](05-GAN-Variants/05-stylegan.md)
- [CycleGAN](05-GAN-Variants/06-cyclegan.md)
- [Pix2Pix](05-GAN-Variants/07-pix2pix.md)

### Unit 6: Normalizing Flows
- [Flow-Based Models Concept](06-Normalizing-Flows/01-flow-based-models-concept.md)
- [Change of Variables](06-Normalizing-Flows/02-change-of-variables.md)
- [Invertible Transformations](06-Normalizing-Flows/03-invertible-transformations.md)
- [RealNVP](06-Normalizing-Flows/04-realnvp.md)
- [Glow](06-Normalizing-Flows/05-glow.md)
- [Applications](06-Normalizing-Flows/06-applications.md)

### Unit 7: Diffusion Models
- [Diffusion Process](07-Diffusion-Models/01-diffusion-process.md)
- [Forward and Reverse Process](07-Diffusion-Models/02-forward-and-reverse-process.md)
- [DDPM](07-Diffusion-Models/03-ddpm.md)
- [Score Matching](07-Diffusion-Models/04-score-matching.md)
- [Noise Scheduling](07-Diffusion-Models/05-noise-scheduling.md)
- [DDIM (Faster Sampling)](07-Diffusion-Models/06-ddim.md)

### Unit 8: Stable Diffusion and DALL-E
- [Latent Diffusion Models](08-Stable-Diffusion-and-DALLE/01-latent-diffusion-models.md)
- [Text-to-Image Generation](08-Stable-Diffusion-and-DALLE/02-text-to-image-generation.md)
- [CLIP for Text-Image Alignment](08-Stable-Diffusion-and-DALLE/03-clip-text-image-alignment.md)
- [Classifier-Free Guidance](08-Stable-Diffusion-and-DALLE/04-classifier-free-guidance.md)
- [ControlNet](08-Stable-Diffusion-and-DALLE/05-controlnet.md)
- [Inpainting and Outpainting](08-Stable-Diffusion-and-DALLE/06-inpainting-and-outpainting.md)

### Unit 9: Audio Generation
- [WaveNet](09-Audio-Generation/01-wavenet.md)
- [Tacotron](09-Audio-Generation/02-tacotron.md)
- [Audio Diffusion Models](09-Audio-Generation/03-audio-diffusion-models.md)
- [Music Generation](09-Audio-Generation/04-music-generation.md)
- [Voice Cloning](09-Audio-Generation/05-voice-cloning.md)

### Unit 10: Video Generation
- [Video Prediction](10-Video-Generation/01-video-prediction.md)
- [Video Synthesis](10-Video-Generation/02-video-synthesis.md)
- [Temporal Consistency](10-Video-Generation/03-temporal-consistency.md)
- [Current State of Video Generation](10-Video-Generation/04-current-state-of-video-generation.md)

---

## Key Libraries

| Library | Purpose |
|---------|---------|
| PyTorch | Core deep learning framework |
| torchvision | Image datasets and transforms |
| diffusers (HuggingFace) | Diffusion model pipelines |
| torchaudio | Audio processing |
| CLIP | Text-image alignment |

---

## Prerequisites

- Deep learning fundamentals (neural networks, CNNs)
- Probability and statistics (distributions, Bayes' theorem, KL divergence)
- PyTorch proficiency
- Linear algebra (matrix operations, eigenvalues)
