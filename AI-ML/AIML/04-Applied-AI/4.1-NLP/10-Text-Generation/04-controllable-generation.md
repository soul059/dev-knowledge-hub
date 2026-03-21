# Controllable Generation

## Overview

Controllable text generation aims to steer language models to produce text with desired attributes — specific topics, styles, sentiments, or safety constraints — while maintaining fluency and coherence. This is crucial for deploying LMs in production where outputs must follow guidelines.

---

## Why Controllability Matters

```
Uncontrolled LM:
  Prompt: "Write a product review"
  Output: Could be positive, negative, toxic, off-topic, or anything

Controlled LM:
  Prompt: "Write a product review"
  Control: sentiment=positive, topic=electronics, tone=professional
  Output: "The new laptop delivers exceptional performance with its 
           fast processor and stunning display. Highly recommended."
```

---

## Control Methods

```
┌─────────────────────────────────────────────┐
│         Controllable Generation Methods      │
├─────────────────────────────────────────────┤
│                                             │
│  Training-time:                             │
│    ├── Conditional training (control codes) │
│    ├── Fine-tuning on curated data          │
│    └── RLHF (Reinforcement Learning        │
│          from Human Feedback)               │
│                                             │
│  Inference-time:                            │
│    ├── Prompt engineering                   │
│    ├── PPLM (Plug and Play LM)             │
│    ├── Classifier-guided decoding           │
│    └── Constrained decoding                 │
│                                             │
│  Post-hoc:                                  │
│    ├── Re-ranking generated candidates      │
│    └── Filtering / safety classifiers       │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Prompt Engineering

```python
from transformers import pipeline

generator = pipeline("text-generation", model="gpt2-medium")

# Style control through prompting
prompts = {
    "formal": "In a formal academic tone, explain quantum computing: ",
    "casual": "Hey, so like quantum computing is basically: ",
    "poetic": "In verse and rhyme, the quantum realm reveals: ",
}

for style, prompt in prompts.items():
    output = generator(prompt, max_length=80, do_sample=True, 
                        temperature=0.7, top_p=0.9)
    print(f"[{style}]: {output[0]['generated_text'][:150]}\n")
```

---

## CTRL: Conditional Transformer

```
Salesforce CTRL uses control codes prepended to input:

  Control code + prompt → generated text

  Examples:
    "Reviews Rating:5.0"  + "This restaurant"  → positive review
    "Reviews Rating:1.0"  + "This restaurant"  → negative review
    "Wikipedia"           + "Machine learning"  → encyclopedia article
    "Links Reddit"        + "TIL that"          → Reddit-style post
    "Horror"              + "The door opened"   → horror story

  Architecture: 1.6B parameter transformer
  48 control codes trained on 140GB of text
```

---

## RLHF (Reinforcement Learning from Human Feedback)

```
The method behind ChatGPT, Claude, and other aligned LMs:

Step 1: Supervised Fine-Tuning (SFT)
  Fine-tune base LM on human-written demonstrations
  LM_SFT = fine_tune(LM_base, human_demonstrations)

Step 2: Reward Model Training
  Humans rank multiple LM outputs for same prompt
  Train reward model: R(prompt, response) → scalar score
  
  Human preference: response_A > response_B
  Loss: -log σ(R(A) - R(B))

Step 3: RL Optimization (PPO)
  Optimize LM to maximize reward while staying close to SFT model
  
  Objective = E[R(prompt, response)] - β × KL(π_RL || π_SFT)
  
  β controls how much the RL model can deviate from SFT

Pipeline:
  Base LM → SFT → Reward Model → PPO Training → Aligned LM
  (GPT-3)  (demos)  (rankings)    (RL)          (ChatGPT)
```

---

## Classifier-Guided Decoding (PPLM)

```
Plug and Play Language Models:

  At each generation step:
    1. Get LM's next-token distribution: P_LM(w | context)
    2. Run attribute classifier: P_attr(attribute | context + w)
    3. Combine: P_final(w) ∝ P_LM(w) × P_attr(attribute | w)^α

  Example (positive sentiment):
    Token     P_LM    P_sentiment   Combined (α=2)
    "great"   0.05    0.95          0.05 × 0.90 = 0.045
    "bad"     0.08    0.10          0.08 × 0.01 = 0.001
    "the"     0.15    0.50          0.15 × 0.25 = 0.038
    → "great" wins despite lower LM probability!

  No need to retrain the LM — just plug in classifiers
```

---

## Constrained Decoding

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import torch

model = GPT2LMHeadModel.from_pretrained("gpt2")
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

# Force specific words to appear
force_words = ["artificial", "intelligence"]
force_words_ids = [tokenizer.encode(w, add_special_tokens=False) 
                   for w in force_words]

input_ids = tokenizer.encode("The future of technology", return_tensors="pt")
output = model.generate(
    input_ids,
    max_length=50,
    num_beams=5,
    force_words_ids=force_words_ids,  # must include these words
)
print(tokenizer.decode(output[0], skip_special_tokens=True))

# Ban specific words (e.g., toxic content)
bad_words = ["stupid", "hate", "ugly"]
bad_words_ids = [tokenizer.encode(w, add_special_tokens=False) 
                 for w in bad_words]

output = model.generate(
    input_ids,
    max_length=50,
    do_sample=True,
    bad_words_ids=bad_words_ids,  # never generate these
)
```

---

## Control Methods Comparison

| Method | Training Required | Flexibility | Quality | Use Case |
|--------|:---:|:---:|:---:|---------|
| Prompt engineering | ✗ | High | Medium | Quick prototyping |
| Control codes (CTRL) | ✓ | Fixed attributes | High | Pre-defined controls |
| Fine-tuning | ✓ | Domain-specific | High | Specialized domains |
| RLHF | ✓ (expensive) | Human-aligned | Very high | Safety, helpfulness |
| PPLM | Classifier only | Flexible | Good | Style/sentiment |
| Constrained decoding | ✗ | Token-level | Medium | Must-include/exclude |

---

## Revision Questions

1. **What is the difference between training-time and inference-time control?**
2. **How does RLHF align language models with human preferences?**
3. **What are control codes and how does CTRL use them?**
4. **How does PPLM combine a language model with an attribute classifier?**
5. **When would you use constrained decoding vs prompt engineering?**

---

[Previous: 03-beam-search.md](03-beam-search.md) | [Next: 05-dialogue-systems.md](05-dialogue-systems.md)
