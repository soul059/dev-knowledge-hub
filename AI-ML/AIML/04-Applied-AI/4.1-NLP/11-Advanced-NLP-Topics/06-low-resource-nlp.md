# Low-Resource NLP

## Overview

Low-resource NLP deals with building NLP systems for languages, domains, or tasks where labeled data is scarce or nonexistent. While English has enormous datasets and pre-trained models, the vast majority of the world's 7,000+ languages have minimal digital resources. Low-resource techniques are critical for language equity and real-world deployment.

---

## The Data Scarcity Spectrum

```
Resource Level:
  High-resource:     English, Chinese, Spanish
                     Millions of labeled examples
                     Large pre-trained models
                     
  Medium-resource:   Hindi, Arabic, Turkish, Korean
                     Thousands of labeled examples
                     Some pre-trained models
                     
  Low-resource:      Swahili, Yoruba, Welsh, Latvian
                     Hundreds of examples or fewer
                     Limited pre-trained models
                     
  Extremely low:     Many indigenous languages
                     Near-zero digital presence
                     No NLP tools exist

~95% of languages have < 1% of available NLP resources
```

---

## Strategies for Low-Resource NLP

```
┌─────────────────────────────────────────────┐
│          Low-Resource NLP Strategies         │
├─────────────────────────────────────────────┤
│                                             │
│  Transfer Learning:                         │
│    ├── Cross-lingual transfer (from English)│
│    ├── Multilingual pre-training (mBERT)    │
│    └── Domain adaptation                    │
│                                             │
│  Data Augmentation:                         │
│    ├── Back-translation                     │
│    ├── Synonym replacement                  │
│    ├── LLM-generated data                   │
│    └── Paraphrase generation                │
│                                             │
│  Few-Shot / Zero-Shot:                      │
│    ├── In-context learning (GPT)            │
│    ├── Prompt-based learning                │
│    └── Meta-learning (MAML)                 │
│                                             │
│  Active Learning:                           │
│    └── Intelligently select what to label   │
│                                             │
│  Community-driven:                          │
│    ├── Masakhane (African NLP)              │
│    └── AmericasNLP                          │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Cross-Lingual Transfer

```python
from transformers import pipeline

# Train model on English, use for other languages (zero-shot)
classifier = pipeline("text-classification", 
                       model="joeddav/xlm-roberta-large-xnli")

# Works on languages it was NEVER fine-tuned on
test_cases = [
    ("This product is excellent!", "English"),
    ("Ce produit est excellent!", "French"),
    ("Dieses Produkt ist ausgezeichnet!", "German"),
    ("Hii bidhaa ni bora sana!", "Swahili"),
    ("यह उत्पाद उत्कृष्ट है!", "Hindi"),
]

for text, lang in test_cases:
    result = classifier(text, candidate_labels=["positive", "negative"])
    print(f"  [{lang:8s}] {text[:40]:40s} → {result['labels'][0]}")
# All correctly identified as positive!
```

---

## Data Augmentation

```python
# Back-translation augmentation
from transformers import MarianMTModel, MarianTokenizer

def back_translate(text, src="en", pivot="de"):
    """Translate to another language and back to get paraphrases"""
    # English → German
    fwd_model = f"Helsinki-NLP/opus-mt-{src}-{pivot}"
    fwd_tok = MarianTokenizer.from_pretrained(fwd_model)
    fwd_mdl = MarianMTModel.from_pretrained(fwd_model)
    
    tokens = fwd_tok(text, return_tensors="pt", padding=True)
    translated = fwd_mdl.generate(**tokens)
    pivot_text = fwd_tok.decode(translated[0], skip_special_tokens=True)
    
    # German → English
    bwd_model = f"Helsinki-NLP/opus-mt-{pivot}-{src}"
    bwd_tok = MarianTokenizer.from_pretrained(bwd_model)
    bwd_mdl = MarianMTModel.from_pretrained(bwd_model)
    
    tokens = bwd_tok(pivot_text, return_tensors="pt", padding=True)
    back = bwd_mdl.generate(**tokens)
    return bwd_tok.decode(back[0], skip_special_tokens=True)

original = "The movie was really enjoyable and fun to watch"
augmented = back_translate(original)
print(f"Original:  {original}")
print(f"Augmented: {augmented}")
# Augmented: "The film was really enjoyable and fun to see"
```

---

## Few-Shot Learning with Prompts

```python
# Using LLM for few-shot classification in low-resource setting
prompt = """Classify the sentiment of Swahili text as positive or negative.

Examples:
"Chakula kilikuwa kitamu sana" → positive
"Huduma mbaya, singependa kurudi" → negative
"Chumba kilikuwa safi na kimepangwa vizuri" → positive

Now classify:
"Hoteli hii ni mbaya sana, hainifai" → """

# Feed to LLM → gets "negative" without ANY Swahili training data!
# The LLM leverages its multilingual knowledge + few examples
```

---

## Active Learning

```
Instead of labeling ALL data, intelligently select the MOST useful 
examples to label:

  ┌────────────────────────────────┐
  │  1. Train model on small seed  │
  │  2. Predict on unlabeled pool  │
  │  3. Select most uncertain      │  ← Uncertainty sampling
  │  4. Human labels those         │
  │  5. Add to training set        │
  │  6. Repeat until budget spent  │
  └────────────────────────────────┘

Selection strategies:
  - Uncertainty: model least confident → most informative
  - Diversity: select diverse examples → better coverage
  - Expected model change: which example would change model most?

Result: Same performance with 20-50% of labels!
```

---

## Key Initiatives

| Initiative | Focus | Languages |
|------------|-------|:-:|
| Masakhane | African NLP | 30+ African languages |
| AmericasNLP | Indigenous Americas | Quechua, Guarani, etc. |
| NLLB (Meta) | Translation | 200 languages |
| Lacuna Fund | Dataset creation | Various low-resource |
| Common Voice (Mozilla) | Speech data | 100+ languages |
| Universal Dependencies | Treebanks | 130+ languages |

---

## Summary of Approaches

| Approach | Data Needed | Effort | Effectiveness |
|----------|:-:|:-:|:-:|
| Cross-lingual transfer | Zero target-lang | Low | Good |
| Multilingual pre-training | Unlabeled text | Medium | Very good |
| Back-translation augmentation | Few examples | Low | Moderate |
| Few-shot prompting | 5-10 examples | Very low | Good |
| Active learning | Labeling budget | Medium | Very good |
| Community data collection | Community effort | High | Best long-term |

---

## Revision Questions

1. **What makes a language "low-resource" in NLP terms?**
2. **How does cross-lingual transfer work with XLM-R?**
3. **How does back-translation help with data augmentation?**
4. **What is active learning and why is it effective for low-resource settings?**
5. **What community initiatives exist for low-resource NLP?**
6. **How can few-shot prompting help with languages that have minimal training data?**

---

[Previous: 05-information-extraction.md](05-information-extraction.md) | [Back to README](../README.md)
