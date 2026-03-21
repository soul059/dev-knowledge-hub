[← Efficient Transformers](../10-Modern-Transformer-Variants/06-efficient-transformers.md) | [Fine-tuning LLMs →](02-fine-tuning-llms.md)

---

# The Large Language Model Landscape

## Overview

Large Language Models (LLMs) have fundamentally transformed artificial intelligence, enabling machines to generate coherent text, reason about complex problems, write code, and engage in nuanced dialogue. Built on the Transformer architecture introduced in 2017, these models scale self-attention mechanisms to billions (or even trillions) of parameters, training on massive internet-scale corpora. The LLM landscape has evolved rapidly since GPT-3 demonstrated the power of scale in 2020, splitting into two major camps — proprietary models from companies like OpenAI, Google, and Anthropic, and open-weight models from Meta, Mistral AI, and the Technology Innovation Institute. Understanding this landscape is essential for practitioners who need to choose the right model for a given task, budget, and deployment constraint.

---

## Timeline of LLM Development

```
2017 ─── Transformer (Vaswani et al.)
  │
2018 ─── GPT-1 (117M) ─── BERT (340M)
  │
2019 ─── GPT-2 (1.5B) ─── T5 (11B)
  │
2020 ─── GPT-3 (175B)
  │
2021 ─── Codex ─── PaLM planning begins
  │
2022 ─── ChatGPT ─── PaLM (540B) ─── LLaMA-1 (65B) ─── Chinchilla (70B)
  │
2023 ─── GPT-4 ─── LLaMA-2 (70B) ─── Claude 2 ─── Mistral 7B ─── Falcon (180B)
  │
2024 ─── Gemini 1.5 ─── LLaMA-3 (405B) ─── Claude 3.5 ─── Mistral Large
  │
2025 ─── GPT-4.5 ─── Claude 4 ─── LLaMA-4 ─── Gemini 2.5 ─── DeepSeek-R1
```

---

## The LLM Family Tree

```
                        ┌──────────────────────────────────────┐
                        │          Transformer (2017)          │
                        └──────────────┬───────────────────────┘
                                       │
                 ┌─────────────────────┼─────────────────────┐
                 │                     │                     │
          ┌──────┴──────┐      ┌──────┴──────┐      ┌──────┴──────┐
          │  Decoder-   │      │  Encoder-   │      │  Encoder    │
          │  Only (GPT) │      │  Decoder    │      │  Only       │
          └──────┬──────┘      │  (T5, BART) │      │  (BERT)     │
                 │             └─────────────┘      └─────────────┘
     ┌───────────┼───────────────────┐
     │           │                   │
  ┌──┴───┐  ┌───┴────┐    ┌────────┴────────┐
  │ GPT  │  │ LLaMA  │    │  Other Decoder  │
  │Series│  │ Series │    │    Models        │
  └──┬───┘  └───┬────┘    └────────┬────────┘
     │          │                  │
  GPT-3     LLaMA-1          ┌────┼─────────┐
  GPT-3.5   LLaMA-2          │    │         │
  GPT-4     LLaMA-3       Mistral Falcon  PaLM
  GPT-4o    LLaMA-4       Mixtral 180B   Gemini
  GPT-4.5                                Claude
```

---

## Major LLM Families

### GPT Series (OpenAI) — Closed Source

OpenAI's Generative Pre-trained Transformer series pioneered the scaling paradigm. GPT-3 (175B parameters) demonstrated few-shot learning, where a model can perform tasks given only a few examples in the prompt. GPT-4 introduced multimodal capabilities (text + vision) and substantially improved reasoning. These models use **decoder-only** Transformer architecture with causal (left-to-right) attention masking.

**Key innovations:**
- Scaling laws: performance improves predictably with more parameters and data
- Instruction tuning via RLHF (Reinforcement Learning from Human Feedback)
- Multimodal inputs (GPT-4V, GPT-4o)
- Long context windows (up to 128K tokens)

### LLaMA Series (Meta) — Open Weights

Meta's LLaMA (Large Language Model Meta AI) series democratised access to high-quality LLMs by releasing model weights. LLaMA-1 showed that smaller, well-trained models can match larger ones (a 13B model rivaling GPT-3 175B), validating the Chinchilla scaling laws.

**Key innovations:**
- RMSNorm (pre-normalization) instead of LayerNorm
- Rotary Positional Embeddings (RoPE) for better length generalization
- SwiGLU activation function in feed-forward layers
- Grouped Query Attention (GQA) in LLaMA-2 and beyond

### Claude Series (Anthropic) — Closed Source

Anthropic's Claude models emphasise safety, helpfulness, and honesty through Constitutional AI (CAI). Claude uses RLHF combined with AI-generated feedback based on a set of principles (a "constitution").

**Key innovations:**
- Constitutional AI: self-critique and revision guided by principles
- Very long context windows (up to 200K tokens in Claude 3)
- Strong instruction following and nuanced reasoning

### PaLM / Gemini Series (Google) — Closed Source

Google's PaLM (Pathways Language Model) scaled to 540B parameters using the Pathways system for distributed training. Gemini represents Google's next-generation multimodal model family natively trained on text, image, audio, and video.

**Key innovations:**
- Pathways: efficient multi-TPU pod training
- Mixture of Experts (MoE) in Gemini 1.5 (reduces active parameters)
- Native multimodality: trained jointly on multiple modalities
- 1M+ token context window (Gemini 1.5 Pro)

### Mistral / Mixtral (Mistral AI) — Open Weights

Mistral AI, a French startup, released highly efficient open models. Mistral 7B outperformed LLaMA-2 13B on many benchmarks. Mixtral 8x7B introduced sparse Mixture of Experts at open-weight scale.

**Key innovations:**
- Sliding Window Attention for efficient long-context processing
- Sparse Mixture of Experts (MoE): 8 experts, 2 active per token
- High performance-to-size ratio
- Byte-fallback BPE tokenizer

### Falcon (TII) — Open Source

The Technology Innovation Institute released Falcon models (7B, 40B, 180B) trained on the RefinedWeb dataset, demonstrating the importance of data quality.

**Key innovations:**
- RefinedWeb: massive, carefully filtered web-only dataset
- Multi-query attention for efficient inference
- True open-source license (Apache 2.0)

---

## Scaling Laws and Compute Requirements

The relationship between model performance and compute follows predictable **scaling laws** (Kaplan et al., 2020; Hoffmann et al., 2022):

```
Loss ≈ A / N^α  +  B / D^β  +  C

Where:
  N = number of parameters
  D = dataset size (tokens)
  α ≈ 0.076  (parameter scaling exponent)
  β ≈ 0.095  (data scaling exponent)
  A, B, C = fitted constants
```

### Chinchilla Optimal Training

Hoffmann et al. (2022) found that **compute-optimal** models should scale parameters and data equally:

```
Optimal tokens ≈ 20 × N

Example:
  7B model   → ~140B tokens
  70B model  → ~1.4T tokens
  400B model → ~8T tokens
```

### Compute Estimates

Training compute is measured in **FLOPs** (floating-point operations):

```
C ≈ 6 × N × D

Where:
  C = total compute (FLOPs)
  N = parameters
  D = training tokens

Example (LLaMA-2 70B):
  C ≈ 6 × 70×10⁹ × 2×10¹² = 8.4 × 10²³ FLOPs
  ≈ 1,720,320 GPU-hours on A100 (at 312 TFLOPS)
```

---

## Open Source vs Closed Source Models

```
┌────────────────────────────────────────────────────────────────┐
│                    Open vs Closed Spectrum                      │
├────────────────┬─────────────────┬─────────────────────────────┤
│  Fully Open    │  Open Weights   │  Closed / API Only          │
│  (code+data+   │  (weights only, │  (no weights, access via    │
│   weights)     │   no data)      │   API only)                 │
├────────────────┼─────────────────┼─────────────────────────────┤
│  Falcon        │  LLaMA-2/3      │  GPT-4 / GPT-4o            │
│  BLOOM         │  Mistral/Mixtral│  Claude 3 / 3.5             │
│  OLMo          │  Gemma          │  Gemini 1.5 / 2.5           │
│  Pythia        │  Qwen           │  o1 / o3                    │
│  StarCoder     │  DeepSeek       │                             │
└────────────────┴─────────────────┴─────────────────────────────┘
```

**When to choose open models:**
- Data privacy: run on your own infrastructure
- Customisation: fine-tune for specific domains
- Cost control: avoid per-token API pricing at scale
- Research reproducibility

**When to choose closed models:**
- Best-in-class performance on complex reasoning
- No infrastructure to manage
- Rapid prototyping without GPU resources
- Built-in safety filters and moderation

---

## Model Comparison Table

| Model | Params | Context Length | Open/Closed | Key Features |
|---|---|---|---|---|
| GPT-4o | ~200B (est.) | 128K | Closed | Multimodal, fast, strong reasoning |
| GPT-4 | ~1.8T MoE (est.) | 128K | Closed | Best-in-class reasoning (2023) |
| Claude 3.5 Sonnet | Unknown | 200K | Closed | Constitutional AI, long context |
| Gemini 1.5 Pro | Unknown (MoE) | 1M+ | Closed | Native multimodal, huge context |
| LLaMA-3.1 405B | 405B | 128K | Open weights | Largest open model, strong perf |
| LLaMA-3.1 70B | 70B | 128K | Open weights | Best open 70B class |
| Mistral 7B | 7.3B | 32K | Open weights | Best-in-class at 7B size |
| Mixtral 8x7B | 46.7B (12.9B active) | 32K | Open weights | Sparse MoE, efficient inference |
| Falcon 180B | 180B | 2K | Open source | RefinedWeb trained |
| DeepSeek-R1 | 671B MoE | 128K | Open weights | Reasoning-focused, MoE |
| Qwen-2.5 72B | 72B | 128K | Open weights | Strong multilingual support |
| Gemma 2 27B | 27B | 8K | Open weights | Google's open model |

---

## Architectural Differences Across Models

### Positional Encoding Variants

```
Absolute PE:   PE(pos, 2i)   = sin(pos / 10000^(2i/d))        # Original Transformer
               PE(pos, 2i+1) = cos(pos / 10000^(2i/d))

RoPE:          q̃ₘ = R(θ, m) · qₘ                              # LLaMA, Mistral, Qwen
               where R(θ, m) is a rotation matrix
               R(θ, m) = diag(e^(imθ₁), e^(imθ₂), ..., e^(imθ_{d/2}))

ALiBi:         softmax(qKᵀ + m · [0, -1, -2, ...])            # Falcon, BLOOM
               where m is a head-specific slope
```

### Attention Variants

```
Multi-Head Attention (MHA):     n_kv_heads = n_heads           # Original, GPT-3
Grouped Query Attention (GQA):  n_kv_heads = n_heads / G       # LLaMA-2 70B
Multi-Query Attention (MQA):    n_kv_heads = 1                 # Falcon, PaLM

GQA KV-cache memory savings:
  MHA  KV-cache: 2 × n_layers × n_heads × d_head × seq_len × batch
  GQA  KV-cache: 2 × n_layers × (n_heads/G) × d_head × seq_len × batch
  MQA  KV-cache: 2 × n_layers × 1 × d_head × seq_len × batch
```

---

## Worked Example: Estimating Model Memory

**Problem:** Estimate the GPU memory needed to load LLaMA-2 70B in different precisions.

```
Model memory ≈ Parameters × bytes_per_param

FP32 (32-bit):   70B × 4 bytes  = 280 GB
FP16 (16-bit):   70B × 2 bytes  = 140 GB
INT8 (8-bit):    70B × 1 byte   =  70 GB
INT4 (4-bit):    70B × 0.5 byte =  35 GB

Available GPU memory (A100):    80 GB per GPU

Therefore:
  FP32 → 4 × A100 GPUs (with tensor parallelism)
  FP16 → 2 × A100 GPUs
  INT8 → 1 × A100 GPU
  INT4 → 1 × A100 GPU (with room for KV cache + activations)
```

**Plus KV-cache for inference (sequence length 4096, batch size 1):**

```
KV-cache ≈ 2 × n_layers × n_kv_heads × d_head × seq_len × 2 bytes (FP16)
         = 2 × 80 × 8 × 128 × 4096 × 2
         = 2 × 80 × 8 × 128 × 4096 × 2
         ≈ 1.07 GB
```

---

## Python Code: Loading and Comparing Open-Source Models

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
import time

def load_and_profile(model_name, quantization=None):
    """Load a model and report memory usage and inference speed."""
    print(f"\n{'='*60}")
    print(f"Loading: {model_name}")
    print(f"{'='*60}")

    tokenizer = AutoTokenizer.from_pretrained(model_name)

    load_kwargs = {"device_map": "auto", "torch_dtype": torch.float16}

    if quantization == "4bit":
        from transformers import BitsAndBytesConfig
        load_kwargs["quantization_config"] = BitsAndBytesConfig(
            load_in_4bit=True,
            bnb_4bit_compute_dtype=torch.float16,
            bnb_4bit_quant_type="nf4",
            bnb_4bit_use_double_quant=True,
        )
    elif quantization == "8bit":
        from transformers import BitsAndBytesConfig
        load_kwargs["quantization_config"] = BitsAndBytesConfig(
            load_in_8bit=True,
        )

    model = AutoModelForCausalLM.from_pretrained(model_name, **load_kwargs)

    # Count parameters
    total_params = sum(p.numel() for p in model.parameters())
    trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
    print(f"Total parameters:     {total_params:,}")
    print(f"Trainable parameters: {trainable_params:,}")

    # Estimate memory
    mem_gb = torch.cuda.max_memory_allocated() / 1e9 if torch.cuda.is_available() else 0
    print(f"GPU memory allocated:  {mem_gb:.2f} GB")

    # Benchmark inference
    prompt = "Explain the theory of relativity in simple terms:"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    start = time.time()
    with torch.no_grad():
        outputs = model.generate(
            **inputs,
            max_new_tokens=100,
            do_sample=False,
        )
    elapsed = time.time() - start

    generated = tokenizer.decode(outputs[0], skip_special_tokens=True)
    tokens_generated = outputs.shape[1] - inputs["input_ids"].shape[1]
    tokens_per_sec = tokens_generated / elapsed

    print(f"Tokens generated:     {tokens_generated}")
    print(f"Time:                 {elapsed:.2f}s")
    print(f"Speed:                {tokens_per_sec:.1f} tokens/sec")
    print(f"Output: {generated[:200]}...")

    return {
        "model": model_name,
        "params": total_params,
        "memory_gb": mem_gb,
        "tokens_per_sec": tokens_per_sec,
    }


# Compare several open-source models
models_to_compare = [
    ("mistralai/Mistral-7B-Instruct-v0.2", "4bit"),
    ("meta-llama/Meta-Llama-3-8B-Instruct", "4bit"),
    ("google/gemma-2-9b-it", "4bit"),
    ("Qwen/Qwen2.5-7B-Instruct", "4bit"),
]

results = []
for model_name, quant in models_to_compare:
    try:
        result = load_and_profile(model_name, quantization=quant)
        results.append(result)
    except Exception as e:
        print(f"Failed to load {model_name}: {e}")

# Print comparison table
print("\n" + "="*80)
print(f"{'Model':<45} {'Params':>12} {'Memory(GB)':>12} {'Tok/s':>8}")
print("="*80)
for r in results:
    print(f"{r['model']:<45} {r['params']:>12,} {r['memory_gb']:>12.2f} {r['tokens_per_sec']:>8.1f}")
```

---

## Real-World Applications

| Application | Recommended Model Type | Example |
|---|---|---|
| Customer support chatbot | Closed API (GPT-4o, Claude) | Automated ticket resolution |
| Code generation | Closed or open (GPT-4, CodeLlama) | GitHub Copilot, Cursor |
| Document summarization | Open 7-13B fine-tuned | Summarising legal contracts |
| Healthcare NLP | Open fine-tuned (privacy) | Clinical note extraction |
| Multilingual translation | Large open (Qwen, LLaMA) | Real-time translation apps |
| Research & experimentation | Open weights (LLaMA, Mistral) | Academic NLP research |
| Enterprise search | RAG with open embeddings | Internal knowledge base |
| Creative writing | Closed API (Claude, GPT-4) | Story generation, copywriting |

---

## Summary Table

| Concept | Key Point |
|---|---|
| Decoder-only architecture | Dominant design for modern LLMs; causal attention masking |
| Scaling laws | Loss decreases predictably with more params and data |
| Chinchilla optimal | Train on ~20× tokens relative to parameter count |
| RoPE | Rotary embeddings for better length generalization |
| GQA / MQA | Reduce KV-cache memory; fewer key-value heads |
| Open vs closed | Trade-off between customisation/privacy and performance |
| Quantization | INT4/INT8 reduces memory 4-8× with modest quality loss |
| MoE | Sparse experts increase capacity without proportional compute |
| Context length | Ranges from 2K (older) to 1M+ (Gemini 1.5) |
| Compute cost | Training 70B model ≈ millions of GPU-hours |

---

## Revision Questions

1. **What are the key architectural differences between GPT-4 and LLaMA-3?** Discuss attention mechanisms, positional encodings, and normalization strategies.

2. **Explain the Chinchilla scaling law.** Why did it change how the community thinks about training compute allocation?

3. **Compare Grouped Query Attention (GQA) and Multi-Query Attention (MQA).** How do they reduce KV-cache memory, and what is the trade-off?

4. **Calculate the minimum GPU memory needed** to load a 13B parameter model in FP16, INT8, and INT4 formats. Which would fit on a single 24GB consumer GPU?

5. **When should a practitioner choose an open-weight model over a closed API?** Give three concrete scenarios with justification.

6. **What is Mixture of Experts (MoE) and why does it matter?** How does Mixtral 8x7B achieve high performance with only 12.9B active parameters per token?

---

[← Efficient Transformers](../10-Modern-Transformer-Variants/06-efficient-transformers.md) | [Fine-tuning LLMs →](02-fine-tuning-llms.md)
