# Sentiment Analysis (Advanced)

## Overview

Sentiment analysis determines the emotional tone of text — positive, negative, or neutral. While basic sentiment classification was covered in text classification, this chapter covers **advanced** aspects: aspect-based sentiment, fine-grained analysis, multimodal sentiment, and modern approaches using LLMs.

---

## Levels of Sentiment Analysis

```
1. Document-level:
   "This phone is amazing! Great camera, fast processor."
   → Overall: Positive

2. Sentence-level:
   "The camera is great but the battery life is terrible."
   → Sentence: Mixed (positive + negative)

3. Aspect-level (ABSA):
   "The camera is great but the battery life is terrible."
   → camera: Positive
   → battery life: Negative

4. Fine-grained (5-class):
   Very Negative | Negative | Neutral | Positive | Very Positive
       ★            ★★        ★★★       ★★★★       ★★★★★
```

---

## Aspect-Based Sentiment Analysis (ABSA)

```
Task: Identify aspects AND their associated sentiment

Input:  "The food was delicious but the service was slow."

Step 1: Aspect extraction
  Aspects: [food, service]

Step 2: Sentiment per aspect
  food → positive
  service → negative

Pipeline:
  Text → [Aspect Extractor] → [Sentiment Classifier per Aspect] → Results
  
  Or joint model:
  Text → [Joint ABSA Model] → {(aspect, sentiment, opinion_term)}

Output:
  {(food, positive, "delicious"), (service, negative, "slow")}
```

---

## ABSA with Transformers

```python
from transformers import pipeline

# Using a fine-tuned ABSA model
absa = pipeline("text-classification", 
                 model="yangheng/deberta-v3-base-absa-v1.1")

reviews = [
    "The pizza was delicious but delivery took forever",
    "Excellent battery life and the screen is stunning",
    "The hotel room was clean but very noisy at night",
]

# For each review, check sentiment toward different aspects
aspects = {
    0: ["pizza", "delivery"],
    1: ["battery", "screen"],
    2: ["room", "cleanliness", "noise"],
}

for i, review in enumerate(reviews):
    print(f"\nReview: {review}")
    for aspect in aspects[i]:
        input_text = f"[CLS] {review} [SEP] {aspect} [SEP]"
        result = absa(input_text)
        print(f"  {aspect}: {result[0]['label']} ({result[0]['score']:.2f})")
```

---

## Sentiment with LLMs

```python
from transformers import pipeline

# Zero-shot sentiment with LLM
classifier = pipeline("zero-shot-classification", 
                       model="facebook/bart-large-mnli")

text = "The new update completely ruined the app. Nothing works anymore."

# Fine-grained sentiment
result = classifier(
    text,
    candidate_labels=["very positive", "positive", "neutral", 
                      "negative", "very negative"]
)
print(f"Sentiment: {result['labels'][0]} ({result['scores'][0]:.2f})")
# Sentiment: very negative (0.78)

# Emotion detection
result = classifier(
    text,
    candidate_labels=["anger", "frustration", "sadness", 
                      "disappointment", "joy", "surprise"]
)
print(f"Emotion: {result['labels'][0]} ({result['scores'][0]:.2f})")
# Emotion: frustration (0.65)
```

---

## Challenges in Sentiment Analysis

```
1. Sarcasm/Irony:
   "Oh great, another Monday. Just what I needed."
   → Appears positive, actually negative

2. Negation:
   "This is not bad at all"
   → Double negation = positive

3. Implicit sentiment:
   "The battery lasted only 2 hours"
   → No sentiment words, but clearly negative

4. Comparative:
   "iPhone camera is better than Samsung"
   → iPhone: positive, Samsung: negative (implicit)

5. Domain-specific:
   "The plot was predictable" (movie → negative)
   "The results were predictable" (science → positive)

6. Multilingual:
   Different languages express sentiment differently
   → Need cross-lingual models
```

---

## Emotion Detection

```
Beyond positive/negative — detect specific emotions:

Plutchik's Wheel of Emotions:
  ┌─────────────────────────────┐
  │       anticipation          │
  │      /           \          │
  │   joy             anger     │
  │    |               |        │
  │  trust          disgust     │
  │    |               |        │
  │  fear           surprise    │
  │      \           /          │
  │        sadness              │
  └─────────────────────────────┘

Datasets:
  GoEmotions (Google): 27 emotion labels, 58K Reddit comments
  EmoInt: 4 emotions with intensity scores
  AffectNet: Facial emotion (multimodal)
```

---

## Summary Table

| Method | Granularity | Training | Performance |
|--------|:-:|:-:|:-:|
| Lexicon (VADER) | Document/sentence | None | Moderate |
| Fine-tuned BERT | Document/sentence | Supervised | High |
| ABSA models | Aspect-level | Supervised | High |
| Zero-shot LLM | Any | None | Good |
| Emotion detection | Fine-grained | Supervised | Moderate |
| Multimodal | Text + image/audio | Supervised | Growing |

---

## Revision Questions

1. **What are the different levels of sentiment analysis?**
2. **How does aspect-based sentiment analysis (ABSA) work?**
3. **Why is sarcasm detection challenging for sentiment models?**
4. **How can zero-shot classification be used for sentiment analysis?**
5. **What is the difference between sentiment analysis and emotion detection?**

---

[Previous: 02-semantic-similarity.md](02-semantic-similarity.md) | [Next: 04-topic-modeling.md](04-topic-modeling.md)
