# Chapter 3.2 — LPS Array Construction

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Building the LPS array efficiently in O(m) is the clever part of KMP.
This chapter shows the algorithm step by step with detailed traces.

---

## 1. Naive Approach (for understanding)

```
  For each i from 0 to m-1:
      For each length L from i down to 1:
          if P[0..L-1] == P[i-L+1..i]:
              LPS[i] = L
              break
      else: LPS[i] = 0

  Time: O(m³)  ← too slow!
```

---

## 2. Efficient Construction — The Algorithm

```
  BUILD_LPS(P, m):
      LPS[0] ← 0
      len ← 0           // length of previous longest prefix-suffix
      i ← 1

      while i < m:
          if P[i] == P[len]:
              len ← len + 1
              LPS[i] ← len
              i ← i + 1
          else:
              if len ≠ 0:
                  len ← LPS[len - 1]    // ← KEY: fallback using LPS itself!
                  // do NOT increment i
              else:
                  LPS[i] ← 0
                  i ← i + 1
      return LPS
```

### 💡 Key Insight

```
  ┌──────────────────────────────────────────────────────┐
  │  When P[i] ≠ P[len] and len > 0:                    │
  │                                                      │
  │  We don't restart len from 0. Instead, we try the    │
  │  NEXT shorter prefix that is also a suffix:          │
  │     len = LPS[len - 1]                               │
  │                                                      │
  │  This is possible because if P[0..len-1] has a       │
  │  prefix-suffix of length LPS[len-1], that shorter    │
  │  prefix-suffix might still extend with P[i].         │
  └──────────────────────────────────────────────────────┘
```

---

## 3. Detailed Trace — P = "ABABCABAB"

```
  Initialize: LPS = [0, _, _, _, _, _, _, _, _]
              len = 0, i = 1

  ─────────────────────────────────────────────────────────
  i=1: P[1]='B' vs P[0]='A' → mismatch
       len == 0 → LPS[1] = 0, i = 2

       LPS = [0, 0, _, _, _, _, _, _, _]

  ─────────────────────────────────────────────────────────
  i=2: P[2]='A' vs P[0]='A' → match!
       len = 1, LPS[2] = 1, i = 3

       LPS = [0, 0, 1, _, _, _, _, _, _]

  ─────────────────────────────────────────────────────────
  i=3: P[3]='B' vs P[1]='B' → match!
       len = 2, LPS[3] = 2, i = 4

       LPS = [0, 0, 1, 2, _, _, _, _, _]

  ─────────────────────────────────────────────────────────
  i=4: P[4]='C' vs P[2]='A' → mismatch
       len = 2 ≠ 0 → len = LPS[1] = 0

  i=4: P[4]='C' vs P[0]='A' → mismatch
       len == 0 → LPS[4] = 0, i = 5

       LPS = [0, 0, 1, 2, 0, _, _, _, _]

  ─────────────────────────────────────────────────────────
  i=5: P[5]='A' vs P[0]='A' → match!
       len = 1, LPS[5] = 1, i = 6

       LPS = [0, 0, 1, 2, 0, 1, _, _, _]

  ─────────────────────────────────────────────────────────
  i=6: P[6]='B' vs P[1]='B' → match!
       len = 2, LPS[6] = 2, i = 7

       LPS = [0, 0, 1, 2, 0, 1, 2, _, _]

  ─────────────────────────────────────────────────────────
  i=7: P[7]='A' vs P[2]='A' → match!
       len = 3, LPS[7] = 3, i = 8

       LPS = [0, 0, 1, 2, 0, 1, 2, 3, _]

  ─────────────────────────────────────────────────────────
  i=8: P[8]='B' vs P[3]='B' → match!
       len = 4, LPS[8] = 4, i = 9

       LPS = [0, 0, 1, 2, 0, 1, 2, 3, 4]   ✓ DONE!
```

---

## 4. Trace with Fallback — P = "AABAABAAA"

```
  LPS = [0, _, _, _, _, _, _, _, _]
  len = 0, i = 1

  i=1: P[1]='A' vs P[0]='A' → match
       len=1, LPS[1]=1, i=2
       LPS = [0, 1, _, _, _, _, _, _, _]

  i=2: P[2]='B' vs P[1]='A' → mismatch
       len=1 → len = LPS[0] = 0
       P[2]='B' vs P[0]='A' → mismatch
       len=0 → LPS[2]=0, i=3
       LPS = [0, 1, 0, _, _, _, _, _, _]

  i=3: P[3]='A' vs P[0]='A' → match
       len=1, LPS[3]=1, i=4
       LPS = [0, 1, 0, 1, _, _, _, _, _]

  i=4: P[4]='A' vs P[1]='A' → match
       len=2, LPS[4]=2, i=5
       LPS = [0, 1, 0, 1, 2, _, _, _, _]

  i=5: P[5]='B' vs P[2]='B' → match
       len=3, LPS[5]=3, i=6
       LPS = [0, 1, 0, 1, 2, 3, _, _, _]

  i=6: P[6]='A' vs P[3]='A' → match
       len=4, LPS[6]=4, i=7
       LPS = [0, 1, 0, 1, 2, 3, 4, _, _]

  i=7: P[7]='A' vs P[4]='A' → match
       len=5, LPS[7]=5, i=8
       LPS = [0, 1, 0, 1, 2, 3, 4, 5, _]

  i=8: P[8]='A' vs P[5]='B' → mismatch ← FALLBACK!
       len=5 → len = LPS[4] = 2

       P[8]='A' vs P[2]='B' → mismatch
       len=2 → len = LPS[1] = 1

       P[8]='A' vs P[1]='A' → match!
       len=2, LPS[8]=2, i=9

       LPS = [0, 1, 0, 1, 2, 3, 4, 5, 2]   ✓

  The fallback chain at i=8:
  len: 5 → 2 → 1 → then match at len=1 → LPS[8]=2
```

---

## 5. Visualization of the Matching Process

```
  At i=8 of "AABAABAAA":

  Pattern:  A A B A A B A A A
            0 1 2 3 4 5 6 7 8

  At len=5, trying to extend P[0..4] = "AABAA":
  P[0..4]:   A A B A A
  P[4..8]:           A B A A A
                     ▲ mismatch (P[5]='B' ≠ P[8]='A')

  Fallback to len=LPS[4]=2, trying P[0..1] = "AA":
  P[0..1]:   A A
  P[7..8]:         A A
                   ▲ P[2]='B' ≠ P[8]='A' → mismatch

  Fallback to len=LPS[1]=1, trying P[0] = "A":
  P[0]:      A
  P[8]:              A
                     ▲ P[1]='A' == P[8]='A' → match!

  So LPS[8] = 1 + 1 = 2 ✓
```

---

## 6. Why Is This O(m)?

```
  ┌──────────────────────────────────────────────────────┐
  │  Amortized Analysis:                                 │
  │                                                      │
  │  Variable `len` tracks the current match length.     │
  │                                                      │
  │  • On match: len increases by 1, i increases by 1    │
  │  • On mismatch with len > 0: len DECREASES           │
  │  • On mismatch with len = 0: i increases by 1        │
  │                                                      │
  │  Key observation:                                    │
  │  • len can increase at most m times total            │
  │    (each increase by 1, i goes to m, so ≤ m times)  │
  │  • Each decrease makes len smaller                   │
  │  • len can decrease at most m times total            │
  │    (it can't go below 0)                             │
  │                                                      │
  │  Total operations ≤ 2m = O(m)                        │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Code (Python)

```python
def build_lps(pattern):
    m = len(pattern)
    lps = [0] * m
    length = 0  # length of previous longest prefix suffix
    i = 1
    
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        else:
            if length != 0:
                length = lps[length - 1]  # fallback
                # don't increment i
            else:
                lps[i] = 0
                i += 1
    
    return lps
```

---

## 📝 Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Two pointers: i scans pattern, len tracks prefix length |
| Match | len++, LPS[i] = len, i++ |
| Mismatch (len > 0) | len = LPS[len-1] (fallback) |
| Mismatch (len = 0) | LPS[i] = 0, i++ |
| Time | O(m) amortized |
| Space | O(m) for the LPS array |
| Key trick | Fallback uses LPS itself (bootstrapping) |

---

## ❓ Quick Revision Questions

1. **What happens when P[i] ≠ P[len] and len > 0?**
   <details><summary>Answer</summary>We set len = LPS[len-1] — fall back to the next shorter prefix-suffix and try again without advancing i.</details>

2. **Why do we NOT increment i during a fallback?**
   <details><summary>Answer</summary>Because we haven't processed position i yet — we're trying shorter prefix-suffixes that might still match at position i.</details>

3. **Compute the LPS array for "ABCABD".**
   <details><summary>Answer</summary>LPS = [0, 0, 0, 1, 2, 0]. "A" matches at i=3, "AB" at i=4, but "ABD" ≠ "ABC" so fallback gives 0 at i=5.</details>

4. **Why is the LPS construction O(m) and not worse?**
   <details><summary>Answer</summary>The variable `len` increases at most m times (once per i increment on match) and decreases at most m times (it stays ≥ 0). So total work ≤ 2m = O(m).</details>

5. **What is the maximum number of fallbacks at a single position i?**
   <details><summary>Answer</summary>At most O(m) theoretically, but amortized over all positions, total fallbacks ≤ m.</details>

---

| [⬅️ Previous: Failure Function](01-failure-function-lps-array.md) | [Next: Pattern Matching with KMP ➡️](03-pattern-matching-with-kmp.md) |
|:---|---:|
