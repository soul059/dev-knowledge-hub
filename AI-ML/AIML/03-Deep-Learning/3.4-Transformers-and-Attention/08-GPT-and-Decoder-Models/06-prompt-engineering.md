# Prompt Engineering Fundamentals

> **Navigation:** [← In-Context Learning](05-in-context-learning.md) | [T5 Architecture →](../09-Encoder-Decoder-Transformers/01-t5-architecture.md)

---

## Overview

**Prompt engineering** is the art and science of crafting effective inputs (prompts) to large language models to elicit desired outputs. As LLMs have grown in capability, the ability to communicate effectively with them through prompts has become a critical skill. This chapter covers the core principles of prompt design — clarity, specificity, and structure — along with advanced techniques like **chain-of-thought prompting**, **role assignment**, and **structured output generation**. Prompt engineering bridges the gap between a model's latent capabilities and practical, reliable outputs, making it essential for anyone building applications on top of LLMs.

---

## 1. Prompt Design Principles

### 1.1 The Three Pillars

```
  ┌─────────────────────────────────────────────────────────┐
  │                  EFFECTIVE PROMPTS                       │
  │                                                         │
  │    ┌──────────┐   ┌──────────────┐   ┌──────────────┐  │
  │    │ CLARITY  │   │ SPECIFICITY  │   │  STRUCTURE   │  │
  │    │          │   │              │   │              │  │
  │    │ Clear    │   │ Precise      │   │ Consistent   │  │
  │    │ language │   │ constraints  │   │ formatting   │  │
  │    │ and      │   │ and expected │   │ with clear   │  │
  │    │ intent   │   │ output       │   │ delimiters   │  │
  │    └──────────┘   └──────────────┘   └──────────────┘  │
  └─────────────────────────────────────────────────────────┘
```

### 1.2 Principles in Detail

| Principle      | Description                                           | Example                           |
|----------------|-------------------------------------------------------|-----------------------------------|
| **Clarity**    | State the task in unambiguous language                 | "Summarize in 3 bullet points"    |
| **Specificity**| Define constraints, format, length, style              | "Reply in JSON with keys: name, age"|
| **Context**    | Provide relevant background information                | "You are analyzing medical records"|
| **Examples**   | Show input-output pairs (few-shot)                    | Include 2-3 demonstrations        |
| **Delimiters** | Use markers to separate sections                       | Use ```, ---, or XML tags         |
| **Role**       | Assign an identity or expertise                        | "You are an expert data scientist" |
| **Output spec**| Describe the desired format explicitly                 | "Return a markdown table"         |

---

## 2. Prompt Anatomy

A well-structured prompt has distinct sections:

```
  ┌─────────────────────────────────────────────────────┐
  │  SYSTEM / ROLE          ← Who the model should be   │
  │  "You are an expert..."                             │
  ├─────────────────────────────────────────────────────┤
  │  TASK INSTRUCTION       ← What to do                │
  │  "Analyze the following text and..."                │
  ├─────────────────────────────────────────────────────┤
  │  CONTEXT / INPUT        ← Data to work with         │
  │  """                                                │
  │  [The actual text or data]                          │
  │  """                                                │
  ├─────────────────────────────────────────────────────┤
  │  EXAMPLES (optional)    ← Demonstrations             │
  │  Input: ... → Output: ...                           │
  ├─────────────────────────────────────────────────────┤
  │  OUTPUT SPECIFICATION   ← Expected format            │
  │  "Return your answer as a JSON object with..."      │
  ├─────────────────────────────────────────────────────┤
  │  CONSTRAINTS            ← Rules and limitations      │
  │  "Do not include personal opinions."                │
  └─────────────────────────────────────────────────────┘
```

---

## 3. Chain-of-Thought (CoT) Prompting

### 3.1 The Key Insight

**Chain-of-thought prompting** (Wei et al., 2022) dramatically improves LLM reasoning by asking the model to show its work step by step.

### 3.2 Standard vs. CoT Prompting

```
  Standard Prompt:                     Chain-of-Thought Prompt:
  ┌────────────────────────────┐       ┌────────────────────────────────────┐
  │ Q: Roger has 5 tennis     │       │ Q: Roger has 5 tennis balls.      │
  │ balls. He buys 2 cans of  │       │ He buys 2 cans of 3 balls each.   │
  │ 3 balls each. How many    │       │ How many total?                   │
  │ total?                    │       │                                    │
  │                           │       │ A: Let's think step by step.       │
  │ A: 11  ← (often wrong)   │       │ Roger starts with 5 balls.         │
  │                           │       │ He buys 2 cans × 3 balls = 6.     │
  │                           │       │ Total = 5 + 6 = 11. ✓             │
  └────────────────────────────┘       └────────────────────────────────────┘
```

### 3.3 Zero-Shot CoT

Simply adding **"Let's think step by step"** to the prompt triggers reasoning:

```
Prompt:  "Q: If a store has 15 apples and sells 40% of them,
          how many are left?
          Let's think step by step."

Output:  "Step 1: Calculate 40% of 15 = 0.4 × 15 = 6
          Step 2: Subtract from total: 15 - 6 = 9
          Answer: 9 apples remain."
```

### 3.4 When CoT Helps Most

| Task Type              | CoT Improvement | Why                                    |
|------------------------|-----------------|----------------------------------------|
| Multi-step math        | +20-40%         | Forces sequential computation          |
| Logical reasoning      | +15-30%         | Makes implicit steps explicit          |
| Word problems          | +25-35%         | Translates language to operations      |
| Common sense           | +5-15%          | Activates relevant knowledge           |
| Simple classification  | ~0%             | No multi-step reasoning needed         |
| Fact retrieval         | ~0%             | Answer is memorized or not             |

---

## 4. System Prompts and Role Assignment

### 4.1 Role-Based Prompting

Assigning a role focuses the model's knowledge and response style:

```
┌─────────────────────────────────────────────────────────┐
│  System: "You are a senior Python developer with 15     │
│  years of experience. You write clean, well-documented  │
│  code following PEP 8. You always include type hints    │
│  and docstrings."                                       │
│                                                         │
│  User: "Write a function to merge two sorted lists."    │
│                                                         │
│  → Model produces PEP 8 compliant code with type hints  │
│    and comprehensive docstrings.                        │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Effective Role Patterns

| Role                          | Use Case                               |
|-------------------------------|----------------------------------------|
| "Expert data scientist"       | Statistical analysis, ML advice        |
| "Technical writer"            | Documentation, clear explanations      |
| "Socratic tutor"              | Educational Q&A, guided discovery      |
| "Code reviewer"               | Finding bugs, suggesting improvements  |
| "Legal analyst"               | Contract review, compliance checking   |
| "Creative writing coach"      | Story structure, character development |

---

## 5. Structured Output Generation

### 5.1 JSON Output

```
Prompt:
  "Extract the following information from the text and return it
   as a JSON object with keys: 'name', 'company', 'role', 'email'.

   Text: 'Contact Dr. Sarah Chen, Chief AI Officer at DeepTech
   Labs, at sarah.chen@deeptech.io for collaborations.'

   JSON:"

Output:
  {
    "name": "Dr. Sarah Chen",
    "company": "DeepTech Labs",
    "role": "Chief AI Officer",
    "email": "sarah.chen@deeptech.io"
  }
```

### 5.2 Table Output

```
Prompt:
  "Compare Python, JavaScript, and Rust across the following
   dimensions: typing, speed, main use case, learning curve.
   Return as a markdown table."

Output:
  | Language   | Typing   | Speed  | Main Use Case    | Learning Curve |
  |------------|----------|--------|------------------|----------------|
  | Python     | Dynamic  | Slow   | ML, scripting    | Easy           |
  | JavaScript | Dynamic  | Medium | Web development  | Medium         |
  | Rust       | Static   | Fast   | Systems, safety  | Hard           |
```

### 5.3 Structured Extraction Pipeline

```
  Raw Text ──► Prompt with ──► LLM ──► Structured ──► Application
               format spec            output (JSON)    logic

  Example pipeline:
  ┌─────────────┐    ┌───────────────┐    ┌──────────────┐
  │ "Invoice    │    │ System: You   │    │ {"invoice":  │
  │  #1234      │ ─► │ extract data  │ ─► │  "1234",     │
  │  Amount:    │    │ as JSON.      │    │  "amount":   │
  │  $500       │    │               │    │  500, ...}   │
  │  Due: 1/15" │    │ User: [text]  │    │              │
  └─────────────┘    └───────────────┘    └──────────────┘
```

---

## 6. Common Prompt Patterns

### 6.1 Classification

```
Prompt:
  "Classify the following customer message into exactly one category:
   billing, technical_support, general_inquiry, or complaint.

   Message: 'I've been charged twice for my subscription this month.'

   Category:"

Output: "billing"
```

### 6.2 Extraction

```
Prompt:
  "Extract all dates and associated events from the text below.
   Return as a numbered list.

   Text: 'The project launched on March 15, 2024. The first
   milestone review is scheduled for June 1, 2024, and the
   final delivery is expected by December 20, 2024.'

   Dates and events:"

Output:
  "1. March 15, 2024 - Project launch
   2. June 1, 2024 - First milestone review
   3. December 20, 2024 - Final delivery"
```

### 6.3 Summarization

```
Prompt:
  "Summarize the following article in exactly 3 bullet points.
   Each bullet should be one sentence. Focus on key findings.

   Article:
   '''
   [long article text]
   '''

   Summary:"
```

### 6.4 Generation with Constraints

```
Prompt:
  "Write a product description for a wireless noise-canceling
   headphone. Requirements:
   - Exactly 50-75 words
   - Mention battery life, comfort, and sound quality
   - Professional but engaging tone
   - End with a call to action

   Description:"
```

---

## 7. Best Practices and Anti-Patterns

### 7.1 Best Practices

```
  ✅ DO:
  ┌─────────────────────────────────────────────────────────┐
  │ • Be specific about the desired output format           │
  │ • Use delimiters (''', ---, <tags>) for input sections  │
  │ • Provide examples for complex tasks                    │
  │ • Specify constraints explicitly                        │
  │ • Break complex tasks into steps                        │
  │ • Test with edge cases                                  │
  │ • Iterate and refine based on outputs                   │
  │ • Use "do X" rather than "don't do Y"                   │
  │ • Include the expected output format in the prompt       │
  └─────────────────────────────────────────────────────────┘
```

### 7.2 Anti-Patterns

```
  ❌ DON'T:
  ┌─────────────────────────────────────────────────────────┐
  │ • Use vague instructions ("make it better")             │
  │ • Combine too many tasks in one prompt                  │
  │ • Assume the model knows your context                   │
  │ • Rely on negation ("don't mention X")                  │
  │ • Use ambiguous pronouns without referents              │
  │ • Expect exact numerical computation                    │
  │ • Skip output format specification                      │
  │ • Ignore prompt length limits                           │
  └─────────────────────────────────────────────────────────┘
```

### 7.3 Before and After Examples

```
  ❌ Bad:   "Summarize this."
  ✅ Good:  "Summarize the following article in 3 bullet points,
             each under 20 words, focusing on the main findings.
             Article: '''[text]'''"

  ❌ Bad:   "Write code."
  ✅ Good:  "Write a Python function that takes a list of integers
             and returns the two numbers that sum to a target value.
             Include type hints, a docstring, and handle edge cases.
             Use O(n) time complexity."

  ❌ Bad:   "Don't be verbose."
  ✅ Good:  "Respond in exactly one sentence."

  ❌ Bad:   "Analyze the data."
  ✅ Good:  "Calculate the mean, median, and standard deviation of
             the following dataset. Present results in a table.
             Dataset: [4, 7, 2, 9, 1, 8, 3]"
```

---

## 8. Advanced Techniques

### 8.1 Self-Consistency (Wang et al., 2022)

Generate multiple chain-of-thought paths and take the majority vote:

```
  Prompt → CoT Path 1 → Answer: 42
  Prompt → CoT Path 2 → Answer: 42
  Prompt → CoT Path 3 → Answer: 38
  Prompt → CoT Path 4 → Answer: 42
  Prompt → CoT Path 5 → Answer: 42

  Majority vote → Final answer: 42 (4/5 agreement)
```

### 8.2 Prompt Chaining

Break complex tasks into a sequence of simpler prompts:

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Step 1:  │     │ Step 2:  │     │ Step 3:  │     │ Step 4:  │
  │ Extract  │ ──► │ Classify │ ──► │ Generate │ ──► │ Format   │
  │ entities │     │ intent   │     │ response │     │ output   │
  └──────────┘     └──────────┘     └──────────┘     └──────────┘
       │                │                │                │
       ▼                ▼                ▼                ▼
  "Names: A, B"    "Intent: Q&A"    "Answer: ..."    "{json}"
```

### 8.3 Meta-Prompting

Ask the model to write or improve its own prompt:

```
Prompt:
  "I need to classify customer support tickets into categories.
   Write an optimized prompt that I can use for this task.
   Include 3 few-shot examples and output format specification."

Output:
  "Classify the following customer support ticket into one of
   these categories: billing, technical, account, shipping.

   Ticket: 'My payment was declined'
   Category: billing
   ..."
```

---

## 9. Code: Prompt Engineering Examples

```python
from dataclasses import dataclass


@dataclass
class Prompt:
    """Structured prompt builder for LLM interactions."""
    system: str = ""
    instruction: str = ""
    context: str = ""
    examples: list = None
    output_spec: str = ""
    constraints: str = ""

    def build(self) -> str:
        """Build the complete prompt string."""
        parts = []
        if self.system:
            parts.append(f"System: {self.system}\n")
        if self.instruction:
            parts.append(f"Task: {self.instruction}\n")
        if self.examples:
            parts.append("Examples:")
            for inp, out in self.examples:
                parts.append(f"  Input: {inp}")
                parts.append(f"  Output: {out}\n")
        if self.context:
            parts.append(f"Context:\n'''\n{self.context}\n'''\n")
        if self.output_spec:
            parts.append(f"Output format: {self.output_spec}\n")
        if self.constraints:
            parts.append(f"Constraints: {self.constraints}\n")
        parts.append("Output:")
        return "\n".join(parts)


def classification_prompt(text, categories):
    """Build a classification prompt with few-shot examples."""
    prompt = Prompt(
        system="You are a precise text classifier.",
        instruction=f"Classify the text into one of: {', '.join(categories)}.",
        examples=[
            ("I need to reset my password", "technical_support"),
            ("Why was I charged twice?", "billing"),
            ("I want to cancel my account", "account_management"),
        ],
        context=text,
        output_spec="Return only the category name, nothing else.",
        constraints="Choose exactly one category from the provided list.",
    )
    return prompt.build()


def chain_of_thought_prompt(question):
    """Build a chain-of-thought math reasoning prompt."""
    prompt = Prompt(
        system="You are a careful math tutor who shows all work.",
        instruction="Solve the following problem step by step.",
        examples=[
            (
                "If a train travels 60 mph for 2.5 hours, how far does it go?",
                "Step 1: Use distance = speed × time\n"
                "Step 2: distance = 60 × 2.5 = 150\n"
                "Answer: 150 miles",
            ),
        ],
        context=question,
        output_spec="Show each step, then give the final answer on a new line "
                    "starting with 'Answer:'.",
    )
    return prompt.build()


def extraction_prompt(text, fields):
    """Build a structured extraction prompt."""
    fields_str = ", ".join(f'"{f}"' for f in fields)
    prompt = Prompt(
        system="You are a precise information extractor.",
        instruction=f"Extract the following fields from the text: {fields_str}.",
        examples=[
            (
                "John Smith, 35, is a software engineer at Google in NYC.",
                '{"name": "John Smith", "age": 35, "role": "software engineer", '
                '"company": "Google", "location": "NYC"}',
            ),
        ],
        context=text,
        output_spec="Return a valid JSON object with the specified keys. "
                    'Use null for missing fields.',
    )
    return prompt.build()


def summarization_prompt(text, num_points=3, max_words_per_point=20):
    """Build a constrained summarization prompt."""
    prompt = Prompt(
        system="You are a concise technical writer.",
        instruction=f"Summarize the text in exactly {num_points} bullet points.",
        context=text,
        output_spec=f"Each bullet point should be at most {max_words_per_point} "
                    f"words. Use '•' as the bullet character.",
        constraints="Focus on key findings and actionable insights. "
                    "Do not include opinions.",
    )
    return prompt.build()


def self_consistency_prompt(question, n_paths=5):
    """Generate multiple reasoning paths for self-consistency."""
    base_prompt = chain_of_thought_prompt(question)
    prompts = []
    for i in range(n_paths):
        path_prompt = base_prompt + f"\n(Reasoning path {i+1} of {n_paths})"
        prompts.append(path_prompt)
    return prompts


# --- Demonstration ---
if __name__ == "__main__":
    # 1. Classification
    print("=" * 60)
    print("CLASSIFICATION PROMPT")
    print("=" * 60)
    categories = ["billing", "technical_support", "account_management", "feedback"]
    text = "My internet has been down for three days!"
    print(classification_prompt(text, categories))

    # 2. Chain-of-thought
    print("\n" + "=" * 60)
    print("CHAIN-OF-THOUGHT PROMPT")
    print("=" * 60)
    question = ("A store sells notebooks for $3.50 each. If you buy 4 notebooks "
                "and pay with a $20 bill, how much change do you get?")
    print(chain_of_thought_prompt(question))

    # 3. Extraction
    print("\n" + "=" * 60)
    print("EXTRACTION PROMPT")
    print("=" * 60)
    text = ("Dr. Emily Watson, Chief Medical Officer at HealthFirst Inc., "
            "announced the clinical trial results at the Boston conference.")
    fields = ["name", "title", "company", "event", "location"]
    print(extraction_prompt(text, fields))

    # 4. Summarization
    print("\n" + "=" * 60)
    print("SUMMARIZATION PROMPT")
    print("=" * 60)
    article = ("Researchers at MIT have developed a new algorithm that reduces "
               "LLM inference time by 40% while maintaining accuracy. The method "
               "uses adaptive computation, skipping unnecessary layers for simple "
               "inputs. Initial tests show consistent results across translation, "
               "summarization, and question answering tasks.")
    print(summarization_prompt(article, num_points=3, max_words_per_point=15))
```

---

## 10. Prompt Engineering Evaluation

### Measuring Prompt Quality

```
  Metric              Description                          How to Measure
  ──────────────────────────────────────────────────────────────────────────
  Accuracy            Correctness of output                 Compare to ground truth
  Consistency         Same input → same output              Run N times, measure variance
  Format compliance   Output matches requested format       Parse and validate
  Robustness          Performance on edge cases             Test with adversarial inputs
  Token efficiency    Achieve goal with fewer tokens        Count prompt + output tokens
  Latency             Time to generate response             Measure wall clock time
```

---

## 11. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| Clarity                    | Unambiguous language and clear task specification           |
| Specificity                | Precise constraints: format, length, style                 |
| Chain-of-thought           | "Let's think step by step" — improves reasoning 20-40%     |
| Few-shot examples          | 2-5 demonstrations of desired input → output               |
| System/role prompts        | Assign expertise to focus model behavior                   |
| Structured outputs         | Specify JSON, tables, or lists for parseable results       |
| Delimiters                 | Use ```, """, --- to separate prompt sections              |
| Self-consistency           | Multiple CoT paths + majority vote for robustness          |
| Prompt chaining            | Break complex tasks into sequential simpler prompts        |
| Anti-patterns              | Avoid vagueness, negation-only instructions, ambiguity     |

---

## 12. Revision Questions

1. **List the three pillars of effective prompt design.** Give a concrete example of how violating each one leads to poor outputs.

2. **Explain chain-of-thought prompting.** Why does adding "Let's think step by step" improve mathematical reasoning?

3. **Design a prompt** for extracting product information (name, price, category, rating) from e-commerce reviews. Include system role, examples, and output format.

4. **What is self-consistency?** How does it improve upon single-path chain-of-thought prompting?

5. **Compare a bad prompt and a good prompt** for the task of translating code comments from English to Spanish. Explain why one is better.

6. **Design a prompt chain** (3 steps) for analyzing a customer review: Step 1 — extract sentiment, Step 2 — identify key issues, Step 3 — generate a response.

---

> **Navigation:** [← In-Context Learning](05-in-context-learning.md) | [T5 Architecture →](../09-Encoder-Decoder-Transformers/01-t5-architecture.md)
