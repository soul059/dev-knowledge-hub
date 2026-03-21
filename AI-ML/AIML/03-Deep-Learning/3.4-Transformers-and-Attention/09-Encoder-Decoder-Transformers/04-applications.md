# Practical Encoder-Decoder Applications

[← Seq2Seq Tasks](03-seq2seq-tasks.md) | [Vision Transformer →](../10-Modern-Transformer-Variants/01-vision-transformer.md)

---

## Overview

Encoder-decoder transformers like T5 and BART power some of the most impactful NLP applications in production today. This chapter bridges theory and practice by walking through complete, end-to-end application pipelines for the four most common tasks: abstractive summarization, machine translation, generative question answering, and paraphrase generation. Each section includes full working code using HuggingFace's `transformers` library, along with practical deployment considerations such as latency optimization, model size tradeoffs, and quality evaluation. Whether you are building a prototype or deploying to millions of users, these patterns form the foundation of modern text generation systems.

---

## 1. Summarization Pipeline

### 1.1 Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│               SUMMARIZATION PIPELINE                             │
│                                                                  │
│  ┌───────────┐    ┌────────────┐    ┌───────────┐    ┌────────┐ │
│  │ Raw       │───►│ Preprocess │───►│ Encoder-  │───►│ Post-  │ │
│  │ Document  │    │ & Tokenize │    │ Decoder   │    │ process│ │
│  └───────────┘    └────────────┘    │ Model     │    └───┬────┘ │
│                                     └───────────┘        │      │
│                                                          ▼      │
│  Preprocessing:         Model:            Postprocessing:       │
│  - Clean HTML/noise     - BART-CNN        - Detokenize          │
│  - Truncate to          - T5              - Remove artifacts    │
│    max_length           - Pegasus         - Length check         │
│  - Tokenize             - Beam search     - Quality filter      │
│                           decoding                              │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 Complete Summarization Pipeline

```python
from transformers import pipeline, BartForConditionalGeneration, BartTokenizer
import textwrap

class SummarizationPipeline:
    """End-to-end abstractive summarization pipeline."""

    def __init__(self, model_name="facebook/bart-large-cnn", device=-1):
        self.tokenizer = BartTokenizer.from_pretrained(model_name)
        self.model = BartForConditionalGeneration.from_pretrained(model_name)
        self.device = device
        if device >= 0:
            import torch
            self.model = self.model.to(f"cuda:{device}")

    def preprocess(self, text, max_length=1024):
        """Clean and truncate input text."""
        text = text.strip()
        text = ' '.join(text.split())  # normalize whitespace
        inputs = self.tokenizer(
            text,
            return_tensors="pt",
            max_length=max_length,
            truncation=True,
        )
        if self.device >= 0:
            inputs = {k: v.to(f"cuda:{self.device}") for k, v in inputs.items()}
        return inputs

    def generate_summary(self, inputs, min_length=30, max_length=150,
                         num_beams=4, length_penalty=2.0):
        """Generate summary using beam search."""
        summary_ids = self.model.generate(
            inputs["input_ids"],
            attention_mask=inputs["attention_mask"],
            max_length=max_length,
            min_length=min_length,
            num_beams=num_beams,
            length_penalty=length_penalty,
            no_repeat_ngram_size=3,
            early_stopping=True,
        )
        return summary_ids

    def postprocess(self, summary_ids):
        """Decode and clean the generated summary."""
        summary = self.tokenizer.decode(summary_ids[0], skip_special_tokens=True)
        summary = summary.strip()
        if summary and not summary.endswith('.'):
            last_period = summary.rfind('.')
            if last_period > len(summary) * 0.5:
                summary = summary[:last_period + 1]
        return summary

    def summarize(self, text, **kwargs):
        """Full pipeline: preprocess → generate → postprocess."""
        inputs = self.preprocess(text)
        summary_ids = self.generate_summary(inputs, **kwargs)
        return self.postprocess(summary_ids)


# Usage
pipe = SummarizationPipeline()

article = """
The International Space Station (ISS) has been continuously occupied since
November 2000, serving as a microgravity and space environment research
laboratory. Astronauts conduct experiments in biology, physics, astronomy,
and other fields. The station orbits Earth at approximately 420 kilometers
altitude, completing one orbit every 90 minutes. It has been visited by
astronauts from 19 different countries and has hosted over 3,000 scientific
experiments. Recently, NASA announced plans to decommission the ISS by 2030,
with commercial space stations expected to take over as orbital research
platforms.
"""

summary = pipe.summarize(article)
print("Summary:", textwrap.fill(summary, width=80))
```

### 1.3 Summarization Quality Metrics

```python
from rouge_score import rouge_scorer

def evaluate_summary(reference, candidate):
    """Evaluate summary quality using ROUGE metrics."""
    scorer = rouge_scorer.RougeScorer(
        ['rouge1', 'rouge2', 'rougeL'], use_stemmer=True
    )
    scores = scorer.score(reference, candidate)
    results = {}
    for metric, score in scores.items():
        results[metric] = {
            'precision': round(score.precision, 4),
            'recall': round(score.recall, 4),
            'f1': round(score.fmeasure, 4),
        }
    return results

# Example evaluation
reference = "The ISS will be decommissioned by 2030, replaced by commercial stations."
candidate = "NASA plans to retire the ISS by 2030 as commercial platforms emerge."
scores = evaluate_summary(reference, candidate)

for metric, values in scores.items():
    print(f"{metric}: P={values['precision']} R={values['recall']} F1={values['f1']}")
```

---

## 2. Translation System

### 2.1 Translation with T5

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

class TranslationPipeline:
    """Multi-language translation pipeline using T5."""

    SUPPORTED_PAIRS = {
        ("en", "de"): "translate English to German",
        ("en", "fr"): "translate English to French",
        ("en", "ro"): "translate English to Romanian",
    }

    def __init__(self, model_name="t5-base"):
        self.tokenizer = T5Tokenizer.from_pretrained(model_name)
        self.model = T5ForConditionalGeneration.from_pretrained(model_name)
        self.model.eval()

    def translate(self, text, src_lang="en", tgt_lang="de",
                  num_beams=4, max_length=512):
        pair = (src_lang, tgt_lang)
        if pair not in self.SUPPORTED_PAIRS:
            raise ValueError(f"Unsupported pair: {pair}")
        prefix = self.SUPPORTED_PAIRS[pair]
        inputs = self.tokenizer(
            f"{prefix}: {text}", return_tensors="pt",
            max_length=max_length, truncation=True
        )
        outputs = self.model.generate(
            inputs.input_ids, max_length=max_length,
            num_beams=num_beams, length_penalty=0.6, early_stopping=True,
        )
        return self.tokenizer.decode(outputs[0], skip_special_tokens=True)


# Usage
translator = TranslationPipeline("t5-base")
print(translator.translate("The weather is beautiful today.", "en", "de"))
print(translator.translate("Machine learning is powerful.", "en", "fr"))
```

### 2.2 Translation Quality Assessment

```python
from nltk.translate.bleu_score import corpus_bleu, SmoothingFunction

refs = [["Das Wetter ist heute schoen".split()]]
cands = ["Das Wetter ist schoen heute".split()]
bleu = corpus_bleu(refs, cands, smoothing_function=SmoothingFunction().method1)
print(f"Corpus BLEU: {bleu:.4f}")
```

---

## 3. Generative Question Answering

### 3.1 Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│           GENERATIVE QUESTION ANSWERING                          │
│                                                                  │
│  ┌───────────────────┐    ┌───────────────┐    ┌──────────────┐ │
│  │ Question +        │───►│ Encoder-      │───►│ Generated    │ │
│  │ Context           │    │ Decoder Model │    │ Answer       │ │
│  └───────────────────┘    └───────────────┘    └──────────────┘ │
│                                                                  │
│  Extractive QA:    Answer is a span from context                 │
│  Generative QA:    Answer is generated (may not be in context)   │
│                                                                  │
│  Generative QA advantages:                                       │
│  - Can synthesize information from multiple parts of context     │
│  - Can provide explanations, not just answer spans               │
│  - Handles "why" and "how" questions better                      │
└──────────────────────────────────────────────────────────────────┘
```

### 3.2 Complete QA Pipeline

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

class GenerativeQAPipeline:
    """Generative question answering with T5."""

    def __init__(self, model_name="t5-base"):
        self.tokenizer = T5Tokenizer.from_pretrained(model_name)
        self.model = T5ForConditionalGeneration.from_pretrained(model_name)
        self.model.eval()

    def answer(self, question, context, max_answer_length=64):
        """Generate an answer given a question and context."""
        input_text = f"question: {question} context: {context}"

        inputs = self.tokenizer(
            input_text, return_tensors="pt",
            max_length=512, truncation=True
        )

        outputs = self.model.generate(
            inputs.input_ids,
            max_length=max_answer_length,
            num_beams=4,
            early_stopping=True,
        )
        return self.tokenizer.decode(outputs[0], skip_special_tokens=True)

    def answer_multiple(self, questions, context):
        """Answer multiple questions about the same context."""
        results = {}
        for q in questions:
            results[q] = self.answer(q, context)
        return results


# Usage
qa = GenerativeQAPipeline("t5-base")

context = """
The Transformer architecture was introduced in 2017 by Vaswani et al.
in the paper 'Attention Is All You Need'. It replaced recurrent neural
networks with self-attention mechanisms, enabling parallel processing
of sequences. The original model had 6 encoder and 6 decoder layers
with 512-dimensional hidden states and 8 attention heads.
"""

questions = [
    "Who introduced the Transformer?",
    "When was the Transformer introduced?",
    "How many layers did the original Transformer have?",
    "What did the Transformer replace?",
]

answers = qa.answer_multiple(questions, context)
for q, a in answers.items():
    print(f"Q: {q}")
    print(f"A: {a}\n")
```

---

## 4. Paraphrase Generation

### 4.1 Task Overview

```
Input:  "The cat sat on the mat."
Output: "A feline was resting on the rug."

Key applications:
  - Data augmentation for training classifiers
  - Plagiarism detection (generate paraphrases, check similarity)
  - Query expansion in search engines
  - Simplification for accessibility
```

### 4.2 Paraphrase Pipeline

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

class ParaphrasePipeline:
    """Generate paraphrases using T5 fine-tuned for paraphrasing."""

    def __init__(self, model_name="t5-base"):
        self.tokenizer = T5Tokenizer.from_pretrained(model_name)
        self.model = T5ForConditionalGeneration.from_pretrained(model_name)
        self.model.eval()

    def paraphrase(self, text, num_return=3, max_length=128):
        """Generate multiple paraphrases of input text."""
        input_text = f"paraphrase: {text}"

        inputs = self.tokenizer(
            input_text, return_tensors="pt",
            max_length=max_length, truncation=True
        )

        outputs = self.model.generate(
            inputs.input_ids,
            max_length=max_length,
            num_beams=num_return * 2,
            num_return_sequences=num_return,
            no_repeat_ngram_size=2,
            temperature=1.5,
            early_stopping=True,
        )

        paraphrases = []
        for output in outputs:
            decoded = self.tokenizer.decode(output, skip_special_tokens=True)
            if decoded.strip() and decoded.strip() != text.strip():
                paraphrases.append(decoded.strip())
        return paraphrases


# Usage
para = ParaphrasePipeline("t5-base")
original = "Machine learning is a subset of artificial intelligence."
results = para.paraphrase(original, num_return=3)

print(f"Original: {original}")
for i, p in enumerate(results, 1):
    print(f"  Paraphrase {i}: {p}")
```

---

## 5. Deployment Considerations

### 5.1 Optimization Techniques

| Technique               | Speedup  | Quality Loss | Implementation               |
|--------------------------|----------|--------------|------------------------------|
| Model distillation       | 2-4×     | Small        | DistilBART, T5-Small         |
| Quantization (INT8)      | 1.5-3×   | Minimal      | `torch.quantization`         |
| ONNX Runtime             | 1.5-2×   | None         | `optimum` library            |
| Shorter max_length       | Linear   | Variable     | Tune per task                |
| Reduce beam width        | Linear   | Small        | beams: 4 → 2                |
| Caching (KV cache)       | Significant | None      | Built into HuggingFace       |
| Batch inference          | N×       | None         | Process multiple inputs      |

### 5.2 Using HuggingFace Pipeline API (Quick Start)

```python
from transformers import pipeline

# Summarization
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
result = summarizer(
    "The Amazon rainforest produces 20% of the world's oxygen. "
    "It spans nine countries and is home to 10% of all species.",
    max_length=40, min_length=10
)
print("Summary:", result[0]["summary_text"])

# Translation
translator = pipeline("translation_en_to_de", model="t5-base")
result = translator("Machine learning is changing the world.")
print("Translation:", result[0]["translation_text"])

# Text generation (for paraphrase/QA)
generator = pipeline("text2text-generation", model="t5-base")
result = generator("question: What is AI? context: Artificial intelligence "
                   "is the simulation of human intelligence by machines.")
print("Answer:", result[0]["generated_text"])
```

---

## 6. Common Pitfalls and Solutions

```
┌────────────────────────┬─────────────────────────────────────────┐
│ Pitfall                │ Solution                                │
├────────────────────────┼─────────────────────────────────────────┤
│ Hallucinated facts     │ Use constrained decoding or             │
│                        │ faithfulness metrics (FactCC)           │
├────────────────────────┼─────────────────────────────────────────┤
│ Repetitive output      │ Set no_repeat_ngram_size=3              │
│                        │ Use repetition_penalty=1.2              │
├────────────────────────┼─────────────────────────────────────────┤
│ Output too short       │ Increase min_length parameter           │
│                        │ Increase length_penalty                 │
├────────────────────────┼─────────────────────────────────────────┤
│ Output too long        │ Decrease max_length parameter           │
│                        │ Decrease length_penalty                 │
├────────────────────────┼─────────────────────────────────────────┤
│ Slow inference         │ Use quantization, reduce beams,         │
│                        │ switch to smaller model variant         │
├────────────────────────┼─────────────────────────────────────────┤
│ Out of memory          │ Use gradient checkpointing,             │
│                        │ reduce batch size, use FP16             │
├────────────────────────┼─────────────────────────────────────────┤
│ Poor quality on domain │ Fine-tune on domain-specific data       │
│                        │ (even 1000 examples helps)              │
└────────────────────────┴─────────────────────────────────────────┘
```

---

## 7. Real-World Applications

| Application             | Model             | Scale                  | Industry              |
|--------------------------|-------------------|------------------------|-----------------------|
| Google Search snippets   | T5 variants       | Billions of queries    | Search                |
| Legal document summary   | BART / Pegasus    | Thousands of docs/day  | Legal tech            |
| Multilingual support     | mBART / mT5       | 100+ languages         | Localization          |
| Medical report generation| Fine-tuned T5     | Radiology, pathology   | Healthcare            |
| Code documentation       | CodeT5            | Millions of functions  | Developer tools       |
| News headline generation | BART-CNN          | Real-time feeds        | Media                 |
| Chatbot responses        | BART / BlenderBot | Customer interactions  | Customer service      |

---

## 8. Summary Table

| Aspect                 | Detail                                                |
|------------------------|-------------------------------------------------------|
| **Key Models**         | T5, BART, Pegasus, mBART, mT5                        |
| **Primary Tasks**      | Summarization, translation, QA, paraphrase            |
| **HuggingFace API**    | `pipeline()` for quick start, manual for control      |
| **Decoding**           | Beam search (quality) vs. greedy (speed)              |
| **Optimization**       | Quantization, distillation, ONNX, batching            |
| **Evaluation**         | ROUGE (summarization), BLEU (translation)             |
| **Common Issues**      | Hallucination, repetition, length control             |
| **Best Practice**      | Start with pipeline API, fine-tune for domain tasks   |
| **Production Stack**   | Model + tokenizer + pre/postprocessing + monitoring   |

---

## 9. Revision Questions

1. **Design a summarization pipeline for news articles.** What preprocessing steps would you include? How would you handle articles longer than the model's maximum input length (1024 tokens)?

2. **Compare deployment of T5-Small vs. BART-Large.** In what scenarios would you choose the smaller model despite lower quality? How much latency reduction can you expect from INT8 quantization?

3. **Explain why generative QA can hallucinate answers.** What techniques can you use to improve faithfulness? How does constrained decoding help?

4. **Build a multi-task NLP service using T5.** How does the text-to-text format enable serving multiple tasks from a single model? What are the tradeoffs vs. task-specific models?

5. **Design a back-translation quality check system.** Translate English → German → English and compare with original. What metrics would you use to flag bad translations?

6. **What are the key differences between using HuggingFace's `pipeline()` API vs. manual model loading?** When should you choose each approach?

---

[← Seq2Seq Tasks](03-seq2seq-tasks.md) | [Vision Transformer →](../10-Modern-Transformer-Variants/01-vision-transformer.md)
