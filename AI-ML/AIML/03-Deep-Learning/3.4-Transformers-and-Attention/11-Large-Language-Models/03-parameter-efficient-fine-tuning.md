[← Fine-tuning LLMs](02-fine-tuning-llms.md) | [RLHF Basics →](04-rlhf-basics.md)

---

# Parameter-Efficient Fine-Tuning (PEFT)

## Overview

Parameter-Efficient Fine-Tuning (PEFT) methods adapt large pre-trained models by training only a small subset of parameters — typically less than 1% of the total — while keeping the rest of the model frozen. This dramatically reduces GPU memory, storage, and training time, making it feasible to fine-tune billion-parameter models on a single consumer GPU. The most popular PEFT method is **LoRA** (Low-Rank Adaptation), which injects trainable low-rank matrices into the attention layers. Other approaches include **QLoRA** (quantised LoRA), **adapters**, **prefix tuning**, and **prompt tuning**. PEFT methods achieve performance close to full fine-tuning at a fraction of the cost, and they allow serving multiple fine-tuned variants from a single base model by swapping lightweight adapter weights.

---

## Why PEFT?

```
Full Fine-Tuning vs PEFT — Resource Comparison (7B model):

                        Full FT          LoRA           QLoRA
  ─────────────────────────────────────────────────────────────
  Trainable params:     7B (100%)      ~20M (0.3%)    ~20M (0.3%)
  GPU memory:           ~84 GB          ~32 GB         ~12 GB
  Storage per task:     14 GB           ~80 MB         ~80 MB
  Multi-task serving:   N copies        1 base + N     1 base + N
                        of full model   adapters       adapters
  Training speed:       1× (baseline)   ~2-3× faster   ~2-3× faster
  Performance:          Best            ~98% of full   ~95-98% of full
```

---

## LoRA: Low-Rank Adaptation

### Core Idea

Instead of updating a full weight matrix W ∈ ℝ^(d×k), LoRA freezes W and learns a low-rank update:

```
W' = W₀ + ΔW = W₀ + B·A

Where:
  W₀ ∈ ℝ^(d×k)   — frozen pre-trained weights
  B  ∈ ℝ^(d×r)    — trainable down-projection
  A  ∈ ℝ^(r×k)    — trainable up-projection
  r  << min(d, k)  — rank (typically 4, 8, 16, 32, 64)

The output with LoRA:
  h = W₀·x + (B·A)·x = W₀·x + B·(A·x)

Scaling factor:
  h = W₀·x + (α/r) · B·(A·x)

Where α (lora_alpha) is a scaling hyperparameter.
```

### Why Low-Rank Works

Pre-trained weight updates during fine-tuning have been shown to have **low intrinsic dimensionality** (Aghajanyan et al., 2020). The change ΔW needed for task adaptation lives in a low-rank subspace:

```
rank(ΔW) << min(d, k)

For a 4096 × 4096 attention weight matrix:
  Full:    4096 × 4096 = 16,777,216 parameters
  LoRA r=8: (4096 × 8) + (8 × 4096) = 65,536 parameters
  Reduction: 256× fewer parameters!
```

### LoRA Architecture Diagram

```
                    ┌─────────────────────┐
                    │   Input x ∈ ℝ^k     │
                    └──────┬──────────────┘
                           │
              ┌────────────┼────────────────┐
              │            │                │
              ▼            │                ▼
    ┌──────────────┐       │      ┌──────────────────┐
    │   W₀ (frozen)│       │      │  A ∈ ℝ^(r×k)     │
    │   d × k      │       │      │  (trainable)     │
    │              │       │      └────────┬─────────┘
    └──────┬───────┘       │               │
           │               │               ▼
           │               │      ┌──────────────────┐
           │               │      │  B ∈ ℝ^(d×r)     │
           │               │      │  (trainable)     │
           │               │      └────────┬─────────┘
           │               │               │
           │               │        × (α/r) scaling
           │               │               │
           ▼               │               ▼
    ┌──────────────────────┴───────────────────┐
    │              h = W₀x + (α/r)·B·A·x       │
    │                  (element-wise add)       │
    └──────────────────────────────────────────┘
                           │
                           ▼
                    ┌─────────────────────┐
                    │  Output h ∈ ℝ^d     │
                    └─────────────────────┘
```

### Initialization

```
A is initialized from:  N(0, σ²)  (Gaussian)
B is initialized as:    zeros

Therefore at the start:  ΔW = B·A = 0
The model starts identical to the pre-trained model.
```

### Choosing Rank r

```
Rank r     Trainable params    Quality vs full FT
────────────────────────────────────────────────
r = 4      minimal             ~95% (simple tasks)
r = 8      low                 ~97% (most tasks)
r = 16     moderate            ~98% (complex tasks)
r = 32     moderate-high       ~99% (demanding tasks)
r = 64     high                ≈100% (diminishing returns)
r = 128    very high           ≈100% (usually unnecessary)

Rule of thumb: start with r=16, increase if quality is insufficient.
```

---

## QLoRA: Quantized LoRA

QLoRA (Dettmers et al., 2023) combines **4-bit quantization** of the base model with LoRA adapters, enabling fine-tuning of 65B models on a single 48GB GPU.

### Key Innovations

```
1. 4-bit NormalFloat (NF4) quantization:
   - Quantizes weights to 4-bit using a data type optimized
     for normally distributed weights
   - Information-theoretically optimal for N(0, σ²) weights

2. Double quantization:
   - Quantizes the quantization constants themselves
   - Saves ~0.37 bits per parameter (3 GB for 65B model)

3. Paged optimizers:
   - Uses CPU RAM for optimizer state overflow
   - Prevents OOM errors during gradient spikes

Memory breakdown (65B model):
  Base model (NF4):     ~33 GB
  LoRA adapters (FP16): ~0.2 GB
  Optimizer states:     ~2 GB
  Activations:          ~8-12 GB
  ──────────────────────────────
  Total:                ~45 GB  (fits on 1× A100 or 1× A6000)
```

### Quantization Math

```
NF4 Quantization:

1. Estimate quantiles of N(0, 1) for 2⁴ = 16 levels
2. Map each weight to nearest quantile:

   w_quantized = Q(w / absmax(W_block))

   where Q maps to the nearest NF4 value and W_block is a
   block of 64 consecutive weights.

3. Dequantization for computation:
   w_dequant = NF4_lookup[w_quantized] × absmax(W_block)

Double quantization:
   absmax values (FP32) → quantized to FP8
   Saves: (32 - 8) / 64 = 0.375 bits per parameter
```

---

## Adapters

Adapter layers (Houlsby et al., 2019) insert **small bottleneck modules** between existing Transformer layers.

### Architecture

```
                ┌─────────────────┐
                │  Layer Output   │
                └────────┬────────┘
                         │
                ┌────────┴────────┐
                │  Layer Norm     │
                └────────┬────────┘
                         │
                ┌────────┴────────┐
                │  Down-project   │   W_down ∈ ℝ^(d×m)
                │  d → m          │   m << d (bottleneck)
                └────────┬────────┘
                         │
                ┌────────┴────────┐
                │    Non-linear   │   GELU or ReLU
                │    activation   │
                └────────┬────────┘
                         │
                ┌────────┴────────┐
                │   Up-project    │   W_up ∈ ℝ^(m×d)
                │   m → d         │
                └────────┬────────┘
                         │
              ┌──────────┴──────────┐
              │  Residual connection │
              │  output = input + f  │
              └──────────┬──────────┘
                         │
                         ▼

Trainable parameters per adapter:
  2 × d × m + m + d  (weights + biases)

Example (d=4096, m=64):
  2 × 4096 × 64 + 64 + 4096 = 528,448 per adapter
  With 2 adapters per layer × 32 layers = ~34M trainable params
```

---

## Prefix Tuning

Prefix tuning (Li & Liang, 2021) prepends **learnable continuous vectors** to the keys and values in every attention layer.

```
Standard attention:
  Attn(Q, K, V) = softmax(Q·Kᵀ / √d) · V

Prefix-tuned attention:
  K' = [P_K ; K]     ← prepend prefix keys
  V' = [P_V ; V]     ← prepend prefix values

  Attn(Q, K', V') = softmax(Q·K'ᵀ / √d) · V'

Where:
  P_K ∈ ℝ^(l×d)   — learnable key prefix (l prefix tokens)
  P_V ∈ ℝ^(l×d)   — learnable value prefix

Trainable parameters:
  2 × n_layers × l × d_model

Example (32 layers, l=20 prefix tokens, d=4096):
  2 × 32 × 20 × 4096 = 5,242,880 ≈ 5.2M parameters
```

### Reparameterization

To stabilize training, prefix vectors are generated through a small MLP:

```
P_K = MLP(P_embedding)

Where P_embedding ∈ ℝ^(l×d') is the actual learnable parameter
and MLP: ℝ^d' → ℝ^d maps to the model dimension.
```

---

## Prompt Tuning

Prompt tuning (Lester et al., 2021) is the simplest PEFT method: it prepends **learnable soft tokens** to the input embeddings.

```
Standard input:         [x₁, x₂, ..., xₙ]
With prompt tuning:     [p₁, p₂, ..., pₘ, x₁, x₂, ..., xₙ]

Where:
  pᵢ ∈ ℝ^d    — learnable soft prompt embeddings
  m            — number of soft prompt tokens (typically 20-100)

Trainable parameters:  m × d_model

Example (m=100, d=4096):
  100 × 4096 = 409,600 ≈ 0.4M parameters

Note: Soft prompts are continuous vectors — they don't correspond
to any real tokens in the vocabulary. They're optimised via
backpropagation to steer model behavior.
```

### Performance vs. Scale

```
Prompt tuning performance relative to full fine-tuning:

  Model size      Prompt tuning quality
  ──────────────────────────────────────
  < 1B            Significantly worse
  1B - 10B        Slightly worse
  > 10B           Comparable to full FT

Key insight: Prompt tuning becomes competitive with full
fine-tuning as model size increases.
```

---

## Comparison of PEFT Methods

| Method | Trainable Params (%) | Memory Savings | Inference Overhead | Performance | Best For |
|---|---|---|---|---|---|
| LoRA | 0.1-1% | ~60-70% | None (merge weights) | ★★★★★ | General fine-tuning |
| QLoRA | 0.1-1% | ~85-90% | Slight (dequant) | ★★★★☆ | Limited GPU memory |
| Adapters | 1-5% | ~50-60% | Slight (extra layers) | ★★★★☆ | Multi-task learning |
| Prefix Tuning | 0.01-0.1% | ~70-80% | Slight (extra KV) | ★★★☆☆ | NLG tasks |
| Prompt Tuning | <0.01% | ~80-90% | None | ★★★☆☆ | Very large models |
| Full FT | 100% | None | None | ★★★★★ | Maximum quality |

---

## Worked Example: LoRA Parameter Count

**Problem:** Calculate the number of trainable parameters when applying LoRA with r=16 to all attention projections (Q, K, V, O) in a model with 32 layers, d_model=4096, n_heads=32, d_head=128.

```
Each attention projection matrix:  d_model × d_model = 4096 × 4096

LoRA parameters per projection:
  A ∈ ℝ^(r × d_model) = 16 × 4096 = 65,536
  B ∈ ℝ^(d_model × r) = 4096 × 16 = 65,536
  Total per projection: 131,072

Four projections per layer (Q, K, V, O):
  4 × 131,072 = 524,288 per layer

All 32 layers:
  32 × 524,288 = 16,777,216 ≈ 16.8M trainable parameters

Original model (attention weights only):
  32 × 4 × 4096 × 4096 = 2,147,483,648 ≈ 2.15B

Percentage trainable:
  16.8M / 2.15B ≈ 0.78%

If the full model has 7B params total:
  16.8M / 7B ≈ 0.24%
```

---

## Worked Example: QLoRA Memory Estimation

**Problem:** Estimate GPU memory for QLoRA fine-tuning of LLaMA-2 70B.

```
Base model (NF4 quantized):
  70B × 0.5 bytes = 35 GB
  Double quantization savings: -0.37 bits/param × 70B / 8 = -3.2 GB
  Net base model: ~32 GB

LoRA adapters (BF16, r=16, QKVO + MLP):
  ~40M params × 2 bytes = ~80 MB

Optimizer states (AdamW, BF16 for LoRA params only):
  40M × 8 bytes = ~320 MB

Gradients:
  40M × 2 bytes = ~80 MB

Activations (batch=1, seq_len=2048):
  ~3-6 GB (depends on gradient checkpointing)

Total: ~36-39 GB → fits on 1× A100 (40GB) or 1× A6000 (48GB)
```

---

## Python Code: LoRA Fine-Tuning with PEFT

```python
import torch
from datasets import load_dataset
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    BitsAndBytesConfig,
)
from peft import (
    LoraConfig,
    get_peft_model,
    prepare_model_for_kbit_training,
    TaskType,
)
from trl import SFTTrainer, SFTConfig

# ── 1. Load model with 4-bit quantization (QLoRA) ───────────────

model_name = "meta-llama/Meta-Llama-3-8B-Instruct"

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto",
    torch_dtype=torch.bfloat16,
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token

# Prepare model for k-bit training (freeze base, enable gradient for LoRA)
model = prepare_model_for_kbit_training(model)

# ── 2. Configure LoRA ───────────────────────────────────────────

lora_config = LoraConfig(
    r=16,                           # Rank
    lora_alpha=32,                  # Scaling: effective lr multiplier = alpha/r = 2
    lora_dropout=0.05,              # Dropout on LoRA layers
    target_modules=[                # Which layers to apply LoRA to
        "q_proj", "k_proj", "v_proj", "o_proj",  # Attention
        "gate_proj", "up_proj", "down_proj",      # MLP (optional, improves quality)
    ],
    bias="none",                    # Don't train biases
    task_type=TaskType.CAUSAL_LM,
)

# Wrap model with LoRA
model = get_peft_model(model, lora_config)

# Print parameter summary
model.print_trainable_parameters()
# Output: trainable params: 41,943,040 || all params: 8,072,204,288 || trainable%: 0.5196

# ── 3. Load and format dataset ──────────────────────────────────

dataset = load_dataset("tatsu-lab/alpaca", split="train[:5000]")

def format_instruction(example):
    """Convert Alpaca format to chat format."""
    if example.get("input", ""):
        text = (
            f"### Instruction:\n{example['instruction']}\n\n"
            f"### Input:\n{example['input']}\n\n"
            f"### Response:\n{example['output']}"
        )
    else:
        text = (
            f"### Instruction:\n{example['instruction']}\n\n"
            f"### Response:\n{example['output']}"
        )
    return {"text": text}

dataset = dataset.map(format_instruction, remove_columns=dataset.column_names)

# ── 4. Training configuration ───────────────────────────────────

training_args = SFTConfig(
    output_dir="./llama3-lora",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    lr_scheduler_type="cosine",
    warmup_ratio=0.05,
    weight_decay=0.01,
    bf16=True,
    max_grad_norm=0.3,
    logging_steps=10,
    save_strategy="epoch",
    max_seq_length=1024,
    packing=False,
    optim="paged_adamw_8bit",
)

# ── 5. Train ────────────────────────────────────────────────────

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset,
    peft_config=lora_config,
)

trainer.train()

# ── 6. Save adapter weights ────────────────────────────────────

model.save_pretrained("./llama3-lora/adapter")
tokenizer.save_pretrained("./llama3-lora/adapter")
print("Adapter size:", sum(
    f.stat().st_size for f in Path("./llama3-lora/adapter").rglob("*") if f.is_file()
) / 1e6, "MB")

# ── 7. Load and use the adapter for inference ──────────────────

from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto",
)

# Load the LoRA adapter on top of the base
model_with_adapter = PeftModel.from_pretrained(base_model, "./llama3-lora/adapter")

# Optionally merge for faster inference (no overhead)
merged_model = model_with_adapter.merge_and_unload()

# Generate
inputs = tokenizer("Explain photosynthesis:", return_tensors="pt").to("cuda")
with torch.no_grad():
    output = merged_model.generate(**inputs, max_new_tokens=200, temperature=0.7)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

---

## Multi-Adapter Serving

One major advantage of PEFT is serving multiple tasks from a single base model:

```python
from peft import PeftModel

# Load base model once
base_model = AutoModelForCausalLM.from_pretrained("meta-llama/Meta-Llama-3-8B")

# Load different adapters for different tasks
medical_model = PeftModel.from_pretrained(base_model, "./adapters/medical")
legal_model = PeftModel.from_pretrained(base_model, "./adapters/legal")
code_model = PeftModel.from_pretrained(base_model, "./adapters/code")

# Switch adapters dynamically at inference time
# Base model (14 GB) + each adapter (~80 MB) = minimal extra memory
```

```
┌──────────────────────────────────────────────────┐
│              Multi-Adapter Serving                │
│                                                   │
│         ┌────────────────────────┐                │
│         │   Base Model (frozen)  │                │
│         │   LLaMA-3 8B ≈ 16GB   │                │
│         └───────────┬────────────┘                │
│                     │                             │
│      ┌──────────────┼──────────────┐              │
│      │              │              │              │
│  ┌───┴────┐    ┌────┴───┐    ┌────┴───┐          │
│  │Medical │    │ Legal  │    │  Code  │          │
│  │Adapter │    │Adapter │    │Adapter │          │
│  │ ~80MB  │    │ ~80MB  │    │ ~80MB  │          │
│  └────────┘    └────────┘    └────────┘          │
│                                                   │
│  Total: 16 GB + 240 MB vs 48 GB for 3 full models│
└──────────────────────────────────────────────────┘
```

---

## Real-World Applications

| Application | PEFT Method | Why This Method |
|---|---|---|
| Domain adaptation (medical, legal) | LoRA / QLoRA | Best quality-to-cost ratio |
| Multi-tenant SaaS (per-customer tuning) | LoRA adapters | Swap adapters per request |
| Edge deployment fine-tuning | QLoRA | Fits on consumer GPUs |
| Rapid prototyping | Prompt tuning | Minimal training, fast iteration |
| Multi-task learning | Adapters | Separate adapter per task, shared base |
| Research experiments | LoRA | Quick experiments, easy reproducibility |

---

## Summary Table

| Concept | Key Point |
|---|---|
| LoRA | W' = W₀ + BA; low-rank update, merge at inference for zero overhead |
| QLoRA | 4-bit NF4 base + LoRA; fine-tune 70B on single GPU |
| Rank r | Start with 16; increase for complex tasks |
| lora_alpha | Scaling factor; effective scale = α/r |
| Adapters | Bottleneck layers (down → nonlinear → up) with residual |
| Prefix tuning | Learnable KV prefixes in every layer |
| Prompt tuning | Learnable soft input tokens; best for very large models |
| Initialization | B=0, A~N(0,σ²) so ΔW=0 at start |
| Multi-adapter | One base model + many small adapters for different tasks |
| Target modules | Apply LoRA to attention (QKVO) + optionally MLP layers |

---

## Revision Questions

1. **Derive the parameter savings of LoRA.** For a weight matrix W ∈ ℝ^(4096×4096) with rank r=8, what fraction of parameters are trainable compared to full fine-tuning?

2. **Explain how QLoRA achieves its memory savings.** What are the three key innovations (NF4, double quantization, paged optimizers)?

3. **Compare LoRA and adapter methods.** What are the trade-offs in terms of inference overhead, parameter count, and performance?

4. **Why does prompt tuning work better for larger models?** What does this suggest about the relationship between model capacity and the ability to be steered by soft prompts?

5. **Design a PEFT strategy** for fine-tuning LLaMA-3 70B on a medical Q&A dataset (10K examples) using a single A100 (80GB). Specify rank, target modules, quantization, and expected memory usage.

6. **Explain LoRA initialization.** Why is B initialised to zeros? What would happen if both A and B were initialised randomly?

---

[← Fine-tuning LLMs](02-fine-tuning-llms.md) | [RLHF Basics →](04-rlhf-basics.md)
