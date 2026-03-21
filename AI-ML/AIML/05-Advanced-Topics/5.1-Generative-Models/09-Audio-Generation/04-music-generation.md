# Music Generation

## Overview

AI music generation has progressed from simple MIDI composition to generating full, high-fidelity music from text descriptions. Modern systems like **MusicGen** (Meta), **MusicLM** (Google), **Jukebox** (OpenAI), and **Riffusion** use large language models, neural audio codecs, and diffusion models to create music that captures melody, harmony, rhythm, and instrumentation. This chapter covers the key architectures and approaches.

---

## Challenges of Music Generation

```
Music is more complex than speech:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Speech:                   Music:                 │
  │  • 1 source (voice)       • Many instruments     │
  │  • Sequential structure   • Polyphonic (harmony)  │
  │  • 1-10 sec context       • Long structure (min.) │
  │  • Phonetic rules exist   • Subjective quality   │
  │  • 16kHz sufficient       • 44.1kHz needed       │
  │                                                    │
  │  Music-specific challenges:                      │
  │  1. Long-range structure (verse-chorus-verse)    │
  │  2. Multiple instruments playing simultaneously  │
  │  3. Harmony and chord progressions               │
  │  4. Rhythm and meter consistency                 │
  │  5. Style coherence (genre, mood)                │
  │  6. Very long sequences (3-5 minutes!)           │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Symbolic vs. Audio Generation

```
Two approaches:

  1. Symbolic (MIDI / sheet music):
     Generate note events: (pitch, duration, velocity)
     Then synthesize with a sound engine
     
     + Small vocabulary (128 pitches × durations)
     + Easy to edit, transpose, rearrange
     - Limited to MIDI instruments
     - No expression, timbre, production quality

  2. Audio (waveform / spectrogram):
     Generate raw audio directly
     
     + Full production quality
     + Captures timbre, expression, effects
     + Can mix instruments naturally
     - Very high dimensional
     - Hard to edit individual notes
     - Requires more data and compute

  Modern trend: audio generation dominates!
```

---

## Neural Audio Codecs

```
Key enabler for audio language models:

  EnCodec (Meta), SoundStream (Google):
  Compress audio into discrete tokens!

  Audio → Encoder → Quantize → Discrete tokens
  Tokens → Dequantize → Decoder → Audio

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Audio (24kHz, 1 sec = 24000 samples)            │
  │       │                                            │
  │       ▼  Encoder (strided convolutions)           │
  │  Continuous latent (75 frames/sec)               │
  │       │                                            │
  │       ▼  Residual Vector Quantization (RVQ)      │
  │  Discrete tokens: 75 tokens/sec × 8 codebooks   │
  │       │                                            │
  │  Compare: 24000 samples → 600 tokens (40× less!) │
  │                                                    │
  │  RVQ: 8 codebooks, each 1024 entries             │
  │  Book 1: coarse structure (most important)       │
  │  Book 8: fine details (least important)          │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Now audio = sequence of tokens → use language models!
```

---

## MusicGen (Meta)

```
MusicGen (Copet et al., 2023):
  Text-conditional music generation using a transformer LM

  Architecture:
  1. T5 text encoder → text conditioning
  2. Transformer LM → generates EnCodec tokens
  3. EnCodec decoder → audio

  Efficient token generation:
  Instead of predicting 8 codebooks autoregressively:
  
  Delay pattern: predict codebooks with time delays
    Book 1: t=1, t=2, t=3, ...
    Book 2:       t=1, t=2, t=3, ...  (delayed by 1)
    Book 3:             t=1, t=2, ...  (delayed by 2)
    ...
  
  Allows parallel prediction across codebooks!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "epic orchestral music, dramatic"               │
  │       │                                            │
  │       ▼                                            │
  │  T5 encoder → text embedding                     │
  │       │ (cross-attention)                         │
  │       ▼                                            │
  │  Transformer LM (1.5B params)                    │
  │  Generates EnCodec tokens with delay pattern     │
  │       │                                            │
  │       ▼                                            │
  │  EnCodec decoder → 🎵 30 seconds of music       │
  │                                                    │
  │  Sizes: Small (300M), Medium (1.5B), Large (3.3B)│
  └──────────────────────────────────────────────────┘
```

---

## MusicLM (Google)

```
MusicLM (Agostinelli et al., 2023):
  Uses hierarchical tokens from multiple models

  Three token types:
  1. MuLan tokens: semantic (music understanding)
  2. w2v-BERT tokens: acoustic (sound features)
  3. SoundStream tokens: fine audio details

  Generation cascades through stages:
    Text → MuLan tokens → w2v-BERT tokens → SoundStream tokens → Audio

  Can also condition on:
  • Hummed melody → generate full arrangement
  • Audio prompt → continue in same style
  • Story/description → soundtrack generation
```

---

## Jukebox (OpenAI)

```
Jukebox (2020): generates raw audio with singing!

  VQ-VAE with 3 levels:
    Level 1: 8× compression (most detail)
    Level 2: 32× compression
    Level 3: 128× compression (most abstract)

  Autoregressive transformers at each level:
    Top level (128×): captures long-range song structure
    Middle (32×): adds melodic/harmonic detail
    Bottom (8×): adds audio quality

  Conditioning: genre, artist, lyrics
  → Generates actual singing with lyrics!

  Limitations:
  • Very slow (hours for 1 minute of audio)
  • Quality below production level
  • But groundbreaking for raw audio generation
```

---

## Riffusion

```
Creative approach: generate music as SPECTROGRAMS!

  Use Stable Diffusion (image model) fine-tuned on spectrograms:
  
  Text → Stable Diffusion → spectrogram image → audio

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  "jazz piano solo"                                │
  │       │                                            │
  │       ▼                                            │
  │  Stable Diffusion (fine-tuned)                   │
  │       │                                            │
  │       ▼                                            │
  │  ┌──────────────────┐                             │
  │  │ ▒▒▓▓██▒▒        │  ← mel spectrogram image   │
  │  │ ▒▓▓██▒▒▒        │                              │
  │  │ ▓██▒▒▒▒▒        │                              │
  │  └──────────────────┘                             │
  │       │                                            │
  │       ▼  Griffin-Lim / vocoder                   │
  │  🎵 Audio                                        │
  │                                                    │
  │  Simple but creative approach!                   │
  │  Interpolation between prompts → smooth music    │
  └──────────────────────────────────────────────────┘
```

---

## Python: Using MusicGen

```python
from transformers import AutoProcessor, MusicgenForConditionalGeneration
import torch
import scipy.io.wavfile

# Load MusicGen
processor = AutoProcessor.from_pretrained("facebook/musicgen-small")
model = MusicgenForConditionalGeneration.from_pretrained(
    "facebook/musicgen-small")

# Text-to-music generation
inputs = processor(
    text=["upbeat electronic dance music with heavy bass"],
    padding=True,
    return_tensors="pt",
)

# Generate 10 seconds of audio at 32kHz
audio_values = model.generate(
    **inputs,
    max_new_tokens=256,  # ~8 seconds
    do_sample=True,
    guidance_scale=3.0,
)

# Save
sample_rate = model.config.audio_encoder.sampling_rate
audio_np = audio_values[0, 0].cpu().numpy()
scipy.io.wavfile.write("generated_music.wav", sample_rate, audio_np)

# Melody-conditioned generation
import torchaudio

melody_waveform, sr = torchaudio.load("hummed_melody.wav")
melody_waveform = torchaudio.functional.resample(melody_waveform, sr, 32000)

inputs = processor(
    text=["orchestral version of this melody"],
    audio=melody_waveform,
    sampling_rate=32000,
    padding=True,
    return_tensors="pt",
)
audio = model.generate(**inputs, max_new_tokens=512)
```

---

## Revision Questions

1. **Why is music generation harder than speech generation?**
2. **How do neural audio codecs (EnCodec) enable language model-based music generation?**
3. **What is MusicGen's delay pattern and why is it efficient?**
4. **How does Jukebox generate singing with lyrics?**
5. **What is Riffusion's creative approach to music generation?**

---

[Previous: 03-audio-diffusion-models.md](03-audio-diffusion-models.md) | [Next: 05-voice-cloning.md](05-voice-cloning.md)
