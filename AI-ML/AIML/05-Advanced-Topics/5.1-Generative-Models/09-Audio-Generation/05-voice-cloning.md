# Voice Cloning

## Overview

Voice cloning is the task of replicating a person's voice so that a text-to-speech (TTS) system can speak any text in that person's voice. Modern voice cloning systems can achieve this with as little as **3–10 seconds** of reference audio. This chapter covers zero-shot and few-shot voice cloning architectures, speaker embedding methods, and the ethical implications.

---

## How Voice Cloning Works

```
Traditional TTS vs. Voice Cloning:

  Traditional TTS (single speaker):
    Text → TTS model → Audio in model's voice

  Voice Cloning (any speaker):
    Text + Reference audio → TTS model → Audio in cloned voice

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Reference audio: "Hi, my name is Alice"          │
  │       │                                            │
  │       ▼                                            │
  │  Speaker Encoder → speaker embedding (256-d)     │
  │       │                                            │
  │  Text: "The weather is beautiful today"           │
  │       │                                            │
  │       ▼                                            │
  │  TTS Synthesizer (conditioned on speaker emb.)   │
  │       │                                            │
  │       ▼                                            │
  │  Audio: "The weather is beautiful today"          │
  │         (in Alice's voice!)                       │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Speaker Embeddings

```
The key to voice cloning: extracting speaker identity

  Speaker Encoder:
    • Trained on speaker verification / recognition tasks
    • Maps any audio → fixed-size speaker embedding
    • Captures: timbre, pitch range, speaking style
    • Discards: words spoken, background noise

  Common architectures:
  ┌─────────────────────────────────────────────┐
  │                                               │
  │  1. d-vector (GE2E loss):                    │
  │     LSTM on mel frames → average pooling     │
  │     → 256-d vector                            │
  │                                               │
  │  2. x-vector:                                │
  │     TDNN layers → statistics pooling          │
  │     → 512-d vector                            │
  │                                               │
  │  3. ECAPA-TDNN:                              │
  │     Squeeze-Excite + Res2Net + attention     │
  │     → 192-d vector (state-of-the-art)        │
  │                                               │
  │  Trained on speaker verification:             │
  │  "Is speaker A = speaker B?"                 │
  │  Datasets: VoxCeleb (7000+ speakers)         │
  │                                               │
  └─────────────────────────────────────────────┘
```

---

## Zero-Shot vs. Few-Shot Voice Cloning

```
  Zero-Shot:
    • Never seen this speaker during training
    • Uses only a few seconds of reference audio
    • Lower quality but more flexible
    • Examples: YourTTS, VALL-E, Bark

  Few-Shot:
    • Fine-tunes on a few minutes of target speaker audio
    • Higher quality, better speaker similarity
    • Requires additional training time
    • Examples: Tortoise TTS, fine-tuned VITS

  One-Shot (special case of zero-shot):
    • Only 1 utterance needed (~3-10 seconds)
    • VALL-E (Microsoft): ~3 sec reference
    • Most practical for real applications
```

---

## Key Architectures

### SV2TTS (Transfer Learning from Speaker Verification to TTS)

```
  Three-stage pipeline (Jia et al., 2018):

  Stage 1: Speaker Encoder (pretrained on SV)
    Audio → d-vector (speaker identity)

  Stage 2: Synthesizer (Tacotron 2)
    Text + d-vector → mel spectrogram

  Stage 3: Vocoder (WaveRNN)
    Mel spectrogram → waveform

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Reference audio ──→ Speaker Encoder ──→ d-vec   │
  │                                           │       │
  │  "Hello world" ──→ Tacotron 2 ←──────────┘       │
  │                        │                           │
  │                        ▼                           │
  │                   Mel spectrogram                 │
  │                        │                           │
  │                        ▼                           │
  │                    WaveRNN vocoder                │
  │                        │                           │
  │                        ▼                           │
  │                   🔊 Audio (cloned voice)         │
  └──────────────────────────────────────────────────┘
```

### VALL-E (Microsoft)

```
  VALL-E (Wang et al., 2023):
  Treats TTS as a language modeling problem!

  Uses EnCodec to represent audio as tokens,
  then generates tokens conditioned on:
    1. Text (phoneme sequence)
    2. Audio prompt (3-sec reference → EnCodec tokens)

  Two stages:
    AR model: generates coarse tokens (codebook 1)
    NAR model: generates fine tokens (codebooks 2-8)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Reference: 3 sec audio → EnCodec tokens         │
  │  Text: phonemes of target sentence               │
  │       │                                            │
  │       ▼                                            │
  │  AR Transformer → coarse audio tokens (book 1)   │
  │       │                                            │
  │       ▼                                            │
  │  NAR Transformer → fine tokens (books 2-8)       │
  │       │                                            │
  │       ▼                                            │
  │  EnCodec Decoder → high-quality cloned audio     │
  │                                                    │
  │  Trained on 60K hours of speech (LibriLight)     │
  │  → Emergent in-context learning for voice clone  │
  └──────────────────────────────────────────────────┘
```

### XTTS / Coqui TTS

```
  Open-source voice cloning:
  
  XTTS v2 (Coqui):
    • Multilingual (17 languages)
    • 6-second reference audio needed
    • GPT-based architecture
    • Streaming capability for real-time use
    
  Architecture:
    Text → GPT model → latent codes
    Speaker ref → speaker conditioning
    Latent codes + speaker → HiFi-GAN vocoder → audio
```

---

## VITS and YourTTS

```
  VITS (Kim et al., 2021):
    End-to-end TTS with VAE + normalizing flows + adversarial training
    Single model: text → waveform (no separate vocoder!)

  YourTTS (Casanova et al., 2022):
    Extends VITS for zero-shot voice cloning:
    
    ┌──────────────────────────────────────────────┐
    │                                                │
    │  Text ──→ Text Encoder                        │
    │               │                                │
    │               ▼                                │
    │  Prior Network (normalizing flow)             │
    │               │                                │
    │               ▼                                │
    │  Posterior ←── Mel encoder                     │
    │               │                                │
    │  Speaker emb ─┘  (from speaker encoder)       │
    │               │                                │
    │               ▼                                │
    │  HiFi-GAN Decoder → waveform                 │
    │                                                │
    │  Features:                                    │
    │  • Multilingual (English, Portuguese, French) │
    │  • Zero-shot cloning from single utterance    │
    │  • Real-time inference speed                  │
    └──────────────────────────────────────────────┘
```

---

## Evaluation Metrics

```
  ┌──────────────────┬────────────────────────────────────┐
  │ Metric           │ Description                        │
  ├──────────────────┼────────────────────────────────────┤
  │ Speaker Sim.     │ Cosine similarity between speaker  │
  │ (SIM)            │ embeddings of real vs. cloned      │
  ├──────────────────┼────────────────────────────────────┤
  │ MOS              │ Mean Opinion Score (human rating   │
  │                  │ 1-5 for naturalness)               │
  ├──────────────────┼────────────────────────────────────┤
  │ WER              │ Word Error Rate (intelligibility   │
  │                  │ via ASR system)                    │
  ├──────────────────┼────────────────────────────────────┤
  │ SMOS             │ Speaker MOS (human rating of       │
  │                  │ speaker similarity)                │
  ├──────────────────┼────────────────────────────────────┤
  │ PESQ             │ Perceptual Evaluation of Speech    │
  │                  │ Quality (signal quality metric)    │
  └──────────────────┴────────────────────────────────────┘
```

---

## Python: Voice Cloning with Coqui TTS

```python
from TTS.api import TTS
import torch

# Initialize XTTS v2 model
tts = TTS("tts_models/multilingual/multi-dataset/xtts_v2")
tts.to("cuda" if torch.cuda.is_available() else "cpu")

# Clone voice from a reference audio file
tts.tts_to_file(
    text="Hello! This is my cloned voice speaking.",
    speaker_wav="reference_audio.wav",   # 6+ seconds of target speaker
    language="en",
    file_path="cloned_output.wav"
)

# Using SV2TTS approach (Real-Time-Voice-Cloning)
from encoder import inference as encoder
from synthesizer.inference import Synthesizer
from vocoder import inference as vocoder

# Load models
encoder.load_model("encoder/saved_models/pretrained.pt")
synthesizer = Synthesizer("synthesizer/saved_models/pretrained.pt")
vocoder.load_model("vocoder/saved_models/pretrained.pt")

# Extract speaker embedding from reference
import numpy as np
from encoder.audio import preprocess_wav

ref_wav = preprocess_wav("reference.wav")
speaker_embedding = encoder.embed_utterance(ref_wav)

# Synthesize new speech in cloned voice
text = "This speech was generated using voice cloning technology."
specs = synthesizer.synthesize_spectrograms([text], [speaker_embedding])
generated_wav = vocoder.infer_waveform(specs[0])
```

---

## Ethical Considerations

```
Voice cloning poses serious ethical challenges:

  ┌──────────────────────────────────────────────────┐
  │  RISKS:                                           │
  │                                                    │
  │  1. Fraud & Impersonation                        │
  │     • Phone scams impersonating family members   │
  │     • CEO fraud (fake voice authorization)       │
  │                                                    │
  │  2. Deepfake Audio                               │
  │     • Fake political speeches                    │
  │     • Fabricated evidence in legal cases         │
  │                                                    │
  │  3. Consent Violations                           │
  │     • Cloning someone's voice without permission │
  │     • Generating inappropriate content           │
  │                                                    │
  │  MITIGATIONS:                                    │
  │                                                    │
  │  1. Audio watermarking (embed imperceptible ID)  │
  │  2. Deepfake detection models                    │
  │  3. Consent verification systems                 │
  │  4. Legislation (EU AI Act, US DEEPFAKES Act)    │
  │  5. Speaker verification before cloning          │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What is the role of the speaker encoder in voice cloning?**
2. **How does VALL-E treat TTS as a language modeling problem?**
3. **What is the difference between zero-shot and few-shot voice cloning?**
4. **How does the SV2TTS pipeline work (3 stages)?**
5. **What ethical risks does voice cloning technology pose?**

---

[Previous: 04-music-generation.md](04-music-generation.md) | [Next: ../10-Video-Generation/01-video-prediction.md](../10-Video-Generation/01-video-prediction.md)
