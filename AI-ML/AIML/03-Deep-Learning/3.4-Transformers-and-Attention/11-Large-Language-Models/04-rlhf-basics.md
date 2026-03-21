[← PEFT](03-parameter-efficient-fine-tuning.md) | [Advanced Prompting →](05-prompt-engineering-advanced.md)

---

# Reinforcement Learning from Human Feedback (RLHF)

## Overview

Reinforcement Learning from Human Feedback (RLHF) is the dominant technique for aligning Large Language Models with human preferences, transforming a capable but undirected pre-trained model into a helpful, harmless, and honest assistant. Pioneered by the InstructGPT paper (Ouyang et al., 2022), RLHF follows a three-stage pipeline: (1) supervised fine-tuning on high-quality demonstrations, (2) training a reward model from human preference comparisons, and (3) optimising the language model's policy using Proximal Policy Optimisation (PPO) against the reward model while staying close to the original policy via a KL divergence penalty. More recently, **Direct Preference Optimisation (DPO)** has emerged as a simpler alternative that eliminates the reward model entirely by directly optimising the policy from preference data. Understanding RLHF and its variants is essential for anyone building or evaluating aligned AI systems.

---

## The Three-Stage RLHF Pipeline

```
┌──────────────────────────────────────────────────────────────────────┐
│                     RLHF Pipeline (InstructGPT)                       │
│                                                                       │
│  Stage 1: Supervised Fine-Tuning (SFT)                                │
│  ┌──────────────┐    ┌────────────────┐    ┌──────────────────┐       │
│  │  Pre-trained  │───→│  Demonstration │───→│  SFT Model       │       │
│  │  LLM          │    │  Dataset       │    │  π_SFT           │       │
│  └──────────────┘    │  (human-written)│    └────────┬─────────┘       │
│                      └────────────────┘             │                 │
│                                                      │                 │
│  Stage 2: Reward Model Training                      │                 │
│  ┌──────────────┐    ┌────────────────┐    ┌────────┴─────────┐       │
│  │  π_SFT       │───→│  Generate      │───→│  Human ranks     │       │
│  │  generates    │    │  K responses   │    │  responses       │       │
│  │  responses    │    │  per prompt    │    │  y_w ≻ y_l       │       │
│  └──────────────┘    └────────────────┘    └────────┬─────────┘       │
│                                                      │                 │
│                                            ┌─────────┴────────┐       │
│                                            │  Reward Model    │       │
│                                            │  r_φ(x, y)       │       │
│                                            └─────────┬────────┘       │
│                                                      │                 │
│  Stage 3: RL Optimization (PPO)                      │                 │
│  ┌──────────────┐    ┌────────────────┐    ┌─────────┴────────┐       │
│  │  π_SFT       │───→│  PPO Training  │←───│  Reward signal   │       │
│  │  (init)      │    │  π_θ           │    │  r_φ(x, y)       │       │
│  └──────────────┘    └────────┬───────┘    └──────────────────┘       │
│                               │                                       │
│                      ┌────────┴───────┐                               │
│                      │  Aligned Model │                               │
│                      │  π_RLHF        │                               │
│                      └────────────────┘                               │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Stage 1: Supervised Fine-Tuning (SFT)

The first stage is standard supervised fine-tuning on a dataset of high-quality **(prompt, response)** pairs written by human labellers.

```
Dataset: D_SFT = {(x₁, y₁), (x₂, y₂), ..., (xₙ, yₙ)}

Loss:  L_SFT(θ) = -𝔼_(x,y)~D_SFT [ Σₜ log π_θ(yₜ | x, y₁:ₜ₋₁) ]

This is identical to standard language model fine-tuning:
  - Minimise cross-entropy on target tokens
  - Learning rate: 1e-5 to 5e-5
  - Epochs: 1-3
  - Result: π_SFT — a model that follows instructions
```

---

## Stage 2: Reward Model Training

### Data Collection

Human annotators compare **K responses** (typically K=4) to the same prompt and **rank them** by quality:

```
Prompt: "Explain quantum computing to a child."

Response A: "Quantum computing uses qubits that can be 0 and 1..."  ← Rank 1 (best)
Response B: "It's like a super fast computer..."                    ← Rank 2
Response C: "Quantum mechanics is a branch of physics..."          ← Rank 3
Response D: "I don't know about that."                             ← Rank 4

From K responses, we get C(K,2) = K(K-1)/2 comparison pairs.
For K=4: 6 pairs per prompt.
```

### Bradley-Terry Model

The reward model is trained using the **Bradley-Terry** preference model:

```
P(y_w ≻ y_l | x) = σ(r_φ(x, y_w) - r_φ(x, y_l))

Where:
  y_w     = preferred (winning) response
  y_l     = dispreferred (losing) response
  r_φ     = reward model (scalar output)
  σ       = sigmoid function
  x       = prompt

Loss function (negative log-likelihood of preferences):

  L_RM(φ) = -𝔼_(x, y_w, y_l) [ log σ(r_φ(x, y_w) - r_φ(x, y_l)) ]
```

### Reward Model Architecture

```
┌──────────────────────────────────────────┐
│           Reward Model r_φ                │
│                                           │
│  ┌────────────────────────────────────┐   │
│  │  Transformer (same as LLM)        │   │
│  │  - Typically initialised from SFT  │   │
│  │  - Remove the LM head              │   │
│  └────────────────┬───────────────────┘   │
│                   │                       │
│                   ▼                       │
│  ┌────────────────────────────────────┐   │
│  │  Linear projection: ℝ^d → ℝ^1    │   │
│  │  (single scalar reward)           │   │
│  └────────────────┬───────────────────┘   │
│                   │                       │
│                   ▼                       │
│              r_φ(x, y) ∈ ℝ               │
│              (scalar reward)              │
└──────────────────────────────────────────┘

The reward is computed on the last token position (or pooled).
```

### Worked Example: Reward Model Loss

```
Given a preference pair with computed rewards:
  r_φ(x, y_w) = 2.5   (preferred response)
  r_φ(x, y_l) = 1.0   (dispreferred response)

Loss = -log σ(r_w - r_l)
     = -log σ(2.5 - 1.0)
     = -log σ(1.5)
     = -log(0.8176)
     = 0.2014

If rewards were reversed (model predicts wrong):
  r_φ(x, y_w) = 1.0, r_φ(x, y_l) = 2.5
  Loss = -log σ(-1.5) = -log(0.1824) = 1.7014

The loss heavily penalises incorrect preference predictions.
```

---

## Stage 3: PPO Optimisation

### The RL Formulation

The language model is treated as a **policy** π_θ that generates token sequences (actions) given a prompt (state). The reward model provides the reward signal.

```
Objective:

  max_θ  𝔼_{x~D, y~π_θ(·|x)} [ r_φ(x, y) ] - β · KL(π_θ || π_ref)

Where:
  π_θ     = current policy (the LLM being optimised)
  π_ref   = reference policy (the SFT model, frozen)
  r_φ     = reward model (frozen)
  β       = KL penalty coefficient (controls deviation from π_ref)
  D       = distribution of prompts

The KL penalty prevents the model from:
  1. Exploiting the reward model ("reward hacking")
  2. Generating degenerate/unnatural text to maximise reward
  3. Straying too far from the pre-trained distribution
```

### KL Divergence Penalty

```
KL(π_θ || π_ref) = 𝔼_{y~π_θ} [ log(π_θ(y|x) / π_ref(y|x)) ]

Per-token KL:
  KL_t = log π_θ(yₜ | x, y₁:ₜ₋₁) - log π_ref(yₜ | x, y₁:ₜ₋₁)

Total KL for a sequence:
  KL = Σₜ KL_t

Modified reward with KL penalty:
  R(x, y) = r_φ(x, y) - β · Σₜ [ log π_θ(yₜ|...) - log π_ref(yₜ|...) ]
```

### PPO Algorithm for LLMs

```
PPO Clipped Objective:

  L_PPO(θ) = 𝔼_t [ min(ρₜ · Âₜ,  clip(ρₜ, 1-ε, 1+ε) · Âₜ) ]

Where:
  ρₜ = π_θ(aₜ|sₜ) / π_θ_old(aₜ|sₜ)     (probability ratio)
  Âₜ = advantage estimate (GAE)
  ε  = clipping parameter (typically 0.2)

For language models:
  - State sₜ = (x, y₁:ₜ₋₁)              (prompt + generated tokens so far)
  - Action aₜ = yₜ                        (next token)
  - ρₜ = π_θ(yₜ|x,y₁:ₜ₋₁) / π_θ_old(yₜ|x,y₁:ₜ₋₁)

Advantage estimation (GAE-λ):
  Âₜ = Σₗ (γλ)ˡ · δₜ₊ₗ
  δₜ = rₜ + γ·V(sₜ₊₁) - V(sₜ)

In practice for LLMs:
  - γ = 1 (no discounting within a response)
  - The reward is only given at the last token
  - Per-token KL penalties serve as intermediate rewards
```

### The PPO Training Loop

```
┌────────────────────────────────────────────────────────────────┐
│                    PPO Training Loop                            │
│                                                                 │
│  for each iteration:                                            │
│    1. Sample batch of prompts x ~ D                             │
│    2. Generate responses y ~ π_θ(·|x)                           │
│    3. Compute rewards r_φ(x, y) from reward model               │
│    4. Compute KL penalties: KL(π_θ || π_ref) per token          │
│    5. Compute advantages Â using value function V               │
│    6. Update π_θ using PPO clipped objective (multiple epochs)  │
│    7. Update value function V                                   │
│    8. Log metrics: reward, KL, entropy, loss                    │
│                                                                 │
│  Typical hyperparameters:                                       │
│    - PPO epochs per iteration:  4                               │
│    - Mini-batch size:           64                              │
│    - Learning rate:             1e-6 to 5e-6                    │
│    - KL coefficient β:         0.01 to 0.2                     │
│    - Clip ε:                   0.2                              │
│    - Total iterations:          ~10,000-50,000                  │
└────────────────────────────────────────────────────────────────┘
```

---

## Direct Preference Optimization (DPO)

DPO (Rafailov et al., 2023) simplifies RLHF by **eliminating the reward model** and optimising the policy directly from preference pairs.

### Key Insight

The optimal policy under the RLHF objective has a closed-form relationship with the reward:

```
r(x, y) = β · log(π*(y|x) / π_ref(y|x)) + β · log Z(x)

This means we can express the reward model implicitly through
the policy itself, eliminating the need to train it separately.
```

### DPO Loss

```
L_DPO(θ) = -𝔼_(x, y_w, y_l) [ log σ( β · ( log π_θ(y_w|x)/π_ref(y_w|x)
                                             - log π_θ(y_l|x)/π_ref(y_l|x) ) ) ]

Simplified notation:
  L_DPO = -𝔼 [ log σ( β · (r̂_w - r̂_l) ) ]

Where:
  r̂_w = log π_θ(y_w|x) - log π_ref(y_w|x)    (implicit reward for preferred)
  r̂_l = log π_θ(y_l|x) - log π_ref(y_l|x)    (implicit reward for dispreferred)
```

### DPO vs RLHF Comparison

```
┌────────────────────────────────────────────────────────────────┐
│                    RLHF (PPO)                                   │
│  Stages:  SFT → Reward Model → PPO                             │
│  Models:  4 (SFT, RM, Policy, Value)                            │
│  Memory:  Very high (multiple model copies)                     │
│  Stability: Tricky (reward hacking, mode collapse)              │
│  Code complexity: High                                          │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│                    DPO                                           │
│  Stages:  SFT → DPO                                             │
│  Models:  2 (Policy + Reference)                                 │
│  Memory:  Lower (no reward model or value function)              │
│  Stability: More stable (standard cross-entropy-like loss)       │
│  Code complexity: Low                                            │
└────────────────────────────────────────────────────────────────┘

Performance:  DPO ≈ PPO-based RLHF on most benchmarks
              PPO may have edge on very complex alignment tasks
```

---

## Worked Example: DPO Loss Computation

**Problem:** Compute DPO loss for a preference pair with β=0.1.

```
Given log-probabilities:
  log π_θ(y_w|x) = -15.2    (preferred response under current policy)
  log π_ref(y_w|x) = -16.0  (preferred response under reference)
  log π_θ(y_l|x) = -18.5    (dispreferred under current policy)
  log π_ref(y_l|x) = -17.0  (dispreferred under reference)

Step 1: Compute implicit rewards
  r̂_w = (-15.2) - (-16.0) = 0.8     (policy assigns more prob than ref → positive)
  r̂_l = (-18.5) - (-17.0) = -1.5    (policy assigns less prob than ref → negative)

Step 2: Compute reward margin
  Δ = β × (r̂_w - r̂_l) = 0.1 × (0.8 - (-1.5)) = 0.1 × 2.3 = 0.23

Step 3: Compute loss
  L = -log σ(0.23) = -log(0.5572) = 0.586

Interpretation: The loss is moderate because the model already
somewhat prefers y_w over y_l, but the margin could be larger.
As training continues, L → 0 as the margin increases.
```

---

## Python Code: Simplified RLHF Training Loop

```python
import torch
import torch.nn.functional as F
from transformers import AutoModelForCausalLM, AutoTokenizer

def compute_reward(reward_model, input_ids, attention_mask):
    """Compute scalar reward for a response."""
    with torch.no_grad():
        outputs = reward_model(input_ids=input_ids, attention_mask=attention_mask)
        # Reward from last hidden state → linear head
        reward = outputs.logits[:, -1, 0]  # scalar per sequence
    return reward

def compute_log_probs(model, input_ids, attention_mask):
    """Compute per-token log probabilities."""
    outputs = model(input_ids=input_ids, attention_mask=attention_mask)
    logits = outputs.logits[:, :-1, :]  # shift for next-token prediction
    labels = input_ids[:, 1:]

    log_probs = F.log_softmax(logits, dim=-1)
    token_log_probs = log_probs.gather(2, labels.unsqueeze(-1)).squeeze(-1)

    # Mask padding tokens
    mask = attention_mask[:, 1:]
    return (token_log_probs * mask).sum(dim=-1) / mask.sum(dim=-1)

def rlhf_training_step(
    policy_model,
    ref_model,
    reward_model,
    prompts,
    tokenizer,
    optimizer,
    beta=0.1,
    max_new_tokens=256,
):
    """Single RLHF training step with simplified PPO."""
    # 1. Generate responses from current policy
    inputs = tokenizer(prompts, return_tensors="pt", padding=True).to(policy_model.device)
    with torch.no_grad():
        generated = policy_model.generate(
            **inputs,
            max_new_tokens=max_new_tokens,
            do_sample=True,
            temperature=0.8,
            top_p=0.95,
        )

    gen_mask = torch.ones_like(generated)
    gen_mask[:, :inputs["input_ids"].shape[1]] = 0  # mask prompt tokens

    # 2. Compute rewards
    rewards = compute_reward(reward_model, generated, gen_mask)

    # 3. Compute log probs under policy and reference
    policy_log_probs = compute_log_probs(policy_model, generated, gen_mask)
    with torch.no_grad():
        ref_log_probs = compute_log_probs(ref_model, generated, gen_mask)

    # 4. KL penalty
    kl_divergence = policy_log_probs - ref_log_probs

    # 5. Modified reward = reward - β × KL
    modified_rewards = rewards - beta * kl_divergence

    # 6. Policy gradient loss (simplified REINFORCE)
    loss = -(modified_rewards * policy_log_probs).mean()

    # 7. Update
    optimizer.zero_grad()
    loss.backward()
    torch.nn.utils.clip_grad_norm_(policy_model.parameters(), max_norm=1.0)
    optimizer.step()

    return {
        "loss": loss.item(),
        "mean_reward": rewards.mean().item(),
        "mean_kl": kl_divergence.mean().item(),
        "mean_modified_reward": modified_rewards.mean().item(),
    }

# ── Training Loop ──────────────────────────────────────────────

policy_model = AutoModelForCausalLM.from_pretrained("path/to/sft_model")
ref_model = AutoModelForCausalLM.from_pretrained("path/to/sft_model")
ref_model.eval()
for p in ref_model.parameters():
    p.requires_grad = False

# reward_model would be trained separately (Stage 2)

optimizer = torch.optim.AdamW(policy_model.parameters(), lr=1e-6)

for step in range(10000):
    batch_prompts = sample_prompts(batch_size=16)
    metrics = rlhf_training_step(
        policy_model, ref_model, reward_model,
        batch_prompts, tokenizer, optimizer, beta=0.1,
    )

    if step % 100 == 0:
        print(f"Step {step}: reward={metrics['mean_reward']:.3f}, "
              f"KL={metrics['mean_kl']:.3f}, loss={metrics['loss']:.4f}")
```

---

## Python Code: DPO with TRL

```python
import torch
from datasets import load_dataset
from transformers import AutoModelForCausalLM, AutoTokenizer
from trl import DPOTrainer, DPOConfig

# ── 1. Load model and reference ─────────────────────────────────

model_name = "meta-llama/Meta-Llama-3-8B-Instruct"

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.bfloat16,
    device_map="auto",
)
ref_model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.bfloat16,
    device_map="auto",
)

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token

# ── 2. Load preference dataset ──────────────────────────────────

# DPO expects columns: "prompt", "chosen", "rejected"
dataset = load_dataset("argilla/ultrafeedback-binarized-preferences", split="train[:5000]")

def format_dpo(example):
    """Format dataset for DPO training."""
    return {
        "prompt": example["instruction"],
        "chosen": example["chosen_response"],
        "rejected": example["rejected_response"],
    }

dataset = dataset.map(format_dpo, remove_columns=dataset.column_names)

# ── 3. Configure DPO training ───────────────────────────────────

training_args = DPOConfig(
    output_dir="./llama3-dpo",
    num_train_epochs=1,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,
    learning_rate=5e-7,
    lr_scheduler_type="cosine",
    warmup_ratio=0.1,
    beta=0.1,                    # KL penalty coefficient
    max_length=1024,
    max_prompt_length=512,
    bf16=True,
    logging_steps=10,
    save_strategy="epoch",
    report_to="wandb",
)

# ── 4. Train with DPO ───────────────────────────────────────────

trainer = DPOTrainer(
    model=model,
    ref_model=ref_model,
    args=training_args,
    train_dataset=dataset,
    processing_class=tokenizer,
)

trainer.train()
trainer.save_model("./llama3-dpo/final")
```

---

## Real-World Applications

| Application | RLHF / DPO Use | Example |
|---|---|---|
| ChatGPT alignment | Full RLHF (SFT → RM → PPO) | Safe, helpful chat responses |
| Claude | Constitutional AI + RLHF | Principle-guided alignment |
| Code assistant safety | DPO on code preferences | Avoiding insecure code patterns |
| Content moderation | RM for toxicity scoring | Filtering harmful outputs |
| Summarization quality | DPO on summary preferences | Human-preferred summaries |
| Custom enterprise bots | DPO on internal feedback | Company policy adherence |

---

## Summary Table

| Concept | Key Point |
|---|---|
| RLHF pipeline | SFT → Reward Model → PPO (three stages) |
| Reward model | Learns scalar reward from human preference comparisons |
| Bradley-Terry | P(y_w ≻ y_l) = σ(r(x,y_w) - r(x,y_l)) |
| PPO | Policy gradient with clipped objective and KL penalty |
| KL penalty | Prevents reward hacking; keeps policy near reference |
| Value function | Estimates expected return for advantage computation |
| DPO | Eliminates reward model; optimises policy directly from prefs |
| DPO loss | -log σ(β · (log-ratio_w - log-ratio_l)) |
| β parameter | Controls deviation from reference (0.1 typical) |
| Constitutional AI | AI self-critique guided by principles + RLHF |

---

## Revision Questions

1. **Explain the three stages of the RLHF pipeline.** What is the role of each stage, and why is each necessary?

2. **Derive the Bradley-Terry reward model loss.** Given three ranked responses (A ≻ B ≻ C), how many comparison pairs are generated, and what is the total loss?

3. **Why is the KL divergence penalty necessary in PPO for LLMs?** What happens if β is set too high or too low?

4. **Compare RLHF (PPO) and DPO.** What are the advantages and disadvantages of each approach in terms of memory, stability, and implementation complexity?

5. **Walk through a DPO loss computation** for a preference pair where the policy strongly prefers the wrong response. What does the gradient signal look like?

6. **What is reward hacking in RLHF?** Give an example and explain how the KL penalty and reward model design can mitigate it.

---

[← PEFT](03-parameter-efficient-fine-tuning.md) | [Advanced Prompting →](05-prompt-engineering-advanced.md)
