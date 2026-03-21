# Dialogue Systems

## Overview

Dialogue systems (conversational AI) enable machines to converse with humans in natural language. They range from simple rule-based chatbots to sophisticated neural models like ChatGPT. Understanding dialogue requires managing context, intent, personality, and multi-turn coherence.

---

## Types of Dialogue Systems

```
Dialogue Systems
├── Task-Oriented (Goal-driven)
│   ├── Slot-filling systems (booking flights, restaurants)
│   ├── Pipeline: NLU → DM → NLG
│   └── Examples: Siri, Alexa, Google Assistant
│
├── Open-Domain (Chit-chat)
│   ├── No specific goal, general conversation
│   ├── End-to-end neural models
│   └── Examples: ChatGPT, Claude, Meena
│
└── Hybrid
    ├── Combines both approaches
    └── Task when needed, chat otherwise
```

---

## Task-Oriented Pipeline

```
User: "Book a table for 2 at an Italian restaurant tonight"
         │
         ▼
┌─────────────────────┐
│   NLU (Understanding)│
│                     │
│  Intent: book_table │
│  Slots:             │
│    party_size: 2    │
│    cuisine: Italian │
│    date: tonight    │
│    restaurant: ?    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Dialogue Manager    │
│                     │
│  State tracking:     │
│    Missing: restaurant│
│  Action: ask_restaurant│
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  NLG (Generation)    │
│                     │
│  "Which Italian      │
│   restaurant would   │
│   you prefer?"       │
└─────────────────────┘
```

---

## Dialogue State Tracking (DST)

```
Tracks what the user wants across multiple turns:

Turn 1: "I want a cheap restaurant"
  State: {price=cheap}

Turn 2: "Make it Italian"
  State: {price=cheap, cuisine=Italian}  ← accumulated

Turn 3: "Actually, make it moderate price"
  State: {price=moderate, cuisine=Italian}  ← updated

Turn 4: "In the city center"
  State: {price=moderate, cuisine=Italian, area=center}

The DST must handle:
  - Slot accumulation (new info)
  - Slot modification (corrections)
  - Coreference ("that one", "there")
  - Implicit confirmation ("So moderate Italian in center?")
```

---

## Neural Dialogue Models

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

# DialoGPT — trained specifically for dialogue
model_name = "microsoft/DialoGPT-medium"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# Multi-turn conversation
conversation_history = []

def chat(user_input):
    # Encode user input
    new_input = tokenizer.encode(user_input + tokenizer.eos_token, 
                                  return_tensors="pt")
    
    # Append to history
    import torch
    if conversation_history:
        bot_input = torch.cat([conversation_history[-1], new_input], dim=-1)
    else:
        bot_input = new_input
    
    # Generate response
    output = model.generate(
        bot_input, max_length=1000, do_sample=True,
        top_p=0.92, temperature=0.7,
        pad_token_id=tokenizer.eos_token_id
    )
    
    response = tokenizer.decode(
        output[:, bot_input.shape[-1]:][0], 
        skip_special_tokens=True
    )
    
    conversation_history.append(output)
    return response

# Example conversation
print(chat("Hi! How are you?"))
print(chat("What do you think about AI?"))
print(chat("That's interesting, tell me more"))
```

---

## Challenges in Dialogue

| Challenge | Description | Solution |
|-----------|-------------|----------|
| **Consistency** | Model contradicts itself across turns | Persona conditioning, memory |
| **Hallucination** | Makes up false facts | Retrieval-augmented generation |
| **Safety** | Toxic or harmful responses | RLHF, content filtering |
| **Engagement** | Boring "I don't know" responses | Diverse decoding, personality |
| **Context length** | Forgetting early conversation | Summarization, memory networks |
| **Grounding** | Disconnected from real world | Knowledge bases, tool use |

---

## Modern Dialogue Architecture

```
Modern chat models (ChatGPT-style):

  ┌──────────────────────────────┐
  │     Base Language Model       │
  │     (GPT-4, LLaMA, etc.)     │
  └──────────────┬───────────────┘
                 │
  ┌──────────────┴───────────────┐
  │  Supervised Fine-Tuning (SFT) │
  │  on human dialogue examples   │
  └──────────────┬───────────────┘
                 │
  ┌──────────────┴───────────────┐
  │  RLHF / DPO / RLAIF          │
  │  Align with human preferences │
  └──────────────┬───────────────┘
                 │
  ┌──────────────┴───────────────┐
  │  System Prompt + Guardrails   │
  │  Control behavior at runtime  │
  └──────────────────────────────┘

Input format:
  [System] You are a helpful assistant...
  [User] What is machine learning?
  [Assistant] Machine learning is...
  [User] Can you give an example?
  [Assistant] ...
```

---

## Evaluation Metrics

| Metric | Type | Measures |
|--------|------|---------|
| Perplexity | Automatic | Fluency (lower = better) |
| BLEU/ROUGE | Automatic | Overlap with reference |
| Distinct-n | Automatic | Diversity (unique n-grams) |
| Human rating | Human | Overall quality (1-5 scale) |
| Sensibleness | Human | Makes sense in context |
| Specificity | Human | Not generic/boring |
| Safety | Human/Auto | No harmful content |

---

## Revision Questions

1. **What is the difference between task-oriented and open-domain dialogue?**
2. **What are the three components of a task-oriented dialogue pipeline?**
3. **What is dialogue state tracking and why is it challenging?**
4. **How do modern chat models (ChatGPT) differ from earlier chatbots?**
5. **What are the main challenges in building reliable dialogue systems?**

---

[Previous: 04-controllable-generation.md](04-controllable-generation.md) | [Next: 06-summarization.md](06-summarization.md)
