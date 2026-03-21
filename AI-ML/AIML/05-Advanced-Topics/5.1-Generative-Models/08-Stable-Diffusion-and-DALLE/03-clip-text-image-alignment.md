# CLIP: Text-Image Alignment

## Overview

CLIP (Contrastive Language-Image Pretraining, Radford et al., 2021) is the bridge between text and images in modern generative models. Trained on 400 million image-text pairs from the internet, CLIP learns a shared embedding space where text and images with similar meanings are close together. CLIP provides the text encoder for Stable Diffusion, the image-text alignment for DALL-E 2, and serves as evaluation metric (CLIP score) for text-to-image generation.

---

## Contrastive Learning

```
Training: match images with their captions

  Batch of N image-text pairs:
  (image₁, text₁), (image₂, text₂), ..., (imageₙ, textₙ)

  Encode all:
    I_i = ImageEncoder(imageᵢ)    (e.g., ViT or ResNet)
    T_j = TextEncoder(textⱼ)      (e.g., Transformer)

  Compute similarity matrix:
  ┌──────────────────────────────────────┐
  │  S[i,j] = I_i · T_j / (τ)          │
  │                                      │
  │       T₁    T₂    T₃    T₄         │
  │  I₁ [HIGH  low   low   low ]       │
  │  I₂ [low   HIGH  low   low ]       │
  │  I₃ [low   low   HIGH  low ]       │
  │  I₄ [low   low   low   HIGH]       │
  │       ↑                              │
  │  Diagonal should be HIGH (matches)  │
  │  Off-diagonal should be LOW         │
  │                                      │
  └──────────────────────────────────────┘

  Loss (symmetric cross-entropy):
    L = ½ (CE_image→text + CE_text→image)
    
    CE_image→text: for each image, correct text has highest sim
    CE_text→image: for each text, correct image has highest sim

  τ = learnable temperature parameter
```

---

## Architecture

```
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Image Encoder:                                   │
  │  • ResNet-50/101 or ViT-B/16, ViT-L/14          │
  │  • Image → patch embeddings → transformer → [CLS]│
  │  • Project to shared dim (512 or 768)            │
  │                                                    │
  │  Text Encoder:                                    │
  │  • Transformer (12 layers, 512 width)            │
  │  • Byte Pair Encoding tokenizer (49152 vocab)    │
  │  • Max 77 tokens                                 │
  │  • [EOS] token embedding → project to shared dim │
  │                                                    │
  │  ┌─────────┐        ┌─────────┐                  │
  │  │  Image   │        │  Text   │                  │
  │  │ Encoder  │        │ Encoder │                  │
  │  └────┬─────┘        └────┬────┘                  │
  │       │                    │                       │
  │       ▼                    ▼                       │
  │  image_emb (512)     text_emb (512)               │
  │       │                    │                       │
  │       └──── cosine sim ────┘                      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## CLIP in Diffusion Models

```
Role in Stable Diffusion:
  CLIP Text Encoder only (not image encoder)
  
  text → CLIP tokenizer → CLIP text transformer → embeddings
  embeddings → cross-attention in U-Net

  NOT just [CLS] token! ALL 77 token embeddings used:
  Shape: (77, 768)  — each token attends to spatial features

  This gives SPATIAL control:
    "red car" → 'red' attends to color features
                'car' attends to shape features

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  How CLIP text encoder is used:                  │
  │                                                    │
  │  "a red car"                                     │
  │  → tokenize: [BOS, a, red, car, EOS, PAD, ...]  │
  │  → CLIP transformer: 77 × 768 tensor            │
  │  → feed to U-Net cross-attention as K and V      │
  │                                                    │
  │  Each denoising step:                            │
  │    Q = spatial features (from U-Net)             │
  │    K = text embeddings (from CLIP)               │
  │    V = text embeddings (from CLIP)               │
  │    attn = softmax(QKᵀ/√d) V                     │
  │                                                    │
  └──────────────────────────────────────────────────┘

Role in DALL-E 2:
  Both CLIP image and text encoders used
  Prior maps text embedding → image embedding space
  Decoder generates image from image embedding
```

---

## CLIP Score (Evaluation Metric)

```
Measure text-image alignment quality:

  CLIP_score(image, text) = cos_sim(CLIP_image(image), CLIP_text(text))

  Higher = better alignment between generated image and prompt

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Evaluation metrics for text-to-image:           │
  │                                                    │
  │  FID:         Image quality (lower = better)     │
  │  CLIP Score:  Text alignment (higher = better)   │
  │  IS:          Image quality + diversity           │
  │                                                    │
  │  Trade-off: guidance scale ↑                     │
  │    → CLIP score ↑ (better alignment)             │
  │    → FID may ↑ (less diversity)                  │
  │    → "Pareto frontier" of quality vs alignment   │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Textual Inversion and Fine-tuning

```
Adapting CLIP for custom concepts:

  Textual Inversion:
    Learn a new token embedding for your concept
    "A photo of [V]" where [V] = your custom object
    Only learns the embedding vector (3-5KB)!

  DreamBooth:
    Fine-tune the entire diffusion model on 3-5 images
    "A [V] dog" → generates your specific dog
    Larger but more powerful than textual inversion

  LoRA (Low-Rank Adaptation):
    Fine-tune low-rank matrices in attention layers
    4-100MB instead of full model (2-6GB)
    Good balance of quality and size
```

---

## Python: CLIP Usage

```python
import torch
from transformers import CLIPModel, CLIPProcessor

# Load CLIP
model = CLIPModel.from_pretrained("openai/clip-vit-large-patch14")
processor = CLIPProcessor.from_pretrained("openai/clip-vit-large-patch14")

# Compute text-image similarity
def clip_score(images, texts):
    inputs = processor(text=texts, images=images, 
                      return_tensors="pt", padding=True)
    outputs = model(**inputs)
    
    image_emb = outputs.image_embeds  # (N, 768)
    text_emb = outputs.text_embeds    # (M, 768)
    
    # Normalize
    image_emb = image_emb / image_emb.norm(dim=-1, keepdim=True)
    text_emb = text_emb / text_emb.norm(dim=-1, keepdim=True)
    
    # Cosine similarity
    similarity = (image_emb @ text_emb.T) * 100  # scaled
    return similarity

# Zero-shot classification with CLIP
def classify(image, candidate_labels):
    inputs = processor(text=candidate_labels, images=image,
                      return_tensors="pt", padding=True)
    outputs = model(**inputs)
    
    logits = outputs.logits_per_image  # (1, num_labels)
    probs = logits.softmax(dim=1)
    
    for label, prob in zip(candidate_labels, probs[0]):
        print(f"  {label}: {prob:.1%}")

# Extract text embeddings for diffusion conditioning
def get_text_embeddings(text, tokenizer, text_encoder):
    tokens = tokenizer(text, padding="max_length", max_length=77,
                      truncation=True, return_tensors="pt")
    with torch.no_grad():
        embeddings = text_encoder(tokens.input_ids)[0]  # (1, 77, 768)
    return embeddings
```

---

## Revision Questions

1. **How does CLIP's contrastive training create a shared text-image space?**
2. **Why does Stable Diffusion use all 77 token embeddings, not just [CLS]?**
3. **How is CLIP score used to evaluate text-to-image generation?**
4. **What is the difference between Textual Inversion and DreamBooth?**
5. **Why is CLIP's text encoder used more than its image encoder in generation?**

---

[Previous: 02-text-to-image-generation.md](02-text-to-image-generation.md) | [Next: 04-classifier-free-guidance.md](04-classifier-free-guidance.md)
