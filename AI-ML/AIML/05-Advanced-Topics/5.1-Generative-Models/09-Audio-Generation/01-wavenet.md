# WaveNet

## Overview

WaveNet (van den Oord et al., 2016) is a deep generative model for raw audio waveforms that produces speech of unprecedented naturalness. It models audio sample-by-sample using **autoregressive** generation with **dilated causal convolutions**, capturing both short-range (individual sound waves) and long-range (words, prosody) dependencies. WaveNet was the first neural TTS system to match human speech quality and directly influenced all subsequent neural audio models.

---

## The Challenge of Audio Generation

```
Raw audio is EXTREMELY high-dimensional:

  Sample rate: 16,000 - 44,100 Hz
  → 16,000 to 44,100 values per SECOND
  → 1 second of audio = 16,000+ dimensional vector!

  Each sample: 16-bit integer (-32768 to 32767)
  or 8-bit μ-law encoded (256 values)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Audio waveform (zoomed in):                     │
  │                                                    │
  │    ╱╲   ╱╲   ╱╲                                  │
  │  ─╱──╲─╱──╲─╱──╲─  ← individual samples         │
  │       ╲╱   ╲╱   ╲╱                               │
  │                                                    │
  │  Each point = one sample = one prediction         │
  │  WaveNet predicts EACH sample one at a time      │
  │                                                    │
  │  Need to capture:                                 │
  │  • Micro: individual wave oscillations (ms)      │
  │  • Macro: words, sentences, prosody (seconds)    │
  │  • Context: speaker identity, emotion            │
  └──────────────────────────────────────────────────┘
```

---

## Autoregressive Model

```
Predict each sample given all previous samples:

  p(x) = Π p(x_t | x₁, ..., x_{t-1})

  At each step: predict probability over 256 values (μ-law)
  → 256-way classification for each audio sample!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  x₁ → predict x₂                                │
  │  x₁, x₂ → predict x₃                           │
  │  x₁, x₂, x₃ → predict x₄                      │
  │  ...                                               │
  │  x₁, ..., x_{t-1} → predict x_t                 │
  │                                                    │
  │  Training: teacher forcing (use true x values)   │
  │  Generation: feed back each prediction (slow!)   │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Causal Dilated Convolutions

```
Problem: need HUGE receptive field (thousands of samples)
  Regular convolution kernel=3: receptive field grows linearly
  Need ~16000 layers for 1 second of audio!

Solution: DILATED convolutions — exponentially growing receptive field

  Dilation rates: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512

  Layer 1 (dilation=1):   ▪ ▪ ▪
  Layer 2 (dilation=2):   ▪ . ▪ . ▪
  Layer 3 (dilation=4):   ▪ . . . ▪ . . . ▪
  Layer 4 (dilation=8):   ▪ . . . . . . . ▪ . . . . . . . ▪

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Output          ●                                │
  │  d=8        ●─────────●                           │
  │  d=4      ●────●    ●────●                       │
  │  d=2    ●──●  ●──●  ●──●  ●──●                  │
  │  d=1   ●●  ●●  ●●  ●●  ●●  ●●  ●●  ●●          │
  │  Input ● ● ● ● ● ● ● ● ● ● ● ● ● ● ● ●        │
  │                                                    │
  │  10 layers → receptive field = 1024 samples      │
  │  = 64ms at 16kHz                                  │
  │  Stack multiple blocks: 30 layers → ~16K samples │
  │  = 1 full second of audio!                       │
  └──────────────────────────────────────────────────┘

  CAUSAL: only look at past (not future) samples
  → Can generate left-to-right
```

---

## Gated Activation

```
Each dilated conv layer uses gated activation:

  z = tanh(W_f * x) ⊙ σ(W_g * x)

  W_f: filter weights (what to generate)
  W_g: gate weights (how much to let through)
  tanh: output range [-1, 1]
  σ: gate range [0, 1]
  ⊙: element-wise multiplication

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  input x                                          │
  │    ├──→ dilated conv (filter) → tanh ──→ ⊙ → z  │
  │    └──→ dilated conv (gate)   → σ    ──↗         │
  │                                                    │
  │  Then: residual + skip connections               │
  │    residual = z + x  (passed to next layer)      │
  │    skip = z          (collected for output)       │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Conditioning

```
WaveNet can be conditioned on:

  Global conditioning (same for all timesteps):
    • Speaker identity (embedding)
    → One model, multiple voices!

  Local conditioning (varies with time):
    • Linguistic features (phonemes, duration)
    • Mel spectrogram (for vocoder use)
    
  Conditioning is ADDED to the dilated conv output:
    z = tanh(W_f * x + V_f * h) ⊙ σ(W_g * x + V_g * h)
    
    h = conditioning vector (upsampled to audio rate)

  TTS pipeline:
    Text → acoustic model → mel spectrogram → WaveNet → audio
```

---

## Python: WaveNet Block

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CausalConv1d(nn.Module):
    def __init__(self, in_ch, out_ch, kernel_size, dilation=1):
        super().__init__()
        self.padding = (kernel_size - 1) * dilation
        self.conv = nn.Conv1d(in_ch, out_ch, kernel_size, 
                             dilation=dilation)
    
    def forward(self, x):
        x = F.pad(x, (self.padding, 0))  # left pad only (causal!)
        return self.conv(x)

class WaveNetBlock(nn.Module):
    def __init__(self, residual_ch, skip_ch, dilation, cond_ch=0):
        super().__init__()
        self.filter_conv = CausalConv1d(residual_ch, residual_ch, 
                                         kernel_size=2, dilation=dilation)
        self.gate_conv = CausalConv1d(residual_ch, residual_ch,
                                       kernel_size=2, dilation=dilation)
        self.residual_conv = nn.Conv1d(residual_ch, residual_ch, 1)
        self.skip_conv = nn.Conv1d(residual_ch, skip_ch, 1)
        
        if cond_ch > 0:
            self.cond_filter = nn.Conv1d(cond_ch, residual_ch, 1)
            self.cond_gate = nn.Conv1d(cond_ch, residual_ch, 1)
        self.cond_ch = cond_ch
    
    def forward(self, x, condition=None):
        f = self.filter_conv(x)
        g = self.gate_conv(x)
        
        if condition is not None and self.cond_ch > 0:
            f = f + self.cond_filter(condition)
            g = g + self.cond_gate(condition)
        
        z = torch.tanh(f) * torch.sigmoid(g)
        
        skip = self.skip_conv(z)
        residual = self.residual_conv(z) + x
        return residual, skip

class WaveNet(nn.Module):
    def __init__(self, layers=10, blocks=3, residual_ch=64, 
                 skip_ch=256, classes=256):
        super().__init__()
        self.input_conv = nn.Conv1d(classes, residual_ch, 1)
        
        self.blocks = nn.ModuleList()
        for b in range(blocks):
            for l in range(layers):
                dilation = 2 ** l
                self.blocks.append(WaveNetBlock(residual_ch, skip_ch, dilation))
        
        self.output = nn.Sequential(
            nn.ReLU(),
            nn.Conv1d(skip_ch, skip_ch, 1),
            nn.ReLU(),
            nn.Conv1d(skip_ch, classes, 1),
        )
    
    def forward(self, x):
        # x: one-hot encoded (batch, 256, time)
        h = self.input_conv(x)
        skip_total = 0
        
        for block in self.blocks:
            h, skip = block(h)
            skip_total = skip_total + skip
        
        return self.output(skip_total)
```

---

## Limitations and Successors

```
  WaveNet limitations:
  • Autoregressive → VERY slow generation
    16kHz audio: 16,000 sequential forward passes per second!
    ~minutes to generate 1 second of audio

  Successors that fixed speed:
  • Parallel WaveNet: teacher-student distillation
  • WaveGlow: flow-based, parallel generation
  • HiFi-GAN: GAN-based vocoder, real-time
  • WaveRNN: efficient single-layer RNN
```

---

## Revision Questions

1. **Why is raw audio generation harder than image generation?**
2. **How do dilated convolutions achieve large receptive fields efficiently?**
3. **Why must WaveNet convolutions be causal?**
4. **What is the gated activation unit and why is it used?**
5. **Why is WaveNet generation slow and how did successors fix this?**

---

[Previous: ../08-Stable-Diffusion-and-DALLE/06-inpainting-and-outpainting.md](../08-Stable-Diffusion-and-DALLE/06-inpainting-and-outpainting.md) | [Next: 02-tacotron.md](02-tacotron.md)
