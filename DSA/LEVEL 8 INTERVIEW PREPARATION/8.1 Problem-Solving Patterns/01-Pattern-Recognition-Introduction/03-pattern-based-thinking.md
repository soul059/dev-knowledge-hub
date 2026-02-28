# Chapter 3: Pattern-Based Thinking

## 📋 Chapter Overview
Develop the **mental framework** for approaching any problem using patterns. Learn to think in templates, recognize structural similarities between problems, and build a systematic solving methodology.

---

## 🧠 The Pattern-Based Mindset

```
  ┌─────────────────────────────────────────────────────────┐
  │             NOVICE vs EXPERT THINKING                   │
  │                                                         │
  │   NOVICE:                                               │
  │   "What's the trick for this specific problem?"         │
  │        │                                                │
  │        ▼                                                │
  │   Tries random approaches → Wastes 30+ minutes          │
  │                                                         │
  │   EXPERT:                                               │
  │   "What pattern does this problem belong to?"           │
  │        │                                                │
  │        ▼                                                │
  │   Applies template → Solves in 10-15 minutes            │
  └─────────────────────────────────────────────────────────┘
```

---

## 🔄 The 5-Step Pattern Matching Process

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │   ┌──────────┐                                       │
  │   │ 1. READ  │  Understand what is asked             │
  │   └────┬─────┘                                       │
  │        │                                             │
  │        ▼                                             │
  │   ┌──────────┐                                       │
  │   │2. EXTRACT│  Pull out keywords + constraints      │
  │   └────┬─────┘                                       │
  │        │                                             │
  │        ▼                                             │
  │   ┌──────────┐                                       │
  │   │ 3. MATCH │  Map to known pattern(s)              │
  │   └────┬─────┘                                       │
  │        │                                             │
  │        ▼                                             │
  │   ┌──────────┐                                       │
  │   │ 4. APPLY │  Use pattern template                 │
  │   └────┬─────┘                                       │
  │        │                                             │
  │        ▼                                             │
  │   ┌──────────┐                                       │
  │   │5. VERIFY │  Test with examples + edge cases      │
  │   └──────────┘                                       │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

### Detailed Breakdown:

| Step | Action | What You Do |
|------|--------|-------------|
| **1. Read** | Parse the problem | Identify input, output, constraints |
| **2. Extract** | Find signal words | "sorted", "contiguous", "shortest", "all" |
| **3. Match** | Compare with patterns | Check against your pattern library |
| **4. Apply** | Code the template | Write the skeleton, then fill details |
| **5. Verify** | Test & debug | Run through examples, check edge cases |

---

## 🧩 Structural Similarity Between Problems

Problems that *look* different often share the *same* structure:

```
  ┌─────────────────────────────────────────────────────┐
  │          SAME PATTERN, DIFFERENT SKIN               │
  │                                                     │
  │  Problem A: "Maximum sum subarray of size K"        │
  │  Problem B: "Maximum average subarray of size K"    │
  │  Problem C: "Count subarrays with exactly K ones"   │
  │                                                     │
  │  All three → SLIDING WINDOW (Fixed Size)            │
  │                                                     │
  │  ┌─────────────────────────────────────────────┐    │
  │  │    [  a  b  c  d  e  f  g  h  i  j  ]      │    │
  │  │    ├──────────┤                              │    │
  │  │    Window of K     ──► slide right           │    │
  │  │       ├──────────┤                           │    │
  │  │          ├──────────┤                        │    │
  │  └─────────────────────────────────────────────┘    │
  │                                                     │
  │  The STRUCTURE is identical.                        │
  │  Only the OPERATION inside the window changes.      │
  └─────────────────────────────────────────────────────┘
```

### Another Example:

```
  ┌─────────────────────────────────────────────────────┐
  │  Problem A: "Two sum in sorted array"               │
  │  Problem B: "Container with most water"             │
  │  Problem C: "Valid palindrome"                      │
  │                                                     │
  │  All three → TWO POINTERS (Opposite Direction)      │
  │                                                     │
  │  ┌───────────────────────────────────────────┐      │
  │  │    [  a  b  c  d  e  f  g  h  i  j  ]    │      │
  │  │    L──►                          ◄──R     │      │
  │  │       L──►                    ◄──R        │      │
  │  │          L──►              ◄──R           │      │
  │  └───────────────────────────────────────────┘      │
  │                                                     │
  │  Different problems, same pointer movement!         │
  └─────────────────────────────────────────────────────┘
```

---

## 🎯 The Template-First Approach

Instead of coding from scratch, start with a template:

```
  TEMPLATE-FIRST METHODOLOGY

  Step 1: Write the pattern skeleton
  ┌────────────────────────────────────┐
  │  function solve(input):            │
  │      // pattern-specific setup     │
  │      // main loop (pattern core)   │
  │      // return result              │
  └────────────────────────────────────┘

  Step 2: Fill in problem-specific logic
  ┌────────────────────────────────────┐
  │  function solve(input):            │
  │      window_sum = 0                │ ← specific to this problem
  │      for i in range(len(input)):   │ ← from template
  │          window_sum += input[i]    │ ← specific
  │          if i >= k:                │ ← from template
  │              window_sum -= input[  │ ← specific
  │                  i - k]            │
  │              max_sum = max(...)     │ ← specific
  │      return max_sum                │
  └────────────────────────────────────┘
```

### Example: Sliding Window Template

```
PSEUDOCODE — Generic Sliding Window Template:

function slidingWindow(arr, condition):
    left = 0
    result = initial_value
    state = {}  // tracks window state

    for right = 0 to len(arr) - 1:
        // EXPAND: add arr[right] to window state
        updateState(state, arr[right])

        // SHRINK: if window violates condition
        while not valid(state, condition):
            removeFromState(state, arr[left])
            left += 1

        // UPDATE: record best answer
        result = best(result, right - left + 1)

    return result
```

---

## 🔗 Pattern Connection Map

Patterns are not isolated — they connect and combine:

```
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │  Hash    │────►│ Sliding  │────►│ Two      │
  │  Map     │     │ Window   │     │ Pointers │
  └──────────┘     └──────────┘     └──────────┘
       │                │                │
       │                │                │
       ▼                ▼                ▼
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Prefix   │     │ Monotonic│     │ Binary   │
  │ Sum      │     │ Stack    │     │ Search   │
  └──────────┘     └──────────┘     └──────────┘
                        │
                        ▼
                   ┌──────────┐     ┌──────────┐
                   │  Greedy  │────►│    DP    │
                   └──────────┘     └──────────┘
                                         │
                                         ▼
                                    ┌──────────┐
                                    │ State    │
                                    │ Machine  │ 
                                    └──────────┘
```

### Common Combinations:

| Combination | Example Problem |
|-------------|----------------|
| Binary Search + Sliding Window | "Minimum window with sum ≥ target" |
| Hash Map + Sliding Window | "Longest substring with K distinct chars" |
| BFS + Hash Map | "Word ladder" |
| DP + Binary Search | "Longest increasing subsequence O(n log n)" |
| Greedy + Heap | "Meeting rooms — minimum conference rooms" |
| Two Pointers + Sorting | "Three Sum" |

---

## 🧪 Thinking Trace: Solving a Problem Step-by-Step

**Problem:** "Given a string, find the length of the longest substring without repeating characters."

### Mental Process:

```
  1. READ:
     Input: string
     Output: integer (length)
     Goal: longest substring, no repeats

  2. EXTRACT SIGNALS:
     ✓ "substring" → contiguous → WINDOW
     ✓ "longest" → maximize → VARIABLE WINDOW
     ✓ "without repeating" → condition to maintain

  3. MATCH PATTERN:
     Contiguous + maximize + condition = SLIDING WINDOW (variable)

  4. APPLY TEMPLATE:
     left = 0, right = 0
     Use a set/hashmap to track characters in window
     Expand right
     If duplicate found → shrink from left
     Track max(right - left + 1)

  5. VERIFY:
     Input: "abcabcbb"
     
     right=0: 'a' → set={a}, len=1, max=1
     right=1: 'b' → set={a,b}, len=2, max=2
     right=2: 'c' → set={a,b,c}, len=3, max=3
     right=3: 'a' → DUPLICATE! shrink left
              left=1 → remove 'a', set={b,c,a}, len=3, max=3
     right=4: 'b' → DUPLICATE! shrink left
              left=2 → remove 'b', set={c,a,b}, len=3, max=3
     right=5: 'c' → DUPLICATE! shrink left
              left=3 → remove 'c', set={a,b,c}, len=3, max=3
     right=6: 'b' → DUPLICATE! shrink left
              left=4 → remove 'a', still dup
              left=5 → remove 'b', set={c,b}, len=2
     right=7: 'b' → DUPLICATE! shrink...

     Answer: 3 ✓
```

---

## 🎓 Building Your Pattern Library

```
  ┌─────────────────────────────────────────────────┐
  │          YOUR PATTERN LIBRARY                   │
  │                                                 │
  │  For each pattern, store:                       │
  │                                                 │
  │  ┌─────────────────────────────────────┐        │
  │  │ NAME: Sliding Window               │        │
  │  │ TRIGGER: subarray, substring, contig│        │
  │  │ TEMPLATE: left/right pointer loop   │        │
  │  │ VARIANTS: fixed, variable, with map │        │
  │  │ COMPLEXITY: O(n) time, O(k) space   │        │
  │  │ EXAMPLES: Max sum K, longest substr │        │
  │  │ GOTCHAS: off-by-one, empty input    │        │
  │  └─────────────────────────────────────┘        │
  │                                                 │
  │  Repeat for all 12 patterns!                    │
  └─────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| 5-Step Process | Read → Extract → Match → Apply → Verify |
| Structural Similarity | Different problems can share the same pattern |
| Template-First | Always start with the pattern skeleton |
| Pattern Combinations | Patterns often combine (e.g., BinSearch + DP) |
| Signal Extraction | Keywords in the problem statement map to patterns |
| Pattern Library | Build a mental catalog of triggers, templates, and variants |

---

## ❓ Quick Revision Questions

1. **What are the 5 steps of pattern matching?**
   > Read → Extract keywords → Match pattern → Apply template → Verify with examples.

2. **Why do "container with most water" and "two sum (sorted)" use the same pattern?**
   > Both use Two Pointers (opposite direction) — they share the same structure of moving pointers inward based on a comparison.

3. **What does "Template-First" mean?**
   > Start with the pattern skeleton code and fill in problem-specific logic, rather than coding from scratch.

4. **Name two common pattern combinations.**
   > Binary Search + Sliding Window, Hash Map + Sliding Window, DP + Binary Search, etc.

5. **How should you build your pattern library?**
   > For each pattern, record: Name, Trigger keywords, Template code, Variants, Complexity, Example problems, and Common gotchas.

6. **When you see "longest substring" in a problem, what pattern should you think of first?**
   > Sliding Window (variable size).

---

[← Previous: Identifying Problem Types](02-identifying-problem-types.md) | [Next: Building Intuition →](04-building-intuition.md)

[← Back to README](../README.md)
