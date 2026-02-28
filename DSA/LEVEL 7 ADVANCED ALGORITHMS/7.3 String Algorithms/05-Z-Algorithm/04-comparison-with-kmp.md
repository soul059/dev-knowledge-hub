# Chapter 5.4 — Z Algorithm vs KMP: A Detailed Comparison

> **Unit 5: Z Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Both Z Algorithm and KMP solve pattern matching in O(n + m), but they take
fundamentally different approaches. Understanding their trade-offs helps you
choose the right tool for each problem.

---

## 1. Side-by-Side Overview

```
  ┌──────────────────┬────────────────────┬────────────────────┐
  │ Aspect           │ Z Algorithm        │ KMP Algorithm      │
  ├──────────────────┼────────────────────┼────────────────────┤
  │ Preprocessing    │ Z-array of S       │ LPS array of P     │
  │ Core array       │ Z[i]: match with   │ LPS[i]: longest    │
  │                  │ prefix from pos i  │ proper prefix=suffix│
  │ Matching string  │ P + "$" + T        │ T (uses P's LPS)   │
  │ Time             │ O(n + m)           │ O(n + m)           │
  │ Space            │ O(n + m)           │ O(m)               │
  │ Online?          │ No (needs full S)  │ Yes (char by char) │
  │ Simplicity       │ Simpler            │ More complex       │
  └──────────────────┴────────────────────┴────────────────────┘
```

---

## 2. Conceptual Difference

```
  Z Algorithm:
  ────────────
  "How much of the prefix matches starting at each position?"
  
  S = "abc$xabcy"
  Z = [9, 0, 0, 0, 0, 3, 0, 0, 0]
       ^                ^
       entire string    "abc" matches prefix
  
  Direct: Z[i] == m → pattern found!
  Approach: Build full info, then scan.

  KMP Algorithm:
  ──────────────
  "When a mismatch occurs, where should I restart in the pattern?"
  
  LPS of "AABA" = [0, 1, 0, 1]
  
  Indirect: Tracks match progress with two pointers.
  Approach: Simulates a finite automaton.
```

---

## 3. Space Comparison

```
  Z Algorithm:
  ┌───────────────────────────────────────────────┐
  │  Must store: S = P + "$" + T   → O(n + m)    │
  │  Z-array of S                  → O(n + m)    │
  │  Total space: O(n + m)                        │
  └───────────────────────────────────────────────┘

  KMP Algorithm:
  ┌───────────────────────────────────────────────┐
  │  Must store: LPS array of P    → O(m)        │
  │  Text processed char-by-char   → O(1)        │
  │  Total space: O(m)                            │
  └───────────────────────────────────────────────┘

  When n >> m, KMP uses significantly less memory.
  Example: n = 10⁸, m = 100
    Z needs ~100 MB just for the Z-array
    KMP needs ~100 bytes for LPS
```

---

## 4. Online vs Offline

```
  ONLINE = can process text one character at a time,
           without knowing future characters.

  KMP is ONLINE:
  ┌──────────────────────────────────────────────────┐
  │  Characters arrive one at a time:                │
  │  A → A → B → A → A → C → ...                   │
  │  KMP updates its state with each character.      │
  │  Can report matches as soon as they complete.    │
  │  Works with: streams, network data, real-time.   │
  └──────────────────────────────────────────────────┘

  Z Algorithm is OFFLINE:
  ┌──────────────────────────────────────────────────┐
  │  Needs the entire concatenated string upfront.   │
  │  Cannot process a stream.                        │
  │  Must know both P and T before starting.         │
  └──────────────────────────────────────────────────┘
```

---

## 5. Conversion Between Z and LPS

```
  Z-array and LPS array encode similar information and can be
  converted between each other in O(n).

  Z → LPS:
  ─────────
  for i = 1 to n-1:
      if Z[i] > 0:
          for j = Z[i]-1 downto 0:
              if LPS[i+j] > 0: break
              LPS[i+j] = j + 1

  LPS → Z:
  ─────────
  (more complex, but possible in O(n))

  Example:
  S    = "aabaaab"
  Z    = [7, 1, 0, 2, 3, 1, 0]
  LPS  = [0, 1, 0, 1, 1, 2, 0]  ← (computed from Z or directly)
```

---

## 6. When to Use Which?

```
  ┌─────────────────────────────────┬──────────────────────────┐
  │ Use Z Algorithm When:          │ Use KMP When:            │
  ├─────────────────────────────────┼──────────────────────────┤
  │ • Quick implementation needed   │ • Memory is limited      │
  │ • Competitive programming       │ • Online/streaming data  │
  │ • Need all prefix matches info  │ • Need automaton model   │
  │ • Learning string algorithms    │ • Building larger systems│
  │ • One-shot matching problem     │ • Multiple texts, same P │
  │ • Need LCP of suffix with prefix│ • Low-level optimization │
  └─────────────────────────────────┴──────────────────────────┘
```

---

## 7. Performance Benchmark (Conceptual)

```
  For pattern matching specifically:

  Both: Exactly O(n + m) time
  
  In practice:
  ┌────────────────────────────────────────────────────────┐
  │  KMP:                                                  │
  │  - Fewer cache misses (processes T in order)           │
  │  - No extra string allocation                          │
  │  - Slightly faster for very large texts                │
  │                                                        │
  │  Z Algorithm:                                          │
  │  - Simpler inner loop                                  │
  │  - Fewer branches in practice                          │
  │  - Often faster for moderate sizes                     │
  │                                                        │
  │  Difference is marginal — both are excellent.          │
  └────────────────────────────────────────────────────────┘
```

---

## 8. Both Solving the Same Problem

```
  Find "ABA" in "ABABABABA"

  ═══ Z APPROACH ═══
  S = "ABA$ABABABABA"
  Z = [13,0,1,0,3,0,3,0,3,0,3,0,1]
  Z[i]==3 at i=4,6,8,10 → positions 0,2,4,6 in T

  ═══ KMP APPROACH ═══
  LPS of "ABA" = [0,0,1]
  
  Matching:        i:  0 1 2 3 4 5 6 7 8
  T =              A B A B A B A B A
  j state: 0→1→2→3→match@0, j=1→2→3→match@2,
           j=1→2→3→match@4, j=1→2→3→match@6

  Same result: positions [0, 2, 4, 6]

  Both O(n + m), same answer, different paths!
```

---

## 📝 Summary Table

| Feature | Z Algorithm | KMP |
|---------|-------------|-----|
| Time complexity | O(n + m) | O(n + m) |
| Space | O(n + m) | O(m) |
| Online processing | ✗ | ✓ |
| Implementation difficulty | Easy | Medium |
| Concatenation needed | Yes (P$T) | No |
| Core data structure | Z-array | LPS array |
| Best for | Competitive programming | Production systems |
| Prefix info | Z[i] = prefix match length | LPS[i] = prefix=suffix length |

---

## ❓ Quick Revision Questions

1. **Which algorithm uses less space and why?**
   <details><summary>Answer</summary>KMP uses O(m) space (only the LPS array). Z Algorithm needs O(n+m) because it must store the concatenated string P+"$"+T and its Z-array.</details>

2. **What does "online algorithm" mean?**
   <details><summary>Answer</summary>An online algorithm processes input one element at a time without needing to see future elements. KMP can process text characters as they arrive; Z requires the full text upfront.</details>

3. **Can Z-array be converted to LPS array?**
   <details><summary>Answer</summary>Yes, in O(n) time. Both arrays encode prefix-matching information in different forms and are mathematically related.</details>

4. **For competitive programming, which is generally preferred?**
   <details><summary>Answer</summary>Z Algorithm, because it's simpler to implement correctly under time pressure and the O(n+m) space is usually not a constraint.</details>

5. **In what scenario would KMP be clearly better than Z?**
   <details><summary>Answer</summary>Processing a large stream of text (e.g., network packets) where we can't store the entire text. KMP processes character-by-character with O(m) space, while Z can't work without the full text.</details>

---

| [⬅️ Previous: Pattern Matching with Z](03-pattern-matching-with-z.md) | [Next: Z Algorithm Use Cases ➡️](05-use-cases.md) |
|:---|---:|
