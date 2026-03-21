# Audio Diffusion Models

## Overview

Diffusion models have been adapted for audio generation, achieving state-of-the-art results in speech synthesis, sound effects, and music. These models apply the forward/reverse diffusion process to audio representations — either raw waveforms or spectrograms. Key systems include **DiffWave**, **WaveGrad**, **Grad-TTS**, and **AudioLDM**, which bring the quality and flexibility of image diffusion models to the audio domain.

---

## Diffusion for Audio

```
Same principle as image diffusion, applied to audio:

  Forward:  clean audio → gradually add noise → pure noise
  Reverse:  noise → iteratively denoise → clean audio

  Two approaches:

  1. Waveform diffusion (DiffWave, WaveGrad):
     Diffuse directly on raw waveform samples
     x_t ∈ ℝ^(samples)  (e.g., 16000 per second)

  2. Spectrogram diffusion (Grad-TTS, AudioLDM):
     Diffuse on mel spectrogram representation
     z_t ∈ ℝ^(mel_bins × time_frames)
     Then use vocoder (HiFi-GAN) to convert to audio

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Waveform diffusion:                             │
  │    + Direct audio, no vocoder artifacts          │
  │    - Very high dimensional (16K samples/sec)     │
  │    - Slow sampling                               │
  │                                                    │
  │  Spectrogram diffusion:                          │
  │    + Lower dimensional (80×100 per second)       │
  │    + Faster training and sampling                │
  │    - Needs separate vocoder                      │
  │    - Vocoder can introduce artifacts             │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## DiffWave

```
DiffWave (Kong et al., 2021): diffusion on raw waveforms

  Architecture: WaveNet-like with dilated convolutions
  But NOT autoregressive — bidirectional!
  
  Key: uses the SAME architecture for ALL timesteps
  Timestep t embedded via sinusoidal encoding + FiLM

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Input: x_t (noisy waveform) + t (timestep)     │
  │  Optional: c (mel spectrogram conditioning)      │
  │                                                    │
  │  x_t → Conv1D → [ResBlock × 30] → Conv1D → ε̂  │
  │                      ↑                            │
  │                    t embedding (FiLM)             │
  │                    c conditioning                 │
  │                                                    │
  │  ResBlock: dilated conv + gated activation       │
  │           + skip connections (like WaveNet)       │
  │           + timestep conditioning via FiLM        │
  │                                                    │
  │  FiLM: h = γ(t) × h + β(t)                      │
  │         (scale and shift by timestep embedding)   │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Results:
  • Quality matches autoregressive WaveNet
  • 5× faster synthesis than WaveNet
  • Unconditional generation of diverse sounds
  • Conditional: mel → waveform (vocoder mode)
```

---

## Grad-TTS

```
Grad-TTS (Popov et al., 2021): diffusion for TTS

  Text → Encoder → Alignment → Diffusion decoder → Mel → Vocoder → Audio

  Key innovation: score-based decoder
  Instead of predicting mel directly, use diffusion!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "Hello world"                                    │
  │       │                                            │
  │       ▼                                            │
  │  Text Encoder (Transformer)                      │
  │       │                                            │
  │       ▼                                            │
  │  Duration Predictor → aligned features           │
  │       │                                            │
  │       ▼                                            │
  │  μ (predicted mean mel spectrogram)              │
  │       │                                            │
  │       ▼                                            │
  │  Score-based Diffusion Decoder:                  │
  │    Start from μ + noise                          │
  │    Iteratively refine toward natural mel          │
  │    Using U-Net to predict score                  │
  │       │                                            │
  │       ▼                                            │
  │  Refined mel spectrogram                         │
  │       │                                            │
  │       ▼                                            │
  │  HiFi-GAN vocoder → Audio                       │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Advantage: can trade quality for speed
    More diffusion steps → better quality
    Fewer steps → faster but still good
```

---

## AudioLDM

```
AudioLDM (Liu et al., 2023): latent diffusion for audio

  Same idea as Stable Diffusion but for audio:
  1. Audio → mel spectrogram → VAE encoder → latent
  2. Diffusion in latent space (conditioned on text/CLAP)
  3. VAE decoder → mel spectrogram → vocoder → audio

  Text conditioning via CLAP (audio version of CLIP):
    CLAP: Contrastive Language-Audio Pretraining
    Learns shared text-audio embedding space

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "birds chirping in a forest"                    │
  │       │                                            │
  │       ▼                                            │
  │  CLAP text encoder → text embedding              │
  │       │                                            │
  │       ▼ (cross-attention)                         │
  │  Latent Diffusion (U-Net on latent mel)          │
  │       │                                            │
  │       ▼                                            │
  │  VAE Decoder → mel spectrogram                   │
  │       │                                            │
  │       ▼                                            │
  │  HiFi-GAN → 🔊 audio                            │
  │                                                    │
  │  Generates: sound effects, music, speech         │
  │  From: text descriptions, audio prompts          │
  └──────────────────────────────────────────────────┘

  AudioLDM 2: unified model for speech, music, sound effects
```

---

## Comparison of Audio Diffusion Models

```
  Model         Domain        Input          Output
  ─────         ──────        ─────          ──────
  DiffWave      Vocoder       Mel spec       Waveform
  WaveGrad      Vocoder       Mel spec       Waveform
  Grad-TTS      TTS           Text           Mel → audio
  AudioLDM      General       Text/audio     Audio
  Make-An-Audio General       Text           Audio
  Noise2Music   Music         Text           Music
  MusicLDM      Music         Text           Music
```

---

## Python: Simple Audio Diffusion

```python
import torch
import torch.nn as nn

class AudioDiffusionBlock(nn.Module):
    """Residual block for 1D audio diffusion."""
    def __init__(self, channels, dilation, time_dim):
        super().__init__()
        self.dilated_conv = nn.Conv1d(channels, channels * 2, 
                                       kernel_size=3, dilation=dilation,
                                       padding=dilation)
        self.time_proj = nn.Linear(time_dim, channels)
        self.residual_conv = nn.Conv1d(channels, channels, 1)
        self.skip_conv = nn.Conv1d(channels, channels, 1)
    
    def forward(self, x, t_emb):
        h = x + self.time_proj(t_emb).unsqueeze(-1)
        h = self.dilated_conv(h)
        gate, filt = h.chunk(2, dim=1)
        h = torch.tanh(filt) * torch.sigmoid(gate)
        
        skip = self.skip_conv(h)
        residual = self.residual_conv(h) + x
        return residual, skip

class AudioDiffusionModel(nn.Module):
    def __init__(self, audio_channels=1, model_channels=64,
                 num_layers=30, time_dim=128):
        super().__init__()
        self.input_conv = nn.Conv1d(audio_channels, model_channels, 1)
        
        self.time_embed = nn.Sequential(
            nn.Linear(time_dim, time_dim * 4),
            nn.SiLU(),
            nn.Linear(time_dim * 4, time_dim),
        )
        
        self.blocks = nn.ModuleList()
        for i in range(num_layers):
            dilation = 2 ** (i % 10)
            self.blocks.append(
                AudioDiffusionBlock(model_channels, dilation, time_dim))
        
        self.output = nn.Sequential(
            nn.ReLU(),
            nn.Conv1d(model_channels, model_channels, 1),
            nn.ReLU(),
            nn.Conv1d(model_channels, audio_channels, 1),
        )
    
    def forward(self, x_t, t):
        t_emb = self.time_embed(self.sinusoidal_embed(t))
        h = self.input_conv(x_t)
        skip_sum = 0
        
        for block in self.blocks:
            h, skip = block(h, t_emb)
            skip_sum = skip_sum + skip
        
        return self.output(skip_sum)
    
    def sinusoidal_embed(self, t, dim=128):
        half = dim // 2
        emb = torch.exp(torch.arange(half, device=t.device) * 
                        -(torch.log(torch.tensor(10000.0)) / half))
        emb = t.float().unsqueeze(1) * emb.unsqueeze(0)
        return torch.cat([torch.sin(emb), torch.cos(emb)], dim=1)
```

---

## Revision Questions

1. **What are the two main approaches to audio diffusion (waveform vs. spectrogram)?**
2. **How does DiffWave differ from WaveNet architecturally?**
3. **What is CLAP and how does it enable text-to-audio generation?**
4. **How does Grad-TTS use diffusion for text-to-speech?**
5. **Why is latent diffusion (AudioLDM) more practical than waveform diffusion?**

---

[Previous: 02-tacotron.md](02-tacotron.md) | [Next: 04-music-generation.md](04-music-generation.md)
