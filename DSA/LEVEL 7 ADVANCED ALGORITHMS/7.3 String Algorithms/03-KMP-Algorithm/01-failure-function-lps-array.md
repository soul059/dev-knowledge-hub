# Chapter 3.1 — Failure Function (LPS Array)

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Failure Function**, also called the **LPS (Longest Proper Prefix which
is also a Suffix) array**, is the heart of the KMP algorithm. Understanding
LPS is the key to understanding KMP.

---

## 1. What Is the LPS Array?

```
  🔑 For each position i in pattern P[0..m-1]:

     LPS[i] = length of the longest PROPER prefix of P[0..i]
              that is also a suffix of P[0..i].

  "Proper" means the prefix cannot be the entire string P[0..i] itself.
```

### Example

```
  P = "ABABCABAB"

  i   P[0..i]      Proper prefixes       Proper suffixes         Match         LPS[i]
  ─   ───────      ───────────────       ───────────────         ─────         ──────
  0   "A"          ""                    ""                      ""            0
  1   "AB"         "", "A"              "", "B"                 ""            0
  2   "ABA"        "", "A", "AB"        "", "A", "BA"           "A"           1
  3   "ABAB"       "", "A", "AB", "ABA" "", "B", "AB", "BAB"   "AB"          2
  4   "ABABC"      ... (none match)     ...                     ""            0
  5   "ABABCA"     ... "A"              ... "A"                 "A"           1
  6   "ABABCAB"    ... "AB"             ... "AB"                "AB"          2
  7   "ABABCABA"   ... "ABA"            ... "ABA"               "ABA"         3
  8   "ABABCABAB"  ... "ABAB"           ... "ABAB"              "ABAB"        4

  LPS = [0, 0, 1, 2, 0, 1, 2, 3, 4]
```

---

## 2. Visual Intuition

```
  P = "ABABCABAB"

  For i=8 (full pattern):

  A B A B C A B A B
  ├───────┤             ← prefix "ABAB" (length 4)
                ├───────┤ ← suffix "ABAB" (length 4)

  These match!   So LPS[8] = 4.

  For i=7:

  A B A B C A B A
  ├─────┤             ← prefix "ABA" (length 3)
              ├─────┤ ← suffix "ABA" (length 3)

  Match! LPS[7] = 3.

  For i=4 ("ABABC"):

  A B A B C
  No proper prefix matches any suffix → LPS[4] = 0.
```

---

## 3. Why Is LPS Important?

```
  ┌──────────────────────────────────────────────────────┐
  │  💡 KEY INSIGHT                                      │
  │                                                      │
  │  When a mismatch occurs at P[j] during matching:     │
  │                                                      │
  │  We've successfully matched P[0..j-1] with some      │
  │  substring of T. The LPS tells us the longest        │
  │  prefix of P that is also a suffix of the matched    │
  │  portion — so we can SKIP to that prefix without     │
  │  re-reading characters in T.                         │
  │                                                      │
  │  Next j = LPS[j-1]  (not 0!)                        │
  │                                                      │
  │  This is what makes KMP O(n + m) instead of O(n×m). │
  └──────────────────────────────────────────────────────┘

  Mismatch at j=4:

  T: ... A B A B X ...
  P:     A B A B C
                 ▲ mismatch

  Matched "ABAB". LPS[3] = 2, so "AB" is both prefix and suffix.

  Instead of starting over:
  T: ... A B A B X ...
  P:         A B A B C      ← resume comparison at j=2
                 ▲
                 We know P[0..1] = "AB" already matches T here!
```

---

## 4. LPS Array Properties

```
  ┌─────────────────────────────────────────────────────┐
  │  Property 1: LPS[0] = 0  always                    │
  │              (single char has no proper prefix)     │
  │                                                     │
  │  Property 2: 0 ≤ LPS[i] ≤ i                       │
  │              (can't be longer than the string)      │
  │                                                     │
  │  Property 3: LPS[i] ≤ LPS[i-1] + 1               │
  │              (can increase by at most 1)            │
  │                                                     │
  │  Property 4: If LPS[i] = k, then LPS[i] can       │
  │              "fall back" to LPS[k-1] on mismatch.  │
  │              This creates a chain of fallbacks.     │
  └─────────────────────────────────────────────────────┘
```

### Fallback Chain Visualization

```
  LPS for "ABABCABAB":  [0, 0, 1, 2, 0, 1, 2, 3, 4]

  If mismatch at j=8 (after matching full pattern? no — during construction):

  Fallback chain from j=4 (LPS[3]=2):
    j=4 → LPS[3]=2 → j=2
    j=2 → LPS[1]=0 → j=0
    j=0 → done (no more fallback)

  This chain is: 4 → 2 → 0
  Each step represents: "try the next-longest prefix that is a suffix"
```

---

## 5. More Examples

### Example 1: "AAAA"

```
  i   P[0..i]   Longest prefix = suffix   LPS[i]
  0   "A"       (none)                    0
  1   "AA"      "A"                       1
  2   "AAA"     "AA"                      2
  3   "AAAA"    "AAA"                     3

  LPS = [0, 1, 2, 3]
```

### Example 2: "ABCABC"

```
  i   P[0..i]    Longest prefix = suffix   LPS[i]
  0   "A"        (none)                    0
  1   "AB"       (none)                    0
  2   "ABC"      (none)                    0
  3   "ABCA"     "A"                       1
  4   "ABCAB"    "AB"                      2
  5   "ABCABC"   "ABC"                     3

  LPS = [0, 0, 0, 1, 2, 3]
```

### Example 3: "AABAABAAA"

```
  i   P[0..i]      LPS[i]   Explanation
  0   "A"          0        —
  1   "AA"         1        prefix "A" = suffix "A"
  2   "AAB"        0        no match
  3   "AABA"       1        "A" = "A"
  4   "AABAA"      2        "AA" = "AA"
  5   "AABAAB"     3        "AAB" = "AAB"
  6   "AABAABA"    4        "AABA" = "AABA"
  7   "AABAABAA"   5        "AABAA" = "AABAA"
  8   "AABAABAAA"  2        Fall back! "AA" = "AA"

  LPS = [0, 1, 0, 1, 2, 3, 4, 5, 2]
```

---

## 6. Connection to Finite Automata

```
  The LPS array defines a deterministic finite automaton (DFA):

  State = number of characters matched so far (j)

  Transition on match:     j → j + 1
  Transition on mismatch:  j → LPS[j - 1]  (fallback)

  For P = "ABABC":

  State diagram:
  ──────────────
         A       B       A       B       C
  (0) ──────► (1) ──────► (2) ──────► (3) ──────► (4) ──────► (5) MATCH!
   ▲           │           │           │           │
   │    not A  │   not B   │   not A   │   not B   │  not C
   └───────────┘     │     │     │     │     │     │
                     │     └──►(1)     └──►(1)     └──►(2)
                     └──►(0)          via LPS       via LPS

  LPS guides the fallback transitions!
```

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| LPS[i] | Longest proper prefix of P[0..i] that is also a suffix |
| LPS[0] | Always 0 |
| Range | 0 ≤ LPS[i] ≤ i |
| Purpose | Tells KMP where to resume after mismatch |
| Fallback chain | LPS[j-1] → LPS[LPS[j-1]-1] → ... → 0 |
| Automaton view | LPS defines mismatch transitions in a DFA |

---

## ❓ Quick Revision Questions

1. **What does LPS[i] represent?**
   <details><summary>Answer</summary>The length of the longest proper prefix of P[0..i] that is also a suffix of P[0..i].</details>

2. **Why must the prefix be "proper"?**
   <details><summary>Answer</summary>A non-proper prefix would be the entire string itself, which is trivially equal to itself and provides no useful information for skipping.</details>

3. **What is LPS[0] for any pattern?**
   <details><summary>Answer</summary>Always 0 — a single character has no proper prefix.</details>

4. **Compute LPS for "ABAB".**
   <details><summary>Answer</summary>LPS = [0, 0, 1, 2]. "A" is a prefix-suffix at i=2, "AB" at i=3.</details>

5. **How does LPS help avoid redundant comparisons?**
   <details><summary>Answer</summary>After a mismatch at P[j], instead of restarting from P[0], we jump to P[LPS[j-1]], skipping characters we already know match because the prefix equals the suffix of the matched portion.</details>

---

| [⬅️ Previous Unit: Problem Variations](../02-Pattern-Matching/04-problem-variations.md) | [Next: LPS Array Construction ➡️](02-lps-array-construction.md) |
|:---|---:|
