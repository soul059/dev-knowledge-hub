# Chapter 1.2 — String Operations

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

This chapter catalogues every fundamental string operation, its time
complexity, and the mental model you need to reason about string problems
quickly. Mastering these operations is the foundation for every algorithm in
this course.

---

## 1. Core Operations & Complexity

```
  ┌──────────────────────────┬────────────┬──────────────────────────────┐
  │       Operation          │  Time      │  Notes                       │
  ├──────────────────────────┼────────────┼──────────────────────────────┤
  │ Access s[i]              │  O(1)      │  Direct array indexing       │
  │ Length |s|               │  O(1)      │  Stored as metadata          │
  │ Concatenation s + t      │  O(n + m)  │  Creates new string          │
  │ Substring s[i..j]        │  O(j - i)  │  Copy of range               │
  │ Comparison s == t        │  O(min(n,m))│  Char-by-char                │
  │ Search (find pattern)    │  O(n × m)  │  Brute force; O(n) with KMP  │
  │ Replace                  │  O(n)      │  Scan + construct new        │
  │ Reverse                  │  O(n)      │  In-place swap or new string │
  │ Sort characters          │  O(n log n)│  Or O(n) with counting sort  │
  │ Insert at position       │  O(n)      │  Shift elements right        │
  │ Delete at position       │  O(n)      │  Shift elements left         │
  └──────────────────────────┴────────────┴──────────────────────────────┘
```

---

## 2. Traversal Patterns

### 2.1 Left-to-Right Scan

```
  s = "abcdef"

  i ──►
  ┌───┬───┬───┬───┬───┬───┐
  │ a │ b │ c │ d │ e │ f │
  └───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5

  for i = 0 to n-1:
      process(s[i])
```

### 2.2 Right-to-Left Scan

```
                        ◄── i
  ┌───┬───┬───┬───┬───┬───┐
  │ a │ b │ c │ d │ e │ f │
  └───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5

  for i = n-1 downto 0:
      process(s[i])
```

### 2.3 Two-Pointer Scan

```
  i ──►             ◄── j
  ┌───┬───┬───┬───┬───┬───┐
  │ a │ b │ c │ c │ b │ a │
  └───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5

  Used for: palindrome check, reversal, partitioning
```

### 2.4 Sliding Window

```
  Window of size k = 3:

  Step 1:  [a  b  c] d  e  f     window = "abc"
  Step 2:   a [b  c  d] e  f     window = "bcd"
  Step 3:   a  b [c  d  e] f     window = "cde"
  Step 4:   a  b  c [d  e  f]    window = "def"

  Total windows = n - k + 1
```

---

## 3. Comparison Operations

### Lexicographic Order

Two strings are compared character by character using their ASCII/Unicode
values. The first mismatch determines the order.

```
  Compare "apple" vs "apply":

  a == a   ✓
  p == p   ✓
  p == p   ✓
  l == l   ✓
  e  < y   ◄── mismatch!  ('e' = 101) < ('y' = 121)

  ∴ "apple" < "apply"
```

### Comparison Rules

```
  1. Compare char by char from left.
  2. First mismatch → smaller char means smaller string.
  3. If one string is a prefix of the other:
       shorter string < longer string.
       e.g., "app" < "apple"
```

---

## 4. String Reversal

### In-Place Reversal (Two Pointers)

```
  Pseudocode:
  ──────────
  REVERSE(s):
      left ← 0
      right ← |s| - 1
      while left < right:
          SWAP(s[left], s[right])
          left ← left + 1
          right ← right - 1

  Trace for s = "hello":
  ─────────────────────
  Step 0:  h e l l o     left=0, right=4
  Step 1:  o e l l h     left=1, right=3   (swap h↔o)
  Step 2:  o l l e h     left=2, right=2   (swap e↔l)
  Done:    o l l e h     → "olleh"
```

**Time**: O(n/2) = O(n)  
**Space**: O(1) in-place

---

## 5. String Rotation

A rotation of string `s` by `k` positions:

```
  s = "abcdef",  k = 2

  Left rotation:   "cdefab"     (first k chars move to end)
  Right rotation:  "efabcd"     (last k chars move to front)
```

### 🔑 Rotation Trick: Reversal Algorithm

```
  Left-rotate by k:
  ──────────────────
  1. Reverse first k chars:   "abcdef" → "bacdef"
                                          ──
  2. Reverse remaining:       "bacdef" → "bafedc"
                                            ────
  3. Reverse entire string:   "bafedc" → "cdefab"  ✓
                               ──────
```

### 🔑 Rotation Check

**Problem**: Is `t` a rotation of `s`?

```
  s = "abcde"    t = "cdeab"

  Trick: t is a rotation of s  ⟺  t is a substring of s + s

  s + s = "abcdeabcde"
                ──────
  Find "cdeab" in "abcdeabcde" → Found at index 2 ✓
```

**Time**: O(n) using KMP on `s + s`

---

## 6. Case Conversion

```
  ┌──────────────────────────────────────────────┐
  │  Uppercase → Lowercase:  c = c | 32          │
  │  Lowercase → Uppercase:  c = c & ~32         │
  │  Toggle Case:            c = c ^ 32          │
  │                                              │
  │  Check lowercase:  'a' ≤ c ≤ 'z'            │
  │  Check uppercase:  'A' ≤ c ≤ 'Z'            │
  │  Check digit:      '0' ≤ c ≤ '9'            │
  └──────────────────────────────────────────────┘

  ASCII:  'A' = 65 = 0100_0001
          'a' = 97 = 0110_0001
                          ▲
                     bit 5 differs (32)
```

---

## 7. Frequency Counting

The most common string technique — used in anagrams, permutations, and
sliding window problems.

```
  Pseudocode:
  ──────────
  COUNT_FREQ(s):
      freq[26] ← {0, 0, ..., 0}
      for i = 0 to |s| - 1:
          freq[s[i] - 'a'] ← freq[s[i] - 'a'] + 1
      return freq

  Example: s = "abracadabra"
  ─────────────────────────
  freq[0] = 5   (a)
  freq[1] = 2   (b)
  freq[2] = 1   (c)
  freq[3] = 1   (d)
  freq[17] = 2  (r)
  All others = 0
```

**Time**: O(n)  |  **Space**: O(|Σ|) = O(26) = O(1)

---

## 8. String to Number / Number to String

### String → Integer

```
  s = "1234"

  result = 0
  for each char c in s:
      result = result × 10 + (c - '0')

  Trace:
    c='1':  result = 0×10 + 1 = 1
    c='2':  result = 1×10 + 2 = 12
    c='3':  result = 12×10 + 3 = 123
    c='4':  result = 123×10 + 4 = 1234
```

### Integer → String

```
  n = 1234

  digits = []
  while n > 0:
      digits.append(n % 10 + '0')
      n = n / 10
  reverse(digits)

  Trace:
    1234 % 10 = 4 → "4"       n = 123
     123 % 10 = 3 → "43"      n = 12
      12 % 10 = 2 → "432"     n = 1
       1 % 10 = 1 → "4321"    n = 0
  Reverse → "1234"
```

---

## 9. Tokenization / Splitting

```
  s = "hello world foo bar"
  delimiter = ' '

  Result: ["hello", "world", "foo", "bar"]

  ┌─────────────────────────────────────┐
  │ h e l l o   w o r l d   f o o   b  │
  │ ───────── ─ ───────── ─ ───── ─ ── │
  │  token1     token2      tok3   tok4 │
  └─────────────────────────────────────┘
```

**Time**: O(n) — single pass, split on delimiter

---

## 10. Common Patterns & Techniques

| Pattern | Use Case | Example Problem |
|---------|----------|-----------------|
| Frequency array | Anagram check | "Are two strings anagrams?" |
| Two pointers | Palindrome check | "Is this string a palindrome?" |
| Sliding window | Fixed-size substring | "Max sum substring of length k" |
| Prefix/Suffix | String matching | KMP, Z-algorithm |
| StringBuilder | Efficient concatenation | Build result string |
| Sorting chars | Grouping anagrams | Group anagram strings |

---

## 📝 Summary Table

| Operation | Time | Space | Key Insight |
|-----------|------|-------|-------------|
| Access `s[i]` | O(1) | O(1) | Array indexing |
| Concatenation | O(n+m) | O(n+m) | New string created |
| Reversal | O(n) | O(1) | Two-pointer swap |
| Rotation by k | O(n) | O(1) | Triple reversal trick |
| Rotation check | O(n) | O(n) | Check `t ⊆ s+s` |
| Frequency count | O(n) | O(|Σ|) | Array of size 26 |
| Comparison | O(n) | O(1) | Lex order, first mismatch |
| Case toggle | O(1) | O(1) | XOR with 32 |

---

## ❓ Quick Revision Questions

1. **What is the time complexity of checking if two strings are equal?**
   <details><summary>Answer</summary>O(min(n, m)) — compare char by char until mismatch or end.</details>

2. **How do you check if string `t` is a rotation of string `s` in O(n)?**
   <details><summary>Answer</summary>Check if `t` is a substring of `s + s` (both must have same length).</details>

3. **Why should you avoid `s += c` in a loop in Java/Python?**
   <details><summary>Answer</summary>Each concatenation copies the entire string → total work is O(1+2+...+n) = O(n²).</details>

4. **How do you convert lowercase to uppercase using bit manipulation?**
   <details><summary>Answer</summary>`c & ~32` or equivalently `c & 0xDF` — clears bit 5.</details>

5. **What is the time complexity of left-rotating a string by k positions using the reversal method?**
   <details><summary>Answer</summary>O(n) — three reversals, each O(n).</details>

6. **How many distinct substrings of length k exist in a string of length n?**
   <details><summary>Answer</summary>At most n - k + 1 substrings (some may be duplicates).</details>

---

| [⬅️ Previous: String Representation](01-string-representation.md) | [Next: Substrings ➡️](03-substrings.md) |
|:---|---:|
