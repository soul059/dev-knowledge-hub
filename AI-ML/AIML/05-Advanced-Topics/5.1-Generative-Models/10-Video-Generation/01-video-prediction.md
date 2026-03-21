# Video Prediction

## Overview

Video prediction is the task of generating future video frames given a sequence of past frames. Unlike image generation, video prediction must model **temporal dynamics** — how objects move, deform, and interact over time. This is a foundational capability for autonomous driving, robotics, weather forecasting, and anomaly detection.

---

## Problem Formulation

```
Given past frames, predict future frames:

  Input:   x_1, x_2, ..., x_T        (observed frames)
  Output:  x̂_{T+1}, x̂_{T+2}, ..., x̂_{T+K}  (predicted frames)

  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐     ┌─────┐ ┌─────┐ ┌─────┐
  │  t=1│ │  t=2│ │  t=3│ │  t=4│ ──→ │  t=5│ │  t=6│ │  t=7│
  │  ●  │ │  ●  │ │   ● │ │    ●│     │    ●│ │    ●│ │    ●│
  │     │ │     │ │     │ │     │     │     │ │    ↓│ │    ↓│
  └─────┘ └─────┘ └─────┘ └─────┘     └─────┘ └─────┘ └─────┘
     observed (context)              predicted (future)
     
  Ball moving right →                Predict it continues!
  
  Key challenges:
  1. Stochastic futures (multiple valid outcomes)
  2. Increasing uncertainty over time
  3. Maintaining visual quality
  4. Physics-aware predictions
```

---

## Deterministic vs. Stochastic Prediction

```
  Deterministic models:
    • Predict single most likely future
    • MSE loss → blurry predictions (average of possibilities)
    • Simple but limited

  Stochastic models:
    • Model distribution of possible futures
    • Sample different plausible outcomes
    • Essential for long-term prediction

  Example: Car at intersection
  ┌────────────────────────────────────────┐
  │                                          │
  │  Past:  🚗 → → → ┼ (intersection)      │
  │                                          │
  │  Possible futures:                      │
  │    a) 🚗 → → → (go straight)          │
  │    b) 🚗 ↗ (turn right)               │
  │    c) 🚗 ↙ (turn left)                │
  │    d) 🚗 ⬛ (stop)                     │
  │                                          │
  │  Deterministic: predicts BLUR of a+b+c+d│
  │  Stochastic:    samples ONE of a,b,c,d  │
  └────────────────────────────────────────┘
```

---

## ConvLSTM-based Approaches

```
  ConvLSTM (Shi et al., 2015):
    Extend LSTM with convolutional operations for spatial data

    Standard LSTM: fully connected gates
    ConvLSTM: convolutional gates (preserve spatial structure)

    Gates use convolution instead of matrix multiply:
      f_t = σ(W_f * [H_{t-1}, X_t] + b_f)
      i_t = σ(W_i * [H_{t-1}, X_t] + b_i)
      o_t = σ(W_o * [H_{t-1}, X_t] + b_o)
      
    where * is convolution, H is (H, W, C) tensor

  PredRNN (Wang et al., 2017):
    Adds spatiotemporal memory flow between layers
    
    ┌──────────────────────────────────────────┐
    │  Layer 4: ─H─ ─H─ ─H─ ─H─             │
    │           ↑    ↑    ↑    ↑               │
    │  Layer 3: ─H─ ─H─ ─H─ ─H─   H: hidden │
    │           ↑    ↑    ↑    ↑    M: memory  │
    │  Layer 2: ─H─ ─H─ ─H─ ─H─             │
    │           ↑    ↑    ↑    ↑               │
    │  Layer 1: ─H─ ─H─ ─H─ ─H─             │
    │           M↗   M↗   M↗   M↗             │
    │  (M flows zigzag across time & layers)  │
    │           t=1  t=2  t=3  t=4            │
    └──────────────────────────────────────────┘
```

---

## SVG: Stochastic Video Generation

```
  SVG (Denton & Fergus, 2018):
    VAE-based stochastic video prediction

    At each time step:
      1. Encode past frame x_t → h_t
      2. Sample latent z_t ~ N(μ_t, σ_t)   (stochasticity!)
      3. Decode (h_t, z_t) → x̂_{t+1}

    Training:
      Prior:     p(z_t | x_{1:t})         ← learned
      Posterior: q(z_t | x_{1:t}, x_{t+1}) ← has future info
      Loss: reconstruction + KL(posterior || prior)

    At test time:
      Sample z_t from prior → different futures!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  x_t ──→ Encoder ──→ h_t ──┐                    │
  │                              │                     │
  │  Prior: h_t → μ,σ → z_t ───┤                    │
  │                              │                     │
  │              Decoder ←──────┘                     │
  │                │                                   │
  │                ▼                                   │
  │             x̂_{t+1}                               │
  │                                                    │
  │  Different z samples → different futures!         │
  └──────────────────────────────────────────────────┘
```

---

## Video Prediction with Transformers

```
  Modern approaches use transformers:

  1. Video GPT / VideoGPT (Yan et al., 2021):
     VQ-VAE encodes video → discrete tokens
     GPT transformer predicts next tokens
     → autoregressively generate future frames

  2. FitVid (Babaeizadeh et al., 2021):
     Large-scale CNN-based model
     Shows architecture choices matter more than novelty

  3. MCVD (Voleti et al., 2022):
     Masked Conditional Video Diffusion
     Diffusion model conditioned on past frames
     → state-of-the-art quality

  Architecture comparison:
  ┌──────────────┬────────────────┬───────────────┐
  │ Approach     │ Pros            │ Cons          │
  ├──────────────┼────────────────┼───────────────┤
  │ ConvLSTM     │ Fast, simple    │ Blurry long   │
  │ VAE-based    │ Stochastic      │ Blurry        │
  │ GAN-based    │ Sharp frames    │ Mode collapse │
  │ Transformer  │ Long-range dep. │ Expensive     │
  │ Diffusion    │ Best quality    │ Slow sampling │
  └──────────────┴────────────────┴───────────────┘
```

---

## Metrics for Video Prediction

```
  ┌─────────────────┬──────────────────────────────────────┐
  │ Metric          │ Description                          │
  ├─────────────────┼──────────────────────────────────────┤
  │ MSE / PSNR      │ Pixel-level accuracy                 │
  │                 │ (penalizes blur less than desired)    │
  ├─────────────────┼──────────────────────────────────────┤
  │ SSIM            │ Structural similarity (better than   │
  │                 │ MSE for perceptual quality)           │
  ├─────────────────┼──────────────────────────────────────┤
  │ LPIPS           │ Learned perceptual similarity        │
  │                 │ (deep feature comparison)            │
  ├─────────────────┼──────────────────────────────────────┤
  │ FVD             │ Fréchet Video Distance (like FID     │
  │                 │ but for videos, using I3D features)   │
  ├─────────────────┼──────────────────────────────────────┤
  │ Best-of-N       │ Generate N samples, report best     │
  │                 │ (for stochastic models)              │
  └─────────────────┴──────────────────────────────────────┘
```

---

## Python: Video Prediction with ConvLSTM

```python
import torch
import torch.nn as nn


class ConvLSTMCell(nn.Module):
    """Single ConvLSTM cell for spatiotemporal prediction."""
    
    def __init__(self, in_channels, hidden_channels, kernel_size=3):
        super().__init__()
        self.hidden_channels = hidden_channels
        padding = kernel_size // 2
        
        # Combined gates: input, forget, output, cell candidate
        self.conv = nn.Conv2d(
            in_channels + hidden_channels,
            4 * hidden_channels,
            kernel_size,
            padding=padding
        )
    
    def forward(self, x, state):
        h, c = state
        combined = torch.cat([x, h], dim=1)
        gates = self.conv(combined)
        
        i, f, o, g = gates.chunk(4, dim=1)
        i = torch.sigmoid(i)  # input gate
        f = torch.sigmoid(f)  # forget gate
        o = torch.sigmoid(o)  # output gate
        g = torch.tanh(g)     # cell candidate
        
        c_next = f * c + i * g
        h_next = o * torch.tanh(c_next)
        return h_next, c_next


class VideoPredictionModel(nn.Module):
    """Simple video prediction with ConvLSTM."""
    
    def __init__(self, in_channels=3, hidden=64):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Conv2d(in_channels, 32, 3, stride=2, padding=1),
            nn.ReLU(),
            nn.Conv2d(32, hidden, 3, stride=2, padding=1),
            nn.ReLU(),
        )
        self.convlstm = ConvLSTMCell(hidden, hidden)
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(hidden, 32, 4, stride=2, padding=1),
            nn.ReLU(),
            nn.ConvTranspose2d(32, in_channels, 4, stride=2, padding=1),
            nn.Sigmoid(),
        )
    
    def forward(self, frames, n_future=5):
        """
        frames: (B, T, C, H, W) - input sequence
        Returns: (B, n_future, C, H, W) - predicted frames
        """
        B, T, C, H, W = frames.shape
        h_size = H // 4
        
        # Initialize hidden state
        h = torch.zeros(B, 64, h_size, h_size, device=frames.device)
        c = torch.zeros_like(h)
        
        # Encode observed frames
        for t in range(T):
            encoded = self.encoder(frames[:, t])
            h, c = self.convlstm(encoded, (h, c))
        
        # Predict future frames
        predictions = []
        last_encoded = encoded
        for _ in range(n_future):
            h, c = self.convlstm(last_encoded, (h, c))
            pred = self.decoder(h)
            predictions.append(pred)
            last_encoded = self.encoder(pred)  # feed prediction back
        
        return torch.stack(predictions, dim=1)


# Training
model = VideoPredictionModel()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

# Dummy training loop
for epoch in range(10):
    # video: (B, T_total, C, H, W) where T_total = context + future
    video = torch.randn(4, 15, 3, 64, 64)
    context = video[:, :10]
    target = video[:, 10:15]
    
    predicted = model(context, n_future=5)
    loss = nn.functional.mse_loss(predicted, target)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    print(f"Epoch {epoch}, Loss: {loss.item():.4f}")
```

---

## Real-World Applications

```
  1. Autonomous Driving:
     Predict what will happen next on the road
     → Plan safe trajectories

  2. Robotics:
     Predict effect of actions on environment
     → Model-based reinforcement learning

  3. Weather Forecasting:
     Predict satellite imagery evolution
     → Nowcasting precipitation

  4. Anomaly Detection:
     Predict normal behavior → flag deviations
     → Surveillance, manufacturing quality control

  5. Video Compression:
     Predict frames → only send residuals
     → More efficient video codecs
```

---

## Revision Questions

1. **Why does deterministic prediction with MSE loss produce blurry outputs?**
2. **How does ConvLSTM differ from standard LSTM?**
3. **How does SVG handle the stochasticity of future predictions?**
4. **What is FVD and why is it preferred over FID for video evaluation?**
5. **Name three real-world applications of video prediction.**

---

[Previous: ../09-Audio-Generation/05-voice-cloning.md](../09-Audio-Generation/05-voice-cloning.md) | [Next: 02-video-synthesis.md](02-video-synthesis.md)
