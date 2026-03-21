# Speech Recognition Basics

> **Unit 8, Chapter 6** — Audio features (spectrograms, MFCCs), CTC loss, and end-to-end speech recognition overview.

---

## 📋 Overview

**Automatic Speech Recognition (ASR)** converts audio waveforms into text. RNNs process sequential audio features (spectrograms or MFCCs) and produce character or word sequences. The key challenge is that the alignment between audio frames and text characters is unknown.

---

## 📊 Audio Feature Pipeline

```
Raw Audio → Pre-emphasis → Windowing → FFT → Mel Filter → Log → DCT → MFCC
                                        │
                                        ▼
                                   Spectrogram

┌─────────────────────────────────────────────────────────┐
│  1. WAVEFORM → Raw audio samples (e.g., 16kHz)        │
│     [0.01, -0.02, 0.05, 0.03, ...]                    │
│                                                         │
│  2. SPECTROGRAM → Time-frequency representation        │
│     Each column = frequency content at one time frame   │
│     Shape: (n_freq_bins, n_time_frames)                │
│                                                         │
│     Freq │████░░░░░░│                                   │
│          │██████░░░░│                                   │
│          │████████░░│                                   │
│          │██████████│                                   │
│          └──────────┘                                   │
│           Time frames                                   │
│                                                         │
│  3. MFCCs → Compact frequency features (13-40 dims)    │
│     Better for speech recognition                      │
│     Captures vocal tract characteristics               │
└─────────────────────────────────────────────────────────┘
```

---

## 🧮 CTC Loss (Connectionist Temporal Classification)

```
Problem: Audio has ~100 frames/sec, but text has ~5 chars/sec
         Alignment between frames and characters is UNKNOWN!

CTC solution: Allow model to output "blank" tokens
  Audio frames: f₁  f₂  f₃  f₄  f₅  f₆  f₇  f₈  f₉  f₁₀
  CTC output:   h   -   -   e   e   -   l   l   -   o
  Collapsed:    h           e           l           o    → "helo"

  Rules:
  1. Remove consecutive duplicates
  2. Remove blank tokens (-)
  
  Multiple paths can produce the same output:
  "hh--ee-llo" → "hello"
  "h-e-l-l-o-" → "hello"
  "-h-el-lo--" → "hello"
  
  CTC loss = -log(sum of probabilities of ALL valid paths)
```

---

## 🏗️ End-to-End ASR Architecture

```
Audio input (waveform)
       │
       ▼
┌──────────────┐
│ Feature       │ → Spectrogram / MFCC
│ Extraction    │   (T frames × D features)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Deep BiLSTM   │ → Process temporal features
│ (3-5 layers)  │   (T × hidden_dim)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ FC + Softmax  │ → Per-frame character probabilities
│               │   (T × |alphabet|+1) including blank
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ CTC Decoding  │ → Beam search with CTC
│ / Beam Search │   Collapses to final text
└──────┬───────┘
       │
       ▼
"hello world"
```

---

## 💻 PyTorch Example

```python
import torch
import torch.nn as nn

class SimpleASR(nn.Module):
    def __init__(self, input_dim=80, hidden_dim=256, num_chars=29, num_layers=3):
        super().__init__()
        # num_chars: 26 letters + space + apostrophe + blank
        self.lstm = nn.LSTM(input_dim, hidden_dim, num_layers,
                            bidirectional=True, batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_dim * 2, num_chars)
    
    def forward(self, x):
        """x: (batch, time_frames, features)"""
        output, _ = self.lstm(x)
        logits = self.fc(output)  # (batch, time_frames, num_chars)
        return logits.log_softmax(dim=2)

# CTC Loss
model = SimpleASR()
ctc_loss = nn.CTCLoss(blank=0, zero_infinity=True)

# Simulated audio: 32 samples, 200 frames, 80 mel features
audio = torch.randn(32, 200, 80)
log_probs = model(audio)  # (32, 200, 29)

# CTC requires: (T, N, C) format
log_probs = log_probs.permute(1, 0, 2)  # (200, 32, 29)
targets = torch.randint(1, 29, (32, 15))  # Target text indices
input_lengths = torch.full((32,), 200)
target_lengths = torch.randint(5, 15, (32,))

loss = ctc_loss(log_probs, targets, input_lengths, target_lengths)
print(f"CTC Loss: {loss.item():.4f}")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Audio Features | Spectrograms or MFCCs from raw waveform |
| CTC Loss | Handles unknown alignment with blank tokens |
| Architecture | Deep BiLSTM + FC + CTC |
| Decoding | Beam search with CTC collapsing |
| Frame Rate | ~100 frames/sec → much longer than text |
| Modern Systems | Transformer-based (Whisper, wav2vec 2.0) |

---

## ❓ Revision Questions

1. **Why can't we simply align each audio frame to one character?**
2. **What is the role of the blank token in CTC? Give an example of collapsing.**
3. **Why are MFCCs preferred over raw waveforms as input to ASR models?**
4. **How does CTC handle the fact that multiple alignments produce the same text?**
5. **What is the typical architecture depth for production ASR systems?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Machine Translation](05-machine-translation.md) |
| ➡️ Next | [Time Series Forecasting](07-time-series-forecasting.md) |
| ⬆️ Unit Overview | [README](../README.md) |
