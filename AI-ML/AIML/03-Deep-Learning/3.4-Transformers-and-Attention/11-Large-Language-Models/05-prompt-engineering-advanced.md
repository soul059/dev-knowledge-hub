[← RLHF Basics](04-rlhf-basics.md) | [RAG →](06-rag.md)

---

# Advanced Prompt Engineering Techniques

## Overview

Prompt engineering is the art and science of crafting inputs to Large Language Models that elicit accurate, useful, and reliable outputs. While basic prompting (zero-shot, few-shot) gets surprisingly far, **advanced techniques** unlock significantly better performance on complex reasoning, multi-step problems, and real-world applications. This chapter covers the most impactful methods: **Chain-of-Thought (CoT)** prompting for step-by-step reasoning, **Tree-of-Thought** for exploring multiple reasoning paths, **Self-Consistency** for robust answers via majority voting, **ReAct** for interleaving reasoning with actions, **Tool Use** for extending LLM capabilities, **Structured Output** for reliable data extraction, and **Prompt Chaining** for building multi-step pipelines. These techniques transform LLMs from simple text generators into powerful reasoning engines and autonomous agents.

---

## Chain-of-Thought (CoT) Prompting

### Core Idea

Chain-of-Thought prompting (Wei et al., 2022) instructs the model to **show its reasoning step by step** before giving a final answer, dramatically improving performance on arithmetic, logic, and commonsense reasoning tasks.

```
Standard prompting:
  Q: If a store has 15 apples and sells 7, then receives
     a shipment of 12, how many apples does it have?
  A: 20  ← Model might guess or skip steps

Chain-of-Thought prompting:
  Q: If a store has 15 apples and sells 7, then receives
     a shipment of 12, how many apples does it have?
  A: Let's think step by step.
     1. Start with 15 apples
     2. Sell 7: 15 - 7 = 8 apples
     3. Receive 12: 8 + 12 = 20 apples
     The store has 20 apples.  ← Reliable, verifiable reasoning
```

### Zero-Shot CoT

Simply append **"Let's think step by step"** to any prompt:

```
Prompt:  <question> Let's think step by step.
```

This single phrase improves accuracy by **10-40%** on reasoning benchmarks (Kojima et al., 2022).

### Few-Shot CoT

Provide **demonstration examples** with explicit reasoning chains:

```
Q: Roger has 5 tennis balls. He buys 2 more cans of
   tennis balls. Each can has 3 balls. How many tennis
   balls does he have now?
A: Roger started with 5 balls. 2 cans × 3 balls = 6 balls.
   5 + 6 = 11. The answer is 11.

Q: The cafeteria had 23 apples. If they used 20 to make
   lunch and bought 6 more, how many apples do they have?
A: Started with 23 apples. Used 20: 23 - 20 = 3. Bought 6
   more: 3 + 6 = 9. The answer is 9.

Q: <your actual question>
A:
```

### When CoT Helps

```
Task complexity vs CoT benefit:

  Simple factual lookup:        CoT adds no benefit (may hurt)
  Basic arithmetic:             Moderate benefit (~10-20%)
  Multi-step word problems:     Large benefit (~30-50%)
  Logical reasoning:            Large benefit (~20-40%)
  Complex math proofs:          Significant benefit (~40-60%)
  Code generation:              Moderate benefit (~15-25%)

Rule of thumb: CoT helps when the task requires
multiple reasoning steps and the answer isn't immediately obvious.
```

---

## Tree-of-Thought (ToT)

### Core Idea

Tree-of-Thought (Yao et al., 2023) extends CoT by exploring **multiple reasoning paths** simultaneously and evaluating which paths are most promising.

```
                         ┌─────────┐
                         │ Problem │
                         └────┬────┘
                              │
               ┌──────────────┼──────────────┐
               │              │              │
          ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
          │Thought 1│   │Thought 2│   │Thought 3│
          └────┬────┘   └────┬────┘   └────┬────┘
               │              │              │
          Eval: 0.8      Eval: 0.3      Eval: 0.6
               │              ✗              │
        ┌──────┼──────┐              ┌──────┼──────┐
        │             │              │             │
   ┌────┴────┐  ┌────┴────┐   ┌────┴────┐  ┌────┴────┐
   │Thought  │  │Thought  │   │Thought  │  │Thought  │
   │  1.1    │  │  1.2    │   │  3.1    │  │  3.2    │
   └─────────┘  └─────────┘   └─────────┘  └─────────┘
   Eval: 0.9    Eval: 0.4     Eval: 0.7    Eval: 0.2
       │             ✗             │             ✗
       ▼                           ▼
   Best Path                  Alternative

Search strategies: BFS (breadth-first) or DFS (depth-first)
```

### ToT Evaluation Prompt

The LLM itself evaluates each thought step:

```
Given the problem and the current reasoning step, evaluate
whether this approach is:
  - "sure"       (clearly correct, continue)
  - "maybe"      (plausible, worth exploring)
  - "impossible" (clearly wrong, abandon)

This evaluation guides the search tree pruning.
```

---

## Self-Consistency

### Core Idea

Self-Consistency (Wang et al., 2022) generates **multiple reasoning chains** and selects the answer via **majority voting**, improving robustness.

```
                    ┌───────────┐
                    │  Prompt   │
                    └─────┬─────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
     ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
     │ Chain 1 │    │ Chain 2 │    │ Chain 3 │
     │ T=0.7   │    │ T=0.7   │    │ T=0.7   │
     └────┬────┘    └────┬────┘    └────┬────┘
          │               │               │
     Answer: 42      Answer: 42      Answer: 38
          │               │               │
          └───────────────┼───────────────┘
                          │
                    ┌─────┴─────┐
                    │  Majority │
                    │  Vote: 42 │
                    │  (2/3)    │
                    └───────────┘

Typically sample 5-20 chains with temperature 0.5-0.8.
Higher sample count → more robust but higher cost.
```

### Mathematical Formulation

```
P(answer = a) ≈ (1/N) × Σᵢ 𝟙[chain_i produces answer a]

Final answer = argmax_a  P(answer = a)

Confidence = P(answer = a*) = count(a*) / N

Example with 10 chains:
  Answer 42: 6 chains → P = 0.6
  Answer 38: 3 chains → P = 0.3
  Answer 45: 1 chain  → P = 0.1

  Final answer: 42 (confidence: 60%)
```

---

## ReAct: Reasoning + Acting

### Core Idea

ReAct (Yao et al., 2022) interleaves **reasoning traces** (thoughts) with **actions** (tool calls), allowing the LLM to plan, execute, and adjust dynamically.

```
┌──────────────────────────────────────────────────────────────────┐
│                    ReAct Loop                                     │
│                                                                   │
│  ┌─────────┐     ┌──────────┐     ┌─────────────┐                │
│  │ Thought │────→│  Action  │────→│ Observation │──┐             │
│  │(reason) │     │(execute) │     │  (result)   │  │             │
│  └─────────┘     └──────────┘     └─────────────┘  │             │
│       ▲                                             │             │
│       └─────────────────────────────────────────────┘             │
│                    (iterate until done)                            │
│                                                                   │
│  Example:                                                         │
│  Thought 1: I need to find the population of France.              │
│  Action 1:  Search("population of France 2024")                   │
│  Obs 1:     France has approximately 68.17 million people.        │
│  Thought 2: Now I need to find the population of Germany.         │
│  Action 2:  Search("population of Germany 2024")                  │
│  Obs 2:     Germany has approximately 84.48 million people.       │
│  Thought 3: I can now compare: 84.48M - 68.17M = 16.31M          │
│  Action 3:  Finish("Germany has 16.31 million more people.")      │
└──────────────────────────────────────────────────────────────────┘
```

---

## Tool Use and Function Calling

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 LLM with Tool Use                                │
│                                                                  │
│   User Query ──→ ┌──────────┐                                    │
│                  │   LLM    │──→ Tool Call Request                │
│                  │          │    {                                │
│                  │          │      "tool": "calculator",         │
│                  │          │      "args": {"expr": "15*7+3"}   │
│                  └──────────┘    }                                │
│                       ▲               │                          │
│                       │               ▼                          │
│                       │         ┌──────────┐                     │
│                       │         │  Tool    │                     │
│                       │         │ Executor │                     │
│                       │         └────┬─────┘                     │
│                       │              │                           │
│                       └──────────────┘                           │
│                    Tool Result: 108                               │
│                                                                  │
│   Available tools:                                               │
│   ┌─────────────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐        │
│   │ Calculator  │ │ Search  │ │ Database │ │ Code     │        │
│   │ API         │ │ Engine  │ │ Query    │ │ Executor │        │
│   └─────────────┘ └─────────┘ └──────────┘ └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### Function Definition Format (OpenAI-style)

```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Get current weather for a location",
    "parameters": {
      "type": "object",
      "properties": {
        "location": {
          "type": "string",
          "description": "City name, e.g., 'London, UK'"
        },
        "unit": {
          "type": "string",
          "enum": ["celsius", "fahrenheit"]
        }
      },
      "required": ["location"]
    }
  }
}
```

---

## Structured Output and Schema Enforcement

### JSON Mode

Force LLMs to produce valid, parseable JSON:

```
System: You are a data extraction assistant. Always respond
        in valid JSON matching the provided schema.

Schema: {
  "entities": [{"name": str, "type": str, "context": str}],
  "relationships": [{"subject": str, "predicate": str, "object": str}]
}

User: Extract entities from: "Apple CEO Tim Cook announced the
      iPhone 16 at the September 2024 event in Cupertino."