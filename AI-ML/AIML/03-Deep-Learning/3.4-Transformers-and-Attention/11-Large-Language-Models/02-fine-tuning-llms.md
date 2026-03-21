[← LLM Landscape](01-llm-landscape.md) | [Parameter-Efficient Fine-tuning →](03-parameter-efficient-fine-tuning.md)

---

# Fine-tuning Large Language Models

## Overview

Fine-tuning adapts a pre-trained Large Language Model to perform well on a specific task, domain, or behavioural style by continuing training on a curated dataset. While pre-training gives the model broad language understanding, fine-tuning sharpens it — teaching the model to follow instructions, respond in a particular tone, handle domain-specific terminology, or produce structured outputs. The three main approaches are **full fine-tuning** (updating all parameters), **instruction tuning** (training on instruction-response pairs), and **supervised fine-tuning (SFT)** on high-quality curated data. Fine-tuning is the critical bridge between a general-purpose foundation model and a production-ready system that consistently meets user expectations.

---

## Full Fine-Tuning

Full fine-tuning updates **every parameter** in the model using standard backpropagation and gradient descent on a task-specific dataset.

### The Objective

The training minimises the cross-entropy loss over the target tokens:

```
L(θ) = - (1/T) Σₜ log P(yₜ | y₁, ..., yₜ₋₁, x; θ)

Where:
  θ   = all model parameters
  x   = input prompt / context
  yₜ  = target token at position t
  T   = total target tokens
```

### Memory Requirements

```
GPU Memory for Full Fine-Tuning (FP16 mixed precision with AdamW):

  Model weights (FP16):          2 × N bytes
  Optimizer states (FP32):       8 × N bytes  (momentum + variance + master weights)
  Gradients (FP16):              2 × N bytes
  Activations:                   ~1-2 × N bytes (varies with batch & seq length)
  ──────────────────────────────────────────
  Total:                        ~12-14 × N bytes

  Example (7B model):
    12 × 7×10⁹ = 84 GB → minimum 2× A100 80GB GPUs

  Example (70B model):
    12 × 70×10⁹ = 840 GB → minimum 12× A100 80GB GPUs
```

### Catastrophic Forgetting

Full fine-tuning can cause **catastrophic forgetting** — the model loses general capabilities while specialising:

```
┌─────────────────────────────────────────────────────────────┐
│              Catastrophic Forgetting                         │
│                                                              │
│  Performance                                                 │
│  ▲                                                           │
│  │   ████  General capability                                │
│  │   ████████                                                │
│  │   ████████████                                            │
│  │   ████████████  ○○○○  Task-specific performance           │
│  │   █████████  ○○○○○○○○○○○○                                 │
│  │   ██████  ○○○○○○○○○○○○○○○○○                               │
│  │   ███  ○○○○○○○○○○○○○○○○○○○○○                              │
│  │   █ ○○○○○○○○○○○○○○○○○○○○○○○○○                             │
│  └──────────────────────────────────► Fine-tuning steps      │
│                                                              │
│  Solution: use low learning rate, keep fraction of general   │
│  data, or use PEFT methods (LoRA, etc.)                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Instruction Tuning

Instruction tuning trains the model on **(instruction, response)** pairs so it learns to follow user instructions rather than just predict the next token.

### Data Format

```
### Instruction:
Summarise the following article in three bullet points.

### Input:
<article text here>

### Response:
• Point one...
• Point two...
• Point three...
```

Or in chat format:

```json
{
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Summarise this article: ..."},
    {"role": "assistant", "content": "• Point one\n• Point two\n• Point three"}
  ]
}
```

### Key Instruction Tuning Datasets

| Dataset | Size | Description |
|---|---|---|
| FLAN Collection | ~15M | Multi-task instruction data from Google |
| OpenAssistant | ~160K | Human-generated multi-turn conversations |
| Alpaca | ~52K | GPT-generated instruction-following data |
| Dolly | ~15K | Databricks employee-generated instructions |
| ShareGPT | ~90K | User-shared ChatGPT conversations |
| UltraChat | ~1.5M | Multi-round, diverse topic conversations |
| SlimOrca | ~518K | Curated subset of OpenOrca dataset |

---

## Supervised Fine-Tuning (SFT)

SFT uses **curated, high-quality** examples — typically the first step in the RLHF pipeline (before reward modeling and PPO). Quality matters far more than quantity.

### The Quality Principle

```
Performance gain vs. dataset size and quality:

  High quality, small (1-10K)  ████████████████████  ✓ Excellent
  High quality, medium (10-50K) █████████████████████████  ✓ Best
  Low quality, large (500K+)   ████████████          ✗ Diminishing returns
  Low quality, small (1-5K)    ███                   ✗ Poor

  Key insight: 1,000 expert-curated examples often outperform
               100,000 noisy, auto-generated examples.
```

### Data Curation Pipeline

```
┌──────────┐   ┌──────────────┐   ┌──────────────┐   ┌────────────┐
│ Raw Data │──→│   Filtering  │──→│  Formatting  │──→│  Quality   │
│ Sources  │   │  & Cleaning  │   │  (Chat/Inst) │   │  Review    │
└──────────┘   └──────────────┘   └──────────────┘   └─────┬──────┘
                                                           │
                                                           ▼
                                                    ┌────────────┐
                                                    │  Balanced  │
                                                    │  Training  │
                                                    │   Dataset  │
                                                    └────────────┘

Filtering steps:
  1. Remove duplicates (exact and near-duplicate)
  2. Filter by length (too short / too long)
  3. Remove toxic or low-quality content
  4. Ensure task diversity
  5. Balance difficulty levels
```

---

## Fine-Tuning Pipeline

```
┌───────────────────────────────────────────────────────────────────┐
│                    Fine-Tuning Pipeline                            │
│                                                                   │
│  ┌──────────┐    ┌──────────────┐    ┌───────────────┐            │
│  │Pre-trained│    │  Task-Specific│    │  Fine-tuned   │            │
│  │  Model    │───→│   Dataset     │───→│    Model      │            │
│  │ (frozen)  │    │ (formatted)   │    │ (all params   │            │
│  │          │    │              │    │  updated)     │            │
│  └──────────┘    └──────────────┘    └───────┬───────┘            │
│       │                                       │                   │
│       │         Hyperparameters:              │                   │
│       │         • lr: 1e-5 to 5e-5           │                   │
│       │         • epochs: 1-5                │                   │
│       │         • batch_size: 4-32           │                   │
│       │         • warmup: 3-10%              ▼                   │
│       │                              ┌───────────────┐            │
│       │                              │  Evaluation   │            │
│       │                              │  (val loss,   │            │
│       │                              │   benchmarks) │            │
│       │                              └───────────────┘            │
└───────────────────────────────────────────────────────────────────┘
```

---

## Practical Considerations

### Learning Rate Selection

```
Recommended learning rates by model size:

  < 1B params:    2e-5 to 5e-5
  1B - 7B:        1e-5 to 2e-5
  7B - 13B:       5e-6 to 1e-5
  13B - 70B:      1e-6 to 5e-6
  > 70B:          5e-7 to 1e-6

Schedule: cosine decay with linear warmup (3-10% of total steps)

  lr(t) = lr_max × 0.5 × (1 + cos(π × (t - t_warmup) / (T - t_warmup)))
  
  for t > t_warmup, and linear ramp for t ≤ t_warmup
```

### Epochs and Overfitting

```
Optimal number of epochs by dataset size:

  < 1K examples:     1-2 epochs   (high risk of overfitting)
  1K - 10K:          2-3 epochs
  10K - 100K:        1-3 epochs
  > 100K:            1 epoch      (single pass often sufficient)

Monitor: validation loss should decrease; stop if it increases for
2+ consecutive checkpoints (early stopping).
```

### Gradient Accumulation for Limited Memory

When batch size is limited by GPU memory:

```
Effective batch size = micro_batch_size × gradient_accumulation_steps × n_gpus

Example:
  Desired batch size:            32
  GPU memory allows:             micro_batch = 2
  Number of GPUs:                2
  Gradient accumulation steps:   32 / (2 × 2) = 8
```

---

## When to Fine-Tune vs When to Prompt

```
┌─────────────────────────────────────────────────────────────────┐
│                Decision Framework                                │
│                                                                  │
│                    Does the task require                          │
│                    specialised knowledge?                         │
│                       /          \                                │
│                     Yes           No                              │
│                      │             │                              │
│              Is the knowledge      │                              │
│              in the base model?    │                              │
│                /          \        │                               │
│              Yes           No      │                               │
│               │             │      │                               │
│          Try prompting  Fine-tune  Try prompting                  │
│          (few-shot,     (with domain (zero/few-shot)              │
│           CoT)          data)                                     │
│               │             │            │                        │
│          Works well?   Enough data?  Works well?                 │
│            /    \       /     \       /     \                     │
│          Yes    No    Yes     No   Yes      No                   │
│           │      │     │       │    │        │                    │
│         Done  Fine-  Fine-   Use   Done   Fine-tune             │
│               tune   tune    RAG          (instruction           │
│                                            tuning)               │
└─────────────────────────────────────────────────────────────────┘
```

| Scenario | Approach | Reason |
|---|---|---|
| General Q&A | Prompting | Base model already capable |
| Domain jargon (medical, legal) | Fine-tune | Need specialised vocabulary & style |
| Structured output (JSON) | Fine-tune or function calling | Consistency in format |
| Up-to-date knowledge | RAG | Fine-tuning data becomes stale |
| Style / persona | Fine-tune (small SFT) | Learn consistent tone |
| Complex reasoning | Prompting (CoT) | Reasoning is emergent, hard to fine-tune |

---

## Worked Example: Fine-Tuning Loss Calculation

**Problem:** A fine-tuning batch has 4 sequences, each with 512 target tokens. Calculate the loss if the model's average per-token negative log-likelihood is 1.8.

```
Total tokens in batch: 4 × 512 = 2048

Batch loss = (1/2048) × Σᵢ (-log P(yᵢ | context))
           = (1/2048) × (2048 × 1.8)
           = 1.8

Perplexity = e^(loss) = e^1.8 ≈ 6.05

Interpretation: On average, the model is as uncertain as choosing
uniformly among ~6 options for each token.

After fine-tuning, we expect:
  Loss:       1.8 → ~0.8-1.2
  Perplexity: 6.05 → ~2.2-3.3
```

---

## Python Code: Fine-Tuning with HuggingFace SFTTrainer

```python
import torch
from datasets import load_dataset
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    TrainingArguments,
    BitsAndBytesConfig,
)
from trl import SFTTrainer, SFTConfig
from peft import LoraConfig

# ── 1. Load base model with 4-bit quantization ──────────────────

model_name = "meta-llama/Meta-Llama-3-8B"

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
    attn_implementation="flash_attention_2",
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"

# ── 2. Prepare the dataset ──────────────────────────────────────

dataset = load_dataset("HuggingFaceH4/ultrachat_200k", split="train_sft[:5000]")

def format_chat(example):
    """Convert messages to the model's chat template."""
    chat = tokenizer.apply_chat_template(
        example["messages"],
        tokenize=False,
        add_generation_prompt=False,
    )
    return {"text": chat}

dataset = dataset.map(format_chat, remove_columns=dataset.column_names)
dataset = dataset.train_test_split(test_size=0.1)

# ── 3. Configure LoRA (for memory efficiency) ───────────────────

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    lora_dropout=0.05,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
    bias="none",
    task_type="CAUSAL_LM",
)

# ── 4. Training arguments ───────────────────────────────────────

training_args = SFTConfig(
    output_dir="./llama3-8b-sft",
    num_train_epochs=3,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    learning_rate=2e-4,
    lr_scheduler_type="cosine",
    warmup_ratio=0.05,
    weight_decay=0.01,
    fp16=False,
    bf16=True,
    max_grad_norm=1.0,
    logging_steps=10,
    save_strategy="steps",
    save_steps=100,
    eval_strategy="steps",
    eval_steps=100,
    max_seq_length=2048,
    packing=True,
    report_to="wandb",
)

# ── 5. Create trainer and start fine-tuning ──────────────────────

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"],
    peft_config=lora_config,
)

# Print trainable parameter count
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print(f"Trainable: {trainable:,} / {total:,} ({100*trainable/total:.2f}%)")

# Start training
trainer.train()

# ── 6. Save and merge ───────────────────────────────────────────

trainer.save_model("./llama3-8b-sft/final")

# Merge LoRA weights into base model for deployment
merged_model = model.merge_and_unload()
merged_model.save_pretrained("./llama3-8b-sft/merged")
tokenizer.save_pretrained("./llama3-8b-sft/merged")
```

---

## Evaluation After Fine-Tuning

```python
from transformers import pipeline

# Load the fine-tuned model
pipe = pipeline(
    "text-generation",
    model="./llama3-8b-sft/merged",
    tokenizer=tokenizer,
    torch_dtype=torch.bfloat16,
    device_map="auto",
)

# Test on held-out examples
test_prompts = [
    "Explain quantum entanglement to a 10-year-old.",
    "Write a Python function to merge two sorted lists.",
    "Summarise the causes of World War I in three bullet points.",
]

for prompt in test_prompts:
    messages = [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": prompt},
    ]
    output = pipe(messages, max_new_tokens=256, do_sample=True, temperature=0.7)
    print(f"\nPrompt: {prompt}")
    print(f"Response: {output[0]['generated_text'][-1]['content']}")
    print("-" * 60)
```

---

## Real-World Applications

| Application | Fine-Tuning Approach | Dataset Size | Notes |
|---|---|---|---|
| Customer service bot | SFT on support transcripts | 5K-50K | Style and policy alignment |
| Medical report generation | Full FT on clinical data | 10K-100K | Requires domain expertise |
| Code assistant | SFT on code + instruction | 50K-500K | Language-specific fine-tuning |
| Legal document drafting | SFT on contracts/briefs | 1K-10K | High-quality expert annotations |
| Content moderation | Classification fine-tuning | 10K-50K | Safety-critical, needs evaluation |
| Language translation | Full FT on parallel corpora | 100K+ | Bilingual or multilingual data |

---

## Summary Table

| Concept | Key Point |
|---|---|
| Full fine-tuning | Updates all parameters; best performance but highest cost |
| Instruction tuning | Trains on (instruction, response) pairs for task following |
| SFT | Curated high-quality data; first stage of RLHF pipeline |
| Learning rate | Smaller for larger models; cosine schedule with warmup |
| Catastrophic forgetting | Model loses general ability; mitigate with low LR or PEFT |
| Data quality | 1K expert examples > 100K noisy examples |
| Memory formula | ~12-14× parameter count in bytes for training |
| Gradient accumulation | Simulates large batch sizes on limited GPU memory |
| Epochs | 1-3 for most fine-tuning; monitor validation loss |
| Evaluation | Perplexity, task-specific metrics, human evaluation |

---

## Revision Questions

1. **Derive the memory requirement** for full fine-tuning of a 13B parameter model with AdamW in mixed precision. How many A100 (80GB) GPUs are needed?

2. **What is catastrophic forgetting** and what strategies can mitigate it during fine-tuning?

3. **Compare instruction tuning and supervised fine-tuning.** When would you use each approach?

4. **A dataset has 5,000 high-quality examples and 200,000 noisy auto-generated examples.** Design a training strategy that maximises model quality.

5. **Explain the role of gradient accumulation** in fine-tuning. If you have 4 GPUs, a micro batch size of 1, and want an effective batch size of 32, how many accumulation steps do you need?

6. **When should you use RAG instead of fine-tuning?** Give two scenarios where each approach is clearly superior.

---

[← LLM Landscape](01-llm-landscape.md) | [Parameter-Efficient Fine-tuning →](03-parameter-efficient-fine-tuning.md)
