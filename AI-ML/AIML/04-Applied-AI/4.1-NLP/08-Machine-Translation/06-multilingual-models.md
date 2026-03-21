# Multilingual Models

## Overview

Multilingual NLP models are trained on data from many languages simultaneously, enabling cross-lingual transfer, zero-shot translation, and shared representations across language families. They address the critical problem that most NLP resources exist only for English, while there are 7,000+ languages in the world.

---

## Evolution of Multilingual NLP

```
Timeline:
  2018  │  mBERT        → Multilingual BERT (104 languages)
  2019  │  XLM           → Cross-lingual Language Model
  2020  │  XLM-R         → XLM-RoBERTa (100 languages, state-of-art)
  2020  │  mBART         → Multilingual denoising for translation
  2020  │  mT5           → Multilingual T5
  2021  │  M2M-100       → Many-to-many (100×100 translation)
  2022  │  NLLB-200      → No Language Left Behind (200 languages)
  2023  │  MADLAD-400    → 400+ languages
```

---

## mBERT (Multilingual BERT)

```
Architecture:
  Same as BERT-Base but trained on Wikipedia in 104 languages
  
  Vocab: 110K WordPiece tokens (shared across languages)
  Layers: 12 transformer layers
  Hidden: 768
  Training: MLM on concatenated Wikipedia corpora

Key finding:
  Despite NO explicit cross-lingual objective,
  mBERT learns language-agnostic representations!

  Train NER on English → Test on Spanish → Works!
  (Zero-shot cross-lingual transfer)

Why it works:
  - Shared subword vocabulary (overlapping tokens across languages)
  - Shared transformer parameters
  - Similar language structures create similar representations
```

---

## XLM-R (XLM-RoBERTa)

```
Improvements over mBERT:
  ┌──────────────────────────────────────────────────┐
  │ Feature          │ mBERT      │ XLM-R            │
  ├──────────────────┼────────────┼──────────────────┤
  │ Training data    │ Wikipedia  │ CC-100 (2.5 TB)  │
  │ Languages        │ 104        │ 100              │
  │ Vocab size       │ 110K       │ 250K             │
  │ Model size       │ 172M       │ 270M / 550M      │
  │ Tokenizer        │ WordPiece  │ SentencePiece     │
  │ Low-resource perf│ Weak       │ Strong            │
  └──────────────────┴────────────┴──────────────────┘

XLM-R excels because:
  - Larger and more diverse training data
  - Better tokenizer coverage for non-Latin scripts
  - Specifically tuned for cross-lingual tasks
```

---

## Translation Models

### NLLB (No Language Left Behind)

```
Goal: High-quality translation for 200 languages

  Architecture: Encoder-decoder transformer
  Key innovations:
    - Language-specific routing (Mixture of Experts)
    - Curriculum learning for low-resource languages
    - FLORES-200 benchmark (200-language evaluation set)
    - Distilled models: 600M → available for deployment

  Coverage: Includes many African, Southeast Asian, 
            and indigenous languages previously ignored
```

### M2M-100 (Many-to-Many)

```
  Traditional: English-centric (pivot through English)
    French → [English] → German (2 steps, error propagation)

  M2M-100: Direct translation between any pair
    French → German (1 step, better quality)

  12B parameter model, 100 languages
  4,450 language pair directions trained directly
```

---

## Cross-Lingual Transfer

```python
from transformers import pipeline

# Train on English, test on other languages (zero-shot)
classifier = pipeline("text-classification", 
                       model="joeddav/xlm-roberta-large-xnli")

# Works across languages without language-specific training!
texts = [
    "This movie is fantastic!",          # English
    "Ce film est fantastique!",          # French
    "Dieser Film ist fantastisch!",      # German
    "この映画は素晴らしいです！",           # Japanese
    "هذا الفيلم رائع",                   # Arabic
]

for text in texts:
    result = classifier(text, candidate_labels=["positive", "negative"])
    print(f"  {text[:30]:30s} → {result['labels'][0]}")
```

---

## Multilingual Translation with NLLB

```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

model_name = "facebook/nllb-200-distilled-600M"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name)

def translate(text, src_lang, tgt_lang):
    tokenizer.src_lang = src_lang
    inputs = tokenizer(text, return_tensors="pt")
    translated = model.generate(
        **inputs,
        forced_bos_token_id=tokenizer.lang_code_to_id[tgt_lang]
    )
    return tokenizer.decode(translated[0], skip_special_tokens=True)

# English to Swahili
print(translate("Machine learning is amazing", "eng_Latn", "swh_Latn"))

# French to Hindi
print(translate("Bonjour le monde", "fra_Latn", "hin_Deva"))
```

---

## Challenges in Multilingual NLP

| Challenge | Description |
|-----------|-------------|
| **Curse of multilinguality** | Adding more languages dilutes per-language capacity |
| **Script diversity** | Latin, Cyrillic, Devanagari, CJK, Arabic, etc. |
| **Tokenizer bias** | Low-resource languages get over-fragmented tokens |
| **Data imbalance** | English has 100× more data than most languages |
| **Evaluation** | Few benchmarks for low-resource languages |
| **Cultural nuance** | Sentiment, idioms vary dramatically across cultures |

---

## Summary Table

| Model | Languages | Task | Parameters | Key Feature |
|-------|:-:|--------|:-:|-------------|
| mBERT | 104 | Understanding | 172M | First multilingual LM |
| XLM-R | 100 | Understanding | 270–550M | Best cross-lingual transfer |
| mBART | 25–50 | Generation | 610M | Multilingual denoising |
| mT5 | 101 | Seq2Seq | 300M–13B | Multilingual T5 |
| M2M-100 | 100 | Translation | 12B | Direct many-to-many |
| NLLB-200 | 200 | Translation | 600M–54B | Low-resource focus |

---

## Revision Questions

1. **How does mBERT achieve cross-lingual transfer without explicit alignment?**
2. **What advantages does XLM-R have over mBERT?**
3. **What is the "curse of multilinguality" and how do models mitigate it?**
4. **How does M2M-100 differ from English-centric translation?**
5. **Why is NLLB important for language equity?**
6. **What is zero-shot cross-lingual transfer?**

---

[Previous: 05-bleu-score.md](05-bleu-score.md) | [Back to README](../README.md)
