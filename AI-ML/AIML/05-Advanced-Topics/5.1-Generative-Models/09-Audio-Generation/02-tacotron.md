# Tacotron

## Overview

Tacotron (Wang et al., 2017) and its successor Tacotron 2 (Shen et al., 2018) are **end-to-end text-to-speech (TTS)** systems that convert text directly into mel spectrograms, which are then converted to audio by a vocoder (WaveNet, WaveGlow, or HiFi-GAN). Tacotron 2 combines a sequence-to-sequence model with attention to produce human-quality speech, establishing the modern two-stage TTS paradigm: text → mel spectrogram → waveform.

---

## Text-to-Speech Pipeline

```
Traditional TTS (complex, many components):
  Text → Linguistic analysis → Duration model → 
  Acoustic model → Vocoder → Audio

Modern TTS (Tacotron 2):
  Text → Tacotron 2 → Mel spectrogram → WaveNet/HiFi-GAN → Audio

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "Hello world"                                    │
  │       │                                            │
  │       ▼                                            │
  │  ┌──────────────┐                                 │
  │  │  Tacotron 2    │  Encoder-Decoder with attention│
  │  │  (seq2seq)     │                                │
  │  └───────┬────────┘                                │
  │          │  mel spectrogram (80 bands × T frames) │
  │          ▼                                         │
  │  ┌──────────────┐                                 │
  │  │  Vocoder      │  WaveNet / WaveGlow / HiFi-GAN│
  │  │  (mel → wav)  │                                │
  │  └───────┬────────┘                                │
  │          │                                         │
  │          ▼                                         │
  │  🔊 Audio waveform                                │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Tacotron 2 Architecture

```
Three main components:

  1. Text Encoder
  2. Attention-based Decoder
  3. Post-net (mel spectrogram refinement)

  ┌──────────────────────────────────────────────────────────┐
  │                                                            │
  │  Characters: "H E L L O"                                 │
  │       │                                                    │
  │       ▼                                                    │
  │  ┌──────────────────────────┐                             │
  │  │ Character Embedding (512) │                             │
  │  └────────────┬──────────────┘                             │
  │               ▼                                            │
  │  ┌──────────────────────────┐                             │
  │  │ 3× Conv1D (512, k=5)    │  Extract local patterns    │
  │  │ + BatchNorm + ReLU       │  (n-gram like features)    │
  │  └────────────┬──────────────┘                             │
  │               ▼                                            │
  │  ┌──────────────────────────┐                             │
  │  │ Bi-LSTM (512)            │  Capture sequence context  │
  │  └────────────┬──────────────┘                             │
  │               │ encoder outputs (N × 512)                 │
  │               │                                            │
  │               ▼  Location-Sensitive Attention             │
  │  ┌──────────────────────────┐                             │
  │  │ Decoder (2× LSTM, 1024)  │  Autoregressive            │
  │  │ Previous mel → PreNet     │  One frame at a time      │
  │  │ Attention context         │                             │
  │  └────────────┬──────────────┘                             │
  │               │ mel frame (80-dim) + stop token           │
  │               ▼                                            │
  │  ┌──────────────────────────┐                             │
  │  │ Post-Net (5× Conv1D)     │  Residual refinement       │
  │  └────────────┬──────────────┘                             │
  │               │                                            │
  │               ▼                                            │
  │  Mel spectrogram (80 × T)                                 │
  │                                                            │
  └──────────────────────────────────────────────────────────┘
```

---

## Location-Sensitive Attention

```
Standard attention can skip or repeat characters!
Location-sensitive attention prevents this:

  Uses cumulative attention weights from previous steps
  → model knows where it HAS attended
  → encourages left-to-right monotonic attention

  score_i = w^T tanh(W_s × s_{t-1} + W_h × h_i + W_f × f_i)
  
  f_i = conv1d(cumulative_attention_weights)
  
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Without location:     With location-sensitive:  │
  │                                                    │
  │  H E L L O             H E L L O                │
  │  ▲     ▲               ▲                          │
  │  │     │               │                          │
  │  ├─────┤               ├─→ smooth left to right  │
  │  (skips, repeats!)     (monotonic, reliable!)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Training

```
  Teacher forcing: use ground truth mel frames as input to decoder

  Losses:
    L = MSE(mel_pred, mel_target)       (before post-net)
      + MSE(mel_postnet, mel_target)    (after post-net)
      + BCE(stop_token_pred, stop_target) (end of utterance)

  Training data:
    • LJSpeech: 24 hours single speaker English
    • LibriSpeech: 960 hours multi-speaker
    • VCTK: 44 hours, 109 speakers

  Training details:
    • Adam optimizer, lr = 1e-3 → decay
    • Batch size 64
    • ~300K iterations
    • Teacher forcing ratio: 1.0 (always use ground truth)
```

---

## Modern Vocoders

```
Tacotron produces mel spectrograms. Vocoders convert to audio:

  Vocoder       Type           Speed        Quality
  ──────────    ──────         ─────        ───────
  WaveNet       Autoregressive Very slow    Excellent
  WaveRNN       Autoregressive Moderate     Very good
  WaveGlow      Flow-based     Real-time    Very good
  HiFi-GAN      GAN-based      Real-time    Excellent
  UnivNet       GAN-based      Real-time    Excellent

  HiFi-GAN (2020) is now the standard:
    • Multi-Period Discriminator + Multi-Scale Discriminator
    • Generator: transposed convolutions + multi-receptive field fusion
    • Real-time on CPU!
    • Quality matches WaveNet
```

---

## Beyond Tacotron: Modern TTS

```
  FastSpeech 2 (2021):
    Non-autoregressive → parallel mel generation!
    Duration predictor replaces attention alignment
    Much faster than Tacotron (parallel, not sequential)

  VITS (2021):
    End-to-end: text → audio directly
    Combines VAE + normalizing flow + adversarial training
    No separate vocoder needed!

  Tortoise TTS (2022):
    Autoregressive + diffusion
    Very natural, can clone voices with 5s of audio

  VALL-E (Microsoft, 2023):
    Treats TTS as language modeling!
    Audio codec codes as "tokens"
    3-second voice cloning
```

---

## Python: Tacotron 2 Inference

```python
# Using pretrained Tacotron 2 + HiFi-GAN
import torch

# Load models
tacotron2 = torch.hub.load('NVIDIA/DeepLearningExamples:torchhub',
                            'nvidia_tacotron2', model_math='fp16')
waveglow = torch.hub.load('NVIDIA/DeepLearningExamples:torchhub',
                           'nvidia_waveglow', model_math='fp16')
tacotron2 = tacotron2.to('cuda').eval()
waveglow = waveglow.to('cuda').eval()

# Text processing
from tacotron2.text import text_to_sequence

def synthesize(text, sigma=0.666):
    sequence = text_to_sequence(text, ['english_cleaners'])
    sequence = torch.LongTensor(sequence).unsqueeze(0).cuda()
    
    # Generate mel spectrogram
    with torch.no_grad():
        mel, mel_postnet, _, alignments = tacotron2.infer(sequence)
    
    # Generate audio
    with torch.no_grad():
        audio = waveglow.infer(mel_postnet, sigma=sigma)
    
    return audio.cpu().numpy()[0]

audio = synthesize("Hello, this is a test of text to speech synthesis.")
# Save or play audio
```

---

## Revision Questions

1. **How does Tacotron 2 convert text to mel spectrograms?**
2. **Why is location-sensitive attention important for TTS?**
3. **What is the role of the Post-Net?**
4. **Why did HiFi-GAN replace WaveNet as the standard vocoder?**
5. **How do modern TTS systems like VITS improve on Tacotron?**

---

[Previous: 01-wavenet.md](01-wavenet.md) | [Next: 03-audio-diffusion-models.md](03-audio-diffusion-models.md)
