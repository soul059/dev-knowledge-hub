# Autoregressive Language Modeling

> **Navigation:** [← GPT Architecture](01-gpt-architecture.md) | [GPT-2 and GPT-3 →](03-gpt2-and-gpt3.md)

---

## Overview

**Autoregressive language modeling** is the foundational training paradigm behind GPT and all decoder-only transformers. The core idea is simple yet powerful: model the joint probability of a sequence by decomposing it into a product of conditional probabilities using the **chain rule of probability**. At each step, the model predicts the next token given all previous tokens. This formulation naturally enables text generation — at inference time, tokens are produced one at a time, each conditioned on the tokens generated so far. This chapter covers the mathematical foundations, training objectives, teacher forcing, and the diverse sampling strategies used to generate high-quality text.

---

## 1. Chain Rule Factorization

The joint probability of a token sequence is decomposed autoregressively:

```
P(x₁, x₂, ..., xₙ) = P(x₁) · P(x₂|x₁) · P(x₃|x₁,x₂) · ... · P(xₙ|x₁,...,xₙ₋₁)

                     = ∏ₜ₌₁ⁿ P(xₜ | x₁, x₂, ..., xₜ₋₁)

                     = ∏ₜ₌₁ⁿ P(xₜ | x<ₜ)
```

### Why This Works

- **Exact**: No approximation — the chain rule is a mathematical identity
- **Ordered**: Imposes a natural left-to-right generation order
- **Tractable**: Each conditional can be parameterized by a neural network

### Diagram: Autoregressive Decomposition

```
  P("The cat sat down")

  = P("The")
  × P("cat"  | "The")
  × P("sat"  | "The", "cat")
  × P("down" | "The", "cat", "sat")

  Step 1:  [  ] ──────────────────────► P("The")       = 0.003
  Step 2:  [The] ─────────────────────► P("cat"|...)   = 0.015
  Step 3:  [The, cat] ────────────────► P("sat"|...)   = 0.042
  Step 4:  [The, cat, sat] ──────────► P("down"|...)   = 0.028

  Joint:   0.003 × 0.015 × 0.042 × 0.028 = 5.3 × 10⁻⁸
```

---

## 2. Next Token Prediction: Training Objective

### Cross-Entropy Loss

The model is trained to minimize the **cross-entropy loss** between the predicted distribution and the true next token:

```
L(θ) = -1/N · Σₜ₌₁ᴺ log P_θ(xₜ | x<ₜ)
```

Where:
- `θ` = model parameters
- `xₜ` = ground-truth token at position t
- `P_θ(xₜ | x<ₜ)` = model's predicted probability of the correct token
- `N` = sequence length

### Relationship to Perplexity

**Perplexity** is the standard evaluation metric for language models:

```
PPL = exp(L) = exp(-1/N · Σₜ log P(xₜ | x<ₜ))
```

| Perplexity | Interpretation                          |
|------------|-----------------------------------------|
| 1          | Perfect prediction (impossible)         |
| ~20-30     | Strong language model                   |
| ~50-100    | Decent language model                   |
| V (vocab)  | Random guessing (worst case)            |

Lower perplexity = better model. A perplexity of 25 means the model is as uncertain as if it were choosing uniformly among 25 options at each step.

---

## 3. Teacher Forcing

During training, the model receives the **ground-truth previous tokens** as input at every position, rather than its own predictions. This is called **teacher forcing**.

```
  Training (Teacher Forcing):
  ┌─────────────────────────────────────────────────┐
  │  Input:    [BOS]   The    cat    sat             │
  │  Target:    The    cat    sat    [EOS]           │
  │                                                  │
  │  At each position, the MODEL gets the TRUE       │
  │  previous token, NOT its own prediction.         │
  └─────────────────────────────────────────────────┘

  Inference (Autoregressive):
  ┌─────────────────────────────────────────────────┐
  │  Step 1:  [BOS]            → predict "The"      │
  │  Step 2:  [BOS, The]       → predict "cat"      │
  │  Step 3:  [BOS, The, cat]  → predict "sat"      │
  │  Step 4:  [BOS, The, cat, sat] → predict [EOS]  │
  │                                                  │
  │  At each step, the MODEL uses its OWN            │
  │  previous predictions as input.                  │
  └─────────────────────────────────────────────────┘
```

### Why Teacher Forcing?

| Advantage                  | Explanation                                       |
|----------------------------|---------------------------------------------------|
| Parallel training          | All positions computed in one forward pass         |
| Stable gradients           | No error accumulation from bad predictions         |
| Faster convergence         | Model learns from correct context                  |

### Exposure Bias

Teacher forcing creates a **train-test mismatch**: during training the model always sees ground-truth context, but during inference it sees its own (possibly incorrect) predictions. This discrepancy is called **exposure bias** and can cause error accumulation during generation.

---

## 4. Autoregressive Generation

At inference time, tokens are generated one at a time:

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │   Prompt: "Once upon a"                              │
  │                                                      │
  │   Step 1: Model("Once upon a")        → logits      │
  │           Sample from logits           → "time"      │
  │                                                      │
  │   Step 2: Model("Once upon a time")   → logits      │
  │           Sample from logits           → ","         │
  │                                                      │
  │   Step 3: Model("Once upon a time,")  → logits      │
  │           Sample from logits           → "there"     │
  │                                                      │
  │   ... continue until [EOS] or max_length             │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 5. Sampling Strategies

Given the logits `z` from the model, different strategies select the next token.

### 5.1 Greedy Decoding

Always pick the highest-probability token:

```
xₜ = argmax P(x | x<ₜ)
```

- **Pro**: Deterministic, fast
- **Con**: Repetitive, boring text; misses high-quality sequences

### 5.2 Temperature Scaling

Scale logits before softmax to control randomness:

```
P(xᵢ) = exp(zᵢ / T) / Σⱼ exp(zⱼ / T)
```

| Temperature T | Effect                                |
|---------------|---------------------------------------|
| T → 0         | Approaches greedy (sharp distribution)|
| T = 1         | Original distribution (no change)     |
| T > 1         | Flatter distribution (more random)    |
| T → ∞         | Uniform distribution                  |

### 5.3 Top-k Sampling (Fan et al., 2018)

Only sample from the **k most probable** tokens:

```
1. Sort tokens by probability
2. Keep top k tokens
3. Zero out all others
4. Renormalize and sample
```

### 5.4 Top-p (Nucleus) Sampling (Holtzman et al., 2020)

Sample from the smallest set of tokens whose cumulative probability ≥ p:

```
1. Sort tokens by probability (descending)
2. Compute cumulative sum
3. Find smallest set S where Σ P(xᵢ) ≥ p
4. Zero out tokens outside S
5. Renormalize and sample
```

### Comparison Diagram

```
  Full Distribution (vocab_size = 10):
  Token:    A     B     C     D     E     F     G     H     I     J
  Prob:   0.35  0.25  0.15  0.10  0.05  0.04  0.03  0.01  0.01  0.01

  Greedy:       [A] ← always picks highest
  Top-k (k=3): [A, B, C] ← sample from top 3
  Top-p (p=0.9):[A, B, C, D, E] ← cumulative = 0.90 ≥ 0.9
  Temperature:   All tokens, but rescaled probabilities
```

---

## 6. Worked Example: Sampling Strategies

Given logits for 5 tokens after a prompt:

```
Tokens:  ["the", "a", "one", "some", "my"]
Logits:  [2.0,   1.5,  0.8,   0.3,   -0.1]
```

### Step 1: Softmax (T = 1.0)

```
exp(2.0) = 7.389    → P = 7.389 / 14.163 = 0.522
exp(1.5) = 4.482    → P = 4.482 / 14.163 = 0.316
exp(0.8) = 2.226    → P = 2.226 / 14.163 = 0.157
exp(0.3) = 1.350    → P = 1.350 / 14.163 = 0.095
exp(-0.1)= 0.905    → P = 0.905 / 14.163 = 0.064
                         Sum of exp = 14.163 (should be ~16.35)

Corrected:
exp(2.0) = 7.389    → P = 7.389 / 16.352 = 0.452
exp(1.5) = 4.482    → P = 4.482 / 16.352 = 0.274
exp(0.8) = 2.226    → P = 2.226 / 16.352 = 0.136
exp(0.3) = 1.350    → P = 1.350 / 16.352 = 0.083
exp(-0.1)= 0.905    → P = 0.905 / 16.352 = 0.055
```

### Step 2: Apply Different Strategies

```
Greedy:       → "the"  (highest P = 0.452)

Top-k (k=3): → sample from {"the": 0.527, "a": 0.319, "one": 0.154}
              (renormalized from top 3)

Top-p (p=0.8):→ sample from {"the": 0.524, "a": 0.317, "one": 0.158}
              (cumulative: 0.452 + 0.274 + 0.136 = 0.862 ≥ 0.8)

Temperature (T=0.5):
  Logits/0.5 = [4.0, 3.0, 1.6, 0.6, -0.2]
  → Sharper distribution: P ≈ [0.649, 0.238, 0.059, 0.022, 0.010]
  → More deterministic, "the" even more likely
```

---

## 7. Implementation: Text Generation with Sampling Strategies

```python
import torch
import torch.nn.functional as F


def generate(model, prompt_ids, max_new_tokens=50, strategy="greedy",
             temperature=1.0, top_k=50, top_p=0.9):
    """
    Autoregressive text generation with multiple sampling strategies.

    Args:
        model: GPT-style language model
        prompt_ids: tensor of shape (1, seq_len) with prompt token IDs
        max_new_tokens: number of tokens to generate
        strategy: "greedy", "temperature", "top_k", "top_p"
        temperature: temperature for scaling (used in all sampling modes)
        top_k: k for top-k sampling
        top_p: p for nucleus sampling
    """
    model.eval()
    generated = prompt_ids.clone()

    with torch.no_grad():
        for _ in range(max_new_tokens):
            logits = model(generated)                 # (1, seq_len, vocab)
            next_logits = logits[:, -1, :]            # (1, vocab) — last position

            # Apply temperature
            if temperature != 1.0:
                next_logits = next_logits / temperature

            if strategy == "greedy":
                next_token = next_logits.argmax(dim=-1, keepdim=True)

            elif strategy == "top_k":
                next_token = top_k_sample(next_logits, k=top_k)

            elif strategy == "top_p":
                next_token = top_p_sample(next_logits, p=top_p)

            else:  # pure temperature sampling
                probs = F.softmax(next_logits, dim=-1)
                next_token = torch.multinomial(probs, num_samples=1)

            generated = torch.cat([generated, next_token], dim=1)

            # Stop at EOS token (assume EOS = 2)
            if next_token.item() == 2:
                break

    return generated


def top_k_sample(logits, k=50):
    """Sample from the top-k most probable tokens."""
    values, indices = torch.topk(logits, k, dim=-1)
    # Set everything outside top-k to -inf
    logits_filtered = torch.full_like(logits, float('-inf'))
    logits_filtered.scatter_(1, indices, values)

    probs = F.softmax(logits_filtered, dim=-1)
    return torch.multinomial(probs, num_samples=1)


def top_p_sample(logits, p=0.9):
    """Nucleus sampling: sample from smallest set with cumulative prob >= p."""
    sorted_logits, sorted_indices = torch.sort(logits, descending=True, dim=-1)
    cumulative_probs = torch.cumsum(F.softmax(sorted_logits, dim=-1), dim=-1)

    # Remove tokens with cumulative probability above the threshold
    # Shift right so the first token above threshold is kept
    sorted_mask = cumulative_probs - F.softmax(sorted_logits, dim=-1) >= p
    sorted_logits[sorted_mask] = float('-inf')

    probs = F.softmax(sorted_logits, dim=-1)
    sorted_token = torch.multinomial(probs, num_samples=1)

    # Map back to original indices
    next_token = sorted_indices.gather(1, sorted_token)
    return next_token


# --- Example usage ---
# Assuming `model` is a trained GPT and `tokenizer` is available:
#
# prompt = "The future of artificial intelligence"
# prompt_ids = tokenizer.encode(prompt, return_tensors="pt")
#
# # Greedy
# out = generate(model, prompt_ids, strategy="greedy")
# print("Greedy:", tokenizer.decode(out[0]))
#
# # Top-k sampling
# out = generate(model, prompt_ids, strategy="top_k", top_k=40, temperature=0.8)
# print("Top-k:", tokenizer.decode(out[0]))
#
# # Nucleus sampling
# out = generate(model, prompt_ids, strategy="top_p", top_p=0.92, temperature=0.9)
# print("Top-p:", tokenizer.decode(out[0]))
```

---

## 8. Beam Search (Bonus Strategy)

Beam search maintains **B** candidate sequences at each step:

```
  Beam width B = 2

  Step 0:  "The"
            ├── "The cat"     (score: -1.2)
            └── "The dog"     (score: -1.5)

  Step 1:  "The cat"
            ├── "The cat sat"   (score: -2.1)  ✓ keep
            └── "The cat ran"   (score: -2.8)
           "The dog"
            ├── "The dog barked" (score: -2.4) ✓ keep
            └── "The dog sat"    (score: -2.9)

  Step 2:  Keep top-B = 2 sequences, continue...
```

- **Pro**: More thorough search than greedy
- **Con**: Can produce generic/bland text; expensive for large B

---

## 9. Real-World Applications

| Application               | Strategy Used         | Why                                    |
|----------------------------|-----------------------|----------------------------------------|
| **Machine Translation**    | Beam search (B=4-5)  | Quality over diversity                 |
| **Creative Writing**       | Top-p (p=0.9-0.95)   | Natural diversity, avoids nonsense     |
| **Code Generation**        | Temperature 0.2-0.4  | Low randomness for correctness         |
| **Chatbots**               | Top-p + temperature  | Balance coherence and engagement       |
| **Summarization**          | Beam search or greedy | Factual accuracy important             |
| **Data Augmentation**      | High temperature      | Maximize diversity                     |

---

## 10. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| Chain rule                 | P(x₁..xₙ) = ∏ P(xₜ|x<ₜ) — exact decomposition           |
| Training objective         | Minimize cross-entropy: -1/N Σ log P(xₜ|x<ₜ)             |
| Perplexity                 | exp(cross-entropy loss) — lower is better                  |
| Teacher forcing            | Use ground-truth tokens during training                    |
| Exposure bias              | Train-test mismatch from teacher forcing                   |
| Greedy decoding            | argmax — fast but repetitive                               |
| Temperature                | Scale logits by T; low T → sharp, high T → flat           |
| Top-k sampling             | Sample from k most probable tokens                         |
| Top-p (nucleus)            | Sample from smallest set with cumulative prob ≥ p          |
| Beam search                | Maintain B candidates; good for translation                |

---

## 11. Revision Questions

1. **Derive the chain rule factorization** for the sequence "I love AI". Write out each conditional probability term.

2. **Why is teacher forcing used during training?** What is the exposure bias problem it introduces?

3. **Calculate the perplexity** of a model with average cross-entropy loss of 3.5 nats. Is this a good language model?

4. **Given logits [3.0, 1.0, 0.5, -1.0, -2.0] and top-p = 0.85**, which tokens would be included in the nucleus? Show your work.

5. **Compare greedy decoding with nucleus sampling.** When would you prefer each approach?

6. **What happens to the softmax distribution as temperature approaches 0?** What about as it approaches infinity?

---

> **Navigation:** [← GPT Architecture](01-gpt-architecture.md) | [GPT-2 and GPT-3 →](03-gpt2-and-gpt3.md)
