# Chapter 8.2 — Transformed String

> **Unit 8: Manacher's Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Manacher's algorithm requires a **transformation** that inserts special
separator characters between every character. This unifies odd-length
and even-length palindromes into a single case.

---

## 1. The Even/Odd Problem

```
  Without transformation, we need two separate expansion loops:

  Odd:  "aba"   → center at 'b' (index 1)
  Even: "abba"  → center between 'b' and 'b' (index 1.5??)

  Problem: indices must be integers!
  Solution: INSERT separators to make ALL palindromes odd-length.
```

---

## 2. The Transformation

```
  Original:     a  b  c  b  a
  Transformed:  # a # b # c # b # a #

  Original:     a  b  b  a
  Transformed:  # a # b # b # a #

  Rules:
  1. Insert '#' between every pair of characters
  2. Add '#' at the beginning and end
  3. Optionally add sentinels: ^...$ (for bounds)

  Full transform: "abba" → "^#a#b#b#a#$"
```

---

## 3. Why It Works

```
  Every even-length palindrome in the original becomes
  an odd-length palindrome centered at a '#':

  Original "abba":
  ┌────────────────────────────────────────┐
  │  "abba" is even, center between b,b   │
  │                                        │
  │  Transformed: # a # b # b # a #       │
  │  Index:       0 1 2 3 4 5 6 7 8       │
  │                                        │
  │  Now center at index 4 ('#'):          │
  │  #a#b # b#a#                           │
  │      ↑ center                          │
  │  Palindrome radius = 4                 │
  └────────────────────────────────────────┘

  Original "aba" (odd):
  ┌────────────────────────────────────────┐
  │  Transformed: # a # b # a #           │
  │  Index:       0 1 2 3 4 5 6           │
  │                                        │
  │  Center at index 3 ('b'):             │
  │  #a# b #a#                            │
  │      ↑ center                          │
  │  Palindrome radius = 3                │
  └────────────────────────────────────────┘

  In both cases: original_length = radius
```

---

## 4. Index Mapping

```
  Transformed T:  ^ # a # b # b # a # $
  Index:          0 1 2 3 4 5 6 7 8 9 10

  Mapping from transformed index i to original index:
    original_index = (i - 1) / 2    (for character positions)

  Character positions in T: indices 2, 4, 6, 8 → original 0, 1, 2, 3
  Separator positions in T: indices 1, 3, 5, 7, 9

  Transformed length = 2n + 3  (with sentinels ^, $)
  Or simply:          2n + 1  (without sentinels)

  Recovering original palindrome from transformed:
  ┌─────────────────────────────────────────────┐
  │  If center = c in transformed, radius = r:  │
  │  Original start = (c - r) / 2              │
  │  Original length = r                        │
  └─────────────────────────────────────────────┘
```

---

## 5. Transform Implementation

```python
def transform(s: str) -> str:
    """Transform 'abc' → '^#a#b#c#$'"""
    return '^#' + '#'.join(s) + '#$'

# Examples:
# transform("aba")  → "^#a#b#a#$"
# transform("abba") → "^#a#b#b#a#$"
# transform("a")    → "^#a#$"
# transform("")     → "^#$"
```

```
  Alternative without sentinels (simpler in some implementations):

  def transform_simple(s):
      return '#' + '#'.join(s) + '#'

  "abba" → "#a#b#b#a#"
  Length: 2n + 1

  Sentinels (^ and $) prevent out-of-bounds checks
  during Manacher's expansion — recommended.
```

---

## 6. Properties of Transformed String

```
  ┌──────────────────────────────────────────────────────┐
  │  Property 1: Length is always ODD                    │
  │  (2n+1 without sentinels, 2n+3 with)                │
  │                                                       │
  │  Property 2: Characters alternate between            │
  │  '#' and original chars                              │
  │  T = #, c₀, #, c₁, #, c₂, #, ..., cₙ₋₁, #         │
  │                                                       │
  │  Property 3: ALL palindromes have odd length in T    │
  │  → Single expansion loop handles everything!         │
  │                                                       │
  │  Property 4: Palindrome radius in T = length in S    │
  │  → Direct conversion without extra calculation       │
  │                                                       │
  │  Property 5: Sentinels ^ and $ never match any char  │
  │  → Natural boundary for expansion                    │
  └──────────────────────────────────────────────────────┘
```

---

## 7. Visualization

```
  Original:     "racecar"

  Transformed:  ^ # r # a # c # e # c # a # r # $
  Index:        0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16

  Palindrome centers in T:
    Center 8 ('e'): radius 7 → palindrome in S has length 7
      T[1..15] = #r#a#c#e#c#a#r#  is palindrome
      S[0..6]  = "racecar"

    Center 6 ('c'): radius 1 → palindrome in S has length 1
      T[5..7] = #c#  is palindrome (just "c")

    Center 5 ('#'): radius 0 → even palindrome of length 0
      Just the '#' itself.

  Original recovery:
    center=8, radius=7
    start_in_S = (8 - 7) / 2 = 0  (integer division)
    length_in_S = 7
    → S[0:7] = "racecar" ✓
```

---

## 📝 Summary Table

| Aspect | Without Transform | With Transform |
|--------|-------------------|----------------|
| Palindrome types | Odd + Even (2 cases) | Only Odd (1 case) |
| Centers to check | 2n - 1 | 2n + 1 (all indices in T) |
| Center index type | Integer or half-integer | Always integer |
| Expansion logic | Two separate loops | Single loop |
| Original length | depends on case | = radius in T |
| Transformed length | — | 2n + 1 (or 2n + 3 with sentinels) |

---

## ❓ Quick Revision Questions

1. **Why is the transformation needed?**
   <details><summary>Answer</summary>It converts even-length palindromes into odd-length ones by inserting '#' separators, allowing a single unified expansion loop. Without it, even palindromes have "virtual" centers between characters.</details>

2. **What are the sentinel characters ^ and $ for?**
   <details><summary>Answer</summary>They serve as boundary markers that never match any other character. This eliminates the need for explicit bounds checking during palindrome expansion in Manacher's algorithm.</details>

3. **How do you convert a center and radius in T back to the original string?**
   <details><summary>Answer</summary>If center = c and radius = r in the transformed string: original start = (c - r) / 2, original length = r. The palindrome in S is S[start : start + length].</details>

4. **What is the length of the transformed string?**
   <details><summary>Answer</summary>2n + 1 without sentinels, 2n + 3 with sentinels (^ and $), where n is the length of the original string.</details>

5. **Why does radius in T equal palindrome length in S?**
   <details><summary>Answer</summary>In the transformed string, each original character occupies one position and each '#' occupies one position. The radius spans r characters on each side, which corresponds to exactly r original characters (interleaved with separators).</details>

---

| [⬅️ Previous: Longest Palindromic Substring](01-longest-palindromic-substring.md) | [Next: Algorithm Steps ➡️](03-algorithm-steps.md) |
|:---|---:|
