# In-Context Learning

> **Navigation:** [← Scaling Laws](04-scaling-laws.md) | [Prompt Engineering →](06-prompt-engineering.md)

---

## Overview

**In-context learning (ICL)** is a remarkable capability of large language models where the model learns to perform a task **at inference time** by conditioning on examples provided in the prompt — without any gradient updates or parameter changes. Demonstrated most prominently in GPT-3 (Brown et al., 2020), in-context learning comes in three flavors: **zero-shot** (task description only), **one-shot** (one example), and **few-shot** (multiple examples). This paradigm represents a fundamental shift from the traditional train-then-fine-tune approach: instead of updating model weights for each new task, you simply describe the task in natural language and optionally provide examples. Understanding in-context learning is essential for effectively using modern LLMs and forms the basis of prompt engineering.

---

## 1. The Three Settings

### 1.1 Zero-Shot Learning

The model receives only a **task description** — no examples.

```
┌─────────────────────────────────────────────────────┐
│  Prompt:                                            │
│  "Classify the sentiment of this review as          │
│   positive or negative.                             │
│                                                     │
│   Review: The movie was absolutely wonderful,       │
│   with stunning visuals and a gripping plot.        │
│   Sentiment:"                                       │
│                                                     │
│  Model output: " positive"                          │
└─────────────────────────────────────────────────────┘
```

### 1.2 One-Shot Learning

The model receives **one example** demonstrating the task.

```
┌─────────────────────────────────────────────────────┐
│  Prompt:                                            │
│  "Classify the sentiment of movie reviews.          │
│                                                     │
│   Review: I loved every minute of this film!        │
│   Sentiment: positive                               │
│                                                     │
│   Review: The movie was absolutely wonderful,       │
│   with stunning visuals and a gripping plot.        │
│   Sentiment:"                                       │
│                                                     │
│  Model output: " positive"                          │
└─────────────────────────────────────────────────────┘
```

### 1.3 Few-Shot Learning

The model receives **multiple examples** (typically 3-20).

```
┌─────────────────────────────────────────────────────┐
│  Prompt:                                            │
│  "Classify the sentiment of movie reviews.          │
│                                                     │
│   Review: I loved every minute of this film!        │
│   Sentiment: positive                               │
│                                                     │
│   Review: Terrible acting and a boring plot.        │
│   Sentiment: negative                               │
│                                                     │
│   Review: An average film, nothing special.         │
│   Sentiment: neutral                                │
│                                                     │
│   Review: The movie was absolutely wonderful,       │
│   with stunning visuals and a gripping plot.        │
│   Sentiment:"                                       │
│                                                     │
│  Model output: " positive"                          │
└─────────────────────────────────────────────────────┘
```

---

## 2. How In-Context Learning Works

### 2.1 No Gradient Updates

```
  Traditional Fine-tuning:                 In-Context Learning:
  ┌────────────────────────┐               ┌────────────────────────┐
  │  Training examples     │               │  Examples in prompt    │
  │         │               │               │         │               │
  │         ▼               │               │         ▼               │
  │  Forward pass           │               │  Forward pass           │
  │         │               │               │         │               │
  │         ▼               │               │         ▼               │
  │  Compute loss           │               │  Generate output       │
  │         │               │               │  (NO loss, NO gradients)│
  │         ▼               │               │                        │
  │  Backpropagation        │               │  Parameters UNCHANGED  │
  │         │               │               │                        │
  │         ▼               │               └────────────────────────┘
  │  Update parameters      │
  │  (θ ← θ - η∇L)         │
  └────────────────────────┘
```

### 2.2 The Mechanism: Pattern Completion

In-context learning works because the model treats the examples as part of a **text pattern** and simply continues the pattern:

```
The model sees:
  "Input: X₁ → Output: Y₁
   Input: X₂ → Output: Y₂
   Input: X₃ → Output: Y₃
   Input: X_test → Output:"

The model has learned during pre-training that text often follows
patterns. It recognizes this as a pattern-completion task and
predicts the output that continues the pattern.
```

### 2.3 Mathematical Perspective

In-context learning can be viewed as **implicit Bayesian inference**:

```
P(y_test | x_test, {(x₁,y₁), ..., (xₖ,yₖ)})

The model computes:
1. From the examples, infer the latent "task" concept c
   P(c | {(x₁,y₁), ..., (xₖ,yₖ)})

2. Given the task concept, predict the output
   P(y_test | x_test, c)

No explicit inference is done — the transformer's forward pass
implicitly performs both steps through its attention mechanism.
```

Recent research (Garg et al., 2022; Akyürek et al., 2023) suggests that transformers can implicitly implement learning algorithms (like linear regression or gradient descent) within their forward pass.

---

## 3. Performance Scaling with Examples

### 3.1 General Trend

```
  Accuracy
  ▲
  │                                    ●──●──●  ← few-shot (plateaus)
  │                              ●──●
  │                         ●──●
  │                    ●
  │               ●
  │          ●
  │     ●
  │  ●  ← zero-shot
  │
  └──────────────────────────────────────────► Number of Examples (k)
     0    1    2    3    4    8   16   32

  Performance improves with more examples but shows
  diminishing returns after ~10-20 examples.
```

### 3.2 Factors Affecting ICL Performance

| Factor                    | Effect on Performance                          |
|---------------------------|------------------------------------------------|
| Number of examples        | More examples → better (diminishing returns)   |
| Model size                | Larger models → much better ICL                |
| Example quality           | Representative examples → better               |
| Example order             | Order matters! (can swing accuracy ±20%)       |
| Label balance             | Balanced labels → more robust                  |
| Format consistency        | Consistent formatting → better pattern matching|
| Task complexity           | Simpler tasks benefit more from few-shot       |

### 3.3 Model Size Matters for ICL

```
  Few-shot Accuracy (%)
  ▲
  │                                          ●  GPT-3 175B
  90│                                     ●
  │                                ●
  80│                          ●
  │                     ●
  70│                ●
  │           ●
  60│      ●
  │   ●
  50│  ●
  │ ●
  40│●
  │
  └──────────────────────────────────────► Model Parameters
   125M  350M  1.3B  2.7B  6.7B  13B  175B

  ICL capability scales strongly with model size.
  Small models (<1B) show minimal ICL ability.
```

---

## 4. Comparison: In-Context Learning vs. Fine-Tuning

| Aspect                  | In-Context Learning         | Fine-Tuning                  |
|-------------------------|-----------------------------|------------------------------|
| Parameter updates       | None                        | Yes (gradient descent)       |
| Examples needed         | 0–32 (in prompt)            | 100s–1000s                   |
| Training time           | Zero                        | Minutes to hours             |
| Compute at inference    | Higher (longer prompts)     | Normal                       |
| Task switching          | Instant (change prompt)     | Requires retraining          |
| Max performance         | Good, not SOTA              | Often higher                 |
| Data storage            | In the prompt               | Separate dataset             |
| Model copies            | One model, many tasks       | One model per task           |
| Risk of forgetting      | None                        | Catastrophic forgetting      |

### When to Use Each

```
  Use In-Context Learning when:          Use Fine-Tuning when:
  ┌───────────────────────────┐          ┌───────────────────────────┐
  │ • Few labeled examples    │          │ • Large labeled dataset   │
  │ • Rapid prototyping       │          │ • Maximum accuracy needed │
  │ • Many diverse tasks      │          │ • Consistent domain       │
  │ • No training infra       │          │ • Cost per query matters  │
  │ • Task may change often   │          │ • Specialized vocabulary  │
  └───────────────────────────┘          └───────────────────────────┘
```

---

## 5. Worked Examples

### Example 1: Named Entity Recognition (Zero-Shot)

```
Prompt:
  "Extract all person names from the following text.

   Text: Dr. Sarah Chen met with Professor James Miller at the
   Stanford AI Lab to discuss their collaboration with OpenAI CEO
   Sam Altman.

   Person names:"

Output:
  "Sarah Chen, James Miller, Sam Altman"
```

### Example 2: Translation (One-Shot)

```
Prompt:
  "Translate English to Japanese:

   English: Good morning
   Japanese: おはようございます

   English: Thank you very much
   Japanese:"

Output:
  "どうもありがとうございます"
```

### Example 3: Classification (Few-Shot)

```
Prompt:
  "Classify each email as 'spam' or 'not spam'.

   Email: 'Congratulations! You've won $1,000,000!'
   Class: spam

   Email: 'Meeting rescheduled to 3pm tomorrow'
   Class: not spam

   Email: 'Buy cheap watches at unbeatable prices!!!'
   Class: spam

   Email: 'Your quarterly report is ready for review'
   Class: not spam

   Email: 'URGENT: Verify your bank account now'
   Class:"

Output:
  "spam"
```

### Example 4: Code Generation (Few-Shot)

```
Prompt:
  "Write a Python function based on the description.

   Description: Return the factorial of n
   Code: def factorial(n): return 1 if n <= 1 else n * factorial(n-1)

   Description: Return the nth Fibonacci number
   Code: def fibonacci(n): return n if n <= 1 else fibonacci(n-1) + fibonacci(n-2)

   Description: Check if a string is a palindrome
   Code:"

Output:
  "def is_palindrome(s): return s == s[::-1]"
```

---

## 6. The Role of Example Selection and Ordering

### 6.1 Example Selection

Choosing the right examples significantly impacts performance:

```
  Strategy               Description                      Effect
  ─────────────────────────────────────────────────────────────────
  Random selection       Pick examples randomly            Baseline
  Similar examples       Select examples similar to query  +5-15%
  Diverse examples       Cover different categories        +3-8%
  Difficult examples     Include edge cases                Variable
  Nearest neighbor       k-NN in embedding space           +10-20%
```

### 6.2 Example Ordering Effects

```
  Same examples, different order → different accuracy!

  Order A: [positive, positive, negative, negative] → 72% accuracy
  Order B: [positive, negative, positive, negative] → 85% accuracy
  Order C: [negative, positive, negative, positive] → 83% accuracy

  Best practice: Alternate labels to avoid recency bias.
  The model tends to be biased toward the label of the last example.
```

---

## 7. Code: Demonstrating Zero/One/Few-Shot with an LLM API

```python
import openai


def query_llm(prompt, model="gpt-3.5-turbo", temperature=0.0, max_tokens=100):
    """Send a prompt to the LLM and return the response."""
    response = openai.ChatCompletion.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=temperature,
        max_tokens=max_tokens,
    )
    return response.choices[0].message.content.strip()


def zero_shot_sentiment(text):
    """Zero-shot sentiment classification."""
    prompt = f"""Classify the sentiment of this text as positive, negative, or neutral.

Text: {text}
Sentiment:"""
    return query_llm(prompt)


def one_shot_sentiment(text):
    """One-shot sentiment classification."""
    prompt = f"""Classify the sentiment of this text as positive, negative, or neutral.

Text: I absolutely loved this restaurant, the food was amazing!
Sentiment: positive

Text: {text}
Sentiment:"""
    return query_llm(prompt)


def few_shot_sentiment(text, examples=None):
    """Few-shot sentiment classification with configurable examples."""
    if examples is None:
        examples = [
            ("I absolutely loved this restaurant!", "positive"),
            ("The service was terrible and rude.", "negative"),
            ("The food was okay, nothing special.", "neutral"),
            ("Best experience I've ever had!", "positive"),
            ("I will never come back here again.", "negative"),
        ]

    prompt = "Classify the sentiment of each text as positive, negative, or neutral.\n\n"
    for ex_text, ex_label in examples:
        prompt += f"Text: {ex_text}\nSentiment: {ex_label}\n\n"
    prompt += f"Text: {text}\nSentiment:"

    return query_llm(prompt)


def compare_settings(test_texts):
    """Compare zero-shot, one-shot, and few-shot on multiple texts."""
    print(f"{'Text':<45} {'Zero-shot':<12} {'One-shot':<12} {'Few-shot':<12}")
    print("-" * 81)

    for text in test_texts:
        zero = zero_shot_sentiment(text)
        one = one_shot_sentiment(text)
        few = few_shot_sentiment(text)
        short_text = text[:42] + "..." if len(text) > 42 else text
        print(f"{short_text:<45} {zero:<12} {one:<12} {few:<12}")


def few_shot_translation(text, source_lang="English", target_lang="Spanish"):
    """Few-shot translation using in-context examples."""
    prompt = f"""Translate {source_lang} to {target_lang}.

{source_lang}: Hello, how are you?
{target_lang}: Hola, ¿cómo estás?

{source_lang}: The weather is nice today.
{target_lang}: El clima está agradable hoy.

{source_lang}: I would like a cup of coffee.
{target_lang}: Me gustaría una taza de café.

{source_lang}: {text}
{target_lang}:"""
    return query_llm(prompt)


def few_shot_structured_output(text):
    """Extract structured data using few-shot prompting."""
    prompt = """Extract the name, age, and occupation from each sentence as JSON.

Sentence: John Smith is a 35-year-old software engineer.
JSON: {"name": "John Smith", "age": 35, "occupation": "software engineer"}

Sentence: Dr. Maria Garcia, 42, works as a cardiologist.
JSON: {"name": "Maria Garcia", "age": 42, "occupation": "cardiologist"}

Sentence: """ + text + """
JSON:"""
    return query_llm(prompt)


# --- Main ---
if __name__ == "__main__":
    test_texts = [
        "This product exceeded all my expectations!",
        "I'm disappointed with the quality.",
        "It works as described, nothing more.",
        "Absolutely fantastic, highly recommend!",
        "The worst purchase I've ever made.",
    ]

    print("=== Sentiment Classification Comparison ===\n")
    compare_settings(test_texts)

    print("\n=== Few-Shot Translation ===")
    translations = [
        "The book is on the table.",
        "Can you help me find the train station?",
    ]
    for text in translations:
        result = few_shot_translation(text)
        print(f"  EN: {text}")
        print(f"  ES: {result}\n")

    print("=== Structured Output Extraction ===")
    result = few_shot_structured_output(
        "Chef Alice Wong, age 28, runs a popular restaurant in San Francisco."
    )
    print(f"  Output: {result}")
```

---

## 8. Real-World Applications

| Application                  | ICL Setting    | Why ICL Works Here                          |
|------------------------------|----------------|---------------------------------------------|
| **Customer support triage**  | Few-shot       | Categorize tickets with example categories  |
| **Content moderation**       | Few-shot       | Flag harmful content with labeled examples  |
| **Data extraction**          | Few-shot       | Extract entities from varied documents      |
| **Rapid prototyping**        | Zero/few-shot  | Test NLP ideas without training             |
| **Multi-language support**   | One-shot       | Quick translation with one example pair     |
| **Code review assistance**   | Few-shot       | Identify patterns from example reviews      |

---

## 9. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| In-context learning        | Learning from examples in the prompt; no weight updates    |
| Zero-shot                  | Task description only; relies on pre-training knowledge    |
| One-shot                   | One labeled example in the prompt                          |
| Few-shot                   | Multiple examples (typically 3-20) in the prompt           |
| No gradient updates        | All "learning" happens in the forward pass                 |
| Mechanism                  | Pattern completion + implicit task inference                |
| Scale dependence           | ICL ability increases strongly with model size             |
| Example ordering           | Order matters — can swing accuracy by 20%+                 |
| vs. Fine-tuning            | Faster, more flexible, but typically lower max accuracy    |
| Limitation                 | Bounded by context window size                             |

---

## 10. Revision Questions

1. **Define zero-shot, one-shot, and few-shot in-context learning.** Provide an example prompt for each using a topic classification task.

2. **Why does in-context learning require no gradient updates?** How does this differ fundamentally from fine-tuning?

3. **How does model scale affect in-context learning ability?** Why do small models (<1B parameters) struggle with ICL?

4. **Explain why example ordering matters in few-shot prompts.** What is recency bias and how can you mitigate it?

5. **Compare in-context learning and fine-tuning** across five dimensions: data requirements, performance, flexibility, cost, and risk of forgetting.

6. **Design a few-shot prompt** for classifying customer emails into categories: billing, technical, general inquiry, and complaint. Include at least one example per category.

---

> **Navigation:** [← Scaling Laws](04-scaling-laws.md) | [Prompt Engineering →](06-prompt-engineering.md)
